最近有个需求需要定时清理服务器上所有的缓存。本来以为很简单的调用一下 MemoryCache.Clear 方法就完事了。谁知道 MemoryCache 类以及 IMemoryCache 扩展方法都没有 Clear 方法。这可给难住了，于是想找到所有的 Keys 来一个个 Remove ，谁知道居然也没有获取所有 Key 的方法。于是研究了一下 ，找到一些方法，下面介绍两个方法：
## 自定义 CacheWrapper 包装类
MemoryCache 构造 Entry 的时候支持传入 CancellationChangeToken 对象，当 CancellationChangeToken.Cancel 触发的时候会自动使该对象过期。那么我们只要对 MemoryCache 类包装一下很容易实现一个自己的 Cache 类。
```
 public class CacheWrapper
    {
        private readonly IMemoryCache _memoryCache;
        private CancellationTokenSource _resetCacheToken = new();

        public CacheWrapper(IMemoryCache memoryCache)
        {
            _memoryCache = memoryCache;
        }

        public void Add(object key, object value, MemoryCacheEntryOptions memoryCacheEntryOptions)
        {
            using var entry = _memoryCache.CreateEntry(key);
            entry.SetOptions(memoryCacheEntryOptions);
            entry.Value = value;

            // add an expiration token that allows us to clear the entire cache with a single method call
            entry.AddExpirationToken(new CancellationChangeToken(_resetCacheToken.Token));
        }

        public object Get(object key)
        {
            return _memoryCache.Get(key);
        }

        public void Remove(object key)
        {
            _memoryCache.Remove(key);
        }

        public void Clear()
        {
            _resetCacheToken.Cancel(); // this triggers the CancellationChangeToken to expire every item from cache

            _resetCacheToken.Dispose(); // dispose the current cancellation token source and create a new one
            _resetCacheToken = new CancellationTokenSource();
        }
    }
```
然后单元测试测试一下：
```

        [TestMethod()]
        public void ClearTest()
        {
            var memCache = new MemoryCache(new MemoryCacheOptions());
            var wrapper = new CacheWrapper(memCache);

            for (int i = 0; i < 10; i++)
            {
                wrapper.Add(i.ToString(), new object(), new MemoryCacheEntryOptions());
            }

            Assert.IsNotNull(wrapper.Get("1"));
            Assert.IsNotNull(wrapper.Get("9"));

            wrapper.Clear();

            for (int i = 0; i < 10; i++)
            {
                Assert.IsNull(wrapper.Get(i.ToString()));
            }

            for (int i = 0; i < 10; i++)
            {
                wrapper.Add(i.ToString(), new object(), new MemoryCacheEntryOptions());
            }

            Assert.IsNotNull(wrapper.Get("1"));
            Assert.IsNotNull(wrapper.Get("9"));

            wrapper.Clear();

            for (int i = 0; i < 10; i++)
            {
                Assert.IsNull(wrapper.Get(i.ToString()));
            }

        }
```
测试通过。
## Compact 方法
以上 CacheWrapper 类虽然可以实现我们想要的功能，但是对于原来的程序有侵入，需要使用 CacheWrapper 类替换默认的 MemoryCache 类，不是太好。于是不死心继续研究，后来直接看了 MemoryCache 的代码（[源码在这](https://github.com/dotnet/runtime/blob/release/6.0/src/libraries/Microsoft.Extensions.Caching.Memory/src/MemoryCache.cs#L382-L393)），开源真香。发现 MemoryCache 有个 Compact 方法好像在干删除的勾当。也怪我英文不好，这单词是压缩的意思，居然才发现。。。。于是我们的清除所有对象的需求不就轻而易举了么？
```
 /// Remove at least the given percentage (0.10 for 10%) of the total entries (or estimated memory?), according to the following policy:
        /// 1. Remove all expired items.
        /// 2. Bucket by CacheItemPriority.
        /// 3. Least recently used objects.
        /// ?. Items with the soonest absolute expiration.
        /// ?. Items with the soonest sliding expiration.
        /// ?. Larger objects - estimated by object graph size, inaccurate.
MemoryCache.Compact(double percentage);
```
Compact 方法会对缓存的对象进行压缩，参数是个double，0.1 表示压缩 10% ，那么传 1.0 就是压缩 100%，那不就是 Clear All 么。所以我可以使用 Compact(1.0) 来清除所有的缓存对象。单元测试跑一下：
```
[TestMethod()]
        public void CompactTest()
        {
            var memCache = new MemoryCache(new MemoryCacheOptions());

            for (int i = 0; i < 10; i++)
            {
                memCache.Set(i.ToString(), new object(), new MemoryCacheEntryOptions());
            }

            Assert.IsNotNull(memCache.Get("1"));
            Assert.IsNotNull(memCache.Get("9"));

            memCache.Compact(1);

            for (int i = 0; i < 10; i++)
            {
                Assert.IsNull(memCache.Get(i.ToString()));
            }

            for (int i = 0; i < 10; i++)
            {
                memCache.Set(i.ToString(), new object(), new MemoryCacheEntryOptions());
            }

            Assert.IsNotNull(memCache.Get("1"));
            Assert.IsNotNull(memCache.Get("9"));

            memCache.Compact(1);

            for (int i = 0; i < 10; i++)
            {
                Assert.IsNull(memCache.Get(i.ToString()));
            }
        }
```
完美通过。    
这里简单介绍下 Compact 方法。根据注释它会按照已下优先级删除对象：
1. 过期的对象
2. CacheItemPriority 设置的优先级，等级越高越不容易被删除
3. 最近最少被使用的对象
4. 绝对过期时间
5. 滑动过期时间
6. 大对象


## 关注我的公众号一起玩转技术   

![](https://static.xbaby.xyz/qrcode.jpg)