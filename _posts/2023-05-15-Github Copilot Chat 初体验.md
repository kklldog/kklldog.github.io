最近因为阳了的缘故一直躺在床上。今天终于从床上爬起来了。不是因为好透了，而是因为我收到了申请Copilot Chat preview 权限通过的邮件。实在忍不住，于是起床开电脑在咳嗽声中进行了一番体验。
## 如何申请
https://github.com/github-copilot/chat_waitlist_signup/join
## 安装插件
申请通过后，邮件上就有说明。插件有2个，一个是基于 VSCode 的，一个是基于 Visual Studio 的。今天使用的是 Visual Studio 版本。
在 Marketplace 搜索 Visual Studio Extension for GitHub Copilot Preview 下载安装重启 VS 即可。
## 使用
在安装完插件后，随便打开一个项目，随便选择一个类，右键就有"询问 Copilot" 菜单：  
![](https://static.xbaby.xyz/QQ%E6%88%AA%E5%9B%BE20230515010920.png)   
点击菜单后，下面就会出现一个对话框:   
![](https://static.xbaby.xyz/QQ%E6%88%AA%E5%9B%BE20230515010958.png)   
用户可以在对话框进行提问   
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20230515011444.png)   
同时在侧边栏也会多出一个 Github Copilot chat 的 tab。

## 对话/问答
Copilot 的基本能力，比如写单元测试等，其实在以前的文章已经演示过了，这次不在重复。既然新版叫 Copilot Chat 那么肯定是跟 ChatGPT 一样是问答模式了。下面就来跟 Copilot 唠唠嗑。
   
minjie：
Copilot Chat 是什么？
Copilot:
```
Copilot Chat是 Microsoft的GitHub Copilot AI的一种特性，允许开发人员使用文本输入与AI交互，并实时获得代码建议和补全。Copilot Chat是一种旨在通过提供基于检测到的代码模式的建议来增强开发人员的生产力的工具。Copilot Chat使用机器学习算法识别模式并为各种编程活动建议代码片段。它还具有学习上一个用户交互的能力，随着时间的推移将变得更加准确。如果您需要使用Copilot Chat获取帮助进行开发工作，请在有具体问题的情况下描述清晰问题的场景与具体需求，以便我更好地帮助您。
```
minjie：
请问 c# int 的最大值是多少
Copilot:
```
The maximum value of an int in C# is 2,147,483,647 (or 0x7FFFFFFF in hexadecimal). This is because an int in C# is a signed 32-bit integer, meaning it can hold values between -2,147,483,648 and 2,147,483,647 inclusive.
Here's an example of assigning int.MaxValue to a variable in C#:
int maxVal = int.MaxValue;
This creates a new variable called maxVal of type int and assigns it the value of the maximum possible integer.
```
minjie：
在 dotnet 中一般怎么建立 websocket 连接，给个示例   
Copilot:
```
using System.Net.WebSockets;
using System.Threading.Tasks;

public async Task ConnectWebSocketAsync(string url) {
    using (ClientWebSocket ws = new ClientWebSocket()) {
        await ws.ConnectAsync(new System.Uri(url), CancellationToken.None);
        await Task.WhenAll(Receive(ws), Send(ws));
    }    
}

private static async Task Receive(ClientWebSocket ws) {
    byte[] buffer = new byte[1024 * 4];
    while (ws.State == WebSocketState.Open) {
        var result = await ws.ReceiveAsync(new ArraySegment<byte>(buffer), CancellationToken.None);
        if (result.MessageType == WebSocketMessageType.Close) {
            await ws.CloseAsync(WebSocketCloseStatus.NormalClosure, "", CancellationToken.None);
        } else {
            // Process received data.
        }
    }
}

private static async Task Send(ClientWebSocket ws) {
    while (ws.State == WebSocketState.Open) {
        // Get data to send.
        byte[] buffer = new byte[1024 * 4];
        var result = await ws.SendAsync(new ArraySegment<byte>(buffer), WebSocketMessageType.Text, true, CancellationToken.None);
        await Task.Delay(2000);
    }
}

```
问答可以说问啥答啥，基本上毫无破绽。
## 对代码进行解释
Copilot Chat 的另外一个能力就是对一段代码进行解释
比如让它对 AgileConfigClient 的核心类 ConfigClient 进行解释。首先打开 AgileConfigClient 项目，然后选中 ConfigClient 类文件，然后就可以问了。   

minjie：
请问 ConfigClient 类是用来干嘛的？   
Copilot:
```
ConfigClient is a class that allows the management and retrieval of configuration data. The class has methods that load configurations from a server and save them to a local storage. The configuration is typically represented in key/value pairs. The Get method is used to retrieve a particular key, and the GetGroup method is used to retrieve all keys under a common group name.
Here's an example usage of the ConfigClient class:
var config = new ConfigClient();
var result = config.Get("keyName");
This creates a new instance of the ConfigClient object and retrieves the value associated with the specified "keyName" string.

```
Copilot 准确的做出了解释，只是使用了英文，我让它使用中文，它不理我。

## 使用 Copilot Chat 一步步生成一个 To-do List 的 webapi 项目
下面的内容演示了如何使用 Copilot chat 一步步通过对话方式生成一个可以运行的项目。在最后我甚至想让它直接写个文章，Copilot 也写出来了，不过反正见过了 ChatGPT 的能力了也不奇怪哈。

minjie：
生成一个ToDo实体类   
Copilot:
```
public class ToDo
{
    public int Id { get; set; }
    public string Task { get; set; }
}

```
minjie：
添加一个字段 Completed   
Copilot:
```
public class ToDo
{
    public int Id { get; set; }
    public string Task { get; set; }
    public bool Completed { get; set; }
}
```
minjie：
对这个实体类生产数据库访问类，使用 efcore   
Copilot:
```
using Microsoft.EntityFrameworkCore;
using System;

public class MyContext : DbContext
{
    public MyContext(DbContextOptions<MyContext> options) : base(options)
    { }

    public DbSet<MyEntity> MyEntities { get; set; }
}
```
minjie:
改成 TodoContext    
Copilot:
```
using Microsoft.EntityFrameworkCore;
using System;

public class TodoContext : DbContext
{
    public TodoContext(DbContextOptions<TodoContext> options) : base(options)
    { }

    public DbSet<ToDo> ToDos { get; set; }
}

```
minjie:
支持 sqlite 
Copilot:
```
using Microsoft.EntityFrameworkCore;
using System;

public class TodoContext : DbContext
{
    public TodoContext(DbContextOptions<TodoContext> options) : base(options)
    { }

    public DbSet<ToDo> ToDos { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlite("Data Source=todos.db");
}
```
minjie:
删除构造函数  
Copilot:
```
using Microsoft.EntityFrameworkCore;
using System;

public class TodoContext : DbContext
{
    public DbSet<ToDo> ToDos { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlite("Data Source=todos.db");
}
```
minjie:
使用 ToDoContext 生成一个 ToDoService 类，这个类包装了该删查方法  
Copilot:
```
public class ToDoService
{
    private readonly TodoContext _context;

    public ToDoService(TodoContext context)
    {
        _context = context;
    }

    public List<ToDo> GetToDos()
    {
        return _context.ToDos.ToList();
    }

    public ToDo GetToDoById(int id)
    {
        return _context.ToDos.Find(id);
    }

    public void AddToDo(ToDo todo)
    {
        _context.ToDos.Add(todo);
        _context.SaveChanges();
    }

    public void UpdateToDoById(int id, ToDo todo)
    {
        var existingTodo = _context.ToDos.Find(id);

        if (existingTodo != null)
        {
            existingTodo.Task = todo.Task;
            existingTodo.Completed = todo.Completed;

            _context.SaveChanges();
        }
    }

    public void RemoveToDoById(int id)
    {
        var existingTodo = _context.ToDos.Find(id);

        if (existingTodo != null)
        {
            _context.ToDos.Remove(existingTodo);
            _context.SaveChanges();
        }
    }
}

```
minjie:
把所有方法改成异步  
Copilot:
```
using CopilotChatPreview;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;

public class ToDoService
{
    private readonly TodoContext _context;

    public ToDoService(TodoContext context)
    {
        _context = context;
    }

    public async Task<List<ToDo>> GetToDos()
    {
        return await _context.ToDos.ToListAsync();
    }

    public async Task<ToDo> GetToDoById(int id)
    {
        return await _context.ToDos.FindAsync(id);
    }

    public async Task AddToDoAsync(ToDo todo)
    {
        _context.ToDos.Add(todo);
        await _context.SaveChangesAsync();
    }

    public async Task UpdateToDoByIdAsync(int id, ToDo todo)
    {
        var existingTodo = await _context.ToDos.FindAsync(id);

        if (existingTodo != null)
        {
            existingTodo.Task = todo.Task;
            existingTodo.Completed = todo.Completed;

            await _context.SaveChangesAsync();
        }
    }

    public async Task RemoveToDoByIdAsync(int id)
    {
        var existingTodo = await _context.ToDos.FindAsync(id);

        if (existingTodo != null)
        {
            _context.ToDos.Remove(existingTodo);
            await _context.SaveChangesAsync();
        }
    }
}

```
minjie:
生产一个控制器并且使用 ToDoService，使用 restful 方式暴露出去  
Copilot:
```
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("[controller]")]
public class ToDoController : ControllerBase
{
    private readonly ToDoService _toDoService;

    public ToDoController(ToDoService toDoService)
    {
        _toDoService = toDoService;
    }

    [HttpGet]
    public async Task<ActionResult<List<ToDo>>> GetAllToDos()
    {
        return await _toDoService.GetToDos();
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<ToDo>> GetToDoById(int id)
    {
        var todo = await _toDoService.GetToDoById(id);

        if (todo == null)
        {
            return NotFound();
        }

        return todo;
    }

    [HttpPost]
    public async Task<IActionResult> CreateToDoAsync(ToDo todo)
    {
        await _toDoService.AddToDoAsync(todo);

        return CreatedAtAction(nameof(GetToDoById), new { id = todo.Id }, todo);
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateToDoByIdAsync(int id, ToDo todo)
    {
        await _toDoService.UpdateToDoByIdAsync(id, todo);

        return NoContent();
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteToDoByIdAsync(int id)
    {
        await _toDoService.RemoveToDoByIdAsync(id);

        return NoContent();
    }
}

```
minjie:
在程序启动的时候把ToDoService注册到容器并且使用scope生命周期  
Copilot:
```
// In Startup.cs file, add the following code to ConfigureServices method
services.AddScoped<ToDoService>();

```

minjie:
可以把以上对话整理成一篇文章吗？文章的题目叫 CopilotChat 初体验   
```
CopilotChat 初体验
在这篇文章中，我们将一起学习如何使用 Microsoft 的 AI 编程助手，GitHub Copilot 来生成 C# 代码。同时，我们将使用一个 To-Do 应用来展示 Copilot 的功能。
注册 Copilot
要使用 GitHub Copilot，你需要注册并安装 GitHub Copilot 扩展。在安装时，你需要登录你的 GitHub 帐户和选择你喜欢的语言。
准备工作
我们将使用 Entity Framework Core 和 ASP.NET Core 创建 To-Do 应用程序。以下是创建应用程序的基本步骤。
步骤 1：创建数据库上下文
我们首先需要创建数据上下文。我们创建一个名为 TodoContext 的类，在其中我们将 ToDo 实体框架创建为数据库表。同时还将添加一个连接字符串以及一些模型构建代码。
using Microsoft.EntityFrameworkCore;

namespace CopilotChatPreview
{
    public class TodoContext : DbContext
    {
        public TodoContext(DbContextOptions<TodoContext> options) : base(options) {}

        public DbSet<ToDo> ToDos { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<ToDo>().HasData(
                new ToDo
                {
                    Id = 1,
                    Task = "Learn AI programming",
                    Completed = false
                },
                new ToDo
                {
                    Id = 2,
                    Task = "Build a chatbot",
                    Completed = false
                });
        }
    }
}


步骤 2：创建服务类
我们创建一个 ToDoService 类，用于在数据库中执行增删改查操作。
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;

public class ToDoService
{
    private readonly TodoContext _context;

    public ToDoService(TodoContext context)
    {
        _context = context;
    }

    ...

    public async Task RemoveToDoByIdAsync(int id)
    {
        var existingTodo = await _context.ToDos.FindAsync(id);

        if (existingTodo != null)
        {
            _context.ToDos.Remove(existingTodo);
            await _context.SaveChangesAsync();
        }
    }
}

步骤 3：创建 To-Do 控制器
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("[controller]")]
public class ToDoController : ControllerBase
{
    private readonly ToDoService _toDoService;

    public ToDoController(ToDoService toDoService)
    {
        _toDoService = toDoService;
    }

    ...

    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteToDoByIdAsync(int id)
    {
        await _toDoService.RemoveToDoByIdAsync(id);

        return NoContent();
    }
}

步骤 4：注册服务
在 ConfigureServices 方法中，我们需要将数据库上下文和服务类注册到容器中。
public void ConfigureServices(IServiceCollection services)
{
    ...
    services.AddDbContextPool<TodoContext>(options => options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
    services.AddScoped<ToDoService>();
    ...
}

总结
在这篇文章里，我们介绍了如何使用 AI 编程助手 Copilot 来生成 C# 代码。我们还介绍了如何使用 Entity Framework Core 创建和操纵数据库表。通过这篇文章，你已经开始掌握了如何使用 AI 编程助手来让开发 C# 应用程序变得更加轻松和高效。

```

## 总结
本想让 Copilot 帮我总结一下，最后还是我自己来总结吧。本次初步体验了 Copilot chat 的一些基本能力。比如如何问答，如何分析代码。特别是最后一步步全程使用对话方式生成一个可以运行的项目的内容，还是挺精彩的。作为一个 AI 助手是完全合格的。不过说实话没有第一次使用 Copilot 的时候的那种震撼。为啥？因为 ChatGPT 珠玉在前，短短的一段时间，已经让我们把对 AI 的期望拉到了前所未有的高度。Copilot chat 作为 ChatGPT 模型在垂直领域的实践，还需好好打磨，让我们期待一下正式版吧。
