# 训练数据合成

本文介绍如何使用 GraphGen 从领域语料合成高质量的 **SFT（监督微调）训练数据**。

---

## 1 整体思路

GraphGen 合成训练数据的核心逻辑：

1. **构建知识图谱**：从领域文档中抽取实体和关系，形成结构化知识图谱
2. **识别知识盲区**：通过 `quiz_and_judge`，让待训练的 Trainee 模型自测，找出理解最薄弱的知识点（高理解损失边）
3. **优先覆盖盲区**：ECE 划分器优先把高损失知识点分配到子图，确保生成的训练数据恰好覆盖模型短板
4. **生成多样化问答**：对每个子图生成高质量问答对，输出为标准训练格式

---

## 2 推荐配置（SFT 通用）

```yaml
# 读取配置
read:
  input_file: your_corpus.jsonl

# 文本切片配置
split:
  chunk_size: 1024
  chunk_overlap: 100

# 知识图谱构建
build_kg:
  max_loop: 3

# 测验 & 判断（识别知识盲区）
quiz_and_judge:
  enabled: true
  quiz_samples: 2
  re_judge: false

# 子图划分（ECE 优先覆盖盲区）
partition:
  method: ece
  method_params:
    bidirectional: true
    edge_sampling: max_loss      # 优先处理高损失边
    expand_method: max_width
    isolated_node_strategy: ignore
    max_depth: 5
    max_extra_edges: 20
    max_tokens: 256
    loss_strategy: only_edge

# 问答对生成
generate:
  mode: aggregated               # 推荐：聚合模式，生成高质量综合性问答
  data_format: ChatML            # 输出为 ChatML 格式
```

---

## 3 多跳推理数据

如果目标是提升模型的多步推理能力，将 `generate.mode` 改为 `multi_hop`：

```yaml
generate:
  mode: multi_hop
  data_format: ChatML
```

`multi_hop` 模式会在包含多条关系路径的子图中生成需要跨实体推理的问题，帮助模型学习复杂关系推断。

---

## 4 思维链（CoT）数据

```yaml
generate:
  mode: cot
  data_format: ChatML
```

`cot` 模式采用两步生成：
1. 先生成推理路径模板（reasoning path）
2. 再据此生成含完整推理链的答案

适合训练具备逐步推理能力的模型（如 o1 风格）。

---

## 5 运行方式

### CLI 运行

```bash
SYNTHESIZER_MODEL=gpt-4o-mini \
SYNTHESIZER_BASE_URL=https://api.openai.com/v1 \
SYNTHESIZER_API_KEY=sk-xxx \
TRAINEE_MODEL=your-trainee-model \
TRAINEE_BASE_URL=https://your-trainee-api/v1 \
TRAINEE_API_KEY=sk-xxx \
graphg --config_file configs/aggregated_config.yaml
```

### Python API 运行

```python
import time, yaml
from graphgen.graphgen import GraphGen

gg = GraphGen(unique_id=int(time.time()), working_dir="cache")

with open("configs/aggregated_config.yaml") as f:
    config = yaml.safe_load(f)

# Step1-3：文档读取 + 切片 + KG 构建
gg.insert(read_config=config["read"], split_config=config["split"])

# Step4：Quiz & Judge（识别知识盲区）
if config["quiz_and_judge"]["enabled"]:
    gg.quiz_and_judge(quiz_and_judge_config=config["quiz_and_judge"])

# Step5-6：划分 + 生成
gg.generate(
    partition_config=config["partition"],
    generate_config=config["generate"],
)
print("数据保存在:", gg.working_dir)
```

---

## 6 输出数据位置

运行完成后，数据保存在：

```
cache/output/{unique_id}/
├── data.jsonl          # 最终 QA 数据
└── config.yaml         # 本次运行的配置快照
```

---

## 7 与训练框架对接

生成的 ChatML 格式数据可直接用于主流微调框架：

- **[LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory)**：将 `data.jsonl` 注册到 `dataset_info.json` 后直接训练
- **[xtuner](https://github.com/InternLM/xtuner)**：通过 `xtuner convert` 工具处理后训练
