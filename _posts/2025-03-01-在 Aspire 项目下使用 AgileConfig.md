## 什么是 Aspire


.NET Aspire 是一组工具、模板和包，用于构建易于监控的、可投入生产的应用程序。.NET Aspire 通过一系列 NuGet 包交付，这些包通过启动或解决现代应用开发中的特定问题来提升开发效率。 如今的应用通常使用大量服务，例如数据库、消息传送和缓存，其中许多服务通过 .NET Aspire 集成得到支持。    
Aspire 是微软发布的一项新技术。最近社区也有人跟我提需求说 AgileConfig 要支持 Aspire。   
因为这不是 Aspire 的介绍文章，所以不过多表述。想要了解可参考以下文档：
https://learn.microsoft.com/zh-cn/dotnet/aspire/get-started/aspire-overview

## 使用 AgileConfig 的传统方式
通常我们使用 AgileConfig 至少需要以下步骤：
1. 使用 docker run 命令把服务端跑起来
2. 配置 admin 密码
3. 添加应用，设置 appId，secret
4. 在客户端项目添加 client 包，修改 appsettings 配置文件

通过以上步骤后，你的应用至少应该是能成功连上 AgileConfig 服务端了。

## 在 Aspire 下使用 AgileConfig
下面让我们看看如何在 Aspire 下使用 AgileConfig。    
相信大家肯定看过一些 Aspire 的案例。一些 infrastructure 的组件（比如 Sqlserver 数据库），可以通过 Aspire 直接运行起来，通过代码进行一些简单的配置后，其他项目就可以使用了。
那么使用 AgileConfig 也是一样。让我们直接看代码吧。
### 新建 Aspire 项目
使用 VS 新建一个标准 Aspire 项目。最后我们会得到这样一个解决方案：
- AspireProjectWithAgileConfig.ApiService
- AspireProjectWithAgileConfig.AppHost
- AspireProjectWithAgileConfig.Web

他们之间的依赖关系如下：

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250301150900.png)


### 在 AppHost 项目上使用 AgileConfig

```
dotnet add package AgileConfig.Aspire.Hosting --version 1.0.0
```
首先安装 AgileConfig.Aspire.Hosting。 这个包是 AgileConfig 服务端的一个扩展。使用它配合 Aspire 可以直接启动 AgileConfig 容器并且简单配置它。

安装完后，我们在 Program 下添加如下代码：

```
using Aspire.Hosting.AgileConfig;

var builder = DistributedApplication.CreateBuilder(args);

var agileConfig = builder.AddAgileConfig(); // 添加 AgileConfig 服务端，这会启动一个 Container

var agileConfig_apiservice = agileConfig.AddApp("apiservice"); // 在 AgileConfig 添加一个应用 apiservice，客户端会从这里读取业务
var agileConfig_webfrontend = agileConfig.AddApp("webfrontend"); // 在 AgileConfig 添加一个应用 webfrontend，客户端会从这里读取业务


var apiService = builder.AddProject<Projects.AspireProjectWithAgileConfig_ApiService>("apiservice");
var webFrontend = builder.AddProject<Projects.AspireProjectWithAgileConfig_Web>("webfrontend").WithExternalHttpEndpoints();

apiService.WithReference(agileConfig_apiservice); // apiservice 项目引用 agileConfig_apiservice 应用
apiService.WaitFor(agileConfig); // apiservice 项目等待 agileConfig container 启动后再启动自己

webFrontend.WithReference(agileConfig_webfrontend);  // webFrontend 项目引用 agileConfig_webfrontend 应用
webFrontend.WaitFor(agileConfig); // webFrontend 项目等待 agileConfig container 启动后再启动自己

webFrontend.WithReference(apiService);
webFrontend.WaitFor(apiService);

builder.Build().Run();

```

让我们解释一下关键代码：  

1. 添加 AgileConfig 服务端
```
var agileConfig = builder.AddAgileConfig();
```
作用：启动一个 AgileConfig 服务端的 Docker 容器，作为配置中心。

2. 在 AgileConfig 中注册应用
```
var agileConfig_apiservice = agileConfig.AddApp("apiservice");
var agileConfig_webfrontend = agileConfig.AddApp("webfrontend");
```
作用：在 AgileConfig 中注册两个应用 apiservice 和 webfrontend，它们的配置信息会被客户端读取。

细节：这两个应用对应实际的后端 API 和前端 Web 项目，后续客户端（如 apiService 和 webFrontend）会从 AgileConfig 中读取它们的配置。

3. 配置依赖关系
```
// API 服务依赖 AgileConfig 中的 apiservice 配置
apiService.WithReference(agileConfig_apiservice);
apiService.WaitFor(agileConfig); // 等待 AgileConfig 容器启动

// Web 前端依赖 AgileConfig 中的 webfrontend 配置
webFrontend.WithReference(agileConfig_webfrontend);
webFrontend.WaitFor(agileConfig); // 等待 AgileConfig 容器启动

// Web 前端依赖 API 服务
webFrontend.WithReference(apiService);
webFrontend.WaitFor(apiService); // 等待 API 服务启动
```
作用：定义服务启动顺序和依赖关系。

关键方法：

WithReference()：声明某个服务依赖另一个服务（如配置或另一个项目）。

WaitFor()：确保被依赖的服务启动后再启动当前服务。


4. 新的依赖关系如下

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20250301153016.png)

### 在客户端项目上使用 AgileConfig.Client

要连接 AgileConfig 服务端，我们需要在客户端项目添加新的包引用：

```
dotnet add package AgileConfig.Client.Aspire --version 1.0.0
```

以 ApiService 项目为例：

```
using Aspire.AgileConfig.Client;

var appName = "apiservice";

var builder = WebApplication.CreateBuilder(args);

builder.Host.UseAspireAgileConfig(appName);
```

客户端项目现在配置起来就超级简单了，只需要一行代码就解决问题了，你甚至不需要去配置 appsettings 来指定 agileconfig 的相关配置它就能运行了。

## 运行
下面让我们运行整个项目看看效果吧。

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250301155119.png)


通过 Aspire 的控制台我们可以看到 AgileConfig 的相关资源以及 2 个 project 项目都已经启动了。

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250301155434.png)


点击 AgileConfig 的终结点可以直接打开 AgileConfig 的控制台。使用 admin/123456 默认密码就可以登录进去。

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250301155720.png)

打开应用配置界面，可以看到 apiService， webfrontend 项目已经自动建立起来。

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250301155737.png)

打开终端界面，可以看到有两个客户端连接在服务端上。

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250301160203.png)

点击 webFrontend 的终结点可以直接打开这个 blazor 项目，可以正常运行。

## 总结

以上我们通过一个简单的示例演示了在 Aspire 下如何使用 AgileConfig。跟传统方案比起来，你不再需要关心：如何使用 docker 运行 AgileConfig 的服务端，不再需要关心如何在 appsettings 下添加 AgileConfig 的相关配置。
可以看到过程还是非常丝滑的。

源代码在这：
https://github.com/kklldog/Aspire.Hosting.AgileConfig

https://github.com/dotnetcore/AgileConfig
