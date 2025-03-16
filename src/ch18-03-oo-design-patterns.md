## 实现面向对象的设计模式

_状态模式_ 是一种面向对象的设计模式。该模式的核心在于我们定义了一组值可以拥有的内部状态。这些状态由一组 _状态对象_ 表示，值的行为会根据其状态而变化。我们将通过一个博客文章结构的示例来演示，该结构有一个字段来保存其状态，状态对象可以是“草稿”、“审核中”或“已发布”中的一种。

状态对象共享功能：在 Rust 中，我们使用结构体和 trait 而不是对象和继承。每个状态对象负责自己的行为，并控制何时应该转换为另一个状态。持有状态对象的值对状态的不同行为或状态之间的转换时机一无所知。

使用状态模式的好处是，当程序的需求发生变化时，我们不需要修改持有状态的值的代码或使用该值的代码。我们只需要更新其中一个状态对象的代码来改变其规则，或者可能添加更多的状态对象。

首先，我们将以更传统的面向对象方式实现状态模式，然后我们将使用一种更符合 Rust 风格的方法。让我们逐步实现一个使用状态模式的博客文章工作流。

最终的功能将如下所示：

1. 博客文章从空草稿开始。
2. 当草稿完成后，请求对文章进行审核。
3. 当文章被批准后，它将被发布。
4. 只有已发布的博客文章会返回要打印的内容，因此未批准的文章不会意外发布。

任何其他尝试对文章进行的更改都应该无效。例如，如果我们尝试在请求审核之前批准一篇草稿博客文章，文章应保持为未发布的草稿。

Listing 18-11 展示了这个工作流的代码形式：这是我们将在一个名为 `blog` 的库 crate 中实现的 API 的示例用法。由于我们还没有实现 `blog` crate，这段代码目前还无法编译。

<Listing number="18-11" file-name="src/main.rs" caption="展示我们希望 `blog` crate 具有的行为的代码">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

我们希望允许用户使用 `Post::new` 创建一个新的草稿博客文章。我们希望允许向博客文章添加文本。如果我们在批准之前立即尝试获取文章的内容，我们不应该得到任何文本，因为文章仍然是草稿。我们在代码中添加了 `assert_eq!` 用于演示目的。一个优秀的单元测试将是断言草稿博客文章从 `content` 方法返回一个空字符串，但我们不会为这个示例编写测试。

接下来，我们希望启用对文章的审核请求，并且希望在等待审核时 `content` 返回一个空字符串。当文章获得批准后，它应该被发布，这意味着当调用 `content` 时，文章的文本将被返回。

请注意，我们从 crate 中交互的唯一类型是 `Post` 类型。该类型将使用状态模式，并将持有一个值，该值将是表示文章可能处于的各种状态的三个状态对象之一——草稿、审核中或已发布。从一个状态到另一个状态的转换将在 `Post` 类型内部管理。状态的变化是对我们库的用户在 `Post` 实例上调用的方法的响应，但他们不必直接管理状态的变化。此外，用户不会在状态上犯错，例如在文章被审核之前发布它。

### 定义 `Post` 并在草稿状态下创建新实例

让我们开始实现这个库！我们知道我们需要一个公共的 `Post` 结构体来保存一些内容，因此我们将从结构体的定义和一个关联的公共 `new` 函数开始，以创建一个 `Post` 实例，如 Listing 18-12 所示。我们还将创建一个私有的 `State` trait，它将定义所有 `Post` 的状态对象必须具有的行为。

然后，`Post` 将在名为 `state` 的私有字段中持有一个 `Box<dyn State>` 的 trait 对象，该字段位于 `Option<T>` 中，以保存状态对象。稍后你会看到为什么需要 `Option<T>`。

<Listing number="18-12" file-name="src/lib.rs" caption="定义 `Post` 结构体和 `new` 函数，创建一个新的 `Post` 实例，以及 `State` trait 和 `Draft` 结构体">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

`State` trait 定义了不同文章状态共享的行为。状态对象是 `Draft`、`PendingReview` 和 `Published`，它们都将实现 `State` trait。目前，该 trait 没有任何方法，我们将从定义 `Draft` 状态开始，因为这是我们希望文章开始的状态。

