---
date: 2026-03-28
tags: [openclaw, xiaohongshu, ai-automation, content-creation]
category: tech-article
cover_image: images/00-cover-xiaohongshu-automation.png
---

![封面：小红书自动化流水线](00-cover-xiaohongshu-automation.png)

# 我用 OpenClaw 把小红书运营做成了自动化流水线

> 每天 30 分钟，产出 10 篇笔记，我是怎么做到的。

---

## 01 为什么要做小红书自动化

先说结论：**小红书可能是 2026 年普通人最容易冷启动的流量平台。**

但做过小红书的人都知道，这玩意儿有几个痛点：

1. **需要持续输出** — 三天打鱼两天晒网，账号基本就废了
2. **选题容易枯竭** — 写到第 7 天你就不知道写什么了
3. **排版耗时** — 找图、排版、加标签，一篇笔记 2 小时
4. **数据反馈慢** — 发了 10 篇，阅读量不过百，心态崩了

我试过自己写，坚持了一周，每天花 3 小时，产出 2 篇，阅读量稳定在 50 左右。

然后我开始想：**能不能用 AI 把这个流程自动化？**

答案是：可以。而且效果比我想象的好。

---

## 02 核心思路：把小红书运营拆成流水线

小红书运营看似复杂，其实可以拆成 6 个步骤：

```
选题 → 搜集素材 → 写文案 → 配图 → 排版 → 发布 → 数据分析
```

传统做法：每一步都人工做，一篇笔记 2-3 小时。

AI 做法：**除了最后一步发布和选题确认，其他全部自动化。**

![流水线对比](01-workflow-comparison.png)

---

## 03 工具选型：为什么是 OpenClaw

我调研了一圈，最后选了 OpenClaw，原因很简单：

| 工具 | 价格 | 自动化程度 | 学习成本 |
|------|------|-----------|---------|
| 手动写 | ¥0 | 0% | 低 |
| 某 AI 写作助手 | ¥199/月 | 30% | 中 |
| 某小红书生成器 | ¥399/月 | 50% | 中 |
| **OpenClaw** | ¥0 | **90%** | 中高 |

OpenClaw 是一个开源的 AI Agent 编排工具，核心能力是：

- 可以定义多个 Agent 分工协作
- 支持读取外部数据源（网页、API、文件）
- 输出结构化内容（Markdown、JSON）
- 完全免费，自己部署

**一句话：它能让多个 AI 代理像流水线工人一样协同工作。**

---

## 04 实战：我的 OpenClaw 小红书流水线

### 整体架构

```
                    ┌─────────────────┐
                    │   选题输入源     │
                    │ (热搜/竞品/关键词)│
                    └────────┬────────┘
                             │
                             ▼
┌────────────────────────────────────────────────┐
│              OpenClaw 编排层                      │
│                                                  │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐    │
│  │ 素材 Agent│ → │ 文案 Agent│ → │ 配图 Agent│    │
│  │ (爬取整理)│   │ (写稿改写)│   │ (生成封面)│    │
│  └──────────┘   └──────────┘   └──────────┘    │
│                                                  │
│                    ┌──────────┐                  │
│                    │ 排版 Agent│                  │
│                    │ (加标签)  │                  │
│                    └──────────┘                  │
└────────────────────────────────────────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   输出待发布     │
                    │ (Markdown+图片)  │
                    └─────────────────┘
```

---

### 第一步：选题输入

选题我用了三个来源：

1. **小红书热搜榜** — 每天更新的热门话题
2. **竞品账号** — 对标账号的爆款笔记
3. **关键词监控** — 我定义的垂直领域关键词

OpenClaw 配置示例：

```yaml
# 热搜监控 Agent
- name: hot-search-scraper
  type: scheduler
  cron: "0 9 * * *"  # 每天早上 9 点
  action:
    - fetch: "https://www.xiaohongshu.com/hot"
    - extract: "top-50-topics"
    - save: "topics/{{date}}.md"
```

---

### 第二步：素材搜集

确定选题后，让 AI 去搜集素材：

```yaml
# 素材搜集 Agent
- name: research-agent
  trigger: new-topic
  actions:
    - search-web: "{{topic}}"
    - fetch-top-10: "search-results"
    - extract-key-points: true
    - save-draft: "research/{{topic}}.md"
```

这个 Agent 会：
- 在 Google/百度搜索相关话题
- 抓取前 10 个结果的核心观点
- 提取关键信息和数据
- 保存成草稿供文案 Agent 使用

---

### 第三步：文案生成

这是核心环节。我的文案 Agent 分三步：

