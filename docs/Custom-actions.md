# GitHub Actions — Three Ways to Create Custom Actions

GitHub Actions allows you to create **three types of custom actions**:

1. **JavaScript (Node.js) Actions**
2. **Docker Actions**
3. **Composite Actions**

Each type has a different purpose, performance profile, and ideal use case.
This document explains all three, with structure, code templates, pros/cons, and when to choose which.

---

# 🟦 1. JavaScript Actions (Node.js)

JavaScript actions run **directly inside GitHub’s runner**, using Node.js.
They are fast, lightweight, and ideal for automation logic.

📌 **Best for:**

* CLI-like tools
* API calls (GitHub API, AWS, Slack...)
* Processing inputs and producing outputs
* High performance tasks that do NOT require Docker

## 📁 File structure

```
my-js-action/
 ├── action.yml
 ├── index.js
 ├── package.json
 └── node_modules/   (bundled)
```

## 📝 action.yml

```yaml
name: "My JS Action"
description: "A simple JavaScript action"
runs:
  using: "node20"
  main: "index.js"
inputs:
  name:
    required: true
outputs:
  upper:
    description: "Uppercase version"
```

## 🧠 index.js

```js
const core = require('@actions/core');

try {
  const name = core.getInput('name');
  const upper = name.toUpperCase();
  core.setOutput('upper', upper);
} catch (err) {
  core.setFailed(err.message);
}
```

## ✔ Pros

* Fastest execution
* Direct access to GitHub Actions Toolkit
* No container overhead
* Best developer experience

## ❌ Cons

* Cannot include system-level dependencies
* Must bundle node_modules

---

# 🟧 2. Docker Actions

Docker actions run your logic inside a **container**, giving full control over environment, dependencies, and binaries.

📌 **Best for:**

* Python, Go, Ruby, Java, etc.
* Tools requiring system libraries
* Running CLIs / custom binaries
* Reproducible environments

## 📁 File structure

```
my-docker-action/
 ├── action.yml
 ├── Dockerfile
 ├── script.py
 └── requirements.txt
```

## 📝 action.yml

```yaml
name: "My Docker Action"
description: "Runs inside a container"
runs:
  using: "docker"
  image: "Dockerfile"
inputs:
  filename:
    required: true
```

## 🐳 Dockerfile

```dockerfile
FROM python:3.12
COPY requirements.txt /requirements.txt
RUN pip install -r /requirements.txt
COPY script.py /script.py
CMD ["python", "/script.py"]
```

## 🧠 `script.py`

```python
import os
print("Processing:", os.getenv('INPUT_FILENAME'))
```

## ✔ Pros

* Full control over runtime environment
* Easy to use any language
* Container reproducibility

## ❌ Cons

* Slower startup
* Larger repo size
* Cannot run on Windows/Mac runners (Linux only)

---

# 🟩 3. Composite Actions

Composite actions allow you to combine **multiple Bash/PowerShell steps** into a reusable action.

📌 **Best for:**

* Reusing workflow logic
* Simple shell scripts
* Multi-step workflows
* Wrapping common patterns (install deps, run tools)

## 📁 File structure

```
my-composite-action/
 └── action.yml
```

## 📝 action.yml

```yaml
name: "My Composite Action"
description: "Reusable workflow steps"
runs:
  using: "composite"
steps:
  - run: echo "Hello ${{ inputs.name }}"
    shell: bash
inputs:
  name:
    required: true
```

## ✔ Pros

* Very simple
* No Docker or Node needed
* Perfect for workflow reuse

## ❌ Cons

* Cannot run Node.js toolkits
* Limited to shells (bash, pwsh)
* No external dependency bundling

---

# 📊 Comparison Table

| Feature             | JavaScript Action | Docker Action     | Composite Action |
| ------------------- | ----------------- | ----------------- | ---------------- |
| Performance         | ⭐⭐⭐⭐ (fast)       | ⭐⭐ (slow startup) | ⭐⭐⭐              |
| Environment Control | Medium            | ⭐⭐⭐⭐ full         | Low              |
| Uses Node.js        | Yes               | Optional          | No               |
| Run on all runners  | Yes               | Linux only        | Yes              |
| Best for            | API, logic        | CLIs, binaries    | Step reuse       |

---

# 🎯 When to Choose Which?

### Use **JavaScript Action** if:

* You want speed
* You need GitHub Toolkit (`core`, `github`, `exec`)
* You process inputs/outputs

### Use **Docker Action** if:

* You need Python, Go, Java…
* You need system dependencies
* Environment must be fully isolated

### Use **Composite Action** if:

* You just want to reuse workflows
* Everything can be done with shell steps
* You don't need heavy logic

---

# 📚 Official Docs

* JavaScript Actions: [https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action)
* Docker Actions: [https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action](https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action)
* Composite Actions: [https://docs.github.com/en/actions/creating-actions/creating-a-composite-action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)
* Storing Actions In Repositories & Sharing Actions With Others: [https://docs.github.com/en/actions/how-tos/create-and-publish-actions/publish-in-github-marketplace#publishing-an-action](https://docs.github.com/en/actions/how-tos/create-and-publish-actions/publish-in-github-marketplace#publishing-an-action)
