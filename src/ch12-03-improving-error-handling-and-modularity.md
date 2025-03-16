## 重构以提高模块化和错误处理

为了改进我们的程序，我们将解决四个与程序结构和潜在错误处理相关的问题。首先，我们的 `main` 函数现在执行两个任务：解析参数和读取文件。随着程序的增长，`main` 函数处理的独立任务数量也会增加。随着函数职责的增加，它变得更加难以理解、测试和修改，而不会破坏其中的某一部分。最好将功能分离，使每个函数只负责一个任务。

这个问题也与第二个问题相关：尽管 `query` 和 `file_path` 是我们程序的配置变量，但像 `contents` 这样的变量用于执行程序的逻辑。`main` 函数越长，我们需要引入作用域的变量就越多；作用域中的变量越多，跟踪每个变量的用途就越困难。最好将配置变量分组到一个结构中，以明确它们的用途。

第三个问题是我们使用 `expect` 在读取文件失败时打印错误消息，但错误消息只是打印 `应该能够读取文件`。读取文件可能会以多种方式失败：例如，文件可能丢失，或者我们可能没有权限打开它。目前，无论情况如何，我们都会为所有情况打印相同的错误消息，这不会给用户提供任何信息！

第四个问题是我们使用 `expect` 来处理错误，如果用户在没有指定足够参数的情况下运行我们的程序，他们会从 Rust 中得到一个 `索引越界` 的错误，这个错误并没有清楚地解释问题。最好将所有错误处理代码放在一个地方，这样未来的维护者只需在一个地方查阅代码，如果错误处理逻辑需要更改的话。将所有错误处理代码放在一个地方还将确保我们打印的消息对最终用户有意义。

让我们通过重构项目来解决这四个问题。

### 二进制项目的职责分离

将多个任务的职责分配给 `main` 函数的组织问题是许多二进制项目的常见问题。因此，Rust 社区开发了一些指导原则，用于在 `main` 函数变得庞大时拆分二进制程序的各个职责。这个过程包括以下步骤：

- 将程序拆分为 `main.rs` 文件和 `lib.rs` 文件，并将程序的逻辑移动到 `lib.rs` 中。
- 只要命令行解析逻辑很小，它可以保留在 `main.rs` 中。
- 当命令行解析逻辑开始变得复杂时，将其从 `main.rs` 中提取出来并移动到 `lib.rs` 中。

在此过程之后，`main` 函数中保留的职责应限于以下内容：

- 使用参数值调用命令行解析逻辑
- 设置任何其他配置
- 调用 `lib.rs` 中的 `run` 函数
- 如果 `run` 返回错误，则处理错误

这种模式是关于职责分离的：`main.rs` 处理程序的运行，`lib.rs` 处理手头任务的所有逻辑。因为你不能直接测试 `main` 函数，这种结构允许你将所有程序逻辑移动到 `lib.rs` 中的函数中进行测试。保留在 `main.rs` 中的代码将足够小，可以通过阅读来验证其正确性。让我们按照这个过程重新编写我们的程序。

#### 提取参数解析器

我们将提取解析参数的功能到一个函数中，`main` 将调用该函数以准备将命令行解析逻辑移动到 `src/lib.rs` 中。Listing 12-5 显示了 `main` 的新开始部分，它调用了一个新的函数 `parse_config`，我们暂时在 `src/main.rs` 中定义它。

<Listing number="12-5" file-name="src/main.rs" caption="从 `main` 中提取 `parse_config` 函数">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

我们仍然将命令行参数收集到一个向量中，但我们不再在 `main` 函数中将索引 1 处的参数值分配给变量 `query`，将索引 2 处的参数值分配给变量 `file_path`，而是将整个向量传递给 `parse_config` 函数。`parse_config` 函数然后包含确定哪个参数进入哪个变量的逻辑，并将值传递回 `main`。我们仍然在 `main` 中创建 `query` 和 `file_path` 变量，但 `main` 不再负责确定命令行参数和变量之间的对应关系。

