# 配置文件参考

以下是一个完整注释了的配置文件：

```yaml
# [可选] 数据库的路径；必须可由 `earendil` 守护进程写入。
# 如果未指定此键，`earendil` 将使用默认路径。
# 如果在同一台机器上启动多个 `earendil` 守护进程，除了一个守护进程配置外，必须在所有其他守护进程配置中指定此字段。
# 这可防止额外的守护进程尝试使用相同的数据库并崩溃。
db_path: ./.cache/earendil

# [可选] 守护进程监听控制命令的 IP 地址。
# 如果未指定此键，`earendil` 将在默认端口监听控制命令。
# 如果在同一台机器上启动多个 `earendil` 守护进程，除了一个守护进程配置外，必须在所有其他守护进程配置中指定此字段。
# 这可防止额外的守护进程尝试在相同的端口上监听并崩溃。
# 目前不支持向远程守护进程发送控制命令，因此这应该是 `127.0.0.1:<空闲端口>`。
control_listen: 127.0.0.1:11111

# ------------------------ 路由配置 ----------------------------
# 作为邻居连接的中继。客户端配置*必须*至少包含一个 `out_route`；对于中继是可选的。
out_routes:
  example-relay:
    # 中继监听传入连接的 IP 地址和端口
    connect: 62.210.93.59:12345
    # 中继的长期身份
    fingerprint: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a
    # 使用的混淆协议，用于抵抗 ISP 级别的审查
    # 目前有 2 个混淆选项：
    # - `none`：无混淆。在受审查的网络环境中，这可能导致您与此中继的连接被阻止。
    # - `sosistab3`：基于 TCP 的带有对称 cookie 的混淆传输，由中继定义。这个混淆协议是作为 [geph5](https://github.com/geph-official/geph5) 的一部分开发的
    obfs:
      sosistab3: shove-mistake-wish-endless-antique-citizen-filter-employ-cigar-clip-acid-defense
    # 此链接的价格和债务配置
    price_config:
      # 每个传入数据包的收费，以 µMEL 为单位
      inbound_price: 0
      # 传入数据包的债务限制，以 µMEL 为单位
      inbound_debt_limit: 0
      # 您愿意为每个传出数据包支付的最高价格，以 µMEL 为单位
      outbound_max_price: 10
      # 您接受的传出数据包的最小债务限制，以 µMEL 为单位
      # 负债务限制意味着需要预付款
      outbound_min_debt_limit: -100
  # 更多要连接的中继
  relay-2:
    connect: ...
    fingerprint: ...
    obfs: none
    price_config: ...

# -------------------- 支付 + Mel 区块链访问 -----------------------
payment_methods:
  # 不需要参数即可支持 PoW 支付
  - pow
  - on_chain: <your-mel-wallet-secret> # 用于发送和接收付款的 melwallet 密钥

# [可选] 如何连接到 Mel 区块链，可以是 Earendil 避风港地址
# 如果未指定此键，则我们将使用默认引导节点通过明网连接到 Mel 区块链。这在有互联网审查的国家可能无法工作。
mel_bootstrap: <address-to-melnode>

# --------------------------- 避风港 + 代理 ---------------------------
# [可选] 启动一个本地 Socks5 服务器，通过 Earendil 代理流量。
# 这使您可以访问 Earendil 避风港。
# 如果未指定此键，`earendil` 将在 **端口 30003** 上启动一个 Socks5 代理，并设置 `fallback: pass_through`。
# 如果在同一台机器上启动多个 `earendil` 守护进程，除了一个守护进程配置外，必须在所有其他守护进程配置中指定此字段。
# 这可防止额外的守护进程尝试在相同的端口上监听并崩溃。
socks5:
  # earendil Socks5 代理监听的本地主机地址
  listen: 127.0.0.1:23456
  # 如何处理非 Earendil 流量（如请求隧道 `google.com:443`）。有 3 个选项：
  # 1) `pass_through`：让所有非 Earendil 流量通过，就像您没有使用 Earendil 一样。对 `google.com` 的请求将表现得与您未连接到 Earendil 代理时相同。
  # 2) `block`：阻止所有非 Earendil 流量。对 `google.com` 的请求将失败。
  # 3) `simple_proxy`：通过指定的 `simple-proxy` 服务器代理非 Earendil 流量，类似于您使用 Tor 作为网络代理的方式。
  fallback:
    simple_proxy: passthrough

# 我们托管的避风港
havens:
  ## 一个 TCP 避风港，例如一个网站
  # 存储长期避风港身份的文件路径。必须可由 earendil 守护进程写入
  - identity_file: /your/path/identity.secret
    # 选择作为此避风港会合点的中继的指纹（使避风港对访问者保持匿名）
    rendezvous: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a
    handler:
      type: tcp_service
      # 托管此 TCP 避风港的码头
      listen_dock: 12345
      # 转发此避风港所有流量的 TCP 地址。此避风港后面的 TCP 服务（例如，网站）应监听此地址。
      upstream: 127.0.0.1:8000

  ## 一个网络代理避风港
  # 存储长期避风港身份的文件路径。必须可由 earendil 守护进程写入
  - identity_file:
      /your/path/identity.secret
      # 选择作为我们会合点的中继。网络代理避风港不能匿名，因此应选择此中继以优化性能。
    rendezvous: 4b7a641b77c2d6ceb8b3fecec2b2978dfe81ae045ed9a25ed78b828009c4967a
    handler:
      # Earendil 的网络代理协议
      type: simple_proxy
      # 代理服务器监听的码头
      listen_dock: 19999

# -------------------------- 仅中继 ----------------------------
relay_config:
  # 存储长期身份的文件路径。
  identity_file: /your/path/identity.secret

  # 接受传入连接的位置和方式
  in_routes:
    main_obfs:
      # 使用的混淆协议，用于抵抗 ISP 级别的审查
      obfs:
        sosistab3: snake-before-antenna-toward-floor-stuff-frozen-power-avocado-retire-grunt-nation
      # 此 in_route 监听的 TCP 端口
      listen: 0.0.0.0:19999
      # 此路由的价格配置，以 µMEL 为单位
      price_config:
        inbound_price: 5
        inbound_debt_limit: 50000
        outbound_max_price: 0
        outbound_min_debt_limit: 0
    # 另一个 in_route，无混淆
    no_obfs:
      obfs: none
      listen: 0.0.0.0:19998
      price_config:
        inbound_price: 3
        inbound_debt_limit: 30000
        outbound_max_price: 0
        outbound_min_debt_limit: 0
```
