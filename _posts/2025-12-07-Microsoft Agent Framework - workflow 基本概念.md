
Microsoft Agent Framework 提供了一个强大的工作流 (Workflow) 系统，使您能够构建集成了 AI 代理和业务流程的智能自动化系统。借助其类型安全的体系结构和直观的设计，您可以编排复杂的工作流，而无需陷入基础设施的复杂性中，从而专注于核心业务逻辑。

## 🤖 AI Agent 与 Workflow 有何不同？

在深入探讨之前，我们先来厘清一个基本概念：AI Agent 和 Workflow 的区别。

*   **AI Agent**: 通常由大型语言模型 (LLM) 驱动，可以访问各种工具来完成任务。Agent 执行的步骤是动态的，由 LLM 根据对话上下文和可用工具决定。

![](https://static.xbaby.xyz/blog/ai-agent.png)

*   **Workflow**: 一个预定义的、由多个操作组成的序列，其中可以包含 AI Agent 作为组件。Workflow 的流程是明确定义的，允许对执行路径进行更精确的控制，非常适合处理复杂的业务流程。

![](https://static.xbaby.xyz/blog/workflows-overview.png)



简而言之，您可以将 Workflow 想象成一个流程图，而 AI Agent 则是这个流程图中的一个或多个“智能”节点。

## 核心概念：Executors 与 Edges

Workflow 由两个核心概念组成：**Executors (执行器)** 和 **Edges (边)**。

*   **Executors**: 代表工作流中的单个处理单元。它们可以是 AI Agent，也可以是自定义的业务逻辑组件。
*   **Edges**: 定义了 Executors 之间的连接，决定了消息的流动方向，并可以附加条件来控制路由。

一个 Workflow 本质上就是一个由 Executors 和 Edges 组成的有向图。

---

## ⚙️ Executors (执行器) 详解

Executors 是处理消息的基础构建块。它们是自主的处理单元，接收特定类型的消息，执行操作，并能产生输出消息。

在 C# 中，一个基础的 Executor 结构如下。它通过实现 `IMessageHandler<TInput, TOutput>` 接口来处理输入消息，并可以简单地通过返回一个值来将消息发送给下游连接的 Executor。

```csharp
using Microsoft.Agents.Workflows;
using Microsoft.Agents.Workflows.Reflection;

// 一个将输入字符串转换为大写的 Executor
internal sealed class UppercaseExecutor() : ReflectingExecutor<UppercaseExecutor>("UppercaseExecutor"), IMessageHandler<string, string>
{
    public async ValueTask<string> HandleAsync(string message, IWorkflowContext context)
    {
        string result = message.ToUpperInvariant();
        return result; // 返回值会自动发送给连接的 Executor
    }
}
```

您也可以通过 `IWorkflowContext` 对象手动发送消息，或者通过实现多个 `IMessageHandler` 接口来让一个 Executor 处理多种不同类型的输入。

---

## ↔️ Edges (边) 详解

Edges 定义了消息如何在 Executors 之间流动。它们是工作流图中的连接线，决定了数据流的路径。 框架支持多种强大的 Edge 模式：

### Direct Edges (直接边)

这是最简单的连接方式，用于在两个 Executor 之间建立一对一的直接连接。

```csharp
WorkflowBuilder builder = new(sourceExecutor);
builder.AddEdge(sourceExecutor, targetExecutor);
```

### Conditional Edges (条件边)

只有当满足特定条件时，消息才会通过此 Edge 流动。这对于实现逻辑判断非常有用。 例如，根据垃圾邮件检测的结果，将邮件路由到不同的处理器。

```csharp
// 根据消息内容进行路由
builder.AddEdge(
    source: spamDetector,
    target: emailProcessor,
    condition: result => result is SpamResult spam && !spam.IsSpam
);

builder.AddEdge(
    source: spamDetector,
    target: spamHandler,
    condition: result => result is SpamResult spam && spam.IsSpam
);
```

### Switch-case Edges (Switch-Case 边)

当您需要根据多个不同条件将消息路由到不同 Executor 时，可以使用 Switch-Case 模式，它类似于编程语言中的 `switch` 语句。

```csharp
builder.AddSwitch(routerExecutor, switchBuilder =>
    switchBuilder
        .AddCase(message => message.Priority < Priority.Normal, executorA)
        .AddCase(message => message.Priority < Priority.High, executorB)
        .SetDefault(executorC)
);
```

### Fan-out Edges (扇出边)

将一个 Executor 的消息分发给多个目标。这对于并行处理任务非常有效。

```csharp
// 将消息发送给所有目标
builder.AddFanOutEdge(splitterExecutor, targets: [worker1, worker2, worker3]);
```

### Fan-in Edges (扇入边)

从多个源收集消息并汇集到一个目标 Executor。这通常用于聚合来自多个并行任务的结果。

```csharp
// 聚合来自多个 worker 的结果
builder.AddFanInEdge(aggregatorExecutor, sources: [worker1, worker2, worker3]);
```

## 总结

通过组合使用 **Executors** 和 **Edges**，您可以构建出功能强大且逻辑清晰的自动化工作流。Executors 作为独立的业务处理单元，而 Edges 则像神经网络一样将它们连接起来，实现了灵活的消息路由和流程控制。

希望这篇介绍能帮助您理解 Microsoft Agent Framework 中 Workflow 的基本概念。如果您想深入学习，建议您查阅官方文档以获取更详细的信息和示例。

---