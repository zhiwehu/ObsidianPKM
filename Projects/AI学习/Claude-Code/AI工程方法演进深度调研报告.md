---
date: 2026-03-27
tags: [ai-engineering, research, prompt-engineering, context-engineering, harness-engineering]
status: active
source: OpenClaw
type: research-report
---

# AI 工程方法演进深度调研报告
## 从 Prompt Engineering 到 Context Engineering 再到 Harness Engineering

**调研时间：** 2026 年 3 月 27 日  
**调研范围：** AI 工程方法三代演进、核心实践、行业案例、未来趋势

---

## 执行摘要

AI 工程方法在 2022-2026 年间经历了三次范式转移：

| 代际 | 时间 | 核心问题 | 关键人物/机构 |
|------|------|----------|---------------|
| **Prompt Engineering** | 2022-2024 | "如何措辞这条指令？" | 社区驱动 |
| **Context Engineering** | 2025 | "模型需要知道什么？" | Andrej Karpathy |
| **Harness Engineering** | 2026 | "如何让代理在数千次推理中可靠运行？" | Mitchell Hashimoto, OpenAI |

**核心发现：**
- 95% 的企业 AI 试点未能交付可衡量回报，问题在于系统设计而非指令质量
- 仅改变 Harness 配置即可将解决率提升 64%（Princeton 研究）
- OpenAI 团队 5 个月构建 100 万行代码，零人工编写——证明 Harness Engineering 在生产规模可行
- **"Agents aren't hard; the Harness is hard."** — OpenAI Codex 团队

---

## 第一代：Prompt Engineering（2022-2024）

### 定义与核心

Prompt Engineering 是通过设计和优化自然语言输入来从生成式 AI 模型产生特定输出的实践。

**核心问题：** "How should I phrase this?"（我该如何措辞？）

### 关键组件

1. **Instructions（指令）** - 定义模型应该做什么
2. **Primary Content（主要内容）** - 被处理或转换的文本
3. **Examples（示例）** - 通过输入输出对展示期望行为（few-shot learning）
4. **Cues（提示）** - 启动模型输出
5. **Supporting Content（支持内容）** - 影响响应但不是主要目标
6. **Chain-of-Thought（思维链）** - 将复杂问题分解为顺序步骤
7. **Temperature Parameters（温度参数）** - 调整随机性

### 适用场景

- 简单任务：摘要、翻译、问答、内容生成
- 快速原型功能
- 自动化重复性任务
- 偶尔不准确风险较低的场景

### 局限性

1. **极度脆弱** - 看似无害的改写可能触发破坏性变化
   - 将 "Output strictly valid JSON" 改为 "Always respond using clean, parseable JSON" 可导致结构化输出错误率飙升
   - 重新排序示例可产生超过 40% 的准确性波动

2. **难以版本控制** - 难以测试，几乎无法在团队间标准化

3. **静默失败** - 输出看起来连贯但包含事实漂移或偏见建议

4. **维护负担** - 在生产环境中成为维护负担而非可扩展解决方案

### 典型案例

- 营销团队使用提示创建草稿
- 客户支持使用提示生成初步回复建议
- 简单查询或创意场景

---

## 第二代：Context Engineering（2025）

### 定义与核心

Context Engineering 设计系统来确定 AI 模型在生成响应之前访问什么信息。它构建了模型周围的完整信息环境。

**核心问题：** "What does the model need to know?"（模型需要知道什么？）

**Andrej Karpathy 的定义：**
> "Context engineering is the delicate art and science of filling the context window with just the right information for the next step."
> （上下文工程是用恰好正确的信息填充上下文窗口的微妙艺术和科学）

### 关键区别

| Prompt Engineering | Context Engineering |
|-------------------|---------------------|
| 优化单次交互 | 管理跨多轮的系统级信息流 |
| 失败源于模糊措辞 | 失败源于错误文档、过时信息或上下文溢出 |
| 调试需要语言优化 | 调试需要数据架构工作 |
| "如何思考" | "可以推理什么" |

### LangChain 四阶段框架

LangChain 将 Context Engineering 分为四个核心策略：

#### 1. Write Context（写入上下文）

将信息保存在上下文窗口之外以帮助代理执行任务。

**Scratchpads（草稿板）：**
- 代理在执行任务时保存信息的机制
- Anthropic 的多代理研究员示例：LeadResearcher 将计划保存到 Memory，因为上下文窗口超过 200,000 token 时会被截断

**Memories（记忆）：**
- 跨会话持久化信息
- ChatGPT、Cursor、Windsurf 都有自动生成跨会话长期记忆的机制

#### 2. Select Context（选择上下文）

将信息拉入上下文窗口以帮助代理执行任务。

