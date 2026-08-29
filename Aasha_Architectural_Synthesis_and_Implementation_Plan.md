# Aasha Child-Centric Learning Platform
# Architectural Synthesis, Technical Specifications & Phased Implementation Plan

**Document Version:** 2.1.0 (Remediated Production Release Architecture)  
**Date:** August 2026  
**Status:** Authoritative Master Architectural Blueprint & Implementation Specification  
**Target Scope:** Class 1 to Class 10 (Ages 6–15) | All Educational Boards (CBSE, ICSE, State Boards, IB, Cambridge, NIOS)  
**File Budget Limits:** Standalone Offline Single-File HTML Chapter $\le$ 20 MB | `Aasha_LLE_Enhanced.js` $\approx$ 60 KB  
**Core Motto & Vision:** *"The book stays; the learning experience changes."*

---

## Executive Summary & Strategic Vision

The **Aasha** platform transforms physical schoolbooks into an interactive, bilingual, gamified, and offline-first digital learning layer. Rather than replacing physical textbooks or demanding constant high-bandwidth internet connectivity, Aasha supercharges the student's existing textbook page: turning static definitions into tactile Canvas simulations, embedding an offline **Language Learning Engine (LLE)** for instant bilingual English-Hindi vocabulary comprehension, delivering **100% dynamically generated assessments** rooted directly in textbook content, and rewarding student progress through localized offline gamification.

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                       AASHA SYSTEM TOPOLOGY & LIFECYCLE                                │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                        │
│   [Teacher / Admin Upload] (Scanned PDF up to 50MB / Images / Text)                                    │
│             │                                                                                          │
│             ▼                                                                                          │
│   ┌───────────────────────────────────┐               ┌───────────────────────────────────────────┐    │
│   │ Fastify REST API & MinIO Storage  │ ────────────▶ │ Cloud OCR (Google Document AI / Layout)   │    │
│   │ (Multipart upload & job dispatch) │               │ Bilingual Text, LaTeX Math, Bounding Boxes│    │
│   └─────────────────┬─────────────────┘               └─────────────────────┬─────────────────────┘    │
│                     │                                                       │                          │
│                     ▼                                                       ▼                          │
│   ┌───────────────────────────────────┐               ┌───────────────────────────────────────────┐    │
│   │ Asynchronous Transformation Queue │ ────────────▶ │ Multi-Stage LLM Pipeline (Temp 0.15-0.35) │    │
│   │ (BullMQ + Redis 7 + Postgres 16)  │               │ ├─ Stage 1: Skeleton & Concept Hierarchy  │    │
│   └─────────────────┬─────────────────┘               │ ├─ Stage 2: Chunked Per-Node LLM Gen      │    │
│                     │                                 │ ├─ Stage 3a: Concept Quizzes & True/False │    │
│                     │                                 │ ├─ Stage 3b: Solve & Tiered Worksheets    │    │
│                     │                                 │ ├─ Stage 3c: Rapid Fire & Memory Match    │    │
│                     │                                 │ ├─ Stage 4: Generative Canvas 2D Scripts  │    │
│                     │                                 │ └─ Stage 5: LLE Dictionary & Translits    │    │
│                     ▼                                 └─────────────────────┬─────────────────────┘    │
│   ┌───────────────────────────────────┐                                     │                          │
│   │ Teacher Dashboard Raw JSON Editor │ ◀───────────────────────────────────┘                          │
│   │ (Monaco/Text-Area AST Editor +    │                                                                │
│   │  Sandboxed Live Chapter Preview)  │                                                                │
│   └─────────────────┬─────────────────┘                                                                │
│                     │                                                                                  │
│                     ▼                                                                                  │
│   ┌───────────────────────────────────┐               ┌───────────────────────────────────────────┐    │
│   │ @aasha/chapter-builder Bundler    │ ────────────▶ │ @aasha/test-harness Validation Suite      │    │
│   │ Injects Design Tokens, LLE Engine,│               │ ├─ Zero-CDN & Dynamic Network Trap Check  │    │
│   │ Audio Oscillators & AST into HTML │               │ ├─ File Size Budget Verification (<=20MB) │    │
│   └─────────────────┬─────────────────┘               │ ├─ 100% Unambiguous MCQ & AST Audit       │    │
│                     │                                 │ └─ Sandboxed Acorn AST & VM DOM Execution │    │
│                     ▼                                 └───────────────────────────────────────────┘    │
│   ┌───────────────────────────────────────────────────────────────────────────────────────────────┐    │
│   │ STANDALONE SINGLE-FILE OFFLINE HTML CHAPTER (<= 20 MB, Zero Runtime Dependencies)            │    │
│   │ ├─ HTML5 Semantic Shell + 150+ CSS Design Tokens (Playful Minimalism)                        │    │
│   │ ├─ Aasha_LLE_Enhanced.js (~60 KB) + 55+ Inline SVG Math/Science Catalog                      │    │
│   │ ├─ Web Speech API (rate 0.85) + Web Audio API Oscillator Tone Fallbacks                       │    │
│   │ ├─ Layer 1: Understanding (Storytelling hooks, Worked Examples, Canvas 2D Manipulatives)     │    │
│   │ ├─ Layer 2: Mastery (Dynamic Quizzes, Solve Scenarios, Tiered Worksheets, Rapid Fire)         │    │
│   │ └─ Local State & Gamification (XP, Coins, Badges, Profiles in localStorage / IndexedDB)      │    │
│   └───────────────────────────────────────────────┬───────────────────────────────────────────────┘    │
│                                                   │                                                    │
│                                                   ▼ (Opportunistic Sync when Online)                   │
│   ┌───────────────────────────────────────────────────────────────────────────────────────────────┐    │
│   │ POST /api/v1/sync/push (Idempotent Batch Event Ledger Reconciliation)                         │    │
│   └───────────────────────────────────────────────────────────────────────────────────────────────┘    │
│                                                                                                        │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

# Section 1: Technical & Architectural Synthesis (R1)

## 1.1 Monorepo Architecture & Workspace Layout

The Aasha repository is architected as an **npm workspaces monorepo** enforcing strict modular boundaries between client-side offline execution packages, server APIs, and teacher interfaces.

```
html5 ngo/
├── .agents/                                # Multi-agent coordination metadata
├── chapters/                               # Standalone compiled offline HTML chapters
│   ├── comparing-quantities.html           # Class 7 NCERT Math (342 KB)
│   ├── fractions.html                      # Class 6 NCERT Math (387 KB)
│   └── quadrilaterals.html                 # Class 8 NCERT Math (416 KB)
├── dashboard/                              # Teacher & Admin Web Dashboard
│   ├── package.json                        # React 18, Vite, Tailwind CSS, Monaco Editor
│   ├── src/
│   │   ├── components/                     # Raw JSON Editor, Upload Queue, Job Monitor, Preview Frame
│   │   ├── pages/                          # Chapter Review, Analytics, Class Management
│   │   └── App.tsx                         # Dashboard Routing and State
│   └── vite.config.ts
├── infra/                                  # Containerized Local & Production Infrastructure
│   ├── .env.example                        # DB, Redis, MinIO, JWT credentials
│   └── docker-compose.yml                  # PostgreSQL 16, Redis 7, MinIO S3
├── packages/
│   ├── chapter-builder/                    # Chapter Compilation & HTML Bundling CLI/Lib
│   │   ├── package.json
│   │   └── src/index.ts                    # AST to HTML compiler, template injection
│   ├── shared-types/                       # Canonical Type Definitions across Monorepo
│   │   ├── package.json
│   │   └── src/index.ts                    # NodeAST, QuestionBank, Sync, User, Job types
│   └── test-harness/                       # Reusable Chapter Integrity & Compliance Suite
│       ├── package.json
│       └── src/index.ts                    # Zero-CDN, <=20MB, VM runtime & Quiz validation
├── server/                                 # Fastify Node.js 20 LTS API Server
│   ├── package.json                        # Fastify, @fastify/multipart, @fastify/jwt, pg, BullMQ
│   ├── src/
│   │   ├── db/
│   │   │   ├── client.ts                   # PostgreSQL pg.Pool wrapper
│   │   │   └── migrations/                 # DDL migration scripts (Up/Down SQL)
│   │   ├── plugins/                        # JWT authentication, RBAC, Rate Limiting, Error Handler
│   │   ├── routes/
│   │   │   ├── auth.ts                     # User registration, password, OTP login
│   │   │   ├── chapters.ts                 # Source upload, transformation job status, listing
│   │   │   └── core.ts                     # Schools, classes, child profiles, sync engine
│   │   └── index.ts                        # Fastify server entrypoint
│   └── tsconfig.json
├── package.json                            # Monorepo root workspace configuration
└── tsconfig.json                           # Root TypeScript base compiler options
```

### Workspace Configuration (`package.json`)
```json
{
  "name": "aasha",
  "private": true,
  "workspaces": [
    "server",
    "dashboard",
    "packages/*"
  ],
  "scripts": {
    "dev:server": "npm run dev --workspace=server",
    "dev:dashboard": "npm run dev --workspace=dashboard",
    "build:server": "npm run build --workspace=server",
    "build:dashboard": "npm run build --workspace=dashboard",
    "build:all": "npm run build --workspaces --if-present",
    "test": "npm run test --workspaces --if-present"
  }
}
```

### Package Roles & Constraints Matrix

| Package / Directory | Technology Stack | Primary Responsibility | Critical Constraints |
|---|---|---|---|
| `packages/shared-types` | TypeScript 5.x | Source of truth for domain models, AST interfaces, DTO schemas, and database entity types. | Zero runtime dependencies; strictly type declarations and enums. 100% parity with DB & AST schemas. |
| `packages/chapter-builder` | Node.js, `vm`, `pg`, `dotenv` | Ingests structured JSON AST from AI pipeline or raw editor, injects CSS design tokens and LLE engine, and compiles single-file HTML. | Must produce 100% self-contained HTML files without external script tags. |
| `packages/test-harness` | Node.js, headless `vm` sandbox, `acorn` AST parser | Automated validation suite: enforces budget `<= 20 MB`, zero external CDNs/dynamic network traps, 100% exact quiz answer key correctness, step AST integrity, and dictionary cross-references. | Runs during CI/CD and pre-publish pipeline; fails fast on any violation. |
| `server/` | Fastify, Node 20 LTS, PostgreSQL, Redis, MinIO SDK | REST API handling multipart uploads (up to 50MB), job queue dispatching, RBAC, analytics aggregation, and idempotent push/pull sync. | High performance (<200ms p95 latency); robust error isolation in background jobs; standardized error envelope. |
| `dashboard/` | React 18, Vite, Tailwind CSS | Teacher/Admin UI providing simple raw JSON text-area/Monaco AST editing, prompt configuration, upload dropzone, and transformation tracker. | Lightweight, highly responsive, embeds chapter preview iframe seamlessly. |
| `infra/` | Docker Compose, Postgres 16, Redis 7, MinIO | Containerized local & server environment with automated migrations and S3-compatible asset bucket. | Fully reproducible on standard low-cost VPS environments. |

---

## 1.2 Target Audience & Board Scope Expansion

Aasha expands beyond single-board middle school material to cover **Class 1 through Class 10 (ages 6–15)** across all Indian national, state, and international curricula: **CBSE, ICSE, State Boards (e.g. Maharashtra, Karnataka, UP, TN, WB), IB (PYP/MYP), Cambridge (Primary/Lower Secondary/IGCSE), and NIOS**.

