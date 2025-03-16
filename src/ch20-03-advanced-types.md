## 高级类型

Rust 的类型系统有一些我们之前提到但尚未讨论的特性。我们将从讨论 newtype 模式开始，探讨为什么 newtype 作为类型是有用的。然后我们将转向类型别名，这是一个与 newtype 类似但语义略有不同的特性。我们还将讨论 `!` 类型和动态大小类型。

### 使用 Newtype 模式实现类型安全和抽象

本节假设你已经阅读了前面的章节 [“使用 Newtype 模式在外部类型上实现外部特性。”][using-the-newtype-pattern]<!-- ignore --> newtype 模式在我们目前讨论的任务之外也非常有用，包括静态地确保值永远不会混淆，并指示值的单位。你在 Listing 20-16 中看到了使用 newtype 来指示单位的示例：回想一下，`Millimeters` 和 `Meters` 结构体将 `u32` 值包装在 newtype 中。如果我们编写一个参数类型为 `Millimeters` 的函数，我们将无法编译一个意外地尝试使用 `Meters` 类型的值或普通的 `u32` 值调用该函数的程序。

我们还可以使用 newtype 模式来抽象掉类型的某些实现细节：新类型可以公开一个与私有内部类型的 API 不同的公共 API。

Newtype 还可以隐藏内部实现。例如，我们可以提供一个 `People` 类型来包装一个 `HashMap<i32, String>`，它存储与名称相关联的人的 ID。使用 `People` 的代码只会与我们提供的公共 API 交互，例如向 `People` 集合添加名称字符串的方法；该代码不需要知道我们在内部为名称分配了一个 `i32` ID。newtype 模式是一种轻量级的方式来实现封装以隐藏实现细节，我们在第 18 章的 [“隐藏实现细节的封装”][encapsulation-that-hides-implementation-details]<!-- ignore --> 中讨论过。

### 使用类型别名创建类型同义词

Rust 提供了声明 _类型别名_ 的能力，以便为现有类型提供另一个名称。为此，我们使用 `type` 关键字。例如，我们可以创建别名 `Kilometers` 为 `i32`，如下所示：

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

现在，别名 `Kilometers` 是 `i32` 的 _同义词_；与我们在 Listing 20-16 中创建的 `Millimeters` 和 `Meters` 类型不同，`Kilometers` 不是一个单独的、新的类型。具有 `Kilometers` 类型的值将与 `i32` 类型的值相同对待：

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

因为 `Kilometers` 和 `i32` 是相同的类型，我们可以将这两种类型的值相加，并且我们可以将 `Kilometers` 值传递给接受 `i32` 参数的函数。然而，使用这种方法，我们不会得到之前讨论的 newtype 模式所带来的类型检查的好处。换句话说，如果我们在某处混淆了 `Kilometers` 和 `i32` 值，编译器不会给我们错误。

类型同义词的主要用例是减少重复。例如，我们可能有一个冗长的类型，如下所示：

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

在函数签名和代码中的类型注释中到处写这个冗长的类型可能会很繁琐且容易出错。想象一下，有一个项目充满了像 Listing 20-25 中的代码。

<Listing number="20-25" caption="在许多地方使用长类型">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

类型别名通过减少重复使代码更易于管理。在 Listing 20-26 中，我们为冗长的类型引入了一个名为 `Thunk` 的别名，并可以用较短的别名 `Thunk` 替换所有使用该类型的地方。

<Listing number="20-26" caption="引入类型别名 `Thunk` 以减少重复">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

这段代码更容易阅读和编写！为类型别名选择一个有意义的名称可以帮助传达你的意图（_thunk_ 是一个稍后评估的代码的词，因此它是一个存储闭包的合适名称）。

类型别名也常用于 `Result<T, E>` 类型以减少重复。考虑标准库中的 `std::io` 模块。I/O 操作通常返回一个 `Result<T, E>` 来处理操作失败的情况。这个库有一个 `std::io::Error` 结构体，表示所有可能的 I/O 错误。`std::io` 中的许多函数将返回 `Result<T, E>`，其中 `E` 是 `std::io::Error`，例如 `Write` 特性中的这些函数：

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

`Result<..., Error>` 重复了很多次。因此，`std::io` 有这个类型别名声明：

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

因为这个声明在 `std::io` 模块中，我们可以使用完全限定的别名 `std::io::Result<T>`；也就是说，一个 `Result<T, E>`，其中 `E` 被填充为 `std::io::Error`。`Write` 特性的函数签名最终看起来像这样：

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

类型别名在两个方面有帮助：它使代码更容易编写 _并且_ 它为我们提供了一个跨所有 `std::io` 的一致接口。因为它是一个别名，它只是另一个 `Result<T, E>`，这意味着我们可以使用任何适用于 `Result<T, E>` 的方法，以及像 `?` 操作符这样的特殊语法。

### 永不返回的 Never 类型

Rust 有一个名为 `!` 的特殊类型，在类型理论术语中称为 _空类型_，因为它没有值。我们更喜欢称它为 _never 类型_，因为它在函数永远不会返回时代表返回类型。以下是一个示例：

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

这段代码被解读为“函数 `bar` 返回 never。”返回 never 的函数称为 _发散函数_。我们无法创建 `!` 类型的值，因此 `bar` 永远不可能返回。

但是，一个你永远无法创建值的类型有什么用呢？回想一下 Listing 2-5 中的代码，数字猜谜游戏的一部分；我们在 Listing 20-27 中重现了其中的一部分。

<Listing number="20-27" caption="一个以 `continue` 结尾的 `match` 分支">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

