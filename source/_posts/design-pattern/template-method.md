---
title: Template Method 模版方法模式
categories:
  - design-pattern
date: 2024-04-21 14:12:56
---

## Intent 意图

模板方法是一种行为设计模式，它定义超类中算法的骨架，但允许子类在不改变其结构的情况下覆盖算法的特定步骤。template-method-header


<div align="center"> <img src="/images/template-method-header.png"/><p style="text-align: center;"></p></div>


## Problem 问题

假设您正在创建一个用于分析公司文档的数据挖掘应用程序。用户以各种格式（PDF、DOC、CSV）向应用程序提供文档，并尝试以统一格式从这些文档中提取有意义的数据。

该应用程序的第一个版本只能与 DOC 文件一起使用。在以下版本中，它能够支持 CSV 文件。一个月后，您“教”它从PDF文件中提取数据。


<div align="center"> <img src="/images/template-method-problem.png"/><p style="text-align: center;">数据挖掘类包含大量重复代码。</p></div>


在某个时候，你注意到这三个类都有很多相似的代码。虽然处理各种数据格式的代码在所有类中都完全不同，但用于数据处理和分析的代码几乎相同。摆脱代码重复，保持算法结构完好无损不是很好吗？

还有另一个与使用这些类的客户端代码相关的问题。它有很多条件，这些条件根据处理对象的类选择适当的操作过程。如果所有三个处理类都具有公共接口或基类，则可以在客户端代码中消除条件，并在对处理对象调用方法时使用多态性。

## Solution 解决方案

模板方法模式建议将算法分解为一系列步骤，将这些步骤转换为方法，并在单个模板方法中放置对这些方法的一系列调用。这些步骤可以是 模板方法模式建议将算法分解为一系列步骤，将这些步骤转换为方法，并在单个模板方法中放置对这些方法的一系列调用。这些步骤可以是 `abstract` ，也可以具有一些默认实现。要使用该算法，客户端应该提供自己的子类，实现所有抽象步骤，并在需要时覆盖一些可选步骤（但不是模板方法本身）。 ，也可以具有一些默认实现。要使用该算法，客户端应该提供自己的子类，实现所有抽象步骤，并在需要时覆盖一些可选步骤（但不是模板方法本身）。

我们看看这将如何在我们的数据挖掘应用程序中发挥作用。我们可以为所有三种解析算法创建一个基类。此类定义一个模板方法，该方法由对各种文档处理步骤的一系列调用组成。


<div align="center"> <img src="/images/template-method-solution.png"/><p style="text-align: center;">模板方法将算法分解为多个步骤，允许子类覆盖这些步骤，但不能覆盖实际方法。</p></div>


首先，我们可以声明所有步骤 `abstract` ，迫使子类为这些方法提供自己的实现。在我们的例子中，子类已经具有所有必要的实现，因此我们唯一需要做的就是调整方法的签名以匹配超类的方法。

现在，让我们看看我们可以做些什么来摆脱重复的代码。对于各种数据格式，打开/关闭文件和提取/解析数据的代码似乎不同，因此没有必要触及这些方法。但是，其他步骤（例如分析原始数据和编写报告）的实现非常相似，因此可以将其拉到基类中，子类可以在其中共享该代码。

正如你所看到的，我们有两种类型的步骤：

- 抽象步骤必须由每个子类实现
- 可选步骤已经有一些默认实现，但如果需要，仍然可以覆盖
- 还有另一种类型的步骤，称为钩子。钩子是具有空体的可选步骤。即使钩子没有被覆盖，模板方法也会起作用。通常，钩子放置在算法的关键步骤之前和之后，为子类提供算法的附加扩展点。

## Real-World Analogy 真实世界的类比


<div align="center"> <img src="/images/template-method-live-example.png"/><p style="text-align: center;">典型的建筑计划可以稍作改动，以更好地满足客户的需求。</p></div>


