# Product Requirements Document: AI Research Agent

> **Last Updated:** 2026-01-02
> **Status:** Backend MVP complete (7/7 MCP tools), Frontend Phase 5 complete (chat display improvements)

## 1. Executive Summary

The AI Research Agent is a document analysis system designed to replace a technical research intern. It provides accurate, citation-backed Q&A on technical documents, strategic analysis capabilities (compare, contrast, categorize), and temporal evolution tracking of positions across meetings over time.

The agent uses an agentic search approach (grep/pandoc/bash) rather than traditional RAG embeddings, following Anthropic's guidance that agentic search is more reliable and transparent for structured document sets. Built on FastAPI with Claude Agent SDK, it exposes a chat completions API that can be consumed by any OpenAI-compatible client.

**MVP Goal:** Deliver a working research agent that can accurately answer questions about documents in a folder, compare company positions, track position evolution across meetings, and provide every answer with exact source citations—achieving zero hallucination through mandatory evidence verification.

---

## 2. Mission

### Mission Statement

Enable technical researchers to interact with large document corpuses through natural language, replacing manual document reading with an AI assistant that provides accurate, verifiable, citation-backed answers.

### Core Principles

1. **No Hallucination:** Every factual claim must be grounded in source documents with `[file:line]` citations
2. **Complete Research:** The agent must search exhaustively before answering—never miss relevant information
3. **Transparent Provenance:** Users can verify any answer by following citations to exact source locations
4. **Token Efficiency:** Use `response_format` parameters to control verbosity and save context
5. **Fewer, Smarter Tools:** Consolidate operations into well-designed tools following Anthropic's principles

---

## 3. Target Users

### Primary Persona: Technical Research Engineer

**Profile:**

- Engineers working with standards bodies (3GPP, IEEE, etc.)
- Analysts tracking company positions across meeting submissions
- Technical writers synthesizing information from multiple sources

**Key Pain Points:**

- Reading 50+ documents per agenda item is time-consuming
- Tracking how company positions evolve over multiple meetings is manual
- Easy to miss relevant information buried in dense technical documents
- Cross-referencing positions across companies requires extensive note-taking

**Primary Needs:**

- Get accurate answers to specific questions without reading everything
- Compare positions between companies on specific topics
- Track how a company's stance has changed over time
- Trust that answers are complete and not hallucinated

---

## 4. MVP Scope

### In Scope (MVP)

**Core Functionality:**

- ✅ Natural language Q&A on documents in `data/files/` directory
- ✅ Document discovery, search, and reading with line-number citations
- ✅ Company position comparison (Company A vs Company B)
- ✅ Temporal evolution tracking (position changes across meetings)
- ✅ Categorization (group companies by approach)
- ✅ Consensus/disagreement identification across corpus
- ✅ Mandatory evidence verification before every response

**Document Formats:**

- ✅ Native: `.md`, `.txt`, `.json`
- ✅ Via openpyxl: `.xlsx` (fast native Python reading, ~21ms)
- ✅ Via pandoc: `.docx`, `.pdf`

**Technical:**

- ✅ FastAPI backend with Claude Agent SDK
- ✅ OpenAI-compatible API (`POST /v1/chat/completions`)
- ✅ Structured logging with request correlation
- ✅ Strict type checking (MyPy + Pyright)
- ✅ Bearer token authentication

### Out of Scope (Future Phases)

- ✅ Web UI / Chat interface (Phase 4 complete - e2e tested)
- ❌ Document ingestion/upload API
- ❌ Semantic/vector search (agentic grep only)
- ❌ Multi-user / session management
- ❌ Database persistence
- ✅ Streaming responses with tool progress events
- ❌ Docker containerization

---

## 5. User Stories

### Primary User Stories

| ID   | Story              | Example Query                                                 | Expected Output                              |
| ---- | ------------------ | ------------------------------------------------------------- | -------------------------------------------- |
| US-1 | Basic Q&A          | "What is Qualcomm's position on positioning-based solutions?" | Answer with `[R1-2509178.docx:45]` citations |
| US-2 | Position Compare   | "Compare Qualcomm vs Ericsson on timing advance"              | Side-by-side comparison with citations       |
| US-3 | Temporal Evolution | "How has Viasat's position changed from RAN1#121 to #123?"    | Timeline with meeting-specific citations     |
| US-4 | Categorization     | "List companies supporting positioning vs timing-advance"     | Categorized matrix with evidence             |
| US-5 | Consensus ID       | "What's the consensus on DL-TDOA positioning?"                | Agreed/disputed points with citations        |
| US-6 | Section Extract    | "Show all proposals from Qualcomm's documents"                | Proposals with document/line references      |

### Technical User Stories

| ID    | Story                | Requirement                                                |
| ----- | -------------------- | ---------------------------------------------------------- |
| US-T1 | Zero Hallucination   | Every response must pass through `evidence_grounding` tool |
| US-T2 | Verifiable Citations | Citations must include exact `file:line` references        |

---

## 5.1 Gap Analysis: Why 7 Tools Are Required

The original 3-tool design does not fully cover all user stories. This analysis identifies gaps and justifies the 7-tool architecture.

### Coverage Gap Matrix

| User Story                  | Gap with 3 Tools             | Solution Tool        |
| --------------------------- | ---------------------------- | -------------------- |
| US-1: Basic Q&A             | Missing `[file:line]` format | `citation_generator` |
| US-2: Position Compare      | No structured extraction     | `position_extractor` |
| US-3: Temporal Evolution    | No meeting-aware tracking    | `temporal_tracker`   |
| US-4: Categorization        | No position classification   | `position_extractor` |
| US-5: Consensus ID          | No agreement detection       | `consensus_analyzer` |
| US-6: Section Extract       | No line-numbered parsing     | `document_reader`    |
| US-T1: Zero Hallucination   | Needs hook enforcement       | `evidence_grounding` |
| US-T2: Verifiable Citations | Needs format generator       | `citation_generator` |

### Implementation Priority

| Phase             | Tools                                                         | User Stories             | Priority |
| ----------------- | ------------------------------------------------------------- | ------------------------ | -------- |
| **Phase 1 (MVP)** | `document_reader`, `citation_generator`, `evidence_grounding` | US-1, US-6, US-T1, US-T2 | Critical |
| **Phase 2**       | `corpus_search`, `position_extractor`                         | US-2, US-4               | High     |
| **Phase 3**       | `temporal_tracker`, `consensus_analyzer`                      | US-3, US-5               | Medium   |

---

## 6. Core Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   FastAPI Application                           │
│              POST /v1/chat/completions (Bearer Auth)            │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Claude Agent SDK                              │
│         ClaudeSDKClient - Agentic Loop                          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │         Built-in Tools: Read | Glob | Grep | Bash | Task │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │   Custom MCP Server: research_tools (7 tools)            │   │
│  │   Phase 1: document_reader, citation_generator,          │   │
│  │            evidence_grounding (MANDATORY)                │   │
│  │   Phase 2: corpus_search, position_extractor             │   │
│  │   Phase 3: temporal_tracker, consensus_analyzer          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │   Hooks: PreToolUse | PostToolUse                        │   │
│  │   - Audit logging (all requests)                         │   │
│  │   - StreamingHooks (progress events via async queue)     │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Document Corpus: data/files/**/*             │
│      .md | .txt | .json (native) | .xlsx (openpyxl) |           │
│                    .docx | .pdf (pandoc)                        │
└─────────────────────────────────────────────────────────────────┘
```

### Directory Structure

```
base-research-agent/
├── CLAUDE.md                    # AI coding instructions
├── data/files/                  # Document storage
├── backend/
│   ├── pyproject.toml
│   ├── .env
│   └── app/
│       ├── main.py              # FastAPI entry point
│       ├── core/
│       │   ├── config.py        # Pydantic Settings
│       │   ├── logging.py       # structlog setup
│       │   ├── agent.py         # Claude Agent SDK wrapper (ResearchClient)
│       │   ├── mcp_server.py    # Custom MCP server with 7 tools
│       │   ├── hooks.py         # PreToolUse/PostToolUse + StreamingHooks
│       │   └── middleware.py    # Request logging
│       ├── shared/
│       │   ├── schemas.py       # Common response models
│       │   ├── document_utils.py # openpyxl (xlsx) + pandoc (docx/pdf)
│       │   └── metadata_extractor.py
│       ├── features/
│       │   └── chat/            # POST /v1/chat/completions
│       └── tools/               # 7 MCP Research Tools
│           ├── document_reader.py
│           ├── citation_generator.py
│           ├── evidence_grounding.py
│           ├── corpus_search.py
│           ├── position_extractor.py
│           ├── temporal_tracker.py
│           └── consensus_analyzer.py
└── frontend/                    # React Chat Interface
    ├── package.json
    ├── vite.config.ts
    └── src/
        ├── App.tsx              # Application entry
        ├── lib/
        │   ├── utils.ts         # Tailwind utilities
        │   ├── api.ts           # OpenAI-compatible API layer
        │   └── supabase.ts      # Supabase client
        ├── types/
        │   ├── api.types.ts     # Chat/Progress event types
        │   └── database.types.ts # Supabase table types
        ├── hooks/
        │   └── useStreamingChat.ts  # SSE streaming state
        └── components/
            └── chat/
                └── ToolProgress.tsx # Tool execution display
