---
title: Visitor 访问者模式
date: 2024-04-21 14:38:18
---

## Intent 意图

Visitor 是一种行为设计模式，可用于将算法与操作算法的对象分开。


<div align="center"> <img src="/images/visitor-header.png"/><p style="text-align: center;"></p></div>


## Problem 问题

想象一下，您的团队开发了一个应用程序，该应用程序使用结构为一个巨大的图形的地理信息。图形的每个节点可能代表一个复杂的实体，例如城市，但也代表更精细的事物，例如工业、观光区等。如果节点所代表的真实对象之间有一条道路，则节点与其他节点相连。在后台，每个节点类型都由自己的类表示，而每个特定节点都是一个对象。


<div align="center"> <img src="/images/visitor-problem1.png"/><p style="text-align: center;">将图形导出为 XML。</p></div>


在某个时候，您接到一项任务，要实现将图形导出为 XML 格式。起初，这项工作似乎很简单。您计划向每个节点类添加一个导出方法，然后利用递归遍历图形的每个节点，从而执行导出方法。解决方案简单而优雅：多亏了多态性，您没有将调用导出方法的代码耦合到具体的节点类。

不幸的是，系统架构师拒绝允许您更改现有的节点类。他说代码已经在生产中，他不想因为更改中的潜在错误而冒着破坏它的风险。


<div align="center"> <img src="/images/visitor-problem1.png"/><p style="text-align: center;">必须将 XML 导出方法添加到所有节点类中，如果任何 bug 随更改而漏掉，则存在破坏整个应用程序的风险。</p></div>


此外，他质疑在节点类中包含XML导出代码是否有意义。这些类的主要工作是处理地理数据。XML 导出行为在那里看起来很陌生。

拒绝还有另一个原因。很有可能在实现此功能后，营销部门的某个人会要求您提供导出为不同格式的功能，或者请求其他一些奇怪的东西。这将迫使你再次改变那些珍贵而脆弱的职业。

## Solution 解决方案

Visitor 模式建议您将新行为放入一个名为 visitor 的单独类中，而不是尝试将其集成到现有类中。现在，必须执行该行为的原始对象将作为参数传递给访问者的方法之一，从而为该方法提供对对象中包含的所有必要数据的访问。

现在，如果该行为可以对不同类的对象执行呢？例如，在我们的 XML 导出示例中，实际实现在各种节点类之间可能会略有不同。因此，访问者类可以定义的不是一个方法，而是一组方法，每个方法都可以采用不同类型的参数，如下所示：

```java
class ExportVisitor implements Visitor is
    method doForCity(City c) { ... }
    method doForIndustry(Industry f) { ... }
    method doForSightSeeing(SightSeeing ss) { ... }
    // ...
```

但是，我们究竟该如何称呼这些方法，尤其是在处理整个图形时呢？这些方法具有不同的特征，因此我们不能使用多态性。要选择能够处理给定对象的适当访问者方法，我们需要检查其类。这听起来不像一场噩梦吗？

```java
foreach (Node node in graph)
    if (node instanceof City)
        exportVisitor.doForCity((City) node)
    if (node instanceof Industry)
        exportVisitor.doForIndustry((Industry) node)
    // ...
}
```

你可能会问，我们为什么不使用方法重载呢？这时，您为所有方法指定相同的名称，即使它们支持不同的参数集。不幸的是，即使假设我们的编程语言完全支持它（就像 Java 和 C# 一样），它也不会帮助我们。由于节点对象的确切类事先未知，因此重载机制将无法确定要执行的正确方法。默认为采用基 `Node` 类对象的方法。

但是，Visitor 模式解决了这个问题。它使用一种称为 Double Dispatch 的技术，它有助于在没有繁琐条件的情况下对对象执行正确的方法。与其让客户端选择要调用的方法的正确版本，不如将此选择委托给作为参数传递给访问者的对象？由于对象知道自己的类，因此它们将能够不那么笨拙地对访问者选择适当的方法。他们“接受”访客并告诉它应该执行哪种访问方法。

