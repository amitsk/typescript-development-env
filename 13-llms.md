# Chapter 13: LLMs and AI-Powered Development

[← Previous: CI/CD](./12-ci-cd.md) | [Back to README](./README.md) | [Next: Putting It All Together →](./14-putting-it-together.md)

---

The way developers write code has changed significantly in the last few years. AI tools can now generate boilerplate, explain complex types, write test cases, and suggest refactors — all in seconds. This chapter introduces the major AI tools available to TypeScript developers and explains how to use them effectively.

## What are LLMs?

**Large Language Models** (LLMs) are AI systems trained on enormous amounts of text — including billions of lines of code from public repositories. Because they have seen so much TypeScript, JavaScript, and documentation, they can:

- Generate new code from a plain-English description
- Explain what an existing piece of code does
- Find bugs and suggest fixes
- Write test cases for your functions
- Translate between programming styles or patterns

### How LLMs help TypeScript developers specifically

TypeScript's type system is powerful but can be verbose and complex. LLMs are particularly good at:

- **Generating boilerplate**: route handlers, database schemas, Zod validators — all the repetitive code that takes time to write from scratch
- **Explaining complex types**: utility types like `ReturnType<typeof fn>` or deeply nested generics can be confusing — an LLM can walk you through them
- **Writing tests**: given a function, an LLM can generate a complete test file with edge cases you might not have thought of
- **Suggesting refactors**: "make this function generic" or "extract this into a reusable hook" are things LLMs do well
- **Debugging type errors**: paste a TypeScript error message and an LLM can usually explain exactly what is wrong and how to fix it

### Practical benefits

- **Faster development**: stop searching Stack Overflow for the right syntax — describe what you want and get working code in seconds
- **Learning new patterns**: when you are unfamiliar with a library, an LLM can show you how to use it in your specific context
- **Reducing context switching**: instead of leaving your editor to search documentation, ask directly in your workflow

---

## AI Tools for TypeScript Development

There are three major tools that TypeScript developers use today. Each has a different interface and different strengths, but they all use similar underlying models.

---

## Cursor

