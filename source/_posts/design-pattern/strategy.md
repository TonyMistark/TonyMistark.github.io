---
title: Strategy 策略模式
categories:
  - design-pattern
date: 2024-04-21 13:43:14
---

## Intent 意图

策略是一种行为设计模式，可用于定义一系列算法，将每个算法放入单独的类中，并使其对象可互换。


<div align="center"> <img src="/images/strategy-header.png"/><p style="text-align: center;"></p></div>


## Problem 问题

有一天，您决定为休闲旅行者创建一个导航应用程序。该应用程序以一张漂亮的地图为中心，可帮助用户在任何城市快速定位自己。

该应用程序最需要的功能之一是自动路线规划。用户应该能够输入地址，并查看地图上显示的到达该目的地的最快路线。

该应用程序的第一个版本只能在道路上构建路线。开车旅行的人们都欢呼雀跃。但显然，并不是每个人都喜欢在度假时开车。因此，在下一次更新中，您添加了一个构建步行路线的选项。在那之后，你添加了另一个选项，让人们在他们的路线中使用公共交通工具。

然而，这仅仅是个开始。后来，您计划为骑自行车的人添加路线建设。甚至后来，还有另一种选择，可以建造穿越城市所有旅游景点的路线。


<div align="center"> <img src="/images/strategy-problem1.png"/><p style="text-align: center;">导航器的代码变得臃肿。</p></div>


虽然从商业角度来看，该应用程序是成功的，但技术部分却让您头疼不已。每次添加新的路由算法时，导航器的主类大小都会增加一倍。在某些时候，野兽变得太难维护了。

对其中一种算法的任何更改，无论是简单的错误修复还是街道分数的轻微调整，都会影响整个班级，从而增加在已经工作的代码中产生错误的机会。

此外，团队合作变得效率低下。你的队友在成功发布后立即被雇用，他们抱怨他们花了太多时间解决合并冲突。实现一个新功能需要你改变同一个巨大的类，与其他人生成的代码冲突。

## Solution 解决方案

策略模式建议你选择一个以多种不同方式做特定事情的类，并将所有这些算法提取到称为策略的单独类中。

原始类（称为 context）必须具有一个字段，用于存储对其中一个策略的引用。上下文将工作委托给链接的策略对象，而不是单独执行它。

上下文不负责为作业选择适当的算法。相反，客户端将所需的策略传递给上下文。事实上，上下文对策略知之甚少。它通过相同的通用接口与所有策略一起工作，该接口仅公开用于触发封装在所选策略中的算法的单一方法。

这样，上下文就独立于具体策略，因此您可以添加新算法或修改现有算法，而无需更改上下文或其他策略的代码。


<div align="center"> <img src="/images/strategy-problem1.png"/><p style="text-align: center;">路线规划策略。</p></div>


在我们的导航应用程序中，可以使用单个 `buildRoute` 方法将每个路由算法提取到其自己的类中。该方法接受起点和终点，并返回路由检查点的集合。

尽管给定相同的参数，每个路由类可能会构建不同的路由，但主导航器类并不真正关心选择哪种算法，因为它的主要工作是在地图上呈现一组检查点。该类具有切换活动路由策略的方法，因此其客户端（如用户界面中的按钮）可以将当前选定的路由行为替换为另一个路由行为。

## Real-World Analogy 真实世界的类比


<div align="center"> <img src="/images/strategy-comic1.png"/><p style="text-align: center;">前往机场的各种策略。</p></div>


想象一下，你必须去机场。您可以搭乘公共汽车、叫出租车或骑自行车。这些是您的运输策略。您可以根据预算或时间限制等因素选择其中一种策略。

## Structure 结构


<div align="center"> <img src="/images/strategy-structure.png"/><p style="text-align: center;"></p></div>


1. 上下文维护对其中一个具体策略的引用，并仅通过策略接口与此对象进行通信。
2. 策略界面是所有具体策略的通用界面。它声明上下文用于执行策略的方法。
3. 具体策略实现上下文使用的算法的不同变体。
4. 上下文每次需要运行算法时都会在链接的策略对象上调用执行方法。上下文不知道它使用哪种类型的策略，也不知道算法是如何执行的。
5. 客户端创建一个特定的策略对象并将其传递给上下文。上下文公开了一个 setter，它允许客户端在运行时替换与上下文关联的策略。

