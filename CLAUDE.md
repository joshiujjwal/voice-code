# CLAUDE.md — VoiceCode

> Context file for AI coding agents. Keep this under 200 lines.  
> Update when you discover non-obvious conventions or gotchas.

---

## Commands

```bash
# Install all dependencies (monorepo root)
npm install

# Start dev servers (Vite UI + Express API concurrently)
npm run dev

# Run all tests
npm test

# Run tests for a specific package
npm run test --workspace=packages/api
npm run test --workspace=packages/ui

# Lint everything
npm run lint

# Type-check everything
npm run typecheck

# Build for production
npm run build
```

## Directory Map

```
src/
├── api/            # Express app: routes, middleware, controllers
│   ├── routes/     # Route handlers (transcribe.ts, generate.ts, health.ts)
│   ├── middleware/ # Rate limiting, validation (zod), error handler
│   └── services/   # LLM client, Whisper client (pure functions, no Express)
├── audio/          # Browser-side audio logic (no DOM — pure TS)
│   ├── recorder.ts       # AudioRecorder class wrapping MediaRecorder
│   └── silence.ts        # Silence detection (energy threshold)
├── codegen/
│   ├── promptBuilder.ts  # Constructs system + user prompts
│   ├── llmClient.ts      # OpenAI / Anthropic abstraction
│   └── responseParser.ts # Extracts code blocks from LLM markdown
└── ui/
    ├── components/   # Atomic/reusable components
    ├── hooks/        # useVoice, useCodegen, useHistory
    └── pages/        # Editor (main page), Settings
tests/               # Mirror of src/ — same file names, .test.ts suffix
docs/
├── spec.md          # THE source of truth for feature requirements
└── adr/             # Architecture Decision Records
```

## Workflow

1. **Read `TODO.md`** — find the next unchecked task in the current phase
2. **Run tests first**: `npm test` — establish baseline (no pre-existing failures)
3. **Write the failing test** before any implementation
4. **Implement** until `npm test` is green
5. **Lint + type-check**: `npm run lint && npm run typecheck`
6. **Review your diff** manually: `git diff --stat && git diff`
7. **Commit** with conventional commit message: `feat(audio): add silence detection`
8. **Update this file** if you learned something non-obvious

## Environment Variables

```bash
# Required for Whisper transcription
OPENAI_API_KEY=sk-...

# OR use Anthropic for code generation (OPENAI_API_KEY still needed for Whisper)
ANTHROPIC_API_KEY=sk-ant-...

# Switch LLM provider: 'openai' (default) | 'anthropic'
LLM_PROVIDER=openai

# LLM model override
LLM_MODEL=gpt-4o

# API server port
PORT=3001

# Allowed CORS origins (comma-separated)
CORS_ORIGINS=http://localhost:5173
```

## Non-Obvious Conventions

### Monorepo layout
- `packages/api` — Express backend (Node.js, no browser APIs)
- `packages/ui` — React frontend (Vite, browser APIs OK)
- `src/audio/` — Pure TypeScript; no DOM imports. Browser environment assumed but not imported directly (so it can be unit-tested with mocks)

### Audio
- `AudioRecorder` wraps `MediaRecorder` — always check `MediaRecorder.isTypeSupported()` before constructing with a mime type; Safari needs `audio/mp4`
- Web Speech API `onresult` fires multiple times per utterance — always read `event.results[event.resultIndex]`, never index 0 blindly
- Silence detection runs on `AnalyserNode.getByteTimeDomainData()` — it measures waveform energy, not amplitude peak

### LLM / Codegen
- `promptBuilder.ts` ALWAYS outputs `{ system, user }` — never a single concatenated string
- `responseParser.ts` extracts the FIRST fenced code block from markdown. If no fenced block exists, return the whole response as `code` with a `parsed: false` flag — never throw
- Retry logic lives in `llmClient.ts` only — no retries in route handlers

### Testing
- Use `vi.mock()` for all external services (OpenAI SDK, Anthropic SDK, MediaRecorder)
- API integration tests use `supertest` against the Express app directly — no live port
- React component tests use `@testing-library/react` with `userEvent` (not `fireEvent`)

### TypeScript
- `strict: true` always — no `any` without a comment explaining why
- Use `zod` schemas for all API request/response validation — infer TS types from schemas (`z.infer<>`)
- Path aliases: `@api/*` → `src/api/*`, `@ui/*` → `src/ui/*`, `@codegen/*` → `src/codegen/*`

## Gotchas

- **Browser mic permission**: `getUserMedia` must be called from a user gesture handler — don't call it on mount
- **Whisper audio format**: Only send `audio/webm` or `audio/mp4` — `audio/ogg` is not accepted by the Whisper API
- **CORS**: In dev, Vite runs on `:5173`, Express on `:3001` — always configure CORS in Express, don't rely on browser defaults
- **SSE for streaming**: If you add streaming code generation, use `res.write()` with `text/event-stream` — don't use WebSockets for this

## Lessons Learned

> Add non-obvious discoveries here as you build.
