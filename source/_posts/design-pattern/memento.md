---
title: Memento 备忘录模式
categories:
  - design-pattern
date: 2024-04-21 09:59:34
---

## Intent 意图

Memento 是一种行为设计模式，可让您保存和恢复对象的先前状态，而无需透露其实现的细节。


<div align="center"> <img src="/images/memento-header.png"/></div>


## Problem 问题

假设您正在创建一个文本编辑器应用程序。除了简单的文本编辑外，您的编辑器还可以设置文本格式、插入内嵌图像等。

在某个时候，您决定让用户撤消对文本执行的任何操作。多年来，此功能已变得如此普遍，以至于如今人们希望每个应用程序都拥有它。对于实现，您选择采用直接方法。在执行任何操作之前，应用程序会记录所有对象的状态并将其保存在某个存储中。稍后，当用户决定还原操作时，应用会从历史记录中获取最新快照，并使用它来还原所有对象的状态。


<div align="center"> <img src="/images/memento-problem1.png"/>在执行操作之前，应用会保存对象状态的快照，稍后可用于将对象还原到以前的状态。</div>


让我们考虑一下这些状态快照。你究竟要如何生产一个？您可能需要遍历对象中的所有字段，并将其值复制到存储中。但是，这只有在对象对其内容的访问限制相当宽松时才有效。不幸的是，大多数真实对象不会让其他人轻易窥视它们，将所有重要数据隐藏在私人字段中。

暂时忽略这个问题，让我们假设我们的对象表现得像嬉皮士：更喜欢开放关系并保持其状态公开。虽然这种方法可以解决眼前的问题，并允许您随意生成对象状态的快照，但它仍然存在一些严重的问题。将来，您可能会决定重构某些编辑器类，或者添加或删除某些字段。听起来很简单，但这还需要更改负责复制受影响对象状态的类。


<div align="center"> <img src="/images/memento-problem1.png"/>如何复制对象的私有状态？</div>


但还有更多。让我们考虑一下编辑器状态的实际“快照”。它包含哪些数据？它至少必须包含实际文本、光标坐标、当前滚动位置等。要制作快照，您需要收集这些值并将它们放入某种容器中。

最有可能的是，您将在表示历史记录的某个列表中存储大量这些容器对象。因此，容器最终可能会成为同一类的对象。该类几乎没有方法，但有很多反映编辑器状态的字段。若要允许其他对象在快照中写入和读取数据，可能需要将其字段设为公共字段。这将暴露编辑器的所有状态，无论是否私密。其他类将依赖于对快照类的每一个微小更改，否则这将发生在私有字段和方法中，而不会影响外部类。

看起来我们已经走到了死胡同：你要么暴露类的所有内部细节，使它们过于脆弱，要么限制对它们的状态的访问，使其无法生成快照。有没有其他方法可以实现“撤消”？

## Solution 解决方案

我们刚刚遇到的所有问题都是由封装损坏引起的。有些对象试图做比它们应该做的更多的事情。为了收集执行某些操作所需的数据，它们会侵入其他对象的私有空间，而不是让这些对象执行实际操作。

Memento 模式将创建状态快照的任务委托给该状态的实际所有者，即发起方对象。因此，编辑器类本身可以制作快照，而不是其他对象试图从“外部”复制编辑器的状态，因为它对自己的状态具有完全访问权限。

该模式建议将对象状态的副本存储在称为 memento 的特殊对象中。纪念品的内容除了制作纪念品的对象外，任何其他对象都无法访问。其他对象必须使用有限的接口与纪念品进行通信，该接口可能允许获取快照的元数据（创建时间、所执行操作的名称等），但不允许获取快照中包含的原始对象的状态。


<div align="center"> <img src="/images/memento-solution.png"/>创建者可以完全访问纪念品，而管理员只能访问元数据。</div>


这种限制性政策允许您将纪念品存放在其他物体中，通常称为看守人。由于管理员只能通过有限的界面处理纪念品，因此它无法篡改存储在纪念品中的状态。同时，发起者可以访问纪念品内的所有字段，允许它随意恢复之前的状态。

在我们的文本编辑器示例中，我们可以创建一个单独的历史类来充当看守人。每次编辑器将要执行操作时，存储在管理员内部的一堆纪念品都会增加。您甚至可以在应用程序的 UI 中呈现此堆栈，向用户显示以前执行的操作的历史记录。

