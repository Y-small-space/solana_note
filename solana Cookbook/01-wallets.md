# wallets

## 如何创建密钥对

在索拉纳区块链上进行任何交易都需要一个密钥对或钱包。如果你连接的是一个钱包，则不需要担心密钥对。否则，必须生成一个密钥对来签署交易。

```javascript
import { Keypair } from "@solana/web3.js";

const keypair = Keypair.generate();
```

## 如何恢复密钥对

如果你已经拥有你的密钥或字节，可以从密钥中恢复你的密钥对来测试你的去中心化应用（dApp）。

从字节恢复

```typescript
// restore-keypair-from-bytes.ts
import { Keypair } from "@solana/web3.js";

const keypair = Keypair.fromSecretKey(
  Uint8Array.from([
    174, 47, 154, 16, 202, 193, 206, 113, 199, 190, 53, 133, 169, 175, 31, 56,
    222, 53, 138, 189, 224, 216, 117, 173, 10, 149, 53, 45, 73, 251, 237, 246,
    15, 185, 186, 82, 177, 240, 148, 69, 241, 227, 167, 80, 141, 89, 240, 121,
    121, 35, 172, 247, 68, 251, 226, 218, 48, 63, 176, 109, 168, 89, 238, 135,
  ])
);
```

从 base58 字符串恢复

```typescript
// restore-keypair-from-base58.ts
import { Keypair } from "@solana/web3.js";
import * as bs58 from "bs58";

const keypair = Keypair.fromSecretKey(
  bs58.decode(
    "5MaiiCavjCmn9Hs1o3eznqDEhRwxo7pXiAYez7keQUviUkauRiTMD8DrESdrNjN8zd9mTmVhRvBJeg5vhyvgrAhG"
  )
);
```

## 如何验证密钥对

如果你有一个密钥对，你可以验证密钥是否匹配给定的公钥。

```typescript
// verify-keypair.ts
import { Keypair, PublicKey } from "@solana/web3.js";

const publicKey = new PublicKey("24PNhTaNtomHhoy3fTRaMhAFCRj4uHqhZEEoWrKDbR5p");

const keypair = Keypair.fromSecretKey(
  Uint8Array.from([
    174, 47, 154, 16, 202, 193, 206, 113, 199, 190, 53, 133, 169, 175, 31, 56,
    222, 53, 138, 189, 224, 216, 117, 173, 10, 149, 53, 45, 73, 251, 237, 246,
    15, 185, 186, 82, 177, 240, 148, 69, 241, 227, 167, 80, 141, 89, 240, 121,
    121, 35, 172, 247, 68, 251, 226, 218, 48, 63, 176, 109, 168, 89, 238, 135,
  ])
);

console.log(keypair.publicKey.toBase58() === publicKey.toBase58());
// 输出: true
```

## 如何验证公钥

在某些特殊情况下（例如程序派生地址），公钥可能没有与之关联的私钥。你可以通过查看公钥是否位于 ed25519 曲线上来检查这一点。只有位于曲线上的公钥才能被有钱包的用户控制。

```typescript
// check-public-key.ts
import { PublicKey } from "@solana/web3.js";

// 注意，Keypair.generate() 总会生成对用户有效的公钥

// 有效的公钥
const key = new PublicKey("5oNDL3swdJJF1g9DzJiZ4ynHXgszjAEpUkxVYejchzrY");
// 位于 ed25519 曲线上的公钥，适用于用户
console.log(PublicKey.isOnCurve(key.toBytes())); // 输出: true

// 有效的公钥
const offCurveAddress = new PublicKey(
  "4BJXYkfvg37zEmBbsacZjeQDpTNx91KppxFJxRqrz48e"
);

// 不位于 ed25519 曲线上的公钥，因此不适用于用户
console.log(PublicKey.isOnCurve(offCurveAddress.toBytes())); // 输出: false

// 无效的公钥
const errorPubkey = new PublicKey("testPubkey");
```

