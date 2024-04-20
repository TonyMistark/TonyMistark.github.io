---
title: Mediator 中介者模式
categories:
  - design-pattern
date: 2024-04-20 12:33:40
---

## Intent  意图

Mediator是一种行为设计模式，可以减少对象之间的混乱依赖关系。该模式限制了对象之间的直接通信，并强制它们只能通过中介对象进行协作。


<div align="center"> <img src="/images/mediator-header.png"/></div>


##  Problem  问题

假设您有一个用于创建和编辑客户配置文件的对话框。它由各种表单控件组成，如文本字段、复选框、按钮等。


<div align="center"> <img src="/images/mediator-problem.png"/>随着应用程序的发展，用户界面元素之间的关系可能会变得混乱。</div>


某些表单元素可能与其他表单元素交互。例如，选择“我有一只狗”复选框可以显示用于输入狗的名字的隐藏文本字段。另一个例子是提交按钮，它必须在保存数据之前验证所有字段的值。


<div align="center"> <img src="/images/mediator-problem2.png"/>元素可以与其他元素有很多关系。因此，某些要素的变化可能会影响其他要素。

</div>


通过直接在表单元素的代码中实现这种逻辑，你会使这些元素的类更难在应用的其他表单中重用。例如，你将无法在另一个表单中使用复选框类，因为它耦合到狗的文本字段。您可以使用呈现配置文件表单所涉及的所有类，也可以不使用任何类。

## Solution  解决方案

中介者模式建议您应该停止您想要使其彼此独立的组件之间的所有直接通信。相反，这些组件必须通过调用一个特殊的中介对象来间接协作，该对象将调用重定向到适当的组件。因此，组件只依赖于单个中介类，而不是耦合到几十个同事。

在我们的配置文件编辑表单示例中，对话框类本身可以充当中介。dialog类很可能已经知道它的所有子元素，因此您甚至不需要在该类中引入新的依赖项。


<div align="center"> <img src="/images/mediator-problem2.png"/>UI元素应该通过中介对象进行间接通信。</div>


最重要的变化发生在实际的表单元素上。让我们考虑一下提交按钮。以前，用户每次单击按钮时，都必须验证所有单个表单元素的值。现在，它的唯一任务是通知对话框有关单击的信息。在收到此通知后，对话框本身将执行验证或将任务传递给各个元素。因此，按钮并不依赖于十几个表单元素，而是仅依赖于dialog类。

您还可以更进一步，通过为所有类型的对话框提取公共接口，使依赖性更加松散。该接口将声明通知方法，所有表单元素都可以使用该方法通知对话框发生在这些元素上的事件。因此，我们的提交按钮现在应该能够与实现该接口的任何对话框一起工作。

通过这种方式，Mediator模式允许您将各种对象之间的复杂关系网封装在单个Mediator对象中。类的依赖项越少，就越容易修改、扩展或重用该类。

## Real-World Analogy  现实世界的类比


<div align="center"> <img src="/images/mediator-live-example.png"/>飞机飞行员在决定下一个降落的时候不会直接互相交谈。所有的通讯都通过控制塔。</div>


接近或离开机场控制区的飞机飞行员不直接相互交流。相反，他们与一位空中交通管制员交谈，他坐在飞机跑道附近的一座高塔上。如果没有空中交通管制员，飞行员将需要了解机场附近的每一架飞机，并与数十名其他飞行员组成的委员会讨论着陆优先级。这可能会使飞机失事的统计数字飙升。

塔台不需要控制整个飞行。它的存在仅仅是为了在终端区域实施约束，因为那里涉及的参与者的数量可能对飞行员来说是压倒性的。

## Structure  结构


<div align="center"> <img src="/images/mediator-structure.png"/></div>


1. 组件是包含一些业务逻辑的各种类。每个组件都有一个对中介器的引用，用中介器接口的类型声明。该组件并不知道中介器的实际类，因此您可以通过将其链接到不同的中介器来在其他程序中重用该组件。
2. Mediator接口声明了与组件通信的方法，这些方法通常只包括一个通知方法。组件可以将任何上下文作为此方法的参数传递，包括它们自己的对象，但只能以这样一种方式传递，即接收组件和发送方的类之间不发生耦合。
3. 具体中介封装了各种组件之间的关系。具体的中介者经常保持对它们管理的所有组件的引用，有时甚至管理它们的生命周期。
4. 组件必须不知道其他组件。如果某个组件内部发生了重要的事情，它必须只通知中介。当中介器收到通知时，它可以轻松地识别发送者，这可能足以决定应该触发哪个组件作为回报。

   从组件的角度来看，这一切看起来像一个完全的黑盒子。发送方不知道谁将最终处理它的请求，接收方也不知道谁首先发送了请求。


