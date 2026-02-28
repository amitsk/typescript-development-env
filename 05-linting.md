# Chapter 5: Linting

[← Previous: Version Control](./04-version-control.md) | [Back to README](./README.md) | [Next: Code Formatting →](./06-code-formatting.md)

---

You've written some TypeScript, your editor isn't showing any red squiggles, but your code still has problems — an unused variable here, a function that could return `undefined` unexpectedly there. A **linter** catches these issues automatically, before you ever run your code. In this chapter you'll learn what linting is, how to set up ESLint with TypeScript support, and how a blazing-fast Rust-based alternative called Oxlint fits into the picture.

---

## What is Linting?

Linting is the process of **statically analyzing your code** — reading it without running it — to find bugs, enforce conventions, and flag potentially dangerous patterns.

The name comes from "lint," a Unix tool from 1978 that found suspicious C code. Modern linters are far more sophisticated, and for TypeScript projects they can catch:

- **Undefined or undeclared variables**
- **Unreachable code** (code after a `return` statement)
- **Unsafe type patterns** — like using `any` when a real type would work
- **Unused imports and variables** that clutter the code
- **Potential null/undefined errors** before they become runtime crashes
- **Inconsistent code style** that makes a codebase harder to read

The benefits go beyond catching bugs. A linter acts as an automated code reviewer that runs in milliseconds. It enforces the same rules across an entire team, so code style discussions happen in the config file once, not in every code review forever.

Here's a concrete example. This TypeScript code compiles without errors, but a linter would flag multiple problems:

```typescript
// Problems a linter catches that TypeScript's compiler misses:

import { readFile } from 'fs';  // unused import

function greet(name: any) {     // "any" disables type safety
  const greeting = "Hello";
  return `Hi, ${name}!`;        // "greeting" is declared but never used
  console.log("Done");          // unreachable — after the return
}
```

A linter sees all four of those issues immediately, without running the code.

---

## ESLint — the Standard

