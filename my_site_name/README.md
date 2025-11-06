# personal_website — Multi-Device Git Workflow (macOS & Windows)

This guide explains how to work on this repository from **multiple computers** (macOS & Windows) safely and consistently.

Repository: `https://github.com/Wximing/personal_website`
Default branch: `main`

---

## 0) One-time setup on each new computer

### A) Install Git

* **macOS**:
  `xcode-select --install` (or install Git via Homebrew)
* **Windows**:
  Install **Git for Windows** from the official site. During setup, keep defaults and ensure **Git Credential Manager** is enabled.

### B) Identify yourself (do once per machine)

```bash
git config --global user.name "Ximing Wang"
git config --global user.email "your_email@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
# line endings (recommended):
# macOS/Linux:
git config --global core.autocrlf input
# Windows:
git config --global core.autocrlf true
```

### C) Authenticate with GitHub

#### Option 1 — SSH (recommended)

* **macOS (Terminal)**:

  ```bash
  ssh-keygen -t ed25519 -C "your_email@example.com"
  eval "$(ssh-agent -s)"
  ssh-add ~/.ssh/id_ed25519
  pbcopy < ~/.ssh/id_ed25519.pub  # copy
  ```
* **Windows (PowerShell or Git Bash)**:

  ```bash
  ssh-keygen -t ed25519 -C "your_email@example.com"
  # Start the ssh-agent service and auto-start (PowerShell):
  Get-Service ssh-agent | Set-Service -StartupType Automatic
  Start-Service ssh-agent
  ssh-add $env:USERPROFILE\.ssh\id_ed25519
  # Copy the public key:
  type $env:USERPROFILE\.ssh\id_ed25519.pub | clip
  ```

Add the copied key to **GitHub → Settings → SSH and GPG keys → New SSH key**.

Then clone with SSH (works the same on macOS/Windows):

```bash
git clone git@github.com:Wximing/personal_website.git
```

#### Option 2 — HTTPS + Personal Access Token (PAT)

* Create a PAT: GitHub → Settings → Developer settings → Personal access tokens.
* **Windows**: Git Credential Manager will securely store your PAT.
* Clone:

  ```bash
  git clone https://github.com/Wximing/personal_website.git
  ```

> **Windows tip (optional):** enable long paths if your project has deep folders:
>
> * Run PowerShell **as Administrator**:
>
>   ```powershell
>   git config --system core.longpaths true
>   ```
> * Or enable long paths in Windows Group Policy / Registry.

---

## 1) Daily workflow (every session, on any computer)

Always start **inside the repo folder**:

```bash
cd personal_website
```

### A. Sync before you change anything

```bash
git status
git checkout main
git pull origin main
```

### B. Make edits locally

```bash
git status
git diff    # optional
```

### C. Stage and commit

```bash
git add .   # or add files selectively
git commit -m "Concise, specific message about the change"
```

### D. Push your work

```bash
git push origin main
```

> Rule of thumb: **Pull first, then push**. Repeat this cycle whenever you switch computers.

---

## 2) If changes exist on GitHub while you were editing

If you forgot to pull first and Git warns about non-fast-forward updates:

```bash
git pull origin main      # merges remote into your work
# Resolve conflicts if prompted (see section 4)
git push origin main
```

---

## 3) Optional: short-lived feature branches

```bash
git checkout -b feat/topic-xyz
# ...edit...
git add .
git commit -m "Implement xyz"
git checkout main
git pull origin main
git merge feat/topic-xyz   # or: git rebase main (advanced)
git push origin main
git branch -d feat/topic-xyz
```

---

## 4) Resolve merge conflicts (quick path)

When `git pull` reports conflicts:

1. Open the listed files and look for conflict markers:

   ```
   <<<<<<< HEAD
   your local content
   =======
   incoming content from origin/main
   >>>>>>> origin/main
   ```
2. Manually edit to keep the correct content, then:

   ```bash
   git add <file1> <file2> ...
   git commit                  # completes the merge
   git push origin main
   ```

If you want to abort a problematic merge:

```bash
git merge --abort
```

---

## 5) Keep the repo clean

Create/update `.gitignore` (adjust to your stack):

```
# macOS
.DS_Store

# Node / Hugo / build outputs
node_modules/
dist/
public/
resources/
.hugo_build.lock

# Logs / temp
*.log
*.tmp
```

Commit once:

```bash
git add .gitignore
git commit -m "chore: add .gitignore"
git push origin main
```

---

## 6) Submodules (only if using an external theme/code as a submodule)

* **Clone with submodules**

  ```bash
  git clone --recurse-submodules git@github.com:Wximing/personal_website.git
  ```

  Or after a normal clone:

  ```bash
  git submodule update --init --recursive
  ```

* **Pull updates including submodules**

  ```bash
  git pull --recurse-submodules
  git submodule update --init --recursive
  ```

* **Update a submodule to its latest remote**

  ```bash
  cd path/to/submodule
  git checkout main && git pull
  cd -
  git add path/to/submodule
  git commit -m "chore: bump submodule"
  git push
  ```

If you **don’t** use submodules, ignore this section.

---

## 7) Common commands (cheat sheet)

```bash
# status / history
git status
git log --oneline --graph --decorate
git diff

# switch / create branches
git checkout main
git checkout -b feat/my-change

# sync
git pull origin main
git push origin main

# undo (use with care)
git restore <file>            # discard unstaged local changes (newer Git)
git checkout -- <file>        # legacy form
git reset --soft HEAD~1       # undo last commit, keep changes staged
git reset --hard HEAD~1       # drop last commit + changes (dangerous)
```

---

## 8) Minimal “session recipe”

When you move to another computer:

```bash
# 1) Go to repo and sync
cd personal_website
git checkout main
git pull origin main

# 2) Work and save
# ...edit files...
git add .
git commit -m "Describe what changed"

# 3) Publish
git push origin main
```

---

## 9) Troubleshooting

* **`fatal: not a git repository`**
  You’re not inside the repo. `cd` into `personal_website`.

* **`Permission denied (publickey)`**
  SSH key not added or wrong key. Re-add your SSH key to GitHub and `ssh-add` (see section 0C).

* **`non-fast-forward` when pushing**
  Someone (or another machine) pushed first. `git pull origin main`, resolve conflicts, then `git push`.

* **Windows line endings look noisy in diffs**
  Ensure `core.autocrlf true` on Windows and `input` on macOS/Linux (see section 0B).

* **Long paths on Windows**
  Enable `core.longpaths true` (see section 0C).
