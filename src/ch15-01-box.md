## 使用 `Box<T>` 指向堆上的数据

最直接的智能指针是 _box_，其类型写作 `Box<T>`。Box 允许你将数据存储在堆上，而不是栈上。栈上保留的是指向堆数据的指针。回顾第 4 章以了解栈和堆的区别。

Box 除了将数据存储在堆上而不是栈上之外，没有性能开销。但它们也没有太多额外的功能。你通常会在以下情况下使用它们：

- 当你有一个在编译时无法确定大小的类型，并且你希望在一个需要确切大小的上下文中使用该类型的值时
- 当你有大量数据并希望转移所有权，但确保在转移时不会复制数据时
- 当你希望拥有一个值，并且只关心它是一个实现了特定 trait 的类型，而不是特定类型时

我们将在 [“使用 Box 启用递归类型”](#enabling-recursive-types-with-boxes)<!-- ignore --> 中演示第一种情况。在第二种情况下，转移大量数据的所有权可能会花费很长时间，因为数据会在栈上来回复制。为了提高这种情况下的性能，我们可以将大量数据存储在堆上的 box 中。然后，只有少量的指针数据会在栈上来回复制，而它引用的数据则保留在堆上的一个位置。第三种情况被称为 _trait 对象_，第 18 章的 [“使用允许不同类型值的 Trait 对象”][trait-objects]<!-- ignore --> 专门讨论了这个主题。因此，你在这里学到的内容将在那一节中再次应用！

### 使用 `Box<T>` 在堆上存储数据

在我们讨论 `Box<T>` 的堆存储用例之前，我们将介绍语法以及如何与存储在 `Box<T>` 中的值进行交互。

Listing 15-1 展示了如何使用 box 在堆上存储一个 `i32` 值。

<Listing number="15-1" file-name="src/main.rs" caption="使用 box 在堆上存储一个 `i32` 值">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

</Listing>

我们定义变量 `b` 的值为一个指向堆上分配的 `5` 的 `Box`。这个程序将打印 `b = 5`；在这种情况下，我们可以像访问栈上的数据一样访问 box 中的数据。就像任何拥有所有权的值一样，当 box 超出作用域时，如 `b` 在 `main` 结束时，它将被释放。释放操作既针对 box（存储在栈上）也针对它指向的数据（存储在堆上）。

将单个值放在堆上并不是很有用，因此你不会经常以这种方式单独使用 box。在大多数情况下，将像单个 `i32` 这样的值存储在栈上（默认情况下它们存储在那里）更为合适。让我们看一个 box 允许我们定义类型的例子，如果没有 box，我们将无法定义这些类型。

### 使用 Box 启用递归类型

_递归类型_ 的值可以将另一个相同类型的值作为自身的一部分。递归类型会带来问题，因为 Rust 需要在编译时知道一个类型占用了多少空间。然而，递归类型的值的嵌套理论上可以无限继续，因此 Rust 无法知道该值需要多少空间。由于 box 有已知的大小，我们可以通过在递归类型定义中插入一个 box 来启用递归类型。

作为递归类型的一个例子，让我们探索 _cons list_。这是函数式编程语言中常见的数据类型。我们将定义的 cons list 类型除了递归之外都很简单；因此，我们在这个例子中使用的概念在你遇到涉及递归类型的更复杂情况时也会有用。

#### 关于 Cons List 的更多信息

_cons list_ 是一种来自 Lisp 编程语言及其方言的数据结构，由嵌套的对组成，是 Lisp 版本的链表。它的名字来源于 Lisp 中的 `cons` 函数（_construct function_ 的缩写），该函数从其两个参数构造一个新的对。通过对一个由值和对组成的对调用 `cons`，我们可以构造由递归对组成的 cons list。

例如，这是一个包含列表 `1, 2, 3` 的 cons list 的伪代码表示，每个对都用括号括起来：

```text
(1, (2, (3, Nil)))
```

cons list 中的每个项目包含两个元素：当前项目的值和下一个项目。列表中的最后一个项目只包含一个名为 `Nil` 的值，没有下一个项目。cons list 是通过递归调用 `cons` 函数生成的。表示递归基本情况的标准名称是 `Nil`。请注意，这与第 6 章讨论的“null”或“nil”概念不同，后者是无效或缺失的值。

cons list 并不是 Rust 中常用的数据结构。大多数情况下，当你在 Rust 中有一个项目列表时，`Vec<T>` 是更好的选择。其他更复杂的递归数据类型在各种情况下 _确实_ 有用，但通过在本章中从 cons list 开始，我们可以在没有太多干扰的情况下探索 box 如何让我们定义一个递归数据类型。

Listing 15-2 包含了一个 cons list 的枚举定义。请注意，这段代码还不能编译，因为 `List` 类型没有已知的大小，我们将演示这一点。

<Listing number="15-2" file-name="src/main.rs" caption="第一次尝试定义一个枚举来表示 `i32` 值的 cons list 数据结构">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

</Listing>

> 注意：为了这个示例的目的，我们实现了一个只包含 `i32` 值的 cons list。我们可以使用泛型来实现它，如第 10 章所讨论的，以定义一个可以存储任何类型值的 cons list 类型。

使用 `List` 类型存储列表 `1, 2, 3` 将如 Listing 15-3 所示。

<Listing number="15-3" file-name="src/main.rs" caption="使用 `List` 枚举存储列表 `1, 2, 3`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

</Listing>

第一个 `Cons` 值持有 `1` 和另一个 `List` 值。这个 `List` 值是另一个 `Cons` 值，它持有 `2` 和另一个 `List` 值。这个 `List` 值又是一个 `Cons` 值，它持有 `3` 和一个 `List` 值，最后是 `Nil`，这是表示列表结束的非递归变体。

如果我们尝试编译 Listing 15-3 中的代码，我们将得到 Listing 15-4 中显示的错误。

<Listing number="15-4" file-name="output.txt" caption="尝试定义递归枚举时得到的错误">

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

</Listing>

错误显示这个类型“具有无限大小”。原因是我们定义了一个递归的 `List` 变体：它直接持有另一个自身的值。因此，Rust 无法确定存储一个 `List` 值需要多少空间。让我们分解一下为什么会出现这个错误。首先，我们将看看 Rust 如何决定存储一个非递归类型的值需要多少空间。

#### 计算非递归类型的大小

回顾我们在第 6 章讨论枚举定义时在 Listing 6-2 中定义的 `Message` 枚举：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

为了确定为一个 `Message` 值分配多少空间，Rust 会遍历每个变体，看看哪个变体需要最多的空间。Rust 发现 `Message::Quit` 不需要任何空间，`Message::Move` 需要足够的空间来存储两个 `i32` 值，依此类推。因为只会使用一个变体，所以 `Message` 值所需的最大空间是存储其最大变体所需的空间。

与此形成对比的是，当 Rust 尝试确定像 Listing 15-2 中的 `List` 枚举这样的递归类型需要多少空间时会发生什么。编译器首先查看 `Cons` 变体，它持有一个 `i32` 类型的值和一个 `List` 类型的值。因此，`Cons` 需要等于 `i32` 的大小加上 `List` 的大小的空间。为了确定 `List` 类型需要多少内存，编译器查看变体，从 `Cons` 变体开始。`Cons` 变体持有一个 `i32` 类型的值和一个 `List` 类型的值，这个过程无限继续，如图 15-1 所示。

<img alt="一个无限的 Cons list" src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">图 15-1: 由无限 `Cons` 变体组成的无限 `List`</span>

#### 使用 `Box<T>` 获取具有已知大小的递归类型

因为 Rust 无法确定为递归定义的类型分配多少空间，编译器给出了一个包含有用建议的错误：

<!-- manual-regeneration
在自动生成后，查看 listings/ch15-smart-pointers/listing-15-03/output.txt 并复制相关行
-->

```text
help: 插入一些间接性（例如，一个 `Box`、`Rc` 或 `&`）来打破循环
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

在这个建议中，_间接性_ 意味着我们不应该直接存储一个值，而是应该通过存储一个指向该值的指针来改变数据结构。

因为 `Box<T>` 是一个指针，Rust 总是知道一个 `Box<T>` 需要多少空间：指针的大小不会根据它指向的数据量而变化。这意味着我们可以在 `Cons` 变体中放入一个 `Box<T>`，而不是直接放入另一个 `List` 值。`Box<T>` 将指向堆上的下一个 `List` 值，而不是在 `Cons` 变体内部。从概念上讲，我们仍然有一个列表，由持有其他列表的列表创建，但这个实现现在更像是将项目彼此相邻放置，而不是彼此嵌套。

我们可以将 Listing 15-2 中的 `List` 枚举定义和 Listing 15-3 中的 `List` 使用改为 Listing 15-5 中的代码，这将编译通过。

<Listing number="15-5" file-name="src/main.rs" caption="使用 `Box<T>` 定义 `List` 以具有已知大小">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

</Listing>

`Cons` 变体需要一个 `i32` 的大小加上存储 box 指针数据的空间。`Nil` 变体不存储任何值，因此它需要的空间比 `Cons` 变体少。我们现在知道任何 `List` 值将占用一个 `i32` 的大小加上一个 box 指针数据的大小。通过使用 box，我们打破了无限的递归链，因此编译器可以确定存储一个 `List` 值需要多少空间。图 15-2 展示了 `Cons` 变体现在的样子。

<img alt="一个有限的 Cons list" src="img/trpl15-02.svg" class="center" />

<span class="caption">图 15-2: 一个不是无限大小的 `List`，因为 `Cons` 持有一个 `Box`</span>

Box 只提供间接性和堆分配；它们没有其他特殊功能，比如我们将在其他智能指针类型中看到的功能。它们也没有这些特殊功能带来的性能开销，因此它们可以在像 cons list 这样的情况下有用，其中间接性是我们唯一需要的功能。我们将在第 18 章中看到更多 box 的用例。

`Box<T>` 类型是一个智能指针，因为它实现了 `Deref` trait，这允许 `Box<T>` 值像引用一样被处理。当 `Box<T>` 值超出作用域时，由于 `Drop` trait 的实现，box 指向的堆数据也会被清理。这两个 trait 对于我们将在本章其余部分讨论的其他智能指针类型提供的功能将更加重要。让我们更详细地探讨这两个 trait。

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types