## 1.4 游戏引擎概览

### 1.4.1 Quake 引擎家族

第一款 3D 第一人称射击游戏（FPS）通常被认为是 *Castle Wolfenstein 3D*（1992）。这款游戏由德克萨斯州的 id Software 为 PC 平台编写，它带领游戏行业进入了一个全新且令人兴奋的方向。随后，id Software 又创造了 *Doom*、*Quake*、*Quake II* 和 *Quake III*。这些引擎在架构上都非常相似，因此我将它们称为 Quake 引擎家族。Quake 技术曾被用于创建许多其他游戏，甚至也被用于创建其他引擎。例如，PC 平台上的 *Medal of Honor* 系列血统大致如下：

- *Quake III*（id Software）；

- *Sin*（Ritual）；

- *F.A.K.K. 2*（Ritual）；

- *Medal of Honor: Allied Assault*（2015 & Dreamworks Interactive）；

- *Medal of Honor: Pacific Assault*（Electronic Arts, Los Angeles）。

许多其他基于 Quake 技术的游戏，也通过许多不同游戏和工作室走出了同样曲折的路径。事实上，Valve 的 Source 引擎（用于制作 *Half-Life* 系列游戏）也与 Quake 技术有着遥远的根源关系。

*Quake* 和 *Quake II* 的源代码是免费提供的，而且最初的 Quake 引擎架构相当不错，也比较“干净”（当然，它们现在已经有些过时，并且完全使用 C 编写）。这些代码库是理解工业级游戏引擎如何构建的极佳示例。*Quake* 和 *Quake II* 的完整源代码可以在 [64] 找到。

如果你拥有 *Quake* 和/或 *Quake II* 游戏，实际上可以使用 Microsoft Visual Studio 构建这些代码，并使用磁盘中真实的游戏资源，在调试器下运行游戏。这会非常有启发性。你可以设置断点，运行游戏，然后通过逐步执行代码来分析引擎实际是如何工作的。我强烈建议你下载其中一个或两个引擎，并以这种方式分析它们的源代码。

### 1.4.2 Unreal Engine 引擎

Epic Games, Inc. 于 1998 年凭借其传奇游戏 *Unreal* 闯入 FPS 领域。从那时起，Unreal Engine 已经成为 FPS 领域中 Quake 技术的主要竞争者。Unreal Engine 2（UE2）是 *Unreal Tournament 2004*（UT2004）的基础，并被用于无数“mod”、大学项目和商业游戏。Unreal Engine 5（UE5）是这一演化过程中的最新一步，它拥有业界一些最出色的工具和最丰富的引擎特性集合，包括用于创建着色器的便捷而强大的图形用户界面，以及一个用于游戏逻辑编程的图形用户界面，称为 Blueprints（此前称为 Kismet）。

Unreal Engine 因其广泛的功能集合和统一、易用的工具而闻名。Unreal Engine 并不完美，大多数开发者都会以各种方式修改它，以便让自己的游戏在特定硬件平台上以最优方式运行。不过，Unreal 是一个极其强大的原型开发工具和商业游戏开发平台，可以用于构建几乎任何 3D 第一人称或第三人称游戏（更不用说其他类型的游戏了）。许多不同类型的精彩游戏都使用 Unreal Engine 开发，包括 Tequila Works 的 *Rime*、Radiation Blue 的 *Genesis: Alpha One*、Hazelight Studios 的 *A Way Out*，以及 Microsoft Studios 的 *Crackdown 3*。

Unreal Developer Network（UDN）提供了关于所有已发布 Unreal Engine 版本的大量文档和其他信息 [65]。Unreal Engine 5 最新版本的完整文档可以在 [66] 找到。还有许多其他有用的网站和 wiki 会介绍 Unreal Engine，其中一个流行网站是 [67]。

值得庆幸的是，Epic 现在以较低的月度订阅费用，加上游戏发布后利润分成的方式，提供 Unreal Engine 5 的完整访问权限，包括源代码在内。这使 Unreal Engine 成为小型独立游戏工作室的一个可行选择。

