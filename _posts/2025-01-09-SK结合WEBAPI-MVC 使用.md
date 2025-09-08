前面几篇文章，大概把 SemanticKernel 的基本用法讲完了。上面的示例都是基于控制台程序实现的。因为微软的文档上是这么实现的，自然我就抄过来了。最近我在考虑如何把 SemanticKernel 导入到生产环境。比如我们想要通过一个 WebAPI 来提供 chat 服务。那么我们就需要把 SemanticKernel 跟 ASP.NET Core 结合起来使用。咋一想好像还挺简单。但是，仔细想想可能还真没那么简单。

让我们先回忆一下 SemanticKernel 的基本用法：
```
        var deployment = "gpt3.5";
             var endpoint = "http://localhost:11434/v1/";
             var apikey = "123";
             var builder = Kernel.CreateBuilder()
                 .AddAzureOpenAIChatCompletion(deployment, endpoint, apikey);
             var kernel = builder.Build();
             kernel.Plugins.AddFromType<WeatherPlugin>();

             var reply = await kernel.InvokePromptAsync(...);
```
它的过程分这么几步：
1. 使用 Kernel.CreateBuilder 扩展方法构造一个 KernelBuilder
2. 在 builder 上调用 Add 扩展方法注册会使用到的服务
3. builder.Build 方法构造真正的 kernel
4. 添加 plugin 等
5. 真正的使用 Kernel 执行 prompt

那么我们最傻瓜的办法就是每次需要使用 kernel 的时候就把上面的代码执行一遍。这太傻了。这里我们有 2 个点至少是要考虑的：
1. 如何使用 ASP.NET Core 的 DI 容器来管理 Kernel 等对象的生命周期
2. ASP.NET Core 是多线程模型，那么就要考虑 thread safe 的问题

## 注入哪个对象呢？
根据上面的代码，我们想一下，注入哪个类或者对象，以及是生命生命周期呢？     
我有一个大胆的想法：在构造一个 kernel 后把它注册成 singleton 生命周期，全局共享它。微软在文档上也没说它是不是 thread safe 的，万一是 safe 的呢？这显然是我想多了，查看了源码，发现一个 public 的 Data 的属性是基于 Dictionary 实现的。这个美好的愿望落空了。
好吧，那就选择把 KernelBuilder 注册成 singleton 吧。让我们试试。

## 注入 KernelBuilder

这种方案是最容易想到的。我们可以把 KernelBuilder 注册成 Singleton 生命周期，然后每次请求进来的时候把 KernelBuilder 注入到 Controller。在 Controller 里使用这个 Build 方法构造出一个新的 Kernel 实例。代码简单如下：   

在 IServiceCollection 上定义一个 AddKernelBuilder 的扩展方法：
```
     public static IServiceCollection AddKernelBuilder(IServiceCollection services)
     {
         services.AddSingleton<IKernelBuilder>(sp =>
         {
             var deployment = "gpt3.5";
             var endpoint = "http://localhost:11434/v1/";
             var apikey = "123";
             var builder = Kernel.CreateBuilder()
                 .AddAzureOpenAIChatCompletion(deployment, endpoint, apikey);

             return builder;
         });

         return services;
     }
```
调用这个扩展方法注册服务
```
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddKernelBuilder();

builder.Services.AddControllers();

var app = builder.Build();
```
在 Controller 里构造 Kernel，然后就可以使用了
```
    [ApiController]
    [Route("[controller]")]
    public class ChatController : ControllerBase
    {
        private readonly Kernel _kernel;

        public ChatController(IKernelBuilder kernelBuilder)
        {
            _kernel = kernelBuilder.Build();
        }
    }
```

## 改进
上面我们演示的是每次请求进来就注入 KernelBuilder。其实我们需要的只是 Kernel 的实例，直接注入 Kernel 会更加方便一点。那么我们在上面代码的基础上再改进一点点。把 Kernel 注册成 Scope 生命周期。
```
        public static IServiceCollection AddSemanticKernel(this IServiceCollection services)
        {
            services.AddSingleton<IKernelBuilder>(sp =>
            {
                var deployment = "gpt3.5";
                var endpoint = "http://localhost:11434/v1/";
                var apikey = "123";
                var builder = Kernel.CreateBuilder()
                    .AddAzureOpenAIChatCompletion(deployment, endpoint, apikey);

                return builder;
            });

            services.AddScoped<Kernel>(sp =>
            {
                var builder = sp.GetRequiredService<IKernelBuilder>();
                return builder.Build();
            });

            return services;
        }
```
直接在 Controller 里注入 Kernel。
```
    [ApiController]
    [Route("[controller]")]
    public class ChatController : ControllerBase
    {
        private readonly Kernel _kernel;

        public ChatController(Kernel Kernel)
        {
            _kernel = Kernel;
        }

    }
```

