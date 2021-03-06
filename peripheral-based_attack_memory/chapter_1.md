# 第 1 章 简介

> 我认为，大多数人甚至都不知道 rootkit 是什么，那么他们为什么要去关心它呢？——Thomas Hesse，索尼全球数字业务前主席

大多数人可能会将 _rootkit_ 这一术语同针对计算机平台的攻击相关联。实际上，对手部署 rootkit 是为了攻击计算机用户。基于 rootkit 的攻击被用于引导行业和政治间谍活动以及网络犯罪 \[参见 16，p.22-25\]。对手通过引导行业间谍活动来偷取知识产权以减少技术开发周期的成本。政治间谍活动不同于行业间谍活动。在政治间谍活动中，对手所感兴趣的是国家机密而非新技术。网络犯罪分子利用 rootkit 来偷取互联网银行凭证、口令以及其他敏感信息。rootkit 也可被用于引导对最终用户的持久监控。rootkit 还可被用于强制执法，以执行对嫌疑人的监控 \[参见 16，p.21\]。但是，准确地说，rootkit 到底是什么？它是 _后门_ 吗？它是 _特洛伊木马_ 吗？换言之，rootkit 所包含的恶意负载是什么类型，以及目标计算机是如何被此 rootkit 所渗透的？

术语 rootkit 的若干种定义可以在这样一些文献中找到，例如 Bill Blunden 所著的 _The Rootkit Arsenal: Escape And Evasion In The Dark Corners Of The System_ \[16\]。他的著作同时评估了 Mark Russinovich（此人以 _Windows Internals_ \[106\] 系列著作而闻名）和 Greg Hoglund（_Rootkits: Subverting the Windows Kernel_ \[60\] 一书的作者）对于 rootkit 的定义。最后，Bill Blunden 得出了他自己的定义 \[参见 16，p.12\]：

> rootkit 在一台设备上建立了一个远程接口，此接口使得系统可以被操纵……数据可以被收集（例如监控），以某种难以被观察到的方式（例如隐藏）。

所有这些定义提示了 rootkit 在总体上所展现出来的一个重要属性，即能够隐秘地运作的能力。攻击者通过部署 rootkit 来掩护用于攻击目标计算机的代码。这一点回答了关于 rootkit 的恶意负载的问题。此负载可以被用于执行在用户看来是恶意行为的任何东西。此恶意行为也可以是后门。后门被用于绕过诸如认证请求等安全机制以获得对某台计算机系统的访问权限。后门还可以为攻击者提供对于某台计算机的远程访问权限。在攻击者看来，将此后门隐藏起来是有意义的。此后门应该在计算机用户毫不知情的情况下被使用。因此，后门可以得益于 rootkit 机制。rootkit 负载的另一个例子是监控程序，此程序通过激活目标计算机的话筒和摄像头以暗中监视该计算机的用户。能够捕获由某位计算机用户所输入的所有击键的击键码记录器也是恶意负载的常见范例。

然而，对于攻击者的挑战在于渗透目标计算机平台。此攻击者必须实现某种类型的 rootkit 安装程序。rootkit 安装程序通常被称为 _释放器_ \[参见 16，p.9\] \[33\]。这样的释放器可以基于最常见的渗透机制之一，即特洛伊木马，或者简称木马。木马的目的在于在目标计算机将要安装某个预想的程序、特性或者功能时对其进行误导。与之相反，其结果是用户安装了诸如击键码记录器或者后门等恶意负载。诸如此类的负载通常被部署于高权限环境，并且利用 rootkit 技术加以掩护。另一种常见的渗透方式是利用某个安全漏洞。此 rootkit 安装程序可以实现某种 _漏洞利用_。漏洞利用是一段利用安全漏洞的攻击代码。所谓的 _零日漏洞_ 相对于非零日漏洞更具威胁性。零日漏洞所利用的是某个此前未知的安全漏洞，这可能对于攻击者更加有利。它使得攻击者能够引导一次对于目标计算机的隐秘渗透。

rootkit 的另一个关键属性是其代码运行于可能的最高权限之上。其目标是获得至少是高于任何潜在的检测机制的权限。这使得 rootkit 能够控制并且修改检测机制。在某种程度下，检测机制将会无法检测 rootkit 或者由此 rootkit 掩护的恶意负载。这是攻击者寻求新的、更加强大的攻击向量的原因。攻击者获得的权限越多，其所获得的对于目标计算机的控制也就越多。

