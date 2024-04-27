---
title: Observer 观察者模式
categories:
  - design-pattern
date: 2024-04-21 10:53:27
---

也称为：Event-Subscriber、Listener

## Intent 意图

观察者是一种行为设计模式，可用于定义订阅机制，以通知多个对象它们正在观察的对象发生的任何事件。


<div align="center"> <img src="/images/observer-header.png"/></div>


##  Problem 问题

假设您有两种类型的对象：a 假设您有两种类型的对象：a `Customer` 和 a  和 a  和 a `Store` 。客户对特定品牌的产品（例如，它是iPhone的新型号）非常感兴趣，该产品应该很快就会在商店中上市。 。客户对特定品牌的产品（例如，它是iPhone的新型号）非常感兴趣，该产品应该很快就会在商店中上市。

客户可以每天访问商店并检查产品可用性。但是，虽然产品仍在途中，但这些旅行中的大多数都是毫无意义的。


<div align="center"> <img src="/images/observer-comic1.png"/>访问商店与发送垃圾邮件</div>


另一方面，每次有新产品上市时，商店都会向所有客户发送大量电子邮件（可能被视为垃圾邮件）。这将使一些客户免于无休止地前往商店。同时，它会让其他对新产品不感兴趣的客户感到不安。

看起来我们发生了冲突。客户要么浪费时间检查产品可用性，要么商店浪费资源通知错误的客户。

##  Solution 解决方案

具有某种有趣状态的对象通常称为 subject，但由于它也会通知其他对象有关其状态的更改，因此我们将其称为 publisher。要跟踪发布者状态更改的所有其他对象称为订阅者。

Observer 模式建议向发布者类添加订阅机制，以便各个对象可以订阅或取消订阅来自该发布者的事件流。不要害怕！一切都不像听起来那么复杂。实际上，这种机制包括 1） 一个数组字段，用于存储对订阅服务器对象的引用列表，以及 2） 几个公共方法，这些方法允许在该列表中添加订阅者和从该列表中删除订阅者。observer-solution1


<div align="center"> <img src="/images/observer-solution1.png"/>订阅机制允许单个对象订阅事件通知。</div>


现在，每当发布者发生重要事件时，它都会遍历其订阅者，并对其对象调用特定的通知方法。

实际应用可能有数十个不同的订阅者类别，这些订阅者类别对跟踪同一发布商类别的事件感兴趣。您不希望将发布者与所有这些类耦合。此外，如果您的发布者类应该被其他人使用，您甚至可能事先不知道其中的一些。

这就是为什么所有订阅者都实现相同的接口，并且发布者仅通过该接口与他们通信至关重要的原因。此接口应声明通知方法以及一组参数，发布者可以使用这些参数将一些上下文数据与通知一起传递。


<div align="center"> <img src="/images/observer-solution2.png"/>Publisher 通过对订阅者的对象调用特定的通知方法来通知订阅者。</div>


如果你的应用有几种不同类型的发布者，并且你想让你的订阅者与所有这些发布者兼容，你可以更进一步，让所有发布者都遵循相同的界面。此接口只需要描述几种订阅方法。该接口将允许订阅者观察发布者的状态，而无需与他们的具体类耦合。

## Real-World Analogy 真实世界的类比


<div align="center"> <img src="/images/observer-comic2.png"/>杂志和报纸订阅。</div>


如果您订阅了报纸或杂志，则不再需要去商店查看下一期是否有货。相反，发布者会在发布后立即甚至提前将新问题直接发送到您的邮箱。

出版商维护一个订阅者列表，并知道他们对哪些杂志感兴趣。当订阅者希望阻止出版商向他们发送新杂志时，他们可以随时离开列表。

## Structure 结构


<div align="center"> <img src="/images/observer-structure1.png"/></div>


1. 发布服务器向其他对象发出感兴趣的事件。当发布者更改其状态或执行某些行为时，会发生这些事件。发布者包含一个订阅基础结构，允许新订阅者加入，而当前订阅者退出列表。
2. 当新事件发生时，发布者将遍历订阅列表，并调用在每个订阅者对象的订阅者界面中声明的通知方法。
3. 订阅者接口声明通知接口。在大多数情况下，它由单一 订阅者接口声明通知接口。在大多数情况下，它由单一 `update` 方法组成。该方法可能具有多个参数，这些参数允许发布者在更新时传递一些事件详细信息。 方法组成。该方法可能具有多个参数，这些参数允许发布者在更新时传递一些事件详细信息。
4. 具体订阅者执行一些操作以响应发布者发出的通知。所有这些类都必须实现相同的接口，这样发布者就不会耦合到具体的类。
5. 通常，订阅者需要一些上下文信息才能正确处理更新。因此，发布者通常会将一些上下文数据作为通知方法的参数传递。发布者可以将自身作为参数传递，让订阅者直接获取任何所需的数据。
6. 客户端分别创建发布者和订阅者对象，然后为发布者更新注册订阅者。

