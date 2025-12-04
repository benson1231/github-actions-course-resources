# 🛡️ Script Injection（腳本注入）說明文件

Script Injection 是在開發自動化流程、CI/CD、GitHub Actions 或 Web 應用程式時必須特別注意的安全議題。本文件將以 **GitHub Actions Issue Labeler 工作流程**作為例子，說明 Script Injection 的概念、風險與安全寫法。

---

## 📌 什麼是 Script Injection？

Script Injection（腳本注入）指的是：

> **攻擊者將惡意字串注入到程式或 Shell 指令中，使該字串被當作程式碼執行的攻擊方式。**

只要你的程式是依賴外部輸入（例如：Issue title、PR body、HTTP 請求內容、環境變數）並在 shell 中執行，就有可能受攻擊。

---

## 🧨 GitHub Actions 中 Script Injection 的來源

在以下工作流程中，Issue 標題來自於使用者輸入：

```yaml
issue_title="${{ github.event.issue.title }}"
```

任何人都能在 GitHub 開 Issue，因此他們可以輸入：

```
bug"; rm -rf .; echo "
```

如果 workflow 將這段字串當作 shell 指令執行，就可能導致攻擊者執行惡意指令。

---

## ✔ 安全的寫法示例

以下是安全的示範，因為使用了 **雙引號包裹字串**，Shell 會將內容視為純字串，而非可執行的指令：

```yaml
issue_title="${{ github.event.issue.title }}"
if [[ "$issue_title" == *"bug"* ]]; then
  echo "Issue is about a bug!"
else
  echo "Issue is not about a bug"
fi
```

### 🔒 為什麼這樣是安全的？

* `${{ github.event.issue.title }}` 的內容被包在 `"..."` 裡
* Bash 會把內容視為字串，不會執行其中的 `; rm -rf /` 等命令
* 沒有使用 `eval`
* 沒有直接執行變數，例如 `run: $issue_title`

---

## ❌ 不安全寫法（容易遭 Script Injection）

以下寫法容易導致 injection：

### 1. 直接執行來自使用者輸入的字串（危險）

```yaml
run: ${{ github.event.issue.title }}
```

若 Issue title 是：

```
rm -rf /
```

→ Runner 會被清空。

### 2. 未加引號的 echo

```yaml
run: echo ${{ github.event.issue.title }}
```

如果輸入包含：

```
$(ls)
```

那麼 $(ls) 會被當成 shell 指令執行。

### 3. 在 shell 裡使用 eval（最高風險）

```bash
eval $issue_title
```

任何惡意輸入都會被執行。

---

## 🧠 Script Injection 的核心風險

* 使用者輸入永遠不可信
* shell 中的未轉義字串可能變成指令
* GitHub Actions 很容易接觸來自外部的 input（issues / PR / labels / comments）

因此：

* 所有 user input 都要包在引號內
* 不要用 eval
* 不要直接執行來自 GitHub context 的內容

---

## ✔ 最終結論

> **Script Injection 是 GitHub Actions 需要注意的安全漏洞，因為所有使用者輸入（Issue Title、PR 內容等）都有可能被惡意利用。如果未妥善處理字串而在 shell 中執行，就會引發腳本注入攻擊。**

安全守則：

* 一律使用 `"$variable"` 保護字串
* 不要直接執行使用者輸入
* 不要使用 `eval`
* 僅將 user input 用於比較、記錄，而非執行