##  Pseudocode  伪代码

在本例中，中介模式帮助您消除各种UI类（按钮、复选框和文本标签）之间的相互依赖


<div align="center"> <img src="/images/mediator-example2.png"/></div>


由用户触发的元素不会直接与其他元素通信，即使它看起来像是应该这样。相反，元素只需要让它的中介者知道事件，并将任何上下文信息沿着一起传递。

在本例中，整个身份验证对话框充当中介。它知道具体元素应该如何协作，并促进它们的间接通信。在接收到关于事件的通知时，对话框决定哪个元素应该处理该事件，并相应地重定向调用。

```java
// The mediator interface declares a method used by components
// to notify the mediator about various events. The mediator may
// react to these events and pass the execution to other
// components.
interface Mediator is
    method notify(sender: Component, event: string)


// The concrete mediator class. The intertwined web of
// connections between individual components has been untangled
// and moved into the mediator.
class AuthenticationDialog implements Mediator is
    private field title: string
    private field loginOrRegisterChkBx: Checkbox
    private field loginUsername, loginPassword: Textbox
    private field registrationUsername, registrationPassword,
                  registrationEmail: Textbox
    private field okBtn, cancelBtn: Button

    constructor AuthenticationDialog() is
        // Create all component objects by passing the current
        // mediator into their constructors to establish links.

    // When something happens with a component, it notifies the
    // mediator. Upon receiving a notification, the mediator may
    // do something on its own or pass the request to another
    // component.
    method notify(sender, event) is
        if (sender == loginOrRegisterChkBx and event == "check")
            if (loginOrRegisterChkBx.checked)
                title = "Log in"
                // 1. Show login form components.
                // 2. Hide registration form components.
            else
                title = "Register"
                // 1. Show registration form components.
                // 2. Hide login form components

        if (sender == okBtn && event == "click")
            if (loginOrRegister.checked)
                // Try to find a user using login credentials.
                if (!found)
                    // Show an error message above the login
                    // field.
            else
                // 1. Create a user account using data from the
                // registration fields.
                // 2. Log that user in.
                // ...


// Components communicate with a mediator using the mediator
// interface. Thanks to that, you can use the same components in
// other contexts by linking them with different mediator
// objects.
class Component is
    field dialog: Mediator

    constructor Component(dialog) is
        this.dialog = dialog

    method click() is
        dialog.notify(this, "click")

    method keypress() is
        dialog.notify(this, "keypress")

// Concrete components don't talk to each other. They have only
// one communication channel, which is sending notifications to
// the mediator.
class Button extends Component is
    // ...

class Textbox extends Component is
    // ...

class Checkbox extends Component is
    method check() is
        dialog.notify(this, "check")
    // ...
```

## Applicability  适用性

- 当由于类与其他类紧密耦合而难以更改某些类时，请使用中介模式。
- 该模式允许您将类之间的所有关系提取到一个单独的类中，从而将对特定组件的任何更改与其他组件隔离开来。
- 当您不能在不同的程序中重用某个组件时，请使用该模式，因为它太依赖于其他组件。
- 应用Mediator后，各个组件将不知道其他组件。它们仍然可以通过中介对象彼此通信，尽管是间接的。要在不同的应用中重用组件，您需要为其提供新的中介器类。
- 当您发现自己创建了大量的组件子类，只是为了在各种上下文中重用一些基本行为时，请使用Mediator。
- 由于组件之间的所有关系都包含在中介器中，因此通过引入新的中介器类，可以很容易地为这些组件定义全新的协作方式，而无需更改组件本身。

## How to Implement 如何实现

1. 识别一组紧密耦合的类，它们将从更加独立中受益（例如，以便更容易地维护或更简单地重用这些类）。

2. 描述中介器接口，并描述中介器和各种组件之间所需的通信协议。在大多数情况下，从组件接收通知的单一方法就足够了。

   当您希望在不同的上下文中重用组件类时，此接口至关重要。只要组件通过通用接口与其中介器一起工作，您就可以将组件与中介器的不同实现链接起来。

