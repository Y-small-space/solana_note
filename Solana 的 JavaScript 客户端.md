# Solana 的 JavaScript 客户端

## 什么是 Solana-Web3.js？

Solana-Web3.js 库旨在全面覆盖 Solana 区块链的功能。该库是建立在 Solana JSON RPC API 之上的，提供了与 Solana 区块链交互所需的工具和实用程序。

您可以在这里找到 `@solana/web3.js` 库的完整文档。

## 常见术语

| 术语 | 定义                                                                           |
| ---- | ------------------------------------------------------------------------------ |
| 程序 | 用于解释指令的无状态可执行代码。                                               |
| 指令 | 客户端可以包含在事务中的程序的最小单位。一个指令可能包含一个或多个跨程序调用。 |
| 事务 | 由客户端签名的一个或多个指令，原子性地执行，只有两种可能的结果：成功或失败。   |

您可以在 Solana 术语页面查看完整术语列表。

## 入门

### 安装

使用 Yarn 安装：

```bash
yarn add @solana/web3.js
```

使用 npm 安装：

```bash
npm install --save @solana/web3.js
```

### 打包

```html
<script src="https://unpkg.com/@solana/web3.js@latest/lib/index.iife.js"></script>

<script src="https://unpkg.com/@solana/web3.js@latest/lib/index.iife.min.js"></script>
```

### 使用

#### JavaScript

使用 CommonJS 导入：

```javascript
const solanaWeb3 = require("@solana/web3.js");
console.log(solanaWeb3);
```

使用 ES6 导入：

```javascript
import * as solanaWeb3 from "@solana/web3.js";
console.log(solanaWeb3);
```

浏览器环境下使用打包：

```javascript
// solanaWeb3 在全局命名空间中由打包脚本提供
console.log(solanaWeb3);
```

## 快速入门

### 连接到钱包

为了让用户在 Solana 上使用您的 dApp 或应用程序，他们需要获取他们的 Keypair。Keypair 是一个带有匹配的公钥的私钥，用于签署事务。

有两种方法可以获取 Keypair：

1. 生成一个新的 Keypair

   ```javascript
   const { Keypair } = require("@solana/web3.js");

   let keypair = Keypair.generate();
   ```

2. 使用秘钥密钥获取 Keypair

   ```javascript
   const { Keypair } = require("@solana/web3.js");

   let secretKey = Uint8Array.from([
     202, 171, 192, 129, 150, 189, 204, 241, 142, 71, 205, 2, 81, 97, 2, 176,
     48, 81, 45, 1, 96, 138, 220, 132, 231, 131, 120, 77, 66, 40, 97, 172, 91,
     245, 84, 221, 157, 190, 9, 145, 176, 130, 25, 43, 72, 107, 190, 229, 75,
     88, 191, 136, 7, 167, 109, 91, 170, 164, 186, 15, 142, 36, 12, 23,
   ]);

   let keypair = Keypair.fromSecretKey(secretKey);
   ```

现在，用户可以使用上述任一方法获取一个新的 Keypair，并将其用于应用程序。

### 创建和发送事务

为了与 Solana 上的程序交互，您需要创建、签署和发送事务到网络。事务是由指令和签名组成的集合。事务中指令的顺序决定了它们执行的顺序。

例如，创建一个转账事务：

```javascript
const {
  Keypair,
  Transaction,
  SystemProgram,
  LAMPORTS_PER_SOL,
} = require("@solana/web3.js");

let fromKeypair = Keypair.generate();
let toKeypair = Keypair.generate();
let transaction = new Transaction();

transaction.add(
  SystemProgram.transfer({
    fromPubkey: fromKeypair.publicKey,
    toPubkey: toKeypair.publicKey,
    lamports: LAMPORTS_PER_SOL,
  })
);
```

现在，您创建了一个准备签名和广播到网络的事务。这个事务中添加了一个 `SystemProgram.transfer` 的指令，包含要发送的 lamports 数量，以及发送和接收方的公钥。

接下来，使用 Keypair 签署事务并将其发送到网络：

```javascript
const {
  sendAndConfirmTransaction,
  clusterApiUrl,
  Connection,
} = require("@solana/web3.js");

let keypair = Keypair.generate();
let connection = new Connection(clusterApiUrl("testnet"));

sendAndConfirmTransaction(connection, transaction, [keypair]);
```

上面的代码通过 SystemProgram 发送一个基本的转账事务。

### 与自定义程序交互

在 Solana 上，所有操作都与不同的程序交互，包括前面部分的转账事务。目前，Solana 上的程序通常是用 Rust 或 C 编写的。

让我们看看 SystemProgram 的例子。在 Rust 中，为您的帐户分配空间的方法签名如下：

```rust
pub fn allocate(
    pubkey: &Pubkey,
    space: u64
) -> Instruction
```

在 Solana 中，当您想要与程序交互时，必须首先知道您将与之交互的所有帐户。

在指令中，必须始终提供程序将在其中运行的每个帐户。不仅如此，您还必须提供帐户是 isSigner 还是 isWritable。

在上面的 allocate 方法中，需要一个 pubkey 帐户和一个分配空间的量。我们知道在内的 allocate 函数内部，这表明 pubkey 必须是 isWritable 的。当你指定运行指令的帐户时，isSigner 是必需的。在这种情况下，签署者是调用自己内部空间的帐户。

让我们看看如何使用 `solana-web3.js` 调用此指令：

```javascript
const {
  Keypair,
  SystemProgram,
  TransactionInstruction,
} = require("@solana/web3.js");
const { Buffer } = require("buffer");

let keypair = Keypair.generate();
let payer = Keypair.generate();
let connection = new Connection(clusterApiUrl("testnet"));

let airdropSignature = await connection.requestAirdrop(
  payer.publicKey,
  LAMPORTS_PER_SOL
);

await connection.confirmTransaction({ signature: airdropSignature });

let allocateTransaction = new Transaction({
  feePayer: payer.publicKey,
});
let keys = [{ pubkey: keypair.publicKey, isSigner: true, isWritable: true }];
let params = { space: 100 };

let allocateStruct = {
  index: 8,
  layout: struct([u32("instruction"), ns64("space")]),
};

let data = Buffer.alloc(allocateStruct.layout.span);
let layoutFields = Object.assign({ instruction: allocateStruct.index }, params);
allocateStruct.layout.encode(layoutFields, data);

allocateTransaction.add(
  new TransactionInstruction({
    keys,
    programId: SystemProgram.programId,
    data,
  })
);

await sendAndConfirmTransaction(connection, allocateTransaction, [
  payer,
  keypair,
]);
```

上面的代码展示了如何在 Solana 上使用 `solana-web3.js` 创建和发送一个事务来调用 allocate 函数。

## 结语

`@solana/web3.js` 提供了与 Solana 区块链交互的便捷工具和实用程序，使开发者能够更轻松地开发 dApp 和应用程序，并与 Solana 上的智能合约进行交互。
