# 🔒 安全最佳实践 SOP

> **目标**: 建立全栈安全防护体系，覆盖移动端/Web/后端
> **参考**: OWASP Top 10, CWE Top 25

---

## 1. 认证 & 授权

### 密码策略

```
□ 最小长度: 8 字符
□ 复杂度: 至少包含大写+小写+数字
□ 存储: bcrypt (cost ≥ 10) 或 argon2
□ 传输: HTTPS only
□ 重试限制: 5 次失败锁定 15 分钟
```

```typescript
// ✅ Hono/JWT 实现
import bcrypt from "bcryptjs"

const hashedPassword = await bcrypt.hash(password, 10)
const isValid = await bcrypt.compare(inputPassword, hashedPassword)
```

### Token 管理

```
□ JWT: 过期时间 ≤ 7 天, 使用 RS256 (非对称)
□ Refresh Token: 一次性使用, 存储 httpOnly cookie
□ 登出: 服务端销毁 refresh token
□ 密钥轮换: 每 90 天更换签名密钥
```

### 会话安全

```
□ iOS: Keychain 存储敏感数据
□ Android: EncryptedSharedPreferences
□ 鸿蒙: HUKS 密钥管理
□ Web: httpOnly + Secure + SameSite=Strict cookie
```

---

## 2. 数据保护

### 传输加密

```nginx
# TLS 1.2+ 配置 (Nginx)
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
ssl_prefer_server_ciphers off;
```

### 静态数据加密

```
□ 数据库: 敏感字段 AES-256-GCM 加密
□ 文件存储: 服务端加密 (S3 SSE / 阿里云 OSS 加密)
□ 备份: 加密后存储, 密钥独立管理
□ 日志: 不记录密码/Token/身份证号
```

### 数据脱敏

```typescript
// 日志脱敏示例
function maskEmail(email: string): string {
  const [name, domain] = email.split("@")
  return `${name.slice(0, 2)}***@${domain}`
}

function maskPhone(phone: string): string {
  return phone.replace(/(\d{3})\d{4}(\d{4})/, "$1****$2")
}
```

---

## 3. 输入验证 & 注入防护

### SQL 注入 → 参数化查询

```typescript
// ✅ Prisma (自动参数化)
await prisma.user.findUnique({ where: { email } })

// ❌ 原生 SQL 拼接
// db.query(`SELECT * FROM users WHERE email = '${email}'`)
```

### XSS 防护

```tsx
// React/Next.js: JSX 自动转义
<div>{userInput}</div>  // ✅ 自动转义

// 危险: dangerouslySetInnerHTML → 必须先用 DOMPurify
import DOMPurify from "dompurify"
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(input) }} />
```

### API 输入验证

```typescript
// ✅ Zod schema 验证
import { z } from "zod"
const schema = z.object({
  email: z.string().email().max(255),
  age: z.number().int().min(0).max(150),
  name: z.string().min(1).max(100).regex(/^[\p{L}\s-]+$/u)
})
```

---

## 4. 依赖安全

```bash
# [SHELL] 定期扫描漏洞 (加入 CI)
npm audit --audit-level=high        # Node.js
bundle audit                        # Ruby
cargo audit                         # Rust
safety check                        # Python

# GitHub Dependabot: 自动创建 PR 更新依赖
# .github/dependabot.yml
```

---

## 5. 安全响应流程

```
发现漏洞
  ↓ 1h 内
评估严重程度 (CVSS 评分)
  ↓
├── Critical (CVSS ≥ 9): 24h 修复
├── High (7-8.9): 72h 修复  
├── Medium (4-6.9): 下次迭代修复
└── Low (< 4): 排期修复
  ↓
发布补丁 → 通知用户 → Postmortem
```

---

> **SOP 版本**: 1.0.0
