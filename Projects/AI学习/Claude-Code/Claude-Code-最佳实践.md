---
date: 2026-03-27
tags: [claude-code, ai-engineering, best-practices, productivity]
status: active
source: OpenClaw
---

# Claude Code 最佳实践：从工具到系统

我使用 Claude Code 已经一年多了，犯过的错误足够写成一本书。

刚开始使用时，"vibe coding"这个词还不存在。我以为它只是终端里的聊天机器人——输入几行代码，得到一些代码，完成。后来我意识到：同样的工具，配置得当与完全不配置，表现起来完全是两回事。

这篇文章是我梳理一年经验的成果。读完之后，你将获得：一个 4 行的全局配置模板、一个权限白名单策略、一个让 AI 三思而后行的工作流程，以及一个用 Hooks 将软规则转化为硬约束的方法。

其中一些来自痛苦的错误，另一些来自比我应该做的更仔细地阅读文档。

---

## 1. CLAUDE.md — 保持简短

我见过一个 800 行的 CLAUDE.md，列出了每个文件夹的用途、每个 API 的参数、每个边界情况的处理方式。作者确实付出了努力，但弊大于利。

Claude 的注意力是有限的。上下文中较早的部分比较晚的部分更有分量。我的经验法则是保持单个 CLAUDE.md 简短，通常在 200 行以内。官方文档说 CLAUDE.md 文件会完整加载，但较短的文件会产生更好的遵循度。任何更长的内容会消耗上下文并降低合规性。溢出的内容应该放入 .claude/rules/ 或使用基于路径的规则。

你的 CLAUDE.md 越长，Claude 遵循它的倾向就越差。我使用 200 行作为个人上限，而不是文档化的硬性限制。

**应该放什么：** Claude 无法从代码中读取的"为什么"。架构决策、历史教训、业务约束、风格偏好。

**不应该放什么：** API 文档、逐文件的描述。Claude 会自己阅读代码。

**我的全局 CLAUDE.md 只有 4 行：**

```markdown
- commit messages use conventional commits
- explain your plan before making changes, wait for my confirmation
- prefer simple solutions, no over-engineering
- don't refactor files I didn't mention
```

就这 4 行，为我节省了数百次"不，那不是我的意思"。

**在项目级别，我放入项目事实：**

```markdown
# Ship Faster

## Architecture
- Next.js 16 App Router + Supabase + Stripe + Vercel

## Off-limits
- Don't modify existing files in supabase/migrations/
- Don't delete .env.local
- Run stripe listen before changing payment webhooks

## Current focus
- Refactoring onboarding flow, goal: fewer steps from signup to first payment
```

**更进一步，我将规则拆分到 .claude/rules/ 中，使用 YAML frontmatter 进行路径匹配：**

```yaml
---
paths: ["src/api/**/*.ts", "src/routes/**/*.ts"]
---
- All API handlers must include input validation
- Error responses use { error: string, code: number } format
- Every new endpoint needs an integration test in tests/api/
```

这条规则只在 Claude 触及 API 目录时激活。不会干扰其他文件，节省上下文空间。

**层级结构：** Claude Code 从多个级别加载指令，大致按以下优先级顺序：托管指令（企业级）、项目指令（./CLAUDE.md 或 ./.claude/CLAUDE.md）、用户指令（~/.claude/CLAUDE.md）、.claude/rules/ 和 ~/.claude/rules/。还有递归目录加载，所以它不是简单的四层堆栈。

我的实际方法很简单：全局用于行为偏好，项目用于项目事实，规则按路径拆分到 .claude/rules/。三层，各司其职，不混用。

---

## 2. 子代理 — 上下文隔离

没有子代理，你的上下文窗口在复杂项目上会迅速填满。运行测试、分析日志、搜索大型代码库，这些操作会产生大量输出。把所有这些都倾倒到主会话中，上下文质量会急剧下降。

子代理的核心价值是上下文隔离。每个子代理都有自己的上下文窗口。输出保留在子代理中，只有摘要返回给你。Claude Code 支持具有可配置工具权限和角色描述的自定义子代理。当描述匹配时，Claude 会自动将任务委托给子代理，每个子代理都在自己隔离的上下文中运行。

**我使用三个代理模板：** explorer（只读，快速代码库搜索）、planner（只读，研究和分析）、executor（全功能，复杂任务）。

