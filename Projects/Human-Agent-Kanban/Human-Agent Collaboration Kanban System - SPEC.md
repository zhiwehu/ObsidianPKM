# Human-Agent Collaboration Kanban System - SPEC

> 人类与 AI Agents 协作的项目看板系统规范文档

**版本**: v1.2  
**日期**: 2026-03-22  
**状态**: 草案

---

## 更新日志

| 版本 | 日期 | 更新内容 |
|-----|------|---------|
| v1.0 | 2026-03-22 | 初始版本 |
| v1.1 | 2026-03-22 | 移除 Webhook，改用 HTTP API 触发 Agent |
| v1.2 | 2026-03-22 | 新增任务依赖管理、任务队列、阻塞状态机制 |

---

## 1. 项目概述

### 1.1 项目背景

构建一个类 Trello 的项目管理看板系统，支持人类用户和 AI Agents 作为平等的"团队成员"共同完成项目开发。AI Agents 具有独立身份，能够：

- 接收任务分配
- 主动创建任务
- 更新任务状态
- 与人类协作
- 自动执行工作流程

### 1.2 核心价值

- **降低人力成本**: AI Agent 处理重复性任务
- **24/7 工作**: Agent 不知疲倦，可全天候工作
- **透明协作**: 人类和 Agent 的工作同样可见可追溯
- **灵活调度**: 根据任务类型智能分配给人或 Agent

### 1.3 目标用户

| 用户类型 | 使用场景 |
|---------|---------|
| 项目经理 | 分配任务、跟踪进度、管理团队 |
| 开发人员 | 认领任务、更新状态、协作开发 |
| AI Agents | 执行任务、汇报进度、主动发现问题 |
| 团队管理员 | 管理成员、配置权限、管理 Agents |

---

## 2. 用户故事

### 2.1 任务管理

| 编号 | 用户故事 | 优先级 |
|-----|---------|--------|
| US-001 | 作为项目经理，我可以创建任务卡片并分配给人类或 Agent | P0 |
| US-002 | 作为 Agent，我可以主动创建任务卡片（如发现代码问题） | P0 |
| US-003 | 作为团队成员，我可以拖拽卡片到不同列表来更新状态 | P0 |
| US-004 | 作为任务负责人，我可以更新任务描述、截止日期、标签 | P0 |
| US-005 | 作为团队成员，我可以在任务下评论 @相关人员 | P1 |
| US-006 | 作为项目经理，我可以将任务指派给多个成员（负责人+协助者） | P1 |

### 2.2 Agent 集成

| 编号 | 用户故事 | 优先级 |
|-----|---------|--------|
| US-007 | 作为管理员，我可以注册 Agent 并分配 API Token | P0 |
| US-008 | 作为 Agent，我可以定时拉取分配给我的任务 | P0 |
| US-009 | 作为 Agent，我完成任务后自动更新卡片状态 | P0 |
| US-010 | 作为 Agent，我遇到问题时可以创建新任务或 @人类求助 | P1 |
| US-011 | 作为管理员，我可以配置 Agent 的自动行为规则 | P2 |

### 2.3 看板管理

| 编号 | 用户故事 | 优先级 |
|-----|---------|--------|
| US-012 | 作为管理员，我可以创建/编辑/删除看板 | P0 |
| US-013 | 作为成员，我可以查看我有权限的看板列表 | P0 |
| US-014 | 作为管理员，我可以管理看板成员和权限 | P1 |
| US-015 | 作为成员，我可以订阅看板动态通知 | P2 |

---

## 3. 功能需求

### 3.1 核心功能

#### 3.1.1 看板 (Board)

| 功能 | 描述 | API |
|-----|------|-----|
| 创建看板 | 创建新看板，指定名称、描述、可见性 | `POST /api/boards` |
| 编辑看板 | 修改看板名称、描述、设置 | `PATCH /api/boards/:id` |
| 删除看板 | 删除看板（软删除） | `DELETE /api/boards/:id` |
| 看板列表 | 获取用户有权限的看板列表 | `GET /api/boards` |
| 看板详情 | 获取看板详情（含列表和成员） | `GET /api/boards/:id` |
| 看板成员 | 管理看板成员和权限 | `GET/POST /api/boards/:id/members` |

#### 3.1.2 列表 (List)

