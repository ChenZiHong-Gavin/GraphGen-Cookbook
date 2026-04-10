# 参数

完整的 GraphGen 配置文件由以下几个顶层模块组成，所有模块均在 `config.yaml` 中配置。

---

## 全局参数 `global_params`

```yaml
global_params:
  working_dir: cache          # 工作目录，存放缓存、日志、输出等
  kv_backend: rocksdb         # KV 存储后端，可选 rocksdb / sqlite / memory
  graph_backend: kuzu         # 图数据库后端，目前仅支持 kuzu
```

---

## 读取配置 `read`

| 参数 | 类型 | 说明 |
|------|------|------|
| `input_file` | str | 输入文件路径或目录，支持 json/jsonl/txt/csv/pdf/parquet 等 |
| `allowed_suffix` | list | 目录扫描时限制文件后缀，如 `["jsonl","txt"]` |
| `recursive` | bool | 目录扫描是否递归，默认 `true` |
| `parallelism` | int | 并发读取线程数，默认 1 |

---

## 文本切片配置 `split`

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `chunk_size` | int | `1024` | 每个 chunk 的最大 token 数 |
| `chunk_overlap` | int | `100` | 相邻 chunk 之间的重叠 token 数 |

---

## 知识图谱构建 `build_kg`

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `max_loop` | int | `3` | LLM 抽取的最大迭代轮数，越大召回率越高但更耗时 |

---

## 网络搜索配置 `search`

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enabled` | bool | `false` | 是否开启搜索扩充 |
| `search_types` | list | `["google"]` | 搜索源，可多选：`google`、`bing`、`uniprot`、`ncbi`、`rnacentral`、`interpro` |

---

## 测验与判断 `quiz_and_judge`

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enabled` | bool | `true` | 是否开启，训练数据合成建议开启 |
| `quiz_samples` | int | `2` | 每条关系生成的测验题数量 |
| `re_judge` | bool | `false` | 是否对已存在的测验结果重新判断 |

---

## 子图划分配置 `partition`

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `method` | str | `ece` | 划分算法：`ece`、`bfs`、`dfs`、`leiden`、`triple`、`quintuple`、`anchor_bfs` |

### `method_params` 参数详情

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `bidirectional` | bool | `true` | 是否双向遍历图谱 |
| `edge_sampling` | str | `max_loss` | 边采样策略：`random` \| `max_loss` \| `min_loss` |
| `expand_method` | str | `max_width` | 扩展方式：`max_width`（按边数）\| `max_depth`（按深度）\| `max_tokens`（按 token）|
| `max_extra_edges` | int | `20` | `max_width` 模式下每方向最多扩展的边数 |
| `max_depth` | int | `5` | `max_depth` 模式下最大遍历深度 |
| `max_tokens` | int | `256` | `max_tokens` 模式下子图文本 token 上限 |
| `loss_strategy` | str | `only_edge` | 损失计算范围：`only_edge`（仅边）\| `both`（节点+边）|
| `isolated_node_strategy` | str | `ignore` | 孤立节点处理：`ignore`（忽略）\| `add`（强制加入）|
| `anchor_type` | str | `null` | `anchor_bfs` 专用：锚点节点类型 |
| `anchor_ids` | list | `null` | `anchor_bfs` 专用：指定锚点节点 ID 列表 |

---

## 问答生成配置 `generate`

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `mode` | str | `aggregated` | 生成模式：`atomic`、`aggregated`、`multi_hop`、`cot`、`vqa`、`multi_choice`、`multi_answer`、`fill_in_blank`、`masked_fill_in_blank`、`true_false` |
| `data_format` | str | `ChatML` | 输出格式：`ChatML`、`Alpaca`、`ShareGPT` |
| `num_of_questions` | int | `5` | 适用于 `multi_choice`、`multi_answer`、`fill_in_blank`、`true_false` 模式，每个子图生成的题目数量 |

---

## 环境变量

以下环境变量需在运行前配置（可写入 `.env` 文件）：

| 变量 | 说明 |
|------|------|
| `SYNTHESIZER_MODEL` | 合成器模型名称（用于 KG 构建和 QA 生成）|
| `SYNTHESIZER_BASE_URL` | 合成器 API Base URL |
| `SYNTHESIZER_API_KEY` | 合成器 API Key |
| `TRAINEE_MODEL` | 待训练模型名称（用于 quiz_and_judge）|
| `TRAINEE_BASE_URL` | 待训练模型 API Base URL |
| `TRAINEE_API_KEY` | 待训练模型 API Key |
| `TOKENIZER_MODEL` | Tokenizer 模型名称，默认 `cl100k_base` |

---

## 完整配置示例

```yaml
global_params:
  working_dir: cache
  kv_backend: rocksdb
  graph_backend: kuzu

read:
  input_file: resources/input_examples/jsonl_demo.jsonl

split:
  chunk_size: 1024
  chunk_overlap: 100

build_kg:
  max_loop: 3

search:
  enabled: false
  search_types: ["google"]

quiz_and_judge:
  enabled: true
  quiz_samples: 2
  re_judge: false

partition:
  method: ece
  method_params:
    bidirectional: true
    edge_sampling: max_loss
    expand_method: max_width
    isolated_node_strategy: ignore
    max_depth: 5
    max_extra_edges: 20
    max_tokens: 256
    loss_strategy: only_edge

generate:
  mode: aggregated
  data_format: ChatML
```
