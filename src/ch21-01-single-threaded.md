## 构建一个单线程的 Web 服务器

我们将从构建一个单线程的 Web 服务器开始。在开始之前，让我们先快速了解一下构建 Web 服务器所涉及的协议。这些协议的细节超出了本书的范围，但简要概述将为你提供所需的信息。

Web 服务器涉及的两个主要协议是**超文本传输协议**（**HTTP**）和**传输控制协议**（**TCP**）。这两种协议都是**请求-响应**协议，意味着**客户端**发起请求，**服务器**监听请求并向客户端提供响应。这些请求和响应的内容由协议定义。

TCP 是较低级别的协议，它描述了信息如何从一个服务器传递到另一个服务器的细节，但没有指定这些信息的内容。HTTP 建立在 TCP 之上，定义了请求和响应的内容。从技术上讲，HTTP 可以与其他协议一起使用，但在绝大多数情况下，HTTP 通过 TCP 发送其数据。我们将处理 TCP 和 HTTP 请求和响应的原始字节。

### 监听 TCP 连接

我们的 Web 服务器需要监听 TCP 连接，因此这是我们首先要处理的部分。标准库提供了一个 `std::net` 模块，让我们可以做到这一点。让我们以通常的方式创建一个新项目：

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

现在在 _src/main.rs_ 中输入代码，如 Listing 21-1 所示。这段代码将在本地地址 `127.0.0.1:7878` 上监听传入的 TCP 流。当它接收到一个传入的流时，它将打印 `Connection established!`。

<Listing number="21-1" file-name="src/main.rs" caption="监听传入的流并在接收到流时打印消息">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

使用 `TcpListener`，我们可以在地址 `127.0.0.1:7878` 上监听 TCP 连接。在地址中，冒号前的部分是代表你计算机的 IP 地址（这在每台计算机上都是相同的，并不特指作者的计算机），而 `7878` 是端口。我们选择这个端口有两个原因：HTTP 通常不接受这个端口，因此我们的服务器不太可能与你机器上运行的其他 Web 服务器冲突，而且 7878 在电话键盘上拼写为 _rust_。

在这个场景中，`bind` 函数的工作方式类似于 `new` 函数，它将返回一个新的 `TcpListener` 实例。这个函数被称为 `bind`，因为在网络编程中，连接到端口以进行监听被称为“绑定到端口”。

`bind` 函数返回一个 `Result<T, E>`，这表明绑定可能会失败。例如，连接到端口 80 需要管理员权限（非管理员只能监听高于 1023 的端口），因此如果我们尝试在没有管理员权限的情况下连接到端口 80，绑定将失败。例如，如果我们运行两个程序实例并让两个程序监听同一个端口，绑定也会失败。因为我们正在编写一个用于学习的基本服务器，所以我们不会担心处理这些类型的错误；相反，我们使用 `unwrap` 在发生错误时停止程序。

`TcpListener` 上的 `incoming` 方法返回一个迭代器，它为我们提供了一系列流（更具体地说，是类型为 `TcpStream` 的流）。单个**流**表示客户端和服务器之间的开放连接。**连接**是客户端连接到服务器、服务器生成响应以及服务器关闭连接的完整请求和响应过程的名称。因此，我们将从 `TcpStream` 中读取客户端发送的内容，然后将我们的响应写入流中以将数据发送回客户端。总的来说，这个 `for` 循环将依次处理每个连接，并为我们生成一系列流来处理。

目前，我们对流的处理包括调用 `unwrap` 以在流出现任何错误时终止程序；如果没有错误，程序将打印一条消息。我们将在下一个 Listing 中为成功的情况添加更多功能。当客户端连接到服务器时，我们可能会从 `incoming` 方法收到错误的原因是，我们实际上并不是在迭代连接。相反，我们是在迭代**连接尝试**。连接可能由于多种原因而不成功，其中许多原因是特定于操作系统的。例如，许多操作系统对它们可以支持的并发开放连接数量有限制；超过该数量的新连接尝试将产生错误，直到一些开放连接被关闭。

让我们尝试运行这段代码！在终端中调用 `cargo run`，然后在 Web 浏览器中加载 _127.0.0.1:7878_。浏览器应该会显示类似“连接重置”的错误消息，因为服务器当前没有返回任何数据。但是当你查看终端时，你应该会看到几条消息，这些消息是在浏览器连接到服务器时打印的！

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

