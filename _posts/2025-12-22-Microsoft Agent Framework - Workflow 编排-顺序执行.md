
在构建复杂的人工智能应用时，我们常常需要将一个大任务拆解成多个小步骤，并让不同的 AI Agent 按顺序依次处理。Microsoft Agent Framework 提供了一个强大而灵活的工具——Workflow，来帮助我们轻松编排和管理这些 Agent 的协作流程。

今天，我们就来深入探讨一种最基本也最常用的工作流模式：**顺序工作流 (Sequential Workflow)**。

### 场景设定

假设我们需要一个翻译流程：用户输入一句中文，我们希望系统能依次提供英文和日文的翻译。

为了实现这个目标，我们将创建两个专门的 AI Agent：
1.  **英语翻译官**：一个只负责将中文翻译成英文的专家。
2.  **日语翻译官**：一个只负责将中文翻译成日文的专家。

然后，我们会使用 `AgentWorkflowBuilder` 将这两个 Agent 构建成一个顺序执行的工作流。

![](https://static.xbaby.xyz/blog/d4a38141-b140-4d5e-b2a1-8889de8d3ca7.png)


### 代码详解

下面是实现这个顺序翻译工作流的 C# 完整代码。

```csharp
using System.ClientModel;
using Microsoft.Extensions.AI;
using OpenAI;
using Microsoft.Agents.AI.Workflows;
using OpenAI.Chat;

namespace MSAgentFramework.Learn.workflow
{
    internal class Seq
    {
        public async Task Run()
        {
            var endpoint = "https://api.deepseek.com/v1";
            var apiKey = "sk-59872d44521c4d798faa8315529abcce";

            // 1. 创建英语翻译 Agent
            var englishTranslator = new OpenAIClient(
                    new ApiKeyCredential(apiKey)
                    , new OpenAIClientOptions()
                    {
                        Endpoint = new Uri(endpoint)
                    }
                    )
                .GetChatClient("deepseek-chat")
                .CreateAIAgent(instructions: "你是一个英语专家，当你收到中文的时候帮忙翻译成英文.", name: "English Translator");

            // 2. 创建日语翻译 Agent
            var japaneseTranslator = new OpenAIClient(
                    new ApiKeyCredential(apiKey)
                    , new OpenAIClientOptions()
                    {
                        Endpoint = new Uri(endpoint)
                    }
                )
                .GetChatClient("deepseek-chat")
                .CreateAIAgent(instructions: "你是一个日语专家，当你收到中文的时候帮忙翻译成日文.", name: "Japanese Translator");

            // 3. 构建顺序工作流
            var workflow = AgentWorkflowBuilder.BuildSequential([englishTranslator, japaneseTranslator]);

            // 4. 准备输入消息并运行工作流
            var messages = new List<Microsoft.Extensions.AI.ChatMessage> { new(ChatRole.User, "人生如一本书，愚者草草翻过，智者细细阅读。") };
            StreamingRun run = await InProcessExecution.StreamAsync(workflow, messages);
            await run.TrySendMessageAsync(new TurnToken(emitEvents: true));

            List<Microsoft.Extensions.AI.ChatMessage> result = new();

            // 5. 监听并处理工作流事件
            await foreach (WorkflowEvent evt in run.WatchStreamAsync().ConfigureAwait(false))
            {
                if (evt is AgentRunUpdateEvent e)
                {
                    Console.WriteLine($"{e.ExecutorId}: {e.Data}");
                }
                else if (evt is WorkflowOutputEvent outputEvt)
                {
                    result = (List<Microsoft.Extensions.AI.ChatMessage>)outputEvt.Data!;
                    break;
                }
            }

            // 6. 显示最终结果
            foreach (var message in result)
            {
                Console.WriteLine($"{message.Role}: {message.Text}");
            }
        }
    }
}
```

#### 代码剖析

1.  **Agent 初始化**:
    *   我们首先配置了 `OpenAIClient`，值得注意的是，这里我们使用了自定义的 `Endpoint` (deepseek) 和对应的 `ApiKey`。
    *   接着，通过 `CreateAIAgent` 方法创建了两个实例：`englishTranslator` 和 `japaneseTranslator`。
    *   每个 Agent 都被赋予了明确的 `instructions` (指令) 和一个唯一的 `name`。这些指令是 Agent 行为的核心，它告诉 Agent 它的角色和职责。

2.  **构建 Workflow**:
    *   最关键的一步在这里：`AgentWorkflowBuilder.BuildSequential([englishTranslator, japaneseTranslator]);`。
    *   我们调用 `BuildSequential` 方法，并传入一个包含我们两个 Agent 的数组。**Agent 在数组中的顺序决定了它们的执行顺序**。在这个例子中，`englishTranslator` 会先执行，然后是 `japaneseTranslator`。

3.  **执行与结果处理**:
    *   我们创建了一个初始的用户消息。
    *   `InProcessExecution.StreamAsync` 启动工作流，并通过 `WatchStreamAsync` 异步监听工作流产生的事件。
    *   `AgentRunUpdateEvent` 事件可以让我们实时看到每个 Agent 执行时产生的数据片段。
    *   `WorkflowOutputEvent` 事件则标志着整个工作流执行完毕，我们可以从中获取最终的完整结果。

### 结果与解读

当上述代码运行后，我们会在控制台看到如下输出：

```text
user: 人生如一本书，愚者草草翻过，智者细细阅读。
assistant: Life is like a book: the fool flips through it, while the wise read it carefully and deliberately.
assistant: 人生は一冊の本のようなもの。愚者はざっとページをめくり、賢者は丹念に読み込む。
```

### 总结

通过这个简单的例子，我们了解了如何使用 `AgentWorkflowBuilder.BuildSequential` 来创建一个按预定顺序执行任务的 Agent 工作流。这种模式非常适合处理需要多步骤、逻辑清晰的复杂任务。

通过链式组合不同功能的 Agent，我们可以构建出功能强大、逻辑严密的 AI 应用，而框架本身则优雅地处理了 Agent 之间的状态和上下文传递。

希望这篇博客能帮助你入门 Microsoft Agent Framework 的顺序工作流。快去尝试构建属于你自己的 Agent 链吧！