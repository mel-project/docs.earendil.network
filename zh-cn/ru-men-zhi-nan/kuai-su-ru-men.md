# 快速入门

`earendil` 是 Earendil 的参考实现，它作为后台守护进程运行，与 `tor` 作为 Tor 守护进程的运行方式类似。

本教程将指导您如何安装 `earendil`，运行客户端和中继节点，并创建一个基础的 `earendil` 配置文件。

## 安装

首先，请确保您已安装最新版本的 [Rust](https://www.rust-lang.org/tools/install) 并将其添加到您的 $PATH 环境变量中。

在终端中，输入以下命令来安装 `earendil`：

```shell-session
rustup update # 确保 Rust 是最新版本
```

```shell-session
cargo install --git https://github.com/mel-project/earendil.git earendil
```

检查 `earendil` 是否安装成功：

```shell-session
earendil
```

您应该看到以下输出：

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

## 运行客户端

运行客户端节点需要创建一个名为 `config.yaml` 的文件，并粘贴以下内容：

```yaml
# 要连接的中继
out_routes:
  example-relay:
    connect: 172.233.162.12:19999
    cookie: fa31361fe5597e14e22592cb98ef0f2eab5c62b3f38472331a2b6c9073991e07
    fingerprint: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz
    protocol: obfsudp
```

现在运行：

```bash
earendil daemon --config config.yaml
```

您应该看到类似这样的输出：

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

恭喜您！您已成功启动了一个 Earendil 客户端节点。

## 配置文件

上面我们制作了一个配制文件，这里简单介绍一下配置文件的原理。

Earendil 有两种类型的节点：客户端和中继。从[网络架构部分](https://docs.earendil.network/wiki/architecture)可以了解到：

- **中继** 构成了 Earendil 网络的骨干，它们在相邻节点之间转发消息：直接连接到此中继的节点。
- **客户端** 不转发任何流量，它们在中继的帮助下访问网络。它们的相邻节点不能是其他客户端。大部分用户运行的是客户端。

相应地，客户端和中继有不同的配置文件。区别在于：中继配置有一个 `in-routes` 部分，指定了如何接受传入连接的位置和方式，而客户端配置则没有。

### 客户端

这是上面的示例客户端配置文件：

```yaml
# 要连接的中继
out_routes:
  example-relay: # 这个中继的任意名称
    fingerprint: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz # 中继的长期身份指纹
    protocol: obfsudp # 用于连接中继的传输协议
    connect: 172.233.162.12:19999 # 中继监听的 IP 和端口
    cookie: fa31361fe5597e14e22592cb98ef0f2eab5c62b3f38472331a2b6c9073991e07 # 建立 obfsudp 连接的 cookie
```

使用这个配置文件，我们的客户端只连接到一个中继。要添加第二个中继，我们在 `out_routes` 部分下放置另一个中继信息块：

```yaml
# 要连接的中继
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

目前，`obfsudp`（一种混淆的 UDP 传输）是 Earendil 中唯一支持的传输协议。

{% hint style="warning" %}
**安全获取中继信息**

上面的配置使用 Mel 团队维护的**公开示例中继**。

需要注意的是，在生产环境中，_Earendil 中继信息通常不会公开_。需要通过聊天、电子邮件或线下方式，亲自认识中继运营者才能获取联系信息。

这是为了确保**抗封锁**：如果任何客户端都可以简单地请求中继信息，攻击者就可以加入网络获取中继列表，这可能让他们阻止或识别 Earendil 流量。（如果您熟悉抗 GFW 的「翻墙机场」，这个理由类似为什么翻墙节点的订阅地址必须保密）

因此，如果您想确保抗封锁，不要使用我们上面给出的中继！您可以来到[我们的 Discord](https://discord.gg/jdVuk4Qj89) 寻求其他用户的帮助，打听他们运营的中继。
{% endhint %}

### 中继

这是一个示例中继配置文件：

```yaml
# 随机的 *秘密* 种子，用于持久化 Earendil 身份
# 使用 "earendil generate-seed" 生成您自己的种子
identity_seed: <your_random_seed>
# 中继设置
in_routes:
  main_udp: # 这个 in-route 的随机名称
    protocol: obfsudp # 这个 in-route 使用的传输协议
    listen: 0.0.0.0:19999 # 这个 in-route 监听的端口
    secret: <your_random_seed> # obfsudp cookie 的随机种子。使用 "earendil generate-seed" 生成您自己的种子

# 邻居，与客户端配置相同
out_routes:
  example-relay:
    fingerprint: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz
    protocol: obfsudp
    connect: 172.233.162.12:19999
    cookie: fa31361fe5597e14e22592cb98ef0f2eab5c62b3f38472331a2b6c9073991e07
```

`identity_seed` 是一个可选字符串，用于生成持久的 Earendil 身份。如果未指定此字段，每次 `earendil` 重启时都会生成一个随机身份。

- **中继** 必须在其配置文件中指定 `identity_seed`，因为它们需要维护一个持久的身份以供客户端连接。
- **客户端** 通常不指定 `identity_seed`，因为它们在 Earendil 网络上没有长期身份。

## 单机多节点

您可以使用 `earendil control` 命令与正在运行的 `earendil` 守护进程进行连接，看目前守护进程上发生的各种事情。使用以下命令检查 `control` 命令列表：

```bash
earendil control
```

```
运行控制协议动词

Usage: earendil control [OPTIONS] <COMMAND>

Commands:
  bind-n2r               绑定到 N2rSocket
  bind-haven             绑定到 HavenSocket
  skt-info               打印 socket 的指纹和 dock
  havens-info            打印所有托管 haven 的信息
  send-msg               使用给定的 socket 向目的地发送消息
  recv-msg               阻塞直到收到消息
  global-rpc             向目的地发送 GlobalRpc 请求
  insert-rendezvous      将 rendezvous haven 定位器插入到 dht 中
  get-rendezvous         查找 rendezvous haven 定位器
  rendezvous-haven-test  插入并获取随机生成的 HavenLocator
  graph-dump             转储图表
  my-routes              转储我的路由
  help                   打印此消息或给定子命令的帮助

Options:
  -c, --connect <CONNECT>  [default: 127.0.0.1:18964]
  -h, --help               打印帮助
```

要在同一台机器上运行多个 Earendil 节点，我们需要通过在它们的配置文件中添加 `control_listen` 部分来覆盖守护进程监听 `control` 命令的默认端口：

```yaml
control_listen: 127.0.0.1:11111
identity_seed: ...
out_routes: ...
```

假设我们将上述内容放在 `bob-cfg.yaml` 中；要向 bob 发送 `control` 命令，我们现在需要添加标志 `--connect 127.0.0.1:11111`。例如：

```
earendil control --connect 127.0.0.1:11111 my-routes
```

确保为每个额外的节点使用不同的端口！

## 运行中继

要运行中继节点，请使用您的中继配置启动 `earendil` 守护进程：

```
earendil daemon --config relay-cfg.yaml
```

要获取您的中继联系信息以供客户端使用，请确保 `earendil` 守护进程正在使用正确的配置文件运行，并使用控制命令 `my-routes`：

```shell-session
earendil control my-routes
```

输出应该如下所示：

```yaml
main_udp:
  connect: <YOUR_IP>:19999
  cookie: 5ab28ed28be06cae621664a18c995a2b06c1f49ac426aa01541b45cbb81b7c38
  fingerprint: kz9typ7nwvyx9w17v8r9nhrzvebmmszc
  protocol: obfsudp
```

客户端只需将此块粘贴到其配置文件的 `out_routes` 部分，将 `<YOUR_IP>` 替换为您服务器的公网 IP 地址，即可添加您的中继作为邻居。

{% hint style="info" %}
作为重放攻击防护的一部分，使用 `obfsudp` 的中继只有在运行至少 60 秒后才开始接受连接。要禁用此功能，请在启动 `earendil` 守护进程时将环境变量 `SOSISTAB2_NO_SLEEP` 设置为 `1`。您不应该为生产中继禁用此功能。
{% endhint %}

为了服务于互联网审查地区的用户，您应该避免公开发布您的中继联系信息。相反，应以一种方式分发它，以便合法用户而非审查者能够接触到——如果审查者了解到其 IP 地址，您的中继将被列入黑名单。

## 检查中继图

现在您已经连接到网络，您可以使用以下命令从您节点的视角检查 Earendil 中继图：

<pre><code><strong>earendil control graph-dump
</strong></code></pre>

这将产生 Graphviz 代码，如下所示：

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

将输出粘贴到 [Graphviz 渲染器](https://dreampuf.github.io/GraphvizOnline/) 中以可视化图表：

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

您是蓝色节点，您的直接邻居是粉红色，所有其他中继都是白色。由于只有关于中继的信息被传播到整个网络，所以除了您自己（如果您是客户端节点），没有其他客户端可以出现在这个图中。

`graph-dump` 还有一个人类可读模式：

<pre class="language-bash"><code class="lang-bash"><strong>earendil control graph-dump -h
</strong>我的指纹：
7vcnrhf8dhjfyyc4djn255sbe3jj3g6r    [中继]

我的邻居：
"xh6tsvafwbgd2wc9akckc8pdwbv4arqw"

中继图：
"7vcnrhf8dhjfyyc4djn255sbe3jj3g6r" -- "xh6tsvafwbgd2wc9akckc8pdwbv4arqw"
"bk6c3s76dzs8exna5vpcxwy92p49v0qt" -- "wbnm5bhq7hr6jwgnjqtd50zryebccq43"
"wbnm5bhq7hr6jwgnjqtd50zryebccq43" -- "xh6tsvafwbgd2wc9akckc8pdwbv4arqw"
</code></pre>

了解网络上的其他节点需要时间，所以当您的节点首次启动时，它不会知道完整的中继图。根据网络中的中继数量，您可能需要等待几分钟到
