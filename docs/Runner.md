# GitHub Actions Runners

Runners are the **machines that execute your workflows**. Every job in a workflow runs on a runner.
Understanding runners is critical for performance, cost, and security.

This document covers:

* What a runner is
* GitHub-hosted vs self-hosted runners
* Available runner images
* Pre-installed software
* Labels and selection
* Security & best practices

---

## 1. What is a Runner?

A runner is:

* A virtual machine or physical machine
* That listens for GitHub Actions jobs
* Runs the job's steps (Shell, Docker, custom actions)
* Reports logs and status back to GitHub

In YAML you choose a runner with:

```yaml
runs-on: ubuntu-latest
```

---

## 2. GitHub-Hosted Runners

GitHub-hosted runners are **managed by GitHub**:

* You do not manage OS updates
* You do not manage scaling
* You pay per minute (for private repos) or use free tier

Common labels:

```yaml
runs-on: ubuntu-latest    # most common
runs-on: windows-latest
runs-on: macos-latest
```

Each job gets a **fresh, clean VM**:

* New filesystem
* Pre-installed tools (Node, Python, Docker, etc.)
* Previous jobs do not share state (unless using cache / artifacts)

Useful docs:

* Pre-installed software on Ubuntu: GitHub-hosted runners → Software

> Tip: For most workflows, `ubuntu-latest` is the best default.

---

## 3. Self-Hosted Runners

Self-hosted runners are machines **you manage yourself**:

* Can be on-prem, cloud VM, or even your laptop
* Useful when you need special hardware (GPU, big memory, custom network)
* Useful for long-running or heavy workloads

### 3.1 How they work (conceptually)

* You install the GitHub runner agent on a machine
* Register the runner to a repository / org
* The runner listens for jobs with matching labels
* Jobs execute on that machine instead of GitHub-hosted VMs

### 3.2 Basic configuration example

In workflow:

```yaml
runs-on: [self-hosted, linux, x64]
```

You can:

* Use `self-hosted` + custom labels (e.g. `gpu`, `high-mem`)
* Assign multiple labels to one runner

### 3.3 When to use self-hosted runners

* Need access to internal network / databases
* Need GPUs or special hardware
* Need custom OS images or pre-installed tools
* Want full control over environment and caching

> Important: **You are responsible** for patching, security, Docker versions, etc.

---

## 4. Selecting Runners with Labels

You can specify **a single label** or **an array of labels**:

```yaml
runs-on: ubuntu-latest           # GitHub-hosted

runs-on: self-hosted            # any self-hosted runner

runs-on: [self-hosted, linux]   # must match all labels

runs-on: [self-hosted, gpu]     # e.g. GPU runner
```

GitHub matches a runner that contains *all* requested labels.

---

## 5. Pre-installed Software (GitHub-hosted)

GitHub-hosted runners come with:

* Multiple versions of Node.js, Python, Java, .NET
* Docker / Docker Compose
* Common build tools (gcc, make, cmake)
* CLI tools (git, gh, aws, kubectl, etc.)

You can always check what is available by visiting the official documentation for `ubuntu-latest` runners.

> Tip: You usually do **not** need to install git, Node, Python yourself.

---

## 6. Performance & Cost Considerations

### GitHub-hosted

* Simple to use
* Scales automatically
* Billed per-minute (private repos)
* Limited concurrency based on your plan

### Self-hosted

* You control machine type and cost
* You can keep tools and caches between runs
* But you must handle auto-scaling, cleanup, and security

---

## 7. Security Best Practices

### For GitHub-hosted runners

* Prefer least privilege permissions (`permissions:` in workflow)
* Avoid checking secrets into the repo (use `secrets.*`)
* Use OIDC instead of long-lived cloud keys when possible

### For self-hosted runners

* Treat runners as **highly privileged** machines
* Do not run untrusted workflows on the same runner
* Use separate runners for public and private repos
* Restrict network access (only what is required)
* Regularly patch OS and Docker

---

## 8. Example: Mixing Different Runners in One Workflow

```yaml
name: Multi-runner Example

on: [push]

jobs:
  linux-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm test

  windows-tests:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm test

  gpu-training:
    runs-on: [self-hosted, linux, gpu]
    needs: linux-tests
    steps:
      - uses: actions/checkout@v3
      - run: python train.py
```

---

## 9. Key Takeaways

* **Runners = machines** that execute your jobs
* **GitHub-hosted** runners are easiest to start with
* **Self-hosted** runners are powerful but require management
* Use **labels** to target specific runner capabilities
* Consider **security** and **cost** when designing your runner strategy
