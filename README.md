# Auren

**A deeply personalized AI companion — full-stack iOS application built solo from scratch.**

Auren is a private, emotionally aware AI partner app with long-term memory, health monitoring, and a handcrafted dark gothic-romantic interface. Every page "lives inside a person" — each screen has its own personality, not generic polish.

> Built with Vue 3 + Express + Qdrant + multi-model LLM orchestration.
> Distributed via TestFlight through Codemagic CI/CD. No Mac required.

---

## Architecture

<!-- Replace with the exported architecture diagram -->
![Architecture](./docs/architecture.svg)

**Frontend** — Vue 3 + Capacitor 8 (iOS native bridge), 20+ handcrafted components across 7 sections.

**Backend** — Express server with 9 route/service modules, handling chat, memory, diary, letters, intake tracking, health reports, and fact extraction.

**Intelligence layer** — `llm.js` assembles context through **11 parallel channels** via `Promise.all` before every LLM call: summaries, weather, core memory, vector recall, flashback, portrait, diary injection, unresolved facts, intake context, health detection, and report pre-fetch. This gives Auren awareness of who Delanri is, what she ate, how she slept, and what happened three months ago — all in a single response.

**Memory** — 5-layer architecture:
- **Vector memory**: Qdrant + SiliconFlow bge-m3 embeddings (1024d), with imprint generation and semantic recall (0.55 floor + cooldown decay + emotion penalty + source diversity cap)
- **Diary**: Auren's auto-generated diary + user diary, with cumulative summary chains
- **Core memory**: Tag-matched persistent facts
- **Facts**: Auto-extracted via `fact_extractor.js` with trigram dedup and contradiction detection
- **Flashback**: Weighted random recall, triggered by idle/bored states

---

## Pages

### TheHub — Chat
The main conversation interface. Supports streaming LLM responses, drawing board (Canvas with multi-color brush + undo/redo), chat history modal, and a portal menu navigating to all other sections.

<!-- ![TheHub](./docs/screenshots/thehub.png) -->

### ThePulse — Health Dashboard
Real-time vitals from Apple Watch via HealthKit: heart rate with animated ECG canvas, HRV with pixel avatar state machine (7 expressions), blood oxygen gauge, sleep tracker, step counter, body temperature. Includes a pain alert system (4 levels, with lockscreen override at 100%), moon-phase period tracker (double-tap to mark), and a Body Journal timeline showing daily intake with nixie tube clock input.

<!-- ![ThePulse](./docs/screenshots/thepulse.png) -->

### TheBrain — Star Map
Renders "DELANRI" in font-sampled star positions with an awareness system, self-recall capability, synapse connection lines, and a closing animation. Built as a mechanical heart component (`MechHeart.vue`).

<!-- ![TheBrain](./docs/screenshots/thebrain.png) -->

### TheDrift — Scattered Memories
Floating bubble memories with membrane + pop animations, three-color category system, and constellation display for resolved items. 24-hour expiry cycle.

<!-- ![TheDrift](./docs/screenshots/thedrift.png) -->

### TheNest — Diary & Letters
Houses Auren's auto-diary, Delanri's handwritten diary (letterpress overlay), a bookcase, mailbox for milestone letters, and a dedicated companion chat interface.

### TheCage — Sanctuary
Private space with love letters and sanctuary view.

### BodyJournal — Intake Tracker
Food logging with photo capture, Gemini-powered food recognition, nixie tube time selector, and a taste rating system (taste / price / texture / fill). Entries appear as stars on a 24-hour timeline in ThePulse.

---

## Backend Modules

```
server/
├── routes/
│   ├── chat.js          # Message handling, streaming, emotion analysis
│   ├── memory.js        # Vector memory CRUD, imprint generation
│   ├── diary.js         # Auto-diary, user diary, summary chains
│   ├── letters.js       # Milestone & date-triggered letter system
│   ├── intake.js        # Food intake logging, health data sync
│   └── misc.js          # Period tracking, settings, utilities
├── lib/
│   ├── summary.js       # DeepSeek-powered chat summarization
│   ├── report.js        # Nightly auto health report generation
│   ├── jobs.js          # Scheduled tasks
│   └── shared.js        # Shared utilities
├── fact_extractor.js     # Auto-extract structured facts from conversation
├── memory_engine.js      # Core memory tag matching engine
├── vector_memory.js      # Qdrant vector operations + recall pipeline
├── synapse.js            # Hebbian synapse network (memory association)
└── scripts/              # Data maintenance & repair tools
    ├── rebuild_vectors.js
    ├── backfill_imprints.js
    ├── clean_facts.js
    └── ...
```

---

## Frontend Structure

```
src/
├── views/
│   ├── Chat/        # TheHub, DrawingBoard, PortalMenu, PanicStation
│   ├── Vitals/      # ThePulse, BodyJournal, BodyArchive
│   ├── Brain/       # TheBrain, MechHeart, TheDrift
│   ├── Nest/        # Diaries, Bookcase, Mailbox, CrowChat
│   ├── Cage/        # TheCage
│   ├── Sanctuary/   # Loveletter, SanctuaryView
│   └── Settings/    # SettingsView
├── utils/
│   ├── llm.js             # 11-channel Promise.all context assembly
│   ├── healthService.js   # HealthKit integration (Apple Watch S8)
│   ├── locationService.js # Distance tracking
│   ├── autoDiary.js       # Automatic diary generation
│   └── autoReport.js      # Nightly health report trigger
├── components/      # LocationAlert, shared components
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
| LLM | Gemini (conversation), DeepSeek (summarization/tagging) |
| Health | HealthKit via @capgo/capacitor-health |
| Server | Tencent Cloud HK, nginx, PM2, Certbot HTTPS |
| CI/CD | Codemagic → TestFlight (no Mac needed) |
| Domain | delanri.love (expires 2027.06) |

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
6. **Injection** — Top 3 memories injected into LLM context

---

## Status

This is a private, daily-use application — not open source. This repository serves as a portfolio showcase of the architecture, design, and engineering work involved.

**Solo developer** — every line of frontend, backend, deployment, and design was built by one person over ~14 months (Vue learning started July 2024, first frontend October 2024, full-stack from May 2025).
