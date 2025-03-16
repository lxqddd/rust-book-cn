## 使用 `Result` 处理可恢复错误

大多数错误并不严重到需要程序完全停止。有时，当函数失败时，原因可能是你可以轻松解释并响应的。例如，如果你尝试打开一个文件，但该操作因为文件不存在而失败，你可能希望创建该文件而不是终止进程。

回想一下第2章中的[“使用 `Result` 处理潜在失败”][handle_failure]，`Result` 枚举被定义为具有两个变体，`Ok` 和 `Err`，如下所示：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` 和 `E` 是泛型类型参数：我们将在第10章中更详细地讨论泛型。现在你需要知道的是，`T` 表示在 `Ok` 变体中成功情况下返回的值的类型，而 `E` 表示在 `Err` 变体中失败情况下返回的错误类型。因为 `Result` 有这些泛型类型参数，我们可以在许多不同的情况下使用 `Result` 类型及其定义的函数，其中我们想要返回的成功值和错误值可能不同。

让我们调用一个返回 `Result` 值的函数，因为该函数可能会失败。在 Listing 9-3 中，我们尝试打开一个文件。

<Listing number="9-3" file-name="src/main.rs" caption="打开一个文件">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

`File::open` 的返回类型是 `Result<T, E>`。泛型参数 `T` 由 `File::open` 的实现填充为成功值的类型，`std::fs::File`，这是一个文件句柄。错误值中使用的 `E` 类型是 `std::io::Error`。这个返回类型意味着调用 `File::open` 可能会成功并返回一个我们可以读取或写入的文件句柄。函数调用也可能会失败：例如，文件可能不存在，或者我们可能没有访问文件的权限。`File::open` 函数需要有一种方式来告诉我们它是否成功，并同时给我们文件句柄或错误信息。这正是 `Result` 枚举所传达的信息。

在 `File::open` 成功的情况下，变量 `greeting_file_result` 中的值将是一个包含文件句柄的 `Ok` 实例。在失败的情况下，`greeting_file_result` 中的值将是一个包含有关发生的错误类型更多信息的 `Err` 实例。

我们需要在 Listing 9-3 的代码中添加一些内容，以根据 `File::open` 返回的值采取不同的操作。Listing 9-4 展示了一种使用基本工具 `match` 表达式来处理 `Result` 的方法，我们在第6章讨论过 `match` 表达式。

<Listing number="9-4" file-name="src/main.rs" caption="使用 `match` 表达式处理可能返回的 `Result` 变体">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

请注意，与 `Option` 枚举一样，`Result` 枚举及其变体已经通过预导入模块引入作用域，因此我们不需要在 `match` 分支中指定 `Result::` 前缀。

当结果是 `Ok` 时，此代码将返回 `Ok` 变体中的内部 `file` 值，然后我们将该文件句柄值分配给变量 `greeting_file`。在 `match` 之后，我们可以使用该文件句柄进行读取或写入。

`match` 的另一个分支处理我们从 `File::open` 得到 `Err` 值的情况。在这个例子中，我们选择调用 `panic!` 宏。如果当前目录中没有名为 _hello.txt_ 的文件，并且我们运行此代码，我们将看到来自 `panic!` 宏的以下输出：

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

像往常一样，这个输出准确地告诉我们出了什么问题。

### 匹配不同的错误

Listing 9-4 中的代码无论 `File::open` 失败的原因是什么都会 `panic!`。然而，我们希望根据不同的失败原因采取不同的操作。如果 `File::open` 因为文件不存在而失败，我们希望创建该文件并返回新文件的句柄。如果 `File::open` 因为任何其他原因失败——例如，因为我们没有打开文件的权限——我们仍然希望代码像 Listing 9-4 中那样 `panic!`。为此，我们添加了一个内部的 `match` 表达式，如 Listing 9-5 所示。

<Listing number="9-5" file-name="src/main.rs" caption="以不同的方式处理不同类型的错误">

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

`File::open` 在 `Err` 变体中返回的值的类型是 `io::Error`，这是标准库提供的一个结构体。这个结构体有一个 `kind` 方法，我们可以调用它来获取一个 `io::ErrorKind` 值。`io::ErrorKind` 枚举由标准库提供，其变体表示可能由 `io` 操作导致的不同类型的错误。我们想要使用的变体是 `ErrorKind::NotFound`，它表示我们尝试打开的文件尚不存在。因此，我们在 `greeting_file_result` 上进行匹配，但我们也对 `error.kind()` 进行内部匹配。

我们在内部匹配中想要检查的条件是 `error.kind()` 返回的值是否是 `ErrorKind` 枚举的 `NotFound` 变体。如果是，我们尝试使用 `File::create` 创建文件。然而，因为 `File::create` 也可能失败，我们需要在内部 `match` 表达式中添加第二个分支。当文件无法创建时，会打印不同的错误消息。外部 `match` 的第二个分支保持不变，因此程序会在除文件缺失错误之外的任何错误上 panic。

> #### 使用 `match` 处理 `Result<T, E>` 的替代方案
>
> 这有很多 `match`！`match` 表达式非常有用，但也非常原始。在第13章中，你将学习闭包，它们可以与 `Result<T, E>` 上定义的许多方法一起使用。这些方法在处理 `Result<T, E>` 值时比使用 `match` 更简洁。
>
> 例如，下面是另一种编写与 Listing 9-5 相同逻辑的方式，这次使用闭包和 `unwrap_or_else` 方法：
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {error:?}");
>             })
>         } else {
>             panic!("Problem opening the file: {error:?}");
>         }
>     });
> }
> ```
>
> 尽管这段代码的行为与 Listing 9-5 相同，但它不包含任何 `match` 表达式，并且更易于阅读。在你阅读第13章后，再回到这个例子，并在标准库文档中查找 `unwrap_or_else` 方法。当你处理错误时，许多这些方法可以清理大量的嵌套 `match` 表达式。

