# Account

## 如何创建账户

创建账户需要使用 System Program 的 createAccount 指令。Solana 运行时将授予账户所有者对其数据进行写入或转移 lamports 的访问权限。在创建账户时，我们需要预分配一定的存储空间（以字节为单位，称为 space），并确保有足够的 lamports 来支付租金。

```ts
import {
  SystemProgram,
  Keypair,
  Transaction,
  sendAndConfirmTransaction,
  Connection,
  clusterApiUrl,
  LAMPORTS_PER_SOL,
} from "@solana/web3.js";

(async () => {
  const connection = new Connection(clusterApiUrl("devnet"), "confirmed");
  const fromPubkey = Keypair.generate();

  // Airdrop SOL for transferring lamports to the created account
  const airdropSignature = await connection.requestAirdrop(
    fromPubkey.publicKey,
    LAMPORTS_PER_SOL
  );
  await connection.confirmTransaction(airdropSignature);

  // amount of space to reserve for the account
  const space = 0;

  // Seed the created account with lamports for rent exemption
  const rentExemptionAmount =
    await connection.getMinimumBalanceForRentExemption(space);

  const newAccountPubkey = Keypair.generate();
  const createAccountParams = {
    fromPubkey: fromPubkey.publicKey,
    newAccountPubkey: newAccountPubkey.publicKey,
    lamports: rentExemptionAmount,
    space,
    programId: SystemProgram.programId,
  };

  const createAccountTransaction = new Transaction().add(
    SystemProgram.createAccount(createAccountParams)
  );

  await sendAndConfirmTransaction(connection, createAccountTransaction, [
    fromPubkey,
    newAccountPubkey,
  ]);
})();
```

## 如何计算账户创建成本

在 Solana 上保持账户的活跃性会产生一种称为租金的存储成本。为了进行计算，您需要考虑您打算存储在账户中的数据量。如果关闭账户，可以完全收回租金。

1. **理解租金和数据存储**：

   - 在 Solana 上，租金是存储账户数据所需的成本。
   - 租金的金额取决于账户中存储的数据量（以字节为单位）。

2. **租金计算**：

   - 租金按每个 epoch（Solana 大约每两天更新一次）的每字节收取。
   - 计算租金的公式如下：

     ```txt
     租金 = 租金豁免额 + （数据大小（字节） \* 租金率）
     ```

     - **租金豁免额**：必须支付的最低租金金额（固定的 lamports）。
     - **数据大小（字节）**：存储在账户中的数据量。
     - **租金率**：每个字节的租金费率。

```ts
import { Connection, clusterApiUrl } from "@solana/web3.js";

(async () => {
  const connection = new Connection(clusterApiUrl("devnet"), "confirmed");

  // length of data in bytes in the account to calculate rent for
  const dataLength = 1500;
  const rentExemptionAmount =
    await connection.getMinimumBalanceForRentExemption(dataLength);
  console.log({
    rentExemptionAmount,
  });
})();
```

## 如何创建 PDA（Program Derived Address）的账户

PDA 账户只能在链上创建，其地址具有一个相关联的非曲线公钥，但没有密钥。

### 生成 PDA

generate-pda.ts

```typescript
import { PublicKey } from "@solana/web3.js";

const programId = new PublicKey("G1DCNUQTSGHehwdLCAmRyAG8hf51eCHrLNUqkgGKYASj");

let [pda, bump] = PublicKey.findProgramAddressSync(
  [Buffer.from("test")],
  programId
);
console.log(`bump: ${bump}, pubkey: ${pda.toBase58()}`);
// 您将发现结果与 `createProgramAddress` 不同。
// 这是预期的，因为我们用于计算的真实seed是 ["test" + bump]
```

### 创建 PDA 账户

create-pda.rs

```rust
use solana_program::{
    account_info::next_account_info, account_info::AccountInfo, entrypoint,
    entrypoint::ProgramResult, program::invoke_signed, pubkey::Pubkey, system_instruction, sysvar::{rent::Rent, Sysvar}
};

entrypoint!(process_instruction);

fn process_instruction(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
    let account_info_iter = &mut accounts.iter();

    let payer_account_info = next_account_info(account_info_iter)?;
    let pda_account_info = next_account_info(account_info_iter)?;
    let rent_sysvar_account_info = &Rent::from_account_info(next_account_info(account_info_iter)?)?;

    // find space and minimum rent required for account
    let space = instruction_data[0];
    let bump = instruction_data[1];
    let rent_lamports = rent_sysvar_account_info.minimum_balance(space.into());

    invoke_signed(
        &system_instruction::create_account(
            &payer_account_info.key,
            &pda_account_info.key,
            rent_lamports,
            space.into(),
            program_id
        ),
        &[
            payer_account_info.clone(),
            pda_account_info.clone()
        ],
        &[&[&payer_account_info.key.as_ref(), &[bump]]]
    )?;

    Ok(())
}
```

### Client

create-pda.ts

