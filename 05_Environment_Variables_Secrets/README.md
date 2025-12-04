# 🚀 Node.js + MongoDB + GitHub Actions CI/CD 樣板專案

此專案示範如何使用 **Node.js + Express + MongoDB Atlas** 搭配 **GitHub Actions** 建立完整的自動化流程

---

## 🔧 環境變數設定（Environment Variables）

本專案遵循 **12-Factor App** 原則，將所有敏感資訊透過環境變數注入。

### 🖥️ 本地端開發（Local Development）

請建立 `.env` 檔案：

```
MONGODB_CLUSTER_ADDRESS=cluster0.xxxxxx.mongodb.net
MONGODB_USERNAME=your_username
MONGODB_PASSWORD=your_password
MONGODB_DB_NAME=gha-demo
PORT=3000
```

👉 **務必確認 `.gitignore` 內已加入 `.env`**，避免被提交到 GitHub。

`.env.example` 提供參考格式：

```
MONGODB_CLUSTER_ADDRESS=
MONGODB_USERNAME=
MONGODB_PASSWORD=
MONGODB_DB_NAME=
PORT=
```

> `PORT` 可自由設定，預設為 3000

---

### ☁️ GitHub Actions（CI）環境變數設定

若要在 GitHub Actions 執行自動化測試，請到：

**GitHub Repo → Settings → Secrets and Variables → Actions → New repository secret**

加入以下項目：

| Secret Name               | 說明                        |
| ------------------------- | ------------------------- |
| `MONGODB_CLUSTER_ADDRESS` | MongoDB Atlas Cluster URL |
| `MONGODB_USERNAME`        | MongoDB 使用者名稱             |
| `MONGODB_PASSWORD`        | MongoDB 使用者密碼             |
| `MONGODB_DB_NAME`         | 資料庫名稱                     |

Workflow 使用方式：

```yaml
env:
  MONGODB_USERNAME: ${{ secrets.MONGODB_USERNAME }}
  MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}
```

GitHub 會自動遮罩（mask）所有 Secret，不會顯示在 log 中。

📸 UI 位置示意圖：

![GitHub Secrets](../../docs/secret.png)

---

## 🤖 GitHub Actions CI Workflow 說明

工作流程檔案位於 `.github/workflows/0502-deploy.yml`。

此 workflow：

1. **當 push 到 main 或 dev 分支時觸發**
2. 使用 secrets 建立程式運行所需環境變數
3. 安裝依賴、啟動 Express Server
4. Playwright 執行端到端測試（E2E）
5. 測試通過後執行 deploy 階段（可擴充）

---

## ▶️ 啟動專案

### 本地端

```bash
npm install
npm start
```

伺服器將在：

```
http://127.0.0.1:8080
```
