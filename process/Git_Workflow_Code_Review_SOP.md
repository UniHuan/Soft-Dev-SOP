# 🌿 Git 工作流 & 代码审查 SOP

> **适用对象**: Claude Code (AI Agent) + 开发团队
> **目标**: 规范化分支管理、提交规范、代码审查流程

---

## 🌿 分支策略 (Trunk-Based Flow)

```
main ──────────────────────────────────────────→ 生产
  │
  ├── feature/login ────→ PR → squash merge → main
  ├── feature/payment ──→ PR → squash merge → main
  ├── fix/crash-on-launch → PR → squash merge → main
  └── release/v1.0.0 ────→ tag → deploy
```

### 分支命名规范

```bash
feature/<功能描述>    # 新功能: feature/user-auth, feature/dark-mode
fix/<问题描述>        # Bug修复: fix/crash-on-null, fix/typo
hotfix/<问题描述>     # 紧急修复: hotfix/payment-crash
release/<版本号>      # 发布分支: release/v1.0.0
chore/<描述>          # 杂项: chore/update-deps, chore/cleanup
```

---

## 📝 提交规范 (Conventional Commits)

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

### Type 类型

| Type | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(auth): add Apple Sign In` |
| `fix` | Bug 修复 | `fix(list): resolve empty state crash` |
| `docs` | 文档 | `docs(readme): update install guide` |
| `style` | 格式 (不影响代码逻辑) | `style: format with prettier` |
| `refactor` | 重构 | `refactor(db): extract query builder` |
| `perf` | 性能优化 | `perf(list): add lazy loading` |
| `test` | 测试 | `test(auth): add login unit tests` |
| `chore` | 构建/工具 | `chore: update dependencies` |
| `ci` | CI/CD | `ci: add Android workflow` |
| `revert` | 回滚 | `revert: feat(auth): add Apple Sign In` |

### 提交粒度

```
✅ 好: 一个 commit 做一件事
❌ 坏: "修复登录bug + 添加新功能 + 更新文档" (三件事混一起)
✅ 好: feat(auth): add biometric login
✅ 好: fix(ui): correct button alignment on iPad
```

---

## 👀 代码审查 (Pull Request) 流程

### PR 模板

```markdown
## 变更说明
[简述此 PR 做了什么, 一句话]

## 变更类型
- [ ] 新功能 (feat)
- [ ] Bug 修复 (fix)
- [ ] 重构 (refactor)
- [ ] 文档 (docs)

## 测试
- [ ] 单元测试通过
- [ ] UI 测试通过
- [ ] 手动验证 (截图/录屏)

## 检查清单
- [ ] 代码符合项目规范
- [ ] 无硬编码密钥/Token
- [ ] 新依赖已审计
- [ ] 无障碍 (accessibility) 已考虑
- [ ] 文档已更新 (如需要)

## 关联 Issue
Closes #123
```

### 审查检查项

```
代码质量:
□ 命名清晰 (变量/函数/类名自解释)
□ 函数单一职责 (一个函数 < 50 行)
□ 无重复代码 (DRY)
□ 无过度设计 (YAGNI)
□ 类型安全 (TypeScript/Swift/ArkTS 类型正确)

性能:
□ 无不必要的重新渲染 (React/Compose)
□ 大列表使用虚拟滚动/LazyColumn
□ 无主线程阻塞操作
□ 图片优化 (压缩/懒加载/适当尺寸)

安全:
□ 无硬编码密钥
□ 输入验证完整
□ SQL 注入防护 (参数化查询)
□ XSS 防护 (HTML 转义)

测试:
□ 核心逻辑有单元测试
□ 关键路径有集成测试
□ 边界条件已测试
```

### 审查意见分类

```
🔴 MUST FIX     — 必须修复 (安全漏洞/崩溃/数据丢失)
🟡 SHOULD FIX   — 应该修复 (性能/可维护性)
🟢 NICE TO HAVE — 可选优化 (代码风格/命名)
💬 QUESTION     — 提问 (不理解需要澄清)
👍 PRAISE       — 表扬 (写得好!)
```

---

## 🔄 分支操作命令

```bash
# [SHELL] 创建功能分支
git checkout main && git pull
git checkout -b feature/user-auth

# [SHELL] 提交 & 推送
git add -A
git commit -m "feat(auth): add email/password login"
git push origin feature/user-auth

# [SHELL] 合并前更新 (rebase)
git checkout feature/user-auth
git rebase main
# 解决冲突 → git rebase --continue

# [SHELL] 发布后打 tag
git checkout main && git pull
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0

# [SHELL] 紧急修复
git checkout main
git checkout -b hotfix/critical-bug
# 修复 → commit → push → PR → merge
git tag -a v1.0.1 -m "Hotfix v1.0.1"
```

---

## 🤖 Claude Code 自动化

```
Claude Code 在每个 SOP Phase 结束时自动:
□ git add -A
□ git commit -m "Phase N: <description>"  (符合 Conventional Commits)
□ 不需要单独 push (由用户决定何时 push)

Claude Code 在创建 PR 时:
□ [WRITE] PR 模板 → [DIALOG] 用户确认 → 通知审查者
```

---

> **SOP 版本**: 1.0.0 | **适用**: 所有平台 SOP