3. 实现具体的中介类。考虑在中介器中存储对所有组件的引用。这样，您就可以从中介器的方法中调用任何组件。
4. 您可以更进一步，让中介器负责组件对象的创建和销毁。在此之后，中介可能类似于工厂模式（factory）或门面模式（facade）。
5. 组件应该存储对中介对象的引用。连接通常是在组件的构造函数中建立的，其中中介器对象作为参数传递。
6. 更改组件的代码，以便它们调用中介的通知方法，而不是其他组件上的方法。将涉及调用其他组件的代码提取到mediator类中。只要中介器收到来自该组件的通知，就执行此代码。

## Pros and Cons 利弊

| 利√                                                                            | 弊×                                              |
| ------------------------------------------------------------------------------ | ------------------------------------------------ |
| 单一责任原则。您可以将各种组件之间的通信提取到一个地方，使其更容易理解和维护。 | 随着时间的推移，一个中介可以进化成一个上帝对象。 |
| 开放/封闭原则。您可以引入新的中介，而不必更改实际的组件。                      |                                                  |
| 您可以减少程序的各个组件之间的耦合。                                           |                                                  |
| 您可以更轻松地重用单个组件。                                                   |                                                  |

## Relations with Other Patterns 与其他模式的关系

- 责任链、命令、调解器和观察者解决了连接请求的接收者和接收者的各种方式：

  - 责任链（Chain of Responsibility）将请求顺序地沿着一个动态的潜在接收者链传递，直到其中一个接收者处理它。
  - 命令在中继器和接收器之间建立单向连接。
  - Mediator消除了发送者和接收者之间的直接连接，迫使它们通过Mediator对象间接通信。
  - 观察者允许接收者动态订阅和取消订阅接收请求。

- Facade和Mediator有着类似的工作：它们试图组织许多紧密耦合的类之间的协作。

  - Facade为对象子系统定义了一个简化的接口，但它没有引入任何新功能。子系统本身不知道facade。子系统内的对象可以直接通信。
  - Mediator集中系统组件之间的通信。组件只知道中介对象，不直接通信

- 调解人和观察员之间的区别往往是难以捉摸的。在大多数情况下，您可以实现这两种模式中的任何一种;但有时您可以同时应用这两种模式。让我们看看如何做到这一点。

  Mediator的主要目标是消除一组系统组件之间的相互依赖性。相反，这些组件依赖于单个中介对象。Observer的目标是在对象之间建立动态的单向连接，其中一些对象充当其他对象的从属对象。

  中介者模式有一个流行的实现依赖于观察者。中介者对象扮演发布者的角色，而组件则充当订阅者，订阅和取消订阅中介者的事件。当Mediator以这种方式实现时，它可能看起来非常类似于Observer。

  当您感到困惑时，请记住您可以用其他方式实现中介者模式。例如，您可以将所有组件永久链接到同一个中介器对象。这个实现与Observer不同，但仍然是Mediator模式的一个实例。

  现在想象一个程序，其中所有组件都成为发布者，允许彼此之间的动态连接。不会有一个集中的中介对象，只有一组分布式的观察者。


## Code Examples  代码示例

# **Mediator** in Python Python中的Mediator

中介者模式是一种行为设计模式，它通过使程序组件之间通过一个特殊的中介器对象进行间接通信来减少组件之间的耦合。

Mediator使修改、扩展和重用单个组件变得容易，因为它们不再依赖于其他几十个类。

## Conceptual Example 概念示例

此示例说明了中介器设计模式的结构。它侧重于回答这些问题：

- 它由哪些类组成？
- 这些班级扮演什么角色？
- 模式中的元素是以什么方式联系在一起的？

#### main.py：概念性示例

