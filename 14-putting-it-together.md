# Chapter 14: Putting It All Together

[← Previous: LLMs and AI-Powered Development](./13-llms.md) | [Back to README](./README.md) | [Next: Using This Setup for JavaScript Projects →](./15-javascript.md)

---

You have learned a lot of individual tools across the previous 13 chapters. Now it is time to put them all together. In this chapter, we will build a complete TypeScript project from scratch — following every step in order — so you can see how all the pieces fit.

## Overview

Here is a quick recap of what we have covered:

| Tool | Purpose |
|---|---|
| Volta | Node.js version management |
| pnpm | Fast, efficient package manager |
| TypeScript | Static types and compilation |
| Biome | Linting and formatting |
| Vitest | Fast unit testing with coverage |
| Vite | Bundler and dev server |
| Fastify | Backend HTTP server |
| TanStack Start | Full-stack React framework |
| Drizzle | Type-safe ORM |
| GitHub Actions | CI/CD automation |

We will apply all of them in a single walkthrough. The result will be a project you can use as a template for future work.

---

## Step 1: Set Up Node.js with Volta

First, make sure Volta is installed. Volta manages your Node.js versions so different projects can use different Node versions without conflicting.

```bash
# Install Volta
curl https://get.volta.sh | bash
# Restart your terminal after installation
volta install node@20
node --version   # should show v20.x.x
```

Volta installs into your shell profile so that the right Node version is activated automatically when you enter a project directory.

---

## Step 2: Scaffold the Project with Vite

Vite provides an excellent project scaffolding command. We will use the `vanilla-ts` template, which gives us a minimal TypeScript setup without any framework — perfect for learning.

```bash
pnpm create vite my-typescript-app -- --template vanilla-ts
cd my-typescript-app
volta pin node@20   # pin Node version in package.json
pnpm install
```

The `volta pin` command adds a `volta` field to `package.json` that records which Node version this project uses. Anyone who clones the repository with Volta installed will automatically get the correct Node version.

After running `pnpm install`, your project structure looks like this:

```
my-typescript-app/
├── src/
│   ├── main.ts
│   ├── counter.ts
│   └── vite-env.d.ts
├── index.html
├── package.json
├── tsconfig.json
└── pnpm-lock.yaml
```

---

## Step 3: Configure TypeScript

Vite generates a basic `tsconfig.json`, but let's replace it with a stricter, production-ready configuration. Open `tsconfig.json` and replace its contents:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "skipLibCheck": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Each option is doing real work here:

- `strict: true` — enables the full suite of strict type checks
- `noUncheckedIndexedAccess` — accessing `array[0]` returns `T | undefined`, not just `T`, which prevents runtime crashes when the index does not exist
- `exactOptionalPropertyTypes` — distinguishes between a property being missing and a property being explicitly set to `undefined`
- `noImplicitReturns` — every code path in a function must return a value
- `noFallthroughCasesInSwitch` — prevents accidental fall-through in switch statements
- `moduleResolution: "Bundler"` — the correct setting when using Vite or another bundler (not the Node.js resolver)

---

## Step 4: Set Up Biome for Linting and Formatting

Biome replaces both ESLint and Prettier with a single, fast tool. Install it and initialize its config:

```bash
pnpm add -D @biomejs/biome
pnpm biome init
```

The `biome init` command creates a `biome.json` configuration file in your project root. The defaults are sensible, but you can customize rules there as your project grows.

Now add these scripts to the `"scripts"` section of `package.json`:

```json
"lint": "biome lint .",
"format": "biome format --write .",
"check": "biome check --write .",
"check:ci": "biome check .",
"typecheck": "tsc --noEmit"
```

Note the difference between `check` and `check:ci`:

- `check` applies fixes automatically — use this locally
- `check:ci` only checks without modifying files — use this in CI so the workflow fails if code is not formatted

---

## Step 5: Set Up Vitest with Coverage

Vitest is the natural testing framework for Vite projects. They share the same configuration format and the same transform pipeline, so TypeScript just works with no extra setup.

```bash
pnpm add -D vitest @vitest/coverage-v8
```

Create `vitest.config.ts` in the project root:

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
      thresholds: { lines: 80 },
    },
  },
});
```

A few things to note:

- `globals: true` — makes `describe`, `it`, `expect`, etc. available without importing them
- `environment: 'node'` — runs tests in a Node.js environment (use `'jsdom'` for DOM-related tests)
- `thresholds: { lines: 80 }` — the `test:coverage` command will fail if less than 80% of your code is covered by tests

Add the test scripts to `package.json`:

```json
"test": "vitest",
"test:run": "vitest run",
"test:coverage": "vitest run --coverage"
```

- `test` runs Vitest in watch mode — it re-runs tests when files change, great for development
- `test:run` runs once and exits — use this in CI and pre-commit hooks
- `test:coverage` runs once with the coverage report

---

## Step 6: Write the Application Code and Tests

Now let's write something worth testing. Create `src/calculator.ts`:

```typescript
export function add(a: number, b: number): number {
  return a + b;
}

