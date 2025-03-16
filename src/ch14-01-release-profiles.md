## 使用发布配置文件自定义构建

在 Rust 中，_发布配置文件_ 是预定义且可自定义的配置文件，它们具有不同的配置，允许程序员对编译代码的各种选项进行更多控制。每个配置文件都是独立配置的。

Cargo 有两个主要的配置文件：`dev` 配置文件，当您运行 `cargo build` 时使用；`release` 配置文件，当您运行 `cargo build --release` 时使用。`dev` 配置文件为开发环境提供了良好的默认设置，而 `release` 配置文件则为发布构建提供了良好的默认设置。

这些配置文件的名称可能从构建输出中看起来很熟悉：

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

`dev` 和 `release` 是编译器使用的不同配置文件。

Cargo 为每个配置文件提供了默认设置，当您在项目的 _Cargo.toml_ 文件中没有显式添加任何 `[profile.*]` 部分时，这些默认设置将生效。通过为您想要自定义的任何配置文件添加 `[profile.*]` 部分，您可以覆盖默认设置的任何子集。例如，以下是 `dev` 和 `release` 配置文件的 `opt-level` 设置的默认值：

<span class="filename">文件名: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` 设置控制 Rust 对代码应用的优化级别，范围从 0 到 3。应用更多的优化会延长编译时间，因此如果您在开发过程中经常编译代码，您可能希望减少优化以加快编译速度，即使生成的代码运行速度较慢。因此，`dev` 的默认 `opt-level` 为 `0`。当您准备发布代码时，最好花更多时间进行编译。您只需在发布模式下编译一次，但会多次运行编译后的程序，因此发布模式以更长的编译时间换取运行更快的代码。这就是为什么 `release` 配置文件的默认 `opt-level` 为 `3`。

您可以通过在 _Cargo.toml_ 中添加不同的值来覆盖默认设置。例如，如果我们希望在开发配置文件中使用优化级别 1，我们可以将以下两行添加到项目的 _Cargo.toml_ 文件中：

<span class="filename">文件名: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

这段代码覆盖了默认的 `0` 设置。现在当我们运行 `cargo build` 时，Cargo 将使用 `dev` 配置文件的默认设置以及我们对 `opt-level` 的自定义设置。因为我们已将 `opt-level` 设置为 `1`，Cargo 将应用比默认更多的优化，但不会像发布构建那样多。

有关每个配置文件的完整配置选项和默认值的列表，请参阅 [Cargo 的文档](https://doc.rust-lang.org/cargo/reference/profiles.html)。