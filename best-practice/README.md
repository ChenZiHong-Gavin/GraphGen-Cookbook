# 案例

本章提供 GraphGen 在不同场景下的完整使用案例，帮助你快速找到最适合自己需求的配置。

---

## 案例一：通用领域 SFT 训练数据合成

**场景**：从 Wikipedia 段落合成通用知识问答对，用于 LLM 的持续预训练或指令微调。

**推荐配置**：

```yaml
read:
  input_file: wiki_corpus.jsonl

split:
  chunk_size: 1024
  chunk_overlap: 100

build_kg:
  max_loop: 3

quiz_and_judge:
  enabled: true
  quiz_samples: 2

partition:
  method: ece
  method_params:
    edge_sampling: max_loss
    expand_method: max_width
    max_extra_edges: 20

generate:
  mode: aggregated
  data_format: ChatML
```

**特点**：ECE + `max_loss` 优先生成模型不熟悉的知识点，避免重复训练已知内容。

---

## 案例二：生物医学领域专业数据合成

**场景**：从 PubMed 摘要合成蛋白质/基因相关问答，同时借助 UniProt 数据库扩充知识。

**推荐配置**：

```yaml
read:
  input_file: pubmed_abstracts.jsonl

split:
  chunk_size: 512
  chunk_overlap: 50

build_kg:
  max_loop: 5           # 专业文本建议增大抽取轮数

search:
  enabled: true
  search_types: ["uniprot", "ncbi"]
  uniprot_params:
    max_results: 3

quiz_and_judge:
  enabled: true
  quiz_samples: 3

partition:
  method: ece
  method_params:
    edge_sampling: max_loss
    expand_method: max_width
    max_extra_edges: 15

generate:
  mode: multi_hop       # 多跳推理，捕获蛋白质-基因-疾病的复杂关系
  data_format: ChatML
```

---

## 案例三：评测 Benchmark 合成（多选题）

**场景**：从领域教材合成客观多选题，用于评估模型在该领域的知识水平。

**推荐配置**：

```yaml
read:
  input_file: textbook_chapters.jsonl

split:
  chunk_size: 1024
  chunk_overlap: 100

build_kg:
  max_loop: 3

quiz_and_judge:
  enabled: false        # 评测数据无需针对性覆盖盲区

partition:
  method: ece
  method_params:
    edge_sampling: random   # 随机采样，保证分布均匀
    expand_method: max_width
    max_extra_edges: 10

generate:
  mode: multi_choice
  data_format: ChatML
  num_of_questions: 4
```

---

## 案例四：CoT 推理数据合成

**场景**：合成含推理链的训练数据，用于训练具备逐步推理能力的模型。

**推荐配置**：

```yaml
read:
  input_file: reasoning_corpus.jsonl

split:
  chunk_size: 1024
  chunk_overlap: 100

build_kg:
  max_loop: 3

quiz_and_judge:
  enabled: false        # CoT 模式无需 quiz_and_judge

partition:
  method: ece
  method_params:
    edge_sampling: random
    expand_method: max_depth  # 深度优先，构建更长的推理路径
    max_depth: 4

generate:
  mode: cot
  data_format: ChatML
```

---

## 案例五：以特定实体为中心的知识抽取

**场景**：针对某个特定实体（如"GPT-4"）抽取其关联的所有知识，生成以该实体为核心的问答对。

**推荐配置**：

```yaml
read:
  input_file: ai_papers.jsonl

split:
  chunk_size: 1024
  chunk_overlap: 100

build_kg:
  max_loop: 3

quiz_and_judge:
  enabled: false

partition:
  method: anchor_bfs    # 锚点 BFS
  method_params:
    anchor_type: "model"       # 实体类型
    anchor_ids: ["GPT-4", "ChatGPT"]   # 指定锚点实体
    expand_method: max_width
    max_extra_edges: 20

generate:
  mode: aggregated
  data_format: ChatML
```
