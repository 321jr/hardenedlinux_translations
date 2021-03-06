# 第 6 章 向外部平台进行合法报告

> 在互联网上使用加密，就像安排一辆装甲车将信用卡信息从一位住在纸箱里的人那里送到一位住在公园长椅上的人那里。——Gene Spafford，计算机科学教授

我们实现一种用于状态报告的合法信道应用的动机是将 BARM 的测定结果发送至某个被保护起来以防止 DMA 攻击的外部平台。此外部通讯伙伴可以评估所传输的测定结果以检查其对端是否被 DMA 恶意软件所攻击。此测试结果基于处理器寄存器的值（参见 5.2 节）。为了排除网卡上的恶意软件修改并且伪造流出的网络数据包，我们需要一种安全的通讯信道。这样一种信道不仅仅保证所传送的数据的机密性、完整性和新鲜性，而且还能保证信道端点的合法性。为了实现这样一种信道，我们采用了我们于早期工作 \[52，10\] 中所呈现的一种可信信道的概念。

可信信道是这样一种通讯信道，它不仅实现了安全信道的属性，并且额外地将通讯端点的状态信息绑定到通讯会话。基于 IPSec 或者 TLS 部署一种安全信道对于我们的案例来说并不足够。基于 IPSec 或者 TLS 的安全信道能够保证所传送的数据的机密性、完整性和新鲜性。然而，这些信道并未绑定到实际的通讯端点。我们为 BARM 实现基于可信信道的报告应用是为了至少能够防止下列攻击。这些攻击可以由执行于网卡上的恶意软件所引导。此类恶意软件可以通过拦截或者损坏流出的网络数据包来阻止 BARM 同外部平台进行通讯。攻击者还可以利用这样的恶意软件来通过 DMA 偷取存在于宿主的主内存中的用于该安全信道的密钥素材。随后，攻击者就可以引导中间人攻击。此恶意软件还可以中继来自第三方平台的平台状态信息。这意味着可以通过引导 _中继攻击_ 来欺骗管理员平台。

对于我们的合法报告信道，我们至少要求安全信道的属性（要求 R1）以保证所传输的数据的机密性、完整性和新鲜性 \[参见 52，p.32\]。机密性属性保证攻击者只能得到最小化的信息。完整性属性保证了损坏的网络数据包将会被立即揭示出来。新鲜性属性防止攻击者引导重放攻击，在此，一次有效的通讯会话被记录下来并且在随后被重放。为了揭示对包括平台状态信息在内的数据包的拦截攻击，我们引入了所谓的 _心跳_ 消息作为负载，它必须在通讯会话过程中被发送。计算机科学中的心跳是这样一种信号，即对应的软件仍然在线并且正在运行 \[132\]。

心跳消息包含当前的 BARM 测定结果，而如果某次攻击被阻止，则还包括日志信息。如果由于攻击而导致网卡被终止，则心跳消息不再被外部平台接收到。此行为将会被外部平台解读为基于网卡的攻击。被传送的信息还包括状态改变。状态改变同样被可信信道概念所考虑 \[52，10\]，但是缺少如同在 BARM 中所实现的那样的具有可忽略的性能开销的高效并且有效的运行时监控 \[参见 52，p.36\]：“某一平台的状态改变将会被 CM（一种假想的高效监控代理）注意到……”在我们的 DMA 恶意软件场景中，BARM 代表了缺失的“监控代理”。

与前期工作 \[52，10\] 相比，用于我们的 DMA 恶意软件场景的信任和对手模型并不要求由 TCG 所提议的可信计算机制，参见 2.7 节。我们的信道并不基于 TPM，由与我们并不依赖加载时的代码完整性检查，参见 3.2.1 节。从信道到存储于 TPM 中的加载时测定结果的连接在我们的应用中并非必需。我们要求由 BARM 确定的结果被绑定到我们的信道（要求 R2）。这在通讯会话协商以及通讯会话本身的过程中都是必要的。

请注意，我们并不依赖诸如 Intel 的 VT-d 实现等 I/OMMU 机制。这是与我们的早期工作 \[52\] 中的信任模型之间的另一个差别。此技术在我们的结果被发表 \[52\] 之前不久被引入。这意味着先前的作者并未遇到如 4.5.1 节所呈现的 I/OMMU 问题。先前的工作假设存在能够正确配置 I/OMMU 的驱动程序。对于本工作，我们更加详细地分析了 I/OMMU，并且我们决定并不依赖 VT-d 用于我们的合法报告信道。我们的先前工作还引入了关于隐私的要求（要求 R3）。这意味着此信道只考虑了最少的信息范型以便将平台状态信息的公开最小化到仅仅包含基本必要信息。

本章的主要贡献如下：

* **将网卡排除在端点之外的合法报告信道**：执行于网卡上的恶意软件能够从主内存中偷取私钥素材以引导中间人攻击。因此，我们开发了一种合法报告信道，它保证只有宿主 CPU 才是通讯的端点。我们的信道基于安全信道协议 TLS。我们采用 TLS 协议以交换 BARM 测定结果，并且将此信道绑定到其本意的端点上。我们的通讯信道的一个额外特性是平台状态改变的报告。这意味着我们的运行时监视器 BARM 通过合法报告信道持久性地将与 DMA 恶意软件有关的每一次状态改变传送至通讯伙伴。我们对于 TLS 的修改基于 TLS 扩展。这意味着我们的信道符合 TLS 规范。我们的 TLS 合规信道是首个考虑到了与 DMA 恶意软件相关的平台状态报告的信道。它还是首个基于一种已实现的、有效和高效的运行时监视器以报告状态改变的信道。先前的工作仅仅预设了这样一种运行时监视器的存在。
* **对于以太网控制器的分析**：我们的通讯信道要求使用网卡。因此，以太网控制器将会引发总线传输。这些总线传输必须被 BARM 所考虑。本章展示了以太网控制器可以被如何整合进 BARM 的检测模型，即如何将以太网控制器用作一个额外的总线代理。
* **利用一个新的参数改良 BARM 的检测模型**：以太网控制器传输数据包，其大小大于地址指针和击键码。我们展示了对于 BARM 的检测模型而言，缓存线的大小是一个重要参数。缓存线大小对于正确计算预期总线传输数量是必要的。
* **利用额外的性能监视单元事件**：我们展示了某些性能监视单元配置可以被利用以区分内存读取总线传输和内存写入总线传输。这允许我们检查由以太网控制器所造成的预期读取总线传输和预期写入总线传输的数量是否被 BARM 的检测模型所正确地确定。

