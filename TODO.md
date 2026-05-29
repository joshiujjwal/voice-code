# VoiceCode — Task Breakdown

## How to Use This File

Workflow per task:
1. **Write tests FIRST** (red phase) — test goes in `tests/` mirroring `src/`
2. **Implement until tests pass** (green phase)
3. **Review diff manually** — read every line you changed
4. **Commit** with a descriptive message: `feat(audio): add silence detection`
5. **Update CLAUDE.md** if you learned something non-obvious (compound loop)
6. **Check off the item** and move to the next

> ⛔ Do NOT skip phases. Each phase is an evidence gate — passing tests + human review before proceeding.

---

## Phase 0: Foundation ⬜

- [ ] Init `package.json` with workspaces (root, `packages/api`, `packages/ui`)
- [ ] Configure TypeScript (`tsconfig.json`, strict mode, path aliases)
- [ ] Add ESLint (`@typescript-eslint/recommended`) + Prettier
- [ ] Set up Vitest for both `api` and `ui` packages
- [ ] Write smoke test: `1 + 1 === 2` — confirm test runner works
- [ ] Add `npm run dev` (concurrently: Vite + ts-node Express)
- [ ] Add `npm run test`, `npm run lint`, `npm run build` to root scripts
- [ ] Create `.env.example` with required keys documented
- [ ] Set up GitHub Actions CI: install → lint → test on push to `main`
- [ ] Review all AI config files (CLAUDE.md, AGENTS.md, copilot-instructions.md)

**Evidence gate**: CI green on `main` with smoke test passing ✅

---

## Phase 1: Voice Capture Pipeline ⬜

- [ ] Write tests for `useVoice` hook: start/stop, state transitions, error cases
- [ ] Implement `useVoice` React hook using Web Speech API (SpeechRecognition)
- [ ] Write tests for `AudioRecorder` class: MediaRecorder start/stop/blob output
- [ ] Implement `AudioRecorder` class with configurable mime type + sample rate
- [ ] Write tests for silence detection utility (energy threshold logic)
- [ ] Implement silence detector — auto-stop after N ms of silence
- [ ] Wire `useVoice` → `AudioRecorder` → auto-stop in a single composable hook
- [ ] Manual test: record "create a function that adds two numbers" in browser
- [ ] Write integration test: full capture → blob → base64 round-trip

**Evidence gate**: Integration test green; manual recording works in browser ✅

---

## Phase 2: Transcription ⬜

- [ ] Write unit tests for Whisper API client (mock OpenAI responses)
- [ ] Implement `src/api/transcribe.ts` POST `/api/transcribe` — accepts audio blob
- [ ] Write tests for Web Speech API transcription fallback path
- [ ] Implement fallback: if no API key, use browser SpeechRecognition transcript
- [ ] Write tests for transcript sanitizer (strip filler words, normalize punctuation)
- [ ] Implement transcript sanitizer
- [ ] Write integration test: audio blob → POST → transcript string returned
- [ ] Manual test: dictate a sentence, verify clean transcript in UI

**Evidence gate**: Integration test green; manual transcription round-trip works ✅

---

## Phase 3: Code Generation ⬜

- [ ] Write unit tests for prompt builder (given transcript + language → prompt string)
- [ ] Implement `src/codegen/promptBuilder.ts` — constructs system + user prompts
- [ ] Write tests for LLM client wrapper (mock OpenAI/Anthropic SDK, test retry logic)
- [ ] Implement `src/codegen/llmClient.ts` with configurable provider (OpenAI / Anthropic)
- [ ] Write tests for response parser: extract code blocks, language tag, explanation
- [ ] Implement `src/codegen/responseParser.ts`
- [ ] Write tests for POST `/api/generate` endpoint (mock LLM client)
- [ ] Implement `/api/generate` controller: transcript in → code block out
- [ ] Write integration test: transcript → generate → code string returned
- [ ] Manual test: dictate "create a debounce utility in TypeScript", verify output

**Evidence gate**: Integration test green; generate endpoint returns valid code ✅

---

## Phase 4: UI — Voice + Code Display ⬜

- [ ] Write tests for `VoiceButton` component: idle/recording/processing states
- [ ] Implement `VoiceButton` — pulsing animation when recording
- [ ] Write tests for `TranscriptDisplay` component: shows live/final transcript
- [ ] Implement `TranscriptDisplay`
- [ ] Write tests for `CodeOutput` component: syntax highlighting, copy button
- [ ] Implement `CodeOutput` using `react-syntax-highlighter` (Prism)
- [ ] Write tests for `useCodegen` hook: orchestrates capture → transcribe → generate
- [ ] Implement `useCodegen` hook
- [ ] Write tests for main `Editor` page layout
- [ ] Implement `Editor` page wiring all components
- [ ] Manual test: full end-to-end — speak → see transcript → see code

**Evidence gate**: All component tests green; full e2e manual flow works ✅

---

## Phase 5: Language + Context Controls ⬜

- [ ] Write tests for language selector component (TypeScript, Python, Rust, Go, etc.)
- [ ] Implement language selector — updates codegen prompt
- [ ] Write tests for context panel: user can paste existing code as context
- [ ] Implement context panel — appended to prompt as "existing code" block
- [ ] Write tests for history sidebar: stores last N (voice, transcript, code) triples
- [ ] Implement history sidebar with local-storage persistence
- [ ] Manual test: switch language mid-session, verify output changes correctly

**Evidence gate**: Tests green; language switching + context injection verified manually ✅

---

## Phase 6: Polish & Harden ⬜

- [ ] Add error boundary around `Editor` page — graceful LLM/network failure UI
- [ ] Add rate-limit middleware to Express API
- [ ] Add request validation (zod) for all API routes
- [ ] Add E2E test with Playwright: full voice → code flow in headless browser
- [ ] Lighthouse accessibility audit — target score ≥ 90 (keyboard nav, ARIA labels)
- [ ] Bundle size analysis — keep initial JS < 200 KB gzipped
- [ ] Security review: no API keys in frontend bundle, CORS locked to localhost in dev
- [ ] Write `docs/adr/0002-llm-provider-abstraction.md`

**Evidence gate**: Playwright E2E green; Lighthouse ≥ 90; bundle size within budget ✅

---

## Phase 7: Ship ⬜

- [ ] Write `CHANGELOG.md` — document all features added
- [ ] Tag `v0.1.0` release
- [ ] Deploy API to Railway / Render (or Docker compose instructions)
- [ ] Deploy UI to Vercel / Netlify
- [ ] Update README with live demo URL
- [ ] Post in `#projects` — include screenshot of first successful voice→code session

**Evidence gate**: Both services live; demo URL reachable from a clean browser ✅

---

## Parking Lot 🅿️

- Offline mode with local Whisper (whisper.cpp WASM)
- VSCode extension that injects code directly into editor
- Multi-turn conversation: "now make it async" → refines previous output
- Team mode: shared session with multiple speakers
- Custom vocabulary / project-specific terms (variable names, domain jargon)
- Voice-activated git commits ("commit this with message…")

---

## Lessons Learned 📝

> Add non-obvious discoveries here as you build. This is how context compounds across sessions.

<!-- Example:
- Web Speech API `onresult` fires multiple times for the same utterance — always use `event.results[event.resultIndex]` not index 0
- OpenAI Whisper returns timestamps; strip them before passing to code gen prompt
-->
