# 第 3 章 BIOS 完整性测定中的功能组件

下列章节为 BIM 的功能组件提供了描述以及与之相关的要求的指导意见：

* 3.1 节探讨可信根（RoT）
* 3.2 节定义了 BIOS 完整性属性和测定数据基线的建立
* 3.3 节探讨 BIOS 完整性报告
* 3.4 节定义了 BIM 数据的采集及其传输至 MAA 的过程
* 3.5 节描述并定义了对 MAA 本身的要求
* 3.6 节探讨了基于所采集和传输的测定数据和属性的潜在恢复行动

## 3.1 可信根（RoT）

可信根（RoT）位于任何 BIOS 完整性担保的基础地位。_RoT_ 是构成一组被无条件信任的功能的组件（软件、硬件，或者混合）和计算引擎，并且必须总是以一种期望中的行为方式运作，由于它们的非正常行为不能被检测到。它们对于 BIOS 完整性测定尤其重要，由于 BIOS 在大体上负责软件执行环境的配置，包括物理内存和设备的映射。

撬动一个或更多的 RoT 的软件代理的可信度取决于 RoT 本身的可信度及其攻击面。全套的 RoT 至少拥有最小的一组能力以允许测定、存储并报告终端机的那些影响其可信度的特征。硬件 RoT 优于软件 RoT，由于它被发现能够在高得多的比例的攻击场景中以某种期望中的方式运作。

可靠和可信的 BIOS 完整性测定和报告取决于软件代理以执行那些验证者所必须依赖的功能。每个软件代理都依赖 RoT，并且代理的可信度级别取决于它的 RoT。任何软件代理的 RoT 是这样的一些计算引擎，它们要么能够可靠和准确地执行该代理的功能，要么负责产生可靠的软件代理以执行此任务。因此，某个代理可以被完全实现于 RoT 内部，或者可以通过一条植根于 RoT 中的信任链由它的 RoT 生成。因此，代理的可信度取决于它所对应的 RoT 以及信任链。

BIOS 完整性测定的最高担保机制提供可变状态容错，在此，此机制能够提供关于 BIOS 及其配置的可靠和可信的报告，而不论设备的总体配置或者 BIOS 存储设备的内容如何。此外，BIOS 完整性测定的最高担保机制提供内存泄露容错，在此，由此机制报告的测定数据必须是可证明地新鲜的（即最近采集的，一般是由于某次具体的请求），并且没有被任何相关方伪造，即使他们完全知道在所有过去的软件执行过程中曾经存在于软件可读内存中的全部内容。因此，这样的机制对于针对软件漏洞的利用更加强壮，并且不会受到存在于软件可读内存中的密码学密钥的泄露的影响。

### 3.1.1 软件代理概述

BIM 需要以下组件的协作：一个测定代理用于收集测定数据、一个存储代理用于保护测定数据不被修改，直到它们可以被报告，以及一个报告代理用于可靠地报告这些测定数据。如果测定数据是在收集之后立即报告的，可能并不需要使用存储代理。保证 BIOS 完整性的完备的系统还会包含对于所采集到的测定数据的某种形式的验证功能，这相应地需要使用验证代理以评估被报告的测定数据。这样的验证代理可以驻留于被测定的终端机上，或者位于管理服务或者基础设施外部。此外，如果任何软件代理可以被修改或者更新，那么这些代理的可信度取决于用于实现这些更新的机制。因此，如果这样的更新是可能的，此终端机应该包括更新代理以便以某种可靠的方式来实施软件更新，以提供相当于 \[NIST-SP800-147\] 中的保护。

### 3.1.2 可信根概述

_测定可信根_（_RTM_）是一种能够进行内在地可靠的完整性测定的计算引擎。它是后续的测定代理的传递性的信任链的根基。在终端机重新初始化之后很快被应用的小型 RTM 可能比之后被实例化的 RTM 拥有更大价值，这主要在于使得测定过程暴露给破坏行为的攻击面最小化。终端机调用 RTM 越晚，对手所拥有的破坏测定信任链的机会就越大。而 RTM 越大，其实现中存在能够为对手提供机会以破坏 RTM 的瑕疵的机会就越大。

_存储可信根_（_RTS_）是一种计算引擎，它能够维持一种使得破坏显而易见的，关于完整性测定值以及这些测定的序列的总结。它并不包括完整性测定序列的细节，而是存储这些序列的完整性散列值。这些完整性散列值可以被用于验证一段包含完整性测定值和这些测定序列的日志的完整性，或者被用于该日志的指标。RTS 在使得破坏显而易见的位置存储这些完整性散列值，例如，其接口可能允许寄存器扩展（定义于 3.2.2.2 节），但不允许直接寄存器写入。这些授权的接口通常称为 _受保护的能力_。使破坏显而易见的位置和受保护的能力共同构成了 RTS。

_报告可信根_（_RTR_）是一种计算引擎，它能够可靠地报告由 RTM 及其测定代理提供或者由 RTS 存储的信息。RTR 作为测定数据报告的完整性和不可抵赖性的能力的基础，它必然会撬动 RTM 和 RTS。对于 RTR 的关键要求之一是终端机以及正在被测定和报告的组件的无歧义的身份，这种身份可以是持久或者临时的。对测定报告数据利用密钥签名是一种常见的机制以提供无歧义的身份。密钥的证书可以认证组织的成员身份或者识别某个特定成员。

### 3.1.3 代理的协作

所有相关代理共同协作的可信度为验证代理对被测定的终端机的 BIOS 完整性进行可靠的评估提供了保证。对由组合系统提供的信任属性进行评估是至关重要的，而对单个代理的信任属性的评估仅在整个系统的上下文环境中才是相关的。

