# Step1 文件读取

GraphGen 会在启动时根据输入的 input\_file 后缀**自动挑选读取器，**&#x628A;文件内容转成 `List[Dict[str, Any]]`，方便后续的处理。目前支持四种格式：`json`、`jsonl`、`csv`、`txt`。

在配置文件中的相关设置：

```yaml
read:
  input_file: your_corpus_path.jsonl # input file path, support json, jsonl, txt. See resources/input_examples for examples
```

### 1 格式要求与示例

#### 1.1 json

顶层是 List，每个元素为 Dict，且包含 `content` 字段。

```json
[
  {"content": "NGC 4414 是一个位于后发座的螺旋星系……"},
  {"content": "M87 是室女座星系团中的超大椭圆星系……"}
]
```

#### 1.2 jsonl

每行一条合法 JSON，且包含 `content` 指定字段；空行会被忽略。

```json
{"content": "北极燕鸥每年往返南北极，迁徙距离长达 7 万公里。"}
{"content": "章鱼有三颗心脏，血液含铜离子故呈蓝色。"}
```

#### 1.3 csv

必须含有与 `content` 同名的列，其余列任意；自动采用 UTF-8 读取。

```csv
id,title,content
1,北极燕鸥,"北极燕鸥每年往返南北极, 迁徙距离长达 7 万公里。"
2,章鱼血液,"章鱼有三颗心脏, 血液含铜离子故呈蓝色。"
```

#### 1.4 txt

每行非空文本会被当成一条独立文档。

```
北极燕鸥每年往返南北极，迁徙距离长达 7 万公里。
章鱼有三颗心脏，血液含铜离子故呈蓝色。
```