| 功能 | 描述 | API |
|-----|------|-----|
| 创建列表 | 在看板上创建新列表 | `POST /api/lists` |
| 编辑列表 | 修改列表名称、位置 | `PATCH /api/lists/:id` |
| 删除列表 | 删除列表（级联删除卡片） | `DELETE /api/lists/:id` |
| 拖拽排序 | 调整列表顺序 | `PUT /api/boards/:id/lists/reorder` |

#### 3.1.3 卡片 (Card)

| 功能 | 描述 | API |
|-----|------|-----|
| 创建卡片 | 在列表中创建新卡片 | `POST /api/cards` |
| 编辑卡片 | 修改卡片内容、日期、标签 | `PATCH /api/cards/:id` |
| 删除卡片 | 删除卡片 | `DELETE /api/cards/:id` |
| 移动卡片 | 移动卡片到不同列表/位置 | `PATCH /api/cards/:id/move` |
| 拖拽排序 | 在列表内调整卡片顺序 | `PUT /api/lists/:id/cards/reorder` |
| 卡片详情 | 获取卡片完整信息 | `GET /api/cards/:id` |
| 批量创建 | Agent 批量创建卡片 | `POST /api/cards/batch` |

#### 3.1.4 成员管理

| 功能 | 描述 | API |
|-----|------|-----|
| 添加成员 | 添加人类或 Agent 为看板成员 | `POST /api/boards/:id/members` |
| 移除成员 | 移除看板成员 | `DELETE /api/boards/:id/members/:userId` |
| 修改权限 | 修改成员角色（Owner/Admin/Member/Viewer） | `PATCH /api/boards/:id/members/:userId` |
| 成员列表 | 获取看板所有成员 | `GET /api/boards/:id/members` |

#### 3.1.5 评论与活动

| 功能 | 描述 | API |
|-----|------|-----|
| 添加评论 | 在卡片下添加评论 | `POST /api/cards/:id/comments` |
| 删除评论 | 删除评论 | `DELETE /api/comments/:id` |
| 活动日志 | 获取卡片/看板活动历史 | `GET /api/cards/:id/activities` |
| @提及 | 评论中 @成员或 Agent | 支持 |

### 3.2 Agent 特有功能

#### 3.2.1 Agent 注册与管理

| 功能 | 描述 | API |
|-----|------|-----|
| 注册 Agent | 管理员注册 Agent，获取 API Token | `POST /api/agents/register` |
| 生成 Token | 为 Agent 生成/重置 API Token | `POST /api/agents/:id/token` |
| Agent 列表 | 列出所有注册的 Agents | `GET /api/agents` |
| Agent 详情 | 获取 Agent 信息和状态 | `GET /api/agents/:id` |
| 启用/禁用 Agent | 启用或禁用 Agent | `PATCH /api/agents/:id/status` |

#### 3.2.2 Agent 任务操作

| 功能 | 描述 | API |
|-----|------|-----|
| 拉取任务 | Agent 获取分配给自己的任务 | `GET /api/agents/:id/tasks` |
| 完成任务 | Agent 标记任务为完成 | `POST /api/cards/:id/complete` |
| 创建任务 | Agent 创建新任务 | `POST /api/cards` |
| 更新任务 | Agent 更新任务信息 | `PATCH /api/cards/:id` |
| 创建子任务 | Agent 创建子任务 | `POST /api/cards/:id/subtasks` |
| 求助评论 | Agent @人类求助 | `POST /api/cards/:id/comments` |

#### 3.2.3 Agent 回调（无需 Webhook）

当 Agent 完成任务后，需要回调看板 API 更新状态：

| 事件 | 描述 | 回调方式 |
|-----|------|---------|
| `task.started` | 开始执行任务 | HTTP API 调用 |
| `task.progress` | 任务进展 | HTTP API 调用 |
| `task.completed` | 任务完成 | HTTP API 调用 |
| `task.commented` | 添加评论 | HTTP API 调用 |
| `task.blocked` | 遇到问题 | HTTP API 调用 |

#### 3.2.4 Agent 自动行为

| 触发条件 | Agent 行为 |
|---------|-----------|
| 新任务分配 | HTTP API 触发 → 开始执行 |
| 定时（每 N 分钟） | 拉取待办任务 → 检查依赖 → 执行 |
| 任务评论 @ | 拉取任务 → 响应处理 |
| 任务阻塞 | 创建新任务解决阻塞 或 @人类 |

### 3.3 用户交互功能

| 功能 | 描述 |
|-----|------|
| 看板视图 | 看板概览，列表+卡片 |
| 卡片详情 | 卡片完整信息+评论+活动 |
| 任务筛选 | 按成员、标签、截止日期筛选 |
| 搜索 | 全局搜索卡片 |
| 导出 | 导出看板为 JSON/CSV |

