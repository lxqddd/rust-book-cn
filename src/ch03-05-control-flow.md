## 控制流

根据条件是否为 `true` 来决定是否运行某些代码，以及在条件为 `true` 时重复运行某些代码，是大多数编程语言中的基本构建块。Rust 代码中最常见的控制执行流的构造是 `if` 表达式和循环。

### `if` 表达式

`if` 表达式允许你根据条件来分支代码。你提供一个条件，然后声明：“如果这个条件满足，运行这段代码。如果条件不满足，则不运行这段代码。”

在你的 _projects_ 目录下创建一个名为 _branches_ 的新项目来探索 `if` 表达式。在 _src/main.rs_ 文件中，输入以下代码：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

所有的 `if` 表达式都以关键字 `if` 开头，后跟一个条件。在这个例子中，条件检查变量 `number` 的值是否小于 5。如果条件为 `true`，我们在大括号内放置要执行的代码块。与 `if` 表达式中的条件相关联的代码块有时被称为 _分支_，就像我们在第 2 章的 [“Comparing the Guess to the Secret Number”][comparing-the-guess-to-the-secret-number]<!-- ignore --> 部分讨论的 `match` 表达式中的分支一样。

可选地，我们还可以包含一个 `else` 表达式，我们在这里选择了这样做，以便在条件评估为 `false` 时给程序提供一个替代的代码块来执行。如果你不提供 `else` 表达式并且条件为 `false`，程序将跳过 `if` 块并继续执行下一段代码。

尝试运行这段代码；你应该会看到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

让我们尝试将 `number` 的值更改为使条件为 `false` 的值，看看会发生什么：

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

再次运行程序，并查看输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

值得注意的是，这段代码中的条件 _必须_ 是 `bool` 类型。如果条件不是 `bool` 类型，我们会得到一个错误。例如，尝试运行以下代码：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

这次 `if` 条件评估为 `3`，Rust 会抛出一个错误：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

错误表明 Rust 期望一个 `bool` 但得到了一个整数。与 Ruby 和 JavaScript 等语言不同，Rust 不会自动尝试将非布尔类型转换为布尔类型。你必须明确地始终为 `if` 提供一个布尔条件。例如，如果我们希望 `if` 代码块仅在数字不等于 `0` 时运行，我们可以将 `if` 表达式更改为以下内容：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

运行这段代码将打印 `number was something other than zero`。

#### 使用 `else if` 处理多个条件

你可以通过组合 `if` 和 `else` 在 `else if` 表达式中使用多个条件。例如：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

这个程序有四种可能的路径。运行后，你应该会看到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

当这个程序执行时，它会依次检查每个 `if` 表达式，并执行第一个条件评估为 `true` 的代码块。注意，即使 6 可以被 2 整除，我们也不会看到输出 `number is divisible by 2`，也不会看到 `else` 块中的 `number is not divisible by 4, 3, or 2` 文本。这是因为 Rust 只执行第一个 `true` 条件的代码块，一旦找到一个，它甚至不会检查其余的。

使用过多的 `else if` 表达式会使代码变得混乱，所以如果你有多个条件，你可能需要重构你的代码。第 6 章将介绍一个强大的 Rust 分支构造 `match`，用于处理这些情况。

#### 在 `let` 语句中使用 `if`

因为 `if` 是一个表达式，我们可以在 `let` 语句的右侧使用它，将结果赋值给一个变量，如 Listing 3-2 所示。

<Listing number="3-2" file-name="src/main.rs" caption="将 `if` 表达式的结果赋值给一个变量">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

`number` 变量将根据 `if` 表达式的结果绑定到一个值。运行这段代码，看看会发生什么：

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

记住，代码块会评估为其中的最后一个表达式，数字本身也是表达式。在这种情况下，整个 `if` 表达式的值取决于哪个代码块被执行。这意味着 `if` 的每个分支的潜在结果值必须是相同的类型；在 Listing 3-2 中，`if` 分支和 `else` 分支的结果都是 `i32` 整数。如果类型不匹配，如下例所示，我们会得到一个错误：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

