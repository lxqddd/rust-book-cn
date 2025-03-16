## 使用哈希映射存储键值对

我们常见的集合中的最后一个是 _哈希映射_。类型 `HashMap<K, V>` 使用 _哈希函数_ 存储键类型 `K` 到值类型 `V` 的映射，哈希函数决定了如何将这些键和值放入内存中。许多编程语言都支持这种数据结构，但它们通常使用不同的名称，例如 _哈希_、_映射_、_对象_、_哈希表_、_字典_ 或 _关联数组_，仅举几例。

当你不想通过索引（就像使用向量那样）来查找数据，而是想通过一个可以是任意类型的键来查找数据时，哈希映射非常有用。例如，在一个游戏中，你可以使用哈希映射来跟踪每个队伍的分数，其中每个键是队伍的名称，值是该队伍的分数。给定一个队伍名称，你可以检索其分数。

我们将在本节中介绍哈希映射的基本 API，但标准库在 `HashMap<K, V>` 上定义的函数中隐藏了更多好东西。一如既往，请查看标准库文档以获取更多信息。

### 创建一个新的哈希映射

创建一个空哈希映射的一种方法是使用 `new` 并通过 `insert` 添加元素。在 Listing 8-20 中，我们跟踪了两个队伍的分数，队伍名称分别为 _Blue_ 和 _Yellow_。Blue 队从 10 分开始，Yellow 队从 50 分开始。

<Listing number="8-20" caption="创建一个新的哈希映射并插入一些键和值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

注意，我们需要首先从标准库的集合部分 `use` `HashMap`。在我们常见的三种集合中，这种集合使用频率最低，因此它没有包含在预导入中自动引入的功能中。哈希映射在标准库中的支持也较少；例如，没有内置的宏来构造它们。

就像向量一样，哈希映射将其数据存储在堆上。这个 `HashMap` 的键类型为 `String`，值类型为 `i32`。与向量一样，哈希映射是同构的：所有的键必须具有相同的类型，所有的值也必须具有相同的类型。

### 访问哈希映射中的值

我们可以通过将键提供给 `get` 方法来获取哈希映射中的值，如 Listing 8-21 所示。

<Listing number="8-21" caption="访问存储在哈希映射中的 Blue 队伍的分数">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

在这里，`score` 将具有与 Blue 队伍关联的值，结果将是 `10`。`get` 方法返回一个 `Option<&V>`；如果哈希映射中没有该键的值，`get` 将返回 `None`。这个程序通过调用 `copied` 来处理 `Option`，以获取一个 `Option<i32>` 而不是 `Option<&i32>`，然后使用 `unwrap_or` 将 `score` 设置为零，如果 `scores` 中没有该键的条目。

我们可以像处理向量一样，使用 `for` 循环遍历哈希映射中的每个键值对：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

这段代码将以任意顺序打印每对键值：

```text
Yellow: 50
Blue: 10
```

### 哈希映射和所有权

对于实现了 `Copy` trait 的类型，如 `i32`，值会被复制到哈希映射中。对于像 `String` 这样的拥有所有权的值，值将被移动，哈希映射将成为这些值的所有者，如 Listing 8-22 所示。

<Listing number="8-22" caption="展示键和值一旦插入哈希映射后，它们的所有权将归哈希映射所有">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

在将 `field_name` 和 `field_value` 变量通过 `insert` 调用移动到哈希映射后，我们无法再使用这些变量。

如果我们将值的引用插入哈希映射，这些值不会被移动到哈希映射中。引用所指向的值在哈希映射有效期间必须保持有效。我们将在第 10 章的 [“使用生命周期验证引用”][validating-references-with-lifetimes]<!-- ignore --> 中详细讨论这些问题。

### 更新哈希映射

尽管键值对的数量是可增长的，但每个唯一的键一次只能有一个与之关联的值（但反之则不然：例如，Blue 队和 Yellow 队都可以在 `scores` 哈希映射中存储值 `10`）。

当你想要更改哈希映射中的数据时，你必须决定如何处理键已经有值的情况。你可以用新值替换旧值，完全忽略旧值。你可以保留旧值并忽略新值，只有在键 _没有_ 值时才添加新值。或者你可以将旧值和新值结合起来。让我们看看如何做到这些！

#### 覆盖一个值

如果我们向哈希映射中插入一个键和一个值，然后用不同的值插入相同的键，与该键关联的值将被替换。尽管 Listing 8-23 中的代码调用了两次 `insert`，哈希映射将只包含一个键值对，因为我们两次都插入了 Blue 队的键的值。

<Listing number="8-23" caption="替换与特定键关联的存储值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