**自定义代理放入 .claude/agents/：**

```yaml
---
name: code-reviewer
description: Code review agent, read-only
tools: Read, Grep, Glob, Bash
model: sonnet
---
You are a code reviewer. Check for: security vulnerabilities, performance issues, maintainability.
Only report problems. Don't modify code, don't suggest refactors.
```

注意：子代理默认继承父会话的工具。如果你想要一个真正的只读代理，你需要显式设置工具白名单。仅仅在描述中写"只读"不会限制工具访问。

**我最反直觉使用子代理的方式：不是写代码，而是模拟用户。**

我设置了 6 个 AI 角色，扮演不同类型的找茬者，每个都作为子代理任务委托。15 分钟后，他们发现了 5 个我自己使用系统两周都没发现的问题。

我构建了一个 AI 配置系统并使用了两周。感觉挺稳固的。但我不确定其他人会如何体验它。所以我创建了 6 个子代理，每个扮演完全不同的用户角色：

- **怀疑的资深工程师：** 10 年经验，默认不信任新工具。专注于安全性、副作用、隐藏依赖。
- **安全审查员：** 安全审计背景。专注于秘密泄露、权限边界、远程执行风险。
- **新维护者：** 刚接手项目，零背景知识。专注于文档清晰度、入门路径。
- **重度 CLI 用户：** 命令行高级用户。专注于命令一致性、可组合性、幂等性。
- **操作员/SRE：** 负责生产运营。专注于可观察性、故障恢复、通知管道。
- **文档优先新手：** 逐步遵循文档，不阅读源代码。专注于 README 完整性、示例准确性。

我将所有 6 个角色任务委托给子代理。15 分钟后，汇总结果揭示了 5 个信任断点：

1. 一个命令静默失败。新维护者不知道出了什么问题。
2. README 缺少"我首先应该做什么"的 30 秒快速入门。
3. 一个功能的描述与其实际行为不符。重度 CLI 用户立即失去信任。
4. 状态命令不是真正的只读。安全审查员首先发现了它。
5. 通知管道没有活性探测。操作员拒绝将其视为健康检查。

在修复每一个之后，同样的 6 个代理进行了第二轮测试。所有高优先级信任断点都清除了。剩余问题降为措辞和可观察性调整。

这种测试手动需要一周时间。AI 在 15 分钟内运行了两轮完整测试。

这不仅仅是一种测试技术。这是一种新的质量保证方法：让 AI 扮演你的用户，在发布前发现问题。

---

## 3. 指令 — 目标、约束、上下文

我曾经给出这样的指令："帮我优化这个函数"。

Claude Code 将函数拆分成三个文件，添加了一个工厂类，并更改了导入路径。我花了二十分钟审查，才意识到这些都不是我想要的。

问题在于指令模糊。好的指令只需要三个要素：目标、约束、上下文。

```plaintext
Goal: Reduce getUserData() response time from 800ms to under 200ms
Constraints: Don't change the interface signature, don't add new dependencies, only touch this one function
Context: @src/services/user.ts#L45-L89
Verification: user.test.ts must all pass
Don't: Don't change the database schema, don't add a caching layer
```

带行号的 @ 引用让 Claude 保持精确聚焦。对于 UI 问题，我直接用 Ctrl+V 粘贴截图，比用文字描述布局准确十倍。

**"不要"比"要"更重要。**

"不要添加新依赖"比列出 10 个允许的库更有效。Claude 有时会过于热心。你让它修复一个 bug，它也会重构周围的代码。说"只改这个函数"能省很多麻烦。

探索实际上受益于模糊指令。"看看这个模块有什么潜在问题"能捕捉到"检查这 5 个具体方面"会遗漏的东西。实施时切换回精确模式。

---

## 4. 计划模式 — 三思而后行

我曾经告诉 Claude Code 直接重构一个 600 行的路由文件。它做到一半，引入了循环依赖，试图修复循环依赖，让一切变得更糟。我按下 Ctrl+C，失去了半小时的上下文。

从那以后，我对复杂任务总是使用计划模式。

**三种进入方式：**

```bash
# 启动时
claude --permission-mode plan

# 会话期间（按 Shift+Tab 直到出现计划模式）
Shift+Tab  # 重复直到出现计划模式

# 自然语言
"Don't change any code yet, give me an implementation plan"
```

