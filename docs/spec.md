# VoiceCode — Feature Specification

**Version**: 0.1.0  
**Status**: Draft  
**Last updated**: 2025

---

## 1. Overview

VoiceCode converts spoken natural-language commands into syntactically correct, typed code. A developer says "create a TypeScript function that debounces a callback" and within ~3 seconds sees a ready-to-paste implementation in their chosen language.

### Problem Statement

- Typing speed is a bottleneck for experienced developers who think faster than they type
- Accessibility barrier: repetitive strain injuries and motor disabilities prevent many from coding efficiently
- LLM code assistants require leaving the keyboard flow to type prompts — VoiceCode removes that friction

### Non-Goals (v0.1.0)

- Voice-activated file saves or git operations (Parking Lot)
- IDE plugin / extension (Parking Lot)
- Real-time streaming of voice directly into existing code files

---

## 2. Functional Requirements

### 2.1 Voice Capture
- [ ] User can start/stop voice recording with a single button or keyboard shortcut (`Space`)
- [ ] Recording auto-stops after 2 seconds of silence (configurable)
- [ ] Visual indicator shows: idle / recording / processing states
- [ ] Maximum recording length: 60 seconds (hard cap)
- [ ] Browser compatibility: Chrome 90+, Firefox 115+, Safari 16+ (WebRTC/MediaRecorder)

### 2.2 Transcription
- [ ] Primary: Audio blob sent to OpenAI Whisper via backend `/api/transcribe`
- [ ] Fallback: Browser-native `SpeechRecognition` API when no API key configured
- [ ] Transcript displayed live (streaming interim results where available)
- [ ] User can manually edit transcript before triggering code generation
- [ ] Transcript sanitized: filler words ("um", "uh", "like") stripped by default (toggleable)

### 2.3 Code Generation
- [ ] POST `/api/generate` accepts: `{ transcript, language, context?, style? }`
- [ ] System prompt instructs LLM to output ONLY code + a brief (1 sentence) explanation
- [ ] Response parsed to extract: `code`, `language`, `explanation`
- [ ] Supported output languages: TypeScript, JavaScript, Python, Go, Rust, Java, C#, SQL
- [ ] Provider abstraction: OpenAI GPT-4o or Anthropic Claude, switchable via env var

### 2.4 Code Output Display
- [ ] Syntax-highlighted code block (Prism via `react-syntax-highlighter`)
- [ ] One-click copy to clipboard with visual confirmation
- [ ] Language badge shown on code block
- [ ] LLM explanation shown below code (collapsible)
- [ ] "Regenerate" button to re-run generation with same transcript

### 2.5 Context & Language Controls
- [ ] Language selector dropdown (persisted to localStorage)
- [ ] "Context" panel: paste existing code to give LLM project context
- [ ] Context is prepended to prompt as a fenced code block with annotation
- [ ] Session history: last 10 (transcript, code) pairs stored in localStorage
- [ ] History panel: click any past entry to restore it

### 2.6 Accessibility
- [ ] All interactive elements keyboard accessible
- [ ] ARIA labels on `VoiceButton`, `CodeOutput`, `TranscriptDisplay`
- [ ] High-contrast mode respects `prefers-color-scheme`
- [ ] Screen reader announces state transitions (recording started/stopped)

---

## 3. Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| Transcription latency (Whisper) | < 2s for 10s of audio |
| Code generation latency (GPT-4o) | < 4s p95 |
| Initial page load (JS bundle) | < 200 KB gzipped |
| API error rate | < 1% under normal load |
| Lighthouse accessibility score | ≥ 90 |
| Test coverage | ≥ 80% lines (unit + integration) |

---

## 4. Data Model

