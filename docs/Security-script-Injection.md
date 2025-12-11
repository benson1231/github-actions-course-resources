# 🛡️ Script Injection — Security Guide

Script Injection is a critical security concern when building automation workflows, CI/CD pipelines, GitHub Actions, or any web-facing system. This document uses a **GitHub Actions Issue Labeler workflow** as an example to explain what Script Injection is, why it is dangerous, and how to avoid it with correct secure practices.

---

## 📌 What Is Script Injection?

**Script Injection** occurs when:

> **An attacker injects malicious text into a program or shell command, and that text is executed as code.**

If your workflow or application executes user-supplied input—such as Issue titles, PR bodies, HTTP requests, or environment variables—an attacker can potentially run arbitrary commands.

GitHub Actions is especially exposed because **anyone can open an Issue or submit a PR** unless a repository is fully private.

---

## 🧨 How Script Injection Can Occur in GitHub Actions

Consider a workflow that reads the Issue title:

```yaml
issue_title="${{ github.event.issue.title }}"
```

Because *anyone* can create an Issue, an attacker could submit a title like:

```
bug"; rm -rf .; echo "
```

If the workflow later executes this string in the shell, the attacker’s injected command (`rm -rf .`) may run on the GitHub runner.

---

## ✔ Secure Example (Safe Pattern)

This example is safe because **the user input is always wrapped in quotes**, preventing the shell from interpreting it as executable code:

```yaml
issue_title="${{ github.event.issue.title }}"
if [[ "$issue_title" == *"bug"* ]]; then
  echo "Issue is about a bug!"
else
  echo "Issue is not about a bug"
fi
```

### 🔒 Why is this safe?

* The string is **always quoted**, so special characters are treated as literal text.
* No `eval` is used.
* The value is only compared—not executed.
* Patterns such as `;`, `&&`, `|`, command substitution `$(...)`, etc., do **not** get executed.

---

## ❌ Unsafe Patterns (Vulnerable to Injection)

### 1️⃣ Executing user input directly

```yaml
run: ${{ github.event.issue.title }}
```

If the Issue title is:

```
rm -rf /
```

The runner will attempt to execute that command.

---

### 2️⃣ Unquoted shell expansion

```yaml
run: echo ${{ github.event.issue.title }}
```

If the input contains:

```
$(ls)
```

Then `$(ls)` **will run as a command**.

---

### 3️⃣ Using `eval` (always dangerous)

```bash
eval $issue_title
```

Any injected command becomes executable in full.

---

## 🧠 Core Risks of Script Injection

* **User input is never trustworthy.**
* **Unquoted shell variables can become executable.**
* GitHub Actions surfaces a lot of external input (Issues, PRs, labels, comments).
* Actions often run with write permissions and access tokens, so the blast radius is serious.

### Therefore:

* Always quote user-controlled variables: `"$var"`
* Never use `eval`
* Never execute text coming from GitHub context
* Only use user input for comparisons, filtering, or logging—not execution

---

## ✔ Final Summary

**Script Injection is a real security threat in GitHub Actions.** Any time workflows read external text such as Issue titles or PR descriptions, attackers can attempt command injection.

### Safe Practices:

* Wrap all variables in quotes
* Avoid executing user input under any circumstance
* Avoid `eval`
* Treat GitHub event content as potentially malicious

By following these principles, GitHub Actions workflows remain secure—even when exposed to untrusted public input.