对于我们的小程序来说，这种重构可能看起来有些过度，但我们正在以小步增量进行重构。在进行此更改后，再次运行程序以验证参数解析是否仍然有效。经常检查你的进度有助于在问题发生时识别问题的原因。

#### 分组配置值

我们可以再迈出一小步来进一步改进 `parse_config` 函数。目前，我们返回一个元组，但随后我们立即将该元组分解为单独的部分。这表明我们可能还没有正确的抽象。

另一个表明有改进空间的指标是 `parse_config` 的 `config` 部分，这意味着我们返回的两个值是相关的，并且都是某个配置值的一部分。我们目前没有在数据结构中传达这种含义，除了将两个值分组到一个元组中；我们将把这两个值放入一个结构体中，并为每个结构体字段赋予一个有意义的名称。这样做将使未来的代码维护者更容易理解不同值之间的关系及其用途。

Listing 12-6 显示了对 `parse_config` 函数的改进。

<Listing number="12-6" file-name="src/main.rs" caption="重构 `parse_config` 以返回 `Config` 结构体的实例">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

我们添加了一个名为 `Config` 的结构体，定义了两个字段 `query` 和 `file_path`。`parse_config` 的签名现在表明它返回一个 `Config` 值。在 `parse_config` 的主体中，我们过去返回引用 `args` 中 `String` 值的字符串切片，现在我们定义 `Config` 包含拥有的 `String` 值。`main` 中的 `args` 变量是参数值的所有者，并且只允许 `parse_config` 函数借用它们，这意味着如果 `Config` 试图获取 `args` 中值的所有权，我们将违反 Rust 的借用规则。

我们可以通过多种方式管理 `String` 数据；最简单但效率较低的方法是调用值的 `clone` 方法。这将为 `Config` 实例创建一个完整的数据副本，这比存储对字符串数据的引用需要更多的时间和内存。然而，克隆数据也使我们的代码非常直接，因为我们不必管理引用的生命周期；在这种情况下，放弃一点性能以获得简单性是一个值得的权衡。

> ### 使用 `clone` 的权衡
>
> 许多 Rust 程序员倾向于避免使用 `clone` 来解决所有权问题，因为它的运行时成本。在 [第 13 章][ch13]<!-- ignore --> 中，你将学习如何在这种类型的情况下使用更高效的方法。但现在，复制一些字符串以继续取得进展是可以的，因为你只会进行一次这些复制，而且文件路径和查询字符串非常小。拥有一个稍微低效但可以工作的程序比在第一次尝试时过度优化代码要好。随着你对 Rust 的经验增加，从最有效的解决方案开始会更容易，但现在调用 `clone` 是完全可接受的。

我们已经更新了 `main`，使其将 `parse_config` 返回的 `Config` 实例放入一个名为 `config` 的变量中，并且我们更新了以前使用单独的 `query` 和 `file_path` 变量的代码，使其现在使用 `Config` 结构体的字段。

现在我们的代码更清楚地传达了 `query` 和 `file_path` 是相关的，并且它们的目的是配置程序的工作方式。任何使用这些值的代码都知道在 `config` 实例中查找它们，这些字段的名称表明了它们的用途。

#### 为 `Config` 创建构造函数

到目前为止，我们已经从 `main` 中提取了负责解析命令行参数的逻辑，并将其放在 `parse_config` 函数中。这样做帮助我们看到了 `query` 和 `file_path` 值是相关的，并且这种关系应该在我们的代码中传达。然后我们添加了一个 `Config` 结构体来命名 `query` 和 `file_path` 的相关用途，并能够从 `parse_config` 函数中返回值的名称作为结构体字段名称。

现在 `parse_config` 函数的目的是创建一个 `Config` 实例，我们可以将 `parse_config` 从一个普通函数更改为一个与 `Config` 结构体关联的名为 `new` 的函数。进行此更改将使代码更符合习惯。我们可以通过调用 `String::new` 来创建标准库中的类型实例，例如 `String`。同样，通过将 `parse_config` 更改为与 `Config` 关联的 `new` 函数，我们将能够通过调用 `Config::new` 来创建 `Config` 的实例。Listing 12-7 显示了我们需要进行的更改。