有时你会看到一条浏览器请求打印了多条消息；原因可能是浏览器正在请求页面以及其他资源，例如出现在浏览器标签中的 _favicon.ico_ 图标。

也可能是浏览器多次尝试连接到服务器，因为服务器没有返回任何数据。当 `stream` 在循环结束时超出范围并被丢弃时，连接会作为 `drop` 实现的一部分关闭。浏览器有时会通过重试来处理关闭的连接，因为问题可能是暂时的。重要的是，我们已经成功获取了 TCP 连接的句柄！

记得在运行特定版本的代码后按 <kbd>ctrl</kbd>-<kbd>c</kbd> 停止程序。然后在每次代码更改后通过调用 `cargo run` 命令重新启动程序，以确保你运行的是最新的代码。

### 读取请求

让我们实现从浏览器读取请求的功能！为了将获取连接和处理连接的操作分开，我们将为处理连接启动一个新函数。在这个新的 `handle_connection` 函数中，我们将从 TCP 流中读取数据并打印出来，以便我们可以看到浏览器发送的数据。将代码更改为 Listing 21-2 所示。

<Listing number="21-2" file-name="src/main.rs" caption="从 `TcpStream` 中读取数据并打印数据">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

我们将 `std::io::prelude` 和 `std::io::BufReader` 引入作用域，以获取允许我们从流中读取和写入的特性和类型。在 `main` 函数的 `for` 循环中，我们现在调用新的 `handle_connection` 函数并将 `stream` 传递给它，而不是打印一条消息说我们建立了连接。

在 `handle_connection` 函数中，我们创建了一个新的 `BufReader` 实例，它包装了对 `stream` 的引用。`BufReader` 通过为我们管理对 `std::io::Read` 特性方法的调用来添加缓冲。

我们创建了一个名为 `http_request` 的变量来收集浏览器发送到我们服务器的请求行。我们通过添加 `Vec<_>` 类型注解来表明我们希望将这些行收集到一个向量中。

`BufReader` 实现了 `std::io::BufRead` 特性，该特性提供了 `lines` 方法。`lines` 方法通过每当看到换行字节时拆分数据流来返回一个 `Result<String, std::io::Error>` 的迭代器。为了获取每个 `String`，我们映射并 `unwrap` 每个 `Result`。如果数据不是有效的 UTF-8 或者从流中读取时出现问题，`Result` 可能是一个错误。再次强调，生产程序应该更优雅地处理这些错误，但为了简单起见，我们选择在错误情况下停止程序。

浏览器通过连续发送两个换行字符来指示 HTTP 请求的结束，因此为了从流中获取一个请求，我们获取行直到得到一个空字符串。一旦我们将这些行收集到向量中，我们就会使用漂亮的调试格式打印它们，以便我们可以查看 Web 浏览器发送给我们服务器的指令。

让我们尝试这段代码！启动程序并在 Web 浏览器中再次发出请求。请注意，我们仍然会在浏览器中收到错误页面，但我们的程序在终端中的输出现在应该类似于以下内容：

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

根据你的浏览器，你可能会得到稍微不同的输出。现在我们正在打印请求数据，我们可以通过查看请求第一行中 `GET` 之后的路径来了解为什么我们会从一个浏览器请求中获得多个连接。如果重复的连接都在请求 _/_，我们知道浏览器正在尝试重复获取 _/_，因为它没有从我们的程序中获得响应。

让我们分解这个请求数据，以了解浏览器对我们的程序提出了什么要求。

### 深入了解 HTTP 请求

HTTP 是一个基于文本的协议，请求的格式如下：

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

第一行是**请求行**，它包含有关客户端请求的信息。请求行的第一部分指示正在使用的**方法**，例如 `GET` 或 `POST`，它描述了客户端如何发出此请求。我们的客户端使用了 `GET` 请求，这意味着它正在请求信息。

请求行的下一部分是 _/_，它指示客户端请求的**统一资源标识符**（**URI**）：URI 几乎与**统一资源定位符**（**URL**）相同，但不完全相同。URI 和 URL 之间的区别对于本章的目的并不重要，但 HTTP 规范使用术语 URI，因此我们可以在这里将 _URL_ 替换为 _URI_。

