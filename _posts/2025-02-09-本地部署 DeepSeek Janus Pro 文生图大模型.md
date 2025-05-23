Hello, 大家新年好。   
在这个春节期间最火的显然是 DeepSeek 了。据不负责统计朋友圈每天给我推送关于 DeepSeek 的文章超过20篇。打开知乎跟B站也全是 DeepSeek 相关的内容。不过大部分的内容都是关于 DeepSeek R1 推理模型有多牛逼。在这里就不多说关于 R1 的废话了，因为大家已经看腻了。    
R1 在本地用 ollama 跑了一下，太简单了，都没必要写个教程。除了 R1 今天还在本地部署成功了 DeepSeek 的 Janus Pro 模型。   
## 什么是 Janus Pro
Janus-Pro是一种创新的自回归框架，其统一了多模态理解与生成任务。该框架通过将视觉编码解耦到不同的处理路径（同时仍使用单一统一的Transformer架构进行信息处理），有效解决了先前方法的局限性。这种解耦机制不仅缓解了视觉编码器在理解与生成双重角色间的冲突，还显著提升了框架的灵活性。Janus-Pro在性能上超越了以往的统一模型，并达到甚至超越了专用任务模型的表现水平。凭借其架构简洁性、高度灵活性和卓越有效性，Janus-Pro有望成为下一代统一多模态模型的重要技术方向。

以上内容来自 Janus Pro github 仓库的介绍，非常的学术。简单说它是文生图的模型，类似 DALL-E 3, Stable Diffusion。   

以下就让我们看看这么在本地的 PC 电脑来运行 Janus Pro 模型吧。
## 1. 安装 conda
从以下地址下载 anaconda 的 windows 安装包
https://www.anaconda.com/download    
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20250202163525.png)


安装完成后需要把安装目录配置到环境变量 PATH 上

## 2. 创建 python 虚拟环境

```
conda create -n janus_pro python=3.10 -y
conda activate janus_pro
```

注意：Python 3.10是经过验证的兼容版本，避免使用其他版本导致依赖冲突

## 3. 克隆 janus 仓库到本地
```
git clone https://github.com/deepseek-ai/Janus.git
cd Janus
```

## 4. 安装依赖
```
# 安装基础依赖
pip install -e .

```
注意：pip 安装依赖的时候可能会遇到网络问题，建议配置代理

## 5. 安装 Gradio

```
pip install -e .[gradio]
```
到时候我们会通过 Gradio 的界面跟 janus 进行交互

## 6. 使用 janus pro 1B 模型
janus pro 默认启动的时候使用的是 7B 参数的模型，本地电脑跑起来的话太卡了。这里我们会修改成使用 1B 模型，这样的话大概 8G 的显存也能勉强跑一跑,7B 的话对显存的要求会更高。


找到我们克隆下来的仓库。使用编辑器打开 demo/
把第15行改成：
```
model_path = "deepseek-ai/Janus-Pro-1B"
```

## 7. 启动 Janus Pro

```
python demo/app_januspro.py
```

注意：启动期间会从 huggingface 拉取 1B 模型，大小大概 4G，所以还是需要指定代理。
启动成功后如下图：

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20250203005730.png)

## 8. 使用 Gradio 界面进行交互
启动成功后在浏览器里输入: http://127.0.0.1:7860 即可访问 Gradio 页面。

![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250203010252.png)

## 9. 图片理解
先来试试 janus pro 对图片的理解。
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250203022406.png)

```
This image is a humorous meme that uses the popular "buff Doge vs. sad Doge" format to compare two different approaches to visual encoding.
...
```
这个解释到位的有点吓人。
## 10. 文生图
再来试试文字生成图片

prompt：
```
A cute and adorable baby fox with big brown eyes, autumn leaves in the background enchanting,immortal,fluffy, shiny mane,Petals,fairyism,unreal engine 5 and Octane Render,highly detailed, photorealistic, cinematic, natural colors.
```
![](https://static.xbaby.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250203025317.png)  
图片是我从 Hugging Face 直接运行得到的。在我本地运行了30分钟都没出结果，我直接 ctrl-c 取消了。可能是我的显卡太垃圾了(RTX4060 Mobile)。 

## 总结
以上我们在本地 windows 上成功部署了 DeepSeek janus pro 模型。按照以上 step by step 的方式也没什么难度。通过测试 janus pro 对图片的理解非常到位。但是文生图的测试失败了，可能是我的显卡太垃圾，如果有同学有 4090 这种显卡可以试一试本地文生图的性能。