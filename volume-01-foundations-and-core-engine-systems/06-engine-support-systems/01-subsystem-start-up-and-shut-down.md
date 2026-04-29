## 6.1 子系统启动与关闭

游戏引擎是一套复杂的软件系统，由许多相互交互的子系统组成。当引擎首次启动时，每个子系统都必须按照特定顺序进行配置和初始化。子系统之间的依赖关系隐含地决定了它们的启动顺序。也就是说，如果子系统 B 依赖于子系统 A，那么 A 必须先启动，B 才能被初始化。关闭过程通常按照相反顺序进行，因此 B 会先关闭，然后才是 A。

### 6.1.1 C++ 静态初始化顺序（或者说缺乏顺序）

由于大多数现代游戏引擎所使用的编程语言是 C++，我们有必要简单考虑一下：能否利用 C++ 原生的启动与关闭语义来启动和关闭引擎的子系统。在 C++ 中，全局对象和静态对象会在程序入口点（`main()`，或者 Windows 下的 `WinMain()`）被调用之前构造。然而，这些构造函数的调用顺序是完全不可预测的。全局对象和静态类实例的析构函数会在 `main()`（或 `WinMain()`）返回之后被调用，而且它们的调用顺序同样不可预测。显然，对于初始化和关闭游戏引擎的各个子系统来说，这种行为并不理想；事实上，对于任何全局对象之间存在相互依赖关系的软件系统来说，这都不是理想行为。

这多少有些遗憾，因为实现游戏引擎这类大型子系统时，一个常见的设计模式是为每个子系统定义一个单例类（singleton class，通常称为 manager，即管理器）。如果 C++ 能够让我们更好地控制全局对象和静态类实例的构造与析构顺序，那么我们就可以把单例实例定义为全局变量，而不需要进行动态内存分配。例如，我们可以写成：

```cpp
class RenderManager
{
public:
    RenderManager()
    {
        // start up the manager...
    }

    ~RenderManager()
    {
        // shut down the manager...
    }

    // ...
};

// singleton instance
RenderManager gRenderManager;
```

可惜的是，由于我们无法直接控制构造和析构顺序，这种做法行不通。

#### 6.1.1.1 按需构造

这里有一个可以利用的 C++“技巧”。在函数内部声明的静态变量不会在 `main()` 被调用之前构造，而是在该函数第一次被调用时才构造。因此，如果我们的全局单例是函数静态变量，那么就可以控制这些全局单例的构造顺序。

```cpp
class RenderManager
{
public:
    // Get the one and only instance.
    static RenderManager& get()
    {
        // This function-static will be constructed on the
        // first call to this function.
        static RenderManager sSingleton;
        return sSingleton;
    }

    RenderManager()
    {
        // Start up other managers we depend on, by
        // calling their get() functions first...
        VideoManager::get();
        TextureManager::get();

        // Now start up the render manager.
        // ...
    }

    ~RenderManager()
    {
        // Shut down the manager.
        // ...
    }
};
```

你会发现，许多软件工程教材都会推荐这种设计，或者推荐一种涉及动态分配单例对象的变体，如下所示。

```cpp
static RenderManager& get()
{
    static RenderManager* gpSingleton = nullptr;
    if (gpSingleton == nullptr)
    {
        gpSingleton = new RenderManager;
    }
    ASSERT(gpSingleton);
    return *gpSingleton;
}
```

遗憾的是，这仍然无法让我们控制析构顺序。C++ 可能会在调用 `RenderManager` 的析构函数之前，就先销毁某个 `RenderManager` 在关闭流程中所依赖的管理器。此外，也很难准确预测 `RenderManager` 单例到底会在什么时候被构造，因为它会发生在第一次调用 `RenderManager::get()` 时——谁知道那会是什么时候呢？更进一步，使用该类的程序员可能并没有意识到，一个看起来无害的 `get()` 函数可能会做一些昂贵的事情，比如分配并初始化一个重量级单例对象。这是一种不可预测且危险的设计。因此，我们需要采用一种更直接的方法，以获得更强的控制能力。

### 6.1.2 一种简单且有效的方法

假设我们仍然想坚持使用单例管理器来表示各个子系统。在这种情况下，最简单的“暴力”做法是为每个单例管理器类定义显式的启动和关闭函数。这些函数取代构造函数和析构函数；事实上，我们应该安排构造函数和析构函数什么都不做。这样，启动函数和关闭函数就可以从 `main()` 内部（或者从某个负责管理整个引擎的更高层单例对象中）按照所需顺序被显式调用。例如：

```cpp
class RenderManager
{
public:
    RenderManager()
    {
        // do nothing
    }

    ~RenderManager()
    {
        // do nothing
    }

    void startUp()
    {
        // start up the manager...
    }

    void shutDown()
    {
        // shut down the manager...
    }

    // ...
};

class PhysicsManager   { /* similar... */ };

class AnimationManager { /* similar... */ };

class MemoryManager    { /* similar... */ };

class FileSystemManager { /* similar... */ };

// ...

RenderManager        gRenderManager;
PhysicsManager       gPhysicsManager;
AnimationManager     gAnimationManager;
TextureManager       gTextureManager;
VideoManager         gVideoManager;
MemoryManager        gMemoryManager;
FileSystemManager    gFileSystemManager;
// ...

int main(int, char**)
{
    // Start up engine systems in the correct order.
    gMemoryManager.startUp();
    gFileSystemManager.startUp();
    gVideoManager.startUp();
    gTextureManager.startUp();
    gRenderManager.startUp();
    gAnimationManager.startUp();
    gPhysicsManager.startUp();
    // ...

    // Run the game.
    gSimulationManager.run();

    // Shut everything down, in reverse order.
    // ...
    gPhysicsManager.shutDown();
    gAnimationManager.shutDown();
    gRenderManager.shutDown();
    gFileSystemManager.shutDown();
    gMemoryManager.shutDown();

    return 0;
}
```

当然，也有一些“更优雅”的方式来完成这件事。例如，可以让每个管理器把自己注册到一个全局优先级队列中，然后遍历这个队列，以正确顺序启动所有管理器。也可以定义管理器到管理器之间的依赖图，让每个管理器显式列出自己依赖的其他管理器，然后编写代码根据这些相互依赖关系计算出最优启动顺序。还可以使用前文所述的按需构造方法。根据我的经验，暴力方法总是胜出，原因如下：

- 它简单，容易实现。
- 它是显式的。只要看代码，你就能立刻看到并理解启动顺序。
- 它容易调试和维护。如果某个东西启动得不够早，或者启动得太早，你只需要移动一行代码。

暴力式手动启动与关闭方法有一个小缺点：你可能会不小心以一个并非严格逆于启动顺序的顺序来关闭某些东西。不过我认为这并不是什么大问题。只要你能够成功启动和关闭引擎的子系统，就已经足够了。

### 6.1.3 来自真实引擎的一些例子

下面我们简要看几个真实游戏引擎中的启动与关闭示例。

#### 6.1.3.1 OGRE

OGRE 的作者承认，OGRE 是一个渲染引擎，而不是严格意义上的游戏引擎。但出于实际需要，它提供了许多完整游戏引擎中常见的底层功能，其中包括一种简单而优雅的启动与关闭机制。OGRE 中的一切都由单例对象 `Ogre::Root` 控制。它包含指向 OGRE 中所有其他子系统的指针，并管理它们的创建和销毁。这使得程序员启动 OGRE 变得非常容易——只需要 `new` 一个 `Ogre::Root` 实例即可。

下面是 OGRE 源代码中的一些节选，让我们看看它具体做了什么：

*OgreRoot.h*

```cpp
class _OgreExport Root : public Singleton<Root>
{
    // <some code omitted...>

    // Singletons
    LogManager* mLogManager;
    ControllerManager* mControllerManager;
    SceneManagerEnumerator* mSceneManagerEnum;
    SceneManager* mCurrentSceneManager;
    DynLibManager* mDynLibManager;
    ArchiveManager* mArchiveManager;
    MaterialManager* mMaterialManager;
    MeshManager* mMeshManager;
    ParticleSystemManager* mParticleManager;
    SkeletonManager* mSkeletonManager;
    OverlayElementFactory* mPanelFactory;
    OverlayElementFactory* mBorderPanelFactory;
    OverlayElementFactory* mTextAreaFactory;
    OverlayManager* mOverlayManager;
    FontManager* mFontManager;
    ArchiveFactory *mZipArchiveFactory;
    ArchiveFactory *mFileSystemArchiveFactory;

    ResourceGroupManager* mResourceGroupManager;
    ResourceBackgroundQueue* mResourceBackgroundQueue;
    ShadowTextureManager* mShadowTextureManager;

    // etc.
};
```

