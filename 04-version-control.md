# Chapter 4: Version Control

[← Previous: Package Management & package.json](./03-package-management.md) | [Back to README](./README.md) | [Next: Linting →](./05-linting.md)

---

Imagine writing a long essay and accidentally deleting three paragraphs you needed. Or working with a teammate who overwrites your changes. Or shipping a bug and not being able to find what changed. Version control solves all of these problems. It's one of the most important tools in any developer's workflow, and in this chapter you'll learn the essentials.

---

## What is Version Control?

Version control is a system that tracks changes to your files over time. Every time you save a "snapshot" of your project (called a **commit**), the version control system records:

- What changed
- When it changed
- Who made the change
- A message explaining why

This gives you a complete history of your project. You can travel back to any previous snapshot, compare what changed between two points in time, and undo mistakes without losing work.

For software development, version control also enables **collaboration**. Multiple developers can work on the same codebase simultaneously by working on separate **branches** — isolated copies of the code — and then merging their work together. Modern platforms like GitHub provide tools for reviewing, discussing, and approving changes before they're merged.

The key benefits:

- **History** — see every change ever made to the project
- **Undo** — revert any file or the entire project to a previous state
- **Collaboration** — multiple people can work in parallel without overwriting each other
- **Branching** — experiment safely without affecting the main codebase
- **Accountability** — every change is attributed to an author with a timestamp

---

## Git

