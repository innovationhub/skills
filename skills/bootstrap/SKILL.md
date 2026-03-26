---
name: bootstrap
description: Set up agentic coding in an existing (brownfield) repository. Analyzes the codebase — languages, frameworks, architecture, build tools, test runners, CI/CD — then generates CLAUDE.md, AGENTS.md, and project-specific skills so AI coding agents can work effectively in the repo. Use this skill whenever a developer wants to add agentic tooling to a repo that doesn't have it yet, or when someone says "bootstrap", "set up agentic", "make this repo work with Claude/Codex/Copilot", or "add AI coding support".
argument-hint: [path-to-repo]
---

# Bootstrap Agentic Coding in a Brownfield Repo

This skill turns any existing repository into an agentic-friendly workspace. It explores the codebase, builds a mental model of the project, and generates the configuration files and skills that AI coding agents need to be effective.

The entire process is autonomous — no interview required. The skill infers everything it can from the code, git history, and project configuration, and produces sensible defaults.

## Phase 1: Codebase Discovery

Explore the repository systematically. The goal is to build a complete picture of the project before generating any files. Use parallel subagents or tool calls where possible to speed this up.

### 1.1 Project Structure

- List top-level directories and files to understand the repo layout
- Identify if it's a monorepo (multiple projects) or a single project
- For monorepos, identify each sub-project and its root directory

### 1.2 Languages & Frameworks

For each project (or the single project), detect:

- **Languages**: Look at file extensions, config files (`tsconfig.json`, `pyproject.toml`, `*.csproj`, `Cargo.toml`, `go.mod`, etc.)
- **Frameworks**: Look for framework-specific configs and imports (`next.config.*`, `vite.config.*`, `angular.json`, `django`, `flask`, `express`, `aspire`, `expo`, etc.)
- **Package managers**: `package-lock.json` (npm), `yarn.lock`, `pnpm-lock.yaml`, `uv.lock`, `Pipfile.lock`, `Gemfile.lock`, etc.

### 1.3 Build, Test & Dev Commands

Discover commands by reading:

- `package.json` scripts
- `Makefile` / `Justfile` / `Taskfile.yml` targets
- `pyproject.toml` scripts or tool configs
- `.csproj` / `.sln` files (implies `dotnet build`, `dotnet test`)
- `Cargo.toml` (implies `cargo build`, `cargo test`)
- `go.mod` (implies `go build`, `go test`)
- CI config files (`.github/workflows/*.yml`, `.gitlab-ci.yml`, `Jenkinsfile`, etc.) — these often reveal the canonical build/test/lint commands

### 1.4 Architecture & Data Flow

Go deeper to understand how the code is organized:

- **Entry points**: `main.*`, `index.*`, `Program.cs`, `app.*`, route definitions
- **API layer**: REST endpoints, GraphQL schemas, gRPC definitions
- **Data layer**: Database configs, ORMs (Prisma, EF Core, SQLAlchemy, Drizzle, etc.), migration directories
- **Key abstractions**: Services, repositories, controllers, middleware, hooks
- **External integrations**: API clients, message queues, cache layers, auth providers
- **Data flow**: How do requests flow through the system? What talks to what?

For monorepos, also map the relationships between projects (shared libraries, API consumers, etc.).

### 1.5 Development Patterns & Conventions

Infer conventions from the existing code:

- **Code style**: Indentation, quote style, naming conventions (check linter configs like `.eslintrc`, `ruff.toml`, `.editorconfig`)
- **Testing patterns**: Test file location, naming conventions, test frameworks, fixture patterns
- **Git conventions**: Read recent commit messages (`git log --oneline -20`) to detect commit message style (conventional commits, etc.)
- **Branch strategy**: Check for `main`/`master`/`develop` branches, PR templates
- **Environment management**: `.env` files, `.env.example`, Docker configs

### 1.6 Existing Agentic Config

Check if any agentic configuration already exists:

- `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, `.cursor/rules/`, `.github/copilot-instructions.md`
- `.agents/skills/` directories
- If config already exists, note what's there and what's missing — the goal is to fill gaps, not overwrite good existing config

---

## Phase 2: Generate Configuration Files

Based on everything discovered in Phase 1, generate the agentic configuration. If any of these files already exist with good content, skip or merge rather than overwrite.

### 2.1 CLAUDE.md

Create `CLAUDE.md` at the repo root (or per-project for monorepos). This is the primary config file for Claude Code.

Structure it as:

```markdown
# {Project Name}

## Build & Test Commands
{Discovered commands as bash code blocks, grouped by project if monorepo}

## Architecture
{Brief description of tech stack, project structure, key patterns, and data flow}
{For monorepos: describe each project and how they relate}

