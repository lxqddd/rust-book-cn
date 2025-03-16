## 数据类型

Rust 中的每个值都有一个特定的**数据类型**，它告诉 Rust 正在指定哪种数据，以便它知道如何处理这些数据。我们将查看两种数据类型的子集：标量类型和复合类型。

请记住，Rust 是一种**静态类型**语言，这意味着它必须在编译时知道所有变量的类型。编译器通常可以根据值以及我们如何使用它来推断我们想要使用的类型。在某些情况下，可能会有多种类型，例如在第 2 章的[“将猜测与秘密数字进行比较”][comparing-the-guess-to-the-secret-number]<!-- ignore -->部分中，我们使用 `parse` 将 `String` 转换为数字类型时，我们必须添加类型注解，如下所示：

```rust
let guess: u32 = "42".parse().expect("不是数字！");
```

如果我们不添加前面代码中所示的 `: u32` 类型注解，Rust 将显示以下错误，这意味着编译器需要更多信息来了解我们想要使用的类型：

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

你会看到其他数据类型的不同类型注解。

### 标量类型

**标量**类型表示单个值。Rust 有四种主要的标量类型：整数、浮点数、布尔值和字符。你可能从其他编程语言中已经认识这些类型。让我们深入了解它们在 Rust 中的工作原理。

#### 整数类型

**整数**是没有小数部分的数字。我们在第 2 章中使用了一个整数类型，即 `u32` 类型。这个类型声明表明它关联的值应该是一个无符号整数（有符号整数类型以 `i` 开头，而不是 `u`），占用 32 位空间。表 3-1 展示了 Rust 中的内置整数类型。我们可以使用这些变体中的任何一种来声明整数值的类型。

<span class="caption">表 3-1: Rust 中的整数类型</span>

| 长度   | 有符号  | 无符号   |
| ------ | ------- | -------- |
| 8-bit  | `i8`    | `u8`     |
| 16-bit | `i16`   | `u16`    |
| 32-bit | `i32`   | `u32`    |
| 64-bit | `i64`   | `u64`    |
| 128-bit| `i128`  | `u128`   |
| arch   | `isize` | `usize`  |

每个变体都可以是有符号或无符号的，并且有一个明确的大小。**有符号**和**无符号**指的是数字是否可以表示负数——换句话说，数字是否需要带有符号（有符号）或者它是否永远为正数，因此可以不带符号表示（无符号）。这就像在纸上写数字：当符号重要时，数字会带有加号或减号；然而，当可以安全地假设数字为正数时，它就不带符号。有符号数字使用[二进制补码][twos-complement]<!-- ignore -->表示法存储。

每个有符号变体可以存储从 −(2<sup>n − 1</sup>) 到 2<sup>n − 1</sup> − 1 的数字，其中 _n_ 是该变体使用的位数。因此，`i8` 可以存储从 −(2<sup>7</sup>) 到 2<sup>7</sup> − 1 的数字，即 −128 到 127。无符号变体可以存储从 0 到 2<sup>n</sup> − 1 的数字，因此 `u8` 可以存储从 0 到 2<sup>8</sup> − 1 的数字，即 0 到 255。

此外，`isize` 和 `usize` 类型取决于程序运行的计算机架构，这在表中表示为“arch”：如果你在 64 位架构上，则为 64 位；如果你在 32 位架构上，则为 32 位。

你可以使用表 3-2 中显示的任何形式编写整数字面量。请注意，可以表示多种数字类型的数字字面量允许使用类型后缀，例如 `57u8`，以指定类型。数字字面量还可以使用 `_` 作为视觉分隔符，使数字更易于阅读，例如 `1_000`，它的值与指定 `1000` 相同。

<span class="caption">表 3-2: Rust 中的整数字面量</span>

| 数字字面量  | 示例       |
| ----------- | ---------- |
| 十进制      | `98_222`   |
| 十六进制    | `0xff`     |
| 八进制      | `0o77`     |
| 二进制      | `0b1111_0000` |
| 字节 (`u8` 专用) | `b'A'` |

那么如何知道使用哪种整数类型呢？如果你不确定，Rust 的默认值通常是一个不错的起点：整数类型默认为 `i32`。使用 `isize` 或 `usize` 的主要情况是在索引某种集合时。

