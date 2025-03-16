## 注释

所有程序员都努力使他们的代码易于理解，但有时需要额外的解释。在这些情况下，程序员会在源代码中留下 _注释_，编译器会忽略这些注释，但阅读源代码的人可能会发现它们很有用。

这是一个简单的注释：

```rust
// hello, world
```

在 Rust 中，惯用的注释风格是以两个斜杠开始注释，注释会一直持续到行尾。对于跨越多行的注释，你需要在每一行都包含 `//`，像这样：

```rust
// 我们在这里做一些复杂的事情，复杂到需要
// 多行注释来解释！哇！希望这个注释能
// 解释清楚发生了什么。
```

注释也可以放在包含代码的行尾：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

但你更常见的是看到它们以这种格式使用，注释位于它所注解的代码上方的一行：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust 还有另一种注释，称为文档注释，我们将在第 14 章的 [“发布 Crate 到 Crates.io”][publishing]<!-- ignore --> 部分讨论。

[publishing]: ch14-02-publishing-to-crates-io.html