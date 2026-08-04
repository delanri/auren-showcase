**English** | [中文](./README_CN.md)


# Auren

**A deeply personalized AI companion — full-stack iOS application built solo from scratch.**

Auren is a private, emotionally aware AI partner app with long-term memory, health monitoring, and a handcrafted dark gothic-romantic interface. Every page "lives inside a person" — each screen has its own personality, not generic polish.

> Built with Vue 3 + Express + Qdrant + multi-model LLM orchestration.
> Distributed via TestFlight through Codemagic CI/CD. No Mac required.

---

## Architecture

![Architecture](./docs/architecture.svg)

**Frontend** — Vue 3 + Capacitor 8 (iOS native bridge), 25 handcrafted components across 7 sections.

**Backend** — Express server organized into route modules, a PM2 scheduled task engine, service libraries, and standalone engines — handling chat, memory, diary, letters, intake tracking, symptom records, health reports, pet health tracking, weekly wishes, and fact extraction.

**Intelligence layer** — The `llm/` module assembles context through **13 parallel channels** via `Promise.all` before every LLM call: cumulative summaries, weather, core memory, vector recall, flashback, portrait, diary injection, unresolved facts, intake context, health detection, report pre-fetch, food-taste archives, and symptom records. This gives Auren awareness of who Delanri is, what she ate, how she slept, what hurt yesterday, and what happened three months ago — all in a single response.

**Memory** — 5-layer architecture:
- **Vector memory**: Qdrant + SiliconFlow bge-m3 embeddings (1024d), with imprint generation and semantic recall (0.55 floor + cooldown decay + emotion penalty + source diversity cap)
- **Diary**: Auren's auto-generated diary + user diary, with cumulative summary chains
- **Core memory**: Tag-matched persistent facts
- **Facts**: Auto-extracted via `fact_extractor.js` with trigram dedup and contradiction detection
- **Flashback**: Weighted random recall, triggered by idle/bored states

---

## Pages

### ThePulse — Health Dashboard

Real-time vitals from Apple Watch via HealthKit: heart rate with animated ECG canvas, HRV with pixel avatar state machine (7 expressions), blood oxygen gauge, sleep tracker, step counter, body temperature. Includes a pain alert system (4 levels, with lockscreen override at 100%), moon-phase period tracker (double-tap to mark), a symptom logging mode, and a Body Journal timeline showing daily intake.

<p align="center">
  <img src="./docs/screenshots/thepulse-top.jpg" width="300" />
  <img src="./docs/screenshots/thepulse-bottom.jpg" width="300" />
</p>

### BodyJournal — Intake Tracker

Food logging with photo capture, Gemini-powered food recognition, nixie tube time selector, and a taste rating system (taste / price / texture / fill) with food source tagging. Entries appear as stars on a 24-hour timeline in ThePulse, and ratings feed a searchable taste archive that Auren can draw on in conversation.

<p align="center">
  <img src="./docs/screenshots/bodyjournal.jpg" width="300" />
</p>

### TheBrain — Star Map

Renders "DELANRI" in font-sampled star positions with an awareness system, self-recall capability, synapse connection lines, and a closing animation. Built as a mechanical heart component (`MechHeart.vue`).

<p align="center">
  <img src="./docs/screenshots/thebrain.jpg" width="300" />
</p>

### The Archives — Bookshelf

Bookshelf-style card interface with three-color classification system (red / blue / gold), row-based organization, and tap-to-expand interaction. Built to house narrative content and roleplay stories.

<p align="center">
  <img src="./docs/screenshots/bookcase.jpg" width="300" />
</p>

### Diagnostic Report — Auto Health Archive

Nightly auto-generated health reports with structured data (intake log, vitals, AI commentary per metric), a "chief physician verdict" section written by DeepSeek, classification stamps, and a randomized ink-imperfection diagnostic seal. A monthly report slot condenses the month's dailies through a two-step LLM pipeline.

<p align="center">
  <img src="./docs/screenshots/diagnostic.jpg" width="300" />
</p>

### Other Pages

- **TheHub** — Main chat interface with progressive typewriter rendering, drawing board (Canvas with multi-color brush + undo/redo), chat history modal, and portal menu
- **TheNest** — Twin diary entry (pixel-art interactive covers with chain-break unlock animation), Auren's auto-diary, user diary (letterpress overlay), bookcase, mailbox for milestone letters, pet health weekly report, and companion chat
- **TheDrift** — Floating bubble memories with membrane + pop animations, three-color category system, constellation display for resolved items
- **TheCage / Sanctuary** — Private spaces with love letters and sanctuary view

---

## System Highlights (selected)

- **Weekly Wish (周愿望)** — Monday-noon cycle where both sides write a weekly wish; injected read-once into context with a dedicated `wish` LLM mode
- **Imprint system** — sensory-level feeling descriptions generated per memory and prepended to vector recall, so retrieved memories carry texture, not just text
- **Status word** — a secondary LLM selects one of 8 state words per reply; the header animates THINKING → CRAVING → selected word
- **Preload store** — three-tier priority preloading (`stores/preload.js`) for instant page navigation, shared chat-history promise, avatar sync
- **Read-once-burn injection** — sensitive one-shot contexts (new meals, wishes, fresh reports, user diary) are injected exactly once and marked consumed
- **Dual-LLM gating** — a secondary model decides YES/NO (~90% NO) whether health context enters the primary model's prompt at all

---

## Backend Modules

