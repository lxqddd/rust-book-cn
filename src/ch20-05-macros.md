## 宏

我们在本书中已经使用了像 `println!` 这样的宏，但还没有完全探讨宏是什么以及它是如何工作的。术语 **宏** 指的是 Rust 中的一系列特性：使用 `macro_rules!` 的 **声明式** 宏和三种 **过程式** 宏：

- 自定义 `#[derive]` 宏，用于指定通过 `derive` 属性在结构体和枚举上添加的代码
- 类属性宏，用于定义可用于任何项的自定义属性
- 类函数宏，看起来像函数调用，但操作的是作为参数传递的标记

我们将依次讨论这些宏，但首先，让我们看看为什么在已经有函数的情况下还需要宏。

### 宏与函数的区别

从根本上说，宏是一种编写代码的方式，它会生成其他代码，这被称为 **元编程**。在附录 C 中，我们讨论了 `derive` 属性，它会为你生成各种 trait 的实现。我们在书中也使用了 `println!` 和 `vec!` 宏。所有这些宏都会 **展开** 以生成比你手动编写的代码更多的代码。

元编程对于减少你需要编写和维护的代码量非常有用，这也是函数的作用之一。然而，宏有一些函数不具备的额外能力。

函数的签名必须声明函数的参数数量和类型。而宏可以接受可变数量的参数：我们可以用一个参数调用 `println!("hello")`，也可以用两个参数调用 `println!("hello {}", name)`。此外，宏在编译器解释代码的含义之前就已经展开了，因此宏可以在给定的类型上实现 trait。函数则不行，因为函数是在运行时调用的，而 trait 需要在编译时实现。

实现宏而不是函数的缺点是宏定义比函数定义更复杂，因为你是在编写 Rust 代码来生成 Rust 代码。由于这种间接性，宏定义通常比函数定义更难阅读、理解和维护。

宏和函数之间的另一个重要区别是，你必须在文件中调用宏之前定义宏或将宏引入作用域，而函数则可以在任何地方定义并在任何地方调用。

### 使用 `macro_rules!` 进行通用元编程的声明式宏

Rust 中最广泛使用的宏形式是 **声明式宏**。这些宏有时也被称为“示例宏”、“`macro_rules!` 宏”或简称为“宏”。声明式宏的核心是允许你编写类似于 Rust `match` 表达式的代码。正如第 6 章所讨论的，`match` 表达式是一种控制结构，它接受一个表达式，将表达式的结果值与模式进行比较，然后运行与匹配模式相关的代码。宏也会将一个值与模式进行比较，这些模式与特定的代码相关联：在这种情况下，值是传递给宏的 Rust 源代码字面量；模式与该源代码的结构进行比较；当匹配时，与每个模式相关的代码会替换传递给宏的代码。这一切都发生在编译期间。

要定义一个宏，你可以使用 `macro_rules!` 结构。让我们通过查看 `vec!` 宏的定义来探索如何使用 `macro_rules!`。第 8 章介绍了如何使用 `vec!` 宏来创建一个包含特定值的新向量。例如，以下宏创建了一个包含三个整数的新向量：

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

我们也可以使用 `vec!` 宏来创建一个包含两个整数的向量或一个包含五个字符串切片的向量。我们无法使用函数来完成同样的操作，因为我们无法提前知道值的数量或类型。

Listing 20-35 展示了 `vec!` 宏的一个简化版本的定义。

<Listing number="20-35" file-name="src/lib.rs" caption="`vec!` 宏定义的简化版本">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> 注意：标准库中 `vec!` 宏的实际定义包含了预先分配正确内存的代码。为了简化示例，我们没有包含这段代码。

`#[macro_export]` 注解表示每当定义宏的 crate 被引入作用域时，这个宏应该可用。如果没有这个注解，宏将无法被引入作用域。

然后我们以 `macro_rules!` 和宏的名称（**不带** 感叹号）开始宏定义。在这个例子中，名称是 `vec`，后面跟着表示宏定义主体的花括号。

`vec!` 主体中的结构与 `match` 表达式的结构类似。这里我们有一个模式为 `( $( $x:expr ),* )` 的分支，后面跟着 `=>` 和与该模式相关的代码块。如果模式匹配，相关的代码块将被生成。由于这是该宏中唯一的模式，因此只有一种有效的匹配方式；任何其他模式都会导致错误。更复杂的宏会有多个分支。

宏定义中的有效模式语法与第 19 章中介绍的模式语法不同，因为宏模式是与 Rust 代码结构匹配的，而不是与值匹配。让我们逐步解释 Listing 20-29 中的模式片段；完整的宏模式语法请参阅 [Rust 参考手册][ref]。

首先，我们使用一组括号来包围整个模式。我们使用美元符号 (`$`) 来声明宏系统中的一个变量，该变量将包含与模式匹配的 Rust 代码。美元符号清楚地表明这是一个宏变量，而不是普通的 Rust 变量。接下来是一组括号，用于捕获与括号内模式匹配的值，以便在替换代码中使用。在 `$()` 内部是 `$x:expr`，它匹配任何 Rust 表达式并将表达式命名为 `$x`。

`$()` 后面的逗号表示在匹配 `$()` 内代码的每个实例之间必须出现一个字面逗号分隔符。`*` 指定该模式匹配零个或多个 `*` 前面的内容。

当我们使用 `vec![1, 2, 3];` 调用这个宏时，`$x` 模式会与三个表达式 `1`、`2` 和 `3` 匹配三次。

现在让我们看看与该分支相关的代码主体中的模式：`temp_vec.push()` 在 `$()*` 内生成，用于每个匹配 `$()` 的部分，根据模式匹配的次数生成零次或多次。`$x` 被替换为每个匹配的表达式。当我们使用 `vec![1, 2, 3];` 调用这个宏时，生成的代码将替换这个宏调用，如下所示：

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

我们已经定义了一个宏，它可以接受任意数量的任意类型的参数，并可以生成代码来创建一个包含指定元素的向量。

要了解更多关于如何编写宏的信息，请查阅在线文档或其他资源，例如由 Daniel Keep 开始并由 Lukas Wirth 继续编写的 [“The Little Book of Rust Macros”][tlborm]。

### 用于从属性生成代码的过程式宏

第二种形式的宏是过程式宏，它的行为更像一个函数（并且是一种过程）。**过程式宏** 接受一些代码作为输入，对这些代码进行操作，并生成一些代码作为输出，而不是像声明式宏那样匹配模式并用其他代码替换代码。过程式宏有三种：自定义 `derive`、类属性和类函数宏，它们的工作方式类似。

在创建过程式宏时，定义必须位于具有特殊 crate 类型的独立 crate 中。这是由于复杂的技术原因，我们希望在未来消除这一限制。在 Listing 20-36 中，我们展示了如何定义一个过程式宏，其中 `some_attribute` 是使用特定宏变体的占位符。

<Listing number="20-36" file-name="src/lib.rs" caption="定义过程式宏的示例">

```rust,ignore
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

定义过程式宏的函数接受一个 `TokenStream` 作为输入，并生成一个 `TokenStream` 作为输出。`TokenStream` 类型由 Rust 附带的 `proc_macro` crate 定义，表示一系列标记。这是宏的核心：宏操作的源代码构成了输入的 `TokenStream`，宏生成的代码是输出的 `TokenStream`。该函数还附加了一个属性，用于指定我们正在创建的过程式宏的类型。我们可以在同一个 crate 中拥有多种类型的过程式宏。

让我们看看不同类型的过程式宏。我们将从自定义 `derive` 宏开始，然后解释其他形式的不同之处。

### 如何编写自定义 `derive` 宏

让我们创建一个名为 `hello_macro` 的 crate，它定义了一个名为 `HelloMacro` 的 trait，并带有一个关联函数 `hello_macro`。与其让用户为每个类型实现 `HelloMacro` trait，我们将提供一个过程式宏，以便用户可以使用 `#[derive(HelloMacro)]` 注解他们的类型，从而获得 `hello_macro` 函数的默认实现。默认实现将打印 `Hello, Macro! My name is TypeName!`，其中 `TypeName` 是定义该 trait 的类型的名称。换句话说，我们将编写一个 crate，使其他程序员能够使用我们的 crate 编写如 Listing 20-37 所示的代码。