<Listing number="12-7" file-name="src/main.rs" caption="将 `parse_config` 更改为 `Config::new`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

我们已经更新了 `main`，将调用 `parse_config` 的地方改为调用 `Config::new`。我们将 `parse_config` 的名称更改为 `new`，并将其移动到一个 `impl` 块中，该块将 `new` 函数与 `Config` 关联起来。再次尝试编译此代码以确保其正常工作。

### 修复错误处理

现在我们将修复错误处理。回想一下，如果 `args` 向量包含少于三个项目，尝试访问索引 1 或索引 2 处的值将导致程序崩溃。尝试在不带任何参数的情况下运行程序；它将如下所示：

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

`index out of bounds: the len is 1 but the index is 1` 这一行是面向程序员的错误消息。它不会帮助我们的最终用户理解他们应该做什么。现在让我们修复这个问题。

#### 改进错误消息

在 Listing 12-8 中，我们在 `new` 函数中添加了一个检查，该检查将在访问索引 1 和索引 2 之前验证切片是否足够长。如果切片不够长，程序将崩溃并显示更好的错误消息。

<Listing number="12-8" file-name="src/main.rs" caption="添加对参数数量的检查">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

这段代码类似于 [我们在 Listing 9-13 中编写的 `Guess::new` 函数][ch9-custom-types]<!-- ignore -->，当 `value` 参数超出有效值范围时，我们调用了 `panic!`。在这里，我们不是检查值的范围，而是检查 `args` 的长度是否至少为 `3`，并且函数的其余部分可以在此条件满足的情况下运行。如果 `args` 少于三个项目，此条件将为 `true`，我们将调用 `panic!` 宏以立即结束程序。

在 `new` 中添加这几行代码后，让我们再次运行程序，看看现在的错误是什么样子的：

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

这个输出更好：我们现在有一个合理的错误消息。然而，我们还有一些我们不希望提供给用户的额外信息。也许我们在 Listing 9-13 中使用的技术不是这里最好的选择：`panic!` 调用更适合编程问题而不是使用问题，[正如第 9 章中讨论的那样][ch9-error-guidelines]<!-- ignore -->。相反，我们将使用你在第 9 章中学到的另一种技术——[返回一个 `Result`][ch9-result]<!-- ignore -->，表示成功或错误。

<!-- 旧标题。不要删除或链接可能会中断。 -->

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### 返回 `Result` 而不是调用 `panic!`

我们可以改为返回一个 `Result` 值，该值在成功情况下包含 `Config` 实例，并在错误情况下描述问题。我们还将函数名称从 `new` 更改为 `build`，因为许多程序员期望 `new` 函数永远不会失败。当 `Config::build` 与 `main` 通信时，我们可以使用 `Result` 类型来指示存在问题。然后我们可以更改 `main` 以将 `Err` 变体转换为对用户更实用的错误，而不包含 `panic!` 调用引起的关于 `thread 'main'` 和 `RUST_BACKTRACE` 的周围文本。

Listing 12-9 显示了我们需要对现在称为 `Config::build` 的函数的返回值和函数体进行的更改，以返回 `Result`。请注意，在我们更新 `main` 之前，这不会编译，我们将在下一个列表中更新 `main`。

<Listing number="12-9" file-name="src/main.rs" caption="从 `Config::build` 返回 `Result`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

我们的 `build` 函数返回一个 `Result`，在成功情况下包含 `Config` 实例，在错误情况下包含一个字符串字面量。我们的错误值将始终是具有 `'static` 生命周期的字符串字面量。

我们在函数体中做了两个更改：当用户没有传递足够的参数时，我们现在返回一个 `Err` 值，而不是调用 `panic!`，并且我们将 `Config` 返回值包装在 `Ok` 中。这些更改使函数符合其新的类型签名。

从 `Config::build` 返回 `Err` 值允许 `main` 函数处理从 `build` 函数返回的 `Result` 值，并在错误情况下更干净地退出进程。

<!-- 旧标题。不要删除或链接可能会中断。 -->