### 1.4.3 Half-Life 的 Source 与 Source 2 引擎

Source 是驱动知名游戏 *Half-Life 2* 及其续作 *HL2: Episode One*、*HL2: Episode Two*、*Team Fortress 2* 和 *Portal* 的游戏引擎（这些游戏曾以 *The Orange Box* 的标题一同发行）。它的最新迭代版本 Source 2，是一个高质量引擎，配备了扎实的工具套件、良好的图形保真度，以及一个自定义物理引擎。包括 *Half-Life: Alyx* 和 *Dota 2* 在内的许多流行游戏，都是使用 Source 2 引擎构建的。

### 1.4.4 DICE 的 Frostbite

Frostbite 引擎源自 DICE 在 2006 年为 *Battlefield Bad Company* 创建游戏引擎的努力。从那时起，Frostbite 引擎已经成为 Electronic Arts（EA）内部使用最广泛的引擎；它被用于 EA 的许多关键系列作品，包括 *Mass Effect*、*Battlefield*、*Need for Speed*、*Dragon Age* 和 *Star Wars Battlefront II*。Frostbite 拥有一个强大的统一资源创建工具 FrostEd，一个称为 Backend Services 的强大工具管线，以及一个强大的运行时游戏引擎。它是专有引擎，因此很遗憾，EA 之外的开发者无法使用它。

### 1.4.5 Rockstar 高级游戏引擎（RAGE）

RAGE 是驱动极其流行的 *Grand Theft Auto V* 的引擎。RAGE 由 RAGE Technology Group 开发，该团队是 Rockstar Games 旗下 Rockstar San Diego 工作室的一个部门。Rockstar Games 内部工作室曾使用 RAGE 为 PlayStation 5、Xbox Series X/S、PlayStation 4、Xbox One、PlayStation 3、Xbox 360、Wii、Windows 和 MacOS 开发游戏。使用这一专有引擎开发的游戏包括 *Grand Theft Auto IV*、*Red Dead Redemption* 和 *Max Payne 3*。

### 1.4.6 CRYENGINE 引擎

Crytek 最初开发其强大的游戏引擎 CRYENGINE，是为了给 NVIDIA 制作一个技术演示。当这项技术的潜力得到认可后，Crytek 将这个演示改造成了一款完整游戏，于是 *Far Cry* 诞生了。此后，许多游戏都使用 CRYENGINE 制作，包括 *Crysis*、*Codename Kingdoms*、*Ryse: Son of Rome* 和 *Everyone’s Gone to the Rapture*。多年来，该引擎已经演化为如今 Crytek 的最新产品 CRYENGINE V。这个强大的游戏开发平台提供了一套功能强大的资源创建工具，以及一个功能丰富的运行时引擎，能够提供高质量实时图形。CRYENGINE 可用于制作面向多种平台的游戏，包括 Xbox Series X/S、Xbox One、Xbox 360、PlayStation 5、PlayStation 4、PlayStation 3、Wii U、Linux、iOS 和 Android。

### 1.4.7 Sony 的 PhyreEngine

为了让 Sony PlayStation 3 平台上的游戏开发更加便利，Sony 于 2008 年在 Game Developer’s Conference（GDC）上推出了 PhyreEngine。到 2013 年，PhyreEngine 已经发展为一个强大且功能完整的游戏引擎，支持一系列令人印象深刻的特性，包括高级光照和延迟渲染。许多工作室使用它构建了 90 多款已发布作品，包括 thatgamecompany 的热门游戏 *flOw*、*Flower* 和 *Journey*，以及 Coldwood Interactive 的 *Unravel*。PhyreEngine 支持 Sony 的 PlayStation 4、PlayStation 3、PlayStation 2、PlayStation Vita 和 PSP 平台。PhyreEngine 让开发者能够使用 PS3 高度并行的 Cell 架构，以及 PS4 的高级计算能力，同时还提供了一个简化的新世界编辑器和其他强大的游戏开发工具。作为 PlayStation SDK 的一部分，任何获得授权的 Sony 开发者都可以免费使用它。

