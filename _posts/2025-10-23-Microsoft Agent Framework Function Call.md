前面我们已经介绍了 Microsoft Agent Framework 的 Agent 的基本使用方法。今天我们来介绍一下工具（Function Call）的用法。
在使用大型语言模型（LLM）构建智能应用时，一个核心能力是让模型能够与外部世界交互。Microsoft Agent Framework 通过强大的“函数调用”（Function Calling）或“工具使用”（Tool Use）机制，赋予了 AI Agent 调用外部代码、API 和服务的能力。这极大地扩展了 Agent 的应用场景，使其不再局限于已有的知识库，而是能够获取实时信息、执行操作。

本文将深入探讨 Microsoft Agent Framework 中函数调用的两种核心模式：

1.  **自主函数调用**：Agent 根据用户意图，自主决定调用哪个工具来完成任务。
2.  **人工审批（Human-in-the-Loop）**：在执行敏感或高成本操作前，暂停执行并请求用户批准。

#### 准备工作

在开始之前，我们需要一个 AI 客户端。这里我们使用 `AzureOpenAIClient` 连接到 Azure OpenAI 服务。

```csharp
var azureAiEndpoint = "https://YOUR_AOAI_ENDPOINT.openai.azure.com";
var apiKey = "YOUR_AOAI_API_KEY";

var client = new AzureOpenAIClient(
    new Uri(azureAiEndpoint),
    new ApiKeyCredential(apiKey));

var chatClient = client.GetChatClient("gpt-4o-mini"); // 你的模型部署名称
```

#### 场景一：自主函数调用

这是最直接的函数调用方式。我们向 Agent 提供一组工具，当用户提出请求时，Agent 会智能地判断是否需要以及如何使用这些工具来生成最佳回复。

##### 第 1 步：定义工具

一个“工具”本质上就是一个 C# 方法。为了让 Agent 理解这个工具的用途，我们必须使用 `[Description]` 特性来清晰地描述函数的功能及其参数。

```csharp
using System.ComponentModel;

[Description("根据给定的用户名获取用户信息。")]
static string GetUserInfo([Description("用户名")] string userName)
    => $"{userName} 来自苏州，男性，28 岁";
```
> **关键点**：`[Description]` 中的描述至关重要。Agent 会依赖这些文本来理解工具的用途，并决定何时调用它。描述越清晰、准确，Agent 的表现就越好。

##### 第 2 步：创建 Agent 并注册工具

接下来，我们创建一个 `AIAgent` 实例，并通过 `AIFunctionFactory.Create()` 将我们的 C# 方法包装成 Agent 可以使用的 `AIFunction`，然后在创建时注册它。

```csharp
using Microsoft.Agents.AI;
using Microsoft.Extensions.AI;

AIAgent agent = chatClient.CreateAIAgent(
    instructions: "你是一个乐于助人的助手",
    tools: [AIFunctionFactory.Create(GetUserInfo)]);
```

##### 第 3 步：运行 Agent

现在，我们可以向 Agent 提问。Agent 会分析我们的问题，发现需要调用 `GetUserInfo` 函数来获取信息，然后自主完成调用并整合结果返回。

```csharp
AgentRunResponse response = await agent.RunAsync("请告诉我 Tom 的信息？");
Console.WriteLine(response.GetFinalAnswer());

/*** 输出 ***/
// Tom 来自苏州，男性，28 岁
```
整个过程对开发者是透明的。Agent Framework 自动处理了意图识别、工具选择、参数提取、函数执行和结果整合的所有步骤。

#### 场景二：人工审批（Human-in-the-Loop）

对于某些操作，比如修改数据库、发送邮件或执行高成本的 API 调用，我们不希望 Agent 自主执行。这时，就需要引入人工审批流程。

##### 第 1 步：创建需要审批的工具

Agent Framework 提供了 `ApprovalRequiredAIFunction` 包装器。只需将任何 `AIFunction` 放入其中，Agent 在调用它之前就会自动暂停并请求批准。

```csharp
// 复用我们之前定义的 GetUserInfo 方法
AIFunction userInfoFunc = AIFunctionFactory.Create(GetUserInfo);

// 将其包装为需要审批的函数
AIFunction approvalRequiredWeatherFunction = new ApprovalRequiredAIFunction(userInfoFunc);
```

##### 第 2 步：创建 Agent 并处理审批请求

为了在多次交互中保持上下文（例如，提出请求 -> 批准 -> 获取结果），我们需要使用 `AgentThread`。

```csharp
// 创建一个使用“需要审批”工具的 Agent
AIAgent agentWithApproval = chatClient.CreateAIAgent(
    instructions: "你是一个乐于助人的助手",
    tools: [approvalRequiredWeatherFunction]);

// 创建一个新的对话线程
AgentThread thread = agentWithApproval.GetNewThread();

// 第一次运行，Agent 会请求批准
AgentRunResponse response = await agentWithApproval.RunAsync("请告诉我 Tom 的信息？", thread);

// 从响应中筛选出函数调用批准请求
var functionApprovalRequests = response.Messages
    .SelectMany(x => x.Contents)
    .OfType<FunctionApprovalRequestContent>()
    .ToList();

if (functionApprovalRequests.Any())
{
    FunctionApprovalRequestContent requestContent = functionApprovalRequests.First();
    Console.WriteLine($"需要您的批准才能执行 '{requestContent.FunctionCall.Name}'");

    // 模拟用户批准
    var approvalMessage = new ChatMessage(ChatRole.User, [requestContent.CreateResponse(approved: true)]);
    
    // 第二次运行，传入批准消息
    AgentRunResponse finalResponse = await agentWithApproval.RunAsync(approvalMessage, thread);
    Console.WriteLine(finalResponse.GetFinalAnswer());
}
```

> **注意**：`#pragma warning disable MEAI001` 用于抑制关于预览版 API 的警告。在生产代码中，请关注 API 的稳定性。

##### 运行结果

```
需要您的批准才能执行 '_Main_g_GetUserInfo_0_0'
Tom 来自苏州，男性，28 岁
```
如上所示，Agent 在第一次调用后暂停，等待我们的批准。在收到批准消息后，它继续执行函数并返回最终结果。如果我们在 `CreateResponse` 中传入 `approved: false`，Agent 将会中止操作。

#### 总结

Microsoft Agent Framework 的函数调用功能为连接 LLM 与外部世界提供了强大而灵活的桥梁。通过简单的 `[Description]` 特性，我们可以轻松地将现有 C# 代码暴露给 AI Agent。无论是需要 Agent 自主决策的便捷场景，还是要求严格控制的敏感操作，Agent Framework 都提供了相应的模式（`AIFunction` 和 `ApprovalRequiredAIFunction`）来满足需求，让开发者可以安全、高效地构建功能丰富的智能应用。