## Pseudocode 伪代码

在此示例中，上下文使用多种策略来执行各种算术运算。

```java
// The strategy interface declares operations common to all
// supported versions of some algorithm. The context uses this
// interface to call the algorithm defined by the concrete
// strategies.
interface Strategy is
    method execute(a, b)

// Concrete strategies implement the algorithm while following
// the base strategy interface. The interface makes them
// interchangeable in the context.
class ConcreteStrategyAdd implements Strategy is
    method execute(a, b) is
        return a + b

class ConcreteStrategySubtract implements Strategy is
    method execute(a, b) is
        return a - b

class ConcreteStrategyMultiply implements Strategy is
    method execute(a, b) is
        return a * b

// The context defines the interface of interest to clients.
class Context is
    // The context maintains a reference to one of the strategy
    // objects. The context doesn't know the concrete class of a
    // strategy. It should work with all strategies via the
    // strategy interface.
    private strategy: Strategy

    // Usually the context accepts a strategy through the
    // constructor, and also provides a setter so that the
    // strategy can be switched at runtime.
    method setStrategy(Strategy strategy) is
        this.strategy = strategy

    // The context delegates some work to the strategy object
    // instead of implementing multiple versions of the
    // algorithm on its own.
    method executeStrategy(int a, int b) is
        return strategy.execute(a, b)


// The client code picks a concrete strategy and passes it to
// the context. The client should be aware of the differences
// between strategies in order to make the right choice.
class ExampleApplication is
    method main() is
        Create context object.

        Read first number.
        Read last number.
        Read the desired action from user input.

        if (action == addition) then
            context.setStrategy(new ConcreteStrategyAdd())

        if (action == subtraction) then
            context.setStrategy(new ConcreteStrategySubtract())

        if (action == multiplication) then
            context.setStrategy(new ConcreteStrategyMultiply())

        result = context.executeStrategy(First number, Second number)

        Print result.
```

## Applicability 适用性

- 如果要在对象中使用算法的不同变体，并且能够在运行时从一种算法切换到另一种算法，请使用 Strategy 模式。
- 策略模式允许您通过将对象与不同的子对象相关联来间接更改对象在运行时的行为，这些子对象可以以不同的方式执行特定的子任务。
- 当您有许多相似的类时，请使用策略，这些类仅在执行某些行为的方式上有所不同。
- Strategy 模式允许您将不同的行为提取到单独的类层次结构中，并将原始类合并为一个类，从而减少重复代码。
- 使用该模式将类的业务逻辑与算法的实现细节隔离开来，这些算法在该逻辑的上下文中可能不那么重要。
- 策略模式允许您将各种算法的代码、内部数据和依赖项与代码的其余部分隔离开来。各种客户端都获得了一个简单的接口来执行算法并在运行时切换它们。
- 当您的类具有在同一算法的不同变体之间切换的大量条件语句时，请使用该模式。
- Strategy 模式允许您通过将所有算法提取到单独的类中来消除这种条件，所有这些类都实现相同的接口。原始对象将执行委托给其中一个对象，而不是实现算法的所有变体。

## How to Implement 如何实现

1. 在上下文类中，标识容易频繁更改的算法。它也可能是一个大规模条件，在运行时选择并执行同一算法的变体。
2. 声明算法所有变体通用的策略接口。
3. 一个接一个地，将所有算法提取到它们自己的类中。他们都应该实现策略接口。
4. 在上下文类中，添加一个用于存储对策略对象的引用的字段。提供用于替换该字段值的 setter。上下文应仅通过策略接口与策略对象一起使用。上下文可以定义一个接口，让策略访问其数据。
5. 上下文的客户端必须将其与合适的策略相关联，该策略与他们期望上下文执行其主要工作的方式相匹配。

## Pros and Cons 优点和缺点

| 优点√                                              | 缺点×                                                                                                                                                      |
| -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 您可以在运行时交换对象内部使用的算法。             | 如果你只有几种算法，而且它们很少改变，那么就没有真正的理由用模式附带的新类和接口来使程序过于复杂。                                                         |
| 您可以将算法的实现详细信息与使用它的代码隔离开来。 | 客户必须意识到策略之间的差异，以便能够选择合适的策略。                                                                                                     |
| 您可以将继承替换为组合。                           | 许多现代编程语言都支持函数类型，允许您在一组匿名函数中实现不同版本的算法。然后，您可以像使用策略对象一样使用这些函数，但不会因额外的类和接口而使代码膨胀。 |
| 开/闭原则。您可以引入新策略，而无需更改上下文。    |                                                                                                                                                            |

