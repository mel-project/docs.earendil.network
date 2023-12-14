# Onion packet format

The onion-packet has two, fixed-size parts:

* The **header** which includes authenticated-encrypted routing messages
* The **body** which is encrypted with layers of ChaCha20, with intentionally no integrity protection.

```
Total: 8,872 bytes
  ├─ Header: 680 bytes
  │    ├─ Box-encrypted routing info for first hop: 68 bytes
  │    └─ Onion-encrypted routing info for next hops: 612 bytes
  └─ Onion-encrypted body: 8,192 bytes
```

One important thing to note is that the encoding within the fixed-size body is not specified here, but it _must_ have some sort of _end-to-end_ integrity protection. The onion packet format will not help with that, at all.

## Box encryption

**Box encryption** (named after the similar construction from NaCl) is a generic way of encrypting a message, with integrity protection, so that only the owner of a particular X25519 secret key can read it.

The format is:

* 32 bytes: a X25519 ephemeral public key from the sender
* ?? bytes: encrypted message, with ChaCha20Poly1305 (RFC 8439), encrypted with the key derived from blake3-hashing the X25519 shared secret computed from the sender's ephemeral key and the X25519 key of the recipient, and an all-zero nonce.

## Header format

The header for a message whose first hop is "Rob" will have:

* 69 bytes: 21 bytes box-encrypted to Rob's X25519 public key. The first byte is a flag for how to interpret the next 20 bytes:
  * Byte 1 = 1 indicates we need to forward the message to the next hop. The next 20 bytes is the fingerprint for next hop.
  * Byte 1 = 0 indicates the packet is addressed to us.&#x20;
    * If the next 20 bytes are all zeros, it means this packet is an ungarbled normal packet.
    * Otherwise, it is a reply packet that is garbled through using a reply block. Bytes 1 through 8 (inclusive) is a 64-bit reply block identifier that we will use to pair this packet with the keys we stored from when we generated the reply block, which we will use to de-garble the message body.&#x20;
* 621 bytes: ChaCha20-encrypted (no authentication, no nonce) with `KDF(s, "header")` where `s` is the shared-secret between the sender and our X25519 keys.
  * The rest of the headers
  * As much randomness to pad the entire header to 690 bytes

The padding of the header to 690 bytes is done to keep the header size fixed, which helps with anonymity and prevents traffic analysis. The maximum route length is 10 hops, so subsequent peeled header is exactly 69 bytes smaller than the previous one, but with 69 bytes more random noise added to the back. This means the intermediary nodes have no idea where they are in the route.

## Body format

The body is ChaCha20-encrypted at every layer of the onion with `KDF(s, "body")`, and is always exactly 8192 bytes in length.

## Key-derivation function

The function `KDF` is implemented by using `blake3`'s keyed-hash mode, with the "key" being padded to 32 bytes with `_`. For instance, `KDF(s, "header")` means blake3-hashing `s` with the key `"header__________________________"`.
