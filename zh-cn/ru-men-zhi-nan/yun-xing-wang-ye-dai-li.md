# 简易网页代理

与 Tor 不同，Earendil 并没有内置浏览明网 Web 的功能。没有现成的方法可以像使用 Tor 那样，不经设置通过 Earendil 作为综合「翻墙工具」。

但是，在 Earendil 中托管和使用**网页代理避风港**非常简单。这些避风港可以由您托管并与其他用户共享，他们可以使用您的避风港作为网页代理来浏览普通互联网。

<figure><img src="../../en/.gitbook/assets/image (4).png" alt=""><figcaption><p>通过 Earendil 代理 Web 流量</p></figcaption></figure>

本教程将指导您如何操作。

## 将 Earendil 用作网页代理

要连接到 Earendil 网页代理，请首先在您的 `earendil` 配置文件中添加以下 SOCKS5 配置块：

```yaml
socks5:
  listen: 127.0.0.1:23456 # 代理服务器监听的本地端口
  fallback:
    simple_proxy: # 所有明网流量的代理服务器
      remote: 4j4wedcnst83v6xttabdppzj7261sccg:23456
```

这与在[上一个教程](bi-feng-gang-havens.md)中设置 `earendil` 访问避风港的方法完全相同，不同之处在于我们现在使用 `simple_proxy` 选项来处理明网流量。我们将代理的指纹和 dock 号放在远程端点（`remote`）中；这里是我们为本教程设置的公共代理。

现在，设置您的浏览器使用 localhost:23456 作为 SOCKS5 代理。像平常一样访问任何网站，所有流量都应该通过 Earendil！您可以通过[检查](https://bgp.he.net/)您的 IP 地址来确认这一点：如果它是 `172.233.162.12`，则说明您已正确连接。

## 托管网页代理避风港

与托管其他类型的 Earendil 避风港一样，客户端和中继都可以托管网页代理避风港。但是，请注意，这样做意味着暴露您的 IP 地址：由于客户端始终可以连接到您的代理并访问 IP 检查网站，因此没有办法隐藏您的 IP 地址。

{% hint style="danger" %}
**托管代理避风港的安全隐患**

目前，我们还不支持对通过您托管的网页代理的用户进行身份验证或对流量进行过滤。

这意味着，使用您架设的代理的用户可以做很多「坏事」，比如：

* 访问本地网络资源，包括 localhost 上的资源
* 无节制地消耗您的带宽
* 发送垃圾邮件，并可能导致您的 IP 地址被您的互联网服务提供商封锁

因此，除非您确切知道自己在做什么，否则目前**不建议通过 Earendil 设置公共网页代理**。

未来，我们将添加对用户身份验证和流量过滤的支持，这将使运行公共 simple\_proxy 服务变得更加可行。
{% endhint %}

要托管网页代理避风港：

1.  在您的 earendil 配置文件中添加以下 `havens` 配置块：

    ```yaml
    # 我们托管的避风港
    havens:
      - identity_file: /your/path/identity.secret # 替换为一个可写入的路径用于存储身份秘钥
        rendezvous: ejqgx2g5jwe2mvjnzqbb6w1htmj9d2mz # 我们选择的会合中继
        handler:
          type: simple_proxy
          listen_dock: 12345 # 代理服务器监听的 dock
    ```

    由于网页代理避风港不能有任何匿名性，因此您选择哪个会合中继并不重要。我们建议选择您自己（如果您是中继）或您的直接邻居（如果您是客户端）作为会合点，以获得最佳性能。在这个例子中，我们继续使用我们在所有教程中使用的公共中继。
2.  最后，重启 earendil 守护进程以重新加载配置文件。现在您可以使用以下命令打印您的避风港地址：

    ```shell-session
    earendil control havens-info
    ```

    您应该看到类似这样的输出：

    ```
    SimpleProxy - x83aqrnq6awfp96qrjg5rmxr49d1bqlm:12345
    ```

    现在客户端可以通过在指定 SOCKS5 fallback 选项时将 `x83aqrnq6awfp96qrjg5rmxr49d1bqlm:12345` 作为 `remote` 来使用您的网页代理避风港了！