#### 错误时 panic 的快捷方式：`unwrap` 和 `expect`

使用 `match` 效果很好，但它可能有点冗长，并且并不总是能很好地传达意图。`Result<T, E>` 类型定义了许多辅助方法来执行各种更具体的任务。`unwrap` 方法是一个快捷方法，其实现方式与我们编写的 `match` 表达式相同（如 Listing 9-4 所示）。如果 `Result` 值是 `Ok` 变体，`unwrap` 将返回 `Ok` 中的值。如果 `Result` 是 `Err` 变体，`unwrap` 将为我们调用 `panic!` 宏。以下是 `unwrap` 的实际示例：

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

如果我们运行此代码而没有 _hello.txt_ 文件，我们将看到来自 `unwrap` 方法调用的 `panic!` 的错误消息：

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

类似地，`expect` 方法让我们可以选择 `panic!` 错误消息。使用 `expect` 而不是 `unwrap` 并提供良好的错误消息可以传达你的意图，并使追踪 panic 的来源更容易。`expect` 的语法如下：

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

我们以与 `unwrap` 相同的方式使用 `expect`：返回文件句柄或调用 `panic!` 宏。`expect` 在其调用 `panic!` 时使用的错误消息将是我们传递给 `expect` 的参数，而不是 `unwrap` 使用的默认 `panic!` 消息。以下是它的样子：

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

在生产质量的代码中，大多数 Rust 开发者选择 `expect` 而不是 `unwrap`，并提供更多关于为什么操作预期总是成功的上下文。这样，如果你的假设被证明是错误的，你将有更多信息用于调试。

### 传播错误

当函数的实现调用可能失败的东西时，与其在函数本身内处理错误，不如将错误返回给调用代码，以便它可以决定如何处理。这被称为_传播_错误，并为调用代码提供了更多的控制权，因为调用代码可能有更多的信息或逻辑来决定如何处理错误，而不是你在代码上下文中可用的信息。

例如，Listing 9-6 展示了一个从文件中读取用户名的函数。如果文件不存在或无法读取，此函数将把这些错误返回给调用该函数的代码。

<Listing number="9-6" file-name="src/main.rs" caption="一个使用 `match` 将错误返回给调用代码的函数">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

