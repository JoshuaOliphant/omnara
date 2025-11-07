# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Note**: This project uses [bd (beads)](https://github.com/steveyegge/beads) for issue tracking. Use `bd` commands instead of markdown TODOs. See AGENTS.md for workflow details.

## Project Overview

Omnara is a platform that enables real-time monitoring and interaction with AI agents (Claude Code, Codex CLI, n8n workflows, etc.) through mobile, web, and API interfaces. Users can see what their agents are doing and respond to questions instantly.

**Key Innovation**: Bidirectional communication - agents send progress updates and questions, users respond from any device.

## Architecture

### Monorepo Structure
```
src/
├── omnara/         # Python CLI & SDK (pip install omnara)
├── backend/        # FastAPI web dashboard API (READ operations)
├── servers/        # Unified agent server (WRITE operations)
│   ├── mcp/       # Model Context Protocol interface
│   ├── api/       # REST API interface
│   └── shared/    # Common business logic
├── shared/        # Database models, migrations, config
├── relay_server/  # WebSocket relay for terminal streaming
├── integrations/  # Agent wrappers, webhooks, n8n nodes
└── mcp-installer/ # NPX tool for MCP config

apps/
├── web/           # Next.js dashboard
└── mobile/        # React Native app

infrastructure/    # Docker, deployment, scripts
```

### Critical Architectural Decisions

**1. Dual Server Architecture**
- **Backend** (`src/backend/`): Web dashboard API - READ operations only
  - Auth: Supabase JWTs for web users
  - Port: 8000

- **Servers** (`src/servers/`): Agent communication - WRITE operations only
  - Auth: Custom JWT with weaker RSA (shorter API keys)
  - Port: 8080
  - Exposes both MCP (`/mcp/`) and REST (`/api/v1/`) interfaces

**2. Unified Messaging System**
All agent interactions flow through the `messages` table:
- `sender_type`: AGENT or USER
- `requires_user_input`: Boolean flag for questions vs. updates
- `last_read_message_id`: Track reading progress per instance
- Agents receive queued user messages when sending new messages

**3. Multi-Protocol Support**
The unified server (`src/servers/app.py`) supports:
- **MCP Protocol**: For MCP-compatible agents (Claude Code, etc.)
- **REST API**: For SDK clients and direct integrations
- Both share identical authentication and business logic

## Development Commands

### Setup
```bash
# First time setup
cp .env.example .env
python infrastructure/scripts/generate_jwt_keys.py
./dev-start.sh  # Starts PostgreSQL (Docker) + all servers

# Stop everything
./dev-stop.sh

# Reset database
./dev-start.sh --reset-db
```

### Daily Development
```bash
make lint          # Run all checks (ruff + pyright)
make format        # Auto-format with ruff
make typecheck     # Type checking only
make test          # All tests
make test-unit     # Skip integration tests
make test-integration  # Docker-dependent tests only

# Run specific test
make test-k ARGS="test_auth"
```

### Database Migrations

**CRITICAL**: Always run from `src/shared/` directory:

```bash
cd src/shared/

# Check current migration status
alembic current

# Create migration after model changes
alembic revision --autogenerate -m "description"

# Apply migrations
alembic upgrade head

# Rollback one migration
alembic downgrade -1
```

**Pre-commit hook enforces**: Model changes must have corresponding migrations.

## Key Technical Constraints

### Authentication Security
- **Backend**: Uses Supabase JWTs - DO NOT mix with server auth
- **Servers**: Custom JWT with weaker RSA implementation
  - ⚠️ Keep BOTH private AND public keys secure
  - API keys are hashed (SHA256) before storage - never store raw tokens

### Database Rules
1. All models in `src/shared/database/models.py`
2. Multi-tenant: ALL queries must filter by `user_id`
3. Use SQLAlchemy 2.0+ async patterns
4. Migrations are version-controlled - commit them with model changes

### Import Patterns
```python
# Always use absolute imports from project root
from shared.database.models import User, AgentInstance
from servers.shared.messages import create_message
from backend.auth.supabase import get_current_user

# Set PYTHONPATH when running manually
export PYTHONPATH="$(pwd)/src"
```

