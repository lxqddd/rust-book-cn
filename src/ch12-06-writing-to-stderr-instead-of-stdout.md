## 将错误信息写入标准错误而不是标准输出

目前，我们使用 `println!` 宏将所有输出写入终端。在大多数终端中，有两种类型的输出：_标准输出_（`stdout`）用于一般信息，_标准错误_（`stderr`）用于错误信息。这种区分使用户可以选择将程序的成功输出重定向到文件，但仍然将错误信息打印到屏幕上。

`println!` 宏只能打印到标准输出，因此我们需要使用其他方法来打印到标准错误。

### 检查错误信息写入的位置

首先，让我们观察一下 `minigrep` 打印的内容当前是如何写入标准输出的，包括我们希望写入标准错误的任何错误信息。我们将通过将标准输出流重定向到一个文件，同时故意引发一个错误来实现这一点。我们不会重定向标准错误流，因此发送到标准错误的任何内容将继续显示在屏幕上。

命令行程序应该将错误信息发送到标准错误流，这样即使我们将标准输出流重定向到文件，我们仍然可以在屏幕上看到错误信息。我们的程序目前表现不佳：我们将看到它将错误信息输出保存到文件中，而不是显示在屏幕上！

为了演示这种行为，我们将使用 `>` 和文件路径 `output.txt` 来运行程序，我们希望将标准输出流重定向到该文件。我们不会传递任何参数，这应该会导致一个错误：

```console
$ cargo run > output.txt
```

`>` 语法告诉 shell 将标准输出的内容写入 `output.txt` 而不是屏幕。我们没有看到预期的错误信息打印到屏幕上，所以这意味着它一定已经写入了文件。以下是 `output.txt` 的内容：

```text
Problem parsing arguments: not enough arguments
```

是的，我们的错误信息被打印到了标准输出。将这样的错误信息打印到标准错误会更有用，这样只有成功运行的数据才会写入文件。我们将对此进行更改。

### 将错误信息打印到标准错误

我们将使用 Listing 12-24 中的代码来更改错误信息的打印方式。由于我们在本章早些时候进行了重构，所有打印错误信息的代码都在一个函数 `main` 中。标准库提供了 `eprintln!` 宏，它可以打印到标准错误流，因此让我们将调用 `println!` 打印错误的两处地方改为使用 `eprintln!`。

<Listing number="12-24" file-name="src/main.rs" caption="使用 `eprintln!` 将错误信息写入标准错误而不是标准输出">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

现在让我们再次以相同的方式运行程序，不传递任何参数，并使用 `>` 重定向标准输出：

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

现在我们在屏幕上看到了错误信息，而 `output.txt` 中没有任何内容，这是我们对命令行程序的期望行为。

让我们再次运行程序，这次传递不会导致错误的参数，但仍然将标准输出重定向到文件，如下所示：

```console
$ cargo run -- to poem.txt > output.txt
```

我们不会在终端上看到任何输出，而 `output.txt` 将包含我们的结果：

<span class="filename">文件名: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

这表明我们现在适当地使用标准输出来处理成功输出，使用标准错误来处理错误输出。

## 总结

本章回顾了你到目前为止学到的一些主要概念，并介绍了如何在 Rust 中执行常见的 I/O 操作。通过使用命令行参数、文件、环境变量以及用于打印错误的 `eprintln!` 宏，你现在已经准备好编写命令行应用程序了。结合前几章的概念，你的代码将组织良好，有效地将数据存储在适当的数据结构中，优雅地处理错误，并经过良好的测试。

接下来，我们将探索一些受函数式语言影响的 Rust 特性：闭包和迭代器。