## 基于 Kernel.Clone 继续改进

上面我们演示了基于 KernelBuilder 的注入方式，已经可以工作了。但是我们还可以改进一点点。通过查看 Build 方法的源码。我们会发现 Build 的时候会做不少工作，比如会构造一个 ServiceProvider 然后再把所有已经 Add 过的 service 再次 Add 进这个 ServiceProvider 里。然后再 new 一个 Kernel，再把这个 ServiceProvider 传进这个 Kernel。看起来有点多此一举。因为那些 service 已经 Add 过一次了，没必要每次 new Kernel 的时候再次 Add 一遍。    
那么有啥办法改进吗？答案是 Clone 方法。Kernel.Clone 方法可以基于现有的 Kernel 实例快速克隆出一个具有相同行为与属性的实例。这个方法会轻量很多。好的，让我们继续改进一下代码。   

定义一个 KernelProvider 注册成 singleton 生命周期。同时这个 KernelProvider 使用 KernelBuilder 构造出一个 Kernel 实例并且始终保持引用。它有一个 GetKernel 方法，每次调用这个方法的时候都从保持的那个 Kernel 实例克隆出一个新实例。这样就不用每次都去 Build 了。
```
       class KernelProvider
       {
           private readonly Kernel _kernel;

           public KernelProvider()
           {
               var deployment = "gpt3.5";
               var endpoint = "http://localhost:11434/v1/";
               var apikey = "123";
               var builder = Kernel.CreateBuilder()
                   .AddAzureOpenAIChatCompletion(deployment, endpoint, apikey);

               _kernel = builder.Build();
           }

           public Kernel GetKernel()
           {
               return _kernel.Clone();
           }
       }
```
把这个 KernelProvider 注册成 singleton 的同时再注册一个 Scoped 的 Kernel。这样每次注入 Kernel 的时候就会自动克隆一个示例了。
```
        public static IServiceCollection AddSemanticKernel2(this IServiceCollection services)
        {
            services.AddSingleton<KernelProvider>();

            services.AddScoped<Kernel>(sp =>
            {
                var provider = sp.GetRequiredService<KernelProvider>();

                return provider.GetKernel();
            });

            return services;
        }
```

## 直接在 ASP.NET Core 的默认容器上注入
以上我们演示的代码都是基于 KernelBuilder 实现的。KernelBuilder 的 Add 扩展方法其实会把 service 跟 plugin 等服务都注册到自己内部的一个容器内。这样会有一个问题。如果我们脱离开了 KernelBuilder 想要直接使用这些服务的时候是拿到不到的。比如我们想在 Controller 里直接注入 AzureOpenAIChatCompletionService 是会失败的。因为这些服务并没有注册到 ASP.NET Core 的默认容器里。    
那么怎么办？其实非常简单。让我们忘记 KernelBuilder，直接在 IServiceCollection 上调用 AddKernel 方法以及其他 Add service 的方法。因为这些 Add 扩展方法，微软在 KernelBuilder 上跟 IServiceCollection 上同时实现了一遍。比如：    
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20250109232930.png)
是不是有点得来全部废功夫的感觉？

```

        public static IServiceCollection AddSemanticKernel3(this IServiceCollection services)
        {
            var deployment = "gpt3.5";
            var endpoint = "http://localhost:11434/v1/";
            var apikey = "123";
            services.AddKernel();
            services.AddOpenAIChatCompletion(deployment, endpoint, apikey);

            return services;
        }
```

## 总结
以上我们总结了 SemanticKernel 如何跟 ASP.NET Core 的 DI 系统结合使用。可以配合 KernelBuilder 做注入，同时推荐使用 Clone 方法获得新的 kernel 对象。也可以直接在 ASP.NET Core 的默认容器上进行注册并进行注入，这样的好处是可以脱离开 KernelBuilder 与 Kernel 来直接注入对应的服务。希望以上对大家有所帮助，谢谢！

参考：https://github.com/microsoft/semantic-kernel   
示例代码已上传到 github
https://github.com/kklldog/SKLearning