在计划模式下，Claude 只输出计划而不执行任何内容。关键动作：按 Ctrl+G 在编辑器中打开并编辑计划文本。删除不必要的步骤，添加约束，确认，然后 Claude 执行更新后的计划。

更改计划只需一句话。更改已经写好的代码需要十倍长的时间。

**标准四步工作流程：**

```plaintext
Explore（Claude 阅读代码）→ Plan（输出计划）→ Implement（逐步执行）→ Verify（运行测试）
```

**什么时候不需要计划模式：** 只有一种可能实现方式的任务。重命名变量、修复拼写错误、添加日志行。不需要绕弯。计划模式适用于有多种选择的任务。

---

## 5. 上下文管理 — 保持干净

有一次我做到一半，运行 /context 看到 78% 的使用率。然后 Claude 突然开始重复我们已经拒绝的方法，就像失忆了一样。就在那时，我把 /compact 变成了常规习惯。

Claude Code 支持非常大的上下文窗口，但塞满的上下文意味着质量急剧下降。

塞满的上下文失败不是因为超过限制。失败是因为注意力分散。

**五种管理策略：**

1. `/clear`：清除一切，完全切换任务时使用。
2. `/compact [focus]`：压缩历史记录同时保留关键点。继续同一任务时使用。
   ```plaintext
   /compact keep: current task goal, completed steps, pending issues
   ```
3. `/context`：检查当前上下文使用情况。了解你的处境。
4. 子代理：单独的模块使用单独的代理。不要污染主会话。
5. 压缩保留指令：在 CLAUDE.md 中告诉 Claude 压缩时要保留什么。
   ```markdown
   # Compression instructions
   When compressing context, always keep:
   - Complete list of modified files
   - Test commands and results
   - Key architecture decisions
   ```

Claude Code 这里有两个机制：CLAUDE.md 文件和自动内存。在 CLAUDE.md 中，我将项目指令视为共享项目事实和团队标准，用户指令（~/.claude/CLAUDE.md）作为跟随我跨项目的个人偏好。自动内存在 ~/.claude/projects/<project>/memory/ 中，会话开始时加载 MEMORY.md 的前 200 行。输入 /memory 查看已加载的 CLAUDE.md 文件，管理自动内存，并编辑记忆条目。在一些客户端中，我也看到 # 打开快速编辑器，但我将其视为特定于版本的功能，而不是文档化的默认功能。

---

## 6. 权限 — 只读就是只读

有一天，Claude 在修复一个 bug 时，随意删除了一个"看起来未使用"的配置文件。该文件是生产环境的密钥参考条目。

之后我花了半小时配置 .claude/settings.json：

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf*)",
      "Bash(*--force*)",
      "Edit(.env*)",
      "Edit(*.pem)"
    ],
    "ask": [
      "Bash(git push*)",
      "Bash(npm publish*)",
      "Edit(src/config/*)"
    ],
    "allow": [
      "Bash(npm run*)",
      "Bash(git commit*)",
      "Bash(git add*)",
      "Read(**)"
    ]
  }
}
```

优先级：deny > ask > allow。deny 最高。如果命令匹配 deny，即使 allow 也匹配，它也不会执行。

设置文件有自己的层级：托管（组织策略）、CLI 参数、本地（.claude/settings.local.json，不提交）、项目（.claude/settings.json，共享）、用户（~/.claude/settings.json）、默认值。一个重要的细微差别：数组值的设置（如权限列表）跨范围合并而不是相互替换。所以用户级别的 deny 规则和项目级别的另一个规则都适用。

我在项目级别拒绝业务风险，如删除迁移。我在全局级别拒绝安全基线，如执行 rm -rf。两层，独立维护。

只读就是只读。没有"顺便"。我的第一个版本的状态命令秘密调用了 sync，理由是"检查时不妨同步一下"。运行 6 个角色测试后，安全审查员立即发现了："你的只读脚本有副作用。我不能信任它的输出。"副作用会摧毁可信度。

---

## 7. Hooks — 将规则转化为约束

我在 CLAUDE.md 中写了"提交前运行 lint"。Claude 有时会跳过它。文件中的规则是建议。AI 有时会决定"这种情况很特殊"并忽略它们。

**CLAUDE.md 中的规则是建议。Hooks 是代码级别的强制，永远不会忘记。**

**四种 Hook 类型：**

```plaintext
command                执行 shell 命令并从 stdin 读取 JSON。
http                   向 URL 发送 POST 请求。
prompt                 进行单次 LLM 调用进行是/否判断。
agent                  生成子代理进行验证。
```

**三个实际场景：**

**自动格式化：** 每次编辑后运行 Prettier。

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{"type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"}]
    }]
  }
}
```