这段代码将打印 `{"Blue": 25}`。原始值 `10` 已被覆盖。

<!-- 旧标题。不要删除，否则链接可能会失效。 -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### 仅在键不存在时插入键和值

通常，我们会检查哈希映射中是否已经存在某个键的值，然后采取以下操作：如果键存在于哈希映射中，现有值应保持不变；如果键不存在，则插入该键及其值。

哈希映射为此提供了一个特殊的 API，称为 `entry`，它将你要检查的键作为参数。`entry` 方法的返回值是一个名为 `Entry` 的枚举，它表示一个可能存在或不存在的值。假设我们想检查 Yellow 队的键是否有关联的值。如果没有，我们想插入值 `50`，Blue 队也是如此。使用 `entry` API，代码如 Listing 8-24 所示。

<Listing number="8-24" caption="使用 `entry` 方法仅在键没有值时插入">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

`Entry` 上的 `or_insert` 方法定义为：如果该键存在，则返回对应 `Entry` 键值的可变引用；如果不存在，则插入参数作为该键的新值，并返回新值的可变引用。这种技术比我们自己编写逻辑要简洁得多，而且与借用检查器的配合也更好。

运行 Listing 8-24 中的代码将打印 `{"Yellow": 50, "Blue": 10}`。第一次调用 `entry` 将插入 Yellow 队的键，值为 `50`，因为 Yellow 队还没有值。第二次调用 `entry` 不会改变哈希映射，因为 Blue 队已经有值 `10`。

#### 根据旧值更新值

哈希映射的另一个常见用例是查找键的值，然后根据旧值进行更新。例如，Listing 8-25 展示了计算一段文本中每个单词出现次数的代码。我们使用一个以单词为键的哈希映射，并递增值以跟踪我们看到该单词的次数。如果我们第一次看到一个单词，我们将首先插入值 `0`。

<Listing number="8-25" caption="使用存储单词和计数的哈希映射来计算单词的出现次数">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

这段代码将打印 `{"world": 2, "hello": 1, "wonderful": 1}`。你可能会看到相同的键值对以不同的顺序打印：回想一下 [“访问哈希映射中的值”][access]<!-- ignore -->，遍历哈希映射的顺序是任意的。

`split_whitespace` 方法返回一个迭代器，该迭代器遍历 `text` 中由空白分隔的子切片。`or_insert` 方法返回指定键值的可变引用（`&mut V`）。在这里，我们将该可变引用存储在 `count` 变量中，因此为了赋值给该值，我们必须首先使用星号（`*`）解引用 `count`。可变引用在 `for` 循环结束时超出范围，因此所有这些更改都是安全的，并且符合借用规则。

### 哈希函数

默认情况下，`HashMap` 使用一种名为 _SipHash_ 的哈希函数，它可以提供对涉及哈希表的拒绝服务（DoS）攻击的抵抗力[^siphash]<!-- ignore -->。这不是最快的哈希算法，但为了更好的安全性而牺牲性能是值得的。如果你分析代码并发现默认的哈希函数对你的用途来说太慢，你可以通过指定不同的哈希器来切换到另一个函数。_哈希器_ 是实现 `BuildHasher` trait 的类型。我们将在 [第 10 章][traits]<!-- ignore --> 中讨论 trait 以及如何实现它们。你不必从头开始实现自己的哈希器；[crates.io](https://crates.io/)<!-- ignore --> 上有其他 Rust 用户共享的库，提供了实现许多常见哈希算法的哈希器。

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## 总结

向量、字符串和哈希映射将在你需要存储、访问和修改数据时提供大量必要的功能。以下是一些你现在应该能够解决的练习：

1. 给定一个整数列表，使用向量并返回中位数（排序后位于中间位置的值）和众数（出现次数最多的值；哈希映射在这里会很有帮助）。
1. 将字符串转换为拉丁猪文字。每个单词的第一个辅音被移动到单词的末尾并加上 _ay_，所以 _first_ 变成 _irst-fay_。以元音开头的单词在末尾加上 _hay_（_apple_ 变成 _apple-hay_）。请记住 UTF-8 编码的细节！
1. 使用哈希映射和向量，创建一个文本界面，允许用户将员工姓名添加到公司的部门中；例如，“Add Sally to Engineering” 或 “Add Amir to Sales”。然后让用户按部门或按部门排序检索公司中所有人员的列表。

标准库 API 文档描述了向量、字符串和哈希映射的方法，这些方法对这些练习会很有帮助！

我们正在进入更复杂的程序，其中操作可能会失败，因此现在是讨论错误处理的最佳时机。我们接下来会讨论这个问题！

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html