图 2 展示了各个 RoT 必须协同作用并且彼此基于对方构建以允许对 BIM 数据进行可靠和可信的测定、报告和验证。此外，这些 RoT 必须适当地整合完整性保护和安全 BIOS 更新认证的机制。这些 RoT 以及其他终端机安全性机制的实现可以通过集成式的方式，在此，它们都由单一的软件组件，诸如 BIOS 构成。不论此实现是整体式的还是分散式的，这些 RoT 的组合以及它们所基于其构建的 RoT 的组合构成了完整的 BIM 解决方案的可信度的基础。

> 图 2：RoT 和代理的协作

### 3.1.4 可信根的安全指导意见

1. 终端机厂商**将要**提供实施可信的 RTM、RTS 和 RTR 所必需的硬件支持，并且利用该支持为 BIM 实现对应的 RoT
2. 终端机厂商**将要**整合 RTM、RTS 和 RTR 的机制以形成集成式的、可靠的 BIM 基础
3. 终端机厂商**应该**为受到完整性保护的，不可绕过的认证或者安全 BIOS 更新实现提供机制，如同符合 \[NIST-SP800-147\] 所必需的
4. 终端机厂商**可以**为实现用于启动时之后的测定的可信的动态 RTM 提供硬件支持。终端机厂商和操作系统厂商**可以**利用此支持为 BIM 实现这样的 RoT。

## 3.2 完整性属性和测定数据的基线

有意义的完整性测定数据比较方案中的一个关键因素是可靠地建立并维持一组关于属性和测定数据的已知基线，以使得人们可以利用它进行决策。基线的建立必须考虑两种场景：具有未知或者可疑来源的 BIOS 的终端机设备，以及具有已知和可信（受保护和签名）的 BIOS 的终端机设备。本节列举了用于基线的属性和测定数据。

### 3.2.1 BIOS 完整性属性概述

属性被用于评估 BIOS 完整性测定数据的可信度。终端机厂商拥有多种方式将属性传达给用户。终端机厂商可以将这些属性连同黄金 BIOS 测定数据放在证书中，通过带外通讯信道（即并非利用终端机本身传达，而是通过其他方式）呈现给用户。终端机厂商可以允许用户直接查询此终端机并且提取属性。或者，终端机厂商可以允许用户将序列号提交至受管理的在线客服，后者以该终端机在送达时的属性的列表作为响应。无论用户如何获取某一特定终端机的属性，这些属性存在的理由是为用户提供一种方式以评估由终端机报告的 BIM 数据的有效性，以及在其接收到的报告中开发一种可信度级别，关于该终端机的仅存于 BIM 数据以外的总体健康状况。

#### 3.2.1.1 BIOS 完整性属性列表

本节列举了 BIOS 完整性属性并且提示它们的可能的值。

1. **RTM**——RTM 的属性提供了关于在 BIOS 测定发生之前可供对手利用的时间/代码量的信息，通常被称为攻击面的大小。终端机能够越早地撬动 RTM 并且进行 BIOS 测定，攻击面就越小，并且评估者能够在其测定数据中拥有更高的可信度。反之，在进行 BIOS 测定之前，终端机在初始化或者重新初始化之后的越晚的时间撬动 RTM，攻击面就越大，并且评估者在其测定数据中拥有的可信度就越少。关于 RTM 的属性的值的范例包括：
    * 第 1 类（即终端机将 RTM 的实例化作为早期 BIOS 加电自检初始化的一部分）
    * 第 2 类（即终端机在早期 BIOS 加电自检初始化之后才进行 RTM 实例化）
2. **RTS**——RTS 的属性提供了关于由能够使得破坏显而易见的存储区域提供的保护的信息。RTS 越有可能抵御来自对手的攻击，则评估者就能在其包含的测定数据中拥有更高的可信度。RTS 的值可以取如下几种：
    * 第 1 类：基于硬件的（例如 TPM 1.2，基于处理器的受保护存储）
    * 第 2 类：基于软件的（例如：操作系统的密钥存储器，系统管理内存（SMRAM））
3. **RTR**——RTR 的属性提供了关于可提供给以下内容的保护的信息：将要报告的数据、用于提供认证的密钥素材、利用密钥进行签名的代码，以及不可抵赖性的服务。RTR 越有可能抵御来自对手的攻击，则评估者就能在报告中拥有更高的可信度。RTR 属性的值可能包括：
    * 第 1 类：基于硬件的（例如 TPM 1.2，基于处理器的）
    * 第 2 类：基于软件的（例如：操作系统，BIOS 启动块和存储于 BIOS 中的密钥）
4. **BIOS 签名存在**——这是关于 BIOS 是否被签名以及签名是否存在于 BIOS 中的指示器。其值包括：
    * 无（BIOS 未被签名）
    * 部分（某些 BIOS 组件被签名，并且签名存在）
    * 完全（所有 BIOS 组件被签名，并且签名存在）
5. **BIOS 更新机制**——此属性提供了关于厂商所使用的 BIOS 更新机制类型的指示器。此属性的重要性在于 BIOS 提供商在提供最新的 BIOS 代码和数据更新时的勤奋度。在安全性的上下文环境中，可靠的 BIOS 更新机制的存在性为最终用户提供了这样的担保，即一旦需求出现，BIOS 提供商可以安全地将其产品从威胁中恢复。拥有不可靠的更新机制或者更新机制不存在的 BIOS 可能处于悬疑之中。其值包括：
    * 无（不可能更新 BIOS）
    * 胶囊式（从操作系统中将新的 BIOS 写入内存，然后执行系统重启）
    * 系统管理模式（SMM）（从操作系统中利用 SMM 以修改 BIOS，然后继续运行而无需重启系统；重启对于实现新的 BIOS 必要）
    * 其他
