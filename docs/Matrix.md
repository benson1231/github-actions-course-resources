# GitHub Actions Matrix

Matrix strategy is one of the most powerful parallelization features in GitHub Actions. It allows you to:

* Test against multiple versions of Node, Python, Java
* Build across different operating systems (Linux / Windows / macOS)
* Run multiple parameter combinations at once (regions, architectures, frameworks)
* Reduce total workflow runtime while increasing CI coverage

This guide covers:

* Matrix fundamentals
* Syntax and variables
* `include` and `exclude`
* Multi-dimensional matrices
* Matrix with cache and artifacts
* Common pitfalls
* Official documentation

---

## 📌 What Is a Matrix?

A matrix lets you define a set of parameters. GitHub Actions automatically generates parallel jobs for every combination.

Example:

```yaml
strategy:
  matrix:
    node: [16, 18, 20]
```

Creates **three jobs**:

* node = 16
* node = 18
* node = 20

---

## 🧱 Basic Example: Test Using Multiple Node Versions

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [16, 18, 20]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci
      - run: npm test
```

Access matrix values with `${{ matrix.<name> }}`.

---

## 🧊 Multi-Dimensional Matrix

Define multiple variables:

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest]
    python: [3.9, 3.10]
```

GitHub will generate:

* ubuntu + Python 3.9
* ubuntu + Python 3.10
* macOS + Python 3.9
* macOS + Python 3.10

Use OS dynamically:

```yaml
runs-on: ${{ matrix.os }}
```

---

## 🎯 Using `include` to Add Special Cases

Example: Run lint only on Node 20

```yaml
strategy:
  matrix:
    node: [16, 18]
  include:
    - node: 20
      lint: true
```

Then conditionally run a step:

```yaml
if: matrix.lint == true
```

---

## 🚫 Using `exclude` to Remove Combinations

```yaml
strategy:
  matrix:
    os: [ubuntu, windows]
    python: [3.7, 3.8]
    exclude:
      - os: windows
        python: 3.7
```

This removes the **windows + py3.7** job.

---

## 📦 Best Practice: Matrix + Cache

Cache keys **must** include matrix variables. Otherwise, caches from different environments will overwrite each other.

```yaml
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: npm-${{ matrix.node }}-${{ hashFiles('package-lock.json') }}
    restore-keys: npm-${{ matrix.node }}-
```

This ensures:

* Node 16 uses its own cache
* Node 18 uses its own cache
* No cross‑pollution

---

## 🔗 Matrix + Artifacts

Each matrix job can upload its own artifact:

```yaml
strategy:
  matrix:
    arch: [amd64, arm64]

steps:
  - run: make build-${{ matrix.arch }}
  - uses: actions/upload-artifact@v4
    with:
      name: build-${{ matrix.arch }}
      path: dist/${{ matrix.arch }}
```

Download a specific artifact:

```yaml
- uses: actions/download-artifact@v4
  with:
    name: build-amd64
```

---

## ⚠️ Common Matrix Pitfalls

### ❌ 1. Using the same cache key for all variants

```
key: npm-cache
```

→ All Node versions share one cache (dangerous).

---

### ❌ 2. Wrong indentation for `include` / `exclude`

YAML spacing errors cause matrix rules to be ignored.

---

### ❌ 3. Too many jobs → workflow queued

Free tier supports **20 concurrent jobs**.

---

### ❌ 4. Windows + node-gyp extremely slow

Avoid heavy Windows builds if not necessary.

---

## 📚 Official Documentation

Matrix strategy → [https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)
