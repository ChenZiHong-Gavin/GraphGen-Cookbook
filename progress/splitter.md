# Step2 分割文本

GraphGen 在读取完原始文档后，会立即对每条文档的 content 字段进行文本分块。分块结果将作为后续提取知识图谱时，输入给 LLM 的最小语义单元。

配置入口 在 config.yaml 中统一放在 split 一级：

```yaml
split:
  chunk_size: 1024 # 分割的 chunk 大小
  chunk_overlap: 100 # chunk 重叠的大小
```

### 1 分块策略

GraphGen 实现了`CharacterSplitter` 、`MarkdownTextRefSplitter` 、`RecursiveCharacterSplitter` 和`ChineseRecursiveTextSplitter` 。默认情况下，会对每条文档进行语种检测，目前支持 en、zh：

* en → 采用 RecursiveCharacterSplitter（按段落/句子/空格递归回落）
* zh → 采用 ChineseRecursiveTextSplitter（按段落/句子/标点/字符递归回落）



### 2 常见问题

#### Q1 切完块大于 chunk\_size？

检查 use\_tokenizer 与单位是否一致；或调小 chunk\_size。

#### Q2 中英文混排如何切？

语种检测以「主要语种」为准；若需更细粒度，可提前把文档按语种拆条后分别调用。

#### Q3 自定义切分器？

继承 `BaseSplitter` 类，重写 split\_text 方法，然后在 `graphgen/operators/split/split_chunks.py`   中的 `_MAPPING` 中注册即可。
