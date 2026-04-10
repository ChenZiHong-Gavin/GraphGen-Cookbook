# 附录

## A 支持列表速查

### 输入格式

| 格式 | 后缀 | 说明 |
|------|------|------|
| JSON | `.json` | 顶层数组，每个元素含 `content` 字段 |
| JSONL | `.jsonl` | 每行一条 JSON，含 `content` 字段 |
| CSV | `.csv` | 含 `content` 列 |
| TXT | `.txt` | 每行作为一条文档 |
| Markdown | `.md` | 同 TXT 处理 |
| PDF | `.pdf` | 自动解析文本内容 |
| Parquet | `.parquet` | 列式格式 |
| RDF/OWL/TTL | `.rdf` `.owl` `.ttl` | 知识图谱本体 |
| HuggingFace | `huggingface://` | 直接加载 HF 数据集 |

### 划分算法

| 算法 | method 值 | 说明 |
|------|-----------|------|
| ECE 划分 | `ece` | 基于理解损失，推荐用于训练数据 |
| BFS 划分 | `bfs` | 广度优先 |
| DFS 划分 | `dfs` | 深度优先 |
| Leiden | `leiden` | 模块度最优社区发现 |
| 三元组 | `triple` | 单条三元组为社区 |
| 五元组 | `quintuple` | 五元组为社区 |
| 锚点 BFS | `anchor_bfs` | 以指定节点为锚点 BFS 扩展 |

### 生成模式

| 模式 | mode 值 | 推荐用途 |
|------|---------|----------|
| 聚合问答 | `aggregated` | SFT 训练数据（通用推荐）|
| 原子问答 | `atomic` | 基础知识单点问答 |
| 多跳问答 | `multi_hop` | 推理能力训练/评测 |
| 思维链 | `cot` | CoT 数据合成 |
| 视觉问答 | `vqa` | 多模态数据 |
| 多选题 | `multi_choice` | 客观题评测 |
| 多答案 | `multi_answer` | 多样化训练 |
| 填空题 | `fill_in_blank` | 知识记忆训练 |
| 掩码填空 | `masked_fill_in_blank` | 知识记忆训练 |
| 判断题 | `true_false` | 客观题评测 |

### 搜索源

| 搜索源 | search_types 值 | 适用场景 |
|--------|----------------|----------|
| Google | `google` | 通用网络搜索 |
| Bing | `bing` | 通用网络搜索 |
| UniProt | `uniprot` | 蛋白质数据库 |
| NCBI | `ncbi` | 基因组学数据库 |
| RNAcentral | `rnacentral` | RNA 数据库 |
| InterPro | `interpro` | 蛋白质家族数据库 |

### 输出格式

| 格式 | data_format 值 | 兼容框架 |
|------|---------------|----------|
| ChatML | `ChatML` | 大多数主流框架 |
| Alpaca | `Alpaca` | Alpaca 系列、LLaMA-Factory |
| ShareGPT | `ShareGPT` | FastChat、LLaMA-Factory |

---

## B 常见错误排查

### `Ray` 初始化失败

```
RuntimeError: Failed to start Ray
```

检查系统内存是否充足，或尝试：

```python
import ray; ray.init(num_cpus=2)
```

### KuzuDB 锁冲突

```
Error: Cannot open database at path ... another process is using it
```

确保只有一个 GraphGen 进程访问同一 `working_dir`，或清理残留的 `.lock` 文件。

### LLM API 超时

```
openai.APITimeoutError
```

增大 API 请求超时时间，或降低并发数以减少速率限制触发。

### 中文实体乱码

确保输入文件使用 **UTF-8** 编码，并在环境中设置：

```bash
export PYTHONIOENCODING=utf-8
```

---

## C 相关链接

| 资源 | 地址 |
|------|------|
| 论文 | [https://arxiv.org/abs/2505.20416](https://arxiv.org/abs/2505.20416) |
| GitHub | [https://github.com/open-sciencelab/GraphGen](https://github.com/open-sciencelab/GraphGen) |
| 使用文档 | [https://chenzihong.gitbook.io/graphgen-cookbook/](https://chenzihong.gitbook.io/graphgen-cookbook/) |
| PyPI | [https://pypi.org/project/graphg/](https://pypi.org/project/graphg/) |
| HuggingFace Demo | [https://huggingface.co/spaces/chenzihong/GraphGen](https://huggingface.co/spaces/chenzihong/GraphGen) |
| ModelScope Demo | [https://modelscope.cn/studios/chenzihong/GraphGen](https://modelscope.cn/studios/chenzihong/GraphGen) |
| OpenXLab Demo | [https://g-app-center-000704-6802-aerppvq.openxlab.space](https://g-app-center-000704-6802-aerppvq.openxlab.space) |
| LLaMA-Factory | [https://github.com/hiyouga/LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory) |
| xtuner | [https://github.com/InternLM/xtuner](https://github.com/InternLM/xtuner) |

---

## D 引用

如果 GraphGen 对你的研究有帮助，请引用：

```bibtex
@article{graphgen2025,
  title   = {GraphGen: Enhancing Supervised Fine-Tuning for LLMs with Knowledge-Driven Synthetic Data Generation},
  year    = {2025},
  url     = {https://arxiv.org/abs/2505.20416}
}
```