6. **虚拟化**——此属性提供了关于 BIOS 是来自终端机硬件还是作为虚拟机（VM）的一部分而被提供的指示器。其值包括：
    * 无（物理 BIOS 存在）
    * 完全（虚拟 BIOS 正在运行）
7. **闪存锁定**——此属性提供了关于终端机保护 BIOS 所驻留于其中的闪存以防止非授权修改的能力的指示器。此属于通常指代某种硬件能力。此属性的重要性在于 BIOS 提供商对 BIOS 代码和数据提供保护以防止非授权修改的勤奋度。在安全性的上下文环境中，可靠的 BIOS 闪存保护的存在性为最终用户提供了关于 BIOS 测定的可信度。具有不可靠的闪存保护或者不存在闪存保护的 BIOS 可能处于悬疑之中。其值包括：
    * 无
    * 芯片组
    * 闪存芯片原生

#### 3.2.1.2 可信根属性的安全指导意见

1. 终端机厂商**将要**提供定义于 3.2.1.1 节的属性
2. 终端机厂商**将要**在其用于提供更新和维护的最低粒度级别上提供关于可执行 BIOS 启动代码的参考测定数据，并且它们可以用于验证从 RTR 返回的测定数据
3. 终端机厂商**将要**提供用于测定和报告 BIOS 配置数据的测定数据基线的机制
4. 终端机厂商**应该**部署那些并不排除扩展至系统 BIOS 外部的 Option ROM 的 BIOS 测定和报告机制
5. 终端机厂商**应该**以标准化的格式提供属性
6. 终端机厂商**可以**提供关于是否符合 \[NIST-SP800-147\] 的指示器。

### 3.2.2 BIOS 完整性测定数据概述

本节定义了所有终端机都应该能够报告的一组最小的基本 BIM 数据。本节探讨这些测定数据的格式以及它们如何生成。

**BIOS 启动代码完整性测定数据**——BIOS 启动代码完整性测定数据要么是 BIOS 启动代码的一段密码学散列值，要么是 BIOS 启动代码模块的一段扩展的密码学散列值。无论何种情况，它可以被简单地称为“散列值”。此散列值包含以下组件，如果它们被实现：

* BIOS 启动块
* SMM 代码
* 高级配置与电源接口（ACPI）代码
* 加电自检（POST）代码
* BIOS 恢复机制代码
* BIOS 更新机制代码
* BBM 软件
* RTM
* 嵌入式 Option ROM（这些 Option ROM 嵌入在 BIOS 启动代码中并且由 BIOS 厂商控制）

**BIOS 配置数据完整性测定数据**——BIOS 配置数据完整性测定数据要么是 BIOS 中的用户可配置的数据组件的一段密码学散列值，要么是独立的配置数据组件的一段扩展的密码学散列值。所有这样的 BIOS 配置数据都必须被测定，除了自动更新的终端机配置信息，诸如时钟寄存器，以及系统独特的信息，诸如财产编号或者序列号以外。以上这些必须不被测定到相同的寄存器中。如果要测定，系统独特的信息，诸如财产编号、序列号、口令散列值等应该被测定到不同的私有寄存器中。这允许在相同配置的系统之间，以及在同一台系统的多次启动测定之间建立起具有一致性的测定数据，也允许系统管理员确定口令是否被更改。

在实践中，对来自同一品牌的同一型号的不同终端机的 BIOS 组件进行现场完整性测定，将会得到不同的和独特的值。这看起来是反直观的，特别是对于静态 BIOS 组件的完整性测定。黄金 BIOS 测定数据的缺失使得进一步调查变得困难，特别是在区分受到攻击的和合法的 BIOS 时。

在实践中，RoT 维护 BIM 数据的组合。尽管规范建议了独立的完整性测定数据应该如何存储和保存，标准的缺失使得解读这些测定数据用于取证变得困难。

#### 3.2.2.1 BIOS 完整性测定数据的生成和存储

为了监测并改进 BIOS 完整性，BIOS 完整性测定数据必须开始被安全地生成和存储。这些测定数据可以随后被传输、分析、报告，以及用于提供恢复。本节描述终端机如何生成并存储 BIM 数据。

终端机的 RTM 必须作为测定信任链的根基。通过使得终端机的 RTM 测定一个或者更多的二进制组件并且在执行这些二进制组件之前将这些测定数据记录在安全的位置，终端机形成了测定信任链。接下来，终端机通过仅仅将测定委托给某些组件来维持测定信任链，这些组件要么是由终端机的 RTM 测定的，要么是由其测定过程可以回溯至终端机的 RTM 的其他组件测定的。测定信任链应该先于终端机使用它们测定所有 BIOS 配置数据（非可执行的二进制数据）。

#### 3.2.2.2 BIOS 完整性测定数据寄存器

为了清楚起见，此文档创造并且使用了若干种寄存器“类型”。“寄存器”一词本意是为了解释所需的不同类型的存储能力以及它们如何使用，而非定义任何特别的实现架构或者命名惯例。

