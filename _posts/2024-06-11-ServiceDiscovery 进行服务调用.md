当然可以，以下是完整的 Markdown 版本：

---

# 使用 Microsoft.Extensions.ServiceDiscovery 进行服务发现并调用

## 简介
在现代微服务架构中，服务发现（Service Discovery）是一项关键功能。它允许微服务动态地找到彼此，而无需依赖硬编码的地址。以前如果你搜 .NET Service Discovery，大概率会搜到一大堆 Eureka，Consul 等的文章。现在微软为我们带来了一个官方的包：Microsoft.Extensions.ServiceDiscovery。这个包出自 Aspire 项目，提供了一个简便的方式在 .NET 中实现服务发现。

## 安装 Nuget 包
首先，需要安装 Microsoft 提供的 Service Discovery 包。使用以下命令添加包到你的项目中：
```bash
dotnet add package Microsoft.Extensions.ServiceDiscovery
```
这一步确保你的项目具有使用 Service Discovery 所需的依赖项。

## 配置和注册服务
接下来，需要在项目中配置和注册 Service Discovery。打开 `Program.cs` 或 `Startup.cs` 文件，并添加以下代码：
```csharp
builder.Services.AddServiceDiscovery();

builder.Services.ConfigureHttpClientDefaults(static http =>
{
    http.AddServiceDiscovery();
});
```
这段代码将 Service Discovery 注册到依赖注入容器中，并配置默认的 HTTP 客户端使用 Service Discovery。

## 配置服务端点
为了让 Service Discovery 知道如何找到其他服务，需要在配置文件（如 `appsettings.json`）中定义服务端点。例如：
```json
{
  "Services": {
    "weatherReport": {
      "http": [
        "localhost:5089",
        "127.0.0.1:5089"
      ],
      "https": []
    }
  }
}
```
在这个配置中，我们定义了名为 `weatherReport` 的服务的 HTTP 端点。Service Discovery 将使用这些信息来查找和访问该服务。

## 使用服务名进行 HTTP 调用
配置完成后，可以通过`服务名称`进行 HTTP 调用。以下代码展示了如何使用 `IHttpClientFactory` 进行服务调用：
```csharp
app.MapGet("/report", async (IHttpClientFactory factory) =>
{
    const string serviceName = "weatherReport";
    var client = factory.CreateClient();
    var response = await client.GetAsync($"http://{serviceName}/weatherforecast");
    var content = await response.Content.ReadAsStringAsync();

    return content;
});
```
这段代码创建了一个 HTTP 客户端，通过服务名 `weatherReport` 发起请求，并返回响应内容。

启动服务后尝试进行调用：   

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20240611015909.png
)    

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20240611020622.png)

通过观察日志可以看到 http://weatherreport/weatherforecast 被转换成 http://127.0.0.1:5089 或 http://localhost:5089 的 http 调用。    

## 负载均衡
如果服务配置了多个 endpoint 。 那么进行服务调用的时候我们往往需要按实际情况配置 Load-balancing 的策略:
```
builder.Services.AddHttpClient<CatalogServiceClient>(
    static client => client.BaseAddress = new("http://weatherReport"));
  .AddServiceDiscovery(RandomServiceEndpointSelector.Instance);
```

- PickFirstServiceEndpointSelectorProvider.Instance: 总是调用第一个

- RoundRobinServiceEndpointSelectorProvider.Instance: 轮询调用

- RandomServiceEndpointSelectorProvider.Instance: 随机调用

- PowerOfTwoChoicesServiceEndpointSelectorProvider.Instance： 解释太长看英文原文吧。Power-of-two-choices, which attempts to pick the least heavily loaded endpoint based on the Power of Two Choices algorithm for distributed load balancing, degrading to randomly selecting an endpoint when either of the provided endpoints do not have the IEndpointLoadFeature  

## 总结
Service Discovery 是实现微服务架构的重要组件。在 .NET 中，通过简单的配置和使用，可以不用 hardcode IP 跟 port 而使用服务名，可以大大简化服务间的调用。同时还能配置不同的调用策略，进行负载均衡。