当我们尝试编译这段代码时，我们会得到一个错误。`if` 和 `else` 分支的值类型不兼容，Rust 准确地指出了程序中问题的位置：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

`if` 块中的表达式评估为一个整数，而 `else` 块中的表达式评估为一个字符串。这不会起作用，因为变量必须有一个单一的类型，Rust 需要在编译时明确知道 `number` 变量的类型。知道 `number` 的类型可以让编译器在我们使用 `number` 的任何地方验证类型是否有效。如果 `number` 的类型仅在运行时确定，Rust 将无法做到这一点；编译器将更加复杂，并且如果必须为任何变量跟踪多个假设类型，它将减少对代码的保证。

### 使用循环进行重复

多次执行一个代码块通常很有用。为此，Rust 提供了几种 _循环_，它们将循环体内的代码运行到末尾，然后立即从头开始。为了试验循环，让我们创建一个名为 _loops_ 的新项目。

Rust 有三种循环：`loop`、`while` 和 `for`。让我们逐一尝试。

#### 使用 `loop` 重复代码

`loop` 关键字告诉 Rust 一遍又一遍地执行一个代码块，直到你明确告诉它停止。

例如，将你的 _loops_ 目录中的 _src/main.rs_ 文件更改为如下内容：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

当我们运行这个程序时，我们会看到 `again!` 一遍又一遍地连续打印，直到我们手动停止程序。大多数终端支持键盘快捷键 <kbd>ctrl</kbd>-<kbd>c</kbd> 来中断一个陷入无限循环的程序。试试看：

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

符号 `^C` 表示你按下了 <kbd>ctrl</kbd>-<kbd>c</kbd>。你可能在 `^C` 之后看到 `again!` 打印出来，也可能看不到，这取决于代码在接收到中断信号时处于循环的哪个位置。

幸运的是，Rust 还提供了一种通过代码跳出循环的方法。你可以在循环内放置 `break` 关键字，告诉程序何时停止执行循环。回想一下，我们在第 2 章的 [“Quitting After a Correct Guess”][quitting-after-a-correct-guess]<!-- ignore --> 部分中在猜谜游戏中这样做了，当用户猜中正确数字时退出程序。

我们还在猜谜游戏中使用了 `continue`，它在循环中告诉程序跳过本次循环的剩余代码并进入下一次迭代。

#### 从循环中返回值

`loop` 的一个用途是重试一个你知道可能会失败的操作，例如检查线程是否完成了它的工作。你可能还需要将该操作的结果传递出循环，以便在代码的其他部分使用。为此，你可以在用于停止循环的 `break` 表达式后面添加你想要返回的值；该值将从循环中返回，以便你可以使用它，如下所示：

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

在循环之前，我们声明了一个名为 `counter` 的变量并将其初始化为 `0`。然后我们声明了一个名为 `result` 的变量来保存从循环返回的值。在循环的每次迭代中，我们将 `1` 加到 `counter` 变量上，然后检查 `counter` 是否等于 `10`。当它等于 `10` 时，我们使用 `break` 关键字并返回值 `counter * 2`。在循环之后，我们使用分号结束赋值给 `result` 的语句。最后，我们打印 `result` 的值，在这个例子中是 `20`。

你也可以从循环内部 `return`。虽然 `break` 只退出当前循环，但 `return` 总是退出当前函数。

#### 使用循环标签消除多个循环之间的歧义

如果你有嵌套循环，`break` 和 `continue` 适用于此时最内层的循环。你可以选择在循环上指定一个 _循环标签_，然后你可以与 `break` 或 `continue` 一起使用，以指定这些关键字适用于带标签的循环而不是最内层的循环。循环标签必须以单引号开头。以下是一个带有两个嵌套循环的示例：

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

