---
name: scaffold-project
description: Use when creating a new project from scratch. Interviews the user, scaffolds a greenfield project with opinionated defaults, and sets up agentic coding configuration (CLAUDE.md, AGENTS.md, skills) so AI coding agents can work effectively from day one.
argument-hint: [project-name]
---

# Scaffold a New Project

> **PRE-REQUISITE:** Before proceeding, invoke `superpowers:brainstorming` (Claude Code) or equivalent discovery process to explore the user's requirements. Use the discovery questions below as your agenda. If brainstorming is unavailable, ask the questions directly.

## Phase 1: Discovery

Gather answers to ALL of these before writing any code:

1. **Project name** — Must be kebab-case.
2. **Project type** — One of:
   - `react-vite` — React + Vite + TypeScript
   - `nextjs` — Next.js App Router + TypeScript
   - `dotnet-api` — .NET Minimal API (optionally with Aspire)
   - `expo-mobile` — Expo React Native
   - `python-cli` — Python CLI with Typer
   - `sst-aws` — SST v3 + Next.js + AWS
   - `custom` — User specifies stack
3. **Purpose** — One-line description of what the project does.
4. **Key dependencies** — Any specific libraries, auth providers, databases, etc.
5. **Dev server port** — If applicable.

**Decision gate:** Confirm all answers with the user before proceeding to Phase 2.

---

## Phase 2: Scaffold Project Structure

### Step 1: Initialise the project

Create the project directory and initialise git:

```bash
mkdir {name} && cd {name}
git init
```

### Step 2: Create agent configuration files

Create **both** of the following so the project works with multiple AI coding agents:

**`AGENTS.md`** (for Codex and other agents that read AGENTS.md):

```markdown
# {Project Title} - {Tech Stack Summary}

## Overview
{One-line description from discovery}.

## Commands
- `{install command}` — Install dependencies
- `{dev command}` — Start development server
- `{build command}` — Production build
- `{test command}` — Run tests
- `{lint command}` — Lint source files

## Project Structure
- `src/` — Source code
  - {list key subdirectories and their purpose}

## Conventions
- {List 4-6 project-specific conventions from the relevant project type section below}

## Skills
Project-specific skills are in `.agents/skills/`:
- {list any project skills created in Step 3}
```

**`CLAUDE.md`** (for Claude Code):

```markdown
# {Project Title}

## Build & Test
{install, dev, build, test, lint commands as bash code blocks}

## Architecture
{Brief description of tech stack, key patterns, and data flow}

## Conventions
{Same conventions as AGENTS.md — keep in sync}
```

### Step 3: Create starter skill and tool bridges

Create `.agents/skills/` with at least one skill relevant to the project type. Good starter skills by type:

- **react-vite:** `add-component` — how to create a reusable component (functional, default export, co-located props interface, hooks for state)
- **nextjs:** `add-route` — how to add a new page route (server component by default, server actions for mutations, `'use client'` only when needed)
- **dotnet-api:** `add-endpoint` — how to add a new minimal API endpoint (MapGet/Post/Put/Delete in Program.cs, record DTOs, no controllers)
- **expo-mobile:** `add-screen` — how to add a new screen (Expo Router file-based routing, SafeAreaView, FlatList patterns)
- **python-cli:** `add-command` — how to add a new Typer subcommand with Rich formatting
- **sst-aws:** `add-route` — how to add a new page with DynamoDB access and Cognito auth
- **custom:** Generate a skill appropriate for the primary unit of work in the chosen stack

Each SKILL.md must have YAML frontmatter with at least `name` and `description` fields.

**Create tool-specific bridges** so the skill is discoverable by all agents:

1. **Claude Code**: Create `.claude/skills/{skill-name}` as a symlink to `../../.agents/skills/{skill-name}`
2. **Cursor**: Create `.cursor/rules/{skill-name}.mdc` with Cursor frontmatter (`description`, `globs`, `alwaysApply: false`) and the skill content
3. **Codex**: Reference the skill in `AGENTS.md` under the Skills section

### Step 4: Create project configuration

Generate the appropriate configuration files for the chosen project type. Follow the specific patterns below — these are opinionated defaults that encode our way of building things.

---

#### react-vite