这个函数可以用更短的方式编写，但我们将从手动完成大部分操作开始，以便探索错误处理；最后，我们将展示更短的方式。首先让我们看一下函数的返回类型：`Result<String, io::Error>`。这意味着函数返回一个类型为 `Result<T, E>` 的值，其中泛型参数 `T` 被具体类型 `String` 填充，泛型类型 `E` 被具体类型 `io::Error` 填充。

如果此函数成功且没有任何问题，调用此函数的代码将收到一个包含 `String` 的 `Ok` 值——该函数从文件中读取的用户名。如果此函数遇到任何问题，调用代码将收到一个包含 `io::Error` 实例的 `Err` 值，该实例包含有关问题的更多信息。我们选择 `io::Error` 作为此函数的返回类型，因为这是我们在函数体中调用的两个可能失败的操作返回的错误值类型：`File::open` 函数和 `read_to_string` 方法。

函数体首先调用 `File::open` 函数。然后我们使用类似于 Listing 9-4 中的 `match` 处理 `Result` 值。如果 `File::open` 成功，模式变量 `file` 中的文件句柄将成为可变变量 `username_file` 中的值，函数继续执行。在 `Err` 情况下，我们使用 `return` 关键字提前从函数中返回，并将 `File::open` 的错误值（现在在模式变量 `e` 中）作为此函数的错误值返回给调用代码。

因此，如果我们在 `username_file` 中有一个文件句柄，函数然后创建一个新的 `String` 变量 `username`，并在 `username_file` 中的文件句柄上调用 `read_to_string` 方法，将文件内容读入 `username`。`read_to_string` 方法也返回一个 `Result`，因为它可能会失败，即使 `File::open` 成功了。因此，我们需要另一个 `match` 来处理该 `Result`：如果 `read_to_string` 成功，那么我们的函数就成功了，我们返回现在在 `username` 中的文件中的用户名，包装在 `Ok` 中。如果 `read_to_string` 失败，我们以与处理 `File::open` 返回值时相同的方式返回错误值。然而，我们不需要显式地说 `return`，因为这是函数中的最后一个表达式。

调用此代码的代码将处理获取包含用户名的 `Ok` 值或包含 `io::Error` 的 `Err` 值。由调用代码决定如何处理这些值。如果调用代码收到 `Err` 值，它可以调用 `panic!` 并崩溃程序，使用默认用户名，或者从文件以外的其他地方查找用户名，例如。我们没有足够的信息来了解调用代码实际想要做什么，因此我们将所有成功或错误信息向上传播，以便它适当地处理。

这种传播错误的模式在 Rust 中非常常见，以至于 Rust 提供了问号运算符 `?` 来使这更容易。

#### 传播错误的快捷方式：`?` 运算符

Listing 9-7 展示了 `read_username_from_file` 的一个实现，其功能与 Listing 9-6 相同，但此实现使用了 `?` 运算符。

<Listing number="9-7" file-name="src/main.rs" caption="一个使用 `?` 运算符将错误返回给调用代码的函数">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

`?` 放在 `Result` 值之后，其定义方式与我们定义的 `match` 表达式几乎相同，用于处理 Listing 9-6 中的 `Result` 值。如果 `Result` 的值是 `Ok`，`Ok` 中的值将从该表达式中返回，程序将继续执行。如果值是 `Err`，`Err` 将从整个函数中返回，就像我们使用了 `return` 关键字一样，因此错误值会传播给调用代码。

Listing 9-6 中的 `match` 表达式与 `?` 运算符之间有一个区别：`?` 运算符调用的错误值会通过标准库中 `From` trait 定义的 `from` 函数进行转换，该函数用于将值从一种类型转换为另一种类型。当 `?` 运算符调用 `from` 函数时，接收到的错误类型将转换为当前函数返回类型中定义的错误类型。这在一个函数返回一个错误类型以表示函数可能失败的所有方式时非常有用，即使部分可能因许多不同的原因而失败。