```java
// Client code
foreach (Node node in graph)
    node.accept(exportVisitor)

// City
class City is
    method accept(Visitor v) is
        v.doForCity(this)
    // ...

// Industry
class Industry is
    method accept(Visitor v) is
        v.doForIndustry(this)
    // ...
```

我承认。毕竟，我们必须更改节点类。但至少这个变化是微不足道的，它允许我们在不再次更改代码的情况下添加进一步的行为。

现在，如果我们为所有访问者提取一个通用界面，则所有现有节点都可以与您引入应用程序的任何访问者一起工作。如果你发现自己引入了一个与节点相关的新行为，你所要做的就是实现一个新的访客类。

## Real-World Analogy 真实世界的类比


<div align="center"> <img src="/images/visitor-comic1.png"/><p style="text-align: center;">一个好的保险代理人随时准备为各种类型的组织提供不同的政策。</p></div>


想象一下，一位经验丰富的保险代理人渴望获得新客户。他可以走访附近的每栋建筑，试图向他遇到的每个人推销保险。根据占用建筑物的组织类型，他可以提供专门的保险单：

- 如果是住宅楼，他就卖医疗保险。
- 如果是银行，他卖盗窃保险。
- 如果是咖啡店，他卖火灾和洪水保险。

## Structure 结构


<div align="center"> <img src="/images/visitor-structure.png"/><p style="text-align: center;"></p></div>


1. Visitor 接口声明了一组访问方法，这些方法可以将对象结构的具体元素作为参数。如果程序是用支持重载的语言编写的，则这些方法可能具有相同的名称，但其参数的类型必须不同。
2. 每个 Concrete Visitor 都实现了相同行为的多个版本，这些版本针对不同的 Concrete 元素类进行了定制。
3. Element 接口声明了一种“接受”访问者的方法。此方法应使用访问器接口的类型声明一个参数。
4. 每个混凝土元素都必须实现验收方法。此方法的目的是将调用重定向到与当前元素类对应的正确访问者方法。请注意，即使基元素类实现了此方法，所有子类仍必须在其自己的类中重写此方法，并在访问器对象上调用相应的方法。
5. 客户端通常表示集合或其他一些复杂对象（例如，复合树）。通常，客户端并不知道所有具体元素类，因为它们通过一些抽象接口处理该集合中的对象。

## Pseudocode 伪代码

在此示例中，Visitor 模式向几何形状的类层次结构添加了 XML 导出支持。


<div align="center"> <img src="/images/visitor-example1.png"/><p style="text-align: center;">通过访问者对象将各种类型的对象导出为 XML 格式。</p></div>


```java
// The element interface declares an `accept` method that takes
// the base visitor interface as an argument.
interface Shape is
    method move(x, y)
    method draw()
    method accept(v: Visitor)

// Each concrete element class must implement the `accept`
// method in such a way that it calls the visitor's method that
// corresponds to the element's class.
class Dot implements Shape is
    // ...

    // Note that we're calling `visitDot`, which matches the
    // current class name. This way we let the visitor know the
    // class of the element it works with.
    method accept(v: Visitor) is
        v.visitDot(this)

class Circle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCircle(this)

class Rectangle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitRectangle(this)

class CompoundShape implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCompoundShape(this)


// The Visitor interface declares a set of visiting methods that
// correspond to element classes. The signature of a visiting
// method lets the visitor identify the exact class of the
// element that it's dealing with.
interface Visitor is
    method visitDot(d: Dot)
    method visitCircle(c: Circle)
    method visitRectangle(r: Rectangle)
    method visitCompoundShape(cs: CompoundShape)

// Concrete visitors implement several versions of the same
// algorithm, which can work with all concrete element classes.
//
// You can experience the biggest benefit of the Visitor pattern
// when using it with a complex object structure such as a
// Composite tree. In this case, it might be helpful to store
// some intermediate state of the algorithm while executing the
// visitor's methods over various objects of the structure.
class XMLExportVisitor implements Visitor is
    method visitDot(d: Dot) is
        // Export the dot's ID and center coordinates.

    method visitCircle(c: Circle) is
        // Export the circle's ID, center coordinates and
        // radius.

    method visitRectangle(r: Rectangle) is
        // Export the rectangle's ID, left-top coordinates,
        // width and height.

    method visitCompoundShape(cs: CompoundShape) is
        // Export the shape's ID as well as the list of its
        // children's IDs.


// The client code can run visitor operations over any set of
// elements without figuring out their concrete classes. The
// accept operation directs a call to the appropriate operation
// in the visitor object.
class Application is
    field allShapes: array of Shapes

    method export() is
        exportVisitor = new XMLExportVisitor()

        foreach (shape in allShapes) do
            shape.accept(exportVisitor)
```

