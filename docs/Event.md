# GitHub Actions Events

GitHub Actions workflows are triggered by **events**, which represent actions or changes happening inside your repository. Understanding these events is essential for building efficient, secure, and practical CI/CD pipelines.

This guide explains:

* What an event is
* The most commonly used events
* When to use each event
* Syntax examples
* A summary table
* Official documentation links

---

## 🚀 What Are GitHub Actions Events?

An **event** is anything that can **trigger a workflow**. For example:

* Code pushed to a branch
* A pull request is created or updated
* A new release is published
* A scheduled CRON job runs
* An issue or PR receives a new comment

Whenever an event occurs, GitHub runs the workflows associated with it.

📘 Official Docs: [https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

---

# 🔥 Common GitHub Actions Events

## 1. `push`

Triggered whenever code is pushed to a branch (or specific branches).

```yaml
ame: On Push
on:
  push:
    branches:
      - main
      - dev
```

### Typical Use Cases

* Continuous Integration (tests)
* Linting
* Auto-deployment

---

## 2. `pull_request`

Triggered when a PR is opened, updated, synchronized, or reopened.

```yaml
ame: On PR
on:
  pull_request:
    branches:
      - main
```

### Typical Use Cases

* PR testing
* Linting and validation
* Automated review bots

---

## 3. `workflow_dispatch` (Manual Trigger)

Allows you to run workflows manually from the GitHub UI.

```yaml
ame: Manual Run
on:
  workflow_dispatch:
    inputs:
      env:
        description: "Select environment"
        required: true
        default: "prod"
```

### Typical Use Cases

* Manual deployment
* Running batch/ETL tasks

---

## 4. `schedule` (CRON Jobs)

Runs workflows on a CRON schedule.

```yaml
ame: Daily Job
on:
  schedule:
    - cron: "0 18 * * *"  # Runs daily at 02:00 Taiwan time
```

### Typical Use Cases

* Daily reporting
* Periodic backups
* Health checks

---

## 5. `release`

Triggered when a new release is published.

```yaml
ame: On Release
on:
  release:
    types: [created]
```

### Typical Use Cases

* Building and uploading binaries
* Generating release notes or changelogs

---

## 6. `workflow_run`

Triggered when **another workflow** finishes.

```yaml
ame: Deploy After Test
on:
  workflow_run:
    workflows: ["Test Workflow"]
    types:
      - completed
```

### Typical Use Cases

* Multi-stage CI/CD pipelines
  (e.g., test → build → deploy)

---

## 7. Issue / PR Events (`issues`, `issue_comment`, `pull_request_review`, etc.)

```yaml
ame: On Issue Comment
on:
  issue_comment:
    types: [created]
```

### Typical Use Cases

* GitHub bot automations
* Comment-based commands (`/retest`, `/deploy`)
* Automatic labeling

---

# 📌 Event Summary Table

| Event               | Trigger Timing                      | Typical Use Cases                        |
| ------------------- | ----------------------------------- | ---------------------------------------- |
| `push`              | Code pushed to a branch             | CI, linting, auto-deployment             |
| `pull_request`      | PR opened/updated                   | PR validation, tests                     |
| `workflow_dispatch` | Manual "Run workflow"               | Manual deploy, maintenance tasks         |
| `schedule`          | CRON schedule                       | Nightly builds, backups, reports         |
| `release`           | Release created                     | Publishing artifacts, release automation |
| `workflow_run`      | Another workflow completes          | Chained workflows / staging pipelines    |
| `issue_comment`     | A comment appears on an issue or PR | Bots, labeling, automation tools         |

---

# 🧪 Example: Support `push` + `pull_request` + Manual Trigger

```yaml
ame: CI Pipeline
on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm test
```

---

# 🎯 Summary

Events are the heart of GitHub Actions automation. Use:

* `push` for CI and deployment
* `pull_request` for PR validation
* `workflow_dispatch` for manual control
* `schedule` for timed jobs
* `release` for publishing automation
* `workflow_run` for multi-stage pipelines

Mastering events enables you to build powerful, flexible, and production-ready workflows.

---

📘 Official Documentation

* Events Overview: [https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
* Workflow Syntax: [https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