```typescript
// Audio capture
interface RecordingSession {
  id: string;           // uuid
  startedAt: Date;
  stoppedAt?: Date;
  durationMs?: number;
  blob?: Blob;          // raw audio, not persisted server-side
  mimeType: string;     // 'audio/webm;codecs=opus'
}

// Transcription
interface TranscriptResult {
  raw: string;          // unprocessed LLM/browser transcript
  sanitized: string;    // filler words stripped
  confidence?: number;  // 0-1, from Whisper word probabilities
  language?: string;    // detected spoken language
}

// Code generation request
interface GenerateRequest {
  transcript: string;
  language: CodeLanguage;
  context?: string;     // user-pasted existing code
  style?: 'concise' | 'verbose' | 'commented';
}

// Code generation response
interface GenerateResponse {
  code: string;
  language: CodeLanguage;
  explanation: string;
  model: string;        // e.g. 'gpt-4o', 'claude-3-5-sonnet'
  tokensUsed: number;
}

// History entry (localStorage)
interface HistoryEntry {
  id: string;
  timestamp: Date;
  transcript: string;
  language: CodeLanguage;
  code: string;
  explanation: string;
}

type CodeLanguage = 'typescript' | 'javascript' | 'python' | 'go' | 'rust' | 'java' | 'csharp' | 'sql';
```

---

## 5. API Design

### POST `/api/transcribe`
```
Content-Type: multipart/form-data
Body: audio (Blob, required), language (string, optional hint)

Response 200:
{ transcript: string, confidence?: number, durationMs: number }

Response 400: { error: "audio_required" | "unsupported_format" }
Response 500: { error: "transcription_failed", details: string }
```

### POST `/api/generate`
```
Content-Type: application/json
Body: { transcript: string, language: CodeLanguage, context?: string, style?: string }

Response 200:
{ code: string, language: string, explanation: string, model: string, tokensUsed: number }

Response 400: { error: "transcript_required" | "invalid_language" }
Response 429: { error: "rate_limited", retryAfterMs: number }
Response 500: { error: "generation_failed", details: string }
```

### GET `/api/health`
```
Response 200: { status: "ok", providers: { whisper: boolean, llm: boolean } }
```

---

## 6. Test Plan

### Unit Tests
- `AudioRecorder`: start/stop, state machine, output blob type
- `useVoice` hook: state transitions, error handling, cleanup on unmount
- `silence detector`: threshold calculations, timing accuracy
- `promptBuilder`: correct system prompt, language injection, context inclusion
- `responseParser`: extracts code block, handles missing explanation, multi-block responses
- `llmClient`: retry on 429/500, provider switching, token counting
- `transcript sanitizer`: filler word removal, punctuation normalization

### Integration Tests
- Voice capture → blob → base64 round-trip
- POST `/api/transcribe` with mocked Whisper → transcript string
- POST `/api/generate` with mocked LLM → valid code block returned
- Full pipeline: transcript → generate → parsed response (mocked LLM)

### Edge Cases
- Empty transcript (< 3 words) → show "Too short, try again" message
- LLM returns no code block → show raw response with warning
- Network timeout on transcription → auto-retry once, then surface error
- Browser denies microphone → show permission guidance modal
- Recording exceeds 60s → auto-stop with user notification
- User edits transcript to empty string → disable generate button

### E2E (Playwright)
- Full flow: grant mic permission → record → transcribe → generate → copy code
- Keyboard-only navigation through full flow
- Offline mode: LLM fails → graceful error state

---

## 7. Open Questions

- [ ] **Streaming code gen**: Should code appear token-by-token (SSE) or on completion? → Better UX to stream, but adds complexity. Decide in Phase 4.
- [ ] **Server-side audio storage**: Do we ever persist audio blobs? → Currently no. If we add replay/history, need privacy policy.
- [ ] **Auth**: Is this single-user (API key in `.env`) or multi-user with accounts? → v0.1.0 is single-user only.
- [ ] **Voice commands vs. voice coding**: Should "hey code, stop" be a command vs. regular speech? → Use a push-to-talk model to avoid confusion in v0.1.0.
- [ ] **Whisper vs. Web Speech accuracy**: Need benchmark on developer vocabulary (function names, types). → Whisper expected to win; confirm in Phase 2 manual testing.
