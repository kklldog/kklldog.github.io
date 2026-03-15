目前 skills 在 AI Agent 处理事务方面大热。微软后知后觉在 Microsoft Agent Framework 上也算跟上了脚本，终于支持了 skills。本文将介绍什么是 Skills，并结合代码示例，展示如何使用它们来增强您的 AI Agent。

#### 1. 什么是 Skills？

根据官方规范，**代理技能 (Agent Skills)** 是一个包含指令、脚本和资源的可移植包，旨在为代理提供专用的功能和领域专业知识。您可以将它看作一个“工具箱”，代理可以根据需要打开并使用其中的工具来完成特定任务。

使用代理技能的主要优势包括：

*   **封装领域专业知识**：将特定领域的知识（如公司报销政策、法律工作流程）打包成可重用的技能。
*   **扩展代理功能**：为代理添加新功能，而无需修改其核心指令。
*   **确保一致性**：将复杂的多步骤任务转换为可重复、可审核的工作流。
*   **启用互操作性**：在不同的、兼容技能规范的代理产品中复用同一个技能。

**技能结构与 `SKILL.md`**

一个技能本质上是一个目录，其核心是 `SKILL.md` 文件。该文件通过 YAML 前置元数据（Frontmatter）和 Markdown 内容来定义技能。

*   **YAML Frontmatter**：
```
expense-report/
├── SKILL.md                          # Required — frontmatter + instructions
├── scripts/
│   └── validate.py                   # Executable code agents can run
├── references/
│   └── POLICY_FAQ.md                 # Reference documents loaded on demand
└── assets/
    └── expense-report-template.md    # Templates and static resources
```
*   **Markdown 正文**：
```
---
name: expense-report
description: File and validate employee expense reports according to company policy. Use when asked about expense submissions, reimbursement rules, or spending limits.
license: Apache-2.0
compatibility: Requires python3
metadata:
  author: contoso-finance
  version: "2.1"
---
```

**渐进式披露 (Progressive Disclosure)**

为了高效利用大语言模型的上下文窗口（Context Window），技能采用了一种三阶段的“渐进式披露”模式：

1.  **宣传 (Advertise)**：在每次运行时，仅将技能的 `name` 和 `description` 注入到系统提示中。这让代理知道有哪些技能可用，但开销极小。
2.  **加载 (Load)**：当代理认为某个任务与特定技能相关时，它会调用 `load_skill` 工具来加载完整的 `SKILL.md` 文件，获取详细的执行指令。
3.  **读取资源 (Read Resource)**：如果指令中需要，代理会进一步调用 `read_skill_resource` 工具来按需读取脚本或参考文档等补充资源。

这种模式确保了代理的上下文始终保持精简，仅在需要时才加载深度领域知识，从而节省了成本并提高了效率。

在 Agent Framework 中，框架提供了一个**技能提供程序 (Skill Provider)**，例如 `FileAgentSkillsProvider`。它的作用是发现文件系统中的技能目录，并自动为代理提供 `load_skill`、`read_skill_resource` 等工具，让代理能够遵循渐进式披露模式来使用这些技能。

#### 2. 代码示例

接下来，我们通过一个具体的 C# 代码示例来看看如何为代理提供并使用技能。下面的代码来自 `SkillTester.cs` 文件。

首先，我们需要创建一个 `FileAgentSkillsProvider` 实例。这个提供程序会扫描指定路径（在这里是 `skills` 目录），发现所有符合规范的技能。

```csharp
            // Discover skills from the 'skills' directory
            var skillsProvider = new FileAgentSkillsProvider(
                skillPath: Path.Combine(AppContext.BaseDirectory, "skills"));
```

然后，在创建代理时，我们将这个 `skillsProvider` 作为 `AIContextProviders` 之一传递给代理。这样，代理在启动时就知道了所有可用技能的“宣传”信息。

```csharp
            // Create an agent with the skills provider
            AIAgent agent = new OpenAIClient(
                new ApiKeyCredential(apiKey)
                , new OpenAIClientOptions()
                    {
                        Endpoint = new Uri(endpoint)
                    }
                    )
                .GetChatClient("gpt-5.4")
                .AsAIAgent(new ChatClientAgentOptions
                {
                    Name = "SkillsAgent",
                    ChatOptions = new()
                    {
                        Instructions = "You are a helpful assistant.",
                    },
                    AIContextProviders = [skillsProvider],
                });
```

#### 3. 运行与结果

现在，代理已经具备了从 `skills` 目录加载的技能。我们可以给它一个具体的任务，比如“写一个关于观看F1上海站比赛的小红书笔记”。代理会根据这个提示，判断是否需要使用某个技能来更好地完成任务。

```csharp
            var response = await agent.RunAsync("写一个关于观看F1上海站比赛的小红书笔记");

            Console.WriteLine(response);
```

假设在 `skills` 目录中有一个专门用于生成社交媒体内容的技能，代理在“加载”该技能的 `SKILL.md` 后，就会按照其中的指令生成格式精美、符合平台风格的内容。

运行代码后，我们得到以下输出：

```plaintext
---

# 标题建议
**现场看F1上海站到底有多爽？一篇告诉你值不值得冲！**

---

# 正文

终于打卡了心心念念的**F1上海站**，如果你问我一句：**值不值得去现场看？**
我的答案是：**太值了，而且和在屏幕前看完全不是一个概念！**

第一次来到上汽国际赛车场，真的会被现场氛围震住。
还没正式进场，就已经看到满满的车迷穿着各个车队的周边：
法拉利红、梅奔黑、迈凯伦橙、红牛深蓝……
感觉不是来“看比赛”，而是直接走进了F1的世界里。

## 1｜现场最震撼的，真的是赛车声音
在电视里你会觉得：哦，车很快。
但到了现场你才知道，**F1的速度感是扑面而来的**。

赛车从眼前冲过去那一瞬间，
不是“看到一辆车开过去”，
而是**你整个人都被引擎轰鸣、轮胎摩擦和空气撕裂感包围**。
尤其是在长直道和刹车点附近，真的会忍不住一直“哇”出来。
那种肾上腺素飙升的感觉，屏幕根本传达不出来。

## 2｜现场气氛比想象中还燃
比赛开始前的暖场、观众席的欢呼、大家一起举手机拍发车、
每一次超车成功，全场都会有很明显的欢呼声。
如果刚好你支持的车手在你面前完成防守或者超越，
那个瞬间真的会直接从座位上弹起来。
```

从结果可以看出，代理不仅理解了任务，还可能利用了某个技能，生成了包含标题建议、正文、表情符号和 Markdown 格式的专业小红书笔记。

#### 4. 总结

通过使用 Skills，我们可以将通用的 AI 代理转变为能够执行特定领域任务的专家。Skills 通过其可移植的结构和高效的“渐进式披露”模式，为扩展代理功能提供了一种标准化、可维护且强大的方式。这使得开发者能够构建出更加智能、可靠和多才多艺的 AI 应用。