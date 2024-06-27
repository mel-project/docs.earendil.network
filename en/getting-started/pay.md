# Pay and get paid

In the Earendil network, nodes pay and get paid by their _immediate neighbors_.

This creates a free market for bandwidth --- if one neighbor is too expensive or unreliable, simply disconnect from them and find a cheaper or more reliable provider. Once you pay your neighbor, it's their responsibility to route your packets to their destinations. This is just like using the internet: you pay your ISP (like T-Mobile) and don't worry about the rest.

## Price & debt limit

Two neighbors agree on a price and debt limit for sending packets when they first connect to each other. They do so in the `price_config` section of the `in_route` or `out_route` block:

```yaml
# every in_route and out_route has a price_config
price_config:
  # how much you charge per incoming packet, in nanoMELs
  inbound_price: 5
  # debt limit for inbound packets, in nanoMELs
  inbound_debt_limit: 50000
  # max price you're willing to pay per outgoing packet, in nanoMELs
  outbound_max_price: 10
  # min debt limit you accept for outbound packets, in nanoMELs
  # Negative debt limit = requires prepayment
  outbound_min_debt_limit: -100
```

As an example, say we have relay Alice and client Bob. Alice has this `price_config` in her in_route for Bob:

```yaml
price_config:
  inbound_price: 5
  inbound_debt_limit: 50000
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

This means Alice charges 5 nMELs for every packet Bob sends her, Bob can owe alice at most 50,000 nMELs before she disconnects from him, and Alice does not pay Bob anything to send packets to him.

## Payment methods

A node specifies all the payment methods they support in the `payment_methods` section of their config file. If two neighbors don't share any supported payment methods in common, they won't be able to connect (unless both of then charge a price of 0).

We currently support 2 payment methods: on-chain payments on the Mel blockchain and proof-of-work.

```yaml
payment_methods:
  on_chain:
    secret: <your-mel-wallet-secret>
  pow # no arguments required to support PoW payments
```

The `secret` field for `on_chain` is the secret key of the Mel wallet you'll use to send and receive payments. [Here's](https://docs.melproject.org/developer-guides/using-wallets) how to set up a Mel wallet. You can export the secret key from an existing wallet using:

```bash
melwallet-cli --wallet-path <path-to-your-mel-wallet> export-sk
```

As an example, this config means that your node only accepts on-chain payments:

```yaml
payment_methods:
  on_chain:
    secret: <your-mel-wallet-secret>
```

## Testing payments

Todo!

To learn more about Earendil's incentive system, read [this post](https://nullchinchilla.me/2023/07/earendil-incentives/).
