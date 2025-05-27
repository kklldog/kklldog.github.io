## IHostedService

```
 //
 // 摘要:
 //     Defines methods for objects that are managed by the host.
 public interface IHostedService
 {
     //
     // 摘要:
     //     Triggered when the application host is ready to start the service.
     //
     // 参数:
     //   cancellationToken:
     //     Indicates that the start process has been aborted.
     //
     // 返回结果:
     //     A System.Threading.Tasks.Task that represents the asynchronous Start operation.
     Task StartAsync(CancellationToken cancellationToken);
     //
     // 摘要:
     //     Triggered when the application host is performing a graceful shutdown.
     //
     // 参数:
     //   cancellationToken:
     //     Indicates that the shutdown process should no longer be graceful.
     //
     // 返回结果:
     //     A System.Threading.Tasks.Task that represents the asynchronous Stop operation.
     Task StopAsync(CancellationToken cancellationToken);
 }
```

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

```
HostServiceTest_A starting.
HostServiceTest_A is doing work.
HostServiceTest_A is doing work.
HostServiceTest_A is doing work.
HostServiceTest_A is doing work.
···
```

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
HostServiceTest_A is doing work.
HostServiceTest_A stop.
```