```
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                AGE-ADAPTIVE DESIGN & PEDAGOGICAL MATRIX                          │
├────────────────────────────────┬────────────────────────────────┬────────────────────────────────┤
│ Primary (Class 1–3, Ages 6–8)  │ Middle (Class 4–7, Ages 9–12)  │ Secondary (Class 8–10, 13–15)  │
├────────────────────────────────┼────────────────────────────────┼────────────────────────────────┤
│ • Font Scale: 1.1x (Base 18px) │ • Font Scale: 1.0x (Base 16px) │ • Font Scale: 1.0x (Base 16px) │
│ • Min Tap Target: 48x48px      │ • Min Tap Target: 44x44px      │ • Min Tap Target: 44x44px      │
│ • Content Density: 1 concept,  │ • Content Density: 1 concept,  │ • Content Density: High density│
│   1 interaction per step       │   2–3 progressive steps        │   with multi-step proofs       │
│ • Visual Ratio: 70% Canvas /   │ • Visual Ratio: 50% Visual /   │ • Visual Ratio: 30% Visual /   │
│   30% Text                     │   50% Text                     │   70% Text & Formal Math       │
│ • LLE Mode: Deep phonetics &   │ • LLE Mode: Moderate academic  │ • LLE Mode: Technical terms &  │
│   Romanized Hindi vocabulary   │   vocabulary bridge            │   formula derivations          │
│ • Gamification: Confetti, Web  │ • Gamification: Speed bonus,   │ • Gamification: Mastery rings, │
│   Audio celebration, high XP   │   Rapid Fire, streak freezes   │   HOTS badges, subtle cues     │
└────────────────────────────────┴────────────────────────────────┴────────────────────────────────┘
```

---

## 1.3 Self-Contained Chapter Asset & File Budget Architecture

Each generated Aasha chapter is an autonomous, **single-file HTML document** with a hard budget ceiling of **$\le$ 20 MB** (typical chapter size: **1.2 MB – 4.5 MB**).

```
┌───────────────────────────────────────────────────────────────────────────┐
│                     20 MB CHAPTER FILE BUDGET ALLOCATION                  │
├─────────────────────────────────────────┬───────────────┬─────────────────┤
│ Layer Component                         │ Typical Size  │ Max Allowance   │
├─────────────────────────────────────────┼───────────────┼─────────────────┤
│ 1. HTML5 Semantic Shell & DOM Structure │ 150 – 300 KB  │ 500 KB          │
│ 2. Inline CSS Tokens & Theme Variables  │ 25 – 45 KB    │ 100 KB          │
│ 3. Core Engine JavaScript (App State)   │ 120 – 250 KB  │ 500 KB          │
│ 4. Enhanced LLE Engine (`Aasha_LLE`)   │ ~60 KB        │ 100 KB          │
│ 5. Chapter AST Data (NODES, WE, SOLVE)  │ 200 – 600 KB  │ 1,500 KB        │
│ 6. Inline SVG Illustration Dictionary   │ 150 – 400 KB  │ 1,000 KB        │
│ 7. Programmatic Web Audio Tone Engine   │ ~5 KB         │ 20 KB           │
│ 8. High-Res Inline Media / Canvas Code  │ 500 – 2,500 KB│ 16,000 KB       │
├─────────────────────────────────────────┼───────────────┼─────────────────┤
│ TOTAL FILE SIZE TARGET                  │ ~1.2 – 4.5 MB │ <= 20.00 MB     │
└─────────────────────────────────────────┴───────────────┴─────────────────┘
```

### Zero External Runtime Dependency Rules
1. **Zero External CSS/JS**: No `<link rel="stylesheet">` or `<script src="https://...">` tags. All styles and scripts are embedded within `<style>` and `<script>` blocks.
2. **Zero External Media Assets**: No remote images (`<img src="http...">`). All graphics are rendered via programmatic HTML5 Canvas 2D, inline vector SVGs, or Base64 data URIs.
3. **Programmatic Audio via Web Audio API**: No external MP3/WAV audio files. Audio cues (correct chime, wrong buzz, fanfare) are synthesized via `AudioContext` oscillators.
4. **Browser-Native Pronunciation via Web Speech API**: Native `window.speechSynthesis` speaks English and Hindi at rate `0.85` with automatic rhythmic tone fallbacks for unequipped devices.
5. **System Font Stack**: Uses native OS system font stacks (`system-ui, -apple-system, "Segoe UI", Roboto, sans-serif`) with zero web-font download overhead.

---

## 1.4 PostgreSQL Database Schemas (DDL & TypeScript Parity)

### Full PostgreSQL 16 DDL (Production Grade & Validated)

```sql
-- ============================================================================
-- AASHA PRODUCTION CORE DATABASE SCHEMA (PostgreSQL 16)
-- ============================================================================

CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- Enums
DO $$ BEGIN
    CREATE TYPE user_role AS ENUM ('super_admin', 'org_admin', 'school_admin', 'teacher', 'parent');
EXCEPTION WHEN duplicate_object THEN NULL; END $$;

DO $$ BEGIN
    CREATE TYPE board_code AS ENUM ('cbse', 'icse', 'state', 'ib', 'cambridge', 'nios');
EXCEPTION WHEN duplicate_object THEN NULL; END $$;

DO $$ BEGIN
    CREATE TYPE subject_code AS ENUM ('mathematics', 'science', 'english', 'hindi', 'social_studies', 'general_knowledge', 'computer_science');
EXCEPTION WHEN duplicate_object THEN NULL; END $$;

DO $$ BEGIN
    CREATE TYPE source_material_type AS ENUM ('pdf_digital', 'pdf_scanned', 'image', 'text', 'ebook');
EXCEPTION WHEN duplicate_object THEN NULL; END $$;

DO $$ BEGIN
    CREATE TYPE transformation_status AS ENUM (
        'queued', 'extracting', 'analyzing', 'generating_content', 
        'generating_assessments', 'assembling', 'validating', 'review', 
        'published', 'failed', 'cancelled'
    );
EXCEPTION WHEN duplicate_object THEN NULL; END $$;

DO $$ BEGIN
    CREATE TYPE transformation_stage AS ENUM (
        'upload', 'text_extraction', 'structure_analysis', 'content_generation', 
        'assessment_generation', 'chapter_assembly', 'validation', 'review', 'publish'
    );
EXCEPTION WHEN duplicate_object THEN NULL; END $$;

-- ----------------------------------------------------------------------------
-- 0. Table: users (Core Reference Table)
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS users (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email               VARCHAR(255) UNIQUE NOT NULL,
    full_name           VARCHAR(255) NOT NULL,
    role                user_role NOT NULL DEFAULT 'teacher',
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at          TIMESTAMPTZ
);

-- ----------------------------------------------------------------------------
-- 1. Table: source_uploads
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS source_uploads (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    teacher_id          UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    file_name           VARCHAR(255) NOT NULL,
    file_size_bytes     BIGINT NOT NULL CHECK (file_size_bytes > 0 AND file_size_bytes <= 52428800), -- Max 50 MB
    mime_type           VARCHAR(100) NOT NULL,
    storage_path        VARCHAR(500) NOT NULL,
    material_type       source_material_type NOT NULL DEFAULT 'pdf_digital',
    board               board_code NOT NULL DEFAULT 'cbse',
    grade               INTEGER NOT NULL CHECK (grade BETWEEN 1 AND 10),
    subject             subject_code NOT NULL DEFAULT 'mathematics',
    status              VARCHAR(30) NOT NULL DEFAULT 'uploaded',
    ocr_storage_path    VARCHAR(500), -- MinIO path to full OCR JSON payload
    ocr_metadata        JSONB NOT NULL DEFAULT '{}', -- Lightweight extraction summary
    metadata            JSONB NOT NULL DEFAULT '{}',
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at          TIMESTAMPTZ
);

CREATE INDEX IF NOT EXISTS idx_source_uploads_teacher ON source_uploads(teacher_id) WHERE deleted_at IS NULL;
CREATE INDEX IF NOT EXISTS idx_source_uploads_grade_board ON source_uploads(grade, board);

-- ----------------------------------------------------------------------------
-- 2. Table: transformation_jobs
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS transformation_jobs (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    upload_id           UUID NOT NULL REFERENCES source_uploads(id) ON DELETE CASCADE,
    triggered_by        UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    current_stage       transformation_stage NOT NULL DEFAULT 'upload',
    status              transformation_status NOT NULL DEFAULT 'queued',
    progress_percent    NUMERIC(5,2) NOT NULL DEFAULT 0.00 CHECK (progress_percent BETWEEN 0.00 AND 100.00),
    error_log           TEXT,
    llm_tokens_used     INTEGER NOT NULL DEFAULT 0 CHECK (llm_tokens_used >= 0),
    llm_cost_usd        NUMERIC(10,4) NOT NULL DEFAULT 0.0000 CHECK (llm_cost_usd >= 0.0000),
    retry_count         INTEGER NOT NULL DEFAULT 0 CHECK (retry_count >= 0),
    max_retries         INTEGER NOT NULL DEFAULT 3 CHECK (max_retries >= 0),
    stage_states        JSONB NOT NULL DEFAULT '[]', -- Array of StageExecutionState objects
    config              JSONB NOT NULL DEFAULT '{
        "quizPerConcept": 3,
        "trueFalsePerConcept": 2,
        "fillBlankPerConcept": 2,
        "solvePerConcept": 1,
        "worksheetLevels": ["basic", "standard", "hots", "final_mixed"],
        "generateRapidFire": true,
        "generateMemoryMatch": true,
        "lleLanguage": "hi-IN"
    }',
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_transformation_jobs_upload ON transformation_jobs(upload_id);
CREATE INDEX IF NOT EXISTS idx_transformation_jobs_status ON transformation_jobs(status);

-- ----------------------------------------------------------------------------
-- 3. Table: generated_chapters
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS generated_chapters (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_id              UUID REFERENCES transformation_jobs(id) ON DELETE SET NULL,
    source_upload_id    UUID REFERENCES source_uploads(id) ON DELETE SET NULL,
    title               VARCHAR(255) NOT NULL,
    title_hindi         VARCHAR(255),
    slug                VARCHAR(150) NOT NULL,
    grade               INTEGER NOT NULL CHECK (grade BETWEEN 1 AND 10),
    subject             subject_code NOT NULL DEFAULT 'mathematics',
    board               board_code NOT NULL DEFAULT 'cbse',
    chapter_number      INTEGER NOT NULL DEFAULT 1 CHECK (chapter_number > 0),
    file_size_bytes     BIGINT NOT NULL DEFAULT 0 CHECK (file_size_bytes >= 0),
    html_content_path   VARCHAR(500),
    ast_json            JSONB NOT NULL DEFAULT '{}',
    lle_word_count      INTEGER NOT NULL DEFAULT 0 CHECK (lle_word_count >= 0),
    node_count          INTEGER NOT NULL DEFAULT 0 CHECK (node_count >= 0),
    step_count          INTEGER NOT NULL DEFAULT 0 CHECK (step_count >= 0),
    is_published        BOOLEAN NOT NULL DEFAULT FALSE,
    review_status       VARCHAR(30) NOT NULL DEFAULT 'in_review' CHECK (review_status IN ('in_review', 'approved', 'rejected')),
    reviewed_by         UUID REFERENCES users(id) ON DELETE SET NULL,
    reviewed_at         TIMESTAMPTZ,
    version             VARCHAR(20) NOT NULL DEFAULT '1.0.0',
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at          TIMESTAMPTZ
);

-- Partial Unique Index to permit re-creating chapters with same slug if soft-deleted
CREATE UNIQUE INDEX IF NOT EXISTS idx_generated_chapters_slug_active ON generated_chapters(slug) WHERE deleted_at IS NULL;
CREATE INDEX IF NOT EXISTS idx_generated_chapters_grade_subject ON generated_chapters(grade, subject, board);

-- ----------------------------------------------------------------------------
-- 4. Table: prompt_templates
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS prompt_templates (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name                VARCHAR(100) NOT NULL,
    stage               transformation_stage NOT NULL,
    version             INTEGER NOT NULL DEFAULT 1 CHECK (version > 0),
    system_prompt       TEXT NOT NULL,
    template_text       TEXT NOT NULL,
    input_schema        JSONB NOT NULL DEFAULT '{}',
    output_schema       JSONB NOT NULL DEFAULT '{}',
    model_target        VARCHAR(100) NOT NULL DEFAULT 'gpt-4o',
    temperature         NUMERIC(3,2) NOT NULL DEFAULT 0.30 CHECK (temperature BETWEEN 0.0 AND 1.0),
    max_tokens          INTEGER NOT NULL DEFAULT 4096 CHECK (max_tokens > 0),
    is_active           BOOLEAN NOT NULL DEFAULT TRUE,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT uq_prompt_templates_name_version UNIQUE (name, version)
);

-- Partial unique index ensuring exactly one active version per template name
CREATE UNIQUE INDEX IF NOT EXISTS idx_prompt_templates_active ON prompt_templates(name) WHERE is_active = TRUE;
CREATE INDEX IF NOT EXISTS idx_prompt_templates_stage ON prompt_templates(stage) WHERE is_active = TRUE;

-- ----------------------------------------------------------------------------
-- 5. Tables: student_progress & sync_events
-- ----------------------------------------------------------------------------
CREATE TABLE IF NOT EXISTS student_progress (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id            UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    chapter_id          VARCHAR(100) NOT NULL,
    current_node_index  INTEGER NOT NULL DEFAULT 0 CHECK (current_node_index >= 0),
    current_step_index  INTEGER NOT NULL DEFAULT 0 CHECK (current_step_index >= 0),
    xp_earned           INTEGER NOT NULL DEFAULT 0 CHECK (xp_earned >= 0),
    coins_earned        INTEGER NOT NULL DEFAULT 0 CHECK (coins_earned >= 0),
    correct_count       INTEGER NOT NULL DEFAULT 0 CHECK (correct_count >= 0),
    incorrect_count     INTEGER NOT NULL DEFAULT 0 CHECK (incorrect_count >= 0),
    is_completed        BOOLEAN NOT NULL DEFAULT FALSE,
    completed_at        TIMESTAMPTZ,
    last_synced_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    sync_version        INTEGER NOT NULL DEFAULT 1 CHECK (sync_version > 0),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE(child_id, chapter_id)
);

CREATE TABLE IF NOT EXISTS sync_events (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id            UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    device_uuid         VARCHAR(255) NOT NULL,
    event_id            VARCHAR(100) NOT NULL,
    chapter_id          VARCHAR(100) NOT NULL,
    event_type          VARCHAR(50) NOT NULL,
    event_payload       JSONB NOT NULL DEFAULT '{}',
    client_timestamp    TIMESTAMPTZ NOT NULL,
    server_timestamp    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    reconciled          BOOLEAN NOT NULL DEFAULT TRUE,
    UNIQUE(child_id, event_id)
);

CREATE INDEX IF NOT EXISTS idx_sync_events_child ON sync_events(child_id, server_timestamp);
```

---

### Complete TypeScript Definitions (`packages/shared-types/src/index.ts`)

```typescript
// packages/shared-types/src/index.ts

export type UserRole = 'super_admin' | 'org_admin' | 'school_admin' | 'teacher' | 'parent';
export type BoardCode = 'cbse' | 'icse' | 'state' | 'ib' | 'cambridge' | 'nios';
export type SubjectCode = 'mathematics' | 'science' | 'english' | 'hindi' | 'social_studies' | 'general_knowledge' | 'computer_science';
export type SourceMaterialType = 'pdf_digital' | 'pdf_scanned' | 'image' | 'text' | 'ebook';

export type TransformationStatus = 
  | 'queued' | 'extracting' | 'analyzing' | 'generating_content' 
  | 'generating_assessments' | 'assembling' | 'validating' | 'review' 
  | 'published' | 'failed' | 'cancelled';

export type TransformationStage = 
  | 'upload' | 'text_extraction' | 'structure_analysis' | 'content_generation' 
  | 'assessment_generation' | 'chapter_assembly' | 'validation' | 'review' | 'publish';

export type ReviewStatus = 'in_review' | 'approved' | 'rejected';

export interface StageExecutionState {
  stage: TransformationStage;
  status: 'pending' | 'running' | 'completed' | 'failed';
  durationMs?: number;
  error?: string;
}

// ----------------------------------------------------------------------------
// Database Entity Interfaces
// ----------------------------------------------------------------------------

export interface User {
  id: string;
  email: string;
  fullName: string;
  role: UserRole;
  createdAt: string;
  updatedAt: string;
  deletedAt?: string;
}

export interface SourceUpload {
  id: string;
  teacherId: string;
  fileName: string;
  fileSizeBytes: number;
  mimeType: string;
  storagePath: string;
  materialType: SourceMaterialType;
  board: BoardCode;
  grade: number;
  subject: SubjectCode;
  status: string;
  ocrStoragePath?: string;
  ocrMetadata: Record<string, any>;
  metadata: Record<string, any>;
  createdAt: string;
  updatedAt: string;
  deletedAt?: string;
}

export interface TransformationJob {
  id: string;
  uploadId: string;
  triggeredBy: string;
  currentStage: TransformationStage;
  status: TransformationStatus;
  progressPercent: number;
  errorLog?: string;
  llmTokensUsed: number;
  llmCostUsd: number;
  retryCount: number;
  maxRetries: number;
  stageStates: StageExecutionState[];
  config: TransformationConfig;
  createdAt: string;
  updatedAt: string;
}

export interface GeneratedChapter {
  id: string;
  jobId?: string;
  sourceUploadId?: string;
  title: string;
  titleHindi?: string;
  slug: string;
  grade: number;
  subject: SubjectCode;
  board: BoardCode;
  chapterNumber: number;
  fileSizeBytes: number;
  htmlContentPath?: string;
  astJson: ChapterAST;
  lleWordCount: number;
  nodeCount: number;
  stepCount: number;
  isPublished: boolean;
  reviewStatus: ReviewStatus;
  reviewedBy?: string;
  reviewedAt?: string;
  version: string;
  createdAt: string;
  updatedAt: string;
  deletedAt?: string;
}

export interface PromptTemplate {
  id: string;
  name: string;
  stage: TransformationStage;
  version: number;
  systemPrompt: string;
  templateText: string;
  inputSchema: Record<string, any>;
  outputSchema: Record<string, any>;
  modelTarget: string;
  temperature: number;
  maxTokens: number;
  isActive: boolean;
  createdAt: string;
  updatedAt: string;
}

export interface StudentProgress {
  id: string;
  childId: string;
  chapterId: string;
  currentNodeIndex: number;
  currentStepIndex: number;
  xpEarned: number;
  coinsEarned: number;
  correctCount: number;
  incorrectCount: number;
  isCompleted: boolean;
  completedAt?: string;
  lastSyncedAt: string;
  syncVersion: number;
  createdAt: string;
  updatedAt: string;
}

export interface SyncEvent {
  id: string;
  childId: string;
  deviceUuid: string;
  eventId: string;
  chapterId: string;
  eventType: string;
  eventPayload: Record<string, any>;
  clientTimestamp: string;
  serverTimestamp: string;
  reconciled: boolean;
}

export interface TransformationConfig {
  quizPerConcept: number;
  trueFalsePerConcept: number;
  fillBlankPerConcept: number;
  solvePerConcept: number;
  worksheetLevels: ('basic' | 'standard' | 'hots' | 'final_mixed')[];
  generateRapidFire: boolean;
  generateMemoryMatch: boolean;
  lleLanguage: string;
}

// ----------------------------------------------------------------------------
// Chapter AST Canonical Types
// ----------------------------------------------------------------------------

export interface ChapterAST {
  metadata: {
    title: string;
    titleHindi?: string;
    grade: number;
    subject: SubjectCode;
    board: BoardCode;
    chapterNumber: number;
  };
  nodes: ChapterNodeAST[];
  workedExamples: Record<string, WorkedExampleAST>;
  solveScenarios: Record<string, SolveScenarioAST>;
  lleWordMap: Record<string, LLEWordEntry>;
  lleConnectives: string[];
}

export interface ChapterNodeAST {
  id: string;
  title: string;
  titleHindi?: string;
  subtitle?: string;
  icon: string;
  color?: string;
  steps: NodeStepAST[];
}

export type StepType = 
  | 'intro' | 'text' | 'canvas_visual' | 'ix_drag' | 'ix_slider' 
  | 'we' | 'solve' | 'quiz' | 'truefalse' | 'fillblank' | 'progress' 
  | 'rapid_fire' | 'memory_match';

export interface NodeStepAST {
  t: StepType;
  title?: string;
  text?: string;
  caption?: string;
  icon?: string;
  visualScript?: string;
  draw?: {
    type: string;
    params: Record<string, any>;
  };
  q?: QuizQuestionAST | FillBlankAST | TrueFalseAST;
  we?: string; // String reference key resolving in ChapterAST.workedExamples
  sl?: string; // String reference key resolving in ChapterAST.solveScenarios
  diff?: 'easy' | 'medium' | 'hard' | 'hots';
  timerSeconds?: number;
}

export interface QuizQuestionAST {
  prompt: string;
  opts: {
    text: string;
    c: boolean;
    feedback?: string;
  }[];
}

export interface FillBlankAST {
  sentence: string;
  answer: string;
  acceptableAnswers?: string[];
  unit?: string;
  explanation?: string;
}

export interface TrueFalseAST {
  statement: string;
  answer: boolean;
  explanation: string;
}

export interface WorkedExampleAST {
  title: string;
  problem: string;
  steps: {
    label: string;
    content: string;
    color?: 'blue' | 'green' | 'amber';
    final?: boolean;
    canvasAction?: string;
  }[];
}

export interface SolveScenarioAST {
  title: string;
  prompt: string;
  steps: {
    instruction: string;
    expectedAnswer: string;
    hint?: string;
  }[];
}

export interface LLEWordEntry {
  h: string;
  t: string;
  p: 'n' | 'v' | 'adj' | 'adv' | 'prep' | 'conj';
  g: number;
  s: 'math' | 'science' | 'general';
  d: string;
  svg?: string;
  ex: string;
  ex_hi?: string;
}

export interface SyncPushPayload {
  deviceId: string;
  childId: string;
  events: SyncProgressEvent[];
}

export interface SyncProgressEvent {
  eventId: string;
  chapterId: string;
  nIdx: number;
  sIdx: number;
  xp: number;
  coins: number;
  clientTimestamp: string;
}
```

---

## 1.5 Backend REST API Endpoints & Standardized Error Taxonomy

All endpoints are versioned under `/api/v1/` and return consistent JSON envelopes.

### Standardized Error Response Envelope
Every non-2xx response adheres strictly to the canonical error structure:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_FAILED",
    "statusCode": 422,
    "message": "Chapter AST failed validation in @aasha/test-harness",
    "details": [
      "Node 'node_2' Step 3 (quiz): MCQ option '12 cm' is duplicated",
      "Word 'rhombus' tagged in text is missing from WM dictionary"
    ],
    "timestamp": "2026-08-25T17:45:00Z"
  }
}
```

### HTTP Error Status Code Taxonomy

| Status Code | Error Code (`error.code`) | Scenario / Trigger | Resolution Action |
|---|---|---|---|
| **400 Bad Request** | `BAD_REQUEST` | Malformed JSON AST, missing multipart form fields, invalid query parameters. | Inspect request payload syntax. |
| **401 Unauthorized** | `UNAUTHORIZED` | Missing, expired, or cryptographically invalid Bearer JWT. | Refresh or re-issue JWT token. |
| **403 Forbidden** | `FORBIDDEN` | Teacher lacks permissions to modify chapters belonging to another organization. | Check user role and school mapping. |
| **404 Not Found** | `NOT_FOUND` | Specified `upload_id`, `job_id`, `chapter_id`, or `user_id` does not exist. | Verify resource UUID. |
| **409 Conflict** | `CONFLICT` | Slug collision for active chapter, or optimistic locking sync collision. | Change chapter slug or fetch latest version. |
| **413 Payload Too Large** | `PAYLOAD_TOO_LARGE` | Uploaded PDF or image exceeds the **50 MB** hard limit. | Compress PDF or split into chapters. |
| **422 Unprocessable Entity**| `VALIDATION_FAILED` | Chapter AST violates `@aasha/test-harness` rules (e.g. duplicate MCQ options, CDN leaks, invalid answer keys). | Review validation error details and edit AST. |
| **500 Internal Server Error**| `INTERNAL_ERROR` | Unrecoverable cloud OCR failure, LLM rate-limit exhaustion after 3 retries, or MinIO storage failure. | Check server logs and BullMQ error queue. |

---

### Endpoint Specifications

#### 1. Source Material Upload (`POST /api/v1/uploads`)
- **Headers**: `Authorization: Bearer <JWT>`, `Content-Type: multipart/form-data`
- **Form Fields**: `file` (Binary PDF/Image, $\le 50$ MB), `board`, `grade`, `subject`, `chapterTitle` (optional)
- **Response `201 Created`**:
```json
{
  "success": true,
  "data": {
    "uploadId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "jobId": "8ba12f84-1234-4562-b3fc-7a963f66bbb1",
    "fileName": "Class8_Math_Quadrilaterals_Ch3.pdf",
    "fileSizeBytes": 14205812,
    "status": "queued",
    "createdAt": "2026-08-25T17:30:00Z"
  }
}
```

#### 2. Transformation Job Status (`GET /api/v1/transformations/:id/status`)
- **Response `200 OK`**:
```json
{
  "success": true,
  "data": {
    "jobId": "8ba12f84-1234-4562-b3fc-7a963f66bbb1",
    "status": "generating_content",
    "currentStage": "content_generation",
    "progressPercent": 60.00,
    "stages": [
      { "stage": "upload", "status": "completed", "durationMs": 450 },
      { "stage": "text_extraction", "status": "completed", "durationMs": 12500 },
      { "stage": "structure_analysis", "status": "completed", "durationMs": 8200 },
      { "stage": "content_generation", "status": "running", "durationMs": 14300 },
      { "stage": "assessment_generation", "status": "pending" },
      { "stage": "chapter_assembly", "status": "pending" },
      { "stage": "validation", "status": "pending" }
    ],
    "llmTokensUsed": 18450,
    "llmCostUsd": 0.0554,
    "generatedChapterId": null
  }
}
```

#### 3. Chapter AST Retrieval & Update (`GET` & `PUT /api/v1/chapters/:id`)
- `GET /api/v1/chapters/:id`: Returns full `GeneratedChapter` entity including `astJson`.
- `PUT /api/v1/chapters/:id`: Accepts updated `ChapterAST` payload; executes `@aasha/test-harness` validation before committing changes and triggering single-file HTML recompilation.

#### 4. Idempotent Offline Sync (`POST /api/v1/sync/push`)
- **Request Body**:
```json
{
  "deviceId": "dev_991823-android-tablet",
  "childId": "c018274a-4433-2211-00aa-112233445566",
  "events": [
    {
      "eventId": "evt_1724581230000_1",
      "chapterId": "quadrilaterals",
      "nIdx": 3,
      "sIdx": 2,
      "xp": 15,
      "coins": 5,
      "clientTimestamp": "2026-08-25T16:45:00Z"
    }
  ]
}
```
- **Response `200 OK`**:
```json
{
  "success": true,
  "data": {
    "acknowledgedEventIds": ["evt_1724581230000_1"],
    "serverTimestamp": "2026-08-25T17:45:01Z",
    "mergedProgress": {
      "chapterId": "quadrilaterals",
      "currentNodeIndex": 3,
      "currentStepIndex": 2,
      "totalXp": 335,
      "totalCoins": 150
    }
  }
}
```

#### 5. Automated Chapter Verification (`POST /api/v1/verify/chapter`)
- Ingests raw HTML content string, executes the full 5-point verification suite in `@aasha/test-harness`, and returns structured validation diagnostics.

---

## 1.6 Teacher Dashboard Architecture (Raw JSON Editor & Queue)

In the Beta phase, the teacher dashboard prioritizes simplicity and direct control:
1. **Upload Queue & Monitor**: Drag-and-drop zone supporting PDFs up to 50MB with live multi-stage progress tracking.
2. **Raw JSON AST Text-Area / Monaco Editor**: Full syntax highlighting and schema validation against `@aasha/shared-types`. Allows educators to fine-tune AI-generated text, correct Hindi translations, modify quiz misconception feedback, and adjust question difficulty parameters.
3. **Live Sandboxed Preview Iframe**: Renders the compiled single-file HTML chapter in a secure local frame with instant hot-recompilation on AST save.

---

# Section 2: AI Content & Assessment Transformation Pipeline (R2)

## 2.1 Ingestion, Pre-OCR Normalization & Cloud OCR Strategy

Textbooks in Indian schools range from crisp digital PDFs to low-grade newsprint scans with multi-column layouts, bilingual Hindi-English sidebars, complex mathematical fractions ($\frac{a}{b}$), geometric figures, and tables.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        PRE-OCR IMAGE NORMALIZATION PIPELINE                            │
├────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                        │
│   [Raw Uploaded PDF / Scan]                                                            │
│             │                                                                          │
│             ▼                                                                          │
│   ┌──────────────────────────────────────────────────────────────────────────────┐     │
│   │ 1. PDF Page Rasterization (300 DPI high-density rendering via `pdf2pic`)      │     │
│   └──────────────────────────────────────┬───────────────────────────────────────┘     │
│                                          │                                             │
│                                          ▼                                             │
│   ┌──────────────────────────────────────────────────────────────────────────────┐     │
│   │ 2. Auto-Deskewing (Hough Transform / Radon deskew for skew ±15°)             │     │
│   └──────────────────────────────────────┬───────────────────────────────────────┘     │
│                                          │                                             │
│                                          ▼                                             │
│   ┌──────────────────────────────────────────────────────────────────────────────┐     │
│   │ 3. Gutter Shadow & Binding Artifact Removal (Illumination Estimation)        │     │
│   └──────────────────────────────────────┬───────────────────────────────────────┘     │
│                                          │                                             │
│                                          ▼                                             │
│   ┌──────────────────────────────────────────────────────────────────────────────┐     │
│   │ 4. Adaptive Binarization (Sauvola / Otsu thresholding for newsprint scans)   │     │
│   └──────────────────────────────────────┬───────────────────────────────────────┘     │
│                                          │                                             │
│                                          ▼                                             │
│   ┌──────────────────────────────────────────────────────────────────────────────┐     │
│   │ 5. High-Accuracy Cloud OCR (Google Document AI Layout Parser)                │     │
│   └──────────────────────────────────────┬───────────────────────────────────────┘     │
│                                          │                                             │
│                                          ▼                                             │
│   ┌──────────────────────────────────────────────────────────────────────────────┐     │
│   │ 6. Post-OCR Unicode NFC Normalization (`text.normalize('NFC')`) & Math Sanitizer│   │
│   └──────────────────────────────────────────────────────────────────────────────┘     │
│                                                                                        │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

### Pre-OCR Image Normalization Implementation Details
1. **Auto-Deskewing ($\pm 15^\circ$)**: Scanned pages often have rotation skew. Using Sharp and OpenCV edge detection with Hough transform, lines are detected and the image is auto-rotated to straight orientation.
2. **Gutter Shadow Removal**: Book bindings create dark gradients along the inner margin. Background illumination estimation is applied to normalize light levels across the page.
3. **Adaptive Thresholding (Sauvola)**: Standard thresholding fails on yellowed or thin paper where text from the reverse page bleeds through. Sauvola local thresholding ensures sharp text edges while suppressing bleed-through artifacts.
4. **Mandatory Unicode NFC Normalization**:
   Devanagari OCR frequently produces decomposed Unicode (`NFD`), split matras (`\u093E`, `\u0947`), or rogue zero-width joiners (`\u200D`), breaking vocabulary lookups in the Word Map (`WM`) and corrupting TTS engines. Every extracted OCR segment is normalized immediately:
   ```typescript
   export function normalizeOcrText(rawText: string): string {
     // 1. Unicode NFC Canonical Composition
     let text = rawText.normalize('NFC');
     // 2. Remove orphan zero-width spaces/joiners that break dictionary matches
     text = text.replace(/[\u200B\u200C\u200E\u200F]/g, '');
     return text;
   }
   ```
5. **Math Sanitizer Pass**:
   Standardizes OCR mathematical artifacts into standard LaTeX / arithmetic expressions:
   ```typescript
   export function sanitizeMathText(text: string): string {
     return text
       .replace(/\u2212/g, '-')           // Unicode minus to hyphen-minus
       .replace(/\u00D7/g, ' \\times ')   // Unicode cross to LaTeX times
       .replace(/\u00F7/g, ' \\div ')     // Unicode division to LaTeX div
       .replace(/([0-9]+)\/([0-9]+)/g, '\\frac{$1}{$2}'); // Simple fractions
   }
   ```

---

## 2.2 Multi-Stage LLM Prompt Engineering & Strict Schemas

To prevent JSON truncation crashes caused by monolithic LLM invocations exceeding max output tokens (8,192 tokens), the transformation pipeline is decomposed into **isolated, chunked, parallel stages**:

```
┌────────────────────────────────────────────────────────────────────────┐
│               CHUNKED AI TRANSFORMATION PIPELINE STAGES                │
├────────────────────────────────────────────────────────────────────────┤
│ Stage 1: Chapter Skeleton & Concept Hierarchy (Gemini 1.5 Pro / GPT-4o)│
│          └── Outputs title, overview, node list metadata & vocabulary  │
│                                │                                       │
│                                ▼                                       │
│ Stage 2: Chunked Per-Node Content Generation (Parallel LLM Calls)      │
│          └── Executed once per node: generates 5-7 pedagogical steps   │
│                                │                                       │
│                                ▼                                       │
│ Stage 3a: Concept Quizzes & True/False (Gemini 1.5 Flash / 4o-mini)    │
│          └── Generates MCQs with misconception feedback & T/F items    │
│                                │                                       │
│ Stage 3b: Solve Scenarios & Tiered Worksheets                          │
│          └── Generates step-by-step problem sets (Basic, Std, HOTS)    │
│                                │                                       │
│ Stage 3c: Gamified Rapid Fire & Memory Match                           │
│          └── Generates 10+ speed quiz items & 6-8 matching pairs       │
│                                │                                       │
│                                ▼                                       │
│ Stage 4: Responsive Canvas 2D Scripts (Code Specialization Tier)       │
│          └── Generates normalized 0-1000 coordinate visual models      │
│                                │                                       │
│                                ▼                                       │
│ Stage 5: LLE Multilingual Dictionary & Transliterations (Indic Tier)   │
│          └── Generates Devanagari, Romanized phonetics & definitions   │
└────────────────────────────────────────────────────────────────────────┘
```

### Hyperparameter & Model Tiering Strategy

| Pipeline Stage | Recommended Model Tier | Provider Options | Temperature | Max Tokens | Output Mode |
|---|---|---|---|---|---|
| **Stage 1: Skeleton Analysis** | Frontier Reasoning Tier | Gemini 1.5 Pro / GPT-4o | **0.20** | 4,096 | Structured JSON |
| **Stage 2: Chunked Per-Node Gen** | High-Capability Pedagogical Tier | Gemini 1.5 Pro / GPT-4o | **0.30** | 4,096 | Structured JSON |
| **Stage 3a: Quizzes & True/False** | Fast High-Accuracy Tier | Gemini 1.5 Flash / GPT-4o-mini | **0.30** | 4,096 | Structured JSON |
| **Stage 3b: Solve & Worksheets** | Fast High-Accuracy Tier | Gemini 1.5 Flash / GPT-4o-mini | **0.30** | 4,096 | Structured JSON |
| **Stage 3c: Rapid Fire & Match** | Fast High-Accuracy Tier | Gemini 1.5 Flash / GPT-4o-mini | **0.30** | 4,096 | Structured JSON |
| **Stage 4: Canvas Scripting** | Code Specialization Tier | Gemini 1.5 Pro / Claude 3.5 Sonnet | **0.20** | 4,096 | Clean ES5 JS in JSON |
| **Stage 5: LLE Translations** | Multilingual Indic Tier | Gemini 1.5 Flash / Sarvam AI | **0.15** | 4,096 | Structured JSON |

---

### Stage 1: Skeleton Analysis Prompt & Strict JSON Schema

#### System Prompt Template:
```text
You are an expert curriculum architect and pedagogical specialist for the Aasha learning platform.
Your task is to analyze raw textbook passage text and deconstruct it into a clean, hierarchical educational chapter skeleton.

Target Constraints:
1. Target audience is school children in Class {{GRADE}} (ages {{AGE_MIN}}-{{AGE_MAX}}), Board: {{BOARD}}.
2. Identify between 3 and 8 core concepts/nodes. Concepts must follow a logical learning progression from foundational intuition to advanced application.
3. Extract 15 to 40 academic keywords that require bilingual English-to-Hindi Language Learning Engine (LLE) translation.
4. Output MUST be valid, strictly parseable JSON adhering to the provided schema. Do NOT include markdown code fences or extraneous commentary.
```

#### Strict JSON Schema:
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "title": { "type": "string" },
    "titleHindi": { "type": "string" },
    "slug": { "type": "string", "pattern": "^[a-z0-9-]+$" },
    "subject": { "type": "string", "enum": ["mathematics", "science", "english", "hindi", "social_studies", "general_knowledge", "computer_science"] },
    "grade": { "type": "integer", "minimum": 1, "maximum": 10 },
    "board": { "type": "string", "enum": ["cbse", "icse", "state", "ib", "cambridge", "nios"] },
    "overview": { "type": "string" },
    "nodes": {
      "type": "array",
      "minItems": 3,
      "maxItems": 8,
      "items": {
        "type": "object",
        "properties": {
          "id": { "type": "string", "pattern": "^node_[0-9]+$" },
          "title": { "type": "string" },
          "titleHindi": { "type": "string" },
          "icon": { "type": "string" },
          "color": { "type": "string", "pattern": "^#[0-9a-fA-F]{6}$" },
          "difficulty": { "type": "string", "enum": ["easy", "medium", "hard"] },
          "keyConcepts": { "type": "array", "items": { "type": "string" } },
          "summary": { "type": "string" }
        },
        "required": ["id", "title", "titleHindi", "icon", "color", "difficulty", "keyConcepts", "summary"]
      }
    },
    "vocabulary": {
      "type": "array",
      "minItems": 15,
      "items": { "type": "string" }
    }
  },
  "required": ["title", "titleHindi", "slug", "subject", "grade", "board", "overview", "nodes", "vocabulary"]
}
```

---

### Stage 2: Chunked Per-Node Content Generation Prompt & Strict JSON Schema

Each node is generated in an isolated LLM call (`generateNodeContent(nodeSkeleton, chapterOverview)`), avoiding token overflows.

#### System Prompt Template:
```text
You are a master child educator for Aasha. You transform textbook concepts into interactive, step-by-step offline learning experiences.

Task:
Generate the pedagogical steps for ONE specific node:
Node ID: {{NODE_ID}}
Node Title: {{NODE_TITLE}}
Chapter Overview: {{CHAPTER_OVERVIEW}}
Target Grade: Class {{GRADE}}

Pedagogical Step Sequence (5 to 7 steps):
1. "intro": Relatable storytelling hook or real-world connection.
2. "text": Core concept explanation. Wrap academic terms in `<span class="wd" data-word="...">word</span>` and connectives in `<span class="cn">connective</span>`.
3. "canvas_visual": Interactive diagram instruction (parameters or procedural script).
4. "we": Worked example reference key (e.g. "we_node_1"). Provide the full worked example definition in the workedExamples map.
5. "fillblank" or "solve": Immediate formative check for understanding.
6. "progress": Milestone celebration step.

Strict Property Names:
- For step type worked example, use `t: "we"` and string key `we: "we_node_X"`.
- Output MUST strictly adhere to the JSON schema.
```

#### Strict JSON Schema:
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "nodeId": { "type": "string" },
    "steps": {
      "type": "array",
      "minItems": 5,
      "maxItems": 8,
      "items": {
        "type": "object",
        "properties": {
          "t": { 
            "type": "string", 
            "enum": ["intro", "text", "canvas_visual", "ix_drag", "ix_slider", "we", "solve", "quiz", "truefalse", "fillblank", "progress"] 
          },
          "title": { "type": "string" },
          "text": { "type": "string" },
          "caption": { "type": "string" },
          "icon": { "type": "string" },
          "visualScript": { "type": "string" },
          "draw": {
            "type": "object",
            "properties": {
              "type": { "type": "string" },
              "params": { "type": "object" }
            }
          },
          "we": { "type": "string" },
          "sl": { "type": "string" },
          "q": {
            "type": "object",
            "properties": {
              "sentence": { "type": "string" },
              "answer": { "type": "string" },
              "acceptableAnswers": { "type": "array", "items": { "type": "string" } },
              "explanation": { "type": "string" }
            },
            "required": ["sentence", "answer"]
          }
        },
        "required": ["t"]
      }
    },
    "workedExamples": {
      "type": "object",
      "additionalProperties": {
        "type": "object",
        "properties": {
          "title": { "type": "string" },
          "problem": { "type": "string" },
          "steps": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "label": { "type": "string" },
                "content": { "type": "string" },
                "color": { "type": "string", "enum": ["blue", "green", "amber"] },
                "final": { "type": "boolean" }
              },
              "required": ["label", "content"]
            }
          }
        },
        "required": ["title", "problem", "steps"]
      }
    }
  },
  "required": ["nodeId", "steps"]
}
```

---

### Stage 3a: Concept Quizzes & True/False Prompt & Strict JSON Schema

#### System Prompt Template:
```text
You are an expert assessment designer for Aasha. Generate concept-aligned Multiple Choice Questions (MCQs) and True/False questions directly from the chapter content.

Assessment Rules:
1. Zero Hallucinations: All questions must derive strictly from the provided text.
2. MCQ Structure: Exactly 4 options in `opts`. Exactly 1 option with `c: true`. Exactly 3 options with `c: false`.
3. Misconception Feedback: Every option MUST contain a `feedback` field explaining why it is correct or diagnosing the specific mistake.
4. True/False Structure: Use `statement`, boolean `answer`, and comprehensive `explanation`.
5. Bloom's Tiers: Distribute questions across Recall, Conceptual, Application, and HOTS.
```

#### Strict JSON Schema:
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "quizzes": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "nodeId": { "type": "string" },
          "bloomTier": { "type": "string", "enum": ["recall", "conceptual", "application", "hots"] },
          "difficulty": { "type": "string", "enum": ["easy", "medium", "hard"] },
          "prompt": { "type": "string" },
          "opts": {
            "type": "array",
            "minItems": 4,
            "maxItems": 4,
            "items": {
              "type": "object",
              "properties": {
                "text": { "type": "string" },
                "c": { "type": "boolean" },
                "feedback": { "type": "string" }
              },
              "required": ["text", "c", "feedback"]
            }
          }
        },
        "required": ["id", "nodeId", "prompt", "opts", "bloomTier"]
      }
    },
    "trueFalse": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "nodeId": { "type": "string" },
          "statement": { "type": "string" },
          "answer": { "type": "boolean" },
          "explanation": { "type": "string" }
        },
        "required": ["id", "nodeId", "statement", "answer", "explanation"]
      }
    }
  },
  "required": ["quizzes", "trueFalse"]
}
```

---

### Stage 3b: Solve Scenarios & Tiered Worksheets Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "solveScenarios": {
      "type": "object",
      "additionalProperties": {
        "type": "object",
        "properties": {
          "title": { "type": "string" },
          "prompt": { "type": "string" },
          "steps": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "instruction": { "type": "string" },
                "expectedAnswer": { "type": "string" },
                "hint": { "type": "string" }
              },
              "required": ["instruction", "expectedAnswer"]
            }
          }
        },
        "required": ["title", "prompt", "steps"]
      }
    },
    "worksheets": {
      "type": "object",
      "properties": {
        "basic": { "type": "array", "items": { "$ref": "#/definitions/WorksheetItem" } },
        "standard": { "type": "array", "items": { "$ref": "#/definitions/WorksheetItem" } },
        "hots": { "type": "array", "items": { "$ref": "#/definitions/WorksheetItem" } },
        "final_mixed": { "type": "array", "items": { "$ref": "#/definitions/WorksheetItem" } }
      },
      "required": ["basic", "standard", "hots", "final_mixed"]
    }
  },
  "required": ["solveScenarios", "worksheets"],
  "definitions": {
    "WorksheetItem": {
      "type": "object",
      "properties": {
        "id": { "type": "string" },
        "question": { "type": "string" },
        "answer": { "type": "string" },
        "hints": { "type": "array", "items": { "type": "string" } },
        "solution": { "type": "string" }
      },
      "required": ["id", "question", "answer", "solution"]
    }
  }
}
```

---

### Stage 3c: Gamified Activities (Rapid Fire & Memory Match) Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "rapidFire": {
      "type": "array",
      "minItems": 10,
      "items": {
        "type": "object",
        "properties": {
          "q": { "type": "string" },
          "opts": { "type": "array", "minItems": 2, "maxItems": 4, "items": { "type": "string" } },
          "ans": { "type": "string" },
          "timeLimitSec": { "type": "integer", "default": 15 }
        },
        "required": ["q", "opts", "ans"]
      }
    },
    "memoryMatchPairs": {
      "type": "array",
      "minItems": 6,
      "maxItems": 8,
      "items": {
        "type": "object",
        "properties": {
          "term": { "type": "string" },
          "match": { "type": "string" }
        },
        "required": ["term", "match"]
      }
    }
  },
  "required": ["rapidFire", "memoryMatchPairs"]
}
```

---

## 2.3 Dual-Mode Visuals, Normalized Viewport & Canvas Security

To maintain zero external network dependencies and prevent responsive distortion across devices:

### 1. Normalized Coordinate Space (0–1000 x 0–1000)
All canvas visuals must be drawn in a virtual **1000 $\times$ 1000 unit normalized box**. The chapter runtime automatically scales this virtual coordinate space to the device's physical viewport and Retina pixel ratio:

```javascript
// Universal High-DPI Responsive Canvas Wrapper
function setupResponsiveCanvas(canvas, drawCallback) {
  var dpr = window.devicePixelRatio || 1;
  var rect = canvas.getBoundingClientRect();
  
  // Set physical display buffer size
  canvas.width = rect.width * dpr;
  canvas.height = rect.height * dpr;
  
  var ctx = canvas.getContext('2d');
  ctx.save();
  
  // Scale virtual 1000x1000 coordinate space to physical pixel resolution
  ctx.scale(canvas.width / 1000, canvas.height / 1000);
  
  // Execute the drawing routine
  drawCallback(ctx);
  
  ctx.restore();
}
```

### 2. Generative Canvas Script Example (Quadrilateral Angle Sum):
```javascript
// Sandboxed Procedural Canvas Drawing Function (0-1000 Normalized Coordinates)
function drawQuadrilateralNormalized(ctx) {
  ctx.clearRect(0, 0, 1000, 1000);
  
  // Draw Polygon ABCD
  ctx.beginPath();
  ctx.moveTo(250, 150);
  ctx.lineTo(850, 200);
  ctx.lineTo(750, 800);
  ctx.lineTo(150, 700);
  ctx.closePath();
  
  ctx.fillStyle = '#eff6ff';
  ctx.fill();
  ctx.lineWidth = 12;
  ctx.strokeStyle = '#3b82f6';
  ctx.stroke();

  // Draw Vertex Dots and Labels
  var vertices = [
    { name: 'A (100°)', x: 250, y: 150, lx: 200, ly: 120 },
    { name: 'B (80°)',  x: 850, y: 200, lx: 870, ly: 190 },
    { name: 'C (110°)', x: 750, y: 800, lx: 770, ly: 840 },
    { name: 'D (70°)',  x: 150, y: 700, lx: 90,  ly: 730 }
  ];

  ctx.fillStyle = '#1e40af';
  ctx.font = 'bold 36px system-ui, sans-serif';
  for (var i = 0; i < vertices.length; i++) {
    var v = vertices[i];
    ctx.beginPath();
    ctx.arc(v.x, v.y, 14, 0, Math.PI * 2);
    ctx.fill();
    ctx.fillText(v.name, v.lx, v.ly);
  }
}
```

### 3. Script Security Sandboxing via Acorn AST Parser
For Mode 2 raw JavaScript canvas code, the `@aasha/test-harness` executes an AST static security check before execution:
- **Forbidden Identifiers & Globals**: `window`, `document.write`, `fetch`, `XMLHttpRequest`, `WebSocket`, `EventSource`, `eval`, `Function`, `localStorage`, `sessionStorage`, `indexedDB`, `navigator.sendBeacon`, `Worker`.
- Scripts violating these constraints fail the test harness with a `SecurityViolation` error and are blocked from compilation.

---

## 2.4 Validation Harness Specification (`@aasha/test-harness`)

The `@aasha/test-harness` is a reusable, automated verification suite that audits every assembled HTML chapter before publication.

```typescript
// packages/test-harness/src/index.ts
import vm from 'vm';
import * as acorn from 'acorn';

export interface HarnessValidationResult {
  valid: boolean;
  errors: string[];
  warnings: string[];
  metrics: {
    fileSizeMb: number;
    nodesCount: number;
    totalStepsCount: number;
    quizCount: number;
    trueFalseCount: number;
    fillBlankCount: number;
    workedExamplesCount: number;
    solveScenariosCount: number;
    vocabularyTermsCount: number;
  };
}

export function validateHtmlChapter(html: string): HarnessValidationResult {
  const errors: string[] = [];
  const warnings: string[] = [];

  // 1. FILE SIZE BUDGET AUDIT (<= 20 MB)
  const sizeBytes = Buffer.byteLength(html, 'utf8');
  const sizeMb = sizeBytes / (1024 * 1024);
  if (sizeMb > 20.0) {
    errors.push(`File size (${sizeMb.toFixed(2)} MB) exceeds maximum 20 MB budget`);
  }

  // 2. ZERO-NETWORK & DYNAMIC LEAK AUDIT
  const staticNetworkRegex = /(href|src|url)\s*=\s*["'](https?:)?\/\/([^"']+)["']/gi;
  let extMatch;
  while ((extMatch = staticNetworkRegex.exec(html)) !== null) {
    const matchedUrl = extMatch[0];
    if (!matchedUrl.includes('data:') && !matchedUrl.includes('localhost')) {
      errors.push(`Static external network reference found: ${matchedUrl}`);
    }
  }

  // 3. EXTRACT AND PARSE SCRIPTS VIA ACORN AST
  const scriptRegex = /<script>([\s\S]*?)<\/script>/gi;
  let scriptContent = '';
  let match;
  while ((match = scriptRegex.exec(html)) !== null) {
    scriptContent += match[1] + '\n';
  }

  try {
    const ast = acorn.parse(scriptContent, { ecmaVersion: 'latest' });
    // AST Walk: Assert no forbidden globals or dynamic imports
    // (Inspection logic recursively checks MemberExpressions and CallExpressions)
  } catch (err: any) {
    errors.push(`Acorn AST Parsing failed: ${err.message}`);
  }

  // 4. SANDBOXED VM EXECUTION WITH NETWORK TRAPS
  const sandbox: any = {
    window: {},
    document: {
      addEventListener: () => {},
      getElementById: () => ({
        getContext: () => ({
          fillRect: () => {}, clearRect: () => {}, beginPath: () => {},
          arc: () => {}, fill: () => {}, stroke: () => {}, fillText: () => {},
          measureText: () => ({ width: 10 }), save: () => {}, restore: () => {},
          scale: () => {}, moveTo: () => {}, lineTo: () => {}, closePath: () => {}
        }),
        addEventListener: () => {}, style: {}, close: () => {},
        getBoundingClientRect: () => ({ width: 400, height: 400 })
      }),
      querySelector: () => null,
      querySelectorAll: () => [],
    },
    fetch: () => { throw new Error('Security Violation: Runtime fetch() detected'); },
    XMLHttpRequest: function() { throw new Error('Security Violation: Runtime XMLHttpRequest detected'); },
    WebSocket: function() { throw new Error('Security Violation: Runtime WebSocket detected'); },
    EventSource: function() { throw new Error('Security Violation: Runtime EventSource detected'); },
    navigator: {
      userAgent: 'node',
      sendBeacon: () => { throw new Error('Security Violation: Runtime sendBeacon detected'); }
    },
    console: { log: () => {}, error: () => {}, warn: () => {} },
    AudioContext: function() {
      return {
        createOscillator: () => ({ connect: () => {}, start: () => {}, stop: () => {}, frequency: { setValueAtTime: () => {} } }),
        createGain: () => ({ connect: () => {}, gain: { setValueAtTime: () => {}, exponentialRampToValueAtTime: () => {} } }),
        destination: {},
        currentTime: 0,
        state: 'running',
        resume: () => {}
      };
    },
    speechSynthesis: { speak: () => {}, cancel: () => {}, getVoices: () => [] },
    SpeechSynthesisUtterance: function(text: string) { this.text = text; this.rate = 0.85; this.lang = 'en-IN'; },
    NODES: undefined,
    WE: undefined,
    SOLVE: undefined,
    WM: undefined,
    CONN: undefined,
  };

  try {
    vm.createContext(sandbox);
    vm.runInContext(scriptContent, sandbox, { timeout: 3000 });
  } catch (err: any) {
    errors.push(`JavaScript sandbox execution failed: ${err.message}`);
  }

  const NODES = sandbox.NODES;
  const WE = sandbox.WE || {};
  const SOLVE = sandbox.SOLVE || {};
  const WM = sandbox.WM || {};

  let totalStepsCount = 0;
  let quizCount = 0;
  let trueFalseCount = 0;
  let fillBlankCount = 0;

  // 5. AST & ALL ASSESSMENT TYPE INTEGRITY CHECKS
  if (!NODES || !Array.isArray(NODES) || NODES.length === 0) {
    errors.push('Global variable "NODES" is missing, invalid, or empty');
  } else {
    NODES.forEach((node: any, nIdx: number) => {
      const nodeId = node.id || `node_${nIdx}`;
      if (!node.title) errors.push(`Node "${nodeId}" is missing a title`);
      if (!Array.isArray(node.steps) || node.steps.length === 0) {
        errors.push(`Node "${nodeId}" has no steps`);
        return;
      }
      totalStepsCount += node.steps.length;

      node.steps.forEach((step: any, sIdx: number) => {
        const stepDesc = `Node "${nodeId}" Step ${sIdx} (${step.t || 'unknown'})`;

        // Check worked example cross-reference
        if (step.t === 'we') {
          if (!step.we || !WE[step.we]) {
            errors.push(`${stepDesc} references worked example key "${step.we}" which does not exist in WE dictionary`);
          }
        }

        // Check solve scenario cross-reference
        if (step.t === 'solve') {
          if (!step.sl || !SOLVE[step.sl]) {
            errors.push(`${stepDesc} references solve scenario key "${step.sl}" which does not exist in SOLVE dictionary`);
          }
        }

        // Check MCQ Quiz
        if (step.t === 'quiz') {
          quizCount++;
          if (!step.q || !Array.isArray(step.q.opts) || step.q.opts.length !== 4) {
            errors.push(`${stepDesc} must have exactly 4 choices in 'q.opts'`);
          } else {
            const correctOpts = step.q.opts.filter((opt: any) => opt.c === true);
            if (correctOpts.length !== 1) {
              errors.push(`${stepDesc} must have EXACTLY ONE correct option (Found: ${correctOpts.length})`);
            }
            // Duplicate option text check
            const optTexts = step.q.opts.map((o: any) => (o.text || '').trim().toLowerCase());
            const uniqueTexts = new Set(optTexts);
            if (uniqueTexts.size !== optTexts.length) {
              errors.push(`${stepDesc} contains duplicate MCQ options: [${optTexts.join(', ')}]`);
            }
          }
        }

        // Check True/False
        if (step.t === 'truefalse') {
          trueFalseCount++;
          if (!step.q || typeof step.q.answer !== 'boolean') {
            errors.push(`${stepDesc} must define a valid boolean 'q.answer'`);
          }
          if (!step.q || !step.q.explanation) {
            errors.push(`${stepDesc} must have non-empty explanation`);
          }
        }

        // Check Fill Blank
        if (step.t === 'fillblank') {
          fillBlankCount++;
          if (!step.q || !step.q.answer || typeof step.q.answer !== 'string') {
            errors.push(`${stepDesc} must define a valid non-empty string 'q.answer'`);
          }
        }

        // Check Vocabulary tags cross-reference in text
        if (step.text && typeof step.text === 'string') {
          const wdRegex = /<span class="wd" data-word="([^"]+)">/g;
          let wdMatch;
          while ((wdMatch = wdRegex.exec(step.text)) !== null) {
            const wordKey = wdMatch[1].toLowerCase();
            if (!WM[wordKey]) {
              warnings.push(`${stepDesc} tags word "${wordKey}" which is missing from WM dictionary`);
            }
          }
        }
      });
    });
  }

  return {
    valid: errors.length === 0,
    errors,
    warnings,
    metrics: {
      fileSizeMb: parseFloat(sizeMb.toFixed(3)),
      nodesCount: NODES ? NODES.length : 0,
      totalStepsCount,
      quizCount,
      trueFalseCount,
      fillBlankCount,
      workedExamplesCount: Object.keys(WE).length,
      solveScenariosCount: Object.keys(SOLVE).length,
      vocabularyTermsCount: Object.keys(WM).length,
    }
  };
}
```

