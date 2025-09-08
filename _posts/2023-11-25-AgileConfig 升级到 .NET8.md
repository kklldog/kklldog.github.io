Hello 大家好。本月圈子里最大的事莫过于 .NET8 正式 release。群友们都在适配 .NET8。抽个周末我也把 AgileConfig 升级到了 .NET8。下面把升级的过程简单记录一下，其中有个小坑，对大家升级的时候可能有所帮助。
## 升级
* 升级 .NET8   
修改所有项目的目标框架为 .NET8.0
* 升级 nuget 包   
在 nuget 包管理器里把所有能更新的包全部更新到最新。   
![](https://static.xbaby.xyz/微信截图_20231124105734.png
)   
有一个包 `Microsoft.AspNetCore.Http.Abstractions` 提示已经弃用，需要处理一下。
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20231124105602.png)    
因为这个包现在微软已经不在 nuget 上提供，需要使用框架引用。   
修改项目文件，在 ItemGroup里添加以下内容：
```
 <ItemGroup>
    <FrameworkReference Include="Microsoft.AspNetCore.App" />
 </ItemGroup>

```
![](https://static.xbaby.xyz/微信截图_20231124110459.png
)    
再次编译，警告消息。   
这个问题其实跟 .NET8 没有关系，应该是我 3.1 升 6 的时候遗忘了。   
* 修改 dockerfile    
原来的 dockerfile 是基于 .NET6 镜像的，需要修改为 .NET8
```
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
....

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
....
```
把 6 改成 8，其他不用改，超级简单。    

通过以上操作，在本地运行没有问题，打包成镜像后在本地 docker desktop 环境下跑也没问题。但是发布到服务器上用镜像跑缺报错：`Failed to create CoreCLR, HRESULT: 0x80070008`    
![](https://static.xbaby.xyz/微信截图_20231124140704.png
)
警告排查是由于低版本的 docker engine 与某些 ubuntu 的镜像不兼容，需要在 docker run 的时候添加参数。
```
--security-opt seccomp=unconfined
```
或者在 docker-compose.yml 上添加参数：
```
security_opt:
    - seccomp=unconfined
```
添加以上参数后一切正常了。
    
参考：https://docs.linuxserver.io/FAQ/#symptoms


## 总结
本次升级可以说相当简单。得益于 .NET 接口的稳定，升级框架几乎不用动任何一行自己的代码。只是最新的 .aspnet8 runtime 的镜像对低版本的 docker engine 兼容性有点问题，使用 docker 跑的同学需要注意一下。