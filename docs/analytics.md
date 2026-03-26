# GEO分析

“检索 → 评估 → 生成”的流程

- 可检索性：内容能否被 AI 爬虫轻松发现和抓取。
- 可信度：品牌和内容是否具备足够的权威性，让 AI 放心引用。
- 易读性：内容是否结构清晰，便于 AI 理解和提取关键信息。

## 改进项

### 开发

1. 静态化页面；--- 可检索
2. 增加Schema；--- 可检索
3. 完善TDK、Sitemap；--- 可检索
4. 语义化页面标签；--- 易读性
5. 过时页面、停维文档做标记；--- 可信度

### 内容

1. 补充重点页面 --- 可检索性
2. 结论前置 --- 可信度
3. 补充带逻辑性文本或FAQ -- 可信度
4. 提供数据表格 --- 易读性
5. 增加重点页面内链密度 --- 可信度

### 流量

### 其他

1. 页面锚点提供锚点链接

## 问题分析

| 问题                                                                                                                                      | 主要角色         | 状态   |
| ----------------------------------------------------------------------------------------------------------------------------------------- | ---------------- | ------ |
| [缺少官方版本发布策略文档，AI平台对MindSpore版本发布节奏的描述失真](https://atomgit.com/mindspore/docs/issues/3654)                       | 运营             | 未启动 |
| [官网活动页面和贡献指南页面采用SPA渲染，AI搜索引擎无法索引内容](https://atomgit.com/mindspore/docs/issues/3655)                           | 开发             | 已完成 |
| [官网FAQ中关于"MindSpore不支持直接读取其他框架模型"的表述力度不足](https://atomgit.com/mindspore/docs/issues/3656)                        | 运营             | 未启动 |
| [FAQ内容过于简短且存在术语歧义，导致AI平台完全误解或无法回答](https://atomgit.com/mindspore/docs/issues/3657)                             | 运营             | 未启动 |
| [版本发布信息分散，AI平台对新版本特性描述出现大规模编造](https://atomgit.com/mindspore/docs/issues/3658)                                  | 运营 + UX        | 未启动 |
| [各AI平台MindSpore主仓库位置说法不一致，贡献指南中CLA签署入口不统一](https://atomgit.com/mindspore/docs/issues/3659)                      | 运营 + UX        | 未启动 |
| [官网FAQ中PyNative模式和Graph模式的对比说明不充分](https://atomgit.com/mindspore/docs/issues/3660)                                        | 运营 + 文档      | 未启动 |
| [FAQ页面缺乏结构化标记和锚点ID，AI平台引用精准度低且版本混乱](https://atomgit.com/mindspore/docs/issues/3661)                             | 运营 + 文档      | 未启动 |
| [提升 SIG 专项页面可发现性：mindspore.cn/sig/\* 未被AI平台引用，跨平台例会信息严重不一致](https://atomgit.com/mindspore/docs/issues/3662) | 开发 + UX        | 整改中 |
| [提高mindspore.cn/activities 可发现性](https://atomgit.com/mindspore/docs/issues/3663)                                                    | 开发 + UX + 运营 | 整改中 |
