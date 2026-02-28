
* Generate a tutorial on setting up a TypeScript development environment using
the details in this file. The audience for the tutorial are students or folks learning programming with some experience in writing code, and who may already have some JavaScript background. Create this as a Github project with a README that has a table of contents and individual chapters. Create navigation links in each document.
* Tutorial on setting up a TypeScript Development environment
** Topics
*** Introduction
    **** Background. Not a TypeScript or JavaScript tutorial. Provide links to popular TypeScript and JavaScript tutorials including the official TypeScript handbook at https://www.typescriptlang.org/docs/ and the MDN JavaScript guide.
    **** Explain what TypeScript is and how it differs from JavaScript.
        ***** TypeScript is a superset of JavaScript - all valid JS is valid TS
        ***** TypeScript adds static types, interfaces, enums, generics
        ***** TypeScript compiles (transpiles) to JavaScript - browsers and Node run JS
        ***** Explain the benefits: catch errors at compile time, better IDE support, self-documenting code, safer refactoring
        ***** Show a side-by-side example: the same function in JS vs TS with type annotations
        ***** Explain tsconfig.json briefly - the TypeScript compiler configuration file
    **** Explain basic software engineering best practices like Version control, Linting, Code formatting, Unit Testing, CI/CD
    **** Set the context about the TypeScript ecosystem. Mention that modern tooling (Vite, Vitest, Biome, Oxlint) will be used. Links to each tool.
    **** Explain how VSCode will be used. Point to VSCode github link https://github.com/microsoft/vscode. Mention TypeScript and ESLint extensions.

*** Node.js Environments
    **** Explain what Node.js is. Why it is needed to run TypeScript/JavaScript outside the browser. Point to the Node.js website https://nodejs.org.
    **** Explain what LTS (Long Term Support) releases are. Why you should prefer LTS versions for production and learning. Show the Node.js LTS release schedule.
    **** Explain the problem: different projects need different Node.js versions, just like Python virtual environments solve package conflicts.
    **** nvm (Node Version Manager)
        ***** Point to nvm github link https://github.com/nvm-sh/nvm
        ***** Installation for macOS/Linux and Windows (nvm-windows)
        ***** How to install a specific Node version: nvm install 20 (LTS)
        ***** How to switch versions: nvm use 20
        ***** How to set a default: nvm alias default 20
        ***** Using .nvmrc file to pin the Node version for a project
    **** volta
        ***** Point to Volta website https://volta.sh and github link https://github.com/volta-cli/volta
        ***** Volta pins Node and package manager versions per project automatically
        ***** Installation instructions for all platforms
        ***** volta install node@20
        ***** volta pin node@20 (adds to package.json, automatic for all team members)
        ***** Why Volta is a good choice: fast, automatic, no manual activation needed
    **** mise (formerly rtx)
        ***** Point to mise website https://mise.jdx.dev and github link https://github.com/jdx/mise
        ***** Mise is a polyglot version manager - manages Node, Python, Ruby, and more with one tool
        ***** Installation instructions
        ***** mise use node@lts
        ***** .mise.toml or .tool-versions file for project pinning
        ***** Good choice for developers who work across multiple language ecosystems
    **** Recommendation summary: which tool to choose and why

*** Package Management and package.json
    **** Explain what package.json is. It is the manifest for a Node.js project. Covers name, version, description, scripts, dependencies, devDependencies, engines.
    **** Walk through each key section of package.json with annotated examples:
        ***** name, version, description, author, license
        ***** scripts: how npm run, yarn, pnpm scripts work. Common scripts: build, dev, test, lint, format, typecheck
        ***** dependencies vs devDependencies - what belongs where
        ***** engines: pinning the required Node.js version
        ***** type: "module" - ES modules vs CommonJS, when to use each
    **** Explain npm (Node Package Manager). Basic commands: npm install, npm run, npm ci, npx.
    **** Explain pnpm as a faster, disk-efficient alternative. Point to https://pnpm.io. Show pnpm install, pnpm run, pnpm dlx.
    **** Explain package-lock.json and pnpm-lock.yaml - why lock files matter for reproducibility.
    **** Explain node_modules and why it should be in .gitignore.
    **** Explain npx and pnpm dlx for running tools without installing them globally.

