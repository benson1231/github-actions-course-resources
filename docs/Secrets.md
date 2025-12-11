# GitHub Actions Secrets Guide

GitHub Actions **Secrets** provide a secure mechanism for storing sensitive information such as:

* API keys
* AWS IAM access keys
* Database passwords
* OAuth / JWT tokens
* Slack / Discord webhooks

This guide explains:

1. What GitHub Actions Secrets are
2. How to create repository secrets
3. How to use secrets in workflows
4. Best practices
5. Official documentation references

---

## 🔐 1. What Are GitHub Actions Secrets?

GitHub Actions Secrets are **encrypted, write-only values** that you can safely use in workflows without exposing them in your repository.

Key properties:

* Secrets are **encrypted at rest and during runtime**.
* Their values **cannot be viewed** after creation—only overwritten.
* Secrets are **masked in logs**, preventing accidental exposure.
* Only workflows with permission can access them.

Official docs → [https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)

---

## 🏗️ 2. How to Create Repository Secrets

1. Go to your GitHub repository.
2. Navigate to **Settings**.
3. Go to **Secrets and variables → Actions**.
4. Click **New repository secret**.
5. Enter:

   * **Name** (uppercase, e.g., `AWS_ACCESS_KEY_ID`)
   * **Value** (the key or password)
6. Save. ![Secrets](../img/secret.png)

### Common secret names

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
SLACK_WEBHOOK_URL
DATABASE_PASSWORD
JWT_SECRET
API_TOKEN
```

Official docs → [https://docs.github.com/en/actions/security-guides/encrypted-secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

---

## ⚙️ 3. Using Secrets in GitHub Actions Workflows

Secrets are accessed via `${{ secrets.SECRET_NAME }}`.

### Example: Deploying to AWS S3

```yaml
ame: Deploy

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

### Using secrets in Composite or Docker Actions

```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

Python example:

```python
import os
key = os.getenv("API_KEY")
```

---

## ⚠️ 4. Important Safety Practices

### ❌ Never commit secrets to the repository

Hard-coded secrets can be leaked, scraped, or abused.

### ❌ Avoid passing secrets to untrusted third-party actions

Unless the action is verified or audited.

### ✔️ Use the **Principle of Least Privilege**

Give each secret only the permissions needed.

### ✔️ Use separate secrets for staging and production

Avoid accidental cross-environment deploys.

### ✔️ Rotate secrets regularly

Expired or compromised secrets should be replaced immediately.

Official security hardening guide → [https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)

---

## 📚 Official Documentation Summary

| Topic                      | Link                                                                                                                                                                                 |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Using secrets in Actions   | [https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)             |
| Encrypted Secrets overview | [https://docs.github.com/en/actions/security-guides/encrypted-secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)                                         |
| Security best practices    | [https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions) |