> ##### 整数溢出
>
> 假设你有一个类型为 `u8` 的变量，它可以存储 0 到 255 之间的值。如果你尝试将变量更改为超出该范围的值，例如 256，将会发生**整数溢出**，这可能导致两种行为之一。当你在调试模式下编译时，Rust 会包含整数溢出检查，如果发生这种行为，程序会在运行时**panic**。Rust 使用术语 **panicking** 来描述程序因错误而退出的情况；我们将在第 9 章的[“使用 `panic!` 处理不可恢复的错误”][unrecoverable-errors-with-panic]<!-- ignore -->部分详细讨论 panic。
>
> 当你在发布模式下使用 `--release` 标志编译时，Rust **不会**包含导致 panic 的整数溢出检查。相反，如果发生溢出，Rust 会执行**二进制补码回绕**。简而言之，大于该类型可以存储的最大值的值会“回绕”到该类型可以存储的最小值。在 `u8` 的情况下，值 256 变为 0，值 257 变为 1，依此类推。程序不会 panic，但变量的值可能不是你期望的值。依赖整数溢出的回绕行为被认为是一个错误。
>
> 为了显式处理溢出的可能性，你可以使用标准库为原始数字类型提供的这些方法系列：
>
> - 使用 `wrapping_*` 方法在所有模式下进行回绕，例如 `wrapping_add`。
> - 如果发生溢出，使用 `checked_*` 方法返回 `None` 值。
> - 使用 `overflowing_*` 方法返回值和布尔值，指示是否发生了溢出。
> - 使用 `saturating_*` 方法在值的最小值或最大值处饱和。

#### 浮点类型

Rust 还有两种用于**浮点数**的原始类型，即带有小数点的数字。Rust 的浮点类型是 `f32` 和 `f64`，分别为 32 位和 64 位大小。默认类型是 `f64`，因为在现代 CPU 上，它的速度与 `f32` 大致相同，但精度更高。所有浮点类型都是有符号的。

以下是一个展示浮点数实际使用的示例：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

浮点数根据 IEEE-754 标准表示。

#### 数值运算

Rust 支持所有数字类型的基本数学运算：加法、减法、乘法、除法和取余。整数除法向零舍入到最接近的整数。以下代码展示了如何在 `let` 语句中使用每种数值运算：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

这些语句中的每个表达式都使用一个数学运算符并计算为单个值，然后绑定到一个变量。[附录 B][appendix_b]<!-- ignore --> 包含了 Rust 提供的所有运算符的列表。

#### 布尔类型

与大多数其他编程语言一样，Rust 中的布尔类型有两个可能的值：`true` 和 `false`。布尔值的大小为一个字节。Rust 中的布尔类型使用 `bool` 指定。例如：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

使用布尔值的主要方式是通过条件语句，例如 `if` 表达式。我们将在[“控制流”][control-flow]<!-- ignore -->部分介绍 `if` 表达式在 Rust 中的工作原理。

#### 字符类型

Rust 的 `char` 类型是该语言最基本的字母类型。以下是一些声明 `char` 值的示例：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

请注意，我们使用单引号指定 `char` 字面量，而字符串字面量使用双引号。Rust 的 `char` 类型大小为四个字节，表示一个 Unicode 标量值，这意味着它可以表示的内容远不止 ASCII。重音字母；中文、日文和韩文字符；表情符号；以及零宽度空格都是 Rust 中的有效 `char` 值。Unicode 标量值的范围从 `U+0000` 到 `U+D7FF` 和 `U+E000` 到 `U+10FFFF` 之间。然而，“字符”并不是 Unicode 中的一个真正概念，因此你对“字符”的人类直觉可能与 Rust 中的 `char` 不完全一致。我们将在第 8 章的[“使用字符串存储 UTF-8 编码的文本”][strings]<!-- ignore -->部分详细讨论这个话题。

### 复合类型

**复合类型**可以将多个值组合成一个类型。Rust 有两种原始复合类型：元组和数组。

#### 元组类型

**元组**是一种将多个不同类型的值组合成一个复合类型的通用方式。元组具有固定长度：一旦声明，它们的大小就不能增长或缩小。