## Relations with Other Patterns 与其他模式的关系

- 桥接、状态、策略（在某种程度上还有适配器）具有非常相似的结构。事实上，所有这些模式都是基于构图的，而构图是将工作委托给其他对象。但是，它们都解决了不同的问题。模式不仅仅是以特定方式构建代码的秘诀。它还可以向其他开发人员传达该模式解决的问题。
- “命令”和“策略”可能看起来很相似，因为您可以使用它们来参数化具有某些操作的对象。但是，他们的意图截然不同。

  - 您可以使用 Command 将任何操作转换为对象。操作的参数将成为该对象的字段。转换允许您延迟操作的执行、排队、存储命令的历史记录、将命令发送到远程服务等。
  - 另一方面，Strategy 通常描述执行相同操作的不同方法，允许您在单个上下文类中交换这些算法。

- Decorator 可让您更改对象的皮肤，而 Strategy 可让您更改内脏。
- 模板方法基于继承：它允许您通过在子类中扩展算法的各个部分来更改这些部分。策略基于组合：您可以通过为对象提供与该行为相对应的不同策略来改变对象的部分行为。Template Method 在类级别工作，因此它是静态的。策略在对象级别工作，允许您在运行时切换行为。
- 状态可以看作是战略的延伸。这两种模式都基于组合：它们通过将一些工作委派给帮助对象来改变上下文的行为。策略使这些对象完全独立，彼此不相知。但是，State 不会限制具体状态之间的依赖关系，而是允许它们随意更改上下文的状态。

## Code Examples 代码示例

# **trategy** in Python

策略是一种行为设计模式，它将一组行为转换为对象，并使它们在原始上下文对象中可互换。

原始对象（称为 context）包含对策略对象的引用。将执行行为的上下文委托给链接的策略对象。为了改变上下文执行其工作的方式，其他对象可能会用另一个对象替换当前链接的策略对象。

使用示例：Strategy 模式在 Python 代码中很常见。它通常用于各种框架中，为用户提供一种在不扩展类的情况下更改类行为的方法。

标识：策略模式可以通过允许嵌套对象执行实际工作的方法以及允许将该对象替换为其他对象的 setter 来识别。

#### main.py：概念示例

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import List


class Context():
    """
    The Context defines the interface of interest to clients.
    """

    def __init__(self, strategy: Strategy) -> None:
        """
        Usually, the Context accepts a strategy through the constructor, but
        also provides a setter to change it at runtime.
        """

        self._strategy = strategy

    @property
    def strategy(self) -> Strategy:
        """
        The Context maintains a reference to one of the Strategy objects. The
        Context does not know the concrete class of a strategy. It should work
        with all strategies via the Strategy interface.
        """

        return self._strategy

    @strategy.setter
    def strategy(self, strategy: Strategy) -> None:
        """
        Usually, the Context allows replacing a Strategy object at runtime.
        """

        self._strategy = strategy

    def do_some_business_logic(self) -> None:
        """
        The Context delegates some work to the Strategy object instead of
        implementing multiple versions of the algorithm on its own.
        """

        # ...

        print("Context: Sorting data using the strategy (not sure how it'll do it)")
        result = self._strategy.do_algorithm(["a", "b", "c", "d", "e"])
        print(",".join(result))

        # ...


class Strategy(ABC):
    """
    The Strategy interface declares operations common to all supported versions
    of some algorithm.

    The Context uses this interface to call the algorithm defined by Concrete
    Strategies.
    """

    @abstractmethod
    def do_algorithm(self, data: List):
        pass


"""
Concrete Strategies implement the algorithm while following the base Strategy
interface. The interface makes them interchangeable in the Context.
"""


class ConcreteStrategyA(Strategy):
    def do_algorithm(self, data: List) -> List:
        return sorted(data)


class ConcreteStrategyB(Strategy):
    def do_algorithm(self, data: List) -> List:
        return reversed(sorted(data))