当时，我们跳过了这段代码中的一些细节。在第 6 章的 [“`match` 控制流操作符”][the-match-control-flow-operator]<!-- ignore --> 中，我们讨论了 `match` 分支必须都返回相同的类型。因此，例如，以下代码不起作用：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

这段代码中的 `guess` 类型必须是一个整数 _和_ 一个字符串，而 Rust 要求 `guess` 只有一个类型。那么 `continue` 返回什么？我们如何在 Listing 20-27 中从一个分支返回 `u32` 并让另一个分支以 `continue` 结尾？

正如你可能已经猜到的，`continue` 有一个 `!` 值。也就是说，当 Rust 计算 `guess` 的类型时，它会查看两个 `match` 分支，前者有一个 `u32` 值，后者有一个 `!` 值。因为 `!` 永远不会有值，Rust 决定 `guess` 的类型是 `u32`。

描述这种行为的形式化方式是，类型为 `!` 的表达式可以被强制转换为任何其他类型。我们被允许以 `continue` 结束这个 `match` 分支，因为 `continue` 不会返回值；相反，它将控制权移回循环的顶部，因此在 `Err` 情况下，我们永远不会为 `guess` 赋值。

never 类型与 `panic!` 宏也很有用。回想一下我们在 `Option<T>` 值上调用的 `unwrap` 函数，以生成一个值或 panic，其定义如下：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

在这段代码中，发生了与 Listing 20-27 中的 `match` 相同的事情：Rust 看到 `val` 的类型是 `T`，而 `panic!` 的类型是 `!`，因此整个 `match` 表达式的结果是 `T`。这段代码有效是因为 `panic!` 不会产生值；它结束了程序。在 `None` 情况下，我们不会从 `unwrap` 返回值，因此这段代码是有效的。

最后一个具有 `!` 类型的表达式是 `loop`：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

在这里，循环永远不会结束，因此 `!` 是表达式的值。然而，如果我们包含一个 `break`，这将不再成立，因为循环将在到达 `break` 时终止。

### 动态大小类型和 `Sized` 特性

Rust 需要知道其类型的某些细节，例如为特定类型的值分配多少空间。这使得其类型系统的一个角落在一开始有点令人困惑：_动态大小类型_ 的概念。有时称为 _DSTs_ 或 _未大小类型_，这些类型让我们可以编写使用值的代码，这些值的大小我们只能在运行时知道。

让我们深入了解一个名为 `str` 的动态大小类型的细节，我们在整本书中一直在使用它。没错，不是 `&str`，而是 `str` 本身，是一个 DST。我们无法知道字符串的长度直到运行时，这意味着我们无法创建类型为 `str` 的变量，也无法接受类型为 `str` 的参数。考虑以下代码，它不起作用：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust 需要知道为特定类型的任何值分配多少内存，并且一个类型的所有值必须使用相同数量的内存。如果 Rust 允许我们编写这段代码，这两个 `str` 值将需要占用相同数量的空间。但它们有不同的长度：`s1` 需要 12 字节的存储空间，而 `s2` 需要 15 字节。这就是为什么不可能创建一个持有动态大小类型的变量。

那么我们该怎么办？在这种情况下，你已经知道答案：我们将 `s1` 和 `s2` 的类型改为 `&str` 而不是 `str`。回想一下第 4 章的 [“字符串切片”][string-slices]<!-- ignore -->，切片数据结构只存储切片的起始位置和长度。因此，尽管 `&T` 是一个存储 `T` 所在内存地址的单个值，`&str` 是 _两个_ 值：`str` 的地址和它的长度。因此，我们可以在编译时知道 `&str` 值的大小：它是 `usize` 长度的两倍。也就是说，我们总是知道 `&str` 的大小，无论它引用的字符串有多长。一般来说，这是在 Rust 中使用动态大小类型的方式：它们有一个额外的元数据位，用于存储动态信息的大小。动态大小类型的黄金法则是，我们必须始终将动态大小类型的值放在某种指针后面。

我们可以将 `str` 与各种指针结合使用：例如，`Box<str>` 或 `Rc<str>`。事实上，你以前见过这个，但使用了一个不同的动态大小类型：特性。每个特性都是一个动态大小类型，我们可以通过使用特性的名称来引用它。在第 18 章的 [“使用允许不同类型值的特性对象”][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore --> 中，我们提到要使用特性作为特性对象，我们必须将它们放在指针后面，例如 `&dyn Trait` 或 `Box<dyn Trait>`（`Rc<dyn Trait>` 也可以工作）。

为了处理动态大小类型（DSTs），Rust 提供了 `Sized` trait 来确定一个类型的大小是否在编译时已知。这个 trait 会自动为所有在编译时已知大小的类型实现。此外，Rust 隐式地为每个泛型函数添加了 `Sized` 的约束。也就是说，像这样的泛型函数定义：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

实际上会被视为我们写了这样的代码：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

默认情况下，泛型函数只会作用于那些在编译时已知大小的类型。然而，你可以使用以下特殊语法来放宽这个限制：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

`?Sized` 的 trait 约束意味着“`T` 可能是 `Sized`，也可能不是 `Sized`”，这种表示法覆盖了泛型类型必须在编译时已知大小的默认行为。这种含义的 `?Trait` 语法仅适用于 `Sized`，不适用于其他 trait。

还要注意，我们将 `t` 参数的类型从 `T` 切换为 `&T`。因为类型可能不是 `Sized`，我们需要在某种指针后面使用它。在这种情况下，我们选择了引用。

接下来，我们将讨论函数和闭包！

[encapsulation-that-hides-implementation-details]: ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-operator]: ch06-02-match.html#the-match-control-flow-operator
[using-trait-objects-that-allow-for-values-of-different-types]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[using-the-newtype-pattern]: ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types