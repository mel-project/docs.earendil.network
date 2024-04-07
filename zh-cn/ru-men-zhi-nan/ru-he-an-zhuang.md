# 安装指南

## 系统要求

* 客户端节点：
  * 至少 1 GB 的可用 RAM 和磁盘空间，用于编译程序
  * Windows 10、macOS 或 Linux
* 中继节点：
  * 一个公网 IP 地址，用于服务客户端。云服务器、VPS、专用服务器等基本都有。
  * 至少 1 GB 的可用 RAM 和磁盘空间。
  * 只有 Linux 经过测试可用，但任何运行 Rust 的平台都很有可能可以使用

## 图形用户界面（GUI）

### Windows 和 Mac

1. 从我们的[发布页面](https://github.com/mel-project/earendil/releases)下载*适用于您平台*的最新可执行文件。

2. 解压您刚下载的可执行文件。
   - 在 Windows 上，您可以通过右键点击文件选择“全部提取...”，而在 macOS 上，您只需双击文件。

3. 双击可执行文件以启动程序！

{% hint style="info" %}
如果您的可执行文件无法工作，首先请确保您下载了正确的适用于您平台的文件！如果这仍然不起作用，请来我们的 [Discord](https://discord.gg/AVsGbhzTzx) 寻求帮助。
{% endhint %}

### Linux

您需要最新版本的[Rust](https://www.rust-lang.org/tools/install)，包含 `cargo` 和 `rustup` 等工具在您的 $PATH 中。目前 Earendil 没有官方的二进制发行版，所以我们将从源代码编译。

```
cargo install earendil-gui
```

## 命令行

在终端中，通过以下命令安装 `earendil`：

```shell-session
rustup update # 确保您的 Rust 是最新的
```

```shell-session
cargo install --locked earendil
```

通过以下命令检查 `earendil` 是否成功安装：

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
