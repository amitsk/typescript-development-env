# Chapter 12: Logging

[← Previous: Database Libraries](./11-databases.md) | [Back to README](./README.md) | [Next: CI/CD →](./13-ci-cd.md)

---

## Why Logging Matters

Every application needs a way to report what it is doing. The simplest approach is `console.log()`:

```typescript
console.log("Starting job...");
console.log(`Processing ${records.length} records`);
console.log("Done.");
```

This works fine during development, but falls apart in production:

- You can't easily filter or silence specific messages
- There's no timestamp, severity level, or structured context attached
- Output goes only to stdout — you can't route errors to a file or external service
- `console.log` is synchronous in Node.js and can block the event loop under load

A proper logging library gives you all of that with minimal overhead.

---

## console.log — The Baseline

`console` has more methods than just `log`:

```typescript
console.debug("Verbose diagnostic info");  // maps to DEBUG
console.log("Normal operation");           // maps to INFO
console.info("Application started");       // same as log
console.warn("Something unexpected");      // maps to WARN (stderr)
console.error("Something failed");         // maps to ERROR (stderr)
```

This is enough for simple scripts and short-lived processes. For long-running services — APIs, background workers, Lambda functions — you want a real logger.

---

## Winston

[Winston](https://github.com/winstonjs/winston) is the most widely used Node.js logging library. It's flexible and configuration-driven: you define transports (destinations) and formatters, and Winston routes messages through them.

```bash
npm install winston
# or
pnpm add winston
```

### Basic Setup

```typescript
import winston from "winston";

const logger = winston.createLogger({
  level: "info",  // log INFO and above
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: "app.log" }),
  ],
});

logger.info("Application started");
logger.warn("Config file not found, using defaults");
logger.error("Database connection failed", { host: "localhost", port: 5432 });
```

JSON output:
```json
{"level":"info","message":"Application started","timestamp":"2024-05-01T12:00:00.000Z"}
{"level":"warn","message":"Config file not found, using defaults","timestamp":"2024-05-01T12:00:00.001Z"}
{"level":"error","message":"Database connection failed","host":"localhost","port":5432,"timestamp":"2024-05-01T12:00:00.002Z"}
```

### Human-Readable Format for Development

```typescript
const devFormat = winston.format.combine(
  winston.format.colorize(),
  winston.format.timestamp({ format: "HH:mm:ss" }),
  winston.format.printf(({ timestamp, level, message, ...meta }) => {
    const extras = Object.keys(meta).length ? " " + JSON.stringify(meta) : "";
    return `${timestamp} ${level}: ${message}${extras}`;
  })
);

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL ?? "info",
  format: process.env.NODE_ENV === "production"
    ? winston.format.combine(winston.format.timestamp(), winston.format.json())
    : devFormat,
  transports: [new winston.transports.Console()],
});
```

### Child Loggers

Add context to a logger for a subsystem or request:

```typescript
const requestLogger = logger.child({
  requestId: "abc-123",
  userId: 42,
});

requestLogger.info("Processing request");
// → {"level":"info","message":"Processing request","requestId":"abc-123","userId":42,...}
```

### Multiple Transports

Route different log levels to different destinations:

```typescript
const logger = winston.createLogger({
  level: "debug",
  format: winston.format.json(),
  transports: [
    // INFO and above to console
    new winston.transports.Console({ level: "info" }),
    // DEBUG and above to a file
    new winston.transports.File({ filename: "debug.log", level: "debug" }),
    // ERROR only to a separate file
    new winston.transports.File({ filename: "errors.log", level: "error" }),
  ],
});
```

### TypeScript Types

Winston ships its own types. For additional transports (Datadog, Elasticsearch, etc.), community packages provide `@types/*` where needed. Core usage requires no extra type packages.

---

## Pino

[Pino](https://github.com/pinojs/pino) is a high-performance JSON logger. It's designed to add as little overhead as possible to your application — logging is done asynchronously via a separate worker, keeping the main thread free. Pino is the default logger in [Fastify](./09-fastify.md) and is popular in any latency-sensitive Node.js service.

```bash
npm install pino
npm install --save-dev @types/pino   # if needed — recent versions ship types
# or
pnpm add pino
```

### Basic Usage

```typescript
import pino from "pino";

const logger = pino({
  level: process.env.LOG_LEVEL ?? "info",
});

logger.info("Application started");
logger.warn({ configFile: null }, "Config file not found, using defaults");
logger.error({ err, host: "localhost" }, "Database connection failed");
```

Output is newline-delimited JSON:
```json
{"level":30,"time":1714561200000,"pid":12345,"hostname":"host","msg":"Application started"}
{"level":40,"time":1714561200001,"pid":12345,"hostname":"host","configFile":null,"msg":"Config file not found, using defaults"}
```

### Pretty Printing for Development

Pino outputs raw JSON, which is fast but hard to read during development. Use `pino-pretty` as a transform:

```bash
npm install --save-dev pino-pretty
```

```typescript
import pino from "pino";

const logger = pino(
  { level: "debug" },
  process.env.NODE_ENV === "development"
    ? pino.transport({ target: "pino-pretty", options: { colorize: true } })
    : process.stdout
);
```

Or pipe from the terminal:
```bash
node server.js | npx pino-pretty
```

### Child Loggers

```typescript
const requestLogger = logger.child({
  requestId: "abc-123",
  userId: 42,
});

requestLogger.info("Processing request");
// → {"level":30,"time":...,"requestId":"abc-123","userId":42,"msg":"Processing request"}
```

### Using with Fastify

Pino is built into Fastify — just enable it:

```typescript
import Fastify from "fastify";

const app = Fastify({
  logger: {
    level: "info",
    transport: process.env.NODE_ENV === "development"
      ? { target: "pino-pretty", options: { colorize: true } }
      : undefined,
  },
});

app.get("/", async (request, reply) => {
  request.log.info("Handling root route");
  return { hello: "world" };
});
```

Fastify automatically logs incoming requests and responses, including method, URL, status code, and response time.

### Serializers

Tell Pino how to format common objects:

```typescript
const logger = pino({
  serializers: {
    req: pino.stdSerializers.req,
    res: pino.stdSerializers.res,
    err: pino.stdSerializers.err,
  },
});
```

The built-in serializers strip sensitive headers and format errors with stack traces.

---

## Comparison

| Feature | `console` | Winston | Pino |
|---|---|---|---|
| **Setup** | None | Moderate | Minimal |
| **Log levels** | Partial (warn/error) | Full | Full |
| **Structured JSON** | No | Yes | Yes (native) |
| **Colorized dev output** | Partial | Yes | Via `pino-pretty` |
| **File output** | No | Built-in | Via transports |
| **Performance** | Sync, blocking | Good | Excellent (async) |
| **Child loggers** | No | Yes | Yes |
| **Ecosystem** | N/A | Large | Growing (Fastify native) |
| **Best for** | Scripts, debugging | Flexible, complex routing | High-throughput APIs |

### Recommendations

- **Fastify project:** Use **Pino** — it's built in and zero-config.
- **Existing Express project:** Use **Winston** — it's flexible and the most widely deployed.
- **High-throughput service:** Use **Pino** — the async design minimizes event loop impact.
- **Complex routing needs** (multiple outputs, custom transports): Use **Winston**.
- **Lambda functions:** Either works well; Pino's JSON output integrates naturally with CloudWatch and Datadog.

---

## Logging in a Fastify API

```typescript
import Fastify from "fastify";

const app = Fastify({
  logger: {
    level: process.env.LOG_LEVEL ?? "info",
    transport:
      process.env.NODE_ENV !== "production"
        ? { target: "pino-pretty", options: { colorize: true } }
        : undefined,
  },
});

app.addHook("onRequest", async (request) => {
  request.log.info({ url: request.url }, "Request received");
});

app.addHook("onResponse", async (request, reply) => {
  request.log.info(
    { url: request.url, statusCode: reply.statusCode, responseTime: reply.elapsedTime },
    "Request completed"
  );
});
```

---

## What's Next?

With logging in place, the next step is CI/CD — automating your tests, linting, and deployments so that every push to GitHub is validated automatically.

[← Previous: Database Libraries](./11-databases.md) | [Back to README](./README.md) | [Next: CI/CD →](./13-ci-cd.md)
