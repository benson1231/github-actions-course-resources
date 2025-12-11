# GitHub Actions: Containers and Service Containers

This document explains the two container execution models in GitHub Actions:

* **Job-level Containers** (the entire job runs inside a Docker image)
* **Service Containers** (additional supporting services such as databases)

You'll learn what they are, when to use them, how they differ, and how to configure them with real workflow examples.

---

# 🐳 1. What Is a Job-level Container?

In GitHub Actions, a *container* means:

> **The entire job runs inside a specific Docker container**, including every step.

### Key characteristics

* The job's execution environment becomes the Docker image you specify.
* Guarantees **fully reproducible environments**.
* Useful when the runner must have **specific tools or versions**.

### ✔ When to use job containers

* Requiring locked versions of Python / Node / Java, etc.
* Running scientific or bioinformatics tools (FASTQC, samtools, OptiType…)
* Ensuring deterministic, identical runtime environments across all runs.

### ✔ Example: Run a job inside a Python container

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: python:3.12

    steps:
      - uses: actions/checkout@v3
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Run tests
        run: pytest -v
```

📌 All steps execute *inside* the `python:3.12` container.

---

# 🧪 2. What Is a Service Container?

A **service container** is:

> **An external service that runs alongside your job**, like a database, cache, or API mock server.

Your workflow steps still run on the runner or container, but they can connect to the services.

### ✔ When to use service containers

* Testing against **MongoDB, PostgreSQL, MySQL, Redis**
* Running integration tests requiring external services
* API mock servers for backend/frontend projects

### ✔ Example: Node.js app with MongoDB service

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      mongo:
        image: mongo:6
        ports:
          - 27017:27017
        env:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: example

    steps:
      - uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        env:
          DB_URL: mongodb://root:example@localhost:27017
        run: npm test
```

📌 Your job runs on Ubuntu, while MongoDB runs in a sidecar container.

---

# 🔍 3. Containers vs Service Containers

| Feature                    | Job Container                   | Service Container                           |
| -------------------------- | ------------------------------- | ------------------------------------------- |
| Purpose                    | Executes all workflow steps     | Provides external services (DB, cache, API) |
| Scope                      | Entire job                      | Only supporting services                    |
| Best for                   | Bioinformatics, pinned runtimes | Databases, Redis, mock APIs                 |
| Workflow steps run inside? | **Yes**                         | **No** (services only)                      |

📌 You can use **both at the same time**.

---

# 🧩 4. Using Both Together

Example: Run Python code inside a container while using PostgreSQL as a service.

```yaml
jobs:
  integration-test:
    runs-on: ubuntu-latest

    container:
      image: python:3.11

    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: example
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v3
      - run: pip install -r requirements.txt
      - run: pytest --db=postgresql://postgres:example@localhost:5432
```

📌 Here:

* Your application runs in the `python:3.11` container.
* PostgreSQL runs as a service container.

---

# 📚 Official Documentation

* Service Containers → [https://docs.github.com/en/actions/using-containerized-services/about-service-containers](https://docs.github.com/en/actions/using-containerized-services/about-service-containers)
* Workflow Syntax → [https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
