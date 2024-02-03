---
title: Chain of Responsibility 责任链
categories:
  - design-pattern
tags: Design Pattern
date: 2024-02-03 13:49:38
---

## 意图

Chain of Responsibility是一种行为设计模式，它允许您沿着处理程序链沿着传递请求。在接收到请求时，每个处理程序决定是处理请求还是将其传递给链中的下一个处理程序。


<div align="center"> <img src="/images/chain-of-responsibility-header.png"/></div>


## 问题

假设你正在开发一个在线订购系统。您希望限制对系统的访问，以便只有经过身份验证的用户才能创建订单。此外，具有管理权限的用户必须对所有订单具有完全访问权限。

经过一段时间的规划后，您意识到这些检查必须按顺序执行。每当应用程序收到包含用户凭据的请求时，它都可以尝试向系统验证用户的身份。但是，如果这些凭据不正确并且身份验证失败，则没有理由继续进行任何其他检查。


<div align="center"> <img src="/images/responsibilty-problem1.png"/>请求必须通过一系列检查，然后订购系统才能处理它。</div>


在接下来的几个月里，您又实现了几个这样的顺序检查。

- 您的一位同事建议将原始数据直接传递到订购系统是不安全的。因此，您添加了一个额外的验证步骤来清理请求中的数据。
- 后来，有人注意到，该系统是脆弱的暴力破解密码。为了消除这种情况，您立即添加了一个检查，该检查过滤来自同一IP地址的重复失败请求。
- 还有人建议，可以通过返回包含相同数据的重复请求的缓存结果来加快系统速度。因此，您添加了另一个检查，仅当没有合适的缓存响应时，才允许请求通过系统。


<div align="center"> <img src="/images/responsibility-problem2.png"/>代码越庞杂，就越混乱。</div>


检查的代码本来看起来就很混乱，随着您添加每个新特性，它变得越来越臃肿。更改一张支票有时会影响其他支票。最糟糕的是，当您试图重用检查来保护系统的其他组件时，您不得不复制一些代码，因为这些组件需要一些检查，但不是所有检查。

这个系统变得很难理解，维护起来也很昂贵。你在代码上挣扎了一段时间，直到有一天你决定重构整个代码。

## 溶液

与许多其他行为设计模式一样，责任链依赖于将特定行为转换为称为处理程序的独立对象。在我们的例子中，每个检查都应该被提取到它自己的类中，并使用一个执行检查的方法。请求沿着数据作为参数传递给此方法。

该模式建议您将这些处理程序链接成一个链。每个链接的处理程序都有一个字段，用于存储对链中下一个处理程序的引用。除了处理请求之外，处理程序还将请求进一步沿着链传递。请求沿着链传递，直到所有处理程序都有机会处理它。

这里是最好的部分：处理程序可以决定不将请求进一步向下传递，并有效地停止任何进一步的处理。

在我们的订单系统示例中，处理程序执行处理，然后决定是否将请求进一步向下传递。假设请求包含正确的数据，所有处理程序都可以执行其主要行为，无论是身份验证检查还是缓存。


<div align="center"> <img src="/images/responsibility-solution1-en.png"/>把手一个接一个地排列，形成一条链条。</div>


然而，还有一种稍微不同的方法（它更规范一点），在这种方法中，在接收到请求时，处理程序决定它是否可以处理它。如果可以，它就不再传递请求。因此，要么只有一个处理程序处理请求，要么根本没有处理程序。这种方法在处理图形用户界面中的元素堆栈中的事件时非常常见。

例如，当用户单击按钮时，事件将通过GUI元素链传播，该元素链从按钮开始，沿着其容器（如窗体或面板）传播，最后到达主应用程序窗口。事件由链中能够处理它的第一个元素处理。这个例子也值得注意，因为它表明链总是可以从对象树中提取。


<div align="center"> <img src="/images/responsibility-solution2.png"/>链可以由对象树的分支形成。</div>


所有的处理程序类实现相同的接口是至关重要的。每个具体的处理程序应该只关心下面的一个具有 所有的处理程序类实现相同的接口是至关重要的。每个具体的处理程序应该只关心下面的一个具有 `execute` 方法的处理程序。通过这种方式，您可以在运行时使用各种处理程序组合链，而无需将代码耦合到它们的具体类。 方法的处理程序。通过这种方式，您可以在运行时使用各种处理程序组合链，而无需将代码耦合到它们的具体类。

## 结构


<div align="center"> <img src="/images/responsibility-structure.png"/></div>


