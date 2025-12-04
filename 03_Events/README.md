# 🚀 GitHub Actions Events 觸發事件完整指南

GitHub Actions 的核心概念之一，就是「**事件（event）觸發 workflow**」。
只要發生某個事件，例如 push、建立 Issue、PR 更新、tag 被建立⋯⋯Workflow 就會自動啟動。

本文件整理 GitHub Actions 常用事件、使用方式、範例與最佳實踐，適合作為快速查詢與教學筆記使用。

---

# 📌 什麼是 GitHub Actions 的事件（Event）？

GitHub Actions 的 workflow **並不會自己執行**。
它需要一個 **事件（event）** 觸發，例如：

* 推送程式碼（push）
* 發 PR（pull_request）
* 建 Issue（issues）
* 建立 Release（release）
* 系統排程（schedule）

事件就像是 CI/CD 的入口開關。

---

# 🧩 基本語法：`on:`

```yaml
on: <事件名稱>
```

最基本範例：

```yaml
on: push
```

也可以接受多個事件：

```yaml
on: [push, pull_request]
```

或是使用物件格式：

```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```

---

# 🟦 常用事件大補帖

以下整理最常用事件與說明：

## 1️⃣ `push`（有人 push 代碼時觸發）

```yaml
on: push
```

可以指定 branch：

```yaml
on:
  push:
    branches:
      - main
      - dev
```

也可指定檔案條件：

```yaml
on:
  push:
    paths:
      - "src/**"
```

用於：

* 自動跑 lint/test
* CI 基本流程

---

## 2️⃣ `pull_request`（有人建立或更新 PR 時）

```yaml
on: pull_request
```

可指定事件類型：

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened]
```

用於：

* PR 驗證（Lint/Test）
* 防止壞程式碼被合併

---

## 3️⃣ `workflow_dispatch`（手動觸發）

```yaml
on: workflow_dispatch
```

可加入自訂參數：

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [dev, staging, prod]
```

適合：

* 手動部署
* 手動重跑工作流程

---

## 4️⃣ `schedule`（類 cron 排程）

```yaml
on:
  schedule:
    - cron: "0 0 * * *"   # 每天午夜
```

Cron 格式（UTC 時區）：

```
分鐘 小時 日 月 星期
```

用於：

* 每天自動備份
* 每周健康檢查
* 自動清除 cache

---

## 5️⃣ `issues`（Issue 建立/編輯/關閉）

```yaml
on:
  issues:
    types: [opened, edited]
```

用於：

* 自動加 label
* 自動回覆（bot）
* 偵測 Issue 類別

---

## 6️⃣ `release`（新 Release 發布）

```yaml
on:
  release:
    types: [published]
```

用於：

* build + upload release artifacts
* 自動通知

---

## 7️⃣ `create`（建立 branch / tag）

```yaml
on:
  create:
    tags:
      - "v*"
```

用於：

* 當你 push 一個 tag → 自動部署
* 用於 semantic versioning

---

## 8️⃣ `delete`（刪除 branch / tag）

```yaml
on: delete
```

可用於：

* 清理環境
* 自動移除 Preview Deployments

---

## 9️⃣ `push_tag`（常用替代寫法）

沒有正式事件叫 `push_tag`，但可用 patterns 達成：

```yaml
on:
  push:
    tags:
      - "v*"
```

用於：

* 以 tag 觸發「正式版部署」

---

# 🟪 多事件組合範例

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:
```

用途：

* push → 跑 CI
* PR → 做全部測試
* workflow_dispatch → 人工部署

---

# 🟩 完整事件支援列表

GitHub 官方事件清單（非常龐大）：
[https://docs.github.com/actions/using-workflows/events-that-trigger-workflows](https://docs.github.com/actions/using-workflows/events-that-trigger-workflows)

常見分類：

* Code events（push、PR、create、delete）
* Project events（issues、labels、milestones）
* Security events（Vulnerability alerts）
* Package events（registry packages）
* Scheduling events（schedule）
* Manual events（workflow_dispatch）

---

# ⭐ Workflow 事件選擇建議

| 使用情境           | 推薦事件                      |
| -------------- | ------------------------- |
| CI（Lint/Test）  | `push`, `pull_request`    |
| 手動部署           | `workflow_dispatch`       |
| 自動部署（正式版）      | `push` + tag pattern      |
| Bot 自動管理 Issue | `issues`, `issue_comment` |
| 定期任務           | `schedule`                |

---

# 🧠 重點總結

1. `on:` 決定 workflow 什麼時候執行
2. 事件可細分為：

   * 只對某些 branches
   * 只對 tags
   * 只對特定 types（opened/closed/edit...）
3. 事件可以同時並列多種
4. `workflow_dispatch` 是手動觸發的好夥伴
5. `schedule` 讓 GitHub Actions 具備 cron 自動化能力

掌握事件，就掌握了 GitHub Actions 的入口！
