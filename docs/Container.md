# GitHub Actions：容器與服務容器完整教學

本文件說明 **GitHub Actions 的兩種容器執行方式：容器(Container)** 與 **服務容器(Service Containers)**，包含用途、差異、使用時機、範例 workflow，以及官方文件連結。

---

## 🐳 1. 什麼是「容器（Container）」？

容器在 GitHub Actions 中代表：

> **整個 Job 會在某個 Docker container 內執行**，包括 steps、工具、環境設定。

主要特色：

* Job 的 runner 變成你指定的 Docker image
* 適用於需要 **一致環境**、**特定版本工具** 的情境
* 所有 steps 都在同一個容器內執行

### ✔ 使用時機

* 需要 Python / Node / Java 的特定版本
* 需要 Bioinformatics 工具（FASTQC, samtools, OptiType…）
* 須保證環境 100% 可重現（infra reproducible）

### ✔ 範例：使用容器執行整個 Job

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: python:3.12

    steps:
      - uses: actions/checkout@v3
      - name: Install deps
        run: pip install -r requirements.txt
      - name: Run tests
        run: pytest -v
```

📌 在這個例子中：
所有 steps 都是在 `python:3.12` image 裡執行。

---

## 🧪 2. 什麼是「服務容器（Service Containers）」？

服務容器是：

> **Job 執行時附加的外部服務**（例如資料庫、快取、API 模擬器）。

你的 steps 仍然在 runner 或 container 中執行，但可以連到可用的 service containers。

### ✔ 使用時機

* 測試資料庫：MongoDB / PostgreSQL / MySQL
* 使用 Redis 做快取測試
* 需要依賴外部 API（mock server）
* 全端應用 E2E 測試

### ✔ 範例：Node.js + MongoDB 服務容器

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      mongo:
        image: mongo:6
        ports:
          - 27017:27017
        env:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: example

    steps:
      - uses: actions/checkout@v3
      - name: Install deps
        run: npm ci
      - name: Run tests
        env:
          DB_URL: mongodb://root:example@localhost:27017
        run: npm test
```

📌 在這例子中：

* Job 在 runner（Ubuntu）中執行
* MongoDB 以「服務容器」方式啟動，可供測試連線

---

## 🔍 3. 容器與服務容器的差異比較

| 功能              | 容器(Container)           | 服務容器(Service Containers) |
| --------------- | ----------------------- | ------------------------ |
| 作用位置            | 整個 Job 的執行環境            | Job 附加的外部服務              |
| 影響範圍            | 所有 steps                | 只提供額外服務（DB, cache…）      |
| 適用情境            | Bioinfo pipeline、語言版本固定 | DB/Redis 等整合測試           |
| 誰在 container 裡？ | **你的 workflow steps**   | **外部服務**（你不會在裡面跑 steps）  |

📌 **你可以同時使用**：Job 在 container 裡執行 + 有服務容器提供 DB。

---

## 🧩 4. 兩者合併使用的例子

例如你想在 Python 容器中執行程式，但需要 PostgreSQL 服務：

```yaml
jobs:
  integration-test:
    runs-on: ubuntu-latest
    container:
      image: python:3.11

    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: example
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v3
      - run: pip install -r requirements.txt
      - run: pytest --db=postgresql://postgres:example@localhost:5432
```

📌 這裡：

* 你的 **程式在 python:3.11 容器**裡跑
* PostgreSQL 是 **外部 service container**

---

## 📚 官方文件

* Service Containers 官方說明：
  [https://docs.github.com/en/actions/using-containerized-services/about-service-containers](https://docs.github.com/en/actions/using-containerized-services/about-service-containers)

* 了解 workflow syntax：
  [https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
