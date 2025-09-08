前些天跟大佬们在群里讨论如何在不使用构造函数，不增加方法参数的情况下把一个上下文注入到方法内部使用，得出的结论是 AsyncLocal 。感叹自己才疏学浅，居然才知道有 AsyncLocal 这种神器。于是赶紧恶补一下。
## ThreadLocal
要说 AsyncLocal 还得先从 ThreadLocal 说起。ThreadLocal 封装的变量，可以在线程间进行隔离。不同线程对同一个变量的修改只在当前线程有效。这个应该大家都比较熟悉不多说了。下面简单演示一下：threadLocal 初始值为1，然后启动多个线程对这个变量进行修改，最后主线程等待1秒，保证其它线程都执行成功后再次打印threadLocal的值。
```
ThreadLocal<int> threadLocal = new ThreadLocal<int>();
threadLocal.Value = 1;
Console.WriteLine("thread id {0} value:{1} START", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);

new Thread(() => {
    threadLocal.Value = 2;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);
}).Start();
new Thread(() => {
    threadLocal.Value = 3;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);
}).Start();
new Thread(() => {
    threadLocal.Value = 4;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);
}).Start();
new Thread(() => {
    threadLocal.Value = 5;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);
}).Start();
new Thread(() => {
    threadLocal.Value = 6;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);
}).Start();

Thread.Sleep(1000);
Console.WriteLine("thread id {0} value:{1} END", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);

Console.Read();
```
输出：
```
Hello, World!
thread id 1 value:1 START
thread id 7 value:2
thread id 8 value:3
thread id 9 value:4
thread id 10 value:5
thread id 11 value:6
thread id 1 value:1 END
```
通过一系列线程修改后 threadLocal 的值在 1 号线程始终为 1 ，这也符合我们对 ThreadLocal 预期。
## 当 ThreadLocal 遇到 await
上面的示例我们使用的是 new Thread 的办法进行多线程操作，现在这种做法已经很少见了。我们现在更多的时候会使用 async/await Task 来帮我们做多线程异步操作。这个时候我们的 ThreadLocal 就会力不从心了，让我们改造一下代码：我们把 new Thread 全部改造成 Task.Run 来执行修改变量的操作。
```
ThreadLocal<int> threadLocal = new ThreadLocal<int>();
threadLocal.Value = 1;
Console.WriteLine("thread id {0} value:{1} START", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);

await Task.Run(() => {
    threadLocal.Value = 2;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);
});
await Task.Run(() => {
    threadLocal.Value = 3;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);
});
await Task.Run(() => {
    threadLocal.Value = 4;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);
});
await Task.Run(() => {
    threadLocal.Value = 5;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);
});
await Task.Run(() => {
    threadLocal.Value = 6;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);
});
Console.WriteLine("thread id {0} value:{1} END", Thread.CurrentThread.ManagedThreadId, threadLocal.Value);

Console.Read();
```
输出：
```
Hello, World!
thread id 1 value:1 START
thread id 7 value:2
thread id 8 value:3
thread id 10 value:4
thread id 11 value:5
thread id 12 value:6
thread id 11 value:5 END
```
通过输出我们可以看到 START 跟 END 的输出已经不一样了。至于为什么，如果理解 Task 的原理，其实也很好理解。简单来说，Task 的异步是一种基于状态机实现方式，编译器碰到 await 会把代码编译成一个代码块，表示一种状态。Task 的任务调度器会调度空闲线程去处理每一个状态。当一个状态完成后，调度器调度一个空闲线程去处理下一个任务，这样一个接一个处理。这里最大的困扰其实是主观上的当前线程（打印 START 跟 END 的线程）已经不是同一个了，打印 START 的是 1 号线程，打印 END 的是 11 号线程，那么 ThreadLocal 自然不适合这种场景了。
##  AsyncLocal
上面我们已经知道 ThreadLocal 已经不适合在新的 TPL 模型下的多线程变量隔离。那么我们该如何进行应对呢？答案就是 AsyncLocal 。   
让我们改造下代码，把 Threadlocal 替换成 AsyncLocal ，其它不变，运行一下代码。
```
AsyncLocal<int> asyncLocal = new AsyncLocal<int>();
asyncLocal.Value = 1;
Console.WriteLine("thread id {0} value:{1} START", Thread.CurrentThread.ManagedThreadId, asyncLocal.Value);

await Task.Run(() => {
    asyncLocal.Value = 2;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, asyncLocal.Value);
});
await Task.Run(() => {
    asyncLocal.Value = 3;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, asyncLocal.Value);
});
await Task.Run(() => {
    asyncLocal.Value = 4;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, asyncLocal.Value);
});
await Task.Run(() => {
    asyncLocal.Value = 5;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, asyncLocal.Value);
});
await Task.Run(() => {
    asyncLocal.Value = 6;
    Console.WriteLine("thread id {0} value:{1}", Thread.CurrentThread.ManagedThreadId, asyncLocal.Value);
});
Console.WriteLine("thread id {0} value:{1} END", Thread.CurrentThread.ManagedThreadId, asyncLocal.Value);

Console.Read();
```
输出：
```
thread id 1 value:1 START
thread id 6 value:2
thread id 7 value:3
thread id 8 value:4
thread id 11 value:5
thread id 7 value:6
thread id 7 value:1 END
```
结果如我们所愿， START 跟 END 的值是一致的。我们可以看到虽然线程发生了切换，但是值被很好的保留在了当前流程下。   
让我们使用另外一个代码实例来演示下 AsyncLocal 的特性。上面的代码演示的是一个 Task 接一个 Task 的场景，一下我们演示下 Task 嵌套 Task 的场景。
```
AsyncLocal<int> asyncLocal = new AsyncLocal<int>();
//block 1
asyncLocal.Value = 1;
Console.WriteLine("thread id {0} value:{1} START", Thread.CurrentThread.ManagedThreadId, asyncLocal.Value);

    await Task.Run(async () =>
    {
        //block 2
        asyncLocal.Value = 2;
        Console.WriteLine("thread id {0} value:{1} ", Thread.CurrentThread.ManagedThreadId, asyncLocal.Value);

            await Task.Run(() => {
                //block 3
                asyncLocal.Value = 3;
                Console.WriteLine("thread id {0} value:{1} ", Thread.CurrentThread.ManagedThreadId, asyncLocal.Value);
            });

        Console.WriteLine("thread id {0} value:{1} ", Thread.CurrentThread.ManagedThreadId, asyncLocal.Value);
    });

Console.WriteLine("thread id {0} value:{1} END", Thread.CurrentThread.ManagedThreadId, asyncLocal.Value);

Console.Read();
```
输出
```
thread id 1 value:1 START
thread id 6 value:2
thread id 7 value:3
thread id 7 value:2
thread id 7 value:1 END
```
跟你预期的结果一致吗？ 结果为：1 2 3 2 1 。 AsyncLocal 的变量值会被隔离在每个 Task 流程内，就算嵌套，子流程对变量的修改也不会影响到父流程的值。

