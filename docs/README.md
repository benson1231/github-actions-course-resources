# GitHub Actions Overview

> Download slide from [academind](https://academind.com/) github resource [academind/github-actions-course-resources](https://github.com/academind/github-actions-course-resources/blob/main/Slides/github-actions.pdf)

This document provides a clean, production-ready **base template for GitHub Actions workflows**, suitable for CI, testing, building, linting, or deployment pipelines.

---

# 📌 Workflow Structure

```yaml
name: My Workflow

# 🟦 Workflow Triggers (Events)
on:
  push:
    branches: [ main ]
  pull_request:
  workflow_dispatch:   # Manual trigger

# 🟩 Jobs
jobs:
  example-job:
    runs-on: ubuntu-latest   # Select runner environment

    # 🟧 Steps executed within this job
    steps:
      # 1. Checkout repository (almost always required)
      - name: Checkout code
        uses: actions/checkout@v3

      # 2. Setup environment (using marketplace actions)
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20

      # 3. Run shell commands
      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      # 4. Additional tasks: cache, artifacts, outputs, etc.
```

---

# 🧩 Breakdown of Workflow Components

## 1️⃣ `name` — Workflow Name

Displayed in the GitHub Actions UI.

```yaml
name: CI Pipeline
```

---

## 2️⃣ `on` — Triggering Events

Defines when the workflow runs.

```yaml
on:
  push:
    branches: [ main ]
  pull_request:
  schedule:
    - cron: "0 2 * * *"   # Run daily at 2 AM UTC
  workflow_dispatch:
```

Most common events:

* `push`
* `pull_request`
* `workflow_dispatch` (manual trigger)
* `schedule` (cron jobs)
* `release`
* `workflow_run`

---

## 3️⃣ `jobs` — The Core of a Workflow

A workflow may contain **one or many jobs**.

* Jobs can run **in parallel**.
* Or define dependencies using `needs:`.

Example:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

  test:
    runs-on: ubuntu-latest
    needs: build
```

---

## 4️⃣ `runs-on` — Selecting a Runner

Specifies the environment for the job.

```yaml
runs-on: ubuntu-latest
```

Available:

* `ubuntu-latest`
* `windows-latest`
* `macos-latest`
* Self-hosted runners

---

## 5️⃣ `steps` — Instructions Executed by a Job

Key step types:

### ✔ Checkout repository

```yaml
- uses: actions/checkout@v3
```

### ✔ Use marketplace actions

```yaml
- uses: actions/setup-node@v3
  with:
    node-version: 20
```

### ✔ Run shell commands

```yaml
- run: echo "Hello GitHub Actions!"
```

### ✔ Set environment variables

```yaml
- run: echo "VERSION=1.0.0" >> $GITHUB_ENV
```

---

# 🎯 Recommended Base CI Template

```yaml
name: Basic CI

on:
  push:
    branches: [ main ]
  pull_request:
  workflow_dispatch:

jobs:
  ci:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build project
        run: npm run build
```

---

# 📚 Official Documentation

* Workflow syntax: [https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
* Events: [https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
* GitHub Actions Marketplace: [https://github.com/marketplace?type=actions](https://github.com/marketplace?type=actions)

---

If you'd like, I can also add:

* A minimal workflow version
* A production deployment template
* A template with caching / matrix / concurrency baked in
