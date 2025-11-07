# Chore Planning

Create a plan to complete the chore using the specified markdown `Plan Format`. Research the codebase and create a thorough plan.

## Variables
adw_id: $1
prompt: $2

## Instructions

- If the adw_id or prompt is not provided, stop and ask the user to provide them.
- Create a plan to complete the chore described in the `prompt`
- The plan should be simple, thorough, and precise
- Create the plan in the `specs/` directory with filename: `chore-{adw_id}-{descriptive-name}.md`
  - Replace `{descriptive-name}` with a short, descriptive name based on the chore (e.g., "update-readme", "add-logging", "refactor-agent")
- Research the codebase starting with `README.md` and `CLAUDE.md`
- Replace every <placeholder> in the `Plan Format` with the requested value

## Codebase Structure

Omnara is a polyglot monorepo for real-time AI agent monitoring and interaction:

### Python Backend
- `src/backend/` - FastAPI web dashboard API (READ operations, Supabase auth)
- `src/servers/` - Unified agent server (WRITE operations, custom JWT)
  - `mcp/` - Model Context Protocol interface
  - `api/` - REST API interface
  - `shared/` - Common business logic
- `src/omnara/` - Python CLI & SDK (pip install omnara)
- `src/shared/` - Database models, migrations, config (SQLAlchemy + Alembic)
- `src/relay_server/` - WebSocket relay for terminal streaming
- `src/integrations/` - Agent wrappers, webhooks, n8n nodes

### Frontend Applications
- `apps/web/` - Next.js dashboard
- `apps/mobile/` - React Native app

### Infrastructure & Docs
- `infrastructure/` - Docker, deployment scripts, database setup
- `docs/` - Architecture, guides, deployment docs
- `Makefile` - Build, test, lint commands
- `CLAUDE.md` - Development guide for AI agents
- `AGENTS.md` - Claude Code agent configurations

### Development Commands
- `./dev-start.sh` - Start PostgreSQL + all servers
- `./dev-stop.sh` - Stop all services
- `make test` - Run all tests (pytest)
- `make test-unit` - Unit tests only
- `make test-integration` - Integration tests (Docker required)
- `make lint` - Run ruff + pyright
- `make format` - Auto-format with ruff

### Key Technologies
- **Backend**: Python 3.10+, FastAPI (async), SQLAlchemy 2.0 (async)
- **Database**: PostgreSQL (dev: Docker), Alembic migrations
- **Frontend**: Next.js, React Native, TypeScript
- **AI Integration**: claude-code-sdk, fastmcp
- **Auth**: Dual system (Supabase for web, custom JWT for agents)

## Plan Format

```md
# Chore: <chore name>

## Metadata
adw_id: `{adw_id}`
prompt: `{prompt}`

## Chore Description
<describe the chore in detail based on the prompt>

## Relevant Files
Use these files to complete the chore:

<list files relevant to the chore with bullet points explaining why. Include new files to be created under an h3 'New Files' section if needed>

## Step by Step Tasks
IMPORTANT: Execute every step in order, top to bottom.

<list step by step tasks as h3 headers with bullet points. Start with foundational changes then move to specific changes. Last step should validate the work>

### 1. <First Task Name>
- <specific action>
- <specific action>

### 2. <Second Task Name>
- <specific action>
- <specific action>

## Validation Commands
Execute these commands to validate the chore is complete:

<list specific commands to validate the work. Be precise about what to run>

Examples:
- `make test-unit` - Run unit tests
- `make lint` - Check code quality
- `make format` - Auto-format code
- `python -m pytest src/backend/tests/test_specific.py -v` - Run specific test

## Notes
<optional additional context or considerations>
```

## Chore
Use the chore description from the `prompt` variable.

## Report

Return the path to the plan file created.