---

## 4. 数据模型

### 4.1 实体关系

```
Board (看板)
  ├── List (列表)
  │     └── Card (卡片)
  │           ├── CardMember (卡片成员)
  │           ├── CardLabel (卡片标签)
  │           ├── Comment (评论)
  │           ├── Activity (活动日志)
  │           └── Subtask (子任务)
  ├── BoardMember (看板成员)
  └── Label (标签)
```

### 4.2 详细 Schema

#### 4.2.1 User (用户)

```prisma
model User {
  id            String    @id @default(uuid())
  email         String    @unique
  name          String
  avatarUrl     String?
  role          UserRole  @default(USER)
  isAgent       Boolean   @default(false)  // 是否为 Agent
  agentConfig   Json?     // Agent 配置 { openclowAgentId, callbackApiToken, capabilities }
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  boards        BoardMember[]
  cards         CardMember[]
  comments      Comment[]
  activities    Activity[]
}
```

#### 4.2.2 Board (看板)

```prisma
model Board {
  id          String      @id @default(uuid())
  name        String
  description String?
  visibility  Visibility @default(PRIVATE)  // PRIVATE, PUBLIC, TEAM
  ownerId     String
  createdAt   DateTime    @default(now())
  updatedAt   DateTime    @updatedAt
  deletedAt   DateTime?   // 软删除

  owner       User        @relation(fields: [ownerId], references: [id])
  members     BoardMember[]
  lists       List[]
  labels      Label[]
}
```

#### 4.2.3 List (列表)

```prisma
model List {
  id        String   @id @default(uuid())
  boardId   String
  name      String
  position  Int      // 排序
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  board     Board    @relation(fields: [boardId], references: [id], onDelete: Cascade)
  cards     Card[]
}
```

#### 4.2.4 Card (卡片)

```prisma
model Card {
  id          String       @id @default(uuid())
  listId      String
  title       String
  description String?
  position    Int          // 列表内排序
  dueDate     DateTime?    // 截止日期
  priority    Priority     @default(MEDIUM)  // LOW, MEDIUM, HIGH, URGENT
  status      CardStatus   @default(TODO)   // TODO, IN_PROGRESS, IN_REVIEW, DONE, BLOCKED
  createdById String
  createdAt   DateTime     @default(now())
  updatedAt   DateTime     @updatedAt
  completedAt DateTime?    // 完成时间
  
  // 任务依赖（前置任务）
  dependsOn   String[]    // 依赖的任务 ID 数组
  blocking    Card[]      @relation("CardDependsOn")  // 被此任务阻塞的任务
  
  // 任务队列（Agent 用）
  queuePosition Int?      // 在 Agent 任务队列中的位置
  assignedAt   DateTime?  // 被分配给 Agent 的时间

  list        List        @relation(fields: [listId], references: [id], onDelete: Cascade)
  createdBy   User        @relation(fields: [createdById], references: [id])
  members     CardMember[]
  labels      CardLabel[]
  comments    Comment[]
  activities  Activity[]
  subtasks    Subtask[]
  
  // 自引用：任务依赖
  dependencies Card[]     @relation("CardDependsOn")
}

model CardDependency {
  id          String   @id @default(uuid())
  cardId      String   // 当前任务
  dependsOnId String   // 依赖的任务
  
  @@unique([cardId, dependsOnId])
}
```

#### 4.2.5 CardMember (卡片成员)

```prisma
model CardMember {
  id        String       @id @default(uuid())
  cardId    String
  userId    String
  role      CardRole     @default(OWNER)  // OWNER(负责人), HELPER(协助者)

  card      Card         @relation(fields: [cardId], references: [id], onDelete: Cascade)
  user      User         @relation(fields: [userId], references: [id])

  @@unique([cardId, userId])
}
```

#### 4.2.6 Agent Configuration (Agent 配置)