```python
from __future__ import annotations
from abc import ABC


class Mediator(ABC):
    """
    The Mediator interface declares a method used by components to notify the
    mediator about various events. The Mediator may react to these events and
    pass the execution to other components.
    """

    def notify(self, sender: object, event: str) -> None:
        pass


class ConcreteMediator(Mediator):
    def __init__(self, component1: Component1, component2: Component2) -> None:
        self._component1 = component1
        self._component1.mediator = self
        self._component2 = component2
        self._component2.mediator = self

    def notify(self, sender: object, event: str) -> None:
        if event == "A":
            print("Mediator reacts on A and triggers following operations:")
            self._component2.do_c()
        elif event == "D":
            print("Mediator reacts on D and triggers following operations:")
            self._component1.do_b()
            self._component2.do_c()


class BaseComponent:
    """
    The Base Component provides the basic functionality of storing a mediator's
    instance inside component objects.
    """

    def __init__(self, mediator: Mediator = None) -> None:
        self._mediator = mediator

    @property
    def mediator(self) -> Mediator:
        return self._mediator

    @mediator.setter
    def mediator(self, mediator: Mediator) -> None:
        self._mediator = mediator


"""
Concrete Components implement various functionality. They don't depend on other
components. They also don't depend on any concrete mediator classes.
"""


class Component1(BaseComponent):
    def do_a(self) -> None:
        print("Component 1 does A.")
        self.mediator.notify(self, "A")

    def do_b(self) -> None:
        print("Component 1 does B.")
        self.mediator.notify(self, "B")


class Component2(BaseComponent):
    def do_c(self) -> None:
        print("Component 2 does C.")
        self.mediator.notify(self, "C")

    def do_d(self) -> None:
        print("Component 2 does D.")
        self.mediator.notify(self, "D")


if __name__ == "__main__":
    # The client code.
    c1 = Component1()
    c2 = Component2()
    mediator = ConcreteMediator(c1, c2)

    print("Client triggers operation A.")
    c1.do_a()

    print("\n", end="")

    print("Client triggers operation D.")
    c2.do_d()
```

#### Output.txt：执行结果

````
Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.


Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.
Component 2 does C.
````

# **Mediator** in Rust Rust中的Mediator

中介者模式是一种行为设计模式，它通过使程序组件之间通过一个特殊的中介器对象进行间接通信来减少组件之间的耦合

## Top-Down Ownership 自上而下的所有权

自上而下的所有权方法允许在Rust中应用Mediator，因为它适合Rust的所有权模型，具有严格的借用检查规则。这不是实现Mediator的唯一方法，但却是一种基本方法。

关键是从所有权的角度思考。

1. 调解人拥有所有组件的所有权。
2. 组件不保留对中介的引用。相反，它通过方法调用获取引用。

```rust
// A train gets a mediator object by reference.
pub trait Train {
    fn name(&self) -> &String;
    fn arrive(&mut self, mediator: &mut dyn Mediator);
    fn depart(&mut self, mediator: &mut dyn Mediator);
}

// Mediator has notification methods.
pub trait Mediator {
    fn notify_about_arrival(&mut self, train_name: &str) -> bool;
    fn notify_about_departure(&mut self, train_name: &str);
}
```

3. 控制流从 控制流从 `fn main()` 开始，其中中介器接收外部事件/命令。 开始，其中中介器接收外部事件/命令。
4. 用于组件之间交互的 用于组件之间交互的 `Mediator` trait（  trait（  trait（ `notify_about_arrival` 、  、  、 `notify_about_departure` ）与其用于接收外部事件的外部API（来自主循环的  ）与其用于接收外部事件的外部API（来自主循环的  ）与其用于接收外部事件的外部API（来自主循环的 `accept` 、  、  、 `depart` 命令）不同。 命令）不同。

```rust
let train1 = PassengerTrain::new("Train 1");
let train2 = FreightTrain::new("Train 2");

// Station has `accept` and `depart` methods,
// but it also implements `Mediator`.
let mut station = TrainStation::default();

// Station is taking ownership of the trains.
station.accept(train1);
station.accept(train2);

// `train1` and `train2` have been moved inside,
// but we can use train names to depart them.
station.depart("Train 1");
station.depart("Train 2");
station.depart("Train 3");
```




<div align="center"> <img src="/images/mediator-rust-approach.png"/></div>


#### train_station.rs 

