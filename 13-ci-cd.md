# Chapter 13: CI/CD

[← Previous: Logging](./12-logging.md) | [Back to README](./README.md) | [Next: LLMs and AI-Powered Development →](./14-llms.md)

---

So far, you have been running tests, type checks, and linting manually on your own machine. That works fine while you are the only developer, but what happens when you push code to a shared repository? Or when someone opens a pull request and you want to make sure it does not break anything? This is where CI/CD comes in.

## What is CI/CD?

**CI** stands for Continuous Integration. It means automatically running your tests, linting, and type-checking every time code is pushed to a repository or a pull request is opened. Instead of relying on developers to remember to run checks before pushing, the process is automated and enforced.

**CD** stands for Continuous Deployment. It means automatically deploying your application to a server or hosting platform after CI passes. If the tests all pass, the code goes live — no manual deployment steps required.

### Why does this matter?

- **Catch bugs early**: Problems are found when a PR is opened, not after merging into `main`.
- **Consistent quality**: Every branch gets the same checks — no one accidentally skips linting.
- **No "works on my machine"**: CI runs in a clean, reproducible environment. If it passes there, it will pass everywhere.
- **Fast feedback loops**: You get notified within minutes whether your code is good to merge.

### Why CI is especially important for TypeScript

TypeScript's type system is enforced at compile time, not runtime. If someone disables type checking locally or ignores errors, type mistakes can slip into a branch unnoticed. CI ensures `tsc --noEmit` runs on every branch, so type errors never make it to `main`.

---

## GitHub Actions — A Short Explainer