```
server/
├── routes/                        # 12 route modules
│   ├── chat.js          # Message handling, emotion analysis
│   ├── diary.js         # Auto-diary, user diary, summary chains
│   ├── food.js          # Food rating & taste archive
│   ├── intake.js        # Food intake logging, health data sync
│   ├── letters.js       # Milestone & date-triggered letter system
│   ├── memory.js        # Vector memory CRUD, imprint generation
│   ├── misc.js          # Period tracking, settings, utilities
│   ├── neven.js         # Pet health data & tracking
│   ├── private.js       # Personal private records (per-date JSON, monthly aggregation)
│   ├── report.js        # Health report storage & retrieval
│   ├── symptom.js       # Symptom records, per-date queries
│   └── wishes.js        # Weekly wish cycle & read-once injection
├── lib/
│   ├── aurenPrompt.js   # Auren persona injection (AUREN_BASE / AUREN_LUST)
│   ├── autoCoreMemory.js # Core memory auto-maintenance
│   ├── autoDiary.js     # Daily auto-diary generation
│   ├── autoLetter.js    # Milestone letter auto-trigger
│   ├── autoNevenComment.js # Pet daily comment generation
│   ├── autoNevenWeekly.js  # Pet health weekly report generation
│   ├── autoPortrait.js  # Portrait auto-generation (24h cycle)
│   ├── autoReport.js    # Nightly health report generation (3-step pipeline)
│   ├── autoWish.js      # Weekly wish auto-trigger
│   ├── callLLM.js       # Backend unified LLM caller
│   ├── jobs.js          # PM2 scheduled tasks (mutex-locked job queue)
│   ├── report.js        # Nightly & monthly health report generation
│   ├── shared.js        # Shared utilities
│   ├── summary.js       # DeepSeek-powered chat summarization
│   └── taskHelpers.js   # apiFetch, callWithRetry, notifyRefresh SSE
├── scripts/                       # Maintenance & repair scripts
│   ├── backfill_imprints.js
│   ├── clean_facts.js
│   ├── clean_synapse_health.js
│   ├── fix_diary.js
│   ├── refill_core.js
│   ├── repair_chat_summaries.js
│   └── repair_letter_summaries.js
├── fact_extractor.js              # Auto-extract structured facts from conversation
├── memory_engine.js               # Core memory tag matching engine
├── vector_memory.js               # Qdrant vector operations + recall pipeline
├── synapse.js                     # Hebbian synapse network (memory association)
├── rebuild_vectors.js             # Maintenance: full vector re-embed
└── rebuild_recent_summaries.js    # Maintenance: summary chain repair
```

---

## Frontend Structure

```
src/
├── views/
│   ├── Brain/       # TheBrain, MechHeart, TheDrift
│   ├── Cage/        # TheCage
│   ├── Chat/        # TheHub, ChatFooter, ChatHistoryModal,
│   │                #   DrawingBoard, PanicStation, PortalMenu
│   ├── Nest/        # TheNest, Aurendiary, DelanriDiary, diary-center,
│   │                #   bookcase, Mailbox, CrowChat, Nevenreport
│   ├── Sanctuary/   # Loveletter, SanctuaryView
│   ├── Settings/    # SettingsView
│   └── Vitals/      # ThePulse, BodyJournal, BodyArchive
├── components/
│   └── LocationAlert.vue
├── composables/
│   └── useImprint.js      # Imprint system composable
├── stores/
│   └── preload.js         # Three-tier priority preload store
├── utils/
│   ├── llm/               # LLM context assembly (split into 5 files)
│   │   ├── index.js       # Entry point + 13-channel Promise.all dispatch
│   │   ├── channels.js    # Channel definitions & skip conditions
│   │   ├── historyBuilder.js  # Chat history construction
│   │   ├── postProcess.js     # Post-processing (status word, emotion, etc.)
│   │   └── promptBuilder.js   # Prompt assembly
│   ├── buildHiddenPrompt.js   # Hidden prompt construction
│   ├── healthService.js       # HealthKit integration (Apple Watch S8)
│   └── locationService.js     # Distance tracking
├── router/          # Vue Router with auth guards
└── assets/          # HRV pixel avatars (5 states), global CSS
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vue 3, Vite, JavaScript, CSS3 animations, Canvas API |
| Mobile | Capacitor 8 (SPM), iOS native bridge |
| Backend | Node.js, Express, REST API |
| Vector DB | Qdrant + SiliconFlow bge-m3 (1024d embeddings) |
| LLM | Multi-model via aggregated API — Gemini / DeepSeek, with fallback handling |
| Health | HealthKit via @capgo/capacitor-health |
| Server | Tencent Cloud HK, nginx, PM2, Certbot HTTPS |
| CI/CD | Codemagic → TestFlight (no Mac needed) |
| Domain | delanri.love |

---

## Design Language

- Background: `#050505`
- Delanri's color: `#A2D2FF` (soft blue)
- Auren's color: `#F9F399` (warm gold)
- English headers: Cinzel
- Chinese body: Noto Serif SC
- All animations hand-written in pure CSS3 (Keyframes + Vue Transition)
- Every page has its own visual identity — no shared component library aesthetic

---

## Recall Pipeline

When Auren recalls a memory, it passes through:

1. **Semantic search** — Qdrant vector similarity against current conversation
2. **Floor + decay** — 0.55 similarity minimum, cooldown decay for recently surfaced memories
3. **Emotion penalty** — Multiplicative penalty based on valence and event type (takes worse of the two)
4. **Weighted sort** — Combines similarity, recency, and emotional relevance
5. **Diversity cap** — Maximum 2 results per source type + trigram deduplication
6. **Injection** — Top 3 memories injected into LLM context, each prefixed with its generated imprint

---

## Status

This is a private, daily-use application — not open source. This repository serves as a portfolio showcase of the architecture, design, and engineering work involved.

**Solo developer** — every line of frontend, backend, deployment, and design was built by one person. First line of code: July 2025 (self-taught). First website shipped: November 2025. Auren was built March–July 2026 — the UI and interaction layer first, with the full backend and memory architecture written from late May onward.