模板方法方法可用于大规模住房建设。建造标准房屋的建筑计划可能包含几个扩展点，这些扩展点可以让潜在所有者调整最终房屋的一些细节。

每个建筑步骤，如奠基、框架、砌墙、安装水管和水电布线等，都可以稍作改动，使最终的房子与其他房子略有不同。

## Structure 结构


<div align="center"> <img src="/images/template-method-structure.png"/><p style="text-align: center;"></p></div>


1. Abstract 类声明充当算法步骤的方法，以及按特定顺序调用这些方法的实际模板方法。这些步骤可以声明 Abstract 类声明充当算法步骤的方法，以及按特定顺序调用这些方法的实际模板方法。这些步骤可以声明 `abstract` ，也可以具有一些默认实现。 ，也可以具有一些默认实现。
2. 具体类可以覆盖所有步骤，但不能覆盖模板方法本身。

## Pseudocode 伪代码

在此示例中，模板方法模式为简单策略视频游戏中的人工智能的各个分支提供了一个“骨架”。


<div align="center"> <img src="/images/template-method-example.png"/><p style="text-align: center;">简单视频游戏的 AI 类。</p></div>


游戏中的所有种族都有几乎相同类型的单位和建筑物。因此，您可以为各种种族重复使用相同的 AI 结构，同时能够覆盖一些细节。通过这种方法，您可以覆盖兽人的 AI 使其更具攻击性，使人类更具防御性，并使怪物无法建造任何东西。向游戏添加新种族需要创建一个新的 AI 子类并覆盖基 AI 类中声明的默认方法。

```java
// The abstract class defines a template method that contains a
// skeleton of some algorithm composed of calls, usually to
// abstract primitive operations. Concrete subclasses implement
// these operations, but leave the template method itself
// intact.
class GameAI is
    // The template method defines the skeleton of an algorithm.
    method turn() is
        collectResources()
        buildStructures()
        buildUnits()
        attack()

    // Some of the steps may be implemented right in a base
    // class.
    method collectResources() is
        foreach (s in this.builtStructures) do
            s.collect()

    // And some of them may be defined as abstract.
    abstract method buildStructures()
    abstract method buildUnits()

    // A class can have several template methods.
    method attack() is
        enemy = closestEnemy()
        if (enemy == null)
            sendScouts(map.center)
        else
            sendWarriors(enemy.position)

    abstract method sendScouts(position)
    abstract method sendWarriors(position)

// Concrete classes have to implement all abstract operations of
// the base class but they must not override the template method
// itself.
class OrcsAI extends GameAI is
    method buildStructures() is
        if (there are some resources) then
            // Build farms, then barracks, then stronghold.

    method buildUnits() is
        if (there are plenty of resources) then
            if (there are no scouts)
                // Build peon, add it to scouts group.
            else
                // Build grunt, add it to warriors group.

    // ...

    method sendScouts(position) is
        if (scouts.length > 0) then
            // Send scouts to position.

    method sendWarriors(position) is
        if (warriors.length > 5) then
            // Send warriors to position.

// Subclasses can also override some operations with a default
// implementation.
class MonstersAI extends GameAI is
    method collectResources() is
        // Monsters don't collect resources.

    method buildStructures() is
        // Monsters don't build structures.

    method buildUnits() is
        // Monsters don't build units.
```

## Applicability 适用性

如果希望客户端仅扩展算法的特定步骤，而不扩展整个算法或其结构，请使用模板方法模式。

模板方法允许您将单体算法转换为一系列单独的步骤，这些步骤可以由子类轻松扩展，同时保持超类中定义的结构不变。

当您有多个类包含几乎相同的算法但有一些细微差异时，请使用该模式。因此，当算法更改时，您可能需要修改所有类。

当您将此类算法转换为模板方法时，还可以将具有类似实现的步骤拉到超类中，从而消除代码重复。子类之间不同的代码可以保留在子类中。

