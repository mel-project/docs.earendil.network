---
description: RPC protocol that any node can call any node
---

# GlobalRPC

GlobalRPC is the primary RPC protocol exposed by all relays, that any node can call. It runs over the node-to-relay protocol (N2R).

Like all RPC protocols in Earendil, it is based on JSON-RPC 2.0.

In the following, the node hosting the RPC is called the **server**, while the node calling the RPC is called the **client**. This terminology has no bearing on their overall network roles as relay, client, haven, etc.

## Request/response transport

### Requests

Every GlobalRPC request is encoded verbatim into a N2R messages addressed to **dock 100001**. The source dock number is arbitrary and can be allocated per-request.

For instance, a call `ping(12345)` will simply be a N2R message containing something like the following string:

```json
{
  "jsonrpc": "2.0",
  "id": 31415926,
  "method": "ping",
  "params": [12345]
}
```

Some important notes:

* Requests that do not fit into one message are not supported.
* `id` must be unique for each request.
* Clients _may_ choose to retransmit requests that have not received a response in a while. The retransmission must contain the same `id` as the original transmission!
* Clients _may_ have multiple requests in-flight to the same destination at the same time.

### Responses

Responses are single N2R messages sent to the source fingerprint and dock number that the request comes from. For instance, the response to the above request will be:

```json
{
  "jsonrpc": "2.0",
  "id": 31415926,
  "result": 12345
}
```

Note that:

* Responses that do not fit into one message are not supported.
* If multiple retransmissions of the same request arrive within 60 seconds, they should receive identical responses, while the logic they trigger must not run more than once. To support this, servers must maintain a cache of at least the last 60 seconds, mapping `id` to the response.
* Clients that support multiple inflight requests must use the `id` field to match responses to requests. The server may respond to the inflight requests in any order.
* Clients must silently discard responses that do not correspond to any inflight request.

## Supported methods

{% hint style="info" %}
We are in the process of finalizing the specific methods of GlobalRPC, so this may be updated in the future
{% endhint %}

```rust
pub trait GlobalRpcProtocol {
    async fn ping(&self, i: u64) -> u64;

    async fn dht_insert(&self, locator: HavenLocator, recurse: bool) -> Result<(), DhtError>;

    async fn dht_get(
        &self,
        key: Fingerprint,
        recurse: bool,
    ) -> Result<Option<HavenLocator>, DhtError>;
    
    /// Called by a haven server to register itself with a rendezvous node
    async fn alloc_forward(&self, forward_req: RegisterHavenReq) -> Result<(), VerifyError>;
}
```
