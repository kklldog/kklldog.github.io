---
categories: [技术分享]
layout: post
title: "OpenClaw 如何白嫖顶级模型｜免费 MiniMax M2.5 实战"
tags: [OpenClaw, AI, MiniMax, LLM, 白嫖]
---

> 写在前面：OpenClaw 虽然强大，但它可是一个 token 消耗大户——每次对话都在燃烧你的 API 配额，钱包表示压力很大 😢

不过最近发现了一个好消息：**opencode zen 平台**正好提供免费的 MiniMax M2.5 模型试用！这波不亏，今天来聊聊如何配置。

## 为什么选择 MiniMax M2.5？

先来看看它的硬实力：

| 属性 | 值 |
|------|-----|
| 上下文窗口 | 100K tokens |
| 最大输出 | 100K tokens |
| 价格 | **免费** |
| 推理能力 | 支持 |

100K 上下文是什么概念？相当于可以一次性处理约 **75 万字**的中文内容，丢个整本小说进去分析都不是问题。

最关键的是——**免费**，这不比香吗？

## 配置步骤

### 1. 修改配置文件

打开 OpenClaw 的配置文件 `~/.openclaw/openclaw.json`，在 `models.providers` 中添加：

```json
"m2-5": {
  "baseUrl": "https://opencode.ai/zen/v1",
  "apiKey": "你的 API Key",
  "api": "openai-completions",
  "models": [
    {
      "id": "minimax-m2.5-free",
      "name": "minimax-m2.5-free",
      "reasoning": false,
      "input": ["text"],
      "cost": {
        "input": 0,
        "output": 0,
        "cacheRead": 0,
        "cacheWrite": 0
      },
      "contextWindow": 100000,
      "maxTokens": 100000
    }
  ]
}
```

### 2. 设置默认模型

在 `agents.defaults.model` 中配置：

```json
"model": {
  "primary": "m2-5/minimax-m2.5-free"
}
```

同时在 `agents.defaults.models` 中注册：

```json
"models": {
  "m2-5/minimax-m2.5-free": {}
}
```

### 3. 重启 Gateway

```bash
openclaw gateway restart
```

## 使用体验

配置完成后直接开用！响应速度快、上下文理解能力强，关键是**免费**。

写代码、回答问题、分析长文档都不在话下。100K 上下文丢个几千行代码库进去让它分析都行。如果你还需要更强的推理能力，可以搭配智谱的 GLM 系列模型一起使用。

## 总结

**白嫖**才是硬道理！MiniMax M2.5 Free = 100K 上下文 + 免费 + 稳定输出，还要什么自行车 🚲

OpenClaw 配这个模型，丝滑又省钱，舒服~

> 悄悄告诉你：zen 平台还提供了其他免费模型，比如 **GLM-5**、**GLM-4.7** 等，使用方式跟上面类似，也可以一起配置体验。

---

如果你有更好的免费模型推荐，欢迎留言交流 🙌