下一节以对合法报告信道的描述开始。随后，我们将会解释我们如何实现这一模型。

## 6.1 独立于实现的模型

我们的信道模型考虑了客户端 _C_（目标平台）和服务器 _S_（外部平台）之间的通讯。每个端点都可以请求该端的平台状态信息（即 BARM 测定结果）。本地安全策略决定了该端的平台状态信息被评估之后准确地将会发生什么。我们的合法报告信道由宿主 CPU 软件所控制。此信道可以通过一块潜在地被攻击的网卡来进行协商。我们在下一节描述了一种用于协商和维持一条合法报告信道的高级协议。请注意，在下文中，我们省略了上标 _C_ 和 _S_，由于该协议的对称特征。

### 6.1.1 协商一条合法报告信道

我们的合法报告信道的重要理念之一是防止平台外设访问诸如私钥素材等与此信道相关的敏感信息。只有宿主 CPU 软件被允许使用信道的敏感信息。请注意，外设可以通过 DMA 偷取这些信息。然而，BARM 将会揭示并且终止此类 DMA 攻击，参见 5.3.3 节。图 6.1 描述了为 BARM 协商一条合法报告信道的握手协议。为了引导握手，双方需要一组签名密钥 _K_<sub>sign</sub>，它是非对称密钥对，即 _K_<sub>sign</sub> := \(_PK_<sub>sign</sub>, _SK_<sub>sign</sub>\)。更进一步地，双方都需要一份证书 _Cert_，它包含 _PK_<sub>sign</sub>，以及宿主 CPU 软件组件标识符（BARM\_ID）。此证书由可信实体签发，该实体可以是外部管理员平台。签名密钥和证书是在协商一条合法报告信道之前创建的。每个端点验证其对端的包含 BARM\_ID 的证书。

> 图 6.1 协商一条合法报告信道

> 在客户端 _C_（目标平台）和服务器 _S_（例如外部管理员平台）之间协商一条合法报告信道

* NIC：网卡
* _K_<sub>sign</sub>：绑定到宿主 CPU 软件组件的非对称签名密钥对 \(_PK_<sub>sign</sub>, _SK_<sub>sign</sub>\)
* _PK_：公钥
* _SK_：私钥
* _Cert_：绑定到宿主 CPU 软件组件的密钥对的经过验证的公钥部分
* BARM\_ID：宿主 CPU 软件组件标识符
* _sec\_param_：所要求的安全性参数
* _state\_data_：由 BARM 测定得到的平台状态数据
* _SigStD_：平台状态数据的签名
* _SeKey_：会话密钥

此信道的创建始于安全性参数的协商。这意味着每个实体以安全性参数的形式将其证书证书和安全性要求发送至其对端。此安全性参数决定了哪些实体需要报告其平台状态信息。每个端点检查其对端的安全性要求是否可接受。在下一步中，每个实体将其平台状态数据（当前 BARM 测定结果）发送至对端。此状态数据经过数字签名，并且对应的签名连同状态数据一同传送。这保证了所接收到的状态数据是由预期的通讯伙伴所发送的。双方利用由该端作为其证书 _Cert_ 的一部分而发送的 _PK_<sub>sign</sub> 验证签名。如果签名有效，则双方验证状态数据。此握手可能会由于攻击该端的 DMA 恶意软件而取消。这是当所传送的 BARM 测定结果大于容错值 _T_（参见 5.2.2 节）时所发生的事情。当客户端和服务器都成功验证了所交换的数据以后，同一会话的密钥被双方平台所计算并且确认。此计算出来的会话密钥将会被绑定到此通讯会话。在合法通讯会话确认就绪以后，双方开始周期性地发送心跳消息。

#### 状态改变

心跳消息要么确认当前平台状态，要么报告某种状态改变。被报告的平台状态可以揭示该端正在受到 DMA 恶意软件的攻击，此时该可疑外设可以被终止，或者并未检测到攻击。如果该端停止发送心跳消息，则本地平台假设该端受到了执行于网卡上的 DMA 恶意软件的攻击。在此情况下，BARM 已经通过终止网卡而成功地终结了正在进行中的 DMA 攻击。取决于本地安全策略，此平台可以中断该信道，继续当前会话密钥，或者重新协商此信道。如果心跳消息报告此次攻击可以被立即终止并且本地安全策略宣称此类案例是可容忍的，则继续当前会话密钥是有利的。更准确地说，如果平台在没有受到影响的外设的情况下仍然能够正常工作，则这可能是有意义的。在涉及管理员平台的情况下，我们期待管理员将会尽快更加详细地分析此次攻击，以便从受到攻击的外设中移除 DMA 恶意软件，或者如果绝对必要，将受到攻击的外设或者芯片组替换为良好的。

## 6.2 用于 BARM 的合法报告信道的实现

第 5 章所述的 BARM 并不足以用于基于合法信道的报告应用。当 BARM 发送网络数据包时，它也会造成需要被 BARM 本身的检测模型所考虑的总线活动。为了为我们的 DMA 恶意软件场景实现一种合法信道应用，我们已经 (i) 改良了 BARM 的检测模型，参见 6.2.1 节，以及 (ii) 修改了 TLS 协议以便将 BARM 测定结果（状态信息）绑定到该信道，参见 6.2.2 节。

### 6.2.1 总线主控分析：以太网控制器

为了在 BARM 的检测模型中考虑以太网适配器，我们必须确定预期的总线活动值 _A_<sub>e</sub><sup>ETH</sup>。因此，我们为我们的目标平台的以太网控制器引入了如 5.2.1 节所述的类似的总线主控分析。我们分析了与之前的实验相同的目标平台的以太网控制器（其名称为 Ethernet Controller: Intel Corporation 82566DM-2 Gigabit Network Connection (rev 02) \[65\]），参见第 4 章和第 5 章。对应的以太网控制器的 Linux 设备驱动程序为 `e1000e.ko`。为了简化我们的分析，我们将此驱动程序配置为使用传统中断，没有中断延时或者中断限速。我们同样禁用了该网络设备的校验和和段卸载。

