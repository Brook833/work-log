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

#### 简单版本的实现1(无控制语句,单个Statement)
1. in main
2. in Program::Program
> + Operator op = Operator::random(nullptr, nullptr, tyep)
type为Statement
3. in OperationBase::random
> + Operation operation = getRandom<OperationBase>(type)
这里从所有Statement树的子节点中随机一个operation(如果配置文件中指定了某个或某些接口，那么子节点中就会只有这些接口)(此时已确定operation)
4. in OperationBase::generateAndRestoreIfFailed
该接口内分为generate和executeOnlySelf
由于generate和executeOnlySelf均为虚函数，所以此时会调用子节点具体的generate和executeOnlySelf

#### 简单版本的实现2(无控制语句，单个Expression)