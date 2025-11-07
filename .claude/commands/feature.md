# Feature Planning

Create a comprehensive plan for a new feature using the specified markdown `Plan Format`. Research the codebase and create a detailed implementation strategy.

## Variables
adw_id: $1
prompt: $2

## Instructions

- If the adw_id or prompt is not provided, stop and ask the user to provide them.
- Create a thorough plan for the feature described in the `prompt`
- The plan should include user stories, acceptance criteria, and step-by-step implementation
- Create the plan in the `specs/` directory with filename: `feature-{adw_id}-{descriptive-name}.md`
  - Replace `{descriptive-name}` with a short, descriptive name based on the feature (e.g., "user-auth", "real-time-notifications", "agent-dashboard")
- Research the codebase starting with `README.md`, `CLAUDE.md`, and `docs/`
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
# Feature: <feature name>

## Metadata
adw_id: `{adw_id}`
prompt: `{prompt}`

## User Story
As a <type of user>, I want <goal> so that <benefit>.

## Problem Statement
<describe the problem this feature solves>

## Solution Overview
<high-level description of how this feature works>

## Relevant Files

### Existing Files to Modify
<list files that need changes with bullet points explaining what changes>

### New Files to Create
<list new files to create with bullet points explaining their purpose>

## Implementation Phases

### Phase 1: <Phase Name (e.g., Database Schema)>
<description of what this phase accomplishes>

#### Tasks
- <specific task>
- <specific task>

### Phase 2: <Phase Name (e.g., Backend API)>
<description of what this phase accomplishes>

#### Tasks
- <specific task>
- <specific task>

### Phase 3: <Phase Name (e.g., Frontend UI)>
<description of what this phase accomplishes>

#### Tasks
- <specific task>
- <specific task>

## Testing Strategy
<describe how to test this feature>

### Unit Tests
- <test description>

### Integration Tests
- <test description>

### Manual Testing
- <manual test steps>

## Acceptance Criteria
- [ ] <criterion>
- [ ] <criterion>
- [ ] <criterion>

## Validation Commands
Execute these commands to validate the feature is complete:

- `make test` - Run all tests
- `make lint` - Check code quality
- `make format` - Auto-format code
- `./dev-start.sh` - Start services and verify feature works

## Security Considerations
<any security implications or requirements>

## Performance Considerations
<any performance implications or optimizations needed>

## Documentation Updates
<list documentation that needs updating>

## Notes
<optional additional context, risks, or future enhancements>
```

## Feature
Use the feature description from the `prompt` variable.

## Report

Return the path to the plan file created.