攻击者的目标是获得对目标计算机的绝对控制。_rootkit 进化_ 记录了攻击者和反恶意软件社区之间的军备竞赛。相对于早期的 rootkit，现在的 rootkit 已经前进到了更高权限的执行环境。近年来 \[35，36，47，134，135\] rootkit 的进化达到了一个新的水平。攻击者开始利用平台外设的隔离执行环境。具有专用处理器、专用内存以及直接访问宿主的运行时内存的硬件特性的外设能够掩护用于攻击目标计算机的恶意负载。这样的攻击被认为是隐秘的。市场上可获得的现代反病毒软件之类尚未考虑基于外设的执行环境。这样的软件执行于宿主处理器上，并且通常只将硬盘和主内存考虑为能够存储恶意代码。

## 1.1 问题表述

恶意软件是对数据的机密性、完整性以及可用性的威胁。对于基于外设的恶意软件的情况，攻击者可以利用外设的隐形潜力。隐藏在平台外设中的恶意软件不会被反病毒软件所考虑。取决于外设，安全软件甚至不能访问该设备的内部工作。例如某些管理控制器拥有对全部宿主内存的访问，并且提供远程管理特性。为了防止滥用，制造商应用保护机制以阻止对此执行环境的内部工作的访问。

这种机制，如果被基于外设的恶意软件所利用以攻击宿主，则称之为直接内存访问或者 DMA。在本工作中，我们将会引入术语 _DMA 恶意软件_ 以用于诸如此类的攻击。DMA 恶意软件拥有同 rootkit 相似的特性。当前的反制措施无法应对 DMA 恶意软件的挑战。例如，对本意将要在外设上运行的代码进行加载时完整性检测等机制并不能阻止运行时攻击。这同样适用于数字签名的固件镜像的情形。另一种方式是基于延时的证明。此类证明要求一段散列值在一定的时间框架内被计算出来。然而，这同样要求修改外设固件，并且不能阻止瞬时攻击。诸如特殊监控以及内存总线嗅探等其他方式基于特殊硬件或者硬件特性。阻止敏感数据出现在主内存中同样不能奏效。这些数据可以通过 DMA 攻击被转储到主内存中 \[脚注 1\]。

> 脚注 1：具体细节可以在 3.2 节“相关工作——反制措施”部分找到。

一种被提议的针对 DMA 攻击的反制措施是使用一种所谓的 _输入/输出内存管理单元_（I/OMMU）。这样的管理单元可以限制外设对宿主的部分主内存的访问。然而，此技术具有重大缺陷。已有证据表明 I/OMMU 可以被攻击并且绕过 \[111，146，147，148\]。因此，I/OMMU 不一定可信。某些操作系统，诸如 Windows，并不提供驱动程序以支持 I/OMMU。此外，并非每种芯片组都提供 I/OMMU。更进一步地，I/OMMU 不能处理内存访问策略冲突。例如，Bulygin \[25\] 展示了如何利用外设以揭示存在于宿主的运行时内存中的恶意软件。我们将相同的执行环境用于第 4 章的攻击研究。例如，如果 I/OMMU 被配置为允许外设扫描宿主的全部运行时内存以揭示 rootkit，那么我们的攻击代码同样可以访问全部运行时内存以偷取敏感数据。因此，本工作并不依赖 I/OMMU 作为一种反制措施。更进一步地，I/OMMU 可能会引入显著的性能开销 \[13，150\]，这使得 I/OMMU 对于某些场景并不理想。由于这些考虑，我们相信缺少这样一种能够检测恶意内存访问并且具有可忽略的性能开销的运行时监视器。这样一种运行时监视器的缺失正是推动本工作的动力之一。

## 1.2 研究问题和方法论

我们的研究兴趣基于现代 x86 平台的隐形能力。这些能力被对手以利用以隐藏恶意代码，如同 rootkit 进化所记载，同时参见 2.1 节。这提出了这样一个问题，即不可被检测到的软件是否能够存在。为了检验这个问题，我们考虑了 rootkit 中的下一个逻辑步骤，即利用平台外设以攻击宿主的运行时内存。

我们开发了一种恶意软件的 _概念验证_（PoC），它执行于某个隔离的外设上。此外设的硬件提供了对宿主的运行时内存的访问。我们以击键码记录器的形式实现了一次攻击。这意味着我们的恶意软件搜索宿主操作系统的键盘缓冲并且监视该缓冲以捕获击键码。对此击键码记录器的评估指导了我们后续的一个研究问题，即宿主系统能否保护自己以对抗基于外设的宿主主内存攻击？为了回答这个问题，我们实现了一个运行时监视器，它执行于宿主 CPU 上。有了这个监视器，我们想要展示这一点，即来自平台外设的对宿主主内存的额外（恶意）访问事实上是可以被检测到的。我们要求基于宿主 CPU 的检测程序能够检测恶意访问，即使它不能访问该恶意外设的隔离执行环境。

我们利用我们的恶意软件样本以得出此类恶意软件的一般属性。随后我们利用了这些属性以检测由恶意软件所引导的内存访问。我们定义了这样一种属性，它是每一种攻击宿主内存的基于外设的恶意软件都体现出来的。基于此，我们将我们的恶意软件概念验证视为对于此类恶意软件典型的。我们实现了基于宿主 CPU 的检测程序以揭示由平台外设通过直接内存访问所引导的非法内存访问。其目标是实现这样一种运行时监视器，它不仅对于宿主 CPU 只造成最小化的性能开销，同时能够阻止瞬时攻击。

我们还在我们的研究的最后一部分考虑了网卡。网卡同样可以托管恶意软件。特别是在企业环境中，某个计算机平台被要求向某个中央管理员平台报告其状态。这样的状态报告可以被执行于网卡上的恶意软件修改。因此，我们开发了一种合法报告信道。此信道有助于揭示针对此类状态报告的攻击。

### 实验研究环境

我们的实验环境基于 Intel x86 硬件。我们用于执行我们的基于外设的恶意软件的隔离环境是 _Intel 管理引擎_（Intel ME \[79\]）。Intel ME 是一种特殊的微控制器，它运行某种强大的平台管理固件。管理员可以利用此管理固件远程重装操作系统，即使操作系统已经不可引导，并且该平台不可通过操作系统的网络栈抵达。ME 同样能够在平台处于待机或者关机的情况下运作。由于这些特性，其制造商 Intel 建立了这样的保护机制，它们不能在不付出巨大努力的情况下被绕过。ME 是同宿主系统相隔离的。Intel ME 环境是同宿主完全隔离的，而其他外设可以通过调试寄存器以及其他机制来访问。

从检测程序的角度来看，ME 是用于托管基于外设的恶意软件的执行环境的最坏的案例。宿主 CPU 无法访问 ME 环境。我们将这一最坏案例环境用于我们的研究。我们通过应用某个只能工作在特定芯片组 \[脚注 2\] 上的漏洞来渗透 ME 环境。请注意，此工作并不致力于查找未被发现的安全漏洞。我们重复利用了某个已知的安全漏洞来设置我们自己的实验环境，是由于缺少一块适合的 Intel 开发板。

> 脚注 2：此漏洞利用仅适用于刚好具有特定版本 BIOS 的 Intel Q35 芯片组。Intel 通过提供 BIOS 更新堵住了对应的安全漏洞。

## 1.3 论文贡献的影响

例如为了引导行业间谍活动或者偷取在线银行凭证等，攻击者要求能够隐秘运作的恶意软件。基于外设的恶意软件保证了攻击仍然不可被检测到。能够满足隐秘恶意软件运作的需求的外设存在于几乎每一部现代计算机平台中。诸如显卡、网卡和管理控制器等是台式机、服务器系统以及其他计算机终端的组成部分。移动电话和平板计算机也具有那些带有独立处理器、内存以及对宿主运行时内存的直接访问的外设。这意味着所有现代平台都易受基于外设的恶意软件的攻击。这样的恶意软件执行于隔离环境中，并且处于由操作系统内核所设置的反病毒软件和安全机制的视野之外。由于缺少针对基于外设的恶意软件的检测程序以及在反病毒软件中缺少类似功能，本论文的贡献可能对上述计算机设备及其用户具有重大影响。我们将本论文的主要贡献总结如下：

### DMA 恶意软件研究

我们定义了 DMA 恶意软件以便能够区分不同的 DMA 代码。这样的恶意软件执行于某个外设上，并且能够通过直接内存访问攻击宿主。我们开发了一种 DMA 恶意软件实现的概念验证，它能够利用某个隔离的外设来引导隐秘攻击。我们的概念验证称为 DAGGER，它来自于 _DmA-based keystroke code loGGER_（基于 DMA 的击键码记录器）。DAGGER 可以攻击不同的宿主操作系统。DAGGER 强调了 DMA 恶意软件在实践中是多么地高效。我们识别了 DMA 恶意软件的核心属性以研究诸如此类的恶意软件的属性。这些属性是 DMA 恶意软件检测程序的基础。在一次早期实验中，我们提供了关于 DMA 的副作用存在的证据。我们展示了这样一种效应可以如何利用通常的宿主 CPU 特性进行测定。这是 DMA 恶意软件检测程序开发的第一步（参见第 4 章）。

### 检测 DMA 恶意软件

我们开发了一种监视器，它能够通过比较实际的内存总线活动和预期的内存总线活动来检测 DMA 恶意软件。我们的方法能够确定并且比较实际的总线活动而不受任何固件或者硬件修改的影响。此检测器基于这样一种特性，它实现了永久的运行时监控并且运行在宿主 CPU 上。我们实现并且评估了这样一种概念验证，我们称之为 _总线代理运行时监视器_（BARM）。我们的监视器实现了这样一种监视策略，它会考虑瞬时攻击。它只会造成可忽略的性能开销。BARM 可以检测并且立即终止 DMA 恶意软件（参见第 5 章）。