此以太网控制器利用所谓的描述符环进行工作，即传送描述符环和接收描述符环，参见图 6.2。每个环包括 256 个描述符。每个描述符的大小为 16 字节。这意味着该设备驱动程序为每个环分配 4096 字节。如果宿主想要发送网络数据包，它需要准备传输描述符并且通知以太网控制器新的描述符已经处理就绪。以太网控制器通过 DMA 从宿主内存中读取描述符。在评估描述符之后，此控制器将网络数据包的数据从存在于描述符中的宿主内存地址（参见图 6.2）复制到其内部的内存，以便能够发送数据包。如果以太网控制器已经处理了描述符，则该控制器通过 DMA 向描述符的 `status` 字段写入 `descriptor done bit` 位，以便将该描述符“返回”宿主。当接收网络数据包时，其过程与之类似，除了以太网控制器是将网络数据包的数据写入宿主内存以外。

> 图 6.2 传送 / 接收描述符环的结构

> 当设备驱动程序通知网卡新的网络数据包已经准备好可供传送时，以太网控制器从描述符环中读取传送描述符。此控制器还会从宿主内存地址读取存储于描述符的 `length` 字段中的对应数据包的大小，而此地址存储于该描述符的 `address` 字段。当描述符处理完毕时，以太网控制器向描述符的 `status` 字段写入 `descriptor done bit` 位。当新的网络数据包通过网络到达时，以太网控制器从描述符环读取接收描述符。随后，此控制器向宿主内存地址中写入存储于描述符的 `length` 字段中的对应数据包的大小。此地址存储于描述符的 `address` 字段。如果此描述符处理完毕，以太网控制器向描述符的 `status` 字段写入 `descriptor done bit` 位。

#### 缓存线大小

为了将以太网控制器作为一个总线主控整合进 BARM 的检测模型中来，我们必须考虑网络数据包的大小通常大于击键码这一事实，参见 5.2 节。击键码是通过一次总线传输完成的。这并不适用于网络数据包，例如后者可能拥有 1514 字节的大小。为了能够确定传输一定数量的数据需要多少次总线传输，我们引入一个新的参数，即 _缓存线大小_。系统缓存被组织为缓存线。内存访问被处理为具有 _C_ ∈ N \[参见 127，p.223\] 的特定大小的缓存线。对于我们的平台，_C_ 为 64 字节 \[参见 63，p.17\]。这意味着，如果从主内存中请求一个字，则一次内存传输中实际传输了 64 字节。假设在宿主内存中相邻存储的数据很可能在一次后续的操作中被访问，如果是这样，则这些字节已经位于缓存中，并且无需额外的传输。外设的内存访问同样是以缓存线的方式被处理的。有可能必须对这样一次传输进行嗅探以保证具有一致性的缓存线 \[参见 63，p.27\]。

对于 `e1000e.ko` 驱动程序的描述符转储描述了网络数据包的宿主内存地址，参见图 6.3。次转储也揭示了并非每个地址都是按照缓存线对齐的。这意味着通过 DMA 传输该网络数据包数据所需的总线传输次数并不一定就是存储于 `length` 字段中的值除以缓存线大小。另一个重要问题于接收描述符的处理相关。根据 Intel \[65\]，以太网控制器对返回接收描述符的过程进行了优化。这意味着，当接收数据包时，以太网控制器并非为每一个描述符都单独写入 `descriptor done bit` 位。与之相反，它会“集齐”属于同一缓存线的 4 个描述符以便能够在一次总线传输中写入 4 个 `descriptor done bit` 位，参见图 6.2。我们为计算由以太网控制器所造成的预期总线传输的公式同时考虑了这两种场景。

> 图 6.3 `e1000e.ko` 驱动程序的传送 / 接收描述符转储

> 此转储揭示了导出由以太网控制器所造成的总线传输数量所必需的最重要信息。某些宿主内存地址并未对齐到缓存线，这可能导致一次额外的总线传输。

#### 以太网控制器的预期总线活动

根据我们的分析，我们将以太网控制器的预期总线活动定义如下：

_A_<sub>e</sub><sup>ETH</sup> = _A_<sub>e</sub><sup>TX<sub>reads</sub></sup> + _A_<sub>e</sub><sup>TX<sub>writes</sub></sup> + _A_<sub>e</sub><sup>RX<sub>reads</sub></sup> + _A_<sub>e</sub><sup>RX<sub>writes</sub></sup> （公式 6.1）

_A_<sub>e</sub><sup>TX<sub>reads</sub></sup> 是传送数据包时的内存读取所造成的预期总线活动。_A_<sub>e</sub><sup>TX<sub>writes</sub></sup> 代表此过程中由内存写入所造成的活动。类似地，_A_<sub>e</sub><sup>RX<sub>reads</sub></sup> 和 _A_<sub>e</sub><sup>RX<sub>writes</sub></sup> 被引入以考虑接收网络数据包时的总线活动。为了计算 _A_<sub>e</sub><sup>TX<sub>reads</sub></sup>，_A_<sub>e</sub><sup>TX<sub>writes</sub></sup>，_A_<sub>e</sub><sup>RX<sub>reads</sub></sup>，和 _A_<sub>e</sub><sup>RX<sub>writes</sub></sup>，对于一个 BARM 采样区间，我们必须考虑被读取和写入的内存缓冲区的缓存线大小。这意味着对于在宿主内存中用于存储网络数据包数据的内存缓冲区，我们必须对齐内存缓冲区的起始地址，它存储于描述符的 `address` 字段（_hma_ ∈ N），将其对齐到上一段与缓存线相对齐的地址。其结果为 _ba\_start_ ∈ N：

_ba\_start_ = _hma_ - \(_hma_ mod _C_\) （公式 6.2）

内存缓冲区的结束地址（_ba\_end_ ∈ N），它是描述符中的 `address` 字段的值（_hma_）和 `length` 字段的值（_len_ ∈ N）之和，的对齐方式如下：

_ba\_end_ = _hma_ + _len_ + _C_ - \(\(_hma_ + _len_\) mod _C_\) （公式 6.3）

描述符的传输要求同样的对齐方式。传输起始地址由之前的采样区间（_d\_old_ ∈ N）的最后一个描述符的描述符编号所决定。传输结束地址由当前采样区间（_d\_cur_ ∈ N）的最后一个描述符的描述符编号所决定。在考虑到缓存线大小时，描述符编号 _d\_start_ ∈ N 和 _d\_end_ ∈ N 的对齐结果如下所示（_D_ ∈ N 是描述符的字节数，即在我们的案例中是 16 字节）：

_d\_start_ = _old\_d_ - \(\(_old\_d_ \* _D_\) mod _C_\) / _D_ （公式 6.4）

_d\_end_ = \(_cur\_d_ \* _D_ + _C_ - \(\(_cur\_d_ \* _D_\) mod _C_\)\) / _D_ （公式 6.5）

对于一个采样区间，_A_<sub>e</sub><sup>TX<sub>reads</sub></sup>，_A_<sub>e</sub><sup>TX<sub>writes</sub></sup>，_A_<sub>e</sub><sup>RX<sub>reads</sub></sup>，和 _A_<sub>e</sub><sup>RX<sub>writes</sub></sup> 的计算如下：

_A_<sub>e</sub><sup>TX<sub>reads</sub></sup> = ∑<sub>_n_=1</sub><sup>_cur\_d_<sup>TX</sup> - _old\_d_<sup>TX</sup></sup> \(1 + \(_ba\_end_<sub>_n_</sub><sup>TX</sup> - _ba\_start_<sub>_n_</sub><sup>TX</sup>\) / _C_\) （公式 6.6）

有必要为每一个传送描述符增加一次内存读取的总线访问，由于对应的描述符存取（根据我们的实验）并非按照缓存线优化。这与写入 `descriptor done bit` 位的处理方式不同。在此种情况下，以太网控制器试图写入尽可能多的 `descriptor done bit` 位。对于一次总线传输的最大值是 4 位。

_A_<sub>e</sub><sup>TX<sub>writes</sub></sup> = \(_d\_end_<sup>TX</sup> - _d\_start_<sup>TX</sup>\) \* _D_ / _C_ （公式 6.7）

在接收网络数据包时，内存读取仅仅会由于存取接收描述符而产生。我们已经确定，以太网控制器在我们的实验过程中利用一次内存读取总线传输存取 4 个接收描述符（等于缓存线大小）。我们在下列公式中利用带有 _N_ 的指示函数：_N_ := \{_n_ ∈ \[_old\_d_<sup>RX</sup>, _cur\_d_<sup>RX</sup>\] \| \(_n_ \* _D_ mod _C_\) = 0 \}：

_A_<sub>e</sub><sup>RX<sub>reads</sub></sup> = ∑<sup>_cur\_d_<sup>RX</sup></sup><sub>_n_=_old\_d_<sup>RX</sup></sub> 1<sub>_N_</sub>\(_n_\) （公式 6.8）

由于内存写入造成的预期总线传输数量如下：

_A_<sub>e</sub><sup>RX<sub>writes</sub></sup> = \(∑<sub>_n_=1</sub><sup>_cur\_d_<sup>RX</sup> - _old\_d_<sup>RX</sup></sup> \(_ba\_end_<sub>_n_</sub><sup>RX</sup> - _ba\_start_<sub>_n_</sub><sup>RX</sup>\) / _C_\) + \(_d\_end_<sup>RX</sup> - _d\_start_<sup>RX</sup>\) \* _D_ / _C_ （公式 6.9）

我们预期网络数据包数据必须被复制到宿主内存并且对应的 `descriptor done bit` 位将会被写入到宿主内存中的描述符。

#### 利用额外的 `BUS_TRANS` 事件

我们利用更多的 `BUS_TRANS` 事件计数器验证了公式 6.1，它们基本上是事件 `BUS_TRANS_MEM` 的子集，参见图 6.4。我们确定了事件计数器 `BUS_TRANS_P` 对外设的内存读取进行计数，而事件计数器 `BUS_TRANS_INVAL` 对外设的内存写入进行计数。我们将这些计数器配合 `THIS_AGENT` 和 `ALL_AGENTS` 的名称扩展共同使用，如 5.2.1 节所述，以区分由宿主 CPU 造成的总线传输和由外设造成的总线传输。在我们的实验中并未发生 `BUS_TRANS_BURST` 事件。当 `e1000e.ko` 驱动程序函数 `e1000_clean_tx_irq` 和 `e1000_clean_rx_irq` 被调用时，由以太网控制器造成的总线传输根据公式 6.1 被计算出来。我们改良了由 5.2.2 节引入的 BARM 以考虑如本节所述的 _A_<sub>e</sub><sup>ETH</sup>。

> 图 6.4 `BUS_TRANS` 事件计数器

> `BUS_TRANS_P`，`BUS_TRANS_INVAL`，和 `BUS_TRANS_BURST` 计数之和为 `BUS_TRANS_MEM` 计数结果 \[71\]。

### 6.2.2 基于 OpenSSL 的实现

OpenSSL 是一种流行的软件工具套件，它实现了诸如 SSL/TLS 协议以及 X.509 证书的加 / 解密等密码学机制。此工具套件为开发者提供了共享库，即 `libssl` 和 `libcrypto`。`openssl` 命令行工具同样使用了这些库。需要使用由 OpenSSL 提供的密码学机制的应用程序可以直接使用这些库。注意，本节所呈现的实现基于我们 \[10\] 之前的可信信道实现。我们的修改基于 TLS 以及与 TLS 相关的 _征求意见稿_（RFC）文档，即 RFC4366 和 RFC4680。因此，这些修改符合 TLS 规范。

用于协商安全信道的会话密钥的 TLS 握手协议需要被适配以考虑 BARM 的测定结果。在握手阶段考虑这些测定结果使得该端能够确定目标平台是否已经受到 DMA 恶意软件的攻击。这有助于该端决定目标平台是否可信。如果另一个端点被认为不可信，则该端可以终止合法报告信道的握手。注意，基于我们的信任模型，我们将宿主 CPU 视为信道的一个端点。诸如网卡等其他计算环境不属于端点。我们利用非对称密码学机制和证书来认证端点。在下列段落中，我们将会描述所使用的密钥交换和证书。我们同样描述用于 TLS 的 _Hello_ 消息的扩展。针对 TLS 协议的扩展由 Dierks 和 Rescorla \[38\] 所考虑。为了传送 BARM 测定结果（平台状态数据），需要使用额外的握手信息。我们出于此目的使用 _补充数据_ 消息。

#### 密钥交换类型

我们关于合法报告信道的实现基于 TLS Diffie-Hellman 短寿命 RSA（DHE-RSA）握手 \[脚注 17\] 的一种适配版本。这意味着，为了认证端点数据，一组 RSA 签名密钥对将会被使用。而对于会话密钥的协商，Diffie-Hellman 值将会被使用。被传送至该端的 Diffie-Hellman 公开部分是由 RSA 签名密钥对的私钥部分签名的。

> 脚注 17：如我们的前期工作 \[10\] 所述，诸如 RSA 和 DH-RSA 等密钥交换方法也可被用于实现一种基于可信信道的合法报告应用。

#### 端点证书

为了认证端点，证书（参见图 6.6 和图 6.7 中的 _cert_）在 TLS 握手阶段被交换。当使用 DHE-RSA 时，证书通过包含签名密钥对 _K_<sub>sign</sub> := \(_SK_<sub>sign</sub>, _PK_<sub>sign</sub>\) 的公钥部分 _PK_<sub>sign</sub> 的 _证书_ 消息而被交换。我们必须保证私钥 _SK_<sub>sign</sub> 仅对该端点可用。我们的认证包含一个与 BARM 相关的标识符以使得基于 TLS 的合法报告信道被 _绑定_ 到该端点。包含 BARM 标识符的证书由可信的第三方签发，它能够担保目标平台上的 BARM 的正确安装，以及私钥部分 _SK_<sub>sign</sub> 仅对该端点可用。因此，证书 _cert_ 将签名密钥 _K_<sub>sign</sub> 同执行 BARM 的端点关联起来。密钥对 _K_<sub>sign</sub> 必须被用于认证在握手过程中由客户端 _C_ 和服务器 _S_ 所发送的数据。这最终将所传送的平台状态数据绑定到此合法报告信道。负责担保正确的 BARM 安装以及签名密钥的私钥部分 _SK_<sub>sign</sub> 的可信第三方可以是同时运行评估平台的管理员。此评估平台从目标平台接收平台状态信息（BARM 测定结果）。所使用的证书实际上是包含 BARM 相关标识符的正常 TLS 证书。此证书连同签名密钥对 _K_<sub>sign</sub> 由 BARM 一同部署，并且被看作长寿命的。

#### 对 Hello 消息的修改

我们利用 _ClientHello_ 和 _ServerHello_ 消息来协商用于合法报告信道的安全性参数，参见图 6.6。运行 BARM 的客户端平台 _C_ 启动适配的 TLS 客户端并且向服务器平台 _S_ 发送 _ClientHello_ 消息。服务器以 _ServerHello_ 消息作为回应。这些 _Hello_ 消息包含对应端的安全性参数 _sec\_param_（参见 6.1 节），参见图 6.6。此安全性参数决定了哪些端点必须提供平台状态数据，即 BARM 测定结果。我们使用 _Hello 消息扩展_ \[38\] 来交换安全性参数。我们的基于 OpenSSL 的实现使用 _TLS Hello 扩展_，如 RFC4366 \[14\] 所述。OpenSSL 的某个补丁（0.9.8.x）实现了 Hello 扩展，参见图 6.5 \[脚注 18\]。此补丁修改了与 `libssl` 库相关联的代码。我们将此补丁用于我们的合法报告信道应用实现。

> 脚注 18：此 TLS Hello 扩展和补充数据补丁可以在此找到：[http://openssl.6102.n7.nabble.com/PATCH-TLS-hello-extensions-and-supplemental-data-td38202.html](http://openssl.6102.n7.nabble.com/PATCH-TLS-hello-extensions-and-supplemental-data-td38202.html) \[访问于 2014 年二月 25 日\]

> 图 6.5 考虑了 Hello 扩展和补充数据扩展的 TLS 握手

> 此 _ClientHello_ 消息包含 `client data` 而 `ServerHello` 消息包含 `server data`。额外的 _SupplementalData_ 包含 `client supplemental data` 和 `server supplemental data`。补充数据也被看作 TLS 扩展（基于 \[142\]）。

> 图 6.6 用于合法报告信道的适配的 TLS-DHE-RSA 握手 (a)

> 我们针对 TLS 握手所作的修改以粗体高亮显示。适配的握手在图 6.7 中继续。

> 图 6.7 用于合法报告信道的适配的 TLS-DHE-RSA 握手 (b)

> 在握手完成之后，合法报告信道被用于 BARM 以便以某种常规的区间来传送心跳消息以通讯平台状态改变，即报告一次基于 DMA 恶意软件的攻击。

此补丁提供了一个接口以允许开发者注册新的 TLS 扩展 \[参见 142\]。由 `TLSEXT_GENERAL` 对象所表示的 TLS 扩展用于传送通用数据。使用 TLS 的应用程序指定此通用数据的数据格式。TLS 扩展包括一种类型、数据长度、通用数据（类型——长度——值格式），以及某些用于实现所需的扩展逻辑的标识 \[脚注 19\] 和回调函数。回调函数（参见图 6.5）只会在实例化了对应的 `TLSEXT_GENERAL` 对象的端上被触发。通过一条 _Hello_ 消息所传送的通用数据是一个通用数据项。在我们的实现中，通过 _Hello_ 消息（Hello 扩展）所交换的 TLS 扩展是：

* `ARCH_NEGOTIATION_EXT`：用于我们的合法报告信道（`ARCH`）的这一扩展（`EXT`）被用于协商安全性参数 _sec\_param_。

> 脚注 19：这些扩展标识包括 `client_required`（当此标识被设置时，如果服务器忽略此扩展，则客户端将会终止协商），`server_send`（当此标识被设置时，服务器将会发送此扩展），以及 `received`（内部使用，例如用于检查重复）。

客户端和服务器都需要注册 Hello 扩展（`TLSEXT_GENERAL` 对象），如果它们想要处理这些扩展。如果某端收到一条包含已注册的扩展的 _Hello_ 消息，则该端调用对应扩展的回调函数，如图 6.5 所示。

#### 用于平台状态数据的补充数据消息

客户端 _C_ 和服务器 _S_ 都能够提供平台状态数据。我们使用由 _互联网工程任务小组之网络小组_ 在 RFC4680 \[113\] 中所指定的所谓 _SupplementalData_ 消息来传送平台状态数据。OpenSSL 补丁（0.9.8.x）\[脚注 20\] 同样实现了用于 OpenSSL 的 _SupplementalData_ 消息。此补丁的实现细节由 Davide Vernizzi \[142\] 所解释。如 RFC4680 所述，补充数据也可被用于传送通用数据。由该端决定是否需要利用 Hello 扩展来传送通用数据。此 OpenSSL 补丁还允许我们定义那些我们需要用于我们的合法报告信道的补充数据扩展。补充数据扩展同样包含一种类型、通用数据、数据长度，以及回调函数。利用 _SupplementalData_ 消息传送的补充数据可以是若干通用数据的栈。在我们的实现中，通过 _SupplementalData_ 消息来交换通用数据的扩展包括：

* `ARCH_SUPP_DATA_C_EXT`：此扩展用于将平台状态数据 _PSD_<sup>C</sup>（补充数据）从客户端 _C_ 传送至服务器 _S_。
* `ARCH_SUPP_DATA_S_EXT`：此扩展用于将平台状态数据 _PSD_<sup>S</sup>（补充数据）从服务器 _S_ 传送至客户端 _C_。

> 脚注 20：参见脚注 18。

此打过补丁的 OpenSSL 软件如图 6.5 所示处理通用数据。同属 TLS 扩展的回调函数被调用以根据所要求的扩展逻辑来处理通用数据。与 Hello 扩展类似，客户端和服务器都必须注册那些它们想要通过对应的补充数据回调函数来处理的补充数据扩展。图 6.6 和 6.7 描述了我们的通用数据（Hello 扩展以及补充数据扩展）是在适配的 TLS 握手期间利用回调函数来处理的。

在我们的概念验证实现中，用于通过补充数据来交换平台状态数据 _PSD_ 的通用数据格式非常简单：

* `barm_measurement`：此数据字段包含由 BARM Linux 内核模块进行的 BARM 测定的结果
* 设备标识对列表：我们使用设备标识对列表来通讯某个外设是否在攻击目标平台。第一个标识代表对应设备是否开始攻击宿主，并且如果是这样，第二个标识表明此恶意设备是否能够被终止。此设备标识对列表的形式如下：
    * \(`uhci_attack`, `uhci_disabled`\)：此标识对代表 UHCI 控制器。
    * \(`..._attack`, `..._disabled`\)：\[_其他设备_\]。
    * \(`me_attack`, `me_disabled`\)：此标识对代表管理引擎。
* _nonceSD_：_nonceSD_ 包含两个元素：
    * _nonce_<sup>C</sup> \(`client_random`\)
    * _nonce_<sup>S</sup> \(`server_random`\)
    
平台状态数据 _PSD_ 上的签名 _Sig_<sub>PSD</sub> 也是通过 _SupplementalData_ 消息发送至该端的，参见图 6.6 和 6.7。通过如此做，平台状态数据 _PSD_ 也被绑定到对应的安全信道。包括于补充数据中的 _nonceSD_ 被同 _nonce_<sup>C</sup> 和 _nonce_<sup>S</sup>（通过 _Hello_ 消息所发送）进行比较以保证所接收到的平台状态数据 _PSD_ 的新鲜性。为了认证平台状态数据 _PSD_ 并且检查其完整性，我们使用签名密钥对的私钥部分 _SK_<sub>sign</sub> 来签名 _PSD_。为了能够验证此签名，每个端点在传送 _SupplementalData_ 消息完成之后立即利用 _证书_ 消息提供包含公钥部分 _PK_<sub>sign</sub> 的证书，参见图 6.6 和 6.7。同样作为补充数据的一部分的 BARM 测定结果被评估以导出该端的可信性。取决于所导出的可信性，本地平台根据本地安全策略采取措施。

#### 会话密钥的计算

会话密钥 _SeK_ 照常由双方计算。由于我们使用 DHE-RSA，使用 _SeK_ 的安全信道最终被连接到端点（宿主 CPU）。所交换的 DH 部分利用 _K_<sub>sign</sub> 的私钥部分（_SK_<sub>sign</sub>）签名，它将 DH 值连接到 _K_<sub>sign</sub>。签名密钥对 _K_<sub>sign</sub> 被准确地绑定到一个端点，由于此证书是由可信第三方签发的，它为 _SK_<sub>sign</sub> 仅对该端点可用这一事实进行了担保。因此，此会话密钥也被绑定到该端点。

#### 心跳消息

当握手完成后，BARM 利用此协商好的信道以某个常规的区间将心跳消息发送至外部管理员平台。这些消息以一种类似于握手过程中所使用过的 _PSD_ 格式包含当前 BARM 测定结果和设备标识对列表。只有 _nonceSD_ 缺失。常规的心跳消息被 BARM 用于报告平台状态改变，即一次基于 DMA 恶意软件的攻击。如果外部平台不再收到心跳消息，我们假设网卡试图攻击宿主平台并且 BARM 能够成功地终止该次攻击。也有可能是某个执行于网卡上的恶意软件拦截了心跳消息。如果是这样，则此次攻击同样被揭示出来。

## 6.3 评估

我们使用与 5.3 节所述的相同的平台和基本评估配置来评估改良的 BARM。请注意，只有客户端平台必须传送平台状态数据。

### 6.3.1 预期总线活动的验证

为了验证公式 6.1，我们引导了不同的测试。评估结果如图 6.8 所示。这些结果揭示了当 `ping`，`scp` 和 `wget` 命令造成网络流量时，BARM 测量结果会产生较大的波动。表 6.1 提供了关于造成这些较大波动的原因的信息。此表呈现了在使用 `wget` 命令下载一个 1 GB 的文件时进行的 BARM 测量的结果。所应用的采样区间为 32 ms。此表描述了一个较大的正偏差（参见 BARM 样本 125924：13 次总线传输）之后跟着一个较大的负偏差（参见 BARM 样本 125925：-12 次总线传输）。我们假设正偏差发生于网络数据包已经由以太网控制器复制到宿主内存，但是 BARM 未能在当前采样区间中评估对应的接收描述符。这些描述符在下一个采样区间中可用。因此，BARM 在下一个区间中评估这些描述符，而这又导致了负偏差。BARM 从测量的传输次数减去预期总线传输，而这部分传输实际上已经在上一个采样区间中测量过了。

如表 6.1 所示，这些正值和负值相互抵消。因此，可以通过简单地累加正负 BARM 测量值而使得波动最小化。如表 6.1 所示，一对正负测量值也可以以另一种方式发生（例如参见 BARM 样本 125926 和 125927）。这意味着负值先于正值被检测到。我们假设这发生于 BARM 已经分析了传送描述符，而对应的数据包尚未被以太网控制器所复制。因此，BARM 在其实际被测量之前，已经从测量到的总线传输次数中减去了这部分预期总线传输。这些传输于下一个采样区间中被测量，从而导致一个较大的正偏差。

> 表 6.1 显示出波动的 BARM 测量值

|BARM 采样编号|BARM 测量值|BARM 采样编号|BARM 测量值|BARM 采样编号|BARM 测量值|BARM 采样编号|BARM 测量值|
|---|---|---|---|---|---|---|---|
|125912|5|125928|5|125944|2|125960|-21|
|125913|2|125929|0|125945|25|125961|25|
|125914|3|125930|-2|125946|-48|125962|2|
|125915|2|125931|5|125947|28|125963|-2|
|125916|3|125932|2|125948|0|125964|3|
|125917|0|125933|5|125949|3|125965|2|
|125918|0|125934|0|125950|5|125966|5|
|125919|1|125935|-2|125951|1|125967|2|
|125920|2|125936|9|125952|13|125968|-1|
|125921|-17|125937|2|125953|-21|125969|2|
|125922|22|125938|3|125954|5|125970|2|
|125923|1|125939|-2|125955|2|125971|6|
|125924|13|125940|8|125956|2|125972|0|
|125925|-12|125941|-3|125957|-1|125973|1|
|125926|-15|125942|0|125958|4|125974|2|
|125927|22|125943|5|125959|3|125975|3|

> 采样编号和对应的测量值取自测量日志，该日志取自从 [http://download.thinkbroadband.com/1GB.zip](http://download.thinkbroadband.com/1GB.zip) \[访问于 2014 年二月 25 日\] 下载一个 1 GB 的文件时。BARM 采样区间为 32 ms。

> 图 6.8 具有网络流量时的预期总线活动评估

> 我们为 6 种不同的测试案例评估了预期总线活动。此差值以从图 5.6 中所知的盒图的形式可视化。在第一个案例（BARM）中，我们只运行了改良的 BARM，而在第二个案例中，我们连同改良的 BARM 一起运行了基于 OpenSSL 的合法报告信道。我们在这两种情况下各自运行了 100 次 BARM 测量。BARM 和合法报告信道同样在其余的测试案例（`ping`，`scp`，`wget` 和 `wget'`）中保持活动。我们以 1000 字节的负载执行了 `ping` 命令 100 次（`ping`）。在 `scp` 案例中，我们将一个 100 MB 的文件从外部平台复制到我们的目标平台 100 次。在 `wget` 案例中，我们使用 `wget` 命令从 [http://download.thinkbroadband.com/1GB.zip](http://download.thinkbroadband.com/1GB.zip) \[访问于 2014 年二月 25 日\] 下载了一个 1 GB 的文件。我们对除了最后一项测试（`wget'`）以外的全部测试使用了 32 ms 的 BARM 采样间隔。`wget'` 案例的盒图代表了在利用 `wget` 下载一个 1 GB 的文件时使用 1024 ms 采样区间的结果。

我们在利用 `wget` 命令下载一个 1 GB 的文件时检测到了发生于两个采样区间之间的上述行为。如图 6.8 所示，当使用 `wget` 并且使用 32 ms 采样区间（参见 `wget`）时的波动大于使用 1024 ms 采样区间（参见 `wget'`）时的。

### 6.3.2 网络性能开销评估

我们引导了一组网络基准测试以揭示由改良的 BARM 所造成的网络性能开销。此改良版本的 BARM 持久地发送心跳消息。其结果如图 6.9 所示。图 6.9 所示的结果揭示了每隔 32 ms 发送一次心跳消息时的相对性能开销约为 4.5%。此区间长度对应于 BARM 的采样区间。并不必须将用于 BARM 测量采样的相同区间长度用于报告，由于我们用于传送平台状态数据的心跳消息的格式。该设备标识对列表代表了关于恶意外设的历史记录。因此，对于同为 32 ms 的采样和报告区间的网络性能开销可以避免。唯一的要求是采样区间小于或者等于报告区间。

> 图 6.9 对于不同报告区间和固定采样区间的相对性能开销

> 此图对比了 3 组系列测量的结果。第一组测量系列（不活动）代表基线。不活动意味着 BARM 并未运行，以及没有心跳消息被发送。图中的条形表示 100 次测量的平均值，同时参见 5.3.2 节。我们（利用时间戳计数器）测量了利用 `scp` 命令从外部平台复制一个 100 MB 文件所需的时钟周期。测量针对 32 ms 和 1024 ms 的报告区间进行。在两种情况下，我们都使用 32 ms 的 BARM 采样区间。每 32 ms 发送一次心跳消息时的相对性能开销约为 4.5%，而每 1024 ms 发送一次该消息时的开销仅约为 0.5%。

### 6.3.3 利用 DAGGER 进行测试

我们利用改良的总线代理运行时监视器 BARM 重复了 DMA 恶意软件 DAGGER 测试（参见 5.3.3 节）。其结果总结于图 6.10，6.11 和 6.12。我们在运行时的任意时间点攻击了目标平台。图 6.10 确认了改良的 BARM 能够揭示 DMA 攻击并且终止恶意外设。图 6.11 和图 6.12 中截取的日志来自于相同的实验，该实验也是图 6.10 的基础。

> 图 6.10 在运行时的任意时间点以及存在合法报告信道的情况下评估改良的 BARM

> 所引导的实验类似于 5.3.3 节所呈现的实验。BARM 的采样区间为 32 ms，并且容错值为 50 次总线传输。这一次，BARM 将以太网控制器视为一个额外的总线主控，这允许我们启动我们的合法报告信道。心跳消息每隔 32 ms 发送一次。此图对比了 3 条曲线，即容错值 _T_、在没有任何攻击时的 BARM 测量结果，以及发生了一次 DAGGER 攻击时的 BARM 测量结果。

> 图 6.11 BARM 合法报告信道——客户端侧

> 此图呈现了 BARM 日志输出的一部分。BARM 被部署于目标平台，即客户端上。此日志输出展示了 BARM 揭示了一次 DMA 攻击，并且 BARM 能够终止该恶意外设。

> 图 6.12 BARM 合法报告信道——服务器侧

> 此图描述了适配的 OpenSSL 服务器的日志输出。此日志包含 TLS 握手消息、回调函数调用消息，以及所接收到的 BARM 测定结果。测定结果值同图 6.11 所呈现的。部署于客户端侧的 BARM 实例能够终止此次攻击。在此范例中，本地安全策略容忍了此次被终止的攻击。或者，服务器也可以终止此信道，如果服务器收到了 441 次总线传输的 BARM 测试结果。服务器同样配置了 _T_ = 50 次总线传输的容错值。

## 6.4 安全性考虑

在本节中，我们非形式化地评估了我们于本章开头所引入的安全性要求。形式化的证明超出了本工作的范围。众多与 TLS 协议的安全性证明相关的研究工作已经于过去被发表。一部综述由 Kohlweiss 等人 \[78\] 呈现。此研究同样考虑了多种 TLS 变体。我们假设我们的基于 TLS 的信道也能被形式化地证明。然而，本章的关注焦点在于考虑了网卡的改良的 BARM。因此，我们重新审视了我们的改良的 BARM 能够在多大程度上满足对于安全信道（R1）、将 BARM 的测定结果绑定到该安全信道（R2），以及隐私（R3）的要求。

### R1——安全信道属性

由于所应用的 TLS 协议，此通讯信道已经保证了安全信道属性中的机密性、完整性、合法性，以及新鲜性。由于改良的 BARM，这些属性同样在端点，即宿主 CPU 上得到了保证。鉴于攻击者必须搜索有价值数据，BARM 保证了存在于主内存中的数据的完整性和机密性。攻击者只能随机写入或者读取主内存而不能搜索有价值数据。攻击者同样需要搜索一次性随机数、密钥素材或者会话密钥 _SeK_ 以及签名密钥对的私钥部分 _SK_<sub>sign</sub> 以攻击此通讯会话。因此，改良的 BARM 同样能够在端点上兼顾合法性和新鲜性的属性，由于在攻击者搜索主内存时检测到了额外的总线流量。

攻击者只能引导一次中间人攻击，如果攻击者能够通过 DMA 偷取私钥素材或者会话密钥。扫描内存以查找这些数据将会被 BARM 检测到。BARM 还能够识别恶意设备。因此，对主内存的访问可以被防止。注意，宿主 CPU 可以通过将一部分敏感数据存储于处理器寄存器中而迫使攻击者造成更多的总线传输。此项技术已经在相关工作中被提议，参见 3.2.6 节。这并不能够保护敏感数据，由于 DMA 攻击可以被用于将处理器寄存器内容转储到主内存。然而，这样的攻击将会造成更多的总线活动，而这也会被 BARM 检测到。

攻击者可以试图修改 BARM 测量结果。为了做到这一点，攻击者可以试图从主内存中查找某些变量，在此，BARM 存储那些我们将其用于揭示 DMA 攻击的性能监视单元的值。然而，基于 DMA 的搜索将会被 BARM 揭示出来。或者，攻击者可以试图修改那些 BARM 所使用的对应于性能监视单元的宿主 CPU 寄存器。攻击者不能直接访问宿主 CPU 寄存器。然而，攻击者需要查找一块用于存储宿主 CPU 指令以修改用于性能监视的处理器寄存器的内存区域。这要求宿主 CPU 迟早将会考虑该内存区域，它包含恶意指令。然而，攻击者还是必须通过 DMA 搜索这样一块区域，而这种基于 DMA 的搜索将会被 BARM 揭示出来。

### R2——将 BARM 测定结果绑定到安全信道

端点的合法性通过提供证书 _cert_ 而得到保证。该证书包含 BARM 标识符以及签名密钥对的公钥部分 _PK_<sub>sign</sub>。此证书由可信实体签名。两个因素保证了 BARM 测定结果被绑定到该信道。其一，在握手阶段被传输的 BARM 测定结果经过该端点的签名密钥中的私钥部分 _SK_<sub>sign</sub> 签名。其二，被用于会话密钥计算的交换的 DH 值也经过 _SK_<sub>sign</sub> 签名。因此，不仅仅是首先被传送的 BARM 测定结果以及 DH 值被绑定到信道端点，而且也包括用于最终建立用于合法状态报告的安全通讯信道的会话密钥 _SeK_。这意味着每一条心跳消息也都被绑定到信道端点。这些消息只会通过由 _SeK_ 所保护的信道以加密的形式而被传送。

端点的合法性同样防止了一种中继攻击，在此，攻击者能够向第三方平台发送请求以签名一条包含少于 50 次总线传输的 BARM 测量值的平台状态数据 _PSD_。第三方平台不能访问目标平台的 _SK_<sub>sign</sub>。这意味着我们可以排除攻击者能够引导中继攻击的可能性。或者，攻击者可以试图伪造 _PSD_ 签名。为了做到这一点，攻击者需要存在于主内存中的 _SK_<sub>sign</sub>。然而，当攻击者通过 DMA 搜索 _SK_<sub>sign</sub> 时，BARM 还是会揭示此次攻击并且内存访问将会被终止。因此，我们可以得出结论，即攻击者不能伪造数字签名。

### R3——隐私

唯一以未加密形式传送的敏感数据是首次 BARM 测量值，它在握手阶段被发送至对端。尽管受到攻击的网卡可以被用于拦截这个值，此首次测量结果的值不太可能对于攻击者有用。它与其他测量值相互独立，后者被用于识别 BARM 于何时检测到 - _T_ 次总线传输。因此，我们可以得出结论，即我们的合法报告信道满足最少信息范型。

## 6.5 本章小结

在本章中，我们为 BARM 开发、实现并且评估了一种合法报告信道应用。此信道基于安全信道协议 TLS。我们修改了 TLS 协议以便在握手阶段以及其余的通讯会话中考虑 BARM 测定结果。我们的修改基于 TLS 扩展，这意味着我们的信道符合 TLS 规范。更进一步地，我们的报告信道的实现满足我们为 DMA 恶意软件场景所定义的安全性要求（宿主 CPU 端点的合法性以及信道绑定）。如果未能满足这些要求，则执行于网卡上的恶意软件对于同外部平台进行合法通讯来说仍然是一种威胁。

我们的信道是用于我们的总线代理运行时监视器的一个应用，如果通讯伙伴要求关于平台状态更改的报告。合法报告信道将状态改变传送至该端。我们利用我们自己的 DMA 恶意软件 DAGGER 配合所实现的报告信道确认了 BARM 的有效性和高效性。与合法平台状态报告相关的前期工作假设存在这样一种高效的运行时监视器。然而，呈现于前期工作中的对应的概念验证实现并未包含这样一种监视器。更进一步地，前期工作也并未考虑 DMA 恶意软件场景。

我们同样可以得出结论，即 BARM 可以处理更加复杂的总线主控。我们展示了 BARM 不仅仅能够处理宿主 CPU、UHCI 控制器等，也能处理以太网控制器。为了将以太网控制器整合进 BARM 的检测模型，我们不得不针对内存读取和内存写入访问对该控制器进行分析。我们通过利用额外的性能监视单元配置能够区分读取和写入访问。然而，为了最终确定由以太网控制器造成的总线传输数量，我们不得不引入了一个新的参数。这个新参数是缓存线大小。根据我们的评估，BARM 测量结果的波动只是略微高于未考虑以太网控制器的版本。此外，波动仍然处于 _T_ = ±50 次总线传输的范围内。我们的经验性测量揭示了合法报告信道应用的性能开销是可忽略的，如果心跳消息大约每秒发送一次。报告区间可以大于 BARM 的采样区间。DMA 恶意软件攻击信息的缺失可以通过在心跳消息中包含一段攻击历史记录而避免。

