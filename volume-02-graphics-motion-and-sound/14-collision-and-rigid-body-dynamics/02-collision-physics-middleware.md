## 14.2 碰撞/物理中间件

编写碰撞系统和刚体动力学仿真是一项具有挑战性且耗时的工作。在典型游戏引擎中，碰撞/物理系统可能占据相当大比例的源代码。这意味着需要编写和维护大量代码！

幸运的是，现在已经有许多健壮、高质量的碰撞/物理引擎可供使用，它们既有商业产品，也有开源形式。下面列出其中一些。

### 14.2.1 ODE

ODE 是 “Open Dynamics Engine” 的缩写 [81]。顾名思义，ODE 是一个开源的碰撞与刚体动力学 SDK。它的功能集类似于 Havok 这类商业产品。它的优点包括免费（对小型游戏工作室和学校项目来说是一大优势！），以及提供完整源代码（这使调试容易得多，也使得开发者可以根据特定游戏的具体需求修改物理引擎）。

### 14.2.2 Bullet

Bullet 是一个开源碰撞检测与物理库，游戏和电影行业都会使用它。它的碰撞引擎与动力学仿真集成在一起，但也提供了钩子，使碰撞系统可以单独使用，或与其他物理引擎集成。它支持**连续碰撞检测**（continuous collision detection, CCD）——也称为**撞击时间**（time of impact, TOI）碰撞检测。如下文所见，当仿真中包含小型、高速运动对象时，这项技术会非常有用。Bullet SDK 及其文档可在 [312] 下载。

### 14.2.3 PhysX

PhysX 最初是一个名为 Novodex 的库，由 Ageia 制作并发行，作为其专用物理协处理器市场策略的一部分。后来它被 NVIDIA 收购并重新设计，使其可以使用 NVIDIA 的 GPU 作为协处理器运行。（它也可以完全在 CPU 上运行，而不需要 GPU 支持。）它可在 [313] 获取。Ageia 和 NVIDIA 的市场策略之一，是完全免费提供 SDK 的 CPU 版本，以推动物理协处理器市场的发展。开发者也可以付费获得完整源代码，并按需定制该库。PhysX 现在与 APEX 结合在一起，APEX 是 NVIDIA 的可扩展多平台动力学框架。PhysX/APEX 可用于 Windows、Linux、Mac、Android、Xbox 360、PlayStation 3、Xbox One、PlayStation 4 和 Wii。

### 14.2.4 Havok

Havok 是商业物理 SDK 中的黄金标准，提供了最丰富的功能集之一，并且在所有受支持平台上都具有出色的性能特征。（它也是最昂贵的解决方案。）Havok 由一个核心碰撞/物理引擎，以及若干可选附加产品组成，其中包括车辆物理系统、可破坏环境建模系统，以及一个功能完整的动画 SDK，并且可以直接集成到 Havok 的布娃娃物理系统中。它可用于 Xbox 360、PlayStation 3、Xbox One、PlayStation 4、PlayStation Vita、Wii、Wii U、Windows 8、Android、Apple Mac 和 iOS。你可以在 [314] 了解更多关于 Havok 的信息。

### 14.2.5 物理抽象层（PAL）

**Physics Abstraction Layer**（PAL，物理抽象层）是一个开源库，允许开发者在单个项目中使用多个物理 SDK。它为 PhysX（Novodex）、Newton、ODE、OpenTissue、Tokamak 以及其他一些 SDK 提供钩子。你可以在 [315] 阅读更多关于 PAL 的内容。

### 14.2.6 数字分子物质（DMM）

位于瑞士日内瓦的 Pixelux Entertainment S.A. 制作了一款独特的物理引擎，名为 **Digital Molecular Matter**（DMM，数字分子物质）。它使用有限元方法来仿真可变形物体和可破坏对象的动力学。该引擎同时具有离线组件和运行时组件。它于 2008 年发布，并可在 LucasArts 的 *Star Wars: The Force Unleashed* 中看到实际应用。关于可变形体力学的讨论超出了本书范围，但你可以在 [316] 阅读更多关于 DMM 的内容。