![OpenClaw 架构图](03-openclaw-architecture.png)

```yaml
# 文案生成 Agent
- name: copywriter-agent
  trigger: research-complete
  actions:
    - read: "research/{{topic}}.md"
    - analyze-style: "xiaohongshu-viral"
    - generate:
        title-options: 5  # 生成 5 个标题
        body-length: "800-1200 字"
        tone: "闺蜜聊天风"
        emoji-density: "medium"
    - output: "drafts/{{topic}}-copy.md"
```

**小红书文案的关键点：**

| 要素 | 要求 | AI 提示词示例 |
|------|------|-------------|
| 标题 | 15-20 字，有情绪 | "生成 5 个让人想点的标题" |
| 开头 | 前 3 句抓住注意力 | "用'姐妹们'开头" |
| 正文 | 分点 + 表情符号 | "每段不超过 3 行" |
| 结尾 | 引导互动 | "加'你们觉得呢？'" |
| 标签 | 5-8 个精准 tag | "#AI 工具 #效率神器" |

---

### 第四步：配图生成

小红书是视觉驱动平台，封面图决定 70% 的点击率。

我集成了 baoyu-image-gen 技能：

```yaml
# 配图生成 Agent
- name: image-generator
  trigger: copy-complete
  actions:
    - analyze-copy: true
    - generate-prompts:
        cover: "16:9, 吸睛，对比强烈"
        body: "3:4, 信息图风格"
    - call-api: "baoyu-image-gen"
    - save: "images/{{topic}}/"
```

生成的图片类型：
- **封面图**：16:9 或 3:4，强对比度，大标题
- **内容图**：信息图、对比图、流程图
- **尾图**：引导关注/收藏

![配图示例](02-image-examples.png)

---

### 第五步：排版输出

最后一步是排版，输出可以直接发布的内容：

```yaml
# 排版 Agent
- name: formatter-agent
  trigger: images-ready
  actions:
    - merge: ["copy", "images"]
    - add-emojis: true
    - add-tags: "auto-generate-8"
    - format: "xiaohongshu-mobile"
    - output: "ready/{{topic}}.md"
    - notify: "telegram://{{user}}"
```

输出格式示例：

```markdown
# 标题：我用 AI 一天写了 10 篇小红书📝

姐妹们！发现一个超好用的 AI 工具🔥
每天 30 分钟，产出 10 篇笔记不是梦！

[图片 1：封面]

❶ 选题哪里来？
→ 热搜榜 + 竞品分析 + 关键词监控
[图片 2：选题流程图]

❷ 文案怎么写？
→ AI 生成初稿 + 人工润色
[图片 3：文案对比]

...

#AI 工具 #小红书运营 #自动化 #效率神器 #副业
```

---

## 05 实际效果：数据说话

我跑了一周，数据如下：

| 指标 | 人工模式 | AI 流水线 | 提升 |
|------|---------|----------|------|
| 日均产出 | 2 篇 | 10 篇 | +400% |
| 单篇耗时 | 2.5 小时 | 0.5 小时 | -80% |
| 平均阅读 | 120 | 450 | +275% |
| 涨粉速度 | 15/天 | 80/天 | +433% |
| 变现收入 | ¥0 | ¥380/周 | - |

![数据对比](04-data-comparison.png)

**最惊喜的发现：** AI 写的笔记，有时候数据比我人工写的还好。

我分析了一下原因：
- AI 不会累，输出的质量稳定
- AI 没有情绪，不会因为几篇数据差就崩
- AI 能快速测试不同风格，找到最优解

---

## 06 踩过的坑

当然，这一路也不是那么顺利。分享几个我踩过的坑：

### 坑 1：一开始就追求全自动

我第一版设计是全自动发布，结果：
- 发了一篇有争议的 AI 生成内容
- 被平台限流
- 账号差点被封

**教训：发布前一定要人工审核。**

现在我的流程是：AI 生成 → 人工确认 → 发布

---

### 坑 2：文案太 AI 味

刚开始 AI 生成的文案一看就很假：
- 太正式，没有小红书那种"闺蜜感"
- 表情符号太少
- 段落太长

**解决方案：**
- 收集 50 篇爆款笔记做 few-shot learning
- 在提示词里明确风格要求
- 加一个"拟人化"Agent 做二次改写

---

### 坑 3：图片质量不稳定

一开始用免费的 AI 绘图，生成的图：
- 分辨率太低
- 有水印
- 风格不统一

**解决方案：**
- 用 baoyu-image-gen，质量稳定
- 定义固定的 prompt 模板
- 建立自己的图片风格库

