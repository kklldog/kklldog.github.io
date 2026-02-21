---
categories: [技术分享]
layout: post
title: "OpenClaw 如何集成 Discord"
tags: [OpenClaw, Discord, AI]
---

最近这几天跟风安装了 OpenClaw 大🦞，体验了一下，感觉还是不错的。我觉得 OpenClaw 跟其他工具拉开差距的点在于：它能和日常 IM 工具深度集成，比如 WhatsApp、Telegram、Discord、飞书等。有了这些 IM App，我们才能在任何时候指挥我们的🦞干活。
虽然支持列表一大串，但官方支持的国内 App 只有飞书。可惜我在安装飞书插件的时候死活过不去，只能先勉为其难地用一下 Discord。    
经过一阵折腾，终于打通 Discord。当然过程还是比较艰辛的。

## 前提条件

在开始之前，你需要准备以下内容：

- 一个 Discord 账号
- 一个 OpenClaw 实例（本文以本地部署为例）
- 一台能够运行 OpenClaw 的服务器

## 第一步：创建 Discord Application（应用）

首先，我们需要前往 [Discord Developer Portal](https://discord.com/developers/applications) 创建一个新的 Application（应用）。

![](https://static.xbaby.xyz/blog/discord-2026-02-21_175021_016.png)


点击右上角的 "New Application" 按钮，输入你的应用名称（比如 "OpenClaw"），点击创建即可。

创建完成后，你会看到 Application（应用）的基本信息页面。

## 第二步：配置 Bot

接下来我们需要配置 Bot。进入左侧菜单的 "Bot" 页面，你会看到一个 "Username" 部分，这就是你的 Bot 名称。    
点击 `Reset Token` 可以获得最新的 Token。记下来，后面要用。

![](https://static.xbaby.xyz/ScreenShot_2026-02-21_175118_088.png)

在这里需要特别注意一个关键的配置：**Message Content Intent（消息内容意图）**。这是一个“特权意图”（Privileged Intent），需要在 Bot 设置页面手动开启。OpenClaw 需要读取消息内容才能进行 AI 对话，所以这个选项必须开启。

![](https://static.xbaby.xyz/blog/discord-2026-02-21_175214_230.png)


> 💡 小提示：如果不开启这个意图，Bot 将无法读取任何消息内容，导致功能失效。

点击 "Save Changes" 保存配置。

## 第三步：在 OpenClaw 配置 Discord Channel

回到服务器终端，运行以下命令：
```
openclaw config
```
弹出如下界面：    
![](https://static.xbaby.xyz/ScreenShot_2026-02-21_174504_097.png)

选择 `local` 回车

![](https://static.xbaby.xyz/ScreenShot_2026-02-21_174555_672.png)

选择 `channels` 回车

![](https://static.xbaby.xyz/ScreenShot_2026-02-21_174755_125.png)

这里一路按回车，直到出现一个地方让你输入 Bot Token，把上面第二步得到的 Token 填入这里。

![](https://static.xbaby.xyz/ScreenShot_2026-02-22_001727_940.png)

继续配置，这里选 `Open`。   

![](https://static.xbaby.xyz/ScreenShot_2026-02-22_001825_191.png)

继续，DM Policy 选择 `Pairing`。


## 第四步：邀请 Bot 到你的服务器

在配置完 Bot 之后，我们开始配置 OAuth2 的 Scopes。

![](https://static.xbaby.xyz/ScreenShot_2026-02-21_175419_118.png)

在 "Scopes" 部分，勾选 "bot"。

![](https://static.xbaby.xyz/ScreenShot_2026-02-21_175447_934.png)

按图配置 Bot 的权限。

![](https://static.xbaby.xyz/ScreenShot_2026-02-21_175525_097.png)

勾选完毕后，下方会生成一个 "Generated URL"，点击 Copy 按钮就能得到这个链接。

## 第五步：邀请 Bot 到 Discord 服务器

把上面得到的 URL 复制到浏览器地址栏，回车后会弹出服务器选择页面，选择你希望这个 Bot 加入的服务器。

![](https://static.xbaby.xyz/ScreenShot_2026-02-22_001431_165.png)

加入成功后，刷新服务器页面，会看到这个 Bot 已经加入到你的服务器。点击头像进行私聊，随便发一句，第一次它会回复一个配对码。

![](https://static.xbaby.xyz/ScreenShot_2026-02-22_001507_399.png)

## 第六步：完成配对

回到 OpenClaw 终端，输入以下命令：
`openclaw pairing approve discord <Pairing code>`

终端会显示 `Approved`，表示成功。

## 第七步：开始使用

完成以上所有配置后，就可以开始使用 OpenClaw Bot 了！

在配置好的频道中，@ 你的 Bot 并发送消息，比如：

![](https://static.xbaby.xyz/ScreenShot_2026-02-22_024609_432.png)

Bot 就会自动识别消息内容，并调用 AI 进行处理。

## 避坑

如果你按照以上步骤，还是不能跟 Discord 配对成功，那么大概率是网络问题。我们需要使用一点点魔法🧙‍♀️（原因不可描述）。而且这里只配置 HTTP 代理还不够，需要更猛一点的魔法：开启 TUN 模式。

## 关注我的公众号一起玩转技术

![](https://static.xbaby.xyz/qrcode.jpg)
