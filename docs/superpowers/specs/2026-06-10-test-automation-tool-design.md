# 测试自动化工具 (sop-test) 设计文档

> 创建日期：2026-06-10
> 状态：已批准

---

## 1. 概述

### 1.1 基本信息

| 项目 | 内容 |
|------|------|
| 名称 | sop-test (SOP 统一测试工具) |
| 架构 | 核心框架 + 平台适配器插件 |
| 测试类型 | 单元测试 + 集成测试 |
| 报告格式 | 终端输出 + HTML + JSON/XML |
| 失败定位 | 代码行定位 + 代码片段展示 + 错误分类 |
| CI支持 | GitHub Actions / GitLab CI / Jenkins |

### 1.2 设计目标

- 提供统一的测试执行接口，覆盖所有SOP平台
- 多格式报告输出，满足开发、归档、CI不同需求
- 失败测试智能定位，提升调试效率
- 开箱即用的CI配置，降低持续集成门槛

---

## 2. 架构设计

### 2.1 项目结构

```
sop-test/
├── core/                   # 核心框架
│   ├── runner.ts           # 测试运行器
│   ├── reporter.ts         # 报告生成器
│   ├── analyzer.ts         # 失败分析器
│   └── cli.ts              # 命令行接口
├── adapters/               # 平台适配器
│   ├── ios.ts              # iOS (xcodebuild test)
│   ├── android.ts          # Android (./gradlew test)
│   ├── harmonyos.ts        # HarmonyOS (./hvigorw test)
│   ├── web.ts              # Web (vitest/jest)
│   ├── backend.ts          # Backend (vitest/jest)
│   ├── miniprogram.ts      # 小程序 (vitest)
│   └── cocos.ts            # Cocos Creator (jest)
├── reporters/              # 报告格式
│   ├── terminal.ts         # 终端彩色输出
│   ├── html.ts             # HTML可视化报告
│   └── json.ts             # JSON/XML标准格式
├── ci/                     # CI配置模板
│   ├── github-actions.yml
│   ├── gitlab-ci.yml
│   └── jenkins.groovy
└── package.json
```

### 2.2 核心命令

```bash
# 运行测试
sop-test run --platform ios --type unit

# 运行所有平台测试
sop-test run --all

# 生成覆盖率报告
sop-test coverage --platform web

# 失败分析
sop-test analyze --report results.json

# CI模式（退出码非0表示失败）
sop-test ci --platform android --output json
```

---

## 3. 平台适配器设计

### 3.1 各平台测试框架映射

| 平台 | 测试框架 | 适配器实现 |
|------|----------|------------|
| **iOS** | XCTest | 调用 `xcodebuild test -scheme`，解析 xcresult |
| **Android** | JUnit + Espresso | 调用 `./gradlew test`，解析 XML 报告 |
| **HarmonyOS** | Hvigor Test | 调用 `./hvigorw test`，解析 JSON 报告 |
| **Web** | Vitest / Jest | 调用 `vitest run` 或 `jest`，解析 JSON 输出 |
| **Backend** | Vitest / Jest | 同 Web，额外支持 API 集成测试 |
| **小程序** | Vitest | 调用 `vitest run`，模拟小程序环境 |
| **游戏** | Jest | 调用 `jest`，Cocos Test Framework 集成 |

### 3.2 适配器接口设计

```typescript
interface TestAdapter {
  // 平台标识
  platform: string

  // 检测项目类型
  detect(projectPath: string): boolean

  // 运行测试
  run(options: TestOptions): Promise<TestResult>

  // 解析原始报告
  parseReport(rawOutput: string): ParsedReport

  // 获取覆盖率
  getCoverage(): Promise<CoverageData>
}

interface TestOptions {
  type: 'unit' | 'integration' | 'all'
  coverage: boolean
  watch: boolean
  parallel: boolean
  filter?: string  // 测试文件过滤
}

interface TestResult {
  success: boolean
  total: number
  passed: number
  failed: number
  skipped: number
  duration: number
  failures: FailureInfo[]
  coverage?: CoverageData
}

interface FailureInfo {
  testPath: string
  testName: string
  error: string
  stack: string
  codeSnippet?: CodeSnippet
}
```

---

## 4. 报告生成器设计

### 4.1 终端报告示例

```
┌─────────────────────────────────────────────────────────────┐
│  SOP Test Report                                             │
│  Platform: iOS • Type: Unit + Integration                    │
│  Duration: 12.5s                                             │
├─────────────────────────────────────────────────────────────┤
│  ✓ PASSED: 45 tests                                          │
│  ✗ FAILED: 3 tests                                           │
│  ○ SKIPPED: 2 tests                                          │
├─────────────────────────────────────────────────────────────┤
│  Coverage: 78.5% ████████████░░░░                            │
│  Statements: 82.1% | Branches: 75.3% | Functions: 79.8%     │
├─────────────────────────────────────────────────────────────┤
│  FAILURES:                                                   │
│                                                              │
│  1. UserServiceTests.testLogin                               │
│     ✗ Expected: true, Received: false                        │
│     📍 UserServices.swift:42                                 │
│     ┌────────────────────────────────────────┐               │
│     │ 40 │ func testLogin() {                │               │
│     │ 41 │   let result = service.login()    │               │
│     │ 42 │   XCTAssertTrue(result.success)   │ ← HERE        │
│     │ 43 │ }                                 │               │
│     └────────────────────────────────────────┘               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 HTML报告结构

```
test-report/
├── index.html              # 总览页
├── suites/                 # 测试套件详情
│   ├── user-service.html
│   └── api-integration.html
├── coverage/               # 覆盖率详情
│   ├── index.html
│   └── src/
│       ├── services/
│       └── models/
└── assets/                 # 样式和脚本
    ├── style.css
    └── script.js
```

### 4.3 JSON报告格式

```json
{
  "summary": {
    "platform": "ios",
    "type": "unit",
    "success": false,
    "total": 50,
    "passed": 45,
    "failed": 3,
    "skipped": 2,
    "duration": 12500
  },
  "coverage": {
    "lines": 78.5,
    "statements": 82.1,
    "branches": 75.3,
    "functions": 79.8
  },
  "failures": [
    {
      "testPath": "UserServiceTests",
      "testName": "testLogin",
      "error": "Expected: true, Received: false",
      "stack": "UserServices.swift:42",
      "codeSnippet": {
        "file": "UserServices.swift",
        "line": 42,
        "context": ["40| func testLogin() {", "41|   let result = service.login()", "42|   XCTAssertTrue(result.success)"]
      }
    }
  ]
}
```

---

## 5. 失败分析器设计

### 5.1 代码关联能力实现

```typescript
class FailureAnalyzer {
  // 解析堆栈追踪，定位源码
  parseStackTrace(stack: string): CodeLocation[] {
    // 匹配文件路径和行号
    const pattern = /at (.+) \((.+):(\d+):\d+\)/
    return stack.split('\n')
      .map(line => pattern.exec(line))
      .filter(Boolean)
      .map(match => ({
        file: match[2],
        line: parseInt(match[3]),
        method: match[1]
      }))
  }

  // 读取源码上下文
  getCodeSnippet(file: string, line: number, context: number = 3): CodeSnippet {
    const content = fs.readFileSync(file, 'utf-8')
    const lines = content.split('\n')
    const start = Math.max(0, line - context - 1)
    const end = Math.min(lines.length, line + context)

    return {
      file,
      line,
      context: lines.slice(start, end).map((l, i) => ({
        number: start + i + 1,
        content: l,
        isTarget: start + i + 1 === line
      }))
    }
  }

  // 智能错误分类
  classifyError(error: string): ErrorType {
    if (error.includes('Expected') && error.includes('Received')) {
      return 'assertion'
    }
    if (error.includes('TypeError') || error.includes('ReferenceError')) {
      return 'runtime'
    }
    if (error.includes('timeout') || error.includes('Timeout')) {
      return 'timeout'
    }
    if (error.includes('network') || error.includes('fetch')) {
      return 'network'
    }
    return 'unknown'
  }
}
```

### 5.2 错误类型与建议

| 错误类型 | 特征 | 建议方向 |
|----------|------|----------|
| **断言失败** | Expected/Received | 检查测试数据、业务逻辑、边界条件 |
| **运行时错误** | TypeError/ReferenceError | 检查变量定义、类型转换、空值处理 |
| **超时错误** | timeout | 检查异步操作、网络请求、死循环 |
| **网络错误** | network/fetch | 检查API地址、网络配置、Mock数据 |
| **配置错误** | Cannot find module | 检查依赖安装、路径配置 |

---

## 6. CI/CD 集成设计

### 6.1 GitHub Actions 配置

```yaml
# .github/workflows/test.yml
name: SOP Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install sop-test
        run: npm install -g @sop/test

      - name: Run tests
        run: sop-test ci --platform web --output json --report test-report.json

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/coverage-final.json

      - name: Upload test report
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: test-report/
```

### 6.2 GitLab CI 配置

```yaml
# .gitlab-ci.yml
stages:
  - test

test:
  stage: test
  image: node:20
  script:
    - npm install -g @sop/test
    - sop-test ci --platform web --output junit --report test-report.xml
  artifacts:
    reports:
      junit: test-report.xml
    paths:
      - coverage/
    expire_in: 1 week
  coverage: '/Coverage: \d+\.\d+%/'
```

### 6.3 Jenkins Pipeline 配置

```groovy
// Jenkinsfile
pipeline {
  agent any

  stages {
    stage('Test') {
      steps {
        sh 'npm install -g @sop/test'
        sh 'sop-test ci --platform web --output junit --report test-report.xml'
      }
      post {
        always {
          junit 'test-report.xml'
          publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'test-report',
            reportFiles: 'index.html',
            reportName: 'Test Report'
          ])
        }
      }
    }
  }
}
```

### 6.4 CI模式输出规范

| CI平台 | 输出格式 | 集成方式 |
|--------|----------|----------|
| GitHub Actions | JSON + HTML | artifact上传 + codecov集成 |
| GitLab CI | JUnit XML | 内置测试报告解析 |
| Jenkins | JUnit XML + HTML | JUnit插件 + HTML Publisher |

---

## 7. 与现有SOP的关系

```
sop-test (独立工具)
├── 服务于: 所有平台SOP的测试需求
├── 调用方式: [VALIDATE] 标签调用 sop-test CLI
├── 扩展性: 新增平台只需添加适配器
└── CI集成: 提供开箱即用的配置模板
```

---

## 8. 后续工作

- [ ] 创建实现计划（writing-plans）
- [ ] 开发 sop-test CLI 工具
- [ ] 实现 iOS/Android/Web 适配器（优先）
- [ ] 实现 HarmonyOS/小程序/游戏 适配器
- [ ] 编写 CI 配置模板文档
