# Chapter 16: AWS Lambda with TypeScript

[← Previous: Using This Setup for JavaScript Projects](./15-javascript.md) | [Back to README](./README.md)

---

Serverless computing lets you deploy functions without managing servers. AWS Lambda is the most widely used serverless platform — you write a function, upload it, and AWS handles scaling, availability, and infrastructure. TypeScript is an excellent fit for Lambda: type-safe event objects catch whole categories of bugs before your code ever runs in the cloud.

This chapter covers how to write Lambda functions in TypeScript, how to use **AWS SAM** to build and test locally, and how to integrate **Oxlint** and **oxfmt** into a Lambda project.

---

## What is AWS Lambda?

AWS Lambda runs your code in response to **events**: an HTTP request through API Gateway, a message on an SQS queue, a file uploaded to S3, a scheduled CloudWatch rule, and many more. You write a **handler function** — a TypeScript function with a specific signature — and Lambda invokes it whenever the triggering event occurs.

Key characteristics:

- **No servers to manage** — AWS handles capacity, patching, and availability
- **Charged per invocation** — you pay only when your function runs, not for idle time
- **Automatic scaling** — Lambda runs as many copies of your function in parallel as needed
- **Stateless** — each invocation gets a fresh execution environment (though the runtime may be reused across warm invocations for performance)

Lambda works with nearly every AWS service as a trigger, making it the glue layer in many event-driven architectures.

---

## Project Setup

### Prerequisites

Install the AWS CLI and AWS SAM CLI:

```bash
# AWS CLI
brew install awscli          # macOS
# or: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

# AWS SAM CLI
brew install aws-sam-cli     # macOS
# or: https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html

# Verify
aws --version
sam --version
```

### Initialize the Project

```bash
mkdir my-lambda-app
cd my-lambda-app
pnpm init
```

### Install Dependencies

```bash
# Runtime dependency — the AWS SDK is provided by Lambda at runtime, but
# you may want specific v3 clients for DynamoDB, S3, etc.
pnpm add @aws-sdk/client-dynamodb @aws-sdk/lib-dynamodb

# Dev dependencies
pnpm add -D typescript @types/node @types/aws-lambda
pnpm add -D esbuild          # fast bundler for Lambda packages
pnpm add -D oxlint           # fast Rust-based linter
pnpm add -D vitest           # testing
```

`@types/aws-lambda` provides TypeScript types for every AWS event object — API Gateway requests, S3 events, SQS messages, and more. This is the most important package for type-safe Lambda development.

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "CommonJS",
    "moduleResolution": "Node",
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

> **Note on modules:** Lambda's Node.js runtime supports both CommonJS and ES modules. CommonJS (`"module": "CommonJS"`) is the simpler choice for getting started. ES module support requires `"type": "module"` in `package.json` and the `.mjs` extension or a specific SAM configuration.

---

## Writing Lambda Handlers in TypeScript

### Basic Handler Structure

Every Lambda handler exports a function with this signature:

```typescript
export const handler = async (event: EventType, context: Context): Promise<ResponseType> => {
  // your logic here
};
```

- `event` — the triggering event, typed differently depending on the source (API Gateway, SQS, S3, etc.)
- `context` — runtime information like the function name, remaining timeout, and request ID
- The return value — depends on the trigger; API Gateway expects a specific response shape

### API Gateway Handler (HTTP Functions)

The most common Lambda use case: an HTTP endpoint via API Gateway v2 (HTTP API):

