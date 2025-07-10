# Protocols

Earendil presents a pretty simple interface (basically a virtual IPv6-like network device with some convenience methods), but under the hood is many layers of protocols.

This is illustrated by the following picture. **Note that A->B means A depends on B**, not that A somehow sends data unidirectionally to B!

```mermaid
flowchart TB
 subgraph network["Network layer"]
        socket["Unified socket abstraction"]
        haven["Haven protocol"]
        dht["Rendezvous DHT"]
        forward["Haven forwarding"]
        rpc["GlobalRPC"]
        mixnet["Mixnet protocol"]
        onion["Onion routing"]
  end
 subgraph lownet["LowNet"]
        link["Link transport"]
  end
    socket --> haven
    haven --> dht & forward
    dht --> rpc
    forward --> mixnet
    rpc --> mixnet
    mixnet --> onion
    socks["SOCKS5 interface"] --> sosistab["Optionally reliable streams (sosistab2)"]
    tcp["TCP port forwarding"] --> sosistab
    sosistab --> socket
    tun["tun-based VPN interface"] --> socket
    onion --> link & routegraph[("Route graph")]
```