```

### Claude Agent SDK Integration

```python
from claude_code_sdk import ClaudeAgentOptions, ClaudeSDKClient, tool, create_sdk_mcp_server

# Create MCP server with research tools
research_server = create_sdk_mcp_server(
    name="research",
    tools=[document_reader_tool, citation_generator_tool, evidence_grounding_tool, ...]
)

# FastAPI endpoint wraps SDK
async def chat_completion(prompt: str):
    options = ClaudeAgentOptions(
        mcp_servers={"research": research_server},
        allowed_tools=[
            "Read", "Glob", "Grep", "Bash", "Task",  # Built-in
            "mcp__research__document_reader",         # Custom MCP
            "mcp__research__citation_generator",
            "mcp__research__evidence_grounding",
        ],
        permission_mode="bypassPermissions",
        cwd="./data/files"
    )
    async with ClaudeSDKClient(options=options) as agent:
        await agent.query(prompt=prompt)
        async for msg in agent.receive_response():
            yield msg
```

### Key Design Patterns

**Claude Agent SDK**

- Use `ClaudeSDKClient` for production (hooks, callbacks, session management)
- Custom tools via `@tool` decorator and `create_sdk_mcp_server()`
- `setting_sources=["project"]` to load CLAUDE.md context

**Anti-Hallucination Pattern**

- Mandatory `evidence_grounding` tool call before every response
- Every claim requires `[file:line]` citation
- Explicit "Not found in documents" when no evidence exists
- PreToolUse hook validates citation requirements

**Streaming Progress Pattern**

- `StreamingHooks` class pushes progress events to an `asyncio.Queue`
- Router drains queue (non-blocking) before each text chunk
- Progress messages use explicit (start, complete) tuples for proper grammar
- Events include `tool_use_id` for correlating start/complete pairs

**Agentic Search (per Anthropic)**

- Use SDK's built-in Glob, Grep, Read for file operations
- Bash tool for pandoc conversion of binary formats
- Task tool spawns subagents for parallel analysis
- No embeddings or vector databases for MVP

---

## 7. MCP Research Tools (7 Tools)

### Tool Summary

| Tool                 | Purpose               | Phase   | Key Inputs                     | Key Outputs                        |
| -------------------- | --------------------- | ------- | ------------------------------ | ---------------------------------- |
| `document_reader`    | Line-numbered parsing | 1 (MVP) | file_path, section, line_range | lines[], sections{}, metadata      |
| `citation_generator` | `[file:line]` format  | 1 (MVP) | source_file, line_numbers      | citation, verification_hash        |
| `evidence_grounding` | Zero hallucination    | 1 (MVP) | claim, source_files            | status, confidence, evidence[]     |
| `corpus_search`      | Find documents        | 2       | query, company, meeting_id     | matches[], total_count             |
| `position_extractor` | Extract positions     | 2       | company, topic                 | position, category, confidence     |
| `temporal_tracker`   | Track evolution       | 3       | company, topic, meetings[]     | timeline[], changes[], trend       |
| `consensus_analyzer` | Detect agreement      | 3       | topic, companies[]             | agreed_points[], disputed_points[] |

---

### Tool 1: document_reader

**Purpose:** Line-numbered document parsing with section extraction

**When to Use:**

- Reading document content with line numbers for citations
- Extracting specific sections (Proposal, Observation, Conclusion)
- Getting metadata from document headers

**Input Schema:**

| Field            | Type        | Description                      |
| ---------------- | ----------- | -------------------------------- |
| file_path        | str         | Relative path from data/files/   |
| section          | str?        | Extract specific section         |
| line_range       | (int, int)? | Read specific lines              |
| include_metadata | bool        | Extract metadata (~50 tokens)    |
| output_format    | literal     | "full", "sections", "lines_only" |

**Output:** Lines with line_num, sections with line ranges, metadata

---

### Tool 2: citation_generator

**Purpose:** Generate properly formatted `[file:line]` citations

**When to Use:**

- Converting document_reader output into citation format
- Creating verifiable citations for responses

**Input Schema:**

| Field         | Type                    | Description                                   |
| ------------- | ----------------------- | --------------------------------------------- |
| source_file   | str                     | Document filename                             |
| line_numbers  | list[int] or (int, int) | Lines to cite                                 |
| quote         | str?                    | Optional quote text                           |
| context_lines | int                     | Lines before/after (default 2)                |
| format        | literal                 | "inline", "footnote", "reference", "markdown" |

**Citation Formats:**

| Format      | Example                      |
| ----------- | ---------------------------- |
| Single line | `[R1-2509178.docx:45]`       |
| Range       | `[R1-2509178.docx:45-52]`    |
| Multi       | `[R1-2509178.docx:45,67,89]` |

---

### Tool 3: evidence_grounding (MANDATORY)

**Purpose:** Zero-hallucination enforcement

**MANDATORY:** This tool MUST be called before generating any response with factual claims.

**When to Use:**

- ALWAYS before generating final response
- Verifying extracted positions
- Checking entire response draft for unsupported claims

**Input Schema:**

| Field                | Type       | Description                               |
| -------------------- | ---------- | ----------------------------------------- |
| claim                | str        | Statement to verify                       |
| source_files         | list[str]? | Limit scope for faster verification       |
| confidence_threshold | float      | Default 0.7                               |
| mode                 | literal    | "verify", "batch", "cite_response"        |
| strict_mode          | bool       | True: remove unverified, False: flag only |

**Verification Status Actions:**

| Status             | Confidence | Action                      |
| ------------------ | ---------- | --------------------------- |
| VERIFIED           | >=0.9      | Include claim               |
| VERIFIED           | 0.7-0.9    | Add "Based on documents..." |
| PARTIALLY_VERIFIED | 0.5-0.7    | Add caveat                  |
| UNVERIFIED         | <0.5       | DO NOT include              |
| CONTRADICTED       | Any        | DO NOT include              |

---

### Tool 4: corpus_search

**Purpose:** Structured search with company/meeting/date filters

**When to Use:**

- Finding documents by company, meeting, or date
- Searching content for specific topics
- Discovering relevant documents before deep reading

**Key Inputs:** query, company, meeting_id, agenda_item, date_range, file_pattern, max_results, output_format

**Output:** matches[] with file_path, line_number, content, score, metadata

---

### Tool 5: position_extractor

**Purpose:** Extract and classify company positions

**When to Use:**

- Extracting a company's position on a topic
- Comparing positions across companies
- Categorizing positions (support/oppose/neutral/conditional)

**Key Inputs:** company, topic, meeting_id, source_files, include_reasoning, output_format

**Output:** position, category (support/oppose/neutral/conditional), confidence, evidence[]

---

### Tool 6: temporal_tracker

**Purpose:** Track position evolution across meetings

**When to Use:**

- Tracking position changes over multiple meetings
- Identifying when positions shifted
- Building timelines for negotiation preparation

**Key Inputs:** company, topic, meetings[], date_range, include_unchanged, output_format

**Output:** timeline[], changes[] with change_type (strengthened/weakened/reversed/refined), trend

---

### Tool 7: consensus_analyzer

**Purpose:** Identify agreement/disagreement across companies

**When to Use:**

- Identifying areas of agreement
- Finding disputed points
- Preparing for meeting facilitation

**Key Inputs:** topic, companies[], meeting_id, agreement_threshold, min_companies, output_format

**Consensus Status:**

| Agreement % | Status         |
| ----------- | -------------- |
| >=90%       | CONSENSUS      |
| 70-90%      | NEAR_CONSENSUS |
| 50-70%      | DIVIDED        |
| <50%        | FRAGMENTED     |

---

## 8. Technology Stack

### Core Stack

| Component       | Technology      | Version  |
| --------------- | --------------- | -------- |
| Runtime         | Python          | 3.12+    |
| Package Manager | UV              | Latest   |
| Agent SDK       | claude-code-sdk | Latest   |
| Web Framework   | FastAPI         | ≥0.115.0 |
| Data Validation | Pydantic        | ≥2.10.0  |
| Logging         | structlog       | ≥24.4.0  |
| Excel Reading   | openpyxl        | ≥3.1.0   |

### Claude Agent SDK Built-in Tools

| Tool | Purpose         | Use in Research Agent   |
| ---- | --------------- | ----------------------- |
| Read | Read files      | Document content        |
| Glob | Find by pattern | Discover documents      |
| Grep | Search contents | Search within documents |
| Bash | Run commands    | pandoc conversion       |
| Task | Spawn subagents | Parallel analysis       |

### Development Dependencies

| Tool                   | Purpose                |
| ---------------------- | ---------------------- |
| pytest, pytest-asyncio | Testing                |
| mypy, pyright          | Type checking (strict) |
| ruff                   | Linting/formatting     |

### System Dependencies

| Dependency      | Purpose                                                        |
| --------------- | -------------------------------------------------------------- |
| Claude Code CLI | Agent SDK runtime (`npm install -g @anthropic-ai/claude-code`) |
| pandoc          | Document conversion (.docx, .pdf only; xlsx uses openpyxl)     |

### Installation

```bash
# 1. Install Claude Code CLI
npm install -g @anthropic-ai/claude-code
claude auth login

