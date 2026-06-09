# 🧪 测试策略 SOP — 全平台覆盖

> **目标**: 建立分层测试体系，确保代码质量和用户体验

---

## 📊 测试金字塔

```
         ╱ E2E ╲          ← 少量: 核心用户流程 (Playwright/XCUITest)
        ╱  集成  ╲         ← 适量: API + 组件交互
       ╱   单元   ╲        ← 大量: 纯函数 + ViewModel 逻辑
      ╱─────────────╲
```

### 各层测试比例

| 层级 | 占比 | 工具 (iOS/Android/鸿蒙/Web/后端) | 目标 |
|------|------|------|------|
| 单元测试 | 70% | XCTest / JUnit / Hypium / Vitest | 覆盖率 ≥ 80% |
| 集成测试 | 20% | XCUITest / Espresso / UiTest / Playwright | 核心流程 |
| E2E 测试 | 10% | XCUITest / Playwright / Detox | 关键路径 |

---

## 1. 单元测试规范

### 测试结构 (AAA 模式)

```typescript
// Arrange → Act → Assert
describe("TaskViewModel", () => {
  it("should filter active tasks only", () => {
    // Arrange: 准备数据
    const vm = new TaskViewModel()
    vm.tasks = [{ id: "1", isCompleted: false }, { id: "2", isCompleted: true }]
    
    // Act: 执行操作
    vm.selectedFilter = "active"
    
    // Assert: 验证结果
    expect(vm.filteredTasks).toHaveLength(1)
    expect(vm.filteredTasks[0].id).toBe("1")
  })
})
```

### 必须测试的场景

```
每个函数至少覆盖:
□ 正常路径 (Happy Path)
□ 边界条件 (空值/最大值/最小值)
□ 错误处理 (网络失败/数据库错误/无效输入)
□ 并发场景 (如适用)
```

---

## 2. UI/集成测试

### iOS — XCUITest

```swift
func testAddTaskFlow() {
    let app = XCUIApplication()
    app.launch()
    app.buttons["添加任务"].tap()
    app.textFields["任务标题"].typeText("Buy milk")
    app.buttons["保存"].tap()
    XCTAssertTrue(app.staticTexts["Buy milk"].exists)
}
```

### Android — Compose Testing

```kotlin
@Test
fun addTask_showsInList() {
    composeTestRule.setContent { HomeScreen() }
    composeTestRule.onNodeWithText("添加任务...").performTextInput("Buy milk")
    composeTestRule.onNodeWithText("添加").performClick()
    composeTestRule.onNodeWithText("Buy milk").assertIsDisplayed()
}
```

### Web — Playwright

```typescript
test("complete task flow", async ({ page }) => {
    await page.goto("/")
    await page.fill("input[name=title]", "Buy groceries")
    await page.click("button[type=submit]")
    await expect(page.getByText("Buy groceries")).toBeVisible()
})
```

---

## 3. 测试覆盖率目标

```bash
# iOS: Xcode → Product → Test (⌘U) → Coverage 标签
# Android: ./gradlew jacocoTestReport
# 鸿蒙: DevEco Studio → Run → Coverage
# Web: pnpm vitest --coverage
# 后端: pnpm vitest --coverage
```

| 指标 | 最低 | 目标 |
|------|------|------|
| 行覆盖率 | 70% | 85% |
| 分支覆盖率 | 60% | 80% |
| 函数覆盖率 | 75% | 90% |

---

## 4. Bug 报告模板

```markdown
### Bug: [简短标题]

**严重程度**: 🔴 Critical / 🟡 Major / 🟢 Minor
**平台/版本**: iOS 17.0 / App v1.0.2
**复现步骤**:
1. 打开 App
2. 点击 "添加任务"
3. 输入空标题 → 点击保存
**期望结果**: 显示 "标题不能为空" 提示
**实际结果**: App 闪退
**复现率**: 100% (每次都复现)
**截图/录屏**: [附件]
```

---

> **SOP 版本**: 1.0.0 | **适用**: 所有平台 SOP
