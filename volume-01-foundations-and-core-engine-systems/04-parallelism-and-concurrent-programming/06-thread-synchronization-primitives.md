## 4.6 线程同步原语

每个支持并发的操作系统都会提供一套称为 thread synchronization primitives（线程同步原语）的工具。这些工具为并发程序员提供两类服务：

1. 通过使 critical operations（关键操作）变为 atomic（原子操作），让线程之间能够共享资源。

2. 使两个或更多线程的操作能够 synchronize（同步）：

   a. 允许一个线程在等待某个资源可用，或等待一个或多个其他线程完成任务时 go to sleep（进入睡眠）；

   b. 允许一个正在运行的线程通过唤醒一个或多个正在睡眠的线程来 notify（通知）它们。

需要注意的是，虽然这些线程同步原语健壮且相对易用，但它们通常相当 expensive（昂贵）。这是因为这些工具由内核提供。因此，与其中任何一个工具交互都需要一次 kernel call（内核调用），而内核调用涉及一次进入 protected mode（保护模式）的 context switch（上下文切换）。这种上下文切换可能消耗 1000 个以上的时钟周期。由于成本较高，一些并发程序员更喜欢实现自己的 atomicity（原子性）和 synchronization（同步）工具，或者转向 lock-free programming（无锁编程），以提高并发软件的效率。尽管如此，扎实理解这些同步原语仍然是任何并发程序员工具箱中的重要组成部分。

### 4.6.1 互斥量

mutex（互斥量）是一种操作系统对象，它允许 critical operations（关键操作）变为 atomic（原子操作）。mutex 可以处于两种状态之一：unlocked（未锁定）或 locked（已锁定）。（这两种状态有时也分别称为 released 和 acquired，或 signaled 和 nonsignaled。）

mutex 最重要的性质是：它保证在任意给定时刻，系统中只有一个线程会持有它的锁。因此，如果我们把某个特定共享数据对象上的所有 critical operations 都包装在一个 mutex lock（互斥锁）中，这些操作相对于彼此就会变成 atomic。换句话说，这些操作变成了 mutually exclusive（互斥的）。这正是 “mutex” 这一名称的来源——它是 “mutual exclusion”（互斥）的缩写。

mutex 可以表示为一个普通的 C++ 对象，也可以表示为指向某个 opaque kernel object（不透明内核对象）的 handle（句柄）。它的 API 通常由以下函数组成：

1. `create()` 或 `init()`：一个函数调用或类构造函数，用于创建 mutex。

2. `destroy()`：一个函数调用或析构函数，用于销毁 mutex。

3. `lock()` 或 `acquire()`：一个 blocking function（阻塞函数），代表调用线程锁定 mutex；如果该锁当前由另一个线程持有，则让当前线程进入睡眠状态（见第 4.4.6.4 节）。

4. `try_lock()` 或 `try_acquire()`：一个 non-blocking function（非阻塞函数），尝试锁定 mutex；如果无法获得锁，则立即返回。

5. `unlock()` 或 `release()`：一个 non-blocking function（非阻塞函数），释放 mutex 上的锁。在大多数操作系统中，只有锁定某个 mutex 的线程才被允许解锁它。

当一个 mutex 被系统中的某个线程锁定时，我们说它处于 nonsignaled（未触发信号）状态。当该线程释放锁时，mutex 变为 signaled（已触发信号）状态。如果有一个或多个其他线程正在 asleep（睡眠）并等待该 mutex，那么触发信号这一动作会使内核选择这些等待线程中的一个并将其唤醒。在某些操作系统中，线程可以显式等待某个内核对象（例如 mutex）变为 signaled。在 Windows 下，`WaitForSingleObject()` 和 `WaitForMultipleObjects()` 这类 OS 调用就用于这一目的。

#### 4.6.1.1 POSIX

既然我们已经理解了 mutex 的工作方式，下面来看几个例子。POSIX thread library 通过 C 风格的函数接口暴露 kernel mutex objects。下面展示如何使用它，把第 4.5.3.2 节中的共享计数器示例转换为一个 atomic operation：

