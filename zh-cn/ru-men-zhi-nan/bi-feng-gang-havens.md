# 避风港（Havens）

**避风港** 是类似于 [Tor 中的洋葱服务](https://community.torproject.org/onion-services/)的匿名托管服务。通过托管一个避风港，您可以提供 TCP 服务，如网站、IRC 服务器等。您和您的用户都将受到 Earendil 的匿名和抗封锁保护。

本教程将教您如何使用和托管基础的 Earendil 避风港。

## 访问避风港

您可以直接在浏览器中访问基于 HTTP 的避风港。将以下配置文件粘贴到您的 Earendil GUI 的“设置”标签中。请确保将“/your/path/”替换为适当的路径：

```yaml
state_cache: /your/path/.cache/earendil # 用于存储持久信息的位置。必须是绝对路径
out_routes: # 要连接的中继
  example-relay: # 此中继的任意名称
    connect: 62.210.93.59:12345 # 中继监听的 IP 和端口
    fingerprint: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a # 中继的长期身份
    obfs: # 使用的混淆协议
      sosistab3: "randomly-generated-cookie-lala-doodoo" # 中继随机生成的混淆密钥
```

然后，设置您的浏览器使用 `localhost:30003` 作为 SOCKS5 代理。在 Firefox 里：

![image](https://hackmd.io/_uploads/SkLZ828Sp.png)

尝试访问：

```!
http://t90bt94h01ezd75zv9rtzam60thnbkvz.haven:12345
```

就像访问任何普通网站一样。您应该会看到：

![image](https://hackmd.io/_uploads/rJMmF3LHT.png)

您刚刚访问了您的第一个 Earendil 避风港！使用这个设置，您即可访问任何您知道地址的 Earendil 避风港。

{% hint style="info" %}
所有 Earendil 避风港网站都仅支持 HTTP，因为证书颁发机构通常不会为 `.haven` 域名发放证书。HTTPS 也是不必要的，因为 Earendil 流量已经是加密的。
{% endhint %}

## 托管避风港

作为托管避风港的介绍，我们来托管一个网站作为避风港。

### 启动本地主机网页服务器

首先，**设置一个在端口 8000 上监听的网页服务器**。在我们的示例中，我们将使用 Nginx。

1. 如果尚未安装 Nginx，请安装。
2. 在 nginx 配置文件中（通常位于 `/etc/nginx/nginx.conf`），找到一个配置服务器监听 8000 端口的部分，并将其更改为以下内容：

```
server {
    listen       8000;
    server_name  localhost;

    location / {
      root /usr/share/nginx/html;
      index index.html;
    }
}
```

3. 启动您的 Nginx 服务器。在 Linux 上：`systemctl start nginx`
4. 现在，您应该能够在 `localhost:8000` 上看到您的服务器了！

### 设置避风港

将此配置文件粘贴到您的 Earendil GUI 的“设置”标签中。确保将“/your/path/”替换为适当的路径：

```yaml
state_cache: /your/path/.cache/earendil # 用于存储持久信息的位置。必须是绝对路径

out_routes: # 要连接的中继
  example-relay: # 此中继的任意名称
    connect: 62.210.93.59:12345 # 中继监听的 IP 和端口
    fingerprint: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a # 中继的长期身份
    obfs: # 使用的混淆协议
      sosistab3: "randomly-generated-cookie-lala-doodoo" # 中继随机生成的混淆密钥

# 我们托管的避风港
havens:
  - identity_file: /your/path/identity.secret # 替换为一个可写入的路径，用于存储此避风港的身份秘钥
    listen_port: 12345
    rendezvous: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a # 作为洋葱路由会合点的中继选择
    handler:
      type: tcp_service
      upstream: 127.0.0.1:8000 # 网页服务器监听的位置
```

- `identity_file`：一个可写入的路径，用于存储您的避风港的身份秘钥。
- `rendezvous` 是您选择的**会合中继**的指纹。这是一个中继节点，负责接收和转发所有意图传送给您的避风港的消息，以便您的 IP 地址对避风港的客户端保持私密。所有避风港都必须有一个会合中继；您可以在[这里](https://docs.earendil.network/wiki/protocols/haven-protocol)阅读更多关于避风港协议架构的信息。在这个示例中，我们将使用我们在本教程中一直使用的同一个测试中继。
- `handler` 指定如何处理流向避风港的流量。这里，我们使用 TCP [端口转发](https://en.wikipedia.org/wiki/Port_forwarding)将所有避风港流量转发到端口 8000 上的网页服务器。

启动 Earendil，并在“Dashboard”标签中找到您的避风港地址：

![](../../en/.gitbook/assets/gui-tcp-haven.png)

{% hint style="info" %}
在 CLI 版本中，您可以使用以下命令获取您的避风港地址：

```shell-session
earendil control havens-info
```

您应该看到类似这样的内容：

```
TcpForward - qcmnt2mbchhanm7fzacybswzknbsw3zp:12345
```

{% endhint %}

在我们的示例中，`qcmnt2mbchhanm7fzacybswzknbsw3zp` 是您避风港的**指纹**，而 `12345` 是它的**端口号**（类似于 TCP 端口号）。

现在，人们可以通过 `http://<您的避风港指纹>.haven:<您的避风端口号>` 找到您的避风港了！
