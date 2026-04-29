## 4.8 并发编程的一些经验法则

我们在上一节中描述的 dining philosophers problem（就餐哲学家问题）的解决方案，暗示了一些可以应用到几乎任何并发编程问题上的通用原则和经验法则。下面我们简要看几个例子。

### 4.8.1 全局排序规则

在并发程序中，事件发生的 order（顺序）并不像单线程程序那样由程序中指令的顺序决定。如果并发系统中需要 ordering（排序/顺序约束），那么这种 ordering 必须在所有线程之间 globally（全局地）施加。

这也是 doubly linked list（双向链表）并不是并发数据结构的原因之一。双向链表之所以被设计成那样，是为了支持在链表任意位置快速插入和删除元素。这个设计背后隐含了一个假设：program order（程序顺序）等价于 data order（数据顺序）——也就是说，当我们对链表执行一组有序操作时，最终得到的链表也会包含相应顺序的元素。例如，假设我们有一个链表，其中包含有序元素 `{ A, B, C }`。如果我们执行下面两个操作：

1. 在 C 前插入 D，

2. 在 C 前插入 E，

那么默认假设是，最终链表会包含有序元素 `{ A, B, D, E, C }`。

在单线程程序中，这个假设成立。但在多线程系统中，program order 不再决定 data order。如果一个线程执行“在 C 前插入 D”，另一个线程执行“在 C 前插入 E”，那么我们就有一个 race condition（竞争条件），它可能导致以下任意一种结果：

- `{ A, B, D, E, C }`，
- `{ A, B, E, D, C }`，或
- `{ A, B, corrupted data }`。

如果数据结构的操作没有被 critical sections（临界区）正确保护，就可能产生 corrupted list（损坏的链表）。例如，D 和 E 的 “next” 指针最终可能都会指向 C。

global ordering rule（全局排序规则）是这个问题唯一可行的解决方案。我们首先需要问自己：链表中元素的顺序为什么重要？它是否真的重要？如果顺序不重要，我们可以使用 singly-linked list（单向链表），并且总是把元素插入到表头——这种操作无论使用锁还是以 lock-free（无锁）方式实现，都可以可靠完成。而如果确实需要 global ordering，我们就需要找到一种 stable（稳定）且 deterministic（确定性）的排序标准，并且这个标准不能依赖程序事件恰好发生的顺序。例如，我们可以按字母顺序、按优先级，或按其他有用标准对链表进行排序。对这些问题的回答，反过来会决定我们想要使用哪种数据结构。试图以并发方式使用双向链表（也就是让多个线程可变地访问该链表），就像把方钉塞进圆孔一样不合适。

### 4.8.2 基于事务的算法

在 dining philosophers problem 的 central arbiter（中央仲裁者）解决方案中，arbiter 或 “waiter”（服务员）会成对发放筷子：哲学家要么获得所需的全部资源（两根筷子），要么什么也得不到。这称为 transaction（事务）。

更准确地说，transaction 可以定义为一个不可分割的资源和/或操作包。并发系统中的线程会向某种 central arbiter 提交 transaction request（事务请求）。一个 transaction 要么整体成功，要么整体失败（例如，当请求到达时，另一个线程的 transaction 正在被处理）。如果 transaction 失败，该线程就会持续重新提交 transaction request，直到它成功为止（重试之间可能会短暂等待）。

Transaction-based algorithms（基于事务的算法）在并发和分布式系统编程中非常常见。而且正如我们将在第 4.9 节中看到的，transaction 这一概念构成了大多数 lock-free data structures（无锁数据结构）和 lock-free algorithms（无锁算法）的基础。

### 4.8.3 最小化竞争

最高效的并发系统，是所有线程都不必等待锁就能运行的系统。当然，这种理想状态永远无法完全实现，但并发系统程序员确实会尽力最小化线程之间的 contention（竞争）。

例如，考虑一组线程正在生成数据，并把数据存储到一个 central repository（中央仓库）中。每当其中一个线程尝试把自己的数据存入仓库时，它都要与所有其他线程竞争这个共享资源。一个有时可行的简单解决方案，是给每个线程自己的 private repository（私有仓库）。这样线程就可以彼此独立地生成数据，没有 contention。当所有线程都完成输出生成后，再由一个 central thread（中央线程）汇总结果。

在 dining philosophers problem 中，与这种方法对应的方案，是一开始就给每位哲学家两根筷子。这样做当然会从问题中移除所有 concurrency（并发）——没有任何共享资源，就没有并发。在真实世界的并发系统中，我们不可能移除所有资源共享，但我们当然可以寻找方法来 minimize resource sharing（最小化资源共享），从而 minimize lock contention（最小化锁竞争）。

### 4.8.4 线程安全

一般来说，如果一个 class（类）或 functional API（函数式 API）中的函数可以被多线程进程中的任何线程安全调用，那么这个类或 API 就被称为 thread-safe（线程安全）。对于任意一个函数，thread safety（线程安全）通常通过在函数体顶部进入一个 critical section，执行一些工作，然后在返回前离开 critical section 来实现。一个所有函数都是 thread-safe 的类，有时称为 monitor。（术语 “monitor” 也用来指一种类：它内部使用 condition variables（条件变量），允许客户端在等待其受保护资源变为可用时进入睡眠。）

Thread-safety 是类或接口可以提供的一个方便特性。但它也会引入有时并不必要的开销。例如，如果某个接口被用在单线程程序中，或者在多线程程序中只被一个线程独占使用，那么这种开销就是浪费。

当一个 interface function（接口函数）需要被 reentrantly（可重入地）调用时，thread-safety 也可能成为问题。例如，如果一个类提供两个 thread-safe 函数 `A()` 和 `B()`，那么这两个函数不能相互调用，因为它们各自都会独立进入和离开 critical section。这个问题的一种解决方案是使用 reentrant locks（可重入锁，见第 4.9.7.3 节）。另一种做法是在接口中实现这些函数的 “unsafe”（不安全）版本，然后把每个函数包装成 thread-safe variant（线程安全变体）：包装版本只负责进入 critical section，调用对应的 “unsafe” 函数，然后离开 critical section。这样，我们就可以在系统内部调用 “unsafe” 函数，而系统的外部接口仍保持 thread-safe。

在我看来，试图仅仅通过创建 100% thread-safe 的类和 API 来处理并发编程，是一个糟糕的想法。这样做会导致接口不必要地 heavyweight（重量级），并鼓励程序员忽视自己正在并发环境中工作的事实。相反，我们应该承认并接受软件中的 concurrency，并致力于设计能够显式处理这种 concurrency 的数据结构和算法。目标应当是生成一个尽量减少线程之间 contention 和 dependencies（依赖），并尽量减少锁使用的软件系统。实现一个完全 lock-free 的系统需要大量工作，而且很难正确完成。（见第 4.9 节。）但朝着 lock-freedom（无锁化）的方向努力，也就是最小化锁的使用，比过度使用锁、徒劳地试图创建能让程序员完全不必意识到并发存在的 “water-tight”（滴水不漏）接口，要好得多。