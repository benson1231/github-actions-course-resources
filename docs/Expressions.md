# GitHub Actions Expressions & Functions

GitHub Actions provides a powerful expression syntax that enables dynamic logic inside workflows. Expressions are used for conditional logic, string manipulation, accessing context values, generating dynamic outputs, and more.

This guide summarizes the core concepts, the most commonly used functions, and practical examples.

---

# 1. Expression Basics

Expressions are always written inside **`${{ }}`**.

```yaml
echo "Branch: ${{ github.ref_name }}"
```

You can use expressions in:

* `if:` conditions
* Step inputs
* Environment variables
* Action parameters
* Dynamic paths, versions, or computed values

---

# 2. Most Common Contexts

Contexts provide structured, read-only runtime data. Here are the most frequently used ones.

## 🔹 `github` Context

Contains event-related metadata.

```yaml
echo "Actor: ${{ github.actor }}"
echo "Repo:  ${{ github.repository }}"
echo "Event: ${{ github.event_name }}"
```

## 🔹 `env` Context

Reads environment variables defined in the workflow.

```yaml
echo "Env value: ${{ env.MY_VAR }}"
```

## 🔹 `steps` Context

Used to read outputs from previous steps.

```yaml
echo "Previous output: ${{ steps.build.outputs.version }}"
```

---

# 3. Built-in Functions (Most Useful)

## ✔ `contains()`

```yaml
if: contains(github.ref, 'feature/')
```

## ✔ `startsWith()` and `endsWith()`

```yaml
if: startsWith(github.ref_name, 'hotfix-')
```

## ✔ `format()` — Similar to `printf`

```yaml
echo: ${{ format('build-{0}-{1}', github.ref_name, github.run_number) }}
```

## ✔ `join()`

```yaml
echo: ${{ join(matrix.node, ', ') }}
```

## ✔ `toJSON()` — The most powerful debugging tool

```yaml
run: echo '${{ toJSON(github) }}'
```

This prints the entire context as JSON.

---

# 4. Boolean Logic & Conditions

## Basic logic operators

```yaml
if: github.ref == 'refs/heads/main'
if: github.actor != 'dependabot'
if: success() && contains(github.ref, 'release')
```

## Workflow status helpers

```yaml
if: failure()
if: cancelled()
if: always()
```

---

# 5. Passing Values Using Expressions

## Define environment variables using expressions

```yaml
env:
  TAG: ${{ github.sha }}
```

## Pass values between steps via outputs

```yaml
- name: Create version
  id: build
  run: echo "version=1.0.${GITHUB_RUN_NUMBER}" >> $GITHUB_OUTPUT

- name: Use version
  run: echo "Build version: ${{ steps.build.outputs.version }}"
```

---

# 6. Practical Usage Examples

## 🚀 Deploy only on tags

```yaml
if: startsWith(github.ref, 'refs/tags/')
```

## 🚫 Skip workflow for documentation-only commits

```yaml
if: github.event_name != 'push' || !startsWith(github.ref_name, 'docs')
```

## 🧪 Combine expressions with matrix strategy

```yaml
name: Test Node ${{ matrix.node }}
strategy:
  matrix:
    node: [16, 18, 20]
```

---

# 7. Troubleshooting Guide

| Issue                                       | Cause                         | Fix                               |
| ------------------------------------------- | ----------------------------- | --------------------------------- |
| Expression prints literal text, not a value | Missing `${{ }}`              | Add the expression wrapper        |
| `if:` condition never matches               | Mixing `ref` vs `ref_name`    | `refs/heads/main` vs `main`       |
| Matrix variables unavailable                | Placed outside job/step scope | Move matrix logic into jobs       |
| JSON output looks messy                     | Expected behavior             | `toJSON()` prints raw object data |

---

# 8. Official Documentation

* Expressions → [https://docs.github.com/actions/learn-github-actions/expressions](https://docs.github.com/actions/learn-github-actions/expressions)
* Contexts → [https://docs.github.com/actions/learn-github-actions/contexts](https://docs.github.com/actions/learn-github-actions/contexts)
* Functions → [https://docs.github.com/actions/learn-github-actions/expressions#functions](https://docs.github.com/actions/learn-github-actions/expressions#functions)
