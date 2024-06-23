# transactions

## 如何发送 SOL

要发送 SOL，您需要与 SystemProgram 进行交互。

send-sol.ts

```typescript
import {
  Connection,
  Keypair,
  SystemProgram,
  LAMPORTS_PER_SOL,
  Transaction,
  sendAndConfirmTransaction,
} from "@solana/web3.js";

(async () => {
  const fromKeypair = Keypair.generate(); // 生成发送方的密钥对
  const toKeypair = Keypair.generate(); // 生成接收方的密钥对

  const connection = new Connection(
    "https://api.devnet.solana.com", // Solana 的开发网络 RPC 地址
    "confirmed" // 确认级别，可以是 'confirmed' 或 'processed'
  );

  const airdropSignature = await connection.requestAirdrop(
    fromKeypair.publicKey, // 发送方的公钥
    LAMPORTS_PER_SOL // 要请求的 SOL 数量，这里是 1 SOL
  );

  await connection.confirmTransaction(airdropSignature); // 等待空投交易确认

  const lamportsToSend = 1_000_000; // 要发送的 lamports 数量，这里是 1,000,000 lamports (即 0.001 SOL)

  const transferTransaction = new Transaction().add(
    SystemProgram.transfer({
      fromPubkey: fromKeypair.publicKey, // 发送方的公钥
      toPubkey: toKeypair.publicKey, // 接收方的公钥
      lamports: lamportsToSend, // 要发送的 lamports 数量
    })
  );

  await sendAndConfirmTransaction(connection, transferTransaction, [
    fromKeypair, // 添加发送方的签名密钥对到交易中
  ]);
})();
```

## 如何发送代币

使用 Token 程序来转移 SPL 代币。为了发送 SPL 代币，您需要知道其 SPL 代币账户地址。您可以使用以下示例获取地址并发送代币。

send-tokens.ts

```typescript
import {
  Connection,
  clusterApiUrl,
  Keypair,
  LAMPORTS_PER_SOL,
  Transaction,
  sendAndConfirmTransaction,
} from "@solana/web3.js";
import {
  createMint,
  getOrCreateAssociatedTokenAccount,
  mintTo,
  createTransferInstruction,
} from "@solana/spl-token";

(async () => {
  // 连接到集群
  const connection = new Connection(clusterApiUrl("devnet"), "confirmed");

  // 生成一个新的钱包密钥对并空投 SOL
  const fromWallet = Keypair.generate();
  const fromAirdropSignature = await connection.requestAirdrop(
    fromWallet.publicKey,
    LAMPORTS_PER_SOL
  );
  // 等待空投确认
  await connection.confirmTransaction(fromAirdropSignature);

  // 生成一个新的钱包密钥对以接收新铸造的代币
  const toWallet = Keypair.generate();

  // 创建新的代币铸造
  const mint = await createMint(
    connection,
    fromWallet,
    fromWallet.publicKey,
    null,
    9
  );

  // 获取或创建 fromWallet Solana 地址的代币账户
  const fromTokenAccount = await getOrCreateAssociatedTokenAccount(
    connection,
    fromWallet,
    mint,
    fromWallet.publicKey
  );

  // 获取或创建 toWallet Solana 地址的代币账户
  const toTokenAccount = await getOrCreateAssociatedTokenAccount(
    connection,
    fromWallet,
    mint,
    toWallet.publicKey
  );

  // 铸造 1 个新的代币到刚刚返回/创建的 fromTokenAccount 账户
  await mintTo(
    connection,
    fromWallet,
    mint,
    fromTokenAccount.address,
    fromWallet.publicKey,
    1000000000, // 这里是 1 个代币，单位是 lamports
    []
  );

  // 添加代币转移指令到交易中
  const transaction = new Transaction().add(
    createTransferInstruction(
      fromTokenAccount.address,
      toTokenAccount.address,
      fromWallet.publicKey,
      1
    )
  );

  // 签名交易，广播并确认
  await sendAndConfirmTransaction(connection, transaction, [fromWallet]);
})();
```

这段代码演示了如何使用 Solana 的 `@solana/spl-token` 库来发送 SPL 代币。它包括从 Solana 网络请求空投、创建代币铸造、获取或创建代币账户，以及最终发送代币的过程。

