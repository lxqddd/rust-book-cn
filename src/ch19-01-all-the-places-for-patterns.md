## 所有可以使用模式的地方

模式在 Rust 中出现在许多地方，你可能已经在不知不觉中大量使用了它们！本节讨论了模式有效的所有地方。

### `match` 分支

正如在第 6 章中讨论的那样，我们在 `match` 表达式的分支中使用模式。形式上，`match` 表达式被定义为关键字 `match`、一个要匹配的值，以及一个或多个由模式和表达式组成的分支，如果值匹配该分支的模式，则运行该表达式，如下所示：

<pre><code>match <em>VALUE</em> {
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
}</code></pre>

例如，这里是来自 Listing 6-5 的 `match` 表达式，它匹配变量 `x` 中的 `Option<i32>` 值：

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

这个 `match` 表达式中的模式是每个箭头左侧的 `None` 和 `Some(i)`。

`match` 表达式的一个要求是它们必须是**穷尽的**，即 `match` 表达式中的所有可能性都必须被覆盖。确保覆盖所有可能性的一种方法是为最后一个分支提供一个通配符模式：例如，一个匹配任何值的变量名永远不会失败，因此可以覆盖所有剩余的情况。

特定的模式 `_` 将匹配任何内容，但它永远不会绑定到变量，因此它经常用于最后一个 `match` 分支。例如，当你想忽略任何未指定的值时，`_` 模式非常有用。我们将在本章后面的[“忽略模式中的值”][ignoring-values-in-a-pattern]<!-- ignore -->部分更详细地介绍 `_` 模式。

### 条件 `if let` 表达式

在第 6 章中，我们讨论了如何使用 `if let` 表达式，主要是作为一种更短的写法来替代只匹配一个情况的 `match` 表达式。可选地，`if let` 可以有一个对应的 `else`，包含在 `if let` 中的模式不匹配时要运行的代码。

Listing 19-1 展示了混合使用 `if let`、`else if` 和 `else if let` 表达式也是可能的。这样做比 `match` 表达式提供了更多的灵活性，因为 `match` 表达式只能表达一个值与模式的比较。此外，Rust 不要求一系列 `if let`、`else if`、`else if let` 分支中的条件彼此相关。

Listing 19-1 中的代码根据对几个条件的检查来确定背景颜色。在这个例子中，我们创建了具有硬编码值的变量，这些值可能是真实程序从用户输入中接收到的。

<Listing number="19-1" file-name="src/main.rs" caption="混合使用 `if let`、`else if`、`else if let` 和 `else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs}}
```

</Listing>

如果用户指定了最喜欢的颜色，则该颜色将用作背景。如果没有指定最喜欢的颜色并且今天是星期二，则背景颜色为绿色。否则，如果用户将他们的年龄指定为字符串并且我们可以成功将其解析为数字，则颜色将根据数字的值是紫色或橙色。如果这些条件都不适用，则背景颜色为蓝色。

这种条件结构使我们能够支持复杂的需求。使用我们这里的硬编码值，这个例子将打印 `Using purple as the background color`。

你可以看到 `if let` 也可以引入新的变量，这些变量会以与 `match` 分支相同的方式遮蔽现有变量：`if let Ok(age) = age` 这一行引入了一个新的 `age` 变量，该变量包含 `Ok` 变体中的值，遮蔽了现有的 `age` 变量。这意味着我们需要将 `if age > 30` 条件放在该块中：我们不能将这两个条件组合成 `if let Ok(age) = age && age > 30`。我们想要与 30 比较的新 `age` 在花括号开始的新作用域之前是无效的。

使用 `if let` 表达式的缺点是编译器不会检查穷尽性，而 `match` 表达式会检查。如果我们省略了最后一个 `else` 块，因此错过了一些情况的处理，编译器不会提醒我们可能的逻辑错误。

### `while let` 条件循环

与 `if let` 结构类似，`while let` 条件循环允许 `while` 循环在模式继续匹配时运行。在 Listing 19-2 中，我们展示了一个 `while let` 循环，它等待线程之间发送的消息，但在这种情况下检查的是 `Result` 而不是 `Option`。

<Listing number="19-2" caption="使用 `while let` 循环在 `rx.recv()` 返回 `Ok` 时打印值">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

这个例子打印 `1`、`2`，然后是 `3`。`recv` 方法从通道的接收端取出第一条消息并返回一个 `Ok(value)`。当我们第一次在第 16 章中看到 `recv` 时，我们直接解包了错误，或者使用 `for` 循环将其作为迭代器进行交互。如 Listing 19-2 所示，我们也可以使用 `while let`，因为只要发送者存在，`recv` 方法每次收到消息时都会返回 `Ok`，然后在发送端断开连接时产生一个 `Err`。

### `for` 循环

在 `for` 循环中，紧跟在关键字 `for` 后面的值是一个模式。例如，在 `for x in y` 中，`x` 是模式。Listing 19-3 演示了如何在 `for` 循环中使用模式来解构或分解元组作为 `for` 循环的一部分。

<Listing number="19-3" caption="在 `for` 循环中使用模式来解构元组">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs:here}}
```