由于测定信任链测定 BIOS 代码和数据，终端机必须将测定数据存储于可信的位置，它将会再随后提供监测和查看，只要终端机管理员想要监测和查看。终端机的 RTM 和测定信任链将测定数据存储于包含在 RTS 中的完整性测定数据寄存器（IMR）中。它们将会计算 BIOS 组件（代码和数据）的散列值，使用某种批准的密码学散列算法，诸如发布于 \[NIST-FIPS180-3\] 中的算法之一。IMR 应该拥有足够的长度以容纳选自批准的列表中的散列算法的全部结果。

BIOS 完整性测定的实现者**将要**把各个 IMR 初始化为某个已知的固定值。它们可以一次性地为一个寄存器计算欲测定的所有 BIOS 组件的散列值，并且将此结果扩展到 BIOS IMR 中。或者，实现者可以创建组合的散列值，通过连续地逐个计算 BIOS 组件的散列值，并且为每个组件按照如下方式来扩展 BIOS 代码的 IMR：

_IMR_<sub>_new_</sub> = _H_\(_IMR_<sub>_old_</sub> \|\| _H_\(_BIOS_<sub>_component_</sub>\)\)

_IMR_<sub>_new_</sub> 是新的 IMR

_IMR_<sub>_old_</sub> 是 IMR 的旧值

_BIOS_<sub>_component_</sub> 是待测定的 BIOS 组件

_H_ 是选自批准的列表中的一种散列算法

\|\| 是拼接运算符

当测定 BIOS 数据时，**可选**地使用某种某种带密钥的散列算法，诸如 HMAC。随机化的散列算法**可以**被应用以获取额外的防碰撞性，诸如 \[NIST-SP800-106\] 中发布的某种算法。在此情况下，IMR 的大小**必须**能够容纳随机化散列算法的输出，连同验证这些散列值所必需的额外参数。

在任何 BIOS 组件的测定发生之前，终端机**将要**把各个 IMR 初始化为某个已知的固定值。在理想情况下，终端机的加电即可初始化这些寄存器。只有那些导致 BIOS 被完整执行的终端机重置**应该**导致这些寄存器的重置。终端机从挂起调用（通常称为待机和休眠模式）中的恢复不会初始化 IMR，而是恢复其在挂起过程中所保持的有效值。

终端机将所有 BIOS 可执行组件的测定数据累积至一个 IMR 中。类似地，终端机为所有 BIOS 配置数据组件执行相同的过程，除了诸如口令、序列号、财产编号、密钥素材等隐私敏感数据，以及诸如时间等元素以外，这些元素使得制作黄金测定数据成为不可能。这些被排除的项目往往是对于平台独特的，并且可能导致管理员已经以相同方式配置的平台投射出不同的配置数据 IMR。并且，由于这些项目是隐私敏感的，管理员可能想要选项以配置这些值的测定和报告。终端机**将要**把 BIOS 可执行代码的测定数据累积至 BIOS 代码完整性测定数据寄存器（CIMR），**将要**把 BIOS 数据的测定数据累积至 BIOS 数据完整性测定数据寄存器（DIMR），并且**应该**将 BIOS 隐私敏感数据累积至 BIOS 隐私完整性测定数据寄存器（PIMR）。

#### 3.2.2.3 BIOS 测定数据日志

除了将 BIOS 代码和数据的测定数据分别扩展至 CIMR 和 DIMR 以外，BIOS 功能还**可以**将可选的测定数据和事件扩展到它们当中来。为了帮助梳理位于寄存器中的测定数据，管理员将会在一段描述了扩展到它们当中的测定数据和事件的日志中发现巨大的价值。一些例子包括：

* 终端机厂商以模块的形式维护并更新 BIOS，并且在模块基础上编写 BIOS 代码以测定并且扩展一个模块上的 CIMR。日志将会帮助 MAA 检测这样的情况，其中一次或者多次的错误或者意料之外的测定数据被扩展至 CIMR 中
* 如果终端机包含多于一份 BIOS 镜像，例如，一份运行的镜像和一份备份，管理员可能想要获得关于哪份副本被测定并且扩展到 CIMR 的额外细节。BIOS 功能的某个早期阶段可能会将一个发出了所作选择的信号的“事件”连同其在日志中的对应项目扩展至 CIMR 中
* BIOS 可能想要刚好在将执行权移交给某个后启动环境之前对测定数据扩展进行“封端”。通过将某个“封端”事件扩展至 CIMR 或者 DIMR 并且在日志中记录对应事件，管理员可以检测在终端机初始化过程中的哪些阶段发生了哪些事件
* BIOS 配置数据拥有众多排列方式。一段记录了那些涉及 BIOS 配置数据的测定数据和事件的事件日志如果被适当地格式化，可以有助于对意料之外的更改进行编程式的审查。

一段存储的测定数据日志（SML）允许终端机的更高级别的代理，诸如报告和采集代理在将其转发至 MAA 之前采集它们以用于预处理和相互关联。由于这些代理通常运行于后启动环境中，BIOS**应该**在将执行权移交至新的环境时注意保留 SML 中的内容。

SML 测定数据通常拥有比 CIMR 或者 DIMR 测定值更好的粒度，但是并不拥有用于包含原始 BIOS 固件或者配置数据的最精细的粒度。尽管系统**将要**使得 SML 数据可用于报告，它**应该**提供某种方式以编程式地报告并解读 BIOS 配置数据。TCG 在 \[TCG-ConvBiosSpec\] 中提供了一种用于 SML 日志项目的标准格式。

#### 3.2.2.4 关于 BIOS 完整性测定数据的生成和存储的安全性指导意见

1. 被测定的组件
    1. 终端机**将要**测定所有 BIOS 可执行组件
    2. 终端机**将要**测定所有 BIOS 配置数据组件，除了隐私敏感数据以外
    3. 终端机**应该**独立于其他 BIOS 配置数据组件来测定隐私敏感数据
