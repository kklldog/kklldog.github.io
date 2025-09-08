Hello 大家好，先祝福大家新年快乐。🎉🎉🎉 `AgileConfig` `1.9.0` 版本终于赶在农历年前发布了。   
`Mongodb` 当前做为一款非常成熟的 `Nosql` 产品，已经有越来越多的产品或项目基于它来构建。在 `AgileConfig` 开源的这几年之间，陆陆续续收到不少同学问为啥不支持 `Mongodb`。我的回答是没有时间（懒）。一是因为确实没有时间，二是要支持 `Mongodb` 这种非关系型数据库，需要对 `AgileConfig` 项目本身做比较大的重构。就在去年 12 月 `AgileConfig` 收到了对于 `Mongodb` 的支持的 PR。这是 `AgileConfig` 开源这几年来收到的一个最大的 PR。往常大家都是嘴上说要这个，要那个功能，但是真正动手的聊聊无几，收到这个 PR 的时候让我非常欣慰。   
这个 PR 当时虽然能工作，但是不够完美。在我跟 `pengqian089` 同学多次沟通后决定对 `AgileConfig` 进行一次比较大的重构：在 `RDB` 与 `Nosql` 之间在抽象一层仓储层。这样对与后续扩展不管是 `RDB` 还是其他 `Nosql` 会更加的容易。同时为了保证项目的可靠性，我们还改进跟添加了更多的单元测试用例。我们共同合作 2 个多月，修改了超过 170 个文件，终于有了当前这个新版本。
## 如何使用 Mongodb 作为存储
要使用 `mongodb` 作为存储，同样非常简单。   
如果是本地使用编译后运行那么请获取最新源码后修改 appsettings.json 文件:
```
"db": {
  "provider": "mongodb",
  "conn": "mongodb://192.168.0.125:27017/agile_config_database"
}
```
如果使用 docker 运行请使用环境变量注入参数：
```
sudo docker run \
--name agile_config \
-e TZ=Asia/Shanghai \
-e adminConsole=true \
-e db__provider=mongodb \
-e db__conn="mongodb://192.168.0.125:27017/agile_config_database" \
-p 5000:5000 \
#-v /your_host_dir:/app/db \
-d kklldog/agile_config:latest
```
运行起来后使用体验跟使用 `mysql` 等数据库并无差异。
> 注意：请尽量使用mongodb集群作为存储，因为单节点 `mongodb` 并不支持事务。

## break change
如果是从老版本升级到 `1.9.0` 版本，那么请注意目前 `agc_sys_log` 系统日志表的主键 id 数据类型从原来的自增 integer 修改成了 varchar36 字符型。升级前请自行修改表结构。原因是 mongodb 不支持自增主键。

## 最后
最后再次感谢 pengqian089 同学的贡献。也期待更多的同学能够支持 AgileConfig， 多多使用，多多 PR。新的一年里让我们为 .NET 生态做更多的贡献。   

✨✨✨ Github地址：[https://github.com/dotnetcore/AgileConfig](https://github.com/dotnetcore/AgileConfig)  开源不易，欢迎 star ✨✨✨   

演示地址：[https://agileconfig-server.xbaby.xyz/](https://agileconfig-server.xbaby.xyz/)  超管账号：admin 密码：123456  

## 关注我的公众号一起玩转技术   

![](https://static.xbaby.xyz/qrcode.jpg)