```typescript
interface AgentConfig {
  // OpenClaw 集成（通过 HTTP API 触发，无需 Webhook）
  openclowAgentId: string;      // OpenClaw 中的 Agent ID
  
  // 能力
  capabilities: {
    canCreateTask: boolean;      // 可以创建任务
    canCompleteTask: boolean;     // 可以完成任务
    canComment: boolean;          // 可以评论
    canAssignTask: boolean;      // 可以分配任务给他人
    autoPullTasks: boolean;       // 自动拉取任务（可选）
    pullInterval: number;         // 拉取间隔（分钟）
  };
  
  // 行为规则
  triggers: {
    onTaskAssigned: 'auto_start' | 'wait_manual';  // 收到任务时
    onTaskBlocked: 'create_subtask' | 'notify_human' | 'ignore';  // 任务阻塞时
    onMention: 'respond' | 'ignore';  // 被 @ 时
  };
  
  // 回调配置
  callback: {
    baseUrl: string;              // 看板 API 基础 URL
    apiToken: string;             // Agent 回调用的 API Token
  };
}
```

---

## 5. API 设计

### 5.1 认证

| 认证方式 | 适用场景 | Header |
|---------|---------|--------|
| JWT Token | 人类用户 | `Authorization: Bearer <token>` |
| API Token | Agent | `X-Agent-Token: <token>` |

### 5.2 核心 API

#### 看板

```yaml
# 获取看板列表
GET /api/boards
Response: { boards: Board[] }

# 创建看板
POST /api/boards
Body: { name, description?, visibility }
Response: { board: Board }

# 获取看板详情
GET /api/boards/:id
Response: { board: Board, lists: ListWithCards[] }

# 更新看板
PATCH /api/boards/:id
Body: { name?, description?, visibility? }
Response: { board: Board }

# 删除看板
DELETE /api/boards/:id
Response: { success: true }
```

#### 列表

```yaml
# 创建列表
POST /api/lists
Body: { boardId, name }
Response: { list: List }

# 更新列表
PATCH /api/lists/:id
Body: { name?, position? }
Response: { list: List }

# 删除列表
DELETE /api/lists/:id
Response: { success: true }

# 重新排序列表
PUT /api/boards/:id/lists/reorder
Body: { listIds: string[] }
Response: { success: true }
```

#### 卡片

```yaml
# 创建卡片
POST /api/cards
Body: {
  listId,
  title,
  description?,
  dueDate?,
  memberIds?: string[],
  labelIds?: string[]
}
Response: { card: Card }

# 批量创建卡片 (Agent 用)
POST /api/cards/batch
Body: {
  listId,
  cards: Array<{
    title,
    description?,
    dueDate?,
    memberIds?,
    labelIds?
  }>
}
Response: { cards: Card[] }

# 更新卡片
PATCH /api/cards/:id
Body: {
  title?,
  description?,
  dueDate?,
  priority?,
  status?,
  listId?,        # 移动到其他列表
  position?,       # 调整位置
  memberIds?,      # 更新成员
  labelIds?        # 更新标签
}
Response: { card: Card }

# 移动卡片
PATCH /api/cards/:id/move
Body: { listId, position }
Response: { card: Card }

# 删除卡片
DELETE /api/cards/:id
Response: { success: true }

# 完成任务
POST /api/cards/:id/complete
Response: { card: Card }

# 获取卡片详情
GET /api/cards/:id
Response: { card: Card, comments: Comment[], activities: Activity[], subtasks: Subtask[] }

# 获取卡片依赖
GET /api/cards/:id/dependencies
Response: { dependsOn: Card[], blockedBy: Card[] }

# 添加任务依赖（前置任务）
POST /api/cards/:id/dependencies
Body: { dependsOnIds: string[] }  // 需要先完成的任务 ID 数组
Response: { card: Card }

# 移除任务依赖
DELETE /api/cards/:id/dependencies/:dependsOnId
Response: { success: true }

# 检查任务是否可执行（前置依赖是否满足）
GET /api/cards/:id/can-start
Response: { canStart: boolean, blockedBy: Card[] }
```

#### 成员管理

```yaml
# 添加成员
POST /api/boards/:id/members
Body: { userId, role }  // role: OWNER, ADMIN, MEMBER, VIEWER
Response: { member: BoardMember }

# 移除成员
DELETE /api/boards/:id/members/:userId
Response: { success: true }

# 分配任务（添加卡片成员）
POST /api/cards/:id/members
Body: { userId, role }  // role: OWNER, HELPER
Response: { member: CardMember }

# 移除卡片成员
DELETE /api/cards/:id/members/:userId
Response: { success: true }
```

### 5.3 Agent 专用 API