##  Pseudocode 伪代码

在此示例中，Observer 模式允许文本编辑器对象通知其他服务对象有关其状态的更改。


<div align="center"> <img src="/images/observer-example1.png"/>将其他对象发生的事件通知对象。</div>


订阅者列表是动态编译的：对象可以在运行时启动或停止侦听通知，具体取决于应用的所需行为。

在此实现中，编辑器类本身不维护订阅列表。它将此工作委托给专门用于此工作的特殊帮助程序对象。您可以将该对象升级为集中式事件调度器，让任何对象充当发布者。

向计划添加新订阅者不需要更改现有发布者类，只要它们通过同一界面与所有订阅者协作即可。

```java
// The base publisher class includes subscription management
// code and notification methods.
class EventManager is
    private field listeners: hash map of event types and listeners

    method subscribe(eventType, listener) is
        listeners.add(eventType, listener)

    method unsubscribe(eventType, listener) is
        listeners.remove(eventType, listener)

    method notify(eventType, data) is
        foreach (listener in listeners.of(eventType)) do
            listener.update(data)

// The concrete publisher contains real business logic that's
// interesting for some subscribers. We could derive this class
// from the base publisher, but that isn't always possible in
// real life because the concrete publisher might already be a
// subclass. In this case, you can patch the subscription logic
// in with composition, as we did here.
class Editor is
    public field events: EventManager
    private field file: File

    constructor Editor() is
        events = new EventManager()

    // Methods of business logic can notify subscribers about
    // changes.
    method openFile(path) is
        this.file = new File(path)
        events.notify("open", file.name)

    method saveFile() is
        file.write()
        events.notify("save", file.name)

    // ...


// Here's the subscriber interface. If your programming language
// supports functional types, you can replace the whole
// subscriber hierarchy with a set of functions.
interface EventListener is
    method update(filename)

// Concrete subscribers react to updates issued by the publisher
// they are attached to.
class LoggingListener implements EventListener is
    private field log: File
    private field message: string

    constructor LoggingListener(log_filename, message) is
        this.log = new File(log_filename)
        this.message = message

    method update(filename) is
        log.write(replace('%s',filename,message))

class EmailAlertsListener implements EventListener is
    private field email: string
    private field message: string

    constructor EmailAlertsListener(email, message) is
        this.email = email
        this.message = message

    method update(filename) is
        system.email(email, replace('%s',filename,message))


// An application can configure publishers and subscribers at
// runtime.
class Application is
    method config() is
        editor = new Editor()

        logger = new LoggingListener(
            "/path/to/log.txt",
            "Someone has opened the file: %s")
        editor.events.subscribe("open", logger)

        emailAlerts = new EmailAlertsListener(
            "admin@example.com",
            "Someone has changed the file: %s")
        editor.events.subscribe("save", emailAlerts)
```

## Applicability 适用性

- 当对一个对象的状态的更改可能需要更改其他对象，并且实际的对象集事先未知或动态更改时，请使用 Observer 模式。
- 在使用图形用户界面的类时，经常会遇到此问题。例如，您创建了自定义按钮类，并且希望让客户端将一些自定义代码挂接到您的按钮上，以便在用户按下按钮时触发该代码。

  Observer 模式允许任何实现订阅者接口的对象订阅发布者对象中的事件通知。您可以将订阅机制添加到按钮中，让客户端通过自定义订阅者类挂接其自定义代码。

- 当应用中的某些对象必须观察其他对象时，请使用该模式，但仅限于有限时间或特定情况。
- 订阅列表是动态的，因此订阅者可以随时加入或退出列表。

## How to Implement 如何实现

1. 查看您的业务逻辑并尝试将其分解为两部分：独立于其他代码的核心功能将充当发布者;其余的将变成一组订阅者类。
2. 声明订阅者接口。至少，它应该声明一个 声明订阅者接口。至少，它应该声明一个 `update` 方法。 方法。
3. 声明发布者接口，并描述一对用于向列表中添加订阅服务器对象并将其从列表中删除的方法。请记住，发布者只能通过订阅者界面与订阅者合作。
4. 确定将实际订阅列表和订阅方法的实现放在何处。通常，对于所有类型的发布者，此代码看起来都相同，因此将其放在直接从发布者接口派生的抽象类中。具体发布者扩展该类，继承订阅行为。

   但是，如果要将该模式应用于现有类层次结构，请考虑一种基于组合的方法：将订阅逻辑放入一个单独的对象中，并让所有真正的发布者都使用它。

