## `RefCell<T>` 和内部可变性模式

_内部可变性_ 是 Rust 中的一种设计模式，它允许你在数据存在不可变引用的情况下仍然能够修改数据；通常情况下，这种行为是被借用规则所禁止的。为了修改数据，该模式在数据结构内部使用了 `unsafe` 代码来绕过 Rust 通常的修改和借用规则。`unsafe` 代码向编译器表明我们正在手动检查这些规则，而不是依赖编译器为我们检查；我们将在第 20 章详细讨论 `unsafe` 代码。

我们只能在确保运行时遵守借用规则的情况下使用内部可变性模式的类型，尽管编译器无法保证这一点。涉及的 `unsafe` 代码随后被包装在一个安全的 API 中，而外部类型仍然是不可变的。

让我们通过查看遵循内部可变性模式的 `RefCell<T>` 类型来探索这个概念。

### 使用 `RefCell<T>` 在运行时强制执行借用规则

与 `Rc<T>` 不同，`RefCell<T>` 类型表示对其持有的数据的单一所有权。那么 `RefCell<T>` 与 `Box<T>` 这样的类型有什么不同呢？回想一下你在第 4 章学到的借用规则：

- 在任何给定时间，你可以拥有 _要么_ 一个可变引用，要么任意数量的不可变引用（但不能同时拥有两者）。
- 引用必须始终有效。

对于引用和 `Box<T>`，借用规则的不变量是在编译时强制执行的。而对于 `RefCell<T>`，这些不变量是在 _运行时_ 强制执行的。对于引用，如果你违反了这些规则，你会得到一个编译错误。对于 `RefCell<T>`，如果你违反了这些规则，你的程序将会 panic 并退出。

在编译时检查借用规则的好处是错误会在开发过程中更早地被捕获，并且不会影响运行时性能，因为所有的分析都在之前完成了。出于这些原因，在大多数情况下，编译时检查借用规则是最佳选择，这也是 Rust 的默认行为。

在运行时检查借用规则的好处是某些内存安全的场景会被允许，而这些场景在编译时检查中会被禁止。静态分析，如 Rust 编译器，本质上是保守的。代码的某些属性无法通过分析代码来检测：最著名的例子是停机问题，这超出了本书的范围，但这是一个有趣的研究主题。

由于某些分析是不可能的，如果 Rust 编译器不能确定代码符合所有权规则，它可能会拒绝一个正确的程序；在这种情况下，它是保守的。如果 Rust 接受了一个错误的程序，用户将无法信任 Rust 所做的保证。然而，如果 Rust 拒绝了一个正确的程序，程序员会感到不便，但不会发生灾难性的事情。`RefCell<T>` 类型在你确信代码遵循借用规则但编译器无法理解和保证这一点时非常有用。

与 `Rc<T>` 类似，`RefCell<T>` 仅用于单线程场景，如果你尝试在多线程上下文中使用它，将会得到一个编译时错误。我们将在第 16 章讨论如何在多线程程序中获得 `RefCell<T>` 的功能。

以下是选择 `Box<T>`、`Rc<T>` 或 `RefCell<T>` 的原因总结：

- `Rc<T>` 允许多个所有者拥有相同的数据；`Box<T>` 和 `RefCell<T>` 有单一所有者。
- `Box<T>` 允许在编译时检查不可变或可变借用；`Rc<T>` 只允许在编译时检查不可变借用；`RefCell<T>` 允许在运行时检查不可变或可变借用。
- 因为 `RefCell<T>` 允许在运行时检查可变借用，所以即使 `RefCell<T>` 是不可变的，你也可以修改 `RefCell<T>` 内部的值。

修改不可变值内部的值是 _内部可变性_ 模式。让我们看看内部可变性有用的情况，并探讨它是如何实现的。

### 内部可变性：对不可变值的可变借用

借用规则的一个后果是，当你有一个不可变的值时，你不能可变地借用它。例如，这段代码无法编译：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

如果你尝试编译这段代码，你会得到以下错误：

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

