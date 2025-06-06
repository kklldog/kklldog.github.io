相信大家都知道如何在 .NET 中执行后台（定时）任务。首先我们会选择实现 IHostedService 接口或者继承BackgroundService 来实现后台任务。然后注册到容器内，然后注册到容器内，之后这些后台任务 service 就会自动被 触发（trigger）。本文不是初级的入门教程，而是试图告诉读者一些容易被忽略的细节。

## IHostedService
IHostedService 是一个.NET Core 的接口，用于实现后台服务。通过实现这个接口，你可以在应用程序运行期间在后台执行任务，例如定时任务、监听事件、处理队列等。IHostedService 提供了 StartAsync() 和 StopAsync() 方法，分别用于启动和停止后台服务，并且框架会根据应用程序的生命周期自动调用这两个方法。    
以下是这个接口的源码：

其中 `StartAsync` 方法由 `IApplicationLifetime.ApplicationStarted` 事件触发
其中 `StopAsync` 方法由 `IApplicationLifetime.ApplicationStopped` 事件触发

```
 //
 // 摘要:
 //     Defines methods for objects that are managed by the host.
 public interface IHostedService
 {
  
     Task StartAsync(CancellationToken cancellationToken);

     Task StopAsync(CancellationToken cancellationToken);
 }
```

通常我们的后台任务会被框在一个while循环里，定时去执行某些逻辑。以下是我们模拟的一段演示代码。StartAsync 方法被 call 的时候就会执行这个 while。代码很简单，不过多解释。
```
    public class HostServiceTest_A : IHostedService
    {
        public async Task StartAsync(CancellationToken cancellationToken)
        {
            Console.WriteLine("HostServiceTest_A starting.");

            while (!cancellationToken.IsCancellationRequested)
            {
                // Simulate some work
                Console.WriteLine("HostServiceTest_A is doing work.");

                await Task.Delay(3000, cancellationToken); // Delay for 3 second
            }
        }

        public Task StopAsync(CancellationToken cancellationToken)
        {
            // to do

            return Task.CompletedTask;
        }
    }

```
把这个服务注册到容器内。
```
    builder.Services.AddHostedService<HostServiceTest_A>();
```
下面让我们启动一下程序试试。可以看到程序可以启动，这个 while 循环也是一直在工作。咋看好像没啥问题，但是仔细看看的话好像缺了点什么。   

### 问题
对了，我们这个 ASP.NET Core 程序启动日志没有了。也就是整个程序的启动过程被 block 住了。原因在于 HostedService 是顺序的，一旦某个 HostedService 的 StartAsync 方法没有尽快 return 的话，后面所有的任务全部不能执行了。比如你注册了多个 HostedService，第一个使用了这种错误的方法来执行任务，后面的 HostedService 全部都没有机会被执行。
```
HostServiceTest_A starting.
HostServiceTest_A is doing work.
HostServiceTest_A is doing work.
HostServiceTest_A is doing work.
HostServiceTest_A is doing work.
···
```
下面让我们改进一下，使用 Task.Run 来让这个任务变成异步，并且不去 await 这个 task。    
```
    public class HostServiceTest_A : IHostedService
    {
        public Task StartAsync(CancellationToken cancellationToken)
        {
            Console.WriteLine("HostServiceTest_A starting.");

            Task.Run(async () => {
                while (!cancellationToken.IsCancellationRequested)
                {
                    // Simulate some work
                    Console.WriteLine("HostServiceTest_A is doing work.");

                    await Task.Delay(3000, cancellationToken); // Delay for 3 second
                }
            });

            return Task.CompletedTask;
        }

        public Task StopAsync(CancellationToken cancellationToken)
        {
            return Task.CompletedTask;
        }
    }
```
再次执行一下程序，可以看到 HostedService 跟 ASP.NET Core 主程序都可以正确执行了。
```
HostServiceTest_A starting.
HostServiceTest_A is doing work.
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5221
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: D:\workspace\BackgroundServiceDemo\BackgroundServiceDemo
HostServiceTest_A is doing work.
```
### 改进
我们的后台任务通常是一个长期任务，这种情况下更加推荐 LongRunning Task 来 handle 这种任务。至于为什么可以参考以下文档：
https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskcreationoptions?view=net-9.0


