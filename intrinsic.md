### Intrinsic测试用例自动生成器:
接口有两种类型:
> + 一种是Statement，如返回值为void类型的接口，无法被其他接口调用，类似这种接口，都实现为Statement。
> + 一种是Expression， 这种接口可以被其他接口调用。

有两颗树，一颗operation树，一颗value树。
> + operation有两个子节点，Statement和Expression(以下为默认开启的节点，不包含intrinsic节点)
>> + Statement有ControlStatement,以及各种Assign
>>> + ControlStatement中有if, for, switch
>> + Expression中有add, sub, less等
> + value子节点为int，float，schar，向量，矩阵等
> + block待补充

无控制语句时:  
在生成一条语句时，语句顶层部分一定为Statement。
先会在Statement中随机一个子节点op，随后会调用该op具体的generate。
在generate过程中，会随机进行Expression和value子节点的随机。
如果随机为Expression的子节点，会继续进行上面的操作(根据canProduce判断该expression是否可用)，直至随机到value节点。

for:
循环控制变量和循环体无关，只是单纯的生成了形似for (auto i = a; i <= b; i++) {}，大括号内和无控制语句的流程相同。

if:
在生成语句时，如果语句顶层为if，那么该语句并不会进行随机，而是属于ValueBase中的bool节点

switch:
在生成语句时，如果语句顶层为switch，那么该语句并不会进行随机，而是属于ValueBase中的int节点



## 记录一下intrinsic内部实现
### main.cc
1.读取解析配置文件
> + 首先设定各配置默认值
> + 配置文件存在并使用时，将默认值改写为文件中的给定值
 
2.随机seed(time)
3.启用各数据类型(如向量，矩阵，if，for等)
4.启用Intrinsic接口
> + 如果未启用配置，则开启全部接口
> + 如果启用配置，但include_only未添加接口，也开启全部接口
> + 将启用的接口写入enable_intrinsics(for debug)

5.emitHeader
> + 输出check函数相关语句，掩码寄存器的初始化等
> + 记录行号，输出行号，便于执行出错debug

6.Program::Program(生成语句数，每间隔n条语句生成对比语句)
> + 如果生成语句数大于0，将生成的intrinsic类型设定为Statement，即左值。
> + 如果配置文件中all_controls = true，则将类型改写为ControlStatement。
> + Operation op = OperationBase::random(nullptr, nullptr, type)
语句最上层，block和operation都为空。
>> + Operation operation = getRandom<OperationBase>(type)
从leaf_vecotr中随机一个(即从OperationBase树随机一个子节点)随机生成一个operation(由于type是statement，所以在这里其实op只能是Statement或ControlStatement)
>> + operation->setBlock(bk) 设置block
>> + operation->setOperation 设置operation
>> + return operation->generateAndRestoreIfFailed() ? operation : nullptr
>>> + 
> + stmt->saveLines() 把每个接口的lines添加到lines_中
> + 判断是否生成对比语句