# 2. Install Python dependencies
uv sync

# 3. Configure environment
cp .env.example .env
# Edit .env with your ANTHROPIC_API_KEY and API_KEY
```

---

## 9. Agent Flows

### Basic Q&A Flow (US-1)

1. **corpus_search** → Find relevant documents
2. **document_reader** → Read with line numbers
3. **citation_generator** → Create [file:line] citations
4. **evidence_grounding** → Verify claims (MANDATORY)
5. **Generate Response** → Include only verified claims

### Zero Hallucination Flow (US-T1)

```
[Gather Evidence] → [Draft Response] → [Verify Each Claim]
                                              ↓
                    ┌─────────────────────────┴─────────────────────────┐
                    │              evidence_grounding                    │
                    │  VERIFIED → Include with citation                  │
                    │  PARTIALLY_VERIFIED → Include with caveat          │
                    │  UNVERIFIED → EXCLUDE from response                │
                    │  CONTRADICTED → EXCLUDE from response              │
                    └────────────────────────────────────────────────────┘
                                              ↓
                              [Generate Grounded Response]
```

### Error Handling (No Evidence Found)

When no documents match the query:

1. `corpus_search` returns empty
2. `evidence_grounding` returns `NO_EVIDENCE`
3. Response: "I could not find information about [topic] in the available documents. Would you like me to search for..."

---

## 10. Security & Configuration

### Authentication

**MVP:** Bearer Token (`Authorization: Bearer <API_KEY>`)

### Environment Variables

```bash
# Required
ANTHROPIC_API_KEY=your-api-key
API_KEY=your-bearer-token