**挑战：**
- 确保选择相关记忆
- 意外或不需要的记忆检索会让用户感觉上下文窗口"不再属于他们"

**技术方案：**
- Embeddings 和知识图用于记忆索引
- 对工具描述应用 RAG 可将工具选择准确性提高 3 倍

#### 3. Compress Context（压缩上下文）

仅保留执行任务所需的 token。

**Context Summarization（上下文摘要）：**
- Claude Code 在超过 95% 上下文窗口后运行 "auto-compact"
- 可使用递归或分层摘要策略

**Context Trimming（上下文修剪）：**
- 使用硬编码启发式方法，如从列表中删除旧消息
- Provence 是用于问答的训练上下文修剪器

#### 4. Isolate Context（隔离上下文）

拆分上下文以帮助代理执行任务。

**Multi-agent（多代理）：**
- 将上下文跨子代理拆分
- Anthropic 的多代理研究员：许多具有隔离上下文的代理优于单代理
- 挑战：token 使用（最多比聊天多 15 倍）、需要仔细规划子代理工作

**Sandboxes（沙箱）：**
- HuggingFace 的 deep researcher 使用 CodeAgent
- 允许将 token 重的对象隔离在环境中

### 核心挑战

**Context Rot（上下文腐烂）：** 随着 token 数量增加，模型准确回忆信息的能力下降

**四种失败模式（Drew Breunig）：**
1. **Context Poisoning** - 幻觉进入上下文
2. **Context Distraction** - 上下文压倒训练
3. **Context Confusion** - 多余上下文影响响应
4. **Context Clash** - 上下文各部分相互矛盾

### 典型技术栈（2025）

- **Memory:** Zep AI (YC W24)
- **Knowledge base:** Milvus (Zilliz)
- **Agent orchestration:** CrewAI
- **Observability & tracing:** Confident AI (YC W25) DeepEval

---

## 第三代：Harness Engineering（2026）

### 定义与核心

Harness Engineering 是设计和实现围绕 AI 代理的完整操作系统的学科：约束、反馈循环、编排层和控制机制，将原始模型输出转化为生产级系统。

**核心问题：** "How do agents operate reliably over thousands of inferences?"（代理如何在数千次推理中可靠运行？）

**Mitchell Hashimoto 的定义：**
> "Every time you discover an agent has made a mistake, you take the time to engineer a solution so that it can never make that mistake again."
> （每次你发现代理犯了错误，你就花时间设计一个解决方案，使其永远不再犯那个错误）

**OpenAI Codex 团队的总结：**
> "Agents aren't hard; the Harness is hard."
> （代理不难；难的是 Harness）

### 马的隐喻

| 元素 | 对应 | 说明 |
|------|------|------|
| **马** | AI 模型 | 强大、快速，但不知道自己去哪里 |
| **马具** | 基础设施 | 约束、护栏、反馈循环，引导模型生产力 |
| **骑手** | 人类工程师 | 提供方向，不做奔跑 |

没有马具，AI 代理就像开阔田野中的纯种马——快速、令人印象深刻，但完全无法完成任何工作。

### 三大支柱（OpenAI 框架）

#### 1. Context Engineering（上下文工程）

**Static Context（静态上下文）：**
- 仓库本地文档（架构规范、API 合约、风格指南）
- AGENTS.md 或 CLAUDE.md 文件，编码项目特定规则
- 由 linter 验证的交叉链接设计文档

**Dynamic Context（动态上下文）：**
- 可观察性数据（日志、指标、跟踪）对代理可访问
- 代理启动时的目录结构映射
- CI/CD 管道状态和测试结果

**关键规则：** 从代理的角度来看，任何它在上下文中无法访问的东西都不存在。Google Docs、Slack 线程或人们头脑中的知识对系统是不可见的。仓库必须是单一事实来源。

#### 2. Architectural Constraints（架构约束）

这是 Harness Engineering 与传统 AI 提示最根本的分歧。不是告诉代理"写好代码"，而是机械地强制执行好代码的样子。

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

**为什么约束改善输出：** 矛盾的是，约束解决方案空间使代理更有生产力，而不是更少。当代理可以生成任何东西时，它浪费 token 探索死胡同。当 Harness 定义清晰边界时，代理更快地收敛到正确解决方案。

#### 3. Entropy Management / Garbage Collection（熵管理/垃圾回收）

这是最被低估的组件。随着时间的推移，AI 生成的代码库会积累熵——文档与现实脱节、命名约定分歧、死代码积累。

**定期清理代理：**
- **Documentation Consistency Agents** - 验证文档与当前代码匹配
- **Constraint Violation Scanners** - 查找绕过早期检查的代码
- **Pattern Enforcement Agents** - 识别并修复偏离既定模式的内容
- **Dependency Auditors** - 跟踪和解决循环或不必要的依赖

