# 运行网页代理

Earendil 客户端和中继节点均可运营网页代理。当您托管一个 Earendil 网页代理时，您选择分享代理信息的客户端可以使用您的节点作为出口节点匿名浏览明网流量。

![](../../en/.gitbook/assets/host-proxy.png)

{% hint style="danger" %}
**运营网页代理的安全隐患**

目前，我们还不支持对通过您托管的网页代理的用户进行身份验证或对流量进行过滤。

这意味着，使用您架设的代理的用户可以做很多「坏事」，比如：

- 访问本地网络资源，包括 localhost 上的资源
- 无节制地消耗您的带宽
- 发送垃圾邮件，并可能导致您的 IP 地址被您的互联网服务提供商封锁

因此，除非您确切知道自己在做什么，否则目前**不建议通过 Earendil 设置公共网页代理**。

未来，我们将添加对用户身份验证和流量过滤的支持，这将使运行公共 simple_proxy 服务变得更加可行。
{% endhint %}

要托管网页代理，请将此配置文件粘贴到您的 Earendil 图形界面的 "Settings" 标签中。请确保将 "/your/path/" 替换为适当的路径：

```yaml
state_cache: /your/path/.cache/earendil # 存储持久信息的位置。必须是绝对路径
out_routes: # 要连接的中继
  example-relay: # 此中继的任意名称
    connect: 62.210.93.59:12345 # 中继监听的 IP 和端口
    fingerprint: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a # 中继的长期身份
    obfs: # 使用的混淆协议
      sosistab3: "randomly-generated-cookie-lala-doodoo" # 混淆秘密，由中继随机生成

havens: # 我们托管的避风港
  - identity_file: /your/path/identity.secret # 替换为一个可写入的路径用于存储身份秘钥
    listen_port: 19999 # 代理服务器监听的端口
    rendezvous: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a # 选择作为洋葱路由会合点的中继
    handler:
      type: simple_proxy # Earendil 的网页代理协议
```

请注意，托管网页代理必然会暴露您的 IP 地址，因为客户可以连接到您的代理并访问 IP 检查网站。

由于网页代理不能有任何匿名性，所以您选择哪个会合中继（`rendezvous`）并不重要。我们建议选择您的直接邻居作为会合点，以获得最佳性能。在这个示例中，我们继续使用我们在所有教程中使用的公开中继。

启动 Earendil。您可以在 "Dashboard" 标签中找到您新设置的网页代理的信息：

![](../../en/.gitbook/assets/gui-proxy-haven.png)

现在，任何拥有此信息的 Earendil 节点都可以使用您刚设置的网页代理匿名浏览互联网，只需在他们的 Earendil 配置文件中包含此块：

```yaml
socks5:
  listen: 127.0.0.1:23456
  fallback:
    simple_proxy:
      remote: <您的代理指纹>:19999
```
