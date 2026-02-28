# Chapter 2: Node.js Environments

[← Previous: Introduction](./01-introduction.md) | [Back to README](./README.md) | [Next: Package Management →](./03-package-management.md)

---

## What Is Node.js?

Node.js is a **JavaScript runtime** — it's the program that lets you run JavaScript (and by extension, TypeScript tooling) outside of a web browser. Before Node.js existed, JavaScript could only run in browsers. Node.js changed that by taking Chrome's V8 JavaScript engine and packaging it so you can run JS on your laptop, a server, or anywhere else.

Official site: [https://nodejs.org](https://nodejs.org)

### Why You Need Node.js

Even if you're building something that runs entirely in a browser, you still need Node.js on your development machine because:

- All the TypeScript tooling (the compiler, Vite, ESLint, Vitest, etc.) is itself written in JavaScript and runs on Node.js
- The package manager (`npm`, which comes with Node.js) is how you install those tools and your project's dependencies
- Build scripts, test runners, and development servers all run via Node.js

In short: Node.js is the foundation that everything else in the TypeScript ecosystem sits on.

---

## LTS Releases: Which Version Should You Use?

Node.js has a structured release schedule that you should understand before installing anything.

### Even vs Odd Versions

Node.js releases new major versions twice a year:

- **Even-numbered versions** (18, 20, 22, 24...) become **Long Term Support (LTS)** releases
- **Odd-numbered versions** (17, 19, 21...) are **Current** releases for early adopters, and they are **never** promoted to LTS

### What LTS Means

An LTS release goes through three phases:

1. **Active LTS** — receives new features, bug fixes, and security patches (roughly 18 months)
2. **Maintenance LTS** — receives only critical bug fixes and security patches (roughly 12 more months)
3. **End of Life (EOL)** — no more updates; you should upgrade

LTS versions get approximately **30 months** of support total. That's long enough to build a project, ship it, and have time to plan upgrades without being forced to rush.

### Current Release Schedule

| Version | Status | Approx. End of Life |
|---|---|---|
| Node 18 | Maintenance LTS | April 2025 |
| Node 20 | Active LTS | April 2026 |
| Node 22 | Active LTS | April 2027 |
| Node 23 | Current (non-LTS) | June 2025 |

For the current, authoritative schedule, always check: [https://nodejs.org/en/about/releases](https://nodejs.org/en/about/releases)

### The Rule of Thumb

**Always use an LTS version for projects and learning.** Unless you have a specific reason to try bleeding-edge features, stick to the most recent Active LTS version (Node 20 or Node 22 at the time of writing). Odd-numbered Current releases are for developers who want to preview upcoming features — they're not suited for production use or new projects.

---

## The Version Problem

Here's a situation you'll run into quickly once you're working on more than one TypeScript project:

- **Project A** was started two years ago and requires Node 18
- **Project B** is a new project and uses Node 22

If you just install one version of Node.js on your machine (the way you'd install most software), you're stuck. You'd have to uninstall and reinstall Node every time you switch projects. That's obviously not practical.

This is the same problem Python developers solve with `venv` and `pyenv` — different projects need isolated environments with specific runtime versions.

The solution is a **Node version manager**: a tool that lets you install multiple versions of Node.js and switch between them easily.

There are three popular options in the Node.js ecosystem: **nvm**, **Volta**, and **mise**. We'll cover all three so you can choose the one that fits your workflow.

---

## nvm — Node Version Manager

[https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

nvm is the original and most widely used Node version manager. It's been around since 2010, has a huge user base, and you'll see it referenced in countless tutorials and documentation pages.

### Installation

**macOS and Linux:**

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
```

This script downloads nvm and adds the necessary configuration to your shell profile (`.bashrc`, `.zshrc`, or similar). After it runs, **close and reopen your terminal** (or run `source ~/.bashrc` / `source ~/.zshrc`) so the changes take effect.

**Windows:**

nvm itself doesn't support Windows natively. Use [nvm-windows](https://github.com/coreybutler/nvm-windows) instead — it's a separate project that provides similar functionality on Windows. Download the installer from that repository's releases page.

### Verifying the Installation

```bash
nvm --version
# Output: 0.40.0 (or similar)
```

### Common Commands

Install the latest LTS version:

```bash
nvm install --lts
```

Install a specific version:

```bash
nvm install 20
```

Switch to an installed version:

```bash
nvm use 20
```

Set the default version (used when you open a new terminal):

```bash
nvm alias default 20
```

See all installed versions:

```bash
nvm ls
```

See all available versions you could install:

```bash
nvm ls-remote --lts
```

### The .nvmrc File

The real power of nvm for project work is the `.nvmrc` file. You create this file in the root of your project with just the Node version number inside:

```bash
echo "20" > .nvmrc
```

Now, anyone who clones your project and has nvm installed can just run:

```bash
nvm use
```

...and nvm will automatically read `.nvmrc` and switch to the right version. No need to remember which version the project needs.

You can also configure your shell to automatically run `nvm use` whenever you `cd` into a directory that has a `.nvmrc` file. The nvm README has instructions for setting this up in bash, zsh, and fish.

### The Downside of nvm

nvm works by modifying your `PATH` environment variable in your shell profile. This means:

- It only works in interactive shells (a minor annoyance in some CI environments)
- It can slow down shell startup slightly
- You have to remember to run `nvm use` when switching projects (unless you set up the auto-switch hook)

These are minor issues, but they motivated the creation of the alternatives below.

---

## Volta — The Hassle-Free Node Manager

[https://volta.sh](https://volta.sh) — [https://github.com/volta-cli/volta](https://github.com/volta-cli/volta)

Volta takes a different approach to the version management problem. Instead of requiring you to run a command to switch versions, Volta intercepts calls to `node`, `npm`, and other tools and automatically routes them to the correct version — transparently, with no action required on your part.

When you `cd` into a project that has a pinned Node version, Volta automatically uses that version. When you leave the project directory, it switches back. You never have to think about it.

### Installation

**macOS and Linux:**

```bash
curl https://get.volta.sh | bash
```

Restart your terminal after installation.

**Windows:**

Download the official installer from [https://volta.sh](https://volta.sh). It's a standard `.msi` installer — just run it.

Verify Volta is installed:

```bash
volta --version
# Output: 2.0.2 (or similar)
```

### Installing a Node Version

```bash
volta install node@20
```

This installs Node 20 and sets it as your default. You can also install the latest LTS:

```bash
volta install node@lts
```

### Pinning a Version to Your Project

This is where Volta really shines. From inside your project directory, run:

```bash
volta pin node@20
```

Volta adds a `"volta"` field to your `package.json`:

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "volta": {
    "node": "20.15.0"
  }
}
```

Now whenever anyone working on this project runs `node` or any npm scripts, Volta ensures they're using exactly Node 20.15.0 — automatically, no `nvm use` required. This is a big deal for teams: Node version mismatches between developers are a common source of subtle bugs, and Volta eliminates them.

### Why Volta Is Great

- **Automatic** — no shell hooks, no `.nvmrc`, no remembering to switch versions
- **Fast** — Volta is written in Rust and adds essentially zero overhead to command invocations
- **Works great in CI** — because it doesn't require shell initialization, it works reliably in GitHub Actions and other CI environments
- **Per-project tool pinning** — you can also pin npm, yarn, and global tools (like `typescript`) per project

---

## mise — The Polyglot Version Manager

[https://mise.jdx.dev](https://mise.jdx.dev) — [https://github.com/jdx/mise](https://github.com/jdx/mise)

mise (pronounced "meez", formerly called rtx) takes a broader approach: it manages runtime versions for **many languages**, not just Node.js. If you work across multiple ecosystems — say, you write TypeScript for some projects and Python or Go for others — mise can manage all of them from a single tool.

mise is compatible with [asdf](https://asdf-vm.com/), another polyglot version manager, so you can use the same `.tool-versions` config files.

### Installation

**macOS (with Homebrew):**

```bash
brew install mise
```

**macOS and Linux (without Homebrew):**

```bash
curl https://mise.run | sh
```

**Windows:**

```bash
winget install jdx.mise
```

After installing, add mise to your shell profile so it activates when you open a terminal.

**bash:**

```bash
echo 'eval "$(mise activate bash)"' >> ~/.bashrc
source ~/.bashrc
```

**zsh:**

```bash
echo 'eval "$(mise activate zsh)"' >> ~/.zshrc
source ~/.zshrc
```

**fish:**

```bash
echo 'mise activate fish | source' >> ~/.config/fish/config.fish
```

Verify the installation:

```bash
mise --version
# Output: 2024.x.x (or similar)
```

### Installing and Using Node

Install the latest LTS version of Node and set it as the project default:

```bash
mise use node@lts
```

This creates a `.mise.toml` file in the current directory:

```toml
[tools]
node = "lts"
```

Or install a specific version:

```bash
mise use node@20.15.0
```

### The .tool-versions Format

mise also supports the `.tool-versions` format from asdf, which looks like this:

```
node 20.15.0
python 3.12.0
```

This is useful if you're on a team that uses asdf, or if you want a single file to declare all your project's runtime versions in one place.

### Managing Other Languages

Here's where mise earns its keep for polyglot developers. You can manage multiple runtimes in one `.mise.toml`:

```toml
[tools]
node = "lts"
python = "3.12"
go = "1.22"
```

Run `mise install` in a directory with this file and all three runtimes will be installed and activated.

---

## Which Tool Should You Choose?

Here's a side-by-side comparison to help you decide:

| Feature | nvm | Volta | mise |
|---|---|---|---|
| **Automatic version switching** | With shell hook setup | Yes, always | Yes, always |
| **Speed** | Good | Excellent (Rust) | Excellent (Rust) |
| **Windows support** | Via nvm-windows | Yes (official) | Yes |
| **Multi-language support** | No (Node only) | No (Node/npm/yarn) | Yes |
| **Config file** | `.nvmrc` | `package.json` (`volta` field) | `.mise.toml` or `.tool-versions` |
| **CI/CD friendliness** | Good | Excellent | Excellent |
| **Community adoption** | Very widespread | Growing | Growing |
| **Installation complexity** | Simple | Simple | Simple |

### Recommendations

- **Choose Volta** if you want the simplest, most automatic experience and you primarily work with JavaScript/TypeScript. The automatic switching and `package.json` integration make it very smooth, especially on teams.

- **Choose mise** if you work with multiple programming languages and want one tool to rule them all. It's especially compelling if you also write Python, Go, Ruby, or other languages.

- **Choose nvm** if you prefer the classic, battle-tested approach and don't mind running `nvm use` manually (or setting up the auto-switch hook). It has the largest community and the most documentation online.

All three are solid choices used by professional developers. Pick one, get comfortable with it, and don't spend too much time agonizing over the decision — you can always switch later.

---

## What's Next?

You've got Node.js installed and a version manager in place. The next big topic is understanding how Node.js projects manage their dependencies — the libraries and tools your project depends on.

In Chapter 3, we'll cover `package.json`, the `node_modules` folder, and the difference between npm, yarn, and pnpm.

[← Previous: Introduction](./01-introduction.md) | [Back to README](./README.md) | [Next: Package Management →](./03-package-management.md)
