# 第 4 章：支持开源和原生监测

到目前为止，我们已经从数据的角度讨论了现代可观测性。但是，现代可观测性还有另一个方面，从长远来看，它可能被证明同样重要：如何对待产生数据的仪表（instrumention）。

大多数软件系统都是用现成的部件构建的：网络框架、数据库、HTTP 客户端、代理服务器、编程语言。在大多数组织中，很少有这种软件基础设施是在内部编写的。相反，这些组件是在许多组织中共享的。最常见的是，这些共享组件是以开放源码（OSS）库的形式出现的，并具有许可权。

由于这些开放源码软件库几乎囊括了一般系统中的所有关键功能，因此获得这些库的高质量说明对大多数可观测性系统来说至关重要。

传统上，仪表是 "单独出售" 的。这意味着，软件库不包括产生追踪、日志或度量的仪表。相反，特定解决方案的仪表是在事后添加的，作为部署可观测系统的一部分。

> **什么是特定解决方案仪表？**
>
> 在本章中，术语 "**特定解决方案仪表**" 是指任何旨在与特定的可观测系统一起工作的仪表，使用的是作为该特定系统的数据存储系统的产物而开发的客户端。在这些系统中，客户端和存储系统常常深深地融合在一起。因此，如果一个应用要从一个观测系统切换到另一个观测系统，通常需要进行全面的重新布设。

针对解决方案的仪表是 "三大支柱" 中固有的垂直整合的遗留问题。每个后端都摄取特定类型的专有数据；因此，这些后端的创建者也必须提供产生这些数据的仪表。

这种工具化的方法给参与软件开发的每个人都带来了麻烦：供应商、用户和开放源码库的作者。

## 可观测性被淹没在特定解决方案的仪表中

从可观测性系统的角度来看，仪表化代表了巨大的开销。

在过去，互联网应用是相当同质化的，可以围绕一个特定的网络框架来建立可观测性系统。Java Spring、Ruby on Rails 或 .NET。但随着时间的推移，软件的多样性已经爆炸性增长。现在为每一个流行的网络框架和数据库客户端维护仪表是一项巨大的负担。

这导致的重复劳动难以估量。传统上，供应商将他们在仪表上的投资作为销售点和把关的一种形式。但是，日益增长的软件开发速度已经开始使这种做法无法维持了。对于一个合理规模的仪表设备团队来说，覆盖面实在是太大了，无法跟上。

这种负担对于新的、新颖的观测系统，特别是开放源码软件的观测项目来说尤其严峻。如果一个新的系统在编写了大量的仪表之前无法在生产中部署，而一个开放源码软件项目在广泛部署之前也无法吸引开发者的兴趣，那么科学进步就会陷入僵局。这对于基于追踪的系统来说尤其如此，它需要端到端的仪表来提供最大的价值。

## 应用程序被锁定在特定解决方案的仪表中

从应用开发者的角度来看，特定解决方案的仪表代表了一种有害的锁定形式。

可观测性是一个交叉性的问题。要彻底追踪、记录或度量一个大型的应用程序，意味着成千上万的仪表 API 调用将遍布整个代码库。改变可观测性系统需要把所有这些工具去掉，用新系统提供的不同工具来代替。

替换仪表是一项重大的前期投资，即使只是为了尝试一个新的系统。更糟的是，大多数系统都太大了以致于在所有服务中同时更换所有仪表是不可行的。大多数系统需要逐步推出新的仪表设备，但这样的上线可能很难设计。

被特定解决方案的仪表所 "困住" 是非常令人沮丧的。在可观测性供应商开始努力提供自己的仪表的同时，用户也开始拒绝采用这种仪表。由于了解到重新安装仪表的工作量，许多用户强烈希望他们正在考虑的任何新的观测系统能与他们目前使用的仪表一起工作。

为了支持这一要求，许多可观测性系统试图与其他几个系统提供的仪表一起工作。但这种拼凑的方式降低了每个系统所摄取的数据的质量。从许多来源摄取数据意味着对输入的数据不再有明确的定义，当预期的数据不均衡且定义模糊时，分析工具就很难完成它们的工作。