1. 该函数声明了接口，对所有具体处理程序都是通用的。它通常只包含一个处理请求的方法，但有时它也可能有另一个方法来设置链上的下一个处理程序。
2. BaseHandler是一个可选类，您可以在其中放置所有处理程序类通用的样板代码。

   通常，这个类定义一个字段来存储对下一个处理程序的引用。客户端可以通过将处理程序传递给前一个处理程序的构造函数或setter来构建链。类也可以实现默认的处理行为：它可以在检查其存在之后将执行传递给下一个处理程序。

3. 具体处理程序包含处理请求的实际代码。在接收到一个请求后，每个处理程序必须决定是否处理它，另外，是否沿着链沿着它。

   处理程序通常是自包含的和不可变的，通过构造函数只接受一次所有必要的数据。

4. 客户端可以只组合一次链，也可以动态组合它们，这取决于应用程序的逻辑。请注意，请求可以发送到链中的任何处理程序-它不必是第一个。

##  伪代码

在本例中，责任链模式负责显示活动GUI元素的上下文帮助信息。


<div align="center"> <img src="/images/responsibility-example1.png"/>GUI类是用Composite模式构建的。每个元素都链接到它的容器元素。在任何时候，您都可以构建一个元素链，该元素链从元素本身开始，遍历它的所有容器元素。</div>


应用程序的GUI通常被构造为对象树。例如，呈现应用程序主窗口的 `Dialog` 类将是对象树的根。对话框包含 `Panels` ，它可能包含其他面板或简单的低级元素，如 `Buttons` 和 `TextFields` 。

一个简单的组件可以显示简短的上下文工具提示，只要该组件有一些帮助文本分配。但更复杂的组件定义了自己显示上下文帮助的方式，例如显示手册摘录或在浏览器中打开页面。responsibility-example2


<div align="center"> <img src="/images/responsibility-solution2.png"/>这就是帮助请求遍历GUI对象的方式。</div>


当用户将鼠标光标指向某个元素并按下 当用户将鼠标光标指向某个元素并按下 `F1` 键时，应用程序将检测指针下的组件并向其发送帮助请求。该请求将在元素的所有容器中冒泡，直到到达能够显示帮助信息的元素。 键时，应用程序将检测指针下的组件并向其发送帮助请求。该请求将在元素的所有容器中冒泡，直到到达能够显示帮助信息的元素。

```java
// The handler interface declares a method for executing a
// request.
interface ComponentWithContextualHelp is
    method showHelp()


// The base class for simple components.
abstract class Component implements ComponentWithContextualHelp is
    field tooltipText: string

    // The component's container acts as the next link in the
    // chain of handlers.
    protected field container: Container

    // The component shows a tooltip if there's help text
    // assigned to it. Otherwise it forwards the call to the
    // container, if it exists.
    method showHelp() is
        if (tooltipText != null)
            // Show tooltip.
        else
            container.showHelp()


// Containers can contain both simple components and other
// containers as children. The chain relationships are
// established here. The class inherits showHelp behavior from
// its parent.
abstract class Container extends Component is
    protected field children: array of Component

    method add(child) is
        children.add(child)
        child.container = this


// Primitive components may be fine with default help
// implementation...
class Button extends Component is
    // ...

// But complex components may override the default
// implementation. If the help text can't be provided in a new
// way, the component can always call the base implementation
// (see Component class).
class Panel extends Container is
    field modalHelpText: string

    method showHelp() is
        if (modalHelpText != null)
            // Show a modal window with the help text.
        else
            super.showHelp()

// ...same as above...
class Dialog extends Container is
    field wikiPageURL: string

    method showHelp() is
        if (wikiPageURL != null)
            // Open the wiki help page.
        else
            super.showHelp()


// Client code.
class Application is
    // Every application configures the chain differently.
    method createUI() is
        dialog = new Dialog("Budget Reports")
        dialog.wikiPageURL = "http://..."
        panel = new Panel(0, 0, 400, 800)
        panel.modalHelpText = "This panel does..."
        ok = new Button(250, 760, 50, 20, "OK")
        ok.tooltipText = "This is an OK button that..."
        cancel = new Button(320, 760, 50, 20, "Cancel")
        // ...
        panel.add(ok)
        panel.add(cancel)
        dialog.add(panel)

    // Imagine what happens here.
    method onF1KeyPress() is
        component = this.getComponentAtMouseCoords()
        component.showHelp()
```

## 适用性

- **当你的程序需要以不同的方式处理不同类型的请求，但请求的确切类型和它们的顺序事先是未知的时，请使用责任链模式。**
- 该模式允许您将多个处理程序链接到一个链中，并在收到请求时“询问”每个处理程序是否可以处理该请求。
- **当必须以特定顺序执行多个处理程序时，请使用该模式。**
- 由于您可以以任何顺序链接链中的处理程序，因此所有请求都将完全按照您的计划通过链。
- 当处理程序集及其顺序应该在运行时更改时，请使用CoR模式。
- 如果在处理程序类中为引用字段提供setter，则可以动态地插入、删除或重新排序处理程序。

## 如何实现

1. 描述处理程序接口并描述处理请求的方法的签名。

   决定客户端如何将请求数据传递到方法中。最灵活的方法是将请求转换为对象，并将其作为参数传递给处理方法。

2. 为了消除具体处理程序中重复的样板代码，可能需要创建一个从处理程序接口派生的抽象基处理程序类。

   这个类应该有一个字段来存储对链中下一个处理程序的引用。考虑使类不可变。但是，如果您计划在运行时修改链，则需要定义一个setter来更改引用字段的值。

   您还可以为处理方法实现方便的默认行为，即将请求转发到下一个对象，除非没有对象了。具体处理程序将能够通过调用父方法来使用此行为。

3. 逐个创建具体的处理程序子类并实现它们的处理方法。每个处理程序在接收请求时应该做出两个决定：

   - 它是否会处理请求。
   - 它是否会沿着链沿着传递请求。

4. 客户端可以自行组装链，也可以从其他对象接收预构建的链。在后一种情况下，您必须实现一些工厂类来根据配置或环境设置构建链
5. 客户端可以触发链中的任何处理程序，而不仅仅是第一个。请求将沿着链传递，直到某个处理程序拒绝进一步传递它，或者直到它到达链的末端。
6. 由于链的动态特性，客户端应该准备好处理以下场景：

   - 链条可以由单个链环组成。
   - 有些请求可能无法到达链的末端。
   - 其他人可能会到达链条的末端。


## 利弊

| √ 利                                                                            | × 弊                       |
| ------------------------------------------------------------------------------- | -------------------------- |
| 您可以控制请求处理的顺序。                                                      | 有些请求可能最终无法处理。 |
| 单一责任原则。可以将调用操作的类与执行操作的类解耦。                            |                            |
| 开放/封闭原则。您可以在应用程序中引入新的处理程序，而不会破坏现有的客户端代码。 |                            |

## 与其他模式的关系

- 责任链、命令、调解器和观察者解决了连接请求的接收者和接收者的各种方式：

  - 责任链（Chain of Responsibility）将请求顺序地沿着一个动态的潜在接收者链传递，直到其中一个接收者处理它。
  - 命令在中继器和接收器之间建立单向连接。
  - Mediator消除了发送者和接收者之间的直接连接，迫使它们通过Mediator对象间接通信。
  - 观察者允许接收者动态订阅和取消订阅接收请求。

- 责任链通常与复合一起使用。在这种情况下，当一个叶组件收到一个请求时，它可以通过所有父组件的链将其传递到对象树的根。
- 责任链中的处理程序可以作为命令实现。在这种情况下，您可以对同一个上下文对象（由请求表示）执行许多不同的操作。

  然而，还有另一种方法，其中请求本身是一个Command对象。在这种情况下，您可以在链接成链的一系列不同上下文中执行相同的操作。

- Chain of Responsibility和Decorator具有非常相似的类结构。这两种模式都依赖于递归组合来通过一系列对象传递执行。然而，有几个关键的区别。

  CoR处理程序可以彼此独立地执行任意操作。他们也可以在任何时候停止进一步传递请求。另一方面，各种装饰器可以扩展对象的行为，同时保持它与基接口的一致性。此外，装饰器不允许中断请求流。


# Python中的责任链

责任链是一种行为设计模式，它允许沿着潜在处理程序链传递请求，直到其中一个处理请求。

该模式允许多个对象处理请求，而无需将发送方类耦合到接收方的具体类。链可以在运行时动态地与遵循标准处理程序接口的任何处理程序组合。

## 概念示例

这个例子说明了责任链设计模式的结构。它侧重于回答这些问题：

- 它由哪些类组成？
- 这些班级扮演什么角色？
- 模式中的元素是以什么方式联系在一起的？

#### main.py：概念性示例

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from typing import Any, Optional