```yaml
# Agent 注册 (管理员)
POST /api/agents/register
Body: {
  name,
  description?,
  openclowAgentId: string,    // OpenClaw 中的 Agent ID
  capabilities: AgentCapabilities,
  triggers: AgentTriggers,
  callback: {                 // 回调配置
    baseUrl: string,
    apiToken: string
  }
}
Response: { agent: User, apiToken: string }

# Agent 获取自己的任务
GET /api/agents/:id/tasks
Query: { status?, limit?, offset? }
Response: { tasks: Card[] }

# Agent 创建任务
POST /api/agents/:id/tasks
Body: {
  boardId,
  listId,
  title,
  description?,
  dueDate?,
  memberIds?
}
Response: { task: Card }

# Agent 完成任务
POST /api/agents/:id/tasks/:cardId/complete
Body: { comment? }
Response: { card: Card }

# Agent 求助
POST /api/agents/:id/tasks/:cardId/ask-help
Body: { message }
Response: { comment: Comment }

# Agent 心跳 (报告状态)
POST /api/agents/:id/heartbeat
Body: { status, currentTaskId?, message? }
Response: { success: true }
```

### 5.4 OpenClaw API 集成（替代 Webhook）

**重要：不需要 Webhook！** 看板服务端通过 HTTP API 直接调用 OpenClaw 来触发 Agent。

```yaml
# 看板服务端 → OpenClaw API

# 方式 1: 触发 Agent 执行任务 (推荐)
POST http://localhost:18789/api/sessions
Headers: { "Authorization": "Bearer <admin_token>" }
Body: {
  "agentId": "frontend-developer",
  "message": "请执行任务：实现登录页面\n卡片ID: card_123\n完成后更新状态为 DONE"
}
Response: { "sessionKey": "agent:xxx", "reply": "收到任务，开始执行..." }

# 方式 2: 通过 message 工具发送消息
POST http://localhost:18789/api/tools/message
Body: {
  "action": "send",
  "target": "agent:frontend-developer",
  "message": "任务详情..."
}

# Agent 完成后回调看板 API
PATCH http://localhost:3000/api/cards/:cardId
Body: { "status": "DONE", "completedAt": "2026-03-22T10:00:00Z" }
```

#### 5.4.1 集成架构

```
┌─────────────────────────────────────────────────────────────┐
│                     看板服务端 (Fastify)                       │
│                                                             │
│   用户操作 ──→ 任务创建 ──→ 判断 assigneeType               │
│                                      │                      │
│                    ┌─────────────────┴─────────────────┐    │
│                    ↓                                   ↓    │
│            分配给人类                    分配给 Agent         │
│                    │                                   │    │
│                    ↓                                   ↓    │
│            正常保存                    调用 OpenClaw API    │
│                                              │             │
│                                              ↓             │
│                                    POST /api/sessions      │
│                                    触发 Agent 执行         │
│                                              │             │
│                                              ↓             │
│                                    Agent 执行 ──→ 回调     │
│                                    PATCH /api/cards/:id   │
└─────────────────────────────────────────────────────────────┘
```

#### 5.4.2 Agent 触发代码示例

```python
# 看板后端 - 任务分配逻辑
@app.post("/api/cards")
async def create_card(card_data: dict, current_user: User):
    card = await save_card(card_data)
    
    # 如果分配给 Agent，触发 Agent 执行
    if card.get("assigneeType") == "agent":
        await trigger_openclaw_agent(
            agent_id=card["assigneeId"],
            card_id=card["id"],
            task_title=card["title"],
            task_description=card.get("description", ""),
            board_id=card["boardId"]
        )
    
    return card

async def trigger_openclaw_agent(
    agent_id: str,
    card_id: str,
    task_title: str,
    task_description: str,
    board_id: str
):
    """通过 OpenClaw API 触发 Agent 执行任务"""
    
    OPENCLAW_API = "http://localhost:18789"
    ADMIN_TOKEN = os.getenv("OPENCLAW_ADMIN_TOKEN")
    
    message = f"""📋 新任务分配

**任务标题**: {task_title}
**任务描述**: {task_description}
**卡片ID**: {card_id}
**看板ID**: {board_id}

请开始执行，完成后更新卡片状态为 DONE。
如有疑问可在卡片下评论。
"""
    
    async with httpx.AsyncClient() as client:
        # 调用 OpenClaw Sessions API
        response = await client.post(
            f"{OPENCLAW_API}/api/sessions",
            json={
                "agentId": agent_id,
                "message": message
            },
            headers={
                "Authorization": f"Bearer {ADMIN_TOKEN}",
                "Content-Type": "application/json"
            }
        )
        
        if response.status_code == 200:
            result = response.json()
            # 保存 sessionKey 以便后续追踪
            await save_agent_session(card_id, result.get("sessionKey"))
        else:
            # Agent 触发失败，记录日志
            logger.error(f"Failed to trigger agent: {response.text}")
```

