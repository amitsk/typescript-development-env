[← Previous: Linting](./05-linting.md) | [Back to README](./README.md) | [Next: Unit Testing with Vitest →](./07-testing.md)

# Chapter 6: Code Formatting

Every developer has opinions about code style — tabs vs spaces, single vs double quotes, trailing commas or not. These debates slow down code reviews and create noisy diffs. Code formatters solve this by automatically enforcing a consistent style for the entire team, so nobody has to think about it.

## What is Code Formatting?

A code formatter automatically rewrites your source files to follow a consistent style. This includes:

- **Indentation** — tabs or spaces, and how many
- **Quotes** — single or double quotes for strings
- **Semicolons** — always, never, or only when required
- **Line length** — wrapping long lines at a set width
- **Trailing commas** — after the last item in arrays, objects, function parameters

The key benefit is that formatting is automatic and non-negotiable. Nobody debates style because the formatter decides for everyone.

### Linting vs Formatting

These two tools serve different purposes:

| Tool | Purpose | Example |
|------|---------|---------|
| **Linter** (ESLint) | Finds bugs and code quality issues | Using `==` instead of `===`, unused variables |
| **Formatter** (Prettier) | Enforces consistent visual style | Indentation, quote style, line wrapping |

Linters catch logic errors. Formatters handle appearance. You typically use both together.

### Benefits of a Code Formatter

- **Consistent codebase** — every file looks the same regardless of who wrote it
- **Faster code reviews** — reviewers focus on logic, not style
- **No manual style work** — stop thinking about where to put line breaks
- **Cleaner git diffs** — changes show only meaningful differences, not whitespace noise

---

## Prettier — the Opinionated Formatter

[Prettier](https://prettier.io) ([GitHub](https://github.com/prettier/prettier)) is the most widely used code formatter in the JavaScript ecosystem. "Opinionated" means it provides very few configuration options — Prettier makes most style decisions for you. This is intentional: the goal is to end style debates entirely.

Prettier supports TypeScript, JavaScript, JSON, CSS, HTML, Markdown, and more.

### Installation

```bash
pnpm add -D prettier
```

### Configuration

Create a `.prettierrc` file in your project root:

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "arrowParens": "always"
}
```

Here is what each option does:

- `semi` — add semicolons at the end of statements
- `singleQuote` — use single quotes instead of double quotes
- `tabWidth` — indent with 2 spaces
- `trailingComma: "es5"` — add trailing commas where valid in ES5 (objects, arrays)
- `printWidth` — wrap lines longer than 100 characters
- `arrowParens: "always"` — always include parens around arrow function parameters: `(x) => x`

### Ignoring Files

Create a `.prettierignore` file to tell Prettier which files to skip:

```
dist/
build/
node_modules/
pnpm-lock.yaml
*.min.js
```

### Running Prettier

```bash
pnpm prettier --write .          # format all files in place
pnpm prettier --check .          # check for issues without changing files (for CI)
pnpm prettier --write src/       # format a specific directory
```

Add these to your `package.json` scripts for convenience:

```json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  }
}
```

Now you can run `pnpm format` to format everything, and `pnpm format:check` in CI to verify that all files are already formatted.

### Integrating with ESLint

ESLint has some rules that overlap with what Prettier handles — for example, rules about spacing or quotes. When both tools are active, they can conflict. The `eslint-config-prettier` package disables all ESLint rules that would conflict with Prettier.

Install it:

```bash
pnpm add -D eslint-config-prettier
```

Then add it to your `eslint.config.ts`. It must be last in the config array so it overrides any earlier formatting rules:

```typescript
import prettierConfig from 'eslint-config-prettier';

export default tseslint.config(
  // ... your other configs
  prettierConfig, // must be last — disables formatting rules that conflict with Prettier
);
```

### VSCode Integration

Install the [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) extension, then add these settings to your `.vscode/settings.json`:

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true
}
```

