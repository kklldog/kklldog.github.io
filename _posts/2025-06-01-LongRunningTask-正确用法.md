
在上一篇文章《如何正确实现一个 BackgroundService》中有提到 `LongRunning` 来优化后台任务始终保持在同一个线程上。
```csharp
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
```
但是被`黑洞视界` 大佬指出这个用法是错误的：以上用法并不能保证任务始终在同一个 `Task`(线程) 上执行。原因是当碰到第一个 `await` 之后运行时会从 `ThreadPool` 中调度一个新的线程来执行后面的代码，而当前线程被释放。这个时候就不符合我们使用 `LongRunning` 的期望了。

在 .NET 中，`Task.Factory.StartNew` 提供了 `TaskCreationOptions.LongRunning` 选项，很多开发者会用它来启动长时间运行的任务，并且想当然的认为它会永远执行在同一个线程上。但是事实上当遇到 `async` `await` 的时候并想象的那么简单。

下面我们还是通过一个错误的示例开始讲解如何正确的使用它。

## 错误用法

很多人会直接在 `Task.Factory.StartNew` 里传入一个 `async` 方法：

```csharp
// See https://aka.ms/new-console-template for more information
Console.WriteLine("Hello, World!");

var task = Task.Factory.StartNew(async () =>
{
    Console.WriteLine($"long running task starting. Thread id: {Thread.CurrentThread.ManagedThreadId}");
    var loopCount = 1;
    while (true)
    {
        Console.WriteLine($"\r\nStart: loop count: {loopCount}, Thread id: {Thread.CurrentThread.ManagedThreadId}");
        await LongRunningJob();
        Console.WriteLine($"End: loop count: {loopCount}, Thread id: {Thread.CurrentThread.ManagedThreadId} \r\n ");
        loopCount++;
    }

}, TaskCreationOptions.LongRunning);


static async Task LongRunningJob()
{
    Console.WriteLine($"task doing. Thread id: {Thread.CurrentThread.ManagedThreadId}");
    await Task.Delay(1000);
}

Console.ReadLine();
```

输出：

```
Hello, World!
long running task starting. Thread id: 12

Start: loop count: 1, Thread id: 12
task doing. Thread id: 12
End: loop count: 1, Thread id: 11


Start: loop count: 2, Thread id: 11
task doing. Thread id: 11
End: loop count: 2, Thread id: 11
```

可以看到，第一次循环后，线程 id 发生了变化。很明显 `LongRunning` 失效了。原因开篇已经讲了，不在赘述。

## 正确用法 1：同步方法

将 `LongRunningJob` 改为同步方法，避免异步切换线程：

```csharp
var task = Task.Factory.StartNew(() =>
{
    Console.WriteLine($"long running task starting. Thread id: {Thread.CurrentThread.ManagedThreadId}");
    var loopCount = 1;
    while (true)
    {
        Console.WriteLine($"\r\nStart: loop count: {loopCount}, Thread id: {Thread.CurrentThread.ManagedThreadId}");
        LongRunningJob();
        Console.WriteLine($"End: loop count: {loopCount}, Thread id: {Thread.CurrentThread.ManagedThreadId} \r\n ");
        loopCount++;
    }

}, TaskCreationOptions.LongRunning);


static void LongRunningJob()
{
    Console.WriteLine($"task doing. Thread id: {Thread.CurrentThread.ManagedThreadId}");
    Thread.Sleep(1000);
}
```

输出：

```
Hello, World!
long running task starting. Thread id: 12

Start: loop count: 1, Thread id: 12
task doing. Thread id: 12
End: loop count: 1, Thread id: 12
```

线程 id 始终不变，说明始终运行在专用线程上。

## 正确用法 2：异步方法同步等待

如果必须用异步方法，可以用 `.Wait()` 让调用变为同步：

```csharp
var task = Task.Factory.StartNew(() =>
{
    Console.WriteLine($"long running task starting. Thread id: {Thread.CurrentThread.ManagedThreadId}");
    var loopCount = 1;
    while (true)
    {
        Console.WriteLine($"\r\nStart: loop count: {loopCount}, Thread id: {Thread.CurrentThread.ManagedThreadId}");
        LongRunningJob().Wait();
        Console.WriteLine($"End: loop count: {loopCount}, Thread id: {Thread.CurrentThread.ManagedThreadId} \r\n ");
        loopCount++;
    }

}, TaskCreationOptions.LongRunning);


static async Task LongRunningJob()
{
    Console.WriteLine($"task doing. Thread id: {Thread.CurrentThread.ManagedThreadId}");
    await Task.Delay(1000);
}
```

输出：

```
Hello, World!
long running task starting. Thread id: 12

Start: loop count: 1, Thread id: 12
task doing. Thread id: 12
End: loop count: 1, Thread id: 12
```

## 总结

- `TaskCreationOptions.LongRunning` 适用于同步、阻塞型任务。
- 不要在 `StartNew` 里直接用 `async` 方法。
- 如果必须用异步方法，需同步等待（如 `.Wait()`）。

希望本文能帮你正确理解和使用 `LongRunning` 任务！

最后，再次感谢`黑洞视界`指出问题。如果对于这个问题大家希望了解更多，可以拜读大佬的这篇文章：
https://www.cnblogs.com/eventhorizon/p/17497359.html

