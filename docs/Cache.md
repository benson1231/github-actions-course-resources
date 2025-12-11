# GitHub Actions Cache

Caching is one of the most effective ways to **speed up GitHub Actions workflows** by reusing previously downloaded or generated files. It is commonly used for:

* npm / pip / conda dependency caching
* Docker layer caching
* Framework build caches (node_modules, .next, .venv, target/)
* Large dependency caches for Rust, Go, Java, R, etc.

This guide explains:

* What cache is and why it matters
* Cache vs Artifact
* How to configure caching correctly
* How cache keys and restore-keys work
* Common pitfalls and debugging tips
* Official documentation links

---

## 📌 What Is Cache?

A **cache** stores reusable files so that future workflow runs don’t need to regenerate or redownload them.

Common cached items include:

* `node_modules`
* Python virtual environment packages
* Docker build layers

**Goal:**

> Reduce workflow execution time by avoiding redundant installations.

Official documentation:
[https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)

---

## 📦 Using Cache — `actions/cache@v3`

### Basic Example

```yaml
- name: Cache npm dependencies
  uses: actions/cache@v3
  with:
    path: node_modules
    key: deps-${{ hashFiles('package-lock.json') }}
```

### Why `hashFiles`?

`hashFiles()` creates a hash based on file contents.

* If `package-lock.json` changes → the key changes → old cache is ignored (correct behavior)
* If it has not changed → the cache is reused

This prevents stale dependencies.

---

## 🔄 Using `restore-keys` for Partial Cache Matching

```yaml
key: npm-${{ hashFiles('package-lock.json') }}
restore-keys: |
  npm-
```

`restore-keys` allows GitHub to fall back to **any cache that begins with `npm-`**.

Useful when you prefer using older caches rather than having none at all.

---

## 🧪 Complete npm Caching Example

```yaml
steps:
  - uses: actions/checkout@v3

  - name: Cache npm global cache
    id: cache
    uses: actions/cache@v3
    with:
      path: ~/.npm
      key: npm-${{ hashFiles('package-lock.json') }}
      restore-keys: |
        npm-

  - name: Install dependencies
    run: npm ci
```

> ⚠️ Best practice: Cache `~/.npm` rather than `node_modules` for better reliability.

---

## 🐍 Python / pip Cache Example

```yaml
- uses: actions/cache@v3
  with:
    path: ~/.cache/pip
    key: pip-${{ hashFiles('requirements.txt') }}
```

---

## 🐳 Docker Layer Cache Example

```yaml
- name: Cache Docker layers
  uses: actions/cache@v3
  with:
    path: /tmp/.buildx-cache
    key: buildx-${{ github.sha }}
    restore-keys: |
      buildx-
```

---

## 🧬 Cache vs Artifact — Key Differences

| Feature            | Artifact                               | Cache                       |
| ------------------ | -------------------------------------- | --------------------------- |
| Purpose            | Store workflow results                 | Speed up workflows          |
| Best for           | build outputs, reports, packaged files | dependencies, Docker layers |
| Retention          | 90 days                                | 7 days (default)            |
| Overwrite behavior | Does not overwrite existing versions   | New key replaces old cache  |
| Size limit         | Larger                                 | 10 GB per cache             |

They are **not interchangeable**.

---

## ⚠️ Common Cache Pitfalls

### ❌ 1. "Cache not found" even though it existed

Likely reasons:

* Key mismatch
* Missing restore-keys
* Different OS runner (Linux/Windows/macOS do not share cache)

### ❌ 2. `npm ci` fails because `package-lock.json` cannot be found

Your `working-directory` is wrong.

### ❌ 3. Cache too large (>10GB)

Delete unnecessary cache data or reduce what is being stored.

### ❌ 4. Multiple projects sharing the same key

Example:

```
key: node
```

This leads to cache pollution. Always use project-specific keys.

---

## 📚 Official Documentation

* Cache overview:
  [https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
* `actions/cache`:
  [https://github.com/actions/cache](https://github.com/actions/cache)
