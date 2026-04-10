# Step3 构建知识图谱

GraphGen 在完成文本分块后，会调用 LLM（Synthesizer）对每个 Chunk 进行**实体与关系抽取**，并将结果合并写入图数据库，形成一张完整的知识图谱（KG）。

配置入口在 `config.yaml` 的 `build_kg` 字段：

```yaml
build_kg:
  max_loop: 3       # LLM 抽取的最大迭代次数，越大越完整但耗时更长，默认 3
```

---

## 1 整体流程

```
Chunks
  │
  ├─ text chunks  ──→  LightRAGKGBuilder  ──→ 实体 / 关系 ──→
  │                                                           ├─→ 合并去重 ──→ Graph Storage (KuzuDB)
  └─ multimodal chunks ─→ MMKGBuilder ────→ 实体 / 关系 ──→
```

1. **分流**：按 chunk 类型拆分为文本（`text`）与多模态（`image`/`video`/`table`/`formula`）两路。
2. **抽取**：并发调用 Synthesizer LLM 对每个 chunk 完成实体和关系抽取，可迭代至多 `max_loop` 轮以提升召回率。
3. **合并**：将同名实体/同边关系做去重合并，写入 KuzuDB 图数据库。

---

## 2 文本知识图谱（LightRAGKGBuilder）

对 `type=text` 的 chunk，GraphGen 使用基于 [LightRAG](https://github.com/HKUDS/LightRAG) 思路实现的 `LightRAGKGBuilder`：

- **实体抽取**：从文本中识别命名实体（人物、机构、地点、概念等），为每个实体生成唯一 ID 与描述。
- **关系抽取**：识别实体之间的关系，以三元组 `(head, tail, description)` 的形式存储。
- **多轮迭代**：每轮会提示 LLM 补充遗漏的实体/关系，最多迭代 `max_loop` 次。

---

## 3 多模态知识图谱（MMKGBuilder）

对 `type` 为 `image`、`video`、`table`、`formula` 的 chunk，使用 `MMKGBuilder`，流程与文本类似，但 prompt 针对多模态内容做了专门设计，支持从图片、表格、公式等非文本内容中抽取结构化知识。

---

## 4 图数据库

GraphGen 默认使用 [KuzuDB](https://kuzudb.com/) 作为图存储后端（`graph_backend: kuzu`），所有抽取的节点和边均持久化至本地。KuzuDB 是一个嵌入式高性能图数据库，无需额外部署服务。

存储路径在 `global_params.working_dir` 指定的目录下自动创建。

---

## 5 如何自定义 KG Builder

继承 `BaseKGBuilder`，实现 `extract`、`merge_nodes`、`merge_edges` 方法，然后在 `BuildKGService` 中替换即可。

---

## 6 常见问题

**Q1：抽取结果实体太少？**

增大 `max_loop`（推荐 3–5），或检查 Synthesizer 模型能力是否足够。

**Q2：运行速度慢？**

KG 构建是计算瓶颈，建议：
- 使用并发数高的推理服务（如 vLLM）
- 减小 `chunk_size` 以降低单次 LLM 调用的 token 量
- 利用 GraphGen 内置的缓存机制：已处理过的 chunk 不会重复抽取

**Q3：如何查看生成的知识图谱？**

GraphGen 内置了可视化工具：

```bash
python vis/visualize.py --working_dir cache
```
