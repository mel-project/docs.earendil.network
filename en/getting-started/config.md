# Config file reference

Here's a fully annotated config file:

```yaml
# [optional] path to database; must be a path writable by the `earendil` daemon. If this key is not specified, then `earendil` uses a default path. This means that if you start more than one `earendil` daemon on the same machine, you must specify this field in all but one of the daemon configs in order to prevent the additional daemons from trying to use the same database and crashing.
db_path: ./.cache/earendil

# [optional] IP address where the daemon listens for control commands. If this key is not specified, then `earendil` listens for control commands on a default port. This means that if you start more than one `earendil` daemon on the same machine, you must specify this field in all but one of the daemon configs in order to prevent the additional daemons from trying to listen on the same port and crashing.
# Currently sending control commands to remote daemons is not supported, so this should be `127.0.0.1:<free port>`.
control_listen: 127.0.0.1:11111

# ------------------------ routing config ----------------------------
# relays to connect to as neighbors. Client configs *must* contain at least one `out_route`; optional for relays.
out_routes:
  example-relay:
    # IP address and port where the relay is listening for incoming connections
    connect: 45.33.109.28:12345
    # long-term identity of the relay
    fingerprint: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a
    # obfuscation protocol to use, for resisting ISP-level censorship
    # There are currently 2 obfuscation options:
    # - `none`: no obfuscation. In a censored network environment, this may lead to your connection to this relay getting blocked.
    # - `sosistab3`: a TCP-based obfuscated transport with a symmetric cookie, defined by the relay. This obfuscation protocol is developed as a part of [geph5](https://github.com/geph-official/geph5)
    obfs:
      sosistab3: shove-mistake-wish-endless-antique-citizen-filter-employ-cigar-clip-acid-defense
    # price and debt config for this link
    price_config:
      # how much you charge per incoming packet, in nanoMELs
      inbound_price: 0
      # debt limit for inbound packets, in nanoMELs
      inbound_debt_limit: 0
      # max price you're willing to pay per outgoing packet, in nanoMELs
      outbound_max_price: 10
      # min debt limit you accept for outbound packets, in nanoMELs
      # Negative debt limit = requires prepayment
      outbound_min_debt_limit: -100
  # more relays to connect to
  relay-2:
    connect: ...
    fingerprint: ...
    obfs: none
    price_config: ...

# -------------------- payments + Mel blockchain access -----------------------
payment_methods:
  on_chain:
    # secret of melwallet to use for sending + receiving payments
    secret: <your-mel-wallet-secret>
  # no arguments required to support PoW payments
  pow
# [optional] how to connect to the Mel blockchain, can be Earendil haven address
# If this key is not specified, then we connect to the Mel blockchain using the default bootstrap node over clearnet. This may not work in countries with internet censorship.
mel_bootstrap: <address-to-melnode>

# -------------------------------- havens + proxy --------------------------------
# [optional] starts a local Socks5 server that proxies traffic through Earendil. This gives you access to Earendil havens. If this key is not specified, then `earendil` starts a Socks5 proxy on **port 30003** with `fallback: pass_through`. This means that if you start more than one `earendil` daemon on the same machine, you must specify this field in all but one of the daemon configs in order to prevent the additional daemons from trying to listen on the same port and crashing.
socks5:
  # localhost address where the earendil Socks5 proxy listens
  listen: 127.0.0.1:23456
  # how to handle non-Earendil traffic (like a request to tunnel `google.com:443`). There are 3 options:
  # 1) `pass_through`: let all non-Earendil traffic through as if you're not using Earendil. Requests to `google.com` will behave the same way as if you weren't connected to the Earendil proxy.
  # 2) `block`: block all non-Earendil traffic. Requests to `google.com` will fail.
  # 3) `simple_proxy`: proxy non-Earendil traffic via a specified `simple-proxy` server, similar to how you use Tor as a web proxy.
  fallback:
    simple_proxy:
      # simple proxy server we're using
      remote: v5k6rydpg9yh9hft6c7qwz9sm3z99ytt:23456

# havens we're hosting
havens:
  ## a TCP haven, e.g. a website
  # path to file storing long-term haven identity. Must be writable to earendil daemon
  - identity_file: /your/path/identity.secret
    # fingerprint of relay chosen as rendezvous point for this haven (keeps haven anonymous to visitors)
    rendezvous: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a
    handler:
      type: tcp_service
      # dock where this TCP haven is hosted
      listen_dock: 12345
      # TCP address to forward all traffic for this haven to. The TCP service behind this haven (e.g., website) should be listening to this address.
      upstream: 127.0.0.1:8000

  ## a web proxy haven
  # path to file storing long-term haven identity. Must be writable to earendil daemon
  - identity_file:
      /your/path/identity.secret
      # relay chosen as our rendezvous point. Web proxy havens cannot be anonymous, so this relay should be chosen to optimize performance.
    rendezvous: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a
    handler:
      # Earendil's web proxy protocol
      type: simple_proxy
      # dock where proxy server listens
      listen_dock: 19999

# -------------------------- relay-only ----------------------------
relay_config:
  # path to file for storing a long-term identity.
  identity_file: /your/path/identity.secret

  # where & how to accept incoming connections
  in_routes:
    main_obfs:
      # obfuscation protocol to use, for resisting ISP-level censorship
      obfs:
        sosistab3: snake-before-antenna-toward-floor-stuff-frozen-power-avocado-retire-grunt-nation
      # TCP port this in_route listens at
      listen: 0.0.0.0:19999
      # price config for this route, in nanoMELs
      price_config:
        inbound_price: 5
        inbound_debt_limit: 50000
        outbound_max_price: 0
        outbound_min_debt_limit: 0
    # another in_route, with no obfuscation
    no_obfs:
      obfs: none
      listen: 0.0.0.0:19998
      price_config:
        inbound_price: 3
        inbound_debt_limit: 30000
        outbound_max_price: 0
        outbound_min_debt_limit: 0
```
