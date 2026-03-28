---
date: 2026-03-27
tags: [learning-note, claude-code, training]
source: Anthropic Skilljar
status: in-progress
---

# Claude Code in Action - 课程笔记

## 📌 来源信息

- **来源**: Anthropic Skilljar
- **课程**: Claude Code in Action
- **链接**: https://anthropic.skilljar.com/claude-code-in-action
- **开始日期**: 2026-03-27

---

## 💡 学习目标

1. 掌握 Claude Code 的核心功能和工作流程
2. 学习在实际开发场景中高效使用 Claude Code
3. 将所学应用到日常开发和项目中

---

## 📝 课程大纲

### Module 1: What is Claude Code?
- [x] Introduction
- [x] What is a coding assistant?
- [x] Claude Code in action

### Module 2: Getting hands on
- [ ] Claude Code setup
- [ ] Project setup
- [ ] Adding context
- [ ] Making changes
- [ ] Course satisfaction survey
- [ ] Controlling context
- [ ] Custom commands
- [ ] MCP servers with Claude Code
- [ ] Github integration

### Module 3: Hooks and the SDK
- [ ] Introducing hooks
- [ ] Defining hooks
- [ ] Implementing a hook
- [ ] Gotchas around hooks
- [ ] Useful hooks!
- [ ] Another useful hook
- [ ] The Claude Code SDK

### Module 4: Wrapping up
- [ ] Quiz on Claude Code
- [ ] Summary and next steps

---

## 💡 关键知识点

### Module 1: What is Claude Code?

#### Introduction
Course goals:
1. What's a coding assistant?
2. Why Claude Code?
3. Working with Claude Code as a partner
4. Getting the most out of Claude Code

![[ClaudeCodeInAction_Introduction_cousegoal.png]]

#### What is a coding assistant?

![[ClaudeCodeInAction_whatisacodingassistant.png]]

**1. 核心定义** 编程助手 = 语言模型 + **工具使用（Tool Use）**。模型本身只能处理文本，必须依靠工具才能读写文件或运行命令。

**2. 工作三部曲**

- **收集上下文**：搞清楚问题和相关文件。
- **制定计划**：构思修复方案。
- **执行行动**：通过工具修改代码和测试。

**3. "工具使用"是如何实现的？**

- **原理**：助手会在后台教模型如何通过"特定格式的文本"来发号施令（例如输出 `ReadFile: filename`）。
- **闭环**：模型发指令 → 助手在本地执行（读文件/跑命令） → 将结果传回模型 → 模型给出最终答案。

**4. 为什么 Claude 更强？** Claude 系列模型对工具指令的理解更精准，能够处理复杂的连续任务，并支持自定义扩展工具，同时无需上传整个代码库，安全性更高。

#### Claude Code in action

本节课是 demo 演示

---

### Module 2: Getting hands on

#### Claude Code setup
1. `Install Claude Code`
    1. `` MacOS (Homebrew): `brew install --cask claude-code` ``
    2. MacOS, Linux, WSL: `curl -fsSL https://claude.ai/install.sh | bash`
    3. Windows CMD: `curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd`
2. After installation, run `claude` at your terminal. The first time you run this command you will be prompted to authenticate

#### Project setup
<!-- 笔记内容 -->

#### Adding context
<!-- 笔记内容 -->

#### Making changes
<!-- 笔记内容 -->

#### Course satisfaction survey
<!-- 笔记内容 -->

#### Controlling context
<!-- 笔记内容 -->

#### Custom commands
<!-- 笔记内容 -->

#### MCP servers with Claude Code
<!-- 笔记内容 -->

#### Github integration
<!-- 笔记内容 -->

---

### Module 3: Hooks and the SDK

#### Introducing hooks
<!-- 笔记内容 -->

#### Defining hooks
<!-- 笔记内容 -->

#### Implementing a hook
<!-- 笔记内容 -->

#### Gotchas around hooks
<!-- 笔记内容 -->

#### Useful hooks!
<!-- 笔记内容 -->

#### Another useful hook
<!-- 笔记内容 -->

#### The Claude Code SDK
<!-- 笔记内容 -->

---

### Module 4: Wrapping up

#### Quiz on Claude Code
<!-- 笔记内容 -->

#### Summary and next steps
<!-- 笔记内容 -->

---

## 🔗 关联内容

### 关联目标
- [[../00-Learning-Index#当前学习项目]]
- [[4. Monthly Goals - April 2026#🟢 Nice to Complete (Bonus)]]

### 关联项目
- [[Projects/Human-Agent-Kanban]]
- [[Projects/getaiti-next]]

### 关联日记
- [[Daily Notes/2026-03-27]]

---

## 💭 行动项

- [x] 完成课程注册/登录
- [x] 浏览课程整体结构
- [x] 开始第一个模块学习

---

## 📅 复习记录

| 日期 | 学习进度 | 关键收获 |
|------|---------|---------|
| 2026-03-27 | Module 1 完成 | 理解编程助手的核心概念 |

---

*Created: 2026-03-27 | Last Updated: 2026-03-27*
