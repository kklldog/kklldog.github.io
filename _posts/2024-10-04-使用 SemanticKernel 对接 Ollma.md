前面的 2 篇文章已经介绍了 ollama 的基本情况。我们也已经能在本地跟 LLM 进行聊天了。但是如何使用代码跟 LLM 进行交互呢？如果是 C# 选手那自然是使用 SK (SemanticKernel) 了。在这篇博客中，我们将探讨如何使用 Microsoft 的 SemanticKernel 框架对接 Ollama 的聊天服务。我们将通过一个简单的 C# 控制台应用程序来展示如何实现这一点。

## 前提条件
在本地安装 ollama 服务，并且安装至少一个模型，这次我们的模型是 `llama3.1:8b`。具体如何安装就不赘述了，请参考以往文章：

## 安装 SK 及 ollama connector
首先在本地创建一个 Console 项目，然后安装以下包：
```
dotnet add package Microsoft.SemanticKernel --version 1.21.1
dotnet add package Microsoft.SemanticKernel.Connectors.Ollama --version 1.21.1-alpha
```
> 注意：ollama connector 还是 alpha 版本，请勿用于生产

## 修改 Program 文件

### 添加命名空间

首先，我们需要引入一些必要的命名空间：
```
using Microsoft.Extensions.DependencyInjection;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Connectors.Ollama;

```

### 配置 Ollama 服务
接下来，我们需要配置 Ollama 服务的端点和模型 ID ，并添加 Ollama 的聊天服务：


```
var endpoint = new Uri("http://localhost:11434");
var modelId = "llama3.1:8b";

var builder = Kernel.CreateBuilder();
#pragma warning disable SKEXP0070 
builder.Services.AddScoped<IChatCompletionService>(_ => new OllamaChatCompletionService(modelId, endpoint));

```
> 注意：OllamaChatCompletionService 为实验性质所以我们需求手工关闭 SKEXP0070 的警告

### 获取聊天服务

```
var chatService = kernel.GetRequiredService<IChatCompletionService>();
var history = new ChatHistory();
history.AddSystemMessage("This is a llama3 assistant ...");

```

### 聊天循环
最后，我们实现一个简单的聊天循环，读取用户输入并获取 Ollama 的回复：
```
while (true)
{
    Console.Write("You:");

    var input = Console.ReadLine();

    if (string.IsNullOrWhiteSpace(input))
    {
        break;
    }

    history.AddUserMessage(input);

    var contents = await chatService.GetChatMessageContentsAsync(history);

    foreach (var chatMessageContent in contents)
    {
        var content = chatMessageContent.Content;
        Console.WriteLine($"Ollama: {content}");
        history.AddMessage(chatMessageContent.Role, content ?? "");
    }
}

```
### 试一下
让我们运行项目在 Console 中跟 ollama 进行对话吧。   

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241004014155.png)

## 总结

通过这篇博客，我们展示了如何使用 Microsoft 的 SemanticKernel 框架对接 Ollama 的聊天服务。希望这篇博客能帮助您更好地理解和使用这些工具。如果您有任何问题或建议，请随时在评论区留言。