```
           Task.Factory.StartNew(async () => {
               while (!cancellationToken.IsCancellationRequested)
               {
                   // Simulate some work
                   Console.WriteLine("HostServiceTest_A is doing work.");

                   await Task.Delay(3000, cancellationToken); // Delay for 3 second
               }
           }, TaskCreationOptions.LongRunning);

           return Task.CompletedTask;
```
### 退出
以上我们都在说如何启动后台任务，还没讨论如何取消这个后台任务。参入的那个 cancellationToken 在 Application 被 stop 的时候并不会主动 cancel。所以我们需要在 StopAsync 方法触发的时候手动来 Cancel 这个 token。
```
    public class HostServiceTest_A : IHostedService
    {
        private CancellationTokenSource _cancellationTokenSource;

        public Task StartAsync(CancellationToken cancellationToken)
        {
            Console.WriteLine("HostServiceTest_A starting.");

            _cancellationTokenSource = CancellationTokenSource.CreateLinkedTokenSource(cancellationToken);

            Task.Factory.StartNew(async () => {
                while (!_cancellationTokenSource.Token.IsCancellationRequested)
                {
                    // Simulate some work
                    Console.WriteLine("HostServiceTest_A is doing work.");

                    await Task.Delay(1000, cancellationToken); // Delay for 3 second
                }

                Console.WriteLine("HostServiceTest_A task done.");

            }, TaskCreationOptions.LongRunning);

            return Task.CompletedTask;
        }

        public Task StopAsync(CancellationToken cancellationToken)
        {

            if (!cancellationToken.IsCancellationRequested)
            {
                _cancellationTokenSource.Cancel();
            }

            Console.WriteLine("HostServiceTest_A stop.");

            return Task.CompletedTask;
        }
    }
```
让我们运行一下，然后按下 Ctrl + C 来主动退出程序，可以看到我们的 while 被安全退出了。
```
HostServiceTest_A starting.
HostServiceTest_A is doing work.
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5221
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: D:\workspace\BackgroundServiceDemo\BackgroundServiceDemo
HostServiceTest_A is doing work.
HostServiceTest_A is doing work.
HostServiceTest_A is doing work.
info: Microsoft.Hosting.Lifetime[0]
      Application is shutting down...
HostServiceTest_A stop.
HostServiceTest_A task done.
```

## BackgroundService
除了，`HostedService`，微软还给我们提供了 `BackgroundService` 这个类。一看这个类名就知道他能干嘛。其实也未必想的这么简单。BackgroundService 实际上是 IHostedService 的一个实现类。它的核心是将后台任务逻辑放在 `ExecuteAsync` 这个抽象方法中。下面我们通过一个具体案例来分析。。
```
    public class BackgroundServiceTest_A : BackgroundService
    {
        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                Console.WriteLine("ExecuteAsyncA is running.");

                await Task.Delay(3000);
            }
        }
    }
```
运行这个代码，可以看到 BackgroundService 正常启动了，而且也没 block 住 ASP.NET Core 的程序。看是一切完美。
```
ExecuteAsyncA is running.
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5221
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: D:\workspace\BackgroundServiceDemo\BackgroundServiceDemo
ExecuteAsyncA is running.
ExecuteAsyncA is running.
ExecuteAsyncA is running.
ExecuteAsyncA is running.
ExecuteAsyncA is running.
```
### 问题
以上代码真的没有问题吗？其实不尽然。让我们上点强度。如果我们在循环中加一个耗时很长的步骤。事实上这个很常见。比如以下代码：
```
    public class BackgroundServiceTest_A : BackgroundService
    {
        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                Console.WriteLine("ExecuteAsyncA is running.");

                LongTermTask();

                await Task.Delay(3000);
            }
        }

        private void LongTermTask()
        {
            // Simulate some work
            Console.WriteLine("LongTermTaskA is doing work.");
            Thread.Sleep(30000);
        }
    }
```
再次运行以下，我们可以发现 ASP.NET Core 的主程序起不来了，被 block 住了。只有等第一个循环周期过后，主程序才能启动起来。
```
ExecuteAsyncA is running.
LongTermTaskA is doing work.
```
那么问题到底出在哪？让我们看看 `BackgroundService` 的源码。
```
        public virtual Task StartAsync(CancellationToken cancellationToken)
        {
            // Create linked token to allow cancelling executing task from provided token
            _stoppingCts = CancellationTokenSource.CreateLinkedTokenSource(cancellationToken);

            // Store the task we're executing
            _executeTask = ExecuteAsync(_stoppingCts.Token);

            // If the task is completed then return it, this will bubble cancellation and failure to the caller
            if (_executeTask.IsCompleted)
            {
                return _executeTask;
            }

            // Otherwise it's running
            return Task.CompletedTask;
        }
```
可以看到 `StartAsync` 方法会调用 `ExecuteAsync`，但是它没有 await 这个方法，也就是说 `StartAsync` 内部实现是个同步方法。也就是说 `ExecuteAsync` 方法跟 `StartAsync` 会在同一个线程上被执行（在遇到第一个 await 之前）。如果你注册了多个 `BackgroundService` 并且他们一次 loop 都非常耗时，那么这个程序启动将会非常耗时。其实微软已经在文档上提醒大家了：
> Avoid performing long, blocking initialization work in ExecuteAsync.


