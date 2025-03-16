## 使用测试驱动开发（TDD）开发库的功能

现在我们已经将逻辑提取到 `src/lib.rs` 中，并将参数收集和错误处理留在 `src/main.rs` 中，这样更容易为代码的核心功能编写测试。我们可以直接调用函数并传入各种参数，检查返回值，而不必从命令行调用二进制文件。

在本节中，我们将使用测试驱动开发（TDD）过程为 `minigrep` 程序添加搜索逻辑，步骤如下：

1. 编写一个失败的测试，并运行它以确保它因你预期的原因而失败。
2. 编写或修改足够的代码以使新测试通过。
3. 重构你刚刚添加或更改的代码，并确保测试继续通过。
4. 从步骤 1 重复！

尽管这只是编写软件的众多方法之一，但 TDD 可以帮助驱动代码设计。在编写使测试通过的代码之前编写测试，有助于在整个过程中保持高测试覆盖率。

我们将通过测试驱动的方式实现一个功能，该功能将在文件内容中搜索查询字符串，并生成与查询匹配的行列表。我们将在名为 `search` 的函数中添加此功能。

### 编写一个失败的测试

因为我们不再需要它们，所以让我们从 `src/lib.rs` 和 `src/main.rs` 中删除用于检查程序行为的 `println!` 语句。然后，在 `src/lib.rs` 中，我们将添加一个 `tests` 模块和一个测试函数，就像我们在[第 11 章][ch11-anatomy]中所做的那样。测试函数指定了我们希望 `search` 函数具有的行为：它将接收一个查询和要搜索的文本，并返回仅包含查询的文本行。Listing 12-15 显示了这个测试，它目前还无法编译。

<Listing number="12-15" file-name="src/lib.rs" caption="为我们希望拥有的 `search` 函数创建一个失败的测试">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

这个测试搜索字符串 `"duct"`。我们搜索的文本有三行，其中只有一行包含 `"duct"`（注意，开头的双引号后的反斜杠告诉 Rust 不要在这个字符串字面量的开头添加换行符）。我们断言从 `search` 函数返回的值仅包含我们期望的行。

我们还不能运行这个测试并观察它失败，因为测试甚至无法编译：`search` 函数还不存在！根据 TDD 原则，我们将添加足够的代码以使测试能够编译和运行，通过添加一个总是返回空向量的 `search` 函数定义，如 Listing 12-16 所示。然后测试应该能够编译并失败，因为空向量与包含行 `"safe, fast, productive."` 的向量不匹配。

<Listing number="12-16" file-name="src/lib.rs" caption="定义足够的 `search` 函数以使我们的测试能够编译">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

注意，我们需要在 `search` 的签名中定义一个显式的生命周期 `'a`，并在 `contents` 参数和返回值中使用该生命周期。回想一下[第 10 章][ch10-lifetimes]，生命周期参数指定了哪个参数的生命周期与返回值的生命周期相关联。在这种情况下，我们指示返回的向量应包含引用 `contents` 参数切片的字符串切片（而不是 `query` 参数）。

换句话说，我们告诉 Rust，`search` 函数返回的数据将与传入 `search` 函数的 `contents` 参数中的数据一样长。这很重要！切片引用的数据需要有效，引用才能有效；如果编译器假设我们正在创建 `query` 的字符串切片而不是 `contents`，它将错误地进行安全检查。

如果我们忘记生命周期注解并尝试编译此函数，我们将得到以下错误：

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust 无法知道我们需要两个参数中的哪一个，所以我们需要明确告诉它。因为 `contents` 是包含我们所有文本的参数，并且我们想要返回与该文本匹配的部分，所以我们知道 `contents` 是应该使用生命周期语法与返回值连接的参数。

其他编程语言不要求你在签名中将参数与返回值连接起来，但这种做法会随着时间的推移变得更容易。你可能想将这个例子与[第 10 章][validating-references-with-lifetimes]中的“使用生命周期验证引用”部分中的示例进行比较。

