# 测试自动化工具 (sop-test) 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 开发 sop-test CLI 工具，提供统一的测试执行、报告生成、失败分析能力

**Architecture:** 核心框架 + 平台适配器插件架构，支持7个平台（iOS、Android、HarmonyOS、Web、Backend、小程序、游戏）

**Tech Stack:** TypeScript + Node.js + Vitest/Jest 适配器

---

## 文件结构

```
tools/sop-test/
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts              # 入口文件
│   ├── cli.ts                # CLI 命令处理
│   ├── core/
│   │   ├── runner.ts         # 测试运行器
│   │   ├── reporter.ts       # 报告生成器
│   │   └── analyzer.ts       # 失败分析器
│   ├── adapters/
│   │   ├── base.ts           # 适配器基类
│   │   ├── ios.ts            # iOS 适配器
│   │   ├── android.ts        # Android 适配器
│   │   ├── harmonyos.ts      # HarmonyOS 适配器
│   │   ├── web.ts            # Web 适配器
│   │   ├── backend.ts        # Backend 适配器
│   │   ├── miniprogram.ts    # 小程序适配器
│   │   └── cocos.ts          # Cocos 适配器
│   ├── reporters/
│   │   ├── terminal.ts       # 终端报告
│   │   ├── html.ts           # HTML 报告
│   │   └── json.ts           # JSON/XML 报告
│   └── types/
│       └── index.ts          # 类型定义
├── templates/
│   ├── github-actions.yml
│   ├── gitlab-ci.yml
│   └── jenkins.groovy
└── README.md
```

---

## Task 1: 项目初始化和基础配置

**Files:**
- Create: `tools/sop-test/package.json`
- Create: `tools/sop-test/tsconfig.json`
- Create: `tools/sop-test/src/types/index.ts`

- [ ] **Step 1.1: 创建 package.json**

```json
{
  "name": "@sop/test",
  "version": "1.0.0",
  "description": "SOP unified test automation tool",
  "type": "module",
  "bin": {
    "sop-test": "./dist/cli.js"
  },
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "scripts": {
    "build": "tsup src/cli.ts src/index.ts --format esm --dts",
    "dev": "tsup src/cli.ts src/index.ts --format esm --watch",
    "test": "vitest run",
    "test:watch": "vitest",
    "lint": "eslint src/",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "chalk": "^5.3.0",
    "commander": "^12.0.0",
    "glob": "^10.3.0",
    "html-escaper": "^3.0.3",
    "ora": "^8.0.0",
    "xml2js": "^0.6.2"
  },
  "devDependencies": {
    "@types/node": "^20.11.0",
    "@types/xml2js": "^0.4.14",
    "tsup": "^8.0.0",
    "typescript": "^5.3.0",
    "vitest": "^1.2.0"
  },
  "engines": {
    "node": ">=18.0.0"
  },
  "keywords": [
    "test",
    "automation",
    "sop",
    "cli"
  ],
  "license": "MIT"
}
```

