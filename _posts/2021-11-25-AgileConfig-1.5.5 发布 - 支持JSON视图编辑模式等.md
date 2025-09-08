本次更新加入了2个新的编辑模式：JSON 编辑模式、TEXT 编辑模式。特别是 JSON 编辑模式是大家比较期待的一个功能。因为大家都习惯了 appsettings.json 的配置编辑模式，所以天生的喜欢 JSON 视图。有了 JSON 编辑模式后，大家就可以直接把原来的 appsettings.json 直接复制过来，点击保存就可以原样导入到 AgileConfig 里了。也可以继续使用对象嵌套对象，数组等高级模式。
## JSON 视图编辑模式

点击右上角“编辑 JSON”按钮会弹出 JSON 编辑视图。该编辑框集成了一个 json 代码编辑器- [monaco-editor](https://www.npmjs.com/package/@monaco-editor/react) 方便用户快速的编辑 json 配置文件。顺便提一下 monaco 这个是微软开源的一个编辑器，看它的官方介绍你就知道他有多牛了：The Monaco Editor is the code editor that powers VS Code 。对没错，它就是 VS Code 的编辑器。   
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211125095252.png)   
现在你可以像使用 appsettings.json 一样来定义配置文件了。比如 { x: {b: 'a' } } 对象嵌套对象，比如数组 ['1', '2' , '3'] 。   
注意：
1. 非法的json文件，编辑器会给出提示，并且不能保存
2. 对于 bool 或者 intger 类型定义的时候没有问题，但是保存后系统会默认给转成文本类型比如 false='false' , 1='1' 。因为所有的json内容转换的时候都会存储成文本类型的键值对。但是放心这不会影响你在 .NET 程序里使用 IConfiguration 来读取绑定使用配置。
   
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211125100425.png)   
编辑好json文件后，点击“保存”按钮，系统会对比新老配置，自动列出哪些是“新增”的配置项，哪里是“编辑”的配置项，哪些是“删除”的配置项。
## TEXT 视图编辑模式

除了 JSON 模式的编辑视图，本次更新还加入了一个 TEXT 编辑模式。TEXT 编辑模式其实就是文本类型的键值对编辑模式。   
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211125100305.png)   
点击右上角的“编辑 TEXT”按钮弹出 TEXT 编辑视图。   
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20211125100441.png)   
该编辑模式一行就代表一个配置项。使用等号进行键值对的分割。   
注意：
1. 请严格按 key=value 的格式进行编辑
2. 每一行必须有一个=号
3. 如果有多个=号，那么程序会按第一个=进行分割

## 最后

✨✨✨Github地址：[https://github.com/dotnetcore/AgileConfig](https://github.com/dotnetcore/AgileConfig)  开源不易，欢迎star✨✨✨   

演示地址：[https://agileconfig-server.xbaby.xyz/](https://agileconfig-server.xbaby.xyz/)  超级管理员账号：admin 密码：123456   

## 关注我的公众号一起玩转技术   

![](https://static.xbaby.xyz/qrcode.jpg)