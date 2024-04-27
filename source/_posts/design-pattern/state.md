---
title: State 状态模式
categories:
  - design-pattern
date: 2024-04-21 12:54:21
---

##  Intent 意图

状态是一种行为设计模式，它允许对象在其内部状态更改时更改其行为。看起来好像对象更改了它的类。


<div align="center"> <img src="/images/state-header.png"/><p style="text-align: center;"></p></div>


## Problem 问题

状态模式与有限状态机的概念密切相关。


<div align="center"> <img src="/images/state-problem1.png"/><p style="text-align: center;">Finite-State Machine. 有限状态机。</p> </div>


主要思想是，在任何给定的时刻，程序可以处于的状态数量是有限的。在任何唯一状态下，程序的行为都不同，程序可以立即从一种状态切换到另一种状态。但是，根据当前状态，程序可能会也可能不会切换到某些其他状态。这些切换规则（称为转换）也是有限的和预先确定的。

您也可以将此方法应用于对象。想象一下，我们有一个 您也可以将此方法应用于对象。想象一下，我们有一个 `Document` 班级。文档可以处于以下三种状态之一：  班级。文档可以处于以下三种状态之一：  班级。文档可以处于以下三种状态之一： `Draft` 、  、  、 `Moderation` 和  和  和 `Published` 。文档  。文档  。文档 `publish` 的方法在每种状态下的工作方式略有不同： 的方法在每种状态下的工作方式略有不同：

- 在 中 在 中 `Draft` ，它会将文档移动到审核状态。 ，它会将文档移动到审核状态。
- 在 中 在 中 `Moderation` ，它使文档公开，但前提是当前用户是管理员。 ，它使文档公开，但前提是当前用户是管理员。
- 在 在 `Published` 中，它根本不做任何事情。 中，它根本不做任何事情。


<div align="center"> <img src="/images/state-problem2.png"/><p style="text-align: center;">文档对象的可能状态和转换。</p></div>


状态机通常使用大量条件语句（ 状态机通常使用大量条件语句（ `if` 或  或  或 `switch` ）实现，这些条件语句根据对象的当前状态选择适当的行为。通常，此“状态”只是对象字段的一组值。即使你以前从未听说过有限状态机，你也可能至少实现过一次状态。以下代码结构是否敲响了警钟？ ）实现，这些条件语句根据对象的当前状态选择适当的行为。通常，此“状态”只是对象字段的一组值。即使你以前从未听说过有限状态机，你也可能至少实现过一次状态。以下代码结构是否敲响了警钟？

```java
class Document is
    field state: string
    // ...
    method publish() is
        switch (state)
            "draft":
                state = "moderation"
                break
            "moderation":
                if (currentUser.role == "admin")
                    state = "published"
                break
            "published":
                // Do nothing.
                break
    // ...
```

一旦我们开始向 一旦我们开始向 `Document` 类中添加越来越多的状态和状态依赖行为，基于条件的状态机的最大弱点就会显现出来。大多数方法都会包含可怕的条件，这些条件根据当前状态选择方法的正确行为。像这样的代码很难维护，因为对转换逻辑的任何更改都可能需要更改每个方法中的状态条件。 类中添加越来越多的状态和状态依赖行为，基于条件的状态机的最大弱点就会显现出来。大多数方法都会包含可怕的条件，这些条件根据当前状态选择方法的正确行为。像这样的代码很难维护，因为对转换逻辑的任何更改都可能需要更改每个方法中的状态条件。

随着项目的发展，问题往往会变得更大。在设计阶段预测所有可能的状态和转变是相当困难的。因此，随着时间的流逝，使用一组有限的条件构建的精益状态机可能会变得一团糟。

## Solution 解决方案

状态模式建议您为对象的所有可能状态创建新类，并将所有特定于状态的行为提取到这些类中。

原始对象（称为 context）不是自行实现所有行为，而是存储对表示其当前状态的状态对象之一的引用，并将所有与状态相关的工作委托给该对象。


<div align="center"> <img src="/images/state-solution.png"/><p style="text-align: center;">文档将工作委托给状态对象。</p></div>


