# Download slide made from [academind/github-actions-course-resources](https://github.com/academind/github-actions-course-resources/blob/main/Slides/github-actions.pdf)

# GitHub Actions 基本架構（Workflow Skeleton）

這份文件提供 **最常用、最標準的 GitHub Actions 基本架構**，適合作為任何 workflow 的起始模板。

---

# 📌 Workflow 基本結構示意

```yaml
name: My Workflow

# 🟦 觸發條件（Events）
on:
  push:
    branches: [ main ]
  pull_request:
  workflow_dispatch:   # 手動觸發

# 🟩 工作集合（Jobs）
jobs:
  example-job:
    runs-on: ubuntu-latest   # 使用哪個 Runner

    # 🟧 steps：一個 job 由多個步驟組成
    steps:
      # 1. 把 repo checkout 下來（任何 workflow 幾乎必備）
      - name: Checkout code
        uses: actions/checkout@v3

      # 2. 安裝環境（使用第三方 action）
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20

      # 3. 執行 Shell 指令
      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      # 4. 使用 outputs / artifacts / cache (按需加入)
```

---

# 🧩 Workflow 元件拆解

## 1️⃣ name — Workflow 名稱

```yaml
name: CI Pipeline
```

可任意命名，用於 GitHub Actions UI 顯示。

---

## 2️⃣ on — 觸發事件

```yaml
on:
  push:
    branches: [ main ]
  pull_request:
  schedule:
    - cron: '0 2 * * *'   # 每天 2 AM
  workflow_dispatch:
```

常用 Events：

* `push`
* `pull_request`
* `workflow_dispatch`（手動觸發）
* `schedule`（排程）
* `release`
* `workflow_run`

---

## 3️⃣ jobs — Workflow 的核心

一個 workflow 可以有多個 jobs，每個 job 可以：

* 平行執行
* 使用 `needs:` 指定依賴

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

  test:
    runs-on: ubuntu-latest
    needs: build
```

---

## 4️⃣ runs-on — 指定 Runner

```yaml
runs-on: ubuntu-latest
```

可用選項：

* `ubuntu-latest`
* `windows-latest`
* `macos-latest`
* 自架 Runner

---

## 5️⃣ steps — Job 的執行步驟

典型步驟：

### ✔ Checkout 程式碼（幾乎所有 workflow 都會用）

```yaml
- uses: actions/checkout@v3
```

### ✔ 使用第三方 Action

```yaml
- uses: actions/setup-node@v3
  with:
    node-version: 20
```

### ✔ 執行 Shell 指令

```yaml
- run: echo "Hello GitHub Actions!"
```

### ✔ 設定環境變數

```yaml
- run: echo "VERSION=1.0.0" >> $GITHUB_ENV
```

---

# 🎯 最標準的 Workflow 基本模板

```yaml
name: Basic CI

on:
  push:
    branches: [ main ]
  pull_request:
  workflow_dispatch:

jobs:
  ci:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build project
        run: npm run build
```

---

# 📚 官方文件

* Workflow syntax：[https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
* Events：[https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
* Actions marketplace：[https://github.com/marketplace?type=actions](https://github.com/marketplace?type=actions)