```typescript
import type {
  APIGatewayProxyEventV2,
  APIGatewayProxyResultV2,
  Context,
} from 'aws-lambda';

interface CreateUserBody {
  name: string;
  email: string;
}

export const handler = async (
  event: APIGatewayProxyEventV2,
  context: Context,
): Promise<APIGatewayProxyResultV2> => {
  try {
    // Parse the request body
    if (!event.body) {
      return {
        statusCode: 400,
        body: JSON.stringify({ error: 'Request body is required' }),
      };
    }

    const body = JSON.parse(event.body) as CreateUserBody;

    // Validate required fields
    if (!body.name || !body.email) {
      return {
        statusCode: 400,
        body: JSON.stringify({ error: 'name and email are required' }),
      };
    }

    // Your business logic here — e.g. write to DynamoDB
    const user = { id: crypto.randomUUID(), ...body, createdAt: new Date().toISOString() };

    return {
      statusCode: 201,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(user),
    };
  } catch (err) {
    console.error('Handler error:', err);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Internal server error' }),
    };
  }
};
```

TypeScript ensures you cannot accidentally return an invalid response shape. If you omit `statusCode` or set `body` to a non-string, the compiler will tell you immediately.

### Available Event Types from `@types/aws-lambda`

| Trigger | Type |
|---------|------|
| API Gateway HTTP API (v2) | `APIGatewayProxyEventV2` |
| API Gateway REST API (v1) | `APIGatewayProxyEvent` |
| SQS queue message | `SQSEvent` |
| S3 object upload | `S3Event` |
| SNS topic notification | `SNSEvent` |
| EventBridge / CloudWatch Events | `EventBridgeEvent<DetailType, Detail>` |
| DynamoDB Streams | `DynamoDBStreamEvent` |
| Scheduled (CloudWatch Events) | `ScheduledEvent` |

All of these are available from a single import:

```typescript
import type { APIGatewayProxyEventV2, SQSEvent, S3Event } from 'aws-lambda';
```

### SQS Handler Example

A Lambda that processes messages from an SQS queue:

```typescript
import type { SQSEvent, SQSRecord, Context } from 'aws-lambda';

interface OrderMessage {
  orderId: string;
  userId: string;
  amount: number;
}

async function processOrder(record: SQSRecord): Promise<void> {
  const message = JSON.parse(record.body) as OrderMessage;
  console.log(`Processing order ${message.orderId} for user ${message.userId}`);
  // ... business logic
}

export const handler = async (event: SQSEvent, context: Context): Promise<void> => {
  const results = await Promise.allSettled(
    event.Records.map(record => processOrder(record))
  );

  const failures = results.filter(r => r.status === 'rejected');
  if (failures.length > 0) {
    // Throwing causes Lambda to put failed messages back on the queue
    throw new Error(`${failures.length} records failed processing`);
  }
};
```

---

## AWS SAM

**AWS SAM** (Serverless Application Model) is an open-source framework that extends AWS CloudFormation for serverless applications. It provides:

- A simplified YAML syntax (`template.yaml`) for defining Lambda functions, API Gateway routes, SQS queues, DynamoDB tables, and IAM permissions
- `sam build` — compiles and packages your functions
- `sam local invoke` — runs a Lambda function locally using Docker
- `sam local start-api` — starts a local HTTP server that routes requests to your functions (just like API Gateway)
- `sam deploy` — deploys everything to AWS

### Project Structure

```
my-lambda-app/
├── src/
│   ├── handlers/
│   │   ├── create-user.ts
│   │   ├── get-user.ts
│   │   └── process-order.ts
│   └── lib/
│       ├── db.ts          # DynamoDB client setup
│       └── types.ts       # Shared TypeScript types
├── template.yaml          # SAM template
├── samconfig.toml         # SAM deployment configuration
├── tsconfig.json
├── biome.json             # or oxlint + oxfmt config
├── vitest.config.ts
└── package.json
```

### template.yaml

The SAM template defines your infrastructure as code:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: My TypeScript Lambda application

Globals:
  Function:
    Runtime: nodejs22.x
    Architectures:
      - arm64          # Graviton — cheaper and faster for most workloads
    Timeout: 30
    MemorySize: 256
    Environment:
      Variables:
        TABLE_NAME: !Ref UsersTable
        AWS_REGION: !Ref AWS::Region

