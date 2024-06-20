# Solana 开发快速入门指南（仅使用浏览器）

欢迎来到 Solana 开发快速入门指南！在本指南中，您将学习如何使用 Solana Playground（一款基于浏览器的 IDE）来创建、构建、部署和交互一个基本的 Solana 程序。

## 学习内容

- 如何开始使用 Solana Playground
- 如何在 Playground 上创建一个 Solana 钱包
- 如何使用 Rust 编写一个基本的 Solana 程序
- 如何构建并部署一个 Solana Rust 程序
- 如何使用 JavaScript 与您的链上程序交互

## 使用 Solana Playground

Solana Playground 是一款基于浏览器的应用程序，允许您在浏览器中编写、构建和部署 Solana 链上程序。无需安装任何软件。

它是开始 Solana 开发的绝佳资源，特别是对于 Windows 用户。

### 导入示例项目

在浏览器的新标签页中，打开我们在 Solana Playground 上的示例“Hello World”项目。

接下来，通过点击“导入”图标并命名您的项目为 hello_world，将项目导入到本地工作区。

### 创建 Playground 钱包

通常在本地开发中，您需要使用 Solana CLI 创建一个文件系统钱包。但在 Solana Playground 中，您只需点击几个按钮即可创建一个基于浏览器的钱包。

点击屏幕左下角的红色状态指示器按钮，选择是否保存钱包的密钥对文件到您的计算机进行备份，然后点击“继续”。

创建 Playground 钱包后，您会注意到窗口底部现在显示您的钱包地址、SOL 余额和连接的 Solana 集群（通常推荐使用 Devnet，但“localhost”测试验证器也可以接受）。

### 创建 Solana 程序

您的 Rust 基于 Solana 程序的代码将存放在 src/lib.rs 文件中。在 src/lib.rs 文件中，您可以导入 Rust 包并定义逻辑。打开 Solana Playground 中的 src/lib.rs 文件。

### 导入 solana_program 包

在 lib.rs 文件的顶部，我们导入 solana_program 包并将所需项目引入本地命名空间：

```rust
use solana_program::{
    account_info::AccountInfo,
    entrypoint,
    entrypoint::ProgramResult,
    pubkey::Pubkey,
    msg,
};
```

### 编写程序逻辑

每个 Solana 程序必须定义一个入口点，告诉 Solana 运行时从哪里开始执行链上代码。您的程序入口点应提供一个名为 process_instruction 的公共函数：

```rust
// 声明并导出程序的入口点
entrypoint!(process_instruction);

// 程序入口点的实现
pub fn process_instruction(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8]
) -> ProgramResult {
    // 向区块链日志记录一条消息
    msg!("Hello, world!");

    // 程序正常退出
    Ok(())
}
```

每个链上程序应返回 Ok 结果枚举，并带有()值。这告诉 Solana 运行时您的程序成功执行，没有错误。

上述程序将向区块链集群记录一条“Hello, world!”消息，然后以 Ok(())优雅地退出。

### 构建程序

在左侧边栏中，选择“Build & Deploy”选项卡。接下来，点击“Build”按钮。

如果查看 Playground 的终端，您应该会看到您的 Solana 程序开始编译。完成后，您将看到成功消息。

### 部署程序

您可以点击“Deploy”按钮，将您的第一个程序部署到 Solana 区块链，具体到您选择的集群（例如 Devnet、Testnet 等）。

每次部署后，您会看到 Playground 钱包余额发生变化。默认情况下，Solana Playground 会自动请求 SOL 空投，以确保您的钱包有足够的 SOL 来支付部署费用。

### 查找程序 ID

在使用 web3.js 或其他 Solana 程序执行程序时，您需要提供程序 ID（即程序的公共地址）。

在 Solana Playground 的 Build & Deploy 侧边栏中，您可以在 Program Credentials 下找到您的程序 ID。

## 恭喜

您已成功使用 Rust 语言在浏览器中设置、构建和部署了一个 Solana 程序。接下来，我们将演示如何与您的链上程序交互。

### 与链上程序交互

成功部署 Solana 程序后，您将希望能够与该程序交互。

像大多数开发人员创建 dApps 和网站一样，我们将使用 JavaScript 与我们的链上程序交互。具体来说，我们将使用开源 NPM 包@solana/web3.js 来帮助我们的客户端应用程序。

### 初始化客户端

我们将使用 Solana Playground 进行客户端生成。在 Playground 终端中运行以下命令以创建客户端文件夹：

```shell
run
```

我们已创建 client 文件夹和一个默认的 client.ts 文件。接下来，我们将在其中完成 hello world 程序的剩余工作。

### Playground 全局变量

在 Playground 中，有许多可供我们使用的全局工具，无需安装或设置。对于我们的 hello world 程序，最重要的是 web3（用于@solana/web3.js）和 pg（用于 Solana Playground 工具）。

### 调用程序

要执行您的链上程序，您必须向其发送交易。每个提交到 Solana 区块链的交易都包含一个指令列表（以及指令将交互的程序）。

在这里，我们创建一个新的交易并向其添加一个指令：

```javascript
// 创建一个空交易
const transaction = new web3.Transaction();

// 向交易中添加一个hello world程序指令
transaction.add(
  new web3.TransactionInstruction({
    keys: [],
    programId: new web3.PublicKey(pg.PROGRAM_ID),
  })
);
```

每个指令必须包含操作中涉及的所有密钥和我们要执行的程序 ID。在这个例子中，keys 是空的，因为我们的程序只记录 hello world 消息，不需要任何账户。

创建交易后，我们可以将其提交到集群：

```javascript
// 将交易发送到Solana集群
console.log("Sending transaction...");
const txHash = await web3.sendAndConfirmTransaction(
  pg.connection,
  transaction,
  [pg.wallet.keypair]
);
console.log("Transaction sent with hash:", txHash);
```

### 运行应用程序

编写客户端应用程序后，您可以通过相同的 run 命令运行代码。

应用程序完成后，您将看到类似如下的输出：

```shell
Running client...
  client.ts:
    My address: GkxZRRNPfaUfL9XdYVfKF3rWjMcj5md6b6mpRoWpURwP
    My balance: 5.7254472 SOL
    Sending transaction...
    Transaction sent with hash: 2Ra7D9JoqeNsax9HmNq6MB4qWtKPGcLwoqQ27mPYsPFh3h8wignvKB2mWZVvdzCyTnp7CEZhfg2cEpbavib9mCcq
```

### 获取交易日志

我们将直接在 Playground 中使用 solana-cli 来获取交易信息：

```shell
solana confirm -v <TRANSACTION_HASH>
```

将<TRANSACTION_HASH>替换为您从调用 hello world 程序时收到的哈希值。

在输出的日志消息部分，您应该看到“Hello, world!”。🎉

## 恭喜

您现在已经为您的链上程序编写了一个客户端应用程序。您现在是一名 Solana 开发人员！

### 下一步

请参阅以下链接，了解更多关于编写 Solana 程序和设置本地开发环境的信息：

- 设置本地开发环境
- 编写 Solana 程序概述
- 了解更多使用 Rust 开发 Solana 程序
- 调试链上程序