*** Version Control
    **** Explain what is version control. Benefits of version control.
    **** Very short explainer about git. Point to git github link https://github.com/git/git
    **** Show a .gitignore for TypeScript/Node.js projects. Cover node_modules, dist, .env, build artifacts, editor files.
    **** Explain GitHub. Creating a repository, pushing code.

*** Linting
    **** Explain what is linting. Benefits of linting in a TypeScript/JavaScript context.
    **** ESLint
        ***** ESLint is the standard linter for JavaScript and TypeScript. Point to https://eslint.org and github link https://github.com/eslint/eslint
        ***** Explain typescript-eslint for TypeScript-specific rules. Point to https://typescript-eslint.io
        ***** Flat config format (eslint.config.js/ts) - the modern way to configure ESLint
        ***** Running ESLint: npx eslint . or pnpm dlx eslint .
        ***** Show an example eslint.config.ts for a TypeScript project
        ***** Common rules and what they catch
    **** Oxlint
        ***** Oxlint is a blazing-fast JavaScript/TypeScript linter written in Rust. Point to https://oxc.rs/docs/guide/usage/linter and github link https://github.com/oxc-project/oxc
        ***** Part of the Oxc (JavaScript Oxidation Compiler) project
        ***** Up to 50-100x faster than ESLint
        ***** Show how to run: npx oxlint or pnpm dlx oxlint
        ***** Current limitations: not a full ESLint replacement yet, great as a companion or first pass
        ***** When to use Oxlint: large codebases, CI speed, alongside ESLint
    **** Recommendation: ESLint for full configurability, Oxlint for speed

*** Code Formatting
    **** Explain what is code formatting. Benefits in a collaborative TypeScript/JavaScript project.
    **** Prettier
        ***** Prettier is the most widely used JavaScript/TypeScript formatter. Point to https://prettier.io and github link https://github.com/prettier/prettier
        ***** Opinionated: minimal configuration, just works
        ***** Show .prettierrc configuration
        ***** Running prettier: npx prettier --write . or pnpm dlx prettier --write .
        ***** Integrating Prettier with ESLint using eslint-config-prettier to avoid conflicts
    **** Biome
        ***** Biome is a fast, unified linter and formatter for JavaScript/TypeScript written in Rust. Point to https://biomejs.dev and github link https://github.com/biomejs/biome
        ***** Replaces both ESLint and Prettier in one tool
        ***** Show biome.json configuration
        ***** Running biome: npx @biomejs/biome format --write . and npx @biomejs/biome lint .
        ***** npx @biomejs/biome check --write . to run both lint and format together
        ***** Comparison: Biome is much faster than the ESLint+Prettier combination
    **** oxfmt (Oxc Formatter)
        ***** Part of the Oxc project. Point to https://oxc.rs
        ***** Very early stage - mention it as an upcoming formatter in the Rust-based JS tooling ecosystem
        ***** When stable it will offer exceptional speed
    **** Recommendation: Prettier for maximum ecosystem compatibility, Biome for speed and simplicity in new projects