class Handler(ABC):
    """
    The Handler interface declares a method for building the chain of handlers.
    It also declares a method for executing a request.
    """

    @abstractmethod
    def set_next(self, handler: Handler) -> Handler:
        pass

    @abstractmethod
    def handle(self, request) -> Optional[str]:
        pass


class AbstractHandler(Handler):
    """
    The default chaining behavior can be implemented inside a base handler
    class.
    """

    _next_handler: Handler = None

    def set_next(self, handler: Handler) -> Handler:
        self._next_handler = handler
        # Returning a handler from here will let us link handlers in a
        # convenient way like this:
        # monkey.set_next(squirrel).set_next(dog)
        return handler

    @abstractmethod
    def handle(self, request: Any) -> str:
        if self._next_handler:
            return self._next_handler.handle(request)

        return None


"""
All Concrete Handlers either handle a request or pass it to the next handler in
the chain.
"""


class MonkeyHandler(AbstractHandler):
    def handle(self, request: Any) -> str:
        if request == "Banana":
            return f"Monkey: I'll eat the {request}"
        else:
            return super().handle(request)


class SquirrelHandler(AbstractHandler):
    def handle(self, request: Any) -> str:
        if request == "Nut":
            return f"Squirrel: I'll eat the {request}"
        else:
            return super().handle(request)


class DogHandler(AbstractHandler):
    def handle(self, request: Any) -> str:
        if request == "MeatBall":
            return f"Dog: I'll eat the {request}"
        else:
            return super().handle(request)


def client_code(handler: Handler) -> None:
    """
    The client code is usually suited to work with a single handler. In most
    cases, it is not even aware that the handler is part of a chain.
    """

    for food in ["Nut", "Banana", "Cup of coffee"]:
        print(f"\nClient: Who wants a {food}?")
        result = handler.handle(food)
        if result:
            print(f"  {result}", end="")
        else:
            print(f"  {food} was left untouched.", end="")


if __name__ == "__main__":
    monkey = MonkeyHandler()
    squirrel = SquirrelHandler()
    dog = DogHandler()

    monkey.set_next(squirrel).set_next(dog)

    # The client should be able to send a request to any handler, not just the
    # first one in the chain.
    print("Chain: Monkey > Squirrel > Dog")
    client_code(monkey)
    print("\n")

    print("Subchain: Squirrel > Dog")
    client_code(squirrel)
```

#### Output.txt：执行结果

````
Chain: Monkey > Squirrel > Dog

Client: Who wants a Nut?
  Squirrel: I'll eat the Nut
Client: Who wants a Banana?
  Monkey: I'll eat the Banana
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.

Subchain: Squirrel > Dog

Client: Who wants a Nut?
  Squirrel: I'll eat the Nut
Client: Who wants a Banana?
  Banana was left untouched.
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.
````

# Rust中的责任链

责任链是一种行为设计模式，它允许沿着潜在处理程序链传递请求，直到其中一个处理请求。

该模式允许多个对象处理请求，而无需将发送方类耦合到接收方的具体类。链可以在运行时动态地与遵循标准处理程序接口的任何处理程序组合。

## 概念示例

该示例演示了如何通过一系列部门处理患者。责任链的构成如下：

```rust
Patient -> Reception -> Doctor -> Medical -> Cashier
```

链是使用 链是使用 `Box` 指针构建的，这意味着在运行时动态分派。为什么？为什么？使用泛型将实现缩小到严格的编译时类型似乎相当困难：为了构造一个完整链的类型，Rust需要完全了解链中的“下一个”链接。因此，它看起来像这样： 指针构建的，这意味着在运行时动态分派。为什么？为什么？使用泛型将实现缩小到严格的编译时类型似乎相当困难：为了构造一个完整链的类型，Rust需要完全了解链中的“下一个”链接。因此，它看起来像这样：

```rust
let mut reception = Reception::<Doctor::<Medical::<Cashier>>>::new(doctor); // 😱
```

相反， 相反， `Box` 允许以任何组合进行链接： 允许以任何组合进行链接：

```rust
let mut reception = Reception::new(doctor); // 👍

let mut reception = Reception::new(cashier); // 🕵️‍♀️
```

#### **patient.rs:** Request

```rust
#[derive(Default)]
pub struct Patient {
    pub name: String,
    pub registration_done: bool,
    pub doctor_check_up_done: bool,
    pub medicine_done: bool,
    pub payment_done: bool,
}
```

#### **department.rs:** Handlers

```rust
mod cashier;
mod doctor;
mod medical;
mod reception;