**保护关键文件：** 阻止对生产配置的修改。

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{"type": "command", "command": ".claude/hooks/protect-files.sh"}]
    }]
  }
}
```

退出代码 0 表示允许，退出代码 2 表示阻止。但只有具有决策控制的事件才能真正阻止操作。PreToolUse 可以在工具调用运行之前阻止它。PostToolUse 不能，因为工具已经执行。选择使用哪个 Hook 事件时要记住这一点。

**压缩后重新注入：** 当长对话被压缩时，重要指令可能会丢失。Hook 会自动重新注入它们：

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "compact",
      "hooks": [{"type": "command", "command": "echo 'Use Bun, not npm. Run bun test before committing.'"}]
    }]
  }
}
```

Hooks 的真正价值是将工程标准转化为系统约束。自动格式化只是额外奖励。人类会忘记规则。代码不会。

---

## 8. 自动学习规则 — 高级实验

> 这是一个高级实验，不是必需的。如果你只是想用好 Hooks，前一章就足够了。

反向方法：让 AI 从我的行为中提取规则。流程：

```plaintext
AI 写草稿 → 我编辑到满意 → observe.py 记录原始/最终版本
→ improve.py 对比两个版本 → 提取"我改变了什么"
→ 规则按置信度排名 P0/P1/P2
→ 人工审查后，P0 规则写回 SKILL.md
```

```python
# observe.py: 记录 AI 草稿和我的最终版本
python3 observe.py record-original draft.md
# ...我进行编辑...
python3 observe.py record-final final.md
```

```python
# improve.py: 从 diff 中提取规则
python3 improve.py auto --skill .
# 自动对比，提取变更模式，生成规则候选
# P0（出现 5+ 次）推送人工确认后写入
# P1 低频，归档
# P2 仅记录
```

结果：48 条规则，740 行。循环越多，AI 写得越像我。

我还设置了一个 cron 作业每晚自动运行提取，无需手动触发：

```bash
0 23 * * * python3 ~/.ai-core/skills/scripts/improve.py auto --skill .
```

**局限性：自动学习不是魔法**

核心安全规则，如权限边界、秘密保护、破坏性命令阻止，必须手写。不能将这些委托给自动化。

偶然编辑可能被错误提取为规则。有一次我临时更改了文件缩进，因为那个特定文件的风格不一致。自动学习者将其提取为"始终使用 2 空格缩进"。这条规则是错误的，因为项目标准是 4 空格。

所有 P0 规则在写入前必须经过人工审查。确认它们反映真实的偏好，而不是一次性编辑。自动学习仅处理风格偏好。核心安全规则必须手写。

---

## 9. Worktrees — 安全实验

我以前在重构前犹豫不决。万一搞砸了怎么办？Git 分支共享工作目录。切换分支会影响当前状态。

Worktrees 不同。每个分支都有自己独立的工作目录：

```bash
# 创建实验 worktree
claude --worktree experiment/refactor-auth

# 或手动
git worktree add ../project-experiment feature/new-auth
```

Claude 可以在实验目录中做任何它想做的事。主目录保持不受影响。如果不行，git worktree remove，零成本。

一个 worktree 花费几秒钟。一个失败的实验花费数小时。

**Git 规则：**

1. 不要让 Claude 自动推送。它可以提交，但推送需要我的确认。一旦推送到远程，回滚代价高昂。
2. 频繁提交。每个原子步骤后提交。/rewind 可以回滚到任何检查点。
3. 注意破坏性操作。git reset --hard、git push --force、rm -rf。批准前想清楚后果。这些操作没有撤销。
4. 从 PR 恢复上下文。claude --from-pr 123 自动加载 PR 的更改和讨论。

所有探索性任务、架构重构和不确定的大型更改都通过 worktrees 进行。

