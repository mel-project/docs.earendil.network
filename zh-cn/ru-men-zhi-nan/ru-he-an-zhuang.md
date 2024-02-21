# 如何安装

## 系统要求

* 最新版的 [Rust](https://www.rust-lang.org/tools/install) ，包括 `cargo` 和 `rustup` 等工具。Earendil 目前没有官方的二进制发行版，所以我们需要从源代码编译。
* 客户端节点：
  * 至少 1 GB 的可用 RAM 和磁盘空间，用于编译程序
  * Windows 10、macOS 或 Linux
* 中继节点：
  * 一个公网 IP 地址，用于服务客户端。云服务器、VPS、专用服务器等基本都有。
  * 至少 1 GB 的可用 RAM 和磁盘空间。
  * 只有 Linux 经过测试可用，但任何运行 Rust 的平台都很有可能可以使用


## 安装命令行版本

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

## 安装图形界面（GUI）版本

### Windows 和 Mac

1. 从我们的 [发布页面](https://github.com/mel-project/earendil/releases) 下载*适用于您平台*的最新可执行文件。

2. 解压您刚刚下载的可执行文件。
   - 在 Windows 上，您可以右击文件并选择“全部提取...”。在 macOS 上，您可以简单地双击文件。

3. 双击可执行文件启动程序！

{% hint style="info" %}
如果您的可执行文件无法工作，首先请确保您下载了正确的适用于您平台的文件！如果这仍然不起作用，请来我们的 [Discord](https://discord.gg/AVsGbhzTzx) 寻求帮助。
{% endhint %}

### Linux

使用 `cargo` 来下载、构建并启动 GUI：

1. 获取 `master` 分支上的最新源代码：
   ```
   $ git clone https://github.com/mel-project/earendil
   ```
2. 进入 `earendil-gui` 目录
   ```
   $ cd earendil/utilities/earendil-gui
   ```
3. 安装 GUI：
   ```
   $ cargo install --path .
   ```
4. 启动 GUI：
   ```
   $ earendil-gui
   ```