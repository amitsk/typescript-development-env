# Chapter 1: Introduction

[Back to README](./README.md) | [Next: Node.js Environments →](./02-node-environments.md)

---

## Background

Before we dive in, let's be clear about what this guide is and what it isn't.

**This is not a TypeScript or JavaScript tutorial.** We won't be teaching you how the language works, what variables are, how functions work, or how to use the standard library. There are already excellent resources for that:

- [TypeScript Handbook](https://www.typescriptlang.org/docs/) — the official, comprehensive guide to TypeScript, maintained by the TypeScript team at Microsoft
- [MDN JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide) — Mozilla's in-depth JavaScript reference, trusted by developers worldwide
- [The Modern JavaScript Tutorial](https://javascript.info) — a well-written, free, and thorough guide to JavaScript from the ground up

**What this guide IS:** a walkthrough of the professional tools, workflows, and practices that TypeScript developers use every day. Things like: how do you manage different versions of Node.js on your machine? How do you automatically catch bugs before you run your code? How do you make sure your whole team formats code the same way? How do you run automated tests every time you push to GitHub?

These are the questions this guide answers. If you're learning TypeScript on the side while reading this, great — you can use the resources above for the language itself, and use this guide to understand the ecosystem around it.

---

## What Is TypeScript, and How Does It Differ from JavaScript?

Even though this isn't a language tutorial, it's worth understanding the fundamental relationship between TypeScript and JavaScript, because it explains a lot about how the tooling works.

### TypeScript Is a Superset of JavaScript

TypeScript was created by Microsoft and first released in 2012. The core idea is simple: TypeScript is JavaScript with added syntax for **static types**.

That means **every valid JavaScript file is also a valid TypeScript file.** You can rename `app.js` to `app.ts` and TypeScript will accept it. From there, you can gradually add type annotations as you see fit.

### What TypeScript Adds

TypeScript extends JavaScript with:

- **Static types** — you declare what type a variable, parameter, or return value should be
- **Interfaces** — define the shape of objects
- **Enums** — named sets of constants
- **Generics** — reusable type-safe abstractions
- **Access modifiers** — `public`, `private`, `protected` on class members

### TypeScript Compiles to JavaScript

Here's something crucial: browsers and Node.js do not run TypeScript. They run JavaScript.

TypeScript has a compiler (called `tsc`) that reads your `.ts` files and outputs `.js` files. This process is sometimes called **transpilation** because it's compiling from one high-level language to another, rather than down to machine code.

Your workflow looks like this:

```
TypeScript source (.ts) → tsc (TypeScript compiler) → JavaScript output (.js) → Browser or Node.js
```

In practice, most modern tools (like Vite, which we cover in Chapter 8) handle this compilation step for you automatically, so you rarely need to invoke `tsc` directly in day-to-day work.

### Why Bother with Types?

Let's look at a concrete example. Here's a simple function in JavaScript:

```javascript
// JavaScript
function add(a, b) {
  return a + b;
}

add("1", 2); // No error! Returns "12" (string concatenation)
```

JavaScript happily runs this code. The `+` operator, when used with a string and a number, converts the number to a string and concatenates them. You get `"12"` instead of `3`. This is a real, common bug — and JavaScript gives you no warning at all.

Now here's the same function in TypeScript:

```typescript
// TypeScript
function add(a: number, b: number): number {
  return a + b;
}

add("1", 2); // Error: Argument of type 'string' is not assignable to parameter of type 'number'
```

TypeScript catches the mistake before you ever run the code. The type annotations (`a: number`, `b: number`, `: number` after the parentheses) tell TypeScript what types are expected, and it enforces them at **compile time** — meaning in your editor or build step, not at runtime when a real user is affected.

The benefits of this approach compound as your codebase grows:

- **Catch errors early** — find bugs when you write the code, not when users hit them in production
- **Better IDE autocomplete** — your editor knows the shape of every object, so it can suggest properties and methods accurately
- **Self-documenting code** — type annotations tell the next developer (or future you) exactly what a function expects and returns
- **Safer refactoring** — when you rename a field or change a function signature, TypeScript tells you everywhere you need to update

### The TypeScript Config: tsconfig.json

Every TypeScript project has a `tsconfig.json` file in its root directory. This file tells the TypeScript compiler how to behave: which files to compile, what JavaScript version to target, how strict to be about type checking, and where to put the output.

Here's a simple example:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Let's walk through the key options:

- `"target": "ES2022"` — compile to JavaScript syntax compatible with ES2022. Modern Node.js and browsers support this.
- `"module": "ESNext"` — use ES module syntax (`import`/`export`) in the output
- `"moduleResolution": "Bundler"` — use module resolution rules suitable for bundlers like Vite
- `"strict": true` — enable all strict type-checking options. Highly recommended — it catches the most bugs.
- `"outDir": "./dist"` — put compiled `.js` files in the `dist/` folder
- `"include"` — which files to compile (here: everything in the `src/` directory)
- `"exclude"` — which files to skip (always exclude `node_modules` and your output directory)

You don't need to memorize this yet — we'll revisit `tsconfig.json` in context as we set up each project type throughout the guide.

---

## Software Engineering Best Practices

TypeScript itself is just a language. The real power of a professional development setup comes from the practices and tools that surround it. Here's a quick preview of the best practices we'll cover:

### Version Control

Version control (using Git) lets you track every change you make to your code, collaborate with teammates, and roll back mistakes. It's the single most important professional practice to adopt. We cover it in [Chapter 4: Version Control](./04-version-control.md).

### Linting

A linter analyzes your code for potential bugs and style issues without running it. It catches things like using a variable before declaring it, forgetting to handle a promise rejection, or using a deprecated API. We cover ESLint in [Chapter 5: Linting](./05-linting.md).

### Code Formatting

A code formatter automatically rewrites your code to follow consistent style rules — indentation, quote style, line length, and so on. This eliminates arguments about style in code reviews and keeps the entire codebase consistent. We cover formatters in [Chapter 6: Code Formatting](./06-code-formatting.md).

### Unit Testing

Tests are automated checks that verify your code does what you expect. Writing tests means you can refactor or add features with confidence, knowing your tests will catch any regressions. We cover Vitest in [Chapter 7: Unit Testing with Vitest](./07-testing.md).

### CI/CD

Continuous Integration / Continuous Deployment automates your testing, linting, and deployment pipeline. Every time you push code, a server runs your tests and checks automatically. We cover this in [Chapter 12: CI/CD](./12-ci-cd.md).

---

## The TypeScript Ecosystem: Tools We'll Use

The JavaScript and TypeScript ecosystem is large. Here are the main tools we'll focus on throughout this guide:

### Build Tools

**[Vite](https://vite.dev)** is a modern build tool that compiles TypeScript, bundles your code for production, and provides a fast development server with hot module replacement. It's become the standard choice for TypeScript frontend and library projects. We cover it in Chapter 8.

### Testing

**[Vitest](https://vitest.dev)** is a testing framework designed to work seamlessly with Vite. It supports TypeScript out of the box, runs tests extremely fast, and has an API very similar to Jest (so it's easy to learn if you've seen Jest before). We cover it in Chapter 7.

### Linting

**[ESLint](https://eslint.org)** is the dominant linting tool in the JavaScript/TypeScript ecosystem. It has a huge plugin ecosystem and supports TypeScript through official plugins. We cover it in Chapter 5.

### Formatting

**[Biome](https://biomejs.dev)** is a fast, all-in-one linter and formatter written in Rust. It's a compelling alternative to running ESLint and Prettier separately — it does both jobs, faster, with simpler configuration. We cover it in Chapter 6.

### Backend / APIs

**[Fastify](https://fastify.dev)** is a high-performance web framework for building REST APIs with Node.js. It has excellent TypeScript support, a great plugin system, and is significantly faster than Express in benchmarks. We cover it in Chapter 9.

### Full-Stack

**[TanStack Start](https://tanstack.com/start)** is a full-stack TypeScript framework built on TanStack Router. It lets you write type-safe server functions and client code in the same project, with great TypeScript integration throughout. We cover it in Chapter 10.

---

## VSCode as Your Editor

This guide assumes you're using **Visual Studio Code** (VSCode) as your editor. It's free, open source, runs on macOS, Linux, and Windows, and has exceptional TypeScript support built in.

- GitHub: [https://github.com/microsoft/vscode](https://github.com/microsoft/vscode)
- Download: [https://code.visualstudio.com](https://code.visualstudio.com)

### Built-In TypeScript Support

VSCode ships with a TypeScript language server built in — you don't need any extension to get TypeScript type checking, autocomplete, and error highlighting. This is one of the biggest advantages of TypeScript: the editor experience is outstanding with zero configuration.

### Recommended Extensions

While the core TypeScript experience needs no extensions, these are worth installing as you progress through the guide:

| Extension | Purpose |
|---|---|
| **ESLint** (`dbaeumer.vscode-eslint`) | Shows ESLint errors and warnings inline as you type |
| **Prettier** (`esbenp.prettier-vscode`) | Formats on save using Prettier |
| **Biome** (`biomejs.biome`) | Formats and lints on save using Biome |
| **Error Lens** (`usernamehw.errorlens`) | Shows error messages inline next to the offending line |
| **GitLens** (`eamodio.gitlens`) | Supercharges the built-in Git support with blame, history, and more |

You can install extensions from the Extensions panel in VSCode (the square icon in the sidebar, or `Ctrl+Shift+X` / `Cmd+Shift+X`).

### A Note on Other Editors

Everything in this guide except the extension list applies regardless of your editor. JetBrains IDEs (WebStorm, IntelliJ), Neovim with LSP plugins, and other editors all work with TypeScript, ESLint, and the other tools we'll cover. The configurations are editor-agnostic. VSCode is just the most common choice in the TypeScript community and the one this guide uses for screenshots and specific instructions.

---

## What's Next?

You now have a conceptual foundation for what TypeScript is, why the tooling ecosystem exists, and what this guide will cover. The next step is getting your environment ready.

In Chapter 2, we'll install Node.js — the runtime that powers TypeScript tooling — and learn how to manage multiple Node.js versions on the same machine.

[Back to README](./README.md) | [Next: Node.js Environments →](./02-node-environments.md)
