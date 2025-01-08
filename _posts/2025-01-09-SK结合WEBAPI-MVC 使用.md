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

```
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddKernelBuilder();

builder.Services.AddControllers();


var app = builder.Build();
```

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

## 
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

##

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

##

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