```cpp
#include <pthread.h>

int g_count = 0;
pthread_mutex_t g_mutex;

inline void IncrementCount()
{
    pthread_mutex_lock(&g_mutex);
    ++g_count;
    pthread_mutex_unlock(&g_mutex);
}
```

为了清晰和简洁起见，这里省略了通常由主线程执行的代码：调用 `pthread_mutex_init()` 来初始化 mutex，然后再生成将使用它的线程；以及在所有其他线程退出后调用 `pthread_mutex_destroy()` 来销毁 mutex。

#### 4.6.1.2 C++ 标准库

从 C++11 开始，C++ 标准库通过 `std::mutex` 类暴露 kernel mutexes。下面展示如何使用它使共享计数器的递增操作变为 atomic：

```cpp
#include <mutex>

int g_count = 0;
std::mutex g_mutex;

inline void IncrementCount()
{
    g_mutex.lock();
    ++g_count;
    g_mutex.unlock();
}
```

`std::mutex` 类的构造函数和析构函数会处理底层 kernel mutex object 的初始化和销毁，因此它比 `pthread_mutex_t` 稍微更容易使用。

#### 4.6.1.3 Windows

在 Windows 下，mutex 表示为一个 opaque kernel object，并通过 handle 引用。mutex 通过使用通用的 `WaitForSingleObject()` 函数等待其变为 signaled 来被 “locked”。解锁 mutex 则通过调用 `ReleaseMutex()` 完成。将我们的简单示例改写为使用 Windows mutex，并再次省略 mutex 创建和销毁的细节后，代码如下：

```cpp
#include <windows.h>

int g_count = 0;
HANDLE g_hMutex;

inline void IncrementCount()
{
    if (WaitForSingleObject(g_hMutex, INFINITE)
        == WAIT_OBJECT_0)
    {
        ++g_count;
        ReleaseMutex(g_hMutex);
    }
    else
    {
        // 学会处理失败情况……
    }
}
```

### 4.6.2 临界区

在大多数操作系统中，mutex 可以在进程之间共享。因此，它是一个由内核在内部管理的数据结构。这意味着，对 mutex 执行的所有操作都涉及一次 kernel call（内核调用），因而会导致 CPU 进入 protected mode（保护模式）的一次 context switch（上下文切换）。这使得 mutex 相对昂贵，即使没有其他线程在争用这把锁也是如此。

有些操作系统提供了比 mutex 成本更低的替代机制。例如，Microsoft Windows 提供了一种称为 critical section（临界区）的锁机制。它的术语和 API 看起来与 mutex 略有不同，但 Windows 下的 critical section 本质上只是一个低成本 mutex。

critical section 的 API 大致如下：

1. `InitializeCriticalSection()`：构造一个 critical section 对象。

2. `DeleteCriticalSection()`：销毁一个已初始化的 critical section 对象。

3. `EnterCriticalSection()`：一个 blocking function（阻塞函数），代表调用线程锁定某个 critical section；如果该锁当前由另一个线程持有，则 busy-wait（忙等）或使线程进入睡眠。

4. `TryEnterCriticalSection()`：一个 non-blocking function（非阻塞函数），尝试锁定某个 critical section；如果无法获得锁，则立即返回。

5. `LeaveCriticalSection()`：一个 non-blocking function（非阻塞函数），释放 critical section 对象上的锁。

下面展示如何使用 Windows critical section API 实现一个 atomic increment（原子递增）：

```cpp
#include <windows.h>

int g_count = 0;
CRITICAL_SECTION g_critsec;

inline void IncrementCount()
{
    EnterCriticalSection(&g_critsec);
    ++g_count;
    LeaveCriticalSection(&g_critsec);
}
```

和前面一样，这里省略了一些细节。主线程通常会在生成使用该 critical section 的线程之前初始化它，并且当然会在线程全部退出后清理它。

