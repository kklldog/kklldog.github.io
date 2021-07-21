## Policy.Handle<T>
Policy.Handle<T> 用来定义异常的类型，表示当执行的方法发生某种异常的时候定义为故障。    
当故障发生的时候 Polly 会为我们自动执行某种恢复策略，比如重试。   
我们演示项目中，订单接口需要获取会员的详细信息。   
http 有一定几率失败，下面我们演示下如果使用 Polly 在实现当请求网络失败的时候进行3次重试。
```
var memberJson = await Policy.Handle<HttpRequestException>().RetryAsync(3).ExecuteAsync(async () =>
                {
                    using (var httpClient = new HttpClient())
                    {
                        httpClient.BaseAddress = new Uri($"http://{memberServiceAddress.Address}:{memberServiceAddress.Port}");
                        var memberResult = await httpClient.GetAsync("/member/" + order.MemberId);
                        memberResult.EnsureSuccessStatusCode();
                        var json = await memberResult.Content.ReadAsStringAsync();
                        return json;
                    }
                });
```
使用 Policy.Handle<HttpRequestException> 在捕获网络异常。当发生 HttpRequestException 的时候触发 RetryAsync 重试，并且最多重试3次。
以下我们接着演示下当 http 的返回值是500的时候进行3次重试：
## Policy.HandleResult<T>
Policy.HandleResult<T> 用来定义返回值的类型，表示当执行的方法返回值达成某种条件的时候定义为故障。    
当故障发生的时候 Polly 会为我们自动执行某种恢复策略，比如重试。      
下面我们演示下如果使用 Polly 在实现当请求结果为 http status_code 500 的时候进行3次重试。
```
  var memberResult = await Policy.HandleResult<HttpResponseMessage>(x => (int)x.StatusCode == 500).RetryAsync(3).ExecuteAsync(async () =>
                  {
                      using (var httpClient = new HttpClient())
                      {
                          httpClient.BaseAddress =
                              new Uri($"http://{memberServiceAddress.Address}:{memberServiceAddress.Port}");
                          var result = await httpClient.GetAsync("/member/" + order.MemberId);
                          return result;
                      }
                  });
```
## Policy.TimeoutAsync
Policy.TimeoutAsync 表示当一个操作超过设定时间时会引发一个 TimeoutRejectedException 。   
这也是一个很常用的故障处理策略。
```
var memberJson = await Policy.TimeoutAsync(10).ExecuteAsync(async () =>
                {
                    using (var httpClient = new HttpClient())
                    {
                        httpClient.BaseAddress =
                            new Uri($"http://{memberServiceAddress.Address}:{memberServiceAddress.Port}");
                        var memberResult = await httpClient.GetAsync("/member/" + order.MemberId);
                        memberResult.EnsureSuccessStatusCode();
                        var json = await memberResult.Content.ReadAsStringAsync();
                        return json;
                    }
                });
            
```
以上代码表示当获取会员详情的接口超过10秒还未返回结果的时候直接抛出一个 TimeoutRejectedException 异常终止执行。

## 服务降级
以上我们演示了出现故障的时候如何进行重试，但是所有重试都失败我们的程序还是会故障。   
因为期间某个服务持续的故障导致更多的服务出现故障，一系列连锁反应后很可能导致整个应用瘫痪。   
面对这种情况我们可以把相关服务进行降级。   
当相关服务调用失败的时候我们可以给出一个统一标准的失败返回值，而不是直接抛出异常。让我们的程序依然能够继续执行下去。   
下面我们演示下如何使用Polly进行服务调用的降级处理。
```
 var fallback = Policy<string>.Handle<HttpRequestException>().FallbackAsync("FALLBACK")
                    .WrapAsync(Policy.Handle<HttpRequestException>().RetryAsync(3));
                var memberJson = await fallback.ExecuteAsync(async () =>
                {
                    using (var httpClient = new HttpClient())
                    {
                        httpClient.BaseAddress =
                            new Uri($"http://{memberServiceAddress.Address}:{memberServiceAddress.Port}");
                        var result = await httpClient.GetAsync("/member/" + order.MemberId);
                        result.EnsureSuccessStatusCode();
                        var json = await result.Content.ReadAsStringAsync();
                        return json;
                    }

                });
                if (memberJson != "FALLBACK")
                {
                    var member = JsonConvert.DeserializeObject<MemberVM>(memberJson);
                    vm.Member = member;
                }
```
首先我们使用 Policy 的 FallbackAsync("FALLBACK") 方法设置降级的返回值。当我们服务需要降级的时候会返回 "FALLBACK" 的固定值。   
同时使用 WrapAsync 方法把重试策略包裹起来。这样我们就可以达到当服务调用失败的时候重试3次，如果重试依然失败那么返回值降级为固定的 "FALLBACK" 值。

## 熔断
通过以上演示，我们的服务当发生故障的时候可以自动重试，自动降级了。虽然现在看起来挺健壮，但是还是会有不小的问题。    
当我们引入重试策略后，如果服务调用一直失败，每次调用都会反复进行重试，虽然最后会进行降级处理，但是这势必会影响服务的处理速度。   
当流量很大的时候，某个接口服务调用很慢有可能会阻塞整个服务，请求不断积压，资源不断耗尽，速度越来越慢，这是一种恶性循环。最终同样可能导致整个应用全部瘫痪的严重后果。   
面对这种情况我们就需要引入熔断机制。当一个服务的调用频繁出新故障的时候我们可以认为它当前是不稳定的，在一段时间内我们不应该再去调用这个服务。
```
        static AsyncCircuitBreakerPolicy circuitBreaker =  Policy.Handle<HttpRequestException>().CircuitBreakerAsync(
            exceptionsAllowedBeforeBreaking: 10,
            durationOfBreak: TimeSpan.FromSeconds(30),
        onBreak: (ex, ts) =>
        {
            Console.WriteLine("circuitBreaker onBreak .");
        },
        onReset: () =>
        {
            Console.WriteLine("circuitBreaker onReset ");
        },
        onHalfOpen: () =>
        {
            Console.WriteLine("circuitBreaker onHalfOpen");
        }
        );

         var retry = Policy.Handle<HttpRequestException>().RetryAsync(3);
                var fallback = Policy<string>.Handle<HttpRequestException>().Or<BrokenCircuitException>().FallbackAsync("FALLBACK")
                    .WrapAsync(circuitBreaker.WrapAsync(retry));
                var memberJson = await fallback.ExecuteAsync(async () =>
                {
                    using (var httpClient = new HttpClient())
                    {
                        httpClient.BaseAddress =
                            new Uri($"http://{memberServiceAddress.Address}:{memberServiceAddress.Port}");
                        var result = await httpClient.GetAsync("/member/" + order.MemberId);
                        result.EnsureSuccessStatusCode();
                        var json = await result.Content.ReadAsStringAsync();
                        return json;
                    }
                });
                if (memberJson != "FALLBACK")
                {
                    var member = JsonConvert.DeserializeObject<MemberVM>(memberJson);
                    vm.Member = member;
                }
```
首先定义 circuitBreaker 熔断器策略。这个策略注意最好定义成静态变量。这样能够已整个完整服务的错误为基础来判断是否开启断路器。   
然后在业务代码内定义重试策略，降级策略。我们是这些策略一一嵌套。fallback => circuitBreaker => retry ，表示当发生异常的时候首先开始重试，
重试失败后尝试熔断，如果达到熔断的条件就抛出 BrokenCircuitException 异常，降级策略捕获到 HttpRequestException 或者 BrokenCircuitException 进行降级操作。

## 使用AOP思想改进体验
