## 可反驳性：模式是否会匹配失败

模式有两种形式：可反驳的（refutable）和不可反驳的（irrefutable）。能够匹配任何可能传递的值的模式是**不可反驳的**。例如，`let x = 5;` 语句中的 `x` 就是不可反驳的，因为 `x` 可以匹配任何值，因此不会匹配失败。而对于某些可能的值会匹配失败的模式是**可反驳的**。例如，`if let Some(x) = a_value` 表达式中的 `Some(x)` 就是可反驳的，因为如果 `a_value` 变量中的值是 `None` 而不是 `Some`，那么 `Some(x)` 模式将不会匹配。

函数参数、`let` 语句和 `for` 循环只能接受不可反驳的模式，因为当值不匹配时，程序无法做任何有意义的事情。`if let` 和 `while let` 表达式以及 `let...else` 语句可以接受可反驳和不可反驳的模式，但编译器会对不可反驳的模式发出警告，因为它们的目的是处理可能的失败：条件语句的功能在于它能够根据成功或失败执行不同的操作。

一般来说，你不需要担心可反驳和不可反驳模式之间的区别；然而，你需要熟悉可反驳性的概念，以便在错误信息中看到它时能够做出反应。在这些情况下，你需要根据代码的预期行为更改模式或使用模式的结构。

让我们来看一个例子，看看当我们尝试在 Rust 要求不可反驳模式的地方使用可反驳模式时会发生什么，反之亦然。Listing 19-8 展示了一个 `let` 语句，但对于模式，我们指定了 `Some(x)`，这是一个可反驳的模式。正如你所料，这段代码将无法编译。

<Listing number="19-8" caption="尝试在 `let` 中使用可反驳模式">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

如果 `some_option_value` 是 `None` 值，它将无法匹配 `Some(x)` 模式，这意味着该模式是可反驳的。然而，`let` 语句只能接受不可反驳的模式，因为代码无法对 `None` 值做任何有效的操作。在编译时，Rust 会抱怨我们试图在需要不可反驳模式的地方使用可反驳模式：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

因为我们没有（也无法！）用 `Some(x)` 模式覆盖所有可能的值，Rust 正确地生成了一个编译错误。

如果我们在需要不可反驳模式的地方使用了可反驳模式，我们可以通过更改使用模式的代码来修复它：我们可以使用 `if let` 而不是 `let`。然后，如果模式不匹配，代码将跳过花括号中的代码，从而使其能够继续有效执行。Listing 19-9 展示了如何修复 Listing 19-8 中的代码。

<Listing number="19-9" caption="使用 `let...else` 和带有可反驳模式的代码块代替 `let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

我们为代码提供了一个出路！这段代码现在完全有效。然而，如果我们给 `if let` 提供一个不可反驳的模式（一个总是会匹配的模式），例如 `x`，如 Listing 19-10 所示，编译器会发出警告。

<Listing number="19-10" caption="尝试在 `if let` 中使用不可反驳模式">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

Rust 抱怨说，使用 `if let` 与不可反驳模式没有意义：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

因此，`match` 分支必须使用可反驳的模式，除了最后一个分支，它应该使用不可反驳的模式来匹配所有剩余的值。Rust 允许我们在只有一个分支的 `match` 中使用不可反驳的模式，但这种语法并不特别有用，可以用更简单的 `let` 语句代替。

现在你已经知道了在哪里使用模式以及可反驳和不可反驳模式之间的区别，让我们来介绍所有可以用来创建模式的语法。