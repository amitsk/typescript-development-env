[← Previous: Code Formatting](./06-code-formatting.md) | [Back to README](./README.md) | [Next: Building with Vite →](./08-vite.md)

# Chapter 7: Unit Testing with Vitest

Writing code is only half the job. The other half is making sure it actually does what you think it does. Unit tests let you verify that individual functions and modules behave correctly — and they keep working correctly as your codebase grows and changes.

## What is Unit Testing?

A unit test is a small, focused piece of code that calls one of your functions and checks that it returns the right result. "Unit" refers to the smallest testable piece of your program — usually a single function or class method.

### Why Write Tests?

- **Confidence your code works** — you have proof that each function behaves correctly
- **Safe refactoring** — change the internals of a function and immediately know if you broke anything
- **Living documentation** — tests show exactly how a function is supposed to be used and what it should return
- **Faster debugging** — a failing test tells you precisely which function broke, instead of hunting through logs

### TypeScript Makes Testing Better

Because TypeScript catches type errors at compile time, an entire category of bugs (passing the wrong type of argument, accessing a property that does not exist) never make it to your tests. This means your tests can focus on logic and behavior rather than type safety.

---

## Testing in the JavaScript Ecosystem

There are two main test runners you will encounter for TypeScript projects:

### Jest

[Jest](https://jestjs.io) is the classic, battle-tested test runner that has dominated the JavaScript ecosystem for years. It works well, but it was designed before native ES modules and TypeScript were common, so using it with TypeScript requires extra setup: you need either Babel (to strip types before running) or `ts-jest` (to compile TypeScript). This adds configuration overhead and can slow down test runs.

### Vitest

[Vitest](https://vitest.dev) ([GitHub](https://github.com/vitest-dev/vitest)) is the modern choice. It was built alongside Vite and has native TypeScript support out of the box. No Babel, no `ts-jest`, no extra configuration.

**Why choose Vitest over Jest:**

- Faster test execution — uses esbuild to transpile TypeScript
- Native ES module support — no workarounds needed
- Minimal configuration — works immediately with a TypeScript project
- Jest-compatible API — if you know Jest, you already know Vitest
- Integrated with Vite — shares the same config if you are using Vite for your build

---

## Vitest — Setup and Usage

### Installation

```bash
pnpm add -D vitest
```

### Configuration

Create a `vitest.config.ts` file in your project root. If you are already using Vite, you can add the `test` block to your existing `vite.config.ts` instead.

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,       // makes describe, it, expect available without imports
    environment: 'node', // use 'jsdom' for browser-like testing
    include: ['src/**/*.{test,spec}.{ts,tsx}'],
    exclude: ['node_modules', 'dist'],
  },
});
```

The `globals: true` option makes the test functions (`describe`, `it`, `expect`) available everywhere without needing to import them. Many developers prefer this for cleaner test files.

### Add Scripts to package.json

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:watch": "vitest --watch"
  }
}
```

- `test` — starts Vitest in interactive watch mode (re-runs tests when files change)
- `test:run` — runs tests once and exits (the right choice for CI pipelines)
- `test:watch` — explicit watch mode, same as the default `test` command

---

## Writing Tests — a TypeScript Module and Its Tests

Let's build a simple calculator module and write tests for it. This will show the full pattern: a source file, a test file, and how they relate to each other.

### The Module: `src/calculator.ts`

First, the code we want to test:

```typescript
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

export function divide(a: number, b: number): number {
  if (b === 0) {
    throw new Error('Cannot divide by zero');
  }
  return a / b;
}
```

Each function has a clear type signature, so TypeScript will catch mistakes like passing a string where a number is expected before the tests even run.

### The Test File: `src/calculator.test.ts`

By convention, test files live alongside the source files they test and use the `.test.ts` suffix:

```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { add, subtract, multiply, divide } from './calculator';

describe('calculator', () => {
  describe('add', () => {
    it('adds two positive numbers', () => {
      expect(add(2, 3)).toBe(5);
    });

    it('adds negative numbers', () => {
      expect(add(-1, 1)).toBe(0);
    });

    it('adds zero', () => {
      expect(add(0, 0)).toBe(0);
    });
  });

  describe('subtract', () => {
    it('subtracts two numbers', () => {
      expect(subtract(5, 3)).toBe(2);
    });
  });

  describe('multiply', () => {
    it('multiplies two numbers', () => {
      expect(multiply(3, 4)).toBe(12);
    });

    it('multiplies by zero', () => {
      expect(multiply(5, 0)).toBe(0);
    });
  });

  describe('divide', () => {
    it('divides two numbers', () => {
      expect(divide(10, 2)).toBe(5);
    });

    it('throws when dividing by zero', () => {
      expect(() => divide(10, 0)).toThrow('Cannot divide by zero');
    });
  });
});
```

### Understanding the Test Structure

| Function | Purpose |
|----------|---------|
| `describe(name, fn)` | Groups related tests together. You can nest `describe` blocks for sub-groups. |
| `it(name, fn)` | Defines a single test case. `test()` is an alias — they are identical. |
| `expect(value)` | Creates an assertion. Chain a matcher onto it to check the value. |
| `beforeEach(fn)` | Runs a setup function before each test in the current `describe` block. Useful for resetting state. |

---

## Running Tests

```bash
pnpm test          # interactive watch mode — re-runs on file changes
pnpm test:run      # run once and exit (good for CI)
pnpm vitest run    # same as test:run
pnpm vitest --watch  # explicit watch mode
```

