# GraphGen 简介

GraphGen 是一个基于知识图谱的数据合成框架。请查看[**论文**](https://arxiv.org/abs/2505.20416)和[最佳实践](https://github.com/open-sciencelab/GraphGen/issues/17)。

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

以下是在超过 50 % 的 SFT 数据来自 GraphGen 及我们的数据清洗流程时的训练后结果：

|  领域 |                            数据集                            |   我们的方案  | Qwen2.5-7B-Instruct（基线） |
| :-: | :-------------------------------------------------------: | :------: | :---------------------: |
|  植物 | [SeedBench](https://github.com/open-sciencelab/SeedBench) | **65.9** |           51.5          |
|  常识 |                           CMMLU                           |   73.6   |         **75.8**        |
|  知识 |                        GPQA-Diamond                       | **40.0** |           33.3          |
|  数学 |                           AIME24                          | **20.6** |           16.7          |
|     |                           AIME25                          | **22.7** |           7.2           |

### 为什么会有这个工具？

TODO

### 核心功能

TODO



在数据生成后，您可以使用[LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory) 和 [xtuner](https://github.com/InternLM/xtuner)对大语言模型进行微调。