**Dependencies:**
- react ^18, react-dom ^18, react-router-dom ^6
- TanStack Query ^5 for all server state (never use local state for API data)
- Axios for HTTP client
- TypeScript ^5 with strict mode

**Dev dependencies:**
- vite ^5, @vitejs/plugin-react ^4
- vitest ^1, @testing-library/react ^14, jsdom ^24, @testing-library/jest-dom
- eslint ^8, @typescript-eslint/parser, @typescript-eslint/eslint-plugin, eslint-plugin-react-hooks, eslint-plugin-react-refresh
- prettier ^3

**TypeScript config (`tsconfig.json`):**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  },
  "include": ["src"]
}
```

**Vite config (`vite.config.ts`):**
```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: { '@': '/src' },
  },
  server: {
    port: {port},
  },
  test: {
    environment: 'jsdom',
    setupFiles: 'src/__tests__/setup.ts',
  },
});
```

**ESLint config (`eslint.config.mjs`):**
```javascript
module.exports = {
  root: true,
  env: { browser: true, es2020: true },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
  ],
  parser: '@typescript-eslint/parser',
  plugins: ['react-refresh'],
  rules: {
    'react-refresh/only-export-components': ['warn', { allowConstantExport: true }],
  },
};
```

**Test setup (`src/__tests__/setup.ts`):**
```typescript
import '@testing-library/jest-dom';
```

**Directory structure:**
```
src/
  components/    — Reusable UI components (default exports, functional + hooks only)
  pages/         — Page-level route components
  hooks/         — Custom React hooks (TanStack Query wrappers)
  api/           — Axios-based API client functions
  types/         — TypeScript type definitions
  __tests__/     — Test files + setup.ts
  main.tsx       — App entry point
  App.tsx        — Root component with router + QueryClientProvider
index.html       — HTML entry point
```

**Code patterns:**
- Functional components with hooks only, default exports
- Props as inline interfaces co-located in the component file: `interface TodoItemProps { todo: Todo; }`
- TanStack Query for ALL server state — query keys like `['todos']` and `['todos', id]`
- Mutations invalidate query keys on success
- Conditional queries with `enabled: !!id`
- Axios client instance with baseURL: `const client = axios.create({ baseURL: '{apiUrl}' });`
- API functions return typed promises: `export async function getTodos(): Promise<Todo[]>`

**Build scripts:**
```json
{
  "dev": "vite",
  "build": "tsc && vite build",
  "test": "vitest",
  "lint": "eslint src --ext .ts,.tsx"
}
```

---

#### nextjs

**Dependencies:**
- next ^15, react ^19, react-dom ^19
- @prisma/client ^6 (if using a database)
- Tailwind CSS ^3
- TypeScript ^5

**Dev dependencies:**
- prisma ^6 (if using a database)
- jest ^29, ts-jest ^29, @testing-library/react ^16, @testing-library/jest-dom
- eslint ^9, eslint-config-next ^15

**TypeScript config (`tsconfig.json`):**
```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "module": "esnext",
    "moduleResolution": "bundler",
    "jsx": "preserve",
    "strict": true,
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

**Jest config (`jest.config.ts`):**
```typescript
import type { Config } from 'jest';
const config: Config = {
  testEnvironment: 'jsdom',
  setupFiles: ['<rootDir>/jest.setup.ts'],
  moduleNameMapper: { '^@/(.*)$': '<rootDir>/src/$1' },
  testPathIgnorePatterns: ['<rootDir>/.next/', '<rootDir>/node_modules/'],
};
export default config;
```

**Prisma setup (if using a database):**

Schema at `prisma/schema.prisma`:
```prisma
generator client {
  provider = "prisma-client-js"
}
datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}
```

Prisma client singleton at `src/lib/prisma.ts`:
```typescript
import { PrismaClient } from '@prisma/client';
const globalForPrisma = globalThis as unknown as { prisma: PrismaClient | undefined };
export const prisma = globalForPrisma.prisma ?? new PrismaClient();
if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

**Directory structure:**
```
src/
  app/
    layout.tsx     — Root layout
    page.tsx       — Home page
    {resource}/
      page.tsx     — List page (async server component)
      [id]/
        page.tsx   — Detail page (async server component)
        edit/
          page.tsx — Edit page ('use client')
  lib/
    prisma.ts      — Prisma client singleton
    actions.ts     — Server actions for mutations
