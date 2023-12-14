# Earendil packet vs. Sphinx

### Packet header

* Similarities
  * Each layer is encrypted and authenticated by a stream cipher generated from the hash of the shared secret produced by a Diffie-Hellman key exchange using an ephemeral keypair from the sender and a long-term keypair from the other mix node
* Differences
  * Unlike us, Sphinx does not store a whole bunch of sender's ephemeral public keys, once for each layer, used for onion encryption inside the packet header. Instead, the sender exploits the structure of Diffie-Hellman to generate a list of keypairs `(sk_1, pk_1), ..., (sk_n, pk_n)` such that knowing `(pk_i)`, plus the `i`th shared secret, allows you to derive `pk_(i+1)`. Each hop on the mix route then derives the next sender ephemeral public key, overwriting that field in the header before forwarding.
  * As such, Sphinx must use Diffie-Hellman to derive the shared secret used in making the onion header, while Earendil can be modified to use anything that has the same API as Diffie-Hellman (e.g. some sort of post-quantum scheme)
  * First hop in Earendil route is the sender themselves; first hop in Sphinx corresponds to second hop in Earendil

### Forward message body

* Sphinx: body is encrypted with a block cipher, whose block size coincides with the message size, keyed with hash of shared secret for each hop on the route. A block cipher is used so that any corruption of the encrypted data makes the entire message irrecoverable
* Earendil: same, except we use a `ChaCha20` stream cipher, which is faster. We handle integrity protection by layering on a MAC (included in the ChaCha20-Poly1305 algorithm used to encrypt the message body in `InnerPacket::seal()`)

### Reply blocks

* Same everywhere, except Sphinx includes a symmetric key in reply blocks to use for encrypting the message body. Earendil does not need this because the first hop in Earendil is the sending node itself, and the dencryption (XOR with stream cipher) done during the processing of the message body encrypts the body

### Message processing by mix nodes

* Sphinx
  * Compute shared secret `s`
  * Perform replay protection by comparing a hash of `s` to a table of seen message tags
  * Check the MAC
  * Pads + decrypts the `header_body` to obtain the next destination
    * IF FORWARD: Prepares the onion header for the next hop, decrypts the body with the hash of our shared secret, and sends the packet off
    * IF RECEIVE MESSAGE: decrypts the body, get the plaintext msg, send off to dest
* Earendil does all the above, except that we has not yet implemented replay protection
  * Note: replay protection is security-critical for mixnets. If a mixnet does not have replay protection, an attacker can duplicate packets of interest going into a mix node and see which next hop receives 2 packets to identify where the packet of interest travels