## 如何生成密钥对的助记词

生成密钥对的一种方法是使用助记词。助记词通常通过使用一串可读单词（而不是一短串随机数字和字母）来提高钱包内的用户体验。

```typescript
// generate-mnemonic.ts
import * as bip39 from "bip39";

const mnemonic = bip39.generateMnemonic();
console.log(mnemonic); // 输出: 生成的助记词
```

## 如何从助记词恢复密钥对

许多钱包扩展使用助记词来表示其密钥。你可以将助记词转换为密钥对进行本地测试。

恢复 BIP39 格式助记词

```typescript
// restore-bip39-mnemonic.ts
import { Keypair } from "@solana/web3.js";
import * as bip39 from "bip39";

const mnemonic =
  "pill tomorrow foster begin walnut borrow virtual kick shift mutual shoe scatter";

// 参数: (mnemonic, password)
const seed = bip39.mnemonicToSeedSync(mnemonic, "");
const keypair = Keypair.fromSeed(seed.slice(0, 32));

console.log(`${keypair.publicKey.toBase58()}`);
// 输出: 5ZWj7a1f8tWkjBESHKgrLmXshuXxqeY9SYcfbshpAqPG
```

恢复 BIP44 格式助记词

```typescript
// restore-bip44-mnemonic.ts
import { Keypair } from "@solana/web3.js";
import { HDKey } from "micro-ed25519-hdkey";
import * as bip39 from "bip39";

const mnemonic =
  "neither lonely flavor argue grass remind eye tag avocado spot unusual intact";

// 参数: (mnemonic, password)
const seed = bip39.mnemonicToSeedSync(mnemonic, "");
const hd = HDKey.fromMasterSeed(seed.toString("hex"));

for (let i = 0; i < 10; i++) {
  const path = `m/44'/501'/${i}'/0'`;
  const keypair = Keypair.fromSeed(hd.derive(path).privateKey);
  console.log(`${path} => ${keypair.publicKey.toBase58()}`);
}
```

## 生成自定义地址（Vanity Address）是一种希望公钥以特定字符开头的需求

例如，一个人可能希望公钥以 "elv1s" 开头，或者 "cook"。这样可以帮助其他人记住公钥的所有者，使公钥更容易识别。

请注意：所需的字符越多，生成自定义地址所需的时间就越长。

你可以使用 Solana CLI 来生成自定义地址：

```bash
solana-keygen grind --starts-with elv1s:1
```

这条命令告诉 Solana 的 keygen 工具要生成一个以 "elv1s" 开头的公钥。`:1` 表示只找到一个符合条件的公钥即可停止搜索。

## 如何签名和验证消息

密钥对的主要功能之一是签名消息并验证签名。签名的验证允许接收方确信数据是由特定私钥的所有者签名的。

下面是使用 TweetNaCl 加密库进行消息签名和验证的示例：

```typescript
// sign-message.ts
import { Keypair } from "@solana/web3.js";
import nacl from "tweetnacl";
import { decodeUTF8 } from "tweetnacl-util";

const keypair = Keypair.fromSecretKey(
  Uint8Array.from([
    174, 47, 154, 16, 202, 193, 206, 113, 199, 190, 53, 133, 169, 175, 31, 56,
    222, 53, 138, 189, 224, 216, 117, 173, 10, 149, 53, 45, 73, 251, 237, 246,
    15, 185, 186, 82, 177, 240, 148, 69, 241, 227, 167, 80, 141, 89, 240, 121,
    121, 35, 172, 247, 68, 251, 226, 218, 48, 63, 176, 109, 168, 89, 238, 135,
  ])
);

const message = "The quick brown fox jumps over the lazy dog";
const messageBytes = decodeUTF8(message);

// 签名消息
const signature = nacl.sign.detached(messageBytes, keypair.secretKey);

// 验证签名
const result = nacl.sign.detached.verify(
  messageBytes,
  signature,
  keypair.publicKey.toBytes()
);

console.log(result); // 输出: true 或 false，取决于签名是否有效
```

