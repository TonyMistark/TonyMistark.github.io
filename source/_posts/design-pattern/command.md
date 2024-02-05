---
title: Command 命令
categories:
  - design-pattern
tags: Design Pattern
date: 2024-02-05 21:36:28
---

## 意图

Command是一种行为设计模式，它将请求转换为包含有关请求的所有信息的独立对象。这种转换允许您将请求作为方法参数传递，延迟或排队请求的执行，并支持可撤消的操作。


<div align="center"> <img src="/images/command-header.png"/></div>


## 问题

假设您正在开发一个新的文本编辑器应用程序，您当前的任务是创建一个工具栏，其中包含一系列用于编辑器各种操作的按钮。您创建了一个非常简洁的 假设您正在开发一个新的文本编辑器应用程序，您当前的任务是创建一个工具栏，其中包含一系列用于编辑器各种操作的按钮。您创建了一个非常简洁的 `Button` 类，可用于工具栏上的按钮，以及各种对话框中的通用按钮。 类，可用于工具栏上的按钮，以及各种对话框中的通用按钮。


<div align="center"> <img src="/images/command-problem1.png"/>应用程序的所有按钮都派生自同一个类。</div>


虽然所有这些按钮看起来都很相似，但它们都应该做不同的事情。您将把这些按钮的各种单击处理程序的代码放在哪里？最简单的解决方案是为每个使用按钮的地方创建大量的子类。这些子类将包含必须在单击按钮时执行的代码。


<div align="center"> <img src="/images/command-problem2.png"/>应用程序的所有按钮都派生自同一个类。</div>


不久，你就会意识到这种方法有很大的缺陷。首先，您有大量的子类，如果您每次修改基类 不久，你就会意识到这种方法有很大的缺陷。首先，您有大量的子类，如果您每次修改基类 `Button` 时不会有破坏这些子类中代码的风险，那么这是可以的。简单地说，GUI代码已经变得笨拙地依赖于业务逻辑的易变代码。 时不会有破坏这些子类中代码的风险，那么这是可以的。简单地说，GUI代码已经变得笨拙地依赖于业务逻辑的易变代码。


<div align="center"> <img src="/images/command-problem3.png"/>应用程序的所有按钮都派生自同一个类。</div>


最丑陋的是。某些操作，如复制/粘贴文本，需要从多个位置调用。例如，用户可以点击工具栏上的一个小的“复制”按钮，或者通过上下文菜单复制一些东西，或者只是点击键盘上的 最丑陋的是。某些操作，如复制/粘贴文本，需要从多个位置调用。例如，用户可以点击工具栏上的一个小的“复制”按钮，或者通过上下文菜单复制一些东西，或者只是点击键盘上的 `Ctrl+C` 。 

最初，当我们的应用程序只有工具栏时，可以将各种操作的实现放在按钮子类中。换句话说，在 最初，当我们的应用程序只有工具栏时，可以将各种操作的实现放在按钮子类中。换句话说，在 `CopyButton` 子类中复制文本的代码是可以的。但是，当您实现上下文菜单、快捷方式和其他东西时，您必须在许多类中复制操作代码，或者使菜单依赖于按钮，这是一个更糟糕的选择。 子类中复制文本的代码是可以的。但是，当您实现上下文菜单、快捷方式和其他东西时，您必须在许多类中复制操作代码，或者使菜单依赖于按钮，这是一个更糟糕的选择。

## 解决方案

好的软件设计通常基于关注点分离的原则，这通常会导致将应用程序分解为多个层。最常见的例子：一层用于图形用户界面，另一层用于业务逻辑。GUI层负责在屏幕上呈现美丽的图片，捕获任何输入并显示用户和应用程序正在执行的操作的结果。然而，当涉及到做一些重要的事情时，比如计算月球的轨迹或撰写年度报告，GUI层将工作委托给业务逻辑的底层。

在代码中，它可能看起来像这样：GUI对象调用业务逻辑对象的方法，并向其传递一些参数。这个过程通常被描述为一个对象向另一个对象发送请求。


<div align="center"> <img src="/images/command-solution1.png"/>GUI对象可以直接访问业务逻辑对象。</div>