*OgreRoot.cpp*

```cpp
Root::Root(const String& pluginFileName,
           const String& configFileName,
           const String& logFileName) :
    mLogManager(0),
    mCurrentFrame(0),
    mFrameSmoothingTime(0.0f),
    mNextMovableObjectTypeFlag(1),
    mIsInitialised(false)
{
    // superclass will do singleton checking
    String msg;

    // Init
    mActiveRenderer = 0;
    mVersion
        = StringConverter::toString(OGRE_VERSION_MAJOR)
        + "."
        + StringConverter::toString(OGRE_VERSION_MINOR)
        + "."
        + StringConverter::toString(OGRE_VERSION_PATCH)
        + OGRE_VERSION_SUFFIX + " "
        + "(" + OGRE_VERSION_NAME + ")";
    mConfigFileName = configFileName;

    // create log manager and default log file if there
    // is no log manager yet
    if(LogManager::getSingletonPtr() == 0)
    {
        mLogManager = new LogManager();
        mLogManager->createLog(logFileName, true, true);
    }

    // dynamic library manager
    mDynLibManager = new DynLibManager();
    mArchiveManager = new ArchiveManager();

    // ResourceGroupManager
    mResourceGroupManager = new ResourceGroupManager();

    // ResourceBackgroundQueue
    mResourceBackgroundQueue
        = new ResourceBackgroundQueue();

    // and so on...
}
```

OGRE 提供了一个模板化的 `Ogre::Singleton` 基类，所有单例管理器类都从它派生。如果查看它的实现，你会发现 `Ogre::Singleton` 并不使用延迟构造，而是依赖 `Ogre::Root` 来显式地 `new` 每个单例。正如我们前面讨论的，这样做是为了确保这些单例以明确定义的顺序被创建和销毁。

#### 6.1.3.2 Naughty Dog 的《神秘海域》和《最后生还者》系列

Naughty Dog, Inc. 为《神秘海域》（*Uncharted*）和《最后生还者》（*The Last of Us*）系列游戏开发的引擎，使用了类似的显式技术来启动各个子系统。观察下面的代码你会发现，引擎启动并不总是简单地按顺序分配单例实例。大量操作系统服务、第三方库等内容都必须在引擎初始化期间启动。此外，只要可能，就会避免动态内存分配，因此许多单例都是静态分配的对象（例如 `g_fileSystem`、`g_languageMgr` 等）。这种做法并不总是漂亮，但它能够完成任务。

```cpp
Err InitEngine()
{
    init_exception_handler();

    U8* pPhysicsHeap = new(kAllocGlobal, kAlign16)
        U8[ALLOCATION_GLOBAL_PHYS_HEAP];
    PhysicsAllocatorInit(pPhysicsHeap,
        ALLOCATION_GLOBAL_PHYS_HEAP);

    g_textDb.Init();
    g_textSubDb.Init();
    g_spuMgr.Init();

    g_drawScript.InitPlatform();

    PlatformUpdate();

    thread_t init_thr;
    thread_create(&init_thr, threadInit, 0, 30,
        64*1024, 0, "Init");

    char masterConfigFileName[256];
    snprintf(masterConfigFileName,
        sizeof(masterConfigFileName),
        MASTER_CFG_PATH);
    {
        Err err = ReadConfigFromFile(
            masterConfigFileName);
        if (err.Failed())
        {
            MsgErr("Config file not found (%s).\n",
                masterConfigFileName);
        }
    }

    memset(&g_discInfo, 0, sizeof(BootDiscInfo));
    int err1 = GetBootDiscInfo(&g_discInfo);
    Msg("GetBootDiscInfo() : 0x%x\n", err1);
    if(err1 == BOOTDISCINFO_RET_OK)
    {
        printf("titleId        : [%s]\n",
            g_discInfo.titleId);
        printf("parentalLevel : [%d]\n",
            g_discInfo.parentalLevel);
    }

    g_fileSystem.Init(g_gameInfo.m_onDisc);

    g_languageMgr.Init();
    if (g_shouldQuit) return Err::kOK;

    // and so on...
}
```