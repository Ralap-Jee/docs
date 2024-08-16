# SwitchAllocator 路由开关/虚拟通道分配器

SwitchAllocator作为路由器的路由开关/虚拟通道分配器，是实现NoC路由功能的核心组件，主要实现对`VC`（Virtual Channel，虚拟通道）的分配，具体包括对三种`flit`类型的分配：`Head`、`Body`和`Tail`。

## BaseVirtualChannel 虚拟通道基类

虚拟通道基类`BaseVirtualChannel`是仿真系统对片上网络`VC`的建模单元，无需继承`AsyncCtrl`类，其属性包括：

- `VCid`：虚拟通道ID，类型为`Int`，数值大小不应超过设置的最大虚拟通道数，是区分不同虚拟通道的标识。
- `VCState`：虚拟通道状态，类型为`VCState`枚举类型，包括`Idle`、`Busy`两个合法状态，以及`Unknown`非法状态。
- `curInport`：当前虚拟通道的输入端口，类型为`Int`，数值含义为该虚拟通道已连接的输入端口`Inport`标识，**用-1表示未连接**。
- `curOutport`：当前虚拟通道的输出端口，类型为`Int`，数值含义为该虚拟通道已连接的输出端口`Outport`标识，**用-1表示未连接**。
- `flitTarget`：当前虚拟通道的路由目标地址，类型为`Seq[Int]`。因只有`head flit`会经过路由计算并得到新的路由目标地址，而该`packet`其他`body flit`和`tail flit`并不会重复经过路由计算，可通过此属性直接获得路由计算后的目标地址。

虚拟通道基类`BaseVirtualChannel`的主方法除了对以上属性实现修改和访问外，还包括：

- `occupy`：将`VCState`状态设置为`Busy`，同时检查此前状态是否为`Idle`，若不是则抛出异常。路由算法已经保证了`VC`的互斥性，因此不会出现多个`flit`同时占用一个`VC`的情况。
- `release`：将`VCState`状态设置为`Idle`，同时检查此前状态是否为`Busy`，若不是则抛出异常。同时，将`curInport`和`curOutport`设置为-1，解除此前的连接关系。