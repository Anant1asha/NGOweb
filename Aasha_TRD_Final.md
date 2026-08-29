# Technical Requirements Document — Aasha

**Product:** Aasha — Safe Learning. Real Impact.  
**Organisation:** Annanth Aasha Foundation  
**Document Type:** Technical Requirements Document (TRD)  
**Version:** 2.0  
**Date:** August 2026  
**Status:** Approved for Implementation  
**Architect:** Senior Software Architect  

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Frontend Stack](#2-frontend-stack)
3. [Backend Stack](#3-backend-stack)
4. [Database](#4-database)
5. [Authentication](#5-authentication)
6. [APIs](#6-apis)
7. [AI Models and Tools](#7-ai-models-and-tools)
8. [Cloud / Deployment Setup](#8-cloud--deployment-setup)
9. [Security Requirements](#9-security-requirements)
10. [Performance Requirements](#10-performance-requirements)
11. [Third-Party Integrations](#11-third-party-integrations)
12. [Technical Decisions with Reasons](#12-technical-decisions-with-reasons)
13. [Deployment Plan](#13-deployment-plan)

---

## 1. Architecture Overview

### 1.1 Design Philosophy: Offline-First, Single-File

Aasha's foundational architectural principle is **offline-first delivery**. Each chapter is a single, self-contained HTML file that renders a complete interactive learning experience with zero network dependency.

```
┌──────────────────────────────────────────────────────────────────────┐
│                    AASHA PLATFORM ARCHITECTURE                        │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐               │
│  │  Chapter 1   │   │  Chapter 2   │   │  Chapter 3   │  ...         │
│  │  Quadrilater │   │  Fractions   │   │  Comparing   │               │
│  │  als.html    │   │  .html       │   │  Quantities  │               │
│  │  (416 KB)    │   │  (387 KB)    │   │  .html       │               │
│  │              │   │              │   │  (340 KB)    │               │
│  │ ┌──────────┐│   │ ┌──────────┐│   │ ┌──────────┐│               │
│  │ │ CSS      ││   │ │ CSS      ││   │ │ CSS      ││               │
│  │ │ (inline) ││   │ │ (inline) ││   │ │ (inline) ││               │
│  │ ├──────────┤│   │ ├──────────┤│   │ ├──────────┤│               │
│  │ │ NODES[]  ││   │ │ NODES[]  ││   │ │ NODES[]  ││               │
│  │ │ WE{}     ││   │ │ WE{}     ││   │ │ WE{}     ││               │
│  │ │ SOLVE{}  ││   │ │ SOLVE{}  ││   │ │ SOLVE{}  ││               │
│  │ │ App{}    ││   │ │ App{}    ││   │ │ App{}    ││               │
│  │ │ WM{}     ││   │ │ WM{}     ││   │ │ WM{}     ││               │
│  │ └──────────┘│   │ └──────────┘│   │ └──────────┘│               │
│  └──────┬──────┘   └──────┬──────┘   └──────┬──────┘               │
│         │                  │                  │                        │
│         ▼                  ▼                  ▼                        │
│  ┌─────────────────────────────────────────────────────┐             │
│  │           localStorage (per-device state)            │             │
│  │  aasha_quad_<name>  →  progress, xp, coins, badges  │             │
│  │  aasha_economy      →  coins, hints, themes, frames  │             │
│  └─────────────────────────────────────────────────────┘             │
│                                                                      │
│  ┌─────────────────────────────────────────────────────┐             │
│  │           Future: Sync Layer (Phase 2+)              │             │
│  │  Backend API  ←→  PostgreSQL  ←→  Redis Cache       │             │
│  └─────────────────────────────────────────────────────┘             │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### 1.2 Current State (Phase 1 — Shipped)

| Metric | Value |
|---|---|
| Chapters delivered | 3 (Quadrilaterals, Fractions, Comparing Quantities) |
| Total nodes | 53 (22 + 20 + 11) |
| Total learning steps | 528 (201 + 193 + 134) |
| Average file size | 381 KB |
| External dependencies | 0 (zero CDN, zero network calls) |
| Browser target | Chrome 90+, Safari 14+, Firefox 88+ |
| Storage | localStorage + sessionStorage |
| Audio | Web Audio API oscillators (no audio files) |
| Graphics | Canvas API + CSS + inline SVG (no image assets) |
| Fonts | System fonts only |
| LLE word map | 78+ English→Hindi translations per chapter |

### 1.3 Architecture Phases

| Phase | Architecture | Status |
|---|---|---|
| Phase 1 | Offline single-file HTML, localStorage state | ✅ Shipped |
| Phase 2 | Optional sync backend, teacher dashboard, content management | Planned |
| Phase 3 | Seva verification, geotagged evidence, programme analytics | Vision |
| Phase 4 | Aasha Economy (full), saving, store, contribution loop | Vision |
| Phase 5 | AI adaptation, personalised learning paths | Vision |

---

## 2. Frontend Stack

### 2.1 Current Stack (Phase 1 — Shipped)

| Layer | Technology | Reason |
|---|---|---|
| Markup | Semantic HTML5 | Universal browser support, accessibility |
| Styling | Inline CSS with CSS custom properties (150+ variables) | Self-contained, theme-able via `body.className` |
| Interactivity | Vanilla JavaScript (ES5-compatible) | No framework, no build step, zero KB overhead, runs on any browser |
| State management | `App{}` singleton object | Simple, predictable, no library |
| Rendering | Direct `innerHTML` manipulation | Fast on low-end Android, no virtual DOM overhead |
| Visuals | Canvas 2D API + CSS3 transforms + inline SVG | Hardware-accelerated, no image assets |
| Audio | Web Audio API (oscillators + gain nodes) | No audio files, ~0 KB, generates tones programmatically |
| Storage | `localStorage` + `sessionStorage` | Persistent per-device, no server needed |
| Fonts | System font stack (`-apple-system, sans-serif`) | Zero download, native rendering |

**Why no framework?**

| Factor | Framework (React/Vue) | Vanilla JS |
|---|---|---|
| Bundle size | 40–140 KB gzipped | 0 KB |
| Build toolchain | Required (webpack/vite) | None |
| Learning curve | Team must learn framework | Team uses standard JS |
| Low-end Android perf | Virtual DOM diff overhead | Direct DOM = faster |
| Offline constraint | Framework code must be inlined | No external code at all |
| File size budget | Competes with content | Full budget for content |

The current largest chapter (Quadrilaterals) is 416 KB. Adding React (~45 KB gzipped) would consume 10% of the 500 KB budget for framework code that provides no benefit when rendering is already direct and fast.

### 2.2 Future Frontend (Phase 2+)

For the teacher dashboard, parent dashboard, and content management system — where offline is not required and interactivity is complex — a framework is justified:

| Layer | Technology | Reason |
|---|---|---|
| Framework | React 18+ (via Vite) | Component reuse, ecosystem, team familiarity |
| Styling | Tailwind CSS + CSS custom properties | Utility-first, theme consistency with chapters |
| State | Zustand or React Context | Lightweight, no Redux boilerplate |
| Charts | Recharts or Chart.js | Analytics visualisation |
| Forms | React Hook Form + Zod | Validation for teacher/parent input |
| PWA | vite-plugin-pwa | Service worker for dashboard offline cache |

The chapter files remain vanilla JS. Only the dashboards use React.

### 2.3 Rendering Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Chapter Rendering Pipeline                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  App.init()                                                 │
│    └→ loadProfile() → localStorage.getItem('aasha_quad_<n>')│
│    └→ applyTheme() → document.body.className = 'theme-X'    │
│    └→ render()                                              │
│         └→ NODES[nIdx].steps[sIdx]                          │
│              ├→ switch(step.t)                              │
│              │   case "intro"      → renderIntro()          │
│              │   case "text"       → rt(text) [LLE]          │
│              │   case "quiz"       → renderQuiz() + timer   │
│              │   case "truefalse"  → renderTrueFalse()       │
│              │   case "fillblank"   → renderFillBlank()      │
│              │   case "we"         → renderWE()              │
│              │   case "solve"      → renderSolve()           │
│              │   case "rapid_fire" → renderRapidFire()       │
│              │   case "memory_match"→ renderMemoryMatch()   │
│              │   case "ix_drag"   → renderIxDrag() [Canvas] │
│              │   case "progress"  → renderProgress() + ring │
│              └→ setTimeout(applyLLE, 50)                     │
│                                                             │
│  User Action (tap/drag/type)                                │
│    └→ App.next() / App.back() / App.selQuiz() / etc.       │
│         └→ checkAnswer() → XP + coins + sound + confetti   │
│         └→ advance() → sIdx++ or nIdx++                    │
│         └→ save() → localStorage.setItem()                 │
│         └→ render() [cycle repeats]                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.4 LLE (Language Learning Engine) Architecture

```javascript
// Input: English text with academic vocabulary
rt("A polygon is a simple closed curve made up of line segments.")

// Output: HTML with interactive word spans
'<div class="lle-text">' +
  '<span class="word" data-hi="बहुभुज">A polygon</span> ' +
  '<span class="word" data-hi="है">is</span> ' +
  '<span class="word" data-hi="एक">a</span> ' +
  'simple closed curve ' +
  '<span class="word" data-hi="बना">made</span> up of ' +
  '<span class="word" data-hi="रेखा">line</span> ' +
  '<span class="word" data-hi="खंड">segments</span>.' +
'</div>'

// On tap: word dialog shows Hindi meaning
// Connectives (if, because, therefore) show inline Hindi automatically
```

**LLE components:**

| Component | Purpose | Size |
|---|---|---|
| `WM{}` | Word Map — English→Hindi dictionary | 78+ entries per chapter |
| `CONN{}` | Connective words with inline Hindi | 12 entries |
| `rt(text, sm)` | Text processor — wraps words in interactive spans | ~2 KB |
| `applyLLE()` | Post-render hook — attaches tap listeners | ~1 KB |
| Word dialog | `<dialog>` showing Hindi meaning on tap | CSS + JS |

### 2.5 CSS Architecture

```css
/* CSS custom properties drive theming */
:root {
  --blue: #3b82f6;
  --purple: #8b5cf6;
  --green: #22c55e;
  --green-lt: #dcfce7;
  --amber: #f59e0b;
  --red: #ef4444;
  --border: #e2e8f0;
  --surface: #f8fafc;
  --text: #1e293b;
  --muted: #64748b;
}

/* Themes override custom properties */
body.theme-ocean { --blue: #0ea5e9; --purple: #06b6d4; --green: #14b8a6; }
body.theme-forest { --blue: #16a34a; --purple: #15803d; --green: #22c55e; }
body.theme-sunset { --blue: #f97316; --purple: #ec4899; --green: #f59e0b; }
```

All components use `var(--blue)` etc., so theme switching is instant via `document.body.className`.

---

## 3. Backend Stack

### 3.1 Current State: No Backend (Phase 1)

Phase 1 is intentionally backend-free. Each chapter file operates independently using `localStorage`. This was a deliberate architectural decision to:
- Eliminate server costs during proof-of-concept
- Ensure zero-downtime (no server = no downtime)
- Prove the learning experience works before investing in infrastructure
- Allow distribution via USB, WhatsApp, SD card — no internet needed

### 3.2 Future Backend (Phase 2+)

When the sync layer, teacher dashboard, and analytics are needed:

| Layer | Technology | Reason |
|---|---|---|
| Runtime | Node.js 20 LTS | Same language as frontend (JS), team familiarity |
| Framework | Fastify | 2× faster than Express, schema validation built-in, TypeScript-ready |
| API style | REST (JSON) | Simple, cacheable, works offline-first with sync |
| Real-time (future) | WebSockets via `@fastify/websocket` | For live teacher dashboard updates |
| File uploads | `@fastify/multipart` | Seva activity photo uploads |
| Rate limiting | `@fastify/rate-limit` | Protect APIs from abuse |
| CORS | `@fastify/cors` | Configure for dashboard domain only |

**Why Fastify over Express?**
- Schema-based serialization (3× faster JSON responses)
- Built-in input validation via JSON Schema
- Lower memory footprint
- Plugin ecosystem is cleaner (no middleware hell)
- TypeScript support is first-class

**Why Node.js over Python/Django?**
- The chapter JS objects (NODES, WE, SOLVE) can be shared between frontend and backend
- Content authoring tools can validate chapter structure using the same schemas
- Single language across the stack reduces team context-switching
- npm ecosystem has mature education/assessment libraries

### 3.3 Backend Services (Phase 2+)

```
┌──────────────────────────────────────────────────────┐
│                  Backend Services                     │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐   │
│  │ Auth Service│  │ Sync Service│  │ Analytics  │   │
│  │             │  │             │  │ Service    │   │
│  │ - Register  │  │ - Push state│  │            │   │
│  │ - Login     │  │ - Pull state│  │ - Aggregate│   │
│  │ - JWT issue │  │ - Merge     │  │ - Report   │   │
│  │ - Refresh   │  │ - Conflict  │  │ - Export    │   │
│  └──────┬─────┘  └──────┬─────┘  └──────┬─────┘   │
│         │                │                │          │
│  ┌──────┴─────┐  ┌──────┴─────┐  ┌──────┴─────┐   │
│  │ Content    │  │ Seva       │  │ Economy    │   │
│  │ Service    │  │ Service    │  │ Service    │   │
│  │            │  │            │  │            │   │
│  │ - Chapters │  │ - Submit   │  │ - Coins    │   │
│  │ - Validate │  │ - Verify   │  │ - Store    │   │
│  │ - Publish  │  │ - Evidence │  │ - Saving   │   │
│  └────────────┘  └────────────┘  └────────────┘   │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 4. Database

### 4.1 Current State: localStorage (Phase 1)

No database. All state is stored in `localStorage` on the device.

**localStorage schema:**

```javascript
// Per-child, per-chapter progress
"aasha_quad_<childName>": {
  nIdx: 5,              // current node index
  sIdx: 3,              // current step index
  coins: 145,           // spendable coins
  xp: 320,              // total XP
  level: 3,             // current level (1-6)
  streak: 4,            // current correct streak
  badges: { first: true, ratio: true, angles: true },
  conceptsMastered: { 0: true, 1: true, 2: true }
}

// Cross-chapter economy (shared)
"aasha_economy": {
  coins: 145,
  totalEarned: 320,
  hintTokens: 2,
  streakFreezes: 1,
  theme: "ocean",
  avatarFrame: "none"
}

// Rapid fire best scores
"aasha_rf_<childName>": {
  bestScore: 8,
  bestStreak: 6,
  timesPlayed: 3
}
```

**Limitations of localStorage:**
- 5–10 MB per origin (sufficient for progress, not for media)
- No cross-device sync
- No query capability
- No concurrent access (single tab)
- Data lost if browser cache cleared

### 4.2 Future Database (Phase 2+)

| Layer | Technology | Reason |
|---|---|---|
| Primary DB | PostgreSQL 16 | Relational, ACID, JSON columns for flexible data, excellent for education records |
| Cache | Redis 7 | Session cache, rate limit counters, leaderboard sets |
| File storage | MinIO (S3-compatible) | Seva activity photos, evidence files |
| Search | PostgreSQL full-text search | Chapter content search (no separate Elasticsearch needed at scale) |

**Why PostgreSQL over MongoDB?**

| Factor | PostgreSQL | MongoDB |
|---|---|---|
| Schema integrity | Enforced via constraints | Flexible but error-prone |
| JSON support | `jsonb` columns with indexing | Native |
| Analytics | SQL aggregations, window functions | Aggregation pipeline (complex) |
| Transaction safety | ACID | Multi-document transactions (slower) |
| Ecosystem | pgAdmin, pg_stat, mature | Compass, mature |
| Cost | Free, self-hosted or managed | Free, self-hosted or managed |

For an education platform where data integrity matters (child progress, assessment scores, coin balances), PostgreSQL's ACID compliance is essential.

**Database schema (Phase 2+):**

```sql
-- Children
CREATE TABLE children (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name        VARCHAR(100) NOT NULL,
  class_level INTEGER DEFAULT 8,
  school_id   UUID REFERENCES schools(id),
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Chapter progress (synced from localStorage)
CREATE TABLE chapter_progress (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  child_id    UUID REFERENCES children(id) ON DELETE CASCADE,
  chapter_id  VARCHAR(50) NOT NULL,  -- e.g. 'quadrilaterals'
  node_idx    INTEGER DEFAULT 0,
  step_idx    INTEGER DEFAULT 0,
  coins       INTEGER DEFAULT 0,
  xp          INTEGER DEFAULT 0,
  level       INTEGER DEFAULT 1,
  streak      INTEGER DEFAULT 0,
  badges      JSONB DEFAULT '{}',
  mastery     JSONB DEFAULT '{}',
  completed   BOOLEAN DEFAULT FALSE,
  synced_at   TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(child_id, chapter_id)
);

-- Assessment attempts
CREATE TABLE assessment_attempts (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  child_id    UUID REFERENCES children(id),
  chapter_id  VARCHAR(50),
  step_type   VARCHAR(30),    -- 'quiz', 'truefalse', 'fillblank', etc.
  question_id VARCHAR(100),
  selected    TEXT,
  is_correct  BOOLEAN,
  time_taken  INTEGER,        -- seconds
  attempted_at TIMESTAMPTZ DEFAULT NOW()
);

-- Seva activities (Phase 3)
CREATE TABLE seva_activities (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  child_id    UUID REFERENCES children(id),
  type        VARCHAR(20),    -- 'eco' or 'jal'
  description TEXT,
  photo_url   VARCHAR(500),
  geotag      POINT,
  status      VARCHAR(20) DEFAULT 'pending', -- pending/verified/rejected
  verified_by UUID REFERENCES users(id),
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Economy transactions (Phase 4)
CREATE TABLE economy_transactions (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  child_id    UUID REFERENCES children(id),
  type        VARCHAR(20),    -- 'earn', 'spend', 'save'
  amount      INTEGER,
  reason      VARCHAR(200),
  balance     INTEGER,        -- balance after transaction
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Users (teachers, parents, admins)
CREATE TABLE users (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email       VARCHAR(255) UNIQUE,
  phone       VARCHAR(20) UNIQUE,
  name        VARCHAR(100),
  role        VARCHAR(20) DEFAULT 'teacher', -- teacher/parent/admin
  password_hash VARCHAR(255),
  created_at  TIMESTAMPTZ DEFAULT NOW()
);
```

---

## 5. Authentication

### 5.1 Current State: No Auth (Phase 1)

Children create named profiles locally. No server authentication. This is intentional:
- Children may not have email addresses or phone numbers
- The product must work fully offline
- Adding auth would require a server, breaking the offline-first principle
- Profile data stays on the device via `localStorage`

### 5.2 Future Auth (Phase 2+)

| Layer | Technology | Reason |
|---|---|---|
| Child auth | None (local profiles) | Children don't need accounts; device-local identity |
| Teacher/Parent auth | JWT + refresh tokens | Stateless, scalable, works with mobile |
| Password hashing | bcrypt (cost factor 12) | Industry standard, resistant to brute force |
| OTP login | Phone OTP via SMS gateway | Teachers/parents may prefer phone over email |
| Session management | Short-lived access token (15 min) + long-lived refresh token (7 days) | Balance security with UX |
| Token storage | `httpOnly` cookie for web; secure storage for mobile | Prevent XSS token theft |

**Why JWT over session cookies?**

| Factor | JWT | Session cookies |
|---|---|---|
| Offline capability | Token can be validated offline (signature check) | Requires server round-trip |
| Horizontal scaling | Stateless, any server validates | Requires shared session store |
| Mobile-friendly | Token in header | Cookie handling on mobile is awkward |
| Revocation | Requires blacklist (Redis) | Simply delete from session store |

For Phase 2, we use JWT with a Redis-based revocation list for logout/password change.

**Child identity resolution:**
- Teacher logs in on their device
- Teacher links children to their class via a dashboard
- Children continue using local profiles on shared devices
- When sync is available, the teacher links a local profile to a server child record
- This preserves the offline-first experience for children

---

## 6. APIs

### 6.1 Current State: No APIs (Phase 1)

All logic runs in the browser. No API calls are made.

### 6.2 Future APIs (Phase 2+)

All APIs are RESTful, JSON, versioned under `/api/v1/`.

### Auth APIs

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/v1/auth/register` | Teacher/parent registration |
| POST | `/api/v1/auth/login` | Email + password login |
| POST | `/api/v1/auth/otp/send` | Send OTP to phone |
| POST | `/api/v1/auth/otp/verify` | Verify OTP, issue JWT |
| POST | `/api/v1/auth/refresh` | Refresh access token |
| POST | `/api/v1/auth/logout` | Revoke refresh token |

### Sync APIs

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/v1/sync/push` | Push localStorage state to server |
| GET | `/api/v1/sync/pull` | Pull latest state from server |
| POST | `/api/v1/sync/merge` | Resolve conflicts (server wins for economy, client wins for progress) |

### Progress APIs

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/v1/progress/:childId` | Get child's progress across all chapters |
| GET | `/api/v1/progress/:childId/:chapterId` | Get progress for specific chapter |
| POST | `/api/v1/progress/assessment` | Record an assessment attempt |

### Content APIs

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/v1/chapters` | List available chapters |
| GET | `/api/v1/chapters/:id` | Download chapter HTML file |
| POST | `/api/v1/chapters` | Upload new chapter (admin only) |
| GET | `/api/v1/chapters/:id/validate` | Validate chapter structure |

### Analytics APIs

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/v1/analytics/cohort/:schoolId` | Cohort learning analytics |
| GET | `/api/v1/analytics/child/:childId` | Individual learning analytics |
| GET | `/api/v1/analytics/misconceptions` | Common misconception patterns |
| GET | `/api/v1/analytics/export` | Export CSV report |

### Seva APIs (Phase 3)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/v1/seva/submit` | Submit activity with photo evidence |
| GET | `/api/v1/seva/pending` | List pending activities for verification |
| POST | `/api/v1/seva/:id/verify` | Verify or reject activity |

### Economy APIs (Phase 4)

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/v1/economy/:childId` | Get coin balance and transaction history |
| POST | `/api/v1/economy/spend` | Spend coins on store item |
| GET | `/api/v1/store/items` | List available store items |

### API Design Principles

- **Offline-first**: APIs accept full state payloads, not deltas. The client sends its complete localStorage snapshot.
- **Idempotent**: Push operations use `If-Match` headers to detect conflicts.
- **Rate-limited**: 100 requests/minute per token for sync; 1000/minute for read.
- **Versioned**: All endpoints under `/api/v1/` with deprecation notices.
- **Paginated**: List endpoints use cursor-based pagination (`?cursor=abc&limit=50`).

---

## 7. AI Models and Tools

### 7.1 Current State: No AI (Phase 1)

The current prototype is a **designed learning engine**, not an AI tutor. All content is pre-authored. All feedback is pre-scripted. This is intentional — Aasha should first prove the learning experience works, then add AI adaptation.

### 7.2 Future AI (Phase 2+)

| Use Case | Model/Tool | Approach | Phase |
|---|---|---|---|
| Misconception detection | Rule-based classifier → ML model | Analyse assessment attempt patterns to identify common misconceptions | Phase 2 |
| Adaptive difficulty | Collaborative filtering / bandit algorithm | Recommend next question difficulty based on performance history | Phase 2 |
| Alternative explanations | LLM (Sarvam-Mistral or Llama 3.1 8B) | Generate alternative explanations in simpler language when a child struggles | Phase 3 |
| Content generation | LLM + RAG | Assist content authors in generating chapter content (not real-time for children) | Phase 3 |
| Language translation | Sarvam AI Indic models | Expand LLE word maps to more regional languages (Tamil, Telugu, Bengali) | Phase 3 |
| Seva image verification | Vision model (CLIP / MobileNet) | Classify submitted activity photos (plant, water, cleanliness) | Phase 3 |

**Why not start with AI?**

The PRD explicitly states: "Aasha should first prove the learning experience works. Then AI should make that experience increasingly adaptive."

Starting with AI would:
- Add 200+ MB of model weight to each deployment
- Require constant internet for API calls
- Make it impossible to debug learning outcomes (was it the AI or the content?)
- Introduce unpredictable behaviour (LLM hallucination in educational content is dangerous)

### 7.3 AI Tooling (Phase 2+)

| Tool | Purpose | License |
|---|---|---|
| Ollama | Local LLM runtime for content authoring assistance | Open source (MIT) |
| Sarvam AI | Indic language models for Hindi/regional translation | Open weights |
| Sentence Transformers | Embedding generation for misconception clustering | Apache 2.0 |
| pgvector | Vector similarity search in PostgreSQL | PostgreSQL license |

**AI architecture (Phase 3):**

```
┌──────────────────────────────────────────────────────┐
│                   AI Layer (Phase 3)                   │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Child struggles with concept                        │
│    └→ Misconception classifier identifies pattern     │
│    └→ Difficulty adjuster selects easier question    │
│    └→ LLM generates alternative explanation           │
│         (pre-generated, cached, reviewed by author)  │
│    └→ LLE translates explanation to Hindi             │
│    └→ Child receives personalised support             │
│                                                      │
│  All AI-generated content is:                        │
│  - Pre-generated (not real-time for children)         │
│  - Reviewed by content authors                        │
│  - Cached in the chapter file or CDN                  │
│  - Never generated live in the child's browser        │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 8. Cloud / Deployment Setup

### 8.1 Current State: File Distribution (Phase 1)

No cloud infrastructure. Chapter HTML files are distributed via:
- WhatsApp / Telegram (send file to teacher, teacher sends to children)
- USB drive (offline schools)
- School WiFi hotspot (teacher's phone as hotspot, children download from local server)
- Google Drive / Dropbox link (one-time download, then offline forever)

### 8.2 Future Cloud (Phase 2+)

| Component | Technology | Reason |
|---|---|---|
| Cloud provider | AWS or Hetzner | AWS for managed services; Hetzner for cost (₹500/month VPS) |
| Container runtime | Docker + Docker Compose | Simple, reproducible, works on any VPS |
| Reverse proxy | Caddy | Automatic HTTPS, simple config, 10× smaller than nginx |
| App hosting | Node.js on Docker | Fastify server in a container |
| Database hosting | Managed PostgreSQL (RDS) or self-hosted on VPS | RDS for reliability; self-hosted for cost |
| Cache | Redis on Docker | Session cache, rate limiting |
| File storage | MinIO (self-hosted S3) or AWS S3 | Seva photos, chapter files |
| CDN | Cloudflare (free tier) | Cache chapter files at edge, DDoS protection |
| Monitoring | Uptime Kuma (self-hosted, open source) | Uptime alerts, no per-seat cost |
| Logs | Loki + Grafana (self-hosted) | Centralised logging, open source |
| CI/CD | GitHub Actions | Free for open source, automated testing |

**Why Hetzner over AWS for Phase 2?**

| Factor | Hetzner CX22 | AWS t3.small |
|---|---|---|
| Cost (monthly) | ₹500 (~$6) | ₹1,500 (~$18) |
| RAM | 4 GB | 2 GB |
| CPU | 2 vCPU (dedicated) | 2 vCPU (burstable) |
| Storage | 40 GB NVMe | 8 GB EBS |
| Setup complexity | Low (Docker on VPS) | Medium (VPC, security groups, etc.) |

For an NGO with budget constraints, Hetzner provides 3× the resources at 1/3 the cost. AWS can be considered when scale requires managed services.

### 8.3 Deployment Architecture (Phase 2+)

```
┌──────────────────────────────────────────────────────────────┐
│                    Production Architecture                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Internet                                                    │
│    │                                                         │
│    ▼                                                         │
│  ┌──────────────┐                                          │
│  │  Cloudflare   │  ← CDN (free tier)                      │
│  │  (DNS + CDN)  │  ← Caches chapter HTML files at edge    │
│  └──────┬───────┘                                          │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                          │
│  │    Caddy      │  ← Reverse proxy (auto-HTTPS)           │
│  │  (Port 443)   │  ← Routes /api/* to Fastify             │
│  └──────┬───────┘    Routes / to MinIO (chapter files)      │
│         │                                                    │
│    ┌────┴────┬────────┬────────┐                            │
│    ▼         ▼        ▼        ▼                            │
│  ┌─────┐ ┌──────┐ ┌──────┐ ┌──────┐                      │
│  │Fastify│ │Postgres│ │ Redis │ │MinIO │                   │
│  │(API)  │ │  (DB)  │ │(Cache)│ │(Files)│                  │
│  └───┬──┘ └───┬──┘ └───┬──┘ └───┬──┘                      │
│      │        │        │        │                           │
│      └────────┴────────┴────────┘                           │
│           Docker Compose network                             │
│                                                              │
│  ┌──────────────────────────────┐                           │
│  │  Monitoring (Uptime Kuma)     │                           │
│  │  Logs (Loki + Grafana)        │                           │
│  └──────────────────────────────┘                           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 8.4 Docker Compose (Phase 2+)

```yaml
# docker-compose.yml
version: '3.8'

services:
  caddy:
    image: caddy:2
    ports: ["80:80", "443:443"]
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
    depends_on: [api, minio]

  api:
    build: ./server
    environment:
      - DATABASE_URL=postgres://aasha:${DB_PASS}@db:5432/aasha
      - REDIS_URL=redis://cache:6379
      - JWT_SECRET=${JWT_SECRET}
    depends_on: [db, cache]

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_DB=aasha
      - POSTGRES_USER=aasha
      - POSTGRES_PASSWORD=${DB_PASS}
    volumes:
      - pg_data:/var/lib/postgresql/data

  cache:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      - MINIO_ROOT_USER=${MINIO_USER}
      - MINIO_ROOT_PASSWORD=${MINIO_PASS}
    volumes:
      - minio_data:/data

volumes:
  caddy_data:
  pg_data:
  redis_data:
  minio_data:
```

### 8.5 Caddyfile (Phase 2+)

```caddyfile
# Caddyfile — automatic HTTPS
aasha.example.org {
    # API
    handle /api/* {
        reverse_proxy api:3000
    }

    # Chapter files (cached at CDN edge)
    handle /chapters/* {
        reverse_proxy minio:9000
        header Cache-Control "public, max-age=31536000, immutable"
    }

    # Dashboard
    handle {
        reverse_proxy dashboard:5173
    }
}
```

---

## 9. Security Requirements

### 9.1 Current Security (Phase 1)

| Requirement | Implementation |
|---|---|
| No external network calls | Zero CDN, zero fetch, zero XHR — verified by grep |
| No user data leaves device | All state in localStorage, never transmitted |
| No cookies | No tracking, no analytics beacons |
| Content Security Policy | `default-src 'self' 'unsafe-inline'` (inline styles/scripts only) |
| No eval() | All JS is static (no dynamic code execution) |
| Child safety | No chat, no social features, no external links |
| File integrity | Each file is self-contained; tampering is visible (content won't render) |

### 9.2 Future Security (Phase 2+)

| Requirement | Implementation |
|---|---|
| HTTPS everywhere | Caddy auto-TLS (Let's Encrypt) |
| Input sanitisation | Fastify JSON Schema validation on all endpoints |
| SQL injection prevention | Parameterised queries only (pg-promise or Prisma) |
| XSS prevention | DOMPurify for any user-generated content; CSP headers |
| CSRF prevention | SameSite=Strict cookies for auth tokens |
| Rate limiting | 100 req/min per IP (Fastify rate-limit plugin) |
| Password storage | bcrypt (cost factor 12) |
| JWT security | RS256 signing, 15-min access, 7-day refresh, Redis revocation list |
| File upload security | Magic byte validation, max 5 MB, virus scan (ClamAV) |
| PII protection | Child names encrypted at rest (pgcrypto); no PII in logs |
| GDPR/DPDP compliance | Data export and deletion endpoints; consent tracking |
| Audit logging | All teacher/admin actions logged with timestamp and IP |

### 9.3 Child Safety Requirements (All Phases)

| Requirement | Implementation |
|---|---|
| No social features | No chat, messaging, comments, followers, or public profiles |
| No external links | Chapter files contain zero outbound URLs |
| No advertising | No ad SDKs, no tracking pixels |
| No in-app purchases | Gem shop uses virtual coins only; no real money |
| No data collection from children | Phase 1 collects nothing; Phase 2 syncs only learning data (not PII) with teacher consent |
| COPPA / DPDP compliance | No personal data collected from children under 14 without guardian consent |
| Content review | All educational content reviewed by educators before publishing |

### 9.4 Content Security Policy

```http
# Phase 1 (offline HTML files)
Content-Security-Policy: default-src 'self' 'unsafe-inline'; 
  img-src 'self' data:; 
  media-src 'none'; 
  connect-src 'none';

# Phase 2+ (with backend)
Content-Security-Policy: default-src 'self'; 
  script-src 'self'; 
  style-src 'self' 'unsafe-inline'; 
  img-src 'self' data: blob:; 
  connect-src 'self' https://api.aasha.org; 
  frame-ancestors 'none';
  base-uri 'self';
```

---

## 10. Performance Requirements

### 10.1 Chapter File Performance (Phase 1 — Current)

| Metric | Target | Measured |
|---|---|---|
| File load time (local file) | < 500 ms | ~200 ms (416 KB) |
| First contentful paint | < 1 second | ~500 ms |
| Step render time | < 16 ms (60fps) | < 5 ms (innerHTML) |
| Canvas render time | < 16 ms | < 3 ms |
| localStorage read/write | < 5 ms | < 2 ms |
| Audio latency | < 50 ms | < 20 ms (Web Audio API) |
| Memory usage | < 50 MB | ~20 MB (no frameworks) |
| File size | < 500 KB | 416 KB (largest chapter) |

### 10.2 Mobile Performance Constraints

| Constraint | Mitigation |
|---|---|
| Low-end Android (1 GB RAM) | No frameworks, no virtual DOM, direct DOM manipulation |
| Slow CPU | Canvas rendering is GPU-accelerated; JS is ES5-compatible |
| Shared device | Profile picker supports quick switching; data in localStorage persists |
| No GPU | CSS transforms degrade to layout (still functional) |
| 2G/3G network (initial download only) | Files are < 500 KB — downloads in < 5 seconds on 2G |
| Battery | No background processes, no polling, no service workers in Phase 1 |

### 10.3 Backend Performance (Phase 2+)

| Metric | Target |
|---|---|
| API response time (p95) | < 200 ms |
| Sync operation (push + pull) | < 2 seconds |
| Chapter file download | < 1 second (CDN cached) |
| Database query (indexed) | < 10 ms |
| Concurrent users per VPS | 500 (Fastify on 2 vCPU / 4 GB) |
| Uptime | 99.5% (managed DB) / 99.0% (self-hosted) |

### 10.4 Performance Testing

| Test | Tool | Frequency |
|---|---|---|
| Chapter file render | Puppeteer (headless Chrome) | Every build (CI) |
| All steps render without crash | Mock DOM test harness | Every build |
| All answers accepted correctly | Data integrity verification | Every build |
| File size budget | `wc -c` check | Every build |
| Backend API load | k6 (open source) | Pre-release |
| Database query performance | `EXPLAIN ANALYZE` | Pre-release |

---

## 11. Third-Party Integrations

### 11.1 Principle: Open-Source / Prebuilt First

Aasha prioritises open-source and self-hostable tools over proprietary SaaS. This reduces recurring costs (critical for an NGO), avoids vendor lock-in, and keeps data within the organisation's control.

### 11.2 Current Integrations (Phase 1)

| Integration | Purpose | Type | Cost |
|---|---|---|---|
| None | Fully self-contained | — | ₹0 |

### 11.3 Future Integrations (Phase 2+)

| Integration | Purpose | Type | License | Cost |
|---|---|---|---|---|
| **Fastify** | API framework | Open source (MIT) | Free |
| **PostgreSQL** | Primary database | Open source (PostgreSQL license) | Free |
| **Redis** | Cache + rate limiting | Open source (BSD) | Free |
| **MinIO** | S3-compatible file storage | Open source (AGPLv3) | Free |
| **Caddy** | Reverse proxy + auto-HTTPS | Open source (Apache 2.0) | Free |
| **Cloudflare** | CDN + DNS + DDoS protection | Freemium | Free tier |
| **Hetzner** | VPS hosting | Commercial | ₹500/month |
| **GitHub Actions** | CI/CD pipeline | Freemium | Free for open source |
| **Uptime Kuma** | Uptime monitoring | Open source (MIT) | Free (self-hosted) |
| **Loki + Grafana** | Logging + dashboards | Open source (AGPLv3) | Free (self-hosted) |
| **DOMPurify** | XSS sanitisation | Open source (Apache 2.0) | Free |
| **bcrypt** | Password hashing | Open source | Free |
| **ClamAV** | Virus scan for file uploads | Open source (GPL) | Free |
| **pgvector** | Vector search for AI | Open source (PostgreSQL license) | Free |
| **Ollama** | Local LLM runtime | Open source (MIT) | Free |
| **Sarvam AI** | Indic language models | Open weights | Free (API tier available) |
| **Twilio / MSG91** | SMS OTP for teacher auth | Commercial | ~₹0.50/SMS |
| **Puppeteer** | Automated browser testing | Open source (Apache 2.0) | Free |

### 11.4 What We Deliberately Avoid

| Tool | Why Avoid |
|---|---|
| Firebase | Vendor lock-in, cost scales poorly, not self-hostable |
| AWS Cognito | Complex pricing, overkill for simple teacher auth |
| Stripe/Razorpay | No real-money transactions in the product |
| Google Analytics | Tracks children; violates child privacy principles |
| Mixpanel/Amplitude | Same privacy concerns; self-hosted alternatives exist |
| MongoDB Atlas | Managed MongoDB is expensive; PostgreSQL is free and superior for this use case |
| Vercel/Netlify | Great for React dashboards but expensive at scale; Caddy + VPS is cheaper |
| Intercom/Zendesk | Not needed; teacher support is via WhatsApp group |

---

## 12. Technical Decisions with Reasons

### Decision 1: Vanilla JS over React/Vue for chapter files

**Decision:** Chapter HTML files use vanilla JavaScript with direct DOM manipulation.

**Reasoning:**
- Chapters must work offline with zero dependencies
- Adding React (45 KB gzipped) consumes 10% of the 500 KB file budget
- Direct `innerHTML` rendering is faster than virtual DOM diffing on low-end Android
- No build toolchain needed — content authors can edit HTML directly
- The rendering pattern is simple: one `render()` function, one `next()` handler

**Trade-off:** More verbose code for complex interactions (e.g., memory match game). Accepted because the code is reviewed and tested before publishing.

### Decision 2: localStorage over IndexedDB for Phase 1

**Decision:** Use `localStorage` for all state management.

**Reasoning:**
- Data volume is small (< 5 KB per child per chapter)
- Synchronous API is simpler to reason about
- No need for queries or indexes at this scale
- IndexedDB's async API would complicate the render cycle

**Trade-off:** 5 MB storage limit, no cross-device sync. Accepted for Phase 1; Phase 2 adds backend sync.

### Decision 3: Canvas API over SVG for interactive visuals

**Decision:** Use Canvas 2D API for geometric drawings (drag-to-manipulate vertices, diagonal drawing).

**Reasoning:**
- Canvas provides pixel-level control for drag interactions
- Performance is better for redraws (clear + redraw vs. DOM manipulation)
- Touch events are simpler on a single canvas element
- SVG would require creating/removing DOM elements on each drag frame

**Trade-off:** Canvas is not accessible (screen readers can't read drawn shapes). Mitigated by always including a text caption for every visual.

### Decision 4: Web Audio API over audio files

**Decision:** All sounds are generated programmatically using oscillator nodes.

**Reasoning:**
- Zero bytes of audio assets (MP3 files would add 50–200 KB)
- Tones are customisable (frequency, duration, waveform)
- No loading delay — AudioContext is ready immediately
- Satisfies the offline-first constraint perfectly

**Trade-off:** Sound quality is basic (sine/sawtooth waves). Accepted because the sounds are short feedback cues, not music.

### Decision 5: Fastify over Express for backend

**Decision:** Use Fastify for the Phase 2+ backend API.

**Reasoning:**
- 2× faster request throughput than Express
- Built-in JSON Schema validation (critical for input security)
- Plugin system is cleaner than Express middleware
- TypeScript support is first-class
- Lower memory footprint

### Decision 6: PostgreSQL over MongoDB

**Decision:** Use PostgreSQL as the primary database.

**Reasoning:**
- ACID compliance is essential for education records and coin balances
- `jsonb` columns provide JSON flexibility when needed
- SQL aggregations are more powerful for analytics
- pgvector extension supports future AI/vector search
- Free and battle-tested

### Decision 7: Caddy over Nginx for reverse proxy

**Decision:** Use Caddy instead of Nginx.

**Reasoning:**
- Automatic HTTPS (Let's Encrypt) with zero configuration
- Config file is 10 lines vs. 50+ for Nginx
- Single binary, 30 MB, no dependencies
- HTTP/3 support built-in
- Adequate performance for the expected load (500 concurrent users)

### Decision 8: No AI in Phase 1

**Decision:** The current prototype has no AI components.

**Reasoning:**
- The PRD explicitly states: prove the learning works first, then add AI
- AI introduces unpredictability (hallucination, latency, cost)
- Pre-authored content is debuggable and reviewable
- AI models add 200+ MB — impossible for offline delivery

**Phase 2+ approach:** AI assists content authors (not children directly). All AI-generated content is reviewed, cached, and distributed as static files.

### Decision 9: Docker Compose over Kubernetes

**Decision:** Use Docker Compose for deployment, not Kubernetes.

**Reasoning:**
- Expected scale: 1–10 schools, 100–500 children initially
- Docker Compose handles 5 containers on a single ₹500/month VPS
- Kubernetes adds operational complexity (etcd, RBAC, ingress controllers)
- K8s is justified at 50+ nodes or multi-region deployment

### Decision 10: Phone OTP over email for teacher auth

**Decision:** Teachers authenticate via phone OTP, not email/password.

**Reasoning:**
- Teachers in government schools may not check email regularly
- Phone penetration in India is near-universal
- MSG91 (Indian SMS gateway) costs ₹0.50/SMS — affordable
- OTP removes password management burden
- Email/password remains as a fallback option

---

## 13. Deployment Plan

### 13.1 Phase 1 — Current Deployment

```
┌──────────────────────────────────────────────────────┐
│              Phase 1: File Distribution                │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Content Author                                      │
│    └→ Builds chapter HTML file (Node.js script)      │
│    └→ Tests with mock DOM harness                    │
│    └→ Verifies all steps render, all answers correct │
│    └→ Delivers .html file                            │
│                                                      │
│  Distribution Channels:                              │
│    ├→ WhatsApp group (teacher → children)            │
│    ├→ USB drive (for schools without internet)        │
│    ├→ Google Drive link (one-time download)          │
│    └→ School WiFi hotspot (local file server)         │
│                                                      │
│  Child Device:                                       │
│    └→ Downloads .html file once                      │
│    └→ Opens in Chrome/Firefox browser                │
│    └→ Works forever offline                          │
│    └→ Progress saved in localStorage                  │
│                                                      │
│  No server. No cloud. No ongoing costs.              │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### 13.2 Phase 2 — Backend Deployment

```
┌──────────────────────────────────────────────────────┐
│              Phase 2: Cloud Backend                   │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Step 1: Provision VPS (Hetzner CX22, ₹500/month)   │
│    └→ Install Docker + Docker Compose                │
│    └→ Clone repository                               │
│    └→ Configure .env (secrets, DB passwords)         │
│    └→ docker compose up -d                            │
│                                                      │
│  Step 2: Configure DNS + CDN                         │
│    └→ Point aasha.example.org to VPS IP              │
│    └→ Enable Cloudflare CDN (free tier)              │
│    └→ Caddy auto-provisions TLS certificate          │
│                                                      │
│  Step 3: Deploy Backend Services                     │
│    └→ Fastify API (Port 3000)                        │
│    └→ PostgreSQL (Port 5432, internal only)          │
│    └→ Redis (Port 6379, internal only)               │
│    └→ MinIO (Port 9000, for chapter file hosting)    │
│    └→ Caddy (Port 443, reverse proxy)                │
│                                                      │
│  Step 4: Deploy Dashboards                           │
│    └→ React teacher dashboard (built with Vite)      │
│    └→ Served via Caddy as static files               │
│                                                      │
│  Step 5: Set Up Monitoring                           │
│    └→ Uptime Kuma (alerts via Telegram)              │
│    └→ Loki + Grafana (log aggregation)               │
│    └→ GitHub Actions (CI/CD: test → build → deploy)  │
│                                                      │
│  Step 6: Content Management                          │
│    └→ Upload chapter HTML to MinIO                    │
│    └→ CDN caches chapter at edge                     │
│    └→ Teachers assign chapters via dashboard         │
│    └→ Children download once, use offline             │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### 13.3 CI/CD Pipeline (Phase 2+)

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm test           # Unit + integration tests
      - run: npm run test:e2e   # Puppeteer chapter tests

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - run: npm run build       # Build dashboards
      - name: Deploy to VPS
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /opt/aasha
            git pull origin main
            docker compose build
            docker compose up -d
            docker image prune -f
```

### 13.4 Testing Strategy

| Test Type | Tool | What It Checks | When |
|---|---|---|---|
| JS syntax validation | `node --check` | Chapter JS parses without errors | Every build |
| Data integrity | Custom mock DOM harness | All NODES, WE, SOLVE structure valid; every quiz has 1 correct answer; every fill-blank has `___` in sentence | Every build |
| Render test | Mock DOM harness | Every step renders without crash (all 201+ steps) | Every build |
| Answer acceptance | Verification script | Correct answers are accepted; wrong answers are rejected; float matching works | Every build |
| End-to-end | Puppeteer (headless Chrome) | Full chapter walkthrough — click through every step, no stuck states | Pre-release |
| Load test | k6 | Backend API handles 500 concurrent users | Pre-release |
| Security scan | npm audit + Trivy | No known vulnerabilities in dependencies | Every build |

### 13.5 Environment Matrix

| Environment | Purpose | Hosting | Cost |
|---|---|---|---|
| Local development | Content authoring, chapter building | Developer laptop | ₹0 |
| CI/CD | Automated testing | GitHub Actions (free tier) | ₹0 |
| Staging | Pre-release validation | Hetzner VPS (shared) | ₹500/month |
| Production | Live platform | Hetzner VPS (dedicated) | ₹500/month |
| CDN | Chapter file distribution | Cloudflare (free tier) | ₹0 |
| File storage | Seva photos, chapter files | MinIO on VPS | ₹0 |
| SMS OTP | Teacher authentication | MSG91 | ~₹500/month (1000 SMS) |
| Monitoring | Uptime alerts | Uptime Kuma on VPS | ₹0 |

**Total monthly cost (Phase 2): ~₹1,000–1,500/month**

---

## Appendix A: File Size Budget

| Component | Size (KB) | % of Budget |
|---|---|---|
| CSS (inline) | 19 | 3.8% |
| HTML shell (header, dialogs, brand) | 250 | 50% |
| JavaScript (NODES, WE, SOLVE, App) | 145 | 29% |
| Base64 logo image | ~2 | 0.4% |
| **Total (Quadrilaterals chapter)** | **416** | **83.2%** |
| **Budget** | **500** | **100%** |
| **Remaining** | **84** | **16.8%** |

Remaining budget is sufficient for 1–2 additional nodes per chapter or minor feature additions.

---

## Appendix B: localStorage Schema (Phase 1)

```javascript
// All keys use the prefix 'aasha_' to avoid collisions

// Per-child, per-chapter progress
"aasha_quad_<childName>": {
  nIdx: 5,                    // Current node index (0-based)
  sIdx: 3,                    // Current step index within node
  coins: 145,                 // Spendable coins
  xp: 320,                    // Total XP earned
  level: 3,                   // Current level (1-6)
  streak: 4,                  // Current consecutive correct streak
  badges: {                   // Earned badges
    first: true,
    ratio: true,
    angles: true
  },
  conceptsMastered: {         // Which nodes are mastered
    0: true, 1: true, 2: true
  }
}

// Cross-chapter economy (shared across all chapters)
"aasha_economy": {
  coins: 145,                 // Total coins (synced from chapter coins)
  totalEarned: 320,           // Lifetime coins earned
  hintTokens: 2,              // Consumable: eliminates 2 wrong options
  streakFreezes: 1,           // Consumable: protects 1 wrong answer
  theme: "ocean",              // "default" | "ocean" | "forest" | "sunset"
  avatarFrame: "none"         // "none" | "gold" | "star"
}

// Rapid fire best scores
"aasha_rf_<childName>": {
  bestScore: 8,               // Best correct count out of 10
  bestStreak: 6,              // Best streak within a round
  timesPlayed: 3              // Total rounds played
}

// Daily challenge (Phase 2)
"aasha_daily_<childName>": {
  date: "2026-08-24",         // Last played date
  score: 4,                   // Correct out of 5
  bestScore: 5,               // Best ever
  streak: 3,                  // Consecutive days played
  lastPlayed: "2026-08-24"
}
```

---

## Appendix C: API Response Format (Phase 2+)

```json
// Success
{
  "status": "success",
  "data": { ... },
  "meta": {
    "page": 1,
    "limit": 50,
    "total": 120
  }
}

// Error
{
  "status": "error",
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "child_id is required",
    "details": [
      { "field": "child_id", "message": "must be a valid UUID" }
    ]
  }
}

// Sync response
{
  "status": "success",
  "data": {
    "merged": true,
    "conflicts": [],
    "serverState": { ... }
  }
}
```
