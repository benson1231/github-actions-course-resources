# Debugging GitHub Actions

Debugging GitHub Actions workflows is an essential skill for understanding failures, diagnosing problems, and improving CI/CD stability. This guide provides practical techniques, tools, and common patterns for troubleshooting workflows effectively.

---

# 1. Enable Debug Logging

GitHub Actions supports two powerful debugging modes:

## ✔ Step Debug Logs

Enable for one run:

```
ACTIONS_STEP_DEBUG=true
```

Use in repository **Secrets → Actions**:

* Name: `ACTIONS_STEP_DEBUG`
* Value: `true`

This reveals:

* Expanded expressions
* Hidden step execution details

---

## ✔ Runner Debug Logs (Very Verbose)

```
ACTIONS_RUNNER_DEBUG=true
```

Produces low-level logs about:

* Workflow bootstrap
* Action resolution
* Job runner internals

Use only when necessary—output can be huge.

---

# 2. Print All Context Values (Super Useful)

Dump the entire GitHub context:

```yaml
run: echo '${{ toJSON(github) }}'
```

Other useful contexts:

```yaml
run: echo '${{ toJSON(env) }}'
run: echo '${{ toJSON(job) }}'
run: echo '${{ toJSON(steps) }}'
run: echo '${{ toJSON(matrix) }}'
```

This is essential for debugging:

* branch name inconsistencies
* PR metadata issues
* comparing push vs pull_request events

---

# 3. Debug Environment Variables

To see everything available inside a step:

```yaml
run: env | sort
```

To debug PATH:

```yaml
run: echo $PATH | tr ':' '\n'
```

Add custom environment variables:

```yaml
env:
  MY_VAR: hello
run: echo "MY_VAR = $MY_VAR"
```

---

# 4. Debug File System Issues

List workspace contents:

```yaml
run: ls -R .
```

Check working directory:

```yaml
run: pwd
```

Check permissions:

```yaml
run: ls -l
```

---

# 5. Debug Cache Behavior

Cache hits/misses are common sources of confusion.

```yaml
run: echo "Cache hit? ${{ steps.cache.outputs.cache-hit }}"
```

Check hash values:

```yaml
run: echo "Hash = ${{ hashFiles('**/package-lock.json') }}"
```

Common problems:

* file path incorrect
* project files not checked out before cache
* using matrix keys incorrectly

---

# 6. Debug Secrets & Permissions

Secrets **never echo directly**, so debug indirectly:

```yaml
run: echo "Secret length: ${#MY_SECRET}"  # safe
```

Check permissions:

```yaml
run: echo "Perms: ${{ toJSON(github.job.permissions) }}"
```

Typical error:

```
Error: Resource not accessible by integration
```

Meaning your workflow lacks permissions.

---

# 7. Debug Action Resolution

Sometimes actions fail to download or resolve:

```yaml
run: cat $GITHUB_ACTION_PATH/action.yml
```

Or when self-referencing:

```yaml
run: echo "Action path: ${{ github.action_path }}"
```

---

# 8. Add Temporary Debug Steps

Instead of editing workflow logic heavily, insert debug checkpoints:

### 🛠 Print variables

```yaml
- name: Debug variables
  run: |
    echo "ref = ${{ github.ref }}"
    echo "sha = ${{ github.sha }}"
```

### 🛠 Debug matrix

```yaml
run: echo "Node = ${{ matrix.node }}"
```

### 🛠 Debug inputs for reusable workflow

```yaml
run: echo "inputs: ${{ toJSON(inputs) }}"
```

---

# 9. Re-run Failed Jobs

GitHub UI 支援重新跑：

* Re-run failed jobs
* Re-run all jobs
* Re-run with debugging enabled

Useful for reproducing intermittent failures.

---

# 10. Debugging Common Problems

## ❌ Workflow does not trigger

Possible causes:

* event type wrong
* paths / paths-ignore rules prevent triggering
* using pull_request from fork without permissions

Use:

```yaml
run: echo '${{ toJSON(github) }}'
```

---

## ❌ Cache always misses

Check:

* Cache `key` 不一致
* `package-lock.json` 不存在（npm ci 會報錯）
* Cache 使用在 checkout 之前

---

## ❌ Deploy step fails

Check:

* AWS / cloud credentials
* permissions:

  ```yaml
  permissions:
    id-token: write
    contents: read
  ```
* dist folder 路徑不正確

---

## ❌ Docker action cannot run

Check:

* ENTRYPOINT 路徑
* Python / Node 套件版本
* file permissions (chmod +x)

---

# 11. 官方 Debugging 文件

* Debug logs → [https://docs.github.com/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging](https://docs.github.com/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging)
* Rerunning workflows → [https://docs.github.com/actions/managing-workflow-runs/re-running-workflows-and-jobs](https://docs.github.com/actions/managing-workflow-runs/re-running-workflows-and-jobs)
* Troubleshooting actions → [https://docs.github.com/actions/monitoring-and-troubleshooting-workflows](https://docs.github.com/actions/monitoring-and-troubleshooting-workflows)

