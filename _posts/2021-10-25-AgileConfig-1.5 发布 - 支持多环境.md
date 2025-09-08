AgileConfig 从发布到现在，收到不同学的 issue 说需要多环境的支持。也就是一个应用在不同的环境下可以配置不同的配置项。这是一个非常有用的功能，就跟我们开发的时候会设置多个 appsettings.json 文件一样，比如 appsettings.development.json 、appsetting.production.json 等等。那么这次 1.5 版本就为大家带来了这个功能。   
下面介绍下如何使用多环境配置功能。

## 运行控制台节点
拉取最新的 latest 或者 release-1.5.0 的 docker 镜像，运行控制台节点即可支持多环境配置。
```
sudo docker run \
--name agile_config \
-e adminConsole=true \
-e db:provider=sqlite \
-e db:conn="Data Source=agile_config.db" \
-p 5000:5000 \
-v /etc/localtime:/etc/localtime \
#-v /your_host_dir:/app/db \
-d kklldog/agile_config:release-1.5.0
```
节点运行起来后，在配置项管理界面的右上角即可切换环境。   
[![5foyh8.png](https://z3.ax1x.com/2021/10/25/5foyh8.png)](https://imgtu.com/i/5foyh8)
## 自定义环境
AgileConfig 默认内置了 DEV, TEST, STAGING, PROD 四个常用的环境，如果用户觉得不够用或者不想要那么多环境的话可以进行自己定义。
找到数据库的 agc_setting 表，对其中 id = environment 的行进行修改。配置名称之间使用英文输入状态的逗号分隔。   
[![5fostf.png](https://z3.ax1x.com/2021/10/25/5fostf.png)](https://imgtu.com/i/5fostf)   
## 为环境单独配置数据库
AgileConfig 默认情况下会把所有的配置项都存储在 db:conn 指定的数据库下面。但是对于多环境来说，集中式的配置存储显然不太合适。特别是对于生产环境来说不太可能跟开发测试环境都部署在同一个数据库上。AgileConfig 支持对某个环境配置单独的数据库。   
在启动节点的时候为某个环境单独配置数据库：
```
-e db:env:TEST:provider=mysql \
-e db:env:TEST:conn= "Database=agile_config_test;Data Source=192.168.0.111;User Id=dev;Password=dev@123;port=3306" \

-e db:env:PROD:provider=mysql \
-e db:env:PROD:conn= "Database=agile_config_prod;Data Source=192.168.0.1111;User Id=dev;Password=dev@123;port=3306" \
```
## 客户端
为配合 AgileConfig 1.5 版本请使用 AgileConfig.Client 1.2 及以上版本。   
```
Install-Package AgileConfig.Client -Version 1.2.1
```
在配置文件上指定环境参数，如果不配置那么默认为  DEV 环境。
```
{
  "AgileConfig": {
    "appId": "test_app",
    "secret": "",
    "env": "DEV"
    "nodes": "http://localhost:5000",
    "name": "client1",
    "tag": "tag1",
  }
}
```
## 最后

✨✨✨Github地址：[https://github.com/dotnetcore/AgileConfig](https://github.com/dotnetcore/AgileConfig)  开源不易，欢迎star✨✨✨   

演示地址：[https://agileconfig-server.xbaby.xyz/](https://agileconfig-server.xbaby.xyz/)  超级管理员账号：admin 密码：123456   

## 关注我的公众号一起玩转技术   

![](https://ftp.bmp.ovh/imgs/2021/07/53dfa51e55de02e9.jpg)