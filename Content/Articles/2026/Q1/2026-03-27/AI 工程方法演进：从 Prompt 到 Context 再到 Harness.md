---
date: 2026-03-27
tags: [ai-engineering, prompt-engineering, context-engineering, harness-engineering]
category: tech-article
cover_image: images/00-cover-harness-engineering.png
---

![封面：AI 工程方法演进](images/00-cover-harness-engineering.png)

# AI 工程方法演进：从 Prompt 到 Context 再到 Harness

> 代理不难，难的是 Harness。

2022 年到 2026 年，AI 工程方法经历了三次范式转移。

如果你还在为每次写提示词绞尽脑汁，或者困惑为什么同一个模型在不同人手里表现差异巨大——这篇文章或许能给你一个更清晰的视角。

---

## 三代演进，三个核心问题

AI 工程方法在短短几年间完成了三次迭代：

| 代际 | 时间 | 核心问题 | 代表人物 |
|------|------|----------|----------|
| **Prompt Engineering** | 2022-2024 | "如何措辞这条指令？" | 社区驱动 |
| **Context Engineering** | 2025 | "模型需要知道什么？" | Andrej Karpathy |
| **Harness Engineering** | 2026 | "如何让代理在数千次推理中可靠运行？" | Mitchell Hashimoto, OpenAI |

每一次迭代，都不是对前一代的否定，而是**范围的扩展**。

---

## 第一代：Prompt Engineering（2022-2024）

### 核心：如何措辞？

Prompt Engineering 是我们大多数人与 AI 的第一次接触。

它的核心问题很简单：**"我该如何措辞这条指令？"**

关键组件包括：
- **Instructions** - 定义模型应该做什么
- **Primary Content** - 被处理或转换的文本
- **Examples** - few-shot learning
- **Chain-of-Thought** - 将复杂问题分解为步骤
- **Temperature** - 调整随机性

### 适用场景

- 简单任务：摘要、翻译、问答、内容生成
- 快速原型功能
- 偶尔不准确风险较低的场景

### 局限性

但很快，人们发现了问题：

1. **极度脆弱** - 看似无害的改写可能触发破坏性变化
   - 将 "Output strictly valid JSON" 改为 "Always respond using clean, parseable JSON" 可导致结构化输出错误率飙升
   - 重新排序示例可产生超过 40% 的准确性波动

2. **难以版本控制** - 难以测试，几乎无法在团队间标准化

3. **静默失败** - 输出看起来连贯但包含事实漂移或偏见建议

4. **维护负担** - 在生产环境中成为维护负担而非可扩展解决方案

Prompt Engineering 是必要的起点，但它远远不够。

![Prompt Engineering 场景](images/01-prompt-engineering.png)

---

## 第二代：Context Engineering（2025）

### 核心：模型需要知道什么？

2025 年，Andrej Karpathy 提出了 Context Engineering 的概念：

> "Context engineering is the delicate art and science of filling the context window with just the right information for the next step."

核心问题从**"如何措辞"**转向了**"模型需要知道什么？"**

### LangChain 四阶段框架

LangChain 将 Context Engineering 分为四个核心策略：

#### 1. Write Context（写入上下文）

将信息保存在上下文窗口之外以帮助代理执行任务。

- **Scratchpads（草稿板）** - 代理在执行任务时保存信息的机制
- **Memories（记忆）** - 跨会话持久化信息

#### 2. Select Context（选择上下文）

将信息拉入上下文窗口以帮助代理执行任务。

- 使用 Embeddings 和知识图进行记忆索引
- 对工具描述应用 RAG 可将工具选择准确性提高 3 倍

#### 3. Compress Context（压缩上下文）

仅保留执行任务所需的 token。

- Claude Code 在超过 95% 上下文窗口后运行 "auto-compact"
- 可使用递归或分层摘要策略

#### 4. Isolate Context（隔离上下文）

拆分上下文以帮助代理执行任务。

- 多代理系统：将上下文跨子代理拆分
- 沙箱：将 token 重的对象隔离在环境中

### 核心挑战：Context Rot

随着 token 数量增加，模型准确回忆信息的能力会下降。

Drew Breunig 总结了四种失败模式：
1. **Context Poisoning** - 幻觉进入上下文
2. **Context Distraction** - 上下文压倒训练
3. **Context Confusion** - 多余上下文影响响应
4. **Context Clash** - 上下文各部分相互矛盾

![Context Engineering 四阶段](images/02-context-engineering.png)

---

## 第三代：Harness Engineering（2026）

### 核心：如何让代理可靠运行？

2026 年，AI 工程方法进入了第三代：**Harness Engineering**。