若要将上下文转换为另一种状态，请将活动状态对象替换为表示该新状态的另一个对象。只有当所有状态类都遵循相同的接口，并且上下文本身通过该接口处理这些对象时，这才有可能。

此结构可能看起来类似于策略模式，但有一个关键区别。在状态模式中，特定状态可能相互了解并启动从一种状态到另一种状态的过渡，而策略几乎从不相互了解。

## Real-World Analogy 真实世界的类比

智能手机中的按钮和开关的行为会根据设备的当前状态而有所不同：

- 手机解锁后，按下按钮可以执行各种功能。
- 当手机锁定时，按任意按钮都会进入解锁屏幕。
- 当手机电量不足时，按任意按钮都会显示充电屏幕。

##  Structure 结构


<div align="center"> <img src="/images/state-structure.png"/><p style="text-align: center;"></p></div>


1. 上下文存储对某个具体状态对象的引用，并将所有特定于状态的工作委托给它。上下文通过状态接口与状态对象进行通信。上下文公开一个 setter 用于向其传递新的状态对象。
2. State 接口声明特定于状态的方法。这些方法应该对所有具体状态都有意义，因为你不希望你的某些状态具有永远不会被调用的无用方法。
3. 具体状态为特定于状态的方法提供自己的实现。为了避免在多个状态下重复相似的代码，您可以提供封装一些常见行为的中间抽象类。

   状态对象可以存储对上下文对象的反向引用。通过此引用，状态可以从上下文对象中获取任何所需的信息，并启动状态转换。

4. 上下文和具体状态都可以设置上下文的下一个状态，并通过替换链接到上下文的状态对象来执行实际的状态转换。

## Pseudocode 伪代码

在此示例中，状态模式允许媒体播放器的相同控件以不同的方式运行，具体取决于当前播放状态。


<div align="center"> <img src="/images/state-structure.png"/><p style="text-align: center;"></p>使用状态对象更改对象行为的示例。</div>


播放器的主要对象始终链接到为播放器执行大部分工作的状态对象。某些操作会将玩家的当前状态对象替换为另一个操作，这会更改玩家对用户交互的反应方式。

```java
// The AudioPlayer class acts as a context. It also maintains a
// reference to an instance of one of the state classes that
// represents the current state of the audio player.
class AudioPlayer is
    field state: State
    field UI, volume, playlist, currentSong

    constructor AudioPlayer() is
        this.state = new ReadyState(this)

        // Context delegates handling user input to a state
        // object. Naturally, the outcome depends on what state
        // is currently active, since each state can handle the
        // input differently.
        UI = new UserInterface()
        UI.lockButton.onClick(this.clickLock)
        UI.playButton.onClick(this.clickPlay)
        UI.nextButton.onClick(this.clickNext)
        UI.prevButton.onClick(this.clickPrevious)

    // Other objects must be able to switch the audio player's
    // active state.
    method changeState(state: State) is
        this.state = state

    // UI methods delegate execution to the active state.
    method clickLock() is
        state.clickLock()
    method clickPlay() is
        state.clickPlay()
    method clickNext() is
        state.clickNext()
    method clickPrevious() is
        state.clickPrevious()

    // A state may call some service methods on the context.
    method startPlayback() is
        // ...
    method stopPlayback() is
        // ...
    method nextSong() is
        // ...
    method previousSong() is
        // ...
    method fastForward(time) is
        // ...
    method rewind(time) is
        // ...


// The base state class declares methods that all concrete
// states should implement and also provides a backreference to
// the context object associated with the state. States can use
// the backreference to transition the context to another state.
abstract class State is
    protected field player: AudioPlayer

    // Context passes itself through the state constructor. This
    // may help a state fetch some useful context data if it's
    // needed.
    constructor State(player) is
        this.player = player

    abstract method clickLock()
    abstract method clickPlay()
    abstract method clickNext()
    abstract method clickPrevious()


// Concrete states implement various behaviors associated with a
// state of the context.
class LockedState extends State is

    // When you unlock a locked player, it may assume one of two
    // states.
    method clickLock() is
        if (player.playing)
            player.changeState(new PlayingState(player))
        else
            player.changeState(new ReadyState(player))

    method clickPlay() is
        // Locked, so do nothing.

    method clickNext() is
        // Locked, so do nothing.

    method clickPrevious() is
        // Locked, so do nothing.


// They can also trigger state transitions in the context.
class ReadyState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.startPlayback()
        player.changeState(new PlayingState(player))

    method clickNext() is
        player.nextSong()

    method clickPrevious() is
        player.previousSong()


class PlayingState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.stopPlayback()
        player.changeState(new ReadyState(player))

    method clickNext() is
        if (event.doubleclick)
            player.nextSong()
        else
            player.fastForward(5)

    method clickPrevious() is
        if (event.doubleclick)
            player.previous()
        else
            player.rewind(5)
```

## Applicability 适用性

1. 如果对象的行为根据其当前状态而有所不同，状态数量巨大，并且特定于状态的代码经常更改，请使用 State 模式。
2. 该模式建议将所有特定于状态的代码提取到一组不同的类中。因此，您可以相互独立地添加新状态或更改现有状态，从而降低维护成本。
3. 当类被大量条件污染时，请使用该模式，这些条件会根据类字段的当前值改变类的行为方式。
4. State 模式允许您将这些条件的分支提取到相应状态类的方法中。执行此操作时，还可以从主类中清除特定于状态的代码中涉及的临时字段和帮助程序方法。
5. 当在基于条件的状态机的类似状态和转换中有许多重复代码时，请使用状态。
6. State 模式允许您编写状态类的层次结构，并通过将公共代码提取到抽象基类中来减少重复。

## How to Implement 如何实现

1. 确定哪个类将充当上下文。它可以是已经具有状态相关代码的现有类;或新类（如果特定于状态的代码分布在多个类中）。
2. 声明状态接口。尽管它可以镜像上下文中声明的所有方法，但仅针对那些可能包含特定于状态的行为的方法。
3. 对于每个实际状态，创建一个派生自状态接口的类。然后检查上下文的方法，并将与该状态相关的所有代码提取到新创建的类中。

   将代码移动到 state 类时，您可能会发现它依赖于上下文的私有成员。有几种解决方法：

   - 将这些字段或方法设为公共。
   - 将要提取的行为转换为上下文中的公共方法，并从 state 类中调用它。这种方式很丑陋，但速度很快，您可以随时稍后修复它。
   - 将状态类嵌套到 context 类中，但前提是您的编程语言支持嵌套类。

4. 在上下文类中，添加状态接口类型的引用字段和允许重写该字段值的公共 setter 。
5. 再次浏览上下文的方法，并将空状态条件替换为对状态对象的相应方法的调用。
6. 若要切换上下文的状态，请创建其中一个状态类的实例并将其传递给上下文。您可以在上下文本身、各种状态或客户端中执行此操作。无论在哪里执行此操作，该类都依赖于它实例化的具体状态类。

## Pros and Cons 优点和缺点

| 优点√                                                     | 缺点×                                                            |
| --------------------------------------------------------- | ---------------------------------------------------------------- |
| 单一责任原则。将与特定状态相关的代码组织到单独的类中。    | 如果状态机只有几个状态或很少更改，则应用该模式可能有点矫枉过正。 |
| 开/闭原则。在不更改现有状态类或上下文的情况下引入新状态。 |                                                                  |
| 通过消除笨重的状态机条件来简化上下文的代码。              |                                                                  |

## Relations with Other Patterns 与其他模式的关系

- 桥接、状态、策略（在某种程度上还有适配器）具有非常相似的结构。事实上，所有这些模式都是基于构图的，而构图是将工作委托给其他对象。但是，它们都解决了不同的问题。模式不仅仅是以特定方式构建代码的秘诀。它还可以向其他开发人员传达该模式解决的问题。
- 状态可以看作是战略的延伸。这两种模式都基于组合：它们通过将一些工作委派给帮助对象来改变上下文的行为。策略使这些对象完全独立，彼此不相知。但是，State 不会限制具体状态之间的依赖关系，而是允许它们随意更改上下文的状态。

## Code Examples 代码示例

# **State** in Python

状态是一种行为设计模式，它允许对象在其内部状态更改时更改行为。

