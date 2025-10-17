# GraphGen 简介

GraphGen 是一个基于知识图谱的数据合成框架。

[**论文: https://arxiv.org/abs/2505.20416**](https://arxiv.org/abs/2505.20416)

[**Github: https://github.com/open-sciencelab/GraphGen**](https://github.com/open-sciencelab/GraphGen)

[**使用文档: https://chenzihong.gitbook.io/graphgen-cookbook/**](https://chenzihong.gitbook.io/graphgen-cookbook/)

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### 为什么会有这个工具？

在训练 LLM 的过程中，存在一些**真实世界中难以获取或暂时缺失的数据**，这些数据通常需要通过 **合成数据** 来弥补。

* **领域专用知识：**&#x533B;学、法律、工程等专业知识普遍门槛高，标注成本是通用数据的 10–100 倍；长尾知识（罕见病、冷僻法条、设备故障案例）在公开语料中占比不足 1 %，但往往是业务落地的关键点。
* **高阶推理与逻辑链数据**
  * 真实世界中高质量的逻辑推理、数学推导、因果链条等数据稀缺。
* **任务特定指令数据**
  * 如代码生成、函数调用、阅读理解等任务，真实标注数据获取成本高。
* ……

### 核心功能

TODO

### 后续步骤

可以查看 [quick\_start](quick_start/ "mention") 来进行下一步。

在数据生成后，您可以使用[LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory) 和 [xtuner](https://github.com/InternLM/xtuner)对大语言模型进行微调。

以下是在超过 50 % 的 SFT 数据来自 GraphGen 及我们的数据清洗流程时的训练后结果：

|  领域 |                            数据集                            |   我们的方案  | Qwen2.5-7B-Instruct（基线） |
| :-: | :-------------------------------------------------------: | :------: | :---------------------: |
|  植物 | [SeedBench](https://github.com/open-sciencelab/SeedBench) | **65.9** |           51.5          |
|  常识 |                           CMMLU                           |   73.6   |         **75.8**        |
|  知识 |                        GPQA-Diamond                       | **40.0** |           33.3          |
|  数学 |                           AIME24                          | **20.6** |           16.7          |
|     |                           AIME25                          | **22.7** |           7.2           |