Mitchell Hashimoto 给出了一个精准的定义：

> "Every time you discover an agent has made a mistake, you take the time to engineer a solution so that it can never make that mistake again."

OpenAI Codex 团队的总结更加直接：

> **"Agents aren't hard; the Harness is hard."**

### 马的隐喻

Harness Engineering 用一个简单的隐喻来解释：

| 元素 | 对应 | 说明 |
|------|------|------|
| **马** | AI 模型 | 强大、快速，但不知道自己去哪里 |
| **马具** | 基础设施 | 约束、护栏、反馈循环，引导模型生产力 |
| **骑手** | 人类工程师 | 提供方向，不做奔跑 |

**没有马具，AI 代理就像开阔田野中的纯种马——快速、令人印象深刻，但完全无法完成任何工作。**

![Harness Engineering 架构](images/03-harness-engineering.png)

### 三大支柱

#### 1. Context Engineering（上下文工程）

这是 Harness 的基础层。

**Static Context（静态上下文）：**
- 仓库本地文档（架构规范、API 合约、风格指南）
- AGENTS.md 或 CLAUDE.md 文件，编码项目特定规则
- 由 linter 验证的交叉链接设计文档

**Dynamic Context（动态上下文）：**
- 可观察性数据（日志、指标、跟踪）对代理可访问
- 代理启动时的目录结构映射
- CI/CD 管道状态和测试结果

**关键规则：** 从代理的角度来看，任何它在上下文中无法访问的东西都不存在。

#### 2. Architectural Constraints（架构约束）

这是 Harness Engineering 与传统 AI 提示最根本的分歧。

不是告诉代理"写好代码"，而是**机械地强制执行**好代码的样子。

**依赖分层：**
```
Types → Config → Repo → Service → Runtime → UI
```
每层只能从左侧的层导入。这不是建议——由结构测试和 CI 验证强制执行。

**约束执行工具：**
- **Deterministic Linters** - 自动标记违规的自定义规则
- **LLM-based Auditors** - 审查其他代理代码的代理
- **Structural Tests** - 类似 ArchUnit，但用于 AI 生成的代码
- **Pre-commit Hooks** - 任何代码提交前的自动检查

**为什么约束改善输出？**

矛盾的是，约束解决方案空间使代理更有生产力，而不是更少。当代理可以生成任何东西时，它浪费 token 探索死胡同。当 Harness 定义清晰边界时，代理更快地收敛到正确解决方案。

#### 3. Entropy Management（熵管理）

这是最被低估的组件。

随着时间的推移，AI 生成的代码库会积累熵——文档与现实脱节、命名约定分歧、死代码积累。

**定期清理代理：**
- **Documentation Consistency Agents** - 验证文档与当前代码匹配
- **Constraint Violation Scanners** - 查找绕过早期检查的代码
- **Pattern Enforcement Agents** - 识别并修复偏离既定模式的内容
- **Dependency Auditors** - 跟踪和解决循环或不必要的依赖

这些代理按计划运行——每日、每周或由特定事件触发——保持代码库对人类审查者和未来 AI 代理都健康。

### 行业验证

#### OpenAI：100 万行零人工代码

**实验详情：**
- 5 个月开发
- 最终产品 100 万 + 行代码
- **零手动编写行**——每行都由 Codex 代理生成
- 构建时间约为人类所需时间的 1/10

工程师的工作？设计 Harness、指定意图、提供反馈。**不是写代码。**

#### Stripe：每周 1,300+ AI 编写的 PR

- 开发者在 Slack 发布任务
- Minion 写代码 → 通过 CI → 开 PR → 人类审查合并
- 步骤 1 到 5 之间无开发者交互

**关键特性：**
- "Blueprint" 编排，将工作流分为确定性节点和代理节点
- 严格执行 CI/CD 两击规则：如果代理的第一次修复失败，任务立即升级给人类

### 性能影响

Harness Engineering 不是理论，它有实实在在的数据支撑：

| 研究/案例 | 改进 | 说明 |
|-----------|------|------|
| Princeton 研究 | +64% 解决率 | 仅改变 Harness 配置 |
| LangChain | 52.8% → 66.5% | Terminal Bench 2.0，从 Top 30 到 Top 5 |
| 同一模型不同 Harness | 2% vs 12% | 6 倍性能差距，完全归因于环境设计 |

**同一模型，仅因 Harness 不同，表现差异可达 6 倍。**

![性能对比](images/04-performance-comparison.png)

---

## 对软件工程师的影响

### 工作正在改变