critical section 的低成本是如何实现的？当一个线程第一次尝试进入（锁定）一个已经被另一个线程锁定的 critical section 时，会先使用一个开销很小的 spin lock（自旋锁）等待，直到另一个线程离开（解锁）该 critical section。spin lock 不需要进入内核的上下文切换，因此比 mutex 便宜几千个时钟周期。只有当线程 busy-wait 太久时，它才会像使用普通 mutex 时一样被置于睡眠状态。这个成本较低的方案能够工作，是因为与 mutex 不同，critical section 不能跨进程边界共享。我们将在第 4.9.7 节更深入讨论 spin lock。

其他一些操作系统也提供“廉价”的 mutex 变体。例如，Linux 支持一种称为 “futex” 的东西，它的作用有点类似 Windows 下的 critical section。它的用法超出了本书范围，但你可以在 [150] 中了解更多关于 futex 的内容。

### 4.6.3 条件变量

在并发编程中，我们经常需要在线程之间发送 signals（信号），以便 synchronize（同步）它们的活动。一个例子就是我们在第 4.4.8 节介绍过的无处不在的 producer-consumer problem（生产者—消费者问题）。在这个问题中，有两个线程：producer thread（生产者线程）负责计算或以其他方式生成某些数据，而这些数据会被 consumer thread（消费者线程）读取并使用。显然，在生产者生产出数据之前，消费者线程无法消费这些数据。因此，生产者线程需要某种方式来 notify（通知）消费者：它的数据已经准备好，可以被消费了。

我们可以考虑使用一个全局 Boolean 变量作为 signaling mechanism（信号机制）。下面的代码片段用 POSIX threads 展示了这一想法。（为了清晰起见，省略了一些细节。）

```cpp
Queue           g_queue;
pthread_mutex_t g_mutex;
bool            g_ready = false;

void* ProducerThread(void*)
{
    // 永远持续生产……
    while (true)
    {
        pthread_mutex_lock(&g_mutex);

        // 用数据填充队列
        ProduceDataInto(&g_queue);

        g_ready = true;

        pthread_mutex_unlock(&g_mutex);

        // 让出剩余的时间片
        // 给消费者一个运行机会
        pthread_yield();
    }
    return nullptr;
}

void* ConsumerThread(void*)
{
    // 永远持续消费……
    while (true)
    {
        // 等待数据准备好
        while (true)
        {
            // 将值读取到局部变量中，
            // 并确保锁定 mutex
            pthread_mutex_lock(&g_mutex);
            const bool ready = g_ready;
            pthread_mutex_unlock(&g_mutex);

            if (ready)
                break;
        }

        // 消费数据
        pthread_mutex_lock(&g_mutex);
        ConsumeDataFrom(&g_queue);
        g_ready = false;
        pthread_mutex_unlock(&g_mutex);

        // 让出剩余的时间片
        // 给生产者一个运行机会
        pthread_yield();
    }
    return nullptr;
}
```

除了这个例子本身稍显刻意之外，它还有一个大问题：消费者线程在一个紧密循环中 spin（自旋），反复轮询 `g_ready` 的值。正如我们在第 4.4.6.4 节中讨论过的，这种 busy-waiting（忙等）会浪费宝贵的 CPU 周期。

理想情况下，我们希望有一种方式能让消费者线程 block（阻塞，也就是进入睡眠），让生产者工作；当数据准备好可以消费时，再唤醒消费者线程。这可以通过一种新的内核对象完成，称为 condition variable（条件变量，CV）。

condition variable 实际上并不是一个存储条件的变量。更准确地说，它是一个 waiting queue（等待队列），其中包含等待中的（睡眠的）线程，并配有一种机制，允许正在运行的线程在自己选择的时刻唤醒这些睡眠线程。（也许 “wait queue” 才是这些东西更好的名字。）sleep 和 wake 操作会借助程序提供的 mutex，并在内核的一点帮助下，以 atomic（原子）方式执行。

condition variable 的 API 通常大致如下：

1. `create()` 或 `init()`：一个函数调用或类构造函数，用于创建 condition variable。

