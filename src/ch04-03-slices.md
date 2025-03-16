## 切片类型

_切片_ 允许你引用一个集合中的连续元素序列，而不是整个集合。切片是一种引用，因此它没有所有权。

这里有一个小的编程问题：编写一个函数，该函数接受一个由空格分隔的单词字符串，并返回该字符串中的第一个单词。如果函数在字符串中没有找到空格，那么整个字符串必须是一个单词，因此应该返回整个字符串。

让我们通过编写这个函数的签名来了解切片将解决的问题，而不使用切片：

```rust,ignore
fn first_word(s: &String) -> ?
```

`first_word` 函数有一个 `&String` 作为参数。我们不需要所有权，所以这没问题。（在惯用的 Rust 中，函数不会获取其参数的所有权，除非它们需要，随着我们继续深入，这一点会变得更加清晰！）但是我们应该返回什么？我们实际上没有办法谈论字符串的一部分。然而，我们可以返回单词末尾的索引，由空格表示。让我们尝试一下，如 Listing 4-7 所示。

<Listing number="4-7" file-name="src/main.rs" caption="返回 `String` 参数中字节索引值的 `first_word` 函数">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

因为我们需要逐个元素地遍历 `String` 并检查某个值是否为空格，所以我们将使用 `as_bytes` 方法将 `String` 转换为字节数组。

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

接下来，我们使用 `iter` 方法在字节数组上创建一个迭代器：

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

我们将在 [第 13 章][ch13]<!-- ignore --> 中更详细地讨论迭代器。现在，只需知道 `iter` 是一个返回集合中每个元素的方法，而 `enumerate` 包装了 `iter` 的结果，并将每个元素作为元组的一部分返回。`enumerate` 返回的元组的第一个元素是索引，第二个元素是对元素的引用。这比我们自己计算索引更方便。

因为 `enumerate` 方法返回一个元组，我们可以使用模式来解构这个元组。我们将在 [第 6 章][ch6]<!-- ignore --> 中更详细地讨论模式。在 `for` 循环中，我们指定一个模式，其中 `i` 是元组中的索引，`&item` 是元组中的单个字节。因为我们从 `.iter().enumerate()` 中获取了对元素的引用，所以我们在模式中使用 `&`。

在 `for` 循环内部，我们使用字节字面量语法搜索表示空格的字节。如果我们找到一个空格，我们返回该位置。否则，我们使用 `s.len()` 返回字符串的长度。

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

我们现在有一种方法可以找到字符串中第一个单词的结尾索引，但有一个问题。我们返回的是一个单独的 `usize`，但它只在 `&String` 的上下文中有意义。换句话说，因为它是一个与 `String` 分离的值，所以不能保证它在将来仍然有效。考虑 Listing 4-8 中的程序，它使用了 Listing 4-7 中的 `first_word` 函数。

<Listing number="4-8" file-name="src/main.rs" caption="存储调用 `first_word` 函数的结果，然后更改 `String` 内容">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

这个程序在没有任何错误的情况下编译，如果我们在调用 `s.clear()` 之后使用 `word`，它也会这样做。因为 `word` 与 `s` 的状态完全无关，`word` 仍然包含值 `5`。我们可以使用该值 `5` 与变量 `s` 尝试提取第一个单词，但这将是一个错误，因为自从我们在 `word` 中保存 `5` 以来，`s` 的内容已经发生了变化。

担心 `word` 中的索引与 `s` 中的数据不同步是繁琐且容易出错的！如果我们编写一个 `second_word` 函数，管理这些索引会更加脆弱。它的签名必须如下所示：

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

现在我们正在跟踪一个起始索引和一个结束索引，我们有更多的值是从特定状态的数据计算出来的，但与那个状态完全没有关联。我们有三个不相关的变量需要保持同步。

幸运的是，Rust 有一个解决这个问题的方法：字符串切片。

### 字符串切片

_字符串切片_ 是对 `String` 的一部分的引用，它看起来像这样：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

`hello` 不是对整个 `String` 的引用，而是对 `String` 的一部分的引用，由额外的 `[0..5]` 指定。我们通过在括号内指定 `[starting_index..ending_index]` 来创建切片，其中 _`starting_index`_ 是切片的第一个位置，_`ending_index`_ 是切片最后一个位置的下一个位置。在内部，切片数据结构存储了切片的起始位置和长度，长度对应于 _`ending_index`_ 减去 _`starting_index`_。因此，在 `let world = &s[6..11];` 的情况下，`world` 将是一个切片，它包含一个指向 `s` 索引 6 处的字节的指针，长度值为 `5`。

图 4-7 以图表形式展示了这一点。

<img alt="三个表：一个表表示 s 的栈数据，它指向堆数据表中索引 0 处的字节，堆数据表包含字符串数据 &quot;hello world&quot;。第三个表表示切片 world 的栈数据，它的长度值为 5，并指向堆数据表的字节 6。"
src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-7: 字符串切片引用 `String` 的一部分</span>