当我们创建一个新的 `Post` 时，我们将其 `state` 字段设置为一个 `Some` 值，该值持有一个 `Box`。这个 `Box` 指向 `Draft` 结构体的一个新实例。这确保了每当我们创建一个新的 `Post` 实例时，它都将以草稿状态开始。由于 `Post` 的 `state` 字段是私有的，因此无法以任何其他状态创建 `Post`！在 `Post::new` 函数中，我们将 `content` 字段设置为一个新的空 `String`。

### 存储文章内容的文本

我们在 Listing 18-11 中看到，我们希望能够调用一个名为 `add_text` 的方法，并传递一个 `&str`，然后将其添加为博客文章的文本内容。我们将其实现为一个方法，而不是将 `content` 字段公开为 `pub`，以便稍后我们可以实现一个方法来控制如何读取 `content` 字段的数据。`add_text` 方法非常简单，因此让我们将 Listing 18-13 中的实现添加到 `impl Post` 块中。

<Listing number="18-13" file-name="src/lib.rs" caption="实现 `add_text` 方法以向文章的 `content` 添加文本">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

`add_text` 方法接受一个对 `self` 的可变引用，因为我们正在更改调用 `add_text` 的 `Post` 实例。然后我们在 `content` 中的 `String` 上调用 `push_str`，并传递 `text` 参数以添加到保存的 `content` 中。这种行为不依赖于文章所处的状态，因此它不是状态模式的一部分。`add_text` 方法根本不与 `state` 字段交互，但它是我们希望支持的行为的一部分。

### 确保草稿文章的内容为空

即使在我们调用了 `add_text` 并向文章添加了一些内容之后，我们仍然希望 `content` 方法返回一个空字符串切片，因为文章仍然处于草稿状态，如 Listing 18-11 的第 7 行所示。现在，让我们用最简单的实现来满足这个要求：始终返回一个空字符串切片。稍后，当我们实现了更改文章状态以便可以发布的功能时，我们将更改这一点。到目前为止，文章只能处于草稿状态，因此文章内容应始终为空。Listing 18-14 展示了这个占位符实现。

<Listing number="18-14" file-name="src/lib.rs" caption="为 `Post` 上的 `content` 方法添加一个占位符实现，始终返回一个空字符串切片">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

通过添加这个 `content` 方法，Listing 18-11 中到第 7 行的所有内容都按预期工作。

<!-- Old link, do not remove -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>

### 请求审核会更改文章的状态

接下来，我们需要添加请求审核文章的功能，这应该将其状态从 `Draft` 更改为 `PendingReview`。Listing 18-15 展示了这段代码。

<Listing number="18-15" file-name="src/lib.rs" caption="在 `Post` 和 `State` trait 上实现 `request_review` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

我们为 `Post` 提供了一个名为 `request_review` 的公共方法，该方法将接受一个对 `self` 的可变引用。然后我们在 `Post` 的当前状态上调用一个内部的 `request_review` 方法，这个第二个 `request_review` 方法会消耗当前状态并返回一个新状态。

我们将 `request_review` 方法添加到 `State` trait 中；所有实现该 trait 的类型现在都需要实现 `request_review` 方法。请注意，方法的第一个参数不是 `self`、`&self` 或 `&mut self`，而是 `self: Box<Self>`。这种语法意味着该方法仅在调用持有该类型的 `Box` 时有效。这种语法会获取 `Box<Self>` 的所有权，使旧状态无效，以便 `Post` 的状态值可以转换为新状态。

为了消耗旧状态，`request_review` 方法需要获取状态值的所有权。这就是 `Post` 的 `state` 字段中的 `Option` 的用武之地：我们调用 `take` 方法从 `state` 字段中取出 `Some` 值，并在其位置留下一个 `None`，因为 Rust 不允许我们在结构体中有未填充的字段。这让我们可以将 `state` 值移出 `Post`，而不是借用它。然后我们将把文章的 `state` 值设置为这个操作的结果。

我们需要暂时将 `state` 设置为 `None`，而不是直接使用类似 `self.state = self.state.request_review();` 的代码来设置它，以获取 `state` 值的所有权。这确保了 `Post` 在我们将其转换为新状态后不能使用旧的 `state` 值。

`Draft` 上的 `request_review` 方法返回一个新的、装箱的 `PendingReview` 结构体实例，该结构体表示文章等待审核时的状态。`PendingReview` 结构体也实现了 `request_review` 方法，但不进行任何转换。相反，它返回自身，因为当我们在已经处于 `PendingReview` 状态的文章上请求审核时，它应该保持在 `PendingReview` 状态。

