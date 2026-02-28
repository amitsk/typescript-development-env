[← Previous: Unit Testing with Vitest](./07-testing.md) | [Back to README](./README.md) | [Next: REST APIs with Fastify →](./09-fastify.md)

# Chapter 8: Building with Vite

TypeScript is a great language for writing code, but browsers and Node.js cannot run it directly. Before your code can be deployed, it needs to be compiled to JavaScript. A build tool handles this — and a lot more.

## What Does a Build Tool Do?

A build tool takes your TypeScript source files and transforms them into something ready to ship. Here is what typically happens during a build:

- **TypeScript compilation** — strips type annotations and converts TypeScript syntax to JavaScript
- **Bundling** — combines many module files into fewer files, reducing the number of network requests a browser has to make
- **Tree-shaking** — analyzes which code is actually used and removes everything else, shrinking the final bundle
- **Asset optimization** — minifies JavaScript, optimizes images, and hashes filenames for cache busting

### A Brief History

The JavaScript ecosystem has gone through several generations of build tools:

- **Webpack** — the dominant tool for years. Extremely powerful, but notoriously slow and complex to configure. Still widely used in large, established projects.
- **Parcel** — aimed to be zero-config. Easier than Webpack but less flexible.
- **Rollup** — focused on libraries and ES modules. Excellent output quality but primarily aimed at library authors.
- **Vite** — the modern standard for new projects. Fast, opinionated, and excellent developer experience out of the box.

---

## Vite — the Next-Generation Build Tool