<Listing number="20-37" file-name="src/main.rs" caption="使用我们的过程式宏时，用户将能够编写的代码">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

当我们完成后，这段代码将打印 `Hello, Macro! My name is Pancakes!`。第一步是创建一个新的库 crate，如下所示：

```console
$ cargo new hello_macro --lib
```

接下来，我们将定义 `HelloMacro` trait 及其关联函数：

<Listing file-name="src/lib.rs" number="20-38" caption="我们将与 `derive` 宏一起使用的简单 trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

我们有一个 trait 及其函数。此时，我们的 crate 用户可以实现该 trait 以实现所需的功能，如 Listing 20-39 所示。

<Listing number="20-39" file-name="src/main.rs" caption="如果用户手动实现 `HelloMacro` trait 时的样子">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

然而，他们需要为每个想要使用 `hello_macro` 的类型编写实现块；我们希望避免他们做这项工作。

此外，我们还无法为 `hello_macro` 函数提供默认实现，该实现将打印实现该 trait 的类型的名称：Rust 没有反射功能，因此无法在运行时查找类型的名称。我们需要一个宏来在编译时生成代码。

下一步是定义过程式宏。在撰写本文时，过程式宏需要位于它们自己的 crate 中。最终，这一限制可能会被取消。构建 crate 和宏 crate 的约定如下：对于名为 `foo` 的 crate，自定义 `derive` 过程式宏 crate 称为 `foo_derive`。让我们在 `hello_macro` 项目中启动一个名为 `hello_macro_derive` 的新 crate：

```console
$ cargo new hello_macro_derive --lib
```

我们的两个 crate 紧密相关，因此我们在 `hello_macro` crate 的目录中创建过程式宏 crate。如果我们更改 `hello_macro` 中的 trait 定义，我们也必须更改 `hello_macro_derive` 中的过程式宏实现。这两个 crate 需要分别发布，使用这些 crate 的程序员需要将两者都添加为依赖项并将它们引入作用域。我们可以让 `hello_macro` crate 使用 `hello_macro_derive` 作为依赖项并重新导出过程式宏代码。然而，我们构建项目的方式使得程序员即使不需要 `derive` 功能也可以使用 `hello_macro`。

我们需要将 `hello_macro_derive` crate 声明为过程式宏 crate。我们还需要 `syn` 和 `quote` crate 的功能，正如你稍后将看到的，因此我们需要将它们添加为依赖项。将以下内容添加到 `hello_macro_derive` 的 _Cargo.toml_ 文件中：

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

要开始定义过程式宏，请将 Listing 20-40 中的代码放入 `hello_macro_derive` crate 的 _src/lib.rs_ 文件中。请注意，这段代码在我们添加 `impl_hello_macro` 函数的定义之前不会编译。

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="大多数过程式宏 crate 需要处理的 Rust 代码">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

请注意，我们将代码拆分为 `hello_macro_derive` 函数，它负责解析 `TokenStream`，以及 `impl_hello_macro` 函数，它负责转换语法树：这使得编写过程式宏更加方便。外部函数（在本例中为 `hello_macro_derive`）的代码几乎适用于你看到或创建的每个过程式宏 crate。你在内部函数（在本例中为 `impl_hello_macro`）主体中指定的代码将根据你的过程式宏的目的而有所不同。

我们引入了三个新的 crate：`proc_macro`、[`syn`] 和 [`quote`]。`proc_macro` crate 随 Rust 一起提供，因此我们不需要将其添加到 _Cargo.toml_ 中的依赖项中。`proc_macro` crate 是编译器的 API，允许我们从代码中读取和操作 Rust 代码。

