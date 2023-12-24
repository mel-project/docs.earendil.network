# Browsing the Web

Unlike Tor, Earendil does not come with functionality for browsing the "normal" unobfuscated Web. There's no out-of-box way of, say, watching YouTube videos "through Earendil" the way you can do with Tor.

But it's quite easy to host and use **web proxy havens** in Earendil. These are havens that you can host and share with other users, who can then use your haven as a web proxy to browse the normal Internet.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption><p>Proxying Web traffic over Earendil</p></figcaption></figure>

This tutorial will teach you how.

## Using Earendil as a web proxy

To connect to an Earendil web proxy, first add this SOCKS5 block to your `earendil` config file:

```yaml
socks5:
  listen: 127.0.0.1:23456 # localhost port where the proxy server listens
  fallback:
    simple_proxy: # proxy server for all clearnet traffic
      remote: 4j4wedcnst83v6xttabdppzj7261sccg:23456
```

This is exactly like setting up `earendil` to access havens in the [previous tutorial](using-havens.md), except that we now use the `simple_proxy`option for handling clearnet traffic. We put the proxy's fingerprint and dock number in the remote endpoint (`remote`); here this is a public proxy we set up for this tutorial.

Now, set your browser to use localhost:23456 as a SOCKS5 proxy. Visit any website as you normally would, and all the traffic should be going through Earendil! You can confirm this by [checking](https://bgp.he.net/) your IP address: you're properly connected if it's `172.233.162.12`.

## Hosting web-proxy havens

As with other kinds of Earendil havens, both clients and relays can host web-proxy havens. Note, however, that doing so means exposing your IP address: there's simply no way to conceal your IP address, since a client can always connect to your proxy and go to an IP-checking website.

To host a web-proxy haven:

1.  Add this `havens` block to your earendil config file:

    ```yaml
    # havens we're hosting
    havens:
      - identity_file: identity_file: /your/path/identity.secret # replace with a writable path for storing identity secret
        rendezvous: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz # our chosen rendezvous relay
        handler:
          type: simple_proxy
          listen_dock: 12345 # dock where proxy server listens
    ```

    Since web-proxy havens cannot have any anonymity, it's not important which rendezvous relay you choose. We recommend choosing yourself (if you're a relay) or an immediate neighbor (if you're a client) as the rendezvous point for optimal performance. In this example, we continue to use the public relay we use in all the tutorials.
2.  Restart the earendil daemon to reload the configuration file. You can now print your haven's address with

    ```shell-session
    earendil control havens-info
    ```

    You should see output like:

    ```
    SimpleProxy - x83aqrnq6awfp96qrjg5rmxr49d1bqlm:12345
    ```

    Now clients can use your web-proxy haven by putting `x83aqrnq6awfp96qrjg5rmxr49d1bqlm:12345` as the `remote` when specifying `simple-proxy` as their SOCKS5 fallback option!