prisma/
  schema.prisma    — Database schema
  migrations/      — Prisma migrations
```

**Code patterns:**
- Server components by default — no `'use client'` unless the component needs interactivity
- Pages are async functions that fetch data directly:
  ```typescript
  export default async function ResourcePage({ params }: { params: Promise<{ id: string }> }) {
    const { id } = await params;
    const resource = await prisma.resource.findUnique({ where: { id: parseInt(id, 10) } });
  }
  ```
- **IMPORTANT:** Always `await params` in dynamic routes (Next.js 15+ requirement)
- Server actions in `src/lib/actions.ts` with `"use server"` directive:
  ```typescript
  "use server";
  import { revalidatePath } from "next/cache";
  import { redirect } from "next/navigation";
  export async function createResource(formData: FormData) {
    const title = formData.get("title") as string;
    await prisma.resource.create({ data: { title } });
    revalidatePath("/resources");
    redirect("/resources");
  }
  ```
- Tailwind CSS for styling — utility classes, no CSS modules

**Build scripts:**
```json
{
  "dev": "next dev",
  "build": "next build",
  "start": "next start",
  "test": "jest",
  "lint": "next lint"
}
```

---

#### dotnet-api

**Project setup:**
- Target framework: net9.0
- Nullable reference types: enabled
- Implicit usings: enabled

**NuGet packages (main project):**
- Microsoft.AspNetCore.OpenApi
- Microsoft.EntityFrameworkCore
- Microsoft.EntityFrameworkCore.SqlServer (or .Sqlite for simple projects)
- Aspire packages if using orchestration:
  - Aspire.Microsoft.EntityFrameworkCore.SqlServer

**NuGet packages (test project):**
- Microsoft.AspNetCore.Mvc.Testing (WebApplicationFactory)
- Microsoft.EntityFrameworkCore.InMemory (test isolation)
- Microsoft.NET.Test.Sdk
- xunit, xunit.runner.visualstudio

**API pattern — Minimal APIs (NEVER controllers):**

All routes defined in `Program.cs` using `MapGet`, `MapPost`, `MapPut`, `MapDelete`:
```csharp
app.MapGet("/resources", async (AppDb db) => {
    var items = await db.Resources.OrderByDescending(r => r.CreatedAt).ToListAsync();
    return Results.Ok(items.Select(ResourceResponse.FromEntity));
});

app.MapPost("/resources", async (CreateResourceRequest request, AppDb db) => {
    var entity = new Resource { Title = request.Title };
    db.Resources.Add(entity);
    await db.SaveChangesAsync();
    return Results.Created($"/resources/{entity.Id}", ResourceResponse.FromEntity(entity));
});

app.MapPut("/resources/{id}", async (int id, UpdateResourceRequest request, AppDb db) => {
    var entity = await db.Resources.FindAsync(id);
    if (entity is null) return Results.NotFound();
    // update fields...
    await db.SaveChangesAsync();
    return Results.Ok(ResourceResponse.FromEntity(entity));
});