### Running Services Manually
```bash
# From project root with PYTHONPATH set
export PYTHONPATH="$(pwd)/src"

# Backend (port 8000)
uvicorn backend.main:app --port 8000

# Unified Server (port 8080)
python -m servers.app

# Relay Server (port 8787)
python -m relay_server.app
```

## Testing Philosophy

- Python 3.10+ required (3.11+ preferred)
- Use pytest with async support (`asyncio_mode = "auto"`)
- Mark integration tests: `@pytest.mark.integration`
- Integration tests need Docker (PostgreSQL)
- Pre-commit hooks run ruff formatting and migration checks

## Common Workflows

### Adding an API Endpoint

**Backend (read operations)**:
1. Add route in `src/backend/api/`
2. Create Pydantic models in `src/backend/models.py`
3. Add query in `src/backend/db/`
4. Write tests in `src/backend/tests/`

**Servers (write operations)**:
1. Add to both `src/servers/mcp/tools.py` AND `src/servers/api/routers.py`
2. Share logic via `src/servers/shared/`
3. Test both MCP and REST interfaces

### Modifying Database Schema
1. Edit `src/shared/database/models.py`
2. `cd src/shared/ && alembic revision --autogenerate -m "description"`
3. Review generated migration (edit if needed)
4. Test: `alembic upgrade head`
5. Update any affected Pydantic schemas
6. Commit both model and migration files

### Working with Messages
```python
# Agent sends a question
create_message(
    agent_instance_id=instance_id,
    sender_type=SenderType.AGENT,
    content="Should I refactor this module?",
    requires_user_input=True
)

# Agent sends progress update
create_message(
    agent_instance_id=instance_id,
    sender_type=SenderType.AGENT,
    content="Analyzing codebase structure",
    requires_user_input=False
)

# User responds
create_message(
    agent_instance_id=instance_id,
    sender_type=SenderType.USER,
    content="Yes, refactor it"
)
```

## Environment Variables

Core variables (see `.env.example` for complete list):
- `DATABASE_URL`: PostgreSQL connection
- `DEVELOPMENT_DB_URL`: Override for local dev
- `JWT_PUBLIC_KEY` / `JWT_PRIVATE_KEY`: Agent auth (RSA keys with newlines)
- `SUPABASE_URL` / `SUPABASE_ANON_KEY`: Web auth
- `ENVIRONMENT`: Set to "development" for auto-reload

## Commit Conventions

Follow conventional commits:
- `feat:` New features
- `fix:` Bug fixes
- `docs:` Documentation only
- `refactor:` Code restructuring
- `test:` Test additions/changes

Example: `feat: add message filtering by date range`

## Type Hints & Code Style

- Python 3.10+ with full type annotations required
- Ruff for linting and formatting (replaces black/flake8)
- Pyright for type checking
- Prefer `Mapped[type]` for SQLAlchemy columns
- Use Pydantic v2 for validation schemas

## Pitfalls to Avoid

1. **Don't run migrations from wrong directory** - Always `cd src/shared/` first
2. **Don't skip migrations** - Pre-commit hook will block commits
3. **Don't mix auth systems** - Backend ≠ Servers authentication
4. **Don't forget user scoping** - All queries need `user_id` filter
5. **Don't store raw API keys** - Hash with SHA256 first
6. **Don't expose JWT keys** - Both public and private keys are sensitive

## Deployment

Uses automated scripts:
- `./dev-start.sh`: Local development (PostgreSQL in Docker)
- `./dev-stop.sh`: Stop all services
- See `infrastructure/` for production deployment configs

## Package Distribution

Omnara is published to PyPI as `omnara`:
- Version defined in `pyproject.toml`
- CLI entry point: `omnara.cli:main`
- Includes: CLI, SDK, MCP server, agent wrappers
- Install: `pip install omnara` or `uv tool install omnara`

## AI Developer Workflows (ADWs)

Omnara includes a complete ADW (AI Developer Workflows) infrastructure for programmatic agent orchestration. This enables Claude Code agents to be invoked programmatically for planning, implementing, testing, reviewing, and deploying features with full worktree isolation.

### Overview

ADWs provide three capability tiers:
1. **Minimal**: Core subprocess execution, basic prompts
2. **Enhanced**: SDK support, compound workflows, TDD planning
3. **Scaled**: Worktree isolation, multi-phase SDLC, beads integration (✅ **Installed**)