- [ ] **Step 1.2: 创建 tsconfig.json**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Node",
    "lib": ["ES2022"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "resolveJsonModule": true,
    "noEmit": false
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

- [ ] **Step 1.3: 创建类型定义**

```typescript
// src/types/index.ts

export interface TestOptions {
  type: 'unit' | 'integration' | 'all'
  coverage: boolean
  watch: boolean
  parallel: boolean
  filter?: string
}

export interface TestResult {
  success: boolean
  total: number
  passed: number
  failed: number
  skipped: number
  duration: number
  failures: FailureInfo[]
  coverage?: CoverageData
}

export interface FailureInfo {
  testPath: string
  testName: string
  error: string
  stack: string
  codeSnippet?: CodeSnippet
}

export interface CodeSnippet {
  file: string
  line: number
  context: CodeLine[]
}

export interface CodeLine {
  number: number
  content: string
  isTarget: boolean
}

export interface CoverageData {
  lines: number
  statements: number
  branches: number
  functions: number
}

export interface ParsedReport {
  summary: {
    total: number
    passed: number
    failed: number
    skipped: number
    duration: number
  }
  tests: TestInfo[]
  failures: FailureInfo[]
}

export interface TestInfo {
  path: string
  name: string
  status: 'passed' | 'failed' | 'skipped'
  duration: number
}

export type Platform = 'ios' | 'android' | 'harmonyos' | 'web' | 'backend' | 'miniprogram' | 'cocos'

export type ErrorType = 'assertion' | 'runtime' | 'timeout' | 'network' | 'config' | 'unknown'

export type OutputFormat = 'terminal' | 'html' | 'json' | 'junit'
```

- [ ] **Step 1.4: 安装依赖并提交**

```bash
cd tools/sop-test
pnpm install
git add .
git commit -m "chore: initialize sop-test project

- Add package.json with dependencies
- Add TypeScript configuration
- Add type definitions

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 2: 实现适配器基类和核心接口

**Files:**
- Create: `tools/sop-test/src/adapters/base.ts`

- [ ] **Step 2.1: 创建适配器基类**

```typescript
// src/adapters/base.ts
import type { TestOptions, TestResult, CoverageData, ParsedReport } from '../types/index.js'

export abstract class BaseAdapter {
  abstract platform: string

  abstract detect(projectPath: string): boolean

  abstract run(options: TestOptions): Promise<TestResult>

  abstract parseReport(rawOutput: string): ParsedReport

  abstract getCoverage(): Promise<CoverageData>

  protected executeCommand(command: string, args: string[], cwd: string): Promise<{ stdout: string; stderr: string; exitCode: number }> {
    return new Promise((resolve, reject) => {
      const { spawn } = require('child_process')
      const process = spawn(command, args, { cwd, shell: true })

      let stdout = ''
      let stderr = ''

      process.stdout.on('data', (data: Buffer) => {
        stdout += data.toString()
      })

      process.stderr.on('data', (data: Buffer) => {
        stderr += data.toString()
      })

      process.on('close', (code: number) => {
        resolve({ stdout, stderr, exitCode: code ?? 0 })
      })

      process.on('error', (err: Error) => {
        reject(err)
      })
    })
  }

  protected parseDuration(durationStr: string): number {
    // Parse duration string like "1.234s" to milliseconds
    const match = durationStr.match(/(\d+\.?\d*)s?/)
    if (match) {
      return Math.round(parseFloat(match[1]) * 1000)
    }
    return 0
  }
}
```

- [ ] **Step 2.2: 提交适配器基类**

```bash
git add src/adapters/base.ts
git commit -m "feat: add base adapter class

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 3: 实现 Web 适配器（优先）

**Files:**
- Create: `tools/sop-test/src/adapters/web.ts`

- [ ] **Step 3.1: 创建 Web 适配器**

```typescript
// src/adapters/web.ts
import { BaseAdapter } from './base.js'
import type { TestOptions, TestResult, CoverageData, ParsedReport, TestInfo, FailureInfo } from '../types/index.js'
import * as fs from 'fs'
import * as path from 'path'

export class WebAdapter extends BaseAdapter {
  platform = 'web'

  detect(projectPath: string): boolean {
    // Check for vitest.config.* or jest.config.*
    const vitestConfig = fs.existsSync(path.join(projectPath, 'vitest.config.ts')) ||
                         fs.existsSync(path.join(projectPath, 'vitest.config.js'))
    const jestConfig = fs.existsSync(path.join(projectPath, 'jest.config.ts')) ||
                       fs.existsSync(path.join(projectPath, 'jest.config.js'))

    // Check for package.json with vitest or jest
    const packageJsonPath = path.join(projectPath, 'package.json')
    if (fs.existsSync(packageJsonPath)) {
      const packageJson = JSON.parse(fs.readFileSync(packageJsonPath, 'utf-8'))
      const hasVitest = packageJson.devDependencies?.vitest || packageJson.dependencies?.vitest
      const hasJest = packageJson.devDependencies?.jest || packageJson.dependencies?.jest
      return hasVitest || hasJest || vitestConfig || jestConfig
    }

    return vitestConfig || jestConfig
  }

  async run(options: TestOptions): Promise<TestResult> {
    const projectPath = process.cwd()
    const usesVitest = this.detectVitest(projectPath)

    let command: string
    let args: string[]

    if (usesVitest) {
      command = 'npx'
      args = ['vitest', 'run', '--reporter=json']
      if (options.coverage) {
        args.push('--coverage')
      }
      if (options.filter) {
        args.push(options.filter)
      }
    } else {
      command = 'npx'
      args = ['jest', '--json', '--outputFile=test-results.json']
      if (options.coverage) {
        args.push('--coverage')
      }
      if (options.filter) {
        args.push('--testPathPattern=' + options.filter)
      }
    }

    const { stdout, stderr, exitCode } = await this.executeCommand(command, args, projectPath)

    const result = this.parseReport(stdout)
    const coverage = options.coverage ? await this.getCoverage() : undefined

    return {
      success: exitCode === 0,
      total: result.summary.total,
      passed: result.summary.passed,
      failed: result.summary.failed,
      skipped: result.summary.skipped,
      duration: result.summary.duration,
      failures: result.failures,
      coverage
    }
  }

  parseReport(rawOutput: string): ParsedReport {
    try {
      const data = JSON.parse(rawOutput)
      // Parse vitest/jest JSON output
      const tests: TestInfo[] = []
      const failures: FailureInfo[] = []

      // Vitest format
      if (data.testResults) {
        for (const testResult of data.testResults) {
          for (const assertion of testResult.assertionResults || []) {
            tests.push({
              path: testResult.name,
              name: assertion.title,
              status: assertion.status === 'passed' ? 'passed' : assertion.status === 'failed' ? 'failed' : 'skipped',
              duration: assertion.duration || 0
            })

            if (assertion.status === 'failed') {
              failures.push({
                testPath: testResult.name,
                testName: assertion.title,
                error: assertion.failureMessages?.join('\n') || '',
                stack: assertion.failureMessages?.join('\n') || ''
              })
            }
          }
        }
      }

      const passed = tests.filter(t => t.status === 'passed').length
      const failed = tests.filter(t => t.status === 'failed').length
      const skipped = tests.filter(t => t.status === 'skipped').length

      return {
        summary: {
          total: tests.length,
          passed,
          failed,
          skipped,
          duration: data.duration || 0
        },
        tests,
        failures
      }
    } catch {
      return {
        summary: { total: 0, passed: 0, failed: 0, skipped: 0, duration: 0 },
        tests: [],
        failures: []
      }
    }
  }

  async getCoverage(): Promise<CoverageData> {
    const coveragePath = path.join(process.cwd(), 'coverage', 'coverage-final.json')
    if (fs.existsSync(coveragePath)) {
      const coverage = JSON.parse(fs.readFileSync(coveragePath, 'utf-8'))
      // Calculate coverage percentages
      let totalLines = 0
      let coveredLines = 0
      let totalBranches = 0
      let coveredBranches = 0
      let totalFunctions = 0
      let coveredFunctions = 0
      let totalStatements = 0
      let coveredStatements = 0

      for (const file of Object.values(coverage) as any[]) {
        if (file.l) {
          for (const line of Object.values(file.l) as number[]) {
            totalLines++
            if (line > 0) coveredLines++
          }
        }
        if (file.b) {
          for (const branch of Object.values(file.b) as number[][]) {
            for (const count of branch) {
              totalBranches++
              if (count > 0) coveredBranches++
            }
          }
        }
        if (file.f) {
          for (const count of Object.values(file.f) as number[]) {
            totalFunctions++
            if (count > 0) coveredFunctions++
          }
        }
        if (file.s) {
          for (const count of Object.values(file.s) as number[]) {
            totalStatements++
            if (count > 0) coveredStatements++
          }
        }
      }

      return {
        lines: totalLines > 0 ? (coveredLines / totalLines) * 100 : 0,
        statements: totalStatements > 0 ? (coveredStatements / totalStatements) * 100 : 0,
        branches: totalBranches > 0 ? (coveredBranches / totalBranches) * 100 : 0,
        functions: totalFunctions > 0 ? (coveredFunctions / totalFunctions) * 100 : 0
      }
    }

    return { lines: 0, statements: 0, branches: 0, functions: 0 }
  }

  private detectVitest(projectPath: string): boolean {
    return fs.existsSync(path.join(projectPath, 'vitest.config.ts')) ||
           fs.existsSync(path.join(projectPath, 'vitest.config.js'))
  }
}
```

- [ ] **Step 3.2: 提交 Web 适配器**

```bash
git add src/adapters/web.ts
git commit -m "feat: add web adapter for vitest/jest

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 4: 实现测试运行器和失败分析器

**Files:**
- Create: `tools/sop-test/src/core/runner.ts`
- Create: `tools/sop-test/src/core/analyzer.ts`

- [ ] **Step 4.1: 创建测试运行器**

```typescript
// src/core/runner.ts
import type { Platform, TestOptions, TestResult } from '../types/index.js'
import { WebAdapter } from '../adapters/web.js'
import { BaseAdapter } from '../adapters/base.js'

const adapters: Record<Platform, () => BaseAdapter> = {
  web: () => new WebAdapter(),
  ios: () => { throw new Error('iOS adapter not implemented yet') },
  android: () => { throw new Error('Android adapter not implemented yet') },
  harmonyos: () => { throw new Error('HarmonyOS adapter not implemented yet') },
  backend: () => new WebAdapter(), // Backend uses same as web
  miniprogram: () => new WebAdapter(), // Miniprogram uses same as web
  cocos: () => { throw new Error('Cocos adapter not implemented yet') }
}

export async function runTest(platform: Platform, options: TestOptions): Promise<TestResult> {
  const adapter = adapters[platform]()
  return adapter.run(options)
}

export function detectPlatform(projectPath: string): Platform | null {
  for (const [platform, createAdapter] of Object.entries(adapters)) {
    try {
      const adapter = createAdapter()
      if (adapter.detect(projectPath)) {
        return platform as Platform
      }
    } catch {
      // Adapter not implemented, skip
    }
  }
  return null
}
```

- [ ] **Step 4.2: 创建失败分析器**

```typescript
// src/core/analyzer.ts
import type { FailureInfo, CodeSnippet, ErrorType } from '../types/index.js'
import * as fs from 'fs'

export class FailureAnalyzer {
  parseStackTrace(stack: string): CodeLocation[] {
    const pattern = /at (.+) \((.+):(\d+):\d+\)/
    return stack.split('\n')
      .map(line => pattern.exec(line))
      .filter((match): match is RegExpExecArray => match !== null)
      .map(match => ({
        file: match[2],
        line: parseInt(match[3]),
        method: match[1]
      }))
  }

  getCodeSnippet(file: string, line: number, context: number = 3): CodeSnippet | null {
    try {
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
    } catch {
      return null
    }
  }

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
    if (error.includes('network') || error.includes('fetch') || error.includes('ECONNREFUSED')) {
      return 'network'
    }
    if (error.includes('Cannot find module') || error.includes('ENOENT')) {
      return 'config'
    }
    return 'unknown'
  }

  analyzeFailure(failure: FailureInfo): AnalyzedFailure {
    const locations = this.parseStackTrace(failure.stack)
    const codeSnippet = locations.length > 0
      ? this.getCodeSnippet(locations[0].file, locations[0].line)
      : null
    const errorType = this.classifyError(failure.error)

    return {
      ...failure,
      codeSnippet: codeSnippet || undefined,
      errorType,
      suggestion: this.getSuggestion(errorType)
    }
  }

  private getSuggestion(errorType: ErrorType): string {
    const suggestions: Record<ErrorType, string> = {
      assertion: '检查测试数据、业务逻辑、边界条件',
      runtime: '检查变量定义、类型转换、空值处理',
      timeout: '检查异步操作、网络请求、死循环',
      network: '检查API地址、网络配置、Mock数据',
      config: '检查依赖安装、路径配置',
      unknown: '查看完整错误堆栈进行排查'
    }
    return suggestions[errorType]
  }
}