```rust
use std::collections::{HashMap, VecDeque};

use crate::trains::Train;

// Mediator has notification methods.
pub trait Mediator {
    fn notify_about_arrival(&mut self, train_name: &str) -> bool;
    fn notify_about_departure(&mut self, train_name: &str);
}

#[derive(Default)]
pub struct TrainStation {
    trains: HashMap<String, Box<dyn Train>>,
    train_queue: VecDeque<String>,
    train_on_platform: Option<String>,
}

impl Mediator for TrainStation {
    fn notify_about_arrival(&mut self, train_name: &str) -> bool {
        if self.train_on_platform.is_some() {
            self.train_queue.push_back(train_name.into());
            false
        } else {
            self.train_on_platform.replace(train_name.into());
            true
        }
    }

    fn notify_about_departure(&mut self, train_name: &str) {
        if Some(train_name.into()) == self.train_on_platform {
            self.train_on_platform = None;

            if let Some(next_train_name) = self.train_queue.pop_front() {
                let mut next_train = self.trains.remove(&next_train_name).unwrap();
                next_train.arrive(self);
                self.trains.insert(next_train_name.clone(), next_train);

                self.train_on_platform = Some(next_train_name);
            }
        }
    }
}

impl TrainStation {
    pub fn accept(&mut self, mut train: impl Train + 'static) {
        if self.trains.contains_key(train.name()) {
            println!("{} has already arrived", train.name());
            return;
        }

        train.arrive(self);
        self.trains.insert(train.name().clone(), Box::new(train));
    }

    pub fn depart(&mut self, name: &'static str) {
        let train = self.trains.remove(name);
        if let Some(mut train) = train {
            train.depart(self);
        } else {
            println!("'{}' is not on the station!", name);
        }
    }
}
```

#### **trains/mod.rs**

```rust
mod freight_train;
mod passenger_train;

pub use freight_train::FreightTrain;
pub use passenger_train::PassengerTrain;

use crate::train_station::Mediator;

// A train gets a mediator object by reference.
pub trait Train {
    fn name(&self) -> &String;
    fn arrive(&mut self, mediator: &mut dyn Mediator);
    fn depart(&mut self, mediator: &mut dyn Mediator);
}
```

#### **trains/freight_train.rs**

```rust
use super::Train;
use crate::train_station::Mediator;

pub struct FreightTrain {
    name: String,
}

impl FreightTrain {
    pub fn new(name: &'static str) -> Self {
        Self { name: name.into() }
    }
}

impl Train for FreightTrain {
    fn name(&self) -> &String {
        &self.name
    }

    fn arrive(&mut self, mediator: &mut dyn Mediator) {
        if !mediator.notify_about_arrival(&self.name) {
            println!("Freight train {}: Arrival blocked, waiting", self.name);
            return;
        }

        println!("Freight train {}: Arrived", self.name);
    }

    fn depart(&mut self, mediator: &mut dyn Mediator) {
        println!("Freight train {}: Leaving", self.name);
        mediator.notify_about_departure(&self.name);
    }
}

```

#### **trains/passenger_train.rs**

```rust
use super::Train;
use crate::train_station::Mediator;

pub struct PassengerTrain {
    name: String,
}

impl PassengerTrain {
    pub fn new(name: &'static str) -> Self {
        Self { name: name.into() }
    }
}

impl Train for PassengerTrain {
    fn name(&self) -> &String {
        &self.name
    }

    fn arrive(&mut self, mediator: &mut dyn Mediator) {
        if !mediator.notify_about_arrival(&self.name) {
            println!("Passenger train {}: Arrival blocked, waiting", self.name);
            return;
        }

        println!("Passenger train {}: Arrived", self.name);
    }

    fn depart(&mut self, mediator: &mut dyn Mediator) {
        println!("Passenger train {}: Leaving", self.name);
        mediator.notify_about_departure(&self.name);
    }
}
```

#### main.rs：客户端代码

```rust
mod train_station;
mod trains;

use train_station::TrainStation;
use trains::{FreightTrain, PassengerTrain};

fn main() {
    let train1 = PassengerTrain::new("Train 1");
    let train2 = FreightTrain::new("Train 2");

    // Station has `accept` and `depart` methods,
    // but it also implements `Mediator`.
    let mut station = TrainStation::default();

    // Station is taking ownership of the trains.
    station.accept(train1);
    station.accept(train2);

    // `train1` and `train2` have been moved inside,
    // but we can use train names to depart them.
    station.depart("Train 1");
    station.depart("Train 2");
    station.depart("Train 3");
}
```

### 输出

````
Passenger train Train 1: Arrived
Freight train Train 2: Arrival blocked, waiting
Passenger train Train 1: Leaving
Freight train Train 2: Arrived
Freight train Train 2: Leaving
'Train 3' is not on the station!
````

