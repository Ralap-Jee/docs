# InputUnit 路由器输入单元

InputUnit是路由器的输入buffer，负责接收来自各个端口的路由数据包并压入FIFO，供路由器后续处理。InputUnit模型包含BaseInputPort基本单元（AsyncCtrl-level抽象）和InputUnit输入单元（Module-level抽象）两个部分。

## BaseInputPort 基本单元

BaseInputPort是仿真系统在`AsyncCtrl`异步控制器层级的建模单元，继承`AsyncCtrl`类，其主要逻辑功能实现`mainFunc`方法是简单的数据传递，并修改微片fLit数据包的两个属性：
- `flitFrom`：当前flit来自于哪个端口，类型为`Int`，数值含义由`GlobalParams.RouterPorts`定义，用于路由算法计算输出端口和debug。
- > 需要注意的是，flitFrom是在flit进入InputUnit时被赋值的，其语义是相对于路由仲裁控制SwitchAllocator单元来说，即SwitchAllocator需要访问`flitFrom`属性来得知这是来自哪个端口的数据包。因此，InputPort修改flitFrom属性为自身的port端口值。
- `flitTo`：当前flit在本节点的输出端口地址，类型为`Int`。由于尚未经过路由算法，因此flitTo属性的值仅为重置上一次路由计算的输出端口值，并设为debug值。

通过修改这两个属性，InputUnit实现了对flit数据包的简单处理，为后续路由算法提供了数据包的基础信息。

## InputUnit 输入单元

InputUnit是仿真系统在`Module`模块层级的建模单元，因其只是对BaseInputPort的模块化封装，不需要独立的握手通信，所以只继承自`Actor`类拥有基本的通信功能即可。在InputUnit内，创建了5个BaseInputPort基本单元，分别对应5个输入端口，并对输入输出消息做简单的`transWhatever`转发，以实现正确`AsyncCtrl`之间的握手连接。

`transWhatever`是InputUnit初始化后进入的工作状态，根据消息的`Req`和`Ack`类型、消息发送者、以及微片flit消息属性`flitTo`，将消息转发给对应的BaseInputPort基本单元。

>需要注意的是，在收到其他节点发送的请求消息时，要根据flit消息属性`flitTo`来判断转发给哪个BaseInputPort基本单元，，而不是`flitFrom`属性，因为这两个属性的语义仍是相对于上一个节点而言。在转发给InputPort之后，`mainFunc`才会调整语义对象到本节点，供后续正确路由。

---