现在我们可以开始看到状态模式的优势：无论 `Post` 的 `state` 值是什么，`request_review` 方法都是相同的。每个状态都负责自己的规则。

我们将 `Post` 上的 `content` 方法保持不变，返回一个空字符串切片。我们现在可以让 `Post` 处于 `PendingReview` 状态以及 `Draft` 状态，但我们希望在 `PendingReview` 状态下具有相同的行为。Listing 18-11 现在可以工作到第 10 行！

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>

### 添加 `approve` 以更改 `content` 的行为

`approve` 方法将与 `request_review` 方法类似：它将把 `state` 设置为当前状态在被批准时应具有的值，如 Listing 18-16 所示：

<Listing number="18-16" file-name="src/lib.rs" caption="在 `Post` 和 `State` trait 上实现 `approve` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

我们将 `approve` 方法添加到 `State` trait 中，并添加一个新的结构体来实现 `State`，即 `Published` 状态。

与 `PendingReview` 上的 `request_review` 类似，如果我们在 `Draft` 上调用 `approve` 方法，它将没有任何效果，因为 `approve` 将返回 `self`。当我们在 `PendingReview` 上调用 `approve` 时，它返回一个新的、装箱的 `Published` 结构体实例。`Published` 结构体实现了 `State` trait，并且对于 `request_review` 方法和 `approve` 方法，它都返回自身，因为在这些情况下，文章应保持在 `Published` 状态。

现在我们需要更新 `Post` 上的 `content` 方法。我们希望从 `content` 返回的值取决于 `Post` 的当前状态，因此我们将让 `Post` 委托给其 `state` 上定义的 `content` 方法，如 Listing 18-17 所示：

<Listing number="18-17" file-name="src/lib.rs" caption="更新 `Post` 上的 `content` 方法以委托给 `State` 上的 `content` 方法">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

因为目标是将所有这些规则保留在实现 `State` 的结构体中，所以我们调用 `state` 值上的 `content` 方法，并将文章实例（即 `self`）作为参数传递。然后我们返回使用 `state` 值上的 `content` 方法返回的值。

我们在 `Option` 上调用 `as_ref` 方法，因为我们想要 `Option` 内部值的引用，而不是该值的所有权。因为 `state` 是一个 `Option<Box<dyn State>>`，当我们调用 `as_ref` 时，会返回一个 `Option<&Box<dyn State>>`。如果我们不调用 `as_ref`，我们会得到一个错误，因为我们不能将 `state` 从函数参数的 `&self` 中移出。

然后我们调用 `unwrap` 方法，我们知道它永远不会 panic，因为我们知道 `Post` 上的方法确保在方法完成后 `state` 将始终包含一个 `Some` 值。这是我们在第 9 章中讨论的 [“编译器无法理解的情况下你有更多信息”][more-info-than-rustc]<!-- ignore --> 的情况之一，我们知道 `None` 值永远不可能，即使编译器无法理解这一点。

此时，当我们在 `&Box<dyn State>` 上调用 `content` 时，解引用强制转换将对 `&` 和 `Box` 生效，因此 `content` 方法最终将在实现 `State` trait 的类型上调用。这意味着我们需要将 `content` 添加到 `State` trait 定义中，这就是我们将根据我们拥有的状态来决定返回什么内容的逻辑所在，如 Listing 18-18 所示：

<Listing number="18-18" file-name="src/lib.rs" caption="将 `content` 方法添加到 `State` trait 中">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

我们为 `content` 方法添加了一个默认实现，返回一个空字符串切片。这意味着我们不需要在 `Draft` 和 `PendingReview` 结构体上实现 `content`。`Published` 结构体将覆盖 `content` 方法并返回 `post.content` 中的值。

请注意，我们需要在这个方法上添加生命周期注解，正如我们在第 10 章中讨论的那样。我们正在将 `post` 的引用作为参数，并返回该 `post` 的一部分的引用，因此返回的引用的生命周期与 `post` 参数的生命周期相关。

我们完成了——Listing 18-11 中的所有内容现在都可以工作了！我们已经使用状态模式实现了博客文章工作流的规则。与规则相关的逻辑位于状态对象中，而不是分散在 `Post` 中。

> ### 为什么不使用枚举？
>
> 你可能想知道为什么我们不使用一个枚举，将不同的文章状态作为变体。这当然是一个可能的解决方案；尝试一下并比较最终结果，看看你更喜欢哪个！使用枚举的一个缺点是，每个检查枚举值的地方都需要一个 `match` 表达式或类似的东西来处理每个可能的变体。这可能会比这个 trait 对象解决方案更重复。

