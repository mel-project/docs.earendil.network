# Link protocol

The link protocol controls every node-node link in the Earendil network, both relay-relay links and relay-client links.

The basic outline is such:

* Every link is a `sosistab2` session, backed by one or more pipes (such as `obfsudp` pipes)
* Each end of a link can call "LinkRPC" methods on the other end, by using JSON-RPC over `sosistab2` streams. These streams have label `"n2n_control"`
* Onion-routed messages are passed through `sosistab2` unreliable messages. These messages are passed using streams with label `"onion_packets"`.

## LinkRPC

Each end of a link separately establishes one or more `sosistab2` streams labelled `"n2n_control"`. Over these streams, JSON-RPC requests are sent in a newline-delimitated manner analogous to JSON-RPC over TCP. For example, here is an example of the send and receive ends of a LinkRPC stream, while one end pings the other:

```json
--> {"jsonrpc":"2.0","method":"ping","params":[123],"id":1}
<-- {"jsonrpc":"2.0","result":123,"id":1}
```

LinkRPC methods are used for actions such as:

* Checking the liveness and performance of a link by "pinging" it
* Gossipping the relay graph
* Negotiating prices and keeping track of debt

We now discuss some of these functions in more detail.

## Relay graph gossip

The purpose of the relay graph gossip is to ensure that between two connected nodes Alice and Bob, they eventually agree on the shape of the relay graph. Since we assume that the network is ultimately globally connected, this means that _everyone_'s view is eventually consistent.

We use a simple pull-based approach. Both ends of every link expose this LinkRPC endpoint:

```
get_adjacencies(fingerprints: string[]) -> AdjacencyDescriptor[]
```

which takes a list of relay fingerprints, and returns all the adjacency descriptors where one end is one of the provided relay fingerprints.

Each side continually calls `get_adjacencies` on the other side, passing in a random subset of fingerprints they know about. The responses are then added to the local relay graph.

This ensures that relay graphs of both of the sides eventually stay in sync.
