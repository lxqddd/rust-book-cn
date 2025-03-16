## 改进我们的 I/O 项目

通过了解迭代器的新知识，我们可以改进第 12 章中的 I/O 项目，使用迭代器使代码中的某些部分更加清晰和简洁。让我们看看迭代器如何改进我们对 `Config::build` 函数和 `search` 函数的实现。

### 使用迭代器移除 `clone`

在 Listing 12-6 中，我们添加了代码，该代码获取 `String` 值的切片并通过索引切片并克隆值来创建 `Config` 结构体的实例，从而使 `Config` 结构体拥有这些值。在 Listing 13-17 中，我们重现了 Listing 12-23 中的 `Config::build` 函数的实现。

<Listing number="13-17" file-name="src/lib.rs" caption="Listing 12-23 中 `Config::build` 函数的重现">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/lib.rs:ch13}}
```

</Listing>

当时，我们说不用担心低效的 `clone` 调用，因为我们将来会移除它们。现在就是那个时候了！

我们需要在这里使用 `clone`，因为参数 `args` 中有一个包含 `String` 元素的切片，但 `build` 函数并不拥有 `args`。为了返回 `Config` 实例的所有权，我们必须克隆 `Config` 的 `query` 和 `file_path` 字段中的值，以便 `Config` 实例可以拥有这些值。

通过我们对迭代器的新知识，我们可以将 `build` 函数改为获取迭代器的所有权作为参数，而不是借用切片。我们将使用迭代器功能，而不是检查切片长度并索引特定位置的代码。这将澄清 `Config::build` 函数的作用，因为迭代器将访问这些值。

一旦 `Config::build` 获取了迭代器的所有权并停止使用借用的索引操作，我们就可以将迭代器中的 `String` 值移动到 `Config` 中，而不是调用 `clone` 并进行新的分配。

#### 直接使用返回的迭代器

打开你的 I/O 项目的 _src/main.rs_ 文件，它应该如下所示：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

我们首先将 Listing 12-24 中的 `main` 函数的开头改为 Listing 13-18 中的代码，这次使用迭代器。在我们更新 `Config::build` 之前，这不会编译。

<Listing number="13-18" file-name="src/main.rs" caption="将 `env::args` 的返回值传递给 `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

`env::args` 函数返回一个迭代器！与其将迭代器值收集到一个向量中，然后将切片传递给 `Config::build`，现在我们直接将 `env::args` 返回的迭代器的所有权传递给 `Config::build`。

接下来，我们需要更新 `Config::build` 的定义。在你的 I/O 项目的 _src/lib.rs_ 文件中，让我们将 `Config::build` 的签名改为如 Listing 13-19 所示。这仍然不会编译，因为我们需要更新函数体。

<Listing number="13-19" file-name="src/lib.rs" caption="更新 `Config::build` 的签名以期望一个迭代器">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/lib.rs:here}}
```

</Listing>

`env::args` 函数的标准库文档显示，它返回的迭代器类型是 `std::env::Args`，并且该类型实现了 `Iterator` trait 并返回 `String` 值。

我们更新了 `Config::build` 函数的签名，使参数 `args` 具有一个泛型类型，其 trait 约束为 `impl Iterator<Item = String>`，而不是 `&[String]`。我们在第 10 章的 [“Traits as Parameters”][impl-trait]<!-- ignore --> 部分讨论的 `impl Trait` 语法的这种用法意味着 `args` 可以是任何实现 `Iterator` trait 并返回 `String` 项的类型。

因为我们正在获取 `args` 的所有权，并且我们将通过迭代它来改变 `args`，所以我们可以在 `args` 参数的规范中添加 `mut` 关键字以使其可变。

#### 使用 `Iterator` Trait 方法代替索引

接下来，我们将修复 `Config::build` 的函数体。因为 `args` 实现了 `Iterator` trait，我们知道我们可以调用它的 `next` 方法！Listing 13-20 更新了 Listing 12-23 中的代码以使用 `next` 方法。

<Listing number="13-20" file-name="src/lib.rs" caption="更改 `Config::build` 的函数体以使用迭代器方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/lib.rs:here}}
```

</Listing>

记住，`env::args` 返回值的第一个值是程序的名称。我们希望忽略它并获取下一个值，所以我们首先调用 `next` 并不处理返回值。然后我们调用 `next` 来获取我们想要放入 `Config` 的 `query` 字段的值。如果 `next` 返回 `Some`，我们使用 `match` 提取值。如果它返回 `None`，这意味着没有提供足够的参数，我们提前返回一个 `Err` 值。我们对 `file_path` 值做同样的事情。

### 使用迭代器适配器使代码更清晰

我们还可以在我们的 I/O 项目中的 `search` 函数中利用迭代器，该函数在 Listing 13-21 中重现，如 Listing 12-19 所示：

<Listing number="13-21" file-name="src/lib.rs" caption="Listing 12-19 中 `search` 函数的实现">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

我们可以使用迭代器适配器方法以更简洁的方式编写此代码。这样做还可以避免使用可变的中间 `results` 向量。函数式编程风格倾向于最小化可变状态的数量，以使代码更清晰。移除可变状态可能会使未来的增强功能更容易实现，例如并行搜索，因为我们不必管理对 `results` 向量的并发访问。Listing 13-22 显示了这一变化：

<Listing number="13-22" file-name="src/lib.rs" caption="在 `search` 函数的实现中使用迭代器适配器方法">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

回想一下，`search` 函数的目的是返回 `contents` 中包含 `query` 的所有行。与 Listing 13-16 中的 `filter` 示例类似，此代码使用 `filter` 适配器仅保留 `line.contains(query)` 返回 `true` 的行。然后我们使用 `collect` 将匹配的行收集到另一个向量中。简单多了！你也可以在 `search_case_insensitive` 函数中做出相同的更改以使用迭代器方法。

### 在循环或迭代器之间选择

下一个逻辑问题是，你应该在自己的代码中选择哪种风格以及为什么：Listing 13-21 中的原始实现还是 Listing 13-22 中使用迭代器的版本。大多数 Rust 程序员更喜欢使用迭代器风格。起初有点难以掌握，但一旦你熟悉了各种迭代器适配器及其功能，迭代器可能会更容易理解。与其摆弄各种循环和构建新向量的细节，代码更关注循环的高级目标。这抽象出了一些常见的代码，因此更容易看到此代码特有的概念，例如迭代器中每个元素必须通过的过滤条件。

但这两个实现真的等价吗？直观的假设可能是低级循环会更快。让我们谈谈性能。

[impl-trait]: ch10-02-traits.html#traits-as-parameters