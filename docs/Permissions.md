# 🛡️ GitHub Actions Permissions Example 說明文件

本文件說明 GitHub Actions 中的 **permissions 權限配置** 與 **Issue Label 自動化範例**，並解釋為什麼在工作流程中需要使用：

```yaml
permissions:
  issues: write
```

此用法與 GitHub API 權限控制密切相關，是 GitHub Actions 安全性的重要部分。

---

## 📌 為什麼需要設定 permissions？

GitHub Actions 使用的 `GITHUB_TOKEN` 有嚴格的權限限制。為了安全，GitHub 會預設提供**最低限度（read-only）**的權限給每個 workflow。

因此，如果 workflow 想要：

* 修改 Issue
* 新增或移除標籤（label）
* 關閉 Issue 或 Pull Request
* 寫入評論（comment）
* 修改專案內容

就必須在 YAML 中手動啟用相對應的權限。

---

## 🔐 `permissions: issues: write` 的作用

在以下 workflow 中，GitHub Actions 要透過 GitHub REST API **自動幫 Issue 加上 label**：

```yaml
permissions:
  issues: write
```

這一行告訴 GitHub：

> **「這個 workflow 需要有寫入 Issue 的權限」**

若不加入這段設定，GitHub 會拒絕執行 API，並回傳：

```
403 Resource not accessible by integration
```

---

## 🧪 實際範例：Issue Labeler Workflow

以下是示例 workflow：

```yaml
name: Label Issues (Permissions Example)
on:
  issues:
    types:
      - opened
jobs:
  assign-label:
    permissions:
      issues: write
    runs-on: ubuntu-latest
    steps:
      - name: Assign label
        if: contains(github.event.issue.title, 'bug')
        run: |
          curl -X POST \
          --url https://api.github.com/repos/${{ github.repository }}/issues/${{ github.event.issue.number }}/labels \
          -H 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}' \
          -H 'content-type: application/json' \
          -d '{
              "labels": ["bug"]
            }' \
          --fail
```

---

## 🧩 Workflow 流程解析

### 1. workflow 觸發

當 Issue 被建立（opened），工作流程開始執行。

### 2. 判斷 Issue 標題是否包含 "bug"

```yaml
if: contains(github.event.issue.title, 'bug')
```

### 3. 使用 GitHub API 新增 label

```bash
POST /repos/{owner}/{repo}/issues/{issue_number}/labels
```

* API 需要 **issues:write** 權限
* 授權使用 auto-generated `GITHUB_TOKEN`

### 4. 加 label 成功

Issue 會自動被加上：

```
"bug"
```

---

## 💡 如果沒有設定 permissions，會發生什麼事？

Workflow 會嘗試執行 curl，但 GitHub API 會拒絕，出現：

```
403 Resource not accessible by integration
```

也就是：

> **workflow 沒有權限修改 Issue，因此無法新增 label**。

---

## 🔒 最佳實踐：使用最小權限原則

GitHub 官方推薦：

> **只啟用 workflow 實際需要的權限，不要全部開啟。**

所以本例只啟用：

```yaml
permissions:
  issues: write
```

比以下寬鬆但危險的設定更好：

```yaml
permissions: write-all
```

---

## 📦 結論

* `permissions:` 用來控制 workflow 中 auto-generated `GITHUB_TOKEN` 的操作權限。

* 如果 workflow 需要修改 Issue，就必須設定：

  ```yaml
  issues: write
  ```

* 若未設定，GitHub 會阻擋所有寫入 Issue 的 API 請求。

此範例展現了正確、安全、可控的 GitHub Actions 權限管理方式。