### 状态模式的权衡

我们已经展示了 Rust 能够实现面向对象的状态模式，以封装文章在每个状态下应具有的不同行为。`Post` 上的方法对不同的行为一无所知。我们组织代码的方式是，我们只需要查看一个地方就可以知道已发布文章的不同行为方式：`Published` 结构体上的 `State` trait 的实现。

如果我们创建一个不使用状态模式的替代实现，我们可能会在 `Post` 上的方法中甚至是在检查文章状态并更改行为的 `main` 代码中使用 `match` 表达式。这意味着我们将不得不在多个地方查看以了解文章处于发布状态的所有含义！随着我们添加更多的状态，这种情况只会增加：每个 `match` 表达式都需要另一个分支。

使用状态模式，`Post` 方法和我们使用 `Post` 的地方不需要 `match` 表达式，要添加一个新状态，我们只需要添加一个新结构体并在该结构体上实现 trait 方法。

使用状态模式的实现很容易扩展以添加更多功能。要查看使用状态模式的代码的简单维护性，请尝试以下一些建议：

- 添加一个 `reject` 方法，将文章的状态从 `PendingReview` 更改回 `Draft`。
- 要求在状态更改为 `Published` 之前调用两次 `approve`。
- 仅允许用户在文章处于 `Draft` 状态时添加文本内容。提示：让状态对象负责可能更改的内容，但不负责修改 `Post`。

状态模式的一个缺点是，由于状态实现了状态之间的转换，一些状态是相互耦合的。如果我们在 `PendingReview` 和 `Published` 之间添加另一个状态，例如 `Scheduled`，我们将不得不更改 `PendingReview` 中的代码以转换到 `Scheduled` 而不是 `Published`。如果 `PendingReview` 不需要随着新状态的添加而更改，那么工作量会少一些，但这将意味着切换到另一种设计模式。

另一个缺点是我们重复了一些逻辑。为了消除一些重复，我们可能会尝试为 `State` trait 上的 `request_review` 和 `approve` 方法提供默认实现，返回 `self`；然而，这不会起作用：当使用 `State` 作为 trait 对象时，trait 不知道具体的 `self` 到底是什么，因此返回类型在编译时是未知的。（这是前面提到的 dyn 兼容性规则之一。）

其他重复包括 `Post` 上 `request_review` 和 `approve` 方法的类似实现。这两种方法都委托给 `Option` 中 `state` 字段值的相同方法的实现，并将 `state` 字段的新值设置为结果。如果我们在 `Post` 上有许多遵循这种模式的方法，我们可能会考虑定义一个宏来消除重复（参见第 20 章中的 [“宏”][macros]<!-- ignore -->）。

通过完全按照面向对象语言定义的状态模式实现，我们没有充分利用 Rust 的优势。让我们看看我们可以对 `blog` crate 进行哪些更改，以使无效状态和转换成为编译时错误。

#### 将状态和行为编码为类型

我们将向你展示如何重新思考状态模式以获得不同的权衡。与其完全封装状态和转换以使外部代码对它们一无所知，不如将状态编码为不同的类型。因此，Rust 的类型检查系统将通过发出编译器错误来防止在只允许发布文章的地方使用草稿文章。

让我们考虑 Listing 18-11 中 `main` 的第一部分：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

我们仍然允许使用 `Post::new` 创建处于草稿状态的新文章，并允许向文章的内容添加文本。但我们不会让草稿文章有一个返回空字符串的 `content` 方法，而是让草稿文章根本没有 `content` 方法。这样，如果我们尝试获取草稿文章的内容，我们会得到一个编译器错误，告诉我们该方法不存在。因此，我们不可能在生产中意外显示草稿文章的内容，因为该代码甚至无法编译。Listing 18-19 展示了 `Post` 结构体和 `DraftPost` 结构体的定义，以及每个结构体上的方法。

<Listing number="18-19" file-name="src/lib.rs" caption="带有 `content` 方法的 `Post` 和不带 `content` 方法的 `DraftPost`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

`Post` 和 `DraftPost` 结构体都有一个私有的 `content` 字段，用于存储博客文章的文本。结构体不再有 `state` 字段，因为我们将状态的编码移到了结构体的类型中。`Post` 结构体将表示已发布的文章，并且它有一个 `content` 方法，返回 `content`。

