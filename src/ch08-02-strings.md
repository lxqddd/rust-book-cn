## 使用字符串存储 UTF-8 编码的文本

我们在第 4 章讨论过字符串，但现在我们将更深入地探讨它们。新 Rustacean 通常会在字符串上遇到困难，原因有三：Rust 倾向于暴露可能的错误、字符串是一种比许多程序员认为的更复杂的数据结构，以及 UTF-8。这些因素结合在一起，可能会让你从其他编程语言转过来时感到困难。

我们在集合的上下文中讨论字符串，因为字符串是作为字节集合实现的，再加上一些方法，以便在将这些字节解释为文本时提供有用的功能。在本节中，我们将讨论每个集合类型都有的 `String` 操作，例如创建、更新和读取。我们还将讨论 `String` 与其他集合的不同之处，即由于人和计算机对 `String` 数据的解释方式不同，索引到 `String` 中会变得复杂。

### 什么是字符串？

我们首先定义术语 _字符串_ 的含义。Rust 在核心语言中只有一种字符串类型，即字符串切片 `str`，通常以借用的形式 `&str` 出现。在第 4 章中，我们讨论了 _字符串切片_，它们是对存储在其他地方的某些 UTF-8 编码字符串数据的引用。例如，字符串字面量存储在程序的二进制文件中，因此它们是字符串切片。

`String` 类型由 Rust 的标准库提供，而不是编码在核心语言中，它是一种可增长、可变、拥有所有权的 UTF-8 编码字符串类型。当 Rustacean 在 Rust 中提到“字符串”时，他们可能指的是 `String` 或字符串切片 `&str` 类型，而不仅仅是其中一种类型。尽管本节主要讨论 `String`，但这两种类型在 Rust 的标准库中都被大量使用，并且 `String` 和字符串切片都是 UTF-8 编码的。

### 创建一个新的字符串

许多与 `Vec<T>` 相同的操作也适用于 `String`，因为 `String` 实际上是作为字节向量的包装器实现的，具有一些额外的保证、限制和功能。一个与 `Vec<T>` 和 `String` 工作方式相同的函数是用于创建实例的 `new` 函数，如 Listing 8-11 所示。

<Listing number="8-11" caption="创建一个新的空 `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-11/src/main.rs:here}}
```

</Listing>

这行代码创建了一个名为 `s` 的新空字符串，我们可以随后将数据加载到其中。通常，我们会有一些初始数据，我们希望用它来启动字符串。为此，我们使用 `to_string` 方法，该方法可用于任何实现了 `Display` trait 的类型，就像字符串字面量一样。Listing 8-12 展示了两个示例。

<Listing number="8-12" caption="使用 `to_string` 方法从字符串字面量创建 `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-12/src/main.rs:here}}
```

</Listing>

这段代码创建了一个包含 `initial contents` 的字符串。

我们还可以使用 `String::from` 函数从字符串字面量创建 `String`。Listing 8-13 中的代码与 Listing 8-12 中使用 `to_string` 的代码等效。

<Listing number="8-13" caption="使用 `String::from` 函数从字符串字面量创建 `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-13/src/main.rs:here}}
```

</Listing>

因为字符串用于许多事情，所以我们可以使用许多不同的通用 API 来处理字符串，这为我们提供了很多选择。其中一些可能看起来冗余，但它们都有其用途！在这种情况下，`String::from` 和 `to_string` 做同样的事情，所以你选择哪一个取决于风格和可读性。

记住，字符串是 UTF-8 编码的，所以我们可以在其中包含任何正确编码的数据，如 Listing 8-14 所示。

<Listing number="8-14" caption="在不同语言的字符串中存储问候语">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:here}}
```

</Listing>

所有这些都是有效的 `String` 值。

### 更新字符串

`String` 可以增长大小，其内容可以更改，就像 `Vec<T>` 的内容一样，如果你向其中推送更多数据。此外，你可以方便地使用 `+` 运算符或 `format!` 宏来连接 `String` 值。

#### 使用 `push_str` 和 `push` 追加到字符串

我们可以使用 `push_str` 方法追加一个字符串切片来增长 `String`，如 Listing 8-15 所示。

<Listing number="8-15" caption="使用 `push_str` 方法将字符串切片追加到 `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-15/src/main.rs:here}}
```

</Listing>

在这两行之后，`s` 将包含 `foobar`。`push_str` 方法接受一个字符串切片，因为我们不一定想要获取参数的所有权。例如，在 Listing 8-16 的代码中，我们希望在将其内容追加到 `s1` 后仍然能够使用 `s2`。

<Listing number="8-16" caption="在将其内容追加到 `String` 后使用字符串切片">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-16/src/main.rs:here}}
```

</Listing>

如果 `push_str` 方法获取了 `s2` 的所有权，我们将无法在最后一行打印其值。然而，这段代码按预期工作！

`push` 方法接受一个字符作为参数并将其添加到 `String` 中。Listing 8-17 使用 `push` 方法将字母 _l_ 添加到 `String` 中。

<Listing number="8-17" caption="使用 `push` 方法向 `String` 值添加一个字符">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-17/src/main.rs:here}}
```

</Listing>

结果，`s` 将包含 `lol`。

#### 使用 `+` 运算符或 `format!` 宏进行连接

通常，你会想要组合两个现有的字符串。一种方法是使用 `+` 运算符，如 Listing 8-18 所示。

<Listing number="8-18" caption="使用 `+` 运算符将两个 `String` 值组合成一个新的 `String` 值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-18/src/main.rs:here}}
```

</Listing>

字符串 `s3` 将包含 `Hello, world!`。`s1` 在加法后不再有效的原因，以及我们使用 `s2` 的引用的原因，与我们在使用 `+` 运算符时调用的方法的签名有关。`+` 运算符使用 `add` 方法，其签名如下所示：

```rust,ignore
fn add(self, s: &str) -> String {
```

在标准库中，你会看到 `add` 使用泛型和关联类型定义。在这里，我们替换了具体类型，这是当我们使用 `String` 值调用此方法时发生的情况。我们将在第 10 章讨论泛型。这个签名为我们提供了理解 `+` 运算符复杂部分所需的线索。

首先，`s2` 有一个 `&`，意味着我们将第二个字符串的 _引用_ 添加到第一个字符串。这是因为 `add` 函数中的 `s` 参数：我们只能将 `&str` 添加到 `String`；我们不能将两个 `String` 值加在一起。但是等等——`&s2` 的类型是 `&String`，而不是 `&str`，正如 `add` 的第二个参数所指定的那样。那么为什么 Listing 8-18 能够编译通过呢？

我们能够在 `add` 调用中使用 `&s2` 的原因是编译器可以将 `&String` 参数强制转换为 `&str`。当我们调用 `add` 方法时，Rust 使用了一个 _解引用强制转换_，在这里将 `&s2` 转换为 `&s2[..]`。我们将在第 15 章更深入地讨论解引用强制转换。因为 `add` 不获取 `s` 参数的所有权，`s2` 在此操作后仍然是一个有效的 `String`。

其次，我们可以在签名中看到 `add` 获取了 `self` 的所有权，因为 `self` 没有 `&`。这意味着 Listing 8-18 中的 `s1` 将被移动到 `add` 调用中，并且在之后不再有效。因此，尽管 `let s3 = s1 + &s2;` 看起来像是会复制两个字符串并创建一个新的字符串，但实际上这个语句获取了 `s1` 的所有权，追加了 `s2` 内容的副本，然后返回结果的所有权。换句话说，它看起来像是做了很多复制，但实际上并没有；实现比复制更高效。

如果我们需要连接多个字符串，`+` 运算符的行为会变得笨拙：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-01-concat-multiple-strings/src/main.rs:here}}
```

此时，`s` 将是 `tic-tac-toe`。由于所有的 `+` 和 `"` 字符，很难看出发生了什么。对于更复杂的字符串组合，我们可以使用 `format!` 宏：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-02-format/src/main.rs:here}}
```

这段代码也将 `s` 设置为 `tic-tac-toe`。`format!` 宏的工作方式类似于 `println!`，但它不是将输出打印到屏幕上，而是返回一个包含内容的 `String`。使用 `format!` 的代码版本更容易阅读，并且 `format!` 宏生成的代码使用引用，因此此调用不会获取其任何参数的所有权。

### 字符串索引

在许多其他编程语言中，通过索引引用字符串中的单个字符是一种有效且常见的操作。然而，如果你尝试在 Rust 中使用索引语法访问 `String` 的部分内容，你会得到一个错误。考虑 Listing 8-19 中的无效代码。

<Listing number="8-19" caption="尝试使用索引语法访问 String">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-19/src/main.rs:here}}
```

</Listing>

这段代码将导致以下错误：

```console
{{#include ../listings/ch08-common-collections/listing-08-19/output.txt}}
```

错误和说明告诉我们：Rust 字符串不支持索引。但为什么不支持呢？要回答这个问题，我们需要讨论 Rust 如何在内存中存储字符串。

#### 内部表示

`String` 是 `Vec<u8>` 的包装器。让我们看看 Listing 8-14 中一些正确编码的 UTF-8 示例字符串。首先，这个：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:spanish}}
```

在这种情况下，`len` 将是 `4`，这意味着存储字符串 `"Hola"` 的向量长度为 4 字节。这些字母在 UTF-8 编码中每个占用一个字节。然而，以下行可能会让你感到惊讶（注意这个字符串以大写西里尔字母 _Ze_ 开头，而不是数字 3）：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:russian}}
```

如果你被问到字符串的长度是多少，你可能会说 12。事实上，Rust 的答案是 24：这是 UTF-8 编码“Здравствуйте”所需的字节数，因为该字符串中的每个 Unicode 标量值占用 2 字节的存储空间。因此，字符串字节的索引并不总是与有效的 Unicode 标量值相关联。为了演示，考虑以下无效的 Rust 代码：

```rust,ignore,does_not_compile
let hello = "Здравствуйте";
let answer = &hello[0];
```

你已经知道 `answer` 不会是 `З`，第一个字母。当用 UTF-8 编码时，`З` 的第一个字节是 `208`，第二个字节是 `151`，所以 `answer` 实际上应该是 `208`，但 `208` 本身并不是一个有效的字符。返回 `208` 可能不是用户想要的，如果他们要求这个字符串的第一个字母；然而，这是 Rust 在字节索引 0 处唯一拥有的数据。用户通常不希望返回字节值，即使字符串只包含拉丁字母：如果 `&"hi"[0]` 是返回字节值的有效代码，它将返回 `104`，而不是 `h`。

因此，答案是为了避免返回意外值并导致可能不会立即发现的错误，Rust 根本不编译此代码，并在开发过程的早期防止误解。

#### 字节、标量值和字素簇！哦，天哪！

关于 UTF-8 的另一点是，实际上有三种相关的方式可以从 Rust 的角度来看待字符串：作为字节、标量值和字素簇（最接近我们称之为 _字母_ 的东西）。

如果我们看一下用天城文书写的印地语单词“नमस्ते”，它存储为一个 `u8` 值的向量，看起来像这样：

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

这是 18 字节，是计算机最终存储这些数据的方式。如果我们把它们看作 Unicode 标量值，也就是 Rust 的 `char` 类型，这些字节看起来像这样：

```text
['न', 'म', 'स', '्', 'त', 'े']
```

这里有六个 `char` 值，但第四个和第六个不是字母：它们是单独没有意义的变音符号。最后，如果我们把它们看作字素簇，我们会得到一个人称之为组成印地语单词的四个字母：

```text
["न", "म", "स्", "ते"]
```

Rust 提供了不同的方式来解释计算机存储的原始字符串数据，以便每个程序可以选择它需要的解释，无论数据是哪种人类语言。

Rust 不允许我们通过索引 `String` 来获取字符的最后一个原因是，索引操作预期总是以恒定时间（O(1)）完成。但是，对于 `String` 来说，无法保证这种性能，因为 Rust 必须从头开始遍历内容到索引，以确定有多少有效字符。

### 字符串切片

索引到字符串中通常不是一个好主意，因为不清楚字符串索引操作的返回类型应该是什么：字节值、字符、字素簇还是字符串切片。因此，如果你确实需要使用索引来创建字符串切片，Rust 要求你更加明确。

你可以使用带有范围的 `[]` 来创建包含特定字节的字符串切片，而不是使用带有单个数字的 `[]`：

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

在这里，`s` 将是一个包含字符串前四个字节的 `&str`。之前我们提到，这些字符中的每一个都是两个字节，这意味着 `s` 将是 `Зд`。

如果我们尝试只切片一个字符的部分字节，比如 `&hello[0..1]`，Rust 会在运行时 panic，就像在向量中访问无效索引一样：

```console
{{#include ../listings/ch08-common-collections/output-only-01-not-char-boundary/output.txt}}
```

在使用范围创建字符串切片时应该小心，因为这样做可能会导致程序崩溃。

### 遍历字符串的方法

操作字符串片段的最佳方式是明确你是想要字符还是字节。对于单个 Unicode 标量值，使用 `chars` 方法。在“Зд”上调用 `chars` 会分离并返回两个 `char` 类型的值，你可以遍历结果以访问每个元素：

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

这段代码将打印以下内容：

```text
З
д
```

或者，`bytes` 方法返回每个原始字节，这可能适合你的领域：

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

这段代码将打印组成此字符串的四个字节：

```text
208
151
208
180
```

但请记住，有效的 Unicode 标量值可能由多个字节组成。

从字符串中获取字素簇，如天城文脚本，是复杂的，因此标准库不提供此功能。如果你需要此功能，可以在 [crates.io](https://crates.io/) 上找到可用的 crate。

### 字符串并不简单

总结一下，字符串是复杂的。不同的编程语言对如何向程序员展示这种复杂性做出了不同的选择。Rust 选择将所有 Rust 程序的默认行为设置为正确处理 `String` 数据，这意味着程序员必须提前更多地考虑如何处理 UTF-8 数据。这种权衡暴露了字符串的复杂性，这在其他编程语言中并不明显，但它可以防止你在开发周期的后期处理涉及非 ASCII 字符的错误。

好消息是，标准库提供了许多基于 `String` 和 `&str` 类型的功能，以帮助正确处理这些复杂情况。请务必查看文档，了解有用的方法，如 `contains` 用于在字符串中搜索，以及 `replace` 用于将字符串的一部分替换为另一个字符串。

让我们转向一些不那么复杂的东西：哈希映射！