[ESLint](https://eslint.org) ([GitHub](https://github.com/eslint/eslint)) is the most widely used linter for JavaScript and TypeScript. It's been the standard for over a decade and has a massive ecosystem of plugins and rules.

What makes ESLint powerful:

- **Hundreds of built-in rules** covering correctness, best practices, and style
- **Plugin system** — add rules for React, TypeScript, accessibility, security, and more
- **Autofix** — many rules can automatically fix the problems they find
- **Highly configurable** — enable or disable any rule, or change it from an error to a warning

### Installation

Install ESLint along with its JavaScript ruleset and the TypeScript integration:

```bash
pnpm add -D eslint @eslint/js typescript-eslint
```

- **`eslint`** — the core linter engine
- **`@eslint/js`** — the official ESLint ruleset for JavaScript (provides `eslint.configs.recommended`)
- **`typescript-eslint`** — the bridge between ESLint and TypeScript (explained below)

### typescript-eslint

[typescript-eslint](https://typescript-eslint.io) is the essential companion for using ESLint with TypeScript. It provides two things:

1. **A TypeScript parser** — ESLint can't read TypeScript syntax on its own. typescript-eslint lets ESLint understand TypeScript's type annotations, generics, and other constructs.
2. **TypeScript-specific rules** — additional lint rules that understand TypeScript's type system, like:
   - `@typescript-eslint/no-explicit-any` — ban the `any` type
   - `@typescript-eslint/explicit-function-return-type` — require return type annotations
   - `@typescript-eslint/no-floating-promises` — catch unhandled promises
   - `@typescript-eslint/strict-boolean-expressions` — prevent accidental falsy checks

The most powerful feature is **type-aware linting**: typescript-eslint can run the TypeScript compiler in the background and use type information to power rules that would be impossible without it. For example, it can warn you when you `await` a value that isn't actually a Promise.

### Flat config format (ESLint v9+)

ESLint v9 introduced a new configuration format called **flat config**. Instead of the old `.eslintrc.json` or `.eslintrc.js`, you now use a single `eslint.config.js` (or `eslint.config.ts`) file.

The flat config format is simpler and more explicit — there's no magic file-lookup behavior, just a regular JavaScript file that exports an array of config objects.

Create `eslint.config.ts` in your project root:

```typescript
import eslint from "@eslint/js";
import tseslint from "typescript-eslint";

export default tseslint.config(
  // Apply ESLint's recommended rules for JavaScript
  eslint.configs.recommended,

  // Apply typescript-eslint's recommended rules with type checking
  ...tseslint.configs.recommendedTypeChecked,

  // Configure the TypeScript parser to find your tsconfig.json
  {
    languageOptions: {
      parserOptions: {
        project: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
  },

  // Add or override specific rules for your project
  {
    rules: {
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/explicit-function-return-type": "warn",
      "no-console": "warn",
    },
  },

  // Tell ESLint to skip generated files
  {
    ignores: ["dist/**", "node_modules/**"],
  },
);
```

Let's walk through this config:

- **`eslint.configs.recommended`** — turns on a curated set of core ESLint rules that catch common JavaScript mistakes.
- **`tseslint.configs.recommendedTypeChecked`** — turns on typescript-eslint's recommended rules, including the type-aware ones that require a `tsconfig.json`. These catch more bugs but are slower to run since they invoke the TypeScript compiler.
- **`parserOptions.project: true`** — tells typescript-eslint to find the nearest `tsconfig.json` automatically. `tsconfigRootDir` anchors the search to your project root.
- **`rules`** — your project-specific overrides. Each rule can be `"error"` (fail the lint check), `"warn"` (show a warning but don't fail), or `"off"` (disabled).
- **`ignores`** — patterns for files ESLint should skip entirely.

### Running ESLint

Add these scripts to your `package.json`:

```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix"
  }
}
```

Then run them:

```bash
# Check all files for lint errors
pnpm lint

# Auto-fix everything that can be fixed automatically
pnpm lint:fix
```

Example output when ESLint finds problems:

```
/home/user/my-app/src/index.ts
  3:1   warning  Unexpected console statement  no-console
  7:14  error    Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any
  12:3  error    This expression is not callable  @typescript-eslint/no-unsafe-call

✖ 3 problems (2 errors, 1 warning)
```

Each line shows the file, line number, column, severity, message, and the rule name. The rule name is useful for looking up documentation or disabling the rule for a specific line.

### Disabling rules for a specific line

Sometimes you genuinely need to suppress a rule. You can do that with a comment:

```typescript
// eslint-disable-next-line @typescript-eslint/no-explicit-any
function legacyHelper(data: any): void {
  // ...
}
```

Use this sparingly. If you find yourself disabling a rule frequently, consider whether the rule should be changed in your config instead.

### Common rules and what they catch

Here are some rules you'll encounter most often:

| Rule | What it catches |
|---|---|
| `no-console` | `console.log()` left in production code |
| `no-unused-vars` | Variables declared but never used |
| `no-undef` | Using a variable that was never declared |
| `@typescript-eslint/no-explicit-any` | Using the `any` type, which defeats type safety |
| `@typescript-eslint/no-floating-promises` | Promises that aren't awaited or `.catch()`-ed |
| `@typescript-eslint/no-unsafe-assignment` | Assigning a value of type `any` to a typed variable |
| `@typescript-eslint/explicit-function-return-type` | Functions missing a return type annotation |
| `@typescript-eslint/prefer-nullish-coalescing` | Using `||` where `??` is safer |

---

## Oxlint — the Fast Rust-Based Linter

[Oxlint](https://oxc.rs/docs/guide/usage/linter) ([GitHub](https://github.com/oxc-project/oxc)) is a newer linter from the [Oxc project](https://oxc.rs) (JavaScript Oxidation Compiler) — a suite of JavaScript/TypeScript tooling written in Rust.

The headline number: Oxlint is **50–100x faster than ESLint**. On a large codebase that takes ESLint 30 seconds to lint, Oxlint might finish in under a second. This speed comes from being written in Rust rather than JavaScript, and from its highly parallel architecture.

### Running Oxlint

Oxlint requires zero configuration to get started. You can run it immediately without even installing it:

```bash
# Run without installing (using pnpm dlx)
pnpm dlx oxlint .
```

Or install it as a dev dependency for consistent use in your project:

```bash
pnpm add -D oxlint
pnpm oxlint .
```

Add it to your `package.json` scripts:

```json
{
  "scripts": {
    "lint:fast": "oxlint ."
  }
}
```

### Example output

```
./src/index.ts
  ⚠ no-unused-vars: 'greeting' is assigned a value but never used.   (4:9)
  ✗ no-debugger: `debugger` statement is not allowed.                 (9:3)

Found 2 problems (1 error, 1 warning) in 1 file. Finished in 12ms.
```

The output is clear and fast. Even on a project with hundreds of files, that "12ms" is realistic.

### Current limitations

Oxlint is impressive, but it's not yet a complete replacement for ESLint:

- **No type-aware rules** — Oxlint does not integrate with the TypeScript compiler. Rules that require type information (like `@typescript-eslint/no-floating-promises`) are not available.
- **Smaller rule set** — Oxlint implements many common rules, but not all ESLint rules or third-party plugin rules are supported yet.
- **No plugin ecosystem** — ESLint has plugins for React, accessibility, import ordering, security, and much more. Oxlint has a growing set of built-in rules but no plugin system yet.
- **Work in progress** — the project is actively developed and improving rapidly, but it's younger and less mature than ESLint.

### When to use Oxlint

| Situation | Recommendation |
|---|---|
| Getting quick feedback while writing code | Oxlint — instant results |
| CI first-pass check before slower checks | Oxlint — catch obvious issues fast |
| Large monorepo where ESLint is slow | Oxlint as a pre-filter |
| Deep TypeScript type-aware analysis | ESLint with typescript-eslint |
| Projects needing React or other plugin rules | ESLint |

---

## Biome — Unified Linter and Formatter

[Biome](https://biomejs.dev) ([GitHub](https://github.com/biomejs/biome)) is a Rust-based tool that combines a linter **and** a formatter in one. Think of it as a single replacement for both ESLint and Prettier. It is fast, requires minimal configuration, and works with JavaScript and TypeScript out of the box.

The linting side of Biome is covered here. Formatting is covered in [Chapter 6](./06-code-formatting.md).

### Installation

```bash
pnpm add -D @biomejs/biome
pnpm biome init    # creates biome.json
```

### Configuration — biome.json

```json
{
  "$schema": "https://biomejs.dev/schemas/1.9.0/schema.json",
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "correctness": {
        "noUnusedVariables": "error",
        "noUnusedImports": "error"
      },
      "suspicious": {
        "noExplicitAny": "error",
        "noDebugger": "error"
      },
      "style": {
        "useConst": "warn",
        "noVar": "error"
      }
    }
  },
  "files": {
    "ignore": ["dist/**", "build/**", "node_modules/**"]
  }
}
```

### Running Biome lint

```bash
# Check for lint errors
pnpm biome lint .

# Check and auto-fix
pnpm biome lint --write .

# Run lint + format + organize imports together (recommended)
pnpm biome check --write .

# CI mode — check without modifying files
pnpm biome check .
```

Add to `package.json` scripts:

```json
{
  "scripts": {
    "lint": "biome lint .",
    "lint:fix": "biome lint --write .",
    "check": "biome check --write .",
    "check:ci": "biome check ."
  }
}
```

### Example output

```
src/index.ts:3:10 lint/correctness/noUnusedImports ━━━━━━━━━━━━━━━━━━
  ✖ This import is never used.

  3 │ import { readFile } from 'fs';
    │          ^^^^^^^^

src/index.ts:7:14 lint/suspicious/noExplicitAny ━━━━━━━━━━━━━━━━━━━━━
  ✖ Unexpected any. Specify a different type.

  7 │ function greet(name: any) {
    │                      ^^^

Found 2 diagnostics in 1 file.
```

Biome's error output is notably readable — it highlights the exact token, shows surrounding context, and links to documentation for each rule.

### Current limitations

- **No type-aware rules** — like Oxlint, Biome does not integrate with the TypeScript compiler. Rules that require type information (such as detecting unhandled promises based on inferred types) are not available.
- **Smaller rule set than ESLint** — Biome implements several hundred rules but does not cover all ESLint plugins (React-specific accessibility rules, for example, are not yet available).
- **No plugin system (yet)** — you cannot write custom Biome rules the way you can write ESLint plugins.

---

## ESLint vs Oxlint vs Biome — Comparison

Here's a side-by-side look at all three tools:

| Feature | ESLint | Oxlint | Biome |
|---|---|---|---|
| **Written in** | JavaScript | Rust | Rust |
| **Speed** | Moderate | 50–100x faster than ESLint | 20–50x faster than ESLint |
| **Includes formatter** | No (use Prettier separately) | No | Yes |
| **Configuration** | Required (`eslint.config.ts`) | Zero-config to start | Minimal (`biome.json`) |
| **TypeScript support** | Excellent (via typescript-eslint) | Basic | Good (no type-aware rules) |
| **Type-aware rules** | Yes | No | No |
| **Plugin ecosystem** | Massive (React, a11y, security…) | None | None |
| **Autofix** | Yes (many rules) | Yes (some rules) | Yes (most rules) |
| **Rule count** | Hundreds (core + plugins) | ~500 built-in | ~300+ built-in |
| **Migrate from Prettier** | N/A | N/A | `biome migrate prettier` |
| **Migrate from ESLint** | N/A | N/A | `biome migrate eslint` |
| **Maturity** | Very mature (2013) | Newer (active) | Newer (active) |
| **Best for** | Deep, type-aware analysis | Fast feedback pass | Unified lint + format, new projects |

## Recommendation

**For a new project wanting simplicity:** Use **Biome**. One tool, one config file, handles both linting and formatting. Fast, minimal setup, and the output is excellent.

**For a project needing deep TypeScript analysis:** Use **ESLint** with typescript-eslint. Type-aware rules like `no-floating-promises` catch real bugs that Biome and Oxlint cannot.

**For a large codebase where CI speed is critical:** Run **Oxlint** as a fast first pass, then ESLint for type-aware analysis. The two complement each other well.

**Don't combine Biome with ESLint+Prettier** on the same project — pick one approach and stick with it.

A practical `package.json` that covers the main scenarios:

```json
{
  "scripts": {
    "lint": "biome lint .",
    "lint:fix": "biome lint --write .",
    "lint:deep": "eslint .",
    "lint:fast": "oxlint .",
    "check": "biome check --write .",
    "check:ci": "biome check ."
  }
}
```

---

## What's Next?

You now know how to catch bugs and enforce code quality automatically with ESLint and Oxlint. Next up is **code formatting** — a related but distinct tool that handles the visual layout of your code so you never have to debate tabs vs. spaces again.

[← Previous: Version Control](./04-version-control.md) | [Back to README](./README.md) | [Next: Code Formatting →](./06-code-formatting.md)
