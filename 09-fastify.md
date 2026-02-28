# Chapter 9: REST APIs with Fastify

[← Previous: Building with Vite](./08-vite.md) | [Back to README](./README.md) | [Next: Full-Stack with TanStack Start →](./10-tanstack-start.md)

---

In the previous chapter you learned to bundle frontend apps with Vite. Now we shift to the backend: building a REST API server that your frontend (or mobile app, or any HTTP client) can talk to. This chapter uses Fastify — a modern, TypeScript-native Node.js web framework that is both fast and pleasant to work with.

## What is a REST API?

A REST API is a server that listens for HTTP requests and responds with data — almost always JSON. Clients send requests using standard HTTP methods:

| Method | Meaning | Example |
|--------|---------|---------|
| `GET` | Read data | `GET /users` — list all users |
| `POST` | Create data | `POST /users` — create a new user |
| `PUT` / `PATCH` | Update data | `PUT /users/1` — update user with id 1 |
| `DELETE` | Delete data | `DELETE /users/1` — delete user with id 1 |

The server processes the request and returns a JSON response, like:

```json
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com"
}
```

REST APIs are consumed by frontend web apps, mobile apps, third-party services, and other backend services. Building yours in TypeScript gives you:

- **Type safety for request and response shapes** — you know exactly what data comes in and goes out
- **Better tooling** — your editor understands your route handlers, catches typos, and autocompletes field names
- **Shared types** — if your frontend is also TypeScript, you can share type definitions between the two projects

## Fastify — Fast and TypeScript-Native