[GitHub Actions](https://github.com/features/actions) is a CI/CD platform built directly into GitHub. It is free for public repositories and has a generous free tier for private repositories (2,000 minutes per month on the free plan).

Actions are triggered by **events** — things that happen in your repository. Common events include:

- `push` — code is pushed to a branch
- `pull_request` — a PR is opened, updated, or synchronized
- `schedule` — a cron-style schedule (e.g., run tests every night)
- `workflow_dispatch` — manually triggered from the GitHub UI

### Key concepts

Before writing your first workflow, it helps to understand the vocabulary:

- **Workflow**: The entire automated process, defined in a `.yml` file. A repo can have multiple workflows.
- **Job**: A group of steps that run together on one machine. Multiple jobs can run in parallel.
- **Step**: A single task within a job — either a shell command or a pre-built action.
- **Runner**: The virtual machine that runs your job. GitHub provides `ubuntu-latest`, `windows-latest`, and `macos-latest`.
- **Action**: A reusable, shareable step published on the GitHub Marketplace (e.g., `actions/checkout@v4`).

Workflows are defined in YAML files inside `.github/workflows/`. GitHub automatically picks them up.

---

## A Complete CI Workflow for a TypeScript Project

Let's build a real workflow. Create the directory and file:

```bash
mkdir -p .github/workflows
```

Then create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  quality:
    name: Quality Checks
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [20.x, 22.x]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install pnpm
        uses: pnpm/action-setup@v4
        with:
          version: 9

      - name: Set up Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Type check
        run: pnpm typecheck

      - name: Lint
        run: pnpm lint

      - name: Check formatting
        run: pnpm format:check

      - name: Run tests with coverage
        run: pnpm test:coverage

      - name: Build
        run: pnpm build
```

This single file gives you a full quality pipeline that runs automatically on every push and every pull request.

---

## Understanding Each Step

Let's walk through the workflow step by step so you know exactly what each part does.

### `actions/checkout@v4`

```yaml
- name: Checkout code
  uses: actions/checkout@v4
```

The runner starts as a blank virtual machine. This step clones your repository onto it so the rest of the workflow has code to work with. It is almost always the first step in any job.

### `pnpm/action-setup@v4`

```yaml
- name: Install pnpm
  uses: pnpm/action-setup@v4
  with:
    version: 9
```

GitHub runners have Node.js pre-installed, but they do not have pnpm. This official pnpm action installs it. Specifying `version: 9` ensures you always get the same pnpm version regardless of what GitHub has available.

### `actions/setup-node@v4` with `cache: 'pnpm'`

```yaml
- name: Set up Node.js ${{ matrix.node-version }}
  uses: actions/setup-node@v4
  with:
    node-version: ${{ matrix.node-version }}
    cache: 'pnpm'
```

This installs the specified version of Node.js. The `cache: 'pnpm'` option tells the action to cache the pnpm store between runs — more on this in the Caching section below. The `${{ matrix.node-version }}` syntax reads from the matrix strategy defined above.

### `pnpm install --frozen-lockfile`

```yaml
- name: Install dependencies
  run: pnpm install --frozen-lockfile
```

This installs your project's dependencies exactly as specified in `pnpm-lock.yaml`. The `--frozen-lockfile` flag means the command will fail if the lockfile is out of date — it will never silently update the lockfile the way a plain `pnpm install` might. This is equivalent to `npm ci` and is the correct way to install dependencies in CI.

### Type check, lint, format:check, test:coverage, build

```yaml
- name: Type check
  run: pnpm typecheck

- name: Lint
  run: pnpm lint

- name: Check formatting
  run: pnpm format:check

- name: Run tests with coverage
  run: pnpm test:coverage

- name: Build
  run: pnpm build
```

Each of these maps directly to a script in your `package.json`. If any script exits with a non-zero code (meaning it found an error), the step fails, the job fails, and GitHub marks the commit or PR as failing. This is how CI enforces quality — it refuses to let bad code pass.

---

## Matrix Builds

The `strategy.matrix` section is one of the most powerful features of GitHub Actions:

```yaml
strategy:
  matrix:
    node-version: [20.x, 22.x]
```

This tells GitHub Actions to run the entire `quality` job **twice** — once with Node 20 and once with Node 22. The jobs run in parallel, so the total time is only slightly longer than running it once.

### Why does this matter?

Node.js has a Long-Term Support (LTS) release schedule. Node 20 is the current LTS; Node 22 is the latest. By testing on both, you ensure your project works for developers on older Node versions and is compatible with the newest version too.

### Extending the matrix to multiple operating systems

You can add an `os` dimension to test across platforms:

```yaml
strategy:
  matrix:
    node-version: [20.x, 22.x]
    os: [ubuntu-latest, windows-latest, macos-latest]
runs-on: ${{ matrix.os }}
```

This creates 6 combinations (2 Node versions × 3 operating systems). For most web projects this is overkill, but it is essential for CLI tools or libraries that run on end-user machines.

---

## Adding a Status Badge to Your README

Once your workflow is running, you can display a live status badge in your README:

```markdown
![CI](https://github.com/yourusername/your-repo/actions/workflows/ci.yml/badge.svg)
```

Replace `yourusername` and `your-repo` with your GitHub username and repository name. The badge shows green when CI is passing and red when it is failing. This gives visitors instant confidence (or a warning) about the health of your project.

---

## Caching for Speed

CI pipelines can be slow if they download all your dependencies from the internet on every run. The `cache: 'pnpm'` option in the `setup-node` step solves this:

- On the **first run**, pnpm downloads all packages and stores them in the pnpm content-addressable store. GitHub Actions saves this store as a cache keyed by your lockfile.
- On **subsequent runs**, if the lockfile has not changed, GitHub restores the cache instead of downloading packages. Installation becomes a matter of linking files rather than downloading them.

The practical effect is dramatic:

- First run (cold cache): 2–3 minutes
- Subsequent runs (warm cache): 15–30 seconds for most projects

The cache is automatically invalidated when `pnpm-lock.yaml` changes, so you always get fresh packages after adding or updating a dependency.

---

## Running CI Locally

You do not have to push to GitHub every time you want to verify your workflow. There are two good options:

### Option 1: Run the commands directly

The simplest approach is to run the same commands your CI workflow runs:

```bash
pnpm typecheck && pnpm lint && pnpm test:run && pnpm build
```

If this passes on your machine, it will almost certainly pass in CI.

### Option 2: Use `act`

[`act`](https://github.com/nektos/act) is an open-source tool that runs GitHub Actions workflows locally using Docker. It downloads the same runner images GitHub uses and executes your `.yml` files on your machine.

```bash
# Install act (macOS)
brew install act

# Run your CI workflow locally
act push
```

`act` is especially useful for debugging complex workflows with multiple jobs or environment variables, where it is tedious to push a commit just to see if a YAML change works.

---

## What's Next?

Your TypeScript project is now automatically tested and verified on every push. In the next chapter, we will look at AI-powered development tools — including Cursor, Claude Code, and GitHub Copilot — and how they can accelerate your TypeScript workflow.

[← Previous: Logging](./12-logging.md) | [Back to README](./README.md) | [Next: LLMs and AI-Powered Development →](./14-llms.md)