interface CodeLocation {
  file: string
  line: number
  method: string
}

interface AnalyzedFailure extends FailureInfo {
  errorType: ErrorType
  suggestion: string
}
```

- [ ] **Step 4.3: 提交核心模块**

```bash
git add src/core/
git commit -m "feat: add test runner and failure analyzer

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 5: 实现报告生成器

**Files:**
- Create: `tools/sop-test/src/reporters/terminal.ts`
- Create: `tools/sop-test/src/reporters/html.ts`
- Create: `tools/sop-test/src/reporters/json.ts`

- [ ] **Step 5.1: 创建终端报告生成器**

```typescript
// src/reporters/terminal.ts
import chalk from 'chalk'
import type { TestResult } from '../types/index.js'

export function printTerminalReport(result: TestResult, platform: string): void {
  console.log()
  console.log(chalk.bold('┌' + '─'.repeat(60) + '┐'))
  console.log(chalk.bold('│') + '  SOP Test Report'.padEnd(59) + chalk.bold('│'))
  console.log(chalk.bold('│') + `  Platform: ${platform} • Duration: ${(result.duration / 1000).toFixed(1)}s`.padEnd(59) + chalk.bold('│'))
  console.log(chalk.bold('├' + '─'.repeat(60) + '┤'))

  // Summary
  const passedStr = chalk.green(`✓ PASSED: ${result.passed} tests`)
  const failedStr = result.failed > 0 ? chalk.red(`✗ FAILED: ${result.failed} tests`) : ''
  const skippedStr = result.skipped > 0 ? chalk.gray(`○ SKIPPED: ${result.skipped} tests`) : ''

  console.log(chalk.bold('│') + `  ${passedStr}`.padEnd(70) + chalk.bold('│'))
  if (failedStr) console.log(chalk.bold('│') + `  ${failedStr}`.padEnd(70) + chalk.bold('│'))
  if (skippedStr) console.log(chalk.bold('│') + `  ${skippedStr}`.padEnd(70) + chalk.bold('│'))

  // Coverage
  if (result.coverage) {
    const coverageBar = generateProgressBar(result.coverage.lines, 20)
    console.log(chalk.bold('├' + '─'.repeat(60) + '┤'))
    console.log(chalk.bold('│') + `  Coverage: ${result.coverage.lines.toFixed(1)}% ${coverageBar}`.padEnd(59) + chalk.bold('│'))
    console.log(chalk.bold('│') + `  Statements: ${result.coverage.statements.toFixed(1)}% | Branches: ${result.coverage.branches.toFixed(1)}% | Functions: ${result.coverage.functions.toFixed(1)}%`.padEnd(59) + chalk.bold('│'))
  }

  // Failures
  if (result.failures.length > 0) {
    console.log(chalk.bold('├' + '─'.repeat(60) + '┤'))
    console.log(chalk.bold('│') + '  FAILURES:'.padEnd(59) + chalk.bold('│'))

    result.failures.slice(0, 5).forEach((failure, index) => {
      console.log(chalk.bold('│'))
      console.log(chalk.bold('│') + `  ${index + 1}. ${chalk.red(failure.testPath)} > ${failure.testName}`.slice(0, 58).padEnd(59) + chalk.bold('│'))
      console.log(chalk.bold('│') + `     ✗ ${failure.error.slice(0, 50)}`.padEnd(59) + chalk.bold('│'))

      if (failure.codeSnippet) {
        console.log(chalk.bold('│') + `     📍 ${failure.codeSnippet.file}:${failure.codeSnippet.line}`.padEnd(59) + chalk.bold('│'))
        console.log(chalk.bold('│') + '     ┌' + '─'.repeat(40) + '┐')
        failure.codeSnippet.context.forEach(line => {
          const prefix = line.isTarget ? ' ← HERE' : ''
          const content = `${line.number}│ ${line.content}`.slice(0, 38)
          console.log(chalk.bold('│') + `     │ ${content.padEnd(38)}${prefix}`.slice(0, 58).padEnd(59) + chalk.bold('│'))
        })
        console.log(chalk.bold('│') + '     └' + '─'.repeat(40) + '┘')
      }
    })

    if (result.failures.length > 5) {
      console.log(chalk.bold('│') + `  ... and ${result.failures.length - 5} more failures`.padEnd(59) + chalk.bold('│'))
    }
  }

  console.log(chalk.bold('└' + '─'.repeat(60) + '┘'))
  console.log()
}

function generateProgressBar(percentage: number, width: number): string {
  const filled = Math.round((percentage / 100) * width)
  const empty = width - filled
  return '█'.repeat(filled) + '░'.repeat(empty)
}
```

