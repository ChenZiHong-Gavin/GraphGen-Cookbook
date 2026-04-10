# 数据准备

在使用 GraphGen 之前，你需要准备好原始语料文件。本文介绍支持的输入格式，以及如何处理不同来源的数据。

---

## 1 支持的输入格式

GraphGen 的 `read` 模块支持以下格式，通过文件后缀自动识别：

| 格式 | 后缀 | 说明 |
|------|------|------|
| JSON | `.json` | 顶层为数组，每个元素包含 `content` 字段 |
| JSONL | `.jsonl` | 每行一条 JSON，包含 `content` 字段 |
| CSV | `.csv` | 必须含 `content` 列，其余列作为 metadata |
| TXT | `.txt` | 每行非空文本作为一条独立文档 |
| Markdown | `.md` | 与 TXT 相同方式读取 |
| PDF | `.pdf` | 自动解析 PDF 文本内容 |
| Parquet | `.parquet` | 列式存储格式，适合大规模数据集 |
| RDF/OWL/TTL | `.rdf` `.owl` `.ttl` | 知识图谱本体文件 |
| HuggingFace | `huggingface://` URI | 直接加载 HuggingFace 数据集 |

---

## 2 格式要求与示例

### 2.1 JSONL（推荐）

每行一条合法 JSON，必须包含 `content` 字段；空行会被忽略；其余字段作为 metadata 保留。

```jsonl
{"content": "北极燕鸥每年往返南北极，迁徙距离长达 7 万公里。", "source": "wiki"}
{"content": "章鱼有三颗心脏，血液含铜离子故呈蓝色。", "source": "wiki"}
```

### 2.2 JSON

顶层为数组，每个元素必须包含 `content` 字段。

```json
[
  {"content": "NGC 4414 是一个位于后发座的螺旋星系……"},
  {"content": "M87 是室女座星系团中的超大椭圆星系……"}
]
```

### 2.3 CSV

必须含有 `content` 列，其余列自动作为 metadata 传递。

```csv
id,title,content
1,北极燕鸥,"北极燕鸥每年往返南北极, 迁徙距离长达 7 万公里。"
2,章鱼血液,"章鱼有三颗心脏, 血液含铜离子故呈蓝色。"
```

### 2.4 TXT

每行非空文本作为一条独立文档，适合结构简单的纯文本语料。

```
北极燕鸥每年往返南北极，迁徙距离长达 7 万公里。
章鱼有三颗心脏，血液含铜离子故呈蓝色。
```

### 2.5 HuggingFace 数据集

在配置文件中使用 `huggingface://` URI，格式为 `dataset_name:subset:split`：

```yaml
read:
  input_file: "huggingface://allenai/c4:en:train"
```

---

## 3 多文件 / 目录输入

`input_file` 支持传入目录路径或通配符，GraphGen 会自动递归扫描目录下所有支持格式的文件：

```yaml
read:
  input_file: /path/to/corpus_dir    # 自动扫描目录下的 json/jsonl/txt/pdf 等
  allowed_suffix: ["jsonl", "txt"]   # 可选：限制只读取特定后缀
  recursive: true                    # 是否递归扫描子目录，默认 true
```

---

## 4 数据清洗建议

为获得更高质量的知识图谱和问答对，建议在输入 GraphGen 之前做好以下预处理：

1. **去重**：移除完全重复的段落，避免 KG 中出现冗余节点。
2. **过滤低质量文本**：过滤过短（< 50 字）或乱码的文本行。
3. **领域聚焦**：如果只需要特定领域数据，建议先做领域过滤，减少噪声。
4. **编码统一**：确保文件使用 UTF-8 编码保存。

---

## 5 输入示例文件

GraphGen 仓库中提供了多种格式的输入示例，可作为参考：

```
graphgen/resources/input_examples/
├── json_demo.json
├── jsonl_demo.jsonl
├── csv_demo.csv
└── txt_demo.txt
```

---

## 6 常见问题

**Q1：PDF 读取乱码？**

PDF 读取依赖 `pdfminer` / `pymupdf`，确保已安装。对于扫描版 PDF，需要先做 OCR 处理转为文本。

**Q2：大文件读取很慢？**

GraphGen 底层使用 Ray 进行并行读取，`parallelism` 参数可控制并发度：

```yaml
read:
  input_file: /path/to/large_dir
  parallelism: 8    # 并发读取线程数
```

**Q3：HuggingFace 数据集下载失败？**

检查网络连接，或配置 HuggingFace 镜像：

```bash
export HF_ENDPOINT=https://hf-mirror.com
```
