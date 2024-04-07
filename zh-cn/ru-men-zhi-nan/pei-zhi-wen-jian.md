# 配置文件参考
以下是一个完整注释了的配置文件：

```yaml
# [可选] 用于存储跨 `earendil` 守护进程重启持久化的信息的文件路径（例如全局中继图和聊天历史）。这必须是 `earendil` 守护进程可写的路径。存储这些信息可能会加快初始连接时间。
db_path: ./.cache/earendil

# [仅中继需要，客户端忽略] 用于存储长期身份的文件路径。
identity_file: /your/path/identity.secret

# [可选] 守护进程监听控制命令的 IP 地址。如果未指定此键，则 `earendil` 将在默认端口监听控制命令。这意味着，如果您在同一台机器上启动多个 `earendil` 守护进程，您必须在除一个守护进程的配置文件中指定此字段，以防止额外的守护进程尝试监听同一端口并崩溃。
# 当前不支持向远程守护进程发送控制命令，所以这应该是 `127.0.0.1:<空闲端口>`。
control_listen: 127.0.0.1:11111

# 作为邻居连接的中继。客户端配置*必须*包含至少一个 `out_route`；中继可选。
out_routes:
  example-relay:
    # 中继监听传入连接的 IP 地址和端口
    connect: 45.33.109.28:12345 
    # 中继的长期身份
    fingerprint: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a
    # 使用的混淆协议，以抵抗 ISP 级别的审查
    # 当前有 2 个混淆选项：
    # - `none`：无混淆。在受审查的网络环境中，这可能导致您与此中继的连接被阻止。
    # - `sosistab3`：一个基于 TCP 的带有对称 cookie 的混淆传输，由中继定义。这个混淆协议是作为 [geph5](https://github.com/geph-official/geph5) 的一部分开发的
    obfs:
      sosistab3: shove-mistake-wish-endless-antique-citizen-filter-employ-cigar-clip-acid-defense
  # 连接更多中继
  relay-2: 
    connect: ...
    fingerprint: ...
    obfs: none

# 启动一个本地 Socks5 服务器，通过 Earendil 代理流量。这使您可以访问 Earendil 避风港。
socks5:
  # earendil Socks5 代理监听的 localhost 地址
  listen: 127.0.0.1:23456 
  # 如何处理非 Earendil 流量（如请求隧道 `google.com:443`）。有 3 个选项：
  # 1) `pass_through`：让所有非 Earendil 流量通过，就好像您没有使用 Earendil。对 `google.com` 的请求将表现得就像您没有连接到 Earendil 代理一样。
  # 2) `block`：阻止所有非 Earendil 流量。对 `google.com` 的请求将失败。
  # 3) `simple_proxy`：通过指定的 `simple-proxy` 服务器代理非 Earendil 流量，类似于您使用 Tor 作为网页代理。
  fallback:
    simple_proxy:
      # 我们使用的 simple proxy 服务器
      remote: v5k6rydpg9yh9hft6c7qwz9sm3z99ytt:23456

# 我们托管的避风港
havens:
## 一个 TCP 避风港，例如一个网站
  # 存储长期避风港身份的文件路径。必须对 earendil 守护进程可写
  - identity_file: /your/path/identity.secret
    # 选择作为此避风港会合点的中继的指纹（保持避风港对访问者匿名）
    rendezvous: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a 
    handler:
      type: tcp_service
      # 托管此 TCP 避风港的端口
      listen_dock: 12345
      # 转发此避风港所有流量的 TCP 地址。此避风港后面的 TCP 服务（例如，网站）应该监听此地址。
      upstream: 127.0.0.1:8000

## 一个网页代理避风港
  # 存储长期避风港身份的文件路径。必须对 earendil 守护进程可写
  - identity_file: /your/path/identity.secret
     # 选为我们会合点的中继。网页代理避风港不能匿名，所以应该选择此中继以优化性能。
    rendezvous: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a
    handler:
        # Earendil 的网页代理协议
        type: simple_proxy 
        # 代理服务器监听的端口
        listen_dock: 19999

# ------------------ 仅限中继 -------------------

# 接受传入连接的方式和位置
in_routes:
  main_obfs:
    # 使用的混淆协议，以抵抗 ISP 级别的审查
    obfs: 
      sosistab3: snake-before-antenna-toward-floor-stuff-frozen-power-avocado-retire-grunt-nation
    # 此 in_route 监听的 TCP 端口
    listen: 0.0.0.0:19999
  # 另一个不带混淆的 in_route
  no_obfs:
    obfs: none
    listen: 0.0.0.0:19998 
```