命令模式建议GUI对象不应该直接发送这些请求。相反，您应该提取所有请求的详细信息，例如被调用的对象，方法的名称和参数列表到一个单独的命令类中，并使用一个触发此请求的方法。

命令对象充当各种GUI和业务逻辑对象之间的链接。从现在开始，GUI对象不需要知道什么业务逻辑对象将接收请求以及如何处理请求。GUI对象只是触发命令，该命令处理所有细节。


<div align="center"> <img src="/images/command-solution2.png"/>通过命令调用业务逻辑层。</div>


下一步是使您的命令实现相同的接口。通常它只有一个不带参数的执行方法。这个接口允许您对同一个请求发送者使用不同的命令，而无需将其耦合到具体的命令类。作为奖励，现在您可以切换链接到发送者的命令对象，有效地改变发送者在运行时的行为。

您可能已经注意到了这个难题中缺少的一个部分，即请求参数。GUI对象可能已经为业务层对象提供了一些参数。由于命令执行方法没有任何参数，我们如何将请求细节传递给接收方？事实证明，该命令应该预先配置了这些数据，或者能够自己获取这些数据。


<div align="center"> <img src="/images/command-solution3.png"/>通过命令调用业务逻辑层。</div>


让我们回到我们的文本编辑器。在我们应用Command模式之后，我们不再需要所有那些按钮子类来实现各种单击行为。将一个字段放入存储命令对象引用的基类 让我们回到我们的文本编辑器。在我们应用Command模式之后，我们不再需要所有那些按钮子类来实现各种单击行为。将一个字段放入存储命令对象引用的基类 `Button` 中，并使按钮在单击时执行该命令就足够了。 中，并使按钮在单击时执行该命令就足够了。

您将为每个可能的操作实现一组命令类，并根据按钮的预期行为将它们与特定按钮链接。

其他GUI元素，如菜单、快捷方式或整个对话框，也可以用同样的方式实现。它们将被链接到一个命令，当用户与GUI元素交互时，该命令将被执行。正如您现在可能已经猜到的，与相同操作相关的元素将链接到相同的命令，从而防止任何代码重复。

因此，命令成为一个方便的中间层，减少了GUI和业务逻辑层之间的耦合。而这只是命令模式所能提供的好处的一小部分！

## 现实世界的类比


<div align="center"> <img src="/images/command-comic1.png"/>在餐馆点菜。</div>


在城市里走了很长一段路后，你来到一家不错的餐馆，坐在靠窗的桌子旁。一个友好的服务员走近你，迅速地把你的订单写在一张纸上。服务员走到厨房，把菜单贴在墙上。过了一段时间，订单到达厨师，谁读它和烹饪相应的饭菜。厨师把饭菜和点菜单一起放在托盘上沿着。服务员发现托盘，检查订单，以确保一切都是你想要的，并把一切都带到你的桌子上。

纸上的命令是命令。在厨师准备上菜之前，它一直处于排队状态。订单包含了烹饪这顿饭所需的所有相关信息。它允许厨师立即开始烹饪，而不是跑来跑去直接向您澄清订单细节。

## 结构


<div align="center"> <img src="/images/command-structure.png"/></div>


1. 类（也称为invoker）负责发起请求。此类必须有一个用于存储对命令对象的引用的字段。发送方触发该命令，而不是直接向接收方发送请求。请注意，发送方不负责创建命令对象。通常，它通过构造函数从客户端获取预先创建的命令。
2. Command接口通常只声明一个执行命令的方法。
3. 在接收对象上执行方法所需的参数可以在具体命令中声明为字段。通过只允许通过构造函数初始化这些字段，可以使命令对象不可变。
4. Receiver类包含一些业务逻辑。几乎任何物体都可以充当接收器。大多数命令只处理如何将请求传递给接收方的细节，而接收方本身则执行实际工作。
5. 客户端创建和配置具体的命令对象。客户端必须将所有请求参数（包括接收器实例）传递到命令的构造函数中。在此之后，所得到的命令可以与一个或多个命令相关联。

## 伪代码

在本例中，Command模式有助于跟踪已执行操作的历史记录，并在需要时恢复操作。


<div align="center"> <img src="/images/command-example.png"/>文本编辑器中可撤消的操作。</div>


导致更改编辑器状态的命令（例如，剪切和粘贴）在执行与该命令相关联的操作之前制作编辑器状态的备份副本。在命令执行之后，它将与编辑器当时状态的备份副本一起沿着放置到命令历史记录（命令对象的堆栈）中。稍后，如果用户需要恢复操作，应用可以从历史记录中获取最新的命令，读取编辑器状态的相关备份，然后将其恢复。

客户端代码（GUI元素、命令历史等）没有耦合到具体的命令类，因为它通过命令接口处理命令。这种方法允许您将新命令引入到应用程序中，而不会破坏任何现有代码。

```java
// The base command class defines the common interface for all
// concrete commands.
abstract class Command is
    protected field app: Application
    protected field editor: Editor
    protected field backup: text

    constructor Command(app: Application, editor: Editor) is
        this.app = app
        this.editor = editor

    // Make a backup of the editor's state.
    method saveBackup() is
        backup = editor.text

    // Restore the editor's state.
    method undo() is
        editor.text = backup

    // The execution method is declared abstract to force all
    // concrete commands to provide their own implementations.
    // The method must return true or false depending on whether
    // the command changes the editor's state.
    abstract method execute()


// The concrete commands go here.
class CopyCommand extends Command is
    // The copy command isn't saved to the history since it
    // doesn't change the editor's state.
    method execute() is
        app.clipboard = editor.getSelection()
        return false

class CutCommand extends Command is
    // The cut command does change the editor's state, therefore
    // it must be saved to the history. And it'll be saved as
    // long as the method returns true.
    method execute() is
        saveBackup()
        app.clipboard = editor.getSelection()
        editor.deleteSelection()
        return true

class PasteCommand extends Command is
    method execute() is
        saveBackup()
        editor.replaceSelection(app.clipboard)
        return true

// The undo operation is also a command.
class UndoCommand extends Command is
    method execute() is
        app.undo()
        return false


// The global command history is just a stack.
class CommandHistory is
    private field history: array of Command

    // Last in...
    method push(c: Command) is
        // Push the command to the end of the history array.

    // ...first out
    method pop():Command is
        // Get the most recent command from the history.


// The editor class has actual text editing operations. It plays
// the role of a receiver: all commands end up delegating
// execution to the editor's methods.
class Editor is
    field text: string

    method getSelection() is
        // Return selected text.

    method deleteSelection() is
        // Delete selected text.

    method replaceSelection(text) is
        // Insert the clipboard's contents at the current
        // position.


// The application class sets up object relations. It acts as a
// sender: when something needs to be done, it creates a command
// object and executes it.
class Application is
    field clipboard: string
    field editors: array of Editors
    field activeEditor: Editor
    field history: CommandHistory

    // The code which assigns commands to UI objects may look
    // like this.
    method createUI() is
        // ...
        copy = function() { executeCommand(
            new CopyCommand(this, activeEditor)) }
        copyButton.setCommand(copy)
        shortcuts.onKeyPress("Ctrl+C", copy)

        cut = function() { executeCommand(
            new CutCommand(this, activeEditor)) }
        cutButton.setCommand(cut)
        shortcuts.onKeyPress("Ctrl+X", cut)

        paste = function() { executeCommand(
            new PasteCommand(this, activeEditor)) }
        pasteButton.setCommand(paste)
        shortcuts.onKeyPress("Ctrl+V", paste)

        undo = function() { executeCommand(
            new UndoCommand(this, activeEditor)) }
        undoButton.setCommand(undo)
        shortcuts.onKeyPress("Ctrl+Z", undo)

    // Execute a command and check whether it has to be added to
    // the history.
    method executeCommand(command) is
        if (command.execute())
            history.push(command)

    // Take the most recent command from the history and run its
    // undo method. Note that we don't know the class of that
    // command. But we don't have to, since the command knows
    // how to undo its own action.
    method undo() is
        command = history.pop()
        if (command != null)
            command.undo()
```

## 适用性

- **如果要通过操作参数化对象，请使用Command模式。**
- Command模式可以将特定的方法调用转换为独立的对象。这一变化开启了许多有趣的用途：你可以将命令作为方法参数传递，将它们存储在其他对象中，在运行时切换链接的命令，等等。

  下面是一个示例：您正在开发一个GUI组件（如上下文菜单），并且希望您的用户能够配置菜单项，以便在最终用户单击某项时触发操作。

