现在几乎所有语言都提供包管理工具，比如 JavaScript 的 npm ，Java 的 Maven ，Dart 的 pub 。.Net 程序当然是 NuGet 。NuGet 也出现很多年了，奇怪的是居然还有很多人不知道。   
现在软件结构越来越复杂，在多个项目中往往需要共享一些库、组件等等。NuGet 为我们提供了方便的包管理功能。但是 NuGet 默认提供外网公开的服务，如果我们希望在公司内部或者自己家里进行一些库的管理，那么就需要自己来搭建 NuGet 私服。   
Nuget 私服有几个工具可以搭建如官方的Nuget.Server 、ProGet 、BaGet 等。这里推荐 BaGet 这个工具，它跨平台又非常轻量化，易于部署，一行 docker 命令就可以运行起来。这里必选吐槽下 Nuget.Server 做为 NuGet 官方提供的一个工具居然还是依赖 Framework 的。
## 运行 BaGet 服务
BaGet 有多种部署方式。比如可以从 Github 上拉取 release 后的发布文件手工 dotnet 运行，也可以直接使用 docker 容器化部署。现在是容器化的时代，那么当然首先 docker 部署咯。   

```
# The following config is the API Key used to publish packages.
# You should change this to a secret value to secure your server.
ApiKey=NUGET-SERVER-API-KEY

Storage__Type=FileSystem
Storage__Path=/var/baget/packages
Database__Type=Sqlite
Database__ConnectionString=Data Source=/var/baget/baget.db
Search__Type=Database
```
先创建一个 baget.env 的环境变量配置文件
```
docker run --rm --name nuget-server -p 5555:80 --env-file baget.env -v "$(pwd)/baget-data:/var/baget" loicsharma/baget:latest
```
使用 docker run 命令运行   
![](https://s3.bmp.ovh/imgs/2021/09/c9826374cbb6f5f2.png)   
访问一下这个服务，可以看到服务成功运行起来了。但是现在一个包都没有，所以显示的是 nothing here ...

## 构建 NuGet 包
要推送 NeGet 包，首先我们需要包我们的库打包成 NuGet 包。
![](https://s3.bmp.ovh/imgs/2021/09/505593cea6807bb0.png)    
打包可以使用 nuget 的 cli 来打包。其实最简单的是在我们的项目上右键属性，在打包这个 tab 页上勾选 “在构建时生成 NuGet 包”，这样在我们每次生成项目完成的时候会在bin目录下生成对应的 nuget 包。

## 推送 NuGet 包
Nuget 包打包完成后，就可以推送自己的包到这个服务了。
```
 dotnet nuget push -s http://localhost:5555/v3/index.json .\AgileConfig.Client.1.1.8.11.nupkg
```
使用 dotnet nuget push 命令进行推送   
![](https://s3.bmp.ovh/imgs/2021/09/83c696908a342dd0.png)   
推送成功会显示“已推送包”，期间有个警告，因为我们没有设置 apikey ，这个忽略。
![](https://s3.bmp.ovh/imgs/2021/09/03717691f1f96753.png)   
再次刷新 BaGet 的页面，就可以看到我们刚才推送上去的包了。
## 使用 BaGet 源
为了能够让我们的 VisualStudio 能够检索 BaGet 服务，我们需要进行简单的配置。
![](https://s3.bmp.ovh/imgs/2021/09/2849b6d62738e875.png)   
打开 VS > 工具 > 选项 > NuGet 包管理器 > 程序包源，点击绿色的加号，配置源名称baget , 地址：
http://192.168.0.117:5555/v3/index.json 点击确定。
![](https://s3.bmp.ovh/imgs/2021/09/3a98076ef7b2168d.png)    
随便打开一个项目解决方案，在 NuGet 包检索页面选择程序包源给 “baget” ,浏览页面就会列出这个源当前具有的包。这样就可以正常进行管理与安装了。

## 总结
通过以上我们简单的演示了如果通过 docker 命令来运行一个 BaGet 服务。BaGet 跨平台、轻量化、易于部署，体验非常不错，大家可以试试。    
[https://github.com/loic-sharma/BaGet](https://github.com/loic-sharma/BaGet)

## 关注我的公众号一起玩转技术   
![](https://ftp.bmp.ovh/imgs/2021/07/53dfa51e55de02e9.jpg)