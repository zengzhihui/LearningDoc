# 编程范式 (Programming paradigm)
编程范式是一类典型的编程风格。编程范型和编程语言之间的关系可能十分复杂，由于一个编程语言可以支持多种范型。例如，C++设计时，支持过程化编程、面向对象编程以及泛型编程。

## 过程式编程 (Procedural programming)

## 面向对象 (Object-oriented programming)
一切都是对象。所有属性封装成一个对象，可以继承和使用多态。

## 函数式 (Funtional programing)
函数是头等公民，可以当做变量来使用。OC里的block，Swift里的closure以及Java的Lambda表达式。

**特征**

* 函数不维护任何状态
* 输入数据不可变动

**优点**

* 无状态就没有伤害，可以并行执行
* 重构代码不影响外部

**缺点**

* 数据复制泛滥。由于输入数据不可变，因此需要拷贝一份数据传出去

**例子**

NSArray的Map，Reduce, Filter

## 响应式 (Reactive programming)

## 泛型编程 (Generic programming)


## 修饰器模式

## 命令式 (Imperative programing)
命令“机器”如何去做事情(how)，这样不管你想要的是什么(what)，它都会按照你的命令实现。

## 声明式 (Declarative programing)
告诉“机器”你想要的是什么(what)，让机器想出如何去做(how)。