5. 创建具体的发布者类。每当发布者内部发生重要事件时，它都必须通知其所有订阅者。
6. 在具体的订阅服务器类中实现更新通知方法。大多数订阅者需要有关事件的一些上下文数据。它可以作为通知方法的参数传递。

   但还有另一种选择。收到通知后，订阅者可以直接从通知中获取任何数据。在这种情况下，发布者必须通过 update 方法传递自身。不太灵活的选项是通过构造函数将发布者永久链接到订阅者。

7. 客户端必须创建所有必要的订阅者，并将其注册到适当的发布者。

## Pros and Cons 优点和缺点

| 优点√                                                                                     | 缺点×                        |
| ----------------------------------------------------------------------------------------- | ---------------------------- |
| 开/闭原则。可以引入新的订阅者类，而无需更改发布者的代码（如果有发布者接口，则反之亦然）。 | 订阅者将按随机顺序收到通知。 |
| 您可以在运行时建立对象之间的关系。                                                        |                              |

## Relations with Other Patterns  
与其他模式的关系

- 责任链、指挥、调解和观察者解决了连接请求发送者和接收者的各种方式：

  - 责任链沿着潜在接收方的动态链按顺序传递请求，直到其中一个接收方处理该请求。
  - 命令在发送方和接收方之间建立单向连接。
  - Mediator 消除了发送方和接收方之间的直接连接，迫使它们通过 Mediator 对象进行间接通信。
  - Observer 允许接收方动态订阅和取消订阅接收请求。

- 调解员和观察者之间的区别往往是难以捉摸的。在大多数情况下，您可以实现其中任一模式;但有时您可以同时应用两者。让我们看看如何做到这一点。

  Mediator 的主要目标是消除一组系统组件之间的相互依赖关系。相反，这些组件依赖于单个中介对象。Observer 的目标是在对象之间建立动态的单向连接，其中某些对象充当其他对象的从属。

  有一种依赖于 Observer 的 Mediator 模式的流行实现。中介对象扮演发布者的角色，组件充当订阅者，订阅和取消订阅中介的事件。当 Mediator 以这种方式实现时，它可能看起来与 Observer 非常相似。

  当您感到困惑时，请记住，您可以通过其他方式实现 Mediator 模式。例如，您可以将所有组件永久链接到同一个中介对象。此实现与 Observer 不同，但仍是 Mediator 模式的实例。

  现在想象一个程序，其中所有组件都已成为发布者，允许彼此之间的动态连接。不会有一个集中的中介对象，只有一组分布式的观察者。


# **Observer** in Python 

观察者是一种行为设计模式，它允许某些对象通知其他对象有关其状态的更改。

Observer 模式提供了一种为实现订阅者接口的任何对象订阅和取消订阅这些事件的方法。

使用示例：Observer 模式在 Python 代码中很常见，尤其是在 GUI 组件中。它提供了一种对其他对象中发生的事件做出反应的方法，而无需耦合到它们的类。

标识：可以通过订阅方法（将对象存储在列表中）以及调用向该列表中的对象发出的更新方法来识别该模式。

## Conceptual Example 概念示例

此示例说明了 Observer 设计模式的结构。它侧重于回答以下问题：

- 它由哪些类组成？
- 这些课程扮演什么角色？
- 模式的元素以何种方式相关？

#### main.py：概念示例

```python
from __future__ import annotations
from abc import ABC, abstractmethod
from random import randrange
from typing import List


class Subject(ABC):
    """
    The Subject interface declares a set of methods for managing subscribers.
    """

    @abstractmethod
    def attach(self, observer: Observer) -> None:
        """
        Attach an observer to the subject.
        """
        pass

    @abstractmethod
    def detach(self, observer: Observer) -> None:
        """
        Detach an observer from the subject.
        """
        pass

    @abstractmethod
    def notify(self) -> None:
        """
        Notify all observers about an event.
        """
        pass


class ConcreteSubject(Subject):
    """
    The Subject owns some important state and notifies observers when the state
    changes.
    """

    _state: int = None
    """
    For the sake of simplicity, the Subject's state, essential to all
    subscribers, is stored in this variable.
    """

    _observers: List[Observer] = []
    """
    List of subscribers. In real life, the list of subscribers can be stored
    more comprehensively (categorized by event type, etc.).
    """

    def attach(self, observer: Observer) -> None:
        print("Subject: Attached an observer.")
        self._observers.append(observer)

    def detach(self, observer: Observer) -> None:
        self._observers.remove(observer)

    """
    The subscription management methods.
    """

    def notify(self) -> None:
        """
        Trigger an update in each subscriber.
        """

        print("Subject: Notifying observers...")
        for observer in self._observers:
            observer.update(self)

    def some_business_logic(self) -> None:
        """
        Usually, the subscription logic is only a fraction of what a Subject can
        really do. Subjects commonly hold some important business logic, that
        triggers a notification method whenever something important is about to
        happen (or after it).
        """

        print("\nSubject: I'm doing something important.")
        self._state = randrange(0, 10)

        print(f"Subject: My state has just changed to: {self._state}")
        self.notify()


class Observer(ABC):
    """
    The Observer interface declares the update method, used by subjects.
    """

    @abstractmethod
    def update(self, subject: Subject) -> None:
        """
        Receive update from subject.
        """
        pass


"""
Concrete Observers react to the updates issued by the Subject they had been
attached to.
"""


class ConcreteObserverA(Observer):
    def update(self, subject: Subject) -> None:
        if subject._state < 3:
            print("ConcreteObserverA: Reacted to the event")


class ConcreteObserverB(Observer):
    def update(self, subject: Subject) -> None:
        if subject._state == 0 or subject._state >= 2:
            print("ConcreteObserverB: Reacted to the event")


if __name__ == "__main__":
    # The client code.

    subject = ConcreteSubject()

    observer_a = ConcreteObserverA()
    subject.attach(observer_a)

    observer_b = ConcreteObserverB()
    subject.attach(observer_b)

    subject.some_business_logic()
    subject.some_business_logic()

    subject.detach(observer_a)

    subject.some_business_logic()

```

#### **Output.txt:** Execution result

````
Subject: Attached an observer.
Subject: Attached an observer.

Subject: I'm doing something important.
Subject: My state has just changed to: 0
Subject: Notifying observers...
ConcreteObserverA: Reacted to the event
ConcreteObserverB: Reacted to the event

Subject: I'm doing something important.
Subject: My state has just changed to: 5
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event

Subject: I'm doing something important.
Subject: My state has just changed to: 0
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event
````

# **Observer** in Rust

观察者是一种行为设计模式，它允许某些对象通知其他对象有关其状态的更改。

Observer 模式提供了一种为实现订阅者接口的任何对象订阅和取消订阅这些事件的方法。

## Conceptual example 概念示例

在 Rust 中，定义订阅者的一种便捷方法是将函数作为具有复杂逻辑的可调用对象传递给事件发布者。

在此 Observer 示例中，订阅者是订阅事件的 lambda 函数或显式函数。显式函数对象也可以取消订阅（尽管某些函数类型可能存在限制）。

#### **editor.rs**

```rust
use crate::observer::{Event, Publisher};

/// Editor has its own logic and it utilizes a publisher
/// to operate with subscribers and events.
#[derive(Default)]
pub struct Editor {
    publisher: Publisher,
    file_path: String,
}

impl Editor {
    pub fn events(&mut self) -> &mut Publisher {
        &mut self.publisher
    }

    pub fn load(&mut self, path: String) {
        self.file_path = path.clone();
        self.publisher.notify(Event::Load, path);
    }

    pub fn save(&self) {
        self.publisher.notify(Event::Save, self.file_path.clone());
    }
}

```

####  **observer.rs**

```rust
use std::collections::HashMap;

/// An event type.
#[derive(PartialEq, Eq, Hash, Clone)]
pub enum Event {
    Load,
    Save,
}

/// A subscriber (listener) has type of a callable function.
pub type Subscriber = fn(file_path: String);

/// Publisher sends events to subscribers (listeners).
#[derive(Default)]
pub struct Publisher {
    events: HashMap<Event, Vec<Subscriber>>,
}

impl Publisher {
    pub fn subscribe(&mut self, event_type: Event, listener: Subscriber) {
        self.events.entry(event_type.clone()).or_default();
        self.events.get_mut(&event_type).unwrap().push(listener);
    }

    pub fn unsubscribe(&mut self, event_type: Event, listener: Subscriber) {
        self.events
            .get_mut(&event_type)
            .unwrap()
            .retain(|&x| x != listener);
    }

    pub fn notify(&self, event_type: Event, file_path: String) {
        let listeners = self.events.get(&event_type).unwrap();
        for listener in listeners {
            listener(file_path.clone());
        }
    }
}

```

####  **main.rs**

```rust
use editor::Editor;
use observer::Event;

mod editor;
mod observer;

fn main() {
    let mut editor = Editor::default();

    editor.events().subscribe(Event::Load, |file_path| {
        let log = "/path/to/log/file.txt".to_string();
        println!("Save log to {}: Load file {}", log, file_path);
    });

    editor.events().subscribe(Event::Save, save_listener);

    editor.load("test1.txt".into());
    editor.load("test2.txt".into());
    editor.save();

    editor.events().unsubscribe(Event::Save, save_listener);
    editor.save();
}

fn save_listener(file_path: String) {
    let email = "admin@example.com".to_string();
    println!("Email to {}: Save file {}", email, file_path);
}

```

### Output 输出

````
Save log to /path/to/log/file.txt: Load file test1.txt
Save log to /path/to/log/file.txt: Load file test2.txt
Email to admin@example.com: Save file test2.txt
````

