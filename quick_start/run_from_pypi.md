# 从PyPI安装

### 1 安装GraphGen

```
pip install graphg==0.1.0.post20250930
```

### 2 准备配置文件

参考[https://github.com/open-sciencelab/GraphGen/tree/main/graphgen/configs](https://github.com/open-sciencelab/GraphGen/tree/main/graphgen/configs)，下面以`aggregated_config.yaml`为例。

```yaml
# 读取配置
read:
  input_file: resources/input_examples/jsonl_demo.jsonl  # 输入文件路径，支持 json、jsonl、txt，参考 resources/input_examples 中的示例

# 文本切片配置
split:
  chunk_size: 1024      # 每块文本的最大 token 数
  chunk_overlap: 100    # 相邻文本块之间的重叠 token 数

# 网络搜索配置
search:
  enabled: false        # 是否启用联网搜索
  search_types: ["google"]  # 搜索引擎类型，可选：google、bing、uniprot、wikipedia

# 测验 & 判断：让 LLM 自检是否真正掌握知识点
quiz_and_judge:
  enabled: true         # 是否开启「出题-判断」流程
  quiz_samples: 2       # 一条关系对应出多少题
  re_judge: false       # 是否对已有测验结果重新判断

# 子图划分配置
partition:
  method: ece           # 分区方法，目前仅内置 ece（基于理解损失的分区算法），以下参数保持默认即可
  method_params:
    bidirectional: true             # 是否双向遍历图谱
    edge_sampling: max_loss         # 边采样策略：random（随机）| max_loss（损失最大）| min_loss（损失最小）
    expand_method: max_width        # 子图扩张方式：max_width（广度优先）| max_depth（深度优先）| max_tokens（按 token 上限）
    isolated_node_strategy: ignore  # 孤立节点处理：ignore（忽略）| add（强制加入）
    max_depth: 5                    # 最大遍历深度（深度优先时生效）
    max_extra_edges: 20             # 每方向最多扩展的边数（广度优先时生效）
    max_tokens: 256                 # 子图文本总 token 上限（max_tokens 模式时生效）
    loss_strategy: only_edge        # 损失计算维度：only_edge（仅边）| both（节点+边）

# 问答对生成配置
generate:
  mode: aggregated      # 生成模式：atomic（单跳）| aggregated（聚合）| multi_hop（多跳）| cot（思维链）
  data_format: ChatML   # 输出格式：Alpaca | ShareGPT | ChatML
```

### 3 运行

#### 2.1 在CLI中运行

1. 运行命令

```bash
TOKENIZER_MODEL=your_tokenizer_model_name \
SYNTHESIZER_MODEL=your_synthesizer_model_name \
SYNTHESIZER_BASE_URL=your_base_url_for_synthesizer_model \
SYNTHESIZER_API_KEY=your_api_key_for_synthesizer_model \
TRAINEE_MODEL=your_trainee_model_name \
TRAINEE_BASE_URL=your_base_url_for_trainee_model \
TRAINEE_API_KEY=your_api_key_for_trainee_model \
graphg --config_file your_path_for_config_file --output_dir cache
```

运行结果如下：

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

#### 2.2 使用GraphGen构建生成程序

下面是使用 GraphGen 生成数据的示例程序：

```python
import argparse
import time
from pathlib import Path
import os
import yaml

from graphgen.graphgen import GraphGen



os.environ["TOKENIZER_MODEL"] = "cl100k_base" # your_tokenizer_model_name
os.environ["SYNTHESIZER_MODEL"] = "gpt-4o-mini" # your_synthesizer_model_name
os.environ["SYNTHESIZER_BASE_URL"] = "https://api.xxx" # your_base_url_for_synthesizer_model
os.environ["SYNTHESIZER_API_KEY"] = "sk-xxx" # your_api_key_for_synthesizer_model
os.environ["TRAINEE_MODEL"] = "gpt-4o-mini" # your_trainee_model_name
os.environ["TRAINEE_BASE_URL"] = "https://api.xxx" # your_base_url_for_trainee_model
os.environ["TRAINEE_API_KEY"] = "sk-xxx" # your_api_key_for_trainee_model



def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--config_file", required=True, help="YAML 配置文件路径")
    parser.add_argument("--output_dir", default="./cache", help="工作目录")
    args = parser.parse_args()

    cfg_path = Path(args.config_file)
    output_dir = Path(args.output_dir)
    output_dir.mkdir(parents=True, exist_ok=True)

    with open(cfg_path, "r", encoding="utf-8") as f:
        config = yaml.safe_load(f)

    # 实例化 GraphGen（自动读环境变量）
    gg = GraphGen(unique_id=int(time.time()), working_dir=str(output_dir))

    # 1. 文档插入 & 知识图谱抽取
    gg.insert(read_config=config["read"], split_config=config["split"])

    # 2. 测验 & 判断
    mode = config["generate"]["mode"]
    if mode in ["atomic", "aggregated", "multi_hop"]:
        if "quiz_and_judge" in config and config["quiz_and_judge"]["enabled"]:
            gg.quiz_and_judge(quiz_and_judge_config=config["quiz_and_judge"])
        else:
            config["partition"]["method_params"]["edge_sampling"] = "random"
    elif mode == "cot":
        pass
    else:
        raise ValueError(f"未知的生成模式: {mode}")

    # 3. 生成问答对
    gg.generate(
        partition_config=config["partition"],
        generate_config=config["generate"],
    )

    print("数据生成完成，结果保存在:", gg.working_dir)


if __name__ == "__main__":
    main()

```

生成结果如下：

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### 4 生成数据示例

```json
    {
        "messages": [
            {
                "role": "user",
                "content": "How has the domestication of rice affected the genetic diversity of the TAC4 gene?"
            },
            {
                "role": "assistant",
                "content": "The process of domestication involves the cultivation and enhancement of wild plants for agricultural use, which has significantly impacted the genetic diversity of TAC4 in rice. Specifically, this domestication has resulted in a bottleneck effect, a genetic phenomenon characterized by a reduction in genetic diversity within the TAC4 population. Therefore, the domestication of rice has not only transformed its cultivation but has also led to a notable decrease in the genetic variability of this important gene."
            }
        ]
    },
```

更多示例可见：[https://github.com/open-sciencelab/GraphGen/tree/main/resources/output\_examples](https://github.com/open-sciencelab/GraphGen/tree/main/resources/output_examples)