我们仍然有一个 `Post::new` 函数，但它不返回 `Post` 的实例，而是返回 `DraftPost` 的实例。因为 `content` 是私有的，并且没有任何函数返回 `Post`，所以现在不可能创建 `Post` 的实例。

`DraftPost` 结构体有一个 `add_text` 方法，因此我们可以像以前一样向 `content` 添加文本，但请注意，`DraftPost` 没有定义 `content` 方法！因此，现在程序确保所有文章都以草稿文章开始，并且草稿文章的内容不可用于显示。任何绕过这些约束的尝试都会导致编译器错误。

#### 将转换实现为转换为不同类型

那么我们如何获得已发布的文章呢？我们希望强制执行规则，即草稿文章必须经过审核和批准才能发布。处于待审核状态的文章仍然不应显示任何内容。让我们通过添加另一个结构体 `PendingReviewPost` 来实现这些约束，定义 `DraftPost` 上的 `request_review` 方法以返回 `PendingReviewPost`，并定义 `PendingReviewPost` 上的 `approve` 方法以返回 `Post`，如 Listing 18-20 所示。

<Listing number="18-20" file-name="src/lib.rs" caption="通过调用 `DraftPost` 上的 `request_review` 创建的 `PendingReviewPost` 和将 `PendingReviewPost` 转换为已发布的 `Post` 的 `approve` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

`request_review` 和 `approve` 方法获取 `self` 的所有权，从而消耗 `DraftPost` 和 `PendingReviewPost` 实例，并将它们分别转换为 `PendingReviewPost` 和已发布的 `Post`。这样，我们在调用 `request_review` 后就不会有任何遗留的 `DraftPost` 实例，依此类推。`PendingReviewPost` 结构体没有定义 `content` 方法，因此尝试读取其内容会导致编译器错误，就像 `DraftPost` 一样。因为唯一获得具有 `content` 方法的已发布 `Post` 实例的方法是调用 `PendingReviewPost` 上的 `approve` 方法，而唯一获得 `PendingReviewPost` 的方法是调用 `DraftPost` 上的 `request_review` 方法，我们现在已将博客文章工作流编码到类型系统中。

但我们还需要对 `main` 进行一些小的更改。`request_review` 和 `approve` 方法返回新实例而不是修改它们所调用的结构体，因此我们需要添加更多的 `let post =` 影子赋值来保存返回的实例。我们也不能让关于草稿和待审核文章内容的断言为空字符串，我们也不需要它们：我们不能再编译尝试使用这些状态下文章内容的代码。更新后的 `main` 代码如 Listing 18-21 所示。

<Listing number="18-21" file-name="src/main.rs" caption="修改 `main` 以使用新的博客文章工作流实现">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

我们需要对 `main` 进行的更改以重新分配 `post` 意味着这个实现不再完全遵循面向对象的状态模式：状态之间的转换不再完全封装在 `Post` 实现中。然而，我们的收获是，由于类型系统和编译时发生的类型检查，无效状态现在是不可能的！这确保了某些错误，例如显示未发布文章的内容，将在它们进入生产环境之前被发现。

尝试在本节开头建议的任务，看看你对这个版本代码设计的看法。请注意，在这个设计中，一些任务可能已经完成。

我们已经看到，尽管 Rust 能够实现面向对象的设计模式，但其他模式，例如将状态编码到类型系统中，在 Rust 中也是可用的。这些模式有不同的权衡。尽管你可能非常熟悉面向对象的模式，但重新思考问题以利用 Rust 的特性可以带来好处，例如在编译时防止某些错误。由于某些特性（如所有权），面向对象的模式在 Rust 中并不总是最佳解决方案。

## 总结

无论你在阅读本章后是否认为 Rust 是一种面向对象的语言，你现在知道你可以使用 trait 对象在 Rust 中获得一些面向对象的特性。动态调度可以为你的代码提供一些灵活性，以换取一点运行时性能。你可以使用这种灵活性来实现有助于代码可维护性的面向对象模式。Rust 还具有其他特性，如所有权，这是面向对象语言所没有的。面向对象的模式并不总是利用 Rust 优势的最佳方式，但它是一个可用的选项。

接下来，我们将看看模式，这是 Rust 的另一个特性，提供了很大的灵活性。我们在整本书中简要地看过它们，但还没有看到它们的全部能力。让我们开始吧！

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros