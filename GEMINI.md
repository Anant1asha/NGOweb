# Aasha Development Rules & Guidelines

To prevent hallucinations, respect the project ecosystem, and ensure all code and design modifications meet or exceed prototype standards, the agent MUST follow these constraints:

## 1. Core Ecosystem Constraints
- **Monorepo Architecture**:
  - `server/`: Fastify API (Node.js) for auth, syncing, and AI transformation.
  - `dashboard/`: React + Vite + Tailwind CSS dashboard for teachers and admins.
  - `chapters/`: Workspace for offline HTML chapters.
  - `packages/`:
    - `@aasha/shared-types`: Type definitions shared across projects.
    - `@aasha/chapter-builder`: CLI & library to compile HTML chapters.
    - `@aasha/test-harness`: Reusable validation suite to verify offline chapters.
  - `infra/`: Postgres, Redis, MinIO (Docker Compose).
- **Target Audience & Board Scope**:
  - Target audience is Class 1 to Class 10 (ages 6–15), NOT Class 5–8 only.
  - All Boards are supported (CBSE, ICSE, State, IB, Cambridge, NIOS). Do NOT assume NCERT is the only board.
- **Content Creation Model**:
  - Driven by the AI transformation pipeline (PDF/Image/Text -> Structure analysis -> Content generation -> Assessment generation -> Assembly).
  - Teachers upload scanned PDFs (up to 50 MB) or paste text. The system automatically transforms it.
  - Do NOT write or assume manual JSON content authoring as the primary flow. AI transformation is Phase 1 core.

## 2. Language Learning Engine (LLE) Guidelines
- **Enhanced LLE**: Use `Aasha_LLE_Enhanced.js` drop-in replacement.
- **Bilingual & Pronunciation**: Use browser-native Web Speech API (`SpeechSynthesis`) for English and Hindi pronunciation. Adjust rate to `0.85` for children. Fall back to tone-based rhythms via Web Audio API. Do NOT use audio files, external APIs, or network requests for pronunciation.
- **Visual Illustrations**: Use inline SVG path data for 50+ key math/science vocabulary words (e.g., polygon, circle, angle) rendered inside a `<dialog>` card. Do NOT use PNG/JPG images or CDNs. Keep total LLE size around 60 KB.
- **Transliteration**: Include Romanized Hindi (transliteration) in word dialogs, alongside part-of-speech, definition, and example sentences.
- **HTML Offline Isolation**: Single-file HTML chapters must be completely self-contained (no external CDNs, fonts, or media files that need network). Target file size: <= 20 MB.
- **Language Layer Limitations**: 
  - Highlight/reveal *only* modal verbs, connectors, and adjective/expression words (e.g., "has", "have", "must", "can", "could", etc.) as default Hindi words in the language layer.
  - Limit the number of visible/highlighted words to a maximum of 2 to 4 depending on the class level (fewer for lower classes).
  - Do NOT target subject/topic terms (e.g., "square", "fraction", "cube", etc.) for the basic bilingual connector layer.

## 3. Implementation Quality Rules
- **No Hallucinations**:
  - Do NOT ignore the gap analysis docs (`Aasha_Gap_Analysis_and_Corrections.md`, `Aasha_LLE_Gap_Analysis_and_Corrections.md`).
  - Do NOT assume standard NCERT boundaries when generating content or questions.
  - Ensure all assessment questions (100%) are generated dynamically from the specific chapter content.
- **Code Integrity**:
  - Never break backward compatibility with the original LLE API (`rt()`, `applyLLE()`, `showWord()`, `WM`, `CONN`).
  - Keep styling consistent with the design tokens in the UI/UX Design Brief.
  - **AST-Compliant Step Nodes**: Ensure all interactive flows and step nodes are AST-compliant.
  - **Local Profile Progression**: Maintain student progression, gamification points, and completed states offline via `localStorage`.

## 4. Vision, Mission & Beta Priority
- **The Aasha Vision**: To make every child's own schoolbook understandable, interactive, and meaningful — helping learning become action, responsibility, and hope beyond the page. The book stays; the learning experience changes.
- **Tone & Pedagogical Style**: The language used throughout chapters must offer a human connection, resembling a PW (Physics Wallah) mentor teaching. It should be highly relatable, engaging, encouraging, and connected, NOT dry, formal, or system-generated.
- **Beta Phase Priority**: The bilingual, offline-first learning engine (Layer 1 — Understanding) and mastery engine (Layer 2 — Practice, LLE, dynamic assessments, local gamification) are the absolute top priority.
- **Ecosystem & Scope Isolation**: While the project includes a server, dashboards, and infra, do not let backend complexity distract from the client-side learning experience in the Beta phase. Keep backend integrations simple and mock when appropriate. All other extensive ecosystem modifications are finalized in other specific chats of this project.

## 5. Architecture Decisions — Phase 1 LOCKED (Do Not Re-Litigate)

These decisions were made during explicit product-owner review on 28 August 2026.
Do NOT override, suggest alternatives, or reopen these without a new explicit HIL gate:

### Tech Stack (Locked)
- **Frontend framework:** Next.js (App Router) on Vercel — SSR, SEO, dynamic routes, ISR. Do NOT suggest React SPA, Remix, or Astro.
- **Database:** Supabase PostgreSQL — the ONLY canonical system of record. No second database.
- **Auth:** Supabase Auth — email + magic link OTP, native RLS integration. Do NOT use custom JWT RS256.
- **Backend split:** Next.js API Routes for simple CRUD/auth; Railway Fastify workers ONLY for heavy jobs (AI pipeline, OCR, long-running processing). Do NOT move all backend to Railway unnecessarily.
- **Game architecture:** One reusable game shell × many chapter datasets. NEVER build a bespoke game per chapter. `game_template_id` is nullable — standard renderer preferred when pedagogically equivalent.
- **Content pipeline:** Teachers upload PDF/text → AI transforms → automated validation → HIL reviews the HTML output file → approved → catalogue. AI NEVER self-approves or self-publishes.
- **HIL validation:** Product owner validates by opening and inspecting the generated HTML output file in a browser. Every learning artifact requires this gate before publication.
- **Build sequence:** All 17 phases, sequential — complete and review each phase before starting the next.
- **Domain:** `aannathaasha.org` on Cloudflare DNS → Vercel frontend, Railway API subdomain.

### Document Folder Roles (Do Not Confuse These)
- `redraft tech prdttrd etc/` → **Technical builder reference** (PRD, TRD, App Flow, UI/UX, Backend Schema, Implementation Plan v3.1). Use for all build decisions.
- `Annanth_Aasha_Phase_0_Frozen_Phase_1/AA_Phase0_Pack/` → **Brand book / PM reference** (Constitution + AIOS Context Pack). Use to understand product vision, positioning, voice, and ecosystem story as a product manager.
- `html5 ngo/chapters/` HTML files → **Field testing prototypes / reference implementations only**. NOT the live production catalogue. Production catalogue lives in Supabase.

### Backend Role (Corrected)
The backend (Supabase + Railway + Next.js API Routes) **extends** the child's journey — progress sync, catalogue delivery, evidence recording, economy. It does NOT gate core learning. A child must be able to learn from a downloaded HTML chapter even if every cloud service is simultaneously unavailable.

### What MUST NOT Change Without Explicit HIL Approval
- Three-layer product architecture (Learning Runtime / Platform / Operations)
- Supabase as the only canonical application database
- Offline-first principle for HTML chapters
- No child login required for core learning
- Evidence lifecycle: Draft → Submitted → Under Review → Verified → Published
- Aasha Coins — NOT cash, wages, or cryptocurrency
- HIL gate requirement before any learning content reaches production

## 6. Build Execution Rules

- Build ALL 17 phases, one after another, in the approved Implementation Plan order
- Each phase requires explicit product-owner review before the next begins
- No phase skips testing — every phase ships with automated tests
- Never hard-code catalogue content into frontend components
- Dynamic catalogue: adding a new approved module = Supabase data operation, NOT a code/deployment change
- No agent-to-production path — no AI artifact goes live without HIL HTML-output-file review
- Rejected AI artifacts do NOT enter Memo0 as validated patterns — only post-HIL-approval patterns are stored

## 7. Default AI Agent — Freebuff CLI (Locked)

**Freebuff (`freebuff` v0.0.160) is the designated default AI agent for all major agentic and coding tasks on this project.**

### What this means
- For all heavy AI coding work — chapter generation, LLE engine changes, pipeline tasks, refactors, multi-file edits — **Freebuff is the first-choice AI agent**.
- Installed globally at: `C:\Users\vanda\AppData\Roaming\npm\freebuff`
- Launch from project root: `cd "C:\Users\vanda\Downloads\html5 ngo"` → `freebuff`
- Pre-authenticated and pointed at this codebase.

### When to use Freebuff
- Generating or editing offline HTML chapter files
- Building or modifying the LLE engine (`Aasha_LLE_Enhanced.js`)
- AI transformation pipeline work (PDF → HTML)
- Multi-file refactors across `server/`, `dashboard/`, or `packages/`
- Code reviews via `/review` and feature planning via `/plan`

### Freebuff Project Context (paste on first prompt in every new session)
```
This is the Aasha EdTech project — offline-first bilingual learning platform
for Class 1–10, all Indian boards (CBSE, ICSE, State, IB, Cambridge, NIOS).
Folders: chapters/ (offline HTML), server/ (Fastify), dashboard/ (React+Vite+Tailwind),
packages/ (chapter-builder, LLE, test-harness).
Always keep chapters fully self-contained (no CDNs), preserve LLE API
(rt(), applyLLE(), showWord(), WM, CONN), never break offline-first isolation.
HIL review required before any artifact goes to production.
```

### Rules
- Do NOT bypass Freebuff for major AI work without explicit product-owner instruction.
- All Freebuff-generated artifacts still require HIL review before production (per Section 6).
- Freebuff does NOT self-approve or self-publish — the evidence lifecycle applies.
