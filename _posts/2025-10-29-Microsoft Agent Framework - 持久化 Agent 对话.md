好的，这是更新后的版本，在步骤四中特别强调了 Agent 如何利用恢复的上下文来“记住”之前的对话。

---

## Microsoft Agent Framework - 持久化 Agent 对话

在构建高级 AI 助手时，一个核心需求是能够跨多个会话记住对话历史。用户期望能够随时中断对话，并在稍后返回时从上次离开的地方继续。Microsoft Agent Framework 提供了一套强大的工具，可以轻松实现对话的持久化。

本文将通过一个具体的示例，演示如何序列化一个 Agent 对话线程，将其保存到数据库中，然后在需要时加载并恢复对话。

### 核心概念

在深入代码之前，我们先了解几个核心概念：

*   `AIAgent`: 代表我们的 AI 智能体，负责处理和响应用户输入。
*   `AgentThread`: 代表一个独立的对话线程。它包含了对话的完整上下文和历史记录。
*   **序列化/反序列化**: Agent Framework 允许我们将 `AgentThread` 对象转换为一种可存储的格式（如 JSON 字符串），并在以后将其转换回原始对象，从而恢复对话状态。

### 步骤 1: 初始化 Agent

首先，我们需要创建一个 `AIAgent` 实例。在这个例子中，我们使用 Azure OpenAI 服务。您需要提供终结点（Endpoint）和 API 密钥。

```csharp
var azureAiEndpoint = "https://<your-resource-name>.openai.azure.com";
var apiKey = "<your-api-key>";

AIAgent agent = new AzureOpenAIClient(
        new Uri(azureAiEndpoint),
        new ApiKeyCredential(apiKey))
    .GetChatClient("gpt-4o-mini")
    .CreateAIAgent(new ChatClientAgentOptions()
    {
        Name = "智能助手",
        Instructions = "你是一个智能助手，可以回答客户的问题.",
    });
```

### 步骤 2: 定义数据模型和数据库上下文

为了存储对话，我们使用 Entity Framework Core。首先，我们定义一个简单的 `Conversation` 实体来保存对话 ID 和序列化后的上下文。然后，我们创建一个 `ConversationDbContext` 来与数据库交互。在这个例子中，我们使用 SQLite 作为我们的数据库，因为它轻量且易于设置。

以下是 `Conversation` 实体和 `ConversationDbContext` 的定义：

```csharp
public class Conversation
{
    public string Id { get; set; }

    public string Context { get; set; }
}

internal class ConversationDbContext: DbContext
{
    public DbSet<Conversation> Conversations { get; set; }
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlite("Data Source=ConversationDb.db");
    }
}
```

`Conversation` 类很简单，`Id` 作为唯一标识符，`Context` 用来存储 JSON 格式的对话历史。`ConversationDbContext` 通过 `OnConfiguring` 方法配置了 SQLite 数据库连接，并将 `Conversation` 实体映射到数据库中的 `Conversations` 表。

### 步骤 3: 开始新对话并持久化

当一个新对话开始时，我们创建一个新的 `AgentThread`。在与 Agent 交互后，我们将该线程序列化为 JSON 字符串，并使用我们刚刚定义的 `ConversationDbContext` 将其保存到数据库中。

```csharp
// 1. 创建新线程并发起对话
var thread = agent.GetNewThread();
var response = await agent.RunAsync("谁是迈克尔杰克逊?", thread);
Console.WriteLine(response);

// 2. 将线程序列化为 JSON 字符串
string serializedJson = thread.Serialize(JsonSerializerOptions.Web).GetRawText();

// 3. 创建实体并保存到数据库
var conversation = new Conversation()
{
    Id = Guid.NewGuid().ToString(),
    Context = serializedJson
};

var db = new ConversationDbContext();
db.Database.EnsureCreated();
db.Conversations.Add(conversation);
await db.SaveChangesAsync();
```

此时，包含完整对话上下文的 JSON 字符串已经存储在 SQLite 数据库 `ConversationDb.db` 的 `Conversations` 表中。

### 步骤 4: 加载对话并利用记忆继续交互

这是实现对话持久化最关键的一步。我们将从数据库中加载序列化的上下文，并将其“注入”回 Agent，从而恢复其记忆。

```csharp
// 1. 从数据库加载已保存的对话
var savedConversation = db.Conversations.First(c => c.Id == conversation.Id);
JsonElement reloaded = JsonSerializer.Deserialize<JsonElement>(savedConversation.Context, JsonSerializerOptions.Web);

// 2. 使用相同的 Agent 实例反序列化线程，恢复对话历史
AgentThread resumedThread = agent.DeserializeThread(reloaded, JsonSerializerOptions.Web);

// 3. 继续对话
var resumedResponse = await agent.RunAsync("他的代表作品有哪些?", resumedThread);
Console.WriteLine(resumedResponse);
```

这里的核心是 `agent.DeserializeThread` 方法。它接收之前存储的 JSON 上下文，并重建了包含所有历史消息的 `AgentThread` 对象。

当我们将这个 `resumedThread` 传递给 `agent.RunAsync` 时，Agent 并不是从零开始。它能够“看到”整个对话历史。在我们的例子中：

1.  **第一轮对话**：我们问了“谁是迈克尔杰克逊?”。
2.  **对话被保存和恢复**。
3.  **第二轮对话**：我们问“他的代表作品有哪些?”。

由于 `resumedThread` 包含了第一轮对话的上下文，Agent 能够准确理解问题中的“他”指的就是“迈克尔·杰克逊”。它利用这个记忆来提供一个相关的、上下文感知的回答，而不是要求用户澄清或重复信息。

这种无缝恢复上下文的能力，正是构建能够进行长期、有意义对话的智能助手的关键。

### 总结

通过利用 Microsoft Agent Framework 内置的序列化功能和 Entity Framework Core，我们可以轻松地实现 Agent 对话的持久化。这种能力对于构建需要长期记忆和上下文感知能力的复杂、真实世界的 AI 应用程序至关重要。无论是用于客户支持、个人助理还是其他场景，持久化对话都能极大地提升用户体验和 Agent 的实用性。