[Cursor](https://cursor.sh) is an AI-first code editor built on top of VSCode. If you already know VSCode, Cursor will feel immediately familiar — all your extensions, themes, and keybindings work. The difference is that AI is deeply integrated into the editing experience rather than bolted on as an extension.

### Key features

**Cmd+K / Ctrl+K — Inline AI editing**

Select a block of code, press Ctrl+K, and describe what you want to change. Cursor makes the edit directly in your file. This is faster than copying code into a chat window, reading the response, and manually applying the changes.

**Cmd+L / Ctrl+L — AI chat with codebase context**

Open a chat panel that has access to your entire project. You can ask questions about your codebase — "how does authentication work in this project?" — and Cursor will read the relevant files and answer based on your actual code, not a generic explanation.

**Tab completion**

As you type, Cursor offers multi-line AI completions. Unlike simple autocomplete that finishes variable names, Cursor can complete entire function bodies or test cases. Press Tab to accept.

**Agent mode**

Describe a multi-step task and Cursor will autonomously edit multiple files, run commands, and verify the results. For example: "add input validation to all the API routes using Zod and write tests for each one."

### TypeScript-specific advantages

Cursor reads your `tsconfig.json`, your type definitions, and your imports. When you ask it to write a function, it knows what types are available in your project and generates code that will actually compile. This is significantly better than asking a general-purpose chatbot that has no context about your project.

### Example prompts for TypeScript development

These are the kinds of prompts that work especially well in Cursor:

```
"Add a type-safe Fastify route for creating a user with Zod validation"
"Write Vitest tests for all functions in src/calculator.ts"
"Refactor this function to use proper TypeScript generics"
"Explain what this complex TypeScript type does"
```

---

## Claude Code

[Claude Code](https://github.com/anthropics/claude-code) is Anthropic's official CLI that brings Claude AI directly to your terminal. Instead of working inside an editor, you interact with Claude from the command line while it reads your files, runs commands, and edits code autonomously.

### Key features

**Terminal-native**

Claude Code works without leaving your terminal. If you prefer working in Neovim, Emacs, or just a plain terminal with your editor of choice, you do not need to switch to a new application.

**Full project context**

Claude Code reads your entire project directory. It understands the structure of your codebase, your configuration files, your dependencies, and how your modules relate to each other. This context makes its responses far more relevant than a generic chatbot.

**Runs commands for you**

Claude Code can run your tests, your linter, and your build. When it makes a change, it can immediately verify that the change works. If a test fails after an edit, it can diagnose the failure and fix it.

**Git-aware**

Claude Code understands your git history. It can help you write commit messages, review the diff before a commit, or summarize what changed in a branch.

### Installation

```bash
npm install -g @anthropic/claude-code
claude
```

Running `claude` without arguments starts an interactive session in your current directory.

### Example TypeScript workflows

```bash
claude "Set up Vitest with coverage for this project"
claude "Add TypeScript types to all the untyped functions in src/"
claude "Write a Fastify plugin for JWT authentication"
claude "Review src/api.ts for type safety issues"
```

Each of these tasks would take a developer several minutes to do manually. Claude Code can do them in seconds, and because it actually runs your type checker and tests after making changes, the results are verified before you even look at them.

---

## GitHub Copilot

[GitHub Copilot](https://github.com/features/copilot) is an AI pair programmer that integrates directly into VSCode, JetBrains IDEs, Neovim, and other editors. It is developed by GitHub and Microsoft and powered by OpenAI's models.

### How it works

Copilot watches what you are typing and offers inline suggestions. When you write a function signature or a comment describing what you want, Copilot predicts what the implementation should look like and shows it as a ghost suggestion. Press Tab to accept, or keep typing to ignore it.

### Copilot Chat

In addition to inline completions, Copilot offers a chat panel in the sidebar. You can highlight code and ask "what does this do?" or "fix the bug in this function" — similar to Cursor's chat feature.

### Where Copilot shines for TypeScript

- **Completing repetitive patterns**: if you have written three similar route handlers, Copilot can complete the fourth one just from the function signature
- **Generating type definitions**: describe a data structure in a comment and Copilot will generate the TypeScript interface
- **Filling out test cases**: write one test case and Copilot can suggest the remaining cases following the same pattern

### Access and pricing

Copilot is subscription-based, but there are free tiers available:

- Free for verified students through GitHub Education
- Free for maintainers of popular open-source projects
- Paid plans for individual developers and teams

---

## Best Practices for AI-Assisted TypeScript Development

AI tools are powerful, but they are not infallible. Here are guidelines for getting the best results and avoiding common pitfalls.

### Always review generated TypeScript types

AI models can generate TypeScript that looks correct but contains subtle type errors. A generated `any` type or an incorrectly widened union type can undermine the whole point of using TypeScript. Always read the generated types carefully, and always run `tsc --noEmit` after accepting AI-generated code.

### Use AI to generate scaffolding, then verify the logic yourself

AI is excellent at generating the structure of tests — the `describe` blocks, the `it` calls, the basic `expect` statements. But it can get the expected values wrong, especially for business logic it does not fully understand. Use AI to create the test file, then verify each assertion is testing the right thing.

### Provide context in your prompts

The more context you give, the better the result. Instead of:

```
"Write a function to validate user input"
```

Try:

```
"Write a TypeScript function that validates a user registration form.
Use Zod. The form has: email (valid email), password (min 8 chars, at least
one number), and username (3-20 chars, alphanumeric only). Return a
discriminated union of success | error."
```

Mentioning your TypeScript version, the libraries you are using, and the specific constraints produces much more useful code.

### Let AI explain TypeScript's utility types

TypeScript's built-in utility types are one of the areas where developers most often get stuck. LLMs are excellent at explaining them in plain English:

```
"Explain the difference between Partial<T>, Required<T>, and Pick<T, K> with examples"
"What does ReturnType<typeof myFunction> actually produce?"
"When should I use Readonly<T> vs as const?"
```

### Run your full check suite after every AI-generated change

Make it a habit: after accepting any significant AI suggestion, run:

```bash
pnpm typecheck   # catch type errors
pnpm lint        # catch style and logic issues
pnpm test:run    # catch broken behavior
```

AI tools are fast, but your CI pipeline is the final arbiter. Running checks locally before pushing saves you the round-trip of a failed CI build.

---

## What's Next?

You now have a full toolkit: TypeScript, testing, linting, databases, CI/CD, and AI assistance. In the next chapter, we put it all together by building a complete project from scratch — applying every tool we have covered in the right order.

[← Previous: CI/CD](./12-ci-cd.md) | [Back to README](./README.md) | [Next: Putting It All Together →](./14-putting-it-together.md)