### Quick Start

```bash
# Execute an adhoc prompt
./adws/adw_prompt.py "Analyze the authentication flow"

# Plan a chore
./adws/adw_slash_command.py /chore $(uuidgen | cut -c1-8) "add error handling"

# Full SDLC workflow (plan → build → test → review → document)
./adws/adw_sdlc_iso.py <beads-task-id>

# Interactive beads workflow picker
./adws/adw_beads_ready.py
```

### Directory Structure

```
adws/
├── adw_modules/          # Core execution engines
│   ├── agent.py          # Subprocess-based execution
│   ├── agent_sdk.py      # SDK-based execution
│   ├── state.py          # Workflow state management
│   ├── git_ops.py        # Git operations
│   ├── worktree_ops.py   # Worktree + port management
│   ├── workflow_ops.py   # Workflow orchestration
│   ├── beads_integration.py  # Beads issue tracker
│   └── github.py         # GitHub integration
├── adw_prompt.py         # CLI: Adhoc prompts
├── adw_chore_implement.py  # Workflow: Plan + implement
├── adw_plan_tdd.py       # Workflow: TDD task breakdown
├── adw_plan_iso.py       # Phase: Planning (isolated)
├── adw_build_iso.py      # Phase: Implementation
├── adw_test_iso.py       # Phase: Testing
├── adw_review_iso.py     # Phase: Review + auto-fix
├── adw_document_iso.py   # Phase: Documentation
├── adw_ship_iso.py       # Phase: Merge to main
├── adw_sdlc_iso.py       # Composite: Full SDLC
└── adw_beads_ready.py    # Interactive: Beads picker

.claude/commands/         # Slash command templates
├── chore.md             # Chore planning
├── feature.md           # Feature planning
├── implement.md         # Implementation
├── test.md              # Test execution
├── review.md            # Code review
├── document.md          # Documentation generation
└── [15+ more commands]

specs/                   # Generated plans
agents/                  # Execution outputs & state
trees/                   # Git worktrees (isolated)
```

### Key Workflows

#### Adhoc Prompt Execution
```bash
# Quick one-off prompts
./adws/adw_prompt.py "Explain the database schema"
./adws/adw_prompt.py "Find security vulnerabilities" --model opus

# With custom working directory
./adws/adw_prompt.py "Analyze tests" --working-dir src/backend/tests/
```

#### Chore Planning + Implementation
```bash
# Generate plan, then implement
./adws/adw_chore_implement.py "add rate limiting to API endpoints"

# Manual two-step process
./adws/adw_slash_command.py /chore abc123 "add logging"
./adws/adw_slash_command.py /implement specs/chore-abc123-*.md
```

#### TDD Planning for Large Tasks
```bash
# Break down a large feature into agent-sized tasks
./adws/adw_plan_tdd.py "Implement real-time collaboration with WebSockets"

# From a spec file
./adws/adw_plan_tdd.py specs/feature-auth.md --spec-file

# Use Opus for complex architecture
./adws/adw_plan_tdd.py "Build event sourcing system" --model opus

# Output: specs/plans/plan-{id}.md with:
# - 25 tasks broken down (agent-optimized sizing)
# - Dependency graph and phases
# - TDD guidance for each task
# - Agent-centric complexity metrics (context load, iterations)
```

#### Individual SDLC Phases (Worktree Isolation)
```bash
# 1. Planning phase - Creates worktree, allocates ports, generates plan
./adws/adw_plan_iso.py poc-123

# 2. Implementation phase - Executes plan in isolated worktree
./adws/adw_build_iso.py poc-123 <adw-id>

# 3. Testing phase - Runs test suite
./adws/adw_test_iso.py poc-123 <adw-id>

# 4. Review phase - Reviews code, auto-fixes blocking issues
./adws/adw_review_iso.py poc-123 <adw-id>

# 5. Documentation phase - Generates docs with screenshots
./adws/adw_document_iso.py poc-123 <adw-id>

# 6. Shipping phase - Merges to main, closes issue
./adws/adw_ship_iso.py poc-123 <adw-id>
```

