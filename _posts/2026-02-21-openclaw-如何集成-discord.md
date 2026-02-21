---
categories: [技术分享]
layout: post
title: "OpenClaw 如何集成 Discord"
tags: [OpenClaw, Discord, AI]
---

在使用 OpenClaw 的过程中，很多同学可能都会想要把它接入 Discord，因为 Discord 是目前最流行的社区通讯工具之一，而且它的 Bot 生态系统非常成熟。本文将详细介绍如何将 OpenClaw 集成到 Discord 中，带你一步步完成整个配置过程。

## 前提条件

在开始之前，你需要准备以下内容：

- 一个 Discord 账号
- 一个 OpenClaw 实例（本文以本地部署为例）
- 一台能够运行 OpenClaw 的服务器

## 第一步：创建 Discord Application

首先，我们需要前往 [Discord Developer Portal](https://discord.com/developers/applications) 创建一个新的 Application。

![](https://static.xbaby.xyz/blog/discord-2026-02-21_174504_097.png)

点击右上角的 "New Application" 按钮，输入你的应用名称（比如 "OpenClaw"），点击创建即可。

创建完成后，你会看到 Application 的基本信息页面，其中包括 **Client ID** 和 **Client Secret**。这两个信息稍后在配置 OpenClaw 时会用到，建议先复制保存下来。

## 第二步：配置 Bot

接下来我们需要配置 Bot。进入左侧菜单的 "Bot" 页面，你会看到一个 "Username" 部分，这就是你的 Bot 名称。

![](https://static.xbaby.xyz/blog/discord-2026-02-21_174612_987.png)

在这里需要特别注意一个关键的配置：**Message Content Intent（消息内容意图）**。这是一个"特权意图"（Privileged Intent），需要在 Bot 设置页面手动开启。OpenClaw 需要读取消息内容才能进行 AI 对话，所以这个选项必须开启。

> 💡 小提示：如果不开启这个意图，Bot 将无法读取任何消息内容，导致功能失效。

点击 "Save Changes" 保存配置。

## 第三步：配置 OAuth2 Redirects

为了让 OpenClaw 能够与 Discord 进行身份验证，我们需要配置 OAuth2 的回调地址。

![](https://openclaw.com/discord/callback)

进入左侧菜单的 "OAuth2" 页面，向下滚动找到 "Redirects" 部分，点击 "Add Redirect" 按钮。

在弹出的输入框中填写你的 OpenClaw 实例的回调地址：
```
http://your-openclaw-domain:3000/api/providers/discord/callback
```

如果你是在本地开发测试，可以使用：
```
http://localhost:3000/api/providers/discord/callback
```

![](https://static.xbaby.xyz/blog/discord-2026-02-21_174707_905.png)

点击 "Save Changes" 保存。

## 第四步：邀请 Bot 到你的服务器

配置完 OAuth2 后，我们还需要生成一个邀请链接，把 Bot 添加到你的 Discord 服务器中。

在 "OAuth2" 页面找到 "OAuth2 URL Generator"，点击进入。

![](https://static.xbaby.xyz/blog/discord-2026-02-22_001507_399.png)

在 "Scopes" 部分，勾选 "bot"。

![](https://static.xbaby.xyz/blog/discord-2026-02-22_001825_191.png)

选择完 Scope 后，下方会出现 "Bot Permissions" 部分。建议勾选 "Administrator"（管理员权限），这样可以确保 Bot 有足够的权限执行各种操作。如果你想要更精细的权限控制，也可以手动选择需要的权限。

勾选完毕后，下方会生成一个 "Invite Link"，点击即可将 Bot 邀请到你的服务器。

## 第五步：配置 OpenClaw Gateway

现在我们需要回到 OpenClaw 进行最后的配置。

进入 OpenClaw 的 Web UI，找到 "Gateway 配置" 页面，点击 "添加 Channel"，选择 "Discord"。

在配置页面中，需要填写以下信息：

- **Bot Token**: 在 Discord Developer Portal 的 Bot 页面可以获取
- **App ID**: 即 Application 的 Client ID
- **Guild ID**: 你的 Discord 服务器 ID（需要在 Discord 开发者模式下获取）

![](https://static.xbaby.xyz/blog/discord-2026-02-21_175021_016.png)

配置完成后，点击保存。

## 第六步：配置消息处理器

Bot 加入服务器后，我们还需要配置消息处理器，让 Bot 知道在哪些频道监听消息，以及使用什么 Prompt。

在 OpenClaw Web UI 的 "消息处理" 页面，点击 "添加消息处理"。

选择 "Discord" 作为消息源，然后配置：

- **监听的频道**：选择 Bot 需要监听的文本频道
- **Prompt**：设置 AI 对话的系统提示词

你可以根据需要配置多个消息处理器，实现不同的功能。

## 第七步：开始使用

完成以上所有配置后，就可以开始使用 OpenClaw Bot 了！

在配置好的频道中，@ 你的 Bot 并发送消息，比如：

```
@OpenClaw 帮我总结一下上面的对话
```

Bot 就会自动识别消息内容，并调用 AI 进行处理。

## 总结

本文详细介绍了 OpenClaw 集成 Discord 的完整步骤：

1. 创建 Discord Application
2. 配置 Bot（重点：开启 Message Content Intent）
3. 配置 OAuth2 Redirects
4. 生成邀请链接将 Bot 添加到服务器
5. 在 OpenClaw 中配置 Discord Channel
6. 配置消息处理器

配置完成后，你就可以在 Discord 中享受 AI 对话的便利了！快去试试吧。

## 关注我的公众号一起玩转技术   

![](https://static.xbaby.xyz/qrcode.jpg)