## 计算交易成本的方法

交易所需的签名数量用于计算交易的成本。除非您正在创建账户，否则这将是基本的交易成本。要了解更多有关创建账户的成本，请查看计算租金成本。

calculate-cost.ts

```typescript
import {
  clusterApiUrl,
  Connection,
  Keypair,
  Message,
  SystemProgram,
  SYSTEM_INSTRUCTION_LAYOUTS,
  Transaction,
} from "@solana/web3.js";
import bs58 from "bs58";

(async () => {
  // 连接到集群
  const connection = new Connection(clusterApiUrl("devnet"), "confirmed");

  // 创建一个付款方和一个接收方的密钥对
  const payer = Keypair.generate();
  const recipient = Keypair.generate();

  // 确定使用的指令类型为 Transfer
  const type = SYSTEM_INSTRUCTION_LAYOUTS.Transfer;
  const data = Buffer.alloc(type.layout.span);
  const layoutFields = Object.assign({ instruction: type.index });
  type.layout.encode(layoutFields, data);

  // 获取最近的区块哈希
  const recentBlockhash = await connection.getLatestBlockhash();

  // 构建消息参数
  const messageParams = {
    accountKeys: [
      payer.publicKey.toString(),
      recipient.publicKey.toString(),
      SystemProgram.programId.toString(),
    ],
    header: {
      numReadonlySignedAccounts: 0,
      numReadonlyUnsignedAccounts: 1,
      numRequiredSignatures: 1,
    },
    instructions: [
      {
        accounts: [0, 1], // 0 和 1 分别对应 payer 和 recipient 的账户索引
        data: bs58.encode(data),
        programIdIndex: 2,
      },
    ],
    recentBlockhash: recentBlockhash.blockhash,
  };

  // 创建消息
  const message = new Message(messageParams);

  // 获取消息的交易费用
  const fees = await connection.getFeeForMessage(message);
  console.log(`估算的 SOL 转账成本: ${fees.value} lamports`);
  // 估算的 SOL 转账成本: 5000 lamports
})();
```

这段代码演示了如何使用 Solana 的 `@solana/web3.js` 库来计算一个基本的交易成本。它包括连接到 Solana 的开发网络，生成付款方和接收方的密钥对，构建交易消息并获取该消息的交易费用。

## 如何向交易添加备注

任何交易都可以利用备忘录程序添加一条消息。目前需要手动添加备忘录程序的程序 ID：MemoSq4gqABAXKb96qnH8TysNcWxMyWCqXgDLGmfcHr。

```ts
import {
  Connection,
  Keypair,
  SystemProgram,
  LAMPORTS_PER_SOL,
  PublicKey,
  Transaction,
  TransactionInstruction,
  sendAndConfirmTransaction,
} from "@solana/web3.js";

(async () => {
  // 生成付款方和接收方的密钥对
  const fromKeypair = Keypair.generate();
  const toKeypair = Keypair.generate();

  // 连接到 Solana 的开发网络
  const connection = new Connection(
    "https://api.devnet.solana.com",
    "confirmed"
  );

  // 为付款方请求空投以支付交易费用
  const airdropSignature = await connection.requestAirdrop(
    fromKeypair.publicKey,
    LAMPORTS_PER_SOL
  );

  // 等待空投交易确认
  await connection.confirmTransaction(airdropSignature);

  // 设置要发送的 lamports
  const lamportsToSend = 10;

  // 创建一个交易对象，并添加转账指令
  const transferTransaction = new Transaction().add(
    SystemProgram.transfer({
      fromPubkey: fromKeypair.publicKey,
      toPubkey: toKeypair.publicKey,
      lamports: lamportsToSend,
    })
  );

  // 添加一个 TransactionInstruction 来包含备注信息
  transferTransaction.add(
    new TransactionInstruction({
      keys: [
        { pubkey: fromKeypair.publicKey, isSigner: true, isWritable: true },
      ],
      data: Buffer.from("Memo message to send in this transaction", "utf-8"),
      programId: new PublicKey("MemoSq4gqABAXKb96qnH8TysNcWxMyWCqXgDLGmfcHr"),
    })
  );

  // 发送并确认交易
  await sendAndConfirmTransaction(connection, transferTransaction, [
    fromKeypair,
  ]);
})();
```