当用户触发撤消时，历史记录会从堆栈中获取最新的纪念品，并将其传递回编辑器，请求回滚。由于编辑器可以完全访问纪念品，因此它会使用从纪念品中获取的值来更改自己的状态。

## Structure 结构

#### Implementation based on nested classes  
基于嵌套类的实现

该模式的经典实现依赖于对嵌套类的支持，嵌套类在许多流行的编程语言（如 C++、C# 和 Java）中都可用。


<div align="center"> <img src="/images/memento-structure.png"/></div>


1. Originator 类可以生成其自身状态的快照，并在需要时从快照还原其状态。
2. Memento 是一个值对象，充当发起方状态的快照。通常的做法是使 memento 不可变，并且仅通过构造函数向其传递一次数据。
3. 看守人不仅知道“何时”和“为什么”捕获发起人的状态，而且知道何时应该恢复状态。

   看守人可以通过存储一堆纪念品来跟踪发起人的历史。当始作俑者必须回到历史中时，看守人会从堆栈中取出最上面的纪念品，并将其传递给始作俑者的修复方法。

4. 在此实现中，memento 类嵌套在发起方中。这允许发起方访问 memento 的字段和方法，即使它们被声明为私有。另一方面，看守人对纪念品的领域和方法的访问非常有限，这使得它可以将纪念品存储在堆栈中，但不能篡改它们的状态。

#### Implementation based on an intermediate interface  
基于中间接口的实现

还有一种替代实现，适用于不支持嵌套类的编程语言(是的，PHP，我说的就是你)。


<div align="center"> <img src="/images/memento-structure2.png"/></div>


1. 在没有嵌套类的情况下，您可以通过建立一种约定来限制对 memento 字段的访问，该约定使管理员只能通过显式声明的中间接口来处理 memento，该接口将仅声明与 memento 的元数据相关的方法。
2. 另一方面，发起人可以直接使用 memento 对象，访问 memento 类中声明的字段和方法。这种方法的缺点是您需要公开声明纪念品的所有成员。

#### Implementation with even stricter encapsulation 实现更严格的封装

当您不想让其他类通过纪念品访问发起者的状态时，还有另一种实现非常有用。


<div align="center"> <img src="/images/memento-structure3.png"/></div>


1. 此实现允许具有多种类型的发起人和纪念品。每个发起人都使用相应的纪念品类。无论是发起人还是纪念品都不会向任何人暴露他们的状态。
2. 现在，看护人被明确限制更改存储在纪念品中的状态。此外，看守人类独立于发起人，因为恢复方法现在在纪念品类中定义。
3. 每个纪念品都与制作它的发起人联系在一起。发起者将自己连同其状态的值一起传递给纪念品的构造者。由于这些类之间的密切关系，纪念品可以恢复其创建者的状态，因为后者已经定义了适当的设置者。

## Pseudocode 伪代码

此示例将 Memento 模式与 Command 模式一起使用，用于存储复杂文本编辑器状态的快照，并在需要时从这些快照恢复早期状态。memento-example1


<div align="center"> <img src="/images/memento-example1.png"/>保存文本编辑器状态的快照。</div>


命令对象充当看守人。它们在执行与命令相关的操作之前获取编辑器的纪念品。当用户尝试撤消最近的命令时，编辑器可以使用该命令中存储的 memento 将自身恢复到以前的状态。

memento 类不声明任何公共字段、getter 或 setter。因此，任何对象都不能更改其内容。纪念品链接到创建它们的编辑器对象。这允许 memento 通过编辑器对象上的 setter 传递数据来恢复链接编辑器的状态。由于纪念品链接到特定的编辑器对象，因此您可以使用集中式撤消堆栈使应用支持多个独立的编辑器窗口。