[Vite](https://vite.dev) ([GitHub](https://github.com/vitejs/vite)) was created by Evan You (the author of Vue.js) and has quickly become the default choice for new TypeScript and JavaScript projects. The name means "fast" in French, and that speed is its defining feature.

### Two Modes

Vite works differently in development and production:

**Development mode** uses the browser's native ES module support. Instead of bundling your files together, Vite serves each module file directly to the browser. This means:
- Instant server start — no bundling step required
- Hot Module Replacement (HMR) — the browser updates changed modules in place, without a full page reload
- Changes appear in the browser in milliseconds

**Production mode** uses [Rollup](https://rollupjs.org) under the hood to produce highly optimized bundles. You get the best developer experience during development and the best output quality for production.

### TypeScript and esbuild

Vite uses [esbuild](https://esbuild.github.io) (written in Go) to transpile TypeScript. esbuild is 10–100x faster than running `tsc` alone.

There is an important caveat: **Vite transpiles TypeScript but does not type-check it.** esbuild simply strips the type annotations — it does not verify that the types are correct. To catch type errors, you still need to run `tsc --noEmit` separately. We will cover how to add this to your build workflow later in this chapter.

---

## Scaffolding a New Project with Vite

The fastest way to start a Vite project is with the `create-vite` scaffolding tool. It sets up the project structure, config files, and `package.json` for you:

```bash
pnpm create vite my-app
# Choose: TypeScript, React / Vue / Vanilla, etc.
cd my-app
pnpm install
pnpm run dev
```

When you run `pnpm create vite`, it prompts you to choose a framework (React, Vue, Svelte, Vanilla, etc.) and a variant (TypeScript or JavaScript). Choose TypeScript.

### Generated Project Structure

After scaffolding, your project looks like this:

```
my-app/
├── public/              # static assets — copied to dist/ as-is, not processed
├── src/
│   ├── main.ts          # entry point — Vite starts here
│   └── vite-env.d.ts    # Vite type declarations (for import.meta.env, etc.)
├── index.html           # HTML entry point — Vite reads this, not public/
├── tsconfig.json        # TypeScript config for your source code
├── tsconfig.node.json   # TypeScript config for vite.config.ts itself
├── vite.config.ts       # Vite configuration
└── package.json
```

A few things worth noting:

- `index.html` lives in the project root, not in `public/`. Vite treats it as the entry point and processes it directly — it resolves `<script type="module" src="/src/main.ts">` by compiling `main.ts`.
- `public/` is for truly static files (like `favicon.ico` or `robots.txt`) that should be served unchanged.
- `vite-env.d.ts` contains type declarations for Vite-specific features like `import.meta.env` and `import.meta.hot`.

---

## vite.config.ts — Configuration

Vite is configured through `vite.config.ts`. Here is a fully annotated example for a React project:

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

export default defineConfig({
  plugins: [
    react(), // enables React Fast Refresh (HMR for React components)
  ],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'), // lets you write: import Foo from '@/components/Foo'
    },
  },
  build: {
    outDir: 'dist',       // where to put the production build
    sourcemap: true,      // include source maps so you can debug minified code
    minify: 'esbuild',    // use esbuild for minification (fastest option)
  },
  server: {
    port: 3000,  // dev server port (default is 5173)
    open: true,  // automatically open the browser when dev server starts
  },
});
```

`defineConfig` is a helper that provides TypeScript autocompletion for the config object. It does not change any behavior — it just gives you type safety when editing the config.

---

## Development Workflow

Start the development server with:

```bash
pnpm run dev
```

Vite starts a local server (typically at `http://localhost:5173`) and serves your app. The terminal output looks like:

```
  VITE v5.0.0  ready in 243 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

### Hot Module Replacement (HMR)

HMR is one of Vite's most useful features. When you save a file, Vite pushes only the changed module to the browser. React components re-render with the new code while preserving their state. CSS updates instantly without any reload at all.

This is very different from a traditional "full page reload" approach. With HMR, you can be editing a form deep in your app and see style changes apply in real time without losing the data you typed into the form.

---

## Production Build

When you are ready to deploy, run the build command:

```bash
pnpm run build    # compile + bundle everything into dist/
pnpm run preview  # serve the dist/ folder locally so you can test it
```

The `preview` command is useful for catching issues that only appear in the production build — for example, paths that work in development but break when served from a CDN.

### What Gets Generated in `dist/`

After a build, the `dist/` folder contains:

- **Hashed filenames** — `assets/index-Bx3Qk2rA.js` instead of `index.js`. The hash changes only when the file content changes, so browsers can cache files aggressively. Users only re-download what actually changed.
- **Minified JavaScript** — whitespace removed, variable names shortened, dead code eliminated
- **Optimized assets** — images may be inlined as base64 if they are small, larger ones are copied with hashed names

---

## TypeScript Type Checking with Vite

Because Vite skips type checking during builds, you need to run `tsc` separately to catch type errors. Add this to your workflow:

```bash
pnpm tsc --noEmit     # type-check without emitting any output files
```

The `--noEmit` flag tells TypeScript to check for errors but not write any `.js` files. This is fast and gives you the full type-checking power of `tsc`.

Update your `package.json` scripts to run type checking before every production build:

```json
{
  "scripts": {
    "build": "tsc --noEmit && vite build",
    "typecheck": "tsc --noEmit"
  }
}
```

Now `pnpm run build` will first verify that all types are correct, then proceed with the Vite bundle. If `tsc` finds an error, the build stops immediately before producing broken output.

---

## Vite Plugins

Vite's functionality is extended through plugins. Some commonly used ones:

| Plugin | Purpose |
|--------|---------|
| `@vitejs/plugin-react` | React support with Fast Refresh |
| `@vitejs/plugin-vue` | Vue 3 support with HMR |
| `vite-plugin-pwa` | Progressive Web App support (service workers, manifest) |

Install a plugin, import it in `vite.config.ts`, and add it to the `plugins` array. For example, to add React support:

```bash
pnpm add -D @vitejs/plugin-react
```

```typescript
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
});
```

For the full list of official and community plugins, see the [Vite plugin directory](https://vitejs.dev/plugins/).

---

## Quick Reference

```bash
pnpm create vite          # scaffold a new Vite project
pnpm run dev              # start the development server with HMR
pnpm run build            # production build — output goes to dist/
pnpm run preview          # serve dist/ locally to test the production build
pnpm tsc --noEmit         # type check without building
```

---

## What's Next?

You now know how to build front-end TypeScript projects with Vite. In the next chapter, we will shift to the back end and build a REST API using Fastify — a fast, TypeScript-friendly Node.js web framework.

---

[← Previous: Unit Testing with Vitest](./07-testing.md) | [Back to README](./README.md) | [Next: REST APIs with Fastify →](./09-fastify.md)