## 针对开源软件的特定解决方案的仪表基本上是不可能的

从一个开放源码库作者的角度来看，特定解决方案的仪表化是一个悲剧。

来自开放源码软件库的遥测数据对于操作建立在它们之上的应用程序来说至关重要。最了解哪些数据对操作至关重要的人，以及操作者应该如何利用这些数据来补救问题的人，就是实际编写软件的开源库的开发者。

但是，库的作者却陷入了困境。正如我们将看到的，没有任何一个特定解决方案的仪表化 API，无论写得多么好，都无法作为开放源码库可以接受的选择。

## 如何挑选一个日志库？

假设你正在编写世界上最伟大的开源网络框架。在生产过程中，很多事情都会出错，你自然希望把错误、调试和性能信息传达给你的用户。你使用哪个日志库？

有很多体面的日志库。事实上，有很多，无论你选择哪个库，你都会有很多用户希望你选择一个不同的库。如果你的 Web 框架选择了一个日志库，而数据库客户端库选择了另一个，怎么办？如果这两个都不是用户想要使用的呢？如果他们选择了同一个库的不兼容的版本呢？

没有一个完美的方法可以将多个特定解决方案的日志库组合成一个连贯的系统。虽然日志足够简单，不同解决方案的大杂烩可能是可行的，但对于特定解决方案的指标和追踪来说，情况并非如此。

因此，开放源码软件库通常没有内置的日志、度量或追踪功能。取而代之的是，库提供了 "可观测性钩子"，这需要用户编写和维护一堆适配器，将使用的库连接到他们的可观测性系统上。

作者们有大量的知识，他们想通信关于他们的系统应该如何运行的知识，但他们没有明确的方法去做。如果你问任何写过大量开源软件的人，他们会告诉你。这种情况是痛苦的！而且是不幸的！一些库的作者**确实**试图选择一个日志库，但却发现他们无意中为一些用户造成了版本冲突，同时迫使其他用户编写日志适配器来捕捉使用其**实际**日志库的数据。

但是对于大多数库来说，可观测性只是一个事后的想法。虽然库的作者经常编写大量的测试套件，但他们很少花时间去考虑运行时的可观测性。考虑到库有大量的测试工具，但可观测性工具为零，这种结果并不令人惊讶。

正如我们在接下来的几章中所看到的，现代可观测性的设计是为了使在可观测性管道中发挥作用的每个人的代理权最大化。但受益最大的是库的作者；对于特定解决方案的仪表，他们目前根本没有选择。

## 分解问题

我们可以通过设计一个可观测性系统来解决上面列出的所有问题，以明确地解决每个人的需求。在本章的其余部分，我们将把现代观测系统的设计分解为基本要求。这些要求将为第五章中描述的 OpenTelemetry 的结构提供动力。

## 要求：独立的仪表、遥测和分析

归根结底，计算机系统实际上就是人类系统。像可观测性这样的跨领域问题，几乎与每一个软件组件都有互动。同时，传输和处理遥测数据可能是一个大批量的活动，以至于一个大规模的观测系统会产生自己的操作问题。这意味着，许多不同的人，以不同的身份，需要与观测系统的不同方面交互。为了很好地服务于他们，这个系统必须确保每个参与其中的人都有他们所需要的代理权，以便快速和独立地执行任务。提供代理权是设计一个有效的观测系统的基本要求。

让我们首先确定与运行中的软件系统有关的每个角色的责任：库的作者、应用程序的所有者、操作者和响应者。

**库的作者了解他们软件的情况**

对于封装了关键功能的软件库，如网络和请求管理，库的作者也必须管理追踪系统的各个方面：注入、提取和上下文传播。

**应用程序拥有者组织软件并管理依赖关系**

应用程序所有者选择构成其应用程序的组件，并确保它们编译成一个连贯的、有功能的系统。应用程序所有者还编写应用程序级别的工具，它必须与库作者提供的指令（和上下文传播）进行正确的交互。

**运维人员管理遥测的生产和传输**

