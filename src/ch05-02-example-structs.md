## 使用结构体的示例程序

为了理解我们何时可能想要使用结构体，让我们编写一个计算矩形面积的程序。我们将从使用单个变量开始，然后逐步重构程序，直到使用结构体为止。

让我们使用 Cargo 创建一个新的二进制项目，名为 _rectangles_，该项目将接收以像素为单位的矩形的宽度和高度，并计算矩形的面积。Listing 5-8 展示了一个简短的程序，展示了如何在我们的项目的 _src/main.rs_ 中实现这一点。

<Listing number="5-8" file-name="src/main.rs" caption="通过单独的宽度和高度变量计算矩形的面积">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

现在，使用 `cargo run` 运行这个程序：

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

这段代码通过调用 `area` 函数并传入每个维度来成功计算出矩形的面积，但我们可以做更多的工作来使这段代码更加清晰和易读。

这段代码的问题在 `area` 函数的签名中显而易见：

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

`area` 函数应该计算一个矩形的面积，但我们编写的函数有两个参数，并且在程序的任何地方都不清楚这两个参数是相关的。将宽度和高度组合在一起会更易读和更易管理。我们已经在第 3 章的 [“元组类型”][the-tuple-type]<!-- ignore --> 部分讨论过一种方法：使用元组。

### 使用元组进行重构

Listing 5-9 展示了我们程序的另一个版本，该版本使用了元组。

<Listing number="5-9" file-name="src/main.rs" caption="使用元组指定矩形的宽度和高度">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

在某种程度上，这个程序更好。元组让我们增加了一些结构，现在我们只传递一个参数。但在另一种方式上，这个版本不太清晰：元组没有为它们的元素命名，所以我们必须通过索引访问元组的部分，这使得我们的计算不那么明显。

混淆宽度和高度对于面积计算来说并不重要，但如果我们想在屏幕上绘制矩形，那就很重要了！我们必须记住 `width` 是元组索引 `0`，而 `height` 是元组索引 `1`。如果其他人要使用我们的代码，这将更难理解和记住。因为我们没有在代码中传达数据的含义，所以现在更容易引入错误。

### 使用结构体进行重构：增加更多意义

我们使用结构体通过为数据添加标签来增加意义。我们可以将我们使用的元组转换为一个结构体，该结构体具有整体的名称以及部分的名称，如 Listing 5-10 所示。

<Listing number="5-10" file-name="src/main.rs" caption="定义一个 `Rectangle` 结构体">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

在这里，我们定义了一个结构体并将其命名为 `Rectangle`。在大括号内，我们将字段定义为 `width` 和 `height`，它们的类型都是 `u32`。然后，在 `main` 中，我们创建了一个特定的 `Rectangle` 实例，其宽度为 `30`，高度为 `50`。

我们的 `area` 函数现在定义了一个参数，我们将其命名为 `rectangle`，其类型是对 `Rectangle` 结构体实例的不可变借用。正如第 4 章提到的，我们希望借用结构体而不是获取其所有权。这样，`main` 保留其所有权并可以继续使用 `rect1`，这就是我们在函数签名和调用函数时使用 `&` 的原因。

`area` 函数访问 `Rectangle` 实例的 `width` 和 `height` 字段（注意，访问借用结构体实例的字段不会移动字段值，这就是为什么你经常看到结构体的借用）。我们的 `area` 函数签名现在准确地表达了我们的意思：使用 `Rectangle` 的 `width` 和 `height` 字段计算其面积。这传达了宽度和高度是相互关联的，并且它为值提供了描述性名称，而不是使用元组索引值 `0` 和 `1`。这是一个清晰度的胜利。

### 使用派生特性添加有用的功能

在调试程序时，能够打印 `Rectangle` 实例并查看其所有字段的值会很有用。Listing 5-11 尝试使用我们在前几章中使用过的 [`println!` 宏][println]<!-- ignore -->。然而，这不会起作用。

<Listing number="5-11" file-name="src/main.rs" caption="尝试打印 `Rectangle` 实例">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

当我们编译这段代码时，会得到以下核心错误信息：

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

