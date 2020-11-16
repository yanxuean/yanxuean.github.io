---
layout: post
title: Clean Architecture
categories: [software engineering]
description: Clean Architecture
keywords: Architecture
---

# Clean Architecture
陈浩的一个观点： 各种范式或微服务架构都是为了解决一个问题：分离控制和逻辑。
控制是指：对程序流转的与业务逻辑无关的代码或系统的控制（如多线程，异步，服务发现，部署，弹缩等）。
逻辑是指：实实在在的业务逻辑，是解决用户问题的逻辑。
控制和逻辑构成了整体的软件复杂度，有效的分离控制和逻辑会让你的系统得到最大的简化。
区分：简单vs简陋，平衡vs妥协，迭代vs半成品

本书的结构：编程范式，设计原则，软件架构


## 架构的定义
架构就是“用最小的人力成本来满足构建和维护系统需求”的设计行为

软件架构的规则就是排列组合代码块的规则

好的架构可以节省软件项目构建与维护的人力成本。用最小的成本，最大程度满足功能性和灵活性的要求

## 第1部分 概述

### 1.设计和架构究竟是什么
两个是同一个事情，没有区别。
架构：这个词一般用在更高级的讨论中，这种讨论一般不考虑底层实现细节。
设计：往往用来指代具体的系统底层组织结构和实现的细节。

例子：建筑设计图既包括大的东西（外形，布局），还包括水电管道等细节，两者不可分割，没有清晰的分界线。

#### 目标：用最小的人力成本来满足构建和维护该系统的需求。
其优劣用它满足用户需求所需的成本来衡量。

差系统特点：生产力下降，成本上升，无解

不要太自信。从开始就低估好的，良好设计的，整洁的代码的重要性。

例子：要想跑得快，先要跑的稳


### 2.两个价值维度
行为价值：实现客户要求的功能。
架构价值：soft 灵活性，ware 产品。

变更实施的难度应该和变更的范畴（scope）成等比关系，与变更的具体形状（shape）无关。   ---- shape是啥？

架构>行为： 因为不能变更的系统没价值。只有架构好的系统才能变更。


紧急重要四象限。 行为都是紧急的，架构都是重要的。

业务部门（需求方）没有能力评估系统架构的重要程度，是研发人员自己的职责。守护系统架构，守护可维护性

## 第2部分 从基础构件开始：编程范式




## 第3部分 设计原则
提出时间： 函数式编程（1958）》面向对象（1966） 》结构化编程（1968）
共同点：都是对用户进行能力限制的

### 结构化编程（限制控制权的直接转移）

一个理论：用顺序结构，分支结构，循环结构这三种可以构造出任何程序。	
	顺序结构：其正确性可以通过枚举法证明。过程：针对序列中的每个输入，跟踪其对应输出值得变化。
	分支结构：其正确性也可以通过枚举法证明。过程：用枚举法证明分支结构中每个路径的正确性。
	循环结构：其正确性要通过数学归纳法证明。过程：用枚举法证明循环1次的正确性，再证明如果循环N次是正确的，则N+1次也是正确的。还要用枚举证明循环结构的起始和结束条件的正确性。
	
	
		
系统正确性的推导方法：
	数学推导方法（欧几里得层级构造定理，一种形式化证明方法）：
		借鉴公理，定理，推论，形成欧几里得结构。 
	    整体 = 已证明的结构子单元 + 对其进行串联的部分父单元的证明。（递归拆分）
		
		太冗长复杂。
		
	科学证明法：
		科学与数学的不同：
			科学理论和科学定理通常是无法证明的。可以被证伪，无法被证明。
			科学方法论不需要证明其是正确的，只需要想办法证明它是错误的。
			数学是证明，科学是证伪
		例子：这句话是假的
		
测试：只能证伪，不能证明。测试的作用是让我们得出某段程序已经足够实现当前目标这一结论。（截止到当前还无法证伪，就是正确。）
		
