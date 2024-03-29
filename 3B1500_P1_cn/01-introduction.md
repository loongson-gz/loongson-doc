龙芯 3B1500 处理器概述
======================

龙芯处理器包括三个并行发展的系列。龙芯 1 号系列处理器主要面向嵌入式应用，龙芯 2
号超标量处理器系列主要面向桌面应用，而龙芯 3 号多核处理器系列主要面向服务器和高
性能机应用。根据需要，部分龙芯 2 号可以会面向高端嵌入式应用， 部分低端龙芯 3 号
处理器也可以用于高性能桌面应用。

本章将对龙芯 3 号处理器系列的互联架构，及 3B1500 处理器的具体实现作简单的介绍。

龙芯 3 号处理器系列互联架构
---------------------------

龙芯 3 号处理器系列采用了可伸缩的互联结构，如图 \ref{fig:gs3-connection} 所示。
这种可伸缩构架包含了两种不同层面的互联结构：（1）片内互联，和（2）片间互联。它
们都是通过二维网格的形式实现的。图 \ref{fig:gs3-connection}（a）显示的是单个结
点片内互联的情形：一个 $8\times8$ 的交叉开关将四个处理器核及四个二级 Cache 模块
连接到一起，而东（E）南（N） 西（W）北（N）四个方向可以用于与其他结点或外设互连
。 图 \ref{fig:gs3-connection}（b）和（c）则是片间互联的示例：（b）显示了一个
$2\times 2$ 的网格连接了四个结点，即总共 16 个处理器核；（c）则是一个 $4\times
4$ 网格连接 64 个处理器核的示意图。

\begin{figure}[htpb]
  \centering
  \begin{tabular}{ccc}
    \includegraphics[scale=1.3]{gs3a-node-simple} &
    \raisebox{1cm}{\includegraphics[scale=.65]{2x2}} &
    \includegraphics[scale=.65]{4x4} \\
  （a）结点结构 & （b）$2\times2$ 结点网格 & （c）$4\times4$ 结点网格
  \end{tabular}
  \caption{龙芯 3 号处理器系列互联架构}
  \label{fig:gs3-connection}
\end{figure}

图 \ref{fig:gs3-node} 给出了一个更详细的龙芯 3 号系列的结点结构图。每个结点内通
过两级 AXI 交叉开关将处理器核、 二级 Cache、 内存控制器以及 IO 控制器连接在一起
。 其中第一级 AXI 交叉开关 （X1 Switch，简称 X1）连接处理器核和二级 Cache， 而
第二级交叉开关（X2 Switch，简称 X2）则连接二级 Cache 和内存控制器及其他外设。

\begin{figure}[htpb]
  \centering
  \setlength\fboxsep{15pt}
  \setlength\fboxrule{.5pt}
  \fbox{\includegraphics[scale=.7]{gs3-node}}
  \caption{龙芯 3 号节点结构}
  \label{fig:gs3-node}
\end{figure}

在每个结点中，$8\times8$ 的 X1 交叉开关通过四个主端口连接四个 GS464V
处理器核（图中 P0、P1、P2、P3），四个从端口连接统一编址的四个
二级 Cache 块（图中 S0、S1、S2、S3）。 另外的四对主从端口连接东、 南、
西、北四个方向的其他结点或 I/O 外设（图中的 EM/ES、SM/SS、 WM/WS、NM/NS）。
X2 交叉开关通过四个主端口连接四个二级 Cache，至少一个从端
口连接一个内存控制器，至少一个从端口连接一个交叉开关的配置模块
（Xconf）用于配置本结点的 X1 和 X2 交叉开关的地址窗口等，还可以根据需要连接更多
的内存控制器和 IO 端口等。

龙芯 3 号系列的互连系统只是定义上层协议，而未对传输协议的实现做任何具体规
定，因此，结点之间的互连即可以采用片上网络进行实现，也可以通过 I/O 控制
链路实现多芯片的互连。以一个 4 结点 16 核系统为例，它既可以通过 4 片 4 核芯
片组成，也可以通过 2 片 8 核芯片，或基于一个单芯片 4 节点 16 核芯片组成。
由于互连系统的物理实现对软件透明， 不同配置的系统可以在相同的操作系统上运行。

龙芯 3B1500 处理器
------------------

龙芯 3B1500 是第一款龙芯 3 号多核处理器系列的处理器。 它基于可伸缩的龙芯 3
号系列多核互连架构设计， 在单个芯片上集成四个 GS464V 处理器核以及四个二级 Cache
模块，并通过高速 I/O 接口实现多芯片的互连以组成更大规模的系统。

\subsection{GS464V 处理器核}

GS464V 是四发射 64 位的 MIPS 兼容的高性能处理器核，面向高性能计算和高端嵌入式等
应用。图 \ref{fig:gs464-arch} 给出了 GS464V 的结构示意图。 GS464V 的主要特点是
在 GS464 的基础上，将两个 64 位浮点功能部件替换成了两个 256 位向量功能部件，每
个向量功能部件每拍可以完成 4 个 64 位运算，或 8 个 32 位运算，或 16 个 16 位运
算，或 32 个 8 位运算。因此，GS464V 在 1G 主频下就可以达到 16GFlops 浮点（双精
度）峰值性能。GS464V 在 GS464 支持的指令集的基础上，引入了大量新的 256 位向量指
令，对媒体处理、信号处理、科学计算和加解密等进行了专门的支持。在龙芯 3B1500 中
的 8 个 GS464V 核通过以及 SCache 模块通过 AXI 互联网络形成一个分布式共享 SCache
的多核结构。
\begin{figure}[htbp]
  \centering
  \includegraphics[scale=.55]{architecture-gs464v}
  \caption{GS464V 处理器核结构示意图}
  \label{fig:gs464-arch}
