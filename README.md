# 使用

安装依赖： `pnpm install`

安装skills：`npx skills install`

[skills来源](https://github.com/aaron-he-zhu/seo-geo-claude-skills)

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

## GEO/SEO分析

对某页面做SEO分析，使用 `audit-page` command: `/audit-page <url>`

直接说明意图即可，例如：

GEO审计/优化建议：

- 帮我对这个页面做GEO分析: [content or URL]

- Make this article more likely to be cited by AI systems

> 使用到的skill: geo-content-optimizer

> **分析某个 URL 的 GEO 时，是只分析这一个页面还是整站？**
>
> 只分析目标页面，但会借用项目级的公开信息（社区数据、市场数据等）作为优化素材。如果直接抓取被拦截，可以先用本地的 `crawl.js` 抓取页面内容后粘贴给我，避免因看不到真实内容而产生误判（例如：误以为 H1 由 JS 渲染缺失，实际上是页面根本没有 H1 元素）。
> 
> **如何对整站做 GEO 分析？**
> 
> - "帮我对 openeuler.org 整站做 GEO 分析"
> - "帮我找出 openeuler.org 哪些页面最适合做 GEO 优化，优先级怎么排"
> - "我希望 AI 回答'国产 Linux 推荐'时能引用到该站，整站应该怎么优化"
> 
> 提供 Google Search Console 数据或 Screaming Frog 爬取结果会让分析更准确。

## 新增markdown内容的 SEO/GEO 标准工作流

VitePress 用 Markdown + Frontmatter 构建静态页面，SEO/GEO 的优化点主要落在三个位置：Frontmatter、Markdown 正文内容、Schema 注入。推荐按以下流程操作：

### 第一步：关键词研究

告诉我页面主题，我来做关键词分析：
> "帮我为 openeuler.org 新增一篇关于'openEuler 迁移 CentOS'的页面做关键词研究"

使用 `/keyword-research`，输出目标关键词、搜索量、竞争度、语义相关词。

---

### 第二步：生成 SEO+GEO 优化的正文内容

> "用上面的关键词，帮我写这个页面的 Markdown 正文，要同时针对 SEO 和 GEO 优化"

使用 `/write-content`，输出结构包含：
- 首段 150 字内的直接定义句（GEO 核心）
- 核心亮点摘要框
- 正文分 H2/H3 章节
- 可引用的带来源数据陈述
- FAQ 区块（6–8 个问答）

---

### 第三步：优化 Frontmatter meta 标签

> "帮我写这个页面的 VitePress Frontmatter，包含 title、description 和 Open Graph 标签"

使用 `/optimize-meta`，输出：

```yaml
---
title: 从 CentOS 迁移到 openEuler 完整指南 | openEuler
description: openEuler 提供官方迁移工具 x2openEuler，支持从 CentOS 7/8 一键平滑迁移，保留数据和配置。本指南覆盖迁移步骤、兼容性检查和常见问题。
head:
  - - meta
    - name: og:title
      content: 从 CentOS 迁移到 openEuler 完整指南
  - - meta
    - name: og:description
      content: 使用 x2openEuler 工具从 CentOS 7/8 迁移到 openEuler，官方支持，数据零丢失。
  - - meta
    - name: og:type
      content: article
---
```

---

### 第四步：生成 JSON-LD Schema

> "帮我为这个迁移指南页面生成 JSON-LD 结构化数据"

使用 `/generate-schema`，根据页面类型输出对应 Schema：

| 页面类型 | 推荐 Schema |
|---|---|
| 教程 / 指南 | HowTo + FAQPage |
| 博客文章 | Article + FAQPage |
| 下载页 | SoftwareApplication |
| 社区/关于页 | Organization |
| 文档页 | TechArticle |

在 VitePress 中注入 Schema 的方式：

```ts
// .vitepress/config.ts
export default {
  head: [
    ['script', { type: 'application/ld+json' },
      JSON.stringify({
        "@context": "https://schema.org",
        "@type": "HowTo",
        "name": "从 CentOS 迁移到 openEuler",
        // ...
      })
    ]
  ]
}
```

或者在页面级 Frontmatter 的 `head` 字段里单独注入（适合页面各不相同的 Schema）。

---

### 第五步：上线前做页面级审计

在本地 `vitepress dev` 跑起来后，用 `audit-page`

```
/audit-page http://localhost:5173/zh/xxx/ keyword='CentOS 迁移 openEuler'
```

---

## 新增 Vue SFC 页面的 SEO/GEO 工作流

VitePress 支持用 Vue SFC（`.vue` 文件）代替 Markdown 来编写页面。此时 Frontmatter 不再适用，meta 标签、H1、Schema 均需手动处理。

---

### 与 Markdown 页面的关键差异

| 方面 | Markdown 页面 | Vue SFC 页面 |
|---|---|---|
| Meta 标签 | Frontmatter `title` / `description` / `head:` | `useHead()` in `<script setup>` |
| H1 | `# 标题` 自动生成 | 模板里手写 `<h1>`，**不写就缺失** |
| JSON-LD | Frontmatter `head:` 或 `config.ts` | `useHead({ script: [...] })` 或模板内 `<script type="ld+json">` |
| Canonical | `config.ts` 全局或 Frontmatter | `useHead({ link: [{ rel: 'canonical', href: '...' }] })` |
| 图片 alt | Markdown 语法提醒填写 | 模板里 `<img>` 必须手写 `alt`，**不写就缺失** |

### 在组件里注入 meta 和 Schema

---

VitePress 内置 `@unhead/vue`，在 `<script setup>` 中直接调用：

```vue
<script setup>
import { useHead } from '@unhead/vue'

useHead({
  title: '页面标题 | 站点名',
  meta: [
    { name: 'description', content: '150字以内，含核心关键词和价值主张' },
    { property: 'og:title', content: '页面标题' },
    { property: 'og:description', content: 'OG 描述' },
    { property: 'og:type', content: 'website' },
    { property: 'og:image', content: 'https://example.com/og-cover.jpg' },
    { name: 'twitter:card', content: 'summary_large_image' },
  ],
  link: [
    { rel: 'canonical', href: 'https://example.com/your-page/' }
  ],
  script: [
    {
      type: 'application/ld+json',
      children: JSON.stringify({
        '@context': 'https://schema.org',
        '@type': 'WebPage',
        // ...
      })
    }
  ]
})
</script>
```

---

### 保证模板里的语义结构

```vue
<template>
  <main>
    <!-- H1 必须显式写，不会自动生成 -->
    <h1>页面主标题</h1>

    <section>
      <h2>章节标题</h2>
      <p>正文段落...</p>
    </section>

    <!-- FAQ 区块（GEO 核心，配合 FAQPage JSON-LD） -->
    <section>
      <h2>常见问题</h2>
      <dl>
        <dt>问题一？</dt>
        <dd>答案...</dd>
      </dl>
    </section>

    <!-- 图片必须写 alt -->
    <img src="/hero.png" alt="描述性文字，说明图片内容">
  </main>
</template>
```

---

### 上线前审计

在本地 `vitepress dev` 跑起来后，用 `audit-page`

```
/audit-page http://localhost:5173/zh/xxx/ keyword='CentOS 迁移 openEuler'
```

或者询问：“帮我对这个页面做GEO分析: http://localhost:5173/zh/xxx/”

---

## GEO 报告生成过程解析

### 第一步：信息获取

报告生成的第一步是**抓取目标页面的内容**。我尝试直接 fetch `openeuler.org/zh/`，但被网络策略拦截了，所以退而求其次，用两次 Web 搜索来获取数据：

1. 搜索中文首页的内容和介绍
2. 搜索英文的社区数据、技术特性、市场数据

这两次搜索返回了关键素材：标题、meta、社区用户数、架构列表、市占率、PR 数量等——这些就是后续优化的原材料。

> 已有`scripts/crawl.js`，可用于抓取网页到本地文件

---

### 第二步：GEO 评分基线（现状诊断）

接着对当前页面做**8 个维度的 GEO 现状评分**：

| 维度 | 评估逻辑 |
|---|---|
| 定义清晰度 | 首屏是否有独立可引用的定义句？ |
| 可引用陈述 | 是否有带数据+来源的独立句子？ |
| 事实密度 | 数据点是否带单位、日期、方法论？ |
| 来源引用 | 每个主张是否有可验证的出处？ |
| Q&A 格式 | 是否有结构化的问答内容？ |
| 权威信号 | 是否有机构背书、专家引用？ |
| 内容新鲜度 | 是否有明确的时间戳？ |
| 结构清晰度 | AI 爬虫能否解析页面语义结构？ |

原始得分 **4.1/10**，核心问题是：三个零分项（来源引用几乎为零、无 FAQ、无数据出处）直接拖垮了整体 AI 可引用性。

---

### 第三步：对照 CORE-EEAT 框架定位优化点

这是报告的**理论骨架**。CORE-EEAT 是一个 80 项评估基准，把内容质量拆分成 8 个维度：

```
C   — Contextual Clarity（语境清晰度）
O   — Organization（内容组织）
R   — Referenceability（可引用性）     ← GEO 最关键
E   — Exclusivity（独特性）
Exp — Experience（经验展示）
Ept — Expertise（专业深度）
A   — Authority（权威性）
T   — Trust（可信度）
```

GEO 优化主要攻的是 **R 维度**（可引用性）和 **C/O 维度**（结构让 AI 能"读懂"）。报告里每个优化项旁边标注的 `C02`、`O03`、`R01` 就是对应的 CORE-EEAT 条目编号，这样优化有据可查、有优先级可排。

---

### 第四步：六类具体优化技术

对应上面的诊断，生成了六类具体的优化产物：

**① 定义块（Definition Block）**
原则：25–50 字，以术语开头，包含分类 + 可验证数据 + 日期 + 来源，能独立存在脱离上下文被引用。这是 Google AI Overview 和 Perplexity 最常抽取的格式。

**② 核心亮点摘要框（Summary Box）**
AI 系统在生成摘要时会优先提取列表和框格内容，因为它们结构化、信息密度高。这就是为什么要在首屏加一个带数字的摘要框。

**③ 竞品对比表（Comparison Table）**
用户问"openEuler 和 CentOS 区别"时，AI 系统更倾向于引用有表格的内容——表格是最"机器可读"的格式，直接映射到结构化数据。

**④ 可引用陈述（Quotable Statements）**
每条陈述遵循固定模式：`具体数字 + 单位 + 时间点 + 来源机构`。AI 引用时需要的正是这种可验证链条，模糊说法（"很多用户"）会被忽略。

**⑤ FAQ 区块 + JSON-LD Schema**
FAQ 直接对应搜索引擎的"People Also Ask"和 AI 的 follow-up query。JSON-LD FAQPage Schema 让 Google 和 AI 爬虫不用猜——直接从结构化数据里读问题和答案。

**⑥ 组织/产品/网站 Schema**
Organization + SoftwareApplication + WebSite 三段 JSON-LD，让 AI 系统知道"这是谁、做什么的、在哪下载、有什么关联实体"——这是知识图谱 (Knowledge Graph) 触发和 AI Overview 引用的基础设施。

---

### 第五步：AI 引擎偏好差异

报告隐含的一个逻辑是，不同 AI 引擎有不同的引用偏好：

| 引擎 | 最优先的信号 |
|---|---|
| Google AI Overview | 结构化内容（FAQ schema、表格、首段定义）|
| ChatGPT Browse | 带来源的具体数据点 |
| Perplexity | 独家数据、第三方引用 |
| Claude | 有推理链的内容、方法论透明 |

所以报告里加了"来源"标注（满足 ChatGPT/Perplexity）、加了 Schema（满足 Google）、加了"基于 Linux 6.6 内核"这类技术推理（满足 Claude 类引擎）。

---

### 核心逻辑一句话总结

> **GEO 的本质是：把人类读者能理解的营销语言，改写成 AI 系统能抽取、验证、并有信心引用的结构化证据链。**

SEO 是让搜索引擎**找到**你，GEO 是让 AI 引擎**引用**你——后者要求更高的事实密度、更强的来源链、更清晰的语义结构。

---