例如，我们可以将 Listing 9-7 中的 `read_username_from_file` 函数更改为返回我们定义的自定义错误类型 `OurError`。如果我们还定义了 `impl From<io::Error> for OurError` 以从 `io::Error` 构造 `OurError` 的实例，那么 `read_username_from_file` 函数体中的 `?` 运算符调用将调用 `from` 并转换错误类型，而无需向函数添加更多代码。

在 Listing 9-7 的上下文中，`File::open` 调用末尾的 `?` 将返回 `Ok` 中的值给变量 `username_file`。如果发生错误，`?` 运算符将提前从整个函数中返回，并将任何 `Err` 值返回给调用代码。同样的逻辑适用于 `read_to_string` 调用末尾的 `?`。

`?` 运算符消除了大量的样板代码，并使此函数的实现更简单。我们甚至可以通过在 `?` 之后立即链接方法调用来进一步缩短此代码，如 Listing 9-8 所示。

<Listing number="9-8" file-name="src/main.rs" caption="在 `?` 运算符之后链接方法调用">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

我们将 `username` 中新的 `String` 的创建移到了函数的开头；这部分没有改变。我们没有创建变量 `username_file`，而是将 `read_to_string` 的调用直接链接到 `File::open("hello.txt")?` 的结果上。我们仍然在 `read_to_string` 调用的末尾有一个 `?`，并且当 `File::open` 和 `read_to_string` 都成功时，我们仍然返回包含 `username` 的 `Ok` 值，而不是返回错误。功能再次与 Listing 9-6 和 Listing 9-7 相同；这只是编写它的另一种更符合人体工程学的方式。

Listing 9-9 展示了使用 `fs::read_to_string` 使此代码更短的方式。

<Listing number="9-9" file-name="src/main.rs" caption="使用 `fs::read_to_string` 而不是打开然后读取文件">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

将文件读入字符串是一个相当常见的操作，因此标准库提供了方便的 `fs::read_to_string` 函数，它打开文件，创建一个新的 `String`，读取文件内容，将内容放入该 `String` 中，并返回它。当然，使用 `fs::read_to_string` 并没有给我们机会解释所有的错误处理，所以我们首先以更长的方式完成了它。

#### `?` 运算符可以在哪里使用

`?` 运算符只能用于返回类型与 `?` 使用的值兼容的函数。这是因为 `?` 运算符被定义为从函数中提前返回值，其方式与我们定义的 `match` 表达式相同（如 Listing 9-6 所示）。在 Listing 9-6 中，`match` 使用的是 `Result` 值，而提前返回的分支返回的是 `Err(e)` 值。函数的返回类型必须是 `Result`，以便与此 `return` 兼容。

在 Listing 9-10 中，让我们看看如果我们在 `main` 函数中使用 `?` 运算符，而返回类型与我们使用 `?` 的值的类型不兼容时，我们会得到什么错误。

<Listing number="9-10" file-name="src/main.rs" caption="尝试在返回 `()` 的 `main` 函数中使用 `?` 不会编译。">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

此代码打开一个文件，这可能会失败。`?` 运算符跟随 `File::open` 返回的 `Result` 值，但此 `main` 函数的返回类型是 `()`，而不是 `Result`。当我们编译此代码时，我们会得到以下错误消息：

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

此错误指出，我们只能在返回 `Result`、`Option` 或实现 `FromResidual` 的其他类型的函数中使用 `?` 运算符。

要修复此错误，你有两个选择。一个选择是更改函数的返回类型，使其与你使用 `?` 运算符的值兼容，只要你没有限制阻止这样做。另一个选择是使用 `match` 或 `Result<T, E>` 方法之一以适当的方式处理 `Result<T, E>`。

错误消息还提到 `?` 也可以与 `Option<T>` 值一起使用。与在 `Result` 上使用 `?` 一样，你只能在返回 `Option` 的函数中对 `Option` 使用 `?`。当在 `Option<T>` 上调用 `?` 运算符时，其行为与在 `Result<T, E>` 上调用时的行为类似：如果值是 `None`，`None` 将从函数中提前返回。如果值是 `Some`，`Some` 中的值是表达式的结果值，函数继续执行。Listing 9-11 有一个函数示例，它查找给定文本中第一行的最后一个字符。

