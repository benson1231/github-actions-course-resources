# GitHub Actions Contexts & toJSON() 全指南

GitHub Actions 提供許多 **contexts（上下文物件）**，例如 `github`、`env`、`job`、`runner`，你可以用它們讀取 workflow 執行時的資料。

本文件說明：

* 什麼是 Context
* 為什麼要用 `toJSON()`
* 常見 Context 詳解
* 如何在 workflow 中輸出全部 Context
* 實戰技巧：debug workflow、創建 metadata file
* 官方文件

---

## ✨ 什麼是 Context？

Context 是 GitHub Actions 在 runtime 提供的 **只讀資料物件**。
例如：

```yaml
echo "Repo: ${{ github.repository }}"
```

會輸出：

```
benson1231/github-actions-course-resources
```

Context 不是環境變數，而是 GitHub 自動提供的 metadata。

---

## 🔍 為什麼使用 `toJSON()`？

因為 Context 是一個複雜物件（dictionary / map），直接印會失敗。

例如：

```yaml
echo "${{ github }}"   # ❌ 不可行
```

但使用：

```yaml
echo '${{ toJSON(github) }}'
```

會輸出整個 JSON：

```json
{
  "token": "***",
  "repository": "owner/repo",
  "event_name": "push",
  ...
}
```

這是 **debug workflow 最強方式**！

---

## 🧪 在 workflow 中完整輸出 Context

### 📌 取得 `github` context

```yaml
- name: Output GitHub context
  run: echo '${{ toJSON(github) }}'
```

### 📌 更多 contexts

```yaml
- run: echo '${{ toJSON(env) }}'
- run: echo '${{ toJSON(job) }}'
- run: echo '${{ toJSON(steps) }}'
- run: echo '${{ toJSON(runner) }}'
- run: echo '${{ toJSON(matrix) }}'   # 如果有用 matrix
```

你也可以把內容寫到檔案：

```yaml
- run: echo '${{ toJSON(github) }}' > github.json
```

並上傳：

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: gh-context
    path: github.json
```

---

## 📋 常見 Contexts 介紹

| Context  | 用途                                         |
| -------- | ------------------------------------------ |
| `github` | repo, event, sha, actor, workflow metadata |
| `env`    | 所有環境變數（包含自訂的）                              |
| `runner` | Runner 機器資訊（OS、架構、暫存路徑）                    |
| `job`    | Job 狀態、id、結果                               |
| `steps`  | 讀取其他 steps 的 outputs                       |
| `matrix` | Matrix job variables                       |
| `inputs` | workflow_dispatch 的參數                      |

例如：

```yaml
echo "Commit: ${{ github.sha }}"
```

---

## 🎯 實戰技巧

### 1️⃣ Debug workflow（最常用）

```yaml
- name: Debug all contexts
  run: |
    echo "GITHUB: ${{ toJSON(github) }}"
    echo "RUNNER: ${{ toJSON(runner) }}"
    echo "ENV: ${{ toJSON(env) }}"
```

### 2️⃣ 自動產生 metadata.json 並部署

```yaml
- name: Generate metadata
  run: |
    echo '${{ toJSON(github) }}' > build/metadata.json
```

### 3️⃣ 讓前端讀 workflow metadata

你的 React 或 static website 可以顯示：

* 目前部署的 commit
* build time
* workflow run number

---

## 📚 官方文件

* Contexts：[https://docs.github.com/en/actions/learn-github-actions/contexts](https://docs.github.com/en/actions/learn-github-actions/contexts)
* 表達式語法：[https://docs.github.com/en/actions/learn-github-actions/expressions](https://docs.github.com/en/actions/learn-github-actions/expressions)
* Workflow 中使用 JSON：[https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#example-using-json](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#example-using-json)