该模式将与状态相关的行为提取到单独的状态类中，并强制原始对象将工作委托给这些类的实例，而不是自行操作。

使用示例：状态模式在 Python 中常用于将大量 使用示例：状态模式在 Python 中常用于将大量 `switch` -base 状态机转换为对象。 -base 状态机转换为对象。

标识：状态模式可以通过根据对象状态改变其行为的方法识别，并由外部控制。

## Conceptual Example 概念示例

此示例说明了状态设计模式的结构。它侧重于回答以下问题：

- 它由哪些类组成？
- 这些课程扮演什么角色？
- 模式的元素以何种方式相关？

#### main.py：概念示例

```python
from __future__ import annotations
from abc import ABC, abstractmethod


class Context:
    """
    The Context defines the interface of interest to clients. It also maintains
    a reference to an instance of a State subclass, which represents the current
    state of the Context.
    """

    _state = None
    """
    A reference to the current state of the Context.
    """

    def __init__(self, state: State) -> None:
        self.transition_to(state)

    def transition_to(self, state: State):
        """
        The Context allows changing the State object at runtime.
        """

        print(f"Context: Transition to {type(state).__name__}")
        self._state = state
        self._state.context = self

    """
    The Context delegates part of its behavior to the current State object.
    """

    def request1(self):
        self._state.handle1()

    def request2(self):
        self._state.handle2()


class State(ABC):
    """
    The base State class declares methods that all Concrete State should
    implement and also provides a backreference to the Context object,
    associated with the State. This backreference can be used by States to
    transition the Context to another State.
    """

    @property
    def context(self) -> Context:
        return self._context

    @context.setter
    def context(self, context: Context) -> None:
        self._context = context

    @abstractmethod
    def handle1(self) -> None:
        pass

    @abstractmethod
    def handle2(self) -> None:
        pass


"""
Concrete States implement various behaviors, associated with a state of the
Context.
"""


class ConcreteStateA(State):
    def handle1(self) -> None:
        print("ConcreteStateA handles request1.")
        print("ConcreteStateA wants to change the state of the context.")
        self.context.transition_to(ConcreteStateB())

    def handle2(self) -> None:
        print("ConcreteStateA handles request2.")


class ConcreteStateB(State):
    def handle1(self) -> None:
        print("ConcreteStateB handles request1.")

    def handle2(self) -> None:
        print("ConcreteStateB handles request2.")
        print("ConcreteStateB wants to change the state of the context.")
        self.context.transition_to(ConcreteStateA())


if __name__ == "__main__":
    # The client code.

    context = Context(ConcreteStateA())
    context.request1()
    context.request2()

```

####  **Output.txt:** Execution result

````
Context: Transition to ConcreteStateA
ConcreteStateA handles request1.
ConcreteStateA wants to change the state of the context.
Context: Transition to ConcreteStateB
ConcreteStateB handles request2.
ConcreteStateB wants to change the state of the context.
Context: Transition to ConcreteStateA
````

# **State** in Rust

状态是一种行为设计模式，它允许对象在其内部状态更改时更改行为。

该模式将与状态相关的行为提取到单独的状态类中，并强制原始对象将工作委托给这些类的实例，而不是自行操作。

状态模式与有限状态机 （FSM） 概念相关，但是，每个状态都由实现公共状态特征的单独类型表示，而不是实现大量条件语句。状态之间的转换取决于每种状态类型的特定特征实现。

## Music Player 音乐播放器

让我们构建一个具有以下状态转换的音乐播放器：


<div align="center"> <img src="/images/state-code-rust-example.jpeg"/><p style="text-align: center;"></p></div>


有一个基本特征 `State` 和 `play` `stop` 方法，可以进行状态转换：

```rust
pub trait State {
    fn play(self: Box<Self>, player: &mut Player) -> Box<dyn State>;
    fn stop(self: Box<Self>, player: &mut Player) -> Box<dyn State>;
}
```

`next` 并且  并且  并且 `prev` 不要更改状态，在单独的  不要更改状态，在单独的  不要更改状态，在单独的 `impl dyn State` 块中存在无法覆盖的默认实现。 块中存在无法覆盖的默认实现。

```rust
impl dyn State {
    pub fn next(self: Box<Self>, player: &mut Player) -> Box<dyn State> {
        self
    }

    pub fn prev(self: Box<Self>, player: &mut Player) -> Box<dyn State> {
        self
    }
}
```

每个状态都是实现以下各项的 `trait State` 类型：

```rust
pub struct StoppedState;
pub struct PausedState;
pub struct PlayingState;

impl State for StoppedState {
    ...
}

impl State for PausedState {
    ...
}
```

无论如何，它的工作原理如下：

```rust
let state = Box::new(StoppedState);   // StoppedState.
let state = state.play(&mut player);  // StoppedState -> PlayingState.
let state = state.play(&mut player);  // PlayingState -> PausedState.
```

在这里，相同的操作 在这里，相同的操作 `play` 会根据调用位置转换到不同的状态： 会根据调用位置转换到不同的状态：

1. `StoppedState` `play` 的实现开始播放并返回 `PlayingState` 。

   ```rust
   fn play(self: Box<Self>, player: &mut Player) -> Box<dyn State> {
       player.play();
   
       // Stopped -> Playing.
       Box::new(PlayingState)
   }
   ```

2. `PlayingState` 再次点击“播放”按钮后暂停播放：

   ````
   fn play(self: Box<Self>, player: &mut Player) -> Box<dyn State> {
       player.pause();
   
       // Playing -> Paused.
       Box::new(PausedState)
   }
   
   ````


这些方法使用特殊 `self: Box<Self>` 表示法定义。

为什么？

1. 首先， `self` 不是参考，它意味着该方法是“一次性”，它消耗 `self` 并交换到另一个状态返回 `Box<dyn State>` 。
2. 其次，该方法使用盒装对象 like `Box<dyn State>` 而不是具体类型的对象 `PlayingState` ，因为具体状态在编译时是未知的。

#### **player.rs**

```rust
/// A music track.
pub struct Track {
    pub title: String,
    pub duration: u32,
    cursor: u32,
}

impl Track {
    pub fn new(title: &'static str, duration: u32) -> Self {
        Self {
            title: title.into(),
            duration,
            cursor: 0,
        }
    }
}

/// A music player holds a playlist and it can do basic operations over it.
pub struct Player {
    playlist: Vec<Track>,
    current_track: usize,
    _volume: u8,
}

impl Default for Player {
    fn default() -> Self {
        Self {
            playlist: vec![
                Track::new("Track 1", 180),
                Track::new("Track 2", 165),
                Track::new("Track 3", 197),
                Track::new("Track 4", 205),
            ],
            current_track: 0,
            _volume: 25,
        }
    }
}

impl Player {
    pub fn next_track(&mut self) {
        self.current_track = (self.current_track + 1) % self.playlist.len();
    }

    pub fn prev_track(&mut self) {
        self.current_track = (self.playlist.len() + self.current_track - 1) % self.playlist.len();
    }

    pub fn play(&mut self) {
        self.track_mut().cursor = 10; // Playback imitation.
    }

    pub fn pause(&mut self) {
        self.track_mut().cursor = 43; // Paused at some moment.
    }

    pub fn rewind(&mut self) {
        self.track_mut().cursor = 0;
    }

    pub fn track(&self) -> &Track {
        &self.playlist[self.current_track]
    }

    fn track_mut(&mut self) -> &mut Track {
        &mut self.playlist[self.current_track]
    }
}

```

####  **state.rs**

```rust
use cursive::views::TextView;

use crate::player::Player;

pub struct StoppedState;
pub struct PausedState;
pub struct PlayingState;

/// There is a base `State` trait with methods `play` and `stop` which make
/// state transitions. There are also `next` and `prev` methods in a separate
/// `impl dyn State` block below, those are default implementations
/// that cannot be overridden.
///
/// What is the `self: Box<Self>` notation? We use the state as follows:
/// ```rust
///   let prev_state = Box::new(PlayingState);
///   let next_state = prev_state.play(&mut player);
/// ```
/// A method `play` receives a whole `Box<PlayingState>` object,
/// and not just `PlayingState`. The previous state "disappears" in the method,
/// in turn, it returns a new `Box<PausedState>` state object.
pub trait State {
    fn play(self: Box<Self>, player: &mut Player) -> Box<dyn State>;
    fn stop(self: Box<Self>, player: &mut Player) -> Box<dyn State>;
    fn render(&self, player: &Player, view: &mut TextView);
}