- 当您希望将操作排队、计划其执行或远程执行时，请使用Command模式。
- 与任何其他对象一样，命令可以序列化，这意味着将其转换为可以轻松写入文件或数据库的字符串。稍后，该字符串可以恢复为初始命令对象。因此，您可以延迟和调度命令的执行。但还有更多！以同样的方式，您可以通过网络排队，记录或发送命令。
- **当你想实现可逆操作时，使用命令模式。**
- 虽然有很多方法可以实现撤销/重做，但命令模式可能是最流行的。

  为了能够还原操作，您需要实现已执行操作的历史记录。命令历史记录是一个堆栈，其中包含所有已执行的命令对象沿着以及应用程序状态的相关备份。

  这种方法有两个缺点。首先，保存应用程序的状态并不容易，因为其中一些状态可能是私有的。这个问题可以通过Memento模式来缓解。

  其次，状态备份可能会消耗大量RAM。因此，有时您可以采用另一种实现方式：命令执行相反的操作，而不是恢复过去的状态。反向操作也有代价：它可能很难甚至不可能实施。


## 如何实现

1. 使用单个执行方法decompose命令接口。
2. 开始将请求提取到实现命令接口的具体命令类中。每个类都必须有一组字段，用于存储请求参数沿着对实际接收器对象的引用。所有这些值都必须通过命令的构造函数初始化。
3. 确定将充当代理的类。将用于存储命令的字段添加到这些类中。发送者应仅通过命令接口与其命令进行通信。发送方通常不会自己创建命令对象，而是从客户端代码中获取命令对象。
4. 更改发送方，使其执行命令，而不是直接向接收方发送请求。
5. 客户端应按以下顺序初始化对象：

   - 创建接收器。
   - 创建命令，并在需要时将它们与接收器相关联。
   - 创建命令行，并将它们与特定命令关联。


## 利弊

| √ 利                                                                      | × 弊                                                                   |
| ------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| 单一责任原则。可以将调用操作的类与执行这些操作的类解耦。                  | 代码可能会变得更加复杂，因为您在接收器和接收器之间引入了一个全新的层。 |
| 开放/封闭原则。您可以在应用程序中引入新命令，而不会破坏现有的客户端代码。 |                                                                        |
| 您可以执行undo/redo。                                                     |                                                                        |
| 您可以实现操作的延迟执行。                                                |                                                                        |
| 您可以将一组简单的命令组合成一个复杂的命令。                              |                                                                        |

## 与其他模式的关系

- 责任链、命令、调解器和观察者解决了连接请求的接收者和接收者的各种方式：

  - 责任链（Chain of Responsibility）将请求顺序地沿着一个动态的潜在接收者链传递，直到其中一个接收者处理它。
  - 命令在中继器和接收器之间建立单向连接。
  - Mediator消除了发送者和接收者之间的直接连接，迫使它们通过Mediator对象间接通信。
  - 观察者允许接收者动态订阅和取消订阅接收请求。

- 责任链中的处理程序可以作为命令实现。在这种情况下，您可以对同一个上下文对象（由请求表示）执行许多不同的操作。

  然而，还有另一种方法，其中请求本身是一个Command对象。在这种情况下，您可以在链接成链的一系列不同上下文中执行相同的操作。

- 你可以使用命令和Memento一起实现“撤消”。在这种情况下，命令负责对目标对象执行各种操作，而memento则在执行命令之前保存该对象的状态。
- Command和Strategy可能看起来很相似，因为您可以使用这两种方法通过某些操作来参数化对象。然而，他们有非常不同的意图。

  - 您可以使用Command将任何操作转换为对象。操作的参数成为该对象的字段。转换允许您延迟操作的执行，将其排队，存储命令的历史记录，将命令发送到远程服务等。
  - 另一方面，Strategy通常描述做同一件事的不同方法，让你在一个上下文类中交换这些算法。

- 当您需要将命令的副本保存到历史记录中时，Prototype可以提供帮助。
- 您可以将Visitor视为Command模式的强大版本。它的对象可以在不同类的各种对象上执行操作。

# Python中的Command

命令是一种行为设计模式，它将请求或简单操作转换为对象。

转换允许延迟或远程执行命令，存储命令历史等。

## 概念示例

这个例子说明了命令设计模式的结构。它侧重于回答这些问题：

- 它由哪些类组成？
- 这些班级扮演什么角色？
- 模式中的元素是以什么方式联系在一起的？

#### main.py：概念性示例

```python
from __future__ import annotations
from abc import ABC, abstractmethod


class Command(ABC):
    """
    The Command interface declares a method for executing a command.
    """

    @abstractmethod
    def execute(self) -> None:
        pass


class SimpleCommand(Command):
    """
    Some commands can implement simple operations on their own.
    """

    def __init__(self, payload: str) -> None:
        self._payload = payload

    def execute(self) -> None:
        print(f"SimpleCommand: See, I can do simple things like printing"
              f"({self._payload})")


class ComplexCommand(Command):
    """
    However, some commands can delegate more complex operations to other
    objects, called "receivers."
    """

    def __init__(self, receiver: Receiver, a: str, b: str) -> None:
        """
        Complex commands can accept one or several receiver objects along with
        any context data via the constructor.
        """

        self._receiver = receiver
        self._a = a
        self._b = b

    def execute(self) -> None:
        """
        Commands can delegate to any methods of a receiver.
        """

        print("ComplexCommand: Complex stuff should be done by a receiver object", end="")
        self._receiver.do_something(self._a)
        self._receiver.do_something_else(self._b)


class Receiver:
    """
    The Receiver classes contain some important business logic. They know how to
    perform all kinds of operations, associated with carrying out a request. In
    fact, any class may serve as a Receiver.
    """

    def do_something(self, a: str) -> None:
        print(f"\nReceiver: Working on ({a}.)", end="")

    def do_something_else(self, b: str) -> None:
        print(f"\nReceiver: Also working on ({b}.)", end="")


class Invoker:
    """
    The Invoker is associated with one or several commands. It sends a request
    to the command.
    """

    _on_start = None
    _on_finish = None

    """
    Initialize commands.
    """

    def set_on_start(self, command: Command):
        self._on_start = command

    def set_on_finish(self, command: Command):
        self._on_finish = command

    def do_something_important(self) -> None:
        """
        The Invoker does not depend on concrete command or receiver classes. The
        Invoker passes a request to a receiver indirectly, by executing a
        command.
        """

        print("Invoker: Does anybody want something done before I begin?")
        if isinstance(self._on_start, Command):
            self._on_start.execute()

        print("Invoker: ...doing something really important...")

        print("Invoker: Does anybody want something done after I finish?")
        if isinstance(self._on_finish, Command):
            self._on_finish.execute()


if __name__ == "__main__":
    """
    The client code can parameterize an invoker with any commands.
    """

    invoker = Invoker()
    invoker.set_on_start(SimpleCommand("Say Hi!"))
    receiver = Receiver()
    invoker.set_on_finish(ComplexCommand(
        receiver, "Send email", "Save report"))

    invoker.do_something_important()
```

#### Output.txt：执行结果

````
Invoker: Does anybody want something done before I begin?
SimpleCommand: See, I can do simple things like printing (Say Hi!)
Invoker: ...doing something really important...
Invoker: Does anybody want something done after I finish?
ComplexCommand: Complex stuff should be done by a receiver object
Receiver: Working on (Send email.)
Receiver: Also working on (Save report.)
````

# **Command** in Rust

命令是一种行为设计模式，它将请求或简单操作转换为对象。

转换允许延迟或远程执行命令，存储命令历史等。

在Rust中，命令实例不应该持有对全局上下文的永久引用，相反，后者应该作为“ 在Rust中，命令实例不应该持有对全局上下文的永久引用，相反，后者应该作为“ `execute` “方法的可变参数从上到下传递： “方法的可变参数从上到下传递：

```rust
fn execute(&mut self, app: &mut cursive::Cursive) -> bool;
```

## 文本编辑器：命令和撤消

关键点：

- 每个按钮运行一个单独的命令。
- 由于命令被表示为对象，因此可以将其推入 由于命令被表示为对象，因此可以将其推入 `history` 数组，以便稍后撤消。 数组，以便稍后撤消。
- TUI使用 TUI使用 `cursive` crate创建。 crate创建。

#### command.rs：Command Interface

```rust
mod copy;
mod cut;
mod paste;

pub use copy::CopyCommand;
pub use cut::CutCommand;
pub use paste::PasteCommand;

/// Declares a method for executing (and undoing) a command.
///
/// Each command receives an application context to access
/// visual components (e.g. edit view) and a clipboard.
pub trait Command {
    fn execute(&mut self, app: &mut cursive::Cursive) -> bool;
    fn undo(&mut self, app: &mut cursive::Cursive);
}
```

#### command/copy.rs：复制命令

```rust
use cursive::{views::EditView, Cursive};

use super::Command;
use crate::AppContext;

#[derive(Default)]
pub struct CopyCommand;

impl Command for CopyCommand {
    fn execute(&mut self, app: &mut Cursive) -> bool {
        let editor = app.find_name::<EditView>("Editor").unwrap();
        let mut context = app.take_user_data::<AppContext>().unwrap();

        context.clipboard = editor.get_content().to_string();

        app.set_user_data(context);
        false
    }

    fn undo(&mut self, _: &mut Cursive) {}
}
```

#### command/cut.rs：剪切命令

```rust
use cursive::{views::EditView, Cursive};

use super::Command;
use crate::AppContext;

#[derive(Default)]
pub struct CutCommand {
    backup: String,
}

impl Command for CutCommand {
    fn execute(&mut self, app: &mut Cursive) -> bool {
        let mut editor = app.find_name::<EditView>("Editor").unwrap();

        app.with_user_data(|context: &mut AppContext| {
            self.backup = editor.get_content().to_string();
            context.clipboard = self.backup.clone();
            editor.set_content("".to_string());
        });

        true
    }

    fn undo(&mut self, app: &mut Cursive) {
        let mut editor = app.find_name::<EditView>("Editor").unwrap();
        editor.set_content(&self.backup);
    }
}
```

#### command/paste.rs：粘贴命令

```rust
use cursive::{views::EditView, Cursive};

use super::Command;
use crate::AppContext;

#[derive(Default)]
pub struct PasteCommand {
    backup: String,
}

impl Command for PasteCommand {
    fn execute(&mut self, app: &mut Cursive) -> bool {
        let mut editor = app.find_name::<EditView>("Editor").unwrap();

        app.with_user_data(|context: &mut AppContext| {
            self.backup = editor.get_content().to_string();
            editor.set_content(context.clipboard.clone());
        });

        true
    }

    fn undo(&mut self, app: &mut Cursive) {
        let mut editor = app.find_name::<EditView>("Editor").unwrap();
        editor.set_content(&self.backup);
    }
}
```

#### main.rs：客户端代码

```rust
mod command;

use cursive::{
    traits::Nameable,
    views::{Dialog, EditView},
    Cursive,
};

use command::{Command, CopyCommand, CutCommand, PasteCommand};

/// An application context to be passed into visual component callbacks.
/// It contains a clipboard and a history of commands to be undone.
#[derive(Default)]
struct AppContext {
    clipboard: String,
    history: Vec<Box<dyn Command>>,
}

fn main() {
    let mut app = cursive::default();

    app.set_user_data(AppContext::default());
    app.add_layer(
        Dialog::around(EditView::default().with_name("Editor"))
            .title("Type and use buttons")
            .button("Copy", |s| execute(s, CopyCommand::default()))
            .button("Cut", |s| execute(s, CutCommand::default()))
            .button("Paste", |s| execute(s, PasteCommand::default()))
            .button("Undo", undo)
            .button("Quit", |s| s.quit()),
    );

    app.run();
}

/// Executes a command and then pushes it to a history array.
fn execute(app: &mut Cursive, mut command: impl Command + 'static) {
    if command.execute(app) {
        app.with_user_data(|context: &mut AppContext| {
            context.history.push(Box::new(command));
        });
    }
}

/// Pops the last command and executes an undo action.
fn undo(app: &mut Cursive) {
    let mut context = app.take_user_data::<AppContext>().unwrap();
    if let Some(mut command) = context.history.pop() {
        command.undo(app)
    }
    app.set_user_data(context);
}
```

# Output 输出


<div align="center"> <img src="/images/command_output.png"/></div>


