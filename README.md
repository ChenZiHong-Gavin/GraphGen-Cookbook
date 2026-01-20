# GraphGen 简介

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

GraphGen 是一个基于知识图谱的数据合成框架。

{% hint style="info" %}
[**论文: https://arxiv.org/abs/2505.20416**](https://arxiv.org/abs/2505.20416)

[**Github: https://github.com/open-sciencelab/GraphGen**](https://github.com/open-sciencelab/GraphGen)

[**使用文档: https://chenzihong.gitbook.io/graphgen-cookbook/**](https://chenzihong.gitbook.io/graphgen-cookbook/)
{% endhint %}

### 为什么会有这个工具？

在训练 LLM 的过程中，存在一些**真实世界中难以获取或暂时缺失的数据**，这些数据通常需要通过 **合成数据** 来弥补。

* **领域专用知识：**&#x533B;学、法律、工程等专业知识普遍门槛高，标注成本是通用数据的 10–100 倍；长尾知识（罕见病、冷僻法条、设备故障案例）在公开语料中占比不足 1 %，但往往是业务落地的关键点。
* **高阶推理与逻辑链数据：**&#x771F;实世界中高质量的逻辑推理、数学推导、因果链条等数据稀缺。
* **任务特定指令数据：**&#x5982;代码生成、函数调用、阅读理解等任务，真实标注数据获取成本高。
* ……

### 核心功能

* 训练数据合成
* 评测数据合成
* 知识抽取

### 后续步骤

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-cover data-type="image"></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><h4>快速开始</h4></td><td></td><td><a href="https://images.unsplash.com/photo-1636056514473-dd532ed74cf2?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHw3fHxzdGFydHxlbnwwfHx8fDE3Njg4MjA4MDB8MA&#x26;ixlib=rb-4.1.0&#x26;q=85">https://images.unsplash.com/photo-1636056514473-dd532ed74cf2?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHw3fHxzdGFydHxlbnwwfHx8fDE3Njg4MjA4MDB8MA&#x26;ixlib=rb-4.1.0&#x26;q=85</a></td><td><a href="quick_start/">quick_start</a></td></tr><tr><td><h4>流程</h4></td><td></td><td><a href="https://images.unsplash.com/photo-1600492515568-8868f609511e?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHw2fHxwcm9jZXNzfGVufDB8fHx8MTc2ODkwMTM3MXww&#x26;ixlib=rb-4.1.0&#x26;q=85">https://images.unsplash.com/photo-1600492515568-8868f609511e?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHw2fHxwcm9jZXNzfGVufDB8fHx8MTc2ODkwMTM3MXww&#x26;ixlib=rb-4.1.0&#x26;q=85</a></td><td><a href="progress/">progress</a></td></tr><tr><td><h4>参数</h4></td><td></td><td><a href="https://images.unsplash.com/photo-1706879349461-1fdfb4f7d519?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwzfHxwYXJhbWV0ZXJ8ZW58MHx8fHwxNzY4OTAxMzYyfDA&#x26;ixlib=rb-4.1.0&#x26;q=85">https://images.unsplash.com/photo-1706879349461-1fdfb4f7d519?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwzfHxwYXJhbWV0ZXJ8ZW58MHx8fHwxNzY4OTAxMzYyfDA&#x26;ixlib=rb-4.1.0&#x26;q=85</a></td><td><a href="parameters.md">parameters.md</a></td></tr></tbody></table>

在数据生成后，您可以使用[LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory) 和 [xtuner](https://github.com/InternLM/xtuner)对大语言模型进行微调。

以下是在超过 50 % 的 SFT 数据来自 GraphGen 及我们的数据清洗流程时的训练后结果：

|  领域 |                            数据集                            |   我们的方案  | Qwen2.5-7B-Instruct（基线） |
| :-: | :-------------------------------------------------------: | :------: | :---------------------: |
|  植物 | [SeedBench](https://github.com/open-sciencelab/SeedBench) | **65.9** |           51.5          |
|  常识 |                           CMMLU                           |   73.6   |         **75.8**        |
|  知识 |                        GPQA-Diamond                       | **40.0** |           33.3          |
|  数学 |                           AIME24                          | **20.6** |           16.7          |
|     |                           AIME25                          | **22.7** |           7.2           |
