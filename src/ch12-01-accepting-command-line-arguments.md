## 接受命令行参数

让我们像往常一样使用 `cargo new` 创建一个新项目。我们将项目命名为 `minigrep`，以便与您系统上可能已经存在的 `grep` 工具区分开来。

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

第一个任务是让 `minigrep` 接受两个命令行参数：文件路径和要搜索的字符串。也就是说，我们希望能够在运行程序时使用 `cargo run`，两个连字符表示接下来的参数是传递给我们的程序而不是 `cargo` 的，一个要搜索的字符串，以及要搜索的文件路径，如下所示：

```console
$ cargo run -- searchstring example-filename.txt
```

目前，由 `cargo new` 生成的程序无法处理我们传递给它的参数。[crates.io](https://crates.io/) 上的一些现有库可以帮助编写接受命令行参数的程序，但由于您刚刚学习这个概念，让我们自己实现这个功能。

### 读取参数值

为了使 `minigrep` 能够读取我们传递给它的命令行参数的值，我们需要使用 Rust 标准库中提供的 `std::env::args` 函数。这个函数返回传递给 `minigrep` 的命令行参数的迭代器。我们将在[第 13 章][ch13]<!-- ignore -->中详细介绍迭代器。现在，您只需要了解关于迭代器的两个细节：迭代器生成一系列值，并且我们可以在迭代器上调用 `collect` 方法将其转换为包含迭代器生成的所有元素的集合，例如向量。

Listing 12-1 中的代码允许您的 `minigrep` 程序读取传递给它的任何命令行参数，然后将这些值收集到一个向量中。

<Listing number="12-1" file-name="src/main.rs" caption="将命令行参数收集到向量中并打印它们">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

首先，我们使用 `use` 语句将 `std::env` 模块引入作用域，以便可以使用它的 `args` 函数。请注意，`std::env::args` 函数嵌套在两个模块层级中。正如我们在[第 7 章][ch7-idiomatic-use]<!-- ignore -->中讨论的那样，在所需函数嵌套在多个模块中的情况下，我们选择将父模块引入作用域，而不是直接引入函数。通过这样做，我们可以轻松使用 `std::env` 中的其他函数。这也比添加 `use std::env::args` 然后仅使用 `args` 调用函数更清晰，因为 `args` 可能很容易被误认为是当前模块中定义的函数。

> ### `args` 函数和无效的 Unicode
>
> 请注意，如果任何参数包含无效的 Unicode，`std::env::args` 将会 panic。如果您的程序需要接受包含无效 Unicode 的参数，请改用 `std::env::args_os`。该函数返回一个生成 `OsString` 值而不是 `String` 值的迭代器。我们在这里选择使用 `std::env::args` 是为了简单起见，因为 `OsString` 值因平台而异，并且比 `String` 值更复杂。

在 `main` 的第一行，我们调用 `env::args`，并立即使用 `collect` 将迭代器转换为包含迭代器生成的所有值的向量。我们可以使用 `collect` 函数创建多种类型的集合，因此我们显式注释 `args` 的类型以指定我们希望得到一个字符串向量。尽管在 Rust 中很少需要注释类型，但 `collect` 是您经常需要注释的函数之一，因为 Rust 无法推断您想要的集合类型。

最后，我们使用调试宏打印向量。让我们先尝试运行代码，不带参数，然后带两个参数：

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

请注意，向量中的第一个值是 `"target/debug/minigrep"`，这是我们的二进制文件的名称。这与 C 语言中的参数列表行为相匹配，允许程序使用它们在执行时被调用的名称。通常，访问程序名称很方便，以防您想在消息中打印它或根据用于调用程序的命令行别名更改程序的行为。但出于本章的目的，我们将忽略它，只保存我们需要的两个参数。

### 将参数值保存在变量中

程序目前能够访问指定为命令行参数的值。现在我们需要将这两个参数的值保存在变量中，以便我们可以在程序的其余部分使用这些值。我们在 Listing 12-2 中这样做。

<Listing number="12-2" file-name="src/main.rs" caption="创建变量以保存查询参数和文件路径参数">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

正如我们在打印向量时看到的那样，程序的名称占据了向量中的第一个值 `args[0]`，因此我们从索引 1 开始获取参数。`minigrep` 接受的第一个参数是我们要搜索的字符串，因此我们将第一个参数的引用放入变量 `query` 中。第二个参数将是文件路径，因此我们将第二个参数的引用放入变量 `file_path` 中。

我们暂时打印这些变量的值，以证明代码按我们的预期工作。让我们再次运行这个程序，参数为 `test` 和 `sample.txt`：

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

太好了，程序正在工作！我们需要的参数值被保存到了正确的变量中。稍后我们将添加一些错误处理来处理某些潜在的错误情况，例如用户没有提供参数；现在，我们将忽略这种情况，转而添加文件读取功能。

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths