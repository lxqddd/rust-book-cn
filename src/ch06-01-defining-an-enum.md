## 定义枚举

结构体提供了一种将相关字段和数据组合在一起的方式，例如带有 `width` 和 `height` 的 `Rectangle`，而枚举则提供了一种表示一个值是可能的值集合中的一种的方式。例如，我们可能想说 `Rectangle` 是可能形状集合中的一种，该集合还包括 `Circle` 和 `Triangle`。为了实现这一点，Rust 允许我们将这些可能性编码为枚举。

让我们来看一个我们可能希望在代码中表达的情况，并了解为什么在这种情况下枚举比结构体更有用且更合适。假设我们需要处理 IP 地址。目前，IP 地址有两个主要标准：版本四和版本六。因为我们的程序只会遇到这两种 IP 地址，所以我们可以**枚举**所有可能的变体，这也是枚举得名的原因。

任何 IP 地址要么是版本四，要么是版本六，但不能同时是两者。IP 地址的这一特性使得枚举数据结构非常合适，因为枚举值只能是其变体之一。版本四和版本六的地址从根本上说仍然是 IP 地址，因此当代码处理适用于任何类型 IP 地址的情况时，它们应该被视为同一类型。

我们可以通过定义一个 `IpAddrKind` 枚举并在其中列出 IP 地址可能的种类 `V4` 和 `V6` 来表达这个概念。这些是枚举的变体：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind` 现在是一个自定义数据类型，我们可以在代码的其他地方使用它。

### 枚举值

我们可以像这样创建 `IpAddrKind` 的两个变体的实例：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

注意，枚举的变体在其标识符下命名空间化，我们使用双冒号来分隔两者。这很有用，因为现在 `IpAddrKind::V4` 和 `IpAddrKind::V6` 都是同一类型：`IpAddrKind`。然后，我们可以定义一个接受任何 `IpAddrKind` 的函数：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

我们可以用任一变体调用这个函数：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

使用枚举还有更多优势。再想想我们的 IP 地址类型，目前我们没有办法存储实际的 IP 地址**数据**；我们只知道它的**种类**。鉴于你刚刚在第 5 章学习了结构体，你可能会想用结构体来解决这个问题，如 Listing 6-1 所示。

<Listing number="6-1" caption="使用 `struct` 存储 IP 地址的数据和 `IpAddrKind` 变体">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

</Listing>

在这里，我们定义了一个结构体 `IpAddr`，它有两个字段：一个 `kind` 字段，类型为 `IpAddrKind`（我们之前定义的枚举），以及一个 `address` 字段，类型为 `String`。我们有两个这个结构体的实例。第一个是 `home`，它的 `kind` 值为 `IpAddrKind::V4`，并带有相关的地址数据 `127.0.0.1`。第二个实例是 `loopback`。它的 `kind` 值是 `IpAddrKind` 的另一个变体 `V6`，并带有地址 `::1`。我们使用了一个结构体将 `kind` 和 `address` 值捆绑在一起，所以现在变体与值相关联。

然而，仅使用枚举来表示相同的概念更加简洁：我们不必将枚举放在结构体中，而是可以直接将数据放入每个枚举变体中。这个新的 `IpAddr` 枚举定义表示 `V4` 和 `V6` 变体都将具有关联的 `String` 值：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

我们将数据直接附加到枚举的每个变体上，因此不需要额外的结构体。在这里，也更容易看到枚举工作的另一个细节：我们定义的每个枚举变体的名称也变成了一个构造枚举实例的函数。也就是说，`IpAddr::V4()` 是一个函数调用，它接受一个 `String` 参数并返回一个 `IpAddr` 类型的实例。我们自动获得了这个构造函数，作为定义枚举的结果。

使用枚举而不是结构体还有另一个优势：每个变体可以具有不同类型和数量的关联数据。版本四的 IP 地址总是有四个数值组件，其值在 0 到 255 之间。如果我们想将 `V4` 地址存储为四个 `u8` 值，但仍然将 `V6` 地址表示为一个 `String` 值，我们无法用结构体做到这一点。枚举可以轻松处理这种情况：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

我们已经展示了多种定义数据结构来存储版本四和版本六 IP 地址的方法。然而，事实证明，存储 IP 地址并编码其类型是如此常见，以至于[标准库中有一个我们可以使用的定义！][IpAddr]<!-- ignore --> 让我们看看标准库如何定义 `IpAddr`：它拥有我们定义和使用的确切枚举和变体，但它将地址数据嵌入到变体中，形式为两个不同的结构体，每个变体的定义方式不同：

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

这段代码说明你可以将任何类型的数据放入枚举变体中：字符串、数值类型或结构体，例如。你甚至可以包含另一个枚举！此外，标准库类型通常不会比你可能想出的复杂得多。

注意，即使标准库包含 `IpAddr` 的定义，我们仍然可以创建和使用我们自己的定义而不会冲突，因为我们没有将标准库的定义引入我们的作用域。我们将在第 7 章中详细讨论将类型引入作用域。

让我们看看 Listing 6-2 中的另一个枚举示例：这个枚举的变体中嵌入了多种类型。

<Listing number="6-2" caption="一个 `Message` 枚举，其每个变体存储不同数量和类型的值">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

</Listing>

这个枚举有四个变体，每个变体具有不同的类型：

- `Quit` 没有任何关联的数据。
- `Move` 有命名字段，就像结构体一样。
- `Write` 包含一个 `String`。
- `ChangeColor` 包含三个 `i32` 值。

定义像 Listing 6-2 中这样的枚举变体类似于定义不同种类的结构体定义，只是枚举不使用 `struct` 关键字，并且所有变体都分组在 `Message` 类型下。以下结构体可以保存与前面枚举变体相同的数据：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

但如果我们使用不同的结构体，每个结构体都有自己的类型，我们就无法像使用 Listing 6-2 中定义的 `Message` 枚举那样轻松地定义一个函数来接受这些消息中的任何一种，因为 `Message` 是一个单一类型。

枚举和结构体之间还有一个相似之处：正如我们可以使用 `impl` 在结构体上定义方法一样，我们也可以在枚举上定义方法。这里有一个名为 `call` 的方法，我们可以在 `Message` 枚举上定义它：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

方法的主体将使用 `self` 来获取我们调用方法的值。在这个例子中，我们创建了一个变量 `m`，它的值是 `Message::Write(String::from("hello"))`，这就是当 `m.call()` 运行时，`call` 方法主体中的 `self` 的值。

让我们看看标准库中另一个非常常见且有用的枚举：`Option`。

### `Option` 枚举及其相对于空值的优势

本节探讨了 `Option` 的案例研究，这是标准库定义的另一个枚举。`Option` 类型编码了一个非常常见的场景，即一个值可能是某个东西，也可能是空。

例如，如果你请求一个非空列表中的第一个项目，你会得到一个值。如果你请求一个空列表中的第一个项目，你会得到空。在类型系统中表达这个概念意味着编译器可以检查你是否处理了所有应该处理的情况；这个功能可以防止其他编程语言中非常常见的错误。

编程语言设计通常被认为是你包含哪些特性，但你排除的特性也很重要。Rust 没有许多其他语言中存在的空值特性。**空**是一个表示没有值的值。在有空值的语言中，变量总是处于两种状态之一：空或非空。

在 2009 年的演讲“空引用：十亿美元的错误”中，空的发明者 Tony Hoare 这样说：

> 我称之为我的十亿美元错误。当时，我正在为面向对象语言中的引用设计第一个全面的类型系统。我的目标是确保所有引用的使用都应该是绝对安全的，由编译器自动执行检查。但我无法抗拒放入空引用的诱惑，仅仅因为它太容易实现了。这导致了无数的错误、漏洞和系统崩溃，可能在过去的四十年中造成了十亿美元的痛苦和损失。

空值的问题在于，如果你尝试将空值用作非空值，你会得到某种错误。因为这种空或非空的属性无处不在，所以很容易犯这种错误。

然而，空试图表达的概念仍然是有用的：空是一个由于某种原因当前无效或缺失的值。

问题并不在于概念本身，而在于特定的实现。因此，Rust 没有空值，但它有一个枚举可以编码值存在或缺失的概念。这个枚举是 `Option<T>`，它由[标准库定义][option]<!-- ignore -->如下：

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`Option<T>` 枚举非常有用，以至于它甚至被包含在预导入模块中；你不需要显式地将其引入作用域。它的变体也被包含在预导入模块中：你可以直接使用 `Some` 和 `None`，而不需要 `Option::` 前缀。`Option<T>` 枚举仍然只是一个常规的枚举，`Some(T)` 和 `None` 仍然是 `Option<T>` 类型的变体。

`<T>` 语法是 Rust 的一个特性，我们还没有讨论过。它是一个泛型类型参数，我们将在第 10 章中更详细地介绍泛型。现在，你只需要知道 `<T>` 意味着 `Option` 枚举的 `Some` 变体可以保存一个任意类型的数据，并且每个用于替换 `T` 的具体类型都会使整个 `Option<T>` 类型成为不同的类型。以下是一些使用 `Option` 值来保存数字类型和字符类型的示例：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

`some_number` 的类型是 `Option<i32>`。`some_char` 的类型是 `Option<char>`，这是一个不同的类型。Rust 可以推断这些类型，因为我们在 `Some` 变体中指定了一个值。对于 `absent_number`，Rust 要求我们注释整个 `Option` 类型：编译器无法仅通过查看 `None` 值来推断相应的 `Some` 变体将保存的类型。在这里，我们告诉 Rust 我们希望 `absent_number` 是 `Option<i32>` 类型。

当我们有一个 `Some` 值时，我们知道一个值是存在的，并且该值保存在 `Some` 中。当我们有一个 `None` 值时，在某种意义上它与空值相同：我们没有有效的值。那么为什么拥有 `Option<T>` 比拥有空值更好呢？

简而言之，因为 `Option<T>` 和 `T`（其中 `T` 可以是任何类型）是不同的类型，编译器不会让我们像使用一个确定有效的值那样使用 `Option<T>` 值。例如，这段代码不会编译，因为它试图将一个 `i8` 添加到一个 `Option<i8>`：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

如果我们运行这段代码，我们会得到类似这样的错误信息：

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

激烈！实际上，这个错误信息意味着 Rust 不知道如何将 `i8` 和 `Option<i8>` 相加，因为它们是不同的类型。当我们在 Rust 中有一个像 `i8` 这样的类型的值时，编译器将确保我们总是有一个有效的值。我们可以自信地继续，而不必在使用该值之前检查空值。只有当我们有一个 `Option<i8>`（或我们正在处理的任何类型的值）时，我们才需要担心可能没有值，编译器将确保我们在使用该值之前处理这种情况。

换句话说，你必须将 `Option<T>` 转换为 `T`，然后才能对其执行 `T` 操作。通常，这有助于捕捉与空值相关的最常见问题之一：假设某些东西不是空值，而实际上它是。

消除错误假设非空值的风险有助于你在代码中更加自信。为了拥有一个可能为空的值，你必须通过将该值的类型设为 `Option<T>` 来显式选择加入。然后，当你使用该值时，你必须显式处理值为空的情况。在任何值类型不是 `Option<T>` 的地方，你**可以**安全地假设该值不是空值。这是 Rust 为了限制空值的普遍性并增加 Rust 代码的安全性而做出的有意设计决策。

那么当你有一个 `Option<T>` 类型的值时，如何从 `Some` 变体中获取 `T` 值以便使用该值呢？`Option<T>` 枚举有大量在各种情况下有用的方法；你可以在[其文档][docs]<!-- ignore -->中查看它们。熟悉 `Option<T>` 上的方法将在你使用 Rust 的过程中非常有用。

通常，为了使用 `Option<T>` 值，你希望有一些代码来处理每个变体。你希望有一些代码只在你有 `Some(T)` 值时运行，并且这段代码可以使用内部的 `T`。你希望有一些其他代码只在你有 `None` 值时运行，并且这段代码没有可用的 `T` 值。`match` 表达式是一个控制流结构，当与枚举一起使用时，它正好能做到这一点：它将根据枚举的变体运行不同的代码，并且该代码可以使用匹配值内部的数据。

[IpAddr]: ../std/net/enum.IpAddr.html
[option]: ../std/option/enum.Option.html
[docs]: ../std/option/enum.Option.html