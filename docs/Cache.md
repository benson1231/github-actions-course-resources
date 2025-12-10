# GitHub Actions Cache 全指南

Cache（快取）是 GitHub Actions 中用來 **加速 workflow 執行** 的核心功能，常用於：

* npm / pip / conda 套件快取
* Docker layers 快取
* Build 產物快取（如 node_modules、.next、.venv）
* 大型專案的依賴快取（Rust、Go、Java、R…）

本指南會清楚解釋：

* Cache 是什麼
* Cache vs Artifact 差異
* 如何設定 Cache（完整範例）
* Key / restore-keys 機制
* 常見錯誤與排查
* 官方文件連結

---

## 📌 什麼是 Cache？

Cache 是 GitHub Actions 用來儲存 **能重複利用的檔案** 的機制，例如：

* node_modules
* pip 的虛擬環境
* Docker 層

目的是：

> 減少 workflow 執行時間，避免每次重新下載依賴。

官方文件：
[https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)

---

## 📦 使用快取 — actions/cache@v3

### 最基本模式

```yaml
- name: Cache npm deps
  uses: actions/cache@v3
  with:
    path: node_modules
    key: deps-${{ hashFiles('package-lock.json') }}
```

### 為什麼要用 hashFiles？

hashFiles() 會根據檔案內容產生唯一值：

* package-lock.json 改 → key 變 → cache 不會命中（因為依賴改了）
* package-lock.json 沒改 → key 一樣 → 使用舊 cache

---

## 🔄 restore-keys — 部分匹配快取

```yaml
key: npm-${{ hashFiles('package-lock.json') }}
restore-keys: |
  npm-
```

restore-keys 允許 GitHub 嘗試找到：

* 以 `npm-` 開頭的任意 cache

當你希望「找不到完全匹配的 key，也能使用舊版 cache」時很好用。

---

## 🧪 npm 快取完整範例

```yaml
steps:
  - uses: actions/checkout@v3

  - name: Cache node modules
    id: cache
    uses: actions/cache@v3
    with:
      path: ~/.npm
      key: npm-cache-${{ hashFiles('package-lock.json') }}
      restore-keys: |
        npm-cache-

  - name: Install deps
    run: npm ci
```

> ⚠️ 注意：npm 官方建議快取 **~/.npm** 而不是 node_modules。

---

## 🐍 pip / Python cache

```yaml
- uses: actions/cache@v3
  with:
    path: ~/.cache/pip
    key: pip-${{ hashFiles('requirements.txt') }}
```

---

## 🐳 Docker layer cache（最常見 CI/CD 使用案例）

```yaml
- name: Cache Docker layers
  uses: actions/cache@v3
  with:
    path: /tmp/.buildx-cache
    key: buildx-${{ github.sha }}
    restore-keys: |
      buildx-
```

---

## 🧬 Cache vs Artifact 差異

| 功能   | Artifact            | Cache                          |
| ---- | ------------------- | ------------------------------ |
| 用途   | 保存 workflow 結果、測試報告 | 加速 workflow（依賴快取）              |
| 適合   | dist/，報告，產物         | node_modules、pip、Docker layers |
| 生命周期 | 90 天                | 7 天（預設）                        |
| 是否覆寫 | 不會覆寫，版本多個           | key 衝突會取代舊 cache               |
| 大小   | 通常較大                | 限制在 10GB 內                     |

它們不能互相取代。

---

## ⚠️ Cache 常見錯誤

### ❌ 1. "Cache not found" 但你確定有跑過

原因多半是：

* key 不一致（hashFiles 改變）
* restore-keys 沒設
* runner OS 不同（Linux / Windows / macOS） → cache 不共用

### ❌ 2. npm ci 失敗：找不到 package-lock.json

通常是 working-directory 設錯。

### ❌ 3. 超過 Cache 限制（10 GB）

解法：刪除不必要的大型 cache。

### ❌ 4. 多個不同專案共用同一 Key

例：

```
key: node
```

→ 絕對會造成 cache 污染。

---


## 📚 官方文件

* Cache 說明：[https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
* actions/cache@v3：[https://github.com/actions/cache](https://github.com/actions/cache)

