# AGENTS.md — VoiceCode

## Setup

```bash
# Install (Node.js 20+ required)
npm install

# Environment
cp .env.example .env
# Set OPENAI_API_KEY in .env (required for Whisper + GPT-4o)
# Optionally set ANTHROPIC_API_KEY if using Claude

# Run dev servers
npm run dev       # starts Vite (5173) + Express (3001) concurrently

# Verify setup
npm test          # all tests should pass
npm run typecheck # no type errors
npm run lint      # no lint errors
```

---

## Code Style

### TypeScript
- `strict: true` — no `any` without an explanatory comment
- Prefer `type` over `interface` for plain data shapes; use `interface` for extensible contracts
- Always use `z.infer<typeof Schema>` to derive types from Zod schemas — don't duplicate
- Path aliases must be used: `@api/`, `@ui/`, `@codegen/`, `@audio/` (never relative `../../`)
- No barrel files (`index.ts` re-exports) unless the folder has > 5 exports

### Naming
- Files: `camelCase.ts` for modules, `PascalCase.tsx` for React components
- Variables/functions: `camelCase`; Classes: `PascalCase`; Constants: `SCREAMING_SNAKE_CASE`
- React hooks: always prefix `use` — `useVoice`, `useCodegen`
- API route files named after the resource: `transcribe.ts`, `generate.ts`

### React
- Functional components only — no class components
- One component per file; co-locate test file in `tests/ui/components/`
- Prop types defined as `type Props = { ... }` directly above the component
- Use `React.FC` sparingly — prefer explicit return types
- State management: React `useState` / `useReducer` for local state; no global state library in v0.1.0
- Avoid `useEffect` with multiple responsibilities — split into focused effects

### API (Express)
- Route handlers delegate to service functions — keep handlers < 20 lines
- All handlers are `async` — wrap with `asyncHandler` middleware to avoid unhandled promise rejections
- Validate all request bodies with Zod before any business logic
- HTTP status codes must be semantically correct (400 client error, 500 server error, 429 rate limit)

---

## Testing

### Philosophy
1. **Write tests FIRST** — red before green, always
2. Tests are documentation — test names describe behavior, not implementation
3. A test that always passes is worse than no test — assert on real outputs

### Commands
```bash
npm test                    # all tests, watch mode off
npm test -- --watch         # watch mode
npm test -- --coverage      # coverage report
npm test -- src/api         # specific directory
```

### Rules
- Unit tests: mock ALL external I/O (OpenAI SDK, Anthropic SDK, MediaRecorder, fetch)
- Integration tests: use `supertest` against Express app — no live server, no live API calls
- React tests: use `@testing-library/react` + `userEvent`; test from user perspective, not implementation
- Test files: `tests/<mirror-of-src>/<filename>.test.ts`
- Coverage target: ≥ 80% lines — focus on logic-heavy modules (`promptBuilder`, `responseParser`, `llmClient`)

### Mocking conventions
```typescript
// Mock external SDK
vi.mock('openai', () => ({
  default: vi.fn().mockImplementation(() => ({
    audio: { transcriptions: { create: vi.fn() } },
    chat: { completions: { create: vi.fn() } }
  }))
}));

// Reset mocks between tests
beforeEach(() => vi.clearAllMocks());
```

---

## Pull Request Instructions

### Before opening a PR
1. `npm test` — all tests green
2. `npm run typecheck` — zero TypeScript errors
3. `npm run lint` — zero lint errors
4. `git diff main` — read every line of your diff

### PR description must include
- **What**: one-sentence summary
- **Why**: link to TODO.md task or issue
- **Evidence**: paste test output (`npm test -- --reporter=verbose`) or attach CI screenshot
- **Manual test**: describe what you verified by hand (e.g., "recorded 'create a debounce function', got correct TypeScript output")

### PR rules
- One feature/fix per PR — no scope creep
- Do NOT refactor unrelated code in the same PR
- Do NOT remove existing tests — if a test is wrong, fix it with a comment explaining why
- Add a `// CHANGEME:` comment anywhere you made a deliberate trade-off for speed over quality