Resources:
  CreateUserFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: .aws-sam/build/CreateUserFunction/
      Handler: create-user.handler
      Events:
        CreateUser:
          Type: HttpApi
          Properties:
            Path: /users
            Method: POST
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref UsersTable
    Metadata:
      BuildMethod: esbuild
      BuildProperties:
        EntryPoints:
          - src/handlers/create-user.ts
        Bundle: true
        Minify: false
        Target: es2022

  GetUserFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: .aws-sam/build/GetUserFunction/
      Handler: get-user.handler
      Events:
        GetUser:
          Type: HttpApi
          Properties:
            Path: /users/{userId}
            Method: GET
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref UsersTable
    Metadata:
      BuildMethod: esbuild
      BuildProperties:
        EntryPoints:
          - src/handlers/get-user.ts
        Bundle: true
        Minify: false
        Target: es2022

  UsersTable:
    Type: AWS::DynamoDB::Table
    Properties:
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: PK
          AttributeType: S
        - AttributeName: SK
          AttributeType: S
      KeySchema:
        - AttributeName: PK
          KeyType: HASH
        - AttributeName: SK
          KeyType: RANGE

Outputs:
  ApiEndpoint:
    Description: HTTP API endpoint URL
    Value: !Sub 'https://${ServerlessHttpApi}.execute-api.${AWS::Region}.amazonaws.com'
```

Key points in the template:

- `Transform: AWS::Serverless-2016-10-31` — enables SAM's simplified syntax on top of CloudFormation
- `BuildMethod: esbuild` — SAM uses esbuild to bundle each TypeScript handler into a single JS file for deployment, no separate compile step needed
- `DynamoDBCrudPolicy` — SAM's predefined policy connector grants the Lambda function the necessary DynamoDB permissions automatically
- `BillingMode: PAY_PER_REQUEST` — DynamoDB on-demand pricing: you pay per operation, not per provisioned capacity

### Building and Running Locally

```bash
# Build all functions (transpiles TypeScript via esbuild)
sam build

# Invoke a single function locally with a test event
sam local invoke CreateUserFunction --event events/create-user.json

# Start the full API locally (requires Docker)
sam local start-api
# → API now running at http://localhost:3000

# Test the local endpoint
curl -X POST http://localhost:3000/users \
  -H 'Content-Type: application/json' \
  -d '{"name": "Alice", "email": "alice@example.com"}'
```

Create test event files in an `events/` directory:

```json
// events/create-user.json
{
  "version": "2.0",
  "routeKey": "POST /users",
  "rawPath": "/users",
  "body": "{\"name\": \"Alice\", \"email\": \"alice@example.com\"}",
  "headers": {
    "content-type": "application/json"
  },
  "requestContext": {
    "http": {
      "method": "POST",
      "path": "/users"
    }
  }
}
```

### Deploying to AWS

```bash
# First deployment — walks through configuration
sam deploy --guided
# This creates samconfig.toml with your choices

# Subsequent deployments — uses saved config
sam deploy
```

`sam deploy` packages your built functions, uploads them to S3, and runs a CloudFormation stack update. The `Outputs` section of the template prints your live API endpoint when the deploy completes.

---

## Linting with Oxlint

[Oxlint](https://oxc.rs/docs/guide/usage/linter) ([GitHub](https://github.com/oxc-project/oxc)) is a natural fit for Lambda projects because of its speed. Lambda functions tend to have many small files, and a fast linter keeps feedback loops tight.

### Installation

```bash
pnpm add -D oxlint
```

### Running Oxlint

```bash
# Lint all source files
pnpm oxlint src/

# With specific rule categories enabled
pnpm oxlint --deny-warnings src/

