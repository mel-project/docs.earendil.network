# Quick start

`earendil` is the reference implementation of Earendil, running as a background daemon similar to how `tor` runs as Tor's daemon.

This tutorial will teach you how to run both **client** and a **relay** Earendil nodes, as well as create a basic `earendil` config file. It will give you the background needed to learn about hosting [havens](../using-havens.md) and [proxying normal Internet traffic](../browsing-web.md).

- **Relays** form the backbone of the Earendil network. They serve other nodes on the network by relaying messages for them.
- **Clients** do not relay any traffic, and they access the network with the help of relays. They cannot be neighbors with other clients.

You can read more about Earendil's architecture in the wiki's [Network architecture](../wiki/architecture.md) section.
### Run a client node

1. Save this config file:

<pre class="language-yaml"><code class="lang-yaml"><strong># Earendil client config file
</strong># relays to connect to
out_routes:
  example-relay: # arbitrary name for this relay
    fingerprint: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz # relay's long-term identity fingerprint
    protocol: obfsudp # transport protocol to use for connecting to relay
    connect: 172.233.162.12:19999 # IP and port where the relay is listening
    cookie: fa31361fe5597e14e22592cb98ef0f2eab5c62b3f38472331a2b6c9073991e07 # cookie for making an obfsudp connection
</code></pre>

- If you're using the **CLI** version: save it into a file named `config.yaml`
- If you're using the **GUI**: paste it into the "Settings" tab

  ![](../../.gitbook/assets/image.png)

2. Run `earendil` with this config:
- **CLI**: 
  ```bash
  earendil daemon --config config.yaml
  ```
- **GUI**: clicking the "Start" button in the bottom tray of the app, toward the left side of the screen

You should see logs output like this:

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

{% hint style="info" %}
GUI: go to the "logs" tab in the top bar to see the logs!

![](../../.gitbook/assets/gui-logs.png)
{% endhint %}

{% hint style="info" %}
Currently, `obfsudp` (an obfuscated UDP transport) is the only supported transport protocol in Earendil.
{% endhint %}

{% hint style="warning" %}
**Obtaining relay information safely**

In the configuration above, we added a _publicly available_ example relay that the Mel team maintains.

It is important to note that in production, _Earendil relay information will not generally be publicly available_. You will need to personally know a relay operator to obtain contact information out-of-band, through chat, email, or offline.

This is to ensure **ban-resistance**: if any client can just request relay information, attackers can simply join the network to get a list of relays, which can let them block or identify Earendil traffic.

Thus, if you want to actually ensure ban-resistance, don't use the relay we gave you above! Instead, you can come to [our Discord](https://discord.gg/jdVuk4Qj89) to ask other users for help.
{% endhint %}

Congratulations! You've successfully started an Earendil client node.

### Run a relay node

We currently only support running relays using the CLI version.
1. Save this config file into a file named `relay-cfg.yaml`:

```yaml
# Earendil relay config file
identity_file: /your/path/identity.secret # replace with a writable path for storing identity secret
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

2. Start the `earendil` daemon using this relay config:

```
earendil daemon --config relay-cfg.yaml
```

3. While the `earendil` daemon is running, obtain your relay's contact information for other nodes to connect with you as a neighbor with the control command `my-routes`:

```shell-session
earendil control my-routes
```

The output should look like:

```yaml
main_udp:
  connect: <YOUR_IP>:19999
  cookie: 5ab28ed28be06cae621664a18c995a2b06c1f49ac426aa01541b45cbb81b7c38
  fingerprint: kz9typ7nwvyx9w17v8r9nhrzvebmmszc
  protocol: obfsudp
```

Replace `<YOUR_IP>` with your server's public IP address. Other nodes (both clients and relays) can simply paste this block into the `out_routes` section of their config file to add your relay as a neighbor.

{% hint style="info" %}
As a part of replay attack protection, relays using `obfsudp` only start accepting connections after they have been running for at least 60 seconds. To disable this, set the environment variable`SOSISTAB2_NO_SLEEP` to `1` when you start the `earendil` daemon. You should _not_ disable this for production relays.
{% endhint %}

To serve users in regions with internet censorship, you should _avoid_ posting your relay's contact information publicly. Instead, distribute it in a way that reaches legitimate users but not censors--your relay will be blacklisted if the censor learns its IP address.

## Config file

Here are a few more handy things to know about the config file.

### Relay vs client?

The defining difference between relay configs and client configs is: relay configs have an `in-routes` section that specifies where and how to accept incoming connections, while client configs do not. So config files that have an `in-routes` section are relay configs, and ones that do not are client configs.

### Adding more neighbors

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

With a config file like this, our node only connects to one neighbor (which for clients, has to be a relay). To add a second neighbor, we put another relay information block under the `out_routes` section:

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

### `identity_file`

Here's the example relay config file from above:

```yaml
# Earendil relay config file
identity_file: /your/path/identity.secret # replace with a writable path for storing identity secret
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

`identity_file` is an optional file path for storing a persistent Earendil identity. When this field is not specified, a random identity is generated every time `earendil` restarts.

* **Relays** must specify `identity_file` in their config files, as they need to maintain a persistent identity for clients to connect to.
* **Clients** generally do not specify `identity_file`, since they have no long-term identity on the Earendil network.



### `db_path`

In order to persist your identity, debts, chat history, and relay graph when your node shuts down, you need to specify a `db_path` in your config file, with a path to the db file, like so:

```yaml
db_path: ~/.cache/earendil
```

## 1+ nodes on 1 machine

You can interact with a running `earendil` daemon with `earendil control` commands. Check the list of `control` commands with:

```bash
earendil control
```

```
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
identity_file: ...
out_routes: ...
```

Say we put the above in `bob-cfg.yaml`; to send `control` commands to bob, we now need to add the flag `--connect 127.0.0.1:11111`. For example:

```
earendil control --connect 127.0.0.1:11111 my-routes
```

Be sure to use a different port for each additional node!

## Inspecting the relay graph

{% hint style="info" %}
GUI: start your node, then go to the "Dashboard" tab to see the relay graph!

![](../../.gitbook/assets/gui-graph-dump.png)
{% endhint %}


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

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

You are the blue node, your immediate neighbors are pink, and all the other relays are in white. Since only information about relays is gossiped to the entire network, no clients other than yourself (if you're a client node) can appear in this graph.

`graph-dump` also has a human readable mode:

<pre class="language-bash"><code class="lang-bash"><strong>earendil control graph-dump -h
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

* Visting and hosting Earendil [**havens**](../using-havens.md): Internet services (like websites) hosted anonymously on the Earendil network
* Using the Internet anonymously by proxying it through an Earendil-based [**web-proxy**](../browsing-web.md).
