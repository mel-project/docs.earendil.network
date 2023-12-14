# 避风港（Havens）

**避风港** 是类似于 [Tor 中的洋葱服务](https://community.torproject.org/onion-services/)的匿名托管服务。通过托管避风港，您可以提供 TCP 或 UDP 服务，如网站、IRC 服务器等。您和您的用户都将受到 Earendil 的匿名和抗封锁保护。

本教程将教您如何使用和托管基本的 Earendil 避风港。

## 使用避风港

您可以直接在浏览器中访问基于 HTTP 的避风港：

1. 在现有配置文件中，通过在 Earendil 配置文件中添加 SOCKS5 部分，将 Earendil 进程设置为 SOCKS5 代理。配置应该类似于这样：

   ```yaml
   socks5:
     listen_port: 23456 # 代理服务器监听的本地端口
     fallback: pass_through # 如何处理非 earendil 流量
   ```

2. `fallback` 指定如何处理非 Earendil 流量（如请求代理 `google.com:443`）。有 3 个选项：
   - `pass_through`：让所有非 Earendil 流量绕过，就像您没有使用 Earendil 一样。对 `google.com` 的请求将表现得就像您没有连接到 Earendil 代理一样。
   - `block`：阻挡所有非 Earendil 流量。对 `google.com` 的请求将失败。
   - `simple_proxy`：通过指定的 `simple-proxy` 服务器代理非 Earendil 流量，类似于您使用 Tor 作为网页代理。下一个教程将涵盖这种用例。
3. 现在，设置您的浏览器使用 `localhost:23456` 作为 SOCKS5 代理。Firefox 的设置如下所示：

   ![image](https://hackmd.io/_uploads/SkLZ828Sp.png)

4. 访问

   ```!
   http://qcmnt2mbchhanm7fzacybswzknbsw3zp.haven:12345
   ```

   就像访问任何普通网站一样。您应该会看到：

   ![image](https://hackmd.io/_uploads/rJMmF3LHT.png)

您刚刚就第一次成功地访问了 Earendil 避风港！有了这个设置，您可以访问任何您知道地址的 Earendil 避风港。

## 托管避风港

这里介绍一下如何将一个简单的 HTTP 网站作为避风港托管。

### 启动本地主机网页服务器

第一步是**设置一个在端口 8000 上监听的网页服务器**。在我们的示例中，我们将托管一个小型的 Nginx 网页服务器。

1. 如果尚未安装 Nginx，请安装它。在 Debian Linux 上，您可以输入 `sudo apt install nginx`，但这在不同的系统上可能有所不同。
2. 在 nginx 配置文件中（通常位于 `/etc/nginx/nginx.conf`），找到一个配置服务器监听 8000 端口的部分，并将其更改为以下示例：

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

4. 然后，启动您的 Nginx 服务器。在 Linux 上，使用 `systemctl start nginx`
5. 现在，您应该能够在 `localhost:8000` 看到您的 Nginx 网页服务器了！

### 设置避风港

接下来，为您的避风港生成一个独特的身份种子：

```shell-session
earendil generate-seed
```

现在，在您的中继配置文件中添加以下 `haven` 部分，将 `your_random_seed` 替换为第 2 步的输出：

```yaml
# 我们托管的避风港
havens:
  - identity_seed: <your_random_seed> # 避风港持久 Earendil 身份的随机种子
    rendezvous: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz # 我们选择的会合中继
    handler:
      type: tcp_forward
      from_dock: 12345
      to_port: 8000 # 网页服务器监听的端口
```

- `identity_seed`：这必须是一个**独特的、不可预测的值**。如上所述，`earendil generate-seed` 是获取一个的简便方法。
- `rendezvous` 是您选择的**会合中继**的指纹。这是一个中继节点，负责接收和转发所有意图传送给您避风港的消息，以便您的 IP 地址可以对避风港的客户端保持私密。所有避风港都必须有一个会合中继；您可以在[这里](https://docs.earendil.network/wiki/protocols/haven-protocol)阅读更多关于避风港协议架构的信息。目前，我们将使用我们在本教程中一直使用的同一个测试中继：`ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz`。
- `handler` 指定如何处理流向避风港的流量。这里，我们使用 TCP [端口转发](https://en.wikipedia.org/wiki/Port_forwarding)将所有避风港流量转发到端口 8000 上的网页服务器。

8. 重启 earendil 守护进程并使用以下命令打印出您的避风港地址：

```shell-session
earendil control havens-info
```

您应该看到类似这样的内容：

```
TcpForward - qcmnt2mbchhanm7fzacybswzknbsw3zp:12345
```

这告诉您，一个在端口 `12345` 上监听的 `tcp_forward` 避风港具有指纹 qcmnt2mbchhanm7fzacybswzknbsw3zp。

客户端现在可以在 `qcmnt2mbchhanm7fzacybswzknbsw3zp.haven:12345` 找到您的避风港了！