<a id="calling-confignew-and-handling-errors"></a>

#### 调用 `Config::build` 并处理错误

为了处理错误情况并打印用户友好的消息，我们需要更新 `main` 以处理 `Config::build` 返回的 `Result`，如 Listing 12-10 所示。我们还将从 `panic!` 中移除退出命令行工具的责任，并手动实现它。非零退出状态是一个约定，用于向调用我们程序的进程发出信号，表示程序以错误状态退出。

<Listing number="12-10" file-name="src/main.rs" caption="如果构建 `Config` 失败，则退出并返回错误代码">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

在这个列表中，我们使用了一个我们还没有详细介绍的方法：`unwrap_or_else`，它由标准库在 `Result<T, E>` 上定义。使用 `unwrap_or_else` 允许我们定义一些自定义的非 `panic!` 错误处理。如果 `Result` 是一个 `Ok` 值，此方法的行为类似于 `unwrap`：它返回 `Ok` 包装的内部值。然而，如果值是一个 `Err` 值，此方法会调用我们定义并作为参数传递给 `unwrap_or_else` 的闭包中的代码。我们将在 [第 13 章][ch13]<!-- ignore --> 中更详细地介绍闭包。现在，你只需要知道 `unwrap_or_else` 会将 `Err` 的内部值（在本例中是我们添加到 Listing 12-9 中的静态字符串 `"not enough arguments"`）传递给出现在垂直管道之间的参数 `err` 中的闭包。然后，闭包中的代码可以在运行时使用 `err` 值。

我们添加了一个新的 `use` 行，将 `process` 从标准库引入作用域。在错误情况下运行的闭包中的代码只有两行：我们打印 `err` 值，然后调用 `process::exit`。`process::exit` 函数将立即停止程序并返回作为退出状态代码传递的数字。这与我们在 Listing 12-8 中使用的基于 `panic!` 的处理类似，但我们不再获得所有额外的输出。让我们试试：

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

太好了！这个输出对我们的用户来说更友好。

### 从 `main` 中提取逻辑

