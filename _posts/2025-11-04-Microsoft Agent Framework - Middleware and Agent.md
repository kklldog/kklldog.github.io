在构建 AI Agent 时，经常需要为“执行对话”和“工具调用”加入横切能力（Cross-Cutting Concerns），例如：
- 日志与审计
- 性能与计时
- 异常捕获与统一包装
- 调试与可观测性
- 安全与访问控制

在 Microsoft Agent Framework（`Microsoft.Extensions.AI` / `Microsoft.Agents.AI`）中，可以通过“函数式 Middleware”直接对 Agent 运行生命周期进行切面化增强，无需定义类或接口，实现极简、模块化、可组合的拦截。

本文示例展示两类切面：
1. Run Middleware（拦截整个对话执行）
2. Function Calling Middleware（拦截工具调用）

## 代码基础：创建带工具的 Agent

```csharp
using Azure.AI.OpenAI;
using Microsoft.Agents.AI;
using Microsoft.Extensions.AI;
using OpenAI;
using System.ClientModel;
using System.ComponentModel;

// 配置（生产环境请用安全方式管理密钥）
var azureAiEndpoint = "https://{your-endpoint}.openai.azure.com";
var apiKey = "{YOUR-API-KEY}";

// 1. 定义工具（函数即工具）
[Description("Get user information by a given user name.")]
static string GetUserInfo([Description("The user name")] string userName)
    => $"{userName} is from Suzhou, male, 28 years old";

// 2. 创建 Agent 并注册工具
AIAgent agent = new AzureOpenAIClient(
        new Uri(azureAiEndpoint),
        new ApiKeyCredential(apiKey))
    .GetChatClient("gpt-4o-mini")
    .CreateAIAgent(
        instructions: "You are a helpful assistant",
        tools: [AIFunctionFactory.Create(GetUserInfo)]);
```

这里通过 `AIFunctionFactory.Create` 将一个普通 C# 方法暴露为可被模型自主调用的工具（Function Calling）。

## 切面 1：Run Middleware（对整次 Run 执行做前后拦截）

作用：记录每次请求输入与模型产生的输出消息。

```csharp
// 自定义 Run Middleware
async Task<AgentRunResponse> CustomAgentRunMiddleware(
    IEnumerable<ChatMessage> messages,
    AgentThread? thread,
    AgentRunOptions? options,
    AIAgent innerAgent,
    CancellationToken cancellationToken)
{
    foreach (var chatMessage in messages)
    {
        Console.WriteLine($"Input: {chatMessage}");
    }

    var response = await innerAgent
        .RunAsync(messages, thread, options, cancellationToken)
        .ConfigureAwait(false);

    foreach (var chatMessage in response.Messages)
    {
        Console.WriteLine($"Output: {chatMessage.Contents[0].ToString()}");
    }
    return response;
}

// 注册 Run Middleware
var agentWithRunMiddleware = agent.AsBuilder()
    .Use(runFunc: CustomAgentRunMiddleware, runStreamingFunc: null)
    .Build();

Console.WriteLine(await agentWithRunMiddleware.RunAsync("Show me the information of Jim ?"));
```

### 对应输出场景（Run Middleware 日志）

```
Input: Show me the information of Jim ?
Output: Microsoft.Extensions.AI.FunctionCallContent
Output: Microsoft.Extensions.AI.FunctionResultContent
Output: Jim is a 28-year-old male from Suzhou.
Jim is a 28-year-old male from Suzhou.
```

说明：
- 第一行：原始用户输入
- 中间的 FunctionCall / FunctionResult：模型决定调用工具并返回结果的中间内容
- 最后一行：最终自然语言回答

## 切面 2：Function Calling Middleware（拦截工具调用过程）

作用：对每次工具调用做审计 / 计时 / 参数检查 / 结果加工等。

```csharp
// 自定义 Function Calling Middleware
async ValueTask<object?> CustomFunctionCallingMiddleware(
    AIAgent agent,
    FunctionInvocationContext context,
    Func<FunctionInvocationContext, CancellationToken, ValueTask<object?>> next,
    CancellationToken cancellationToken)
{
    Console.WriteLine($"Function Name: {context.Function.Name}");
    var result = await next(context, cancellationToken);
    Console.WriteLine($"Function Call Result: {result}");
    return result;
}

// 注册 Function Calling Middleware
var agentWithFunctionCallingMiddleware = agent.AsBuilder()
    .Use(CustomFunctionCallingMiddleware)
    .Build();

Console.WriteLine(await agentWithFunctionCallingMiddleware.RunAsync("Show me the information of Jim ?"));
```

### 对应输出场景（Function Calling Middleware 日志）

```
Function Name: _Main_g_GetUserInfo_0_1
Function Call Result: Jim is from Suzhou, male, 28 years old
Jim is a 28-year-old male from Suzhou.
```

说明：
- Function Name：模型选择调用的工具函数内部生成的名称（可用于审计 / 白名单校验）
- Function Call Result：该工具函数真实返回值
- 最后一行：模型基于工具结果生成的最终回答

## 可扩展示例：增加计时切面

你可以继续添加更多函数式 Middleware：

```csharp
async ValueTask<object?> TimingFunctionMiddleware(
    AIAgent agent,
    FunctionInvocationContext context,
    Func<FunctionInvocationContext, CancellationToken, ValueTask<object?>> next,
    CancellationToken ct)
{
    var sw = System.Diagnostics.Stopwatch.StartNew();
    var result = await next(context, ct);
    sw.Stop();
    Console.WriteLine($"Function {context.Function.Name} elapsed: {sw.ElapsedMilliseconds} ms");
    return result;
}

var enhancedAgent = agent.AsBuilder()
    .Use(CustomFunctionCallingMiddleware)
    .Use(TimingFunctionMiddleware)
    .Use(runFunc: CustomAgentRunMiddleware, runStreamingFunc: null)
    .Build();
```

## 常见可插入的切面思路

- 参数校验与拒绝危险输入
- 结果缓存（相同参数短期复用）
- 异常捕获统一格式化返回
- OpenTelemetry 观测（埋点发送 Span / Metric）
- 权限控制（基于用户上下文阻断某些工具）

## 总结

Microsoft Agent Framework 的 Middleware 是以“函数”形态实现的，而不是传统的类 / 接口，这种模式非常接近 JavaScript 生态（如 Express / Koa 的中间件管道）。它充分体现了 C# 在现代版本中的函数式编程能力：一等函数、委托、闭包与组合式构建，使得为 Agent 增加切面逻辑变得简洁、高效、低侵入。

借助这种函数式 Middleware，你可以快速迭代智能体的可观测性、调试性与扩展能力，专注核心价值而非样板结构。