# Optional
DATA_FOLDER=data/files
LOG_LEVEL=INFO
```

### Permission Modes

| Mode                | Description               | Use Case         |
| ------------------- | ------------------------- | ---------------- |
| `bypassPermissions` | Full autonomous execution | Production API   |
| `plan`              | Read-only, creates plans  | Complex research |
| `acceptEdits`       | Auto-approves edits       | Interactive      |

### Tool Restrictions

```python
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Glob", "Grep", "mcp__research__*"],
    disallowed_tools=["Write", "Edit", "Bash"],  # Read-only
    cwd="./data/files",
)
```

---

## 11. API Specification

### POST /v1/chat/completions

**Request:**

```json
{
  "model": "research-agent",
  "messages": [{ "role": "user", "content": "What is Qualcomm's position?" }],
  "temperature": 0.2,
  "max_tokens": 4096,
  "stream": true
}
```

**Non-Streaming Response:**

```json
{
  "id": "chatcmpl-abc123",
  "model": "research-agent",
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "Qualcomm proposes... [R1-2509178.docx:45]\n\nSources: 12 docs | Confidence: High"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": { "prompt_tokens": 150, "completion_tokens": 320 }
}
```

**Streaming Response (SSE):**

Streaming responses include tool progress events interleaved with content chunks:

```
data: {"object": "chat.progress", "type": "tool_start", "tool": "Glob", "tool_use_id": "toolu_123", "message": "Finding files...", "timestamp": 1735489157.96, "id": "chatcmpl-abc123", "created": 1735489151}