#### 5.4.3 Agent 回调逻辑

```python
# Agent 端 prompt 示例（告诉 Agent 完成后如何回调）
AGENT_PROMPT = """
你是 {agent_name}，负责执行看板任务。

执行流程：
1. 读取任务详情（卡片ID: {card_id}）
2. 检查任务依赖是否都已完成
3. 执行任务内容
4. 完成任务后，调用 API 更新状态：
   
   PATCH {kanban_url}/api/cards/{card_id}
   Body: {{
       "status": "DONE",
       "completedAt": "{timestamp}",
       "comment": "任务已完成：{result_summary}"
   }}

5. 如果遇到问题，创建子任务或 @人类求助
"""
```

#### 5.4.4 任务队列与依赖处理

当 Agent 被分配多个任务，或任务有前置依赖时，按以下规则处理：

```python
# 任务队列选择逻辑
async def get_next_task_for_agent(agent_id: str) -> Card | None:
    """获取 Agent 下一个应执行的任务"""
    
    # 1. 查找分配给该 Agent 的所有待办任务
    pending_tasks = await db.cards.find_many({
        "members.userId": agent_id,
        "status": "TODO",
        # 排除已阻塞的任务
    })
    
    # 2. 检查每个任务的前置依赖
    available_tasks = []
    for task in pending_tasks:
        dependencies = await get_dependencies(task.id)
        completed = all(d.status == "DONE" for d in dependencies)
        
        if completed:
            # 3. 按优先级和截止日期排序
            available_tasks.append(task)
    
    # 4. 返回最优先的任务
    return sorted(available_tasks, key=lambda t: (t.priority, t.dueDate))[0]
```

##### 任务依赖示例

```
任务关系：
┌─────────────┐      ┌─────────────┐
│  任务 A     │      │  任务 B     │
│ (前端登录页) │ ──→ │ (后端登录API)│
│  dependsOn: []     │  dependsOn: [A]
└─────────────┘      └─────────────┘
                          │
                          ↓
                    ┌─────────────┐
                    │  任务 C     │
                    │ (集成测试)  │
                    │ dependsOn: [A, B]
                    └─────────────┘

执行顺序：
1. A 和 B 可以同时开始（无依赖）
2. C 必须等 A 和 B 都完成才能开始
```

##### 任务阻塞状态

```python
# 检查任务是否被阻塞
async def check_task_blocked(card_id: str) -> dict:
    """检查任务是否因前置依赖未完成而被阻塞"""
    
    card = await get_card(card_id)
    dependencies = await get_dependencies(card_id)
    
    blocked_by = [d for d in dependencies if d.status != "DONE"]
    
    if blocked_by:
        # 自动更新状态为 BLOCKED
        await update_card(card_id, {
            "status": "BLOCKED",
            "comment": f"等待前置任务完成: {', '.join([d.title for d in blocked_by])}"
        })
        return {"blocked": True, "reason": blocked_by}
    
    return {"blocked": False}
```

##### Agent 任务处理流程

```python
# Agent 执行任务的完整流程
async def agent_execute_task(agent_id: str, card_id: str):
    """Agent 执行单个任务"""
    
    # 1. 检查前置依赖
    check = await check_task_blocked(card_id)
    if check["blocked"]:
        # 被阻塞，跳过此任务，获取下一个
        next_task = await get_next_task_for_agent(agent_id)
        if next_task:
            await agent_execute_task(agent_id, next_task.id)
        return
    
    # 2. 更新状态为进行中
    await update_card(card_id, {"status": "IN_PROGRESS"})
    
    # 3. 执行任务
    result = await do_task(card_id)
    
    # 4. 完成任务
    await update_card(card_id, {
        "status": "DONE",
        "completedAt": now(),
        "comment": f"任务完成: {result}"
    })
    
    # 5. 检查是否有任务因此解除阻塞
    await check_unblocked_tasks(card_id)
    
    # 6. 获取下一个任务
    next_task = await get_next_task_for_agent(agent_id)
    if next_task:
        await agent_execute_task(agent_id, next_task.id)
```

---

## 6. Agent 集成方案

### 6.1 Agent 工作流程

**重要：本方案使用 HTTP API 触发 Agent，无需 Webhook！**

