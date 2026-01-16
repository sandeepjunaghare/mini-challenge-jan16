# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Research Agent**: A research agent using **(frontend) React/TypeScript + (backend) FastAPI + Pydantic AI + Claude Agent SDK**, vertical slice architecture, optimized for AI-assisted development. Python 3.12+, strict type checking with MyPy and Pyright.

## Core Principles

**KISS** (Keep It Simple, Stupid)

- Prefer simple, readable solutions over clever abstractions

**YAGNI** (You Aren't Gonna Need It)

- Don't build features until they're actually needed

**Vertical Slice Architecture**

- Each feature owns its models, schemas, routes, and business logic (or tools for agent features)
- Features live in separate directories under `app/` (e.g., `app/ui/`, `app/agent`, `app/core`)
- Shared utilities go in `app/shared/` only when used by 3+ features
- Core infrastructure (`app/core/`) includes database, config, logging, and the Pydantic AI agent

**Type Safety (CRITICAL)**

- Strict type checking enforced (MyPy + Pyright in strict mode)
- All functions, methods, and variables MUST have type annotations
- Zero type suppressions allowed
- All functions must have complete type hints
- Strict mypy & pyright configuration is enforced
- No `Any` types without explicit justification
- Test files have relaxed typing rules (see pyproject.toml)

**AI-Optimized Patterns**

- **Pydantic AI Agent**: Single agent instance with tool registration via `@agent.tool` decorators
- **Tool Design**: Follow Anthropic's "fewer, smarter tools" principle; consolidate related operations
- **Structured Logging**: Use `domain.component.action_state` pattern (hybrid dotted namespace)
  - Format: `{domain}.{component}.{action}_{state}`
  - Examples: `agent.tool.execution_started`
  - See `docs/logging-standard.md` for complete event taxonomy
- **Request Correlation**: All logs include `request_id` automatically via context vars
- **Verbose Naming**: Predictable patterns for AI code generation

## Essential Commands

### Development

```bash
# Start development server (port 8173)
uv run uvicorn app.main:app --reload --port 8173
```

### Testing

```bash
# Run all tests (34 tests, <1s execution)
uv run pytest -v

# Run integration tests only
uv run pytest -v -m integration

# Run specific test
uv run pytest -v app/core/tests/test_database.py::test_function_name
```

### Type Checking must be green

```bash
# MyPy (strict mode)
uv run mypy app/

# Pyright (strict mode)
uv run pyright app/
```

### Linting & Formatting must be green

```bash
# Check linting
uv run ruff check .

# Auto-format code
uv run ruff format .
```

### Database Migrations

```bash
# Create new migration
uv run alembic revision --autogenerate -m "description"

# Apply migrations
uv run alembic upgrade head

# Rollback one migration
uv run alembic downgrade -1

# Start PostgreSQL (Docker)
docker-compose up -d
```

### Docker

```bash
# Build and start all services
docker-compose up -d --build

# View app logs
docker-compose logs -f app

# Stop all services
docker-compose down
```

## Architecture

### Directory Structure

```
app/
├── core/           # Infrastructure (config, database, logging, middleware, agent, health, exceptions)
├── shared/         # Cross-feature utilities (pagination, timestamps, manager, etc.)
├── main.py         # FastAPI application entry point
└── features/       # Feature slices - standard features (models, routes) OR agent tools (tools.py)
    ├── products/   # Example: Standard database feature (models, routes, service)
    └── search_query/ # Example: Agent tool feature (tools.py, models.py)
```

**Agent Integration Pattern:**

- Define agent in `app/core/agent.py`: `research_agent = Agent('anthropic:claude-sonnet-4-0')`
- Register tools via `@research_agent.tool` in feature `tools.py` files
- Import tool modules in `main.py` for side-effect registration
- OpenAI compatibility layer in chat routes converts formats

**Tool Registry Import Order (CRITICAL):**

Tool registry imports have side effects and must preserve order to avoid circular imports:

```python
# At top of tool_registry.py import block
# ruff: noqa: I001

# Import existing tools first, then new tools
import app.features.search_query_tool.document_query_tool  # noqa: F401
import app.features.document_overview_query_tool.overview_query_tool  # noqa: F401
import app.features.research_get_context_tool.research_get_context_tool  # noqa: F401
```

**Verify import correctness:**

```bash
uv run python -c "from app.main import app; print('OK')"
```

If this fails with ImportError, check import order in tool_registry.py.

### Logging

**Philosophy:** Logs are optimized for AI agent consumption. Include enough context for an LLM to understand and fix issues without human intervention.

**Structured Logging (structlog)**

- JSON output for AI-parseable logs
- Request ID correlation using `contextvars`
- Logger: `from app.core.logging import get_logger; logger = get_logger(__name__)`
- Event naming: Hybrid dotted namespace pattern `domain.component.action_state`
  - Examples: `user.registration_completed`, `database.connection_initialized`, `request.http_received`
  - Detailed taxonomy: See `docs/logging-standard.md`
- Exception logging: Always include `exc_info=True` for stack traces

**Event Pattern Guidelines:**

- **Format:** `{domain}.{component}.{action}_{state}`
- **Domains:** application, request, database, health, agent, external, feature-name
- **States:** `_started`, `_completed`, `_failed`, `_validated`, `_rejected`, `_retrying`
- **Why:** OpenTelemetry compliant, AI/LLM parseable, grep-friendly, scalable for agents

**Middleware**

- `RequestLoggingMiddleware`: Logs all requests with correlation IDs
- `CORSMiddleware`: Configured for local development (see `app.core.config`)
- Adds `X-Request-ID` header to all responses

### Documentation Style

**Use Google-style docstrings** for all functions, classes, and modules:

```python
def process_request(user_id: str, query: str) -> dict[str, Any]:
    """Process a user request and return results.

    Args:
        user_id: Unique identifier for the user.
        query: The search query string.

    Returns:
        Dictionary containing results and metadata.

    Raises:
        ValueError: If query is empty or invalid.
        ProcessingError: If processing fails after retries.
    """
```

**Agent Tool Docstrings (CRITICAL):**
Tool docstrings guide LLM tool selection - they differ from standard code documentation:

```python
@research_agent.tool
async def document_query_tool(operation: str, query: str) -> QueryResult:
    """Search and discover in the document folders. Use for finding, listing, or exploring content.

    WHEN TO USE: Any discovery task - finding key information by content/tags/date, listing structure,
    exploring relationships. This is READ-ONLY - use document_manager for modifications.

    EFFICIENCY: Use response_format='concise' for simple searches to save tokens.
    Set to 'detailed' only when you need full metadata for synthesis tasks.

    COMPOSITION: Chain with get_context to read full content after finding relevant notes.

    Args:
        operation: One of: search, list_documents, find_related, search_by_tag, recent_changes
        query: Natural language query or specific search terms

    Returns:
        QueryResult with matching notes, paths, and optional metadata
    """
```

**Key Principles:**

1. Guide tool selection vs alternatives
2. Prevent token waste with efficiency guidance
3. Show composition patterns with other tools
4. Set performance expectations
5. Provide concrete usage examples

### Shared Utilities

**Pagination** (`app.shared.schemas`)

- `PaginationParams`: Query params with `.offset` property
- `PaginatedResponse[T]`: Generic response with `.total_pages` property

**Timestamps** (`app.shared.models`)

- `TimestampMixin`: Adds `created_at` and `updated_at` columns
- `utcnow()`: Timezone-aware UTC datetime helper

**Error Handling** (`app.shared.schemas`, `app.core.exceptions`)

- `ErrorResponse`: Standard error response format
- Global exception handlers configured in `app.main`

### Configuration

**Environment variables via Pydantic Settings (`app.core.config`):**

- Database: `DATABASE_URL` (postgresql+asyncpg://...)
- Agent: `LLM_PROVIDER`, `LLM_MODEL`, `LLM_API_KEY` (for Pydantic AI)
- Data folder: `DATA_FOLDER` (absolute path or relative path to the project root)
- API: `API_KEY` (for Claude Agent SDK authentication)
- CORS: `ALLOWED_ORIGINS` (app://localhost,capacitor://localhost)
- Copy `.env.example` to `.env` for local development
- Settings singleton: `get_settings()` from `app.core.config`

## Development Guidelines

**When Creating New Features**

\*Create a new feature directory under `app/` (e.g., `app/ui/`)

**Agent Tool Features:**

1. Create feature directory under `app/` (e.g., `app/search_query/`)
2. Structure: `tools.py`, `models.py`, `tests/`
3. Import agent: `from app.core.agent import research_agent`
4. Register tools: `@research_agent.tool` with LLM-optimized docstrings
5. Follow logging pattern: `agent.tool.execution_started`, `search.operation_completed`
6. Import in `main.py` for side-effect registration: `import app.search_query.tools  # noqa: F401`

**Type Checking**

- Run both MyPy and Pyright before committing
- No type suppressions (`# type: ignore`, `# pyright: ignore`) unless absolutely necessary
- Document suppressions with inline comments explaining why

**Testing**

- Write tests alongside feature code in `tests/` subdirectory
- Use `@pytest.mark.integration` for tests requiring real database
- Fast unit tests preferred (<1s total execution time)
- Test fixtures in `app/tests/conftest.py`

**Integration Test Pattern for Agent Tools:**

Test the service layer directly, NOT the tool registration function:

```python
# ✅ CORRECT - test service function
from app.features.tool_name.tool_service import execute_function

@pytest.mark.asyncio
async def test_integration(test_document_manager: DocumentManager) -> None:
    result = await execute_function(test_document_manager, param="value")
    assert result.field == expected

# ❌ WRONG - requires RunContext setup
from app.features.search_query_tool.document_query_tool import document_query_tool
from pydantic_ai import RunContext

async def test_integration(test_agent_deps: AgentDeps) -> None:
    ctx = RunContext(deps=test_agent_deps, retry=0, messages=[])  # Missing model/usage!
    result = await tool_function(ctx, param="value")
```

**Why:** Tool functions use RunContext which requires model/usage parameters not relevant for integration testing. Service functions have cleaner interfaces.

**Pattern Source:** See `app/features/search_document_tool/tests/` for reference.

**Logging Best Practices**

- Start action: `logger.info("feature.action_started", **context)`
- Success: `logger.info("feature.action_completed", **context)`
- Failure: `logger.error("feature.action_failed", exc_info=True, error=str(e), error_type=type(e).__name__, **context)`
- Agent tools: `logger.info("agent.tool.execution_started", tool="document_query_tool", operation="search")`
- Include context: IDs, durations, error details, `fix_suggestion` for AI self-correction
- Avoid generic events like "processing" or "handling"
- Use standard states: `_started`, `_completed`, `_failed`, `_validated`, `_rejected`
- **DO NOT log sensitive data:** passwords, API keys, tokens (mask: `api_key[:8] + "..."`)
- **DO NOT spam logs in loops:** Log batch summaries instead
- **DO NOT silently catch exceptions:** Always log with `logger.exception()` or re-raise

**API Patterns**

- **Agent Endpoint**: `POST /v1/chat/completions` (Claude Agent SDK compatible)
- **Health Checks**: `/health`, `/health/db`, `/health/ready`
- **Pagination**: Use `PaginationParams` and `PaginatedResponse[T]`
- **Error Responses**: Use `ErrorResponse` schema
- **Route Prefixes**: Use router `prefix` parameter for feature namespacing
- **Authentication**: Bearer token via `Authorization: Bearer <API_KEY>` header