软件开发其是不是一个数学研究过程，更像是一门科学研究学科，我们通过无法证伪来证明其正确性。

结构化编程范式：最有价值的地方是赋予了我们创造可证伪程序单元的能力。
	“goto语句在单元间的跳转用法”会导致上面的方法不可用，无法递归拆分。
	“goto语句在单元内的跳转用法”没有问题，但可以用if-else或do-while更好表现。
		
	要增加限制：限制无限制的跳转，避免导致无法做功能降解拆分，

总结：递归拆解 + 测试证伪

### 面向对象编程（限制控制权的间接转移）

“封装，继承，多态” 不是面向对象自带的，面向对象只是提高了使用便利性。
“多态”类似接口
程序应该与设备无关。需要的是在不同设备上实现相同的功能，不是支持不同的设备

面向对象范式的核心本质：可以通过多态（接口）完全控制系统中所有的源代码依赖关系，不再受控制流的限制。

好处：可以方便的构造插件式架构，让高层策略性组件与底层实现性组件相分离，底层组件可以被编译成插件，实现独立于高层组件的开发和部署。


### 函数式编程（限制赋值操作。）
变量不可变。
引子：所有竞争问题，死锁问题，并发更新问题都是由可变变量导致。

两种方法：
可变性的隔离：
	将内部服务进行拆分，划分为“可变的，不可变的”两种。不可变组件用纯函数方式执行，
	不可变组件通过与非函数组件通讯的方式来修改变量状态。
	变量通过“事务型内存”来保护，避免同步更新和竞争状态的发生。

事件溯源：
	只存储事务，不存储具体状态。需要具体状态时，通过从头计算所有事务得到。
	优点：不存在删除和更新操作，即不是CRUD，而是CR
	


### 7. SRP 单一职责原则

定义：
	正解：任何一个软件模块都应该只对某一类行为者actor负责。
	      软件模块：指一组紧密相关的函数和数据结构。
	误解：每个模块都应该只做一件事。 or 任何一个模块都硬件有且仅有一个被修改的理由。


例子没看特别明白。

### 8.OCP开闭原则
设计良好的软件应该易于扩展，同时抗拒修改（Add，no Modify）。

对旧代码的修改尽量降低（先通过SRP分组，在通过DIP调整分组之间的依赖关系。）
可根据相关函数被修改的原因，修改的方式，修改的时间来进行分组隔离，并将这些分组整理成组件结构，使高阶组件不依赖低阶组件。

将系统划分为一系列组件，并将组件间的依赖关系按层次结构进行组织，使高阶组件不会因低阶组件被修改而受到影响。

用接口反转依赖关系。
接口也可用于信息隐藏

#### 怎么防止过度设计？
仅仅着眼于当前，跨一步都嫌多。或处处增加灵活性，耗费很多成本。
都不可取！如何把握度？具体如何操作？
方法1：第1次设计时不考虑，第二次有需求时再引入灵活性。
方法2：第1次设计时如果引入灵活性的成本不大，就引入。如果比较大就不引入。

### 9、 LSP里氏替换原则

指导接口与其实现方式的原则


### 10、ISP：接口隔离原则
LSP是管兄弟关系，ISP是管客户关系。

不要依赖不需要的东西。

### 11、DIP：依赖反转
要关注经常会变动的模块，不用到处引入灵活性。

接口比实现更稳定。

编码守则：
	应在代码中多使用抽象接口，尽量避免使用那些多变的具体实现类。对象的创建也应该收到严格限制，多用抽象工厂。
	不要再具体实现类上创建衍生类。继承关系最难修改，尽量少用。
	不要覆盖override包含具体实现的函数。可用抽象函数解决
	要避免在代码中写入与任何具体实现相关的名字，或是其他容易变动的事务的名字。
	
具体实现层要依赖于抽象层。



依赖关系 与 控制流 相反。
不可能完全消除违反DIP的情况，尽量集中起来。