2. BIOS 测定数据的生成和存储
    1. 终端机**将要**支持启动时测定
    2. 终端机**将要**使用某个 RTM 或者植根于某个 RTM 中的测定信任链中的某个组件来生成 BIOS 完整性测定数据
    3. 终端机**将要**使用批准的标准密码学技术以生成 BIOS 完整性测定数据
    4. 终端机**将要**使用标准的 BIOS 完整性测定数据格式
    5. BIM 数据的格式和报告——黄金测定数据和终端机测定数据——**将要**使用开放格式或者规范，诸如 \[TCG-Manifest\] 和 \[TCG-Integrity\] 中的
    6. 终端机厂商**应该**使用可扩展的系统以允许添加对于 Option ROM 等的测定支持
    7. 终端机**将要**在 RTS 中存储 BIM 数据
    8. 终端机厂商**应该**使得原始 BIOS 配置数据可用于报告，并且**应该**拥有某种方式以解读该可用的数据
3. 测定数据日志
    1. BIOS 功能**将要**把 SML 测定数据和事件扩展记录至 CIMR 和 DIMR 中
    2. BIOS 功能**应该**把 SML 测定数据和事件扩展记录至 PIMR 中
    3. 在将 BIM 数据和其他 BIOS 相关事件记录于 SML 中时，BIOS 功能**将要**使用一种标准格式，诸如 \[TCG-ConvBiosSpec\]
    4. BIOS **将要**保证恰当地将 SML 移交至后启动环境
4. 测定数据支持
    1. 操作系统厂商**将要**支持一种标准应用程序接口（API）以用于采集测定数据
    2. 终端机、操作系统和应用程序厂商**将要**支持一种 RTR 以用于采集和报告 BIM 数据

#### 3.2.2.5 相关标准和规范

有若干种与这些要求相关的记录标准。\[NIST-SP800-147\] 规范详细说明了获得并维持一种 RTM 所依赖的不可变的 BIOS 启动块所必需的条件。\[TCG-ConvBiosSpec\] 规范定义了 RTM（即 BIOS）应该对哪些 BIOS 组件进行测定并且将其存储于 RTS（例如 TPM）中。TCG EFI 平台规范版本 1.20 修订版本 1.0 \[TCG-EFI\] 定义了应该由 RTM 或者在 EFI 内部通过扩展信任链进行测定并且存储于 RTS（例如 TPM）中的可扩展固件接口（EFI）组件。最后，TCG 基础设施工作小组参考清单纲要规范版本 1.0 \[TCG-Manifest\] 具体说明了厂商可以用于报告 BIOS 完整性黄金测定数据、终端机测定数据以及 BIOS 完整性测定数据属性的格式。

## 3.3 BIOS 完整性报告

对存储于 RTS 中的 BIOS 完整性测定值的报告由终端机的报告代理执行。

### 3.3.1 终端机报告代理

终端机的 RTR 和植根于 RTR 中的报告代理负责提取存储于 RTS 中的 IMR 中的测定数据。RTR 和报告代理对此数据应用完整性和不可抵赖性的保护，以使得后续代理不能在不被 MAA 检测到的情况下更改或者替换这些数据是至关重要的。此外，无歧义的身份信息是应用于该数据的一项重要属性，以使得 MAA 知道该数据的来源。通常，利用 MAA 所认识的密钥对此数据进行签名即足以满足完整性、不可抵赖性和无歧义的身份的目标。

此外，报告代理还负责为 IMR 数据提供对应的 SML。基于唯一地标识出制造商、型号和版本的组件 ID 来识别所测定的组件，以便允许 MAA 将其测定数据同它的黄金测定数据进行对比是理想的。

终端机报告代理将会得到一个临时随机数，此临时随机数来自 MAA，并且通过传输代理和采集代理而被传递给它。它随后将此临时随机数传递给 RTR，并且收到对于此临时随机数连同存储于 RTS 中的测定数据的散列值表征的签名。它还会得到 SML 并且将此二者整合为一种标准格式。此报告最终必须可以被 MAA 读取。因此：

1. 定义的报告格式**将要**使用 \[TCG-Integrity\]
2. 此报告**应该**包含一份以某种业界标准格式呈现的 BIOS 配置副本，如果想要得到分析并确定此配置的可接受性的能力。

除了以上指导意见以及列出于整个 3.3 节的标准以外，下述 3.3.2 节中的要求适用于将要提供给 MAA 中的验证代理的 BIOS 测定数据报告。

### 3.3.2 报告协议的安全性指导意见

1. BIOS 报告机制**可以**准备并签发一份 BIOS 完整性报告至某个合法的 MAA，当符合下列任何条件时：
    1. 如果 BIOS 配置发生更改
    2. 如果 BIOS 软件发生更改
    3. 如果本地端口被用于访问 BIOS 或其配置
    4. 如果 BIOS 错误条件被生成
    5. 当连接至由某个合法 MAA 控制的网络时
2. BIOS 报告机制**可以**支持配置特定事件以引发 BIOS 完整性测定数据报告的能力
3. 报告请求**可以**由此报告的报告方或者接收方引发。下列要求适用于由任意一方引发的请求：
    1. 机制**将要**能够提供可证明地新鲜的 BIOS 报告以发送给 MAA
    2. 报告**应该**包含一段关于此报告所使用的格式化标准/纲要的引用
    3. 报告格式**应该**包含一个版本字段，每当此报告格式发生扩展或者其他更改时，此字段可以随着时间更新
    4. 所有来自非法实体的签发或者提供报告的请求**应该**被忽略
    5. 提供或者签发报告的请求**可以**包含优先级信息，以指示此请求的紧急程度
    6. 与请求相关的响应时间框架**可以**由授权用户基于优先级进行配置