2. `destroy()`：一个函数调用或析构函数，用于销毁 condition variable。

3. `wait()`：一个 blocking function（阻塞函数），使调用线程进入睡眠。

4. `notify()`：一个 non-blocking function（非阻塞函数），唤醒当前正在 condition variable 上睡眠等待的任意线程。

下面我们使用 CV 重写简单的生产者—消费者示例：

```cpp
Queue           g_queue;
pthread_mutex_t g_mutex;
bool            g_ready = false;
pthread_cond_t  g_cv;

void* ProducerThreadCV(void*)
{
    // 永远持续生产……
    while (true)
    {
        pthread_mutex_lock(&g_mutex);

        // 用数据填充队列
        ProduceDataInto(&g_queue);

        // 通知并唤醒消费者线程
        g_ready = true;
        pthread_cond_signal(&g_cv);
        pthread_mutex_unlock(&g_mutex);
    }
    return nullptr;
}

void* ConsumerThreadCV(void*)
{
    // 永远持续消费……
    while (true)
    {
        // 等待数据准备好
        pthread_mutex_lock(&g_mutex);
        while (!g_ready)
        {
            // 进入睡眠直到收到通知……mutex
            // 会由内核替我们释放
            pthread_cond_wait(&g_cv, &g_mutex);

            // 当它醒来时，内核会确保
            // 该线程再次持有 mutex
        }

        // 消费数据
        ConsumeDataFrom(&g_queue);
        g_ready = false;
        pthread_mutex_unlock(&g_mutex);
    }
    return nullptr;
}
```

消费者线程调用 `pthread_cond_wait()` 进入睡眠，直到 `g_ready` 变为 true。生产者先工作一段时间来生成数据。当数据准备好后，生产者将全局 `g_ready` 标志设为 true，然后通过调用 `pthread_cond_signal()` 唤醒正在睡眠的消费者。随后消费者消费数据。在这个例子中，消费者和生产者会像这样无限地来回 ping-pong。

你可能已经注意到，消费者线程在进入检查 `g_ready` 标志的 `while` 循环之前会锁定它的 mutex。当它在 condition variable 上等待时，它看起来是在 holding the mutex lock（持有 mutex 锁）的状态下进入睡眠。通常这会是一个很大的禁忌：如果一个线程在持有锁时进入睡眠，几乎肯定会导致 deadlock（死锁）情况（见第 4.7.1 节）。然而，使用 condition variable 时这并不是问题。这是因为内核实际上做了一个小“手法”：在线程被安全地放入睡眠之后解锁 mutex。稍后，当睡眠线程被唤醒时，内核又会做另一个小“手法”，确保这个刚被唤醒的线程再次持有这把锁。

你可能还注意到了另一个奇怪之处：消费者线程虽然使用了 condition variable 等待标志变为 true，但仍然使用 `while` 循环检查 `g_ready` 的值。这个循环之所以必要，是因为线程有时可能会被内核 spuriously（伪）唤醒。因此，当 `pthread_cond_wait()` 调用返回时，`g_ready` 的值可能还没有真正变为 true。所以我们必须继续在循环中轮询，直到条件确实为真。

### 4.6.4 信号量

正如 mutex 类似于一个 atomic Boolean flag（原子布尔标志），semaphore（信号量）类似于一个 atomic counter（原子计数器），其值永远不允许低于零。我们可以把 semaphore 看成一种特殊的 mutex，它允许 more than one thread（多个线程）同时获得它。

semaphore 可用于允许一组线程共享有限数量的资源。例如，假设我们正在实现一个渲染系统，允许把文本和 2D 图像渲染到离屏缓冲区中，以便绘制游戏的 heads-up display（HUD）和游戏内菜单。由于内存限制，我们进一步假设只能分配四个这样的缓冲区。semaphore 可以用来保证在任何给定时刻，最多只有四个线程被允许渲染到这些缓冲区中。

semaphore 的 API 通常由以下函数组成：

1. `init()`：初始化一个 semaphore 对象，并将它的计数器设为指定的初始值。

