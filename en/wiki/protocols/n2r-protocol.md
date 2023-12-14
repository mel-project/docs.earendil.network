# N2R (node-to-relay) protocol

N2R allows communication between any two nodes, _as long as one of them is a relay_.

## Packet format

The packet format is based on the `InnerPacket` Rust enum:

```rust
/// Represents the actual end-to-end packet that is carried in the 8192-byte payloads. Either an application-level message, or a batch of reply blocks.
pub enum InnerPacket {
    /// Normal messages
    Message(Message),
    /// Reply blocks, used to construct relay->anon messages
    ReplyBlocks(Vec<ReplyBlock>),
    
/// An inner packet message with corresponding UDP port-like source and destinaton docks
pub struct Message {
    pub source_dock: Dock,
    pub dest_dock: Dock,
    pub body: Vec<Bytes>,
}

pub type Dock = u32;
```

An `InnerPacket` is stuffed into the 8192-byte payload by this process:

1. First, we encode the packet using [bincode](https://docs.rs/bincode/latest/bincode/), getting `box_plaintext`.
2. We then box-encrypt using an ephemeral keypair, whose public half is `box_epk`, getting `box_ciphertext`.
3. We sign `box_epk` with our identity secret key `identity_sk`, whose public half is `identity_pk`, producing `identity_signature`
4. We then package everything into a tuple `(identity_pk, identity_signature, box_ciphertext)`, bincode it, and pad it to 8192 bytes.

This picture roughly illustrates the structure of a fully encoded `InnerPacket` that stores a normal `Message`.

![](../../.gitbook/assets/n2r\_innerpacket.png)

## Socket abstraction

The typical interface exposed by N2R is not raw functions for sending and receiving packets. Instead, we use a _socket_ abstraction inspired by UDP. Each socket represents an `Endpoint`, a _local fingerprint:dock pair,_ that can receive and send messages. More specifically:

* The user constructs a socket by **binding** to an identity and a dock number.
  * This identity can either be the long-term identity of the node, or a temporary anonymous identity.
* The socket has a method to **send** a message to a fingerprint and dock number.
  * This formats a message with:
    * source identity public key and source dock number taken from the identity and dock number of the socket
    * destination identity looked up by fingerprint
  * If the socket is bound to a temporary identity, reply blocks must be sent to the destination fingerprint as well so that they can talk back to us.
* There's also a **receive** method, which returns a message and a source endpoint, which consists of a fingerprint and a dock number.
  * This blocks until there is an incoming message addressed to the identity and dock number that the socket is bound to.
  * In the implementation, there must be some sort of demultiplexing done to separate incoming messages addressed to different sockets.