### 3.3.3 相关标准和规范

BIOS 测定数据需要以一种标准纲要来提供，以使其可以被接收它的实体基于策略使用并且采取行动。因此，下列标准和指南应该被用于建立一种关于报告和提供标准报告格式以及 BIOS 测定数据传输的信任模型。

TCG 基础设施工作小组的完整性报告纲要规范 \[TCG-Integrity\] 定义了在不同的实体之间进行完整性信息通讯所用的结构。TCG 可信网络连接（TNC）工作小组的 IF-M 规范 \[TCG-IF-M\] 定义了一种用于 MAA 从终端机请求测定数据以及用于终端机向 MAA 提供这些测定数据的协议。

TCG 用于可互操作性的可信网络连接架构规范 \[TCG-ARCH-INTER\] 描述了一种开放的解决方案架构，它允许网络操作员根据终端机的安全状态来强制执行策略，以决定是否赋予其访问所请求的网络基础设施的权限。终端机完整性策略可能涉及横跨一系列系统组件（硬件、固件、软件和应用程序设置）的完整性参数，并且可以包括或者不包括 TPM 的证据。对于每一台终端机的此类安全评估通过使用一套涵盖了终端机的操作环境的各个方面的声明的完整性测定来实现。IF-M、IF-TNCCS 和 IF-T 是用于将测定数据从终端机传输至 MAA 的关键 TNC 协议。

NIST IR 7802 \[NIST-IR-7802\] 提供了关于在安全自动化领域中的 XML 文档的上下文环境下如何使用现存规范以代表签名、散列值、密钥信息和身份信息的建议。NIST IR 7694 \[NIST-IR-7694\] 描述了财产报告格式（ARF），这是一种用于表达财产以及财产和报告之间的关系的信息传输格式的数据模型。此标准化的数据模型有助于在组织机构内部各处以及组织机构之间报告、关联和融合财产信息。NIST SP 800-126 \[NIST-SP800-126\] 定义了安全内容自动化协议（SCAP），这是一套规范，它们标准化了安全软件产品用于通讯安全内容，特别是软件瑕疵和安全配置信息所使用的格式和命名法。SCAP 是一种多目的协议，它支持自动配置、漏洞和补丁检查、技术控制合规行为以及安全测定。

DMTF 的系统管理 BIOS（SMBIOS）参考规范 \[DMTF-SMBIOS\] 应对的是主板和系统厂商如何通过扩展 x86 架构系统上的 BIOS 以便以一种标准格式呈现关于它们的产品的管理信息。此规范的本意是提供足够的信息，以使得 BIOS 开发者可以实现必要的扩展以允许他们的产品上的硬件以及其他系统相关信息可以被这些已定义接口的用户准确地确定。此外，在那些实施者对系统上的非易失性存储器提供了写入访问权限的情况下，当一台系统部署于场地中以后，某些信息可以由管理应用程序更新以记录持续存在于系统的多次启动之间的数据。此规范的目的还在于为管理仪器的开发者提供足够的信息以开发用于将 SMBIOS 格式翻译为他们所选择的管理技术所使用的格式的通用程式，不论该管理技术是诸如 DMI 或者 CIM 的 DMTF 技术或是其他技术。为了支持这种用于 DMTF 技术的翻译，此规范中的章节描述了 DMI 组和 CIM 类，它们的本意是通过此文档中描述的接口传达获取自 SMBIOS 兼容系统的信息。

最后，TCG 证明 PTS 协议：捆绑于 TNC IF-M 规范 \[TCG-Att-PTS\] 和 TCG 基础设施工作小组的完整性报告纲要规范 \[TCG-Integrity\] 定义了如何使用 IF 以请求和发送完整性报告。它们还添加了发送临时随机数的方法和对基于 TLV 的完整性报告数据的支持。

## 3.4 BIOS 完整性测定数据的采集和传输

### 3.4.1 完整性数据的采集和传输概述

BIM 测定数据由采集代理采集并置于某种标准格式，然后由传输代理将其从终端机传输至 MAA。本节描述了采集代理和传输代理所必须满足的条件。

BIM 数据的安全传输保证了测定数据并未被修改、泄露或者在传输过程中被恶意相关方非法复制。更进一步地，传输协议的适当选择应该保证最大化的可互操作性、新鲜度和效率。此外，始于 RTR 的完整性密码学保护和测定数据来源认证保证了即使采集代理、传输代理或者其他软件被攻击，测定结果也不能被证伪。

### 3.4.2 测定数据的采集和传输的安全性指导意见

传输代理**将要**安全地将 BIOS 完整性测定数据从终端机传输至 MAA，如下所述：

1. 所有报告会话**将要**：
    1. 拥有提供机密性保护的能力
    2. 提供完整性保护和新鲜度
    3. 认证 MAA
    4. 提供终端机身份认证
2. 传输代理：
    1. **将要**允许测定数据在从 RTR 到验证组件的全程被保护完整性
    2. **应该**支持一系列服务，诸如不间断监测、网络访问控制和应用程序层级的完整性监测
    3. **应该**保证所得到的系统是廉价、非强迫性、高性能和安全的（尽管更高级的安全性可能要求更高级的开销）