2. `destroy()`：销毁一个 semaphore 对象。

3. `take()` 或 `wait()`：如果某个 semaphore 封装的计数器值大于零，该函数会递减计数器并立即返回。如果它的计数器值当前为零，该函数会 block（阻塞，使线程进入睡眠），直到 semaphore 的计数器再次上升到零以上。

4. `give()`、`post()` 或 `signal()`：将封装的计数器值加一，从而为另一个线程打开一个 “slot”，使其能够 `take()` 该 semaphore。如果某个线程当前正睡眠等待该 semaphore，那么调用 `give()` 时，该线程会从它对 `take()` 或 `wait()` 的调用中被唤醒。<sup>7</sup>

> **脚注 7**：当你阅读关于 semaphores 的资料时，可能会发现有些作者使用函数名 `p()` 和 `v()`，而不是 `wait()` 和 `signal()`。这两个字母来自这两个操作的荷兰语名称。

因此，为了实现一个最多可由 N 个线程同时访问的 resource pool（资源池），我们只需创建一个 semaphore，并将它的计数器初始化为 N。线程通过调用 `take()` 获得资源池的访问权；当它完成工作后，通过调用 `give()` 释放对资源池的占用。

当 semaphore 的计数大于零时，我们说该 semaphore 是 signaled（已触发信号）的；当它的计数器等于零时，它是 nonsignaled（未触发信号）的。这就是为什么在某些 API 中，获取和释放 semaphore 的函数分别被命名为 `wait()` 和 `signal()`：如果线程调用该函数时 semaphore 并未处于 signaled 状态，那么线程就会 wait（等待）该 semaphore 变为 signaled。

#### 4.6.4.1 互斥量与二元信号量

初始值设为 1 的 semaphore 称为 binary semaphore（二元信号量）。人们可能会以为 binary semaphore 与 mutex 完全相同。确实，这两个对象都只允许一个线程在某个时刻获得它。

然而，这两个同步对象并不等价，并且通常用于截然不同的目的。

mutex 和 binary semaphore 的关键区别在于：mutex 只能由锁定它的线程解锁。另一方面，semaphore 的计数器可以由一个线程递增，稍后再由另一个线程递减。这意味着 binary semaphore 可以由与“锁定”它的线程不同的线程来“解锁”。或者更准确地说，binary semaphore 可以由一个线程 `give`，再由另一个线程 `take`。

mutex 和 binary semaphore 之间这个看似微妙的差异，会导致这两类同步对象具有非常不同的使用场景。mutex 用于使某个操作变为 atomic（原子操作）。而 binary semaphore 通常用于从一个线程向另一个线程发送 signal（信号）。

再次考虑我们的生产者—消费者示例：生产者需要在它产生的数据准备好消费时通知消费者。这个通知机制可以用两个 binary semaphores 实现：一个允许生产者唤醒消费者，另一个允许消费者唤醒生产者。我们可以把这些 semaphores 理解为表示某个 buffer（缓冲区）中已使用和空闲元素的数量，该缓冲区由两个线程共享；不过在这个简单例子中，缓冲区只能容纳一个 item。因此，我们将这两个 semaphores 分别称为 `g_semUsed` 和 `g_semFree`。下面是使用 POSIX semaphores 时的代码：

```cpp
Queue g_queue;
sem_t g_semUsed; // initialized to 0
sem_t g_semFree; // initialized to 1

void* ProducerThreadSem(void*)
{
    // 永远持续生产……
    while (true)
    {
        // 生产一个 item（因为它是局部数据，
        // 所以可以非原子地完成）
        Item item = ProduceItem();

        // 递减空闲计数
        // （等待直到有空间）
        sem_wait(&g_semFree);

        AddItemToQueue(&g_queue, item);

        // 递增已使用计数
        // （通知消费者已有数据）
        sem_post(&g_semUsed);
    }

    return nullptr;
}

void* ConsumerThreadSem(void*)
{
    // 永远持续消费……
    while (true)
    {
        // 递减已使用计数
        // （等待数据准备好）
        sem_wait(&g_semUsed);

        Item item = RemoveItemFromQueue(&g_queue);

        // 递增空闲计数
        // （通知生产者已有空间）
        sem_post(&g_semFree);

        // 消费 item（因为它是局部数据，
        // 所以可以非原子地完成）
        ConsumeItem(item);
    }
    return nullptr;
}
```

#### 4.6.4.2 实现一个信号量

事实证明，我们可以用一个 mutex、一个 condition variable 和一个整数来实现 semaphore。从这个意义上说，semaphore 是比 mutex 或 condition variable 都更“高层”的构造。下面是这个实现的大致样子：

```cpp
class Semaphore
{
private:
    int             m_count;
    pthread_mutex_t m_mutex;
    pthread_cond_t  m_cv;

public:
    explicit Semaphore(int initialCount)
    {
        m_count = initialCount;
        pthread_mutex_init(&m_mutex, nullptr);
        pthread_cond_init(&m_cv, nullptr);
    }

    void Take()
    {
        pthread_mutex_lock(&m_mutex);
        // 只要计数为零，就让线程进入睡眠
        while (m_count == 0)
            pthread_cond_wait(&m_cv, &m_mutex);
        --m_count;
        pthread_mutex_unlock(&m_mutex);
    }

    void Give()
    {
        pthread_mutex_lock(&m_mutex);
        ++m_count;
        // 如果递增前计数为零，
        // 则唤醒一个等待线程
        if (m_count == 1)
            pthread_cond_signal(&m_cv);
        pthread_mutex_unlock(&m_mutex);
    }

    // 其他常用函数名的别名
    void Wait()   { Take(); }
    void Post()   { Give(); }
    void Signal() { Give(); }
    void Down()   { Take(); }
    void Up()     { Give(); }
    void P()      { Take(); }
                  // Dutch "proberen" = "test"

    void V()      { Give(); }
                  // Dutch "verhogen" = "increment"
};
```

### 4.6.5 Windows 事件

Windows 提供了一种称为 event object（事件对象）的机制，其功能类似于 condition variable，但使用起来要简单得多。一旦创建了 event object，线程就可以通过调用 `WaitForSingleObject()` 进入睡眠，而另一个线程可以通过调用 `SetEvent()` 唤醒该线程。用 event objects 重写我们的生产者—消费者示例，可以得到如下非常简单的实现：

```cpp
#include <windows.h>

Queue g_queue;
Handle g_hUsed; // initialized to false (nonsignaled)
Handle g_hFree; // initialized to true (signaled)

void* ProducerThreadEv(void*)
{
    // 永远持续生产……
    while (true)
    {
        // 生产一个 item（因为它是局部数据，
        // 所以可以非原子地完成）
        Item item = ProduceItem();

        // 等到有空间
        WaitForSingleObject(&g_hFree);

        AddItemToQueue(&g_queue, item);

        // 通知消费者已有数据
        SetEvent(&g_hUsed);
    }
    return nullptr;
}

void* ConsumerThreadEv(void*)
{
    // 永远持续消费……
    while (true)
    {
        // 等待数据准备好
        WaitForSingleObject(&g_hUsed);

        Item item = RemoveItemFromQueue(&g_queue);

        // 通知生产者已有空间
        SetEvent(&g_hFree);

        // 消费 item（因为它是局部数据，
        // 所以可以非原子地完成）
        ConsumeItem(item);
    }
    return nullptr;
}

void MainThread()
{
    // 以 nonsignalled 状态创建 event
    g_hUsed = CreateEvent(nullptr, false,
                          false, nullptr);
    g_hFree = CreateEvent(nullptr, false,
                          true, nullptr);

    // 生成我们的线程
    CreateThread(nullptr, 0x2000, ConsumerThreadEv,
                 0, 0, nullptr);
    CreateThread(nullptr, 0x2000, ProducerThreadEv,
                 0, 0, nullptr);

    // ...
}
```
