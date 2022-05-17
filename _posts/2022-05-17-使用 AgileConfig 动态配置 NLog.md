NLog 是我们在 .NET 领域使用非常广泛的日志组件。它默认使用 xml 来维护它的配置。最近有几个同学问我当使用 AgileConfig 的时候如何配置 NLog 。因为 AgileConfig 不支持集成 xml 格式的配置。其实 NLog 是支持从 appsettings.json / IConfiguration 读取配置的，那么肯定跟我们的 AgileConfig 集合是没有问题的。以下介绍下 NLog 如何跟 AgileConfig 进行集成，以及支持动态化的配置。
## 使用 AgileConfig 配置 NLog
NLog 默认的配置是通过 xml 来配置的。现在我们的 .NET 程序大多数都是通过 appsettings.json 来配置的。NLog 提供了从 appsettings.json / IConfiguration 读取配置的的扩展。既然支持 IConfiguration 读取那么跟我们的 AgileConfig 起来就非常简单了。

### 修改 program.cs 

从 nuget 安装：
```
NLog.Extensions.Hosting
NLog.Web.AspNetCore
``` 
使用 `UseAgileConfig` 扩展开启 AgileConfig 支持。在 `builder.Services.AddLogging` 方法内手动设置 `LogManager.Configuration` 的值。
```
//use agileconfig client
builder.Host.UseAgileConfig();

//add nlog porvider
builder.Services.AddLogging(b => {
    b.ClearProviders();
     IConfiguration config = builder.Configuration;
    NLog.LogManager.Configuration = new NLogLoggingConfiguration(config.GetSection("NLog"));
    b.AddNLogWeb();
});
```
### 在 AgileConfig 维护配置
修改好代码后，我们需要把 json 配置文件维护到 AgileConfig 上。   
AgileConfig 的基础使用不再赘述，看以前的文章。[AgileConfig 资料](https://github.com/dotnetcore/AgileConfig) 。

- 新建应用 Nlog_test
在 AgileConfig 控制台新建一个应用 Nlog_test 。
     
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220517171743.png)

- 维护 Nlog 配置
把以下 json 配置维护到 Nlog_test 应用下。
```
{
  "NLog": {
    "rules": [
      {
        "logger": "*",
        "minLevel": "Trace",
        "writeTo": "logfile2"
      }
    ],
    "targets": {
      "async": "True",
      "logconsole": {
        "type": "Console"
      },
      "logfile1": {
        "fileName": "d:/nlogs/nlog-${shortdate}.log",
        "type": "File"
      },
      "logfile2": {
        "fileName": "d:/nlogs/nlog-${shortdate}-file2.log",
        "type": "File"
      }
    },
    "throwConfigExceptions": "True"
  }
}
```
复制以上 json 文件粘贴到 “编辑 JSON” 文本框：

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220517171829.png) 
- 发布配置
点击发布按钮，上线 Nlog 配置。
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220517171906.png)
## 运行项目
运行项目后我们可以看到日志已经写到指定的位置，说明 Nlog 成功从 AgileConfig 读取到了配置。

## 动态刷新 NLog 配置
上面的代码我们实现了脱离 xml 从 Agileconfig 读取配置来 NLog ，但是我们这个配置是一次性的，当我们在 AgileConfig 控制台修改配置的时候并不会更改 Nlog 的配置。这个显然不符合我们 AgileConfig 动态配置的气质。既然 NLog 不会自动监听 `IConfiguration` 的变化，那么我们就通过 AgileConfig 的配置变化事件来手动 reload NLog 的配置吧。
```
void loadNlogConfig()
{
    IConfiguration config = builder.Configuration;
    NLog.LogManager.Configuration = new NLogLoggingConfiguration(config.GetSection("NLog"));
    NLog.LogManager.Configuration.Reload();
}

//use agileconfig client
builder.Host.UseAgileConfig((ConfigChangedArg e) => {
    loadNlogConfig();
});

//add nlog porvider
builder.Services.AddLogging(b => {
    b.ClearProviders();
    NLog.LogManager.ConfigurationChanged += (_, _) => NLog.LogManager.ReconfigExistingLoggers();
    loadNlogConfig();
    b.AddNLogWeb();
});
```
通过以上配置，当我们在 AgileConfig 修改 Nlog 配置规则的时候，只要点击发布，应用的 Nlog 配置就会实时更改。

## AgileConfig
AgileConfig 是一个轻量级配置中心   
✨✨✨Github地址：[https://github.com/dotnetcore/AgileConfig](https://github.com/dotnetcore/AgileConfig)  开源不易，欢迎star✨✨✨   

演示地址：[http://agileconfig_server.xbaby.xyz/](http://agileconfig_server.xbaby.xyz/)  超级管理员账号：admin 密码：123456   

## 关注我的公众号一起玩转技术   

![](https://static.xbaby.xyz/qrcode.jpg)