```java
// The originator holds some important data that may change over
// time. It also defines a method for saving its state inside a
// memento and another method for restoring the state from it.
class Editor is
    private field text, curX, curY, selectionWidth

    method setText(text) is
        this.text = text

    method setCursor(x, y) is
        this.curX = x
        this.curY = y

    method setSelectionWidth(width) is
        this.selectionWidth = width

    // Saves the current state inside a memento.
    method createSnapshot():Snapshot is
        // Memento is an immutable object; that's why the
        // originator passes its state to the memento's
        // constructor parameters.
        return new Snapshot(this, text, curX, curY, selectionWidth)

// The memento class stores the past state of the editor.
class Snapshot is
    private field editor: Editor
    private field text, curX, curY, selectionWidth

    constructor Snapshot(editor, text, curX, curY, selectionWidth) is
        this.editor = editor
        this.text = text
        this.curX = x
        this.curY = y
        this.selectionWidth = selectionWidth

    // At some point, a previous state of the editor can be
    // restored using a memento object.
    method restore() is
        editor.setText(text)
        editor.setCursor(curX, curY)
        editor.setSelectionWidth(selectionWidth)

// A command object can act as a caretaker. In that case, the
// command gets a memento just before it changes the
// originator's state. When undo is requested, it restores the
// originator's state from a memento.
class Command is
    private field backup: Snapshot

    method makeBackup() is
        backup = editor.createSnapshot()

    method undo() is
        if (backup != null)
            backup.restore()
    // ...
```

## Applicability 适用性

- 当您希望生成对象状态的快照以便能够恢复对象的先前状态时，请使用Memento模式。
- Memento 模式允许您创建对象状态的完整副本（包括私有字段），并将它们与对象分开存储。虽然由于“撤消”用例，大多数人都记得这种模式，但在处理事务时（即，如果您需要在错误时回滚操作）时，它也是必不可少的。
- 当直接访问对象的字段/getter/setter 违反其封装时，请使用该模式。
- Memento 使对象本身负责创建其状态的快照。没有其他对象可以读取快照，从而确保原始对象的状态数据安全可靠。

## How to Implement 如何实现

1. 确定哪个类将扮演发起人的角色。重要的是要知道程序是使用此类型的一个中心对象还是多个较小的对象。
2. 创建 memento 类。逐个声明一组字段，这些字段镜像在发起方类中声明的字段。
3. 使 memento 类不可变。纪念品应该只通过构造函数接受一次数据。该类不应有二传手。
4. 如果您的编程语言支持嵌套类，请将 memento 嵌套在发起方中。如果没有，请从 memento 类中提取一个空白接口，并使所有其他对象使用它来引用 memento。您可以向接口添加一些元数据操作，但不会公开发起方的状态。
5. 将用于生成纪念品的方法添加到 originator 类。发起人应通过纪念品构造函数的一个或多个参数将其状态传递给纪念品。

   该方法的返回类型应该是您在前一步中提取的接口(假设您提取了它)。在底层，纪念品生成方法应该直接与纪念品类一起工作。

6. 添加一个将发起人状态恢复到其类的方法。它应该接受纪念品对象作为参数。如果在前面的步骤中提取了一个接口，请将其作为参数的类型。在这种情况下，您需要将传入对象类型转换为memorento类，因为发起者需要对该对象进行完全访问。
7. 看守人，无论它代表的是命令对象、历史记录还是完全不同的东西，都应该知道何时向发起人请求新的纪念品，如何存储它们，以及何时使用特定的纪念品恢复发起人。
8. 看护人和发起人之间的联系可以移到纪念品类中。在这种情况下，每个纪念品都必须连接到创建它的发起人。恢复方法也将转移到纪念品类。但是，只有当 memento 类嵌套在 originator 中，或者 originator 类提供足够的 setter 来覆盖其状态时，这才有意义。

## Pros and Cons 优点和缺点

| 优点√                                                          | 缺点×                                                                                   |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| 您可以在不违反其封装的情况下生成对象状态的快照。               | 如果客户端过于频繁地创建纪念品，则该应用程序可能会消耗大量 RAM。                        |
| 您可以通过让管理员维护发起方状态的历史记录来简化发起方的代码。 | 看护人应跟踪发起人的生命周期，以便能够销毁过时的纪念品。                                |
|                                                                | 大多数动态编程语言，如 PHP、Python 和 JavaScript，都不能保证 memento 中的状态保持不变。 |

## Relations with Other Patterns 与其他模式的关系

- 在实现“撤消”时，您可以同时使用 Command 和 Memento。在这种情况下，命令负责对目标对象执行各种操作，而纪念品则在执行命令之前保存该对象的状态。
- 您可以将 Memento 与 Iterator 一起使用来捕获当前迭代状态，并在必要时回滚。
- 有时，Prototype 可以成为 Memento 的更简单替代品。如果要存储在历史记录中的对象的状态相当简单，并且没有指向外部资源的链接，或者链接易于重新建立，则此方法有效。