---

## 10. 努力级别 — 匹配任务复杂度

思考默认开启。Claude Code 总是进行一定程度的推理。但努力级别不应该总是最大化。在简单任务上运行高强度是纯粹的 token 浪费，推理过程会消耗上下文，影响后续对话。

在简单任务上高努力是纯粹的 token 浪费。推理过程也会消耗上下文，影响后续对话质量。

有一次我在设计一个复杂的权限系统，涉及多个角色、继承和动态规则。默认努力产生了一个有明显漏洞的计划。切换到 /effort high 后，它思考了近一分钟，带回了一个完全不同的架构，涵盖了我没有考虑到的边界情况。

```bash
# 调整推理深度
/effort high    # 更深的推理
/effort low     # 简单任务，节省 token
```

截至撰写本文时，控制推理深度的文档化方式是 /effort。在实践中，输入 ultrathink 有时可以促使 Claude 在当前回合进行更深的推理，但我将其视为非正式的提示提示，而不是保证的规则。

我的规则：只有当更改涉及超过 3 个文件时才提高努力。单文件编辑，默认即可。架构设计、复杂 bug 诊断、需要多约束推理的任务，这些值得花费 token。

---

## 11. MCP — 节制使用

MCP（Model Context Protocol）让 Claude Code 可以连接到外部服务，如数据库、支付平台和 GitHub。安装一个 MCP 服务器，Claude 就可以在对话中直接操作该服务。

每个 MCP 服务器都会在你的上下文中添加工具定义，开销会累积。根据我的经验，每个服务器大约消耗 100 到 500+ token，尽管确切数字会有所不同。我尝试同时运行 8 个。有效上下文明显下降，代码生成质量变差。

每个额外的 MCP 服务器意味着更少的有效上下文。按项目需要添加，完成后删除。

我只保留 2 到 3 个：

```bash
claude mcp add supabase -- npx @supabase/mcp-server-supabase --project-ref $REF
claude mcp add stripe -- node /tools/stripe-mcp/index.js
```

.mcp.json 配置：

```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": ["@supabase/mcp-server-supabase", "--project-ref", "xxxxx"]
    },
    "stripe": {
      "command": "node",
      "args": ["/tools/stripe-mcp/index.js"]
    }
  }
}
```

连接 Supabase 和 Stripe 后，我只需在对话中说"检查过去 7 天的失败支付"。Claude 直接查询 Stripe，返回结果并分析。过去需要编写一次性脚本。现在只是对话而已。

使用 /mcp 管理服务器连接并禁用不需要的连接。我将每个服务器的上下文开销视为实用启发式方法，而不是每个客户端都暴露的文档化 /mcp 读取。

---

## 12. 双会话审查 — 作者/审查者模式

三个常见问题：

1. **过度工程化：** 让它写一个简单的工具函数，它构建了一个带有泛型、接口和工厂模式的完整抽象层。修复：在 CLAUDE.md 中写"prefer simple solutions"。
2. **幻觉 API：** 使用不存在的 SDK 方法。名称看起来合理，文档说明不然。修复：运行测试。
3. **范围蔓延：** 更改一个函数，它触及五个文件。修复：明确说"只改 X，不要碰其他任何东西"。

**作者/审查者双会话模式：** 一个会话写，一个会话审查。角色分离，无干扰。

一个会话写代码，另一个审查。审查者不知道代码是如何生成的。它只看到结果并提出问题。两个会话，角色分离，无交叉污染。

```plaintext
# Reviewer session instructions
You are a code reviewer. Does this code have:
1. Hallucinated APIs (methods that don't exist)
2. Race conditions
3. Changes outside the task scope
Only report problems. Don't suggest fixes.
@src/auth/service.ts
```

审查者发现的问题返回给作者修复。通常 2 到 3 轮，质量比单会话好得多。

---

## 13. 四条红线 — 保护你的判断

有一次我让 Claude Code 配置一个我一无所知的 Kubernetes 集群。它工作了。它运行了。我什么都不懂。两周后出了问题，我不知道从哪里开始。这是我犯过的最昂贵的错误之一。

Claude 可以生成任何代码，但如果你不理解它，你就无法接手。

**四条红线：**