如果您想知道为什么我们需要此示例中的方法 如果您想知道为什么我们需要此示例中的方法 `accept` ，我的文章 Visitor and Double Dispatch 详细解决了这个问题。 ，我的文章 Visitor and Double Dispatch 详细解决了这个问题。

## Applicability 适用性

- 当需要对复杂对象结构(例如，对象树)的所有元素执行操作时，可以使用Visitor。
- Visitor 模式允许您通过让 Visitor 对象实现同一操作的多个变体（对应于所有目标类）对一组具有不同类的对象执行操作。
- 使用 Visitor 清理辅助行为的业务逻辑。
- 该模式允许您通过将所有其他行为提取到一组访客类中，使应用的主要类更专注于其主要工作。
- 当行为仅在类层次结构的某些类中有意义，而在其他类中没有意义时，请使用该模式。
- 您可以将此行为提取到单独的访问者类中，并仅实现那些接受相关类对象的访问方法，其余部分为空。

## How to Implement 如何实现

1. 使用一组“访问”方法声明访问者接口，每个程序中存在的每个具体元素类一个。
2. 声明元素接口。如果要使用现有的元素类层次结构，请将抽象的“acceptance”方法添加到层次结构的基类中。此方法应接受访问者对象作为参数。
3. 在所有具体图元类中实现验收方法。这些方法必须简单地将调用重定向到与当前元素的类匹配的传入访问对象上的访问方法。
4. 元素类应仅通过访客界面与访客一起使用。但是，访问者必须了解所有具体元素类，这些类被引用为访问方法的参数类型。
5. 对于无法在元素层次结构中实现的每个行为，请创建一个新的具体访问者类并实现所有访问方法。

   您可能会遇到这样的情况：访问者需要访问元素类的某些专用成员。在这种情况下，您可以公开这些字段或方法，从而违反元素的封装，或者将访问者类嵌套在元素类中。只有当你有幸使用支持嵌套类的编程语言时，后者才有可能。

6. 客户端必须创建访问者对象，并通过“接受”方法将它们传递到元素中。

## Pros and Cons 优点和缺点

| 优点√                                                                                                                                        | 缺点×                                                                          |
| -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| 开/闭原则。您可以引入一种新行为，该行为可以处理不同类的对象，而无需更改这些类。                                                              | 每次将类添加到元素层次结构中或从元素层次结构中删除类时，都需要更新所有访问者。 |
| 单一责任原则。您可以将同一行为的多个版本移动到同一类中。                                                                                     | 访问者可能缺乏对他们应该使用的元素的私有字段和方法的必要访问权限。             |
| 访客对象在处理各种对象时可以积累一些有用的信息。当您想要遍历一些复杂的对象结构（如对象树）并将访问者应用于此结构的每个对象时，这可能很方便。 |                                                                                |

## Relations with Other Patterns 与其他模式的关系

- 您可以将 Visitor 视为 Command 模式的强大版本。它的对象可以对不同类的各种对象执行操作。
- 您可以使用 Visitor 对整个复合树执行操作。
- 您可以将 Visitor 与 Iterator 一起使用来遍历复杂的数据结构，并对其元素执行一些操作，即使它们都具有不同的类。

## Code Examples 代码示例

# **Visitor** in Python

Visitor 是一种行为设计模式，允许在不更改任何现有代码的情况下向现有类层次结构添加新行为。

#### main.py：概念示例

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import List


class Component(ABC):
    """
    The Component interface declares an `accept` method that should take the
    base visitor interface as an argument.
    """

    @abstractmethod
    def accept(self, visitor: Visitor) -> None:
        pass


