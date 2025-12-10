# GitHub Actions Matrix 全指南

Matrix（矩陣策略）是 GitHub Actions 中強大的平行化功能，可以讓你：

* 用不同版本的 Node、Python、Java 跑測試
* 平行建置多個 OS（Linux / Windows / macOS）
* 一次建置多組參數（例如：region、architecture、framework）
* 減少 workflow 時間並提升 CI 覆蓋度

本文件說明：

* Matrix 基本概念
* Matrix 語法與變數
* include / exclude
* 多維矩陣
* 搭配 cache / artifact
* 常見錯誤
* 官方文件連結

---

## 📌 什麼是 Matrix？

Matrix 允許你定義一組參數，GitHub Actions 會自動產生多個平行 job。

例如：

```yaml
strategy:
  matrix:
    node: [16, 18, 20]
```

會啟動三個 job：

* node = 16
* node = 18
* node = 20

---

## 🧱 基本範例：不同 Node 版本跑測試

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [16, 18, 20]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci
      - run: npm test
```

Matrix 值可以透過 `${{ matrix.<name> }}` 取得。

---

## 🧊 多維 Matrix

你可以同時定義多組變數：

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest]
    python: [3.9, 3.10]
```

會產生：

* ubuntu / py3.9
* ubuntu / py3.10
* macOS / py3.9
* macOS / py3.10

### 搭配運行器：

```yaml
runs-on: ${{ matrix.os }}
```

---

## 🎯 include：新增特別案例

例如只在 Node 20 額外跑一個 Lint：

```yaml
strategy:
  matrix:
    node: [16, 18]
  include:
    - node: 20
      lint: true
```

用法：

```yaml
if: matrix.lint == true
```

---

## 🚫 exclude：排除某些組合

```yaml
strategy:
  matrix:
    os: [ubuntu, windows]
    python: [3.7, 3.8]
    exclude:
      - os: windows
        python: 3.7
```

會跳過 windows + py3.7。

---

## 📦 Matrix 搭配 Cache（最佳實務）

Cache key 必須依 Matrix 變數分開，否則會互相污染：

```yaml
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: npm-${{ matrix.node }}-${{ hashFiles('package-lock.json') }}
    restore-keys: npm-${{ matrix.node }}-
```

這樣：

* Node16 用 Node16 的 cache
* Node18 用 Node18 的 cache
  不會互相覆寫。

---

## 🔗 Matrix 與 Artifacts

你可以讓每個 matrix job 上傳自己的 artifact。

```yaml
name: Build
strategy:
  matrix:
    arch: [amd64, arm64]
steps:
  - run: make build-${{ matrix.arch }}
  - uses: actions/upload-artifact@v4
    with:
      name: build-${{ matrix.arch }}
      path: dist/${{ matrix.arch }}
```

在 deploy job：

```yaml
- uses: actions/download-artifact@v4
  with:
    name: build-amd64
```

---

## ⚠️ Matrix 常見錯誤

### ❌ 1. Key 未依變數區分 → Cache 污染

```
key: npm-cache
```

→ 所有 Node 版本共用 cache（很危險）。

### ❌ 2. include / exclude 格式錯誤

YAML 的縮排錯一格就會無法作用。

### ❌ 3. job 太多導致 workflow 排隊

免費 tier 只有 **20 concurrent jobs**。

### ❌ 4. Windows + node-gyp 耗時極長

建議 Windows job 減少。

---

## 📚 官方文件

* Strategy & Matrix：[https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)