运维管理从应用到响应者的可观测性数据的传输。他们必须能够选择数据的格式以及数据的发送地点。当数据在产生时，他们必须操作传输系统：管理缓冲、处理和传送数据所需的所有资源。

**响应者消费遥测数据并产生有用的见解**

要做到这一点，应对者必须了解数据结构及其内在意义（结构和意义将在第三章中详细描述）。当新的和改进的分析工具出现时，反应者还需要将其添加到他们的工具箱中。

这些角色代表不同的决策点：

- 库的作者只能通过发布其代码的新版本来进行修改。
- 应用程序所有者只能通过部署其可执行文件的新版本来进行更改。
- 运维人员只能通过管理可执行文件的拓扑结构和配置来进行改变。
- 响应者只能根据他们收到的数据做出改变。

传统的三大支柱方法扰乱了所有这些角色。垂直整合的一个副作用是，几乎所有的数据变化都需要进行代码修改。几乎任何对可观测性系统的非微不足道的改变都需要应用程序所有者进行代码修改。要求其他人进行你所关心的改变，这样会有很大的阻力，并可能导致压力、冲突和不作为。

显然，一个设计良好的可观测性系统应该侧重于允许每个人尽可能多的代理和直接控制，它应该避免将开发者变成意外的看门人。

## 要求：零依赖性

应用程序是由依赖关系（网络框架、数据库客户端），加上依赖关系的依赖关系（OpenTelemetry 或其他仪表库），加上它们的依赖关系的依赖关系的依赖关系（无论这些仪表库依赖什么）组成。这些都被称为反式的依赖关系。

如果任何两个依赖关系之间有冲突，应用程序就无法运行。例如，两个库可能分别需要一个不同的（不兼容的）底层网络库的版本，如 gRPC。这可能会导致一些不好的情况。例如，一个新版本的库可能包括一个需要的安全补丁，但也包括一个升级的依赖关系，这就产生了依赖关系冲突。

诸如此类的过渡性依赖冲突给应用程序所有者带来了很大的麻烦，因为这些冲突无法独立解决。相反，应用程序所有者必须联系库的作者，要求他们提供一个解决方案，这最终需要时间（假设库的作者回应了这个请求）。

为了使现代可观测性发挥作用，库必须能够嵌入仪表，而不必担心当他们的库被用于组成应用程序时而导致问题。因此，可观测性系统必须提供不包含可能无意中引发横向依赖冲突的依赖性的仪表。

## 要求：严格的后向兼容和长期支持

当一个仪表化的 API 破坏了向后的兼容性，坏事就会发生。一个精心设计的应用程序最终可能会有成千上万的仪表调用站点。由于 API 的改变而不得不更新数以千计的调用站点是一个相当大的工作量。

这就产生了一种特别糟糕的依赖性冲突，即一个库中的仪表化不再与另一个库中的仪表化兼容。

因此，仪表化 API 必须在很长的时间范围内具有严格的向后兼容能力。理想的情况是，仪表化 API 一旦变得稳定，就永远不会破坏向后兼容。新的、实验性的 API 功能的开发方式必须保证它们的存在不会在包含稳定仪表的库之间产生冲突。

## 分离关注点是良好设计的基础

在下一章中，我们将深入研究 OpenTelemetry 的架构，看看它是如何满足上面提出的要求的。但在这之前，我想说的是一个重要的问题。

如果你分析这些需求，你可能会注意到一些奇特的现象：它们中几乎没有任何专门针对可观测性的内容。相反，重点是尽量减少依赖性，保持向后的兼容性，并确保不同的用户可以在没有无谓干扰的情况下发挥作用。

每一个要求都指出了关注点分离是一个关键的设计特征。但是这些特性并不是 OpenTelemetry 所独有的。任何寻求广泛采用的软件库都会很好地包括它们。在这个意义上，OpenTelemetry 的设计也可以作为设计一般的开源软件的指南。下次当你开始一个新的开放源码软件项目时，请预先考虑这些要求，相应地设计你的库及你的用户会感谢你的。