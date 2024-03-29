# 快速入门

`earendil` 是 Earendil 的参考实现，它作为后台守护进程运行，与 `tor` 作为 Tor 守护进程的运行方式类似。

本教程将教您如何安装 `earendil`，运行**客户端**和**中继**节点，以及创建一个最基本的 `earendil` 配置文件。这样，我们就有基础下一步学习托管[避风港](bi-feng-gang-havens.md)和[代理普通互联网流量](jian-yi-wang-ye-dai-li.md)。


* **中继** 构成了 Earendil 网络的骨干。它们为网络上的其他节点提供传递消息的服务。
* **客户端** 不转发任何流量，它们只能在中继的帮助下访问网络。它们的相邻节点不能是其他客户端。

您可以在 wiki 的[网络架构部分](https://docs.earendil.network/wiki/architecture)更深入了解 Earendil 的架构。

### 运行客户端节点
1. 保存此配置文件：

<pre class="language-yaml"><code class="lang-yaml"><strong># Earendil 客户端配置文件
</strong>
db_path: ./.cache/earendil # 存储持久信息的位置
# 要连接的中继
out_routes:
  example-relay: # 此中继的任意名称
    fingerprint: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz # 中继的长期身份指纹
    protocol: obfsudp # 用于连接中继的传输协议
    connect: 172.233.162.12:19999 # 中继监听的 IP 和端口
    cookie: fa31361fe5597e14e22592cb98ef0f2eab5c62b3f38472331a2b6c9073991e07 # 用于建立 obfsudp 连接的 cookie
</code></pre>

- 如果您使用的是 **命令行界面（CLI）** ：将其保存为名为 `config.yaml` 的文件
- 如果您使用的是 **图形界面（GUI）**：将其粘贴到 “Settings” 标签中

  ![](../../en/.gitbook/assets/image.png)

2. 使用此配置运行 `earendil`:
   - **命令行界面（CLI）**：
     ```bash
     earendil daemon --config config.yaml
     ```
   - **图形界面（GUI）**：点击应用底部托盘中的 "Start" 按钮，位于屏幕左下方g

您应该看到如下日志输出：

```shell-session
[2023-11-29T20:03:11Z INFO  earendil] 准备初始化守护进程！
[2023-11-29T20:03:11Z INFO  earendil::daemon] 为主守护进程启动后台任务
[2023-11-29T20:03:11Z INFO  earendil::daemon] 守护进程启动，指纹为 4dewxch9f0y2zwja65qhwa62bppdn2zm
[2023-11-29T20:03:11Z DEBUG earendil::daemon::gossip] 由于没有邻居，跳过 gossip
[2023-11-29T20:03:11Z DEBUG earendil::daemon::inout_route] obfsudp 出口路线 example-relay 正在尝试...
[2023-11-29T20:03:11Z DEBUG earendil::socket::n2r_socket] 0 个数据包在排队
[2023-11-29T20:03:11Z DEBUG earendil::socket::n2r_socket] 0 个数据包在排队
[2023-11-29T20:03:16Z DEBUG earendil::daemon::gossip] 由于没有邻居，跳过 gossip
```

{% hint style="info" %}
图形界面（GUI）：前往顶栏中的 "日志" 标签查看日志！

![](../../en/.gitbook/assets/gui-logs.png)
{% endhint %}

{% hint style="info" %}
目前，`obfsudp`（一种混淆的 UDP 传输）是 Earendil 中唯一支持的传输协议。
{% endhint %}

{% hint style="warning" %}
**安全获取中继信息**

上面的配置使用 Mel 团队维护的**公开示例中继**。

需要注意的是，在生产环境中，_Earendil 中继信息通常不会公开_。需要通过聊天、电子邮件或线下方式，亲自认识中继运营者才能获取联系信息。

这是为了确保**抗封锁**：如果任何客户端都可以简单地请求中继信息，攻击者就可以加入网络获取中继列表，这可能让他们阻止或识别 Earendil 流量。（如果您熟悉抗 GFW 的「翻墙机场」，这个理由类似为什么翻墙节点的订阅地址必须保密）

因此，如果您想确保抗封锁，不要使用我们上面给出的中继！您可以来到[我们的 Discord](https://discord.gg/jdVuk4Qj89) 寻求其他用户的帮助，打听他们运营的中继。
{% endhint %}

恭喜您！您已成功启动了一个 Earendil 客户端节点。

### 运行中继节点
目前我们只支持使用命令行界面（CLI）运行中继。
1. 将此配置文件保存为名为 `relay-cfg.yaml` 的文件：

```yaml
# Earendil 中继配置文件
db_path: ./.cache/earendil # 存储持久信息的位置
identity_file: /your/path/identity.secret # 替换为一个可写入的路径，用于存储身份秘钥
# 中继设置
in_routes:
  main_udp: # 此入口路线的任意名称
    protocol: obfsudp # 此入口路线使用的传输协议
    listen: 0.0.0.0:19999 # 此入口路线监听的端口
    secret: <your_random_seed> # obfsudp cookie 的随机种子。使用 "earendil generate-seed" 生成您自己的种子

# 邻居，与客户端配置相同
out_routes:
  example-relay:
    fingerprint: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz
    protocol: obfsudp
    connect: 172.233.162.12:19999
    cookie: fa31361fe5597e14e22592cb98ef0f2eab5c62b3f38472331a2b6c9073991e07
```

2. 使用此中继配置启动 `earendil` 守护进程：

```
earendil daemon --config relay-cfg.yaml
```

3. 在 `earendil` 守护进程运行时，使用控制命令 `my-routes` 获取您的中继节点联系信息，用于其他节点连接您的中继，和您成为相邻节点：

```shell-session
earendil control my-routes
```

输出应该像这样：

```yaml
main_udp:
  connect: <YOUR_IP>:19999
  cookie: 5ab28ed28be06cae621664a18c995a2b06c1f49ac426aa01541b45cbb81b7c38
  fingerprint: kz9typ7nwvyx9w17v8r9nhrzvebmmszc
  protocol: obfsudp
```

请将 `<YOUR_IP>` 替换为您服务器的公网 IP 地址。其他节点（客户端和中继都一样）只需将此块内容粘贴到他们的配置文件的 `out_routes` 部分，并，即可添加您的中继作为邻居。

{% hint style="info" %}
作为重放攻击防护的一部分，使用 `obfsudp` 的中继只有在运行至少 60 秒后才开始接受连接。要禁用此功能，请在启动 `earendil` 守护进程时将环境变量 `SOSISTAB2_NO_SLEEP` 设置为 `1`。您不应该为生产中继禁用此功能。
{% endhint %}

为了服务于互联网审查地区的用户，您应该避免公开发布您的中继联系信息。相反，应以一种方式分发它，以便合法用户而非审查者能够接触到——如果审查者了解到其 IP 地址，您的中继将被列入黑名单。

## 配置文件

以下是一些关于配置文件的实用信息。

### 中继节点与客户端？

中继配置和客户端配置的区别在于：中继配置有一个 `in-routes` 部分，指定了如何接受传入连接的位置和方式，而客户端配置则没有。因此，具有 `in-routes` 部分的配置文件是中继配置，没有的则是客户端配置。

### 添加更多邻居

这是上面示例中的客户端配置文件：

```yaml
db_path: ./.cache/earendil # 存储持久信息的位置
# 要连接的中继
out_routes:
  example-relay: # 这个中继的任意名称
    fingerprint: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz # 中继的长期身份指纹
    protocol: obfsudp # 用于连接中继的传输协议
    connect: 172.233.162.12:19999 # 中继监听的 IP 和端口
    cookie: fa31361fe5597e14e22592cb98ef0f2eab5c62b3f38472331a2b6c9073991e07 # 建立 obfsudp 连接的 cookie
```

使用这个配置文件时，我们的客户端只连接到一个中继。如果要添加第二个中继，我们需要在 `out_routes` 部分下放置另一个中继信息块：

```yaml
db_path: ./.cache/earendil # 存储持久信息的位置
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


### `identity_file`

这是上面示例中的中继配置文件：

```yaml
# Earendil 中继配置文件
db_path: ./.cache/earendil # 存储持久信息的位置
# 中继设置
identity_file: /your/path/identity.secret # 替换为一个可写入的路径用于存储身份秘钥
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

`identity_file` 是用于存储持久 Earendil 身份的可选文件路径。如果未指定此字段，每次 `earendil` 重新启动时都会生成一个随机身份。

* **中继** 必须在其配置文件中指定 `identity_file`，因为它们需要维护一个持久的身份以供客户端连接。
* **客户端** 通常不指定 `identity_file`，因为它们在 Earendil 网络上没有长期身份。

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
identity_file: ...
out_routes: ...
```

假设我们将上述内容放在 `bob-cfg.yaml` 中；要向 bob 发送 `control` 命令，我们现在需要添加参数 `--connect 127.0.0.1:11111`。例如：

```
earendil control --connect 127.0.0.1:11111 my-routes
```

确保为每个额外的节点使用不同的端口！


## 检查中继图

{% hint style="info" %}
图形界面（GUI）：启动您的节点，然后前往 “Dashboard” 标签查看中继图！

![](../../en/.gitbook/assets/gui-graph-dump.png)
{% endhint %}

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

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

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

了解网络中其他节点需要时间，因此您的节点在刚刚启动时并不会了解完整的中继图。根据网络中的中继数量，图表稳定下来可能会需要几分钟到几个小时。

## 接下来

现在您已经了解了如何运行基本的客户端和中继节点，以及如何检查中继图！接下来的两个教程将教您 Earendil 的两个最基本特性：&#x20;

* 访问和托管 Earendil [**避风港**](https://docs.earendil.network/v/zh-cn/ru-men-zhi-nan/bi-feng-gang-havens)，即在 Earendil 网络上匿名托管的互联网服务（如网站）&#x20;
* 通过 Earendil 上的[**网络代理**](https://docs.earendil.network/v/zh-cn/ru-men-zhi-nan/jian-yi-wang-ye-dai-li)匿名使用互联网。