### 1.4.8 Microsoft 的 XNA Game Studio

Microsoft 的 XNA Game Studio 是一个易用且高度易访问的游戏开发平台，基于 C# 语言和公共语言运行时（Common Language Runtime，CLR）。它旨在鼓励玩家创建自己的游戏，并将其分享到在线游戏社区中，就像 YouTube 鼓励人们创作和分享家庭自制视频一样。

无论好坏，Microsoft 于 2014 年正式退役了 XNA。不过，开发者仍然可以通过名为 MonoGame 的 XNA 开源实现，将自己的 XNA 游戏移植到 iOS、Android、Mac OS X、Linux 和 Windows 8 Metro。更多细节见 [68]。

### 1.4.9 Unity 引擎

Unity 是一个强大的跨平台游戏开发环境和运行时引擎，支持范围广泛的平台。使用 Unity，开发者可以将游戏部署到移动平台（例如 Apple iOS、Google Android）、主机平台（Microsoft Xbox 360、Xbox One 和 Xbox Series X/S、Sony PlayStation 3、PlayStation 4 和 PlayStation 5，以及 Nintendo Wii、Wii U）、掌机游戏平台（例如 PlayStation Vita、Nintendo Switch）、桌面计算机（Microsoft Windows、Apple Macintosh 和 Linux）、电视盒（例如 Android TV 和 tvOS），以及虚拟现实（VR）系统（例如 Oculus Rift、Steam VR、Gear VR）。

Unity 的主要设计目标是降低开发难度，并支持跨平台游戏部署。因此，Unity 提供了一个易用的集成编辑器环境，你可以在其中创建和操纵构成游戏世界的资源与实体，并在编辑器中或直接在目标硬件上快速预览游戏运行效果。Unity 还提供了一套强大的工具，用于在每个目标平台上分析和优化游戏；一个完整的资源处理管线；以及在每个部署平台上独立管理性能—质量权衡的能力。Unity 支持 C# 脚本；拥有一个强大的动画系统，支持动画重定向（即在一个角色上创作的动画可以在完全不同的角色上播放）；并支持联网多人游戏。

Unity 已被用于创建各种各样已发布的游戏，包括 N-Fusion/Eidos Montreal 的 *Deus Ex: The Fall*、Team Cherry 的 *Hollow Knight*，以及 StudioMDHR 的颠覆性复古风格游戏 *Cuphead*。获得 Webby Award 的短片 *Adam* 也是使用 Unity 实时渲染的。

### 1.4.10 其他商业游戏引擎

市面上还有许多其他商业游戏引擎。虽然独立开发者可能没有预算购买引擎，但其中许多产品都有优秀的在线文档和/或 wiki，可以作为了解游戏引擎和一般游戏编程的重要信息来源。例如，可以查看 Terathon Software 的 Tombstone/C4 引擎 [69]、LeadWerks 引擎 [70]，以及 Idea Fabrik, PLC 的 HeroEngine [71]。

### 1.4.11 专有内部引擎

许多公司都会构建并维护专有的内部游戏引擎。Electronic Arts 使用一个名为 Sage 的专有引擎构建了许多 RTS 游戏，该引擎由 Westwood Studios 开发。Naughty Dog 的 *Crash Bandicoot* 和 *Jak and Daxter* 系列，是基于一个专门为 PlayStation 和 PlayStation 2 定制的专有引擎构建的。对于 *Uncharted* 系列，Naughty Dog 开发了一个全新的引擎，专门针对 PlayStation 3 硬件定制。这个引擎不断演化，并最终被用于创建 Naughty Dog 的 *The Last of Us*（PlayStation 3、PlayStation 4 和 PC），以及 *Uncharted 4: A Thief’s End*（PlayStation 4、PlayStation 5、PC）、*Uncharted: The Lost Legacy*（PlayStation 4、PlayStation 5）和 *The Last of Us Part II*（PlayStation 5 和 PC）。当然，许多商业授权游戏引擎，例如 Quake、Source、Unreal Engine 和 CRYENGINE，最初也都是专有内部引擎。

