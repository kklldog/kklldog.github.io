好的，这是您要求的 Markdown 格式的博文。

---

# Microsoft Agent Framework 简单介绍与使用

最近，微软推出了一个全新的 `Microsoft.Agents`（即 Microsoft Agent Framework），旨在简化和统一与大型语言模型（LLM）的交互方式，让开发者可以更轻松地构建、协调和管理多代理（multi-agent）AI 系统。本文将结合官方文档和具体代码示例，对这个新框架进行一个简单的介绍和上手体验。

## 什么是 Microsoft Agent Framework？

根据官方文档，Microsoft Agent Framework 提供了一套用于创建和协调 AI 代理的工具和库。它的核心思想是，无论底层使用的是 OpenAI、Azure OpenAI 还是其他模型，开发者都可以通过一套统一的抽象（如 `AIAgent`）来与之交互。这大大降低了在不同模型或服务之间切换的复杂性，并为构建更复杂的代理协作系统（如两个代理对话）奠定了基础。

该框架目前作为 `Microsoft.Extensions.AI` 库的一部分提供。

## 为什么需要 Agent Framework？

在 `Agent Framework` 出现之前，`Semantic Kernel` 和 `AutoGen` 已经为 AI 代理和多代理编排的概念奠定了基础。`Agent Framework` 正是由打造这两个框架的同一个团队开发的直接后继者。

可以将其理解为集大成者，它融合了两者的核心优势：
*   **继承自 AutoGen**：为单代理和多代理交互模式提供了简洁、易于上手的抽象。
*   **继承自 Semantic Kernel**：提供了企业级功能，例如基于线程的状态管理、类型安全、过滤器、遥测以及对多种模型和嵌入的广泛支持。

更重要的是，`Agent Framework` 不仅仅是两者的合并。它还引入了全新的**工作流（Workflows）**概念，让开发者能够明确地控制多代理的执行路径。此外，它还提供了一个更强大的状态管理系统，专门用于支持长时间运行和需要人工介入的复杂场景。

简单来说，`Agent Framework` 是 `Semantic Kernel` 和 `AutoGen` 的下一代演进版本，旨在提供一个更强大、更灵活、更适合企业级应用的 AI 代理开发平台。

## 快速上手

要开始使用 Agent Framework，首先需要安装相关的 NuGet 包并准备好你的 AI 服务凭据。

1.  **安装 NuGet 包**:
```powershell
dotnet add package Microsoft.Extensions.AI
    dotnet add package Microsoft.Agents.AI
    dotnet add package Azure.AI.OpenAI
```

2.  **创建客户端**:
    你需要准备好你的 AI 服务终结点（Endpoint）和密钥（API Key）。在下面的示例中，我们使用 Azure OpenAI 服务。

```csharp
using Azure.AI.OpenAI;
    using Azure.Identity;
    using Microsoft.Agents.AI;
    using System.ClientModel;

    var azureAiEndpoint = "你的 Azure OpenAI 终结点";
    var apiKey = "你的 Azure OpenAI 密钥";

    // 创建 Azure OpenAI 客户端
    var client = new AzureOpenAIClient(
        new Uri(azureAiEndpoint),
        new ApiKeyCredential(apiKey));
```

## 核心用法示例

接下来，我们将通过几个具体的代码示例来展示 `AIAgent` 的核心功能。

### 1. 创建 Agent 并进行对话

创建 `AIAgent` 非常简单。你只需要从客户端获取一个聊天模型（例如 `gpt-4o-mini`），然后调用 `CreateAIAgent` 方法。你可以通过 `instructions` 参数给代理设定一个初始“人设”。

```csharp
// 1. 创建 Agent 进行对话
AIAgent agent = client
    .GetChatClient("gpt-4o-mini")
    .CreateAIAgent(instructions: "你很擅长讲笑话.", name: "Joker");

var response = await agent.RunAsync("说一个关于川普的笑话");

Console.WriteLine(response);
```