pub use cashier::Cashier;
pub use doctor::Doctor;
pub use medical::Medical;
pub use reception::Reception;

use crate::patient::Patient;

/// A single role of objects that make up a chain.
/// A typical trait implementation must have `handle` and `next` methods,
/// while `execute` is implemented by default and contains a proper chaining
/// logic.
pub trait Department {
    fn execute(&mut self, patient: &mut Patient) {
        self.handle(patient);

        if let Some(next) = &mut self.next() {
            next.execute(patient);
        }
    }

    fn handle(&mut self, patient: &mut Patient);
    fn next(&mut self) -> &mut Option<Box<dyn Department>>;
}

/// Helps to wrap an object into a boxed type.
pub(self) fn into_next(
    department: impl Department + Sized + 'static,
) -> Option<Box<dyn Department>> {
    Some(Box::new(department))
}
```

#### **department/cashier.rs**

```rust
use super::{Department, Patient};

#[derive(Default)]
pub struct Cashier {
    next: Option<Box<dyn Department>>,
}

impl Department for Cashier {
    fn handle(&mut self, patient: &mut Patient) {
        if patient.payment_done {
            println!("Payment done");
        } else {
            println!("Cashier getting money from a patient {}", patient.name);
            patient.payment_done = true;
        }
    }

    fn next(&mut self) -> &mut Option<Box<dyn Department>> {
        &mut self.next
    }
}
```

#### **department/doctor.rs**

```rust
use super::{into_next, Department, Patient};

pub struct Doctor {
    next: Option<Box<dyn Department>>,
}

impl Doctor {
    pub fn new(next: impl Department + 'static) -> Self {
        Self {
            next: into_next(next),
        }
    }
}

impl Department for Doctor {
    fn handle(&mut self, patient: &mut Patient) {
        if patient.doctor_check_up_done {
            println!("A doctor checkup is already done");
        } else {
            println!("Doctor checking a patient {}", patient.name);
            patient.doctor_check_up_done = true;
        }
    }

    fn next(&mut self) -> &mut Option<Box<dyn Department>> {
        &mut self.next
    }
}
```

####  **department/medical.rs**

```rust
use super::{into_next, Department, Patient};

pub struct Medical {
    next: Option<Box<dyn Department>>,
}

impl Medical {
    pub fn new(next: impl Department + 'static) -> Self {
        Self {
            next: into_next(next),
        }
    }
}

impl Department for Medical {
    fn handle(&mut self, patient: &mut Patient) {
        if patient.medicine_done {
            println!("Medicine is already given to a patient");
        } else {
            println!("Medical giving medicine to a patient {}", patient.name);
            patient.medicine_done = true;
        }
    }

    fn next(&mut self) -> &mut Option<Box<dyn Department>> {
        &mut self.next
    }
}
```

####  **department/reception.rs**

```rust
use super::{into_next, Department, Patient};

#[derive(Default)]
pub struct Reception {
    next: Option<Box<dyn Department>>,
}

impl Reception {
    pub fn new(next: impl Department + 'static) -> Self {
        Self {
            next: into_next(next),
        }
    }
}

impl Department for Reception {
    fn handle(&mut self, patient: &mut Patient) {
        if patient.registration_done {
            println!("Patient registration is already done");
        } else {
            println!("Reception registering a patient {}", patient.name);
            patient.registration_done = true;
        }
    }

    fn next(&mut self) -> &mut Option<Box<dyn Department>> {
        &mut self.next
    }
}
```

#### **main.rs:** Client

```rust
mod department;
mod patient;

use department::{Cashier, Department, Doctor, Medical, Reception};
use patient::Patient;

fn main() {
    let cashier = Cashier::default();
    let medical = Medical::new(cashier);
    let doctor = Doctor::new(medical);
    let mut reception = Reception::new(doctor);

    let mut patient = Patient {
        name: "John".into(),
        ..Patient::default()
    };

    // Reception handles a patient passing him to the next link in the chain.
    // Reception -> Doctor -> Medical -> Cashier.
    reception.execute(&mut patient);

    println!("\nThe patient has been already handled:\n");

    reception.execute(&mut patient);
}
```

### 输出

````
Reception registering a patient John
Doctor checking a patient John
Medical giving medicine to a patient John
Cashier getting money from a patient John

The patient has been already handled:

Patient registration is already done
A doctor checkup is already done
Medicine is already given to a patient
Payment done
````