*** Unit Testing with Vitest
    **** Explain what is unit testing. Benefits of unit testing in TypeScript projects.
    **** Overview of testing in the JavaScript ecosystem: Jest (popular but older), Vitest (modern, fast).
    **** Vitest
        ***** Point to https://vitest.dev and github link https://github.com/vitest-dev/vitest
        ***** Vitest is the recommended test runner for Vite-based projects. Works great without Vite too.
        ***** Compatible with Jest's API - easy migration from Jest
        ***** Native TypeScript support - no babel or ts-jest needed
        ***** Installation: pnpm add -D vitest
        ***** Configuring vitest in vite.config.ts or vitest.config.ts
        ***** Writing tests: describe, it/test, expect, beforeEach, afterEach
        ***** Show a full example test file for a TypeScript module
        ***** Running tests: pnpm run test or pnpm vitest
        ***** Watch mode: pnpm vitest --watch
    **** Code Coverage with Vitest
        ***** Explain what code coverage is. Why it matters.
        ***** Setting up coverage with @vitest/coverage-v8 or @vitest/coverage-istanbul
        ***** pnpm add -D @vitest/coverage-v8
        ***** Configure coverage in vitest.config.ts: thresholds, reporters, include/exclude
        ***** Running coverage: pnpm vitest run --coverage
        ***** Reading the coverage report - html report and terminal summary
        ***** Setting coverage thresholds (e.g., 80% lines) and failing CI if not met

*** Building with Vite
    **** Explain what a build tool does in the TypeScript/JavaScript world. Why you need to compile/bundle.
    **** Vite
        ***** Point to https://vite.dev and github link https://github.com/vitejs/vite
        ***** Vite is a next-generation build tool that is extremely fast using native ES modules in development
        ***** esbuild for development speed, Rollup for production bundles
        ***** Scaffolding a new project: pnpm create vite
        ***** vite.config.ts configuration: plugins, resolve, build options
        ***** Development server: pnpm run dev (hot module replacement)
        ***** Production build: pnpm run build
        ***** Preview production build: pnpm run preview
        ***** TypeScript support: Vite handles TS natively (transpile only, no type checking)
        ***** Note: use tsc --noEmit or a type checker separately for type errors
        ***** Vite plugins: @vitejs/plugin-react for React, etc.

*** REST APIs with Fastify
    **** Explain what a REST API is and why you might build one in TypeScript.
    **** Fastify
        ***** Point to https://fastify.dev and github link https://github.com/fastify/fastify
        ***** Fastify is one of the fastest Node.js web frameworks with excellent TypeScript support
        ***** Benchmarks vs Express - Fastify handles significantly more requests per second
        ***** Installation: pnpm add fastify
        ***** Creating a basic Fastify server in TypeScript
        ***** Route definitions with type-safe request/reply using TypeScript generics
        ***** JSON Schema validation with TypeBox or Zod for request validation
        ***** Fastify plugins: @fastify/cors, @fastify/jwt, @fastify/swagger
        ***** Organizing routes with plugin architecture
        ***** Show a complete working example: a REST API with GET and POST routes, validation, and TypeScript types
        ***** Starting the server and testing with curl or httpie
        ***** Brief mention of other frameworks: Express (ubiquitous), Hono (ultralight, edge-friendly), Elysia (Bun-first)

*** Full-Stack with TanStack Start
    **** Explain what a full-stack framework provides: server-side rendering, routing, data fetching, API routes in one project.
    **** TanStack Start
        ***** Point to https://tanstack.com/start and github link https://github.com/TanStack/router
        ***** TanStack Start is a full-stack TypeScript framework built on TanStack Router with Vite
        ***** File-based routing with type-safe routes
        ***** Server functions (RPC-style) with full type safety from client to server
        ***** Streaming and suspense support
        ***** Works with React (TanStack is React-first but framework-agnostic planned)
        ***** Scaffolding: pnpm create tsrouter-app
        ***** Project structure walkthrough
        ***** Creating a route, fetching data server-side, rendering on the client
        ***** When to choose TanStack Start vs a pure REST API backend (Fastify) + separate frontend