#### Composite SDLC Workflows
```bash
# Full SDLC (all 5 phases)
./adws/adw_sdlc_iso.py poc-123

# Partial SDLC (skip documentation)
./adws/adw_plan_build_test_review_iso.py poc-123

# Skip end-to-end tests (faster)
./adws/adw_sdlc_iso.py poc-123 --skip-e2e

# Skip auto-resolution of review issues
./adws/adw_sdlc_iso.py poc-123 --skip-resolution
```

#### Interactive Beads Workflows
```bash
# Pick from ready tasks, run full SDLC
./adws/adw_beads_ready.py

# Pick task, run partial workflow
./adws/adw_beads_ready.py --workflow plan-build-test-review
```

### Worktree Isolation

Every ADW execution gets its own isolated environment:

**Worktree Location**: `trees/{adw_id}/`
- Fresh checkout from `origin/main`
- Fully isolated git working directory
- Independent package installation
- Preserved for debugging after completion

**Port Allocation**: Deterministic based on ADW ID
- Backend: 9100-9114 (15 slots)
- Frontend: 9200-9214 (15 slots)
- Prevents conflicts when running parallel workflows
- Configured via `.ports.env` in worktree

**State Management**: `agents/{adw_id}/adw_state.json`
- Persistent across workflow phases
- Tracks: issue, branch, plan, ports, worktree path
- Validated with Pydantic models

### Issue Tracking Integration

**Beads (Local SQLite)** - Primary system for Omnara:
```bash
# Create task
bd add "Add real-time notifications" --type feature

# List ready tasks
bd ready

# Show task details
bd show poc-123

# ADW workflows automatically:
# - Fetch beads issues
# - Update status to "in_progress"
# - Close on successful ship
```

**GitHub Issues** - Also supported:
```bash
# Fetch by issue number
./adws/adw_plan_iso.py 42

# ADW workflows automatically:
# - Fetch issue details
# - Post status comments
# - Link to PR on ship
```

### Configuration

**Mode A: Claude Max Subscription (Default)**
- No configuration needed
- Claude Code authenticates via subscription
- Perfect for interactive development

**Mode B: API-Based Automation**
```bash
# Create .env file
cp .env.sample .env

# Add API key
echo "ANTHROPIC_API_KEY=sk-ant-..." >> .env

# Now workflows work in headless mode (CI/CD, webhooks)
```

**Optional: GitHub Integration**
```bash
# Add GitHub PAT to .env for GitHub issue operations
echo "GITHUB_PAT=ghp_..." >> .env
```

### Output & Observability

All ADW executions create structured outputs in `agents/{adw_id}/`:

```
agents/abc12345/
├── adw_state.json          # Persistent state
├── planner/                # Planning agent outputs
│   ├── cc_raw_output.jsonl    # Raw JSONL stream
│   ├── cc_raw_output.json     # Parsed JSON array
│   ├── cc_final_object.json   # Final result object
│   └── custom_summary_output.json  # High-level summary
├── implementor/            # Implementation outputs
├── test_validator/         # Test results
└── review_agent/           # Review findings + screenshots
    └── review_img/         # UI screenshots
```

### Slash Commands

ADW workflows execute via slash commands defined in `.claude/commands/`:

**Planning**:
- `/chore` - Plan small maintenance tasks
- `/feature` - Plan new features with user stories
- `/bug` - Plan bug fixes with reproduction steps
- `/plan-tdd` - Break large tasks into agent-sized chunks

**Execution**:
- `/implement` - Execute a plan file
- `/patch` - Quick fix for review findings
- `/test` - Run test suite with structured output
- `/review` - Code review with auto-fix capability
- `/document` - Generate documentation with screenshots

**Workflow Management**:
- `/install_worktree` - Set up worktree dependencies
- `/cleanup_worktrees` - Remove stale worktrees
- `/classify_issue` - Determine issue type
- `/generate_branch_name` - Create standardized branch name
- `/pull_request` - Generate PR description

### Agent-Centric Task Sizing

ADW uses **agent-centric complexity metrics** (not human time estimates):

**Size S** (Simple):
- Read 1-2 files, modify 1-2
- Write 5-10 tests
- 1-2 iterations expected
- Example: "Add logging to endpoint"

**Size M** (Medium):
- Read 3-5 files, modify 2-4
- Write 10-20 tests
- 2-4 iterations expected
- Example: "Refactor authentication module"

