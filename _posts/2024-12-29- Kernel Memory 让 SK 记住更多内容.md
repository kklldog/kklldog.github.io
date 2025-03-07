Kernel Memory (KM) 是一种多模态 AI 服务，专注于通过自定义的连续数据混合管道高效索引数据集。它支持检索增强生成（RAG）、合成记忆、提示工程以及自定义语义记忆处理。KM 支持自然语言查询，从已索引的数据中获取答案，并提供完整的引用和原始来源链接。   

通过 KM 我们可以让 LLM 认识更多新的知识。比如认识新的文本内容，WORD文档，PDF, PPT，甚至是直接爬取一个网页然后进行 embedding，连爬虫都帮你写好了。
![](https://static.xbaby.xyz/253485255-31894afa-d19e-4e9b-8d0f-cb889bf5c77f.png)    
![](https://static.xbaby.xyz/253485301-c5f0f6c3-814f-45bf-b055-063f23ed80ea.png)

KM 看起来是专为 RAG 设计的一套框架。很多同学可能已经知道 SK 里面有 Semantic Memory (SM)，它可以用来做 RAG。咋一看很容易就把 KM 当作了 SM。但其实 KM 跟 SM 并不是一回事。虽然 KM 是从 SM 发展而来的。但现在 KM 已经可以脱离 SK 独立运行。    

KM 现在可以方便的集成进 .NET Backend/Console/Desktop 应用程序里面，使这些程序立马获得本地识别文档的能力。这种模式叫做`Synchronous Memory API (aka “serverless”)`。
![](https://static.xbaby.xyz/infra-sync.png)   

如果你的场景是想要搭建大规模的文档识别跟问答平台那么你可能需要把 KM 作为一个完整的服务，异步来处理这些文档与问答请求。这种模式叫做`Memory as a Service - Asynchronous API`。    
![](https://static.xbaby.xyz/infra-async.png)    

## 使用 KM 导入文本
使用 KM 还是需要搭配 LLM 的能力。这里还是使用本地的 Ollama 来运行 llama3.1:8b 的模型。下面让我们看看怎么使 KM 认识以下这段我刚编的关于 QIQI 动物园的文字。
```
Qiqi Zoo features 10 monkeys, 8 tigers, 6 elephants, 4 horses, 100 ostriches, and 99 koalas.\n\n" +
                       "Ticket Prices:\n\n" +
                       "Adults: 100 RMB\n" +
                       "Children: 50 RMB\n" +
                       "Contact: 13813818188\n" +
                       "Address: 999 Xinghu Street, Suzhou Industrial Park, Jiangsu, China.
```
以下代码我们指示了使用 ollama 来进行文本生成跟文本 embedding 生成。同时指定了使用一个简易的内存数据库来存储跟检索向量。然后把 Qiqi zoo 的文本内容导入进去，之后就可以问相关的问题了。
```
            var modelName = "llama3.1:8b";
            var ollamaEndpoint = "http://localhost:11434";
            var ollamaApiClient = new OllamaApiClient(new Uri(ollamaEndpoint), modelName);
            var ollamaModelConfig = new OllamaModelConfig() { ModelName = modelName };
            var textEmbeddingGenerator = new OllamaTextEmbeddingGenerator(ollamaApiClient, ollamaModelConfig);

            var memory = new KernelMemoryBuilder()
                .WithOllamaTextGeneration(modelName, ollamaEndpoint)
                .WithOllamaTextEmbeddingGeneration(modelName, ollamaEndpoint)
#pragma warning disable KMEXP03
                .AddIngestionMemoryDb(new SimpleVectorDb(SimpleVectorDbConfig.Volatile, textEmbeddingGenerator))
#pragma warning restore KMEXP03
                .Build<MemoryServerless>();

            var text = "Qiqi Zoo features 10 monkeys, 8 tigers, 6 elephants, 4 horses, 100 ostriches, and 99 koalas.\n\n" +
                       "Ticket Prices:\n\n" +
                       "Adults: 100 RMB\n" +
                       "Children: 50 RMB\n" +
                       "Contact: 13813818188\n" +
                       "Address: 999 Xinghu Street, Suzhou Industrial Park, Jiangsu, China.";

            await memory.ImportTextAsync(text, "doc01");

            var query = Console.ReadLine();

            while (!string.IsNullOrEmpty(query))
            {
                var answer = await memory.AskAsync(query);

                Console.WriteLine(answer);

                query = Console.ReadLine();
            }
```
问几个关于这段文字的问题，回答的非常精准。   

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241230005110.png)    

## 导入文档
我们还可以使用 KM 来直接识别 word，ppt，pdf 等文档。你都不用自己预处理这些文档，微软简直太贴心了。
```
 await memory.ImportDocumentAsync(new Document("file001").AddFile("memory/QiqiZoo.docx"));
```
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241230010015.png)    

## 导入网页
除了本地的文本，文档这些内容，KM 还能直接从远程网页上获取内容。简直了，爬虫都不用自己写了。
```
await memory.ImportWebPageAsync("https://www.cnblogs.com/kklldog/p/18538651", "web001");
```

## 总结
KM 是微软从 SK Semantic memory 的开发经历与用户反馈总结孵化出来的一个框架。它提供了许多开箱即用的能力来让开发者获取 RAG 的能力。它支持导入多种多样的文档（docx，pdf，ppt，json，html...）。它可以直接集成进你的应用内，也可以作为后端服务提供更强大的处理与扩展能力。如果你想快速构建一个问答知识库，不妨试试 Kernel Memory。

参考：https://microsoft.github.io/kernel-memory/