## AsyncLocal 实用
AsyncLocal 的特性说的差不多了。那么 AsyncLocal 到底该使用在什么场景呢？   
当我们重构代码的时候如果需要把一个上下文参数传递进去，最傻瓜的办法就是在所有的调用类的构造函数上加入这个参数，或者在所有的方法调用上加入这个参数。但是这种办法是破坏性比较大的，因为函数签名被破坏意味着接口（广义上）约束被破坏了。这个时候我们可以通过 AsyncLocal 把上下文传递进去。  
定义一个 MyContext 类：
```
    public class MyContext : IDisposable
    {
        static AsyncLocal<MyContext> _scope = new AsyncLocal<MyContext>();

        public MyContext(object val)
        {
            Value = val;
            _scope.Value = this;
        }
        public object Value { get;}

        public static MyContext? Current
        {
            get
            {
                return _scope.Value;
            }
        }

        public void Dispose()
        {
            if (Value != null)
            {
                (Value as IDisposable)?.Dispose();
            }
        }
    }
```
假设我们已经有了 Func1 方法，现在在不破坏任何接口约束的情况下可以把 MyContext 直接通过静态变量 MyContext.Current 获取到。
```
void Func1()
{
    Console.WriteLine(MyContext.Current?.Value);
}

using (var ctx = new MyContext("context 1"))
{
    Func1();
}

using (var ctx = new MyContext("context 2"))
{
    Func1();
}
using (var ctx = new MyContext("context 3"))
{
    await Task.Run(Func1);
    await Task.Run(Func1);
    await Task.Run(Func1);
}
```
另外一个实现其实是大家非常常见的 HttpContextAccessor 。ASP.NET Core 下我们获取 HttpContext 会通过 HttpContextAccessor 获取。HttpContextAccessor 通常被注册为单例。大家有没有想过为啥单例的 HttpContextAccessor.HttpContext 变量不会被多线程或者异步方法打乱？原因也就在于 AsyncLocal 。源码在这 [HttpContextAccessor](https://github.com/dotnet/aspnetcore/blob/a450cb69b5e4549f5515cdb057a68771f56cefd7/src/Http/Http/src/HttpContextAccessor.cs) ，并不复杂大家可以看看。

## 关注我的公众号一起玩转技术   

![](https://static.xbaby.xyz/qrcode.jpg)