这些代理按计划运行——每日、每周或由特定事件触发——保持代码库对人类审查者和未来 AI 代理都健康。

### 行业验证案例

#### OpenAI：100 万行零人工代码

**实验详情：**
- 5 个月开发
- 最终产品 100 万 + 行代码
- 零手动编写行——每行都由 Codex 代理生成
- 构建时间约为人类所需时间的 1/10
- 产品有内部日常用户和外部 alpha 测试者
- 它发布、部署、崩溃、修复——全部由 Harness 内的代理完成

**工程师的工作？** 设计 Harness、指定意图、提供反馈。不是写代码。

#### Stripe：Minions 规模化

**数据：**
- 每周 1,300+ 个 AI 编写的合并 PR
- 开发者在 Slack 发布任务
- Minion 写代码 → 通过 CI → 开 PR → 人类审查合并
- 步骤 1 到 5 之间无开发者交互

**关键特性：**
- "Blueprint" 编排，将工作流分为确定性节点（运行 linter、提交）和代理节点（实现功能、修复 CI 失败）
- 严格执行 CI/CD 两击规则：如果代理的第一次修复失败，任务立即升级给人类

#### LangChain：中间件优先

**Harness 结构为可组合中间件层：**
```
Agent Request
 → LocalContextMiddleware（映射代码库）
 → LoopDetectionMiddleware（防止重复）
 → ReasoningSandwichMiddleware（优化计算）
 → PreCompletionChecklistMiddleware（强制执行验证）
 → Agent Response
```

每个中间件层添加特定功能而不修改核心代理逻辑。

#### Anthropic：Generator/Evaluator 分离

**核心洞察：** 模型无法可靠评估自己的工作。

**GAN 启发式解决方案：**
- **Generator Agent** - 编写代码、设计 UI 或执行主要任务
- **Evaluator Agent** - 充当 QA 工程师，使用 Playwright 等工具与应用程序交互、点击按钮、检查 API 响应、验证数据库状态

**关键发现：** 使用专门的评估器代理比教导生成器代理自我批评容易得多。

### 性能影响数据

| 研究/案例 | 改进 | 说明 |
|-----------|------|------|
| Princeton 研究 | +64% 解决率 | 仅改变 Harness 配置 |
| LangChain | 52.8% → 66.5% | Terminal Bench 2.0，从 Top 30 到 Top 5 |
| 同一模型不同 Harness | 2% vs 12% | 6 倍性能差距，完全归因于环境设计 |

### 构建 Harness 的三级框架

#### Level 1：基础 Harness（单开发者）

**适用：** 使用 Claude Code、Cursor 或 Codex 的个人项目

**设置内容：**
- CLAUDE.md 或 .cursorrules 文件，包含项目约定
- 用于 linting 和格式化的预提交钩子
- 代理可以运行以自我验证的测试套件
- 具有命一致性的清晰目录结构

**设置时间：** 1-2 小时  
**影响：** 防止最常见的代理错误

#### Level 2：团队 Harness（小团队）

**适用：** 3-10 人团队共享代码库

**在 Level 1 基础上增加：**
- AGENTS.md 包含团队范围约定
- CI 强制执行的架构约束
- 常见任务的共享提示模板
- 由 linter 验证的文档即代码
- 专门针对代理生成 PR 的代码审查清单

**设置时间：** 1-2 天  
**影响：** 团队间一致的代理行为

#### Level 3：生产 Harness（工程组织）

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

### 常见错误

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

## 三代对比总结

| 属性 | Prompt Engineering | Context Engineering | Harness Engineering |
|------|-------------------|---------------------|---------------------|
| **定义** | 结构化自然语言输入以产生指定输出 | 设计系统确定模型访问什么信息 | 设计代理周围的完整基础设施 |
| **主要焦点** | 使用自然语言而非代码编写指令 | 管理模型周围的完整信息环境 | 构建具有安全、监控和控制机制的生产级系统 |
| **核心问题** | "我该如何措辞？" | "模型需要知道什么？" | "代理如何在数千次推理中可靠运行？" |
| **范围** | 单次交互 | 跨多轮的系统级信息流 | 跨数天或数周的多步系统 |
| **关键组件** | 指令、主要内容、示例、提示、支持内容、思维链、温度参数 | 系统指令、记忆管理、检索信息、工具编排、输出结构化、查询增强 | 上下文工程、架构约束（linters、结构测试）、垃圾回收（定期代理） |
| **最佳用例** | 简单任务：摘要、翻译、问答、内容生成 | 需要对话记忆、多信息源、长期任务的复杂工作流 | 涉及客户记录、财务数据、合规工作流的生产系统 |
| **失败点** | 模糊措辞、脆弱措辞、静默失败 | 错误文档、过时信息、上下文溢出、上下文腐烂 | 通过 Harness 设计预防失败 |
| **调试方法** | 语言优化 | 数据架构工作 | 将代理失败视为改进 Harness 的信号 |
| **性能影响** | 重新排序示例可产生超过 40% 准确性波动 | 针对 LLM 约束优化 token 效用 | Harness 配置改进解决率 64%；同一模型 2% vs 12%（6 倍差距） |
| **生产适用性** | 有限——脆弱、难版本控制、难测试、维护负担 | 中等——管理信息流但需要额外基础设施 | 高——专为生产设计，具有安全护栏和监控 |
| **关系** | 上下文工程的子集 | Harness 工程的子集 | 包含上下文工程及其他一切：预防、测量、控制、修复 |
| **真实案例** | 营销草稿创建、客户支持回复建议 | 具有记忆和工具访问的 AI 代理 | OpenAI 100 万 + 行代码产品；Stripe 每周 1,300 个 AI 编写 PR |

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