export function divide(a: number, b: number): number {
  if (b === 0) throw new Error('Cannot divide by zero');
  return a / b;
}
```

These functions are simple, but they demonstrate two important patterns: a happy-path function and a function that can throw. Both need tests.

Create `src/calculator.test.ts`:

```typescript
import { describe, it, expect } from 'vitest';
import { add, divide } from './calculator';

describe('calculator', () => {
  it('adds numbers', () => expect(add(2, 3)).toBe(5));
  it('divides numbers', () => expect(divide(10, 2)).toBe(5));
  it('throws on divide by zero', () => {
    expect(() => divide(1, 0)).toThrow('Cannot divide by zero');
  });
});
```

Run the tests to confirm everything works:

```bash
pnpm test:run
```

You should see three passing tests. Now run with coverage:

```bash
pnpm test:coverage
```

The HTML coverage report is written to `coverage/index.html` — open it in a browser to see which lines are covered.

---

## Step 7: Set Up Git and .gitignore

Before initializing git, create a `.gitignore` file so you do not accidentally commit build output or dependencies:

```
# Dependencies
node_modules/

# Build output
dist/

# Coverage reports
coverage/

# Environment files
.env
.env.local
.env.*.local

# Editor directories
.vscode/
.idea/

# OS files
.DS_Store
Thumbs.db

# TypeScript cache
*.tsbuildinfo
```

Now initialize git and make the first commit:

```bash
git init
git add .
git commit -m "Initial TypeScript project setup"
```

---

## Step 8: Set Up GitHub Actions CI

Create the workflow directory and file:

```bash
mkdir -p .github/workflows
```

Create `.github/workflows/ci.yml` with the full workflow from Chapter 12:

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
        run: pnpm check:ci

      - name: Run tests with coverage
        run: pnpm test:coverage

      - name: Build
        run: pnpm build
```

Commit this file:

```bash
git add .github/
git commit -m "Add GitHub Actions CI workflow"
```

---

## The Complete package.json Scripts Section

After all the steps above, your `package.json` scripts section should look like this:

```json
"scripts": {
  "dev": "vite",
  "build": "tsc --noEmit && vite build",
  "preview": "vite preview",
  "test": "vitest",
  "test:run": "vitest run",
  "test:coverage": "vitest run --coverage",
  "lint": "biome lint .",
  "format": "biome format --write .",
  "check": "biome check --write .",
  "check:ci": "biome check .",
  "typecheck": "tsc --noEmit",
  "ci": "pnpm typecheck && pnpm check:ci && pnpm test:coverage && pnpm build"
}
```

Notice the `ci` script at the bottom. It chains all the quality checks together in the same order as the CI workflow. Running `pnpm ci` locally before pushing lets you catch any failures before they hit GitHub Actions.

---

## Step 9: Push to GitHub

If you have the [GitHub CLI](https://cli.github.com) installed, creating a GitHub repository and pushing is a single command:

```bash
gh repo create my-typescript-app --public --source=. --remote=origin
git push -u origin main
```

After pushing, navigate to your repository on GitHub. Under the **Actions** tab, you will see your CI workflow running. Watch it go green.

If you do not have the GitHub CLI, create a repository manually on github.com and follow the instructions to add it as a remote.

---

## Project Templates

Building a project from scratch is the best way to learn, but for future projects you often want to start from a template that already has all the boilerplate configured.

Some good starting points:

- **[cookiecutter-typescript-library](https://github.com/amitsk/cookiecutter-typescript-library)** — a template for TypeScript libraries with testing, linting, and CI already wired up
- **`pnpm create vite`** — official Vite scaffolding for frontend projects; supports React, Vue, Svelte, and vanilla TypeScript templates
- **`pnpm create tsrouter-app`** — scaffolds a TanStack Router project with TypeScript configured

Templates save you 30–60 minutes of setup on every new project. Once you have a setup you like, consider creating your own template repository on GitHub and using it as the starting point for future projects.

---

## Daily Development Workflow Summary

Once your project is set up, your day-to-day workflow is straightforward:

```bash
# Start dev server with hot reloading
pnpm dev

# Run tests in watch mode while you code
pnpm test

# Before committing: lint, format, type check, and run all tests
pnpm check        # lint + format (applies fixes automatically)
pnpm typecheck    # catch type errors
pnpm test:run     # verify all tests pass

# Commit and push
git add .
git commit -m "Add feature X"
git push
# CI runs automatically on GitHub
```

The `pnpm check` step handles both linting and formatting in one command. If Biome reformats any files, you will see them as modified in `git status` — stage them along with your code changes. Some developers add a pre-commit hook with [lefthook](https://github.com/evilmartians/lefthook) to run these checks automatically on every `git commit`.

---

## What's Next?

In the final chapter, we look at how to use everything you have learned for plain JavaScript projects — no TypeScript required. The whole toolchain works equally well with `.js` files, and the chapter also covers JSDoc typing and how to migrate from JavaScript to TypeScript incrementally.

[← Previous: LLMs and AI-Powered Development](./13-llms.md) | [Back to README](./README.md) | [Next: Using This Setup for JavaScript Projects →](./15-javascript.md)
