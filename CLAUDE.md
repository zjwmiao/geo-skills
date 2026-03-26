# CLAUDE.md — SEO/GEO 分析工具

## 项目概述

通用 SEO/GEO（Generative Engine Optimization）分析工具集，适用于任意网站。提供：

- **crawl.js** — 本地页面抓取，输出 HTML / 纯文本
- **analyze.js** — 本地 SEO 元素快速诊断（canonical、OG、Schema、H1、alt 等）
- Claude 技能集成（`/audit-page`、`/geo-content-optimizer`、`/keyword-research` 等）

当前已分析站点记录在 `reports/` 各子目录或文件注释中。

---

## 目录结构

```
seo-geo/
├── scripts/
│   ├── crawl.js          # 页面抓取脚本
│   └── analyze.js        # SEO 元素本地快速诊断脚本
├── reports/              # 所有分析报告（.md）
├── crawled/              # 抓取结果（.html / .txt）
├── package.json
└── pnpm-lock.yaml
```

报告输出到 `reports/`，抓取结果输出到 `crawled/`。

---

## 页面抓取：crawl.js

### 基本用法

```bash
node scripts/crawl.js <URL> [选项]
```

### 选项

| 选项 | 说明 |
|---|---|
| `--mode=http` | 纯 HTTP 抓取（默认，适用于非纯 SPA 站点） |
| `--mode=browser` | 无头浏览器抓取，需系统已安装 Chrome，适用于 JS 渲染页面 |
| `--out=<路径>` | 指定输出文件名（不含扩展名） |
| `--format=html` | 只保存 HTML（默认） |
| `--format=text` | 只保存纯文本 |
| `--format=both` | 同时保存 HTML 和纯文本 |

### 常用示例

```bash
# 抓取并保存 HTML + 纯文本
node scripts/crawl.js https://example.com/ --out=example-home --format=both

# JS 渲染站点（React/Vue SPA）使用 browser 模式
node scripts/crawl.js https://example.com/ --mode=browser --format=both

# 抓取本地开发服务器（必须用 browser 模式，本地开发服务器为客户端渲染）
node scripts/crawl.js http://localhost:5173/some-page/ --mode=browser --out=local-page --format=both
```

### 输出说明

抓取完成后终端会打印页面基本信息（Title、Meta Description、Canonical、H1），可快速判断 SEO 基础状态。

### 分析时用哪个格式

| 分析目的 | 传给 Claude 的格式 |
|---|---|
| SEO 审计（title/meta/h1/schema/alt）| **HTML** |
| GEO 内容质量分析（可引用性、FAQ、结构）| **纯文本** 即可 |
| 完整 SEO + GEO 联合审计 | **HTML**（包含所有信息）|

---

## SEO 元素本地诊断：analyze.js

读取 HTML 文件，在本地秒级输出 SEO 诊断摘要，无需将 HTML 上传给 Claude。

### 基本用法

```bash
node scripts/analyze.js <html文件路径>
```

### 检测项目

| 类别 | 检测内容 |
|---|---|
| 基础 Meta | Title 长度、Description 长度、Keywords、Robots |
| 技术标签 | Canonical、Hreflang（含各 lang 值） |
| 社交分享 | og:title / og:description / og:image / og:type |
| Twitter/X | twitter:card、twitter:title |
| 结构化数据 | JSON-LD 数量及 @type |
| 标题层次 | H1（数量+内容）、H2、H3 列表 |
| 图片 | 总计、有效 alt、无 alt、空 alt 各自数量及覆盖率 |
| 链接 | 内部链接数、外部链接数及示例 |

### 常用示例

```bash
# 先抓取 HTML，再本地分析
node scripts/crawl.js https://example.com/ --out=example-home --format=html
node scripts/analyze.js crawled/example-home.html
```

### 与 /audit-page 的分工

| 工具 | 速度 | 用途 |
|---|---|---|
| `analyze.js` | 秒级，本地离线 | 快速确认 Critical 缺失项，定位问题 |
| `/audit-page` | 需 Claude | 完整 SEO 评分 + CORE-EEAT 80项内容质量审计 |

建议先跑 `analyze.js` 确认问题范围，再把 HTML 提供给 Claude 做 `/audit-page` 深度分析。

---

## 分析工作流

### 单页 SEO + GEO 审计

1. 用 `crawl.js` 抓取目标页面到 `crawled/`（`--format=html`）
2. 用 `analyze.js` 本地快速诊断，确认 Critical 缺失项
3. 将 HTML 文件内容提供给 Claude，运行 `/audit-page`
4. 报告保存到 `reports/`

### 新增页面优化流程（适用于 VitePress / 静态站）

1. `/keyword-research` — 确定目标关键词
2. `/write-content` — 生成 SEO+GEO 优化的 Markdown 正文
3. `/optimize-meta` — 生成 Frontmatter（title、description、OG 标签）
4. `/generate-schema` — 生成 JSON-LD（HowTo / FAQPage / TechArticle）
5. 本地开发服务器启动后，用 `crawl.js` 抓取 localhost 做上线前审计（**必须加 `--mode=browser`**，本地开发服务器为客户端渲染）

### GEO 优化核心原则

AI 引擎引用内容需满足：
- **首段 150 字内**有直接定义句（含具体数字 + 来源）
- **FAQ 区块**覆盖用户会向 AI 提问的 5–8 个问题，并配套 FAQPage JSON-LD
- **数据有出处**：每个统计数字需注明来源机构和日期
- **表格优于段落**：对比数据、硬件规格、功能列表一律用表格

---

## Token 使用建议

- 整站全量抓取不现实，采用**按页面类型采样**策略（每类 1–2 个代表页）
- 只需了解站点结构时，先读 sitemap.xml（约 1 万 token），再按需抓取具体页面
- GEO 内容分析传纯文本（.txt），节省 token；SEO 元素审计传 HTML

---

## 包管理

使用 **pnpm**。安装依赖：

```bash
pnpm install
```

当前依赖：`axios`、`jsdom`、`playwright-core`