`syn` crate 将 Rust 代码从字符串解析为我们可以操作的数据结构。`quote` crate 将 `syn` 数据结构转换回 Rust 代码。这些 crate 使得解析我们可能想要处理的任何类型的 Rust 代码变得更加简单：编写一个完整的 Rust 代码解析器并非易事。

当我们的库用户在类型上指定 `#[derive(HelloMacro)]` 时，将调用 `hello_macro_derive` 函数。这是可能的，因为我们在这里用 `proc_macro_derive` 注解了 `hello_macro_derive` 函数，并指定了名称 `HelloMacro`，它与我们的 trait 名称匹配；这是大多数过程式宏遵循的约定。

`hello_macro_derive` 函数首先将 `input` 从 `TokenStream` 转换为我们可以解释和执行操作的数据结构。这就是 `syn` 发挥作用的地方。`syn` 中的 `parse` 函数接受一个 `TokenStream` 并返回一个表示解析后的 Rust 代码的 `DeriveInput` 结构体。Listing 20-40 展示了我们从解析 `struct Pancakes;` 字符串得到的 `DeriveInput` 结构体的相关部分。

<Listing number="20-40" caption="当我们解析包含宏属性的代码时得到的 `DeriveInput` 实例">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

这个结构体的字段显示我们解析的 Rust 代码是一个名为 `Pancakes` 的单元结构体。这个结构体上有更多字段用于描述各种 Rust 代码；有关更多信息，请参阅 [`syn` 文档中的 `DeriveInput`][syn-docs]。

很快我们将定义 `impl_hello_macro` 函数，这是我们构建要包含的新 Rust 代码的地方。但在我们这样做之前，请注意我们的 `derive` 宏的输出也是一个 `TokenStream`。返回的 `TokenStream` 被添加到我们的 crate 用户编写的代码中，因此当他们编译他们的 crate 时，他们将获得我们在修改后的 `TokenStream` 中提供的额外功能。

你可能已经注意到，我们在这里调用 `unwrap` 来使 `hello_macro_derive` 函数在 `syn::parse` 函数调用失败时 panic。我们的过程式宏必须在错误时 panic，因为 `proc_macro_derive` 函数必须返回 `TokenStream` 而不是 `Result` 以符合过程式宏 API。我们通过使用 `unwrap` 简化了这个示例；在生产代码中，你应该通过使用 `panic!` 或 `expect` 提供更具体的错误信息。

现在我们已经有了将注解的 Rust 代码从 `TokenStream` 转换为 `DeriveInput` 实例的代码，让我们生成在注解类型上实现 `HelloMacro` trait 的代码，如 Listing 20-42 所示。

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="使用解析后的 Rust 代码实现 `HelloMacro` trait">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

我们使用 `ast.ident` 获取包含注解类型名称（标识符）的 `Ident` 结构体实例。Listing 20-33 中的结构体显示，当我们在 Listing 20-31 中的代码上运行 `impl_hello_macro` 函数时，我们得到的 `ident` 将具有值为 `"Pancakes"` 的 `ident` 字段。因此，Listing 20-34 中的 `name` 变量将包含一个 `Ident` 结构体实例，当打印时，它将是字符串 `"Pancakes"`，即 Listing 20-37 中结构体的名称。

`quote!` 宏允许我们定义我们想要返回的 Rust 代码。编译器期望与 `quote!` 宏执行的直接结果不同的东西，因此我们需要将其转换为 `TokenStream`。我们通过调用 `into` 方法来实现这一点，该方法消耗这个中间表示并返回所需的 `TokenStream` 类型的值。

`quote!` 宏还提供了一些非常酷的模板机制：我们可以输入 `#name`，`quote!` 将用变量 `name` 中的值替换它。你甚至可以做一些类似于常规宏工作的重复操作。查看 [the `quote` crate 的文档][quote-docs] 以获取详细介绍。

我们希望我们的过程式宏为用户注解的类型生成 `HelloMacro` trait 的实现，我们可以通过使用 `#name` 来获取。trait 实现有一个函数 `hello_macro`，其主体包含我们想要提供的功能：打印 `Hello, Macro! My name is`，然后是注解类型的名称。

