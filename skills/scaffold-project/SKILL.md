---
name: scaffold-project
description: Use when creating a new project in the monorepo. Interviews the user, scaffolds project structure following exemplar patterns, and integrates with CI/docs/health checks.
argument-hint: [project-name]
---

# Scaffold a New Project

> **PRE-REQUISITE:** Before proceeding, invoke `superpowers:brainstorming` (Claude Code) or equivalent discovery process to explore the user's requirements. Use the discovery questions below as your agenda. If brainstorming is unavailable, ask the questions directly.

## Phase 1: Discovery

Gather answers to ALL of these before writing any code:

1. **Project name** — Must be kebab-case. Verify no directory with this name exists at the repo root.
2. **Project type** — One of:
   - `react-vite` — React + Vite + TypeScript (exemplar: `web-react/`)
   - `nextjs` — Next.js App Router + TypeScript (exemplar: `web-next/`)
   - `dotnet-api` — .NET Minimal API + Aspire (exemplar: `api/`)
   - `expo-mobile` — Expo React Native (exemplar: `mobile/`)
   - `python-cli` — Python CLI with Typer (exemplar: `tools/`)
   - `sst-aws` — SST v3 + Next.js + AWS (exemplar: `cloud-todo/`)
   - `custom` — User specifies stack; follow general repo conventions
3. **Purpose** — One-line description of what the project does.
4. **API connection** — Does it connect to the existing Todo API on port 5000, or is it self-contained?
5. **Key dependencies** — Any specific libraries, auth providers, databases, etc.
6. **Dev server port** — Must not conflict with existing ports: 5000 (api), 5173 (web-react), 3000 (web-next), 8081 (mobile).

**Decision gate:** Confirm all answers with the user before proceeding to Phase 2.

---

## Phase 2: Scaffold Project Structure

### Step 1: Create project directory

Create `{name}/` at the repository root.

### Step 2: Create agent configuration files

Create **both** of the following so the project works with multiple AI coding agents:

