Hello 大家好，最新版的 AgileConfig 1.9.4 发布了。现在它可以通过 OpenTelemetry 对外提供 logs，traces，metrics 三个维度的数据。用户可以自由选择支持 otlp 协议的工具来进行查询与分析。比如 Seq，loki，prometheus, grafana 等等。    
本来 AgileConfig 的日志是通过 Nlog 写在本地的。但是文本日志无法进行集中的查询与分析，况且现在绝大多数同学使用 docker 或者 k8s 来运行 AgileConfig 的服务端，这就导致日志会存储在容器来，查看起来特别费劲。   
于是就有同学提出对接第三日志组件的需求，比如要我直接对接 Seq 等等。当然 AgileConfig 对接 Seq 或者 ELK 等组件当然技术上没有难度。但是，如果对接了特定的三方组件，这个就太定制化了，耦合性太强。如果用户不用 Seq 也不用 ELK 而是使用 Loki 呢？难道要让用户特地安装一套 Loki 吗？    
为了解决这个问题：既要把日志可以对外提供出来，又不依赖特定的三方组件，所以我决定让 AgileConfig 对接 OpenTelemetry。    
## 什么是 OpenTelemetry
OpenTelemetry 是一个开放源代码项目，用于帮助开发人员生成、收集、处理和导出软件性能和行为的可观测性数据。它主要涵盖三个方面的数据：分布式追踪（tracing）、指标（metrics）和日志（logs）。OpenTelemetry 的目标是提供一个统一的工具和规范来简化可观测性数据的收集和分析，从而帮助开发者更好地理解和优化他们的应用程序。   
也就是说 OpenTelemetry 不光支持 logs，而且还支持 tracing 跟 metrics。所以这次干脆除了 logs 其他两个也一并集成进来了。
## OTLP
OTLP（OpenTelemetry Protocol，OpenTelemetry 协议）是 OpenTelemetry 项目定义的一个协议，用于传输可观测性数据，包括追踪（traces）、指标（metrics）和日志（logs）。OTLP 的设计目标是高效、可扩展且易于实现，旨在统一不同类型的可观测性数据传输，简化数据收集和分析的过程。

OTLP 的特点
- 统一性：OTLP 提供了一个统一的协议，可以传输不同类型的可观测性数据，减少了系统集成的复杂性。
- 高效性：OTLP 的设计注重性能和资源效率，确保在高负载情况下仍能稳定传输数据。
- 扩展性：协议具有良好的扩展性，可以适应未来的需求和新增的可观测性数据类型。
多种传输方式：支持 gRPC 和 HTTP/JSON 传输方式，方便集成到不同的系统环境中。


## 查看 Logs
如果你想查看 AgileConfig 的 Logs 内容，你可以选择任意一个支持 otlp 的 log 组件，如 Seq，loki 等等。    
在 AgileConfig 服务启动之前，你需要配置 logs 的 endpoint：
```
  "otlp": {
    "instanceId": "agileconfig_server_admin", // if empty, will generate a new one
    "logs": {
      "protocol": "http", // http grpc
      "endpoint": "http://xxxx:5341/ingest/otlp/v1/logs"
    },
```
或者配置环境变量：
```
otlp:logs:protocal=http
otlp:logs:endpoint=http://xxxx:5341/ingest/otlp/v1/logs
```
三方 log 组件的文档上都会表明 otlp 的 endpoint 地址，如 Seq：
- http://xxxx:5341/ingest/otlp/v1/logs

或者 loki：
- http://xxxx:3100/otlp/v1/logs

如果配置正确服务启动后就能在 log 组件里查询到了。   

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20240513022608.png)

## 查询 Traces
跟上面的 logs 一样，如果你想查看 AgileConfig 的 Traces 内容，你可以选择任意一个支持 otlp 的 traces 组件，如 Seq，Jaeger 等等。    
在 AgileConfig 服务启动之前，你需要配置 traces 的 endpoint：
```
  "otlp": {
    "traces": {
      "protocol": "http", // http grpc
      "endpoint": "http://xxxx:5341/ingest/otlp/v1/traces"
    },
```
或者配置环境变量：
```
otlp:traces:protocal=http
otlp:traces:endpoint=http://xxxx:5341/ingest/otlp/v1/traces
```
三方 traces 组件的文档上都会表明 otlp 的 endpoint 地址，如 Seq：
- http://xxxx:5341/ingest/otlp/v1/traces

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20240519232717.png)  