## How to Implement 如何实现

1. 分析目标算法，看看是否可以将其分解为步骤。考虑哪些步骤对所有子类都是通用的，哪些步骤将始终是唯一的。
2. 创建抽象基类并声明模板方法和一组表示算法步骤的抽象方法。通过执行相应的步骤，在模板方法中概述算法的结构。请考虑创建模板方法 `final` 以防止子类重写它。
3. 如果所有步骤最终都是抽象的，那也没关系。但是，某些步骤可能会受益于默认实现。子类不必实现这些方法。
4. 考虑在算法的关键步骤之间添加钩子。
5. 对于算法的每个变体，创建一个新的具体子类。它必须实现所有抽象步骤，但也可以覆盖一些可选步骤。

## Pros and Cons 优点和缺点

| 优点√                                                                                | 缺点×                                                  |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| 您可以让客户端仅覆盖大型算法的某些部分，从而减少它们受到算法其他部分发生更改的影响。 | 某些客户端可能会受到算法提供的框架的限制。             |
| 您可以将重复的代码拉取到超类中。                                                     | 通过子类抑制默认步骤实现，可能会违反 Liskov 替换原则。 |
|                                                                                      | 模板方法的步骤越多，维护起来就越难。                   |

## Relations with Other Patterns  
与其他模式的关系

- 工厂方法是模板方法的专业化。同时，工厂方法可以作为大型模板方法中的一个步骤。
- 模板方法基于继承：它允许您通过在子类中扩展算法的各个部分来更改这些部分。策略基于组合：您可以通过为对象提供与该行为相对应的不同策略来改变对象的部分行为。Template Method 在类级别工作，因此它是静态的。策略在对象级别工作，允许您在运行时切换行为。

# **Template Method** in Python

模板方法是一种行为设计模式，它允许您在基类中定义算法的骨架，并让子类在不更改整体算法结构的情况下覆盖这些步骤。

#### main.py：概念示例

```python
from abc import ABC, abstractmethod


class AbstractClass(ABC):
    """
    The Abstract Class defines a template method that contains a skeleton of
    some algorithm, composed of calls to (usually) abstract primitive
    operations.

    Concrete subclasses should implement these operations, but leave the
    template method itself intact.
    """

    def template_method(self) -> None:
        """
        The template method defines the skeleton of an algorithm.
        """

        self.base_operation1()
        self.required_operations1()
        self.base_operation2()
        self.hook1()
        self.required_operations2()
        self.base_operation3()
        self.hook2()

    # These operations already have implementations.

    def base_operation1(self) -> None:
        print("AbstractClass says: I am doing the bulk of the work")

    def base_operation2(self) -> None:
        print("AbstractClass says: But I let subclasses override some operations")

    def base_operation3(self) -> None:
        print("AbstractClass says: But I am doing the bulk of the work anyway")

    # These operations have to be implemented in subclasses.

    @abstractmethod
    def required_operations1(self) -> None:
        pass

    @abstractmethod
    def required_operations2(self) -> None:
        pass

    # These are "hooks." Subclasses may override them, but it's not mandatory
    # since the hooks already have default (but empty) implementation. Hooks
    # provide additional extension points in some crucial places of the
    # algorithm.

    def hook1(self) -> None:
        pass

    def hook2(self) -> None:
        pass


class ConcreteClass1(AbstractClass):
    """
    Concrete classes have to implement all abstract operations of the base
    class. They can also override some operations with a default implementation.
    """

    def required_operations1(self) -> None:
        print("ConcreteClass1 says: Implemented Operation1")

    def required_operations2(self) -> None:
        print("ConcreteClass1 says: Implemented Operation2")


class ConcreteClass2(AbstractClass):
    """
    Usually, concrete classes override only a fraction of base class'
    operations.
    """

    def required_operations1(self) -> None:
        print("ConcreteClass2 says: Implemented Operation1")

    def required_operations2(self) -> None:
        print("ConcreteClass2 says: Implemented Operation2")

    def hook1(self) -> None:
        print("ConcreteClass2 says: Overridden Hook1")


def client_code(abstract_class: AbstractClass) -> None:
    """
    The client code calls the template method to execute the algorithm. Client
    code does not have to know the concrete class of an object it works with, as
    long as it works with objects through the interface of their base class.
    """

    # ...
    abstract_class.template_method()
    # ...


if __name__ == "__main__":
    print("Same client code can work with different subclasses:")
    client_code(ConcreteClass1())
    print("")

    print("Same client code can work with different subclasses:")
    client_code(ConcreteClass2())

```

