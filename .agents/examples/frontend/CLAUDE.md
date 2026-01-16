# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

React + TypeScript frontend for the AI Agent Mastery course. Provides a chat interface for interacting with a Pydantic AI backend agent, with Supabase for auth/data and Langfuse for observability.

## Commands

```bash
# Development
npm install        # Install dependencies (uses npm, not pnpm - see package-lock.json)
npm run dev        # Start dev server at http://localhost:8080

# Building
npm run build      # Production build
npm run build:dev  # Development build
npm run preview    # Preview production build

# Quality
npm run lint       # Run ESLint

# Testing (Playwright)
npm run test           # Run all tests headless
npm run test:ui        # Run with Playwright UI
npm run test:headed    # Run tests in visible browser
```

## Architecture

### Data Flow
1. User authenticates via Supabase Auth (email/password or Google OAuth)
2. Chat messages sent to backend via `VITE_AGENT_ENDPOINT` with SSE streaming
3. Conversations/messages persisted in Supabase by the backend
4. Frontend fetches conversation history from Supabase directly
5. User feedback submitted to Langfuse via `submitScore()`

### Key Modules

**Authentication** (`src/hooks/useAuth.tsx`)
- `AuthProvider` wraps the app with auth context
- `useAuth()` hook provides `user`, `session`, `signIn`, `signUp`, `signOut`
- OAuth callback handled at `/auth/callback`

**Chat State** (`src/pages/Chat.tsx`)
- Orchestrates `useConversationManagement` and `useMessageHandling` hooks
- Conversation rating triggered periodically via `useConversationRating`

**API Communication** (`src/lib/api.ts`)
- `sendMessage()` - Posts to agent endpoint, handles both streaming (SSE) and non-streaming responses
- `fetchConversations()` / `fetchMessages()` - Direct Supabase queries
- Streaming controlled by `VITE_ENABLE_STREAMING` env var

**Feedback** (`src/lib/langfuse.ts`)
- `submitScore()` sends user ratings to Langfuse traces
- Gracefully degrades if Langfuse not configured

### Component Structure
```
src/components/
├── chat/           # ChatLayout, ChatInput, MessageList, MessageHandling
├── sidebar/        # ChatSidebar, SettingsModal
├── admin/          # Admin dashboard (ConversationsTable, UsersTable)
├── auth/           # AuthForm, AuthCallback
└── ui/             # Shadcn UI components (do not modify directly)
```

### Database Types (`src/types/database.types.ts`)
- `Conversation` - id, user_id, session_id, title
- `Message` - session_id, message (type: human|ai, content, trace_id)
- Messages queried by `session_id`, not `computed_session_user_id`

## Environment Variables

```env
VITE_SUPABASE_URL=           # Supabase project URL
VITE_SUPABASE_ANON_KEY=      # Supabase anon key (not service key)
VITE_AGENT_ENDPOINT=         # Backend agent API (e.g., http://localhost:8001/api/pydantic-agent)
VITE_ENABLE_STREAMING=true   # Enable SSE streaming (false for n8n backend)
VITE_LANGFUSE_PUBLIC_KEY=    # Optional: Langfuse public key for feedback
VITE_LANGFUSE_HOST=          # Optional: Langfuse host URL
```

## Testing

Tests use Playwright with Supabase mocks defined in `tests/mocks.ts`. The mock system:
- Injects `window.__MOCK_AUTH__` and `window.__supabaseMock__` via `addInitScript`
- `setupUnauthenticatedMocks()` for login flow tests
- `setupAuthenticatedMocks()` for authenticated state tests
- `setupAgentAPIMocks()` mocks the streaming agent response

Test server runs on port 8083 (configured in `playwright.config.ts`).

## Path Alias

`@/` maps to `src/` (configured in `vite.config.ts` and `tsconfig.json`).