```
┌─────────────────────────────────────────────────────────────┐
│                      Agent 工作流程                          │
└─────────────────────────────────────────────────────────────┘

1. 注册 & 配置
   ┌──────────────┐
   │  管理员注册   │ ──→ 绑定 OpenClaw Agent ID + 回调 API Token
   │   Agent     │
   └──────────────┘

2. 任务触发 (HTTP API 方式)
   ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
   │  人类分配    │ ───→ │  调用 API    │ ───→ │  OpenClaw   │
   │  任务       │      │  sessions   │      │  触发 Agent  │
   └──────────────┘      └──────────────┘      └──────────────┘

3. 执行任务
   ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
   │  读取任务    │ ───→ │  执行工作    │ ───→ │  更新状态    │
   │  详情       │      │  (代码/文档) │      │  IN_PROGRESS │
   └──────────────┘      └──────────────┘      └──────────────┘

4. 完成任务
   ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
   │  调用 API    │ ───→ │  更新状态    │ ───→ │  添加评论    │
   │  回调       │      │  DONE       │      │  通知人类    │
   └──────────────┘      └──────────────┘      └──────────────┘
```

### 6.2 Agent 任务来源

| 来源 | 触发方式 | 示例 |
|-----|---------|------|
| 人类分配 | HTTP API | 项目经理分配代码审查任务 → 调用 sessions API → Agent 执行 |
| 自动拉取 | Cron（可选） | Agent 每 5 分钟拉取分配给自己的 TODO 任务 |
| Agent 创建 | 主动发现 | Agent 发现代码问题，主动创建 Task |

### 6.3 Agent 能力矩阵

| Agent 类型 | 可创建任务 | 可完成任务 | 可评论 | 可分配任务 |
|-----------|-----------|-----------|-------|-----------|
| Code Reviewer | ✅ | ✅ | ✅ | ❌ |
| Frontend Developer | ✅ | ✅ | ✅ | ❌ |
| Backend Architect | ✅ | ✅ | ✅ | ✅ |
| QA Tester | ✅ | ✅ | ✅ | ❌ |
| Technical Writer | ✅ | ✅ | ✅ | ❌ |

### 6.4 OpenClaw 集成

#### 方案 A: HTTP API 触发（推荐）

```
看板服务端 ──→ HTTP POST /api/sessions ──→ OpenClaw Agent
     ↑                                              │
     │                                              │
     └────── 回调 PATCH /api/cards/:id ←───────────┘
```

- 看板服务通过 OpenClaw API 触发 Agent 执行
- Agent 执行完成后回调看板 API 更新状态
- 全程 HTTP API，无需 Webhook

#### 方案 B: Agent 嵌入看板服务

```
┌──────────────────────────────────────────┐
│              看板系统后端                  │
│  ┌─────────────┐    ┌─────────────────┐ │
│  │ Fastify API │    │  Agent 调度器    │ │
│  └─────────────┘    └─────────────────┘ │
│         ↑                   ↑            │
│         │    ┌──────────────┘            │
│         ↓    ↓                            │
│  ┌─────────────────────────────────────┐  │
│  │      OpenClaw Gateway (嵌入)        │  │
│  │   ┌────────┐ ┌────────┐ ┌────────┐ │  │
│  │   │Agent A │ │Agent B │ │Agent C │ │  │
│  │   └────────┘ └────────┘ └────────┘ │  │
│  └─────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

- Agent 直接在看板后端运行
- 共享数据库，无需 API 调用
- 更紧密的集成

---

## 7. 安全设计

### 7.1 权限模型

| 角色 | 看板权限 | 卡片权限 | 成员管理 | Agent 管理 |
|-----|---------|---------|---------|-----------|
| Owner | 完整 | 完整 | 完整 | 完整 |
| Admin | 编辑 | 完整 | 管理 | 管理 |
| Member | 创建/编辑 | 创建/编辑 | ❌ | ❌ |
| Viewer | 查看 | 查看 | ❌ | ❌ |
| Agent | 按配置 | 按配置 | ❌ | ❌ |

### 7.2 API 认证

```yaml
# 人类用户
Authorization: Bearer <jwt_token>
  - Token 有效期: 24h
  - 刷新 Token: 7 天

# Agent
X-Agent-Token: <api_token>
  - Token 绑定到 Agent
  - 可随时重置