现在让我们运行测试：

```console
{{#include ../listings/ch12-an-io-project/listing-12-16/output.txt}}
```

很好，测试失败了，正如我们预期的那样。让我们让测试通过！

### 编写代码以使测试通过

目前，我们的测试失败了，因为我们总是返回一个空向量。为了修复这个问题并实现 `search`，我们的程序需要遵循以下步骤：

1. 遍历内容的每一行。
2. 检查该行是否包含我们的查询字符串。
3. 如果包含，则将其添加到我们要返回的值列表中。
4. 如果不包含，则不做任何操作。
5. 返回匹配的结果列表。

让我们逐步完成每个步骤，从遍历行开始。

#### 使用 `lines` 方法遍历行

Rust 有一个方便的方法来处理字符串的逐行迭代，恰当地命名为 `lines`，如 Listing 12-17 所示。请注意，这还无法编译。

<Listing number="12-17" file-name="src/lib.rs" caption="遍历 `contents` 中的每一行">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

`lines` 方法返回一个迭代器。我们将在[第 13 章][ch13-iterators]中深入讨论迭代器，但回想一下，你在[Listing 3-5][ch3-iter]中看到了这种使用迭代器的方式，在那里我们使用 `for` 循环和迭代器对集合中的每个项目运行一些代码。

#### 在每行中搜索查询

接下来，我们将检查当前行是否包含我们的查询字符串。幸运的是，字符串有一个名为 `contains` 的有用方法可以为我们做到这一点！在 `search` 函数中添加对 `contains` 方法的调用，如 Listing 12-18 所示。请注意，这仍然无法编译。

<Listing number="12-18" file-name="src/lib.rs" caption="添加功能以检查行是否包含 `query` 中的字符串">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

目前，我们正在构建功能。为了使代码能够编译，我们需要从函数体中返回一个值，正如我们在函数签名中所指示的那样。

#### 存储匹配的行

为了完成这个函数，我们需要一种方法来存储我们要返回的匹配行。为此，我们可以在 `for` 循环之前创建一个可变的向量，并调用 `push` 方法将 `line` 存储在向量中。在 `for` 循环之后，我们返回向量，如 Listing 12-19 所示。

<Listing number="12-19" file-name="src/lib.rs" caption="存储匹配的行以便我们可以返回它们">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

现在 `search` 函数应该只返回包含 `query` 的行，我们的测试应该通过。让我们运行测试：

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

我们的测试通过了，所以我们知道它有效！

此时，我们可以考虑在保持测试通过以保持相同功能的同时，重构搜索函数的实现。搜索函数中的代码并不算太糟糕，但它没有利用迭代器的一些有用功能。我们将在[第 13 章][ch13-iterators]中回到这个例子，详细探讨迭代器，并看看如何改进它。

#### 在 `run` 函数中使用 `search` 函数

现在 `search` 函数已经工作并通过了测试，我们需要从 `run` 函数中调用 `search`。我们需要将 `config.query` 值和 `run` 从文件中读取的 `contents` 传递给 `search` 函数。然后 `run` 将打印从 `search` 返回的每一行：

<span class="filename">文件名: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/src/lib.rs:here}}
```

我们仍然使用 `for` 循环来返回 `search` 中的每一行并打印它。

现在整个程序应该可以工作了！让我们试试看，首先用一个应该从 Emily Dickinson 的诗中返回一行的词：_frog_。

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

很好！现在让我们尝试一个会匹配多行的词，比如 _body_：

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

最后，让我们确保当我们搜索一个不在诗中的词时，不会得到任何行，比如 _monomorphization_：

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

太棒了！我们已经构建了自己的经典工具的迷你版本，并学到了很多关于如何构建应用程序的知识。我们还学到了一些关于文件输入和输出、生命周期、测试和命令行解析的知识。

为了完成这个项目，我们将简要演示如何使用环境变量以及如何打印到标准错误，这两者在编写命令行程序时都非常有用。

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html