这并不意味着工程师变得不那么技术化。相反，Harness Engineering 需要更深的架构思维——你正在设计必须在没有持续干预的情况下工作的系统。

### 重要技能

基于 NxCode 构建 AI 驱动产品的经验：

1. **系统思维** - 理解约束、反馈循环和文档如何交互
2. **架构设计** - 定义可执行且富有成效的边界
3. **规范编写** - 精确表达意图以便代理执行
4. **可观察性** - 构建揭示代理行为模式的监控
5. **迭代速度** - 快速测试和完善 Harness 配置

### 实际有效的模式

1. **仓库优先文档** - 每个架构决策、命名约定和部署流程都在仓库中。没有东西存在于 Slack 或 Google Docs 中。

2. **增量约束构建** - 从基本 linting 开始，随着模式出现添加架构约束，不要试图 upfront 设计完美的 Harness。

3. **代理特定审查清单** - AI 生成的代码具有与人类代码不同的失败模式。审查流程考虑常见代理模式（过度抽象、不必要的错误处理、文档漂移）。

4. **多提供者 Harness 设计** - Harness 适用于 Claude、GPT 和 Gemini 模型。提供者无关设计意味着我们可以切换模型而无需重建整个系统。

---

## 关键结论

### 核心洞察

1. **模型是商品，Harness 是护城河**
   - LangChain 仅通过改变 Harness 就从基准 Top 30 跃升至 Top 5
   - 同一模型在不同 Harness 中表现差异可达 6 倍

2. **约束创造自由**
   - 限制代理的解决方案空间矛盾地提高生产力
   - 清晰边界让代理更快收敛到正确方案

3. **三层叠加策略**
   - 从提示开始获取快速收益
   - 工作流复杂时添加上下文工程
   - 生产部署前实施 Harness 基础设施

4. **工程师角色演变**
   - 从编写代码到设计环境
   - 从调试代码到调试代理行为
   - 从编写测试到设计测试策略

### 行动建议

**立即开始（Level 1）：**
1. 设置 4 行 CLAUDE.md——写下你最常纠正代理的行为
2. 设置权限白名单——首先拒绝危险操作（rm -rf*、Edit(.env*)、git push --force）
3. 尝试一次计划模式——下次涉及超过 3 个文件的任务时，先进入计划模式

**中期目标（Level 2）：**
1. 创建 AGENTS.md 包含团队约定
2. 实施 CI 强制执行的架构约束
3. 建立代理生成 PR 的审查清单

**长期愿景（Level 3）：**
1. 构建自定义中间件层
2. 集成可观察性系统
3. 部署熵管理代理
4. 建立 Harness 版本控制和 A/B 测试

### 最终思考

AI 编码工具的终局不是关于人类不写代码。而是关于人类只做值得做的决定。配置、权限、规则、边界——这些不是开销。它们是你保持控制的方式。

**真正的障碍不是你能否使用 AI 代理。而是你能否让它保持克制。**

---

## 参考资料

1. NxCode. "Harness Engineering: The Complete Guide to Building Systems That Make AI Agents Actually Work (2026)"
2. Epsilla. "The Third Evolution: Why Harness Engineering Replaced Prompting in 2026"
3. Dev.to. "Prompt Engineering vs Context Engineering vs Harness Engineering: What's the Difference in 2026?"
4. Martin Fowler. "Harness Engineering"
5. LangChain Blog. "Context Engineering for Agents"
6. Mitchell Hashimoto. "My AI Adoption Journey"
7. PureAI. "Karpathy Puts Context at the Core of AI Coding"
8. OpenAI. "Harness engineering: leveraging Codex in an agent-first world"

---

*报告生成时间：2026-03-27 11:30 CST*