然而，在某些情况下，一个值在其方法中能够自我修改，但对其他代码来说看起来是不可变的，这将非常有用。值的方法之外的代码将无法修改该值。使用 `RefCell<T>` 是一种获得内部可变性能力的方法，但 `RefCell<T>` 并不能完全绕过借用规则：编译器中的借用检查器允许这种内部可变性，而借用规则是在运行时检查的。如果你违反了这些规则，你会得到一个 `panic!` 而不是编译错误。

让我们通过一个实际的例子来使用 `RefCell<T>` 来修改一个不可变的值，并看看为什么这很有用。

#### 内部可变性的用例：模拟对象

有时在测试中，程序员会使用一个类型来代替另一个类型，以便观察特定的行为并断言其实现是否正确。这个占位符类型被称为 _测试替身_。可以将其想象为电影制作中的替身演员，一个人代替演员来完成一个特别棘手的场景。测试替身在我们运行测试时代表其他类型。_模拟对象_ 是特定类型的测试替身，它们记录测试期间发生的事情，以便你可以断言发生了正确的操作。

Rust 没有像其他语言那样的对象，Rust 的标准库中也没有像其他语言那样内置模拟对象功能。然而，你绝对可以创建一个结构体，它将起到与模拟对象相同的作用。

这是我们将测试的场景：我们将创建一个库，用于跟踪一个值与最大值的接近程度，并根据当前值接近最大值的程度发送消息。例如，这个库可以用于跟踪用户允许的 API 调用次数的配额。

我们的库只提供跟踪值与最大值的接近程度以及在什么时间发送什么消息的功能。使用我们库的应用程序需要提供发送消息的机制：应用程序可以将消息放入应用程序中，发送电子邮件，发送短信，或者做其他事情。库不需要知道这些细节。它只需要一个实现了我们将提供的 `Messenger` trait 的东西。Listing 15-20 显示了库代码。

<Listing number="15-20" file-name="src/lib.rs" caption="A library to keep track of how close a value is to a maximum value and warn when the value is at certain levels">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

这段代码的一个重要部分是 `Messenger` trait 有一个名为 `send` 的方法，它接受对 `self` 的不可变引用和消息的文本。这个 trait 是我们的模拟对象需要实现的接口，以便模拟对象可以像真实对象一样使用。另一个重要部分是我们想要测试 `LimitTracker` 上的 `set_value` 方法的行为。我们可以改变传递给 `value` 参数的内容，但 `set_value` 不会返回任何东西供我们进行断言。我们希望能够说，如果我们创建了一个 `LimitTracker`，它使用实现了 `Messenger` trait 的东西和一个特定的 `max` 值，当我们传递不同的 `value` 值时，messenger 会被告知发送适当的消息。

我们需要一个模拟对象，当调用 `send` 时，它不会发送电子邮件或短信，而只会跟踪它被告知要发送的消息。我们可以创建一个模拟对象的新实例，创建一个使用模拟对象的 `LimitTracker`，调用 `LimitTracker` 上的 `set_value` 方法，然后检查模拟对象是否有我们期望的消息。Listing 15-21 展示了一个尝试实现模拟对象的代码，但借用检查器不允许这样做。

<Listing number="15-21" file-name="src/lib.rs" caption="An attempt to implement a `MockMessenger` that isn’t allowed by the borrow checker">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

这个测试代码定义了一个 `MockMessenger` 结构体，它有一个 `sent_messages` 字段，其中包含一个 `String` 值的 `Vec`，用于跟踪它被告知要发送的消息。我们还定义了一个关联函数 `new`，以便方便地创建新的 `MockMessenger` 值，这些值从一个空的消息列表开始。然后我们为 `MockMessenger` 实现了 `Messenger` trait，以便我们可以将 `MockMessenger` 提供给 `LimitTracker`。在 `send` 方法的定义中，我们将作为参数传递的消息存储在 `MockMessenger` 的 `sent_messages` 列表中。