| 之前 | 之后 |
|------|------|
| 编写代码 | 设计 AI 编写代码的环境 |
| 调试代码 | 调试代理行为 |
| 审查代码 | 审查代理输出 + Harness 有效性 |
| 编写测试 | 设计测试策略 |
| 维护文档 | 构建文档作为机器可读基础设施 |

这并不意味着工程师变得不那么技术化。相反，Harness Engineering 需要**更深的架构思维**——你正在设计必须在没有持续干预的情况下工作的系统。

### 重要技能

1. **系统思维** - 理解约束、反馈循环和文档如何交互
2. **架构设计** - 定义可执行且富有成效的边界
3. **规范编写** - 精确表达意图以便代理执行
4. **可观察性** - 构建揭示代理行为模式的监控
5. **迭代速度** - 快速测试和完善 Harness 配置

---

## 如何开始：三级框架

### Level 1：基础 Harness（单开发者）

**适用：** 使用 Claude Code、Cursor 或 Codex 的个人项目

**设置内容：**
- CLAUDE.md 或 .cursorrules 文件，包含项目约定
- 用于 linting 和格式化的预提交钩子
- 代理可以运行以自我验证的测试套件
- 具有命一致性的清晰目录结构

**设置时间：** 1-2 小时
**影响：** 防止最常见的代理错误

### Level 2：团队 Harness（小团队）

**适用：** 3-10 人团队共享代码库

**在 Level 1 基础上增加：**
- AGENTS.md 包含团队范围约定
- CI 强制执行的架构约束
- 常见任务的共享提示模板
- 由 linter 验证的文档即代码
- 专门针对代理生成 PR 的代码审查清单

**设置时间：** 1-2 天
**影响：** 团队间一致的代理行为

### Level 3：生产 Harness（工程组织）

**适用：** 运行数十个并发代理的组织

**在 Level 2 基础上增加：**
- 自定义中间件层（循环检测、推理优化）
- 可观察性集成（代理读取日志和指标）
- 按计划运行的熵管理代理
- Harness 版本控制和 A/B 测试
- 代理性能监控仪表板
- 代理卡住时的升级策略

**设置时间：** 1-2 周
**影响：** 代理作为自主贡献者运行

![三级框架](images/05-three-levels-framework.png)

---

## 常见错误

1. **过度工程化控制流**
   - "如果你过度工程化控制流，下一个模型更新会破坏你的系统"
   - 构建可拆卸的 Harness——当模型足够智能时，你应该能够移除"智能"逻辑

2. **将 Harness 视为静态**
   - Harness 需要随模型演进
   - 每个主要模型更新时审查和更新 Harness 组件

3. **忽视文档层**
   - 最有影响力的 Harness 改进通常最简单：更好的文档
   - 如果 AGENTS.md 模糊，代理输出将模糊

4. **无反馈循环**
   - 没有反馈的 Harness 是笼子，不是指南
   - 代理需要知道何时成功、何时失败

5. **仅人类文档**
   - 如果架构决策存在于人们头脑中或代理无法访问的 Confluence 页面中，Harness 有缺口
   - 代理需要的一切必须在仓库中

---

## 核心洞察

### 1. 模型是商品，Harness 是护城河

- LangChain 仅通过改变 Harness 就从基准 Top 30 跃升至 Top 5
- 同一模型在不同 Harness 中表现差异可达 6 倍

### 2. 约束创造自由

- 限制代理的解决方案空间矛盾地提高生产力
- 清晰边界让代理更快收敛到正确方案

### 3. 三层叠加策略

- 从提示开始获取快速收益
- 工作流复杂时添加上下文工程
- 生产部署前实施 Harness 基础设施

### 4. 工程师角色演变

- 从编写代码到设计环境
- 从调试代码到调试代理行为
- 从编写测试到设计测试策略

---

## 最终思考

AI 编码工具的终局不是关于人类不写代码。而是关于人类只做值得做的决定。

配置、权限、规则、边界——这些不是开销。它们是你保持控制的方式。

**真正的障碍不是你能否使用 AI 代理。而是你能否让它保持克制。**

---

## 参考资料

1. NxCode. "Harness Engineering: The Complete Guide to Building Systems That Make AI Agents Actually Work (2026)"
2. Epsilla. "The Third Evolution: Why Harness Engineering Replaced Prompting in 2026"
3. Dev.to. "Prompt Engineering vs Context Engineering vs Harness Engineering: What's the Difference in 2026?"
4. Mitchell Hashimoto. "My AI Adoption Journey"
5. OpenAI. "Harness engineering: leveraging Codex in an agent-first world"

---

*本文基于深度调研报告撰写，原文档包含更多技术细节和案例研究。*
