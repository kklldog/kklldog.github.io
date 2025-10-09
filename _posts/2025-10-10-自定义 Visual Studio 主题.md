大家好，今天来说个有意思的东西。熟悉的 Visual Studio 的都知道，它内置了几套主题：深色，浅色，蓝色。现在貌似大家都喜欢深色主题。当然了我们也可以在 Market Place 里下载其他主题。但是每个主题总有一些设置不太喜欢的设定。   
那么有没有办法自己定义一套主题呢？   
今天就教大家如何自定义 VS 主题。

## 下载 Theme 设计器插件
首先在 Visual Studio Marketplace 下载名为：Visual Studio Color Theme Designer 2022 的插件。
![](https://static.xbaby.xyz/ScreenShot_2025-10-09_235811_922.png)
下载完成后进行安装，这不多赘述。

## 建立新的主题项目
安装完这个插件后，新建项目的时候会多一个模版。   
![](https://static.xbaby.xyz/ScreenShot_2025-10-10_000743_399.png)
搜索 Theme 选择 VSThemeProject 。
![](https://static.xbaby.xyz/ScreenShot_2025-10-10_000833_814.png)
取个名字 Mytheme 后打开这个这个项目。双击 "CsutomThem.vstheme" 会跳出主题色设置界面。
![](https://static.xbaby.xyz/ScreenShot_2025-10-10_001016_211.png)
选择一个 base 的主题，通过修改这个主题来得到一个新的主题。为什么不从头开始定义一个主题呢？因为要设置颜色的地方是在是太多了，初步估计有几千吧。

## 设置颜色
我们以 Dark 颜色为基础尝试吧 string 字符串的颜色改成绿色，把注释 comment 的颜色改成红色。
![](https://static.xbaby.xyz/ScreenShot_2025-10-10_005958_501.png)
我们从 All elements 页面上找到这个2个设置。眼神一点要好一点，不然真的找不到，因为实在是太多了。
## 应用主题
当我们改好了颜色，点击 Apply 按钮会自动安装这个新的主题。但是现在你还看不到它。需要关闭 Visual Studio 后重新打开，然后在主题下拉列表就会看到刚才自定义的 Mytheme 主题了。
![](https://static.xbaby.xyz/ScreenShot_2025-10-10_010308_697.png)
让我们试试新主题的效果。
![](https://static.xbaby.xyz/ScreenShot_2025-10-10_005746_485.png)