When you run `pnpm test:run`, the output looks something like this:

```
 RUN  v1.6.0

 ✓ src/calculator.test.ts (8 tests) 3ms
   ✓ calculator > add > adds two positive numbers
   ✓ calculator > add > adds negative numbers
   ✓ calculator > add > adds zero
   ✓ calculator > subtract > subtracts two numbers
   ✓ calculator > multiply > multiplies two numbers
   ✓ calculator > multiply > multiplies by zero
   ✓ calculator > divide > divides two numbers
   ✓ calculator > divide > throws when dividing by zero

 Test Files  1 passed (1)
      Tests  8 passed (8)
   Start at  12:34:56
   Duration  412ms
```

If a test fails, Vitest shows you exactly which assertion failed, what value it received, and what it expected.

---

## Useful Vitest Assertions

Vitest's `expect` function supports a rich set of matchers. Here are the ones you will use most often:

```typescript
// Strict equality — use for primitives (numbers, strings, booleans)
expect(add(1, 1)).toBe(2);
expect(result).toBe('hello');

// Deep equality — use for objects and arrays
expect(getUser()).toEqual({ id: 1, name: 'Alice' });
expect(getNumbers()).toEqual([1, 2, 3]);

// Nullability and truthiness
expect(result).toBeNull();
expect(result).toBeUndefined();
expect(result).toBeDefined();
expect(result).toBeTruthy();
expect(result).toBeFalsy();

// Arrays and strings
expect([1, 2, 3]).toContain(2);
expect('hello world').toContain('world');
expect([1, 2, 3]).toHaveLength(3);
expect('hello').toHaveLength(5);

// Objects — checks a subset of properties
expect(getUser()).toMatchObject({ name: 'Alice' });

// Errors
expect(() => divide(1, 0)).toThrow();
expect(() => divide(1, 0)).toThrow('Cannot divide by zero');
expect(() => divide(1, 0)).toThrow(Error);

// Async — resolved values
await expect(fetchUser(1)).resolves.toEqual({ id: 1, name: 'Alice' });

// Async — rejected errors
await expect(fetchData('bad-url')).rejects.toThrow('Not found');
```

---

## Testing Async Code

Most real-world code is asynchronous — fetching from APIs, reading files, querying databases. Vitest handles async tests cleanly with `async/await`:

```typescript
import { describe, it, expect } from 'vitest';

async function fetchUser(id: number): Promise<{ id: number; name: string }> {
  // simulate an async operation like a database query
  return { id, name: 'Alice' };
}

describe('fetchUser', () => {
  it('returns a user with the correct id', async () => {
    const user = await fetchUser(1);
    expect(user.id).toBe(1);
    expect(user.name).toBe('Alice');
  });

  it('resolves with the expected shape', async () => {
    await expect(fetchUser(1)).resolves.toMatchObject({ id: 1 });
  });
});
```

Mark the test callback with `async` and `await` the result just as you would in regular TypeScript. Vitest automatically detects that the test is async and waits for the promise to settle before evaluating the assertions.

---

## Code Coverage with Vitest

Code coverage measures what percentage of your code is actually executed during your test suite. If a function exists but no test calls it, coverage reports it as uncovered.

### Coverage Types

| Type | What it measures |
|------|-----------------|
| **Statements** | Individual executable statements |
| **Branches** | Each branch of `if/else`, `switch`, ternary operators |
| **Functions** | Whether each function was called at least once |
| **Lines** | Physical lines of code executed |

### Installation

Vitest uses `@vitest/coverage-v8` (powered by Node's built-in V8 coverage) as the coverage provider:

```bash
pnpm add -D @vitest/coverage-v8
```

### Configuration

Add a `coverage` block to your `vitest.config.ts`:

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],  // text: terminal output, html: visual report
      include: ['src/**/*.ts'],
      exclude: ['src/**/*.test.ts', 'src/**/*.spec.ts'],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 80,
        statements: 80,
      },
    },
  },
});
```

The `thresholds` section is powerful: if your coverage drops below 80% on any metric, the command exits with an error. This makes it easy to enforce a coverage floor in your CI pipeline.

### Running Coverage

Add a script to `package.json`:

```json
{
  "scripts": {
    "test:coverage": "vitest run --coverage"
  }
}
```

Then run:

```bash
pnpm test:coverage
```

The terminal output shows a table like this:

```
 RUN  v1.6.0

 ✓ src/calculator.test.ts (8 tests) 4ms

 % Coverage report from v8
--------------------|---------|----------|---------|---------|
File                | % Stmts | % Branch | % Funcs | % Lines |
--------------------|---------|----------|---------|---------|
All files           |   100   |   100    |   100   |   100   |
 calculator.ts      |   100   |   100    |   100   |   100   |
--------------------|---------|----------|---------|---------|
```

### The HTML Report

When you include `'html'` in the `reporter` array, Vitest generates a visual coverage report in the `coverage/` directory. Open `coverage/index.html` in your browser to see a file-by-file breakdown with line-level highlighting — uncovered lines are shown in red, covered lines in green. This makes it easy to find exactly which code paths your tests are missing.

---

## What's Next?

You now have a solid testing setup — your TypeScript functions are tested and you can track coverage. The next step is learning how to build and bundle your TypeScript project for production. In the next chapter, we will set up Vite as a build tool.

---

[← Previous: Code Formatting](./06-code-formatting.md) | [Back to README](./README.md) | [Next: Building with Vite →](./08-vite.md)
