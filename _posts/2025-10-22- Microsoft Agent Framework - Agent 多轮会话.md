好的，这是一篇关于 Microsoft Agent Framework 多轮对话的技术博客。

---

### Microsoft Agent Framework - Agent 多轮对话

上一篇我们分享了 Microsoft Agent Framework 一些背景信息已经 Agent 的基本使用。这次我们继续介绍 Agent 的会话功能。   

在构建智能对话机器人时，最核心的功能之一就是能够理解上下文并进行多轮对话。这使得交流不再是简单的一问一答，而是流畅、自然的互动。Microsoft Agent Framework 通过一个简洁而强大的设计，让开发者能够轻松实现这一功能。本文将深入探讨如何使用该框架来管理和维持多轮对话。

#### 核心概念：`AgentThread`

在 Microsoft Agent Framework 中，`AgentThread` 是实现多轮对话的关键。它代表一个独立的对话线程，负责跟踪和管理整个对话的上下文历史。

当您与 Agent 进行交互时，框架会将每一次的请求和响应都记录在 `AgentThread` 中。在后续的对话中，框架会自动将此前的历史记录一并发送给 AI 模型。这样，AI 模型就能“记住”之前的对话内容，从而给出与上下文相关的回复。

服务的类型决定了对话历史的存储方式。例如，当使用 `ChatCompletion` 服务时，对话历史存储在 `AgentThread` 对象中，并在每次调用时发送到服务。而当使用 Azure AI Agent 服务时，对话历史存储在服务中，每次调用只发送对话的引用。

#### 多轮对话

下面我们通过一个具体的代码示例，来了解如何实现一个简单的多轮对话。

**1. 初始化 Agent**

首先，我们需要创建一个 `AIAgent` 实例。这里我们使用 Azure OpenAI 服务，并为 Agent 设置一个指令，告诉它是一个了解中国古典文学的专家。

```csharp
using Azure.AI.OpenAI;
using Microsoft.Agents.AI;
using System.ClientModel;

// 初始化 Azure OpenAI 客户端
var azureAiEndpoint = "https://your-endpoint.openai.azure.com";
var apiKey = "your-api-key";

AIAgent agent = new AzureOpenAIClient(
        new Uri(azureAiEndpoint),
        new ApiKeyCredential(apiKey))
    .GetChatClient("gpt-4o-mini")
    .CreateAIAgent(instructions: "你很了解中国古典文学.", name: "Joker");
```

**2. 创建对话线程并进行交流**

通过调用 `agent.GetNewThread()` 来创建一个新的对话线程。然后，在每次调用 `agent.RunAsync()` 时传入同一个 `thread` 对象，即可实现连续对话。

```csharp
// 创建一个新的对话线程
AgentThread thread = agent.GetNewThread();

// 第一轮对话
Console.WriteLine(await agent.RunAsync("红楼梦里有多少女性角色？", thread));

// 第二轮对话
// Agent 会结合上一轮的问题和回答进行回复
Console.WriteLine(await agent.RunAsync("其中结局最悲惨的是哪个？", thread));
```

在上面的例子中，第二个问题“其中结局最悲惨的是哪个？”本身没有主语。但由于我们传入了同一个 `thread` 对象，Agent 能够从上一轮的对话中理解“其中”指代的是“《红楼梦》中的女性角色”，从而给出准确的回答（林黛玉）。

#### 对话隔离

在实际应用中，Agent 可能需要同时与多个用户进行独立的对话。Microsoft Agent Framework 通过创建不同的 `AgentThread` 实例来轻松实现对话隔离。每个 `AgentThread` 都维护着自己独立的上下文，互不干扰。

下面的示例展示了如何同时处理两个完全不同主题的对话：

```csharp
// 为两个不同的对话创建独立的线程
AgentThread thread1 = agent.GetNewThread(); // 对话 1: 关于《红楼梦》
AgentThread thread2 = agent.GetNewThread(); // 对话 2: 关于《西游记》

// 同时发起两个不同主题的对话
var answer1_1 = await agent.RunAsync("贾宝玉是谁？", thread1);
var answer2_1 = await agent.RunAsync("西游记主角团有几个人？", thread2);

Console.WriteLine($"对话1, 第1轮: {answer1_1}");
Console.WriteLine($"对话2, 第1轮: {answer2_1}");

// 继续各自的对话
// thread1 的上下文是贾宝玉，thread2 的上下文是西游记
var answer1_2 = await agent.RunAsync("他的结局如何？", thread1);
var answer2_2 = await agent.RunAsync("谁最厉害？", thread2);

Console.WriteLine($"对话1, 第2轮: {answer1_2}");
Console.WriteLine($"对话2, 第2轮: {answer2_2}");
```

在这个例子中：
*   `thread1` 的第二轮对话 `agent.RunAsync("他的结局如何？", thread1)`，Agent 知道“他”指的是贾宝玉。
*   `thread2` 的第二轮对话 `agent.RunAsync("谁最厉害？", thread2)`，Agent 知道“谁”指的是《西游记》主角团中的角色。

两个对话线程的上下文完全隔离，确保了 Agent 能够准确地处理并发且独立的交流。

#### 总结

Microsoft Agent Framework 通过 `AgentThread` 对象提供了一种直观且高效的方式来管理对话状态。无论是需要进行有上下文记忆的深度多轮对话，还是需要处理多个并发的独立会话，开发者都可以通过简单地创建和传递 `AgentThread` 实例来轻松实现。这为构建更复杂、更智能的 AI 应用奠定了坚实的基础。