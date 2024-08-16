# Asynchronous Library 异步库

异步库定义了异步控制器（`AsyncCtrl`）的基类`BaseAsyncCtrl`，以及它的子类`AsyncCtrl`和`AsyncCtrlDatapathCoupled`，用于控制异步消息的传递和处理。这些异步控制器共享`abstract`抽象特性、`stash`消息管理特性、以及相同的`FSM`有限状态机状态转换特性。

## `abstarct` 抽象类

为了隔离/解耦控制通路和数据通路，同时尽量少地暴露异步握手控制流以实现编程高安全性，akkanmore采用了`abstract`抽象类的设计方案，将异步握手控制模型`AsyncCtrl`作为本仿真系统的基础设施，在建模功能模块（functional units）时必须`extends`继承本库中对应的异步控制器`AsyncCtrl`类，并`override`实现功能模块对数据的`mainFunc`处理方法。不同功能模块对数据处理的方式不同，因此`mainFunc`方法在本库中未实现，导致`AsyncCtrl`各种类是抽象的。

`mainFunc`方法是建模功能模块的核心功能实现方法，该方法：
- 接收`AsyncCtrl`传递的异步消息`InputMsg`作为数据输入
- 根据异步消息的类型`MsgType`，执行不同的数据处理逻辑
- 生成新的异步消息`OutputMsg`作为数据输出，并指定输出对象`OutputAR`（`ActorRef`）
- 由于单个模块可以接收一个数据并同时发出多个数据，因此`OutputMsg`和`OutputAR`是`Seq`序列类型
- `AsyncCtrl`在适当的时候激活`mainFunc`方法，以此过程建模异步控制器对数据寄存器的***fire***触发过程

## `stash` 消息管理

`AsyncCrtl`异步控制器的私有属性`maxDepth`定义了消息队列的最大深度，即消息队列的最大容量。当消息队列的消息数量超过`maxDepth`时，消息队列将会阻塞，直到消息队列的消息数量小于`maxDepth`时，消息队列将会解除阻塞。

`maxDepth`的值在创建`AsyncCtrl`异步控制器时通过构造函数传入，当`maxDepth`的值大于1时，模拟的对象为具有buffer深度为`maxDepth`的异步控制器，当`maxDepth`的值等于1时，模拟的对象为无buffer的异步控制器。`maxDepth`的功能特性，在仿真建模一些带有buffer深度的电路模块时起到简化建模的作用。

`AsyncCtrl`针对`maxDepth`的行为特点是：
- 若当前深度`curDepth`小于`maxDepth`，则表明控制器的buffer深度未满，消息队列不阻塞，可以继续接收新消息的请求`Req`。控制器`AsyncCtrl`每接收一个消息请求`Req`，`curDepth`加1，同时向请求端发送应答`Ack`。
- 控制器每接收一个消息应答`Ack`，表明发送的消息请求`Req`已被接收端成功接收，可以解除其在buffer中的占用，即`curDepth`减1。
- 若当前深度`curDepth`大于等于`maxDepth`，则表明控制器的buffer深度已满，消息队列阻塞，不再接收新消息的请求`Req`。控制器`AsyncCtrl`不接收新消息请求`Req`，但接收消息应答`Ack`，直到消息队列的消息数量小于`maxDepth`时，消息队列解除阻塞，继续接收新消息的请求`Req`。

基于AKKA Actor的MailBox邮箱特性，本级控制器发送请求`Req`时即把消息发送至下一级控制器的MailBox邮箱中，而MailBox**默认可视为无界的**，不会丢失消息。因此，不同于实际电路中只有当请求`Req`被应答`Ack`之后，数据才会真正地流动到下一级，akkanmore异步控制器的数据流是依赖`maxDepth`控制的一种基于信誉的流控机制（credit-based flow control），消息数据随请求`Req`直接发送到下一级，上一级通过`maxDepth`判断是否继续发送请求`Req`，而下一级依赖理想无界的MailBox邮箱来保证消息不丢失。

