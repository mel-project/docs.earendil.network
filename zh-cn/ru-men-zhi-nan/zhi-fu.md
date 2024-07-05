# 支付与获得支付

在 Earendil 网络中，节点通过他们的**直接邻居**进行支付与受支付。

这创造了一个带宽的自由市场 --- 如果一个邻居太贵或不可靠，只需断开连接并找到一个更便宜或更可靠的提供者。一旦你支付你的邻居，就由他们负责将你的数据包路由到目的地。这就像使用互联网一样：你支付你的 ISP（例如 T-Mobile），然后不用担心其余的。

要了解更多关于 Earendil 带宽市场的信息，请阅读[此文章](https://nullchinchilla.me/2023/07/earendil-incentives/)。

## 价格与债务限额

两个邻居在首次连接时**离线**同意一个发送数据包的价格和债务限额。然后他们在 `in_route` 或 `out_route` 块的 `price_config` 部分中指定这些信息：

```yaml
# 每个 in_route 和 out_route 都有一个 price_config
price_config:
  # 你对每个传入的数据包收取的费用，以 µMEL 为单位
  inbound_price: 5
  # 传入数据包的债务限额，以 µMEL 为单位
  inbound_debt_limit: 50000
  # 你愿意为每个传出数据包支付的最高价格，以 µMEL 为单位
  # 该字段防止你的邻居向你收取超过约定金额的费用
  outbound_max_price: 10
  # 你接受的传出数据包的最低债务限额，以 µMEL 为单位
  # 负债限额表示需要预付款
  # 该字段防止你的邻居向你收取超过约定金额的预付款
  outbound_min_debt_limit: -100
```

例如，假设我们有中继 Alice 和客户端 Bob。Alice 在她的连接到 Bob 的`in_route`中有以下 `price_config`：

```yaml
price_config:
  inbound_price: 1
  inbound_debt_limit: 5000
  outbound_max_price: 0
  outbound_min_debt_limit: 0
```

而 Bob 在他连接到 Alice 的`out_route`中有以下 `price_config`：

```yaml
price_config:
  inbound_price: 0
  inbound_debt_limit: 0
  outbound_max_price: 10
  outbound_min_debt_limit: -100
```

这意味着 Alice 对 Bob 发送给她的每个数据包收费 1 µMEL，Bob 最多欠 Alice 5,000 µMEL 才能继续连接，Alice 发送数据包给 Bob 无需支付任何费用。

## 支付方法

一个节点在其配置文件的`payment_methods`部分中指定所有支持的支付方法。如果两个邻居没有共同支持的支付方法，他们将无法连接（除非他们都不收费）。

我们目前支持 2 种支付方法：在 Mel 区块链上的链上支付和工作量证明（PoW）。

```yaml
payment_methods:
  - on_chain:
    secret: <your-mel-wallet-secret>
  # PoW 支付不需要参数
  - pow
```

`on_chain`的`secret`字段是你将用来发送和接收支付的 Mel 钱包的密钥。[这里](https://docs.melproject.org/developer-guides/using-wallets)有如何设置 Mel 钱包的指南。你可以使用以下命令从现有钱包中导出密钥：

```bash
melwallet-cli --wallet-path <path-to-your-mel-wallet> export-sk
```

例如，这个配置表示你的节点只接受链上支付：

```yaml
payment_methods:
  on_chain:
    secret: <your-mel-wallet-secret>
```

## 测试支付

在整个教程中我们使用的默认引导节点是完全免费的。要测试支付，使用这个支持链上支付和工作量证明（PoW）支付的节点：

```yaml
example-relay-paid:
  connect: 172.233.162.12:19998
  fingerprint: 14154070117b3c1a71fa2fc6bc7d20e5afc93fbe98a13b86b013d0a91215f74f
  obfs:
    sosistab3: correct-horse-battery-pink-staple-pasta-apple
  price_config:
    inbound_price: 0.5
    inbound_debt_limit: 1000
    outbound_max_price: 0
    outbound_min_debt_limit: 0

# 别忘了指定你支持的支付方法
payment_methods:
  pow:
```