app.MapDelete("/resources/{id}", async (int id, AppDb db) => {
    var entity = await db.Resources.FindAsync(id);
    if (entity is null) return Results.NotFound();
    db.Resources.Remove(entity);
    await db.SaveChangesAsync();
    return Results.NoContent();
});
```

**DTO pattern — Records (never classes for DTOs):**
```csharp
public record CreateResourceRequest(string Title, string? Description = null);
public record UpdateResourceRequest(string? Title = null, string? Description = null);
public record ResourceResponse(int Id, string Title, string? Description, DateTime CreatedAt, DateTime UpdatedAt)
{
    public static ResourceResponse FromEntity(Resource entity) =>
        new(entity.Id, entity.Title, entity.Description, entity.CreatedAt, entity.UpdatedAt);
}
```

**Entity model:**
```csharp
public class Resource
{
    public int Id { get; set; }
    [Required] public string Title { get; set; } = string.Empty;
    public string? Description { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime UpdatedAt { get; set; } = DateTime.UtcNow;
}
```

**DbContext:**
```csharp
public class AppDb : DbContext
{
    public AppDb(DbContextOptions<AppDb> options) : base(options) { }
    public DbSet<Resource> Resources => Set<Resource>();
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Resource>(entity => {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Title).IsRequired().HasMaxLength(200);
        });
    }
}
```

**Service registration (Program.cs):**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Aspire (if using):
if (!builder.Environment.IsEnvironment("Testing"))
{
    builder.AddServiceDefaults();
    builder.AddSqlServerDbContext<AppDb>("appdb");
}

builder.Services.AddCors(options => {
    options.AddPolicy("AllowFrontend", policy => {
        policy.WithOrigins("http://localhost:{frontendPort}")
            .AllowAnyHeader().AllowAnyMethod();
    });
});

var app = builder.Build();
app.UseCors("AllowFrontend");

// Ensure DB created
using (var scope = app.Services.CreateScope()) {
    var db = scope.ServiceProvider.GetRequiredService<AppDb>();
    db.Database.EnsureCreated();
}
// ... map endpoints ...
app.Run();
```

**Testing pattern — WebApplicationFactory:**
```csharp
public class AppFactory : WebApplicationFactory<Program>
{
    protected override IHost CreateHost(IHostBuilder builder)
    {
        builder.UseEnvironment("Testing");
        builder.ConfigureServices(services => {
            // Remove real DB registrations
            var descriptors = services.Where(
                d => d.ServiceType == typeof(DbContextOptions<AppDb>) ||
                     d.ServiceType == typeof(AppDb)).ToList();
            foreach (var descriptor in descriptors)
                services.Remove(descriptor);
            // Replace with in-memory DB for test isolation
            services.AddDbContext<AppDb>(options => {
                options.UseInMemoryDatabase($"TestDb_{Guid.NewGuid()}");
            });
        });
        return base.CreateHost(builder);
    }
}

public class EndpointTests : IClassFixture<AppFactory>
{
    private readonly AppFactory _factory;
    public EndpointTests(AppFactory factory) => _factory = factory;
    private HttpClient CreateClient() => _factory.CreateClient();

    [Fact]
    public async Task GetResources_ReturnsEmptyList() {
        var client = CreateClient();
        var response = await client.GetAsync("/resources");
        response.EnsureSuccessStatusCode();
        var items = await response.Content.ReadFromJsonAsync<List<ResourceResponse>>();
        Assert.NotNull(items);
        Assert.Empty(items);
    }
}
```

**Project structure (if using Aspire):**
```
{PascalName}.sln
AppHost/              — Aspire orchestration (service dependencies)
ServiceDefaults/      — Shared config (OpenTelemetry, health checks, resilience)
{PascalName}/         — Main API project
  Program.cs          — Entry point + all endpoint definitions
  Models/             — Entity classes
  {PascalName}.csproj
{PascalName}.Tests/   — xUnit integration tests
  {PascalName}Tests.csproj
```

**Project structure (without Aspire):**
```
{PascalName}.sln
{PascalName}/
  Program.cs
  Models/
  {PascalName}.csproj
{PascalName}.Tests/
  {PascalName}Tests.csproj
```

**Build commands:**
```bash
dotnet build
dotnet test
dotnet run --project {PascalName}    # API only
# With Aspire:
dotnet run --project AppHost         # Full orchestration + dashboard
```

---

#### expo-mobile

**Dependencies:**
- expo ~52, react-native 0.76, react 18.3
- expo-router ~4 (file-based navigation — NEVER use React Navigation directly)
- expo-constants, expo-status-bar, expo-linking
- react-native-safe-area-context, react-native-screens
- TanStack Query ^5 for server state, Axios ^1 for HTTP
- @react-native-async-storage/async-storage for offline/draft storage
- @expo/vector-icons for icons

**Dev dependencies:**
- typescript ~5.3
- jest ^29, jest-expo ~52, @testing-library/react-native ^12

**CRITICAL: Expo managed workflow — NEVER eject.**

**TypeScript config (`tsconfig.json`):**
```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "paths": { "@/*": ["./*"] }
  }
}
```