MailBox每次收到新消息都会触发`onReceive`方法，`AsyncCtrl`在此刻被唤醒来判断是否处理该消息。对于暂时不能被处理的消息，需要通过`Stash`方法被重新放回MailBox邮箱中，如当当前深度`curDepth`大于等于`maxDepth`时，新收到的`Req`消息则不能被即刻处理（**即`Stash`此`Req`消息使其被追加到MailBox的Stash队列里**），而新收到的`Ack`消息立刻被处理（不需要`Stash`）；当收到足够多的`Ack`消息使得`curDepth`小于`maxDepth`时，再**使用`UnStashAll`方法一次性MailBox里Stash队列里的所有消息依次处理**。

不难发现，存在这种情况：若某控制器瞬间收到巨量的请求`Req`而都被追加到Stash队列里，当收到少量应答`Ack`使得`curDepth`小于`maxDepth`时，`UnStashAll`方法会一次性处理所有Stash队列里的消息并遍历，但至少极少量的请求消息`Req`会被处理，其余会被重新`Stash`进队列中。因此，请求消息`Req`在被处理前会经过大量无效的重复遍历访问，这种情况下会导致消息处理的效率降低。`Slice`是解决这种低效仿真的一种解决方案，细节在后续章节中讨论。

## `FSM` 有限状态机

异步电路的性能与电路状态相关，其特性具体表现在异步控制器`AsyncCtrl`在不同的电路状态下对`req`请求的`ack`应答行为不同。

在系统级层次仅能观察到有限的电路状态和行为。对于异步电路来说，我们重点关心以下特殊的电路状态：
- **数据令牌前向传播（forward pass of tokens）状态**：当流水线空闲时，即`inReq`输入请求到达本级流水之前所有`outReq`输出请求都`acked`已被应答，亦即在***Joint-Link***模型视角下`outputLink`输出路径的`empty`信号早于`inputLink`输入路径的`full`信号准备好，此时异步电路处于前向传播（forward）状态。Forward状态下，数据令牌tokens的前向传递速度较快，此效应在由Latch组成的异步电路尤其显著，因为空闲时的Latch对于输入数据是透明的（transparent），传递省去了clk-q的时间。对于由Flip-Flop组成的异步电路，效应有所减弱但仍可观察到。
- **空位气泡反向传播（backward pass of bubbles）状态**：当流水线繁忙时，即`inReq`输入请求到达本级流水时仍有`outReq`输出请求尚未`acked`被应答，亦即在***Joint-Link***模型视角下`outputLink`输出路径的`empty`信号晚于`inputLink`输入路径的`full`信号准备好，此时异步电路数据的前向传播相当于等待空位气泡向后传播，因而称为反向传播（backward）。Backward状态下，数据令牌tokens的前向传递的速度受限于空位气泡向后传播的速度，相比forward状态电路传播速度明显减慢，具体数值因电路而异。

在不同的状态下模拟不同的行为，从而实现对异步电路准确的仿真建模。

## 特殊异步控制器

一般情况下，异步控制器的控制通路与功能模块的数据通路的解耦的，以确保异步控制器的高安全性。但在某些特殊情况下，需要将控制通路与数据通路耦合在一起，以实现特定的功能。这种情况下，需要使用`AsyncCtrlDatapathCoupled`异步控制器，该异步控制器将控制通路与数据通路耦合在一起，以实现特定的功能。

常用的普通的异步控制器`AsyncCtrl`与功能模块的接口包括：
- 控制器向功能模块传递请求的消息`InputMsg`，在适当的时候激活功能模块的`mainFunc`方法，以模拟电路的***fire***触发过程；
- 功能模块向控制器传递处理后的消息`OutputMsg`及需要发送到的控制器代理`OutputAR`，以模拟电路数据流传递的过程。

特殊的异步控制器`AsyncCtrlDatapathCoupled`与功能模块除了以上接口，还包括：
- 功能模块在处理完数据消息并准备好`OutputMsg`和`OutputAR`后，向控制器额外发送一个`isDatapathReady`的布尔类型信号，参与控制器决定是否向该请求消息的发送端应答`Ack`。

换言之，普通解耦的异步控制器`AsyncCtrl`在决定是否应答新到的请求时只需考虑`cueDepth`是否小于`maxDepth`即是否有足够的buffer深度来接收处理消息，而耦合的异步控制器`AsyncCtrlDatapathCoupled`除此之外还需考虑数据通路的逻辑情况，数据通路可以设置任意的逻辑来控制异步控制器的握手。这在仿真建模某些异步电路时是必要的。

---