**输出:**

```
当然可以！这里有一个关于川普的笑话：

为什么川普的计算机总是被病毒感染？

因为它总是打开“墙”（防火墙）！
```

### 2. 流式响应

对于需要实时反馈的场景（例如聊天机器人），你可以使用 `RunStreamingAsync` 方法来获取流式响应。这样可以逐字或逐词地接收模型的输出，提升用户体验。

```csharp
// 2. 流式响应
await foreach (var update in agent.RunStreamingAsync("说个关于川普的顶级笑话."))
{
    Console.WriteLine(update);
}
```

**输出:**

```
为什么
川
普
在
网
球
比赛
中
总
是
输
？


因为
他
总
是
试
图
“
发
球
”
而
不是
“
接
球
”
！
```

### 3. 多模态输入（图文识别）

Agent Framework 原生支持多模态输入。你可以通过 `ChatMessage` 和 `UriContent` 将图片 URL 和文本一起发送给模型。这对于需要图像理解能力的场景非常有用。    
![](https://static.xbaby.xyz/ScreenShot_2025-10-21_011132_152.png)

我准备了一张图片，让大模型帮忙我们看看。
```csharp
// 3. Running the agent with ChatMessages
ChatMessage message = new(ChatRole.User, [
    new TextContent("简单描述图片的内容?"),
    new UriContent("https://static.xbaby.xyz/ScreenShot_2025-10-21_011132_152.png", "image/png")
]);

Console.WriteLine(await agent.RunAsync(message));
```

**输出:**

```
这张图片展示了两个卡通角色，一只牛和一匹马，它们坐在一个看起来像办公室的地方。
牛看起来很友好，正在看着电脑屏幕，而马则穿着一件带领子的衣服，似乎在认真地工作。
背景有一块牌子，上面写着“24小时营业 欢迎光临”，表明这个地方是开放的，可能是在接待顾客。
整个场景给人一种轻松和幽默的感觉。
```

### 4. 使用 System Message 指导代理行为

除了在创建代理时使用 `instructions`，你还可以在每次请求时通过 `ChatRole.System` 类型的 `ChatMessage` 来动态地指导代理的行为。这提供了更灵活的上下文控制。

例如，我们可以让代理扮演一个“职场打工人”，并用带有苦涩感的口吻来回答问题。

```csharp
// 4. 使用 System Message 来指导代理的行为
ChatMessage systemMessage = new(
    ChatRole.System,
    """
    你是一位职场打工人，你的回答普遍带有职场打工人的苦涩。
    """);
ChatMessage userMessage = new(ChatRole.User, [new TextContent("简单描述图片的内容"), new UriContent("https://static.xbaby.xyz/ScreenShot_2025-10-21_011132_152.png", "image/png")]);

Console.WriteLine(await agent.RunAsync([systemMessage, userMessage]));
```

**输出:**

```
这幅图片展示了两只卡通动物，分别是一头牛和一匹马，它们坐在办公室里，面前有一台电脑。墙上悬挂着一块招牌，上面写着“24小时营业 欢迎光临”。
牛和马的表情似乎有些专注或者困惑，整个场景给人一种幽默而轻松的氛围，有点像职场打工人的日常，可能是在忙于处理一些琐碎的工作。
```
可以看到，这次的描述带上了一点“职场人”的味道。

## 总结

Microsoft Agent Framework 为 .NET 开发者提供了一个强大而统一的接口来与大语言模型进行交互。作为 `Semantic Kernel` 和 `AutoGen` 的演进，它结合了前两者的优点并引入了工作流等新特性，旨在成为构建企业级 AI 代理应用的首选框架。通过 `AIAgent`，我们可以轻松实现文本对话、流式输出、多模态识别以及通过系统消息进行行为指导等功能。随着这个框架的不断成熟，未来构建复杂的 AI 代理系统将会变得更加简单和高效。