With `formatOnSave` enabled, your file is automatically formatted every time you save it.

---

## Biome — the Unified Linter and Formatter

[Biome](https://biomejs.dev) ([GitHub](https://github.com/biomejs/biome)) is a newer tool that replaces both ESLint and Prettier in a single package. It is written in Rust, which makes it dramatically faster than JavaScript-based tools — often orders of magnitude faster than running ESLint and Prettier separately.

Biome is largely compatible with Prettier's formatting output, so switching is usually painless.

### Installation

```bash
pnpm add -D @biomejs/biome
pnpm biome init   # creates biome.json with a starter config
```

### Configuration

The `biome.json` file controls linting, formatting, and import organization all in one place:

```json
{
  "$schema": "https://biomejs.dev/schemas/1.9.0/schema.json",
  "organizeImports": {
    "enabled": true
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "trailingCommas": "es5",
      "semicolons": "always"
    }
  },
  "files": {
    "ignore": ["dist/**", "build/**", "node_modules/**"]
  }
}
```

### Running Biome

```bash
pnpm biome format --write .      # format files in place
pnpm biome lint .                # lint files
pnpm biome check --write .       # format + lint + organize imports (recommended)
pnpm biome check .               # check without changing files (for CI)
```

The `check` command is the recommended daily driver — it handles everything at once.

Add these to your `package.json` scripts:

```json
{
  "scripts": {
    "lint": "biome lint .",
    "format": "biome format --write .",
    "check": "biome check --write .",
    "check:ci": "biome check ."
  }
}
```

### Migrating from Existing Tools

If you are moving an existing project to Biome, it can automatically import your current configuration:

```bash
pnpm biome migrate prettier --write   # import settings from .prettierrc
pnpm biome migrate eslint --write     # import rules from eslint.config.*
```

### VSCode Integration

Install the [Biome](https://marketplace.visualstudio.com/items?itemName=biomejs.biome) extension, then add these settings to `.vscode/settings.json`:

```json
{
  "editor.defaultFormatter": "biomejs.biome",
  "editor.formatOnSave": true
}
```

### Biome vs ESLint + Prettier

| | Biome | ESLint + Prettier |
|---|---|---|
| **Speed** | Extremely fast (Rust) | Slower (JavaScript) |
| **Configuration** | Single `biome.json` | Separate config files for each tool |
| **Setup** | One package to install | Multiple packages, integration work |
| **Migration** | Built-in migration commands | N/A (it is the existing setup) |
| **Ecosystem** | Smaller, growing | Mature, large plugin ecosystem |
| **Best for** | New projects, speed-sensitive workflows | Teams with existing ESLint configs or specialized plugins |

---

## oxfmt (Oxc Formatter)

The [Oxc project](https://oxc.rs) (JavaScript Oxidation Compiler) is building a complete suite of Rust-based JavaScript tooling: a parser, linter, formatter, and bundler. The formatter, oxfmt, is part of this effort.

As of 2025, oxfmt is in very early development and is not yet production-ready. However, the Oxc project as a whole is worth watching — it aims to be one of the fastest JavaScript toolchains available once complete.

Check https://oxc.rs for the latest status before considering it for a project.

---

## Recommendation

- **Use Prettier** if your team already has ESLint configured, or if you need plugins from the broader ESLint ecosystem. It is the safest, most widely supported choice.
- **Use Biome** for new projects where you want simplicity and maximum speed. One tool, one config file, instead of two.
- **Do not mix approaches** — do not run ESLint+Prettier alongside Biome on the same project. Pick one and stick with it.

---

## What's Next?

Your code is now consistently formatted. The next step is making sure it actually works correctly. In the next chapter, we will set up Vitest to write and run automated tests for your TypeScript code.

---

[← Previous: Linting](./05-linting.md) | [Back to README](./README.md) | [Next: Unit Testing with Vitest →](./07-testing.md)