[Git](https://github.com/git/git) is by far the most widely used version control system in the world. It's:

- **Distributed** — every developer has a full copy of the repository history on their machine, not just the latest version
- **Fast** — most operations happen locally without needing a network connection
- **Free and open source** — maintained by a global community
- **Battle-tested** — used by millions of projects, from small personal tools to the Linux kernel

### Key concepts

| Concept | What it means |
|---|---|
| **Repository (repo)** | A project folder tracked by Git, containing all files and their history |
| **Commit** | A saved snapshot of changes, with a message describing what changed |
| **Branch** | An independent line of development — changes on one branch don't affect others |
| **Remote** | A copy of the repository hosted somewhere else (e.g., GitHub) |
| **Clone** | Downloading a remote repository to your local machine |
| **Push** | Uploading your local commits to the remote |
| **Pull** | Downloading new commits from the remote to your local machine |

---

## Installing Git

Git may already be installed on your system. Check with:

```bash
git --version
```

If it's not installed:

**macOS** (using [Homebrew](https://brew.sh)):
```bash
brew install git
```

**Linux (Debian/Ubuntu)**:
```bash
sudo apt update && sudo apt install git
```

**Linux (Fedora/RHEL)**:
```bash
sudo dnf install git
```

**Windows**: Download the installer from [git-scm.com](https://git-scm.com/downloads). This also installs Git Bash, a terminal emulator for running Git commands on Windows.

After installing, verify it works:
```bash
git --version
# git version 2.45.0
```

---

## Basic Git Workflow for TypeScript Projects

Here's the workflow you'll use when starting a new project and making your first commits.

### Step 1: Configure your identity

Before your first commit, tell Git who you are. This information is embedded in every commit you make.

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

The `--global` flag applies this to all repositories on your machine. You only need to do this once.

### Step 2: Initialize a repository

Navigate to your project folder and initialize a Git repository:

```bash
git init
```

This creates a hidden `.git/` folder that stores all the version history. Your project is now a Git repository.

### Step 3: Check what's changed

```bash
git status
```

This shows you which files are new, modified, or deleted since your last commit. Run this often — it's your main tool for understanding the current state of your working directory.

### Step 4: Stage your changes

Before committing, you tell Git exactly which changes to include. This is called **staging**:

```bash
# Stage a specific file
git add src/index.ts

# Stage all changes in the current directory
git add .
```

Staging lets you make multiple changes but commit them in logical groups. For example, you might stage and commit a bug fix separately from a new feature, even if you wrote both at the same time.

### Step 5: Commit

```bash
git commit -m "Initial commit"
```

The `-m` flag lets you write a short commit message inline. A good commit message is a brief, present-tense description of what changed and why:

```bash
git commit -m "Add user authentication endpoint"
git commit -m "Fix crash when input is empty string"
git commit -m "Upgrade fastify to v5"
```

### Step 6: View history

```bash
git log --oneline
```

This shows a compact list of commits, newest first:

```
a3f1c2e Add user authentication endpoint
b7d4f91 Fix crash when input is empty string
c1e8a30 Initial commit
```

Putting it all together in a real session:

```bash
# Initialize a new project
mkdir my-typescript-app
cd my-typescript-app
pnpm init
git init

# Configure identity (first time only)
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Check what's untracked
git status

# Stage everything
git add .

# Commit the initial project setup
git commit -m "Initial commit"

# View the history
git log --oneline
```

---

## .gitignore for TypeScript and Node.js Projects

Not everything in your project folder should be committed to Git. The `node_modules/` folder is enormous and can be recreated from the lock file. Build output in `dist/` is generated from source code. Environment files often contain secrets like API keys.

A `.gitignore` file tells Git which files and folders to ignore. Place it in the root of your project:

```bash
touch .gitignore
```

Here is a comprehensive `.gitignore` for TypeScript and Node.js projects:

```gitignore
# Dependencies
node_modules/

# Build output
dist/
build/
.output/
out/

# TypeScript cache
*.tsbuildinfo

# Test coverage
coverage/
.nyc_output/

# Environment variables — never commit secrets!
.env
.env.local
.env.*.local

# Logs
logs/
*.log
npm-debug.log*
pnpm-debug.log*

# Editor files
.vscode/
.idea/
*.swp
*.swo
*~

# OS files
.DS_Store
Thumbs.db

# Vite
.vite/

# Misc
.cache/
```

A few entries deserve special attention:

- **`node_modules/`** — always ignore this. It can be hundreds of megabytes and is fully reproducible from your lock file.
- **`.env` and friends** — environment files often contain API keys, database passwords, and other secrets. Committing these to a public repository is a serious security risk. Ignore them and share secrets through a secure channel (like a password manager or a secrets management service).
- **`dist/`** — generated build output. It's derived from your source code, so it doesn't belong in version history.
- **`.DS_Store`** — a macOS metadata file. Not relevant to the project and clutters the repository.

> **Note:** You'll notice `.vscode/` is in the ignore list. This is a common convention for personal editor settings. However, some teams choose to commit shared VS Code settings like recommended extensions or workspace-specific debug configurations. If your team does this, remove `.vscode/` from `.gitignore` and be selective about which files inside it you commit.

---

## Branching Workflow

Branches let you develop features and fix bugs in isolation, without affecting the main codebase until you're ready.

The standard approach used by most teams is called **GitHub Flow**:

1. The `main` branch always contains working, production-ready code
2. For every new feature or fix, create a new branch from `main`
3. Make commits on your feature branch
4. Open a **Pull Request (PR)** on GitHub to propose merging your branch into `main`
5. A teammate reviews your code and leaves comments
6. Once approved, the branch is merged into `main`
7. The feature branch is deleted

```bash
# Create and switch to a new branch
git checkout -b feature/add-user-auth

# Make changes, then stage and commit
git add .
git commit -m "Add user authentication endpoint"

# Push the branch to GitHub
git push -u origin feature/add-user-auth

# After the PR is merged, switch back to main and update
git checkout main
git pull
```

Naming conventions for branches:

- `feature/description` — new features
- `fix/description` — bug fixes
- `chore/description` — maintenance tasks (dependency updates, config changes)
- `docs/description` — documentation changes

---

## GitHub

[GitHub](https://github.com) is the most popular platform for hosting Git repositories. It adds a web interface on top of Git with tools for collaboration: Pull Requests, Issues, code review, project boards, and GitHub Actions for CI/CD.

### Creating a repository

1. Sign in at [github.com](https://github.com)
2. Click the **+** button in the top right and choose **New repository**
3. Give it a name and description
4. Choose **Public** or **Private**
5. Do NOT initialize with a README if you already have a local project — you'll push your own

### Connecting your local project to GitHub

After creating the repository on GitHub, connect your local project and push your code:

```bash
# Add the GitHub repository as the "origin" remote
git remote add origin https://github.com/username/my-typescript-app.git

# Rename the default branch to "main" (if it's not already)
git branch -M main

# Push your local commits to GitHub for the first time
# The -u flag sets "origin main" as the default upstream, so future pushes
# can just be: git push
git push -u origin main
```

After this, your code is on GitHub. Future pushes are simply:

```bash
git push
```

And to download changes made by teammates (or from another machine):

```bash
git pull
```

### Authenticating with GitHub

When you push to GitHub, you'll need to authenticate. The recommended method is **SSH keys**, which let you push without entering a password each time.

To set up SSH authentication:

```bash
# Generate an SSH key (accept the default file location)
ssh-keygen -t ed25519 -C "you@example.com"

# Copy the public key to your clipboard (macOS)
pbcopy < ~/.ssh/id_ed25519.pub

# Copy the public key (Linux)
cat ~/.ssh/id_ed25519.pub
```

Then go to GitHub → Settings → SSH and GPG keys → New SSH key, and paste your public key. After that, use the SSH remote URL:

```bash
git remote add origin git@github.com:username/my-typescript-app.git
```

---

## What's Next?

You now have the tools to track your code, collaborate with others, and maintain a clean project history. In the next chapter, we'll look at linting — automated tools that read your code and flag problems before you even run it.

[← Previous: Package Management & package.json](./03-package-management.md) | [Back to README](./README.md) | [Next: Linting →](./05-linting.md)
