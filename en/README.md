# About

{% hint style="warning" %}
Currently, Earendil is in **pre-alpha**. Not all features are ready, and some documentation pages document in-development, unreleased features.
{% endhint %}

[**Earendil**](https://earendil.network) is a decentralized, censorship-resistant, and incentive-compatible communication and value transfer network. It allows any two users of the Earendil network to communicate and transact freely, even against powerful state-level attackers.

<figure><img src=".gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

With Earendil, you can:

- Build anonymity-protecting apps and peer-to-peer networks that cannot be taken down
- Browse normal Internet websites anonymously, hiding your IP address and location
- Earn fees by running Earendil infrastructure nodes
- Send Mel-based cryptocurrencies off-chain at very low cost

## Why Earendil?

At first sight, Earendil seems similar to existing peer-to-peer onion routing networks like I2P or mixnets like Nym. But it has several distinguishing features:

### Robust ban resistance

Earendil resists both [type-I](https://nullchinchilla.me/2023/05/two-kinds-of-censorship-resistance/) censorship (filtering content or users within the network, or **filter resistance**) and [type-II](https://nullchinchilla.me/2023/05/two-kinds-of-censorship-resistance/) censorship (blocking access to Earendil entirely, or **ban resistance**).

Strong ban resistance is rare in other projects. Even when present, it's generally limited to special-case defenses (e.g. Tor obfuscated bridges) against nation-state firewalls like the Great Firewall of China.

On the other hand, Earendil is designed to work _even_ _if the GFW were deployed worldwide_. It makes no assumptions as to most of the network existing in the "free world".

### Decentralized, sybil-resistant incentives

Earendil **optionally** allows every node to set a price that their peers must pay to consume its resources, through MEL-settled cryptocurrency micropayments.

Micropayments elegantly solve sybil-resistance (preventing bad nodes from flooding the network), incentives for honest nodes, and DoS resistance. Nodes behaving badly will not be paid by their peers, while honest nodes are incentivized to compete in a free market to provide the best service to their clients. Bad actors attempting to spam Earendil must pay the network accordingly.

This has important advantages over other incentive/sybil-resistance mechanisms, such as "tax/subsidy" models based on paying fees into a central smart contract and proving contributions to the network to withdraw from it. This is explained further in this [blog post](https://nullchinchilla.me/2023/07/earendil-incentives/).

## How does Earendil work?

Earendil's architecture, in short, is _a mix network overlaid onto a ban-resistant "internet"_. This means a two-part design:

### The "ban-resistant internet" part

We overlay a peer-to-peer packet routing network onto the Internet, and this network itself works very much like the Internet --- packets have a destination, and **relays** relay packets hop by hop closer to their destination.

This layer hides from hostile ISPs and achieve ban resistance by combining two features:

- **Link-by-link obfuscation**: Earendil traffic is, by default, difficult to distinguish from "normal" network traffic. Furthermore, the protocol used for any particular node-to-node link can be switched out, using a "pluggable transport" architecture similar to those used for Tor bridges, for particular severe network environments (e.g. networks that only allow plaintext HTTP and man-in-the-middle all HTTPS traffic)
- **Invite-only, limited-information architecture**: Earendil routes traffic using a unique, invite-only [architecture](wiki/architecture.md) that only reveals the IP address of the immediate peers of every participant, making it difficult for even powerful attackers to compile a list of Earendil nodes' IP addresses for surveillance or censorship. Censors blocking all nodes that they know of is likely to only block themselves from the network.

This layer _does not provide anonymity_, only reliability and ban resistance. We also do all incentives in this layer through a simple mechanism where users directly pay peers for all resources used on them.

### The "mix network" part

On top of this peer-to-peer network, we overlay a _mix network_ similar in design to Nym, providing strong anonymity through onion encryption and delays. All relays must also participate in the mix network to be part of the Earendil network.

The mix network is designed to maintain anonymity even with the attacker completely monitoring all traffic on the underlying network. This prevents any information about who's communicating with whom from leaking no matter how severely the obfuscation protocols or the incentive payment system leaks information.

## Development status and roadmap

Currently, Earendil is in **pre-alpha**. You can join the Earendil network and communicate over it, but some important features are rudimentary or incomplete:

| Feature                                    | Completion status                                                 | Notes                                                                                                                                     |
| ------------------------------------------ | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Joining as a relay                         | :white_check_mark: Done                                           |                                                                                                                                           |
| Joining as a client                        | :white_check_mark: Done                                           |                                                                                                                                           |
| Onion routing                              | 🚧 Rudimentary                                                    | _No ability to customize route selection._                                                                                                |
| Havens (anon hosting)                      | :white_check_mark:                                                |                                                                                                                                           |
| Web proxying                               | 🚧 Rudimentary                                                    | _You can host web proxies to help other users use the Web through Earendil. But no user authentication or access control is implemented._ |
| Debt accounting                            | :white_check_mark:                                                | _Calculates how much to pay neighbors for their resources._                                                                               |
| Manual debt settlement                     | :white_check_mark:                                                | _Allows settling the calculated debt out-of-band and manually resetting it in the protocol._                                              |
| Mixnet delays                              | 🚧 Rudimentary                                                    | _Delays messages to defend anonymity against large-scale attackers_                                                                       |
| Auto debt settlement (MVP)                 | 🚧 Rudimentary                                                    | _Automatically settle MEL-denominated debt using on-chain transfers_                                                                      |
| Mel-backed Sybil resistance                | <p>❌ Not implemented<br>(planned for <strong>0.6.x</strong>)</p> | _Limit the number of relays by requiring staking assets on the Mel blockchain_                                                            |
| Auto debt settlement with payment channels | <p>❌ Not implemented<br>(planned for <strong>0.6.x</strong>)</p> | _Settle debt off-chain using anonymous payment channels. Earendil can then be used as an off-chain asset transfer layer._                 |
