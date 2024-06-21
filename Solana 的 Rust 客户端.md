# Solana 的 Rust 客户端

Solana 的 Rust 包已经发布到 crates.io，并且可以在 docs.rs 上以 solana- 前缀找到。

## HELLO WORLD：快速开始 Solana 开发

要快速开始 Solana 开发并构建您的第一个 Rust 程序，请查看以下详细的快速入门指南：

- 使用浏览器构建和部署您的第一个 Solana 程序。无需安装。
- 设置本地环境并使用本地测试验证器。

## Rust 包

以下是 Solana 开发中最重要和常用的 Rust 包：

- **solana-program** — 在 Solana 上运行的程序导入的包，编译为 SBF。该包包含许多基本数据类型，并且从 solana-sdk 重新导出，不能从 Solana 程序中导入。
- **solana-sdk** — 基本的离线 SDK，它重新导出 solana-program，并在其上添加更多 API。大多数不在链上运行的 Solana 程序将导入此包。
- **solana-client** — 通过 JSON RPC API 与 Solana 节点进行交互。
- **solana-cli-config** — 加载和保存 Solana CLI 配置文件。
- **solana-clap-utils** — 使用 clap 设置 CLI 的例程，主要由 Solana CLI 使用。包括用于加载 CLI 支持的所有类型签名者的函数。

以上是 Solana Rust 客户端的基本信息。这些工具和库使开发人员能够轻松构建、部署和与 Solana 区块链进行交互的应用程序。