### 排除了 DMA 恶意软件干扰的合法平台状态报告

我们展示了我们的检测方法同样适用于这样的场景，在此，一部计算机平台必须向某个中央管理员平台报告其状态。我们建立了这样一种合法报告信道，它能够揭示由执行于网卡上的恶意软件所引导的攻击。这意味着我们改良了 BARM 以揭示 _中间人_（MitM）攻击，并且阻止由网卡引导的中继攻击。我们实现了一种信道以便将平台状态信息安全地传输至一台外部计算机。此平台状态信息允许远程实体评估 BARM 的测定结果。这意味着远程实体可以确定其对端是否受到了 DMA 恶意软件的攻击。我们的信道将宿主 CPU 视为信道的端点，而非完整的目标平台。这排除了网卡作为端点的一部分。我们改良了 BARM 以说明由网卡所造成的内存总线活动。此改良的 BARM 利用 _OpenSSL_ 来实现合法报告信道。我们还修改了 TLS 握手协议以便在通讯会话的最初阶段就说明平台状态信息。我们的修改仍然符合 TLS 规范（参见第 6 章）。

具体的工作细节可以在各对应章节找到。

## 1.4 论文结构

根据我们的方法论，我们按照如下方式组织论文结构。下一章，我们将会介绍必需的技术背景、预备知识以及假设。用于我们的评估的目标平台是一种基于 Intel x86 的现代系统，参见 2.2，2.3，2.4，2.5 和 2.6 节。这些章节介绍了关于该目标平台的大部分重要术语，特别是宿主 CPU、_直接内存访问_（DMA）、总线主控以及 _输入/输出内存管理单元_。我们还在 2.7 节介绍了我们的假设以及由此得出的对手模型。第 3 章覆盖了相关工作。由于我们同时考虑攻击以及攻击检测和防护两端，我们必须认真研究两端的相关工作。关于 DMA 攻击的相关工作描述于 3.1 节。3.2 节呈现了那些考虑了反制措施的前期工作。更进一步地，我们想要使得我们的目标平台能够向外部平台报告其关于基于 DMA 的恶意软件的状态。为了达成这一目的，我们要求一种能够揭示由网卡发动的中间人攻击的通讯信道。这是必需的，由于我们同时将网卡视为能够隐藏 DMA 攻击代码的专用硬件。

我们引导了针对 DMA 恶意软件的研究并且将其结果呈现于第 4 章。关于 DMA 恶意软件的定义给出于 4.1 节。在 4.2 节，我们呈现了 DMA 恶意软件的核心功能。我们自己的 DMA 恶意软件的设计和实现呈现于 4.3 节。4.4 节描述了对于 DAGGER 的评估。4.5 节考虑了反制措施并且特别讨论了 I/OMMU 的问题。在同一节中，我们描述了我们如何能够利用这些属性以显示首个 DMA 副作用。由于宿主 CPU 无法直接意识到由受到攻击的外设所引导的非法内存访问，我们试图触发一种发生于外设访问主内存时发生的副作用。

第 4 章所呈现的 DMA 副作用是推动我们实现我们于第 5 章所介绍的运行时监视器的动力。在第 5 章“DMA 恶意软件检测初步”中，我们展示了 DMA 副作用可以被如何利用以开发检测工具。我们定义了一种通用检测模型，它有助于我们构建检测工具，参见 5.1 节。随后，我们在 5.2 节中呈现了一种基于流行的 Intel x86 平台的概念验证实现。我们在 5.3 节评估了我们的实现。我们同时利用我们于第 4 章开发的 DMA 恶意软件测试了 BARM。最后，BARM 利用了这一事实，即我们的 DMA 恶意软件必须搜索有价值的数据，由此造成了一定量的总线传输。

在第 6 章，我们改良了我们的检测工具以实现一种合法状态报告应用程序。此应用程序将 BARM 测定结果发送至某个外部平台。其目标是实现这样一种安全通讯信道，此信道排除了由运行于网卡之上的恶意软件引导中间人攻击的可能性。在 6.1 节，我们呈现了一种模型以协商一种合法报告信道。我们需要一种诸如 TLS 的安全信道，它绑定于实际的通讯端点，即宿主 CPU。我们关于合法报告应用程序的概念验证基于 OpenSSL，参见 6.2 节。此实现章节同时描述了为了考虑网卡而必需的 BARM 改良。关于我们的实现的评估呈现于 6.3 节。我们还利用我们自己的 DMA 恶意软件 DAGGER 测试了关于网络的 BARM 改良。合法报告信道的安全性考虑于 6.4 节讨论。我们关于此论文的结论以及今后的工作呈现于最后一章，第 7 章。