外部循环带有标签 `'counting_up`，它将从 0 计数到 2。没有标签的内部循环将从 10 倒数到 9。第一个没有指定标签的 `break` 将只退出内部循环。`break 'counting_up;` 语句将退出外部循环。这段代码将打印：

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

#### 使用 `while` 进行条件循环

程序通常需要在循环中评估条件。当条件为 `true` 时，循环运行。当条件不再为 `true` 时，程序调用 `break`，停止循环。你可以使用 `loop`、`if`、`else` 和 `break` 的组合来实现这种行为；如果你愿意，现在可以在程序中尝试。然而，这种模式非常常见，以至于 Rust 有一个内置的语言构造，称为 `while` 循环。在 Listing 3-3 中，我们使用 `while` 循环程序三次，每次倒数，然后在循环后打印一条消息并退出。

<Listing number="3-3" file-name="src/main.rs" caption="使用 `while` 循环在条件评估为 `true` 时运行代码">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

这种构造消除了如果你使用 `loop`、`if`、`else` 和 `break` 时所需的许多嵌套，并且更清晰。当条件评估为 `true` 时，代码运行；否则，它退出循环。

#### 使用 `for` 遍历集合

你也可以使用 `while` 构造来遍历集合的元素，例如数组。例如，Listing 3-4 中的循环打印数组 `a` 中的每个元素。

<Listing number="3-4" file-name="src/main.rs" caption="使用 `while` 循环遍历集合中的每个元素">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

在这里，代码从索引 `0` 开始，遍历数组中的元素，直到达到数组的最后一个索引（即当 `index < 5` 不再为 `true` 时）。运行这段代码将打印数组中的每个元素：

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

所有五个数组值都出现在终端中，正如预期的那样。即使 `index` 在某个时候会达到 `5`，循环在尝试从数组中获取第六个值之前停止执行。

然而，这种方法容易出错；如果索引值或测试条件不正确，我们可能会导致程序 panic。例如，如果你将 `a` 数组的定义更改为有四个元素，但忘记将条件更新为 `while index < 4`，代码将会 panic。它也很慢，因为编译器在每次循环迭代时都会添加运行时代码来执行索引是否在数组范围内的条件检查。

作为一种更简洁的替代方案，你可以使用 `for` 循环并为集合中的每个项目执行一些代码。`for` 循环看起来像 Listing 3-5 中的代码。

<Listing number="3-5" file-name="src/main.rs" caption="使用 `for` 循环遍历集合中的每个元素">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

当我们运行这段代码时，我们将看到与 Listing 3-4 相同的输出。更重要的是，我们现在提高了代码的安全性，并消除了可能由于超出数组末尾或未达到足够远而遗漏某些项目而导致的错误。

使用 `for` 循环，如果你更改了数组中值的数量，你不需要记住更改任何其他代码，就像在 Listing 3-4 中使用的方法那样。

`for` 循环的安全性和简洁性使其成为 Rust 中最常用的循环构造。即使在你想要运行某些代码一定次数的情况下，如在 Listing 3-3 中使用 `while` 循环的倒计时示例中，大多数 Rust 开发者也会使用 `for` 循环。这样做的方法是使用标准库提供的 `Range`，它生成从一个数字开始并在另一个数字之前结束的所有数字序列。

以下是使用 `for` 循环和我们尚未讨论的另一个方法 `rev` 来反转范围的倒计时示例：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

这段代码看起来更好，不是吗？

## 总结

你做到了！这是一个相当大的章节：你学习了变量、标量和复合数据类型、函数、注释、`if` 表达式和循环！为了练习本章讨论的概念，尝试构建程序来完成以下任务：

- 在 Fahrenheit 和 Celsius 之间转换温度。
- 生成第 *n* 个斐波那契数。
- 打印圣诞颂歌 “The Twelve Days of Christmas” 的歌词，利用歌曲中的重复部分。

当你准备好继续前进时，我们将讨论 Rust 中一个在其他编程语言中 _不常见_ 的概念：所有权。

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess