## 使用环境变量

我们将通过添加一个额外的功能来改进 `minigrep`：用户可以通过环境变量开启不区分大小写的搜索功能。我们可以将这个功能作为一个命令行选项，并要求用户在每次想要使用时都输入它，但通过将其作为环境变量，我们允许用户设置一次环境变量，并在该终端会话中的所有搜索都不区分大小写。

### 为不区分大小写的 `search` 函数编写失败的测试

我们首先添加一个新的 `search_case_insensitive` 函数，当环境变量有值时将调用该函数。我们将继续遵循 TDD 流程，因此第一步仍然是编写一个失败的测试。我们将为新的 `search_case_insensitive` 函数添加一个新测试，并将旧测试从 `one_result` 重命名为 `case_sensitive`，以明确两个测试之间的区别，如 Listing 12-20 所示。

<Listing number="12-20" file-name="src/lib.rs" caption="为我们即将添加的不区分大小写函数添加一个新的失败测试">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

注意，我们也编辑了旧测试的 `contents`。我们添加了一行文本 `"Duct tape."`，其中使用了大写的 _D_，当我们在区分大小写的方式下搜索时，它不应该匹配查询 `"duct"`。通过这种方式更改旧测试有助于确保我们不会意外破坏已经实现的区分大小写的搜索功能。这个测试现在应该通过，并且在我们处理不区分大小写的搜索时应该继续通过。

用于不区分大小写搜索的新测试使用 `"rUsT"` 作为查询。在我们即将添加的 `search_case_insensitive` 函数中，查询 `"rUsT"` 应该匹配包含大写 _R_ 的 `"Rust:"` 行，并且匹配 `"Trust me."` 行，尽管它们的字母大小写与查询不同。这是我们的失败测试，它将无法编译，因为我们还没有定义 `search_case_insensitive` 函数。你可以随意添加一个总是返回空向量的骨架实现，类似于我们在 Listing 12-16 中为 `search` 函数所做的，以查看测试编译并失败。

### 实现 `search_case_insensitive` 函数

`search_case_insensitive` 函数，如 Listing 12-21 所示，将与 `search` 函数几乎相同。唯一的区别是我们将 `query` 和每行 `line` 都转换为小写，这样无论输入参数的大小写如何，它们在检查行是否包含查询时都将具有相同的大小写。

<Listing number="12-21" file-name="src/lib.rs" caption="定义 `search_case_insensitive` 函数，在比较之前将查询和行转换为小写">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

首先，我们将 `query` 字符串转换为小写并将其存储在一个同名的新变量中，遮蔽了原始的 `query`。对查询调用 `to_lowercase` 是必要的，这样无论用户的查询是 `"rust"`、`"RUST"`、`"Rust"` 还是 `"rUsT"`，我们都会将查询视为 `"rust"` 并且不区分大小写。虽然 `to_lowercase` 会处理基本的 Unicode，但它不会 100% 准确。如果我们在编写一个真正的应用程序，我们会希望在这里做更多的工作，但本节是关于环境变量的，而不是 Unicode，所以我们在这里就到此为止。

注意，`query` 现在是一个 `String` 而不是字符串切片，因为调用 `to_lowercase` 会创建新数据而不是引用现有数据。以查询 `"rUsT"` 为例：该字符串切片不包含小写的 `u` 或 `t` 供我们使用，因此我们必须分配一个包含 `"rust"` 的新 `String`。当我们现在将 `query` 作为参数传递给 `contains` 方法时，我们需要添加一个 `&`，因为 `contains` 的签名定义为接受一个字符串切片。

接下来，我们在每行 `line` 上添加 `to_lowercase` 调用，将所有字符转换为小写。现在我们已经将 `line` 和 `query` 转换为小写，无论查询的大小写如何，我们都会找到匹配项。

让我们看看这个实现是否通过了测试：

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

太好了！它们通过了。现在，让我们从 `run` 函数中调用新的 `search_case_insensitive` 函数。首先，我们将在 `Config` 结构体中添加一个配置选项，以在区分大小写和不区分大小写的搜索之间切换。添加此字段将导致编译器错误，因为我们还没有在任何地方初始化此字段：

<span class="filename">文件名: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/lib.rs:here}}
```

我们添加了 `ignore_case` 字段，它保存一个布尔值。接下来，我们需要 `run` 函数检查 `ignore_case` 字段的值，并使用它来决定是调用 `search` 函数还是 `search_case_insensitive` 函数，如 Listing 12-22 所示。这仍然无法编译。

<Listing number="12-22" file-name="src/lib.rs" caption="根据 `config.ignore_case` 中的值调用 `search` 或 `search_case_insensitive`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/lib.rs:there}}
```

</Listing>

最后，我们需要检查环境变量。用于处理环境变量的函数位于标准库中的 `env` 模块中，因此我们将该模块引入到 _src/lib.rs_ 的顶部。然后我们将使用 `env` 模块中的 `var` 函数来检查是否设置了名为 `IGNORE_CASE` 的环境变量，如 Listing 12-23 所示。

<Listing number="12-23" file-name="src/lib.rs" caption="检查名为 `IGNORE_CASE` 的环境变量是否有值">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/lib.rs:here}}
```

</Listing>

在这里，我们创建了一个新变量 `ignore_case`。为了设置它的值，我们调用 `env::var` 函数并将 `IGNORE_CASE` 环境变量的名称传递给它。`env::var` 函数返回一个 `Result`，如果环境变量设置为任何值，它将是一个包含环境变量值的成功 `Ok` 变体。如果环境变量未设置，它将返回 `Err` 变体。

我们使用 `Result` 上的 `is_ok` 方法来检查环境变量是否设置，这意味着程序应该执行不区分大小写的搜索。如果 `IGNORE_CASE` 环境变量未设置任何值，`is_ok` 将返回 `false`，程序将执行区分大小写的搜索。我们不关心环境变量的 _值_，只关心它是否设置，因此我们检查 `is_ok` 而不是使用 `unwrap`、`expect` 或我们在 `Result` 上看到的其他方法。

我们将 `ignore_case` 变量中的值传递给 `Config` 实例，以便 `run` 函数可以读取该值并决定是调用 `search_case_insensitive` 还是 `search`，正如我们在 Listing 12-22 中实现的那样。

让我们试试吧！首先，我们将在没有设置环境变量的情况下运行我们的程序，并使用查询 `to`，它应该匹配任何包含小写 _to_ 的行：

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

看起来仍然有效！现在让我们在设置 `IGNORE_CASE` 为 `1` 的情况下运行程序，但使用相同的查询 _to_：

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

如果你使用的是 PowerShell，你需要将设置环境变量和运行程序作为单独的命令：

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

这将使 `IGNORE_CASE` 在你的 shell 会话的剩余时间内保持有效。可以使用 `Remove-Item` cmdlet 取消设置：

```console
PS> Remove-Item Env:IGNORE_CASE
```

我们应该得到包含 _to_ 的行，这些行可能包含大写字母：

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

太好了，我们也得到了包含 _To_ 的行！我们的 `minigrep` 程序现在可以通过环境变量控制进行不区分大小写的搜索。现在你知道如何管理使用命令行参数或环境变量设置的选项了。

一些程序允许同时使用参数 _和_ 环境变量进行相同的配置。在这些情况下，程序决定哪一个优先。作为另一个练习，尝试通过命令行参数或环境变量控制大小写敏感性。如果程序运行时一个设置为区分大小写，另一个设置为忽略大小写，请决定命令行参数或环境变量应该优先。

`std::env` 模块包含许多用于处理环境变量的有用功能：查看其文档以了解可用的功能。