# LowNet: low-level datagram transport

The lowest-level abstraction in Earendil is **LowNet**, an IP-like, best-effort datagram network.  
LowNet hides the physical topology of the Earendil relay graph and gives every node—relay _or_ client—an addressable, flat namespace.  

## Address format

LowNet uses addresses that look like the following

```
na-<relay-fingerprint>-<client-id>
```

| Field | Size | Meaning |
|-------|------|---------|
| `relay‐fingerprint` | 32 B (printed as 64 hex chars) | the `RelayFingerprint` (public key hash) of some relay |
| `client-id` | 8 B unsigned integer | `0` identifies the relay itself; any other value is a client slot behind that relay. |


For instance, `na-d4c5b1e6a0ff…ce09` refers to  client #42 behind the relay `d4c5b1e6a0ff…ce09`.


## Datagram structure

Every packet that crosses LowNet is a `Datagram`:

```rust
pub struct Datagram {
    pub ttl: u8,
    pub dest_addr: NodeAddr,
    pub payload: Bytes,
}
```

* **TTL** is decremented at every hop; frames with `ttl == 0` are dropped.  
* **Payload** is an opaque byte slice—mixing, congestion control and higher-level framing live at upper layers.
---

## Learn more

* 🗂 **Source code**: <https://github.com/mel-project/earendil/tree/new-refactor/libraries/earendil_lownet>