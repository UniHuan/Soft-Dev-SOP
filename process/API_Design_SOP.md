# 🔗 API 设计规范 SOP

> **目标**: 统一 RESTful API 设计标准，前后端协作零摩擦
> **适用**: 移动端 + Web + 后端所有 API 设计

---

## 1. URL 设计

```
GET    /api/v1/tasks              # 列表
POST   /api/v1/tasks              # 创建
GET    /api/v1/tasks/:id          # 详情
PATCH  /api/v1/tasks/:id          # 部分更新
DELETE /api/v1/tasks/:id          # 删除

GET    /api/v1/tasks/:id/comments # 子资源
```

### 命名规则

```
✅ 名词复数: /tasks, /users
✅ 小写 + 连字符: /task-lists
✅ 层级 ≤ 3: /tasks/:id/comments  (非 /users/:uid/tasks/:tid/comments/:cid)
❌ 动词: /getTasks, /createUser
❌ 大小写混用: /TaskLists
```

---

## 2. 请求 & 响应

### 统一响应格式

```json
// 成功
{
  "success": true,
  "data": { "id": "xxx", "title": "任务" }
}

// 列表 (分页)
{
  "success": true,
  "data": [...],
  "pagination": { "total": 100, "page": 1, "limit": 20, "totalPages": 5 }
}

// 错误
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "标题不能为空",
    "details": [{ "field": "title", "message": "必填字段" }]
  }
}
```

### HTTP 状态码

| 状态码 | 含义 | 使用场景 |
|--------|------|---------|
| 200 | OK | GET/PATCH 成功 |
| 201 | Created | POST 创建成功 |
| 204 | No Content | DELETE 成功 |
| 400 | Bad Request | 参数校验失败 |
| 401 | Unauthorized | Token 缺失/无效 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | 资源不存在 |
| 409 | Conflict | 资源冲突 (重复) |
| 422 | Unprocessable | 业务逻辑错误 |
| 429 | Too Many | 限流 |
| 500 | Internal Error | 服务器错误 |

---

## 3. 版本管理

```
策略: URL 路径版本 (推荐)
  /api/v1/tasks     ← v1
  /api/v2/tasks     ← v2 (breaking changes)

版本升级原则:
  - 新增字段: 不升版本
  - 删除/重命名字段: MAJOR 版本
  - 响应格式变更: MAJOR 版本
  - 默认返回最新版, sunset 通知旧版废弃日期
```

---

## 4. 性能 & 限流

```
□ 列表默认分页: page=1&limit=20 (最大 100)
□ 字段过滤: ?fields=id,title (减少传输)
□ 压缩: gzip/brotli (服务端自动)
□ 缓存: ETag + Cache-Control
□ 限流: 100 req/min (普通), 1000 req/min (认证)
```

---

> **SOP 版本**: 1.0.0
