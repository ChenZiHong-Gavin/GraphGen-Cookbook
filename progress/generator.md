# Step5 问答生成

社区划分完成后，GraphGen 将每个子图（社区）的实体和关系送入 Synthesizer LLM，生成结构化的**问答对（QA Pairs）**，输出为可直接用于训练或评测的数据集。

配置入口在 `config.yaml` 的 `generate` 字段：

```yaml
generate:
  mode: aggregated      # 生成模式，见下方说明
  data_format: ChatML   # 输出数据格式：Alpaca | ShareGPT | ChatML
```

---

## 1 生成模式

GraphGen 内置了多种问答生成模式，对应不同的数据需求：

| 模式 | 说明 | 典型用途 |
|------|------|----------|
| `atomic` | 针对单个实体或单条关系生成简单问答 | 基础知识问答 |
| `aggregated` | **两步法**：先将子图重新表述为连贯文本（answer），再从中生成问题（question）| 训练数据合成（**推荐**）|
| `multi_hop` | 生成需要跨多个实体/关系进行推理的多跳问答 | 复杂推理能力训练 |
| `cot` | 先生成推理路径模板，再据此生成含思维链的问答 | CoT 数据合成 |
| `vqa` | 针对多模态内容（图片/表格）生成视觉问答 | 多模态数据合成 |
| `multi_choice` | 生成多选题（含干扰项）| 评测数据合成 |
| `multi_answer` | 生成多个相关问题 | 多样化训练数据 |
| `fill_in_blank` | 生成填空题 | 知识记忆训练 |
| `masked_fill_in_blank` | 对文本中的关键词做掩码填空 | 知识记忆训练 |
| `true_false` | 生成判断题 | 评测数据合成 |

---

## 2 输出格式

通过 `data_format` 指定输出格式，支持主流训练框架：

### ChatML（默认，推荐）

```json
{
    "messages": [
        {"role": "user", "content": "问题内容"},
        {"role": "assistant", "content": "回答内容"}
    ]
}
```

### Alpaca

```json
{
    "instruction": "问题内容",
    "input": "",
    "output": "回答内容"
}
```

### ShareGPT

```json
{
    "conversations": [
        {"from": "human", "value": "问题内容"},
        {"from": "gpt", "value": "回答内容"}
    ]
}
```

---

## 3 生成示例

以下是 `aggregated` 模式生成的 ChatML 格式示例：

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
}
```

更多示例见：[resources/output\_examples](https://github.com/open-sciencelab/GraphGen/tree/main/resources/output_examples)

---

## 4 各模式使用建议

### 训练数据合成

- **通用 SFT 数据**：`aggregated` + `ChatML`
- **多跳推理数据**：`multi_hop` + `ChatML`
- **思维链数据**：`cot` + `ChatML`

### 评测数据合成

- **客观题**：`multi_choice` 或 `true_false`
- **主观题**：`aggregated` 或 `multi_hop`

---

## 5 常见问题

**Q1：生成的问题质量不高或重复率高？**

- 尝试更换更强的 Synthesizer 模型（如 GPT-4o）
- 调整 partition 参数，让子图更有信息量
- 开启 `quiz_and_judge` 让 ECE 优先处理高价值知识点

**Q2：如何控制生成问题的数量？**

问题数量由子图（社区）数量决定，间接由以下参数控制：
- `max_extra_edges` / `max_tokens`：越小，社区数量越多，问题越多
- 输入文档数量和内容丰富程度

**Q3：支持中文生成吗？**

支持。GraphGen 会自动检测子图内容的语言（中文/英文），并切换对应的 prompt 模板，生成对应语言的问答对。