data: {"object": "chat.progress", "type": "tool_complete", "tool": "Glob", "tool_use_id": "toolu_123", "message": "Found files", "timestamp": 1735489157.98, "id": "chatcmpl-abc123", "created": 1735489151}

data: {"id": "chatcmpl-abc123", "object": "chat.completion.chunk", "created": 1735489151, "model": "research-agent", "choices": [{"index": 0, "delta": {"content": "Found 47 .docx files..."}, "finish_reason": null}]}

data: {"id": "chatcmpl-abc123", "object": "chat.completion.chunk", "created": 1735489151, "model": "research-agent", "choices": [{"index": 0, "delta": {}, "finish_reason": "stop"}]}

data: [DONE]
```

**Progress Event Types:**

| Type            | Description              | Message Example    |
| --------------- | ------------------------ | ------------------ |
| `tool_start`    | Tool execution beginning | "Finding files..." |
| `tool_complete` | Tool execution finished  | "Found files"      |
| `tool_error`    | Tool execution failed    | "Glob failed"      |

**Progress Event Schema:**

```python
class ProgressEvent(BaseModel):
    object: Literal["chat.progress"]  # Event type identifier
    type: ProgressEventType           # tool_start, tool_complete, tool_error
    tool: str                         # Tool name (e.g., "Glob", "Read")
    tool_use_id: str | None           # Correlation ID for Pre/Post matching
    message: str                      # Human-readable progress message
    timestamp: float                  # Unix timestamp
