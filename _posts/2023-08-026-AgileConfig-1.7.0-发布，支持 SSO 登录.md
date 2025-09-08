
AgileConfig 已经好久好久没有更新过比较大的功能了。一是 AgileConfig 本身的定位就是比较轻量，不想集成太多的功能。二是比较忙（懒）。但是本次升级给大家带来了一个比较有用的功能 SSO。   
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20230827014736.png)   
SSO 嘛大家都懂，单点登录，稍微上点规模的公司内部都会有统一的单点登录服务。   
目前 SSO 主流协议基本上就是两种：   
* OIDC(OAuth2.0) - OpenID Connect
* SAML 2.0 - Security Assertion Markup Language

本次 SSO 的实现采用了基于 OIDC 协议的 Code Flow 模式来实现，可以说这是目前市面上最流行的集成方案。  
由于这次不是讨论 OIDC 的具体实现，关于 OIDC 相关的知识就不多说了。   
![](https://static.xbaby.xyz/code-flow.webp)  
图片出处：https://docs.walt.id/v/idpkit/concepts/oidc-recap
## 如何使用
1. 升级 AgileConfig 到最新版本或者 tag:1.7.0 以上
2. 在配置文件或者环境变量中配置 SSO 相关信息

以下对配置的参数进行详细说明：
## 参数说明
| 参数 | 说明 | 示例 |
|---|---|---|
| SSO:enabled   | 是否开启 sso  |  true |
|  SSO:loginButtonText |  自定义 SSO 跳转按钮的文字 | Azure SSO  |
|  SSO:OIDC:clientId | OIDC 客户端 ID  | 2bb823b7-f1ad-48c7-a9a1-713e9a885a5d  |
| SSO:OIDC:clientSecret | OIDC 客户端 密钥 | 6B29FC40-CA47-1067-B31D-00DD010662DA |
| SSO:OIDC:redirectUri | OIDC Server 授权成功后的回调地址, 默认为服务部署域名（或者ip+port）+ /sso | http://localhost:5000/sso |
| SSO:OIDC:tokenEndpoint | code 获取 token 的地址，这个地址一般在 OIDC 服务商那里会明确告知 | https://login.microsoftonline.com/7aa25791-9a8c-4be4-872f-289bfec8cddb/oauth2/v2.0/token |
| SSO:OIDC:tokenEndpointAuthMethod | 获取 token 接口的认证方案，目前支持：client_secret_post, client_secret_basic, none 三种方案，默认为：client_secret_post  | client_secret_post |
| SSO:OIDC:authorizationEndpoint | OIDC Server 授权地址，通常是 OIDC 服务商会明确告知。本地服务会加上 response_type，redirect_uri 等参数，构造出完整的授权 URL | https://login.microsoftonline.com/7aa25791-9a8c-4be4-872f-289bfec8cddb/oauth2/v2.0/authorize |
| SSO:OIDC:userIdClaim | ID token 中用户 ID 的 claim key，默认为 sub | sub |
| SSO:OIDC:userNameClaim | ID token 中用户 name 的 claim key，默认为 name | name |
| SSO:OIDC:scope | token 携带的 claim 的范围，默认 openid profile | openid profile |

如果使用源码运行请对 appsettings.json 进行修改，示例如下：
```
  "SSO": {
    "enabled": true, 
    "loginButtonText": "SSO",
    "OIDC": {
      "clientId": "2bb823b7-f1ad-48c7-a9a1-713e9a885a5d",
      "clientSecret": "", 
      "redirectUri": "http://localhost:5000/sso", 
      "tokenEndpoint": "https://login.microsoftonline.com/7aa25791-9a8c-4be4-872f-289bfec8cddb/oauth2/v2.0/token",
      "tokenEndpointAuthMethod": "client_secret_post", client_secret_post, client_secret_basic, none. default=client_secret_post.
      "authorizationEndpoint": "https://login.microsoftonline.com/7aa25791-9a8c-4be4-872f-289bfec8cddb/oauth2/v2.0/authorize", 
      "userIdClaim": "sub", 
      "userNameClaim": "name", 
      "scope": "openid profile" 
    }
  }
```
如果使用 docker compose 运行请使用环境变量修改配置：
```

  agile_config:
    image: "kklldog/agile_config:latest"
    ports:
      - "15000:5000"
    networks:
      - net0
    volumes:
      - /etc/localtime:/etc/localtime
    environment:
      - TZ=Asia/Shanghai
      - adminConsole=true
      - db:provider=mysql
      - db:conn= Allow User Variables=true;database=agile_config_preview;data source=mysql8;User Id=root;password=1;

      - SSO:enabled=true
      - SSO:loginButtonText=Azure SSO
      - SSO:OIDC:clientId=2bb823b7-f1ad-48c7-a9a1-713e9a885a5d
      - SSO:OIDC:clientSecret=1
      - SSO:OIDC:redirectUri=https://agileconfig-server.xbaby.xyz/sso
      - SSO:OIDC:tokenEndpoint=https://login.microsoftonline.com/7aa25791-9a8c-4be4-872f-289bfec8cddb/oauth2/v2.0/token
      - SSO:OIDC:authorizationEndpoint=https://login.microsoftonline.com/7aa25791-9a8c-4be4-872f-289bfec8cddb/oauth2/v2.0/authorize

```
## 数据库表更新
本次发布对 `agc_user` 表进行了修改，如是从低版本升级上来的请手动调整数据库：   
* id 长度增加到 50
* 新增一个字段`source` ，mysql的类型为 enum(Normal, SSO)，sql server 的类型为 int

## 后续
目前 SSO、OIDC 的相关配置通过配置文件或者环境变量来配置略显麻烦，后面如有时间会新增相关界面来进行配置，敬请期待。如果同学你有时间，那么可以给我 PR ，让我们一起为 .NET 的生态尽一份力。

## 最后

✨✨✨ Github地址：[https://github.com/dotnetcore/AgileConfig](https://github.com/dotnetcore/AgileConfig)  开源不易，欢迎 star ✨✨✨   

演示地址：[https://agileconfig-server.xbaby.xyz/](https://agileconfig-server.xbaby.xyz/)  超管账号：admin 密码：123456  

## 关注我的公众号一起玩转技术   

![](https://static.xbaby.xyz/qrcode.jpg)