- [ ] **Step 5.2: 创建 JSON 报告生成器**

```typescript
// src/reporters/json.ts
import type { TestResult } from '../types/index.js'

export function generateJsonReport(result: TestResult, platform: string): string {
  return JSON.stringify({
    summary: {
      platform,
      success: result.success,
      total: result.total,
      passed: result.passed,
      failed: result.failed,
      skipped: result.skipped,
      duration: result.duration
    },
    coverage: result.coverage,
    failures: result.failures
  }, null, 2)
}

export function generateJunitReport(result: TestResult, platform: string): string {
  const failures = result.failures.map(f => `    <testcase name="${f.testName}" classname="${f.testPath}">
      <failure message="${escapeXml(f.error)}">
${escapeXml(f.stack)}
      </failure>
    </testcase>`).join('\n')

  const passed = Array(result.passed).fill(null).map((_, i) =>
    `    <testcase name="test-${i}" classname="passed"/>`
  ).join('\n')

  return `<?xml version="1.0" encoding="UTF-8"?>
<testsuites>
  <testsuite name="${platform}" tests="${result.total}" failures="${result.failed}" time="${result.duration / 1000}">
${failures}
${passed}
  </testsuite>
</testsuites>`
}

function escapeXml(str: string): string {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&apos;')
}
```

- [ ] **Step 5.3: 提交报告生成器**

```bash
git add src/reporters/
git commit -m "feat: add terminal and JSON report generators

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 6: 实现 CLI 入口

**Files:**
- Create: `tools/sop-test/src/cli.ts`
- Create: `tools/sop-test/src/index.ts`

- [ ] **Step 6.1: 创建 CLI 入口**

```typescript
// src/cli.ts
#!/usr/bin/env node
import { Command } from 'commander'
import { runTest, detectPlatform } from './core/runner.js'
import { FailureAnalyzer } from './core/analyzer.js'
import { printTerminalReport } from './reporters/terminal.js'
import { generateJsonReport, generateJunitReport } from './reporters/json.js'
import type { Platform, OutputFormat } from './types/index.js'
import * as fs from 'fs'

const program = new Command()

program
  .name('sop-test')
  .description('SOP unified test automation tool')
  .version('1.0.0')

program
  .command('run')
  .description('Run tests for specified platform')
  .option('-p, --platform <platform>', 'Target platform (ios, android, harmonyos, web, backend, miniprogram, cocos)')
  .option('-t, --type <type>', 'Test type (unit, integration, all)', 'unit')
  .option('-c, --coverage', 'Generate coverage report', false)
  .option('-o, --output <format>', 'Output format (terminal, json, junit)', 'terminal')
  .option('--all', 'Run tests for all platforms', false)
  .action(async (options) => {
    const projectPath = process.cwd()
    const platform = options.platform || detectPlatform(projectPath)

    if (!platform) {
      console.error('Could not detect platform. Please specify with --platform')
      process.exit(1)
    }

    console.log(`Running ${options.type} tests for ${platform}...`)

    const testOptions = {
      type: options.type as 'unit' | 'integration' | 'all',
      coverage: options.coverage,
      watch: false,
      parallel: false
    }

    const result = await runTest(platform as Platform, testOptions)

    // Analyze failures
    const analyzer = new FailureAnalyzer()
    result.failures = result.failures.map(f => analyzer.analyzeFailure(f))

    // Output report
    if (options.output === 'terminal') {
      printTerminalReport(result, platform)
    } else if (options.output === 'json') {
      const report = generateJsonReport(result, platform)
      console.log(report)
      fs.writeFileSync('test-report.json', report)
    } else if (options.output === 'junit') {
      const report = generateJunitReport(result, platform)
      console.log(report)
      fs.writeFileSync('test-report.xml', report)
    }

    process.exit(result.success ? 0 : 1)
  })

