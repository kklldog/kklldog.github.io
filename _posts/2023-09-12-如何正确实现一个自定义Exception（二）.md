上一篇《[如何正确实现一个自定义 Exception](https://www.cnblogs.com/kklldog/p/how-to-design-exception.html)》发布后获得不少 star。有同学表示很担忧，原来自己这么多年一直写错了。其实大家不用过分纠结，如果写的是 .NET CORE 1.0+ 的程序，那么大概率是没有问题的。   
有大佬已经在评论区指出这些信息是过时的了。确实在.NET CORE 发布之后，`Exception` 已经不在推荐实现 `ISerializable` 接口。让我们细说一下。

## BinaryFormatter security vulnerabilities 
上一篇我们谈论了这么多，其实都是在说 `ISerializable` 的 patten。`ISerializable` 主要的作用就是给 `BinaryFormatter` 序列化器提供指示如何进行序列化/反序列化。也就是说这个接口基本上就是给 `BinaryFormatter` 设计的。`BinaryFormatter` 主要是给 .NET remoting 技术服务（一种古老的 RPC 技术，听过的都是老司机，不太确定 WCF 的 Binary 序列化是否使用该技术）。
但是很不幸，这个 `BinaryFormatter` 存在严重的安全风险。   
> `BinaryFormatter` 类型会带来风险，不建议将其用于数据处理。 即使应用程序认为自己正在处理的数据是可信的，也应尽快停止使用 `BinaryFormatter`。 `BinaryFormatter` 不安全，无法确保安全。  

不光是 `BinaryFormatter` 有风险，以下这些序列化器同样存在风险，应避免使用：    
- SoapFormatter
- LosFormatter
- NetDataContractSerializer
- ObjectStateFormatter

当前微软主要推荐使用：   
 - System.Text.Json 
 - XmlSerializer

https://learn.microsoft.com/zh-cn/dotnet/standard/serialization/binaryformatter-security-guide   

其实不光是 .NET，其他语言在序列化反序列化上都很容易引入安全漏洞，比如 JAVA 的 `jackson` 就爆过序列化安全漏洞。
## BinaryFormatter Obsolete
由于 remoting 技术在 .NET CORE 中已经废弃，并且有严重的安全风险，所以微软开始慢慢淘汰 `BinaryFormatter` 这个接口。    
以下是 `binaryformatter-obsoletion` 的 roadmap：   
- .NET 5 (Nov 2020)
1. Allow disabling BinaryFormatter via an opt-in feature switch
2. ASP.NET projects disable BinaryFormatter by default but can re-enable
3. WASM projects disable BinaryFormatter with no ability to re-enable
4. All other project types (console, WinForms, etc.) enable BinaryFormatter by default
5. .NET produces guidance document on migrating away from BinaryFormatter
6. All outstanding BinaryFormatter-related issues resolved won't fix
7. Introduce a BinaryFormatter tracing event source
8. Serialize and Deserialize marked obsolete as warning
- .NET 6 (Nov 2021)
1. No new [Serializable] types introduced
2. No new calls to BinaryFormatter from any first-party dotnet org code base
3. All first-party dotnet org code bases begin migration away from BinaryFormatter
- .NET 7 (Nov 2022)
1. All first-party dotnet org code bases continue migration away from BinaryFormatter
2. References to BinaryFormatter APIs marked obsolete as warnings in .NET 5 now result
3. in build errors
4. A back-compat switch is made available to turn these back to warnings
- .NET 8 (Nov 2023)
1. All first-party dotnet org code bases complete migration away from BinaryFormatter
2. BinaryFormatter disabled by default across all project types
3. All not-yet-obsolete BinaryFormatter APIs marked obsolete as warning
4. Additional legacy serialization infrastructure marked obsolete as warning
5. No new [Serializable] types introduced (all target frameworks)
- .NET 9 (Nov 2024)
1. Remainder of legacy serialization infrastructure marked obsolete as warning
2. BinaryFormatter infrastructure removed from .NET
3. Back-compat switches also removed

从 .NET5 开始会出现警告，到.NET6 BUILD 的时候直接会是 ERROR，但是可以强制启用。到.NET9 会完全从 .NET 中移除。   
那么既然 `BinaryFormatter` 在目前已经不在推荐使用，自然我们的自定义 `Exception` 也不用遵循 `ISerializable` patten 了。以下链接是微软给出的当前自定义 `Exception` 实现的建议，太长就不复制了。总之已经不在需求实现 `protected` 的序列化构造器，也不用 `override GetObjectData` 方法。

https://github.com/dotnet/docs/issues/34893

感谢   
- 饭勺oO
- czd890

等大佬提供相关链接。

## 关注我的公众号一起玩转技术   

![](https://static.xbaby.xyz/qrcode.jpg)