```typescript
import {
  clusterApiUrl,
  Connection,
  Keypair,
  Transaction,
  SystemProgram,
  PublicKey,
  TransactionInstruction,
  LAMPORTS_PER_SOL,
  SYSVAR_RENT_PUBKEY,
} from "@solana/web3.js";

(async () => {
  // program id
  const programId = new PublicKey(
    "7ZP42kRwUQ2zgbqXoaXzAFaiQnDyp6swNktTSv8mNQGN"
  );

  // connection
  const connection = new Connection(clusterApiUrl("devnet"), "confirmed");

  // setup fee payer
  const feePayer = Keypair.generate();
  const feePayerAirdropSignature = await connection.requestAirdrop(
    feePayer.publicKey,
    LAMPORTS_PER_SOL
  );
  await connection.confirmTransaction(feePayerAirdropSignature);

  // setup pda
  let [pda, bump] = await PublicKey.findProgramAddress(
    [feePayer.publicKey.toBuffer()],
    programId
  );
  console.log(`bump: ${bump}, pubkey: ${pda.toBase58()}`);

  const data_size = 0;

  let tx = new Transaction().add(
    new TransactionInstruction({
      keys: [
        {
          pubkey: feePayer.publicKey,
          isSigner: true,
          isWritable: true,
        },
        {
          pubkey: pda,
          isSigner: false,
          isWritable: true,
        },
        {
          pubkey: SYSVAR_RENT_PUBKEY,
          isSigner: false,
          isWritable: false,
        },
        {
          pubkey: SystemProgram.programId,
          isSigner: false,
          isWritable: false,
        },
      ],
      data: Buffer.from(new Uint8Array([data_size, bump])),
      programId: programId,
    })
  );

  console.log(`txhash: ${await connection.sendTransaction(tx, [feePayer])}`);
})();
```

这些示例展示了如何在 Solana 上创建 PDA 账户，包括使用 Rust 编写的智能合约程序和使用 TypeScript 编写的客户端代码。 PDA 账户的地址由相关的程序衍生，但不具有私钥，只能通过特定的程序和种子生成。

## 如何使用 PDA 账户进行签名

程序衍生地址（PDA）可用于拥有由程序拥有并能够签名的账户。如果您希望程序拥有一个代币账户，并希望程序能够将代币从一个账户转移到另一个账户，则这非常有用。

sign-with-pda.rs

```rust
use solana_program::{
    account_info::next_account_info, account_info::AccountInfo, entrypoint,
    entrypoint::ProgramResult, program::invoke_signed, pubkey::Pubkey, system_instruction,
};

entrypoint!(process_instruction);

fn process_instruction(
    _program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
    let account_info_iter = &mut accounts.iter();

    let pda_account_info = next_account_info(account_info_iter)?;
    let to_account_info = next_account_info(account_info_iter)?;
    let system_program_account_info = next_account_info(account_info_iter)?;

    // pass bump seed for saving compute budget
    let bump_seed = instruction_data[0];

    invoke_signed(
        &system_instruction::transfer(
            &pda_account_info.key,
            &to_account_info.key,
            100_000_000, // 0.1 SOL
        ),
        &[
            pda_account_info.clone(),
            to_account_info.clone(),
            system_program_account_info.clone(),
        ],
        &[&[b"escrow", &[bump_seed]]],
    )?;

    Ok(())
}
```

这段代码展示了如何在 Solana 上使用 PDA 账户进行签名，其中 PDA 账户用于执行系统指令（如转账）而无需实际的私钥。

## 如何关闭账户

关闭账户可以让您收回用于打开账户的 SOL，但需要在关闭账户时删除所有账户中的信息。当关闭账户时，请确保在同一条指令中将数据清零，以防止在同一交易中重新打开账户并访问数据。这是因为账户直到交易完成之前并未真正关闭。

close-account.rs

```rust
use solana_program::{
    account_info::next_account_info, account_info::AccountInfo, entrypoint,
    entrypoint::ProgramResult, pubkey::Pubkey,
};

entrypoint!(process_instruction);

fn process_instruction(
    _program_id: &Pubkey,
    accounts: &[AccountInfo],
    _instruction_data: &[u8],
) -> ProgramResult {
    let account_info_iter = &mut accounts.iter();

    let source_account_info = next_account_info(account_info_iter)?;
    let dest_account_info = next_account_info(account_info_iter)?;

    let dest_starting_lamports = dest_account_info.lamports();
    **dest_account_info.lamports.borrow_mut() = dest_starting_lamports
        .checked_add(source_account_info.lamports())
        .unwrap();
    **source_account_info.lamports.borrow_mut() = 0;

    let mut source_data = source_account_info.data.borrow_mut();
    source_data.fill(0);

    Ok(())
}
```

这段代码展示了如何在 Solana 上关闭账户。关闭账户会将账户中的所有 LAMPORTS 转移给目标账户，并清空源账户中的数据。

## 如何获取账户余额

get-account-balance.ts

```ts
import {
  clusterApiUrl,
  Connection,
  PublicKey,
  LAMPORTS_PER_SOL,
} from "@solana/web3.js";

(async () => {
  const connection = new Connection(clusterApiUrl("devnet"), "confirmed");

  let wallet = new PublicKey("G2FAbFQPFa5qKXCetoFZQEvF9BVvCKbvUZvodpVidnoY");
  console.log(
    `${(await connection.getBalance(wallet)) / LAMPORTS_PER_SOL} SOL`
  );
})();
```