#### 输出.txt：执行结果

````
Same client code can work with different subclasses:
AbstractClass says: I am doing the bulk of the work
ConcreteClass1 says: Implemented Operation1
AbstractClass says: But I let subclasses override some operations
ConcreteClass1 says: Implemented Operation2
AbstractClass says: But I am doing the bulk of the work anyway

Same client code can work with different subclasses:
AbstractClass says: I am doing the bulk of the work
ConcreteClass2 says: Implemented Operation1
AbstractClass says: But I let subclasses override some operations
ConcreteClass2 says: Overridden Hook1
ConcreteClass2 says: Implemented Operation2
AbstractClass says: But I am doing the bulk of the work anyway
````

# **Template Method** in Rust

模板方法是一种行为设计模式，它允许您在基类中定义算法的骨架，并让子类在不更改整体算法结构的情况下覆盖这些步骤。

## Conceptual Example 概念示例

####  **main.rs**

```rust
trait TemplateMethod {
    fn template_method(&self) {
        self.base_operation1();
        self.required_operations1();
        self.base_operation2();
        self.hook1();
        self.required_operations2();
        self.base_operation3();
        self.hook2();
    }

    fn base_operation1(&self) {
        println!("TemplateMethod says: I am doing the bulk of the work");
    }

    fn base_operation2(&self) {
        println!("TemplateMethod says: But I let subclasses override some operations");
    }

    fn base_operation3(&self) {
        println!("TemplateMethod says: But I am doing the bulk of the work anyway");
    }

    fn hook1(&self) {}
    fn hook2(&self) {}

    fn required_operations1(&self);
    fn required_operations2(&self);
}

struct ConcreteStruct1;

impl TemplateMethod for ConcreteStruct1 {
    fn required_operations1(&self) {
        println!("ConcreteStruct1 says: Implemented Operation1")
    }

    fn required_operations2(&self) {
        println!("ConcreteStruct1 says: Implemented Operation2")
    }
}

struct ConcreteStruct2;

impl TemplateMethod for ConcreteStruct2 {
    fn required_operations1(&self) {
        println!("ConcreteStruct2 says: Implemented Operation1")
    }

    fn required_operations2(&self) {
        println!("ConcreteStruct2 says: Implemented Operation2")
    }
}

fn client_code(concrete: impl TemplateMethod) {
    concrete.template_method()
}

fn main() {
    println!("Same client code can work with different concrete implementations:");
    client_code(ConcreteStruct1);
    println!();

    println!("Same client code can work with different concrete implementations:");
    client_code(ConcreteStruct2);
}

```

### Output 输出

````
Same client code can work with different concrete implementations:
TemplateMethod says: I am doing the bulk of the work
ConcreteStruct1 says: Implemented Operation1
TemplateMethod says: But I let subclasses override some operations
ConcreteStruct1 says: Implemented Operation2
TemplateMethod says: But I am doing the bulk of the work anyway

Same client code can work with different concrete implementations:
TemplateMethod says: I am doing the bulk of the work
ConcreteStruct2 says: Implemented Operation1
TemplateMethod says: But I let subclasses override some operations
ConcreteStruct2 says: Implemented Operation2
TemplateMethod says: But I am doing the bulk of the work anyway
````