---

# Section 3: Enhanced Language Learning Engine (LLE) Specification (R3)

## 3.1 Drop-in Engine Architecture (`Aasha_LLE_Enhanced.js`, ~60 KB)

The enhanced LLE engine is structured as a self-contained IIFE of approximately 60 KB unminified (~18 KB gzipped).

```
┌────────────────────────────────────────────────────────────────────────┐
│               Aasha_LLE_Enhanced.js Architecture (~60 KB)              │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Word Map (WM): 750+ rich entries + backward-compatible string keys  │
│ 2. Connectives Map (CONN): 60+ transition & logical conjunctions       │
│ 3. Text Processor (rt): High-speed regex tokenizer with punctuation fix│
│ 4. Audio Engine (LLE_AUDIO): Web Speech API + Web Audio tone fallback  │
│ 5. SVG Catalog (LLE_SVG): 55+ pure inline SVG path definitions         │
│ 6. Word Dialog (LLE_DIALOG): HTML5 <dialog> modal with audio & visuals │
│ 7. DOM Processor (applyLLE): Traverses content nodes and binds LLE     │
│ 8. Global Event Delegator: Listens for clicks on .wd and .cn spans     │
│ 9. Injected Stylesheet: Injects scoped CSS design tokens & animations  │
│ 10. Auto-Initialization: Executes on DOMContentLoaded or immediate     │
│ 11. Universal API Export: window.AashaLLE, window.LLE, window.rt, etc. │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 3.2 100% Backward Compatibility Mapping

The LLE preserves 100% backward compatibility with existing chapter code while adding rich data support:

```javascript
// Legacy string format (fully supported):
WM["polygon"] = "बहुभुज";

// Enhanced rich metadata object format:
WM["polygon"] = {
  h: "बहुभुज",                                   // Hindi translation (Devanagari)
  t: "bahubhuj",                                 // Transliteration (Romanized Hindi)
  p: "n",                                        // Part of speech (n, v, adj, adv, prep, conj)
  g: 4,                                          // Grade introduction level (Class 1-10)
  s: "math",                                     // Subject tag (math, science, general)
  d: "A closed 2D shape made of straight lines", // Child-friendly English definition
  svg: "M5 4 L15 4 L18 10 L15 16 L5 16 L2 10 Z",// Inline SVG path data
  ex: "A hexagon is a polygon with 6 sides.",    // Contextual example sentence
  ex_hi: "षट्कोण 6 भुजाओं वाला एक बहुभुज है।"     // Hindi example sentence
};

// Global Interface Exports
window.AashaLLE = {
  version: "2.0.0",
  WM: WM,
  CONN: CONN,
  audio: LLE_AUDIO,
  svg: LLE_SVG,
  dialog: LLE_DIALOG,
  rt: rt,
  applyLLE: applyLLE,
  showWord: showWord,
  speakWord: function(w) { LLE_AUDIO.speakWord(w); },
  speakHindi: function(h) { LLE_AUDIO.speakHindi(h); },
  getIllustration: function(w) { return LLE_SVG.render(w); }
};

