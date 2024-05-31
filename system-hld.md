# AkkanMore HLD 高层次描述

# 项目概述

基于 [Akka](https://akka.io/) 的分布式编程框架，本项目设计了一个系统级异步类脑芯片仿真器，借助 [AkkA Actor](https://doc.akka.io/docs/akka/current/actors.html) 
的分布式异步通信行为来仿真模拟异步电路 各个模块之间的握手行为和数据流，在系统级高层次为异步类脑芯片的架构提供先验性的仿真性能分析，并保留丰富的可拓展接口，提供了异步芯片架构仿真系统的开发解决方案。

![](images\system-architecture.png "akkanmore-system-architecture")

## 类脑架构概述

本项目聚焦于大规模异步类脑芯片系统级架构的仿真，该架构是基于数据流（data flow）结构进行计算的，包含多个异步处理单元（PE），这些PE通过片上网络（NoC）进行互连和通信。为了匹配类脑计算的脉冲稀疏特性和事件驱动特性，大多数类脑架构都使用异步电路来设计实现，以充分提高能效、降低功耗。与实现矩阵或卷积运算的PE脉动阵列不同，由于类脑计算传递脉冲的稀疏特性，并没有显著多的数据搬运（data movement），因而类脑架构里的PE总是通过NoC路由器来进行脉冲地址的传递和通信。

**类脑架构的基本组成单元有**：
1. **异步处理单元（PE）**：类脑芯片的基本计算单元。
   1. **脉冲神经元（Neuron）**：类脑芯片的基本计算单元，PE内部包含多个并行计算的神经元。
   2. **权重存储器（Weight SRAM）**：PE内部的权重存储SRAM，存储PE内部神经元的对应每个突触（synapse）的权重。
   3. **脉冲地址表达编码器（AER）**：脉冲地址编码器，将PE内发放脉冲的神经元编码成对应脉冲地址，并根据轴突共享原则（axon sharing rule）索引需要发送至的目标PE。
   4. **脉冲地址解码器（LUT）**：解码器，将PE内接收到的脉冲地址解码成应激活的神经元，与查找表的功能相似。
   5. **网络接口（Network Interface）**：接口桥梁，连接PE与NoC，负责PE的脉冲数据与NoC的flit数据之间的转换。
2. **片上网络路由器（NoC Router）**：类脑芯片的通信路由器，负责网络接口与路由器、路由器与路由器之间的NoC数据包（packet）和数据微片（flit）的传输。
   1. **开关分配器（Switch Allocator）**：输入端口和虚拟通道的仲裁器、以及输出端口的开关分配器，负责实现NoC路由核心功能，包括仲裁、开关分配、路由算法等。
   2. **输入端口（Input Unit）**：路由器的输入端口，负责缓冲接收到的数据。
   3. **输出端口（Output Unit）**：路由器的输出端口，负责缓冲准备发送的数据。


## 项目结构

    docs/                   # The documention directory.
    logs/                   # The simulation ouput log directory.
    project/                # The building files of the project using Sbt.
    src/main/               # The main source code directory.
    ├── resources/
    │   ├── logback.xml     # The logback configuration file.
    ├── scala/
    │   ├── akkanmore/      # The main package of the project.
    │   │   ├── AsyncLib/   # The asynchronous library package.
    │   │   │   ├── ...
    │   │   ├── NoC/        # The network-on-chip modeling package.
    │   │   │   ├── ...
    │   │   ├── PE/         # The processing element modeling package.
    │   │   │   ├── ...
    │   │   ├── config/     # The configuration parameter package.
    │   │   │   ├── ...
    │   │   ├── example/    # The example usage.
    │   │   │   ├── ...
    │   │   ├── util/       # The utility package.
    │   │   │   ├── ...
    └────
    target/                 # The building output directory.
    README.md               # The README file of the project.
    build.sbt               # The building configuration file of the project.

## 项目信息

开发人员名单可见于 [贡献者](https://github.com/Ralap-Jee/akkanmore/graphs/contributors)
