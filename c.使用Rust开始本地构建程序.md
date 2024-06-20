# Solana Rust 程序开发快速入门指南

本快速入门指南将演示如何快速设置、构建和部署您的第一个基于 Rust 的 Solana 程序到区块链上。

## 您将学到什么

- 如何在本地安装 Rust 语言
- 如何初始化一个新的 Solana Rust 程序
- 如何编写一个基本的 Solana Rust 程序
- 如何构建和部署您的 Rust 程序

## 安装 Rust 和 Cargo

要能够编译基于 Rust 的 Solana 程序，请使用 Rustup 安装 Rust 语言和 Cargo（Rust 的包管理器）：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## 运行本地验证器

Solana CLI 自带了测试验证器。此命令行工具允许您在本地计算机上运行一个完整的区块链集群。

```bash
solana-test-validator
```

**提示：** 请在一个新的/单独的终端窗口中运行 Solana 测试验证器，此命令行程序必须保持运行状态，以便您的本地验证器保持在线并准备执行操作。

配置 Solana CLI 以在所有未来的终端命令和 Solana 程序部署中使用您的本地验证器：

```bash
solana config set --url localhost
```

## 使用 Cargo 创建新的 Rust 库

Solana 程序以 Rust 库的形式编写，编译为 BPF 字节码并保存为.so 格式。

使用 Cargo 命令行初始化一个名为 hello_world 的新 Rust 库：

```bash
cargo init hello_world --lib
cd hello_world
```

向您的新 Rust 库添加 solana-program 依赖：

```bash
cargo add solana-program
```

**提示：** 强烈建议将 solana-program 和其他 Solana Rust 依赖项与您安装的 Solana CLI 版本保持一致。例如，如果您正在运行 Solana CLI 1.17.17，可以运行：

```bash
cargo add solana-program@"=1.17.17"
```

这样将确保您的 crate 仅使用 1.17.17 版本。如果遇到 Solana 依赖项的兼容性问题，请查看 Solana Stack Exchange。

打开 Cargo.toml 文件并添加这些必需的 Rust 库配置设置，根据需要更新您的项目名称：

```toml
[lib]
name = "hello_world"
crate-type = ["cdylib", "lib"]
```

## 创建您的第一个 Solana 程序

您的基于 Rust 的 Solana 程序的代码将存储在 src/lib.rs 文件中。在 src/lib.rs 中，您将能够导入 Rust crate 并定义您的逻辑。在您喜欢的编辑器中打开 src/lib.rs 文件。

在 lib.rs 的顶部，导入 solana-program crate 并将所需的项引入本地命名空间：

```rust
use solana_program::{
    account_info::AccountInfo,
    entrypoint,
    entrypoint::ProgramResult,
    pubkey::Pubkey,
    msg,
};

entrypoint!(process_instruction);

pub fn process_instruction(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8]
) -> ProgramResult {
    msg!("Hello, world!");
    Ok(())
}
```

每个 Solana 程序必须定义一个入口点，告诉 Solana 运行时从哪里开始执行您的链上代码。您的程序的入口点应提供一个名为 process_instruction 的公共函数。

上述程序将简单地向区块链集群记录一条消息"Hello, world!"，然后优雅地退出并返回 Ok(())。

## 构建您的 Rust 程序

在终端窗口中，您可以通过在项目的根目录下运行以下命令来构建您的 Solana Rust 程序：

```bash
cargo build-bpf
```

**信息：** 每次构建您的 Solana 程序后，上述命令将输出编译程序的.so 文件的构建路径以及将用于程序地址的默认密钥文件。cargo build-bpf 会安装来自当前安装的 solana CLI 工具的工具链。如果遇到任何版本不兼容问题，您可能需要升级这些工具。

## 部署您的 Solana 程序

使用 Solana CLI，您可以将您的程序部署到当前选择的集群：

```bash
solana program deploy ./target/deploy/hello_world.so
```

一旦您的 Solana 程序部署成功（并且交易完成），上述命令将输出您程序的公共地址（也称为"程序 ID"）。

**示例输出：**

```
Program Id: EFH95fWg49vkFNbAdw9vy75tM7sWZ2hQbTTUmuACGip3
```

## 祝贺您

您已成功设置、构建和部署了一款使用 Rust 语言编写的 Solana 程序。

## 检查您的钱包余额

部署后，请再次检查您的 Solana 钱包余额。看看部署这个简单程序花费了多少 SOL？

---

通过这些步骤，您的本地 Solana Rust 开发环境已经设置完成，现在可以开始开发和部署 Solana 程序到区块链上了！