</Listing>

Listing 19-3 中的代码将打印以下内容：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-03/output.txt}}
```

我们使用 `enumerate` 方法调整迭代器，使其生成一个值及其索引，放入一个元组中。生成的第一个值是元组 `(0, 'a')`。当这个值与模式 `(index, value)` 匹配时，`index` 将是 `0`，`value` 将是 `'a'`，打印输出的第一行。

### `let` 语句

在本章之前，我们只明确讨论了在 `match` 和 `if let` 中使用模式，但实际上，我们在其他地方也使用了模式，包括在 `let` 语句中。例如，考虑这个简单的变量赋值：

```rust
let x = 5;
```

每次你使用这样的 `let` 语句时，你都在使用模式，尽管你可能没有意识到！更正式地说，`let` 语句看起来像这样：

<pre>
<code>let <em>PATTERN</em> = <em>EXPRESSION</em>;</code>
</pre>

在像 `let x = 5;` 这样的语句中，`_PATTERN_` 槽中的变量名只是模式的一种特别简单的形式。Rust 将表达式与模式进行比较，并分配它找到的任何名称。因此，在 `let x = 5;` 的例子中，`x` 是一个模式，意思是“将匹配这里的内容绑定到变量 `x`”。因为名称 `x` 是整个模式，所以这个模式实际上意味着“将所有内容绑定到变量 `x`，无论值是什么。”

为了更清楚地看到 `let` 的模式匹配方面，考虑 Listing 19-4，它使用 `let` 模式来解构一个元组。

<Listing number="19-4" caption="使用模式解构元组并一次性创建三个变量">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

在这里，我们将一个元组与模式匹配。Rust 将值 `(1, 2, 3)` 与模式 `(x, y, z)` 进行比较，并看到值匹配模式，因为两者的元素数量相同，所以 Rust 将 `1` 绑定到 `x`，`2` 绑定到 `y`，`3` 绑定到 `z`。你可以将这个元组模式视为嵌套了三个单独的变量模式。

如果模式中的元素数量与元组中的元素数量不匹配，整体类型将不匹配，我们将得到一个编译器错误。例如，Listing 19-5 展示了一个尝试将具有三个元素的元组解构为两个变量的例子，这将无法工作。

<Listing number="19-5" caption="错误地构造了一个变量数量与元组元素数量不匹配的模式">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

尝试编译此代码会导致以下类型错误：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

要修复错误，我们可以使用 `_` 或 `..` 忽略元组中的一个或多个值，正如你将在[“忽略模式中的值”][ignoring-values-in-a-pattern]<!-- ignore -->部分看到的那样。如果问题是模式中的变量太多，解决方案是通过删除变量使类型匹配，使变量数量等于元组中的元素数量。

### 函数参数

函数参数也可以是模式。Listing 19-6 中的代码声明了一个名为 `foo` 的函数，它接受一个名为 `x` 的 `i32` 类型参数，现在应该看起来很熟悉。

<Listing number="19-6" caption="函数签名在参数中使用模式">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

`x` 部分是一个模式！就像我们在 `let` 中所做的那样，我们可以在函数的参数中将元组与模式匹配。Listing 19-7 在将元组传递给函数时拆分元组中的值。

<Listing number="19-7" file-name="src/main.rs" caption="一个参数解构元组的函数">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

这段代码打印 `Current location: (3, 5)`。值 `&(3, 5)` 与模式 `&(x, y)` 匹配，因此 `x` 是值 `3`，`y` 是值 `5`。

我们也可以在闭包参数列表中以与函数参数列表相同的方式使用模式，因为闭包与函数类似，正如第 13 章中讨论的那样。

到目前为止，你已经看到了几种使用模式的方式，但模式在我们可以使用它们的每个地方的工作方式并不相同。在某些地方，模式必须是不可反驳的；在其他情况下，它们可以是可反驳的。我们接下来将讨论这两个概念。

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern