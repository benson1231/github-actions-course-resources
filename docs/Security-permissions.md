# 🛡️ GitHub Actions Permissions Example — Documentation

This document explains how **permissions** work in GitHub Actions and why certain workflows—such as an Issue Labeler—require explicit permission declarations like:

```yaml
permissions:
  issues: write
```

Understanding permissions is critical for both **security** and **functional correctness** when interacting with GitHub APIs inside workflows.

---

## 📌 Why Do We Need to Set `permissions`?

Workflows run using an auto-generated token called `GITHUB_TOKEN`. For security reasons, this token is granted **minimal (read-only) permissions by default**.

If your workflow needs to:

* Modify Issues
* Add or remove labels
* Close Issues or Pull Requests
* Post comments
* Push changes to the repository
* Trigger other workflows

Then you must **explicitly enable the required permission scopes** in your workflow YAML.

---

## 🔐 What Does `permissions: issues: write` Do?

In the workflow below, GitHub Actions automatically assigns a label to an Issue whenever it is opened:

```yaml
permissions:
  issues: write
```

This line tells GitHub:

> **“This workflow must have permission to modify Issues.”**

Without this declaration, GitHub will block the request and return:

```
403 Resource not accessible by integration
```

---

## 🧪 Example Workflow: Automatic Issue Labeler

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

## 🧩 Workflow Breakdown

### 1️⃣ Trigger

Triggered when a new Issue is opened.

### 2️⃣ Conditional Check

Only label the Issue if its title contains the text `"bug"`:

```yaml
if: contains(github.event.issue.title, 'bug')
```

### 3️⃣ Apply Label Using GitHub REST API

The workflow calls:

```
POST /repos/{owner}/{repo}/issues/{issue_number}/labels
```

This API requires **issues:write** permission.

### 4️⃣ Label Successfully Added

If authorized, the Issue receives the label:

```
bug
```

---

## ⚠️ What Happens If You Omit Permissions?

GitHub rejects the API request and you will see:

```
403 Resource not accessible by integration
```

Meaning: the workflow **does not have permission** to modify Issues.

---

## 🔒 Best Practice: Principle of Least Privilege

GitHub recommends enabling **only the minimum permissions** required.

Good (safe):

```yaml
permissions:
  issues: write
```

Bad (dangerous):

```yaml
permissions: write-all
```

The latter grants the workflow far more access than necessary, increasing security risk.

---

## 📦 Summary

* `permissions:` controls what `GITHUB_TOKEN` can access.

* Workflows modifying Issues must explicitly declare:

  ```yaml
  issues: write
  ```

* Without proper permissions, all write operations will be blocked.

* Following least-privilege principles improves workflow security.

This example demonstrates the correct and secure way to work with GitHub API permissions within Actions workflows.
