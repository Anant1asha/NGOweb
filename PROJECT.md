# Project: Aasha Child-Centric Learning Platform Architecture & Implementation Plan

## Architecture
- **Offline HTML Chapters (`chapters/`, single self-contained HTML <= 20 MB)**: Client-side learning & mastery engine (Layer 1 Understanding, Layer 2 Practice & Assessment, Layer 3 Gamification & LLE). Zero network dependency at runtime.
- **Enhanced LLE (`Aasha_LLE_Enhanced.js`, ~60 KB)**: Drop-in backward-compatible LLE engine (`rt()`, `applyLLE()`, `showWord()`, `WM`, `CONN`), native Web Speech API (rate 0.85) + Web Audio oscillator fallback, 55+ inline SVGs, Romanized Hindi transliteration.
- **AI Transformation Pipeline (`server/src/pipeline/`)**: Multi-stage pipeline (PDF up to 50MB -> Cloud OCR -> Skeleton Analysis -> Chunked Node Gen -> Partitioned Dynamic Assessment Gen -> HTML Assembly & Validation).
- **Backend API (`server/`, Fastify Node.js)**: REST API for file uploads, transformation jobs, offline sync (push/pull), chapter verification, and telemetry. Standardized error taxonomy and JSON envelopes.
- **Teacher/Admin Dashboard (`dashboard/`, React + Vite + Tailwind)**: Simple raw JSON text-area editor for chapter customization, upload monitor, job status viewer.
- **Monorepo Packages (`packages/`)**:
  - `@aasha/shared-types`: Canonical TypeScript interfaces for entities (`GeneratedChapter`, `PromptTemplate`, `StudentProgress`, `SyncEvent`), nodes, assessments, LLE dictionary, and sync payloads.
  - `@aasha/chapter-builder`: CLI & library to assemble and compile self-contained single-file HTML chapters.
  - `@aasha/test-harness`: Reusable validation suite verifying offline HTML chapters (zero CDN/network calls, budget <= 20MB, valid quiz answer keys, sandboxed VM DOM execution, Acorn AST security checks).
- **Infrastructure (`infra/`)**: Docker Compose for PostgreSQL 16, Redis 7 (job queue), MinIO (S3-compatible object storage).

## Feature Inventory
| # | Feature | Description | Milestone | Source |
|---|---------|-------------|-----------|--------|
| 1 | Monorepo Structure & Ecosystem Config | Server, dashboard, chapters, packages (@aasha/shared-types, @aasha/chapter-builder, @aasha/test-harness), infra | M1 | Completed |
| 2 | Audience & Board Scope Expansion | Class 1-10 (ages 6-15), CBSE, ICSE, State Boards, IB, Cambridge, NIOS across 3 age bands | M1 | Completed |
| 3 | Single-file Chapter Budget <= 20MB | Self-contained HTML with embedded assets, zero external network dependency | M1 | Completed |
| 4 | DB Schemas for Source Material & Jobs | `source_uploads`, `transformation_jobs`, `generated_chapters`, `prompt_templates`, `sync_events`, `student_progress` | M1 | Completed |
| 5 | Backend REST API Endpoints | Uploads, transformation status, sync push/pull, verification, telemetry, standardized error envelopes | M1 | Completed |
| 6 | Cloud OCR & Document Preprocessing | High-accuracy Cloud OCR API integration for up to 50MB PDFs, Sharp auto-deskew/shadow removal, Devanagari NFC normalization | M2 | Completed |
| 7 | Multi-Stage LLM Prompt Engineering | Skeleton analysis, chunked per-node generation, partitioned dynamic assessment generation with temperature tuning & fallback | M2 | Completed |
| 8 | Visual Coordinates & Raw Canvas Scripts | Dual-mode visual representation: structured coordinate bounding boxes and generative raw JS canvas drawing scripts in 0-1000 space | M2 | Completed |
| 9 | Chapter Validation & Test Harness | Automated verification: zero CDNs, budget <= 20MB, 100% correct quiz answer keys, Acorn AST security checks, VM DOM integrity | M2 | Completed |
| 10 | Enhanced LLE Drop-in Engine | `Aasha_LLE_Enhanced.js` (~60 KB), 100% backward compatibility with `rt()`, `applyLLE()`, `showWord()`, `WM`, `CONN` | M3 | Completed |
| 11 | Audio Pronunciation & Tone Fallback | Web Speech API (`SpeechSynthesis`) at rate 0.85 for English & Hindi; Web Audio API oscillator rhythm fallback | M3 | Completed |
| 12 | Visual SVG Catalog & Transliteration | 55+ inline SVG math/science vocabulary illustrations, Romanized Hindi transliteration in word dialogs | M3 | Completed |
| 13 | Teacher Dashboard Raw JSON Editor | Simple raw JSON text-area editor for chapter customization and prompt tuning | M3 | Completed |
| 14 | Phased Beta Implementation Plan | Chronologically sequenced 5-phase roadmap with strict Beta scope isolation (prioritizing offline learning & AI pipeline) | M4 | Completed |

## Milestones
| # | Name | Scope | Dependencies | Status |
|---|------|-------|-------------|--------|
| 1 | M1: Technical & Architectural Synthesis | Monorepo layout, DB schemas (`source_uploads`, `transformation_jobs`, `generated_chapters`, `prompt_templates`, `student_progress`, `sync_events`), REST APIs, audience/budget models | none | DONE |
| 2 | M2: AI Transformation Pipeline Specification | Cloud OCR strategy, multi-stage LLM prompt architecture, visual coordinate & canvas generation, test harness validation | M1 | DONE |
| 3 | M3: Enhanced LLE & Client Experience Engine | `Aasha_LLE_Enhanced.js` drop-in specification, Web Speech / Web Audio fallback, 55+ SVG dictionary, transliteration, raw JSON dashboard editor | M1 | DONE |
| 4 | M4: Phased Beta Build Plan & Synthesis Artifacts | Phased roadmap, Beta scope isolation, risk mitigation, and comprehensive architectural deliverable authoring | M1, M2, M3 | DONE |

## Master Artifacts Produced
- `c:\Users\vanda\Downloads\html5 ngo\Aasha_Architectural_Synthesis_and_Implementation_Plan.md` (Version 2.1.0)
- `C:\Users\vanda\.gemini\antigravity\brain\ae11f9fd-f67e-4fbf-b9cf-3d085bb44172\Aasha_Architectural_Synthesis_and_Implementation_Plan.md`