在测试中，我们测试当 `LimitTracker` 被告知将 `value` 设置为超过 `max` 值的 75% 时会发生什么。首先，我们创建一个新的 `MockMessenger`，它将从一个空的消息列表开始。然后我们创建一个新的 `LimitTracker`，并给它一个新的 `MockMessenger` 的引用和一个 `max` 值为 `100`。我们调用 `LimitTracker` 上的 `set_value` 方法，传递一个值为 `80`，这超过了 100 的 75%。然后我们断言 `MockMessenger` 正在跟踪的消息列表现在应该有一条消息。

然而，这个测试有一个问题，如下所示：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

我们无法修改 `MockMessenger` 来跟踪消息，因为 `send` 方法接受对 `self` 的不可变引用。我们也不能接受错误文本中的建议，在 `impl` 方法和 `trait` 定义中都使用 `&mut self`。我们不想仅仅为了测试而改变 `Messenger` trait。相反，我们需要找到一种方法，使我们的测试代码能够与现有设计正确工作。

这是一个内部可变性可以发挥作用的情况！我们将 `sent_messages` 存储在 `RefCell<T>` 中，然后 `send` 方法将能够修改 `sent_messages` 以存储我们看到的消息。Listing 15-22 展示了这一点。

<Listing number="15-22" file-name="src/lib.rs" caption="Using `RefCell<T>` to mutate an inner value while the outer value is considered immutable">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

`sent_messages` 字段现在是 `RefCell<Vec<String>>` 类型，而不是 `Vec<String>`。在 `new` 函数中，我们创建了一个新的 `RefCell<Vec<String>>` 实例，包装了一个空向量。

对于 `send` 方法的实现，第一个参数仍然是对 `self` 的不可变借用，这与 trait 定义匹配。我们调用 `self.sent_messages` 中的 `RefCell<Vec<String>>` 的 `borrow_mut` 方法，以获取对 `RefCell<Vec<String>>` 内部值的可变引用，即向量。然后我们可以对向量的可变引用调用 `push` 来跟踪测试期间发送的消息。

我们必须做的最后一个更改是在断言中：为了查看内部向量中有多少项，我们调用 `RefCell<Vec<String>>` 的 `borrow` 方法以获取对向量的不可变引用。

现在你已经看到了如何使用 `RefCell<T>`，让我们深入了解它的工作原理！

#### 使用 `RefCell<T>` 在运行时跟踪借用

在创建不可变和可变引用时，我们分别使用 `&` 和 `&mut` 语法。对于 `RefCell<T>`，我们使用 `borrow` 和 `borrow_mut` 方法，它们是 `RefCell<T>` 的安全 API 的一部分。`borrow` 方法返回智能指针类型 `Ref<T>`，而 `borrow_mut` 返回智能指针类型 `RefMut<T>`。这两种类型都实现了 `Deref`，所以我们可以像对待常规引用一样对待它们。

`RefCell<T>` 跟踪当前有多少 `Ref<T>` 和 `RefMut<T>` 智能指针是活动的。每次我们调用 `borrow` 时，`RefCell<T>` 都会增加其活动不可变借用的计数。当 `Ref<T>` 值超出范围时，不可变借用的计数会减少 1。就像编译时的借用规则一样，`RefCell<T>` 允许我们在任何时候拥有多个不可变借用或一个可变借用。

如果我们试图违反这些规则，与引用不同，我们不会得到编译错误，而是 `RefCell<T>` 的实现会在运行时 panic。Listing 15-23 展示了 Listing 15-22 中 `send` 实现的修改。我们故意尝试在同一作用域内创建两个活动的可变借用，以说明 `RefCell<T>` 会在运行时阻止我们这样做。

<Listing number="15-23" file-name="src/lib.rs" caption="Creating two mutable references in the same scope to see that `RefCell<T>` will panic">

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

我们为 `borrow_mut` 返回的 `RefMut<T>` 智能指针创建了一个变量 `one_borrow`。然后我们以同样的方式在变量 `two_borrow` 中创建了另一个可变借用。这使得在同一作用域内有两个可变引用，这是不允许的。当我们运行库的测试时，Listing 15-23 中的代码将编译通过，但测试会失败：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

