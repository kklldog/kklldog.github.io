AI Agent 无疑是今年最火爆的概念，从科技巨头的战略布局到创业公司的创新产品，AI 智能体正在重塑我们与机器交互的方式。无论是自动化任务、个性化服务，还是复杂问题的协同解决，AI Agent 都展现出了前所未有的潜力。

而在众多备受瞩目的框架中，微软 Autogen 凭借其灵活的多智能体协作能力，迅速成为开发者与企业的关注焦点。它不仅能高效整合多个 AI Agent，还能根据任务需求动态调整工作流程，让智能协作变得更简单、更强大。

在这篇博客中，我们将深入探索 Autogen 的核心特性、应用场景，以及它如何为下一代 AI 应用铺平道路。

## AutoGen

AutoGen是一种框架，用于使用多个代理来开发大型语言模型(LLM) 应用程序，这些代理相互对话以解决任务。 使用AutoGen 生成的代理可以在采用LLM、人工输入和工具组合的各种模式下运行。 AutoGen 代理的一个重要工具类型是代码执行程序。 它们使代理能够编写和执行代码以执行复杂的任务。

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250406145607.png)

## AutoGen Studio

AutoGen Studio 是微软研发的一款功能强大的低代码界面工具，旨在简化多智能体应用的构建流程。 它基于AutoGen 框架之上，该框架是一个用于定义、配置和组合AI 代理以驱动多智能体应用的开源Python 框架。

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250406145311.png)

## 安装 AutoGen Studio
不多赘述，参考微软的文档，没啥好说的。

[autogenstudio installation](https://microsoft.github.io/autogen/stable/user-guide/autogenstudio-user-guide/installation.html)


## 模拟软件开发团队
下面让我们使用 AutoGenStudio 来模拟一个软件开发团队。当这个团队接受到开发任务的时候，每个队员可以各施其职，配合着完成任务。

首先让我们在 AutoGenStudio 里面定义一个 team。定义的时候需要指定使用的模型，推出条件。以及 prompt。

这个 team 的定义本质上也是一个 Agent，它的任务是根据上下文选择团队成员（其他 Agent）去执行对应的任务。


![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250406141803.png)

### Lily 前端开发工程师
首先我们创建一个 Agent 来模拟前端开发工程师。他的任务是开发前端代码，比如编写 html，css 等等。
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250406141841.png)
这里我们同样需要指定使用的模型以及 System Message。System Message 是在描述这个 Agent 的能力与职责。

### Jim 后端开发工程师
我们使用同样的方式定义一个后端开发工程师的 Agent。不同的是这次它能够使用 Tool，来执行一些 python 代码。

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250406141858.png)

### UserProxyAgent

UserProxyAgent 是个特殊的 Agent，它不与 LLM 进行交互，它的职责是跟真实的人类进行交互。当某些情况需要人类介入的时候，会以一个输入的方式等待人类给出明确的信息。

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250406141913.png)

在完成所有定义后，我们的软件开发团队结构如下：

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250406141741.png)


## 设计一个登录界面
当我们定义好这个team后，就可以给安排任务了。下面我们就给他们安排一个活：设计一个网页的登录页面。
要求如下：
```
设计一个登录界面，包含一个用户名名输入框，密码输入框，一个登录按钮。但是不需要调用任何后端API。因为我只想看看前端的效果。前端的代码请全部包含在一个页面里，不要把 css,javascript 等分开。一旦前端完成代码后，请把结果交给后端开发工程师，后端开发请使用 fastapi 建立一个服务，用户通过这个服务在浏览器里直接对前端设计的页面进行预览。
```
任务输入进去后可以看到各个 Agent 开始工作了，先是前端设计了页面，输出了 html，css 文件。最后后端工程师使用 python 直接生成了一个 web service 承载了这页面。我们访问能直接输入这个刚刚设计的登录框。

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250406142503.png)

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250406142833.png)

这个讨论的过程大家可以见以下视频：

【AutogenStudio 构建你的私人开发团队】 
https://www.bilibili.com/video/BV1Qzo1YAEk7/?share_source=copy_web&vd_source=3f96a750277e9e3babf014a139c50726


