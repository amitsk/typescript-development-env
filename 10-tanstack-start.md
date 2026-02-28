# Chapter 10: Full-Stack with TanStack Start

[← Previous: REST APIs with Fastify](./09-fastify.md) | [Back to README](./README.md) | [Next: Database Libraries →](./11-databases.md)

---

In the last chapter you built a standalone REST API with Fastify. That works great when you need a backend that serves multiple clients — a web app, a mobile app, a third-party service. But when you are building a web application where the frontend and backend always go together, maintaining two separate projects has costs: two deployment pipelines, two sets of dependencies, and you have to manually keep the request and response types in sync between them.

Full-stack frameworks solve this by putting both the server and the client in the same project. This chapter covers TanStack Start — one of the most TypeScript-forward full-stack frameworks available today.

## What is a Full-Stack Framework?

A full-stack framework handles both the frontend (React components, routing, rendering) and the backend (serving pages, fetching data, running server-side logic) from a single codebase and a single deployment.

Here is how the two architectures compare:

**Split architecture (Chapter 9 approach):**
- Fastify API server — handles all data access and business logic, returns JSON
- React SPA (built with Vite) — fetches data from the API at runtime in the browser
- Two separate projects, two deployments, types duplicated in both

**Full-stack framework approach:**
- One project, one deployment
- React components on the frontend
- Server functions and loaders on the backend
- Types defined once, shared automatically across the entire stack — your database schema type flows all the way to the React component prop without any manual duplication

The full-stack approach trades some flexibility (one frontend per backend) for a dramatically better developer experience on web-first projects.

## TanStack Start