**`{name}/AGENTS.md`** (for Codex and other agents that read AGENTS.md):

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
- {List 4-6 project-specific conventions matching the exemplar's style}

## Skills
Project-specific skills are in `.agents/skills/`:
- {list any project skills created in Step 3}
```

**`{name}/CLAUDE.md`** (for Claude Code):

```markdown
# {Project Title}

## Build & Test
{install, dev, build, test, lint commands as bash code blocks}

## Architecture
{Brief description of tech stack, key patterns, and data flow}

## Conventions
{Same conventions as AGENTS.md — keep in sync}
```

Read the exemplar's agent config for tone and detail:
- `react-vite`: Read `web-react/AGENTS.md`
- `nextjs`: Read `web-next/AGENTS.md`
- `dotnet-api`: Read `api/AGENTS.md`
- `expo-mobile`: Read `mobile/AGENTS.md`
- `python-cli`: Read `tools/AGENTS.md`
- `sst-aws`: Read `cloud-todo/AGENTS.md`

### Step 3: Create starter skill and tool bridges

Create `{name}/.agents/skills/` with at least one skill relevant to the project type.

Read the exemplar's skills for reference:
- `react-vite`: Read `web-react/.agents/skills/add-component/SKILL.md`
- `nextjs`: Read `web-next/.agents/skills/add-route/SKILL.md`
- `dotnet-api`: Read `api/.agents/skills/add-endpoint/SKILL.md`
- `expo-mobile`: Read `mobile/.agents/skills/add-screen/SKILL.md`
- `python-cli`: Read `tools/.agents/skills/add-command/SKILL.md`
- `sst-aws`: Adapt from `web-next` skills

Each SKILL.md must have frontmatter with `name` and `description` fields to pass `agentic-tools skills validate`.

**Create tool-specific bridges** so the skill is discoverable by all agents:

1. **Claude Code**: Create `{name}/.claude/skills/{skill-name}` as a symlink to `../../.agents/skills/{skill-name}`
2. **Cursor**: Create `{name}/.cursor/rules/{skill-name}.mdc` with Cursor frontmatter (`description`, `globs`, `alwaysApply: false`) and the skill content
3. **Codex**: Reference the skill in `{name}/AGENTS.md` under the Skills section
4. **Copilot**: The root `.github/copilot-instructions.md` references skills automatically

### Step 4: Create project configuration

Read the exemplar's config files and create analogous ones for the new project.

#### react-vite

Read and adapt from `web-react/`:
- `package.json` — Update name, description, port in dev script
- `tsconfig.json` — Copy, adjust `paths` alias to `{name}/src/*`
- `vite.config.ts` — Copy, update port
- `eslint.config.mjs` — Copy as-is
- Create `src/` with: `components/`, `pages/`, `hooks/`, `api/`, `types/`, `__tests__/`
- Create `src/main.tsx` and `src/App.tsx` with minimal starter content
- Create `index.html` at project root

#### nextjs

Read and adapt from `web-next/`:
- `package.json` — Update name, description
- `tsconfig.json` — Copy Next.js TypeScript config
- `next.config.ts` — Minimal config
- `jest.config.ts` — Copy test setup with `@/` path mapping
- Create `src/app/` with `layout.tsx`, `page.tsx`
- Create `src/lib/` for utilities
- If using a database: set up Prisma (`prisma/schema.prisma`)

#### dotnet-api

Read and adapt from `api/`:
- Create `{PascalName}/{PascalName}.csproj` with .NET 9, nullable enabled
- Create `{PascalName}/Program.cs` with minimal API pattern and a health endpoint
- Create `{PascalName}/appsettings.json`
- Create `{PascalName}.Tests/{PascalName}Tests.csproj` with xUnit + WebApplicationFactory
- Create `{PascalName}.sln` referencing both projects
- If using Aspire: create `AppHost/` and `ServiceDefaults/` following `api/` structure

#### expo-mobile

Read and adapt from `mobile/`:
- `package.json` — Update name, add expo dependencies
- `app.config.ts` — Configure app name, slug, scheme
- `tsconfig.json` — Expo TypeScript config
- Create `app/` with Expo Router structure: `_layout.tsx`, `index.tsx`
- Create `components/`, `hooks/`, `utils/` directories
- Create `jest.config.ts` with `jest-expo` preset

#### python-cli

Read and adapt from `tools/`:
- `pyproject.toml` — Update name, entry point, dependencies, ruff/mypy config
- Create `src/{package_name}/` with `__init__.py`, `cli.py`
- Create `tests/` with a starter test file
- Entry point uses Typer CLI framework

#### sst-aws

Read and adapt from `cloud-todo/`:
- `package.json` — Update name, add sst and aws-sdk dependencies
- `sst.config.ts` — Minimal orchestrator importing from `infra/`
- Create `infra/` directory for infrastructure definitions
- Create `src/app/` with Next.js App Router structure
- Create `src/lib/` for AWS client utilities

#### custom

No exemplar — use the user's specified tech stack. Still create:
- Project config file appropriate for the stack
- `src/` directory with sensible structure
- Test setup with at least one passing test
- Lint configuration

### Step 5: Create README.md

Create `{name}/README.md` with:
- Project title and description
- Prerequisites
- Getting started commands (install, dev, test)
- Project structure overview

---

## Phase 3: Monorepo Integration

### Step 6: Add CI job

Edit `.github/workflows/ci.yml` — add a new job before the `build` job.

**For Node.js projects**, follow this pattern:
```yaml
  test-{name}:
    name: Test {name}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
          cache-dependency-path: {name}/package-lock.json

      - name: Install
        working-directory: {name}
        run: npm ci

      - name: Lint
        working-directory: {name}
        run: npm run lint

      - name: Test
        working-directory: {name}
        run: npm test
```

**For .NET projects**, follow the `test-api` job pattern in the CI file.

**For Python projects**, follow the `test-tools` job pattern in the CI file.

### Step 7: Register health check

Edit `tools/src/agentic_tools/commands/health.py` — add an entry to the `PROJECT_CHECKS` dict:

```python
PROJECT_CHECKS: dict[str, list[str]] = {
    # ... existing entries ...
    "{name}": ["{appropriate build/check command}"],
}
```

Common check commands by type:
- Node.js: `["npm", "run", "build", "--if-present"]`
- .NET: `["dotnet", "build", "--no-restore"]`
- Python: `["python", "-m", "py_compile", "src/{package}/cli.py"]`
- Expo: `["npx", "expo", "doctor"]`

### Step 8: Update root documentation

**Edit `CLAUDE.md`** (root) — add:
1. Repository Overview (~line 6) — `- **{name}/** — {tech stack} ({purpose}, port {port})`
2. Build & Test Commands — new `### {name}` section with commands
3. Key patterns per project — bullet describing project patterns
4. Data flow — note API connection or self-contained status

**Edit `AGENTS.md`** (root) — add:
1. Project Overview (~line 5) — `- **{name}/** — {tech stack} ({purpose})`

**Edit `README.md`** (root) — add row to the Projects table.

---

## Phase 4: Verification

Run these checks and confirm each passes:

- [ ] `ls {name}/` — directory exists with expected structure
- [ ] `{name}/AGENTS.md` exists (Codex compatibility)
- [ ] `{name}/CLAUDE.md` exists (Claude Code compatibility)
- [ ] `{name}/.agents/skills/` contains at least one SKILL.md
- [ ] `{name}/.claude/skills/` symlinks resolve to `.agents/skills/`
- [ ] `{name}/.cursor/rules/` contains matching `.mdc` files
- [ ] `agentic-tools validate` — all AGENTS.md files pass
- [ ] `agentic-tools skills validate` — all SKILL.md files pass
- [ ] Install dependencies and run the dev server — starts on chosen port
- [ ] Run the test command — tests pass (even if just a placeholder)
- [ ] Run the lint command — no errors
- [ ] `agentic-tools health` — new project passes health check
- [ ] Review CI diff — new job follows existing patterns
- [ ] Review root `CLAUDE.md` diff — new project documented
- [ ] Review root `AGENTS.md` diff — new project documented