---

## 07 如何开始：三级起步方案

看完想试试？给你三个方案：

### Level 1：最小可行方案（0 成本）

**适合：** 想先试试水

**工具：**
- OpenClaw（免费）
- 手动选题
- AI 写文案（用免费额度）

**时间投入：** 每天 1 小时
**预期产出：** 3-5 篇/天

**步骤：**
1. 部署 OpenClaw（参考官方文档）
2. 配置文案 Agent
3. 手动选题 → AI 写 → 人工改 → 发布

---

### Level 2：半自动化（时间换效率）

**适合：** 已经有账号，想提升效率

**工具：**
- OpenClaw 完整版
- baoyu-image-gen
- 自动热搜监控

**时间投入：** 每天 30 分钟
**预期产出：** 8-12 篇/天

**步骤：**
1. 在 Level 1 基础上加素材 Agent
2. 加图片生成 Agent
3. 选题确认后全自动

---

### Level 3：全链路自动化（认真搞钱）

**适合：** 把小红书当副业/主业

**工具：**
- 多账号管理
- 自动发布（用官方 API 或模拟）
- 数据监控 + 自动优化

**时间投入：** 每天 15 分钟审核
**预期产出：** 20+ 篇/天

**步骤：**
1. 在 Level 2 基础上加自动发布
2. 配置数据监控 Agent
3. 根据数据自动调整选题和风格

![三级起步方案](05-three-levels-start.png)

---

## 08 一些思考

做这个项目一个月，我有几个感受：

### 1. AI 不是替代，是放大

AI 没有替代我的创造力，它替代的是：
- 重复性的搜集整理
- 机械性的排版
- 耗时的找图

**我的时间花在了更有价值的地方：**
- 选题判断
- 内容审核
- 账号定位

---

### 2. 流水线思维很重要

不要把 AI 当成一个"写手"，要把整个流程拆成：
- 输入 → 处理 → 输出

每一环都可以：
- 用 AI 替代人工
- 用代码固化流程
- 用数据优化决策

---

### 3. 合规是底线

小红书对 AI 内容的态度是：
- 不禁止 AI 辅助创作
- 禁止纯 AI 生成的低质内容
- 要求标注 AI 生成（部分品类）

**我的建议：**
- AI 生成 + 人工润色
- 保证内容有价值
- 必要时标注"AI 辅助"

---

## 09 资源分享

### OpenClaw 相关

- 官方文档：https://docs.openclaw.ai
- GitHub: https://github.com/openclaw
- 配置文件模板：见本文末尾

### 小红书运营资源

- 热搜榜：https://www.xiaohongshu.com/hot
- 对标账号分析工具：新红、千瓜
- 标签生成器：OpenClaw 内置

### 配图工具

- baoyu-image-gen（我用这个）
- Midjourney
- 可画（Canva）

---

## 10 配置模板分享

最后，分享我的 OpenClaw 配置文件核心部分：

```yaml
# xiaohongshu-pipeline.yaml
version: "1.0"
pipeline: xiaohongshu-auto

triggers:
  - name: daily-hot-search
    type: scheduler
    cron: "0 9 * * *"

  - name: manual-topic
    type: webhook
    endpoint: "/submit-topic"

agents:
  - name: topic-selector
    type: decision
    input: ["hot-search", "manual-topic"]
    output: selected-topic.md

  - name: researcher
    type: scraper
    actions:
      - search-web: "{{topic}}"
      - fetch-top: 10
      - summarize: true

  - name: copywriter
    type: llm
    model: "claude-sonnet-4-6"
    prompt-template: "xiaohongshu-copy.txt"
    few-shot: "examples/viral-posts.json"

  - name: image-gen
    type: api-call
    api: "baoyu-image-gen"
    params:
      quality: "2k"
      style: "xiaohongshu-cover"

  - name: formatter
    type: template
    template: "xiaohongshu-format.md"
    output-dir: "ready/{{date}}/"

notifications:
  - type: telegram
    events: ["pipeline-complete", "error"]
    template: "今日产出{{count}}篇，请审核发布"
```

---

## 11 结语

AI 时代，不会用工具的人会被会用工具的人替代。

小红书运营这件事，本质上是在：
- 捕捉用户需求
- 生产有价值的内容
- 建立信任和影响力

AI 能帮我们把"生产"这个环节的效率提升 10 倍，但"捕捉需求"和"建立信任"还是需要人。

**工具是杠杆，但支点和方向在你手里。**

共勉。

---

*配置文件和完整代码已开源，可在评论区获取。*

*— END —*
