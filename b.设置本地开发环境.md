# Solana 开发环境快速入门指南

本指南将演示如何快速安装和设置本地开发环境，使您能够开始开发和部署 Solana 程序到区块链上。

## 您将学到什么

- 如何在本地安装 Solana CLI
- 如何设置本地 Solana 集群/验证器
- 如何为开发创建 Solana 钱包
- 如何向您的钱包空投 SOL 代币

## 1. 安装依赖项

### Windows 系统的依赖项

您可以在 Windows 上使用 WSL（Windows Subsystem for Linux）开始 Solana 开发。WSL 允许您轻松地在 Windows 上运行 Linux 软件，使用一个轻量级的虚拟机。

**设置 WSL 进行 Solana 开发**
首先，安装 WSL 到您的系统。安装完成后，请重新启动您的计算机，然后继续进行下面的步骤。

```bash
wsl --install
```

安装 WSL 并重新启动计算机后，使用 WSL 打开一个新的 Linux 终端会话：

```bash
wsl
```

对于接下来的 Solana 开发，您将在此 Linux 终端会话中运行所有命令、构建 Solana 程序和部署程序（除非本指南另有说明）。

如果您使用 VS Code 作为代码编辑器，请按照 VS Code 官方网站的教程，正确配置 VS Code 和 WSL，以获得最佳的开发体验。

### Linux 系统的依赖项

在您的 Linux 系统上安装以下依赖项：

```bash
sudo apt-get install -y \
    build-essential \
    pkg-config \
    libudev-dev llvm libclang-dev \
    protobuf-compiler libssl-dev
```

### macOS 系统的依赖项

在 macOS 上，可以通过 Xcode 命令行工具来获取构建工具。您可以直接从 Apple 下载 Xcode 命令行工具，根据需要进行安装。

您可以使用以下命令检查 Xcode CLI 是否安装：

```bash
xcode-select -p
```

如果没有返回路径，您需要安装 CLI 工具：

```bash
xcode-select --install
```

### Rust 工具链的安装

Rust 编程语言是一种多范式、通用目的的编程语言，强调性能、类型安全和并发性。我们将使用 rustup 来安装和管理 Rust 工具链。

在您的终端中运行以下命令以安装 Rust 工具链：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
```

安装完成后，请重新启动终端或运行以下命令手动刷新您的 PATH 设置：

```bash
source ~/.bashrc
```

### 安装 Solana CLI

Solana CLI 是进行本地开发的关键工具，它提供了所有必要的命令来执行常见任务，如创建和管理文件系统 Solana 钱包/密钥对、连接 Solana 集群、构建 Solana 程序以及部署程序到区块链上。

在 Linux、macOS、WSL 或其他类 Unix 系统上安装 Solana CLI：

```bash
sh -c "$(curl -sSfL https://release.solana.com/stable/install)"
```

安装完成后，可能需要更新 PATH 环境变量以包含 Solana 程序。请按照 Solana CLI 安装器的建议更新 PATH 环境变量。

验证安装是否成功，检查 Solana CLI 的版本：

```bash
solana --version
```

### 安装 Anchor for Solana

Anchor 是 Solana 运行时的一个框架，提供了几个方便的开发工具来编写链上程序。它使用 Rust 的宏抽象了许多安全检查和常见样板代码。

使用 cargo（Rust 包管理器）安装 avm，Anchor 版本管理器：

```bash
cargo install --git https://github.com/coral-xyz/anchor avm --locked --force
```

使用 avm 安装最新版本的 Anchor 框架：

```bash
avm install latest
avm use latest
```

安装完成后，通过检查已安装的版本来验证 Anchor 是否安装成功：

```bash
anchor --version
```

### 设置本地区块链集群

Solana CLI 自带了测试验证器。此命令行工具将允许您在本地计算机上运行一个完整的区块链集群：

```bash
solana-test-validator
```

**提示：** 请在一个新的/单独的终端窗口中运行 Solana 测试验证器，该命令行程序必须保持运行状态，以便您的本地集群在线并准备处理交易和请求（如部署程序）。

配置 Solana CLI 以使用您的本地验证器执行所有未来的终端命令：

```bash
solana config set --url localhost
```

您可以随时查看当前的 Solana CLI 配置设置：

```bash
solana config get
```

### 创建文件系统钱包

要使用 Solana CLI 部署程序，您需要一个 Solana 钱包，并且其中需要有 SOL 代币来支付区块链上的交易和数据存储成本。

让我们创建一个简单的文件系统钱包，用于本地开发：

```bash
solana-keygen new
```

默认情况下，`solana-keygen`命令会创建一个新的文件系统钱包，位于`~/.config/solana/id.json`。您可以使用`--outfile /path`选项手动指定输出文件的位置。

**注意：** 如果默认位置已经存在一个文件系统钱包，此命令不会覆盖它，除非您显式使用`--force`标志进行强制覆盖。

将您的新钱包设置为默认钱包：

```bash
solana config set -k ~/.config/solana/id.json
```

### 向您的钱包空投 SOL 代币

一旦您的新钱包设置为默认，您可以请求向其空投免费的 SOL 代币：

```bash
solana airdrop 2
```

**注意：** `solana airdrop`命令对每个集群（测试网或开发网）的每次空投请求有限制。如果您的空投交易失败，请减少空投请求的数量并重试。

您随时可以检查您当前钱包的 SOL 余额：

```bash
solana balance
```

这些步骤完成后，您的本地 Solana 开发环境已设置完毕，您可以开始开发和部署 Solana 程序到区块链上了！