## Conventions
{Coding style, naming, testing patterns, git workflow — only what's non-obvious}
```

Guidelines for writing good CLAUDE.md content:
- **Be specific**: `cd api && dotnet test` is better than "run the tests"
- **Be concise**: This is a reference, not documentation. Bullet points over paragraphs.
- **Focus on the non-obvious**: Don't document that a React app uses components. Do document that the project uses a specific state management pattern or has an unusual file structure.
- **Include the commands someone needs on day one**: install, dev server, test, lint, build

For monorepos, also create a `CLAUDE.md` in each sub-project with project-specific details.

### 2.2 AGENTS.md

Create `AGENTS.md` at the repo root. This is the universal config read by Codex and other agents.

Structure it following the pattern:

```markdown
# {Project/Repo Name}

## Project Structure & Module Organization
{One-paragraph overview of repo layout and what lives where}

## Build, Test, and Development Commands
{Bullet list of key commands}

## Coding Style & Naming Conventions
{Key conventions to follow}

## Testing Guidelines
{Test framework, file naming, patterns}

## Commit & Pull Request Guidelines
{Commit style, PR expectations}

## Skills
{List any generated skills and where they live}
```

### 2.3 Tool Bridges

Set up discovery bridges so skills work across different AI coding tools:

1. **Claude Code**: Create `.claude/skills/{skill-name}` symlinks pointing to `../../.agents/skills/{skill-name}` (or appropriate relative path)
2. **Cursor**: Create `.cursor/rules/{skill-name}.mdc` files with Cursor frontmatter and skill content
3. **Codex**: Reference skills in `AGENTS.md`

---

## Phase 3: Generate Project Skills

Based on the codebase analysis, generate skills that will be genuinely useful for development in this repo. Each skill goes in `.agents/skills/{skill-name}/SKILL.md` (repo-wide) or `{project}/.agents/skills/{skill-name}/SKILL.md` (project-specific).

Every SKILL.md must have YAML frontmatter with at least `name` and `description` fields.

### Which skills to generate

Choose from this menu based on what the codebase analysis revealed. Don't generate skills that aren't relevant — a Python CLI doesn't need an "add component" skill.

**Always generate these (adapt to the project's tech stack):**

1. **build-and-test** — How to build, test, and lint the project. Includes dependency installation, environment setup, and common gotchas. This is the "day one" skill.

2. **architecture-overview** — A deeper dive into the codebase architecture than what's in CLAUDE.md. Entry points, data flow diagrams (as text), key abstractions, and where to find things. Think of this as the skill an agent reads when it needs to understand the system before making changes.

**Generate if applicable:**

3. **add-feature** — The standard workflow for adding a new feature in this codebase. Where to create files, what patterns to follow, what tests to write, what to update.

4. **add-endpoint** / **add-route** / **add-screen** — Framework-specific skill for adding the primary unit of work (API endpoint, page route, mobile screen, CLI command, etc.)

5. **add-component** — For frontend projects: how to create a reusable component following the project's patterns.

6. **debugging** — Project-specific debugging guidance: how to run in debug mode, where logs go, how to connect to the database, common failure modes.

7. **deployment** — If CI/CD or deployment config is present: how deploys work, environments, rollback procedures.

### Skill quality guidelines

- **Be specific to this project**, not generic. "Create a file in `src/components/`" is better than "create a component file in the appropriate directory."
- **Include real file paths and command examples** from the actual codebase.
- **Reference existing code as examples**: "Follow the pattern in `src/services/UserService.ts`" teaches more than abstract instructions.
- **Keep skills focused**: One skill per workflow. Don't create a mega-skill that covers everything.
- **Include the why**: If there's a non-obvious convention (e.g., "always use server components unless you need interactivity"), explain why.

---

## Phase 4: Verification

After generating everything, verify the setup:

- [ ] `CLAUDE.md` exists at repo root with accurate build/test commands
- [ ] `AGENTS.md` exists at repo root with project overview
- [ ] `.agents/skills/` contains generated skills with valid frontmatter
- [ ] `.claude/skills/` symlinks resolve correctly
- [ ] For monorepos: per-project config files exist where appropriate
- [ ] No existing config was accidentally overwritten
- [ ] Generated commands actually work (spot-check 1-2 by running them)

Present a summary to the user of everything that was created, with a brief description of each file and skill.

---

## Handling Edge Cases

- **Massive monorepo**: Focus on the most active projects first (check git log frequency). Generate repo-root config and skills for the top 3-5 most active projects. Note the others for future bootstrapping.
- **No tests**: Still document how to run them if a test framework is configured but empty. Note in CLAUDE.md that test coverage is sparse.
- **No CI**: Skip the CI-related parts. Mention in CLAUDE.md that there's no CI pipeline.
- **Existing partial config**: Merge with what's there. If CLAUDE.md already exists but is missing architecture info, add it. Don't delete what the developer already wrote.
- **Unfamiliar stack**: Do your best with what's in the config files and code. Be honest in the generated docs about what you're less certain about.
