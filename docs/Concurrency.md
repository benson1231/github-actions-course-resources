# GitHub Actions Concurrency

Concurrency is an essential but often overlooked feature in GitHub Actions. It prevents unnecessary or conflicting workflow executions by allowing you to:

* **Avoid running duplicate workflows** (e.g., repeated pushes to the same PR)
* **Prevent deployment conflicts** (e.g., two deploys running at the same time)
* **Save runner time and cost**
* **Automatically cancel outdated executions** (e.g., previous builds that are no longer relevant)

GitHub Actions provides concurrency controls through the `concurrency:` configuration block.

---

# 1. Basic Usage

```yaml
concurrency:
  group: build-${{ github.ref }}
  cancel-in-progress: true
```

### Explanation

* **group** — Defines a "group identifier" for this workflow. Workflows with the same group cannot run simultaneously.
* **cancel-in-progress: true** — Automatically cancels any previous run in the same group.

### Ideal for:

* Frequent `push` activity
* CI/CD pipelines where only the latest run matters

---

# 2. Ensure Only One Workflow Runs per Pull Request

```yaml
concurrency:
  group: pr-${{ github.event.pull_request.number }}
  cancel-in-progress: true
```

### Effect

If PR #12 receives 10 pushes → **only the newest workflow run continues**.

---

# 3. Repository-Wide Concurrency (Global Lock)

```yaml
concurrency:
  group: global-build
  cancel-in-progress: false
```

### Effect

* Only one instance of this workflow can run at a time **across the entire repository**.
* With `cancel-in-progress: false`, new runs will **wait in queue** instead of canceling the current one.

### Ideal for:

* Terraform apply
* Production deployments
* Workloads requiring strict mutual exclusion

---

# 4. Environment-Level Concurrency

GitHub Environments (e.g., `staging`, `production`) include built-in concurrency locks.

```yaml
environment: production
```

### Advantages

* Environment protections (required reviewers, wait timers)
* Natural concurrency: only **one deployment per environment** at a time

---

# 5. Dynamic Concurrency Groups (Highly Practical)

You can construct unique groups using workflow metadata:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref_name }}-${{ matrix.node }}
  cancel-in-progress: true
```

This prevents duplicate jobs for:

* Same workflow
* Same branch
* Same matrix variant (e.g., Node version)

---

# 6. Using Concurrency with Deployments

GitHub Deployments already lock per environment, but you can combine both for full control:

```yaml
environment: production
concurrency:
  group: deploy-production
  cancel-in-progress: true
```

### Benefits

* Ensures **only one deployment** runs at a time
* Cancels outdated deploy attempts

---

# 7. Common Pitfalls

### ❌ Group defined too broadly

```yaml
group: build
```

This locks *every branch* into one group—often unintended.

---

### ❌ Illegal characters in group

Group names should avoid:

* Spaces
* `/` (slashes)
* Special characters

Stick to:

* `-` (dash)
* Alphanumeric characters

---

### ❌ Concurrency conflicts with matrix builds

If using:

```yaml
strategy:
  matrix: { node: [16, 18, 20] }
```

and:

```yaml
group: build-${{ github.ref }}
```

All matrix jobs share the same group → **only one job runs**.

#### Fix:

```yaml
group: build-${{ github.ref }}-${{ matrix.node }}
```

---

# 8. Official Documentation

* Concurrency → [https://docs.github.com/actions/using-jobs/using-concurrency](https://docs.github.com/actions/using-jobs/using-concurrency)
* Deployment environments → [https://docs.github.com/actions/deployment/targeting-different-environments](https://docs.github.com/actions/deployment/targeting-different-environments)
