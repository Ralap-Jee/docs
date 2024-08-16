# Asynchronous Message 异步消息库

异步消息定义了异步控制器（`AsyncCtrl`）传递的消息类型，所有基于异步控制器的建模模块的数据流都属于异步消息类。

异步消息类包括三种类型：PE内传递的脉冲异步消息（`SpkMsg`）、NoC内传递的微片异步消息（`FlitMsg`）、以及它们的基类普通异步消息（`CommonMsg`）。

此外消息库还定义了异步事件（`AsyncEvent`），用于异步控制器报告和记录异步事件以进行后续分析。

## 异步消息类

### CommonMsg 普通异步消息

普通异步消息是异步消息的基类，定义了异步消息的基本属性：
- `MsgType`：异步消息类型，由`GlobalParams.AsyncMsgType`定义，包括请求消息（`Req`）、应答消息（`Ack`）、非法消息（`Udefined`）。
- `MsgSrc`：异步消息源发送端。
- `MsgSink`：异步消息尾接收端。
- `MsgTiming`：异步消息的时序时间，类型为`(Int,Int,Int)`，分别表示该异步消息所属的时间步ID`timestep`、算法层ID`layer`、以及划片ID`slice`。
- `MsgCtx`：数条异步消息传递的数据内容，类型定义为`String`，控制器之间传递时可以协定类型并从`String`转换。

异步消息的方法是分别获得和修改这些私有属性的值。

### SpkMsg 脉冲异步消息

脉冲异步消息类继承自普通异步消息类，增加了脉冲异步消息的特有属性：
- `spkAddr`：脉冲异步消息的脉冲地址，类型为`(Int,Int)`，表示要激活的目标神经元的算法层ID`layerIdx`和该神经元在此层的ID`neuronIdxInLayer`。
- `tarAxon`：脉冲异步消息的目标轴突地址，类型为`Seq[Int]`，表示要激活的目标PE的ID列表。

脉冲异步消息的方法增加了获得和修改特有属性的值。

### FlitMsg 微片异步消息

微片异步消息类继承自普通异步消息类，增加了微片异步消息的特有属性：
- `flitType`：微片异步消息的微片类型，由`GlobalParams.FlitType`定义，包括`Head` `Body` `Tail` `Head/Tail`和`Undefined`非法类型。
- `flitSrc`：微片异步消息的发送端地址，类型为`(Int,Int)`，表示初始发送脉冲的神经元的地址`(layerIdx, neuronIdxInLayer)`。
- `flitTar`：微片异步消息的目标端地址，类型为`Seq[Int]`，序列长度大于1时为多播（Multicast）。
- `flitIdx`：当前flit在整个packet里的排序，最大值由`GlobalParams.NoC.numFlitPerPacket`定义。
- `flitFrom`：当前flit在上一个节点的输出端口地址，类型为`Int`，含义由`GlobalParams.RouterPorts`定义，用于路由算法计算输出端口和debug。
- `flitTo`：当前flit在本节点的输出端口地址，类型为`Int`，含义由`GlobalParams.RouterPorts`定义，用于路由算法计算输入端口和debug。
- `flitHop`：类型为`Int`，用于记录flit在NoC节点中的传输次数。
- `flitCong`：类型为`Int`，用于记录flit在NoC节点中的拥塞情况，在未来版本的分析中会用到。

## 异步事件

异步事件类定义了异步控制器的基础活动，用于异步控制器报告和记录异步事件以进行后续分析。

异步事件的私有属性包括：
- `workerName`：异步事件的持有者（Actor）昵称，类型为`String`。
- `workerAddress`: 异步事件的持有者（Actor）在Actor System中的地址，类型为`ActorPath`。
- `eventType`：异步事件的类型，由`GlobalParams.AsyncEventType`定义，包括`forwardPass` `backwardPass` `fakeBubbleTag`和`UndefinedEvent`非法类型。
- `systemTime`：异步事件发生的系统时间，类型为`java.time`，用于记录异步事件发生的时间戳。

异步事件的方法是获得这些私有属性的值或判断Worker和Type。异步事件一旦被创建，不允许被修改。

---