```

### Health Endpoints

| Endpoint          | Response                                          |
| ----------------- | ------------------------------------------------- |
| GET /health       | `{"status": "healthy", "version": "0.1.0"}`       |
| GET /health/ready | `{"status": "ready", "data_folder_exists": true}` |

---

## 12. Success Criteria

### MVP Success Definition

1. Ask factual question → Receive accurate, cited answer
2. Compare companies → Get evidence for each position
3. Track evolution → See timeline with meeting citations
4. Query without evidence → Receive "Not found in documents"
5. Verify citation → Check source file and line matches

### Quality Indicators

| Metric               | Target      |
| -------------------- | ----------- |
| Hallucination rate   | 0%          |
| Citation accuracy    | 100%        |
| Query response time  | <30 seconds |
| Type check pass rate | 100%        |
| Test coverage        | >80%        |

---

## 13. Implementation Phases

### Phase 1: Foundation (Complete)

| Component           | Status |
| ------------------- | ------ |
| FastAPI entry point | ✅     |
| Pydantic Settings   | ✅     |
| structlog logging   | ✅     |
| Request middleware  | ✅     |
| Exception handlers  | ✅     |
| Health endpoints    | ✅     |
| document_utils      | ✅     |
| metadata_extractor  | ✅     |

### Phase 2: Core MCP Tools

| Tool               | Status | Notes                                                              |
| ------------------ | ------ | ------------------------------------------------------------------ |
| document_reader    | ✅     | Line-numbered parsing with section extraction                      |
| citation_generator | ✅     | [file:line] citation formatting                                    |
| evidence_grounding | ✅     | Zero hallucination verification (MANDATORY)                        |
| corpus_search      | ✅     | Structured search with company/meeting filters                     |
| position_extractor | ✅     | Company stance classification (support/oppose/neutral/conditional) |
| temporal_tracker   | ✅     | Position evolution tracking with change/trend detection            |
| consensus_analyzer | ✅     | Agreement/disagreement detection across companies                  |

### Phase 3: API Integration

| Component              | Status | Notes                                    |
| ---------------------- | ------ | ---------------------------------------- |
| agent.py (SDK wrapper) | ✅     | ResearchClient with MCP tools            |
| mcp_server.py          | ✅     | 7 tools registered (MVP complete)        |
| chat feature           | ✅     | POST /v1/chat/completions                |
| hooks.py               | ✅     | PreToolUse/PostToolUse for audit logging |
| Authentication         | ✅     | Bearer token via HTTPBearer dependency   |
| Streaming progress     | ✅     | Tool progress events via SSE             |

### Phase 4: Frontend Chat Interface

| Component | Status | Notes |
| --------- | ------ | ----- |
| Project setup (Vite + React + TypeScript) | ✅ | Phase 1 complete |
| Tailwind CSS + shadcn/ui | ✅ | Phase 1 complete |
| API types (OpenAI-compatible) | ✅ | Phase 2 complete |
| Supabase client + database types | ✅ | Phase 2 complete |
| SSE streaming API layer | ✅ | Phase 2 complete |
| useStreamingChat hook | ✅ | Phase 2 complete |
| ToolProgress component | ✅ | Phase 2 complete |
| Shadcn UI components (50) | ✅ | Phase 3 complete (copied) |
| Chat components (9) | ✅ | Phase 4 adapted for streaming API |
| Auth components | ✅ | Phase 3 complete (copied) |
| Sidebar components | ✅ | Phase 3 complete (copied) |
| Admin components | ✅ | Phase 3 complete (copied) |
| Pages (5) | ✅ | Phase 3 complete (copied) |
| Hooks (5) | ✅ | Phase 3 complete (copied) |
| Component adaptation | ✅ | Phase 4 complete - MessageHandling uses useStreamingChat |
| App.tsx routing | ✅ | Phase 4 complete - React Router + providers |
| Build passing | ✅ | Phase 4 complete - TypeScript + Vite build pass |

### Phase 5: Chat Display Improvements

| Component | Status | Notes |
| --------- | ------ | ----- |
| Content classifier utility | ✅ | Pattern-based answer/reasoning separation |
| MarkdownContent component | ✅ | Shared markdown renderer (DRY refactoring) |
| Tabbed AI messages | ✅ | Answer/Reasoning tabs for structured responses |
| Enhanced ToolProgress | ✅ | Container styling with "Agent Activity" header |
| Chat color CSS variables | ✅ | Light/dark mode support |
| Fade-in animation | ✅ | Smooth transitions for new content |
| Streaming display fix | ✅ | Fixed duplicate message after completion |

### Current State Summary

**Backend (Complete - 7/7 tools):**

- Phase 1 MVP tools: `document_reader`, `citation_generator`, `evidence_grounding`
- Phase 2 tools: `corpus_search`, `position_extractor`
- Phase 3 tools: `temporal_tracker`, `consensus_analyzer`
- Claude Agent SDK integration with ResearchClient
- Chat completions API endpoint
- PreToolUse/PostToolUse hooks for audit logging
- Bearer token authentication via HTTPBearer dependency
- Streaming tool progress events via SSE
- Integration testing with real document corpus (12 tests, all passing)

**Frontend (Complete - Phase 5):**

- Project scaffolding with Vite + React 19 + TypeScript
- Tailwind CSS + shadcn/ui component library (47 components after cleanup)
- OpenAI-compatible API types matching backend schemas
- SSE streaming parser for `chat.progress` and `chat.completion.chunk` events
- `useStreamingChat` hook for tool progress state management
- `ToolProgress` component for visual tool execution feedback
- Supabase integration for conversation persistence
- All UI components adapted for streaming API (Phase 4):
  - Chat: ChatInput, ChatLayout, MessageHandling (rewritten), MessageList, MessageItem
  - Auth: AuthForm, AuthCallback
  - Sidebar: ChatSidebar, SettingsModal
  - Admin: ConversationsTable, UsersTable, conversations/
  - Pages: Login, Admin, Index, NotFound, Chat
  - Hooks: useAuth, useAdmin, useConversationRating, use-mobile, use-toast
- App.tsx with React Router, TanStack Query, Auth providers
- Chat display improvements (Phase 5):
  - Tabbed AI messages separating reasoning from answers
  - Content classifier for pattern-based answer detection
  - Shared MarkdownContent component (DRY refactoring)
  - Enhanced ToolProgress with container and "Agent Activity" header
  - Chat color CSS variables for light/dark mode

**Build Status:** ✅ Passing (TypeScript + Vite build, e2e tested)

### Next Steps

1. Docker containerization for easy deployment
2. Response caching with Redis for performance
3. Production deployment configuration

---

## 14. Future Considerations

### Post-MVP Enhancements

**Infrastructure:**

- Response caching with Redis
- Session persistence
- ~~Streaming responses~~ ✅ Implemented (tool progress via SSE)
- Document upload API
- Docker containerization

**Advanced Features:**

- Semantic search fallback
- Knowledge graph
- Multi-user with RBAC
- ~~Web UI~~ ✅ Complete (Phase 4 - e2e tested, streaming chat working)

### Integration Opportunities

- 3GPP document portal
- Meeting calendar sync
- Slack/Teams integration

---

## 15. Risks & Mitigations

| Risk                         | Impact              | Mitigation                                       |
| ---------------------------- | ------------------- | ------------------------------------------------ |
| Pandoc conversion failures   | Missing content     | Log errors, return partial with warning          |
| xlsx conversion timeout      | Missing spreadsheet | ✅ RESOLVED: Use openpyxl (~21ms vs 60s timeout) |
| Large corpus performance     | Slow responses      | max_results limit, parallel processing           |
| Metadata extraction failures | Wrong filtering     | Multiple formats, filename fallback              |
| Citation line drift          | Wrong references    | Include quote snippet, timestamp docs            |
| Context limit exceeded       | Incomplete analysis | response_format, auto-chunking                   |

---

## 16. Appendix

### Validation Commands

```bash
# Install
npm install -g @anthropic-ai/claude-code
uv sync

# Type checking
uv run mypy backend/app/
uv run pyright backend/app/

# Linting
uv run ruff check . --fix
uv run ruff format .

# Tests
uv run pytest -v

# Development server
cd backend && uv run uvicorn app.main:app --reload --port 8173

# Test endpoints
curl http://localhost:8173/health
curl -X POST http://localhost:8173/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "List documents"}]}'
```

### Key References

| Resource              | URL                                                                             |
| --------------------- | ------------------------------------------------------------------------------- |
| Anthropic Tool Design | https://www.anthropic.com/engineering/writing-tools-for-agents                  |
| Claude Agent SDK      | https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk |
| FastAPI Docs          | https://fastapi.tiangolo.com/                                                   |
