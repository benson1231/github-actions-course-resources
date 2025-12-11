# Reusable Workflows

Reusable workflows allow you to define CI/CD logic **once** and reuse it across repositories or inside the same repository. They help you follow DRY principles, reduce duplication, improve maintainability, and enforce consistent standards across teams.

---

# 1. What Is a Reusable Workflow?

A reusable workflow is simply a normal GitHub Actions workflow that exposes **inputs**, **secrets**, and **outputs** so other workflows can call it.

It is triggered by:

```yaml
on: workflow_call
```

Example use cases:

* Standardized build/test pipeline across repos
* Centralized security checks
* Shared deploy workflows
* Shared AWS / Kubernetes / Terraform workflows

---

# 2. Basic Structure of a Reusable Workflow

Create the workflow in:

```
.github/workflows/<name>.yml
```

Example:

```yaml
name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      node-version:
        type: string
        required: true
    secrets:
      NPM_TOKEN:
        required: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm install
      - run: npm test
```

---

# 3. Calling a Reusable Workflow

You call a reusable workflow using `uses:` inside a job:

```yaml
jobs:
  call-reusable:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: 20
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

You can also call a reusable workflow **across repositories**:

```yaml
uses: org/repo/.github/workflows/build.yml@main
```

---

# 4. Passing Inputs

```yaml
with:
  node-version: 18
```

Inputs support:

* `string`
* `boolean`
* `number`
* `choice`

Example input definition:

```yaml
inputs:
  environment:
    type: choice
    options: [dev, staging, prod]
    required: true
```

---

# 5. Passing Secrets

Secrets must be explicitly passed:

```yaml
secrets:
  MY_SECRET: ${{ secrets.MY_SECRET }}
```

You may define optional secrets:

```yaml
secrets:
  API_KEY:
    required: false
```

---

# 6. Returning Outputs From a Reusable Workflow

Inside reusable workflow:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version-step.outputs.version }}
    steps:
      - id: version-step
        run: echo "version=1.2.3" >> $GITHUB_OUTPUT
```

Caller workflow:

```yaml
jobs:
  call:
    uses: ./.github/workflows/reusable.yml

  deploy:
    needs: call
    runs-on: ubuntu-latest
    steps:
      - run: echo "Version from reusable: ${{ needs.call.outputs.version }}"
```

---

# 7. Real-World Example: Shared Deployment Workflow

Reusable workflow:

```yaml
on:
  workflow_call:
    inputs:
      bucket:
        type: string
        required: true
    secrets:
      AWS_ACCESS_KEY_ID:
        required: true
      AWS_SECRET_ACCESS_KEY:
        required: true
```

Caller:

```yaml
jobs:
  deploy:
    uses: org/repo/.github/workflows/s3-deploy.yml@v1
    with:
      bucket: my-site
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

# 8. Common Mistakes & Troubleshooting

### ❌ Secrets not passed

Reusable workflows **cannot automatically inherit secrets**.

```yaml
# Must explicitly pass
secrets:
  TOKEN: ${{ secrets.TOKEN }}
```

### ❌ Missing @ref when calling other repos

Always specify a branch or tag:

```yaml
uses: org/repo/.github/workflows/build.yml@main
```

### ❌ Trying to use matrix in reusable workflow caller

Matrix must be defined in the caller.

---

# 9. Security Considerations

* Ensure reusable workflows have **least privilege** when dealing with secrets.
* Prefer calling workflows via **tags** instead of `main`.
* Public repos should avoid exposing secret-requiring workflows to forks.
* Use `permissions:` block inside reusable workflows.

Example:

```yaml
permissions:
  contents: read
  id-token: write
```

---

# 10. Official Documentation

* Reusable workflows → [https://docs.github.com/actions/using-workflows/reusing-workflows](https://docs.github.com/actions/using-workflows/reusing-workflows)
* workflow_call trigger → [https://docs.github.com/actions/using-workflows/events-that-trigger-workflows#workflow_call](https://docs.github.com/actions/using-workflows/events-that-trigger-workflows#workflow_call)
* Passing inputs & secrets → [https://docs.github.com/actions/using-workflows/reusing-workflows#providing-inputs-and-secrets](https://docs.github.com/actions/using-workflows/reusing-workflows#providing-inputs-and-secrets)