**App config (`app.config.ts`):**
```typescript
import { ConfigContext, ExpoConfig } from 'expo/config';
export default ({ config }: ConfigContext): ExpoConfig => ({
  ...config,
  name: "{Project Name}",
  slug: "{project-slug}",
  version: "1.0.0",
  orientation: "portrait",
  userInterfaceStyle: "automatic",
  newArchEnabled: true,
  extra: {
    apiBaseUrl: process.env.API_BASE_URL || "http://localhost:5000",
  },
  plugins: ["expo-router"],
});
```

**Directory structure:**
```
app/
  _layout.tsx           — Root layout with QueryClientProvider + SafeAreaProvider
  (tabs)/
    _layout.tsx         — Tab navigator layout
    index.tsx           — Primary list screen
    add.tsx             — Create form screen
    settings.tsx        — Settings screen
  {resource}/
    [id].tsx            — Detail/edit modal screen
api/                    — Axios-based API client
hooks/                  — TanStack Query hooks
components/             — Reusable UI components
utils/                  — AsyncStorage helpers, formatters
```

**Code patterns:**
- All screens use `SafeAreaView` from react-native-safe-area-context or `useSafeAreaInsets`
- Platform-specific styling via `Platform.select()`
- Input screens wrap in `KeyboardAvoidingView`
- Lists use `FlatList` with pull-to-refresh (`refreshing` + `onRefresh` props)
- Auto-save drafts to AsyncStorage for offline resilience
- API base URL from `app.config.ts` extras, configurable at runtime via Settings screen

**Screen example:**
```typescript
export default function ListScreen() {
  const router = useRouter();
  const { data, isLoading, error, refetch, isRefetching } = useResources();
  const deleteMutation = useDeleteResource();

  return (
    <SafeAreaView style={styles.container} edges={['bottom']}>
      <FlatList
        data={data}
        renderItem={({ item }) => <ResourceItem item={item} />}
        keyExtractor={(item) => String(item.id)}
        refreshing={isRefetching}
        onRefresh={refetch}
        ListEmptyComponent={<EmptyState />}
      />
    </SafeAreaView>
  );
}
```

**TanStack Query hooks:**
```typescript
export function useResources() {
  return useQuery<Resource[]>({
    queryKey: ['resources'],
    queryFn: getResources,
  });
}

export function useDeleteResource() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (id: number) => deleteResource(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['resources'] });
    },
  });
}
```

**Jest config (in `package.json`):**
```json
{
  "jest": {
    "preset": "jest-expo",
    "transformIgnorePatterns": [
      "node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@sentry/react-native|native-base|react-native-svg)"
    ]
  }
}
```

**Build scripts:**
```json
{
  "start": "expo start",
  "test": "jest",
  "lint": "eslint .",
  "typecheck": "tsc --noEmit"
}
```

---

#### python-cli

**Dependencies:**
- Python >= 3.11
- typer[all] (CLI framework)
- rich (terminal formatting)

**Dev dependencies:**
- pytest
- ruff (linting + formatting)
- mypy (type checking)

**`pyproject.toml`:**
```toml
[project]
name = "{name}"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = ["typer[all]>=0.9", "rich>=13"]

[project.optional-dependencies]
dev = ["pytest>=7", "ruff>=0.4", "mypy>=1.10"]

[project.scripts]
{name} = "{package_name}.cli:app"

[tool.ruff]
target-version = "py311"
line-length = 88
[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "B"]

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_configs = true
```

**Directory structure:**
```
src/
  {package_name}/
    __init__.py
    cli.py           — Typer app with subcommands
    commands/
      __init__.py
      {command}.py   — One file per command group
    utils/
      __init__.py
tests/
  __init__.py
  test_{command}.py  — One test file per command
pyproject.toml
```

**CLI entry point (`src/{package_name}/cli.py`):**
```python
import typer
from rich.console import Console

app = typer.Typer(help="{Project description}")
console = Console()

@app.command()
def hello(name: str = typer.Argument("world", help="Name to greet")) -> None:
    """Say hello."""
    console.print(f"[green]Hello, {name}![/green]")

if __name__ == "__main__":
    app()
```

**Conventions:**
- Type hints on ALL functions
- Ruff for linting + import sorting
- Mypy strict mode
- One Typer command per public function
- Rich console for all terminal output

