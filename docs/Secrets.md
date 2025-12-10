# GitHub Actions Secrets 使用指南

GitHub Actions Secrets 用於安全儲存敏感資訊，例如：

* API keys
* AWS IAM Access Keys
* Database passwords
* Tokens（如 GitHub Token, Slack Token）

本指南將介紹：

1. 什麼是 GitHub Actions Secrets
2. 如何建立 Repository Secrets
3. 如何在 Workflow 中使用 Secrets
4. 官方文件連結

---

## 🔐 1. 什麼是 GitHub Actions Secrets？

GitHub Actions Secrets 是一種 **加密儲存機制**，提供以下功能：

* 儲存敏感資訊且不暴露在程式碼中
* 在 workflow runtime 解密
* 在 log 中自動遮蔽（masking）

Secrets 只能被 Actions 使用，無法在 Repo UI 中查看內容。

官方說明：
👉 [https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)

---

## 🏗️ 2. 如何建立 Repository Secrets

### 步驟：

1. 打開 GitHub repository
2. 點選 **Settings**
3. 左側選單 → **Secrets and variables** → **Actions**
4. 按下 **New repository secret**
5. 填入：

   * **Name**（全部大寫，例如：`AWS_ACCESS_KEY_ID`）
   * **Value**（密鑰內容）
6. 儲存即可

常見 Secrets 名稱示例：

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
SLACK_WEBHOOK_URL
DATABASE_PASSWORD
JWT_SECRET
```

官方文件：
👉 [https://docs.github.com/en/actions/security-guides/encrypted-secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

---

## ⚙️ 3. 如何在 GitHub Actions Workflow 中使用 Secrets

你可以在 workflow 中透過 `${{ secrets.SECRET_NAME }}` 存取 secrets。

### 🎯 範例：部署 AWS S3

```yaml
name: deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Deploy to S3
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws s3 sync dist/ s3://my-bucket --delete
```

### 🎯 在 Composite Action 或 Docker Action 使用 secrets

Secrets 會作為環境變數傳入：

```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

在程式中讀取：

```python
import os
key = os.getenv("API_KEY")
```

---

## ⚠️ 4. Secrets 使用注意事項

* ❌ **不要把 secrets 寫進程式碼或 commit**
* ❌ 不要傳給第三方（非 GitHub 官方）未信任的 actions
* ✔️ 儘量使用最小權限（Principle of Least Privilege）
* ✔️ 建議在 production 及 staging 使用不同 secrets
* ✔️ 建議定期 rotate（更新）密鑰

官方最佳實踐：
👉 [https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)

---

## 📚 官方文件總覽

| 主題                    | 連結                                                                                                                                                                                   |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 使用 Actions Secrets    | [https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)             |
| Encrypted secrets 概念  | [https://docs.github.com/en/actions/security-guides/encrypted-secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)                                         |
| GitHub Actions 安全最佳實踐 | [https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions) |