这里使用的 `stringify!` 宏是 Rust 内置的。它接受一个 Rust 表达式，例如 `1 + 2`，并在编译时将表达式转换为字符串字面量，例如 `"1 + 2"`。这与 `format!` 或 `println!` 宏不同，后者会评估表达式然后将结果转换为 `String`。`#name` 输入可能是一个要逐字打印的表达式，因此我们使用 `stringify!`。使用 `stringify!` 还可以通过在编译时将 `#name` 转换为字符串字面量来节省分配。

此时，`cargo build` 应该在 `hello_macro` 和 `hello_macro_derive` 中成功完成。让我们将这些 crate 连接到 Listing 20-31 中的代码，看看过程式宏的实际效果！在你的 _projects_ 目录中使用 `cargo new pancakes` 创建一个新的二进制项目。我们需要在 `pancakes` crate 的 _Cargo.toml_ 中添加 `hello_macro` 和 `hello_macro_derive` 作为依赖项。如果你将 `hello_macro` 和 `hello_macro_derive` 的版本发布到 [crates.io](https://crates.io/)，它们将是常规依赖项；如果没有，你可以将它们指定为 `path` 依赖项，如下所示：

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:7:9}}
```

将 Listing 20-37 中的代码放入 _src/main.rs_，并运行 `cargo run`：它应该打印 `Hello, Macro! My name is Pancakes!` 过程式宏的 `HelloMacro` trait 实现被包含在内，而 `pancakes` crate 不需要实现它；`#[derive(HelloMacro)]` 添加了 trait 实现。

接下来，让我们探讨其他类型的过程式宏与自定义 `derive` 宏的不同之处。

### 类属性宏

类属性宏类似于自定义 `derive` 宏，但它们不是为 `derive` 属性生成代码，而是允许你创建新的属性。它们也更灵活：`derive` 仅适用于结构体和枚举；属性可以应用于其他项，例如函数。以下是一个使用类属性宏的示例。假设你有一个名为 `route` 的属性，在使用 Web 应用程序框架时用于注解函数：

```rust,ignore
#[route(GET, "/")]
fn index() {
```

这个 `#[route]` 属性将由框架定义为过程式宏。宏定义函数的签名将如下所示：

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

在这里，我们有两个 `TokenStream` 类型的参数。第一个用于属性的内容：`GET, "/"` 部分。第二个是属性附加的项的主体：在本例中，`fn index() {}` 和函数的其余主体。

除此之外，类属性宏的工作方式与自定义 `derive` 宏相同：你创建一个具有 `proc-macro` crate 类型的 crate，并实现一个生成你想要的代码的函数！

### 类函数宏

类函数宏定义了看起来像函数调用的宏。与 `macro_rules!` 宏类似，它们比函数更灵活；例如，它们可以接受未知数量的参数。然而，`macro_rules!` 宏只能使用我们在 [“使用 `macro_rules!` 进行通用元编程的声明式宏”][decl]<!-- ignore --> 中讨论的类似匹配的语法定义。类函数宏接受一个 `TokenStream` 参数，并且它们的定义使用 Rust 代码操作该 `TokenStream`，就像其他两种过程式宏一样。类函数宏的一个示例是 `sql!` 宏，它可能像这样调用：

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

这个宏将解析其中的 SQL 语句并检查其语法是否正确，这比 `macro_rules!` 宏所能做的处理要复杂得多。`sql!` 宏将像这样定义：

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

这个定义类似于自定义 `derive` 宏的签名：我们接收括号内的标记并返回我们想要生成的代码。

## 总结

哇！现在你工具箱中有一些 Rust 特性，你可能不会经常使用，但你会知道它们在非常特殊的情况下是可用的。我们介绍了几个复杂的主题，以便当你在错误消息建议或其他人的代码中遇到它们时，你能够识别这些概念和语法。使用本章作为参考来指导你找到解决方案。

接下来，我们将把我们在整本书中讨论的所有内容付诸实践，并再做一个项目！

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[`syn`]: https://crates.io/crates/syn
[`quote`]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming