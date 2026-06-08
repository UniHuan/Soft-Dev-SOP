# 🔧 CI/CD 集成 SOP — iOS + 鸿蒙 双平台自动化

> **适用对象**: Claude Code + GitHub Actions / hvigor CI
> **目标**: push → build → test → archive → deploy 全自动
> **参考 SOP**: `ios/iOS_App_2Day_Development_SOP.md` + `harmonyos/HarmonyOS_App_2Day_Development_SOP.md`

---

## 🧠 Claude Code 技能调用

| 技能 | 说明 |
|------|------|
| `[SHELL]` | 执行 bash、创建 yml 配置 |
| `[WRITE]` | 创建 GitHub Actions / hvigor 配置 |
| `[DIALOG]` | 确认 Secrets 配置 |
| `[VALIDATE]` | 验证 CI 流水线通过 |

---

## 📋 iOS — GitHub Actions

### Step 1: 创建 Workflow

> **[WRITE]** `.github/workflows/ios-ci.yml`:

```yaml
name: iOS CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: macos-15
    timeout-minutes: 30

    steps:
      - uses: actions/checkout@v4

      - name: Select Xcode
        run: sudo xcode-select -s /Applications/Xcode.app

      - name: Build
        run: |
          xcodebuild build \
            -project ${PROJECT_NAME}.xcodeproj \
            -scheme ${PROJECT_NAME} \
            -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
            -quiet

      - name: Test
        run: |
          xcodebuild test \
            -project ${PROJECT_NAME}.xcodeproj \
            -scheme ${PROJECT_NAME} \
            -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
            -only-testing:${PROJECT_NAME}Tests

      - name: Archive (main only)
        if: github.ref == 'refs/heads/main'
        run: |
          xcodebuild archive \
            -project ${PROJECT_NAME}.xcodeproj \
            -scheme ${PROJECT_NAME} \
            -archivePath ./build/App.xcarchive \
            -destination 'generic/platform=iOS' \
            -allowProvisioningUpdates
        env:
          APPSTORE_KEY_ID: ${{ secrets.APPSTORE_KEY_ID }}
          APPSTORE_ISSUER_ID: ${{ secrets.APPSTORE_ISSUER_ID }}
          APPSTORE_KEY_PATH: ${{ secrets.APPSTORE_KEY_PATH }}
```

### Step 2: 配置 Secrets

> **[DIALOG]** 指导用户在 GitHub 仓库配置:

```
GitHub Repo → Settings → Secrets and variables → Actions → New secret:

□ APPSTORE_KEY_ID      — App Store Connect API Key ID
□ APPSTORE_ISSUER_ID   — App Store Connect Issuer ID
□ APPSTORE_KEY_CONTENT — .p8 私钥文件内容 (base64)
□ MATCH_PASSWORD       — Fastlane Match 密码 (如使用)
```

### Step 3: 添加 Fastlane (可选)

```ruby
# fastlane/Fastfile
platform :ios do
  lane :ci_build do
    run_tests(scheme: ENV['PROJECT_NAME'])
    build_app(scheme: ENV['PROJECT_NAME'], export_method: 'app-store')
  end
end
```

---

## 📋 鸿蒙 — hvigor CI + 自建 Runner

### Step 1: 创建 hvigor CI 配置

> **[WRITE]** `.ci/build.yml` (DevEco Studio CI 配置):

```yaml
version: '1.0'

jobs:
  build:
    name: Build & Test
    runs-on: self-hosted
    steps:
      - name: Checkout
        uses: checkout

      - name: Clean Build
        run: |
          ./hvigorw clean
          ./hvigorw assembleHap

      - name: Unit Test
        run: |
          ./hvigorw test

      - name: Archive (main only)
        if: branch == 'main'
        run: |
          ./hvigorw assembleApp
          # 产物: build/outputs/default/app-default-signed.app
```

### Step 2: 自建 Runner 配置

> **[DIALOG]** 指导用户配置 DevEco Studio 构建环境:

```bash
# 1. 在 CI 机器上安装 DevEco Studio
# 2. 配置环境变量
export DEVECO_SDK_HOME="$HOME/Library/Huawei/devecostudio/sdk"
export JAVA_HOME=$(/usr/libexec/java_home -v 17)

# 3. 注册 Runner
# GitHub: Settings → Actions → Runners → New self-hosted runner
# GitLab: Settings → CI/CD → Runners → Register
```

### Step 3: GitHub Actions 调用 hvigor

```yaml
# .github/workflows/harmonyos-ci.yml
name: HarmonyOS CI

on: [push, pull_request]

jobs:
  build:
    runs-on: [self-hosted, macos]
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: |
          source /tmp/sop_harmony.env 2>/dev/null
          cd ${PROJECT_DIR}
          ./hvigorw assembleHap
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: harmonyos-app
          path: build/outputs/default/*.app
```

---

## 📋 提交前自动检查清单

> **[VALIDATE]** CI 流水线应包含的质量门禁:

```
iOS CI 门禁:
□ Build 通过 (xcodebuild build)
□ Test 通过 (xcodebuild test)
□ Archive 成功 (main 分支)
□ 无 SwiftLint 错误

鸿蒙 CI 门禁:
□ Build 通过 (./hvigorw assembleHap)
□ Lint 通过 (ArkTS 规范检查)
□ Archive 成功 (main 分支)

通用门禁:
□ 版本号已更新 (CFBundleVersion / versionCode)
□ 无敏感信息泄露 (git-secrets scan)
□ CHANGELOG 已更新
```

---

## 🚀 一键部署脚本

```bash
#!/bin/bash
# [SHELL] 一键初始化 CI/CD 配置
# 使用: bash setup_cicd.sh [ios|harmonyos|all]

PLATFORM=${1:-all}

setup_ios_ci() {
  echo "📱 配置 iOS CI..."
  mkdir -p .github/workflows
  # Claude Code 用 Edit 创建上述 ios-ci.yml
  echo "  ✅ .github/workflows/ios-ci.yml"
}

setup_harmonyos_ci() {
  echo "🦋 配置鸿蒙 CI..."
  mkdir -p .ci
  # Claude Code 用 Edit 创建上述 build.yml
  echo "  ✅ .ci/build.yml"
}

case $PLATFORM in
  ios)      setup_ios_ci ;;
  harmonyos) setup_harmonyos_ci ;;
  all)      setup_ios_ci; setup_harmonyos_ci ;;
  *)        echo "用法: bash setup_cicd.sh [ios|harmonyos|all]" ;;
esac

echo "✅ CI/CD 配置完成!"
echo "   下一步: 配置 Secrets → push → 验证 CI 通过"
```

---

> **SOP 版本**: 1.0.0 | **最后更新**: 2026-06-08
> **关联文档**: `../ios/iOS_App_2Day_Development_SOP.md` + `../harmonyos/HarmonyOS_App_2Day_Development_SOP.md`
