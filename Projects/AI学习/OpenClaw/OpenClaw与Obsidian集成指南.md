---
date: 2026-03-26
tags: [openclaw, obsidian, pkm, knowledge-management]
status: active
source: Twitter/X @sitinme
type: guide
---

# 我把 OpenClaw 所有内容丢进了 Obsidian，再也不怕丢信息了！

AI Agent 现在生成内容分分钟的事，也打通了内容沉淀和发布的各个平台，不得不说最好的沉淀知识资产的方式还是文档，一切要落到文档才行。

我会把 OpenClaw Agent 产出的所有信息，全部同步到 Obsidian 里面。Notion 也同步，但 Obsidian 作为本地知识库，查起来更快，离线也能用，数据完全在自己手上。

我用 OpenClaw 跑了好几个 Agent，每天从各种渠道收集信息、写内容、做分析。这些东西如果只停留在 Telegram 的聊天记录里，过几天就找不到了。

这里有个技巧，就是大家一定要把跟 AI 聊天的东西同步到 Notion 里面，同时再同步到 Obsidian 里面。这样就可以避免后续信息出问题，查起来也方便。

我直接买了 Obsidian 的付费会员，把 OpenClaw 的所有内容丢进 Obsidian。Obsidian 的双链笔记能力在这里很关键，Agent 写进去的每一条信息都可以通过 `[[wikilinks]]` 和其他笔记建立联系，时间长了会自然形成一张知识网络。

你看某条旧笔记的时候，Obsidian 会告诉你还有哪些笔记引用了它，这种反向链接的能力是纯文档工具没有的。

Obsidian 的 Vault 本质上就是一个本地文件夹，里面全是 Markdown 文件。OpenClaw 操作它主要靠两个东西。

## 第一个是 obsidian skill

OpenClaw 的 skill 机制可以理解为给 Agent 一本操作手册，装了 obsidian skill 之后，Agent 就知道该怎么组织笔记结构、怎么用 frontmatter 管理元数据、怎么建立双链。

配合 OpenClaw 自带的 write 工具，Agent 就能直接读写 Vault 里的 Markdown 文件。你在 Telegram 跟 Agent 说一句"把这些信息同步到 Obsidian"，它就会自动找到你的 Vault 路径，创建对应的文件和目录结构。

## 第二个是 obsidian-cli

一个用 Go 写的命令行工具。它补充了纯文件操作做不到的事情，最重要的一个能力是维护 wikilinks 的完整性。当你重命名或移动笔记的时候，它会自动扫描整个 Vault，更新所有引用了这篇笔记的链接。

直接操作文件系统做不到这一点，链接一断，知识网络就散了。

安装用 Homebrew：

```bash
brew tap yakitrak/yakitrak
brew install yakitrak/yakitrak/obsidian-cli
```

装好之后设一下默认 Vault：

```bash
obsidian-cli set-default "<你的 vault 名>"
```

之后 Agent 就可以用 `obsidian-cli move` 来移动笔记并自动更新链接，用 `obsidian-cli search-content` 来全文检索。日常大部分操作，Agent 直接写文件就够了，obsidian-cli 主要用在需要维护链接的场景。

## 我的做法

用 OpenClaw 直接从 Telegram 到飞书、Notion、Obsidian，所有知识库打通，以后再也不怕丢信息了。

我的 Agent 在 Telegram 里运行，每天做信息收集、内容创作、数据分析这些活。所有产出，Agent 会自动按类别写入 Obsidian 对应的目录。比如我的 Vault 里有"策略规划"、"核心商业目标"、"网站架构"这些分类，Agent 根据内容判断该放到哪里。

整个过程就是你在 Telegram 聊完一个话题，Agent 把关键信息整理成结构化的 Markdown 笔记，同步到 Obsidian。你打开 Obsidian 就能看到最新的内容，想改就改，想加标签加链接随意。Agent 后续做任务的时候，也能反过来查 Obsidian 里的历史记录。相当于给 Agent 装了一个长期记忆，它不会因为会话结束就把之前的上下文全忘了。

如果想更进一步，可以用 OpenClaw 的 Heartbeat 定时任务做自动同步。比如每隔一段时间扫描特定标签的笔记，把更新推到 Notion 或其他平台。OpenClaw 的 cron 系统支持独立会话执行，结合 `sessions_send` 做跨 Agent 协调，能搭出比较灵活的同步链路。

也有人用 Git 来做 workspace 的版本管理，每隔几分钟自动 commit 一次，相当于给整个知识库加了一层版本历史。

## 使用感受

用了一段时间下来，最大的感受是 Obsidian 的本地优先设计在这个场景下太合适了。Agent 的所有产出在你本地有一份完整备份，不依赖任何云服务。如果你用 Obsidian Sync（付费功能），还能在手机上随时翻，多设备同步也很方便。

另外建议给 Agent 的 Obsidian skill 里加上明确的目录结构规范。不然 Agent 可能随意创建文件，时间长了 Vault 会变乱。在 `AGENTS.md` 里写清楚笔记放哪个目录、用什么命名规则、frontmatter 需要哪些字段，Agent 就会按规矩来。

---

如果你对 OpenClaw 感兴趣，或者在实践中遇到问题，欢迎加入 OpenClaw 中文交流群。

49 元入群，送 $50 算力额度。群里不卖课，都是实际在用 OpenClaw 的玩家，每天分享使用技巧和踩坑经验，氛围很活跃。

---

**原文链接：** https://x.com/sitinme/status/2026169692348727389