**Size L** (Large):
- Read 6+ files, modify 3-5
- Write 20+ tests
- 4-6+ iterations expected
- Example: "Implement WebSocket real-time updates"

**Rationale**: Measures **context switching cost** and **iteration depth**, not developer time.

### Advanced Features

**Auto-Resolution in Review Phase**:
```bash
# Review phase automatically fixes blocking issues
./adws/adw_review_iso.py poc-123 <adw-id>

# Creates patch plans for each blocker
# Implements patches using separate agents
# Re-reviews until blockers are resolved

# Skip auto-resolution if needed
./adws/adw_review_iso.py poc-123 <adw-id> --skip-resolution
```

**State Passing Between Phases**:
- Persistent: File-based JSON (`adw_state.json`)
- Validated: Pydantic schemas enforce correctness
- Portable: Can resume workflows on different machines

**Port Conflict Prevention**:
- Deterministic allocation based on ADW ID hash
- Automatic fallback if ports occupied
- Stored in `.ports.env` for worktree services

### Cleanup

**Worktrees** (manual cleanup):
```bash
# List all worktrees
git worktree list

# Remove specific worktree
git worktree remove trees/abc12345

# Automated cleanup
./adws/adw_slash_command.py /cleanup_worktrees
```

**State Files** (safe to delete after workflow complete):
```bash
# Remove specific ADW outputs
rm -rf agents/abc12345/

# Clean all agent outputs (nuclear option)
rm -rf agents/
```

### Troubleshooting

**Claude Code not found**:
```bash
# Verify installation
claude --version

# Set explicit path in .env
echo "CLAUDE_CODE_PATH=/path/to/claude" >> .env
```

**Import errors in worktree**:
```bash
# Reinstall dependencies
cd trees/{adw-id}/
pip install -e .
```

**Port conflicts**:
```bash
# Check assigned ports
cat trees/{adw-id}/.ports.env

# Verify availability
lsof -i :9100
```

**Worktree validation errors**:
```bash
# Three-way check: state + filesystem + git
git worktree list                    # Git's view
ls -la trees/{adw-id}/               # Filesystem
cat agents/{adw-id}/adw_state.json   # State file

# If mismatched, remove and recreate
git worktree remove trees/{adw-id}/ --force
./adws/adw_plan_iso.py <issue-id>  # Recreates worktree
```

### Best Practices

1. **Use worktree isolation for all non-trivial work** - Prevents conflicts
2. **Run full SDLC before shipping** - Ensures quality gates pass
3. **Let review phase auto-fix blockers** - Saves manual iteration
4. **Use TDD planning for large features** - Breaks into agent-sized tasks
5. **Preserve worktrees after completion** - Valuable for debugging
6. **Track ADW IDs in commit messages** - Improves traceability

### Integration with Omnara Development

ADWs are **production-ready** for Omnara development:
- ✅ Adapted to monorepo structure (Python + TypeScript)
- ✅ Uses Makefile commands (`make test`, `make lint`)
- ✅ Respects dual auth system (Supabase + custom JWT)
- ✅ Handles database migrations correctly (run from main repo)
- ✅ Supports both backend and frontend development
- ✅ Integrated with Beads for offline-first issue tracking

## Docs

### Getting Started
- [README](README.md): Project overview, quick start, and installation guide
- [CONTRIBUTING](CONTRIBUTING.md): Guide for contributing to the Omnara project
- [AGENTS](AGENTS.md): Claude Code development guide for working on Omnara

### CLI & Usage
- [CLI Reference](docs/cli-reference.md): Complete command-line interface documentation with all commands, options, and examples

### Architecture & Integration
- [Architecture Diagram](docs/guides/architecture-diagram.md): System architecture overview with Mermaid diagrams showing component interactions and data flow
- [n8n Integration](docs/n8n.md): Comprehensive n8n workflow integration architecture, webhook configuration, and AI agent tool setup

### Deployment
- [Fly.io Setup Guide](docs/deployment/fly-io-setup.md): Step-by-step guide for deploying Omnara to Fly.io with Supabase authentication
- [Deployment Quick Start](DEPLOYMENT_QUICK_START.md): Rapid deployment instructions
- [Deployment Summary](DEPLOYMENT_SUMMARY.md): Overview of deployment architecture and decisions
