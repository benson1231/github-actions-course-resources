# GitHub Actions Events 全面指南

GitHub Actions 的工作流程（Workflow）是由不同的 **事件（events）** 觸發的。了解各種事件的行為，能幫助你設計更有效率、更安全、也更符合 CI/CD 需求的自動化流程。

本文件整理了常見事件、使用情境、以及官方文件連結，並提供範例讓你可以快速上手。

---

## 🚀 什麼是 GitHub Actions Events？

Event 是用來**觸發 workflow 的條件**。例如：

* push 到某個 branch
* 有人開 PR
* 新版本釋出（release）
* CRON 排程時間到
* issue 被開啟、關閉

你的 workflow 就會根據這些事件自動執行。

官方 Docs： [https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

---

# 🔥 常見 GitHub Actions Events

## 1. `push`

當程式碼推送到任何分支（或指定分支）時觸發。

```yaml
ame: On Push
on:
  push:
    branches:
      - main
      - dev
```

### 常見用途

* 自動測試
* 自動部署
* Lint 檢查

---

## 2. `pull_request`

在 PR 建立、同步、重新開啟時觸發。

```yaml
ame: On PR
on:
  pull_request:
    branches:
      - main
```

### 常見用途

* 跑 PR 測試、Lint
* 在 PR 上留言檢查結果

---

## 3. `workflow_dispatch`（手動觸發）

允許你從 GitHub UI 手動按下 **Run workflow**。

```yaml
ame: Manual Run
on:
  workflow_dispatch:
    inputs:
      env:
        description: "選擇環境"
        required: true
        default: "prod"
```

### 常見用途

* 手動部署
* 手動觸發 ETL / Batch 任務

---

## 4. `schedule`（排程）

透過 cron 自動定時執行。

```yaml
ame: Daily Job
on:
  schedule:
    - cron: "0 18 * * *"  # 每天臺灣時間 02:00
```

### 常見用途

* 每日報表
* 定期備份
* 健康檢查

---

## 5. `release`

當發布版本（release）時觸發。

```yaml
ame: On Release
on:
  release:
    types: [created]
```

### 常見用途

* Build / Upload binaries
* 自動建立 changelog

---

## 6. `workflow_run`

當 **其他 workflow 執行完後** 觸發（適合分段 CI/CD）。

```yaml
ame: Deploy After Test
on:
  workflow_run:
    workflows: ["Test Workflow"]
    types:
      - completed
```

### 常見用途

* 在測試通過後才部署

---

## 7. Issue / PR 事件（如：`issues`、`issue_comment`、`pull_request_review`）

```yaml
ame: On Issue Comment
on:
  issue_comment:
    types: [created]
```

### 常見用途

* Bot 自動回覆
* 自動標籤（label）管理

---

# 📌 常用 Event 對照表

| Event               | 什麼時候觸發            | 常見用途                 |
| ------------------- | ----------------- | -------------------- |
| `push`              | 推送程式碼             | CI、Lint、自動部署         |
| `pull_request`      | PR 建立/更新          | 檢查 PR、跑測試            |
| `workflow_dispatch` | 手動按下 Run workflow | 手動部署、工具任務            |
| `schedule`          | Cron 時間到          | 排程任務、ETL             |
| `release`           | 新版本釋出             | Build artifacts、發布套件 |
| `workflow_run`      | 另一 workflow 完成    | 多階段 pipeline         |
| `issue_comment`     | Issue/PR 留言       | Bot 回覆、自動管理          |

---

# 🧪 範例：同時支援 push + PR + 手動觸發

```yaml
ame: CI Pipeline
on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm test
```

---

# 🎯 小結

Event 是 GitHub Actions 自動化的核心，你可以：

* 用 `push` 執行 CI
* 用 `pull_request` 檢查 PR
* 用 `workflow_dispatch` 手動觸發部署
* 用 `schedule` 做定時任務