3. 终端机上的采集代理和传输代理**应该**支持开放标准，这些标准允许在众多不同类型的终端机和 MAA 之间，以及在众多不同形式的通讯媒体之间实现可互操作性，直到能够存在这样的可用标准的程度

### 3.4.3 相关标准和规范

存在若干与上述要求相关的标准。PA-TNC 规范 \[IETF-RFC-5792\] 相当于 TCG 的 IF-M 1.0 \[TCG-IF-M\]；它描述了一种用于请求和传输完整性测定数据的标准结构。PB-TNC 规范 \[IETF-RFC-5793\] 相当于 TCG 的 IF-TNCCS 2.0。它描述了一种用于指导完整性测定数据交换的标准协议。TCG 可信网络连接 TNC IF-T：绑定到 TLS 规范 \[TCG-IF-T-TLS\] 描述了在网络连接性被建立以后，完整性测定数据交换可以如何通过 TLS 协议传输。TCG 可信网络连接 INC IF-T：用于隧道 EAP 方法的协议绑定规范 \[TCG-IF-T-EAP\] 描述了在网络连接性被建立之前，完整性测定数据交换可以如何通过 EAP 传输。最后，TCG 证明 PTS 协议：绑定到 TNC IF-M 规范 \[TCG-Att-PTS\] 定义了如何使用 IF 以请求并发送完整性报告。它还添加了发送临时随机数的方法以及对基于 TLV 的完整性报告数据的支持。

## 3.5 测定数据评估权威机构

### 3.5.1 概述

MAA 负责若干事情。它将一个临时随机数传输至客户端的传输代理以保证它将会从客户端收到返回的新鲜测定数据。它从客户端的传输代理收到作为响应的数据传输，这将会包含一段详细记录 BIM 数据的报告，连同对于此临时随机数和一组散列值的签名，这些将会共同保证该报告的完整性。

如果这些项目被可靠和强壮地报告，这将会允许企业管理系统准确地确定系统上的与 BIOS 配置项目相关的安全性状态。有了此信息，MAA 就可以针对组织机构所关注的具体配置项目进行报告以及采取行动。它并没有被强制为只能依据这样的事实来采取行动，即某些未具体指定的配置项目发生的更改已经导致整体测定数据偏离其上次记录的值，如果此发生更改的配置可能甚至与安全性不相关。BIOS 报告机制中的管理组件能够选择哪些属性将被报告应该成为可能。对于独立的 BIOS 配置项目，安全管理员可以被期望为组织机构所关注的项目提供理想值。

被采集和报告的 BIOS 测定数据可以被用于制定访问决策，这可以是隔离、拒绝访问网络资源，或者恢复。它还提供了关于基于网络上的设备的完整性的网络安全状态的一方面的场景警觉性。终端机厂商对于符合 TCG 规范的 BIOS 的采纳将会提供某些必要的基础设施以可靠地建立、测定并且报告终端机的 BIOS 状态。MAA 至少必须能够提供系统状态以显示给 IT 管理员或者自动化的访问/强制执行点。所采集到的信息可以被报告给策略强制执行/决策点，以获取某种形式的恢复或者响应，或者也可以被提供给这样的可视化能力，它可以为网络操作者或者分析者提供关于网络上的财产的完整性的聚合的、归一化的视图。当今存在对于采集其他设备的测定数据的可视化/场景警觉性的能力；对 BIOS 完整性测定的附加是一种自然和直白的扩展。

BIOS 可能会发生 _比特衰减_ 或者修改，在此，BIOS 内容的位可能发生虚假的变化，作为由 BIOS 存储设备硬件的物理属性所决定的随机故障的结果。取决于 BIOS 存储设备硬件，这些虚假的修改可能会在非常大规模的安装中以一定的频率发生；其结果是，意外的 BIOS 测定数据可能会被潜在地每周或者每天报告。为了避免这样的假设，即所有意外的测定数据都是由于比特衰减而造成，并且帮助操作者作出正确决策以及采取行动，MAA **可以**提供机制以允许评估此测定结果是否可能是由于比特衰减而造成。

### 3.5.2 MAA 安全性指导意见

1. MAA **将要**能够存储期望值（黄金测定数据）
2. MAA **将要**能够将新报告的值同期望值和/或之前报告（最后已知）的测定数据进行比较
3. MAA **将要**能够存储被报告的值
4. MAA **将要**支持开放标准，它们允许在众多不同种类的终端机和 MAA 之间以及在众多不同形式的通讯媒体之间保持可互操作性，直到能够存在这样的可用标准的程度
5. MAA **可以**能够提供某种相似性评估，以确定某个意外、异常的测定数据是由比特衰减所造成的结果的可能性，如果同时提供给它产生异常测定数据的 BIOS 内容以及存在黄金测定数据的 BIOS 内容

## 3.6 恢复活动

_恢复_ 是指修正计算机系统的问题的过程。在 BIOS 安全性的上下文环境中，恢复指的是诸如将老旧的 BIOS 升级到某种已知良好的版本以及重新配置 BIOS 以使其符合某组织机构的安全性要求等行为。用于修复 BIOS 安全性问题的过程通常有若干种选项。适合于每种情况的一种或者多种过程选项可能会由于若干因素而异，包括问题的类型（BIOS 更新、配置更改等）、问题的本质（有意的攻击还是无意的更改）、系统的相对重要性，以及组织机构的事故处理策略和实践等。每个组织机构都需要决定哪些因素是和它们的环境相关的，并且决定如何将它们整合到恢复活动当中。关于此问题的更深入讨论超出了此出版物的范围。

