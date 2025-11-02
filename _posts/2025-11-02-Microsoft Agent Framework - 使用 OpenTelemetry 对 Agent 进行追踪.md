好的，这是一篇关于在 Microsoft Agent Framework 中使用 OpenTelemetry 的技术博客文章。

---

## Microsoft Agent Framework - 使用 OpenTelemetry 对 Agent 进行追踪

在构建复杂的 AI 代理（Agent）系统时，理解其内部行为、诊断问题和监控性能至关重要。可观测性（Observability）是现代软件开发的基石，而 OpenTelemetry 作为业界标准，为我们提供了强大的工具集来实现分布式追踪。

本文将向您展示如何将 OpenTelemetry 集成到 Microsoft Agent Framework 中，以捕获和可视化 Agent 的执行流程。

### 什么是 OpenTelemetry？

OpenTelemetry (OTEL) 是一个开源的可观测性框架，由云原生计算基金会 (CNCF) 支持。它提供了一组标准的 API、SDK 和工具，用于生成、收集和导出遥测数据（如追踪、指标和日志），帮助您更好地理解您的软件。

### 准备工作

在开始之前，请确保您已安装 .NET SDK。我们将创建一个简单的控制台应用程序来演示整个过程。

首先，我们需要安装几个必要的 NuGet 包：

*   **Microsoft.Agents.AI**: Agent Framework 的核心库。
*   **OpenTelemetry**: OpenTelemetry 的核心 API 和 SDK。
*   **OpenTelemetry.Exporter.Console**: 将追踪数据导出到控制台，便于快速调试。
*   **OpenTelemetry.Exporter.OpenTelemetryProtocol**: 使用 OTLP 协议将数据发送到兼容的后端，如 Jaeger、Zipkin 或 .NET Aspire 仪表板。

您可以使用以下命令安装这些包：

```sh
dotnet add package Microsoft.Agents.AI
dotnet add package OpenTelemetry
dotnet add package OpenTelemetry.Exporter.Console
dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol
```

### 第 1 步：配置 OpenTelemetry Tracer

第一步是配置 `TracerProvider`。这是 OpenTelemetry 的核心组件，它负责处理由应用程序生成的追踪数据，并将其发送到指定的导出器（Exporter）。

在您的 `Program.cs` 文件中，添加以下代码来构建一个 `TracerProvider`：

```csharp
using OpenTelemetry;
using OpenTelemetry.Trace;

using var tracerProvider = Sdk.CreateTracerProviderBuilder()
    .AddSource("agent-telemetry-source")
    .AddConsoleExporter()
    .AddOtlpExporter(op =>
    {
        op.Endpoint = new Uri("http://localhost:4317");
    })
    .Build();
```

让我们分解一下这段代码：

*   `Sdk.CreateTracerProviderBuilder()`: 创建一个用于配置追踪的构建器。
*   `.AddSource("agent-telemetry-source")`: 指定要监听的追踪源（`ActivitySource`）的名称。只有来自这个源的追踪数据才会被处理。您可以把它看作一个命名空间，用于隔离不同组件的遥测数据。
*   `.AddConsoleExporter()`: 添加一个控制台导出器。这会将追踪信息直接打印到控制台，非常适合在开发初期进行验证。
*   `.AddOtlpExporter(...)`: 添加 OTLP 导出器，它允许我们将数据发送到标准的可视化后端。这里我们将其配置为本地运行的收集器端点 `http://localhost:4317`。

### 第 2 步：将 OpenTelemetry 集成到 Agent

Microsoft Agent Framework 提供了便捷的扩展方法，可以轻松地将 OpenTelemetry 集成到 Agent 的构建流程中。

在创建 Agent 时，只需调用 `.UseOpenTelemetry()` 方法即可：

```csharp
using Azure.AI.OpenAI;
using Microsoft.Agents.AI;
using OpenAI;
using System.ClientModel;

// ... (TracerProvider 配置代码)

var azureAiEndpoint = "https://YOUR_AZURE_OPENAI_ENDPOINT.openai.azure.com";
var apiKey = "YOUR_API_KEY";

AIAgent agent = new AzureOpenAIClient(
        new Uri(azureAiEndpoint),
        new ApiKeyCredential(apiKey))
    .GetChatClient("gpt-4o-mini")
    .CreateAIAgent(new ChatClientAgentOptions()
    {
        Name = "智能助手",
        Instructions = "你是一个智能助手，可以回答客户的问题.",
    })
    .AsBuilder()
    .UseOpenTelemetry(sourceName: "agent-telemetry-source") // 启用 OpenTelemetry
    .Build();

// 运行 Agent
var thread = agent.GetNewThread();
var response = await agent.RunAsync("谁是迈克尔杰克逊?", thread);
Console.WriteLine(response);
```

关键在于 `.UseOpenTelemetry(sourceName: "agent-telemetry-source")` 这一行。它做了两件事：
1.  它指示 Agent Framework 在执行期间生成追踪活动（Activities/Spans）。
2.  它将这些活动的源名称设置为 `"agent-telemetry-source"`，这与我们在 `TracerProvider` 中配置的 `AddSource` 名称完全匹配，从而确保这些追踪数据能被正确捕获和导出。

### 第 3 步：使用 .NET Aspire 仪表板进行可视化

虽然控制台输出很有用，但图形化界面能让我们更直观地理解分布式追踪。`.NET Aspire Dashboard` 是一个轻量级的本地遥测数据查看器，非常适合本地开发。

您可以使用 Docker 轻松运行它：

```sh
docker run --rm -it -p 18888:18888 -p 4317:18889 -d --name aspire-dashboard mcr.microsoft.com/dotnet/aspire-dashboard:8.0
```

这条命令会：
*   从 Microsoft Container Registry (MCR) 拉取 Aspire 仪表板镜像。
*   将主机的 `18888` 端口映射到容器的 `18888` 端口（用于访问 Web UI）。
*   将主机的 `4317` 端口（我们的 OTLP 导出器目标端口）映射到容器的 `18889` 端口（Aspire 仪表板的 OTLP/gRPC 监听端口）。

启动容器后，执行您的 .NET 应用程序。然后，在浏览器中打开 `http://localhost:18888`。

![](https://static.xbaby.xyz/ScreenShot_2025-11-02_172530_351.png)

在仪表板的 "Traces" 选项卡中，您将看到 `agent.RunAsync` 的完整调用链。您可以点击追踪记录，查看每个操作（Span）的耗时、层级关系以及详细属性，从而清晰地了解 Agent 在处理请求时内部发生了什么。
为了方便您参考，这里是完整的 `Program.cs` 文件：

### 总结

通过简单的几步配置，我们成功地为 Microsoft Agent Framework 集成了 OpenTelemetry，实现了对 Agent 行为的端到端追踪。这不仅极大地提升了开发和调试效率，也为后续的性能优化和生产环境监控奠定了坚实的基础。立即开始在您的 Agent 项目中实践吧！