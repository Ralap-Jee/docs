# OutputUnit 路由器输出单元

OutputUnit是路由器的输入buffer，负责缓存本节点发送到其他节点的路由数据包。**与InputUnit大致相同**，OutputUnit模型包含BaseOutputPort基本单元（AsyncCtrl-level抽象）和OutputUnit输出单元（Module-level抽象）两个部分。

## BaseOutputPort 基本单元

BaseOutputPort是仿真系统在`AsyncCtrl`异步控制器层级的建模单元，继承`AsyncCtrl`类，其主要逻辑功能实现`mainFunc`方法是简单的数据传递。与InputUnit的BaseInputPort相比，BaseOutputPort的并没有修改flit数据包的属性，因为这些属性在路由算法计算后已经被正确地处理了。

## OutputUnit 输出单元

OutputUnit是仿真系统在`Module`模块层级的建模单元，因其只是对BaseOutputPort的模块化封装，不需要独立的握手通信，所以只继承自`Actor`类拥有基本的通信功能即可。在OutputUnit内，创建了5个BaseOutputPort基本单元，分别对应5个输出端口，并对输入输出消息做简单的`transWhatever`转发，以实现正确`AsyncCtrl`之间的握手连接。OutputUnit的`transWhatever`方法与InputUnit的`transWhatever`方法大致相同，只是在转发给BaseOutputPort基本单元时，要根据flit消息属性`flitTo`来判断转发给哪个BaseOutputPort基本单元，而不是`flitFrom`属性。

---