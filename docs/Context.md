# GitHub Actions Contexts

GitHub Actions provides multiple **contexts**—structured metadata objects you can access inside your workflows. These include `github`, `env`, `job`, `runner`, and others. They allow your workflow to react dynamically to events, branches, commit metadata, inputs, matrix values, and more.

This document explains:

* What a *context* is
* Why `toJSON()` is important
* Common contexts and their uses
* How to print entire contexts for debugging
* Practical workflow examples
* Links to official documentation

---

## ✨ What Is a Context?

A **context** is a **read‑only metadata object** automatically provided by GitHub Actions at runtime.

Example:

```yaml
echo "Repo: ${{ github.repository }}"
```

Output:

```
owner/repo
```

Contexts are **not environment variables**—they are structured objects containing event, repository, commit, and workflow information.

---

## 🔍 Why Use `toJSON()`?

Most contexts are complex objects (maps / dictionaries). You **cannot print them directly**:

```yaml
echo "${{ github }}"   # ❌ This fails
```

But with `toJSON()`:

```yaml
echo '${{ toJSON(github) }}'
```

You get a fully expanded JSON dump:

```json
{
  "repository": "owner/repo",
  "event_name": "push",
  "sha": "abc123...",
  ...
}
```

This is **the single most powerful debugging method** in GitHub Actions.

---

## 🧪 Outputting Contexts Inside a Workflow

### Print the `github` context

```yaml
- name: Output GitHub context
  run: echo '${{ toJSON(github) }}'
```

### Print multiple contexts

```yaml
- run: echo '${{ toJSON(env) }}'
- run: echo '${{ toJSON(job) }}'
- run: echo '${{ toJSON(steps) }}'
- run: echo '${{ toJSON(runner) }}'
- run: echo '${{ toJSON(matrix) }}'   # If using matrix
```

### Save context output to a file

```yaml
- run: echo '${{ toJSON(github) }}' > github.json
```

Upload it as an artifact:

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: gh-context
    path: github.json
```

---

## 📋 Common Contexts Explained

| Context    | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| **github** | Event metadata: repo, SHA, actor, workflow, event payload    |
| **env**    | All environment variables (including those you set manually) |
| **runner** | OS, architecture, temp directories, workspace paths          |
| **job**    | Job-level metadata, status, IDs                              |
| **steps**  | Access outputs from previous steps                           |
| **matrix** | Values defined by matrix strategies                          |
| **inputs** | Parameters passed via `workflow_dispatch`                    |

Example usage:

```yaml
echo "Commit: ${{ github.sha }}"
```

---

## 🎯 Practical Techniques

### 1️⃣ Full Debug Dump — highly recommended

```yaml
- name: Debug all contexts
  run: |
    echo "GITHUB: ${{ toJSON(github) }}"
    echo "RUNNER: ${{ toJSON(runner) }}"
    echo "ENV: ${{ toJSON(env) }}"
```

### 2️⃣ Generate a metadata.json for deployment

Useful for apps showing their build info.

```yaml
- name: Generate metadata
  run: |
    echo '${{ toJSON(github) }}' > build/metadata.json
```

### 3️⃣ Let frontend apps read runtime metadata

You can show:

* Deployed commit SHA
* Build timestamp
* Workflow run number
* Triggering user

---

## 📚 Official Documentation

* **Contexts**: [https://docs.github.com/en/actions/learn-github-actions/contexts](https://docs.github.com/en/actions/learn-github-actions/contexts)
* **Expression syntax**: [https://docs.github.com/en/actions/learn-github-actions/expressions](https://docs.github.com/en/actions/learn-github-actions/expressions)
* **Using JSON in workflows**: [https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#example-using-json](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#example-using-json)