## Code Examples 代码示例

# **Memento** in Python

Memento 是一种行为设计模式，允许制作对象状态的快照并在将来恢复它。

Memento 不会损害它所处理的对象的内部结构，以及保存在快照中的数据。

使用示例：Memento的原理可以通过序列化来实现，这在Python中很常见。虽然它不是制作对象状态快照的唯一和最有效的方法，但它仍然允许存储状态备份，同时保护发起方的结构免受其他对象的影响。

## Conceptual Example 概念示例

此示例说明了 Memento 设计模式的结构。它侧重于回答以下问题：

- 它由哪些类组成？
- 这些课程扮演什么角色？
- 模式的元素以何种方式相关？

#### main.py：概念示例

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from datetime import datetime
from random import sample
from string import ascii_letters


class Originator:
    """
    The Originator holds some important state that may change over time. It also
    defines a method for saving the state inside a memento and another method
    for restoring the state from it.
    """

    _state = None
    """
    For the sake of simplicity, the originator's state is stored inside a single
    variable.
    """

    def __init__(self, state: str) -> None:
        self._state = state
        print(f"Originator: My initial state is: {self._state}")

    def do_something(self) -> None:
        """
        The Originator's business logic may affect its internal state.
        Therefore, the client should backup the state before launching methods
        of the business logic via the save() method.
        """

        print("Originator: I'm doing something important.")
        self._state = self._generate_random_string(30)
        print(f"Originator: and my state has changed to: {self._state}")

    @staticmethod
    def _generate_random_string(length: int = 10) -> str:
        return "".join(sample(ascii_letters, length))

    def save(self) -> Memento:
        """
        Saves the current state inside a memento.
        """

        return ConcreteMemento(self._state)

    def restore(self, memento: Memento) -> None:
        """
        Restores the Originator's state from a memento object.
        """

        self._state = memento.get_state()
        print(f"Originator: My state has changed to: {self._state}")


class Memento(ABC):
    """
    The Memento interface provides a way to retrieve the memento's metadata,
    such as creation date or name. However, it doesn't expose the Originator's
    state.
    """

    @abstractmethod
    def get_name(self) -> str:
        pass

    @abstractmethod
    def get_date(self) -> str:
        pass


class ConcreteMemento(Memento):
    def __init__(self, state: str) -> None:
        self._state = state
        self._date = str(datetime.now())[:19]

    def get_state(self) -> str:
        """
        The Originator uses this method when restoring its state.
        """
        return self._state

    def get_name(self) -> str:
        """
        The rest of the methods are used by the Caretaker to display metadata.
        """

        return f"{self._date} / ({self._state[0:9]}...)"

    def get_date(self) -> str:
        return self._date


class Caretaker:
    """
    The Caretaker doesn't depend on the Concrete Memento class. Therefore, it
    doesn't have access to the originator's state, stored inside the memento. It
    works with all mementos via the base Memento interface.
    """

    def __init__(self, originator: Originator) -> None:
        self._mementos = []
        self._originator = originator

    def backup(self) -> None:
        print("\nCaretaker: Saving Originator's state...")
        self._mementos.append(self._originator.save())

    def undo(self) -> None:
        if not len(self._mementos):
            return

        memento = self._mementos.pop()
        print(f"Caretaker: Restoring state to: {memento.get_name()}")
        try:
            self._originator.restore(memento)
        except Exception:
            self.undo()

    def show_history(self) -> None:
        print("Caretaker: Here's the list of mementos:")
        for memento in self._mementos:
            print(memento.get_name())


if __name__ == "__main__":
    originator = Originator("Super-duper-super-puper-super.")
    caretaker = Caretaker(originator)

    caretaker.backup()
    originator.do_something()

    caretaker.backup()
    originator.do_something()

    caretaker.backup()
    originator.do_something()

    print()
    caretaker.show_history()

    print("\nClient: Now, let's rollback!\n")
    caretaker.undo()

    print("\nClient: Once more!\n")
    caretaker.undo()

