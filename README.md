#BehaviourTree
##写在最前面的话

如果你是刚接触到行为树的概念的话。我推荐你先到本项目的Wiki中了解一下。行为树究竟是怎么一回事比较好。当然如果你是一名老手，只是想借用本项目中的代码的话，请你继续看下去吧

##使用本行为树的项目
- 比特小队 https://www.taptap.com/app/23452

##项目说明
~~~
本行为树的代码使用Lua编写，所有的内容也建立的Lua的基础语法之上

因为公司项目需求，需要一套Lua的行为树代码，所以尝试从饥荒中抽离了行为树相关的代码。绝大多数节点行为与饥荒中相同，不过部分节点因为也许需求也有部分变动
~~~
##通用说明
~~~
行为树状态基本分为4种
READY：准备状态，节点还没有被调用过。或者已经调用结束被Reset之后的状态
RUNNING：正在运行的状态，通常父节点会等待子节点Runing结束才会将自己的状态标示为结束，当然部分节点不会理会子节点的Runing状态
SUCCESS：运行成功
FAILED：运行失败

绝大部分节点都有对应的中文名字，不过部分节点我也不知道怎么翻译

行为树会有被打断的情况，被打断时。子节点的Reset函数会被调用。必要的情况下重写Reset函数可以处理被打断的情况

行为树在某些情况下也会被暂停。这棵树中留下相关的暂停（Suspend）与重启（Restart）函数。需要树被暂停的时候外部调用，这个方法内部会将对应的方法传播到每一个节点之中
~~~
##节点说明
###BehaviourTree
~~~
树节点，主要用于子节点的调用，状态保存等。没有实际的业务功能
~~~
###BehaviourNode
~~~
所有节点的父节点，没有实际的业务意义，主要是维护了各种基础状态的方法
~~~
###DecoratorNode（装饰节点）
~~~
没有实际的业务意义，不过通常是只有一个子节点的节点的父类
~~~
###ConditionNode（判定节点）
~~~
创建这个节点的时候传入一个能够获取判定值的方法，这个节点会根据运行到这个节点时，传入方法的调用后返回的值，改变当前节点的状态。nil 或者 false 转换为FAILED，否则转换为SUCCESS
~~~
###ConditionWaitNode（条件等待节点）
~~~
与判定基本相同。不同的是在原本会判定为FAILED的情况下判定为RUNNING
~~~
###ActionNode（动作节点）
~~~
创建这个节点的时候传入一个函数。当运行到这个节点的时候。会调用这个函数，并且将节点的状态标示为SUCCESS
~~~
###WaitNode（等待节点）
~~~
创建这个节点时传入一个时间值。当运行到这个节点时，从开始到时间结束这个节点的状态会一直是RUNNING。等待时间结束后，节点状态会修改为SUCCESS。
PS：等待过程中，这个节点会休眠。用于减少性能消耗，其他类似功能节点在编写时，应该也需要做出适当的休眠
~~~
###SequenceNode（顺序节点）
~~~
创建这个节点时，需要传入一个节点队列。当运行到这个节点时。他的子节点会一个接一个的运行。如果他的子节点状态是SUCCESS，那么他会运行下一个；如果他的子节点状态是RUNNING，那么他会将自身也标识成RUNNING，并且等待节点返回其他结果；如果他的子节点状态是FAILED，那么他会把自己的状态标识为FAILED并且直接返回。所有节点都返回结尾为SUCCESS那么他会将自身标识成为SUCCESS并且返回。
~~~
###SelectorNode（选择节点）
~~~
与顺序节点类似，创建时需要传入一个节点列表，当运行到这个节点时，他的节点会一个接一个的运行。如果他的子节点是SUCCESS，那么他会将自身标识成为SUCCESS并且直接返回；如果他的子节点状态是RUNNING，那么他会将自身也标识成RUNNING，并且等待节点返回其他结果；如果他的子节点状态是FAILED，那么他会运行下一个。任何一个节点都没有返回SUCCESS的情况下，他将会将自身标识成为FAILED并且返回
~~~
###NotDecorator（取反节点）
~~~
创建这个节点的时候需要传入一个子节点。并且将子节点的运行结果除了RUNNING外颠倒下结果并且返回
~~~
###FailIfRunningDecorator
~~~
创建这个节点的时候需要传入一个子节点。并且将子节点的除了RUNNING结果以外标示为自己的结果；如果子节点的状态是RUNNING那么就讲自身的状态标识为FAILED。并且返回
~~~
###RunningIfFailDecorator
~~~
创建这个节点的时候需要传入一个子节点。并且将子节点的除了FAILED结果以外标示为自己的结果；如果子节点的状态是FAILED那么就讲自身的状态标识为RUNNING。并且返回
~~~
###LoopNode（循环节点）
~~~
创建这个节点的时候需要传入一个节点队列和最多循环的次数。循环节点将尝试执行N次后将自身标识为SUCCESS后返回结果。运行中的状态与顺序节点的逻辑大体一致，子节点的RUNNING会阻止下一个的运行，子节点的FAILED会中断运行
~~~
###RandomNode（随机节点）
~~~
创建这个节点的时候需要传入一个节点队列和可以不传的列表对应的权重。在没有权重的情况下，所有的子节点完全随机。在有权重的情况下，子节点会按照权重的设置值来选择某一个节点来运行，并且将这个节点的运行结果标识为自己的状态，并且返回。
PS：权重的个数可以与子节点列表的个数不同。权重个数不足的将以1填充，超过的部分会被截断
~~~
###ParallelNode（并行节点）
~~~
创建这个节点的时候需要传入一个节点队列。一个接一个的运行子节点。如果子节点的状态是FAILED，那么它会将自己标识为FAILED并且直接返回；如果子节点的状态是SUCCESS或者RUNNING，那么它会运行下一个节点。只有所有的节点都标识为SUCCESS它会将自己的标识为SUCCESS并且返回，否则他会将自己标识为RUNNING。
PS：部分节点（ConditionNode、NotDecorator）会在运行前被强制重启，用于判定
~~~
###ParallelNodeAny
~~~
与ParallelNode基本相同，唯一不同的地方在于任何一个节点标识为SUCCESS的时候，它就会将自己的状态标识为SUCCESS并且返回
~~~
###WhileNode
~~~
ParallelNode节点的扩展节点，创建这个节点的时候需要传入一个可以获取判定值的方法和一个子节点。当运行到这个节点的时候，需要先做判定，然后在执行后边的节点。如果判定失败则直接将自身标识为FAILED并且返回。否则将子节点的状态标识为自己的状态并且返回
~~~
###IfNode
~~~
SequenceNode节点的扩展节点。与WhileNode节点基本一致，不同的是，判定的条件只有第一次进入的时候生效，之后的运行将直接运行后边的节点
~~~
###WeightSelectNode
~~~
WeightSelectNode权重选择节点，与选择节点功能类似。重新运行的时候会按照权重进行排序。权重大的选在在前。权重值可以设置为数字或者函数。如果使用函数返回数据必须是一个数字。可以做一些动态调整节点顺序的相关AI
~~~