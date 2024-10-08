前几篇我们介绍了如何使用 SK + ollama 跟 LLM 进行基本的对话。如果只是对话的话其实不用什么 SK 也是可以的。今天让我们给 LLM 整点活，让它真的给我们干点啥。
## What is Plugin?
> Plugins are a key component of Semantic Kernel. If you have already used plugins from ChatGPT or Copilot extensions in Microsoft 365, you’re already familiar with them. With plugins, you can encapsulate your existing APIs into a collection that can be used by an AI. This allows you to give your AI the ability to perform actions that it wouldn’t be able to do otherwise.
Behind the scenes, Semantic Kernel leverages function calling, a native feature of most of the latest LLMs to allow LLMs, to perform planning and to invoke your APIs. With function calling, LLMs can request (i.e., call) a particular function. Semantic Kernel then marshals the request to the appropriate function in your codebase and returns the results back to the LLM so the LLM can generate a final response.

以上是微软文档的原话。说人话：    
Plugins 是 SK 的关键组件。基于 Plugins 你可以封装已有的 API 给 AI 使用。这给了 AI 执行动作的能力。在背后，SK 利用了大多数最新大语言模型 (LLMs) 的本地功能调用功能，使LLMs能够进行规划并调用您的API。通过功能调用，LLMs可以请求（即调用）特定的函数。然后，Semantic Kernel 将请求传递给代码库中的相应函数，并将结果返回给LLM，以便LLM生成最终的响应。   
说的更直白一点，Plugins 给 LLM 提供了方法调用的能力。这就比较有意思了。我们知道 LLM 是基于过往的内容训练出来的，也就是说 LLM 并不能回答当前的一些信息，因为它不知道。比如你问它今天有什么新闻，它肯定不知道，因为这不在它的训练集里面。或者你问它今天天气怎么样它也不知道。同样它也没有办法给你做一些特定领域的事情，比如你让他们给某某发一条短信，它做不到，因为它没有这个能力。但是现在有了 Plugin，这一切就有了可能。   

以下让我们使用 SK + Plugin 给 LLM 添加感知天气的能力。当我们问 LLM 某个城市的天气的时候，它能精确的给出回答。   
在我们开始之前还是先试试直接问 LLM 天气问题会得到什么结果：    
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241006003229.png)   

可以看到当我问 What is the weather now of SuZhou ? 后 LLM 直接说它不能获得实时数据。

## 定义 WeatherPlugin

```
using System.ComponentModel;
using Microsoft.SemanticKernel;

namespace SKLearning
{
    public sealed class WeatherPlugin
    {
        [KernelFunction, Description("Gets the weather details of a given location")]
        [return: Description("Weather details")]        
        public static async Task<string> GetWeatherByLocation([Description("name of the location")]string location)
        {
            var key = "...";
            var url = @$"http://api.weatherapi.com/v1/current.json?key={key}&q={location}";
            
            using var client = new HttpClient();
            var response = await client.GetAsync(url);
            var content = await response.Content.ReadAsStringAsync();
            
            Console.WriteLine(content);

            return content;
        }
    }
}
```
一个 plugin 就是一个 C# 的类。在这个类里面承载了一个或N个方法。我们给方法添加描述，给入参，出参添加描述好让 LLM 认识这个方法的作用。这个描述非常重要，请使用尽量简洁明了的语言。    
我们可以看到 WeatherPlugin 里的 GetWeatherByLocation 是通过要给 API 实时获取某个城市的天气信息。这个返回值是 JSON 格式的，并不需要进行特殊的处理，LLM 可以直接识别里面的内容。

## 添加 Plugin 到 kernel
```
var httpClient = new HttpClient();
var builder = Kernel.CreateBuilder()
    .AddOpenAIChatCompletion(modelId: modelId!, apiKey: null, endpoint: endpoint, httpClient: httpClient);
var kernel = builder.Build();
kernel.Plugins.AddFromType<WeatherPlugin>();
```
以上代码片段演示了如何添加 OpenAI 的 Chat 服务以及如何添加 Plugin 。

## 与 AI 对话
```
var settings = new OpenAIPromptExecutionSettings()
    { Temperature = 0.0, ToolCallBehavior = ToolCallBehavior.AutoInvokeKernelFunctions };
var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();
var history = new ChatHistory();
var initMessage =
    "I am a weatherman. I can tell you the weather of any location. Try asking me about the weather of a location.";
history.AddSystemMessage(initMessage);
Console.WriteLine(initMessage);

while (true)
{
    Console.BackgroundColor = ConsoleColor.Black;
    Console.Write("You:");

    var input = Console.ReadLine();

    if (string.IsNullOrWhiteSpace(input))
    {
        break;
    }

    history.AddUserMessage(input);
    // Get the response from the AI
    var contents = await chatCompletionService.GetChatMessageContentsAsync(history, settings, kernel);

    foreach (var chatMessageContent in contents)
    {
        var content = chatMessageContent.Content;
        Console.BackgroundColor = ConsoleColor.DarkGreen;
        Console.WriteLine($"AI: {content}");
        history.AddMessage(chatMessageContent.Role, content ?? "");
    }
}
```
以上内容跟上次演示的内容没啥特别大的区别，无非就是读取用户的输入，等待 LLM 的回答。

## 试一下
让我们运行程序，然后再次问 What is the weather now of SuZhou？
可以看到 LLM 精确的回答出了当前苏州的天气：
> I am a weatherman. I can tell you the weather of any location. Try asking me about the weather of a location.

> You: What is the weather now of SuZhou?

> AI: The current weather in Suzhou, China is patchy light drizzle with a temperature of 17.2°C (62.9°F). The wind is blowing at 3.6 mph (5.8 kph) from the NNE direction. The humidity is 86% and the visibility is 5 km (3 miles).


![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241006001041.png)


## 总结
通过以上演示，我们自定义了一个实时获取天气信息的 plugin 给 LLM 使用。当我们问到天气信息的时候 LLM 会实时调用这个方法，然后使用方法结果构造一个可读性非常高的回答。有了 plugin 之后 LLM 真正的可以触发一些动作，执行一些任务，获取实时信息了。
    
希望此文对你有所帮助，谢谢。