注意，代码 panic 并显示消息 `already borrowed: BorrowMutError`。这是 `RefCell<T>` 在运行时处理借用规则违规的方式。

选择在运行时而不是编译时捕获借用错误，正如我们在这里所做的，意味着你可能会在开发过程的后期发现代码中的错误：可能直到你的代码部署到生产环境时才发现。此外，由于在运行时而不是编译时跟踪借用，你的代码会遭受一点运行时性能损失。然而，使用 `RefCell<T>` 使得编写一个模拟对象成为可能，该对象可以在只允许不可变值的上下文中修改自身以跟踪它看到的消息。尽管有这些权衡，你仍然可以使用 `RefCell<T>` 来获得比常规引用提供的更多功能。

<!-- Old link, do not remove -->

<a id="having-multiple-owners-of-mutable-data-by-combining-rc-t-and-ref-cell-t"></a>

### 使用 `Rc<T>` 和 `RefCell<T>` 允许多个所有者拥有可变数据

使用 `RefCell<T>` 的一种常见方式是与 `Rc<T>` 结合使用。回想一下，`Rc<T>` 允许你拥有某些数据的多个所有者，但它只提供对该数据的不可变访问。如果你有一个持有 `RefCell<T>` 的 `Rc<T>`，你可以获得一个可以有多个所有者 _并且_ 你可以修改的值！

例如，回想一下 Listing 15-18 中的 cons 列表示例，我们使用 `Rc<T>` 允许多个列表共享另一个列表的所有权。因为 `Rc<T>` 只持有不可变的值，所以我们一旦创建了列表中的值，就无法更改它们。让我们添加 `RefCell<T>` 以改变列表中的值。Listing 15-24 展示了通过在 `Cons` 定义中使用 `RefCell<T>`，我们可以修改所有列表中的值。

<Listing number="15-24" file-name="src/main.rs" caption="Using `Rc<RefCell<i32>>` to create a `List` that we can mutate">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

我们创建了一个 `Rc<RefCell<i32>>` 的实例，并将其存储在一个名为 `value` 的变量中，以便稍后可以直接访问它。然后我们在 `a` 中创建了一个 `List`，其中包含一个持有 `value` 的 `Cons` 变体。我们需要克隆 `value`，以便 `a` 和 `value` 都拥有内部值 `5` 的所有权，而不是将所有权从 `value` 转移到 `a` 或让 `a` 从 `value` 借用。

我们将列表 `a` 包装在 `Rc<T>` 中，以便当我们创建列表 `b` 和 `c` 时，它们都可以引用 `a`，正如我们在 Listing 15-18 中所做的那样。

在我们创建了 `a`、`b` 和 `c` 中的列表之后，我们想要将 `value` 中的值增加 10。我们通过调用 `value` 上的 `borrow_mut` 来实现这一点，它使用了我们在第 5 章中讨论的自动解引用功能（[“`->` 运算符在哪里？”][wheres-the---operator]<!-- ignore -->）来解引用 `Rc<T>` 到内部的 `RefCell<T>` 值。`borrow_mut` 方法返回一个 `RefMut<T>` 智能指针，我们使用解引用操作符来改变内部值。

当我们打印 `a`、`b` 和 `c` 时，我们可以看到它们都有修改后的值 `15` 而不是 `5`：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

这个技巧非常巧妙！通过使用 `RefCell<T>`，我们有一个表面上不可变的 `List` 值。但我们可以使用 `RefCell<T>` 上的方法来访问其内部可变性，以便在需要时修改我们的数据。借用规则的运行时检查保护我们免受数据竞争的影响，有时为了数据结构的灵活性而牺牲一点速度是值得的。请注意，`RefCell<T>` 不适用于多线程代码！`Mutex<T>` 是 `RefCell<T>` 的线程安全版本，我们将在第 16 章讨论 `Mutex<T>`。

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator