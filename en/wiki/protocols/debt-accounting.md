# WIP: Debt accounting

{% hint style="warning" %}
This is a **design document** for an _unimplemented_ feature.
{% endhint %}

Debt accounting in Earendil has three important parts:

* **Negotiating** the price paid on a link
* **Tracking** the net debt on a link
* **Settling** the debt

This is all done through GlobalRPC verbs.

## Negotiating a price

Before any data packets can be processed, both sides call `push_price` on the other side. This RPC method _pushes_ a **price**, in MEL/packet, to the other side, as well as a **debt limit**, returning whether or not it is accepted. It also contains metadata about how the caller wishes to be paid.

Each sides refuses to peel/relay any packets until the other side has signaled acceptance of the price.

`push_price` may be called at any point, with the understanding that the other side may take a while to apply the changes to how debt etc is calculated.

`in_route` and `out_route` in config files specify maximum allowable prices and debt limit ranges.

## Tracking debt

Due to packet loss and delays, we cannot know _exactly_ how many packets were received by the other side of the link. Continually doing two-way synchronized communication to come to consensus on the debt level is also impractical.

This means that in the steady state, we _conservatively_ estimate the debt by using how much we sent the other side to estimate how much we owe them, and how much we actually received to estimate how much they owe us.

We then continually _correct_ this, by both sides periodically (say, every 1 minute) calling `push_debt` on the other side. This method will provide a _signed and timestamped_ claim that according to the pusher, the ledger looks a certain way.

Note that at this phase, we do not "net" the debt yet.

## Settling the net debt

At any point, either party can initiate a **settlement**. This is done by calling . This starts a process that reduces debt for both sides.

The object passed to `start_settlement` needs to contain:

* How much debt to reduce for left
* How much debt to reduce for right
* Proof of payment --- in the "net" amount. This can be in either direction!
* A signature

Note that the "proof of payment" is highly payment-method specific. It can also be a absent, in which case `start_settlement` will not automatically return until the other side presses a button the UI

## Manual debt settlement UX

There will be a GUI "node dashboard" that every node operator can access, with a chat interface for chatting with the operator's immediate neighbors. There are LinkRPC methods for pushing a message to the other side.

In each chat conversation, the UX will look something like:

```
+------------------+------------------+------------------+
|     Neigbbor     |     Messages     |   Debt Status    |
|     List         |                  |                  |
|------------------|------------------|------------------|
| Alice            | Alice: Hi!       |   Net Debt:      |
| Bob              | You: Hello!      |   $150.00        |
| Charlie          | Alice: How are   |                  |
| Dave             | you?             |                  |
| Eve              | You: I'm good,   |                  |
|------------------| you?             |                  |
|                  | Alice: I'm fine  |   [Manual        |
|                  | thanks.          |   Settlement]    |
|                  |                  |                  |
|                  |                  |                  |
|                  |                  |                  |
+------------------+------------------+------------------+
```

Pressing the "manual settlement" button will call `start_settlement` with an empty proof of payment.

The other person will receive a notification to either accept or reject the settlement.
