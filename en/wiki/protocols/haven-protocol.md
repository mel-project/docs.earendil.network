# Haven protocol

## Overview

The **haven rendezvous protocol** allows Alice and Bob to communicate without either of them being a relay or revealing their location in the network topology.

The haven protocol is built upon network-wide client-relay communication and requires first implementing the [GlobalRPC](globalrpc.md) protocol. It has two main pieces:

* The **rendezvous DHT**, which maps each **haven** (publicly reachable anonymous endpoints with stable, well-known public keys, such as "darknet" websites) to their **rendezvous relay** (relay that handles traffic to and from that haven)
* The **forwarding protocol** that coordinates traffic to flow user<->relay<->haven.

## Rendezvous DHT

The rendezvous DHT maps haven fingerprints to **haven locators.** A haven locator is a document, signed by the haven's public identity key, containing:

* the haven's full public identity key
* the haven's medium-term onion key
* the fingerprint of the rendezvous relay

This is through this process:

* Every relay has RPC methods `get_dht` and `insert_dht` that can retrieve and insert keys stored locally at that relay. These keys expire after a short period of time (e.g. 10 minutes)
* Rendezvous hashing, computed over `(key, hour)`, where `hour` is the number of hours since the Unix epoch, assigns 3 relays to be responsible for a particular DHT key at a given point in time.
* Havens continually insert their own fingerprint->locator mapping into nodes. They insert them not only in the 3 relays of the current hour, but also the previous and next hour.
* When looking up locators, clients simply ask any of the 3 relays that `(key, current_hour)` map to.

{% hint style="info" %}
We use 3 relays rather than 1 for fault tolerance.

Changing the relays every hour is also for fault tolerance, so that keys that are unluckily assigned to bad relays get rotated away from them.

Inserting into three adjacent hours further increases fault-tolerance, as well as mitigating issues related to slightly out-of-sync clocks.
{% endhint %}

## Forwarding protocol

In this example, Bob is a haven, Rob is the rendezvous relay, and Alice is a client who talks to it.

There is one RPC method exposed in the global RPC: `alloc_forward` which takes in a signed request from a particular fingerprint and instructs the relay to process forwarding packets to this particular fingerprint. Bob must call this method before any of the following happens:

* Alice sends N2R messages to Rob to a special, well-known dock (**dock number 100002**). The messages are of the format `(bob_endpoint, inner)`. The inner part is separately end-to-end encrypted between Alice and Bob, using the same packet format as the [N2R protocol](n2r-protocol.md), but without the padding and onion encryption.
* Rob sends to Bob the messages with the inner part verbatim, tagged as `(inner, bob_endpoint)`.
* Bob sends messages to Alice by sending messages of the format `(inner, alice_endpoint)` to Rob. Rob's internal state is something like a NAT table --- it allows anyone to send messages to Bob, and it allows Bob to send messages to anybody who has talked to Bob, but it refuses to forward any other messages, so it's not an open proxy. Rob identifies "backwards" messages because they are coming from Bob, somebody who has registered forwarding with Rob.

## Socket abstraction

Similar to sockets in the [N2R protocol](n2r-protocol.md), we have a socket abstraction for haven communication.

Both havens and clients who wish to talk to havens use the same protocol:

* _Bind_ which binds to a local dock, but only optionally a _haven host decriptor_.
  * Without an descriptor, a random identity is generated. This socket thus is a client socket, only suitable for sending messages to havens and receiving messages from them.
  * The descriptor includes info like:
    * The identity keypair of the haven (must be unlinkable to the node identity)
    * The fingerprint of the appointed rendezvous relay
* _Send_ has as its destination a fingerprint and a dock number.
  * For client sockets, we assume this fingerprint identifies a haven we want to talk to. We look up the fingerprint in the DHT to find its rendezvous relay, and send it a properly formatted N2R message that will get routed to the haven we want to talk to.
  * For haven sockets, we assume that this identifies some endpoint who has talked to our haven in the past. We send the appropriately formatted forward-protocol message to our own rendezvous relay
* _Receive_ returns a message, a source fingerprint, and a source dock number.
  * For both client and haven sockets, we simply listen for incoming packets over N2R (at the identity and dock number indicated in _Bind_), and decode out the "true" source and contents as forwarding-packet messages.

Haven-protocol (N2H) sockets are easiest to implement simply by containing an N2R socket inside.

## Haven encryption

Haven messages are protected with an additional layer of end-to-end authenticated encryption. Unlike encryption elsewhere in Earendil, this encryption is _stateful_ and session-based.

First, note that in havens we can make a distinction between the client and the haven --- havens cannot directly talk to each other. Let's say Alice is the client and Bob is a haven she talks to.

### Initial handshake

Before the first time Alice talks to Bob, Alice generates an ephemeral x25519 keypair, and sends to Bob a message that looks like this:

* 8 bytes: `0xffffffffffffffff` (indicating that this is a handshake packet)
* 32 bytes: Alice's identity public key
* 32 bytes: Alice's ephemeral x25519 public key
* 32 bytes: signature

Bob also generates an ephemeral x25519 keypair, just for communication with Alice. He replies with

* 8 bytes: `0xffffffffffffffff` (indicating that this is a handshake packet)
* 32 bytes: Bob's identity public key
* 32 bytes: Bob's ephemeral x25519 public key
* 32 bytes: signature

After this, Bob and Alice have a shared secret `shared_secret`. We can now derive "upload" and "download" symmetric keys:

* `up_key = keyed_hash(hash("haven-up"), shared_secret)`
* `dn_key = keyed_hash(hash("haven-dn"), shared_secret)`

### After the handshake

Messages are encrypted using ChaCha20-Poly1305, in the following format:

* 8 bytes: incrementing 64-bit nonce, little-endian (pad to 96 bits before using)
* n bytes: ChaCha20-Poly1305 message

Both parties must reject messages with duplicate nonces.

Bob here knows which messages come from Alice by looking at the source fingerprint and dock (tunneled through the rendezvous point).

### Closing sessions

Sessions are never explicitly closed. Instead, their state should be forgotten if not used for more than a reasonable timeout value, at least 10 minutes.

Alice should renegotiate the session at least every 30 minutes, plus if it's been more than 1 minute since she's _heard back from_ Bob, to ensure forward secrecy and allow recovery if Bob forgets state.
