## 奇怪的问题
本周被一个奇怪的问题困扰了一天。事情的起因是这样的：在某个 PR 合并后，我拉了最新代码，但是在我本地F5调试始终报错。示例代码如下：   
```
    public interface Interface1
    {
        void Method1();
    }

    public class MockSerivce
    {
        public MockSerivce(Interface1 interface1)
        {

        }
    }

builder.Services.AddSingleton<MockSerivce>();

```
报的错误呢也显而易见：
```
Unable to resolve service for type 'DevelopmentTest.Interface1' while attempting to activate 'DevelopmentTest.MockSerivce'
```
我们没有注册 `Interface1` 到 DI 容器里，那自然实例化 MockSerivce 的时候就找不到依赖了。   
但是奇怪的是：我其他同事们都没有这个问题，他们在本地调试的时候都好好的，并不会报错。并且在这个分支编译后的代码在开发服务器上运行的都很完美。    
这个就有点冲击到我了，难道是我电脑有问题，VS 有问题，还是我人品有问题？

## 寻找答案
当然了代码是不会骗人的，造成以上问题一定不是我人品问题而是代码的问题。   
经过一番尝试，我发现这个问题跟系统运行在哪个环境有关系。只要我把 `launchSettings.json` 里的 `  ASPNETCORE_ENVIRONMENT` 从 `Development` 改成别的什么值，那么一切都运行正常了。正巧在我们组其他同事都维护一个自己的 appestings.username.json 然后运行在这个环境之下，也就是说他们都不运行在 `Development` 下。这就是为啥只有我会报错的原因了。    
事情到了这一步，那么我们很容易猜测： .NET DI 系统在 `Development` 下是有骚操作的。在 `Development` 下它会进行依赖分析，如果依赖关系有错误，那么直接会报错。但是在其他环境下就不会提交分析校验，只有在运行时真正尝试实例化对象的时候才会报错。   
当然靠猜测总是不太靠谱，干脆翻翻代码吧。很快就找到了：
```
    internal static ServiceProviderOptions CreateDefaultServiceProviderOptions(HostBuilderContext context)
        {
            bool isDevelopment = context.HostingEnvironment.IsDevelopment();
            return new ServiceProviderOptions
            {
                ValidateScopes = isDevelopment,
                ValidateOnBuild = isDevelopment,
            };
        }
```
在 `HostingHostBuilderExtensions` 这个扩展类里很清楚的看到，只有在 `Development` 下 DefaultServiceProviderOptions 的 `ValidateScopes` 与 `ValidateOnBuild` 会被设置为 `True`。这就直接证明了上面的猜想。只有在 `Development` 下才会在启动的时候去校验依赖关系。

## 强制校验
既然找到了答案，那么让我们来试一下：强制开启依赖关系的校验。
```
var builder = WebApplication.CreateBuilder(args);

builder.Host.UseDefaultServiceProvider(op =>
{
    op.ValidateOnBuild = true;
    op.ValidateScopes = true;
});
```
代码如上在 Host 上调用 `UseDefaultServiceProvider` 扩展方法，指定 `ValidateScopes` 与 `ValidateOnBuild` 都为 `True`。
再次运行我们的项目，这个时候不管是在 `Development` 还是 `Production` 还是别的任何环境下，都会报错了。
```
Unable to resolve service for type 'DevelopmentTest.Interface1' while attempting to activate 'DevelopmentTest.MockSerivce'
```

## 总结
通过以上我们可以发现 .NET 的 DI 系统，在 `Development` 环境下跟其他环境的行为是不同的。在 `Development` 下会提交进行依赖关系的校验，如果有问题会提前报错。所以我们调试的时候请尽量选择在 `Development` 下进行或者手动强制开启校验。这个问题很容易被忽视，至少我没在其他博文里见有人提到过。其实在微软的官方文档上是提到了，但也确实就是提了一嘴而已。    
关于这个话题其实还没完，还有一个更有意思的问题：`Captive dependency` 可以聊一下。但是今天太晚了，改天吧。     
参考：https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#scope-validation
