## 函数

函数在 Rust 代码中非常常见。你已经见过这门语言中最重要的函数之一：`main` 函数，它是许多程序的入口点。你也见过 `fn` 关键字，它允许你声明新的函数。

Rust 代码使用 **蛇形命名法（snake case）** 作为函数和变量名称的常规风格，其中所有字母都是小写，并使用下划线分隔单词。以下是一个包含函数定义示例的程序：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

我们在 Rust 中通过输入 `fn` 后跟函数名和一组括号来定义函数。大括号告诉编译器函数体的开始和结束位置。

我们可以通过输入函数名后跟一组括号来调用任何已定义的函数。因为 `another_function` 在程序中定义了，所以可以从 `main` 函数中调用它。注意，我们在源代码中 `main` 函数 **之后** 定义了 `another_function`；我们也可以在之前定义它。Rust 不关心你在何处定义函数，只要它们在调用者可见的范围内定义即可。

让我们启动一个名为 _functions_ 的新二进制项目来进一步探索函数。将 `another_function` 示例放入 _src/main.rs_ 并运行它。你应该会看到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

这些行按照它们在 `main` 函数中出现的顺序执行。首先打印 “Hello, world!” 消息，然后调用 `another_function` 并打印其消息。

### 参数

我们可以定义带有 **参数** 的函数，这些参数是函数签名的一部分的特殊变量。当函数有参数时，你可以为这些参数提供具体的值。从技术上讲，具体的值被称为 **实参**，但在日常对话中，人们倾向于将 **参数** 和 **实参** 这两个词互换使用，无论是函数定义中的变量还是调用函数时传入的具体值。

在这个版本的 `another_function` 中，我们添加了一个参数：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

尝试运行这个程序；你应该会得到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

`another_function` 的声明有一个名为 `x` 的参数。`x` 的类型被指定为 `i32`。当我们向 `another_function` 传递 `5` 时，`println!` 宏将 `5` 放在格式字符串中包含 `x` 的大括号对的位置。

在函数签名中，你 **必须** 声明每个参数的类型。这是 Rust 设计中的一个深思熟虑的决定：在函数定义中要求类型注释意味着编译器几乎不需要你在代码的其他地方使用它们来推断你想要的类型。如果编译器知道函数期望的类型，它还能提供更有帮助的错误消息。

当定义多个参数时，用逗号分隔参数声明，如下所示：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

这个例子创建了一个名为 `print_labeled_measurement` 的函数，它有两个参数。第一个参数名为 `value`，类型为 `i32`。第二个参数名为 `unit_label`，类型为 `char`。然后函数打印包含 `value` 和 `unit_label` 的文本。

让我们尝试运行这段代码。将当前在 _functions_ 项目的 _src/main.rs_ 文件中的程序替换为前面的示例，并使用 `cargo run` 运行它：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

因为我们用 `5` 作为 `value` 的值，`'h'` 作为 `unit_label` 的值调用了函数，所以程序输出包含这些值。

### 语句和表达式

函数体由一系列语句组成，可以选择以表达式结尾。到目前为止，我们介绍的函数还没有包含结束表达式，但你已经看到过作为语句一部分的表达式。因为 Rust 是一种基于表达式的语言，这是一个需要理解的重要区别。其他语言没有相同的区别，所以让我们看看什么是语句和表达式，以及它们的区别如何影响函数体。

- **语句** 是执行某些操作但不返回值的指令。
- **表达式** 会计算出一个结果值。让我们看一些例子。

实际上我们已经使用过语句和表达式。使用 `let` 关键字创建变量并为其赋值是一个语句。在 Listing 3-1 中，`let y = 6;` 是一个语句。

<Listing number="3-1" file-name="src/main.rs" caption="包含一个语句的 `main` 函数声明">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

函数定义也是语句；前面的整个例子本身就是一个语句。（正如我们将在下面看到的，**调用** 函数不是一个语句。）

语句不返回值。因此，你不能将一个 `let` 语句赋值给另一个变量，如下面的代码尝试做的；你会得到一个错误：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

当你运行这个程序时，你会得到的错误如下：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

`let y = 6` 语句不返回值，所以 `x` 没有可以绑定的值。这与 C 和 Ruby 等其他语言中的情况不同，在这些语言中，赋值会返回赋值的值。在这些语言中，你可以写 `x = y = 6`，并使 `x` 和 `y` 都具有值 `6`；但在 Rust 中不是这样。

表达式会计算出一个值，并构成你在 Rust 中编写的大部分代码。考虑一个数学运算，比如 `5 + 6`，这是一个计算为值 `11` 的表达式。表达式可以是语句的一部分：在 Listing 3-1 中，语句 `let y = 6;` 中的 `6` 是一个计算为值 `6` 的表达式。调用函数是一个表达式。调用宏是一个表达式。用大括号创建的新作用域块是一个表达式，例如：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

这个表达式：

```rust,ignore
{
    let x = 3;
    x + 1
}
```

是一个块，在这种情况下，它计算为 `4`。这个值作为 `let` 语句的一部分绑定到 `y`。注意，`x + 1` 行的末尾没有分号，这与目前为止你看到的大多数行不同。表达式不包括结束分号。如果你在表达式的末尾添加一个分号，你就把它变成了一个语句，然后它不会返回值。在你接下来探索函数返回值和表达式时，请记住这一点。

### 带有返回值的函数

函数可以向调用它们的代码返回值。我们不为返回值命名，但必须在箭头 (`->`) 后声明它们的类型。在 Rust 中，函数的返回值与函数体块中最后一个表达式的值同义。你可以通过使用 `return` 关键字并指定一个值来提前从函数返回，但大多数函数隐式返回最后一个表达式。以下是一个返回值的函数示例：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

`five` 函数中没有函数调用、宏，甚至没有 `let` 语句——只有数字 `5` 本身。这在 Rust 中是一个完全有效的函数。注意，函数的返回类型也被指定为 `-> i32`。尝试运行这段代码；输出应该如下所示：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

`five` 函数中的 `5` 是函数的返回值，这就是为什么返回类型是 `i32`。让我们更详细地研究一下。有两个重要的部分：首先，行 `let x = five();` 显示我们正在使用函数的返回值来初始化一个变量。因为函数 `five` 返回 `5`，所以这行代码等同于以下代码：

```rust
let x = 5;
```

其次，`five` 函数没有参数并定义了返回值的类型，但函数体是一个孤零零的 `5`，没有分号，因为它是一个我们想要返回其值的表达式。

让我们看另一个例子：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

运行这段代码将打印 `The value of x is: 6`。但如果我们在包含 `x + 1` 的行末尾加上一个分号，将其从表达式改为语句，我们会得到一个错误：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

编译这段代码会产生一个错误，如下所示：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

主要的错误信息 `mismatched types` 揭示了这段代码的核心问题。函数 `plus_one` 的定义表明它将返回一个 `i32`，但语句不会计算出一个值，这由 `()`（单元类型）表示。因此，没有返回任何值，这与函数定义相矛盾，导致错误。在这个输出中，Rust 提供了一个可能帮助纠正这个问题的消息：它建议删除分号，这将修复错误。