使用 Rust 的 `..` 范围语法，如果你想从索引 0 开始，可以省略两个点之前的值。换句话说，以下两种写法是等价的：

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

同样地，如果你的切片包括 `String` 的最后一个字节，你可以省略结尾的数字。这意味着以下两种写法是等价的：

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

你也可以省略两个值来获取整个字符串的切片。因此，以下两种写法是等价的：

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> 注意：字符串切片的范围索引必须位于有效的 UTF-8 字符边界。如果你尝试在多字节字符的中间创建字符串切片，你的程序将会出错。为了介绍字符串切片，我们在本节中假设只使用 ASCII；关于 UTF-8 处理的更详细讨论在 [第 8 章][strings]<!-- ignore --> 的“使用字符串存储 UTF-8 编码文本”部分。

有了这些信息，让我们重写 `first_word` 以返回一个切片。表示“字符串切片”的类型写作 `&str`：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

我们以与 Listing 4-7 相同的方式获取单词的结尾索引，即查找第一个空格的出现。当我们找到一个空格时，我们返回一个字符串切片，使用字符串的起始位置和空格的索引作为起始和结束索引。

现在当我们调用 `first_word` 时，我们得到一个与底层数据绑定的单一值。该值由切片的起始点的引用和切片中的元素数量组成。

返回切片也适用于 `second_word` 函数：

```rust,ignore
fn second_word(s: &String) -> &str {
```

我们现在有一个更不容易出错的直接 API，因为编译器将确保对 `String` 的引用保持有效。还记得 Listing 4-8 中的程序中的错误吗？当我们获取第一个单词的结尾索引，然后清空字符串，使我们的索引无效时？那段代码在逻辑上是错误的，但没有立即显示任何错误。如果我们继续尝试使用清空字符串的第一个单词索引，问题会在以后出现。切片使这个错误不可能发生，并让我们更早地知道代码有问题。使用切片版本的 `first_word` 会抛出一个编译时错误：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

以下是编译器错误：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

回想一下借用规则，如果我们对某物有一个不可变引用，我们就不能同时获取一个可变引用。因为 `clear` 需要截断 `String`，它需要获取一个可变引用。`clear` 调用后的 `println!` 使用了 `word` 中的引用，因此不可变引用在此时必须仍然有效。Rust 不允许 `clear` 中的可变引用和 `word` 中的不可变引用同时存在，因此编译失败。Rust 不仅使我们的 API 更易于使用，而且还在编译时消除了整个类别的错误！

<!-- 旧标题。不要删除或链接可能会中断。 -->

<a id="string-literals-are-slices"></a>

#### 字符串字面量是切片

回想一下，我们讨论过字符串字面量存储在二进制文件中。现在我们知道切片，我们可以正确理解字符串字面量：

```rust
let s = "Hello, world!";
```

这里的 `s` 类型是 `&str`：它是一个指向二进制文件中特定位置的切片。这也是为什么字符串字面量是不可变的；`&str` 是一个不可变引用。

#### 字符串切片作为参数

知道你可以获取字面量和 `String` 值的切片，这让我们对 `first_word` 的签名有了进一步的改进：

```rust,ignore
fn first_word(s: &String) -> &str {
```

更有经验的 Rustacean 会编写 Listing 4-9 中所示的签名，因为它允许我们在 `&String` 值和 `&str` 值上使用相同的函数。

<Listing number="4-9" caption="通过使用字符串切片作为 `s` 参数的类型来改进 `first_word` 函数">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

如果我们有一个字符串切片，我们可以直接传递它。如果我们有一个 `String`，我们可以传递 `String` 的切片或对 `String` 的引用。这种灵活性利用了 _解引用强制转换_，我们将在 [第 15 章][deref-coercions]<!--ignore--> 的“函数和方法的隐式解引用强制转换”部分讨论这一特性。

定义一个函数来接受字符串切片而不是对 `String` 的引用，使我们的 API 更通用且更有用，而不会失去任何功能：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### 其他切片

字符串切片，正如你可能想象的那样，是特定于字符串的。但也有一个更通用的切片类型。考虑这个数组：

```rust
let a = [1, 2, 3, 4, 5];
```

就像我们可能想引用字符串的一部分一样，我们可能想引用数组的一部分。我们可以这样做：

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

这个切片的类型是 `&[i32]`。它的工作方式与字符串切片相同，通过存储对第一个元素的引用和长度。你会为各种其他集合使用这种切片。我们将在第 8 章讨论向量时详细讨论这些集合。

## 总结

所有权、借用和切片的概念确保 Rust 程序在编译时的内存安全。Rust 语言让你像其他系统编程语言一样控制内存使用，但拥有数据的拥有者在超出作用域时自动清理数据，这意味着你不必编写和调试额外的代码来获得这种控制。

所有权影响 Rust 的许多其他部分的工作方式，因此我们将在本书的其余部分进一步讨论这些概念。让我们继续第 5 章，看看如何将数据分组到一个 `struct` 中。

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods