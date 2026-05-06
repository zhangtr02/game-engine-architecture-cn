## 8.3 游戏循环架构风格

游戏循环可以通过许多不同方式实现。不过从核心来看，它们通常都可以归结为一个或多个简单循环，再加上各种修饰和扩展。下面我们将探讨几种较常见的架构。

### 8.3.1 Windows 消息泵

在 Windows 平台上，除了服务游戏引擎自身的各种子系统外，游戏还需要服务来自 Windows 操作系统的消息。因此，Windows 游戏会包含一段称为**消息泵**（message pump）的代码。其基本思想是：只要 Windows 消息到达，就处理这些消息；只有在没有待处理 Windows 消息时，才服务游戏引擎。一个消息泵通常看起来像这样：

    while (true)
    {
        // 处理所有待处理的 Windows 消息。
        MSG msg;

        while (PeekMessage(&msg, nullptr, 0, 0) > 0)
        {
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }

        // 没有更多 Windows 消息要处理了——运行一次
        // “真正的”游戏循环迭代。
        RunOneIterationOfGameLoop();
    }

以这种方式实现游戏循环的一个副作用是：Windows 消息会优先于游戏渲染和模拟。因此，当你在桌面上调整游戏窗口大小或拖动游戏窗口时，游戏会暂时冻结。

### 8.3.2 回调驱动框架

大多数游戏引擎子系统和第三方游戏中间件包都被组织为**库**（libraries）。库是一组函数和/或类，应用程序员可以按自己认为合适的任何方式调用它们。库为程序员提供了最大的灵活性。但是，库有时也比较难用，因为程序员必须理解如何正确使用库提供的函数和类。

相比之下，一些游戏引擎和游戏中间件包被组织为**框架**（frameworks）。框架是一种部分构建好的应用程序；程序员通过在框架中提供缺失功能的自定义实现（或覆盖其默认行为）来完成应用程序。但程序员对应用程序整体控制流几乎没有控制权，因为控制流由框架控制。

在基于框架的渲染引擎或游戏引擎中，主游戏循环已经替我们写好了，但它基本上是空的。游戏程序员可以编写回调函数来“填充”缺失的细节。OGRE 渲染引擎就是一个被包装成框架的库示例。在最底层，OGRE 提供了一些函数，游戏引擎程序员可以直接调用它们。然而，OGRE 也提供了一个框架，它封装了如何有效使用底层 OGRE 库的知识。如果程序员选择使用 OGRE 框架，就需要从 `Ogre::FrameListener` 派生一个类，并覆盖两个虚函数：`frameStarted()` 和 `frameEnded()`。正如你可能猜到的，这两个函数分别会在 OGRE 渲染主 3D 场景之前和之后被调用。OGRE 框架内部游戏循环的实现看起来大致如下所示。（实际源代码见 `OgreRoot.cpp` 中的 `Ogre::Root::renderOneFrame()`。）

    while (true)
    {
        for (each frameListener)
        {
            frameListener.frameStarted();
        }

        renderCurrentScene();

        for (each frameListener)
        {
            frameListener.frameEnded();
        }

        finalizeSceneAndSwapBuffers();
    }

某个具体游戏的帧监听器实现可能如下所示：

    class GameFrameListener : public Ogre::FrameListener
    {
    public:
        virtual void frameStarted(const FrameEvent& event)
        {
            // 执行在 3D 场景渲染之前必须发生的事情
            // （即服务所有游戏引擎子系统）。
            pollJoypad(event);
            updatePlayerControls(event);
            updateDynamicsSimulation(event);
            resolveCollisions(event);
            updateCamera(event);

            // 等等。
        }

        virtual void frameEnded(const FrameEvent& event)
        {
            // 执行在 3D 场景渲染之后必须发生的事情。
            drawHud(event);

            // 等等。
        }
    };

### 8.3.3 基于事件的更新

在游戏中，**事件**（event）是指游戏或其环境状态中任何有趣的变化。例子包括：人类玩家按下手柄上的按钮、一次爆炸发生、敌方角色发现玩家，等等。大多数游戏引擎都拥有一个**事件系统**（event system），允许各种引擎子系统登记自己对特定类型事件的兴趣，并在这些事件发生时作出响应（详见 Section 17.8）。游戏的事件系统通常非常类似于几乎所有图形用户界面底层的事件/消息系统，例如 Microsoft Windows 的窗口消息、Java AWT 中的事件处理系统，或者 C# 的 `delegate` 和 `event` 关键字提供的服务。

有些游戏引擎会利用其事件系统来实现部分或全部子系统的周期性服务。为了让这种方式能够工作，事件系统必须允许事件被投递到未来，也就是说，事件可以入队等待稍后分发。随后，游戏引擎可以通过简单地投递一个事件来实现周期性更新。在事件处理函数中，代码可以执行任何所需的周期性服务。然后，它可以在未来的 1/30 秒或 1/60 秒再次投递一个新事件，从而只要仍然需要，就持续进行周期性服务。