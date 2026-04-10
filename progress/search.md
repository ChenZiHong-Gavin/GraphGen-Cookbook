# Step3.5 网络搜索

GraphGen 支持在构建知识图谱之前，通过网络搜索或专业数据库查询来**扩充原始文档的信息**，从而让 KG 更加丰富、权威。

该步骤为**可选步骤**，默认关闭。

配置入口在 `config.yaml` 的 `search` 字段：

```yaml
search:
  enabled: false                   # 是否启用搜索，默认 false
  search_types: ["google"]         # 搜索源，可多选，见下方支持列表
```

---

## 1 支持的搜索源

| 搜索源 | 说明 | 适用场景 |
|--------|------|----------|
| `google` | Google 网络搜索 | 通用知识扩充 |
| `bing` | Bing 网络搜索 | 通用知识扩充 |
| `uniprot` | UniProt 蛋白质数据库 | 生物/蛋白质组学 |
| `ncbi` | NCBI 基因数据库 | 基因组学、生物医学 |
| `rnacentral` | RNAcentral RNA 数据库 | RNA 研究 |
| `interpro` | InterPro 蛋白质家族数据库 | 蛋白质功能注释 |

---

## 2 网络搜索（Google / Bing）

使用 Google 或 Bing 时，GraphGen 会以每条 chunk 的 `content` 字段作为查询词，获取相关网页摘要并追加到原始文档中。

**环境变量配置：**

```bash
# Google Search
GOOGLE_API_KEY=your_google_api_key
GOOGLE_CSE_ID=your_custom_search_engine_id

# Bing Search
BING_SUBSCRIPTION_KEY=your_bing_subscription_key
```

---

## 3 专业数据库搜索

对于生物信息学场景，GraphGen 支持直接查询专业数据库。以 UniProt 为例：

```yaml
search:
  enabled: true
  search_types: ["uniprot"]
  uniprot_params:
    max_results: 5      # 每次查询返回的最大条数
```

其余数据库同理，参数名分别为 `ncbi_params`、`rnacentral_params`、`interpro_params`。

---

## 4 搜索缓存

搜索结果会自动缓存在 `working_dir/search/` 目录下（KV 存储）。相同查询词不会重复发起网络请求，节省 API 用量。

---

## 5 常见问题

**Q1：Google 搜索报错 `403` 或 `quota exceeded`？**

检查 `GOOGLE_API_KEY` 和 `GOOGLE_CSE_ID` 是否正确配置，或是否超出免费配额（每天 100 次）。

**Q2：搜索到的内容质量不高？**

建议改用专业数据库（如 UniProt/NCBI）替代通用搜索，或在数据预处理阶段过滤低质量文档。

**Q3：搜索会增加多少耗时？**

取决于网络环境和数据库响应速度，通常在 chunk 数量较多时建议并发数控制在合理范围内。GraphGen 内置并发搜索，会自动管理并发量。
