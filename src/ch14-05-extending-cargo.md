## 使用自定义命令扩展 Cargo

Cargo 的设计允许你通过添加新的子命令来扩展它，而无需修改 Cargo 本身。如果你的 `$PATH` 中有一个名为 `cargo-something` 的二进制文件，你可以通过运行 `cargo something` 来像运行 Cargo 子命令一样运行它。这样的自定义命令也会在你运行 `cargo --list` 时列出。能够使用 `cargo install` 安装扩展并像内置的 Cargo 工具一样运行它们，是 Cargo 设计的一个非常方便的好处！

## 总结

通过 Cargo 和 [crates.io](https://crates.io/)<!-- ignore --> 共享代码是使 Rust 生态系统适用于许多不同任务的一部分。Rust 的标准库小而稳定，但 crate 易于共享、使用和改进，并且它们的时间线与语言的时间线不同。不要害羞，将对你来说有用的代码分享到 [crates.io](https://crates.io/)<!-- ignore --> 上；很可能它对其他人也有用！