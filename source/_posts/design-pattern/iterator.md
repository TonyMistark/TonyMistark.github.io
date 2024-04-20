---
title: iterator 迭代器模式
categories:
  - design-pattern
date: 2024-02-19 22:43:15
---

## Intent  意图

迭代器是一种行为设计模式，它允许您遍历集合的元素，而不暴露其底层表示（列表、堆栈、树等）。


<div align="center"> <img src="/images/iterator-header.png"/></div>


## Problem  问题

集合是编程中最常用的数据类型之一。尽管如此，集合只是一组对象的容器。


<div align="center"> <img src="/images/iterator-problem1.png"/>各种类型的集合</div>


大多数集合将它们的元素存储在简单的列表中。然而，其中一些是基于堆栈，树，图和其他复杂的数据结构。

但无论集合是如何构造的，它都必须提供某种访问其元素的方法，以便其他代码可以使用这些元素。应该有一种方法可以遍历集合中的每个元素，而不必反复访问相同的元素。

如果你有一个基于列表的集合，这听起来可能是一个简单的工作。你只需遍历所有元素。但是，如何顺序遍历复杂数据结构（如树）的元素呢？例如，有一天您可能会很好地使用深度优先遍历树。但是第二天你可能需要广度优先遍历。接下来的一周，你可能需要一些其他的东西，比如对树元素的随机访问。


<div align="center"> <img src="/images/iterator-problem2.png"/>同一个集合可以用几种不同的方式遍历。</div>


向集合中添加越来越多的遍历算法逐渐模糊了它的主要职责，即高效的数据存储。此外，有些算法可能是为特定的应用程序定制的，因此将它们包含到泛型集合类中会很奇怪。

另一方面，应该处理各种集合的客户端代码可能甚至不关心它们如何存储元素。但是，由于集合都提供了访问其元素的不同方式，因此除了将代码耦合到特定的集合类之外，您别无选择。

## Solution 解决方案

迭代器模式的主要思想是将集合的遍历行为提取到称为迭代器的单独对象中。


<div align="center"> <img src="/images/iterator-solution1.png"/>迭代器实现各种遍历算法。多个迭代器对象可以同时遍历同一个集合。</div>


除了实现算法本身之外，迭代器对象还封装了所有遍历细节，比如当前位置和最后还剩多少元素。因此，多个迭代器可以同时遍历同一个集合，彼此独立。

通常，迭代器提供了一个获取集合元素的主要方法。客户端可以继续运行这个方法，直到它不返回任何东西，这意味着迭代器已经遍历了所有的元素。

所有迭代器必须实现相同的接口。这使得客户端代码与任何集合类型或任何遍历算法兼容，只要有合适的迭代器即可。如果你需要一种特殊的方法来遍历一个集合，你只需要创建一个新的迭代器类，而不必改变集合或客户端。

## Real-World Analogy  现实世界的类比


<div align="center"> <img src="/images/iterator-comic-1-en.png"/>在罗马有各种各样的步行方式。</div>


另一方面，你可以为你的智能手机购买一个虚拟指南应用程序，并将其用于导航。这是智能和廉价的，你可以留在一些有趣的地方，只要你想。

第三种选择是，你可以花一些旅行的预算，聘请一个当地导游谁知道城市像他的手背。导游将能够根据您的喜好量身定制旅游，向您展示每一个景点，并讲述许多令人兴奋的故事。那会更有趣;但是，唉，也更贵了。

所有这些选项--你脑中随机产生的方向、智能手机导航器或人类向导--都是位于罗马的大量景点和景点的迭代器。

## Structure  结构


<div align="center"> <img src="/images/iterator-structure.png"/>在罗马有各种各样的步行方式。</div>


1. Iterator接口声明了遍历集合所需的操作：获取下一个元素，检索当前位置，重新开始迭代等。
2. 具体迭代器实现了遍历集合的特定算法。迭代器对象应该自己跟踪遍历进度。这允许多个迭代器彼此独立地遍历同一集合。
3. Collection接口声明了一个或多个方法来获取与集合兼容的迭代器。注意，方法的返回类型必须声明为迭代器接口，以便具体集合可以返回各种迭代器。
4. 每次客户端请求一个具体迭代器类时，具体集合返回一个特定具体迭代器类的新实例。您可能会想，集合的其余代码在哪里？别担心，应该在同一个班级。只是这些细节对实际的模式并不重要，所以我们省略了它们。
5. 客户端通过集合和迭代器的接口与它们一起工作。这样，客户端就不会耦合到具体的类，从而允许您在同一客户端代码中使用各种集合和迭代器。

   通常，客户端不会自己创建迭代器，而是从集合中获取迭代器。然而，在某些情况下，客户端可以直接创建一个;例如，当客户端定义自己的特殊迭代器时。