本节所探讨的恢复并未考虑事故处理和取证活动，这些方面超出了此出版物的范围。参阅 NIST SP 800-61 以获得关于事故处理的额外信息，参阅 NIST SP 800-86 以获得通用取证指南。

### 3.6.1 隔离策略

_隔离_ 是在事实上将受到攻击的系统从具有宽带网络和资源访问权限（或者潜在的访问权限，在该系统将要加入某个网络的情况下）的位置移至访问受限的联网系统池中的过程，以使得目标系统在恢复完成之前“不会造成伤害”。完全隔离限制所有网络访问，而部分隔离允许该系统访问网络上的某些部分，而非更加敏感的区域。

隔离在本意上是一种短期状态。如果威胁化解成功，通常隔离将会结束，并且此系统被允许其常规的网络访问。如果威胁化解不成功，将此系统完全移出服务可能是恰当的，以使得管理员可以手动处理威胁化解过程。

支持 BIM 的产品可以撬动现存的标准以支持隔离。TCG 可信网络连接 IF-PEP \[TCG-IF-PEP\] 可以被用于在 MAA 和强制执行点之间发送命令以支持完整或者部分隔离。RFC 3576，远端用户拨入验证服务的动态授权扩展 \[IETF-RFC-3576\] 可以被用于支持动态授权，以允许 MAA 和强制执行点在一段时间以后重新测试一台终端机，并且潜在地采取恢复行动，包括隔离该系统。

### 3.6.2 恢复过程自动化

组织机构可以拥有若干种选项以使其 BIOS 恢复行为部分或者完全自动化。不同的选项适合于不同的情况和环境。例如，某个组织机构可能想要使其 BIOS 恢复完全由外部系统（例如企业网络和系统管理）驱动，而其他组织机构可能想让恢复行为完全自包含于产品中，并且利用完全自动化的方式以使得恢复行动被尽快执行。在这些极端情况之间还有众多其他方式，其中大多数制造商在发售其产品时附带某种默认的自动化策略。可能的策略包含以下这些：

1. 完全自动化，自包含的。对于某些系统的情况，正常系统的带外可能会有被特别信任的硬件，这些硬件可以在发生攻击事件时为系统提供完整的恢复
2. 完全自动化，外部驱动的。大多数企业和大型组织机构将会对那些将要允许远程恢复以修复问题的系统应用基于服务器的恢复。在此情况下，完全重启可能将会成为必需，由于软件就其自身而言通常不可信任
3. 要求物理操作员介入的部分自动化。例如，如果恢复软件受到攻击，从某个次要来源，诸如一张 CD 重启系统以实施客户端软件的可信恢复可能是必需的。无论如何，此系统的操作员可以被信任以涉及恢复，而非要求一位可信的 IT 代表以执行此任务
4. 要求远程所有者介入的部分自动化。IT 代表可能必须被派遣以处理该系统，而非其操作员

产品可以实施不同的恢复阶段，其中包含应变策略，以防最初的恢复不成功。每个阶段都可以是完全自动化、客户可配置，或者允许手动介入的。类似地，恢复阶段的顺序也可以是自动化或者客户可配置的。在分阶段的线性恢复过程中，有单一的预定义恢复路径；在分阶段的分支恢复过程中，基于一个或者更多的变量输入，可以有一系列灵活的恢复路径选择点。此情形的一个范例是一次基于服务器的恢复在重启时被发现是不成功的，由于基于客户端的恢复软件受到攻击。此时，该客户端可以被重启至一台预启动执行环境（PXE）服务器，该服务器可以随后被用于运行已知良好的基于客户端的恢复软件，或者，一位技术人员可以被派遣至该系统以对其进行本地恢复。

PA-TNC 规范 \[IETF-RFC-5792\] 相当于 TCG 的 IF-M 1.0 \[TCG-IF-M\]，它可以支持要求用户物理介入客户端的部分自动化的恢复过程。这些标准支持将诸如网址或者人类可读的字符串等统一资源标志符（URI）从 MAA 发送至终端机以辅助用户驱动的恢复活动。

NIST SCAP 规范也可以支持完全或者部分自动化的恢复活动。NIST IR 7670 草案，用于企业安全恢复的开放规范提案 \[NIST-IR-7670\] 检视了企业恢复的应用案例，标识出了这些应用案例的高级要求，并且提议了一系列可用于应对这些要求的新兴规范。

### 3.6.3 恢复选项

有若干种可能的选项用于将 BIOS 软件或者配置恢复至某种“已知良好的状态”，包括下列选项：

1. 恢复至制造商的默认设置
2. 恢复至用于一般应用的客户默认设置
3. 恢复至用于某种特定应用的客户默认操作配置
4. 恢复至上一次已知良好的配置
5. 强制此系统进入某种非操作状态（例如隔离、移出服务等）

关于恢复的另一个考虑是对于系统的非易失性数据的处理策略。可能的选项包括：

1. 无备份
2. 部分备份
3. 完全备份
4. 应用程序门控策略——如果此攻击被定位于某个可以被选择性地禁用的特定 BIOS 服务，则可以应用此策略
5. 如果某个特定服务（诸如某个低级可管理性的接口）被攻击，可能可以安全地关闭此服务，而非重新刷入整个 BIOS

### 3.6.4 验证选项

在 BIOS 被恢复之后，组织机构可以选择验证其完整性。这通常涉及重启该系统并且将其置入其正常的测试序列中，以保证在赋予此系统网络资源访问权限之前，其完整性已经得到恢复。或者，该组织机构可以选择允许系统返回操作而无需经历标准的重新测试。在此情况下，此恢复策略至少和通常在启动时发生的可信测定被同等程度地信任。