# Auto-fix where possible
pnpm oxlint --fix src/
```

### package.json scripts

```json
{
  "scripts": {
    "lint": "oxlint src/",
    "lint:fix": "oxlint --fix src/"
  }
}
```

### oxlint.json (optional configuration)

Oxlint works zero-config but accepts an `oxlint.json` for customization:

```json
{
  "rules": {
    "no-unused-vars": "error",
    "no-console": "warn",
    "eqeqeq": "error"
  },
  "ignore": ["dist/**", ".aws-sam/**"]
}
```

Oxlint catches the most common correctness issues — unused variables, unreachable code, debugger statements, implicit any — without the configuration overhead of ESLint. For TypeScript-specific type-aware rules, you can add ESLint with `typescript-eslint` alongside Oxlint, using Oxlint as the fast first pass (see [Chapter 5](./05-linting.md) for details).

---

## Formatting with oxfmt

**oxfmt** is the Oxc project's TypeScript/JavaScript formatter — the formatting counterpart to Oxlint. As of early 2025, oxfmt is in early development and not yet production-ready, but it is part of the broader Oxc tooling ecosystem that also includes Oxlint.

While oxfmt matures, use **Biome** for formatting in Lambda projects. Biome is already production-ready, formats TypeScript identically to how Prettier does, and is written in Rust — making it extremely fast:

```bash
pnpm add -D @biomejs/biome
pnpm biome init
```

A `biome.json` suitable for a Lambda project:

```json
{
  "$schema": "https://biomejs.dev/schemas/1.9.0/schema.json",
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "linter": {
    "enabled": false
  },
  "files": {
    "ignore": ["dist/**", ".aws-sam/**", "node_modules/**"]
  }
}
```

Linting is handled by Oxlint; Biome handles formatting only — this avoids duplicating lint rules across two tools.

Add to `package.json`:

```json
{
  "scripts": {
    "format": "biome format --write src/",
    "format:check": "biome format src/"
  }
}
```

> **When oxfmt is stable:** Replace `@biomejs/biome` with the Oxc formatter. The command will likely be `oxfmt src/` — a single consistent tool for both linting and formatting in the Oxc ecosystem. Watch the [Oxc repository](https://github.com/oxc-project/oxc) for the stable release.

---

## Testing Lambda Handlers

Lambda handlers are just TypeScript functions — they are easy to unit test with Vitest. You do not need Docker or the SAM CLI to run your tests.

### Setup

```bash
pnpm add -D vitest
```

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
    },
  },
});
```

### Writing Handler Tests

```typescript
// src/handlers/create-user.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import type { APIGatewayProxyEventV2, Context } from 'aws-lambda';
import { handler } from './create-user';

// Mock the DynamoDB module so tests don't need a real database
vi.mock('../lib/db', () => ({
  ddb: {
    send: vi.fn().mockResolvedValue({}),
  },
}));

function makeEvent(body: unknown): APIGatewayProxyEventV2 {
  return {
    version: '2.0',
    routeKey: 'POST /users',
    rawPath: '/users',
    rawQueryString: '',
    body: JSON.stringify(body),
    headers: { 'content-type': 'application/json' },
    requestContext: {
      http: { method: 'POST', path: '/users', protocol: 'HTTP/1.1', sourceIp: '', userAgent: '' },
      accountId: '',
      apiId: '',
      requestId: '',
      routeKey: 'POST /users',
      stage: '$default',
      time: '',
      timeEpoch: 0,
    },
    isBase64Encoded: false,
  };
}

const mockContext = {} as Context;

describe('createUser handler', () => {
  it('returns 201 with a valid body', async () => {
    const event = makeEvent({ name: 'Alice', email: 'alice@example.com' });
    const result = await handler(event, mockContext);
    expect(result.statusCode).toBe(201);
    const body = JSON.parse(result.body as string);
    expect(body.name).toBe('Alice');
    expect(body.id).toBeDefined();
  });

  it('returns 400 when body is missing', async () => {
    const event = makeEvent(null);
    // Simulate missing body
    const noBodyEvent = { ...event, body: undefined };
    const result = await handler(noBodyEvent as APIGatewayProxyEventV2, mockContext);
    expect(result.statusCode).toBe(400);
  });

  it('returns 400 when required fields are missing', async () => {
    const event = makeEvent({ name: 'Alice' }); // missing email
    const result = await handler(event, mockContext);
    expect(result.statusCode).toBe(400);
  });
});
```