## Pseudocode  伪代码

在本例中，迭代器模式用于遍历一种特殊的集合，该集合封装了对Facebook社交图的访问。该集合提供了几个迭代器，它们可以以各种方式遍历概要文件。


<div align="center"> <img src="/images/iterator-example.png"/>迭代社交配置文件的示例。</div>


“friends”迭代器可用于查看给定配置文件的好友。“colleagues”迭代器做同样的事情，只是它忽略了与目标人不在同一家公司工作的朋友。这两个迭代器都实现了一个公共接口，允许客户端获取配置文件，而无需深入研究实现细节，如身份验证和发送REST请求。

客户端代码没有耦合到具体的类，因为它只通过接口与集合和迭代器一起工作。如果您决定将应用连接到新的社交网络，则只需提供新的集合和迭代器类，而无需更改现有代码。

```java
// The collection interface must declare a factory method for
// producing iterators. You can declare several methods if there
// are different kinds of iteration available in your program.
interface SocialNetwork is
    method createFriendsIterator(profileId):ProfileIterator
    method createCoworkersIterator(profileId):ProfileIterator


// Each concrete collection is coupled to a set of concrete
// iterator classes it returns. But the client isn't, since the
// signature of these methods returns iterator interfaces.
class Facebook implements SocialNetwork is
    // ... The bulk of the collection's code should go here ...

    // Iterator creation code.
    method createFriendsIterator(profileId) is
        return new FacebookIterator(this, profileId, "friends")
    method createCoworkersIterator(profileId) is
        return new FacebookIterator(this, profileId, "coworkers")


// The common interface for all iterators.
interface ProfileIterator is
    method getNext():Profile
    method hasMore():bool


// The concrete iterator class.
class FacebookIterator implements ProfileIterator is
    // The iterator needs a reference to the collection that it
    // traverses.
    private field facebook: Facebook
    private field profileId, type: string

    // An iterator object traverses the collection independently
    // from other iterators. Therefore it has to store the
    // iteration state.
    private field currentPosition
    private field cache: array of Profile

    constructor FacebookIterator(facebook, profileId, type) is
        this.facebook = facebook
        this.profileId = profileId
        this.type = type

    private method lazyInit() is
        if (cache == null)
            cache = facebook.socialGraphRequest(profileId, type)

    // Each concrete iterator class has its own implementation
    // of the common iterator interface.
    method getNext() is
        if (hasMore())
            result = cache[currentPosition]
            currentPosition++
            return result

    method hasMore() is
        lazyInit()
        return currentPosition < cache.length


// Here is another useful trick: you can pass an iterator to a
// client class instead of giving it access to a whole
// collection. This way, you don't expose the collection to the
// client.
//
// And there's another benefit: you can change the way the
// client works with the collection at runtime by passing it a
// different iterator. This is possible because the client code
// isn't coupled to concrete iterator classes.
class SocialSpammer is
    method send(iterator: ProfileIterator, message: string) is
        while (iterator.hasMore())
            profile = iterator.getNext()
            System.sendEmail(profile.getEmail(), message)


// The application class configures collections and iterators
// and then passes them to the client code.
class Application is
    field network: SocialNetwork
    field spammer: SocialSpammer

    method config() is
        if working with Facebook
            this.network = new Facebook()
        if working with LinkedIn
            this.network = new LinkedIn()
        this.spammer = new SocialSpammer()

    method sendSpamToFriends(profile) is
        iterator = network.createFriendsIterator(profile.getId())
        spammer.send(iterator, "Very important message")

    method sendSpamToCoworkers(profile) is
        iterator = network.createCoworkersIterator(profile.getId())
        spammer.send(iterator, "Very important message")
```

## Applicability  适用性

- **当您的集合具有复杂的数据结构时，请使用迭代器模式，但您希望对客户机隐藏其复杂性（出于方便或安全原因）。**
- 迭代器封装了处理复杂数据结构的细节，为客户端提供了几种访问集合元素的简单方法。虽然这种方法对客户端来说非常方便，但它也可以保护集合免受客户端在直接使用集合时可能执行的粗心或恶意操作的影响。
- **使用该模式可以减少应用中重复的遍历代码。**
- 非平凡迭代算法的代码往往非常庞大。当放置在应用程序的业务逻辑中时，它可能会模糊原始代码的责任，并使其减少维护。将遍历代码移动到指定的迭代器可以帮助您使应用程序的代码更加精简和干净。
- **当你希望你的代码能够遍历不同的数据结构，或者这些结构的类型事先是未知的时，使用迭代器。**
- 该模式为集合和迭代器提供了两个泛型接口。假设你的代码现在使用这些接口，如果你传递给它实现这些接口的各种集合和迭代器，它仍然可以工作。

