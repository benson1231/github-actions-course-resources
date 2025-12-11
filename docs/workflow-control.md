# GitHub Actions Workflow Control

GitHub Actions is not a full programming language, but it provides powerful **workflow control mechanisms** that allow you to:

* Control which steps or jobs execute
* Define dependencies between jobs (`needs`)
* Use conditional execution (`if`)
* Continue execution even when errors occur (`continue-on-error`)
* Control matrix behavior (fail-fast, filtering, dynamic logic)
* Adjust workflow behavior based on events, branches, file changes, or runtime outputs

This guide covers every essential control-flow feature you will use when designing advanced CI/CD pipelines.

---

# 📌 1. `needs`: Define Job Dependencies

`needs` ensures that a job only runs after another job completes successfully.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

  test:
    runs-on: ubuntu-latest
    needs: build
```

### Multiple dependencies

```yaml
needs: [build, lint]
```

### Accessing outputs from dependent jobs

```yaml
echo "Version: ${{ needs.build.outputs.version }}"
```

---

# 📌 2. `if`: Conditional Execution

`if:` determines whether a job or step should run.

### Step-level condition

```yaml
- name: Run only on main
  if: github.ref == 'refs/heads/main'
  run: echo "Running on main branch"
```

### Job-level condition

```yaml
if: github.event_name == 'pull_request'
```

### Common conditions

| Purpose    | Example                                    |
| ---------- | ------------------------------------------ |
| Branch     | `github.ref == 'refs/heads/main'`          |
| Event      | `github.event_name == 'push'`              |
| PR merged  | `github.event.pull_request.merged == true` |
| Tag        | `startsWith(github.ref, 'refs/tags/')`     |
| Matrix     | `matrix.node == 20`                        |
| Job result | `if: failure()`                            |

---

# 📌 3. `continue-on-error`: Allow Failures Without Stopping Workflow

```yaml
- name: Test experimental feature
  run: npm run test-experimental
  continue-on-error: true
```

### With strategy: prevent early termination

```yaml
strategy:
  fail-fast: false
```

---

# 📌 4. Matrix Control: `fail-fast`

By default, matrix jobs stop when one job fails.

Disable fail-fast:

```yaml
strategy:
  fail-fast: false
  matrix:
    node: [16, 18, 20]
```

---

# 📌 5. Workflow Cancellation: `concurrency`

Avoid running multiple redundant workflows (e.g., on rapid commits).

```yaml
concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: true
```

---

# 📌 6. Conditional Artifact Upload

```yaml
- uses: actions/upload-artifact@v4
  if: failure()
  with:
    name: test-report
    path: report.json
```

---

# 📌 7. Use Outputs to Control Flow

### Step outputs

```yaml
- id: check
  run: echo "ok=true" >> $GITHUB_OUTPUT

- name: Continue only if ok
  if: steps.check.outputs.ok == 'true'
  run: echo "Proceeding..."
```

### Job outputs → next job

```yaml
jobs:
  build:
    outputs:
      tag: ${{ steps.get_tag.outputs.tag }}

  deploy:
    needs: build
    run: echo "Tag = ${{ needs.build.outputs.tag }}"
```

---

# 📌 8. Event-Based Workflow Control

### Execute only when PR is opened

```yaml
if: github.event.action == 'opened'
```

### Skip bot commits

```yaml
if: github.actor != 'dependabot[bot]'
```

---

# 📌 9. Run Only When Specific Files Change

```yaml
on:
  push:
    paths:
      - 'src/**'
      - '!docs/**'
```

---

# 📌 10. Complete Example: `needs` + `if` + `continue-on-error`

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

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

# 📚 Official Documentation

* Conditions: [https://docs.github.com/en/actions/using-jobs/using-conditions-to-control-job-execution](https://docs.github.com/en/actions/using-jobs/using-conditions-to-control-job-execution)
* Expressions: [https://docs.github.com/en/actions/learn-github-actions/expressions](https://docs.github.com/en/actions/learn-github-actions/expressions)
* Outputs: [https://docs.github.com/en/actions/u](https://docs.github.com/en/actions/u)
