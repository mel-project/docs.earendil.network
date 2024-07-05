# Pay and get paid

In the Earendil network, nodes pay and get paid by their _immediate neighbors_.

This creates a free market for bandwidth --- if one neighbor is too expensive or unreliable, simply disconnect from them and find a cheaper or more reliable provider. Once you pay your neighbor, it's their responsibility to route your packets to their destinations. This is just like using the internet: you pay your ISP (like T-Mobile) and don't worry about the rest.

To learn more about Earendil's incentive system, read [this post](https://nullchinchilla.me/2023/07/earendil-incentives/).

## Price & debt limit

Two neighbors agree **out-of-band** on a price and debt limit for sending packets when they first connect to each other. They then specify this information in the `price_config` section of the `in_route` or `out_route` block:

```yaml
# every in_route and out_route has a price_config
price_config:
  # how much you charge per incoming packet, in µMEL
  inbound_price: 5
  # debt limit for inbound packets, in µMEL
  inbound_debt_limit: 50000
  # max price you're willing to pay per outgoing packet, in µMEL
  # this field prevents your neighbor from charging you more than the agreed amount
  outbound_max_price: 10
  # min debt limit you accept for outbound packets, in µMEL
  # negative debt limit means prepayment is required
  # this field prevents your neighbor from charging you a prepayment larger than the agreed amount
  outbound_min_debt_limit: -100
```

As an example, say we have relay Alice and client Bob. Alice has this `price_config` in her in_route for Bob:

```yaml
price_config:
  inbound_price: 1
  inbound_debt_limit: 5000
  outbound_max_price: 0
  outbound_min_debt_limit: 0
```

While Bob has this `price_config` in his out_route for connecting to Alice:

```yaml
price_config:
  inbound_price: 0
  inbound_debt_limit: 0
  outbound_max_price: 10
  outbound_min_debt_limit: -100
```

This means Alice charges 1 µMEL for every packet Bob sends her, Bob can owe alice at most 5,000 µMELs before she disconnects from him, and Alice does not pay Bob anything to send packets to him.

## Payment methods

A node specifies all the payment methods they support in the `payment_methods` section of their config file. If two neighbors don't share any supported payment methods in common, they won't be able to connect (unless they both charge a price of 0).

We currently support 2 payment methods: on-chain payments on the Mel blockchain and proof-of-work (PoW).

```yaml
payment_methods:
  - on_chain: <your-mel-wallet-secret>
  # no arguments required to support PoW payments
  - pow
```

The `secret` field for `on_chain` is the secret key of the Mel wallet you'll use to send and receive payments. [Here's](https://docs.melproject.org/developer-guides/using-wallets) how to set up a Mel wallet. You can export the secret key from an existing wallet using:

```bash
melwallet-cli --wallet-path <path-to-your-mel-wallet> export-sk
```

As an example, this config means that your node only accepts on-chain payments:

```yaml
payment_methods:
  - on_chain: <your-mel-wallet-secret>
```

## Testing payments

The default bootstrap node we've been using throughout the tutorials is entirely free. To test payments, use this node that supports both on-chain and PoW payments:

```yaml
example-relay-paid:
  connect: 62.210.93.59:19999
  fingerprint: 14154070117b3c1a71fa2fc6bc7d20e5afc93fbe98a13b86b013d0a91215f74f
  obfs:
    sosistab3: correct-horse-battery-pink-staple-pasta-apple
  price_config:
    inbound_debt_limit: 0.0
    inbound_price: 0.0 # do not set a price > 0; our example payments server will not pay!
    outbound_max_price: 1
    outbound_min_debt_limit: 0.0

# don't forget to specify payment_methods you support
payment_methods:
  - pow
```
