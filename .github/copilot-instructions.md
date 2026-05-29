# GitHub Copilot Instructions — VoiceCode

## Project Context

VoiceCode is a voice-driven coding assistant. Developers speak natural-language commands and receive generated, typed code. Stack: **TypeScript + React 18 + Vite** (frontend), **Node.js + Express** (backend), **OpenAI Whisper** (transcription), **GPT-4o / Claude** (code generation).

---

## Stack Conventions

### TypeScript (all files)
- `strict: true` — never suggest `any` without a `// eslint-disable` + explanation
- Derive types from Zod schemas with `z.infer<>` — don't write duplicate type definitions
- Use path aliases (`@api/`, `@ui/`, `@codegen/`, `@audio/`) — never suggest relative `../../` imports that cross package boundaries

### React (src/ui/)
- Functional components only
- Custom hooks in `src/ui/hooks/` — always prefix `use`
- Props typed as `type Props = { ... }` above the component, not inline
- No global state library — `useState` / `useReducer` + context only when necessary
- `useEffect` — keep each effect single-purpose; add a comment explaining the dependency array if non-obvious

### Express (src/api/)
- Route handlers should be ≤ 20 lines; extract logic to `services/`
- All handlers `async` — always use the `asyncHandler` wrapper
- Validate request bodies with Zod before any other logic
- Never log full request bodies — they may contain audio data or prompts

### LLM / Codegen (src/codegen/)
- `promptBuilder.ts` always returns `{ system: string; user: string }` — never a single string
- `responseParser.ts` must never throw — return `{ code, parsed: boolean }` even on malformed input
- Retry logic belongs in `llmClient.ts` only — not in route handlers

---

## Testing Conventions

- Write tests BEFORE implementation (red/green TDD)
- Test file location: `tests/<same-path-as-src>/<filename>.test.ts`
- Mock all external APIs (`openai`, `@anthropic-ai/sdk`, `MediaRecorder`)
- Use `vi.mock()` (Vitest) — not Jest
- React component tests: `@testing-library/react` + `userEvent`, assert on DOM state not internals
- Test names: describe observable behavior — `"returns sanitized transcript when filler words present"` not `"calls sanitize()"`

---

## What NOT to Do

- Do NOT suggest class components in React
- Do NOT add `console.log` to production code paths — use a logger utility
- Do NOT suggest `fetch()` in Express handlers — use the OpenAI/Anthropic SDK
- Do NOT remove or skip existing tests
- Do NOT suggest storing audio blobs server-side — privacy constraint (see spec.md §7)
- Do NOT suggest inline styles in React — use Tailwind classes
- Do NOT refactor unrelated files in the same change — keep scope tight
- Do NOT add dependencies without checking bundle size impact (target: < 200 KB gzipped)