class ConcreteComponentA(Component):
    """
    Each Concrete Component must implement the `accept` method in such a way
    that it calls the visitor's method corresponding to the component's class.
    """

    def accept(self, visitor: Visitor) -> None:
        """
        Note that we're calling `visitConcreteComponentA`, which matches the
        current class name. This way we let the visitor know the class of the
        component it works with.
        """

        visitor.visit_concrete_component_a(self)

    def exclusive_method_of_concrete_component_a(self) -> str:
        """
        Concrete Components may have special methods that don't exist in their
        base class or interface. The Visitor is still able to use these methods
        since it's aware of the component's concrete class.
        """

        return "A"


class ConcreteComponentB(Component):
    """
    Same here: visitConcreteComponentB => ConcreteComponentB
    """

    def accept(self, visitor: Visitor):
        visitor.visit_concrete_component_b(self)

    def special_method_of_concrete_component_b(self) -> str:
        return "B"


class Visitor(ABC):
    """
    The Visitor Interface declares a set of visiting methods that correspond to
    component classes. The signature of a visiting method allows the visitor to
    identify the exact class of the component that it's dealing with.
    """

    @abstractmethod
    def visit_concrete_component_a(self, element: ConcreteComponentA) -> None:
        pass

    @abstractmethod
    def visit_concrete_component_b(self, element: ConcreteComponentB) -> None:
        pass


"""
Concrete Visitors implement several versions of the same algorithm, which can
work with all concrete component classes.

You can experience the biggest benefit of the Visitor pattern when using it with
a complex object structure, such as a Composite tree. In this case, it might be
helpful to store some intermediate state of the algorithm while executing
visitor's methods over various objects of the structure.
"""


class ConcreteVisitor1(Visitor):
    def visit_concrete_component_a(self, element) -> None:
        print(f"{element.exclusive_method_of_concrete_component_a()} + ConcreteVisitor1")

    def visit_concrete_component_b(self, element) -> None:
        print(f"{element.special_method_of_concrete_component_b()} + ConcreteVisitor1")


class ConcreteVisitor2(Visitor):
    def visit_concrete_component_a(self, element) -> None:
        print(f"{element.exclusive_method_of_concrete_component_a()} + ConcreteVisitor2")

    def visit_concrete_component_b(self, element) -> None:
        print(f"{element.special_method_of_concrete_component_b()} + ConcreteVisitor2")


def client_code(components: List[Component], visitor: Visitor) -> None:
    """
    The client code can run visitor operations over any set of elements without
    figuring out their concrete classes. The accept operation directs a call to
    the appropriate operation in the visitor object.
    """

    # ...
    for component in components:
        component.accept(visitor)
    # ...


if __name__ == "__main__":
    components = [ConcreteComponentA(), ConcreteComponentB()]

    print("The client code works with all visitors via the base Visitor interface:")
    visitor1 = ConcreteVisitor1()
    client_code(components, visitor1)

    print("It allows the same client code to work with different types of visitors:")
    visitor2 = ConcreteVisitor2()
    client_code(components, visitor2)

```

#### 输出.txt：执行结果

````
The client code works with all visitors via the base Visitor interface:
A + ConcreteVisitor1
B + ConcreteVisitor1
It allows the same client code to work with different types of visitors:
A + ConcreteVisitor2
B + ConcreteVisitor2
````

# **Visitor** in Rust

Visitor 是一种行为设计模式，允许在不更改任何现有代码的情况下向现有类层次结构添加新行为。

## Deserialization 反序列化

1. Visitor 模式的一个真实示例是 serde 序列化框架及其反序列化模型（请参阅 Serde 数据模型）。
2. `Visitor` 传递给 a  传递给 a  传递给 a `Deserializer` （根据 Visitor 模式的“元素”），它接受并驱动 以  （根据 Visitor 模式的“元素”），它接受并驱动 以  （根据 Visitor 模式的“元素”），它接受并驱动 以 `Visitor` 构造所需的类型。 构造所需的类型。

让我们在示例中重现此反序列化模型。

#### **visitor.rs**

```rust
use crate::{TwoValuesArray, TwoValuesStruct};

