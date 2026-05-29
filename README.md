# VoiceCode 🎙️→💻

> **Voice-driven coding assistant** — dictate function names, logic, or architecture descriptions and get generated code output.

![Status](https://img.shields.io/badge/status-🚧%20Early%20Development-orange)
![Stack](https://img.shields.io/badge/stack-TypeScript%20%7C%20Node.js%20%7C%20React-blue)
![License](https://img.shields.io/badge/license-private-red)

## What it does

VoiceCode lets developers speak natural-language commands ("create a React hook that fetches user data with loading state") and instantly receive typed, runnable code. Designed for accessibility, speed, and eyes-free coding sessions.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + Vite + TypeScript |
| Backend API | Node.js + Express + TypeScript |
| Voice Capture | Web Speech API / MediaRecorder |
| Transcription | OpenAI Whisper (or Web Speech API fallback) |
| Code Generation | OpenAI GPT-4 / Anthropic Claude |
| Styling | Tailwind CSS |
| Testing | Vitest + React Testing Library + Supertest |
| Linting | ESLint + Prettier |

## Getting Started

```bash
# Clone
git clone https://github.com/joshiujjwal/voice-code.git
cd voice-code

# Install dependencies
npm install

# Set up environment
cp .env.example .env
# Add your OPENAI_API_KEY (or ANTHROPIC_API_KEY) to .env

# Start dev (frontend + backend concurrently)
npm run dev

# Run all tests
npm test

# Lint
npm run lint
```

## Project Structure

```
voice-code/
├── src/
│   ├── api/          # Express routes, middleware, controllers
│   ├── audio/        # Voice capture, transcription pipeline
│   ├── codegen/      # LLM prompt construction, response parsing
│   ├── ui/
│   │   ├── components/   # Reusable React components
│   │   ├── hooks/        # Custom React hooks (useVoice, useCodegen)
│   │   └── pages/        # Page-level components
├── tests/            # Mirrors src/ structure
├── docs/
│   ├── spec.md       # Feature specification
│   └── adr/          # Architecture Decision Records
├── .github/
│   ├── copilot-instructions.md
│   └── instructions/
├── README.md
├── TODO.md
├── CLAUDE.md
└── AGENTS.md
```

## Contributing

1. **Read `TODO.md`** — pick the next unchecked task in the current phase
2. **Write tests first** (red phase) — no implementation without a failing test
3. **Implement to pass tests** (green phase)
4. **Review your own diff** before committing — check for regressions
5. **PR requires evidence**: attach test output screenshot or CI link
6. **Keep PRs small and focused** — one feature/fix per PR
7. If you learn something non-obvious, add it to `CLAUDE.md` → `Lessons Learned`