现在我们已经完成了配置解析的重构，让我们转向程序的逻辑。正如我们在 [“二进制项目的职责分离”](#separation-of-concerns-for-binary-projects)<!-- ignore --> 中所述，我们将提取一个名为 `run` 的函数，该函数将包含当前在 `main` 函数中的所有逻辑，这些逻辑不涉及设置配置或处理错误。完成后，`main` 将简洁且易于通过阅读验证，我们将能够为所有其他逻辑编写测试。

Listing 12-11 显示了提取的 `run` 函数。目前，我们只是进行小的增量改进，提取函数。我们仍然在 `src/main.rs` 中定义函数。

<Listing number="12-11" file-name="src/main.rs" caption="提取包含程序其余逻辑的 `run` 函数">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

`run` 函数现在包含 `main` 中从读取文件开始的所有剩余逻辑。`run` 函数将 `Config` 实例作为参数。

#### 从 `run` 函数返回错误

将剩余的程序逻辑分离到 `run` 函数中后，我们可以改进错误处理，就像我们在 Listing 12-9 中对 `Config::build` 所做的那样。`run` 函数将在出现问题时返回 `Result<T, E>`，而不是通过调用 `expect` 让程序崩溃。这将使我们能够进一步将错误处理逻辑整合到 `main` 中，以用户友好的方式处理错误。Listing 12-12 显示了我们需要对 `run` 的签名和主体进行的更改。

<Listing number="12-12" file-name="src/main.rs" caption="将 `run` 函数更改为返回 `Result`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

我们在这里做了三个重要的更改。首先，我们将 `run` 函数的返回类型更改为 `Result<(), Box<dyn Error>>`。此函数以前返回单元类型 `()`，我们将其保留为 `Ok` 情况下返回的值。

对于错误类型，我们使用了 _trait 对象_ `Box<dyn Error>`（并且我们已经在顶部使用 `use` 语句将 `std::error::Error` 引入作用域）。我们将在 [第 18 章][ch18]<!-- ignore --> 中介绍 trait 对象。现在，只需知道 `Box<dyn Error>` 意味着函数将返回一个实现了 `Error` trait 的类型，但我们不必指定返回值的具体类型。这使我们能够灵活地在不同的错误情况下返回不同类型的错误值。`dyn` 关键字是 _dynamic_ 的缩写。

其次，我们删除了对 `expect` 的调用，转而使用 `?` 操作符，正如我们在 [第 9 章][ch9-question-mark]<!-- ignore --> 中讨论的那样。`?` 操作符将在出现错误时从当前函数返回错误值，供调用者处理，而不是在错误时 `panic!`。

第三，`run` 函数现在在成功情况下返回一个 `Ok` 值。我们在签名中将 `run` 函数的成功类型声明为 `()`，这意味着我们需要将单元类型值包装在 `Ok` 值中。这种 `Ok(())` 语法一开始可能看起来有点奇怪，但像这样使用 `()` 是表示我们调用 `run` 仅为了其副作用的标准方式；它不返回我们需要使用的值。

当你运行此代码时，它将编译但会显示一个警告：

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust 告诉我们我们的代码忽略了 `Result` 值，并且 `Result` 值可能表明发生了错误。但我们没有检查是否有错误，编译器提醒我们可能应该在这里添加一些错误处理代码！现在让我们解决这个问题。

#### 在 `main` 中处理 `run` 返回的错误

我们将使用类似于我们在 Listing 12-10 中对 `Config::build` 使用的技术来检查错误并处理它们，但有一些细微差别：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

我们使用 `if let` 而不是 `unwrap_or_else` 来检查 `run` 是否返回 `Err` 值，并在返回时调用 `process::exit(1)`。`run` 函数不像 `Config::build` 那样返回我们想要 `unwrap` 的值。因为 `run` 在成功情况下返回 `()`，我们只关心检测错误，所以我们不需要 `unwrap_or_else` 来返回解包的值，这只会是 `()`。

`if let` 和 `unwrap_or_else` 函数的主体在两种情况下是相同的：我们打印错误并退出。

### 将代码拆分为库 crate

我们的 `minigrep` 项目到目前为止看起来不错！现在我们将拆分 `src/main.rs` 文件，并将一些代码放入 `src/lib.rs` 文件中。这样，我们可以测试代码，并拥有一个职责更少的 `src/main.rs` 文件。

让我们将所有不在 `main` 函数中的代码从 `src/main.rs` 移动到 `src/lib.rs`：

- `run` 函数定义
- 相关的 `use` 语句
- `Config` 的定义
- `Config::build` 函数定义

`src/lib.rs` 的内容应具有 Listing 12-13 中显示的签名（为了简洁起见，我们省略了函数的主体）。请注意，在我们修改 `src/main.rs` 之前，这不会编译，我们将在 Listing 12-14 中进行修改。

<Listing number="12-13" file-name="src/lib.rs" caption="将 `Config` 和 `run` 移动到 *src/lib.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs:here}}
```

</Listing>

我们大量使用了 `pub` 关键字：在 `Config` 上、其字段和其 `build` 方法上，以及在 `run` 函数上。我们现在有一个库 crate，它有一个我们可以测试的公共 API！

现在我们需要将我们移动到 `src/lib.rs` 的代码引入二进制 crate 的作用域中，如 Listing 12-14 所示。

<Listing number="12-14" file-name="src/main.rs" caption="在 *src/main.rs* 中使用 `minigrep` 库 crate">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

我们添加了一个 `use minigrep::Config` 行，将 `Config` 类型从库 crate 引入二进制 crate 的作用域，并在 `run` 函数前加上我们的 crate 名称。现在所有功能应该都已连接并且应该可以正常工作。使用 `cargo run` 运行程序，确保一切正常。

哇！这真是很多工作，但我们为未来的成功奠定了基础。现在处理错误要容易得多，而且代码更加模块化。从这里开始，几乎所有的工作都将在 `src/lib.rs` 中完成。

让我们利用这种新发现的模块化来做一些在旧代码中很难但在新代码中很容易的事情：我们将编写一些测试！

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator