# 流程

GraphGen 的数据合成流程分为五个步骤，整体如下图所示：

```
输入文档
   │
   ▼
Step1  读取文件       ──→  解析 json/jsonl/txt/csv/pdf 等，输出 List[Doc]
   │
   ▼
Step2  分割文本       ──→  按语言自动选择分块器，输出 List[Chunk]
   │
   ▼
Step3  构建知识图谱   ──→  LLM 抽取实体/关系，写入 KuzuDB
   │
   ▼ (可选)
Step3.5 网络搜索     ──→  Google/Bing/UniProt/NCBI 等，扩充文档信息
   │
   ▼
Step4  社区划分      ──→  ECE/BFS/Leiden 等算法，将大图切分为子图
   │
   ▼
Step5  问答生成      ──→  LLM 对每个子图生成 QA，输出 ChatML/Alpaca/ShareGPT
```

---

## 两个 LLM 角色

GraphGen 在流程中使用两个不同的 LLM：

| 角色 | 环境变量 | 用途 |
|------|---------|------|
| **Synthesizer** | `SYNTHESIZER_MODEL` | KG 构建（实体/关系抽取）+ 问答对生成 |
| **Trainee** | `TRAINEE_MODEL` | Quiz & Judge（评估模型对知识点的掌握程度）|

两个角色可以是同一个模型，也可以是不同的模型。Trainee 通常是你**实际要训练的目标模型**。

---

## 步骤导航

<table data-view="cards">
  <thead><tr><th></th><th></th></tr></thead>
  <tbody>
    <tr><td><strong>Step1 读取文件</strong></td><td>支持 json/jsonl/txt/csv/pdf/parquet 等多种格式</td></tr>
    <tr><td><strong>Step2 分割文本</strong></td><td>中英文自动识别，递归分块，支持自定义分块器</td></tr>
    <tr><td><strong>Step3 构建知识图谱</strong></td><td>LLM 驱动实体/关系抽取，KuzuDB 存储</td></tr>
    <tr><td><strong>Step3.5 网络搜索</strong></td><td>可选，通过网络/专业数据库扩充知识</td></tr>
    <tr><td><strong>Step4 社区划分</strong></td><td>ECE 算法优先覆盖知识盲区，多种划分策略可选</td></tr>
    <tr><td><strong>Step5 问答生成</strong></td><td>10 种生成模式，支持 SFT/CoT/多选/判断等多种格式</td></tr>
  </tbody>
</table>