`println!` 宏可以进行多种格式化，默认情况下，大括号告诉 `println!` 使用称为 `Display` 的格式化：输出直接供最终用户使用。我们目前见过的原始类型默认实现了 `Display`，因为只有一种方式可以向用户显示 `1` 或其他原始类型。但对于结构体，`println!` 应该如何格式化输出就不那么清楚了，因为有更多的显示可能性：你想要逗号吗？你想打印大括号吗？应该显示所有字段吗？由于这种歧义，Rust 不会尝试猜测我们想要什么，结构体也没有提供 `Display` 的实现供 `println!` 和 `{}` 占位符使用。

如果我们继续阅读错误信息，我们会发现这个有用的提示：

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

让我们试试看！`println!` 宏调用现在看起来像 `println!("rect1 is {rect1:?}");`。将 `:?` 放在大括号内告诉 `println!` 我们想要使用称为 `Debug` 的输出格式。`Debug` 特性使我们能够以对开发者有用的方式打印结构体，这样我们就可以在调试代码时看到它的值。

用这个更改编译代码。糟糕！我们仍然得到一个错误：

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

但再次，编译器给了我们一个有用的提示：

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust _确实_ 包含了打印调试信息的功能，但我们必须显式选择才能使该功能对我们的结构体可用。为此，我们在结构体定义之前添加外部属性 `#[derive(Debug)]`，如 Listing 5-12 所示。

<Listing number="5-12" file-name="src/main.rs" caption="添加属性以派生 `Debug` 特性并使用调试格式化打印 `Rectangle` 实例">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

现在当我们运行程序时，不会得到任何错误，并且会看到以下输出：

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

很好！这不是最漂亮的输出，但它显示了该实例的所有字段的值，这肯定会在调试时有所帮助。当我们有更大的结构体时，输出稍微易读一些会很有用；在这些情况下，我们可以在 `println!` 字符串中使用 `{:#?}` 而不是 `{:?}`。在这个例子中，使用 `{:#?}` 样式将输出以下内容：

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

另一种使用 `Debug` 格式打印值的方法是使用 [`dbg!` 宏][dbg]<!-- ignore -->，它获取表达式的所有权（与 `println!` 不同，后者获取引用），打印 `dbg!` 宏调用在代码中发生的位置的文件和行号以及该表达式的结果值，并返回该值的所有权。

> 注意：调用 `dbg!` 宏会打印到标准错误控制台流（`stderr`），而不是 `println!`，后者打印到标准输出控制台流（`stdout`）。我们将在第 12 章的 [“将错误消息写入标准错误而不是标准输出” 部分][err]<!-- ignore --> 中更多地讨论 `stderr` 和 `stdout`。

这里有一个例子，我们对分配给 `width` 字段的值以及 `rect1` 中整个结构体的值感兴趣：

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

我们可以将 `dbg!` 放在表达式 `30 * scale` 周围，因为 `dbg!` 返回表达式值的所有权，`width` 字段将获得与没有 `dbg!` 调用时相同的值。我们不希望 `dbg!` 获取 `rect1` 的所有权，所以我们在下一个调用中使用 `rect1` 的引用。以下是这个例子的输出：

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

我们可以看到第一部分的输出来自 _src/main.rs_ 的第 10 行，我们在那里调试表达式 `30 * scale`，其结果值为 `60`（为整数实现的 `Debug` 格式化只打印它们的值）。第 14 行的 `dbg!` 调用输出了 `&rect1` 的值，即 `Rectangle` 结构体。这个输出使用了 `Rectangle` 类型的漂亮 `Debug` 格式化。`dbg!` 宏在你试图弄清楚代码在做什么时非常有用！

除了 `Debug` 特性外，Rust 还为我们提供了许多特性，可以与 `derive` 属性一起使用，这些特性可以为我们的自定义类型添加有用的行为。这些特性及其行为列在 [附录 C][app-c]<!-- ignore --> 中。我们将在第 10 章中介绍如何实现这些特性以及如何创建自己的特性。还有许多其他属性；更多信息请参见 [Rust 参考中的 “属性” 部分][attributes]。

我们的 `area` 函数非常具体：它只计算矩形的面积。将这个行为与我们的 `Rectangle` 结构体更紧密地绑定在一起会很有帮助，因为它不适用于任何其他类型。让我们看看如何通过将 `area` 函数转换为定义在 `Rectangle` 类型上的 `area` _方法_ 来继续重构这段代码。

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html