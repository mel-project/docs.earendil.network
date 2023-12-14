# Reply blocks

A particular challenge in packet-based onion routing is when Alice sends a message to Bob, how can Bob reply.

The trivial solution is to attach a header to the message with a "source address". This is okay in the normal case, where anonymity against the counterparty is not desired, but not okay if Alice wants to hide her identity from Bob.

In that case, we use **reply blocks**. Let's say $$A$$ sends a message to $$B$$ over intermediary relays $$R_1,\dots,R_n$$. To help $$B$$ respond, $$A$$ attaches a reply block containing:

* A randomly generated fingerprint: the **source ID**
* The first hop on the return-path, $$R_n$$. WLOG we assume there is only one path between the parties, and the reply packet follows the same path but in the other direction.
* An onion-encrypted header encoding the path $$B,R_n,\dots,R_1,A$$, as well as ephemeral pubkeys for each layer
* An ephemeral onion public key $$k_{eph}$$

$$B$$ then encrypts and stuffs the response (which is an InnerPacket) into 8192 bytes using $$k_{eph}$$and an randomly generated, ephemeral onion public key. He attaches the header verbatim, then sends the packet off to _himself_, as the first hop in Earendil routes is always the sending node itself. In $$B$$'s attempt to peel an layer of the onion while forwarding the packet, the mssage body is garbled (that is, ChaCha20-encrypted, because encryption and decryption are identical for stream ciphers), making this reply packet indistinguishable from forward packets to the next hop, $$R_n$$.

The message will eventually pass back to $$A$$, garbled by the intermediary nodes' attempts to "decrypt" the message with layers of stream-ciphers. But $$A$$ can reconstruct the keystream that was XOR'ed against the payload, and thus reconstruct the ciphertext that $$B$$ sent.

$$A$$ is then able to decrypt the response from $$B$$.

## How this is exposed in the API

The `recv_from` function will return a packet and _source_. This source is either

* a _direct_ address: a fingerprint of a relay or client
* a _reply_ address: a randomly-generated fingerprint that serves as the source ID

When `send_to` processes a reply address, it looks up an unused reply block corresponding to the source ID and routes the message through it.

When a node calls `send_to` using an anonymous source address, we send a packet of 8 reply blocks in addition to the message, so that the destination can use them to respond to us.

## DoS resistance

Only the last N reply blocks received from a source, within the last T seconds are kept around. Memory usage can be tightly bound with something like a weighted `moka` cache.

Source IDs that are too inactive become dead.
