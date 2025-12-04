# 🚀 GitHub Actions 基礎教學

本文件整理 **GitHub Actions 的基礎觀念、常用保留字（keywords）解釋、jobs/steps 的執行順序**，並逐段說明以下 workflow 範例：

```yaml
name: 0205-deployment2
on: push
jobs:
  lint:
  test:
  deploy:
```

這份 workflow 示範最基本的 CI/CD：Lint → Test → Build → Deploy。

---

# 🧩 GitHub Actions Workflow 基本結構

一份 workflow 大致由以下部分組成：

```yaml
name: Workflow 名稱
on: 觸發事件
jobs:
  job1:
    runs-on: 執行環境
    steps: 流程步驟
  job2:
    needs: job1  # 依賴 job1
    steps:
```

---

# 📌 1. `name:` Workflow 的名稱

```yaml
name: 0205-deployment2
```

* 只是讓你在 GitHub Actions UI 裡更容易辨識。
* 不影響程式執行。

---

# 📌 2. `on:` 什麼事件會觸發 workflow？

```yaml
on: push
```

常見事件：

| 事件                  | 說明                          |
| ------------------- | --------------------------- |
| `push`              | 每次 push 觸發 workflow         |
| `pull_request`      | PR 建立/更新時觸發                 |
| `issues`            | Issue 事件觸發（opened、edited 等） |
| `workflow_dispatch` | 手動觸發                        |

更多可用事件：[https://docs.github.com/actions/using-workflows/events-that-trigger-workflows](https://docs.github.com/actions/using-workflows/events-that-trigger-workflows)

---

# 📌 3. `jobs:` Workflow 的主要執行單位

```yaml
jobs:
  lint:
  test:
  deploy:
```

* 每個 job 都在**獨立的 Runner（虛擬機）** 中執行。
* job 之間不共享資源（如 node_modules 必須重新安裝）。

---

# 📌 4. `runs-on:` 指定 Runner 作業系統

```yaml
runs-on: ubuntu-latest
```

Runner 可選擇：

* ubuntu-latest（最常用）
* windows-latest
* macos-latest

Ubuntu 執行速度快、成本低、相容性最好，是預設選擇。

---

# 📌 5. `defaults:` 設定 steps 的預設行為

```yaml
defaults:
  run:
    shell: bash
    working-directory: "./02_Basics/05_Practice_Project(Finished)"
```

* 所有後續 `run:` 都會使用 bash
* `working-directory` 指定所有指令的執行路徑

避免每一行都寫：

```yaml
run: cd ./folder && npm ci
```

讓 workflow 更乾淨。

---

# 📌 6. `steps:` job 的實際步驟

每個 step 是 job 內的獨立指令。

### 🟦 常見 step 類型

#### **① 使用 action（uses）**

```yaml
uses: actions/checkout@v3
```

代表使用別人（或官方）寫好的 GitHub Action 套件。

常見官方 actions：

* `actions/checkout` → 抓取 repo 原始碼
* `actions/setup-node` → 安裝 Node.js
* `actions/cache` → 啟用快取
* `actions/upload-artifact` → 上傳檔案

#### **② 執行命令（run）**

```yaml
run: npm ci
```

在 runner 的 shell 裡執行任意命令。

---

# 🧪 Workflow 範例逐段解說

以下是你提供的 CI/CD 範例，包含 Lint → Test → Deploy。

---

# 🟦 Job 1：Lint

```yaml
lint:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - run: npm ci
    - run: npm run lint
```

作用：

* 抓取程式碼
* 安裝依賴
* 執行 Lint 檢查（通常會跑 ESLint）

---

# 🟦 Job 2：Test

```yaml
test:
  needs: lint
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - run: npm ci
    - run: npm run test
```

### ⭐ 重點：`needs: lint`

代表：

> Test job **只有在 lint job 成功後才會執行**。

這建立了 **直線式 CI pipeline**：

```
lint → test → deploy
```

---

# 🟦 Job 3：Deploy

```yaml
deploy:
  needs: test
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - run: npm ci
    - run: npm run build
    - run: echo "Deploying..."
```

當 test 成功後：

* 再 checkout 原始碼
* 再安裝依賴
* 執行 build（通常產生 dist/）
* 最後部署（此例只是模擬 echo）

實際部署通常會是：

* 部署到 Vercel / Netlify
* 部署到 AWS S3 / CloudFront
* 部署到 Render / Railway
* 透過 SSH 部署到自家伺服器

---

# 🧩 GitHub Actions 常用保留字（Keywords）對照表

| Keyword       | 說明                 |
| ------------- | ------------------ |
| `name`        | Workflow 名稱        |
| `on`          | 什麼事件觸發 workflow    |
| `jobs`        | 多個 job 的集合         |
| `runs-on`     | Job 使用的 runner 系統  |
| `needs`       | Job 之間的依賴關係        |
| `steps`       | Job 內的執行步驟         |
| `uses`        | 使用現成 GitHub Action |
| `run`         | 執行 shell 指令        |
| `env`         | 設定環境變數             |
| `defaults`    | Step 的預設設定         |
| `with`        | 傳入 action 的參數      |
| `permissions` | Token 權限控制         |
| `if:`         | 條件判斷（決定 step 是否執行） |

---

# 🧠 最後整理：GitHub Actions 基礎概念

1. Workflow = 由 `on:` 觸發的 CI/CD 流程
2. Job = 跑在 runner 上的獨立執行單位
3. Step = Job 裡的每一個動作
4. Jobs 互相獨立 → 必須每次重新 checkout、安裝依賴
5. `needs:` 讓你能建立 pipeline（相依關係）
6. `uses:` 使用現成 actions
7. `run:` 執行自己的指令

掌握這些就是 GitHub Actions 的核心基礎。