*** Database Libraries
    **** Explain the landscape of database access in Node.js: raw SQL drivers, query builders, ORMs.
    **** Drizzle ORM
        ***** Point to https://orm.drizzle.team and github link https://github.com/drizzle-team/drizzle-orm
        ***** Drizzle is a lightweight TypeScript ORM that feels close to SQL
        ***** Schema defined in TypeScript - the schema is the source of truth
        ***** Works with PostgreSQL, MySQL, SQLite, and more
        ***** Type-safe queries: select, insert, update, delete
        ***** Drizzle Kit for migrations: drizzle-kit generate, drizzle-kit migrate
        ***** Show a complete example: define a schema, connect to a database, run queries
        ***** No magic - easy to understand what SQL is being generated
    **** Prisma
        ***** Point to https://www.prisma.io and github link https://github.com/prisma/prisma
        ***** Prisma is the most popular Node.js ORM. Schema-first using the Prisma Schema Language (.prisma files)
        ***** Prisma Client: auto-generated, fully type-safe database client
        ***** Prisma Migrate for schema migrations
        ***** Prisma Studio: visual database browser
        ***** Installation: pnpm add prisma @prisma/client
        ***** Initializing: npx prisma init
        ***** Defining a schema and running: npx prisma generate and npx prisma migrate dev
        ***** Show a complete example with a User model and basic CRUD
        ***** When to choose Prisma: rapid development, teams who prefer a schema-first workflow
    **** Kysely
        ***** Point to https://kysely.dev and github link https://github.com/kysely-org/kysely
        ***** Kysely is a type-safe SQL query builder - NOT an ORM
        ***** Write SQL-like queries with full TypeScript type safety
        ***** No magic migrations, no abstracted schema - you are in full control of SQL
        ***** Works with pg (PostgreSQL), mysql2, better-sqlite3
        ***** Great for developers who want SQL control with TypeScript safety
        ***** Show an example: defining the DB type, creating the Kysely instance, running a typed query
        ***** When to choose Kysely: complex queries, performance-critical apps, SQL experts
    **** Comparison table: Drizzle vs Prisma vs Kysely - abstraction level, migration support, query style, TypeScript integration

*** CI/CD
    **** Explain what is CI/CD. Benefits of CI/CD.
    **** Very short explainer about GitHub Actions. Point to https://github.com/features/actions
    **** Show a complete .github/workflows/ci.yml for a TypeScript project that:
        ***** Installs Node.js (with LTS matrix)
        ***** Installs pnpm
        ***** Runs type checking (tsc --noEmit)
        ***** Runs linting (eslint or biome)
        ***** Runs formatting check
        ***** Runs tests with coverage (vitest)
        ***** Builds the project (vite build)

*** LLMs and AI-Powered Development
    **** Explain what LLMs are. Benefits of using LLMs in TypeScript/JavaScript projects.
    **** Very short explainer about Cursor. Point to Cursor github link https://github.com/cursor-ai/cursor
    **** Short explainer about Claude Code. Point to Claude Code github link https://github.com/anthropics/claude-code
    **** Short explainer about GitHub Copilot. Point to https://github.com/features/copilot

*** Putting It All Together
    **** Walk through creating a complete TypeScript project from scratch using all the tools covered.
    **** Create a new project with Vite (or TanStack Start), configure TypeScript, set up ESLint+Prettier (or Biome), add Vitest with coverage, connect to a database with Drizzle, add a Fastify API layer.
    **** Show the complete package.json scripts section that ties everything together.
    **** Short explainer about project templates and scaffolding tools. Point to https://github.com/amitsk/cookiecutter-typescript-library (or a similar TS template).
    **** Walk through the GitHub workflow: create repo, push, open PR, CI runs automatically.

*** Using This Setup for JavaScript Projects
    **** Explain that everything in this tutorial (except TypeScript-specific compiler steps) works equally well for plain JavaScript projects.
    **** How to configure Vite, Vitest, ESLint, Prettier, Biome, Fastify, and the database libraries without TypeScript.
    **** Using JSDoc annotations as a lighter-weight alternative to TypeScript for type hints in JS.
    **** When to choose JavaScript over TypeScript: small scripts, quick prototypes, teams not ready for TS.
    **** When to add TypeScript later: using allowJs in tsconfig.json to incrementally migrate a JS project to TypeScript.
    **** The key message: the tooling ecosystem (Vite, Vitest, Biome, Fastify, Drizzle) is shared - learning this setup makes you effective in both JS and TS.