Run tests:

```bash
pnpm vitest run           # run once
pnpm vitest               # watch mode
pnpm vitest run --coverage  # with coverage report
```

---

## CI/CD with GitHub Actions

Combine linting, testing, building, and deploying into an automated pipeline:

```yaml
# .github/workflows/lambda-ci.yml
name: Lambda CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'

      - uses: pnpm/action-setup@v4
        with:
          version: latest

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Type check
        run: pnpm tsc --noEmit

      - name: Lint
        run: pnpm lint

      - name: Format check
        run: pnpm format:check

      - name: Test
        run: pnpm vitest run --coverage

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'

    permissions:
      id-token: write   # required for OIDC authentication with AWS
      contents: read

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'

      - uses: pnpm/action-setup@v4
        with:
          version: latest

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_DEPLOY_ROLE_ARN }}
          aws-region: us-east-1

      - name: Install SAM CLI
        uses: aws-actions/setup-sam@v2

      - name: Build
        run: sam build

      - name: Deploy
        run: sam deploy --no-confirm-changeset --no-fail-on-empty-changeset
```

Key points:
- **OIDC authentication** (`id-token: write`) avoids storing long-lived AWS credentials as secrets. The GitHub Action assumes an IAM role configured to trust GitHub's OIDC provider.
- `--no-confirm-changeset` — deploys without prompting for confirmation (safe in CI since the `test` job already validates the build)
- The deploy job only runs on pushes to `main`, not on pull requests

---

## Complete package.json

```json
{
  "name": "my-lambda-app",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "build": "sam build",
    "typecheck": "tsc --noEmit",
    "lint": "oxlint src/",
    "lint:fix": "oxlint --fix src/",
    "format": "biome format --write src/",
    "format:check": "biome format src/",
    "test": "vitest",
    "test:run": "vitest run",
    "coverage": "vitest run --coverage",
    "local": "sam local start-api",
    "deploy": "sam deploy"
  },
  "devDependencies": {
    "@biomejs/biome": "^1.9.0",
    "@types/aws-lambda": "^8.10.0",
    "@types/node": "^22.0.0",
    "@vitest/coverage-v8": "^2.0.0",
    "esbuild": "^0.24.0",
    "oxlint": "^0.15.0",
    "typescript": "^5.7.0",
    "vitest": "^2.0.0"
  },
  "dependencies": {
    "@aws-sdk/client-dynamodb": "^3.0.0",
    "@aws-sdk/lib-dynamodb": "^3.0.0"
  }
}
```

---

## Summary

| Tool | Role in a Lambda project |
|------|--------------------------|
| TypeScript + `@types/aws-lambda` | Type-safe handler code and event objects |
| AWS SAM | Define infrastructure, build with esbuild, test locally, deploy to AWS |
| esbuild | Bundle each handler into a single JS file optimized for cold-start time |
| Oxlint | Fast Rust-based linting — catches correctness issues before deployment |
| Biome | Fast TypeScript formatting (oxfmt replacement until it stabilizes) |
| Vitest | Unit tests for handler logic — no AWS required |
| GitHub Actions + OIDC | Automated test, build, and deploy pipeline |

AWS Lambda and TypeScript are a natural pair: short-lived, event-driven functions benefit enormously from the compile-time safety that TypeScript provides. Combined with SAM's local development tools, Oxlint's fast feedback, and Vitest's test runner, you get a professional serverless development workflow that feels as productive as working on a traditional Node.js server.

---

[← Previous: Using This Setup for JavaScript Projects](./15-javascript.md) | [Back to README](./README.md)