if __name__ == "__main__":
    # The client code picks a concrete strategy and passes it to the context.
    # The client should be aware of the differences between strategies in order
    # to make the right choice.

    context = Context(ConcreteStrategyA())
    print("Client: Strategy is set to normal sorting.")
    context.do_some_business_logic()
    print()

    print("Client: Strategy is set to reverse sorting.")
    context.strategy = ConcreteStrategyB()
    context.do_some_business_logic()

```

####  **Output.txt:** Execution result

````
Client: Strategy is set to normal sorting.
Context: Sorting data using the strategy (not sure how it'll do it)
a,b,c,d,e

Client: Strategy is set to reverse sorting.
Context: Sorting data using the strategy (not sure how it'll do it)
e,d,c,b,a

````

# **Strategy** in Rust

策略是一种行为设计模式，它将一组行为转换为对象，并使它们在原始上下文对象中可互换。

原始对象（称为 context）包含对策略对象的引用。将执行行为的上下文委托给链接的策略对象。为了改变上下文执行其工作的方式，其他对象可能会用另一个对象替换当前链接的策略对象。

## Conceptual Example 概念示例

通过特征的概念策略示例。

#### **conceptual.rs**

```rust
/// Defines an injectable strategy for building routes.
trait RouteStrategy {
    fn build_route(&self, from: &str, to: &str);
}

struct WalkingStrategy;

impl RouteStrategy for WalkingStrategy {
    fn build_route(&self, from: &str, to: &str) {
        println!("Walking route from {} to {}: 4 km, 30 min", from, to);
    }
}

struct PublicTransportStrategy;

impl RouteStrategy for PublicTransportStrategy {
    fn build_route(&self, from: &str, to: &str) {
        println!(
            "Public transport route from {} to {}: 3 km, 5 min",
            from, to
        );
    }
}

struct Navigator<T: RouteStrategy> {
    route_strategy: T,
}

impl<T: RouteStrategy> Navigator<T> {
    pub fn new(route_strategy: T) -> Self {
        Self { route_strategy }
    }

    pub fn route(&self, from: &str, to: &str) {
        self.route_strategy.build_route(from, to);
    }
}

fn main() {
    let navigator = Navigator::new(WalkingStrategy);
    navigator.route("Home", "Club");
    navigator.route("Club", "Work");

    let navigator = Navigator::new(PublicTransportStrategy);
    navigator.route("Home", "Club");
    navigator.route("Club", "Work");
}

```

### Output 输出

````
Walking route from Home to Club: 4 km, 30 min
Walking route from Club to Work: 4 km, 30 min
Public transport route from Home to Club: 3 km, 5 min
Public transport route from Club to Work: 3 km, 5 min
````

## Functional approach 功能方法

函数和闭包简化了策略实现，因为您可以将行为直接注入到对象中，而无需复杂的接口定义。

似乎 Strategy 在 Rust 的现代开发中经常被隐式和广泛地使用，例如，它就像迭代器一样工作：

```rust
let a = [0i32, 1, 2];

let mut iter = a.iter().filter(|x| x.is_positive());
```

#### **functional.rs**

```rust
type RouteStrategy = fn(from: &str, to: &str);

fn walking_strategy(from: &str, to: &str) {
    println!("Walking route from {} to {}: 4 km, 30 min", from, to);
}

fn public_transport_strategy(from: &str, to: &str) {
    println!(
        "Public transport route from {} to {}: 3 km, 5 min",
        from, to
    );
}

struct Navigator {
    route_strategy: RouteStrategy,
}

impl Navigator {
    pub fn new(route_strategy: RouteStrategy) -> Self {
        Self { route_strategy }
    }

    pub fn route(&self, from: &str, to: &str) {
        (self.route_strategy)(from, to);
    }
}

fn main() {
    let navigator = Navigator::new(walking_strategy);
    navigator.route("Home", "Club");
    navigator.route("Club", "Work");

    let navigator = Navigator::new(public_transport_strategy);
    navigator.route("Home", "Club");
    navigator.route("Club", "Work");

    let navigator = Navigator::new(|from, to| println!("Specific route from {} to {}", from, to));
    navigator.route("Home", "Club");
    navigator.route("Club", "Work");
}

```

### Output 输出

````
Walking route from Home to Club: 4 km, 30 min
Walking route from Club to Work: 4 km, 30 min
Public transport route from Home to Club: 3 km, 5 min
Public transport route from Club to Work: 3 km, 5 min
Specific route from Home to Club
Specific route from Club to Work
````