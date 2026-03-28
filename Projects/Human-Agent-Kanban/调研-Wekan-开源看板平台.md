# Wekan 开源看板平台调研

> 调研日期：2026-03-27
> 目的：评估作为 Human-Agent-Kanban 的任务看板底层平台

---

## 基本信息

| 项目 | 信息 |
|------|------|
| **GitHub** | github.com/wekan/wekan |
| **Stars** | ~9,400+ |
| **技术栈** | Node.js + Meteor + MongoDB |
| **License** | MIT |
| **部署** | Docker / 手动部署 |

---

## API 支持

Wekan 有 REST API（通过 `/api` 端点），主要能力：

| 功能 | 支持情况 |
|------|---------|
| 登录认证 | ✅ Cookie-based + Token |
| Boards CRUD | ✅ |
| Lists CRUD | ✅ |
| Cards CRUD | ✅ |
| Comments | ✅ |
| User/Member 管理 | ✅ |
| Webhook | ✅ |
| Swimlanes | ✅ Wekan 特有（泳道）|

**API 端点示例：**
```
GET  /api/boards
POST /api/boards/:boardId/lists
POST /api/lists/:listId/cards
PUT  /api/cards/:cardId
POST /api/cards/:cardId/move
POST /api/cards/:cardId/comments
```

---

## Docker 部署

```bash
# 最简方式
docker run -d --name wekan \
  -p 3000:3000 \
  -e WRITEDB_PATH=/data \
  -v /path/to/data:/data \
  wekan/wekan
```

**注意：** Wekan 使用 MongoDB，需要额外的数据卷/链接，部署比 Plane 稍复杂。

---

## 与 Agent 集成的适合度

| 维度 | 评分 | 说明 |
|------|------|------|
| API 完整性 | ⭐⭐⭐ | 够用，但不如 Plane 规范 |
| 认证方式 | ⭐⭐⭐ | UserId + Token 可以用 |
| 卡片操作 | ⭐⭐⭐⭐ | move, copy, archive 都有 |
| 实时通知 | ⭐⭐ | Webhook 有但不如 Plane 完善 |
| 部署复杂度 | ⭐⭐⭐ | Docker + MongoDB 稍复杂 |
| 界面美观 | ⭐⭐ | 比较朴素，像老版 Trello |

---

## 主要缺点

1. **界面偏老旧** — 不如 Plane 好看
2. **文档较分散** — Wiki、README、GitHub issues 里都有
3. **MongoDB** — 个人部署比 PostgreSQL 麻烦
4. **API 认证** — 基于 userId + auth-token，不如 JWT 规范
5. **活跃度** — 更新频率一般

---

## 结论

| 适合 | 不适合 |
|------|--------|
| 快速验证 Agent + 看板协作 | 长期产品化 |
| 轻量级 | 需要好看 UI |
| API 够用 | 需要 Jira 级的复杂功能 |
| Docker 熟悉 | 不熟悉 MongoDB |

---

## 三平台对比（Wekan / Plane / Taiga）

| 维度 | **Plane** | **Wekan** | **Taiga** |
|------|-----------|-----------|-----------|
| **定位** | 类 Linear/Jira | 类 Trello（看板） | 类 Jira（敏捷） |
| **界面美观度** | ⭐⭐⭐⭐ 现代简洁 | ⭐⭐ 传统卡片风 | ⭐⭐⭐ 功能导向 |
| **API 成熟度** | ⭐⭐⭐⭐⭐ REST API 完整 | ⭐⭐⭐ REST API 完整 | ⭐⭐⭐ REST API 完整 |
| **Webhook** | ✅ 支持 | ✅ 支持 | ✅ 支持 |
| **看板视图** | ✅ Projects + Issues + Cycles | ✅ 看板 + 列表 + 泳道 | ✅ 看板 + 列表 + 燃尽图 |
| **成员管理** | ✅ Member + Guest | ✅ 成员角色 | ✅ Member + Guest |
| **任务依赖** | ✅ 支持 | ❌ 不支持 | ✅ Epic/Story/Task |
| **部署难度** | ⭐⭐ Docker-compose | ⭐⭐ Docker + MongoDB | ⭐⭐ Docker |
| **文档完善度** | ⭐⭐⭐⭐ 官方文档完整 | ⭐⭐⭐ 社区文档 | ⭐⭐⭐ 文档全但分散 |
| **活跃度** | ⭐⭐⭐⭐⭐ GitHub 14k+ star | ⭐⭐⭐ GitHub 9k+ star | ⭐⭐⭐ GitHub 6k+ star |
| **Agent 集成** | ⭐⭐⭐⭐ API 够用，可深度集成 | ⭐⭐⭐ API 简单够用 | ⭐⭐⭐ API 面向人而非机器 |

### 推荐结论

| 场景 | 推荐 |
|------|------|
| **快速验证 Agent 协作** | **Wekan** — 最像 Trello，API 简单 |
| **长期产品/认真做** | **Plane** — 最接近 Linear，UI 最好，API 设计最规范 |
| **已有敏捷开发流程** | **Taiga** — Epic/Story 天然支持，适合研发团队 |

### 对于 Human-Agent-Kanban 的建议

- **Wekan** — 适合快速验证 Agent + 看板协作，轻量
- **Plane** — 如果追求更好体验和长期维护，更优（界面好、API 规范、PostgreSQL 部署简单）

## 备选对比

| 平台 | 优势 | 劣势 |
|------|------|------|
| **Wekan** | API 够用、轻量 | 界面老、文档散、MongoDB |
| **Plane** | 界面好、API 规范、PostgreSQL | 部署稍复杂 |
| **Taiga** | Epic/Story 支持、敏捷友好 | 面向人而非机器的 API 设计 |

---

## 下一步

待定：确认底层平台选择后，更新 SPEC.md

---

*Last Updated: 2026-03-27*