### 改进
那么改进方法，同样使用 Task.Factory.StartNew 来构造一个  LongRunning 的 task 就可以解决。
```
    public class BackgroundServiceTest_A : BackgroundService
    {
        protected override Task ExecuteAsync(CancellationToken stoppingToken)
        {
            return Task.Factory.StartNew(async () =>
            {
                while (!stoppingToken.IsCancellationRequested)
                {
                    // Simulate some work
                    Console.WriteLine("HostServiceTest_A is doing work.");

                    LongTermTask();

                    await Task.Delay(1000, stoppingToken); // Delay for 1 second
                }

                Console.WriteLine("HostServiceTest_A task done.");

            }, TaskCreationOptions.LongRunning);
        }

        private void LongTermTask()
        {
            // Simulate some work
            Console.WriteLine("LongTermTaskA is doing work.");
            Thread.Sleep(30000);
        }
    }
```
运行一下，完美启动后台任务跟主程序。
```
HostServiceTest_A is doing work.
LongTermTaskA is doing work.
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5221
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: D:\workspace\BackgroundServiceDemo\BackgroundServiceDemo
```

### 继续改进
如果要继续吹毛求疵的话，我们还可以改进一下。从 .NET6 开始 `PeriodicTimer` 被加入进来。它是一个 timer，可以替换一部分 `Task.Delay` 活。使用 `PeriodicTimer` 话相对于 `Task.Delay` 来说可以让 loop 的间隔更加精准的被控制。
详见这里 https://learn.microsoft.com/en-us/dotnet/api/system.threading.periodictimer.waitfornexttickasync?view=net-9.0
```
     protected override Task ExecuteAsync(CancellationToken stoppingToken)
     {
         return Task.Factory.StartNew(async () =>
         {
             var timer = new PeriodicTimer(TimeSpan.FromSeconds(1));

             while (await timer.WaitForNextTickAsync(stoppingToken))
             {
                 // Simulate some work
                 Console.WriteLine("HostServiceTest_A is doing work.");
                 LongTermTask();
             }

             Console.WriteLine("HostServiceTest_A task done.");

         }, TaskCreationOptions.LongRunning);
     }
```

## 总结
通过以上的演示，我们可以感受到，实现一个后台任务还是有非常多的点需要被注意的。特别是不要在 StartAsync 或者 `ExcuteAsync` 方法内执行耗时的同步方法。如果有耗时任务请包裹在新的 Task 内执行。我们要保证这两个方法轻量化能够被快速的执行完毕，这样的话不会影响应用程序的启动。