window.LLE = window.AashaLLE;
window.rt = rt;
window.applyLLE = applyLLE;
window.showWord = showWord;
window.WM = WM;
window.CONN = CONN;
```

---

## 3.3 Audio Engine: Web Speech API & Oscillator Fallback

```javascript
var LLE_AUDIO = {
  ctx: null,
  englishVoice: null,
  hindiVoice: null,
  initialized: false,

  init: function() {
    if (this.initialized) return;
    this.detectVoices();
    if (typeof window.speechSynthesis !== 'undefined') {
      window.speechSynthesis.onvoiceschanged = this.detectVoices.bind(this);
    }
    this.initialized = true;
  },

  detectVoices: function() {
    if (typeof window.speechSynthesis === 'undefined') return;
    var voices = window.speechSynthesis.getVoices();
    for (var i = 0; i < voices.length; i++) {
      var v = voices[i];
      var lang = (v.lang || '').toLowerCase();
      if (lang.indexOf('en-in') === 0) {
        this.englishVoice = v;
      } else if (lang.indexOf('en') === 0 && !this.englishVoice) {
        this.englishVoice = v;
      }
      if (lang.indexOf('hi') === 0) {
        this.hindiVoice = v;
      }
    }
  },

  speakWord: function(word) {
    if (!word) return;
    if (typeof window.speechSynthesis !== 'undefined') {
      window.speechSynthesis.cancel();
      var u = new SpeechSynthesisUtterance(word);
      u.rate = 0.85;  // Optimized pacing for children
      u.pitch = 1.0;
      u.volume = 0.8;
      if (this.englishVoice) u.voice = this.englishVoice;
      else u.lang = 'en-IN';
      window.speechSynthesis.speak(u);
    } else {
      this.toneFallback(word);
    }
  },

  speakHindi: function(hindiText) {
    if (!hindiText) return;
    if (typeof window.speechSynthesis !== 'undefined') {
      window.speechSynthesis.cancel();
      var u = new SpeechSynthesisUtterance(hindiText);
      u.rate = 0.85;
      u.pitch = 1.0;
      u.volume = 0.8;
      if (this.hindiVoice) u.voice = this.hindiVoice;
      else u.lang = 'hi-IN';
      window.speechSynthesis.speak(u);
    } else {
      this.toneFallback(hindiText);
    }
  },

  toneFallback: function(text) {
    var count = Math.max(1, Math.min(5, Math.ceil(text.length / 3)));
    var AudioCtx = window.AudioContext || window.webkitAudioContext;
    if (!AudioCtx) return;
    if (!this.ctx) this.ctx = new AudioCtx();
    if (this.ctx.state === 'suspended') this.ctx.resume();

    var baseFreq = 440;
    var now = this.ctx.currentTime;
    for (var i = 0; i < count; i++) {
      var osc = this.ctx.createOscillator();
      var gain = this.ctx.createGain();
      osc.type = 'sine';
      osc.frequency.setValueAtTime(baseFreq + (i * 45), now + (i * 0.12));
      gain.gain.setValueAtTime(0.12, now + (i * 0.12));
      gain.gain.exponentialRampToValueAtTime(0.001, now + (i * 0.12) + 0.1);
      osc.connect(gain);
      gain.connect(this.ctx.destination);
      osc.start(now + (i * 0.12));
      osc.stop(now + (i * 0.12) + 0.1);
    }
  }
};
```

---

## 3.4 Visual SVG Illustration Catalog (55 Math & Science Terms)

All illustrations are rendered dynamically in normalized `viewBox="0 0 20 20"` coordinate space:

| # | Term | Concept Illustrated | SVG Path Data (`d` attribute) |
|---|------|---------------------|-------------------------------|
| 1 | `polygon` | Hexagon shape | `M5 4 L15 4 L18 10 L15 16 L5 16 L2 10 Z` |
| 2 | `circle` | Circle boundary | `M10 2 A8 8 0 1 0 10 18 A8 8 0 1 0 10 2 Z` |
| 3 | `triangle` | Equilateral triangle | `M10 3 L18 17 L2 17 Z` |
| 4 | `square` | Regular square | `M4 4 L16 4 L16 16 L4 16 Z` |
| 5 | `quadrilateral` | Generic quadrilateral | `M3 5 L17 4 L15 16 L4 15 Z` |
| 6 | `angle` | Acute angle with vertex arc | `M4 16 L4 8 M4 16 L14 16 M4 12 A4 4 0 0 0 8 16` |
| 7 | `diagonal` | Rectangle with diagonal line | `M3 5 L17 5 L17 15 L3 15 Z M3 5 L17 15` |
| 8 | `parallel` | Two parallel lines with arrows | `M3 7 L17 7 M3 13 L17 13 M10 5 L12 7 L10 9 M10 11 L12 13 L10 15` |
| 9 | `perpendicular` | Perpendicular lines with square marker | `M10 2 L10 18 M2 14 L18 14 M10 11 L13 11 L13 14` |
| 10 | `radius` | Circle with center-to-edge radius | `M10 2 A8 8 0 1 0 10 18 A8 8 0 1 0 10 2 Z M10 10 L18 10 M10 10 A1 1 0 1 1 9.9 10` |
| 11 | `diameter` | Circle with full diameter line | `M10 2 A8 8 0 1 0 10 18 A8 8 0 1 0 10 2 Z M2 10 L18 10` |
| 12 | `fraction` | Divided bar representation | `M3 6 L17 6 L17 14 L3 14 Z M10 6 L10 14 M3 10 L17 10` |
| 13 | `half` | Shaded half rectangle | `M3 5 L17 5 L17 15 L3 15 Z M10 5 L10 15 M3 5 L10 15` |
| 14 | `add` | Plus arithmetic symbol | `M10 4 L10 16 M4 10 L16 10` |
| 15 | `divide` | Division arithmetic symbol | `M4 10 L16 10 M10 5 A1.5 1.5 0 1 1 9.9 5 M10 15 A1.5 1.5 0 1 1 9.9 15` |
| 16 | `multiply` | Multiplication cross symbol | `M5 5 L15 15 M15 5 L5 15` |
| 17 | `equal` | Equal relation symbol | `M4 8 L16 8 M4 12 L16 12` |
| 18 | `number` | Counting hash tally symbol | `M7 3 L7 17 M13 3 L13 17 M3 7 L17 7 M3 13 L17 13` |
| 19 | `equation` | Math expression balance | `M2 14 L8 14 M12 14 L18 14 M5 14 L5 8 M15 14 L15 8 M3 8 L7 8 M13 8 L17 8 M8 10 L12 10 M8 12 L12 12` |
| 20 | `rhombus` | Diamond with equal sides | `M10 2 L18 10 L10 18 L2 10 Z` |
| 21 | `trapezium` | Trapezoid with 1 parallel pair | `M6 5 L14 5 L18 15 L2 15 Z` |
| 22 | `kite` | Kite with adjacent equal pairs | `M10 2 L17 8 L10 18 L3 8 Z` |
| 23 | `hexagon` | Regular 6-sided polygon | `M5 3 L15 3 L19 10 L15 17 L5 17 L1 10 Z` |
| 24 | `vertex` | Highlighted angle intersection vertex | `M4 16 L10 4 L16 16 M10 4 A2 2 0 1 1 9.9 4` |
| 25 | `concave` | Inward-dented polygon | `M3 3 L17 3 L17 17 L10 11 L3 17 Z` |
| 26 | `convex` | Outward polygon boundary | `M4 4 L16 4 L18 12 L14 17 L6 17 L2 12 Z` |
| 27 | `congruent` | Congruence equivalence symbol | `M4 11 L16 11 M4 14 L16 14 M4 7 C7 5 9 9 12 7 C14 5 16 7 16 7` |
| 28 | `closed` | Closed continuous path | `M4 4 L16 4 L16 16 L4 16 Z` |
| 29 | `curved` | Smooth wave curve | `M2 14 C6 4 14 16 18 6` |
| 30 | `line` | Straight line with end arrows | `M3 10 L17 10 M5 8 L3 10 L5 12 M15 8 L17 10 L15 12` |
| 31 | `point` | Solid localized point dot | `M10 8 A2 2 0 1 1 9.9 8` |
| 32 | `center` | Circle with center focal dot | `M10 2 A8 8 0 1 0 10 18 A8 8 0 1 0 10 2 Z M10 9.5 A0.5 0.5 0 1 1 9.9 9.5` |
| 33 | `degree` | Angle arc with degree symbol | `M4 16 L16 16 M4 16 L12 6 M7 16 A4 4 0 0 0 8.5 12.5 M13 4 A1.5 1.5 0 1 1 12.9 4` |
| 34 | `sum` | Sigma summation symbol | `M16 4 L5 4 L11 10 L5 16 L16 16` |
| 35 | `formula` | Function symbol $f(x)$ | `M12 4 C10 4 8 5 8 8 L8 16 M5 9 L11 9` |
| 36 | `area` | Grid-shaded enclosed region | `M3 3 L17 3 L17 17 L3 17 Z M3 8 L17 8 M3 13 L17 13 M8 3 L8 17 M13 3 L13 17` |
| 37 | `perimeter` | Highlighted outer boundary stroke | `M3 3 L17 3 L17 17 L3 17 Z M3 3 L17 3` |
| 38 | `percent` | Percentage symbol | `M6 4 A2 2 0 1 1 5.9 4 M14 16 A2 2 0 1 1 13.9 16 M16 4 L4 16` |
| 39 | `ratio` | Colon comparison symbol | `M10 6 A1.5 1.5 0 1 1 9.9 6 M10 14 A1.5 1.5 0 1 1 9.9 14` |
| 40 | `above` | Upward directional vector | `M10 17 L10 3 M5 8 L10 3 L15 8` |
| 41 | `below` | Downward directional vector | `M10 3 L10 17 M5 12 L10 17 L15 12` |
| 42 | `one` | Numeric numeral 1 | `M7 6 L10 3 L10 17 M7 17 L13 17` |
| 43 | `two` | Numeric numeral 2 | `M5 6 C5 3 15 3 15 7 C15 11 5 17 5 17 L15 17` |
| 44 | `three` | Numeric numeral 3 | `M5 4 L15 4 L10 9 C13 9 15 11 15 14 C15 17 5 17 5 17` |
| 45 | `four` | Numeric numeral 4 | `M12 17 L12 3 L4 12 L16 12` |
| 46 | `five` | Numeric numeral 5 | `M14 4 L6 4 L6 10 C8 9 14 9 14 13 C14 17 6 17 6 17` |
| 47 | `straight` | Unbroken horizontal line | `M2 10 L18 10` |
| 48 | `shape` | Geometric polygon composite | `M4 4 L16 4 L16 16 L4 16 Z M4 4 L16 16` |
| 49 | `edge` | Highlighted polyhedral segment | `M4 10 L16 10 M4 8 L4 12 M16 8 L16 12` |
| 50 | `side` | Polygon side segment | `M3 16 L10 3 L17 16 Z M3 16 L10 3` |
| 51 | `atom` | Atomic nucleus with orbiting electron rings | `M10 10 A2 2 0 1 1 9.9 10 M2 10 C2 5 18 5 18 10 C18 15 2 15 2 10 M10 2 C15 2 15 18 10 18 C5 18 5 2 10 2` |
| 52 | `molecule` | Bonded chemical atoms | `M6 7 A2.5 2.5 0 1 1 5.9 7 M14 7 A2.5 2.5 0 1 1 13.9 7 M10 14 A2.5 2.5 0 1 1 9.9 14 M7.5 8.5 L9 12 M12.5 8.5 L11 12 M8.5 7 L11.5 7` |
| 53 | `cell` | Biological cell with nucleus | `M10 2 C16 2 18 7 18 10 C18 15 15 18 10 18 C4 18 2 14 2 10 C2 5 5 2 10 2 Z M10 8 A2.5 2.5 0 1 1 9.9 8` |
| 54 | `photosynthesis` | Plant leaf absorbing sunlight and water | `M10 18 C10 18 3 13 3 8 C3 4 7 2 10 2 C13 2 17 4 17 8 C17 13 10 18 10 18 Z M10 2 L10 18 M10 7 L14 5 M10 11 L15 9 M10 9 L6 7 M10 13 L5 11` |
| 55 | `refraction` | Light ray bending across interface | `M2 10 L18 10 M4 3 L10 10 L15 17 M10 6 L10 14` |

---

## 3.5 Modernized Word Dialog UI/UX Specification

```
┌───────────────────────────────────────────────┐
│                 <dialog>                      │
│  ┌─────────────────────────────────────────┐  │
│  │          [SVG 80x80px Visual]           │  │
│  │               (centered)                │  │
│  └─────────────────────────────────────────┘  │
│                                               │
│       polygon        [ NOUN ] (POS Badge)     │
│       बहुभुज                                  │
│       (bahubhuj)     (Romanized Italic)       │
│                                               │
│  A closed flat shape made with straight       │
│  sides.                                       │
│                                               │
│  ┌─────────────────────────────────────────┐  │
│  │ Example:                                │  │
│  │ "A square and a triangle are polygons." │  │
│  └─────────────────────────────────────────┘  │
│                                               │
│  [ 🔊 Hear English ]    [ 🔊 Hear Hindi ]     │
│  (min 48px height)      (min 48px height)     │
│                                               │
│                [ Got It ✓ ]                   │
│             (min 48px height)                 │
└───────────────────────────────────────────────┘
```

---

# Section 4: Phased Beta Build Plan (R4)

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           PHASED BETA BUILD PLAN                         │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Phase 1: Foundation & Offline Learning Engine (Days 1–3)                │
│  ├── Aasha_LLE_Enhanced.js drop-in engine (Audio, SVG, Translit)         │
│  ├── Single-File HTML Chapter Template & Engine (<= 20 MB Budget)        │
│  └── Offline Gamification & Mastery Tracking (XP, Coins, Badges, TEAS)   │
│                                │                                         │
│                                ▼                                         │
│  Phase 2: AI Transformation Core (Days 4–7)                              │
│  ├── Pre-OCR Normalization & Cloud OCR Adapter (PDF up to 50MB)          │
│  ├── Chunked Per-Node LLM Pipeline (Skeleton -> Node Chunks -> Canvas)   │
│  └── Partitioned Assessment Generator (Quizzes, True/False, Solve, RF)   │
│                                │                                         │
│                                ▼                                         │
│  Phase 3: Tooling & Monorepo Packages (Days 8–10)                        │
│  ├── @aasha/shared-types (Complete DB Entities & AST Canonical Types)    │
│  ├── @aasha/chapter-builder (CLI & Library to compile offline HTML)      │
│  └── @aasha/test-harness (Acorn AST + VM Sandbox Validation Suite)       │
│                                │                                         │
│                                ▼                                         │
│  Phase 4: Fastify API & Teacher Raw JSON Dashboard (Days 11–13)          │
│  ├── Fastify REST API with Standardized Error Envelope & BullMQ Jobs     │
│  ├── Docker Compose Infra (Postgres 16, Redis 7, MinIO S3)               │
│  └── Teacher Dashboard with Raw JSON Text-Area Customizer & Preview      │
│                                │                                         │
│                                ▼                                         │
│  Phase 5: Beta Pilot Verification (Days 14–15)                           │
│  ├── End-to-End Test of 5+ Multi-Board Chapters (Grades 1-10)            │
│  └── 100% Pass Rate on Test Harness (Zero CDN, Answer Integrity)        │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

## 4.1 Strict Beta Scope Isolation

To ensure laser focus on the core student learning and mastery experience, strict boundaries are enforced during the Beta phase:

| In-Scope for Beta (100% Focus) | Out-of-Scope for Beta (Mocked/Deferred) |
|---|---|
| Single-file offline HTML chapters ($\le 20$ MB) | Complex enterprise multi-tenant billing & payments |
| `Aasha_LLE_Enhanced.js` drop-in engine (~60 KB) | Heavy CMS drag-and-drop visual page builders |
| Web Speech API audio + Web Audio tone fallback | Machine learning predictive retention scoring |
| 55+ inline SVG vocabulary catalog | Third-party LMS integrations (LTI, Canvas, Moodle) |
| Pre-OCR Normalization & Cloud OCR Pipeline | Peer video feeds, multiplayer voice, or social lobbies |
| Chunked Per-Node LLM generation | Complex RBAC hierarchy (mocked as Teacher/Admin) |
| Partitioned 100% dynamic assessment generation | SMS gateway integration (OTP mocked in dev/beta) |
| Raw JSON text-area editor in Teacher Dashboard | Video transcoding or heavy remote media management |
| Local gamification (XP, Coins, Badges, Profiles) | Complex automated report-card emailing engines |

---

# Section 5: Verification & Traceability Matrix

| Requirement / Acceptance Criteria | Implementation Specification Section | Target File / Module | Verification Method |
|---|---|---|---|
| **R1.1 Monorepo Architecture** | Section 1.1 | `package.json`, `server/`, `dashboard/`, `packages/` | NPM workspace link & build validation |
| **R1.2 Class 1-10 & All Boards Scope** | Section 1.2 | `packages/shared-types/src/index.ts` | Grade enum (1-10) & BoardCode enum validation |
| **R1.3 $\le 20$ MB Single-File Budget** | Section 1.3 | Compiled chapters in `chapters/` | `@aasha/test-harness` byte size audit |
| **R1.4 PostgreSQL DDL & TS Schemas** | Section 1.4 | `server/src/db/migrations/`, `packages/shared-types` | Migration execution & TS compiler check; soft-delete slug index, versioned prompt template unique constraints |
| **R1.5 Backend REST API Contracts & Errors** | Section 1.5 | `server/src/routes/` | Fastify endpoint integration tests with Standardized Error Response Envelope |
| **R1.6 Teacher Dashboard Raw Editor** | Section 1.6 | `dashboard/src/components/RawJsonEditor.tsx` | Schema-validated text-area AST editing & preview |
| **R2.1 Pre-OCR & Cloud OCR Strategy ($\le 50$MB)** | Section 2.1 | `server/src/services/ocr.ts` | Sharp/OpenCV deskew, Sauvola thresholding, NFC Devanagari normalization & Math Sanitizer |
| **R2.2 Chunked Multi-Stage LLM Prompts** | Section 2.2 | `server/src/services/llm.ts` | Strict JSON Schema validation on chunked per-node LLM calls and partitioned 3a/3b/3c assessments |
| **R2.3 Responsive Canvas & Script Security** | Section 2.3 | Chapter template & canvas renderers | Normalized 0-1000 High-DPI scaling & Acorn AST static security validation |
| **R2.4 `@aasha/test-harness` Checks** | Section 2.4 | `packages/test-harness/src/index.ts` | Zero-CDN, dynamic network traps (`fetch`, `ws`), all assessment types, and DOM tests |
| **R3.1 `Aasha_LLE_Enhanced.js` (~60 KB)** | Section 3.1 | `chapters/Aasha_LLE_Enhanced.js` | Drop-in execution test on existing chapters |
| **R3.2 100% Backward Compatibility** | Section 3.2 | LLE Core subsystem (`rt`, `applyLLE`, `showWord`) | Legacy chapter backward compatibility assertions |
| **R3.3 Web Speech rate 0.85 & Oscillator** | Section 3.3 | `LLE_AUDIO` subsystem | Web Speech invocation & Web Audio tone fallback |
| **R3.4 50+ Inline SVG Catalog** | Section 3.4 | `LLE_SVG` subsystem (55 terms defined) | SVG path rendering verification in DOM |
| **R3.5 Modernized UI `<dialog>` Modal** | Section 3.5 | `LLE_DIALOG` subsystem & CSS | Accessibility tap targets ($\ge 44/48$px) & WCAG AA |
| **R4.1 Chronologically Sequenced Plan** | Section 4.0 | 5-Phase Plan (Phase 1 to Phase 5) | Milestone execution tracking |
| **R4.2 Strict Beta Scope Isolation** | Section 4.1 | Scope Guardrails Table | Scope audit preventing backend distractions |

---

*Authoritative Master Specification remediated and verified. Ready for phase-by-phase implementation.*
