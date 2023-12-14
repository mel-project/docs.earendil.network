# About

{% hint style="warning" %}
Currently, Earendil is in **pre-alpha**. Not all features are ready, and some documentation pages document in-development, unreleased features.
{% endhint %}

[**Earendil**](https://earendil.network) is a decentralized, censorship-resistant, and incentive-compatible communication and value transfer network. It allows any two users of the Earendil network to communicate and transact freely, even against powerful state-level attackers.

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Or more simply, Earendil is an **unstoppable magic internet wormhole**: bytes and money enter on one end and exit on the other, and nobody can stop the transfers. With Earendil, you can:

* Build anonymity-protecting apps and peer-to-peer networks that cannot be taken down &#x20;
* Browse normal Internet websites anonymously, hiding your IP address and location
* Earn fees by running Earendil infrastructure nodes
* Send Mel-based cryptocurrencies off-chain at very low cost

## Why Earendil?

At first sight, Earendil seems similar to existing peer-to-peer onion routing networks like I2P or mixnets like Nym. But it has several distinguishing features:

### Robust ban resistance

Earendil resists both [type-I](https://nullchinchilla.me/2023/05/two-kinds-of-censorship-resistance/) censorship (filtering content or users within the network, or **filter resistance**) and [type-II](https://nullchinchilla.me/2023/05/two-kinds-of-censorship-resistance/) censorship (blocking access to Earendil entirely, or **ban resistance**).&#x20;

Strong ban resistance is very rare in other projects. Even when present, it's generally limited to special-case defenses (e.g. Tor obfuscated bridges) against nation-state firewalls like the Great Firewall of China.

On the other hand, Earendil is designed to work _even_ _if the GFW were deployed worldwide_. It makes no assumptions as to most of the network existing in the "free world". This is possible because:

* **Link-by-link obfuscation**: Earendil traffic is, by default, difficult to distinguish from "normal" network traffic. Furthermore, the protocol used for any particular node-to-node link can be switched out, using a "pluggable transport" architecture similar to those used for Tor bridges, for particular severe network environments (e.g. networks that only allow plaintext HTTP and man-in-the-middle all HTTPS traffic)
* **Friend-to-friend routing**: Earendil routes traffic using a unique, invite-only ["friend-to-friend" architecture ](wiki/architecture.md)that does not reveal information about the entire network to every node, making it difficult for even powerful attackers to compile a list of Earendil nodes useful for surveillance or censorship.

### Decentralized, sybil-resistant incentives

Earendil **optionally** allows every node to set a price that their peers must pay to consume its resources, through MEL-settled cryptocurrency micropayments.

Micropayments elegantly solve sybil-resistance (preventing bad nodes from flooding the network), incentives for honest nodes, and DoS resistance. Nodes behaving badly will not be paid by their peers, while honest nodes are incentivized to compete in a free market to provide the best service to their clients. Bad actors attempting to spam Earendil must pay the network accordingly.

This has important advantages over other incentive/sybil-resistance mechanisms, such as "tax/subsidy" models based on paying fees into a central smart contract and proving contributions to the network to withdraw from it. This is explained further in this [blog post](https://nullchinchilla.me/2023/07/earendil-incentives/).

### Tunable anonymity/performance tradeoff

Like Tor and Nym, Earendil messages use onion encryption to protect the anonymity and privacy of user traffic.&#x20;

But unlike these other systems, Earendil is not intended _purely_ as a privacy-protecting tool. Users are able to freely trade off anonymity against performance. By selecting different route-selection algorithms and mixnet delay distributions, Earendil can be used as any of the following:

* A `ngrok` -like NAT-traversal tool with strong ban-resistant network, useful for things like VPNs and phone calls
* A Tor-like low-latency onion routing network, useful for lower-security darknet websites and the like
* A Nym-like or even slower high-latency mix network, useful for highly anonymous communication

To some extent, the anonymity sets of these use cases overlap, so even the low-latency traffic achieve greater anonymity than possible in a purely low-latency network.

## Development status and roadmap

Currently, Earendil is in **pre-alpha**. You can join the Earendil network and communicate over it, but some important features are rudimentary or incomplete:

| Feature                                    | Completion status                                                | Notes                                                                                                                                      |
| ------------------------------------------ | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Joining as a relay                         | :white\_check\_mark: Done                                        |                                                                                                                                            |
| Joining as a client                        | :white\_check\_mark: Done                                        |                                                                                                                                            |
| Onion routing                              | [🚧](https://emojipedia.org/construction) Rudimentary            | _No ability to customize or randomize route selection. Shortest path only._                                                                |
| Havens (anon hosting)                      | :white\_check\_mark:                                             |                                                                                                                                            |
| Web proxying                               | [🚧](https://emojipedia.org/construction) Rudimentary            | _You can host web proxies to help other users use the Web through Earendil. But no user authentication or access control is implemented._  |
| Debt accounting                            | <p>❌ Not implemented<br>(planned for <strong>0.2.x</strong>)</p> | _Calculates how much to pay neighbors for their resources._                                                                                |
| Manual debt settlement                     | <p>❌ Not implemented<br>(planned for <strong>0.2.x</strong>)</p> | _Allows settling the calculated debt out-of-band and manually resetting it in the protocol._                                               |
| Mixnet delays                              | <p>❌ Not implemented<br>(planned for <strong>0.3.x</strong>)</p> | _Delays messages to defend anonymity against large-scale attackers_                                                                        |
| Auto debt settlement (MVP)                 | <p>❌ Not implemented<br>(planned for <strong>0.3.x</strong>)</p> | _Automatically settle MEL-denominated debt using on-chain transfers_                                                                       |
| Mel-backed Sybil resistance                | <p>❌ Not implemented<br>(planned for <strong>0.4.x</strong>)</p> | _Limit the number of relays by requiring staking assets on the Mel blockchain_                                                             |
| Auto debt settlement with payment channels | <p>❌ Not implemented<br>(planned for <strong>0.4.x</strong>)</p> | _Settle debt off-chain using anonymous payment channels. Earendil can then be used as an off-chain asset transfer layer._                  |