```

#### **Output.txt:** Execution result

````
Originator: My initial state is: Super-duper-super-puper-super.

Caretaker: Saving Originator's state...
Originator: I'm doing something important.
Originator: and my state has changed to: wQAehHYOqVSlpEXjyIcgobrxsZUnat

Caretaker: Saving Originator's state...
Originator: I'm doing something important.
Originator: and my state has changed to: lHxNORKcsgMWYnJqoXjVCbQLEIeiSp

Caretaker: Saving Originator's state...
Originator: I'm doing something important.
Originator: and my state has changed to: cvIYsRilNOtwynaKdEZpDCQkFAXVMf

Caretaker: Here's the list of mementos:
2019-01-26 21:11:24 / (Super-dup...)
2019-01-26 21:11:24 / (wQAehHYOq...)
2019-01-26 21:11:24 / (lHxNORKcs...)

Client: Now, let's rollback!

Caretaker: Restoring state to: 2019-01-26 21:11:24 / (lHxNORKcs...)
Originator: My state has changed to: lHxNORKcsgMWYnJqoXjVCbQLEIeiSp

Client: Once more!

Caretaker: Restoring state to: 2019-01-26 21:11:24 / (wQAehHYOq...)
Originator: My state has changed to: wQAehHYOqVSlpEXjyIcgobrxsZUnat
````

# **Memento** in Rust

Memento 是一种行为设计模式，允许制作对象状态的快照并在将来恢复它。

## Conceptual example 概念示例

这是 Memento 模式的概念示例。

#### **conceptual.rs**

```rust
trait Memento<T> {
    fn restore(self) -> T;
    fn print(&self);
}

struct Originator {
    state: u32,
}

impl Originator {
    pub fn save(&self) -> OriginatorBackup {
        OriginatorBackup {
            state: self.state.to_string(),
        }
    }
}

struct OriginatorBackup {
    state: String,
}

impl Memento<Originator> for OriginatorBackup {
    fn restore(self) -> Originator {
        Originator {
            state: self.state.parse().unwrap(),
        }
    }

    fn print(&self) {
        println!("Originator backup: '{}'", self.state);
    }
}

fn main() {
    let mut history = Vec::<OriginatorBackup>::new();

    let mut originator = Originator { state: 0 };

    originator.state = 1;
    history.push(originator.save());

    originator.state = 2;
    history.push(originator.save());

    for moment in history.iter() {
        moment.print();
    }

    let originator = history.pop().unwrap().restore();
    println!("Restored to state: {}", originator.state);

    let originator = history.pop().unwrap().restore();
    println!("Restored to state: {}", originator.state);
}

```

### Output 输出

````
Originator backup: '1'
Originator backup: '2'
Restored to state: 2
Restored to state: 1
````

## Serde 序列化框架

使结构可序列化的一种常见方法是从 serde 序列化框架派生 使结构可序列化的一种常见方法是从 serde 序列化框架派生 `Serialize` 和  和  和 `Deserialize` 特征。然后，可序列化类型的对象可以转换为许多不同的格式，e.g. JSON使用serde_json crate。 特征。然后，可序列化类型的对象可以转换为许多不同的格式，e.g. JSON使用serde_json crate。

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct Originator {
    state: u32,
}
```

#### **serde.rs**

```rust
use serde::{Deserialize, Serialize};

/// An object to be stored. It derives a default
/// `Serialize` and `Deserialize` trait implementation, which
/// allows to convert it into many different formats (e.g. JSON).
#[derive(Serialize, Deserialize)]
struct Originator {
    state: u32,
}

impl Originator {
    /// Serializes an originator into a string of JSON format.
    pub fn save(&self) -> String {
        serde_json::to_string(self).unwrap()
    }

    /// Deserializes an originator into a string of JSON format.
    pub fn restore(json: &str) -> Self {
        serde_json::from_str(json).unwrap()
    }
}

fn main() {
    // A stack of mementos.
    let mut history = Vec::<String>::new();

    let mut originator = Originator { state: 0 };

    originator.state = 1;
    history.push(originator.save());

    originator.state = 2;
    history.push(originator.save());

    for moment in history.iter() {
        println!("{}", moment);
    }

    let originator = Originator::restore(&history.pop().unwrap());
    println!("Restored to state: {}", originator.state);

    let originator = Originator::restore(&history.pop().unwrap());
    println!("Restored to state: {}", originator.state);
}

```

### Output 输出

````
{"state":1}
{"state":2}
Restored to state: 2
Restored to state: 1
````