## 如何向交易添加优先费用

交易（TX）的优先级可以通过支付优先费用来实现，此外还需支付基本费用。默认情况下，计算预算是每个指令的 200,000 个计算单元（CU）的乘积，最大为 1.4M CU。基本费用是每个签名的 5,000 Lamports。一个微 Lamport 等于 0.000001 Lamports。

**信息**：您可以在这里找到有关如何使用优先费用的详细指南。

单个 TX 的总计算预算或优先费用可以通过添加 ComputeBudgetProgram 的指令来更改。

ComputeBudgetProgram.setComputeUnitPrice({ microLamports: number }) 将在基本费用（5,000 Lamports）之上添加一个优先费用。microLamports 中提供的值将乘以 CU 预算，以确定以 Lamports 计的优先费用。例如，如果您的 CU 预算为 1M CU，并且添加了 1 个微 Lamport/CU，那么优先费用将为 1 Lamport（1M \* 0.000001）。总费用将为 5001 Lamports。

使用 ComputeBudgetProgram.setComputeUnitLimit({ units: number }) 来设置新的计算预算。提供的值将替换默认值。交易应请求执行所需的最小 CU 量，以最大化吞吐量或最小化费用。

```ts
import { BN } from "@coral-xyz/anchor";
import {
  Keypair,
  Connection,
  LAMPORTS_PER_SOL,
  sendAndConfirmTransaction,
  ComputeBudgetProgram,
  SystemProgram,
  Transaction,
} from "@solana/web3.js";

(async () => {
  const payer = Keypair.generate();
  const toAccount = Keypair.generate().publicKey;

  const connection = new Connection("http://127.0.0.1:8899", "confirmed");

  const airdropSignature = await connection.requestAirdrop(
    payer.publicKey,
    LAMPORTS_PER_SOL
  );

  await connection.confirmTransaction(airdropSignature);

  // request a specific compute unit budget
  const modifyComputeUnits = ComputeBudgetProgram.setComputeUnitLimit({
    units: 1000000,
  });

  // set the desired priority fee
  const addPriorityFee = ComputeBudgetProgram.setComputeUnitPrice({
    microLamports: 1,
  });

  // Total fee will be 5,001 Lamports for 1M CU
  const transaction = new Transaction()
    .add(modifyComputeUnits)
    .add(addPriorityFee)
    .add(
      SystemProgram.transfer({
        fromPubkey: payer.publicKey,
        toPubkey: toAccount,
        lamports: 10000000,
      })
    );

  const signature = await sendAndConfirmTransaction(connection, transaction, [
    payer,
  ]);
  console.log(signature);

  const result = await connection.getParsedTransaction(signature);
  console.log(result);
})();
```

## 如何优化计算请求

优化交易中请求的计算量对于确保交易及时处理并避免支付过多的优先费用至关重要。

欲了解更多有关请求最佳计算量的信息，请参阅完整指南。您也可以在这份详细指南中找到有关使用优先费用的更多信息。

```ts
// import { ... } from "@solana/web3.js"

async function buildOptimalTransaction(
  connection: Connection,
  instructions: Array<TransactionInstruction>,
  signer: Signer,
  lookupTables: Array<AddressLookupTableAccount>
) {
  const [microLamports, units, recentBlockhash] = await Promise.all([
    100 /* Get optimal priority fees - https://solana.com/developers/guides/advanced/how-to-use-priority-fees*/,
    getSimulationComputeUnits(
      connection,
      instructions,
      signer.publicKey,
      lookupTables
    ),
    connection.getLatestBlockhash(),
  ]);

  instructions.unshift(
    ComputeBudgetProgram.setComputeUnitPrice({ microLamports })
  );
  if (units) {
    // probably should add some margin of error to units
    instructions.unshift(ComputeBudgetProgram.setComputeUnitLimit({ units }));
  }
  return {
    transaction: new VersionedTransaction(
      new TransactionMessage({
        instructions,
        recentBlockhash: recentBlockhash.blockhash,
        payerKey: signer.publicKey,
      }).compileToV0Message(lookupTables)
    ),
    recentBlockhash,
  };
}
```
