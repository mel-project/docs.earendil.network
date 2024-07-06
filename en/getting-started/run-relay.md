# Run a relay

We currently only support running relays using the CLI version. Relays should be run on machines with public IP addresses.

Relays and clients nodes use the same `earendil` executable. The defining difference is in their config file: relay configs have a `relay_config` section that specifies `identity_file` (to store the relay's long-term identity) and `in_routes` (where and how to accept incoming connections), while client configs do not.

To run a relay, save this config file into a file named `relay-cfg.yaml`. Be sure to replace "/your/path/\` with an appropriate path:

```yaml
# neighbors, same as in client config
out_routes:
  example-relay-free:
    connect: 62.210.93.59:12345
    fingerprint: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a
    obfs:
      sosistab3: "randomly-generated-cookie-lala-doodoo"
    price_config:
      inbound_price: 0
      inbound_debt_limit: 0
      outbound_max_price: 0
      outbound_min_debt_limit: 0

# relay-only settings
relay_config:
  # replace with a writable path for storing identity secret
  identity_file: /your/path/earendil-relay-id.secret

  in_routes:
    main_udp:
      obfs:
        # random seed for obfsudp cookie. Generate your own with `earendil generate-seed`
        sosistab3: <your_random_seed>
      # port where this in-route listens
      listen: 0.0.0.0:19999
      # price, debt limit etc. for this in-route
      price_config:
        inbound_price: 0
        inbound_debt_limit: 0
        outbound_max_price: 0
        outbound_min_debt_limit: 0
```

You can learn about paying and getting paid on the Earendil network, as well as the `price_config` [here](pay.md).

Start the `earendil` daemon using this relay config:

```
earendil daemon --config relay-cfg.yaml
```

While the `earendil` daemon is running, obtain your relay's contact information for other nodes to connect with you as a neighbor with the control command `my-routes`:

```shell-session
earendil control my-routes
```

The output should look like:

```yaml
main_udp:
  connect: <YOUR_IP>:19999
  fingerprint: 57a407e50c1f4d0cdfb16332f6a836b27cc3409941fa26d85bc2b1eca604e536
  obfs:
    sosistab3: <your_random_seed>
  price_config:
    inbound_price: 0
    inbound_debt_limit: 0
    outbound_max_price: 0
    outbound_min_debt_limit: 0
```

Replace `<YOUR_IP>` with your server's public IP address. Other nodes (both clients and relays) can simply paste this block into the `out_routes` section of their config file to add your relay as a neighbor.

{% hint style="warning" %}
To serve users in regions with internet censorship, you should _avoid_ posting your relay's contact information publicly. Instead, distribute it in a way that reaches legitimate users but not censors--your relay will be blacklisted if the censor learns its IP address!
{% endhint %}
