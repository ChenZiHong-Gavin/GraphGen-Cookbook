# 评测数据合成

本文介绍如何使用 GraphGen 从领域语料合成高质量的**评测（Benchmark）数据集**。

---

## 1 整体思路

评测数据的核心要求是**客观性**和**可验证性**。GraphGen 可生成：

- **多选题（multi_choice）**：含干扰项，答案明确
- **判断题（true_false）**：是非判断
- **主观问答（aggregated / multi_hop）**：用于参考答案比对

---

## 2 推荐配置（多选题）

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

# 关闭 quiz_and_judge（评测数据无需针对性覆盖盲区）
quiz_and_judge:
  enabled: false

# 子图划分（随机采样）
partition:
  method: ece
  method_params:
    bidirectional: true
    edge_sampling: random        # 随机采样，保证评测数据分布均匀
    expand_method: max_width
    isolated_node_strategy: ignore
    max_depth: 5
    max_extra_edges: 15
    loss_strategy: only_edge

# 多选题生成
generate:
  mode: multi_choice
  data_format: ChatML
  num_of_questions: 4            # 每道题选项数量
```

---

## 3 判断题（True/False）

```yaml
generate:
  mode: true_false
  data_format: ChatML
  num_of_questions: 3            # 每个子图生成几道判断题
```

---

## 4 多跳主观题

适合评估模型的推理能力：

```yaml
partition:
  method: ece
  method_params:
    edge_sampling: random
    expand_method: max_depth     # 深度优先，保证多跳路径
    max_depth: 3

generate:
  mode: multi_hop
  data_format: ChatML
```

---

## 5 填空题

```yaml
generate:
  mode: fill_in_blank
  data_format: ChatML
  num_of_questions: 5
```

---

## 6 评测数据与训练数据的关键区别

| 方面 | 训练数据 | 评测数据 |
|------|----------|----------|
| `edge_sampling` | `max_loss`（优先盲区）| `random`（均匀分布）|
| `quiz_and_judge` | 建议开启 | 通常关闭 |
| 生成模式 | `aggregated` / `multi_hop` / `cot` | `multi_choice` / `true_false` / `multi_hop` |
| 数据用途 | 模型微调 | 效果验证 |

---

## 7 输出数据位置

```
cache/output/{unique_id}/
├── data.jsonl          # 最终评测题目
└── config.yaml         # 本次运行的配置快照
```

---

## 8 注意事项

1. **保留原始文档作为参考答案来源**：对于主观题评测，建议同时保存分块后的原文，方便人工核验答案正确性。
2. **避免数据污染**：评测数据应来自与训练数据不重叠的语料，建议将语料集提前划分为 train/test 两部分。
3. **选项干扰质量**：`multi_choice` 模式的干扰项由 Synthesizer LLM 生成，使用更强的模型（GPT-4o 等）可显著提升干扰项质量。