[TanStack Start](https://tanstack.com/start) ([GitHub](https://github.com/TanStack/router)) is a full-stack React framework built on top of two well-established TanStack libraries:

- **TanStack Router** — widely regarded as the most type-safe React router. Route params, search params, and loader data are all fully typed with zero boilerplate.
- **Vite** — the build tool you learned about in Chapter 8.

### Key Features

**File-based routing** means your file system is your route configuration. Create `app/routes/about.tsx` and you have an `/about` route. Create `app/routes/users/$userId.tsx` and you have a `/users/:userId` route. No manual route registration, no forgetting to add a new page to the router config.

**Type-safe routes** take this further: TanStack Router infers the types of route params, search params, and loader data from the file structure itself. If a route has a `$userId` param, `request.params.userId` is typed as `string` automatically — no interface needed.

**Server functions** are the standout feature. You write a function, annotate it with `createServerFn()`, and TanStack Start ensures it only ever runs on the server — but you call it from the client like any ordinary async function. No HTTP routes to define, no `fetch` calls to write, no request/response types to maintain. The TypeScript types flow through automatically.

**Full-stack type safety** is the result: you can write a database query in a server function and use the return type directly in a React component — TypeScript tracks the type through the entire chain.

**Streaming SSR** with React Suspense is built-in. You can stream parts of a page to the browser as they become ready, giving users faster perceived load times without extra setup.

## Scaffolding a TanStack Start Project

TanStack Start has an interactive project scaffolder:

```bash
pnpm create tsrouter-app my-fullstack-app
```

The scaffolder will prompt you for options. For following along with this chapter, select:

- Framework: **TanStack Start**
- Language: **TypeScript**
- Styling: **Tailwind CSS** (optional, but recommended)

Then install and start the development server:

```bash
cd my-fullstack-app
pnpm install
pnpm dev
```

Open `http://localhost:3000` and you will see the starter app running.

## Project Structure Walkthrough

The scaffolder creates this directory layout:

```
my-fullstack-app/
├── app/
│   ├── routes/
│   │   ├── __root.tsx          # root layout (wraps all pages)
│   │   ├── index.tsx           # / route (home page)
│   │   └── users/
│   │       ├── index.tsx       # /users route
│   │       └── $userId.tsx     # /users/:userId (dynamic param)
│   ├── components/             # shared React components
│   ├── server/                 # server-only code
│   │   └── db.ts               # database connection
│   └── router.tsx              # router configuration
├── public/                     # static assets
├── app.config.ts               # TanStack Start config
├── tsconfig.json
└── package.json
```

The most important directory is `app/routes/`. Every file here becomes a page in your application. The filename determines the URL:

| File | URL |
|------|-----|
| `app/routes/index.tsx` | `/` |
| `app/routes/about.tsx` | `/about` |
| `app/routes/users/index.tsx` | `/users` |
| `app/routes/users/$userId.tsx` | `/users/:userId` |
| `app/routes/__root.tsx` | Wraps every page (layout) |

The `app/server/` directory is a convention for code that should only run on the server — database connections, secrets, business logic. Files here are never bundled into the browser JavaScript bundle.

## Creating a Route

Every route file exports a `Route` object created with `createFileRoute`. The route object holds the loader (server-side data fetching) and the component (what gets rendered).

Here is a minimal home page at `app/routes/index.tsx`:

```typescript
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/')({
  component: HomePage,
});

function HomePage() {
  return (
    <div>
      <h1>Welcome to TanStack Start!</h1>
      <p>Edit app/routes/index.tsx to change this page.</p>
    </div>
  );
}
```

The string `'/'` passed to `createFileRoute` must match the file's path in the `routes/` directory. TanStack Router uses this to connect the file to the right URL and to infer the correct types.

## Server Functions — The Key TanStack Start Feature

Server functions are the heart of TanStack Start. They let you write server-side code — database queries, calls to external APIs, anything that should not run in the browser — and call it from your components or loaders as if it were a normal TypeScript function.

Create `app/server/users.ts`:

```typescript
import { createServerFn } from '@tanstack/start';

interface User {
  id: number;
  name: string;
  email: string;
}

// This code runs on the SERVER only — it is never sent to the browser.
// In a real app, replace this array with a database query.
const usersData: User[] = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' },
];

export const getUsers = createServerFn().handler(async (): Promise<User[]> => {
  return usersData;
});

export const getUserById = createServerFn()
  .validator((id: string) => id)
  .handler(async ({ data: id }): Promise<User | undefined> => {
    return usersData.find((u) => u.id === parseInt(id));
  });
```

A few important points:

- `createServerFn()` marks the function as server-only. TanStack Start's build step ensures this code is never included in the browser bundle — not even the import.
- `.validator()` validates and types the input. If you pass the wrong type, TypeScript catches it at the call site.
- `.handler()` is the actual function body. `data` is the validated input.
- The return type `Promise<User[]>` flows through to wherever the function is called — no extra type annotation needed on the consuming side.

### Using Server Functions in a Loader

The most common pattern is calling a server function from a route's `loader`. The loader runs on the server before the page renders, so the data is available immediately — no loading spinner on first page load.

Create `app/routes/users/index.tsx`:

```typescript
import { createFileRoute } from '@tanstack/react-router';
import { getUsers } from '../../server/users';

export const Route = createFileRoute('/users/')({
  // loader runs on the server before the component renders
  loader: () => getUsers(),
  component: UsersPage,
});

function UsersPage() {
  // useLoaderData() returns the data from the loader — fully typed as User[]
  const users = Route.useLoaderData();

  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>
            {user.name} — {user.email}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

`Route.useLoaderData()` returns exactly the type that `loader` returns — in this case `User[]`. TypeScript knows this without any type annotation. If you rename a field in `User`, TypeScript will flag every place in the component that uses the old name.

## Dynamic Routes

A dynamic route segment starts with `$` in the filename. The `$userId` part of `app/routes/users/$userId.tsx` becomes the `userId` param.

Create `app/routes/users/$userId.tsx`:

```typescript
import { createFileRoute, notFound } from '@tanstack/react-router';
import { getUserById } from '../../server/users';

export const Route = createFileRoute('/users/$userId')({
  loader: ({ params }) => getUserById({ data: params.userId }),
  component: UserDetailPage,
});

function UserDetailPage() {
  const user = Route.useLoaderData();

  if (!user) {
    return <div>User not found.</div>;
  }

  return (
    <div>
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
    </div>
  );
}
```

Notice that `params.userId` is typed as `string` automatically — TanStack Router reads the `$userId` in the filename and infers the param name. If you mistype `params.userId` as `params.userID`, TypeScript gives you an error immediately.

## The Root Layout

`app/routes/__root.tsx` is a special file that wraps every page in your app. It is where you put navigation, headers, footers, and global providers.

```typescript
import { createRootRoute, Link, Outlet } from '@tanstack/react-router';

export const Route = createRootRoute({
  component: RootLayout,
});

function RootLayout() {
  return (
    <html lang="en">
      <head>
        <title>My App</title>
      </head>
      <body>
        <nav>
          <Link to="/">Home</Link>
          {' | '}
          <Link to="/users">Users</Link>
        </nav>
        <main>
          {/* Outlet renders the current route's component */}
          <Outlet />
        </main>
      </body>
    </html>
  );
}
```

`<Outlet />` is a placeholder — TanStack Router replaces it with the current page's component. `<Link>` generates type-safe anchor tags: if you pass a `to` value that does not match any route, TypeScript warns you.

## When to Choose TanStack Start vs Fastify + React

Both architectures are valid. The right choice depends on what you are building.

| Dimension | TanStack Start | Fastify API + React SPA |
|-----------|---------------|------------------------|
| **Project structure** | Single repo, single deployment | Two separate repos/deployments |
| **Type safety** | End-to-end, automatic | Manual — you maintain shared types |
| **Rendering** | SSR by default | Client-side rendering (CSR) |
| **Use case** | Web apps with a UI | APIs serving multiple clients |
| **Multiple frontends** | Not ideal | Excellent — one API, many clients |
| **Mobile app support** | Frontend is web-only | API serves mobile apps natively |
| **Flexibility** | More opinionated | Full control |
| **Learning curve** | Moderate | Lower for API-only work |

**Choose TanStack Start when:**
- You are building a web application with a user interface
- You want full-stack type safety without duplicating types
- You want SSR for better initial page load performance and SEO
- You are working solo or on a small team and want to ship quickly

**Choose Fastify (or another API framework) when:**
- You are building a pure API consumed by mobile apps, third-party developers, or multiple frontends
- You need maximum flexibility in how the backend is structured
- Your team has separate frontend and backend developers who prefer to work independently
- You already have a frontend and just need to add a backend

## What's Next?

Whether you use TanStack Start or a standalone Fastify API, you will eventually need to store data somewhere. The next chapter covers database access in Node.js — from lightweight SQLite to PostgreSQL — and compares three popular TypeScript-first options: Drizzle ORM, Prisma, and Kysely.

---

[← Previous: REST APIs with Fastify](./09-fastify.md) | [Back to README](./README.md) | [Next: Database Libraries →](./11-databases.md)