**Build commands:**
```bash
pip install -e ".[dev]"
{name} --help              # Run CLI
pytest                      # Tests
ruff check .                # Lint
mypy src/                   # Type check
```

---

#### sst-aws

**Dependencies:**
- next ^16, react ^19, react-dom ^19
- @aws-sdk/client-dynamodb ^3, @aws-sdk/lib-dynamodb ^3
- aws-jwt-verify ^4 (for Cognito JWT validation)
- Tailwind CSS ^4, @tailwindcss/postcss ^4
- ulidx ^2 (sortable distributed IDs)
- TypeScript ^5

**Dev dependencies:**
- sst (latest v3 Ion)
- jest ^29, ts-jest ^29, aws-sdk-client-mock ^4
- eslint, eslint-config-next

**TypeScript config (`tsconfig.json`):**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["dom", "dom.iterable", "esnext"],
    "module": "esnext",
    "moduleResolution": "bundler",
    "jsx": "preserve",
    "strict": true,
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["sst-env.d.ts", "**/*.ts", "**/*.tsx"]
}
```

**Jest config (`jest.config.ts`):**
```typescript
import type { Config } from 'jest';
const config: Config = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  moduleNameMapper: { '^@/(.*)$': '<rootDir>/src/$1' },
  setupFilesAfterSetup: ['<rootDir>/jest.setup.ts'],
  testPathIgnorePatterns: ['/node_modules/', '/.next/'],
};
export default config;
```

**SST config (`sst.config.ts`):**
```typescript
export default $config({
  app(input) {
    return {
      name: "{name}",
      home: "aws",
      removal: input?.stage === "production" ? "retain" : "remove",
    };
  },
  async run() {
    const { table } = await import("./infra/database");
    const { userPool } = await import("./infra/auth");
    const { site } = await import("./infra/web");
    return { url: site.url, userPoolId: userPool.id, tableName: table.name };
  },
});
```

**Infrastructure — split across `infra/` modules:**

`infra/database.ts` — DynamoDB single-table design:
```typescript
export const table = new sst.aws.Dynamo("{PascalName}Table", {
  fields: { pk: "string", sk: "string" },
  primaryIndex: { hashKey: "pk", rangeKey: "sk" },
});
```

`infra/auth.ts` — Cognito with OAuth2 code flow (uses raw @pulumi/aws resources since SST v3 has no built-in Cognito component):
```typescript
import * as aws from "@pulumi/aws";

export const userPool = new aws.cognito.UserPool("{PascalName}UserPool", {
  autoVerifiedAttributes: ["email"],
  usernameAttributes: ["email"],
  passwordPolicy: { minimumLength: 8, requireLowercase: true, requireNumbers: true, requireSymbols: true, requireUppercase: true },
});

export const userPoolClient = new aws.cognito.UserPoolClient("{PascalName}UserPoolClient", {
  userPoolId: userPool.id,
  allowedOauthFlows: ["code"],
  allowedOauthScopes: ["email", "openid", "profile"],
  callbackUrls: ["http://localhost:3000/auth/callback"],
  logoutUrls: ["http://localhost:3000"],
  supportedIdentityProviders: ["COGNITO"],
});
```

`infra/web.ts` — Next.js deployed to AWS Lambda:
```typescript
export const site = new sst.aws.Nextjs("{PascalName}Site", {
  path: "./",
  link: [table],
  environment: {
    NEXT_PUBLIC_COGNITO_USER_POOL_ID: userPool.id,
    NEXT_PUBLIC_COGNITO_CLIENT_ID: userPoolClient.id,
    NEXT_PUBLIC_COGNITO_DOMAIN: $interpolate`https://${userPoolDomain.domain}.auth.${region.name}.amazoncognito.com`,
  },
});
```

**Directory structure:**
```
infra/
  database.ts       — DynamoDB table definition
  auth.ts           — Cognito User Pool + Client (raw Pulumi AWS resources)
  web.ts            — sst.aws.Nextjs site
src/
  app/              — Next.js App Router pages
  lib/
    dynamo.ts       — DynamoDB DocumentClient singleton + table name resolution
    auth.ts         — Cognito URL builders + JWT token extraction from cookies
    {resource}.ts   — CRUD operations against DynamoDB