impl State for StoppedState {
    fn play(self: Box<Self>, player: &mut Player) -> Box<dyn State> {
        player.play();

        // Stopped -> Playing.
        Box::new(PlayingState)
    }

    fn stop(self: Box<Self>, _: &mut Player) -> Box<dyn State> {
        // Change no state.
        self
    }

    fn render(&self, _: &Player, view: &mut TextView) {
        view.set_content("[Stopped] Press 'Play'")
    }
}

impl State for PausedState {
    fn play(self: Box<Self>, player: &mut Player) -> Box<dyn State> {
        player.pause();

        // Paused -> Playing.
        Box::new(PlayingState)
    }

    fn stop(self: Box<Self>, player: &mut Player) -> Box<dyn State> {
        player.pause();
        player.rewind();

        // Paused -> Stopped.
        Box::new(StoppedState)
    }

    fn render(&self, player: &Player, view: &mut TextView) {
        view.set_content(format!(
            "[Paused] {} - {} sec",
            player.track().title,
            player.track().duration
        ))
    }
}

impl State for PlayingState {
    fn play(self: Box<Self>, player: &mut Player) -> Box<dyn State> {
        player.pause();

        // Playing -> Paused.
        Box::new(PausedState)
    }

    fn stop(self: Box<Self>, player: &mut Player) -> Box<dyn State> {
        player.pause();
        player.rewind();

        // Playing -> Stopped.
        Box::new(StoppedState)
    }

    fn render(&self, player: &Player, view: &mut TextView) {
        view.set_content(format!(
            "[Playing] {} - {} sec",
            player.track().title,
            player.track().duration
        ))
    }
}

// Default "next" and "prev" implementations for the trait.
impl dyn State {
    pub fn next(self: Box<Self>, player: &mut Player) -> Box<dyn State> {
        player.next_track();

        // Change no state.
        self
    }

    pub fn prev(self: Box<Self>, player: &mut Player) -> Box<dyn State> {
        player.prev_track();

        // Change no state.
        self
    }
}

```

####  **main.rs**

```rust
mod player;
mod state;

use cursive::{
    event::Key,
    view::Nameable,
    views::{Dialog, TextView},
    Cursive,
};
use player::Player;
use state::{State, StoppedState};

// Application context: a music player and a state.
struct PlayerApplication {
    player: Player,
    state: Box<dyn State>,
}

fn main() {
    let mut app = cursive::default();

    app.set_user_data(PlayerApplication {
        player: Player::default(),
        state: Box::new(StoppedState),
    });

    app.add_layer(
        Dialog::around(TextView::new("Press Play").with_name("Player Status"))
            .title("Music Player")
            .button("Play", |s| execute(s, "Play"))
            .button("Stop", |s| execute(s, "Stop"))
            .button("Prev", |s| execute(s, "Prev"))
            .button("Next", |s| execute(s, "Next")),
    );

    app.add_global_callback(Key::Esc, |s| s.quit());

    app.run();
}

fn execute(s: &mut Cursive, button: &'static str) {
    let PlayerApplication {
        mut player,
        mut state,
    } = s.take_user_data().unwrap();

    let mut view = s.find_name::<TextView>("Player Status").unwrap();

    // Here is how state mechanics work: the previous state
    // executes an action and returns a new state.
    // Each state has all 4 operations but reacts differently.
    state = match button {
        "Play" => state.play(&mut player),
        "Stop" => state.stop(&mut player),
        "Prev" => state.prev(&mut player),
        "Next" => state.next(&mut player),
        _ => unreachable!(),
    };

    state.render(&player, &mut view);

    s.set_user_data(PlayerApplication { player, state });
}
```

### 截图


<div align="center"> <img src="/images/state-rust-example-screenshots1.png"/><p style="text-align: center;"></p></div>





<div align="center"> <img src="/images/state-rust-example-screenshots2.png"/><p style="text-align: center;"></p></div>