## 创建 Metrics Dashboard
对 Metrics 进行查询其实跟上面也是类似，只要配置好 otlp 地址就好了。但是为了更好的监控效果，metrics 一般会做成 dashboard 实时挂在大屏上展示。下面我们就以 prometheus + grafana 的方案来给 AgileConfig 定制一个 dashboard。

### 安装 Prometheus

首先我们要安装 prometheus，这次我偷懒直接在 windonws 下安装了 prometheus。下载好安装包后，直接解压，如果运行如下命令：
```
./prometheus --enable-feature=otlp-write-receiver
```
访问以下 localhost:9090 如果可以登录那么就是成功了。可以看到 prometheus 其实自带界面，但是功能上来说还是跟 grafana 差远了。
### 安装 Grafana
使用 docker 来安装最新版的 grafana：
```
docker run --name=vk-grafana -p 3000:3000 grafana/grafana:6.7.4
```
访问以下 localhost:3000 如果可以登录那么就是成功了。

### 在 Grafana 内添加 Prometheus 数据源
在 grafana 的 datasource 菜单下添加 Prometheus 数据源。    
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20240603013747.png)   

填写 Prometheus 的地址    

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20240603013509.png)

### 配置服务端
跟 logs，traces 一样，在服务启动之前需要对 metrics 的 endpoint 进行配置：
```
  "otlp": {
    "metrics": {
      "protocol": "http", // http grpc
      "endpoint": "http://xxx:9090/api/v1/otlp/v1/metrics"
    },
```
或者配置环境变量：
```
otlp:metrics:protocal=http
otlp:metrics:endpoint=http://xxx:9090/api/v1/otlp/v1/metrics
```

### 支持的 Metrics
如果以上步骤都没问题，那么我们启动 AgileConfig 的服务端后，稍等片刻就能在 grafana 里查询到这些 metrics 了。   

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20240603013747.png)    

AgileConfig 对外提供了以下 metrics：

|  名称   | 说明  |
|  ----  | ----  |
| AppCount  | 应用数量 |
| NodeCount  | 节点数量 |
| ClientCount | 客户端数量 | 
| ServiceCount | 注册的服务数量 | 
| ConfigCount | 配置项数量 | 
| CpuUsed | CPU 使用率，0~100 内的浮点数 | 
| MemoryUsed | 内存使用量，单位 MB | 
| PullAppConfigCounter | client 通过 http 接口拉取 app 配置的次数统计 | 

### 制作 Dashboard
有了以上指标，我们就可以在 grafana 上来创建一个 dashboard 来监控我们的 AgileConfig 服务啦。因为本文不是 grafana 的介绍文章，所以如何来创建 dashboard 就不在这多说了。让我们看一下效果吧。   

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240603015018.png)


## 总结
通过以上介绍。我们可以看到现在要采集 AgileConfig logs 或者进行 tracing 都非常简单。只要找个支持 otlp 的组件配置一下 endpoint 就可以了。而且这并不依赖特定的组件。这次 AgileConfig 还支持了 opentelemetry metrics。有了这些 metrics，我们可以通过 grafana 轻松的创建出一个漂亮的 dashboard 来对 AgileConfig 进行监控。   

## 最后
✨✨✨ Github地址：[https://github.com/dotnetcore/AgileConfig](https://github.com/dotnetcore/AgileConfig)  开源不易，欢迎 star ✨✨✨   

演示地址：[https://agileconfig-server.xbaby.xyz/](https://agileconfig-server.xbaby.xyz/)  超管账号：admin 密码：123456  

## 关注我的公众号一起玩转技术   

![](https://static.xbaby.xyz/qrcode.jpg)