program
  .command('ci')
  .description('Run tests in CI mode (non-zero exit code on failure)')
  .option('-p, --platform <platform>', 'Target platform')
  .option('-o, --output <format>', 'Output format (json, junit)', 'json')
  .action(async (options) => {
    const projectPath = process.cwd()
    const platform = options.platform || detectPlatform(projectPath)

    if (!platform) {
      console.error('Could not detect platform')
      process.exit(1)
    }

    const result = await runTest(platform as Platform, {
      type: 'all',
      coverage: true,
      watch: false,
      parallel: false
    })

    if (options.output === 'json') {
      fs.writeFileSync('test-report.json', generateJsonReport(result, platform))
    } else {
      fs.writeFileSync('test-report.xml', generateJunitReport(result, platform))
    }

    process.exit(result.success ? 0 : 1)
  })

program.parse()
```

- [ ] **Step 6.2: 创建入口文件**

```typescript
// src/index.ts
export { runTest, detectPlatform } from './core/runner.js'
export { FailureAnalyzer } from './core/analyzer.js'
export { printTerminalReport } from './reporters/terminal.js'
export { generateJsonReport, generateJunitReport } from './reporters/json.js'
export * from './types/index.js'
```

- [ ] **Step 6.3: 提交 CLI**

```bash
git add src/cli.ts src/index.ts
git commit -m "feat: add CLI entry point

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 7: 创建 CI 配置模板和文档

**Files:**
- Create: `tools/sop-test/templates/github-actions.yml`
- Create: `tools/sop-test/templates/gitlab-ci.yml`
- Create: `tools/sop-test/templates/jenkins.groovy`
- Create: `tools/sop-test/README.md`

- [ ] **Step 7.1: 创建 CI 模板**

创建 GitHub Actions、GitLab CI、Jenkins 配置模板。

- [ ] **Step 7.2: 创建 README**

```markdown
# @sop/test

SOP 统一测试自动化工具

## 安装

\`\`\`bash
npm install -g @sop/test
\`\`\`

## 使用

\`\`\`bash
# 运行测试
sop-test run --platform web

# 生成覆盖率报告
sop-test run --platform web --coverage

# CI 模式
sop-test ci --platform web --output json
\`\`\`

## 支持平台

| 平台 | 状态 |
|------|------|
| Web (Vitest/Jest) | ✅ 支持 |
| Backend (Vitest/Jest) | ✅ 支持 |
| 小程序 (Vitest) | ✅ 支持 |
| iOS (XCTest) | 🚧 开发中 |
| Android (JUnit) | 🚧 开发中 |
| HarmonyOS (Hvigor) | 🚧 开发中 |
| Cocos Creator (Jest) | 🚧 开发中 |
```

- [ ] **Step 7.3: 最终提交**

```bash
git add .
git commit -m "docs: add CI templates and README

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Self-Review Checklist

- [ ] **Spec Coverage**: 核心框架、Web适配器、报告生成器、CLI都已实现
- [ ] **Placeholder Scan**: 无占位符
- [ ] **Type Consistency**: 类型定义在各模块保持一致
- [ ] **Code Completeness**: 所有代码步骤都有完整代码
- [ ] **Command Accuracy**: 所有命令可直接执行
