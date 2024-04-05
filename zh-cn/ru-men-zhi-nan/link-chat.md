Earendil 的 `link-chat` 是一个极简的聊天工具，用于与 Earendil 网络中的直接邻居交流。您可以使用它与邻居协商价格和讨论您的债务。

## 图形界面（GUI）
`link-chat` 位于 Earendil GUI 的 "Chat" 标签中。您可以通过在屏幕左侧的菜单选择任何邻居的指纹，与其聊天：

![](../../en/.gitbook/assets/gui-chat.png)

## 命令行界面（CLI）
在终端中，输入以下命令来使用 `link-chat`：

```bash
earendil control [--connect 127.0.0.1:control_listen_port] chat <COMMAND>
```

其中 `<COMMAND>` 是以下命令之一：
- `list` - 显示您所有对话的摘要
- `start <neighbor-prefix>` - 与指纹前缀为 `<neighbor-prefix>` 的邻居开始互动聊天会话。如果 `<neighbor-prefix>` 与多于一位邻居匹配，这个命令将会失败（在这种情况下，请使用更长的前缀）。

一些例子：

```bash
$ earendil control chat start zpy
<starting chat with zpyzw3hpax9fnwww08h9bhr866qvh6wn>
<- 嘿 Alice！[2024-01-10 19:59:55]
-> 嘿 Bob！[2024-01-10 19:59:58]
<- 你们那儿今天天气如何？[2024-01-10 20:00:04]
晴空万里！
-> 晴空万里！[2024-01-10 20:01:15]

```

```!bash
$ earendil control chat list
+------------------------------------+-------------------+-----------------------------------+
| Neighbor                           | # of Messages     | Last chat                         |
+------------------------------------+-------------------+-----------------------------------+
| 4b7a641b77c2d6ceb8b3fecec2b2978... | 4                 | sunny! [2024-01-10 15:29:05]
+------------------------------------+-------------------+-----------------------------------+
```