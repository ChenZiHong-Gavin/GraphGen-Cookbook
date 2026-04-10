# Step4 社区划分

知识图谱构建完成后，GraphGen 需要将整张大图**分割成若干子图（社区）**，每个子图作为一条问答对的知识背景送入 LLM 进行生成。

配置入口在 `config.yaml` 的 `partition` 字段：

```yaml
partition:
  method: ece           # 划分算法，见下方算法列表
  method_params:        # 算法参数
    bidirectional: true
    edge_sampling: max_loss
    expand_method: max_width
    isolated_node_strategy: ignore
    max_depth: 5
    max_extra_edges: 20
    max_tokens: 256
    loss_strategy: only_edge
```

---

## 1 划分算法

GraphGen 内置了多种社区划分算法，可通过 `method` 字段切换：

| 方法 | 说明 | 推荐场景 |
|------|------|----------|
| `ece` | 基于期望校准误差（ECE/理解损失）的划分，优先把 LLM 掌握最差的知识点分到同一社区 | **默认推荐**，训练数据合成 |
| `bfs` | 广度优先遍历扩展子图 | 快速验证、简单场景 |
| `dfs` | 深度优先遍历扩展子图 | 长链推理场景 |
| `leiden` | 基于模块度最优的 Leiden 社区发现算法 | 无监督社区结构探索 |
| `triple` | 以单条三元组（head-relation-tail）为最小社区单元 | 原子级问答生成 |
| `quintuple` | 以五元组为社区单元 | 细粒度问答生成 |
| `anchor_bfs` | 以指定节点为锚点做 BFS 扩展 | 以特定实体为中心的问答生成 |

---

## 2 ECE 算法详解（推荐）

ECE（Expected Calibration Error）划分器是 GraphGen 的核心创新之一。

**核心思路：**

1. **理解损失（Comprehension Loss）**：通过 `quiz_and_judge` 步骤，让 Trainee LLM 对图谱中每条边回答问题，并计算其答错概率（理解损失）。损失越高说明模型对该知识点掌握越差。
2. **优先选择高损失边**：子图扩展时，`edge_sampling: max_loss` 会优先把 LLM 不熟悉的边纳入社区，确保生成的训练数据覆盖知识盲区。
3. **BFS 扩展**：以高损失边为种子，用 BFS 向外扩展，直到达到 token 上限或边数上限。

**参数说明：**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `bidirectional` | bool | `true` | 是否双向遍历图谱 |
| `edge_sampling` | str | `max_loss` | 边采样策略：`random` \| `max_loss` \| `min_loss` |
| `expand_method` | str | `max_width` | 扩展方式：`max_width`（按边数）\| `max_depth`（按深度）\| `max_tokens`（按 token）|
| `max_extra_edges` | int | `20` | `max_width` 模式下每方向最多扩展的边数 |
| `max_depth` | int | `5` | `max_depth` 模式下最大遍历深度 |
| `max_tokens` | int | `256` | `max_tokens` 模式下子图文本总 token 上限 |
| `loss_strategy` | str | `only_edge` | 损失计算范围：`only_edge`（仅边）\| `both`（节点+边）|
| `isolated_node_strategy` | str | `ignore` | 孤立节点处理：`ignore`（忽略）\| `add`（强制加入）|

---

## 3 Quiz & Judge 前置步骤

使用 ECE 划分且 `edge_sampling` 不为 `random` 时，需要先运行 `quiz_and_judge` 步骤以获取每条边的理解损失：

```yaml
quiz_and_judge:
  enabled: true       # 是否开启
  quiz_samples: 2     # 每条关系对应生成多少道测验题
  re_judge: false     # 是否对已有测验结果重新判断
```

若跳过此步骤（或将 `edge_sampling` 设为 `random`），则所有边损失默认相等，退化为普通 BFS。

---

## 4 常见问题

**Q1：社区太大导致 LLM 上下文超限？**

调小 `max_extra_edges` 或 `max_tokens`，或将 `expand_method` 切换为 `max_tokens` 并设置合适上限。

**Q2：生成的问答对知识覆盖度不够？**

开启 `quiz_and_judge` 并使用 `edge_sampling: max_loss`，让 ECE 优先覆盖模型薄弱环节。

**Q3：如何使用 Leiden 算法？**

```yaml
partition:
  method: leiden
  method_params: {}
```

Leiden 算法不依赖 `quiz_and_judge`，可独立运行，适合无监督的社区探索场景。