最后一部分是客户端使用的 HTTP 版本，然后请求行以 CRLF 序列结束。（CRLF 代表**回车**和**换行**，这是打字机时代的术语！）CRLF 序列也可以写成 `\r\n`，其中 `\r` 是回车，`\n` 是换行。**CRLF 序列**将请求行与请求数据的其余部分分开。请注意，当打印 CRLF 时，我们看到的是新行的开始，而不是 `\r\n`。

查看我们从运行程序到目前为止收到的请求行数据，我们看到 `GET` 是方法，_/_ 是请求 URI，`HTTP/1.1` 是版本。

在请求行之后，从 `Host:` 开始的其余行是标头。`GET` 请求没有主体。

尝试从不同的浏览器发出请求或请求不同的地址，例如 _127.0.0.1:7878/test_，以查看请求数据如何变化。

现在我们知道浏览器在请求什么，让我们发送一些数据回去！

### 编写响应

我们将实现发送数据以响应客户端请求的功能。响应的格式如下：

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

第一行是**状态行**，它包含响应中使用的 HTTP 版本、总结请求结果的数字状态代码以及提供状态代码文本描述的原因短语。在 CRLF 序列之后是任何标头，另一个 CRLF 序列，以及响应的主体。

以下是一个使用 HTTP 版本 1.1、状态代码为 200、原因短语为 OK、没有标头且没有主体的示例响应：

```text
HTTP/1.1 200 OK\r\n\r\n
```

状态代码 200 是标准的成功响应。文本是一个微小的成功 HTTP 响应。让我们将这个写入流中作为我们对成功请求的响应！从 `handle_connection` 函数中删除打印请求数据的 `println!`，并将其替换为 Listing 21-3 中的代码。

<Listing number="21-3" file-name="src/main.rs" caption="将微小的成功 HTTP 响应写入流中">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

第一行定义了 `response` 变量，它保存了成功消息的数据。然后我们在 `response` 上调用 `as_bytes` 将字符串数据转换为字节。`stream` 上的 `write_all` 方法接受一个 `&[u8]` 并将这些字节直接发送到连接中。因为 `write_all` 操作可能会失败，所以我们像以前一样对任何错误结果使用 `unwrap`。再次强调，在实际应用程序中，你应该在这里添加错误处理。

通过这些更改，让我们运行我们的代码并发出请求。我们不再向终端打印任何数据，因此除了 Cargo 的输出外，我们不会看到任何输出。当你在 Web 浏览器中加载 _127.0.0.1:7878_ 时，你应该得到一个空白页面而不是错误。你已经手动编码接收 HTTP 请求并发送响应！

### 返回真实的 HTML

让我们实现返回不仅仅是空白页面的功能。在你的项目目录的根目录中创建新文件 _hello.html_，而不是在 _src_ 目录中。你可以输入任何你想要的 HTML；Listing 21-4 显示了一种可能性。

<Listing number="21-4" file-name="hello.html" caption="在响应中返回的示例 HTML 文件">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

这是一个带有标题和一些文本的最小 HTML5 文档。为了在收到请求时从服务器返回此内容，我们将修改 `handle_connection`，如 Listing 21-5 所示，以读取 HTML 文件，将其作为主体添加到响应中并发送。

<Listing number="21-5" file-name="src/main.rs" caption="将 *hello.html* 的内容作为响应的主体发送">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

我们将 `fs` 添加到 `use` 语句中，以将标准库的文件系统模块引入作用域。读取文件内容到字符串的代码应该看起来很熟悉；我们在 Listing 12-4 中读取文件内容时使用了它。

接下来，我们使用 `format!` 将文件的内容添加为成功响应的主体。为了确保有效的 HTTP 响应，我们添加了 `Content-Length` 标头，它设置为响应主体的大小，在这种情况下是 `hello.html` 的大小。

使用 `cargo run` 运行此代码，并在浏览器中加载 _127.0.0.1:7878_；你应该会看到你的 HTML 被渲染！

目前，我们忽略了 `http_request` 中的请求数据，只是无条件地发送回 HTML 文件的内容。这意味着如果你在浏览器中请求 _127.0.0.1:7878/something-else_，你仍然会得到相同的 HTML 响应。目前，我们的服务器非常有限，没有做大多数 Web 服务器所做的事情。我们希望根据请求自定义我们的响应，并且只对格式良好的 _/_ 请求发送回 HTML 文件。

