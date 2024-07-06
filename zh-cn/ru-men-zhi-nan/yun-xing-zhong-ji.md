# 运行中继节点

目前我们只支持使用命令行界面（CLI）版本运行中继节点。中继节点应该运行在具有公网 IP 地址的机器上。

中继和客户端节点使用相同的 `earendil` 可执行文件。区别在于他们的配置文件：中继配置有一个 `in-routes` 部分，指定了如何接受客户连接的位置和方式，而客户端配置则没有。

要运行中继节点，请将此配置文件保存为名为 `relay-cfg.yaml` 的文件。请确保将 "/your/path/" 替换为适当的路径：

```yaml
# Earendil 中继配置文件
state_cache: /your/path/.cache/earendil # 存储持久信息的位置。必须是绝对路径

# 邻居，与客户端配置相同
out_routes:
  example-relay:
    connect: 62.210.93.59:12345
    fingerprint: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a
    obfs:
      sosistab3: "randomly-generated-cookie-lala-doodoo"

# 中继设置
identity_file: /your/path/identity.secret # 替换为一个可写入的路径用于存储身份秘钥

in_routes:
  main_udp:
    obfs:
      sosistab3: <your_random_seed> # obfsudp cookie 的随机种子。使用 `earendil generate-seed` 生成您自己的种子
    listen: 0.0.0.0:19999 # 此入口路线监听的端口
```

使用此中继配置启动 `earendil` 守护进程：

```
earendil daemon --config relay-cfg.yaml
```

确保 `earendil` 守护进程运行的同时，使用控制命令 `my-routes` 获取您的中继节点联系信息，以便其他节点可以作为邻居节点与您连接：

```shell-session
earendil control my-routes
```

输出应该如下所示：

```yaml
main_udp:
  connect: <YOUR_IP>:19999
  fingerprint: 57a407e50c1f4d0cdfb16332f6a836b27cc3409941fa26d85bc2b1eca604e536
  obfs:
    sosistab3: <your_random_seed>
```

请将 `<YOUR_IP>` 替换为您服务器的公网 IP 地址。其他节点（客户端和中继都一样）可以地将这个块粘贴到他们的配置文件的 `out_routes` 部分，以添加您的中继作为邻居。

{% hint style="warning" %}
如果您想服务于有着互联网审查地区的用户，请避免公开发布您的中继联系信息。您应使用一种避免让审查者发现中继信息的方式把信息传递给真正的用户——如果审查者了解到您中继的 IP 地址，您的中继会被列入黑名单。
{% endhint %}
