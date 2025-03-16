## 变量与可变性

正如在[“使用变量存储值”][storing-values-with-variables]<!-- ignore -->部分提到的，默认情况下，变量是不可变的。这是 Rust 提供的众多提示之一，旨在帮助你以利用 Rust 提供的安全性和简单并发性的方式编写代码。然而，你仍然可以选择使变量可变。让我们探讨一下 Rust 如何以及为何鼓励你优先选择不可变性，以及为什么有时你可能希望选择可变性。

当一个变量是不可变的时，一旦一个值被绑定到一个名称上，你就不能改变这个值。为了说明这一点，请在 _projects_ 目录下使用 `cargo new variables` 生成一个名为 _variables_ 的新项目。

然后，在你的新 _variables_ 目录中，打开 _src/main.rs_ 并将其代码替换为以下代码，这段代码目前还无法编译：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

保存并使用 `cargo run` 运行程序。你应该会收到一个关于不可变性错误的错误消息，如下所示：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

这个例子展示了编译器如何帮助你发现程序中的错误。编译器错误可能会让人沮丧，但它们实际上只是意味着你的程序还没有安全地执行你想要的操作；它们并不意味着你不是一个好的程序员！即使是经验丰富的 Rust 开发者仍然会遇到编译器错误。

你收到的错误消息是 `` 不能对不可变变量 `x` 进行二次赋值 ``，因为你尝试为不可变的 `x` 变量赋第二个值。

当我们尝试更改一个被指定为不可变的值时，获得编译时错误非常重要，因为这种情况可能会导致错误。如果我们代码的一部分假设一个值永远不会改变，而另一部分代码改变了这个值，那么代码的第一部分可能不会按预期执行。这种错误的根源在事后可能很难追踪，尤其是当第二段代码只是偶尔改变这个值时。Rust 编译器保证当你声明一个值不会改变时，它就真的不会改变，因此你不必自己跟踪它。这样，你的代码更容易推理。

但可变性有时非常有用，可以使代码更方便编写。尽管变量默认是不可变的，但你可以通过在变量名前添加 `mut` 来使它们可变，就像你在[第 2 章][storing-values-with-variables]<!-- ignore -->中所做的那样。添加 `mut` 还可以向未来的代码读者传达意图，表明代码的其他部分将改变这个变量的值。

例如，让我们将 _src/main.rs_ 改为以下内容：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

现在当我们运行程序时，会得到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

当使用 `mut` 时，我们允许将绑定到 `x` 的值从 `5` 更改为 `6`。最终，决定是否使用可变性取决于你，并取决于你认为在特定情况下哪种方式最清晰。

### 常量

与不可变变量类似，_常量_ 是绑定到名称且不允许更改的值，但常量与变量之间有一些区别。

首先，你不能对常量使用 `mut`。常量不仅仅是默认不可变的——它们总是不可变的。你使用 `const` 关键字而不是 `let` 关键字来声明常量，并且必须注解值的类型。我们将在下一节[“数据类型”][data-types]<!-- ignore -->中介绍类型和类型注解，所以现在不必担心细节。只需知道你必须始终注解类型。

常量可以在任何作用域中声明，包括全局作用域，这使得它们对于代码的多个部分需要了解的值非常有用。

最后一个区别是，常量只能设置为常量表达式，而不能设置为只能在运行时计算的结果。

以下是一个常量声明的示例：

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

常量的名称是 `THREE_HOURS_IN_SECONDS`，其值设置为 60（一分钟的秒数）乘以 60（一小时的分钟数）乘以 3（我们想要计算的小时数）的结果。Rust 的常量命名约定是使用全大写字母，单词之间用下划线分隔。编译器能够在编译时评估一组有限的操作，这使我们能够选择以一种更容易理解和验证的方式写出这个值，而不是将这个常量设置为值 10,800。有关在声明常量时可以使用的操作的更多信息，请参阅 [Rust 参考中的常量评估部分][const-eval]。

常量在程序的整个运行期间都是有效的，在其声明的作用域内。这个特性使得常量对于应用程序域中的值非常有用，这些值可能是程序的多个部分需要了解的，例如游戏中任何玩家允许获得的最大点数，或光速。

将程序中使用的硬编码值命名为常量有助于向未来的代码维护者传达该值的含义。此外，如果将来需要更新硬编码值，只需在代码中的一个地方进行更改即可。

### 变量遮蔽

正如你在[第 2 章][comparing-the-guess-to-the-secret-number]<!-- ignore -->的猜谜游戏教程中看到的，你可以声明一个与先前变量同名的新变量。Rust 开发者说第一个变量被第二个变量遮蔽了，这意味着当你使用变量名时，编译器将看到第二个变量。实际上，第二个变量遮蔽了第一个变量，直到它自己被遮蔽或作用域结束为止。我们可以通过使用相同的变量名并重复使用 `let` 关键字来遮蔽一个变量，如下所示：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

这个程序首先将 `x` 绑定到值 `5`。然后它通过重复 `let x =` 创建一个新变量 `x`，取原始值并加 `1`，因此 `x` 的值变为 `6`。然后，在用大括号创建的内部作用域中，第三个 `let` 语句也遮蔽了 `x` 并创建了一个新变量，将前一个值乘以 `2`，使 `x` 的值为 `12`。当该作用域结束时，内部遮蔽结束，`x` 恢复为 `6`。当我们运行这个程序时，它将输出以下内容：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

遮蔽与将变量标记为 `mut` 不同，因为如果我们不小心尝试在不使用 `let` 关键字的情况下重新赋值给这个变量，我们会得到一个编译时错误。通过使用 `let`，我们可以对一个值进行一些转换，但在这些转换完成后，变量仍然是不可变的。

`mut` 和遮蔽之间的另一个区别是，因为当我们再次使用 `let` 关键字时，我们实际上是创建了一个新变量，所以我们可以改变值的类型，但重用相同的名称。例如，假设我们的程序要求用户通过输入空格字符来显示他们希望在某些文本之间有多少空格，然后我们希望将该输入存储为一个数字：

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

第一个 `spaces` 变量是字符串类型，第二个 `spaces` 变量是数字类型。遮蔽使我们不必想出不同的名称，例如 `spaces_str` 和 `spaces_num`；相反，我们可以重用更简单的 `spaces` 名称。然而，如果我们尝试为此使用 `mut`，如下所示，我们将得到一个编译时错误：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

错误信息说我们不允许改变变量的类型：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

现在我们已经探讨了变量如何工作，让我们看看它们可以具有的更多数据类型。

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: ../reference/const_eval.html