### 验证请求并选择性响应

目前，我们的 Web 服务器将返回文件中的 HTML，无论客户端请求什么。让我们添加功能来检查浏览器是否在请求 _/_，然后再返回 HTML 文件，并在浏览器请求其他内容时返回错误。为此，我们需要修改 `handle_connection`，如 Listing 21-6 所示。这段新代码将接收到的请求内容与我们已知的 _/_ 请求进行比较，并添加 `if` 和 `else` 块以不同方式处理请求。

<Listing number="21-6" file-name="src/main.rs" caption="处理 */* 请求与其他请求不同">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

我们只会查看 HTTP 请求的第一行，因此我们调用 `next` 来获取迭代器中的第一项，而不是将整个请求读入向量。第一个 `unwrap` 处理 `Option`，如果迭代器没有项，则停止程序。第二个 `unwrap` 处理 `Result`，其效果与 Listing 21-2 中添加的 `map` 中的 `unwrap` 相同。

接下来，我们检查 `request_line` 是否等于 _/_ 路径的 GET 请求行。如果是，`if` 块返回我们的 HTML 文件的内容。

如果 `request_line` 不等于 _/_ 路径的 GET 请求行，这意味着我们收到了其他请求。我们稍后将在 `else` 块中添加代码以响应所有其他请求。

现在运行此代码并请求 _127.0.0.1:7878_；你应该会得到 _hello.html_ 中的 HTML。如果你发出任何其他请求，例如 _127.0.0.1:7878/something-else_，你将得到一个连接错误，就像你在运行 Listing 21-1 和 Listing 21-2 中的代码时看到的那样。

现在让我们将 Listing 21-7 中的代码添加到 `else` 块中，以返回状态代码为 404 的响应，该响应表示未找到请求的内容。我们还将返回一些 HTML，以便在浏览器中呈现给最终用户的响应页面。

<Listing number="21-7" file-name="src/main.rs" caption="如果请求的不是 */*，则返回状态代码 404 和错误页面">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

在这里，我们的响应有一个状态行，状态代码为 404，原因短语为 `NOT FOUND`。响应的主体将是文件 _404.html_ 中的 HTML。你需要在 _hello.html_ 旁边创建一个 _404.html_ 文件作为错误页面；再次随意使用任何你想要的 HTML 或使用 Listing 21-8 中的示例 HTML。

<Listing number="21-8" file-name="404.html" caption="随任何 404 响应发送回的页面示例内容">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

通过这些更改，再次运行你的服务器。请求 _127.0.0.1:7878_ 应该返回 _hello.html_ 的内容，而任何其他请求，如 _127.0.0.1:7878/foo_，应该返回 _404.html_ 中的错误 HTML。

### 一点重构

目前，`if` 和 `else` 块中有很多重复：它们都在读取文件并将文件的内容写入流中。唯一的区别是状态行和文件名。让我们通过将这些差异提取到单独的 `if` 和 `else` 行中来使代码更简洁，这些行将状态行和文件名的值分配给变量；然后我们可以在代码中无条件地使用这些变量来读取文件并写入响应。Listing 21-9 显示了替换大 `if` 和 `else` 块后的结果代码。

<Listing number="21-9" file-name="src/main.rs" caption="重构 `if` 和 `else` 块，使其仅包含两种情况之间不同的代码">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

现在 `if` 和 `else` 块只返回状态行和文件名的适当值；然后我们使用解构将这些值分配给 `status_line` 和 `filename`，如第 19 章中讨论的那样。

之前重复的代码现在位于 `if` 和 `else` 块之外，并使用 `status_line` 和 `filename` 变量。这使得更容易看到两种情况之间的差异，并且这意味着如果我们想更改文件读取和响应写入的工作方式，我们只有一个地方可以更新代码。Listing 21-9 中的代码行为将与 Listing 21-7 中的代码行为相同。

太棒了！我们现在有一个大约 40 行 Rust 代码的简单 Web 服务器，它响应一个请求并返回一个内容页面，并响应所有其他请求并返回 404 响应。

目前，我们的服务器在单线程中运行，这意味着它一次只能处理一个请求。让我们通过模拟一些慢请求来检查这如何成为一个问题。然后我们将修复它，以便我们的服务器可以同时处理多个请求。