\end{figure}

GS464V 的主要特点如下：

 - MIPS64 兼容，并增加了 256 位向量指令、 SIMD 多媒体指令以及 x86 虚拟机指令；
 - 四发射超标量结构，两个定点、两个向量、一个访存部件；
 - 每个向量部件都支持 4 个全流水 64 位/8 个双 32 位定点或浮点乘加运算；
 - 访存部件支持 128 位存储访问，虚地址和物理地址各为 48 位；
 - 支持寄存器重命名、动态调度、转移预测等乱序执行技术；
 - 64 项全相联 TLB，独立的 16 项指令 TLB，可变页大小；
 - 一级指令 Cache 64KB，4 路组相联；
 - 一级数据 Cache 64KB，4 路组相联；
 - 独立不共享 Victim Cache 128KB，减少冲突失效；
 - 支持 Non-blocking 访问及 Load-Speculation 等访存优化技术；
 - 支持 Cache 一致性协议，可用于片内多核处理器；
 - 指令 Cache 实现奇偶校验，数据 Cache 实现 ECC 校验；
 - 支持标准的 EJTAG 调试标准，方便软硬件调试；
 - 标准的 128 位 AXI 接口。

\noindent 关于 GS464V 更详细的介绍请参考 龙芯3B1500（下）：GS464V 处理器核用户
手册。

\subsection{龙芯 3B1500 处理器实现}

作为龙芯 3 号多核处理器系列的第一款产品，龙芯 3B1500 是一款配置为单节点 4
核的处理器： 基于两级互连的结构， 四个 GS464V 核通过AXI 互联网络与二级 Cache
形成了一个分布式共享二级 Cache 的多核结构。 图 \ref{fig:ls3a-structure}
给出了龙芯 3B1500 的具体结构示意。

\begin{figure}[htbp]
  \centering
  \setlength\fboxsep{15pt}
  \setlength\fboxrule{.5pt}
  \fbox{\includegraphics[scale=.5]{gs3a-chip}}
  \caption{龙芯 3B1500 芯片结构}
  \label{fig:ls3a-structure}
\end{figure}

龙芯 3B1500 的第一层互连采用了 $6\times6$ 的交叉开关，用于连接四个 GS464V 处理器核
（作为主设备） 、四个 二级 Cache 模块（作为从设备） 、以及两个 IO
端口（每个使用一个主端口和一个从端口） 。一级互连开关连接的每个 IO 端口连接一个
16 位的 Hypertransport （HT） 控制器， 每个 16 位的 HT 端口还可以作为两个 8
位的 HT 端口使用。HT 控制器通过一个DMA 控制器和一级互联开关相连，DMA 控制器负责
IO 的 DMA 控制并负责片 间一致性的维护。 龙芯 3B1500 的 DMA
控制器还可以通过配置实现预取和矩阵转置。 第二级互连采用 5x4 的交叉开关， 连接 4
个 2 级 Cache 模块 （作为主设备） ， 两个 DDR2 内存控制器、低速高速 I/O（包括
PCI、LPC、SPI 等）以及芯片内部 的控制寄存器模块。
上述两级互连开关都采用读写分离的数据通道，数据通道宽度为 128 位，工
作在与处理器核相同的频率，用以提供高速的片上数据传输。 

龙芯 3B1500 采用 65nm 工艺制造， 最高工作主频为 1GHz。 其主要技术特征如下：
\begin{itemize}
	\item 片内集成 4 个 64 位的四发射超标量 GS464V 高性能处理器核；
	\item 片内集成 4 MB 的分体共享二级 Cache(由 4 个体模块组成，每个容量为 1MB) ；
	\item 通过目录协议维护多核及 I/O DMA 访问的 Cache 一致性；
	\item 片内集成 2 个 64 位 400MHz 的 DDR2/3 控制器；
	\item 片内集成 2 个 16 位 800MHz 的 HyperTransport 控制器；
	\item 每个 16 位的 HT 端口可以拆分成两个 8 路的 HT 端口使用；
	\item 片内集成 1 个 32 位的 100MHz PCIX/66MHz PCI 接口；
	\item 片内集成 1 个 LPC、2 个 UART、1 个 SPI、16 路 GPIO 接口。
\end{itemize}

\noindent 根据系统组成结构的不同，龙芯 3B1500 有两种工作模式：
\begin{enumerate}
  \item 单芯片模式： 系统只集成 1 片龙芯 3B1500， 构成一个对称多处理器系统 （SMP）；
  \item 多芯片互连模式： 系统中包含 2 片或 4 片龙芯 3B1500，通过龙芯 3B1500 的 HT
    端口进行互连，构成一个非均匀访存多处理器系统（CC-NUMA）。
\end{enumerate}
基于龙芯 3
号可扩展互联架构，4 片四核龙芯 3B1500 可以通过 HT 端口连接构成 4 芯片 16
核的对称多处理器系统 (SMP) 结构。

Cache 一致性
------------

龙芯 3B1500 实现了通过硬件维护

 - 通过 HT0 互联的处理器之间的 Cache 一致性，即片间 Cache 一致性；
 - 通过 HT 端口接入的外设之间的 Cache 一致性。

注意，龙芯 3B1500 的硬件不维护通过 PCI 等慢速总线接入到系统中的 I/O 设备的
Cache 一致性。在驱动程序开发时， 对通过 PCI 接入的设备进行 DMA （Direct Memory
Access） 传输时，仍然需要软件进行 Cache 一致性维护。