<Listing number="9-11" caption="在 `Option<T>` 值上使用 `?` 运算符">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

此函数返回 `Option<char>`，因为那里可能有一个字符，但也可能没有。此代码接受 `text` 字符串切片参数，并对其调用 `lines` 方法，该方法返回字符串中行的迭代器。因为此函数想要检查第一行，它调用迭代器上的 `next` 以获取迭代器中的第一个值。如果 `text` 是空字符串，此 `next` 调用将返回 `None`，在这种情况下，我们使用 `?` 停止并从 `last_char_of_first_line` 返回 `None`。如果 `text` 不是空字符串，`next` 将返回一个包含 `text` 中第一行字符串切片的 `Some` 值。

`?` 提取字符串切片，我们可以对该字符串切片调用 `chars` 以获取其字符的迭代器。我们对第一行的最后一个字符感兴趣，因此我们调用 `last` 以返回迭代器中的最后一项。这是一个 `Option`，因为第一行可能是空字符串；例如，如果 `text` 以空行开头但在其他行上有字符，如 `"\nhi"`。然而，如果第一行上有最后一个字符，它将在 `Some` 变体中返回。中间的 `?` 运算符为我们提供了一种简洁的方式来表达此逻辑，允许我们在一行中实现该函数。如果我们不能在 `Option` 上使用 `?` 运算符，我们将不得不使用更多的方法调用或 `match` 表达式来实现此逻辑。

请注意，你可以在返回 `Result` 的函数中对 `Result` 使用 `?` 运算符，也可以在返回 `Option` 的函数中对 `Option` 使用 `?` 运算符，但你不能混用。`?` 运算符不会自动将 `Result` 转换为 `Option`，反之亦然；在这些情况下，你可以使用 `Result` 上的 `ok` 方法或 `Option` 上的 `ok_or` 方法显式进行转换。

到目前为止，我们使用的所有 `main` 函数都返回 `()`。`main` 函数是特殊的，因为它是可执行程序的入口点和退出点，并且对其返回类型有限制，以便程序按预期行为。

幸运的是，`main` 也可以返回 `Result<(), E>`。Listing 9-12 包含 Listing 9-10 中的代码，但我们将 `main` 的返回类型更改为 `Result<(), Box<dyn Error>>`，并在末尾添加了返回值 `Ok(())`。此代码现在将编译。

<Listing number="9-12" file-name="src/main.rs" caption="将 `main` 更改为返回 `Result<(), E>` 允许在 `Result` 值上使用 `?` 运算符。">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

`Box<dyn Error>` 类型是一个_特征对象_，我们将在第18章的[“使用允许不同类型值的特征对象”][trait-objects]中讨论它。现在，你可以将 `Box<dyn Error>` 理解为“任何类型的错误”。在 `main` 函数中使用 `?` 运算符处理 `Result` 值是允许的，因为它允许任何 `Err` 值提前返回。即使此 `main` 函数体只会返回 `std::io::Error` 类型的错误，通过指定 `Box<dyn Error>`，即使向 `main` 函数体添加更多返回其他错误的代码，此签名也将继续正确。

当 `main` 函数返回 `Result<(), E>` 时，如果 `main` 返回 `Ok(())`，可执行文件将以 `0` 退出；如果 `main` 返回 `Err` 值，可执行文件将以非零值退出。用 C 编写的可执行文件在退出时返回整数：成功退出的程序返回整数 `0`，而错误的程序返回其他非零整数。Rust 也从可执行文件返回整数以与此约定兼容。

`main` 函数可以返回任何实现[`std::process::Termination` trait][termination]的类型，该 trait 包含一个返回 `ExitCode` 的 `report` 函数。有关为你自己的类型实现 `Termination` trait 的更多信息，请参阅标准库文档。

现在我们已经讨论了调用 `panic!` 或返回 `Result` 的细节，让我们回到在哪些情况下使用哪种方法更合适的主题。

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[termination]: ../std/process/trait.Termination.html