## How to Implement 如何实施

1. 对迭代器接口进行Declare。至少，它必须有一个从集合中获取下一个元素的方法。但是为了方便起见，您可以添加其他一些方法，例如获取前一个元素，跟踪当前位置，以及检查迭代的结束。
2. decode集合接口并描述获取迭代器的方法。返回类型应该等于迭代器接口的返回类型。如果你计划有几个不同的迭代器组，你可以声明类似的方法。
3. 为您希望使用迭代器遍历的集合实现具体的迭代器类。迭代器对象必须与单个集合实例链接。通常，这个链接是通过迭代器的构造函数建立的。
4. 在集合类中实现集合接口。主要思想是为客户端提供一个创建迭代器的快捷方式，为特定的集合类定制。集合对象必须将自身传递给迭代器的构造函数，以在它们之间建立链接。
5. 检查客户端代码，使用迭代器替换所有集合遍历代码。客户端每次需要遍历集合元素时都会获取一个新的迭代器对象。

## Pros and Cons 利弊

| √ 利                                                                                        | × 弊                                                               |
| ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| 单一责任原则。您可以通过将庞大的遍历算法提取到单独的类中来清理客户机代码和集合。            | 如果你的应用只处理简单的集合，那么应用这种模式可能会有些矫枉过正。 |
| 开放/封闭原则。您可以实现新类型的集合和迭代器，并将它们传递给现有代码，而不会破坏任何东西。 | 使用迭代器可能比直接遍历某些专用集合的元素效率更低。               |
| 可以并行遍历同一个集合，因为每个迭代器对象都包含自己的迭代状态。                            |                                                                    |
| 出于同样的原因，您可以延迟迭代并在需要时继续它。                                            |                                                                    |

## Relations with Other Patterns  
与其他模式的关系

- 你可以使用迭代器来遍历复合树。
- 可以将工厂方法沿着使用迭代器，让集合子类返回与集合兼容的不同类型的迭代器。
- 您可以使用Memento沿着Iterator来捕获当前的迭代状态，并在必要时将其回滚。
- 您可以使用Visitor沿着Iterator来遍历复杂的数据结构，并对其元素执行某些操作，即使它们都具有不同的类。

## Code Examples  代码示例

# Python中的迭代器

迭代器是一种行为设计模式，它允许顺序遍历复杂的数据结构，而不暴露其内部细节。

## Conceptual Example 概念示例

这个例子说明了迭代器设计模式的结构。它侧重于回答这些问题：

- 它由哪些类组成？
- 这些班级扮演什么角色？
- 模式中的元素是以什么方式联系在一起的？

#### main.py：概念性示例

```python
from __future__ import annotations
from collections.abc import Iterable, Iterator
from typing import Any


"""
To create an iterator in Python, there are two abstract classes from the built-
in `collections` module - Iterable,Iterator. We need to implement the
`__iter__()` method in the iterated object (collection), and the `__next__ ()`
method in theiterator.
"""


class AlphabeticalOrderIterator(Iterator):
    """
    Concrete Iterators implement various traversal algorithms. These classes
    store the current traversal position at all times.
    """

    """
    `_position` attribute stores the current traversal position. An iterator may
    have a lot of other fields for storing iteration state, especially when it
    is supposed to work with a particular kind of collection.
    """
    _position: int = None

    """
    This attribute indicates the traversal direction.
    """
    _reverse: bool = False

    def __init__(self, collection: WordsCollection, reverse: bool = False) -> None:
        self._collection = collection
        self._reverse = reverse
        self._position = -1 if reverse else 0

    def __next__(self) -> Any:
        """
        The __next__() method must return the next item in the sequence. On
        reaching the end, and in subsequent calls, it must raise StopIteration.
        """
        try:
            value = self._collection[self._position]
            self._position += -1 if self._reverse else 1
        except IndexError:
            raise StopIteration()

        return value


class WordsCollection(Iterable):
    """
    Concrete Collections provide one or several methods for retrieving fresh
    iterator instances, compatible with the collection class.
    """

    def __init__(self, collection: list[Any] | None = None) -> None:
        self._collection = collection or []


    def __getitem__(self, index: int) -> Any:
        return self._collection[index]

    def __iter__(self) -> AlphabeticalOrderIterator:
        """
        The __iter__() method returns the iterator object itself, by default we
        return the iterator in ascending order.
        """
        return AlphabeticalOrderIterator(self)

    def get_reverse_iterator(self) -> AlphabeticalOrderIterator:
        return AlphabeticalOrderIterator(self, True)

    def add_item(self, item: Any) -> None:
        self._collection.append(item)


if __name__ == "__main__":
    # The client code may or may not know about the Concrete Iterator or
    # Collection classes, depending on the level of indirection you want to keep
    # in your program.
    collection = WordsCollection()
    collection.add_item("First")
    collection.add_item("Second")
    collection.add_item("Third")

    print("Straight traversal:")
    print("\n".join(collection))
    print("")

    print("Reverse traversal:")
    print("\n".join(collection.get_reverse_iterator()), end="")
```