在这个示例中：

- 我们首先从给定的密钥数组中创建了一个 Solana 的 `Keypair`。
- `message` 是要签名的消息，将其转换为字节数组 `messageBytes`。
- 使用 `nacl.sign.detached` 函数对消息进行签名，生成 `signature`。
- 使用 `nacl.sign.detached.verify` 函数来验证签名的有效性，通过比较消息字节数组、签名和公钥的字节数组。

确保在实际应用中，密钥对的私钥和公钥的管理及存储都遵循最佳的安全实践。

## 如何在 React 中连接钱包

Solana 的 wallet-adapter 库使得在客户端管理钱包连接变得简单。如需完整指南，请参阅如何将 wallet-adapter 添加到 Next.js 中。

### 在 React 中连接钱包的步骤

#### 使用快速设置

如果要快速设置 React 应用，请使用以下命令：

```bash
npx create-solana-dapp <app-name>
```

#### 手动设置

如果要手动设置，请运行以下命令安装所需的依赖项：

```bash
npm install --save \
    @solana/wallet-adapter-base \
    @solana/wallet-adapter-react \
    @solana/wallet-adapter-react-ui \
    @solana/wallet-adapter-wallets \
    @solana/web3.js \
    react
```

#### 设置 WalletProvider

可以通过 WalletProvider 设置连接到用户钱包并进行后续交易的配置。

```typescript
import React, { FC, useMemo } from "react";
import {
  ConnectionProvider,
  WalletProvider,
} from "@solana/wallet-adapter-react";
import { WalletAdapterNetwork } from "@solana/wallet-adapter-base";
import { UnsafeBurnerWalletAdapter } from "@solana/wallet-adapter-wallets";
import {
  WalletModalProvider,
  WalletDisconnectButton,
  WalletMultiButton,
} from "@solana/wallet-adapter-react-ui";
import { clusterApiUrl } from "@solana/web3.js";

// 导入默认样式，可以被你的应用覆盖
require("@solana/wallet-adapter-react-ui/styles.css");

export const Wallet: FC = () => {
  // 网络可以设置为 'devnet', 'testnet', 或 'mainnet-beta'
  const network = WalletAdapterNetwork.Devnet;

  // 你也可以提供自定义的 RPC 端点
  const endpoint = useMemo(() => clusterApiUrl(network), [network]);

  const wallets = useMemo(
    () => [
      /**
       * 实现以下标准之一的钱包将自动可用。
       *
       *   - Solana Mobile Stack Mobile Wallet Adapter Protocol
       *     (https://github.com/solana-mobile/mobile-wallet-adapter)
       *   - Solana Wallet Standard
       *     (https://github.com/anza-xyz/wallet-standard)
       *
       * 如果你希望支持一个不支持这些标准的钱包，
       * 可以在此处实例化其传统钱包适配器。常见的传统适配器可以在
       * npm 包 `@solana/wallet-adapter-wallets` 中找到。
       */
      new UnsafeBurnerWalletAdapter(),
    ],
    // eslint-disable-next-line react-hooks/exhaustive-deps
    [network]
  );

  return (
    <ConnectionProvider endpoint={endpoint}>
      <WalletProvider wallets={wallets} autoConnect>
        <WalletModalProvider>
          <WalletMultiButton />
          <WalletDisconnectButton />
          {/* 你的应用组件在这里嵌套在上下文提供者中。 */}
        </WalletModalProvider>
      </WalletProvider>
    </ConnectionProvider>
  );
};
```

这段代码实现了在 React 应用中使用 Solana 的 `wallet-adapter` 库来管理钱包连接和交易操作。务必确保在实际应用中使用时，密钥管理和安全性得到充分考虑。