```

### 7.3 安全措施

| 措施 | 说明 |
|-----|------|
| HTTPS | 生产环境强制 HTTPS |
| Rate Limiting | API 限流 100/分钟/IP |
| 输入验证 | 所有输入进行 Schema 验证 |
| SQL 注入 | 使用 Prisma 参数化查询 |
| XSS | 响应进行 HTML 转义 |
| CORS | 限制允许的来源 |
| 审计日志 | 记录所有敏感操作 |

---

## 8. 技术栈

### 8.1 推荐技术栈

| 层级 | 技术选型 | 理由 |
|-----|---------|------|
| 前端 | React 18 + TypeScript | 成熟稳定，类型安全 |
| 状态管理 | Zustand | 轻量，简单 |
| 拖拽 | @dnd-kit | 现代，可访问性好 |
| 后端 | Fastify | 高性能，Schema 验证 |
| ORM | Prisma | 类型安全，迁移方便 |
| 数据库 | PostgreSQL | 关系型，功能强大 |
| 缓存 | Redis | Session、任务队列 |
| 认证 | JWT + API Token | 简单可靠 |

### 8.2 目录结构

```
human-agent-kanban/
├── client/                    # 前端
│   ├── src/
│   │   ├── components/       # 组件
│   │   ├── pages/            # 页面
│   │   ├── hooks/            # 自定义 Hooks
│   │   ├── services/         # API 服务
│   │   ├── stores/           # Zustand Stores
│   │   └── types/            # TypeScript 类型
│   └── package.json
│
├── server/                    # 后端
│   ├── src/
│   │   ├── routes/           # API 路由
│   │   ├── services/         # 业务逻辑
│   │   ├── middleware/       # 中间件
│   │   ├── agents/          # Agent 调度
│   │   └── prisma/          # 数据库 Schema
│   ├── prisma/
│   │   └── schema.prisma
│   └── package.json
│
└── README.md
```

---

## 9. 开发计划

### Phase 1: MVP (10 天)

| 天数 | 任务 | 交付物 |
|-----|------|-------|
| 1-2 | 项目初始化、数据库设计 | 代码框架、Schema |
| 3-4 | 用户认证、看板 CRUD | 基础 API |
| 5-6 | 列表管理、卡片 CRUD | 核心功能 |
| 7-8 | 拖拽排序、实时同步 | 前端交互 |
| 9-10 | 测试、修复、部署 | 可运行系统 |

### Phase 2: Agent 集成 (7 天)

| 天数 | 任务 | 交付物 |
|-----|------|-------|
| 1-2 | Agent 注册、回调 Token 管理 | Agent API |
| 3-4 | HTTP API 触发 Agent、执行 | Agent 集成 |
| 5-6 | Agent 回调、状态更新 | 回调机制 |
| 7 | 测试、文档 | 完成交付 |

### Phase 3: 增强功能 (5 天)

| 天数 | 任务 | 交付物 |
|-----|------|-------|
| 1-2 | 评论、活动日志 | 协作功能 |
| 3-4 | 搜索、筛选、导出 | 高级功能 |
| 5 | 优化、性能 | 完善 |

---

## 10. 附录

### 10.1 错误码

```typescript
enum ErrorCode {
  // 通用
  UNAUTHORIZED = 10001,
  FORBIDDEN = 10002,
  NOT_FOUND = 10003,
  VALIDATION_ERROR = 10004,
  
  // 看板
  BOARD_NOT_FOUND = 20001,
  BOARD_ACCESS_DENIED = 20002,
  
  // 卡片
  CARD_NOT_FOUND = 30001,
  CARD_MOVE_FAILED = 30002,
  
  // Agent
  AGENT_NOT_FOUND = 40001,
  AGENT_DISABLED = 40002,
  AGENT_TASK_NOT_FOUND = 40003,
}
```

### 10.2 事件列表

| 事件名 | 描述 | 触发者 |
|-------|------|-------|
| board.created | 看板创建 | 人类 |
| board.updated | 看板更新 | 人类/Agent |
| board.deleted | 看板删除 | 人类 |
| list.created | 列表创建 | 人类/Agent |
| card.created | 卡片创建 | 人类/Agent |
| card.moved | 卡片移动 | 人类/Agent |
| card.completed | 卡片完成 | 人类/Agent |
| card.status_changed | 状态变更 | 人类/Agent |
| member.added | 成员添加 | 人类 |
| comment.added | 评论添加 | 人类/Agent |

### 10.3 参考资料

- [Trello API](https://developer.atlassian.com/cloud/trello/)
- [Prisma Schema](https://www.prisma.io/docs/concepts/components/prisma-schema)
- [Fastify](https://www.fastify.io/)

---

**文档结束**