[Fastify](https://fastify.dev) ([GitHub](https://github.com/fastify/fastify)) is one of the fastest Node.js web frameworks available. It was designed from the start to have excellent TypeScript support — unlike Express, which was written before TypeScript existed and relies on a separate `@types/express` package that sometimes lags behind.

### Why Fastify over Express?

Fastify processes significantly more requests per second than Express on equivalent hardware. Here is a rough comparison based on publicly available benchmarks:

| Framework | Requests/sec (approx.) | TypeScript support | JSON Schema validation |
|-----------|------------------------|-------------------|----------------------|
| Fastify | ~75,000+ | Built-in | Built-in |
| Express | ~15,000–20,000 | Via `@types/express` | Requires middleware |
| Koa | ~25,000–30,000 | Via `@types/koa` | Requires middleware |

The gap comes from several optimizations: Fastify uses a compiled JSON serializer (faster than `JSON.stringify`), a highly optimized router, and avoids unnecessary middleware overhead.

Beyond raw speed, Fastify's key advantages are:

- **Built-in TypeScript support** — generics for request params, body, query, and reply types
- **JSON Schema validation** — validates incoming request data automatically, before your handler runs
- **Plugin architecture** — keeps large codebases organized; everything from CORS to authentication is a plugin
- **Auto-generated OpenAPI docs** — add `@fastify/swagger` and your API documents itself

## Setting Up a Fastify Project

Start by creating a new directory and initializing a project:

```bash
mkdir my-api
cd my-api
pnpm init
pnpm add fastify
pnpm add -D typescript @types/node tsx
```

`tsx` is the development runner — it executes TypeScript files directly without a separate compile step, and `tsx watch` restarts the server automatically when you save a file.

Create a `tsconfig.json` in the project root:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```

Create the `src/` directory:

```bash
mkdir src
```

## Creating a Basic Fastify Server

Create `src/server.ts`:

```typescript
import Fastify from 'fastify';

const fastify = Fastify({
  logger: true,
});

// Basic health check route
fastify.get('/health', async (request, reply) => {
  return { status: 'ok', timestamp: new Date().toISOString() };
});

// Start the server
const start = async () => {
  try {
    await fastify.listen({ port: 3000, host: '0.0.0.0' });
    console.log('Server running at http://localhost:3000');
  } catch (err) {
    fastify.log.error(err);
    process.exit(1);
  }
};

start();
```

A few things to notice here:

- `Fastify({ logger: true })` enables the built-in structured logger (powered by [pino](https://getpino.io)). Every request is logged automatically.
- Route handlers return a value directly — Fastify serializes it to JSON for you. You do not need `res.json(...)` like in Express.
- `host: '0.0.0.0'` makes the server accessible on all network interfaces, which is important when running inside Docker.

Add scripts to `package.json`:

```json
"scripts": {
  "dev": "tsx watch src/server.ts",
  "build": "tsc",
  "start": "node dist/server.js"
}
```

Run the development server:

```bash
pnpm dev
```

Visit `http://localhost:3000/health` in your browser and you should see:

```json
{ "status": "ok", "timestamp": "2024-01-15T10:30:00.000Z" }
```

## Type-Safe Routes with TypeScript Generics

Fastify uses TypeScript generics to let you declare the expected shape of request parameters, query strings, request body, and reply. This is where Fastify's TypeScript support really shines.

Create `src/routes/users.ts`:

```typescript
import { FastifyInstance, FastifyRequest, FastifyReply } from 'fastify';

interface User {
  id: number;
  name: string;
  email: string;
}

interface CreateUserBody {
  name: string;
  email: string;
}

interface UserParams {
  id: string;
}

// In-memory store for demo purposes
const users: User[] = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' },
];

export async function userRoutes(fastify: FastifyInstance) {
  // GET /users — list all users
  fastify.get('/users', async (request, reply): Promise<User[]> => {
    return users;
  });

  // GET /users/:id — get one user
  fastify.get<{ Params: UserParams }>(
    '/users/:id',
    async (request, reply) => {
      const id = parseInt(request.params.id);
      const user = users.find((u) => u.id === id);
      if (!user) {
        return reply.status(404).send({ error: 'User not found' });
      }
      return user;
    },
  );

  // POST /users — create a user
  fastify.post<{ Body: CreateUserBody }>(
    '/users',
    async (request, reply) => {
      const newUser: User = {
        id: users.length + 1,
        name: request.body.name,
        email: request.body.email,
      };
      users.push(newUser);
      return reply.status(201).send(newUser);
    },
  );
}
```

The generic parameter `<{ Params: UserParams }>` tells Fastify (and TypeScript) what shape `request.params` has. Without it, `request.params.id` would be typed as `unknown` and you would get a TypeScript error. The same pattern works for `Body`, `Querystring`, and `Reply`.

Now register these routes in `src/server.ts`. Update the file to add the import and registration:

```typescript
import Fastify from 'fastify';
import { userRoutes } from './routes/users';

const fastify = Fastify({
  logger: true,
});

fastify.get('/health', async (request, reply) => {
  return { status: 'ok', timestamp: new Date().toISOString() };
});

// Register route plugin
await fastify.register(userRoutes);

const start = async () => {
  try {
    await fastify.listen({ port: 3000, host: '0.0.0.0' });
  } catch (err) {
    fastify.log.error(err);
    process.exit(1);
  }
};

start();
```

Fastify's plugin system (`fastify.register`) is how you organize your routes. Each plugin receives its own encapsulated Fastify instance, which means plugins do not accidentally interfere with each other.

## JSON Schema Validation with TypeBox

So far, if someone sends a POST request with a missing `name` field, your handler will crash at runtime. Fastify has a built-in validation system based on JSON Schema — you provide a schema, and Fastify rejects invalid requests automatically before they reach your handler.

Writing JSON Schema by hand is verbose, but [TypeBox](https://github.com/sinclairzx81/typebox) lets you generate JSON Schema objects from TypeScript-like syntax. The `@fastify/type-provider-typebox` package wires TypeBox into Fastify so that the schema and the TypeScript types stay in sync.

Install the packages:

```bash
pnpm add @fastify/type-provider-typebox @sinclair/typebox
```

Here is how to use TypeBox with Fastify:

```typescript
import Fastify from 'fastify';
import { TypeBoxTypeProvider } from '@fastify/type-provider-typebox';
import { Type } from '@sinclair/typebox';

// .withTypeProvider<TypeBoxTypeProvider>() enables TypeBox inference
const fastify = Fastify().withTypeProvider<TypeBoxTypeProvider>();

const CreateUserSchema = Type.Object({
  name: Type.String({ minLength: 1 }),
  email: Type.String({ format: 'email' }),
});

fastify.post(
  '/users',
  { schema: { body: CreateUserSchema } },
  async (request, reply) => {
    // request.body is fully typed as { name: string; email: string }
    // AND validated before this code runs
    const { name, email } = request.body;
    return reply.status(201).send({ id: 1, name, email });
  },
);
```

If a request comes in with an invalid email format or a missing `name`, Fastify rejects it with a `400 Bad Request` response before your handler ever runs. No `if (!body.name) return reply.status(400)...` clutter needed.

## Useful Fastify Plugins

Fastify has a rich ecosystem of official plugins. Here are the most commonly needed ones:

| Plugin | Purpose |
|--------|---------|
| `@fastify/cors` | Add CORS headers so browser clients can call your API |
| `@fastify/jwt` | JWT-based authentication |
| `@fastify/swagger` + `@fastify/swagger-ui` | Auto-generate and serve OpenAPI documentation |
| `@fastify/rate-limit` | Rate limit requests to prevent abuse |
| `@fastify/static` | Serve static files |
| `@fastify/multipart` | Handle file uploads |

### Example: Setting Up CORS

If your Fastify API and your React frontend run on different ports (common in development — API on 3000, frontend on 5173), the browser will block requests unless the server sends the right CORS headers.

```bash
pnpm add @fastify/cors
```

Register the plugin in `src/server.ts`:

```typescript
import cors from '@fastify/cors';

await fastify.register(cors, {
  origin: ['http://localhost:5173'],  // allow requests from your Vite dev server
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
});
```

In production, replace `'http://localhost:5173'` with your actual frontend domain.

## Testing Your API with curl

While your server is running (`pnpm dev`), open a second terminal and test each endpoint:

```bash
# Health check
curl http://localhost:3000/health

# Get all users
curl http://localhost:3000/users

# Get a specific user
curl http://localhost:3000/users/1

# Create a new user (POST with JSON body)
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Charlie","email":"charlie@example.com"}'

# Try to get the newly created user
curl http://localhost:3000/users/3

# Test 404 handling
curl http://localhost:3000/users/999
```

The `-H "Content-Type: application/json"` header is important — it tells Fastify the body is JSON so it knows to parse it. Without it, `request.body` will be empty.

## Alternatives Worth Knowing

Fastify is an excellent default choice, but the Node.js/TypeScript ecosystem has other solid options:

**[Express](https://expressjs.com)** is the classic, most widely used Node.js framework. Nearly every Node.js tutorial from the past decade uses it. Its ecosystem is enormous — there is a middleware package for everything. The downsides: it is slower than Fastify, TypeScript support is bolted on via `@types/express`, and the API design is older (callback-based originally). If you are joining a team with existing Express code, you will absolutely need to know it.

**[Hono](https://hono.dev)** is an ultralight, edge-first framework designed to run anywhere — Cloudflare Workers, Deno, Bun, Node.js. It has excellent TypeScript support, a very small bundle size, and a clean API. If you are building for edge runtimes or Cloudflare Workers, Hono is the standout choice.

**[Elysia](https://elysiajs.com)** is a Bun-first framework with exceptional TypeScript inference and extreme performance. It pushes the boundaries of what TypeScript inference can do for API type safety. If you are using Bun as your runtime and want the best possible developer experience, Elysia is worth a serious look.

## What's Next?

You now have a working REST API with type-safe routes and request validation. The natural next step is connecting it to a real database — but before that, let's look at an alternative architecture: full-stack frameworks that combine your frontend and backend into a single project with shared types. That is what TanStack Start, covered in the next chapter, is all about.

---

[← Previous: Building with Vite](./08-vite.md) | [Back to README](./README.md) | [Next: Full-Stack with TanStack Start →](./10-tanstack-start.md)