sst.config.ts       — SST orchestrator
sst-env.d.ts        — Auto-generated SST types
```

**DynamoDB patterns:**

Singleton DocumentClient (`src/lib/dynamo.ts`):
```typescript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({});
export const docClient = DynamoDBDocumentClient.from(client, {
  marshallOptions: { removeUndefinedValues: true },
});

export function getTableName(): string {
  try {
    const { Resource } = require("sst");
    return Resource.{PascalName}Table.name;
  } catch {
    return process.env.TABLE_NAME ?? "{PascalName}Table";
  }
}
```

Single-table access patterns (`pk=USER#<userId>`, `sk=RESOURCE#<resourceId>`):
```typescript
export async function listResources(userId: string): Promise<Resource[]> {
  const result = await docClient.send(new QueryCommand({
    TableName: getTableName(),
    KeyConditionExpression: "pk = :pk AND begins_with(sk, :sk)",
    ExpressionAttributeValues: { ":pk": `USER#${userId}`, ":sk": "RESOURCE#" },
  }));
  return (result.Items ?? []).map(mapItemToResource);
}

export async function createResource(userId: string, input: CreateInput): Promise<Resource> {
  const id = ulid();
  const now = new Date().toISOString();
  await docClient.send(new PutCommand({
    TableName: getTableName(),
    Item: { pk: `USER#${userId}`, sk: `RESOURCE#${id}`, id, ...input, createdAt: now, updatedAt: now },
  }));
  return { id, ...input, createdAt: now, updatedAt: now };
}
```

**Auth patterns:**
- JWT tokens stored in HTTP-only cookies
- Extract userId from `sub` claim: `payload.sub`
- Server components check auth via `cookies()` → extract token → redirect if missing
- Build Cognito login/logout URLs with `URLSearchParams`

**Testing with aws-sdk-client-mock:**
```typescript
import { mockClient } from "aws-sdk-client-mock";
import { DynamoDBDocumentClient, QueryCommand } from "@aws-sdk/lib-dynamodb";

const ddbMock = mockClient(DynamoDBDocumentClient);

beforeEach(() => ddbMock.reset());

test("listResources returns items", async () => {
  ddbMock.on(QueryCommand).resolves({ Items: [/* mock items */] });
  const result = await listResources("user-123");
  expect(result).toHaveLength(1);
});
```

**Build scripts:**
```json
{
  "dev": "next dev",
  "build": "next build",
  "test": "jest",
  "lint": "next lint"
}
```

**SST commands:**
```bash
npx sst dev                     # Live dev with personal AWS stage
npx sst deploy --stage prod     # Production deploy
npx sst remove                  # Tear down resources
```

---

#### custom

No template — use the user's specified tech stack. Still create:
- Project config file appropriate for the stack
- `src/` directory with sensible structure
- Test setup with at least one passing test
- Lint configuration
- Follow the general conventions: strict types, integration tests over unit tests, minimal abstractions

---

### Step 5: Create README.md

```markdown
# {Project Title}

{One-line description}

## Prerequisites

- {Runtime} {version}+
- {Package manager}

## Getting Started

{install command}
{dev command}

## Testing

{test command}

## Project Structure

{brief directory overview}
```

### Step 6: Create .gitignore

Create an appropriate `.gitignore` for the project type (node_modules, dist, .env, build artifacts, bin/obj, __pycache__, .next, etc.).

---

## Phase 3: Verification

Run these checks and confirm each passes:

- [ ] Project directory exists with expected structure
- [ ] `git init` was run and `.gitignore` is in place
- [ ] `AGENTS.md` exists with accurate commands and conventions
- [ ] `CLAUDE.md` exists with accurate commands and architecture
- [ ] `.agents/skills/` contains at least one SKILL.md with valid frontmatter (`name` + `description`)
- [ ] `.claude/skills/` symlinks resolve to `.agents/skills/`
- [ ] `.cursor/rules/` contains matching `.mdc` files
- [ ] Install dependencies succeeds
- [ ] Dev server starts on the chosen port (if applicable)
- [ ] Tests pass (even if just a placeholder)
- [ ] Lint passes with no errors
