# Install Worktree Dependencies

Set up a fresh worktree with all dependencies installed for isolated development.

## Instructions

Install all dependencies in the worktree environment to ensure it matches the main repository.

## Installation Steps

Run these commands in order within the worktree directory:

### 1. Install Backend Dependencies
```bash
pip install -r src/backend/requirements.txt
pip install -r src/servers/requirements.txt
```

### 2. Install Shared Dependencies
```bash
pip install -r src/relay_server/requirements.txt
```

### 3. Install Development Dependencies
```bash
pip install -r requirements-dev.txt
```

### 4. Install Omnara Package in Development Mode
```bash
pip install -e .
```

This installs the `omnara` CLI, SDK, and MCP server components in editable mode.

### 5. Verify Installation
```bash
# Check if omnara CLI is available
which omnara

# Check Python imports work
python -c "from shared.database.models import User; print('✓ Imports working')"

# Check if tests can discover modules
python -m pytest --collect-only src/backend/tests/ | head -20
```

## Environment Setup

The worktree should have a `.ports.env` file created by the workflow system containing:

```bash
BACKEND_PORT=<unique_port_9100_range>
FRONTEND_PORT=<unique_port_9200_range>
```

These ports prevent conflicts when multiple worktrees run simultaneously.

## Database Migrations

**IMPORTANT**: Worktrees share the development database with the main repo.

- **DO NOT run migrations in worktrees** unless explicitly needed
- Migrations should be run from the main repo: `cd src/shared && alembic upgrade head`
- The worktree will use the same database schema as main

## Frontend Dependencies (Optional)

If working on frontend code:

```bash
# Web app
cd apps/web
npm install

# Mobile app
cd apps/mobile
npm install
```

## Validation

After installation, verify the worktree is functional:

```bash
# Run a simple test
make test-unit

# Check linting works
make lint

# Start services (uses unique ports from .ports.env)
./dev-start.sh
```

## Troubleshooting

### Import Errors
If you see `ModuleNotFoundError`:
```bash
# Ensure PYTHONPATH includes src/
export PYTHONPATH="$(pwd)/src:$PYTHONPATH"

# Or reinstall in editable mode
pip install -e .
```

### Database Connection Errors
```bash
# Ensure PostgreSQL is running
docker ps | grep postgres

# Check DATABASE_URL or DEVELOPMENT_DB_URL in .env
cat .env | grep DATABASE
```

### Port Conflicts
If ports are already in use:
```bash
# Check .ports.env for assigned ports
cat .ports.env

# Verify ports are available
lsof -i :9100  # Check if backend port is free
lsof -i :9200  # Check if frontend port is free
```

## Notes

- **Virtual Environment**: Worktrees use the system Python, not a separate venv
- **Dependencies Sync**: Keep worktree deps in sync with main repo
- **Cleanup**: Worktrees are temporary; main repo is source of truth
- **Git Config**: Worktrees inherit git config from main repo