我们通过在括号内写入逗号分隔的值列表来创建元组。元组中的每个位置都有一个类型，元组中不同值的类型不必相同。我们在这个示例中添加了可选的类型注解：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

变量 `tup` 绑定到整个元组，因为元组被视为单个复合元素。要从元组中获取单个值，我们可以使用模式匹配来解构元组值，如下所示：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

这个程序首先创建一个元组并将其绑定到变量 `tup`。然后它使用 `let` 模式将 `tup` 分解为三个单独的变量 `x`、`y` 和 `z`。这被称为**解构**，因为它将单个元组分解为三个部分。最后，程序打印出 `y` 的值，即 `6.4`。

我们还可以通过使用句点（`.`）后跟我们想要访问的值的索引来直接访问元组元素。例如：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

这个程序创建了元组 `x`，然后使用各自的索引访问元组的每个元素。与大多数编程语言一样，元组中的第一个索引是 0。

没有任何值的元组有一个特殊的名称，称为**单元**。这个值及其对应的类型都写为 `()`，表示一个空值或空返回类型。如果表达式不返回任何其他值，则隐式返回单元值。

#### 数组类型

另一种拥有多个值集合的方式是使用**数组**。与元组不同，数组中的每个元素必须具有相同的类型。与某些其他语言中的数组不同，Rust 中的数组具有固定长度。

我们将数组中的值写为方括号内的逗号分隔列表：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

当你希望数据分配在栈上（与我们迄今为止看到的其他类型相同）而不是堆上时，数组非常有用（我们将在[第 4 章][stack-and-heap]<!-- ignore -->中详细讨论栈和堆），或者当你希望确保始终具有固定数量的元素时。数组不如向量类型灵活。**向量**是标准库提供的一种类似的集合类型，它**允许**增长或缩小大小。如果你不确定是使用数组还是向量，很可能你应该使用向量。[第 8 章][vectors]<!-- ignore -->将更详细地讨论向量。

然而，当你知道元素数量不需要改变时，数组更有用。例如，如果你在程序中使用月份的名称，你可能会使用数组而不是向量，因为你知道它总是包含 12 个元素：

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

你使用方括号编写数组的类型，其中包含每个元素的类型、分号以及数组中的元素数量，如下所示：

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

在这里，`i32` 是每个元素的类型。分号后的数字 `5` 表示数组包含五个元素。

你还可以通过指定初始值、分号以及方括号中的数组长度来初始化数组，使每个元素包含相同的值，如下所示：

```rust
let a = [3; 5];
```

名为 `a` 的数组将包含 `5` 个元素，这些元素最初都将设置为值 `3`。这与编写 `let a = [3, 3, 3, 3, 3];` 相同，但更简洁。

##### 访问数组元素

数组是一个已知的、固定大小的单个内存块，可以分配在栈上。你可以使用索引访问数组的元素，如下所示：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

在这个示例中，名为 `first` 的变量将获得值 `1`，因为这是数组中索引 `[0]` 处的值。名为 `second` 的变量将从数组中索引 `[1]` 处获得值 `2`。

##### 无效的数组元素访问

让我们看看如果你尝试访问超出数组末尾的元素会发生什么。假设你运行这段代码，类似于第 2 章中的猜谜游戏，以从用户那里获取数组索引：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

这段代码成功编译。如果你使用 `cargo run` 运行这段代码并输入 `0`、`1`、`2`、`3` 或 `4`，程序将打印出数组中该索引处的相应值。如果你输入超出数组末尾的数字，例如 `10`，你将看到如下输出：

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

程序在使用无效值进行索引操作时导致了**运行时**错误。程序退出了错误消息，并没有执行最后的 `println!` 语句。当你尝试使用索引访问元素时，Rust 会检查你指定的索引是否小于数组长度。如果索引大于或等于长度，Rust 会 panic。这种检查必须在运行时进行，尤其是在这种情况下，因为编译器不可能知道用户在运行代码时会输入什么值。

这是 Rust 内存安全原则的一个例子。在许多低级语言中，这种检查不会进行，当你提供不正确的索引时，可能会访问无效的内存。Rust 通过立即退出而不是允许内存访问并继续来保护你免受这种错误的影响。第 9 章将详细讨论 Rust 的错误处理以及如何编写既不会 panic 也不会允许无效内存访问的可读、安全的代码。

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md