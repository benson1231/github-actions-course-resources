# GitHub Actions workflow control 全指南

GitHub Actions 雖然不是一個完整程式語言，但提供了多種 **流程控制（Control Flow）機制**，讓你可以：

* 控制哪些 steps / jobs 執行
* 建立 Job 之間的依賴（needs）
* 使用 if 條件判斷
* 使用 continue-on-error 忽略錯誤
* 在 Matrix 中控制特定組合
* 依據事件類型、分支、輸入參數決定流程

本章整理所有你會用到的 Control Flow 技巧。

---

# 📌 1. needs：建立 Job 依賴

`needs` 決定一個 job 必須在另一個 job 成功後才會執行。

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

  test:
    runs-on: ubuntu-latest
    needs: build
```

### 多個 needs：

```yaml
needs: [build, lint]
```

### 使用 needs 取得其他 job 的 outputs：

```yaml
echo "Result: ${{ needs.build.outputs.version }}"
```

---

# 📌 2. if：條件判斷控制流程

`if:` 是 workflow 裡最重要的控制語句。

### Step 層級：

```yaml
- name: Run only on main
  if: github.ref == 'refs/heads/main'
  run: echo "main branch"
```

### Job 層級：

```yaml
if: github.event_name == 'pull_request'
```

### 常用條件：

| 條件        | 例子                                         |
| --------- | ------------------------------------------ |
| 分支        | `github.ref == 'refs/heads/main'`          |
| 事件        | `github.event_name == 'push'`              |
| PR        | `github.event.pull_request.merged == true` |
| tag       | `startsWith(github.ref, 'refs/tags/')`     |
| matrix 條件 | `matrix.node == 20`                        |
| job 成功/失敗 | `if: failure()`                            |

---

# 📌 3. continue-on-error：忽略錯誤但不終止流程

```yaml
- name: Test unstable feature
  run: npm run experimental-test
  continue-on-error: true
```

### 配合 job strategy：允許部分步驟失敗

```yaml
strategy:
  fail-fast: false
```

---

# 📌 4. fail-fast：Matrix 的流程控制

Matrix 預設 **某一組失敗就會取消其他組**。

```yaml
strategy:
  fail-fast: false
  matrix:
    node: [16, 18, 20]
```

---

# 📌 5. 工作流程早停：cancel-in-progress

例如避免 Push 時重複跑：

```yaml
concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: true
```

---

# 📌 6. 只有某些條件才上傳 Artifact

```yaml
- uses: actions/upload-artifact@v4
  if: failure()
  with:
    name: test-report
    path: report.json
```

---

# 📌 7. 使用 outputs 建立邏輯流程

### Step 輸出：

```yaml
- id: check
  run: echo "ok=true" >> $GITHUB_OUTPUT

- name: Next step
  if: steps.check.outputs.ok == 'true'
  run: echo "Continue"
```

### Job 輸出 → 下一個 Job 使用

```yaml
jobs:
  first:
    outputs:
      tag: ${{ steps.get_tag.outputs.tag }}

  second:
    needs: first
    run: echo "Tag is ${{ needs.first.outputs.tag }}"
```

---

# 📌 8. 依事件控制流程（event-based control flow）

### 只在 PR 開啟時：

```yaml
if: github.event.action == 'opened'
```

### 只在 push 但排除 bot：

```yaml
if: github.actor != 'dependabot[bot]'
```

---

# 📌 9. 只針對特定檔案變動執行

```yaml
on:
  push:
    paths:
      - 'src/**'
      - '!docs/**'
```

---

# 📌 10. 完整 Control Flow 示範（含 needs + if + continue-on-error）

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "build"

  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - run: npm test
        continue-on-error: true

  deploy:
    runs-on: ubuntu-latest
    needs: test
    if: needs.test.result == 'success'
    steps:
      - run: echo "Deploying..."
```

---

# 📚 官方文件

* Conditions: [https://docs.github.com/en/actions/using-jobs/using-conditions-to-control-job-execution](https://docs.github.com/en/actions/using-jobs/using-conditions-to-control-job-execution)
* Expressions: [https://docs.github.com/en/actions/learn-github-actions/expressions](https://docs.github.com/en/actions/learn-github-actions/expressions)
* Using outputs: [https://docs.github.com/en/actions/using-jobs/defining-outputs-for-jobs](https://docs.github.com/en/actions/using-jobs/defining-outputs-for-jobs)

