# Chapter 6: Code Formatting

[← Previous: Linting](./05-linting.md) | [Back to README](./README.md) | [Next: Unit Testing with Vitest →](./07-testing.md)

---

Nothing derails a pull request faster than two developers arguing over single vs. double quotes or tabs vs. spaces. A **code formatter** ends the debate by automatically rewriting your files into one consistent style every time you hit Save.

- **Linters (Chapter 5)** care about *logic and bugs* (`===` vs `==`, unused variables).
- **Formatters (this chapter)** care about *visual layout* (indentation, quotes, line width, semicolons).

```mermaid
flowchart LR
    Save["1. Save File in Editor (formatOnSave)"] --> Hook["2. Pre-Commit Hook (Format Staged Files)"]
    Hook --> CI["3. CI Pipeline (biome check / prettier --check)"]
    CI --> Clean["Consistent Codebase & Noise-Free Git Diffs"]
```

---

## Option 1: Prettier — The Industry Classic

[Prettier](https://prettier.io) ([GitHub](https://github.com/prettier/prettier)) is the battle-tested formatter used across millions of repos. It formats TypeScript, JavaScript, JSON, CSS, HTML, and Markdown.

```bash
pnpm add -D prettier
```

Create `.prettierrc`:

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100
}
```

And `.prettierignore`:

```text
dist/
node_modules/
pnpm-lock.yaml
```

Run `pnpm prettier --write .` to format your files, or `pnpm prettier --check .` in CI.

> **Using Prettier with ESLint?** Install `pnpm add -D eslint-config-prettier` and place it last in `eslint.config.ts` so ESLint doesn't yell about formatting rules Prettier owns.

---

## Option 2: Biome — The Fast All-in-One

As introduced in Chapter 5, [Biome](https://biomejs.dev) formats code at Rust speed while matching Prettier's style nearly identically — without needing a separate tool.

```json
{
  "$schema": "https://biomejs.dev/schemas/2.0.0/schema.json",
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "semicolons": "always"
    }
  }
}
```

Run `pnpm biome format --write .` (or `pnpm biome check --write .` to lint + format together). Already have Prettier? Migrate in one command with `pnpm biome migrate prettier --write`.

### Turn on Format-on-Save in VSCode

Add this to `.vscode/settings.json` (use `"esbenp.prettier-vscode"` if using Prettier):

```json
{
  "editor.defaultFormatter": "biomejs.biome",
  "editor.formatOnSave": true
}
```

Write messy code on one line, hit `Ctrl+S` / `Cmd+S`, and watch it snap into place.

---

## Option 3: oxfmt (Oxc Formatter)

The [Oxc project](https://oxc.rs) also ships **[oxfmt](https://oxc.rs/docs/guide/usage/formatter)** (beta in 2026) — a Rust formatter built for drop-in Prettier compatibility at ~30x Prettier's speed. It's a natural companion if you already use Oxlint.

---

## Recommendations

- **Use Biome** for new projects where you want one fast tool and one config file for both linting and formatting.
- **Use Prettier** if your project uses ESLint or relies on Prettier plugins (like Tailwind class sorting) and broad file-type coverage (Markdown, YAML, HTML).
- **Try oxfmt** when pairing with Oxlint and you want Prettier-compatible output at Rust speeds.
- **Never mix two formatters** on the same files — they will passive-aggressively reformat each other's output forever.

---

## What's Next?

Your code is clean and linted — now let's prove it actually works in [Chapter 7: Unit Testing with Vitest](./07-testing.md).

[← Previous: Linting](./05-linting.md) | [Back to README](./README.md) | [Next: Unit Testing with Vitest →](./07-testing.md)