#### Output.txt：执行结果

````
Straight traversal:
First
Second
Third

Reverse traversal:
Third
Second
First
````

# Rust中的迭代器

迭代器是一种行为设计模式，它允许顺序遍历复杂的数据结构，而不暴露其内部细节。

## Standard Iterator 标准迭代器

```rust
let array = &[1, 2, 3];
let iterator = array.iter();

// Traversal over each element of the vector.
iterator.for_each(|e| print!("{}, ", e));
```

## Custom Iterator 自定义迭代器

在Rust中，定义自定义迭代器的推荐方法是使用标准的 在Rust中，定义自定义迭代器的推荐方法是使用标准的 `Iterator` trait。该示例不包含合成迭代器接口，因为建议使用惯用的Rust方式。 trait。该示例不包含合成迭代器接口，因为建议使用惯用的Rust方式。

```rust
let users = UserCollection::new();
let mut iterator = users.iter();

iterator.next();
```

`next` 方法是唯一必须实现的  方法是唯一必须实现的  方法是唯一必须实现的 `Iterator` trait方法。它可以访问大量的标准方法，例如  trait方法。它可以访问大量的标准方法，例如  trait方法。它可以访问大量的标准方法，例如 `fold` ， `map` ， `for_each` 。

```rust
impl Iterator for UserIterator<'_> {
    fn next(&mut self) -> Option<Self::Item>;
}
```

#### users.rs：集合和迭代器

```rust
pub struct UserCollection {
    users: [&'static str; 3],
}

/// A custom collection contains an arbitrary user array under the hood.
impl UserCollection {
    /// Returns a custom user collection.
    pub fn new() -> Self {
        Self {
            users: ["Alice", "Bob", "Carl"],
        }
    }

    /// Returns an iterator over a user collection.
    ///
    /// The method name may be different, however, `iter` is used as a de facto
    /// standard in a Rust naming convention.
    pub fn iter(&self) -> UserIterator {
        UserIterator {
            index: 0,
            user_collection: self,
        }
    }
}

/// UserIterator allows sequential traversal through a complex user collection
/// without exposing its internal details.
pub struct UserIterator<'a> {
    index: usize,
    user_collection: &'a UserCollection,
}

/// `Iterator` is a standard interface for dealing with iterators
/// from the Rust standard library.
impl Iterator for UserIterator<'_> {
    type Item = &'static str;

    /// A `next` method is the only `Iterator` trait method which is mandatory to be
    /// implemented. It makes accessible a huge range of standard methods,
    /// e.g. `fold`, `map`, `for_each`.
    fn next(&mut self) -> Option<Self::Item> {
        if self.index < self.user_collection.users.len() {
            let user = Some(self.user_collection.users[self.index]);
            self.index += 1;
            return user;
        }

        None
    }
}
```

#### main.rs：客户端代码

```rust
use crate::users::UserCollection;

mod users;

fn main() {
    print!("Iterators are widely used in the standard library: ");

    let array = &[1, 2, 3];
    let iterator = array.iter();

    // Traversal over each element of the array.
    iterator.for_each(|e| print!("{} ", e));

    println!("\n\nLet's test our own iterator.\n");

    let users = UserCollection::new();
    let mut iterator = users.iter();

    println!("1nd element: {:?}", iterator.next());
    println!("2nd element: {:?}", iterator.next());
    println!("3rd element: {:?}", iterator.next());
    println!("4th element: {:?}", iterator.next());

    print!("\nAll elements in user collection: ");
    users.iter().for_each(|e| print!("{} ", e));

    println!();
}
```

### 输出

````
Iterators are widely used in the standard library: 1 2 3

Let's test our own iterator.

1nd element: Some("Alice")
2nd element: Some("Bob")
3rd element: Some("Carl")
4th element: None


All elements in user collection: Alice Bob Carl
````