1. **不要做你不理解的事。** Claude 可以生成任何东西，但如果你读不懂它，你就无法维护它。你不理解的生产代码既是技术债务也是安全风险。
2. **不要在没有版本控制的情况下使用它。** 没有 Git 意味着无法回滚。Claude 的更改可能会覆盖你的文件。没有版本控制，你就是在盲目飞行。
3. **不要给它扔大规模任务。** "重构整个认证系统"是灾难性提示。"将 JWT 解析从 validateToken 提取到它自己的函数中"是正确的方法。较小的任务更准确，范围保持可控。
4. **不要期望第一次就完美。** 初稿总是草稿。/rewind 回滚，精确反馈哪里出错了。迭代是正确的方法。

这四条红线不是关于限制 AI。它们是关于保护你的判断。你每一个不理解就交出的决定都会让你失去对系统的控制。

---

## 14. 多系统协作 — 仅限高级用户

> 如果你只使用一个 AI 工具且不运行自动化服务器任务，本章对你几乎没有价值。直接跳到结论。这涵盖了我将 Claude Code 放入更大工作流程后所做的事情。不是一般做法。

我遇到的问题是具体的：Claude Code 和 Codex 各自维护自己的规则。我在一方更新了提示，忘了同步另一方，三周后两个系统生成的代码风格和决策逻辑不同。我学到的是：唯一需要单一事实来源的是知识层。我让 ~/.ai-core 成为唯一来源，仅包含共享行为规则、共享命令和共享技能。每个系统从那里生成自己的配置。

**我当前的三系统拆分：**

```plaintext
OpenClaw: 调度、通知、Telegram 交互
Codex: 重型重编码、跨文件重构
Claude Code: 快速编辑、探索、交互式调试
```

关键是清晰的边界。共享知识层，不共享运行时层。权限设置、认证凭据、MCP token、本地客户端状态，每个系统管理自己的。把这个边界弄对，多系统协作变得稳定。弄错只会放大复杂性。

---

## 从这里开始

1. **设置一个 4 行的 CLAUDE.md。** 写下你最常纠正 Claude 的行为。提交格式、更改前计划、不要过度工程化。一次编写，永远有效。
2. **设置权限白名单。** 首先拒绝危险操作。至少添加三个：rm -rf*、Edit(.env*)、git push --force。你不是在防御恶意，你是在防御 Claude 的热心。
3. **尝试一次计划模式。** 下次面对涉及超过 3 个文件的任务时，先进入计划模式。查看计划，然后确认执行。

AI 编码工具的终局不是关于人类不写代码。而是关于人类只做值得做的决定。配置、权限、规则、边界，这些不是开销。它们是你保持控制的方式。

真正的障碍不是你能否使用 Claude Code。而是你能否让它保持克制。

---

## 核心快捷键和命令（2026 年 3 月）

**键盘快捷键：**

| 快捷键 | 操作 |
|--------|------|
| Ctrl+C | 取消当前输入或生成 |
| Ctrl+G | 在默认文本编辑器中打开 |
| Ctrl+V | 从剪贴板粘贴图片 |
| Esc + Esc | 回滚或总结 |
| Option+T / Alt+T | 切换扩展思考 |
| Shift+Tab | 循环权限模式 |

**Slash 命令：**

| 命令 | 描述 |
|------|------|
| /compact [focus] | 压缩对话，可选聚焦指令 |
| /clear | 清除对话历史并释放上下文 |
| /context | 可视化当前上下文使用情况 |
| /memory | 编辑 CLAUDE.md 记忆文件并管理自动内存 |
| /rewind | 回滚到之前的检查点或从选定消息总结 |
| /resume | 恢复之前的对话 |
| /effort [level] | 设置模型努力级别 |
| /init | 用 CLAUDE.md 指南初始化项目 |
| /mcp | 管理 MCP 服务器连接和 OAuth 认证 |
| /cost | 显示 token 使用统计 |

**注意：**

- `/rewind` 已经救了我多次。如果你犯了错误，回滚一步并重新发出指令。
- `/init` 对新项目很棒。它分析你的代码库并生成 CLAUDE.md 草稿。

---

大多数教程专注于编写更好的提示。我认为真正的杠杆在于没人阅读的文件——CLAUDE.md、settings.json、hooks。哪一个配置更改对你产生了最大的影响？
