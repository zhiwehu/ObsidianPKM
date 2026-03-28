# getaiti-next (AITi / getaiti.com)

## Goal Link
Supports: [[1. Yearly Goals#Goal 2: 建立 AI 创业产品]]
Supports: [[2. Monthly Goals#AI 资讯收集平台]]

---

## 📋 Project Overview

**Project Name:** AITi (getaiti.com) — AI 能力发现与学习平台

**Type:** AI 内容平台 / 知识社区 / 工具导航

**Stage:** 🚧 开发阶段

**Start Date:** 2026-03-28（Day 1 开发启动）

**Target:** 2026-03-31 完成 MVP 上线

**代码位置:** `/Users/huzhiwei/Projects/getaiti-next`

---

## 🧭 网站定位（2026-03-28 更新）

**一句话定位：** 让不同层次用户快速学习使用 AI 的导航与学习平台

**收录范围：**

| 类别 | 示例 |
|------|------|
| **AI Agents** | Claude Code, CodeX, OpenClaw, OpenCode, OpenManus |
| **Plugins / Skills** | Claude Plugins, Claude Code Skills, VS Code AI 插件 |
| **平台** | n8n, Coze（扣子）, 腾讯元器, Dify |
| **AI 方法论** | Prompt Engineering, RAG, Agent 设计模式 |
| **自动化 & 工作流** | 工作流自动化、AI 集成最佳实践 |
| **常用 IDE** | VS Code, Cursor, Windsurf, Trae 等 AI 增强 IDE |

**目标用户：**
- 初学者：想了解 AI 能做什么
- 进阶用户：想掌握 Agent、自动化工作流
- 专业用户：想深入使用 Agent 系统进行深度定制

---

## 🎯 ONE THING

**"7 天内从零到 MVP 上线"**

---

## 📊 Milestones

### Phase 1: MVP 开发（Day 1-7）

| Day       | 任务                           | 状态    |
| --------- | ---------------------------- | ----- |
| **Day 1** | 项目初始化 + 首页 + 搜索页 + 详情页 + 多语言 | ✅ 完成  |
| **Day 2** | 分类重构（Agents/Plugins/平台/方法论）+ 提交页 | ⏳ 待开始 |
| **Day 3** | URL 处理脚本                   | ⏳ 待开始 |
| **Day 4** | AI 流水线配置                   | ⏳ 待开始 |
| **Day 5** | 测试 + 修复                    | ⏳ 待开始 |
| **Day 6** | 真实数据集成                    | ⏳ 待开始 |
| **Day 7** | 部署上线                      | ⏳ 待开始 |

---

## 🔧 Technical Stack

| 组件 | 技术 |
|------|------|
| 前端 | Next.js 15 + React 18 + TailwindCSS v4 |
| AI 处理 | OpenClaw + baoyu-skills |
| 存储 | Markdown 文件 + Git 版本控制 |
| 搜索 | 前端过滤（可升级为 Fuse.js）|
| 部署 | GitHub + Vercel |
| 域名 | getaiti.com |
| Logo | 原始 SVG logo（已集成）|

---

## 📁 页面结构

```
src/app/
├── page.tsx              # 首页 ✅
├── search/
│   └── page.tsx          # 搜索页 ✅
├── tools/
│   └── [slug]/
│       └── page.tsx      # 工具详情页 ✅
└── submit/
    └── page.tsx          # 提交页 ⏳
```

---

## 📝 Next Actions

### 今天完成
- [x] 初始化 Next.js 项目（Next.js 15 + React 18）
- [x] 开发首页（Hero + 工具卡片）
- [x] 开发搜索页（实时搜索 + 分类筛选）
- [x] 开发工具详情页（特性 + 优缺点）
- [x] 集成原始 logo
- [x] 多语言支持（中/英，localStorage 存储）
- [x] 专业详情页（AI 评测 + 用户评测 + 点赞/收藏/分享 + 字体调节）

### 待完成
- [ ] 重构分类体系（Agents / Plugins / 平台 / 方法论）
- [ ] 创建 GitHub 仓库并推送代码
- [ ] 配置 Vercel 自动部署
- [ ] 注册 X 账号 @AITiHQ
- [ ] 收录更多 Agent 类工具

---

## 📂 Related Notes

- [[1. Yearly Goals]] - 年度目标
- [[2. Monthly Goals]] - 本月目标
- [[3. Weekly Review]] - 周进度追踪
- [[DAY1-PROGRESS]] — Day 1 开发进度
- [DAY1-PROGRESS](DAY1-PROGRESS.md) - Day 1 详细进度

---

*Last Updated: 2026-03-28*
*Phase: Active — Day 1 开发中*
*Progress: 90% (Day 1 基本完成，多语言已集成)*
