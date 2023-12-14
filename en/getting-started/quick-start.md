# Quick start

`earendil` is the reference implementation of Earendil, running as a background daemon similar to how `tor` runs as Tor's daemon.

This tutorial will teach you how to install `earendil`, run both a _client_ and a _relay_ node, and create a basic `earendil` config file.

## Install

First, make sure you have the latest [Rust](https://www.rust-lang.org/tools/install) installed and put in your $PATH.

In a terminal, install `earendil` by typing:

```shell-session
$ rustup update # to make sure your Rust is up to date
$ cargo install --git https://github.com/mel-project/earendil.git earendil
```

Check that `earendil` is successfully installed:

```shell-session
$ earendil
```

You should see:

```shell-session
Usage: earendil <COMMAND>

Commands:
  daemon         Runs an Earendil daemon
  control        Runs a control-protocol verb
  generate-seed  
  help           Print this message or the help of the given subcommand(s)

Options:
  -h, --help     Print help
  -V, --version  Print version
```

## Running a client

To run a client node, create a file named `config.yaml` and paste in the following:

```yaml
# relays to connect to
out_routes:
  example-relay:
    connect: 172.233.162.12:19999
    cookie: fa31361fe5597e14e22592cb98ef0f2eab5c62b3f38472331a2b6c9073991e07
    fingerprint: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz
    protocol: obfsudp
```

Now run

```bash!
earendil daemon --config config.yaml
```

You should see output like this:

```shell-session
[2023-11-29T20:03:11Z INFO  earendil] about to init daemon!
[2023-11-29T20:03:11Z INFO  earendil::daemon] starting background task for main_daemon
[2023-11-29T20:03:11Z INFO  earendil::daemon] daemon starting with fingerprint 4dewxch9f0y2zwja65qhwa62bppdn2zm
[2023-11-29T20:03:11Z DEBUG earendil::daemon::gossip] skipping gossip due to no neighs
[2023-11-29T20:03:11Z DEBUG earendil::daemon::inout_route] obfsudp out_route example-relay trying...
[2023-11-29T20:03:11Z DEBUG earendil::socket::n2r_socket] 0 packets queued up
[2023-11-29T20:03:11Z DEBUG earendil::socket::n2r_socket] 0 packets queued up
[2023-11-29T20:03:16Z DEBUG earendil::daemon::gossip] skipping gossip due to no neighs
```

Congratulations! You've successfully started an Earendil client node.

## Config file

Next, let's understand how the config file works. Earendil has two kinds of nodes: clients and relays. From the [Network architecture section](https://docs.earendil.network/wiki/architecture):

* **Relays** form the backbone of the Earendil network and relay messages between their neighbors: nodes that are directly connected to this relay.
* **Clients** do not relay any traffic, and they access the network with the help of relays. None of their neighbors can be other clients.

Correspondingly, clients and relays have different config files. The defining difference is: relay configs have an `in-routes` section that specifies where and how to accept incoming connections, while client configs do not.

### Client

Here is the example client config file from above:

```yaml
# relays to connect to
out_routes:
  example-relay: # arbitrary name for this relay
    fingerprint: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz # relay's long-term identity fingerprint
    protocol: obfsudp # transport protocol to use for connecting to relay
    connect: 172.233.162.12:19999 # IP and port where the relay is listening
    cookie: fa31361fe5597e14e22592cb98ef0f2eab5c62b3f38472331a2b6c9073991e07 # cookie for making an obfsudp connection
```

With this config file, our client only connects to one relay. To add a second relay, we put another relay information block under the `out_routes` section:

```yaml
# relays to connect to
out_routes:
  example-relay:
    fingerprint: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz
    protocol: obfsudp
    connect: 172.233.162.12:19999
    cookie: fa31361fe5597e14e22592cb98ef0f2eab5c62b3f38472331a2b6c9073991e07
  relay-2:
    fingerprint: ...
    protocol: obfsudp
    connect: ...
    cookie: ...
```

Currently, `obfsudp` (an obfuscated UDP transport) is the only supported transport protocol in Earendil.

Clients obtain relays' contact information out of band--by chat, email, or any means necessary. This information is not by default public to ensure _ban-resistance_: if any client can request a complete list of relays' IP addresses, then a censor can simply join the network, request all relays' contact information, and blacklist all the relays' IP addresses. You can read more about how earendil achieves ban-resistance in the [About](https://docs.earendil.network/) section.

Thus, unlike with `example-relay` above, you'll need to ask around for access to more relays. Come to [our Discord](https://discord.gg/jdVuk4Qj89) for more relays!

### Relay

Here's an example relay config file:

```yaml
# random *SECRET* seed for persistent Earendil identity
# Generate your own with "earendil generate-seed"
identity_seed: <your_random_seed>
# relay settings
in_routes:
  main_udp: # random name for this in-route
    protocol: obfsudp # transport protocol used for this in-route
    listen: 0.0.0.0:19999 # port where this in-route listens
    secret: <your_random_seed> # random seed for obfsudp cookie. Generate your own with "earendil generate-seed"

# neighbors, same as in the client config
out_routes:
  example-relay:
    fingerprint: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz
    protocol: obfsudp
    connect: 172.233.162.12:19999
    cookie: fa31361fe5597e14e22592cb98ef0f2eab5c62b3f38472331a2b6c9073991e07
```

`identity_seed` is an optional string that seeds a persistent Earendil identity. When this field is not specified, a random identity is generated every time `earendil` restarts.

* **Relays** must specify `identity_seed` in their config files, as they need to maintain a persistent identity for clients to connect to.
* **Clients** generally do not specify `identity_seed`, since they have no long-term identity on the Earendil network.

## 1+ nodes on 1 machine

You can interact with a running `earendil` daemon with `earendil control` commands. Check the list of `control` commands with:

```bash
$ earendil control
Runs a control-protocol verb

Usage: earendil control [OPTIONS] <COMMAND>

Commands:
  bind-n2r               Binds to a N2rSocket
  bind-haven             Binds to a HavenSocket
  skt-info               Prints the fingerprint and dock of a socket
  havens-info            Prints the information of all hosted havens
  send-msg               Sends a message using a given socket to a destination
  recv-msg               Blocks until a message is received
  global-rpc             Send a GlobalRpc request to a destination
  insert-rendezvous      Insert a rendezvous haven locator into the dht
  get-rendezvous         Looks up a rendezvous haven locator
  rendezvous-haven-test  Insert and get a randomly generated HavenLocator
  graph-dump             Dumps the graph
  my-routes              Dumps my own routes
  help                   Print this message or the help of the given subcommand(s)

Options:
  -c, --connect <CONNECT>  [default: 127.0.0.1:18964]
  -h, --help               Print help

```

To run multiple Earendil nodes on the same machine, we need to override the default port where the daemon listens for `control` commands for the additional nodes, by putting a `control_listen` section in their config files:

```yaml
control_listen: 127.0.0.1:11111
identity_seed: ...
out_routes: ...
```

Say we put the above in `bob-cfg.yaml`; to send `control` commands to bob, we now need to add the flag `--connect 127.0.0.1:11111`. For example:&#x20;

```
earendil control --connect 127.0.0.1:11111 my-routes
```

Be sure to use a different port for each additional node!

## Running a relay

To run a relay node, start the `earendil` daemon with your relay config:

```
earendil daemon --config relay-cfg.yaml
```

To obtain your relay's contact information for clients, make sure the `earendil` daemon is running with the correct config file and use the control command `my-routes`:

```shell-session
$ earendil control my-routes
```

The output should look like:

```yaml
main_udp:
  connect: <YOUR_IP>:19999
  cookie: 5ab28ed28be06cae621664a18c995a2b06c1f49ac426aa01541b45cbb81b7c38
  fingerprint: kz9typ7nwvyx9w17v8r9nhrzvebmmszc
  protocol: obfsudp
```

Clients simply paste this block, with `<YOUR_IP>` replaced by your server's public IP address, into the `out_routes` section of their config file to add your relay as a neighbor.

{% hint style="info" %}
As a part of replay attack protection, relays using `obfsudp` only start accepting connections after they have been running for at least 60 seconds. To disable this, set the environment variable`SOSISTAB2_NO_SLEEP` to `1` when you start the `earendil` daemon. You should _not_ disable this for production relays.
{% endhint %}

To serve users in regions with internet censorship, you should avoid posting your relay's contact information publicly. Instead, distribute it in a way that reaches legitimate users but not censors--your relay will be blacklisted if the censor learns its IP address.

## Inspecting the relay graph

Now that you're connected to the network, you can inspect the graph of Earendil relays from your node's perspective using:

<pre><code><strong>earendil control graph-dump
</strong></code></pre>

This produces Graphviz code that looks like:

```
digraph G {
    subgraph cluster_0 {
        color=lightblue;
        label="myself       [relay]";
        node [shape=Mdiamond,color=lightblue,style=filled];
        "7vcnrhf8dhjfyyc4djn255sbe3jj3g6r"
    }
    subgraph cluster_1 {
        color=lightpink
        label="my neighbors";
        node [color=lightpink,style=filled]
        "xh6tsvafwbgd2wc9akckc8pdwbv4arqw"
    }
    "7vcnrhf8dhjfyyc4djn255sbe3jj3g6r" [label="7vcn..3g6r"]
    "xh6tsvafwbgd2wc9akckc8pdwbv4arqw" [label="xh6t..arqw"]
    "bk6c3s76dzs8exna5vpcxwy92p49v0qt" [label="bk6c..v0qt"]
    "wbnm5bhq7hr6jwgnjqtd50zryebccq43" [label="wbnm..cq43"]
    
    "7vcnrhf8dhjfyyc4djn255sbe3jj3g6r" -> "xh6tsvafwbgd2wc9akckc8pdwbv4arqw";
    "bk6c3s76dzs8exna5vpcxwy92p49v0qt" -> "wbnm5bhq7hr6jwgnjqtd50zryebccq43";
    "wbnm5bhq7hr6jwgnjqtd50zryebccq43" -> "xh6tsvafwbgd2wc9akckc8pdwbv4arqw";
}
```

Paste the output into a [Graphviz renderer](https://dreampuf.github.io/GraphvizOnline/) to visualize the graph:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

You are the blue node, your immediate neighbors are pink, and all the other relays are in white. Since only information about relays is gossiped to the entire network, no clients other than yourself (if you're a client node) can appear in this graph.

`graph-dump` also has a human readable mode:

<pre class="language-bash"><code class="lang-bash"><strong>$ earendil control graph-dump -h
</strong>My fingerprint:
7vcnrhf8dhjfyyc4djn255sbe3jj3g6r    [relay]

My neighbors:
"xh6tsvafwbgd2wc9akckc8pdwbv4arqw"

Relay graph:
"7vcnrhf8dhjfyyc4djn255sbe3jj3g6r" -- "xh6tsvafwbgd2wc9akckc8pdwbv4arqw"
"bk6c3s76dzs8exna5vpcxwy92p49v0qt" -- "wbnm5bhq7hr6jwgnjqtd50zryebccq43"
"wbnm5bhq7hr6jwgnjqtd50zryebccq43" -- "xh6tsvafwbgd2wc9akckc8pdwbv4arqw"
</code></pre>

Learning about other nodes on the network takes time, so your node will not know the complete relay graph when it first starts. Depending on how many relays are in the network, you may have to wait a couple minutes to several hours for the graph to stabilize.

## Next up

Now you know how to run basic client and relay nodes, plus how to inspect the relay graph! The next two tutorials will teach you the two most fundamental features of Earendil:

* Visting and hosting Earendil [**havens**](using-havens.md): Internet services (like websites) hosted anonymously on the Earendil network
* Using the Internet anonymously by proxying it through an Earendil-based [**web-proxy**](browsing-web.md).