/// Visitor can visit one type, do conversions, and output another type.
///
/// It's not like all visitors must return a new type, it's just an example
/// that demonstrates the technique.
pub trait Visitor {
    type Value;

    /// Visits a vector of integers and outputs a desired type.
    fn visit_vec(&self, v: Vec<i32>) -> Self::Value;
}

/// Visitor implementation for a struct of two values.
impl Visitor for TwoValuesStruct {
    type Value = TwoValuesStruct;

    fn visit_vec(&self, v: Vec<i32>) -> Self::Value {
        TwoValuesStruct { a: v[0], b: v[1] }
    }
}

/// Visitor implementation for a struct of values array.
impl Visitor for TwoValuesArray {
    type Value = TwoValuesArray;

    fn visit_vec(&self, v: Vec<i32>) -> Self::Value {
        let mut ab = [0i32; 2];

        ab[0] = v[0];
        ab[1] = v[1];

        TwoValuesArray { ab }
    }
}

```

####  **main.rs**

```rust
#![allow(unused)]

mod visitor;

use visitor::Visitor;

/// A struct of two integer values.
///
/// It's going to be an output of `Visitor` trait which is defined for the type
/// in `visitor.rs`.
#[derive(Default, Debug)]
pub struct TwoValuesStruct {
    a: i32,
    b: i32,
}

/// A struct of values array.
///
/// It's going to be an output of `Visitor` trait which is defined for the type
/// in `visitor.rs`.
#[derive(Default, Debug)]
pub struct TwoValuesArray {
    ab: [i32; 2],
}

/// `Deserializer` trait defines methods that can parse either a string or
/// a vector, it accepts a visitor which knows how to construct a new object
/// of a desired type (in our case, `TwoValuesArray` and `TwoValuesStruct`).
trait Deserializer<V: Visitor> {
    fn create(visitor: V) -> Self;
    fn parse_str(&self, input: &str) -> Result<V::Value, &'static str> {
        Err("parse_str is unimplemented")
    }
    fn parse_vec(&self, input: Vec<i32>) -> Result<V::Value, &'static str> {
        Err("parse_vec is unimplemented")
    }
}

struct StringDeserializer<V: Visitor> {
    visitor: V,
}

impl<V: Visitor> Deserializer<V> for StringDeserializer<V> {
    fn create(visitor: V) -> Self {
        Self { visitor }
    }

    fn parse_str(&self, input: &str) -> Result<V::Value, &'static str> {
        // In this case, in order to apply a visitor, a deserializer should do
        // some preparation. The visitor does its stuff, but it doesn't do everything.
        let input_vec = input
            .split_ascii_whitespace()
            .map(|x| x.parse().unwrap())
            .collect();

        Ok(self.visitor.visit_vec(input_vec))
    }
}

struct VecDeserializer<V: Visitor> {
    visitor: V,
}

impl<V: Visitor> Deserializer<V> for VecDeserializer<V> {
    fn create(visitor: V) -> Self {
        Self { visitor }
    }

    fn parse_vec(&self, input: Vec<i32>) -> Result<V::Value, &'static str> {
        Ok(self.visitor.visit_vec(input))
    }
}

fn main() {
    let deserializer = StringDeserializer::create(TwoValuesStruct::default());
    let result = deserializer.parse_str("123 456");
    println!("{:?}", result);

    let deserializer = VecDeserializer::create(TwoValuesStruct::default());
    let result = deserializer.parse_vec(vec![123, 456]);
    println!("{:?}", result);

    let deserializer = VecDeserializer::create(TwoValuesArray::default());
    let result = deserializer.parse_vec(vec![123, 456]);
    println!("{:?}", result);

    println!(
        "Error: {}",
        deserializer.parse_str("123 456").err().unwrap()
    )
}

```

### Output 输出

````
Ok(TwoValuesStruct { a: 123, b: 456 })
Ok(TwoValuesStruct { a: 123, b: 456 })
Ok(TwoValuesArray { ab: [123, 456] })
Error: parse_str unimplemented
````