### 1.4.12 开源引擎

开源 3D 游戏引擎是由业余或专业游戏开发者构建，并免费在线提供的引擎。“开源”（open source）一词通常意味着源代码可以免费获得，并且采用某种开放开发模式，也就是说几乎任何人都可以贡献代码。许可协议如果存在，通常会采用 GNU Public License（GPL）或 Lesser GNU Public License（LGPL）。前者允许任何人自由使用代码，只要他们自己的代码也同样免费开放；后者则允许这些代码甚至被用于专有的营利性应用。开源项目也可以采用许多其他免费或半免费的授权方案。

网络上有数量惊人的开源引擎。有些相当不错，有些平平无奇，还有些则非常糟糕！在线提供的游戏引擎列表 [72] 会让你感受到可用引擎数量之多。（列表 [73] 稍微更容易消化一些。）这两个列表都包括开源和商业游戏引擎。

OGRE 是一个架构良好、易学且易用的 3D 渲染引擎。它拥有一个功能完整的 3D 渲染器，包括高级光照和阴影；一个优秀的骨骼角色动画系统；一个用于抬头显示和图形用户界面的二维叠加系统；以及一个用于 bloom 等全屏效果的后处理系统。OGRE 按其作者自己的说法，并不是一个完整游戏引擎，但它确实提供了一个功能相当完整的游戏引擎所需的许多基础组件。

Godot 是一个功能完整、跨平台、免费且开源的游戏引擎。该引擎围绕其称为节点（nodes）的构建块进行设计。它支持使用自定义组件扩展引擎，并鼓励在开发社区中共享代码。Godot 的渲染引擎支持从桌面 PC 到移动设备的高端和低端设备。它提供了一种内置脚本语言 GDScript，其语法受到 Python 启发。Godot 被设计为可以使用 C# 编程，但也提供 Rust、Nim、Python 和 JavaScript 的语言绑定。

其他一些知名开源引擎如下：

- Panda3D 是一个基于脚本的引擎。该引擎的主要接口是 Python 自定义脚本语言。它旨在让 3D 游戏和虚拟世界的原型开发变得方便而快速。

- Yake 是一个构建在 OGRE 之上的游戏引擎。

- Crystal Space 是一个拥有可扩展模块化架构的游戏引擎。

- Torque 和 Irrlicht 也是知名的开源游戏引擎。

- Lumberyard 引擎虽然从技术上说并不是开源的，但它确实向开发者提供源代码。它是 Amazon 开发的免费跨平台引擎，基于 CRYENGINE 架构。

### 1.4.13 面向非程序员的 2D 游戏引擎

随着休闲网页游戏和 Apple iPhone/iPad、Google Android 等平台上的移动游戏近年爆发式增长，二维游戏变得极其流行。许多流行的游戏/多媒体创作工具包已经出现，使小型游戏工作室和独立开发者能够为这些平台创建 2D 游戏。这些工具包强调易用性，并允许用户通过图形用户界面创建游戏，而不需要使用编程语言。可以查看这个 YouTube 视频 [74]，了解你可以用这些工具包创建哪些类型的游戏。

- *Clickteam Fusion 2.5* [75] 是 Clickteam 开发的 2D 游戏/多媒体创作工具包。Fusion 被行业专业人士用于创建游戏、屏幕保护程序和其他多媒体应用。Fusion 及其更简单的对应产品 The Games Factory 2，也被 PlanetBravo [76] 这样的教育营地用于教授孩子们游戏开发和编程/逻辑概念。Fusion 支持 iOS、Android、Flash 和 Java 平台。

- *Game Salad Creator* [77] 是另一个面向非程序员的图形化游戏/多媒体创作工具包，在许多方面与 Fusion 类似。

- *Scratch* [78] 是一个创作工具包和图形化编程语言，可用于创建交互式演示和简单游戏。它是年轻人学习条件语句、循环和事件驱动编程等编程概念的好方法。Scratch 于 2003 年由 Lifelong Kindergarten 小组开发，该小组由 MIT Media Lab 的 Mitchel Resnick 领导。
