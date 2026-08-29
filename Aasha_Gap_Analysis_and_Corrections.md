# Aasha — Gap Analysis & Corrections

**Product:** Aasha — Safe Learning. Real Impact.  
**Organisation:** Annanth Aasha Foundation  
**Document Type:** Gap Analysis & Corrections to All Prior Documents  
**Version:** 1.0  
**Date:** August 2026  
**Status:** CRITICAL — Must be applied before implementation  
**Purpose:** This document identifies every gap, hallucination, and misalignment across the six prior documents (PRD, TRD, App Flow, UI/UX Brief, Backend Schema, Implementation Plan) against two clarified requirements:

1. **Audience:** Aasha serves children from Class 1 to Class 10, all boards (CBSE, ICSE, State, IB, Cambridge, NIOS), any school — not just NCERT Class 5-8.
2. **Content Creation Model:** Teachers upload scanned PDFs of children's school books or copied text/ebooks. The system uses AI to transform the uploaded content into interactive HTML chapters, including generating 100% of the assessment questions from that specific chapter's content.

---

## Table of Contents

1. [The Two Critical Clarifications](#1-the-two-critical-clarifications)
2. [Gap Analysis by Document](#2-gap-analysis-by-document)
3. [New Architecture: The Transformation Pipeline](#3-new-architecture-the-transformation-pipeline)
4. [Corrections to PRD](#4-corrections-to-prd)
5. [Corrections to TRD](#5-corrections-to-trd)
6. [Corrections to App Flow Document](#6-corrections-to-app-flow-document)
7. [Corrections to UI/UX Design Brief](#7-corrections-to-uiux-design-brief)
8. [Corrections to Backend Schema](#8-corrections-to-backend-schema)
9. [Corrections to Implementation Plan](#9-corrections-to-implementation-plan)
10. [New Tables Required for Transformation Pipeline](#10-new-tables-required-for-transformation-pipeline)
11. [New API Endpoints Required](#11-new-api-endpoints-required)
12. [New Screens Required](#12-new-screens-required)
13. [Revised Implementation Sequence](#13-revised-implementation-sequence)

---

## 1. The Two Critical Clarifications

### 1.1 Audience Correction

| What Documents Say | What Is Actually True |
|---|---|
| "School-going children aged 10–14 (Class 5–8)" (PRD L54) | Class 1 to Class 10, ages 6-15 |
| "NCERT Math" as the only subject context (PRD L22-24) | All boards: CBSE, ICSE, State, IB, Cambridge, NIOS |
| "Indian government schools and low-income private schools" (PRD L54) | Any school — government, private, international, homeschool |
| "Full NCERT coverage" as roadmap goal (PRD L583) | Full coverage across all boards, not just NCERT |
| File size "under 500 KB" (PRD L59, TRD L884) | Up to 20 MB per chapter file (corrected in later docs but PRD still says 500 KB) |

### 1.2 Content Creation Model Correction

| What Documents Say | What Is Actually True |
|---|---|
| "Content Author builds chapter HTML file (Node.js script)" (TRD L1089-1090) | Teacher uploads scanned PDF of school book or copied text |
| "A content author can write a chapter definition in JSON, run npm run build:chapter" (ImplPlan L751) | System auto-transforms PDF/text into HTML chapter using AI |
| "POST /api/v1/chapters — Upload new chapter (admin only)" (TRD L542) | Teachers (not just admins) upload source material; system generates the chapter |
| AI is "Phase 3, future" (TRD L592-593, PRD L405-407) | AI transformation is the CORE product workflow, not a future phase |
| "No AI in Phase 1" (TRD Decision 8, L1047-1055) | AI-driven content transformation is Phase 1 — it is how chapters are created |
| Assessments are "pre-authored" by content authors | Assessments are AI-generated from the uploaded chapter content (100%) |
| "content_json field... allows the content authoring tool to write directly to this table" (Backend L826) | AI transformation pipeline writes to this table automatically |

---

## 2. Gap Analysis by Document

### 2.1 PRD Gaps

| # | Gap | Location | Severity |
|---|---|---|---|
| 1 | Target audience restricted to Class 5-8, ages 10-14 | L54 | HIGH — excludes 60% of intended users |
| 2 | Only NCERT mentioned, no other boards | L22-24, L222, L392, L583 | HIGH — excludes ICSE, State, IB, etc. |
| 3 | No mention of teacher uploading school book PDF | Entire document | CRITICAL — missing core workflow |
| 4 | No mention of AI transforming PDF into chapter | Entire document | CRITICAL — missing core workflow |
| 5 | No mention of AI generating assessment questions from chapter content | Section 6.3 (Question Types) | CRITICAL — missing core feature |
| 6 | AI positioned as "Phase 3, future" | L405-407 | CRITICAL — AI is the core, not future |
| 7 | File size says "under 500 KB" | L59 | MEDIUM — should be "up to 20 MB" |
| 8 | No user story for teacher uploading content | Section 7.9 (Teacher stories) | HIGH — missing primary teacher action |
| 9 | No user story for assessment generation from book content | Section 7 | HIGH — missing core feature |
| 10 | Worksheet assessments described as "drawn from NCERT worksheets" | L222 | MEDIUM — should be generated from any uploaded content |

### 2.2 TRD Gaps

| # | Gap | Location | Severity |
|---|---|---|---|
| 1 | No architecture for PDF upload and processing | Entire document | CRITICAL |
| 2 | No OCR pipeline defined | N/A | CRITICAL — scanned PDFs need OCR |
| 3 | No AI transformation pipeline (PDF → structured content → HTML chapter) | N/A | CRITICAL |
| 4 | No LLM prompt engineering architecture for content transformation | Section 7 (AI Models) | CRITICAL |
| 5 | AI described as "Phase 3" and "not for children directly" | L592-593, L1055 | CRITICAL — AI transformation IS the content creation mechanism |
| 6 | Content authoring described as manual JSON writing | L1089-1090 | HIGH — should be AI-driven |
| 7 | File upload security mentions "max 5 MB" | L823 | MEDIUM — PDFs may be larger (up to 50 MB for scanned books) |
| 8 | No mention of PDF text extraction vs. scanned image OCR | N/A | HIGH |
| 9 | No mention of question auto-generation from chapter text | N/A | CRITICAL |
| 10 | "No AI in Phase 1" is Decision 8 | L1047-1055 | CRITICAL — must be reversed |

### 2.3 App Flow Document Gaps

| # | Gap | Location | Severity |
|---|---|---|---|
| 1 | No screen for teacher uploading PDF/text | Screen inventory (S01-S44) | CRITICAL — missing S45+ |
| 2 | No screen for transformation progress/status | N/A | CRITICAL |
| 3 | No screen for AI-generated chapter review/editing | N/A | HIGH |
| 4 | No screen for AI-generated assessment review | N/A | HIGH |
| 5 | No flow for teacher selecting which chapter to transform | N/A | CRITICAL |
| 6 | No mention of assessment auto-generation in any flow | N/A | CRITICAL |

### 2.4 UI/UX Design Brief Gaps

| # | Gap | Location | Severity |
|---|---|---|---|
| 1 | No design for PDF upload screen | Component inventory | HIGH |
| 2 | No design for transformation progress UI | N/A | HIGH |
| 3 | No design for teacher content review interface | N/A | HIGH |
| 4 | No design for drag-and-drop file upload | N/A | MEDIUM |
| 5 | No mention of file format support (PDF, JPG, PNG, TXT) | N/A | HIGH |

### 2.5 Backend Schema Gaps

| # | Gap | Location | Severity |
|---|---|---|---|
| 1 | No table for source material uploads (PDFs, text) | 43 tables defined, none for uploads | CRITICAL |
| 2 | No table for transformation jobs | N/A | CRITICAL |
| 3 | No table for AI-generated content drafts | N/A | CRITICAL |
| 4 | No table for assessment generation jobs | N/A | CRITICAL |
| 5 | No table for transformation review/approval | N/A | HIGH |
| 6 | No enum for transformation status | Section 2 | CRITICAL |
| 7 | No enum for source material type | Section 2 | HIGH |
| 8 | chapter builder script assumes manual JSON input | ImplPlan Phase 4 | CRITICAL |
| 9 | No stored procedure or job queue for AI transformation | N/A | CRITICAL |
| 10 | No table for LLM prompt templates | N/A | HIGH |

### 2.6 Implementation Plan Gaps

| # | Gap | Location | Severity |
|---|---|---|---|
| 1 | No phase for PDF upload and processing | Phases 0-14 | CRITICAL |
| 2 | No phase for AI transformation pipeline | N/A | CRITICAL |
| 3 | No phase for assessment auto-generation | N/A | CRITICAL |
| 4 | Phase 4 (Content Pipeline) assumes manual authoring | L649-751 | CRITICAL — must be completely rewritten |
| 5 | No mention of OCR, text extraction, or LLM processing | Entire plan | CRITICAL |
| 6 | AI is pushed to "Phase 3" of the PRD roadmap | N/A | CRITICAL — must be Phase 1 |
| 7 | No infrastructure for AI model hosting (Ollama, GPU) | Phase 13 | HIGH |
| 8 | No cost estimate for AI model inference | Effort estimates | HIGH |

---

## 3. New Architecture: The Transformation Pipeline

This is the core workflow that was missing from all documents. It must be designed, specified, and implemented as a first-class citizen of the system.

### 3.1 The Teacher-to-Child Content Flow

```
TEACHER                                    AASHA SYSTEM                                     CHILD
  │                                                                                             │
  │  1. Teacher has a school book                                                               │
  │     (any board, any class, any subject)                                                     │
  │                                                                                             │
  │  2. Teacher scans pages or copies text                                                      │
  │     (PDF, JPG, PNG, or pasted text)                                                        │
  │                                                                                             │
  │  3. Teacher uploads to Aasha                                                               │
  │     ──────────────────────────────────▶                                                     │
  │                                          4. System extracts text                              │
  │                                             ├─ If PDF with text: extract directly           │
  │                                             └─ If scanned images: OCR (Tesseract/cloud)     │
  │                                                                                             │
  │                                          5. System structures content                       │
  │                                             ├─ Identify chapter boundaries                  │
  │                                             ├─ Identify concepts/topics                     │
  │                                             ├─ Identify key terms for LLE                   │
  │                                             └─ Identify difficulty level per concept        │
  │                                                                                             │
  │                                          6. AI transforms into learning nodes             │
  │                                             ├─ Generate concept introductions              │
  │                                             ├─ Generate plain-language explanations        │
  │                                             ├─ Generate visual descriptions (Canvas)        │
  │                                             ├─ Generate interactive activities              │
  │                                             ├─ Generate worked examples                     │
  │                                             └─ Generate LLE word map entries                │
  │                                                                                             │
  │                                          7. AI generates assessments (100%)                │
  │                                             ├─ Quiz questions (MCQ) from chapter content    │
  │                                             ├─ True/False questions from facts              │
  │                                             ├─ Fill-in-the-blank from key terms             │
  │                                             ├─ Solve problems from exercises                 │
  │                                             ├─ Worksheet questions (Basic→HOTS)             │
  │                                             └─ Misconception feedback for wrong answers     │
  │                                                                                             │
  │                                          8. System assembles HTML chapter                  │
  │                                             ├─ CSS (design system tokens)                   │
  │                                             ├─ JS (App object, NODES, WE, SOLVE)           │
  │                                             ├─ LLE (WM, CONN)                              │
  │                                             ├─ Gamification (timer, shop, badges)            │
  │                                             └─ Validation harness check                    │
  │                                                                                             │
  │  9. Teacher reviews generated chapter    │                                                  │
  │     ◀──────────────────────────────────  │                                                  │
  │     ├─ Review nodes and steps             │                                                  │
  │     ├─ Review questions for accuracy      │                                                  │
  │     ├─ Edit any content                   │                                                  │
  │     └─ Approve and publish               │                                                  │
  │                                                                                             │
  │ 10. Teacher assigns chapter to class     │                                                  │
  │     ──────────────────────────────────▶  │                                                  │
  │                                          11. Chapter available for download                │
  │                                                                                             │
  │                                                                                             │ 12. Child downloads
  │                                                                                             │     chapter HTML
  │                                                                                             │     (one-time, then offline)
  │                                                                                             │
  │                                                                                             │ 13. Child learns
  │                                                                                             │     (interactive, gamified,
  │                                                                                             │      bilingual, offline)
```

### 3.2 Transformation Pipeline Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                  AASHA TRANSFORMATION PIPELINE                         │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐                                                    │
│  │  Upload API   │  Teacher uploads PDF/JPG/PNG/TXT                  │
│  │  (Fastify)    │  POST /api/v1/content/upload                      │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐                                                    │
│  │  File Storage  │  Store original in MinIO                         │
│  │  (MinIO/S3)    │  aasha/source-materials/<job_id>/original.pdf    │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐                                                    │
│  │  Text Extract │  Determine: text PDF or scanned image?            │
│  │  Service       │  ├─ Text PDF: pdf-parse (Node.js)                 │
│  │               │  ├─ Image PDF: pdf-to-image → OCR                  │
│  │               │  └─ Image: Tesseract.js or cloud OCR               │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐                                                    │
│  │  Structure    │  LLM analyzes extracted text:                     │
│  │  Analysis     │  ├─ Identify chapter title and subject             │
│  │  (LLM)        │  ├─ Identify concepts/topics within chapter        │
│  │               │  ├─ Identify key vocabulary for LLE               │
│  │               │  ├─ Identify difficulty progression                │
│  │               │  └─ Output: structured JSON (chapter skeleton)     │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐                                                    │
│  │  Content      │  LLM generates learning content per concept:       │
│  │  Generation   │  ├─ Concept introduction (plain language)          │
│  │  (LLM)        │  ├─ Text explanation (with LLE word tags)          │
│  │               │  ├─ Visual description (Canvas drawing instructions)│
│  │               │  ├─ Interactive activity design                     │
│  │               │  ├─ Worked example (step-by-step)                   │
│  │               │  └─ LLE word map entries (English→Hindi)           │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐                                                    │
│  │  Assessment   │  LLM generates 100% of assessments from content:   │
│  │  Generation   │  ├─ Quiz (MCQ, 4 options, 1 correct)              │
│  │  (LLM)        │  ├─ Misconception feedback for each wrong option   │
│  │               │  ├─ True/False with explanation                    │
│  │               │  ├─ Fill-in-the-blank with answer + acceptable vars│
│  │               │  ├─ Solve problems (multi-step)                    │
│  │               │  ├─ Worksheet: Basic, Standard, HOTS, Final Mixed   │
│  │               │  └─ Rapid fire questions                          │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐                                                    │
│  │  Chapter      │  Assemble final HTML file:                         │
│  │  Assembler    │  ├─ CSS (design tokens from UI/UX brief)           │
│  │               │  ├─ HTML shell (header, dialogs, brand)             │
│  │               │  ├─ JS (App object, render functions)              │
│  │               │  ├─ NODES[], WE{}, SOLVE{}, WM{}, CONN{}          │
│  │               │  ├─ Gamification features (timer, shop, badges)    │
│  │               │  └─ Sync module                                    │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐                                                    │
│  │  Validation   │  Run validation harness:                           │
│  │  Harness      │  ├─ Every quiz has 1 correct answer               │
│  │               │  ├─ Every fill-blank has answer                    │
│  │               │  ├─ No external URLs                               │
│  │               │  ├─ File size < 20 MB                               │
│  │               │  └─ All step types have render functions            │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐                                                    │
│  │  Review &     │  Teacher reviews generated chapter:                │
│  │  Approval     │  ├─ Preview rendered chapter                       │
│  │               │  ├─ Edit any node/step/question                    │
│  │               │  ├─ Regenerate specific sections                   │
│  │               │  └─ Approve and publish                           │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐                                                    │
│  │  Publish      │  Chapter HTML uploaded to MinIO                   │
│  │               │  Chapter metadata stored in PostgreSQL             │
│  │               │  Available for download by children                │
│  └──────────────┘                                                    │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### 3.3 AI Models Required

| Stage | Model | Purpose | Hosting |
|---|---|---|---|
| Text extraction | pdf-parse (Node.js) | Extract text from digital PDFs | Server-side (no AI) |
| OCR | Tesseract.js or Google Vision API | Extract text from scanned images | Server-side / cloud API |
| Structure analysis | LLM (Llama 3.1 70B or GPT-4o) | Identify concepts, structure, key terms | API (OpenAI/Together) or self-hosted (Ollama) |
| Content generation | LLM (Llama 3.1 70B or GPT-4o) | Generate explanations, examples, visual descriptions | Same as above |
| Assessment generation | LLM (Llama 3.1 70B or GPT-4o) | Generate quiz, T/F, fill-blank, solve, worksheet questions | Same as above |
| Hindi translation | Sarvam AI or LLM | Generate LLE word map entries | API or self-hosted |
| Validation | Rule-based + LLM checker | Verify generated content quality | Server-side |

### 3.4 Why This Changes the Architecture

The current documents treat content as something a developer manually creates (writing JSON, running Node.js scripts). The clarified model treats content as something the system generates from any teacher's uploaded material. This means:

1. **AI is not Phase 3 — it is Phase 1.** Without AI transformation, the platform cannot create chapters. The 3 existing chapters were manually built as a prototype to prove the learning experience works. The production system must automate this.

2. **The backend needs a job processing system.** Transformation jobs are long-running (30 seconds to 5 minutes per chapter). They need a queue, worker, and status tracking.

3. **The teacher dashboard needs an upload and review interface.** This is not just a "content management" page — it is the primary teacher action.

4. **The assessment system is not a static question bank.** It is a dynamic, AI-generated set of questions specific to each uploaded chapter. The `question_bank` table in the backend schema stores these generated questions, but the generation pipeline is missing.

5. **The file size budget increases.** Scanned PDFs can be 10-50 MB. The system must handle large uploads, extract text, and discard the original (the final HTML chapter is still ≤20 MB, but the source material can be larger).

---

## 4. Corrections to PRD

### 4.1 Section 3.1 — Target Users (CORRECTED)

**Replace:**
> School-going children aged 10–14 (Class 5–8) in Indian government schools and low-income private schools

**With:**
> School-going children from Class 1 to Class 10 (ages 6–15) in any school — government, private, international, or homeschool — following any board (CBSE, ICSE, State, IB, Cambridge, NIOS, or other). The platform adapts to the child's class level, board, and medium of instruction.

### 4.2 Section 3.2 — Teacher / Facilitator (CORRECTED)

**Add to teacher description:**
> The teacher is the primary content contributor. A teacher uploads a scanned PDF of the child's school book chapter, or pastes/copies text from the textbook or ebook. The Aasha system then uses AI to transform this source material into a complete interactive HTML learning chapter — including concept explanations, visuals, interactive activities, worked examples, and 100% of the assessment questions. The teacher reviews and approves the generated chapter before it is assigned to children.

### 4.3 New Section 6.11 — Content Transformation Pipeline (NEW)

Add to Core Features:

> ### 6.11 Content Transformation Pipeline
>
> Aasha does not require developers or content authors to manually create chapters. Instead, teachers upload their school book content and the system transforms it automatically.
>
> **Upload:**
> - Teacher uploads a scanned PDF (pages photographed or scanned), an image (JPG/PNG of textbook pages), or pastes text from an ebook/textbook
> - Supported formats: PDF, JPG, PNG, TXT, and pasted rich text
> - Maximum file size: 50 MB (scanned books can be large)
> - The system accepts any board, any class (1-10), any subject
>
> **Text Extraction:**
> - If the PDF has embedded text (digital PDF): extract directly using pdf-parse
> - If the PDF is scanned images (common for photocopied textbooks): OCR using Tesseract or cloud OCR
> - If an image is uploaded: OCR
> - If text is pasted: use directly
>
> **AI Transformation:**
> - An LLM analyzes the extracted text and structures it into concepts
> - For each concept, the LLM generates:
>   - Concept introduction (plain language)
>   - Text explanation (with LLE word tags for difficult vocabulary)
>   - Visual description (Canvas drawing instructions)
>   - Interactive activity design (drag, slider, tap)
>   - Worked example (step-by-step solved problem from the chapter's exercises)
>   - LLE word map entries (English → Hindi translations for key terms)
>
> **Assessment Generation (100% AI-generated):**
> - The LLM generates all assessment questions from the chapter content itself:
>   - Quiz (MCQ) with 4 options, 1 correct, and misconception feedback for each wrong option
>   - True/False with explanation
>   - Fill-in-the-blank with answer and acceptable variants
>   - Solve problems (multi-step, from the chapter's exercises)
>   - Worksheet: Basic, Standard, HOTS, and Final Mixed assessment
>   - Rapid Fire questions
>   - Memory Match pairs
> - Questions are generated at appropriate difficulty levels for the class
> - All questions are tied to the specific content of the uploaded chapter
>
> **Chapter Assembly:**
> - The system assembles the generated content into a self-contained HTML file
> - Includes the full design system (CSS tokens, typography, components)
> - Includes the App object, render functions, gamification features
> - Runs the validation harness (checks structural integrity, answer correctness)
> - File size target: ≤20 MB
>
> **Teacher Review:**
> - Teacher previews the generated chapter
> - Teacher can edit any node, step, or question
> - Teacher can regenerate specific sections
> - Teacher approves and publishes
> - Published chapter is available for children to download

### 4.4 New User Stories (ADD)

**Teacher — Content Creation:**

| ID | Story |
|---|---|
| US-30 | As a teacher, I want to upload a scanned PDF of my school book chapter so that Aasha can transform it into an interactive learning experience for my students. |
| US-31 | As a teacher, I want to paste text from an ebook or textbook so that Aasha can create a chapter without needing a scanner. |
| US-32 | As a teacher, I want the system to automatically extract text from my uploaded PDF, whether it is a digital PDF or a scanned image. |
| US-33 | As a teacher, I want the system to generate interactive explanations, visuals, and activities from my chapter content. |
| US-34 | As a teacher, I want the system to generate quiz questions, true/false, fill-in-the-blank, and solve problems from the chapter content — I should not have to write any questions myself. |
| US-35 | As a teacher, I want the system to generate worksheet assessments at Basic, Standard, and HOTS difficulty levels from the chapter content. |
| US-36 | As a teacher, I want to review the generated chapter before it is assigned to children, so I can verify accuracy and edit if needed. |
| US-37 | As a teacher, I want to upload chapters from any board (CBSE, ICSE, State, etc.) and any class (1-10), not just NCERT. |

### 4.5 Roadmap Correction (CORRECTED)

**Replace:**
> Phase 1 — Prove Learning (Current)
> Phase 2 — Personalise Learning
> Phase 3 — Connect Learning to Action

**With:**
> Phase 1 — Prove Learning + Build Transformation Pipeline
> - 3 manually-built chapters prove the learning experience works (DONE)
> - Build the AI transformation pipeline so ANY teacher can upload ANY chapter
> - This is the core product — without it, Aasha cannot scale beyond 3 chapters
>
> Phase 2 — Teacher Dashboards + Sync
> - Teacher dashboard with upload, review, and class management
> - Sync layer for offline progress
> - Analytics
>
> Phase 3 — Personalise + Connect to Action
> - Adaptive difficulty
> - Eco Seva, Jal Seva
> - Aasha Economy

### 4.6 File Size Correction

**Replace all instances of:**
> Files must be small, under 500 KB each

**With:**
> Chapter HTML files must be under 20 MB each. Source material uploads (PDFs, images) may be up to 50 MB.

---

## 5. Corrections to TRD

### 5.1 AI Section — Complete Rewrite

**Replace Section 7 (AI Models and Tools) with:**

> ### 7. AI Models and Tools
>
> AI is the core content creation mechanism for Aasha. It is not a future feature — it is how chapters are built in production.
>
> #### 7.1 Transformation Pipeline (Phase 1 — Critical)
>
> | Stage | Model/Tool | Purpose | License |
> |---|---|---|---|
> | Text extraction (digital PDF) | pdf-parse (npm) | Extract text from PDFs with embedded text | MIT |
> | Image conversion | pdf2pic / sharp | Convert PDF pages to images for OCR | MIT |
> | OCR (scanned PDFs/images) | Tesseract.js | Extract text from scanned textbook pages | Apache 2.0 |
> | OCR (cloud fallback) | Google Vision API | Higher accuracy OCR for difficult scans | Paid API |
> | Structure analysis | LLM (GPT-4o or Llama 3.1 70B) | Identify concepts, structure, key terms, difficulty | API / self-hosted |
> | Content generation | Same LLM | Generate explanations, examples, visuals, worked solutions | Same |
> | Assessment generation | Same LLM | Generate quiz, T/F, fill-blank, solve, worksheet, rapid fire | Same |
> | Hindi translation | Sarvam AI or LLM | Generate LLE word map (English → Hindi) | Open weights / API |
> | Quality validation | Rule-based checker | Verify generated content structure and answer correctness | Custom |
>
> #### 7.2 LLM Configuration
>
> ```typescript
> // server/src/services/llm.ts
> interface LLMConfig {
>   provider: 'openai' | 'together' | 'ollama' | 'anthropic';
>   model: string;
>   apiKey?: string;
>   baseUrl?: string;
>   maxTokens: number;
>   temperature: number;
> }
>
> // Content generation: lower temperature for accuracy
> const contentConfig: LLMConfig = {
>   provider: 'openai',
>   model: 'gpt-4o',
>   maxTokens: 4096,
>   temperature: 0.3,  // Low — educational content must be accurate
> };
>
> // Assessment generation: slightly higher for variety
> const assessmentConfig: LLMConfig = {
>   provider: 'openai',
>   model: 'gpt-4o',
>   maxTokens: 4096,
>   temperature: 0.4,  // Slightly higher for question variety
> };
> ```
>
> #### 7.3 Prompt Architecture
>
> The transformation pipeline uses structured prompts that produce JSON output:
>
> 1. **Structure analysis prompt:** "Analyze this chapter text. Identify the main concepts (3-8 per chapter). For each concept, provide: title, subtitle, key vocabulary words, and difficulty level (easy/medium/hard). Output as JSON."
>
> 2. **Content generation prompt (per concept):** "For this concept [title], generate: (a) a plain-language introduction, (b) an explanation with difficult words tagged for LLE, (c) a Canvas visual description, (d) an interactive activity, (e) a worked example with 4-5 steps. Match the language level of Class [N]. Output as JSON matching the NODES structure."
>
> 3. **Assessment generation prompt (per concept):** "From this concept content, generate: (a) 3 quiz MCQs with 4 options each (1 correct, 3 wrong with misconception explanations), (b) 2 true/false with explanations, (c) 2 fill-in-the-blank with answers, (d) 1 solve problem with 3 steps. Output as JSON matching the question_bank schema."
>
> 4. **Worksheet generation prompt:** "From this entire chapter, generate: (a) 10 Basic-level MCQs and fill-blanks, (b) 10 Standard-level questions, (c) 5 HOTS questions, (d) 10 Final Mixed questions. Output as JSON."

### 5.2 Decision 8 — REVERSED

**Replace Decision 8 ("No AI in Phase 1") with:**

> ### Decision 8 (REVISED): AI Transformation is Phase 1
>
> **Decision:** The AI content transformation pipeline is built in Phase 1, not Phase 3.
>
> **Reasoning:**
> - Without AI transformation, Aasha cannot scale beyond the 3 manually-built chapters
> - The core product value is: any teacher uploads any book → children get interactive learning
> - Manual chapter building is a proof-of-concept activity, not a production workflow
> - The 3 existing chapters proved the learning experience works; now the system must automate creation
>
> **Implementation:**
> - LLM API calls for content and assessment generation
> - OCR for scanned PDFs
> - Job queue for long-running transformations
> - Teacher review interface for quality control
> - All AI-generated content is reviewed by the teacher before publishing

### 5.3 File Upload Size Correction

**Replace:**
> File upload security: Magic byte validation, max 5 MB, virus scan (ClamAV)

**With:**
> File upload security: Magic byte validation, max 50 MB for source materials (PDFs, images), max 20 MB for chapter HTML files, virus scan (ClamAV)

---

## 6. Corrections to App Flow Document

### 6.1 New Screens to Add

| # | Screen | Phase | User | Description |
|---|---|---|---|---|
| S45 | Upload Source Material | 1 | Teacher | Upload PDF/image/text for transformation |
| S46 | Transformation Progress | 1 | Teacher | Live progress of AI transformation job |
| S47 | Chapter Review — Node List | 1 | Teacher | Review generated nodes and steps |
| S48 | Chapter Review — Question Editor | 1 | Teacher | Review and edit AI-generated questions |
| S49 | Chapter Review — Preview | 1 | Teacher | Preview the rendered chapter as a child would see it |
| S50 | Chapter Publish | 1 | Teacher | Approve and publish chapter to children |

### 6.2 Upload Flow (S45)

```
[Teacher Dashboard → "Upload Chapter" button]
    │
    ▼
┌─────────────────────────────────────────────────────┐
│  SCREEN: Upload Source Material (S45)                │
│                                                       │
│  ┌─────────────────────────────────────────────┐    │
│  │  "Transform a School Book Chapter"            │    │
│  │                                               │    │
│  │  Step 1: Chapter Information                  │    │
│  │  Subject: [Mathematics ▼]                     │    │
│  │  Class: [1-10 ▼]                             │    │
│  │  Board: [CBSE ▼]                             │    │
│  │  Chapter Title: [________________]            │    │
│  │  Medium: [English ▼]                         │    │
│  │                                               │    │
│  │  Step 2: Upload Source Material                │    │
│  │  ┌───────────────────────────────────────┐  │    │
│  │  │                                         │  │    │
│  │  │     [📁 Drop PDF/Image here]            │  │    │
│  │  │     or [ Paste Text ]                   │  │    │
│  │  │                                         │  │    │
│  │  │  Supported: PDF, JPG, PNG, TXT           │  │    │
│  │  │  Max size: 50 MB                         │  │    │
│  │  │                                         │  │    │
│  │  └───────────────────────────────────────┘  │    │
│  │                                               │    │
│  │  Step 3: Assessment Configuration             │    │
│  │  Number of quiz questions per concept: [3]    │    │
│  │  Worksheet levels: [✓ Basic] [✓ Standard]     │    │
│  │                    [✓ HOTS] [✓ Final Mixed]   │    │
│  │  Rapid Fire: [✓]  Memory Match: [✓]          │    │
│  │  Language: [✓ Hindi LLE] [○ Tamil] [○ Telugu]│    │
│  │                                               │    │
│  │  [ Cancel ]           [ Start Transformation ]│    │
│  └─────────────────────────────────────────────┘    │
│                                                       │
│  EMPTY STATE: No file uploaded → drop zone visible    │
│  ERROR: File too large → "File exceeds 50 MB limit"   │
│  ERROR: Unsupported format → "Please upload PDF,     │
│         JPG, PNG, or TXT"                             │
└─────────────────────────────────────────────────────┘
```

### 6.3 Transformation Progress Flow (S46)

```
[Start Transformation tapped]
    │
    ▼
┌─────────────────────────────────────────────────────┐
│  SCREEN: Transformation Progress (S46)               │
│                                                       │
│  ┌─────────────────────────────────────────────┐    │
│  │  Transforming: "Understanding Quadrilaterals"│    │
│  │  Class 8, CBSE, Mathematics                  │    │
│  │                                               │    │
│  │  ✅ Uploading source material                 │    │
│  │  ✅ Extracting text (OCR)                     │    │
│  │  ✅ Analyzing chapter structure               │    │
│  │  🔄 Generating learning content (3/6 concepts)│    │
│  │  ⏳ Generating assessments                     │    │
│  │  ⏳ Assembling HTML chapter                    │    │
│  │  ⏳ Validating                                 │    │
│  │                                               │    │
│  │  [============>          ] 45%                │    │
│  │  Estimated time remaining: 2 minutes          │    │
│  │                                               │    │
│  │  [ Cancel ]                                   │    │
│  └─────────────────────────────────────────────┘    │
│                                                       │
│  Progress updates via WebSocket or polling (5s)      │
│                                                       │
│  ON COMPLETION → navigate to S47 Chapter Review      │
│  ON ERROR → "Transformation failed. [Retry] [Edit]"  │
└─────────────────────────────────────────────────────┘
```

### 6.4 Chapter Review Flow (S47-S49)

```
[Transformation complete]
    │
    ▼
┌─────────────────────────────────────────────────────┐
│  SCREEN: Chapter Review — Node List (S47)            │
│                                                       │
│  ┌─────────────────────────────────────────────┐    │
│  │  Back    Review: Understanding Quadrilaterals │    │
│  │  6 concepts generated · 45 questions generated│    │
│  └─────────────────────────────────────────────┘    │
│                                                       │
│  ┌─────────────────────────────────────────────┐    │
│  │  ⬢ Node 1: What is a Polygon?    [Edit] [✓]  │    │
│  │     8 steps · 6 questions                      │    │
│  ├─────────────────────────────────────────────┤    │
│  │  ⬣ Node 2: Classifying Polygons  [Edit] [✓]  │    │
│  │     7 steps · 5 questions                      │    │
│  ├─────────────────────────────────────────────┤    │
│  │  ✋ Node 3: Diagonals            [Edit] [⚠️]  │    │
│  │     6 steps · 4 questions · 1 warning         │    │
│  ├─────────────────────────────────────────────┤    │
│  │  ... (all nodes)                               │    │
│  └─────────────────────────────────────────────┘    │
│                                                       │
│  [ Preview Chapter ]  [ Publish Chapter ]            │
│                                                       │
│  ⚠️ = validation warning (e.g., MCQ has 2 correct    │
│     answers, fill-blank has no answer) — teacher      │
│     must fix before publishing                       │
└─────────────────────────────────────────────────────┘
```

### 6.5 Navigation Update

Add to the navigation map:

```
Teacher Dashboard (S38)
  └→ [Upload Chapter] → S45 Upload Source Material
       └→ S46 Transformation Progress
            └→ S47 Chapter Review (Node List)
                 ├→ S48 Question Editor (per node)
                 ├→ S49 Chapter Preview (rendered as child sees)
                 └→ S50 Publish Chapter
                      └→ back to S38 with success toast
```

---

## 7. Corrections to UI/UX Design Brief

### 7.1 New Components to Add

| Component | Variants | Used In |
|---|---|---|
| File Upload Drop Zone | drag-active, uploading, error, success | S45 |
| Progress Stepper | step-complete, step-active, step-pending | S46 |
| Transformation Progress Bar | indeterminate, determinate | S46 |
| Node Review Card | approved, warning, error, editing | S47 |
| Question Editor | MCQ, T/F, fill-blank, solve | S48 |
| Chapter Preview Frame | iframe rendering the HTML | S49 |
| Assessment Config Form | checkboxes, number inputs | S45 |

### 7.2 Upload Drop Zone Design

```
┌─────────────────────────────────────────────────────┐
│  Drop Zone Component                                  │
│                                                       │
│  Default state:                                       │
│  ┌─────────────────────────────────────────────┐    │
│  │  [📁 icon]                                    │    │
│  │  Drop PDF or image here                       │    │
│  │  or [ Browse Files ]                          │    │
│  │  or [ Paste Text ]                            │    │
│  │  PDF, JPG, PNG, TXT · Max 50 MB              │    │
│  └─────────────────────────────────────────────┘    │
│  Border: 2px dashed --border                          │
│  Background: --surface                                 │
│                                                       │
│  Drag-active state:                                   │
│  Border: 2px solid --bl (blue)                       │
│  Background: --green-lt (light tint)                  │
│  Text: "Drop to upload"                               │
│                                                       │
│  Uploading state:                                     │
│  Progress bar (blue, --bl)                             │
│  Text: "Uploading... 45%"                             │
│                                                       │
│  Success state:                                       │
│  Border: 2px solid --green                            │
│  Text: "✓ Uploaded: chapter8-math.pdf (3.2 MB)"      │
│  [ Remove ] button                                     │
│                                                       │
│  Error state:                                         │
│  Border: 2px solid --red                              │
│  Text: "✗ File too large. Max 50 MB."                │
│  [ Try Again ]                                        │
└─────────────────────────────────────────────────────┘
```

### 7.3 Age-Adaptive Design — Correction

The age-adaptive section (Section 13) already covers Class 1-3, 4-7, and 8-10. This is correct. No changes needed — just ensure the rest of the document doesn't contradict it by implying Class 5-8 only.

---

## 8. Corrections to Backend Schema

### 8.1 New Enum Types Required

```sql
-- Source material type
CREATE TYPE source_material_type AS ENUM (
    'pdf_digital',      -- PDF with embedded text
    'pdf_scanned',      -- PDF with scanned images
    'image',            -- JPG/PNG of textbook pages
    'text',             -- Pasted text
    'ebook'             -- Copied ebook text
);

-- Transformation status
CREATE TYPE transformation_status AS ENUM (
    'queued',           -- Job created, waiting for worker
    'extracting',       -- Text extraction / OCR in progress
    'analyzing',        -- LLM analyzing structure
    'generating_content', -- LLM generating learning content
    'generating_assessments', -- LLM generating questions
    'assembling',       -- Assembling HTML chapter
    'validating',       -- Running validation harness
    'review',           -- Ready for teacher review
    'published',        -- Teacher approved and published
    'failed',           -- Transformation failed
    'cancelled'         -- Teacher cancelled
);

-- Transformation stage
CREATE TYPE transformation_stage AS ENUM (
    'upload',
    'text_extraction',
    'structure_analysis',
    'content_generation',
    'assessment_generation',
    'chapter_assembly',
    'validation',
    'review',
    'publish'
);
```

### 8.2 New Tables Required

See Section 10 below for full DDL.

### 8.3 Corrections to Existing Tables

**`chapters` table — add columns:**

```sql
ALTER TABLE chapters ADD COLUMN source_material_id UUID REFERENCES source_materials(id) ON DELETE SET NULL;
ALTER TABLE chapters ADD COLUMN transformation_job_id UUID REFERENCES transformation_jobs(id) ON DELETE SET NULL;
ALTER TABLE chapters ADD COLUMN is_ai_generated BOOLEAN NOT NULL DEFAULT FALSE;
ALTER TABLE chapters ADD COLUMN review_status VARCHAR(20) DEFAULT 'not_reviewed'; -- not_reviewed, in_review, approved, rejected
ALTER TABLE chapters ADD COLUMN reviewed_by UUID REFERENCES users(id) ON DELETE SET NULL;
ALTER TABLE chapters ADD COLUMN reviewed_at TIMESTAMPTZ;
```

**`question_bank` table — add columns:**

```sql
ALTER TABLE question_bank ADD COLUMN is_ai_generated BOOLEAN NOT NULL DEFAULT FALSE;
ALTER TABLE question_bank ADD COLUMN generation_job_id UUID REFERENCES transformation_jobs(id) ON DELETE SET NULL;
ALTER TABLE question_bank ADD COLUMN teacher_verified BOOLEAN NOT NULL DEFAULT FALSE;
ALTER TABLE question_bank ADD COLUMN verified_by UUID REFERENCES users(id) ON DELETE SET NULL;
ALTER TABLE question_bank ADD COLUMN verified_at TIMESTAMPTZ;
```

---

## 9. Corrections to Implementation Plan

### 9.1 New Phase: Phase 4B — AI Transformation Pipeline

**Insert between Phase 4 (Content Pipeline) and Phase 5 (Sync Layer):**

**Goal:** Build the AI transformation pipeline that converts teacher-uploaded PDFs/text into interactive HTML chapters with 100% AI-generated assessments.

**Duration:** 8 days

**Steps:**

1. **Upload API and storage** (1 day): File upload endpoint, MinIO storage, file type validation, size limits
2. **Text extraction service** (1 day): pdf-parse for digital PDFs, Tesseract.js for scanned, text passthrough
3. **LLM integration** (1 day): OpenAI/Together API client, prompt templates, JSON output parsing
4. **Structure analysis pipeline** (1 day): LLM identifies concepts, key terms, difficulty from extracted text
5. **Content generation pipeline** (1 day): LLM generates nodes, steps, explanations, visuals, worked examples
6. **Assessment generation pipeline** (1 day): LLM generates quiz, T/F, fill-blank, solve, worksheet, rapid fire from chapter content
7. **Chapter assembler** (1 day): Combine generated content into HTML file using design system
8. **Teacher review interface** (1 day): Dashboard screens S45-S50 for upload, progress, review, edit, publish

**Deliverables:**
- [ ] Upload API (PDF, JPG, PNG, TXT, max 50 MB)
- [ ] Text extraction service (digital PDF, scanned PDF, image, text)
- [ ] LLM service with structured prompt architecture
- [ ] Structure analysis → concept identification
- [ ] Content generation → nodes, steps, explanations
- [ ] Assessment generation → quiz, T/F, fill-blank, solve, worksheet (100% from chapter content)
- [ ] Chapter assembler → validated HTML file
- [ ] Teacher review dashboard (upload, progress, review, edit, publish)
- [ ] Job queue for long-running transformations
- [ ] Transformation test suite

**Definition of Done:** A teacher can upload a scanned PDF of any school book chapter (any board, any class 1-10), and the system generates a complete interactive HTML chapter with all learning content and assessments within 5 minutes. The teacher can review, edit, and publish the chapter.

### 9.2 Phase 4 Correction

**Phase 4 (Content Pipeline) is renamed to "Chapter Management & Validation"** and its scope is reduced to:
- Chapter validation harness (already specified)
- Content migration tool (existing 3 chapters into database)
- Chapter download/distribution via API

The actual content creation (building new chapters) is now handled by Phase 4B (Transformation Pipeline).

### 9.3 Revised Phase Sequence

```
Phase 0 — Setup
Phase 1 — Database (add transformation tables)
Phase 2 — Auth
Phase 3 — Core API (add upload/transformation endpoints)
Phase 4 — Chapter Management & Validation (reduced scope)
Phase 4B — AI Transformation Pipeline (NEW — 8 days)
Phase 5 — Sync Layer
Phase 6 — Teacher Dashboard (add upload/review screens)
Phase 7 — Parent Dashboard
Phase 8 — Gamification Hardening
Phase 9 — Seva
Phase 10 — Economy
Phase 11 — Chapter Enhancement
Phase 12 — Testing (add transformation tests)
Phase 13 — Deployment (add AI model infrastructure)
Phase 14 — Launch
```

### 9.4 Revised Effort Estimate

| Phase | Original | Revised | Change |
|---|---|---|---|
| Phase 0 — Setup | 2 days | 2 days | — |
| Phase 1 — Database | 3 days | 4 days | +1 (transformation tables) |
| Phase 2 — Auth | 4 days | 4 days | — |
| Phase 3 — Core API | 5 days | 6 days | +1 (upload/transformation APIs) |
| Phase 4 — Chapter Mgmt | 3 days | 2 days | -1 (reduced scope) |
| **Phase 4B — Transformation** | **0 days** | **8 days** | **+8 (NEW)** |
| Phase 5 — Sync | 4 days | 4 days | — |
| Phase 6 — Teacher Dashboard | 6 days | 8 days | +2 (upload/review screens) |
| Phase 7 — Parent Dashboard | 3 days | 3 days | — |
| Phase 8 — Gamification | 4 days | 4 days | — |
| Phase 9 — Seva | 4 days | 4 days | — |
| Phase 10 — Economy | 5 days | 5 days | — |
| Phase 11 — Chapter Enhancement | 5 days | 5 days | — |
| Phase 12 — Testing | 5 days | 6 days | +1 (transformation tests) |
| Phase 13 — Deployment | 3 days | 4 days | +1 (AI infrastructure) |
| Phase 14 — Launch | 3 days | 3 days | — |
| **Total** | **59 days** | **72 days** | **+13** |

**With 2 engineers:** ~42 calendar days  
**With 3 engineers:** ~30 calendar days

### 9.5 AI Infrastructure Cost Addition

| Item | Cost (monthly) | Notes |
|---|---|---|
| LLM API (OpenAI GPT-4o) | ~₹2-5/chapter | ~$0.03-0.06 per chapter transformation |
| OCR (Google Vision, fallback) | ~₹0.50/page | Only for difficult scans |
| Ollama (self-hosted, optional) | ₹0 | Free but needs GPU VPS (~₹2,000/month) |
| **Total AI cost per chapter** | **~₹5-15** | Depending on length and OCR needs |

For 100 chapters/month: ~₹500-1,500/month AI costs. Very affordable.

---

## 10. New Tables Required for Transformation Pipeline

```sql
-- 10.1 source_materials — Teacher-uploaded content
CREATE TABLE source_materials (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    uploaded_by     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    -- Material info
    material_type   source_material_type NOT NULL,
    original_filename VARCHAR(255),
    file_size_bytes BIGINT NOT NULL,
    file_checksum   VARCHAR(64) NOT NULL,              -- SHA-256
    
    -- Storage
    file_url        VARCHAR(500) NOT NULL,             -- MinIO URL
    file_bucket     VARCHAR(100) NOT NULL DEFAULT 'aasha',
    file_key        VARCHAR(500) NOT NULL,
    
    -- Content metadata
    subject_code    subject_code,
    grade           INTEGER CHECK (grade BETWEEN 1 AND 10),
    board           board_code,
    chapter_title   VARCHAR(200),
    medium          VARCHAR(20) DEFAULT 'english',
    
    -- Extracted text (stored for re-processing without re-upload)
    extracted_text  TEXT,                              -- Full extracted text
    extraction_method VARCHAR(20),                     -- 'pdf_parse', 'tesseract', 'google_vision', 'pasted'
    extraction_completed_at TIMESTAMPTZ,
    
    -- Status
    is_processed    BOOLEAN NOT NULL DEFAULT FALSE,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at      TIMESTAMPTZ
);

-- 10.2 transformation_jobs — AI transformation pipeline jobs
CREATE TABLE transformation_jobs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    source_material_id UUID NOT NULL REFERENCES source_materials(id) ON DELETE CASCADE,
    triggered_by    UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    -- Configuration
    config          JSONB NOT NULL DEFAULT '{}',      -- {questionsPerConcept: 3, worksheetLevels: [...], language: 'hi', ...}
    
    -- Status
    status          transformation_status NOT NULL DEFAULT 'queued',
    current_stage   transformation_stage,
    
    -- Progress
    progress_percentage NUMERIC(5,2) DEFAULT 0.00,
    estimated_time_remaining_seconds INTEGER,
    
    -- Results
    generated_chapter_id UUID REFERENCES chapters(id) ON DELETE SET NULL,
    nodes_generated    INTEGER DEFAULT 0,
    steps_generated    INTEGER DEFAULT 0,
    questions_generated INTEGER DEFAULT 0,
    
    -- Error handling
    error_message   TEXT,
    error_stage     transformation_stage,
    retry_count     INTEGER NOT NULL DEFAULT 0,
    max_retries     INTEGER NOT NULL DEFAULT 3,
    
    -- LLM metadata
    llm_model       VARCHAR(100),
    llm_tokens_used INTEGER,
    llm_cost_usd   NUMERIC(10,4),
    
    -- Timing
    started_at      TIMESTAMPTZ,
    completed_at    TIMESTAMPTZ,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 10.3 transformation_stages — Detailed log of each pipeline stage
CREATE TABLE transformation_stages (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_id          UUID NOT NULL REFERENCES transformation_jobs(id) ON DELETE CASCADE,
    
    stage           transformation_stage NOT NULL,
    status          VARCHAR(20) NOT NULL DEFAULT 'pending', -- pending, processing, completed, failed
    input_data      JSONB,                                 -- Input to this stage
    output_data     JSONB,                                 -- Output from this stage
    
    -- LLM details (if applicable)
    llm_prompt      TEXT,
    llm_response    TEXT,
    llm_tokens_input INTEGER,
    llm_tokens_output INTEGER,
    
    duration_ms     INTEGER,
    error_message   TEXT,
    
    started_at      TIMESTAMPTZ,
    completed_at    TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 10.4 llm_prompt_templates — Reusable prompt templates
CREATE TABLE llm_prompt_templates (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    template_name   VARCHAR(100) NOT NULL UNIQUE,        -- 'structure_analysis', 'content_generation', etc.
    template_type   transformation_stage NOT NULL,
    
    -- Prompt content
    system_prompt   TEXT NOT NULL,                        -- System message
    user_prompt_template TEXT NOT NULL,                   -- Template with {{placeholders}}
    
    -- LLM config
    model           VARCHAR(100) NOT NULL DEFAULT 'gpt-4o',
    temperature     NUMERIC(3,2) NOT NULL DEFAULT 0.30,
    max_tokens      INTEGER NOT NULL DEFAULT 4096,
    
    -- Output schema (JSON Schema for validation)
    output_schema   JSONB NOT NULL DEFAULT '{}',
    
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    version         INTEGER NOT NULL DEFAULT 1,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by      UUID REFERENCES users(id) ON DELETE SET NULL
);

-- 10.5 chapter_review_log — Teacher review actions
CREATE TABLE chapter_review_log (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    job_id          UUID NOT NULL REFERENCES transformation_jobs(id) ON DELETE CASCADE,
    chapter_id      UUID REFERENCES chapters(id) ON DELETE SET NULL,
    reviewer_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    action          VARCHAR(30) NOT NULL,                 -- 'approve', 'reject', 'edit_node', 'edit_question', 'regenerate', 'publish'
    
    entity_type     VARCHAR(50),                          -- 'node', 'step', 'question', 'chapter'
    entity_id       UUID,
    
    changes         JSONB NOT NULL DEFAULT '{}',          -- Before/after diff
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 10.6 assessment_generation_config — Per-job assessment settings
CREATE TABLE assessment_generation_config (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_id          UUID NOT NULL REFERENCES transformation_jobs(id) ON DELETE CASCADE,
    
    -- Question counts per concept
    quiz_per_concept        INTEGER NOT NULL DEFAULT 3,
    truefalse_per_concept   INTEGER NOT NULL DEFAULT 2,
    fillblank_per_concept   INTEGER NOT NULL DEFAULT 2,
    solve_per_concept       INTEGER NOT NULL DEFAULT 1,
    
    -- Worksheet settings
    generate_worksheet_basic    BOOLEAN NOT NULL DEFAULT TRUE,
    generate_worksheet_standard BOOLEAN NOT NULL DEFAULT TRUE,
    generate_worksheet_hots     BOOLEAN NOT NULL DEFAULT TRUE,
    generate_worksheet_final    BOOLEAN NOT NULL DEFAULT TRUE,
    worksheet_basic_count       INTEGER NOT NULL DEFAULT 10,
    worksheet_standard_count    INTEGER NOT NULL DEFAULT 10,
    worksheet_hots_count        INTEGER NOT NULL DEFAULT 5,
    worksheet_final_count       INTEGER NOT NULL DEFAULT 10,
    
    -- Game settings
    generate_rapid_fire      BOOLEAN NOT NULL DEFAULT TRUE,
    rapid_fire_count         INTEGER NOT NULL DEFAULT 10,
    generate_memory_match    BOOLEAN NOT NULL DEFAULT TRUE,
    
    -- Language
    lle_language             VARCHAR(10) NOT NULL DEFAULT 'hi-IN',
    generate_lle_words       BOOLEAN NOT NULL DEFAULT TRUE,
    
    created_at               TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 10.1 Indexes for New Tables

```sql
CREATE INDEX idx_source_material_uploader ON source_materials(uploaded_by) WHERE deleted_at IS NULL;
CREATE INDEX idx_source_material_type ON source_materials(material_type);
CREATE INDEX idx_source_material_grade ON source_materials(grade, board);

CREATE INDEX idx_transform_job_source ON transformation_jobs(source_material_id);
CREATE INDEX idx_transform_job_status ON transformation_jobs(status) WHERE status IN ('queued', 'extracting', 'analyzing', 'generating_content', 'generating_assessments', 'assembling', 'validating');
CREATE INDEX idx_transform_job_triggered ON transformation_jobs(triggered_by);
CREATE INDEX idx_transform_job_chapter ON transformation_jobs(generated_chapter_id) WHERE generated_chapter_id IS NOT NULL;

CREATE INDEX idx_transform_stages_job ON transformation_stages(job_id);
CREATE INDEX idx_transform_stages_stage ON transformation_stages(stage, status);

CREATE INDEX idx_prompt_template_name ON llm_prompt_templates(template_name) WHERE is_active = TRUE;
CREATE INDEX idx_prompt_template_type ON llm_prompt_templates(template_type) WHERE is_active = TRUE;

CREATE INDEX idx_review_log_job ON chapter_review_log(job_id);
CREATE INDEX idx_review_log_chapter ON chapter_review_log(chapter_id);
CREATE INDEX idx_review_log_reviewer ON chapter_review_log(reviewer_id);

CREATE INDEX idx_assess_config_job ON assessment_generation_config(job_id);
```

---

## 11. New API Endpoints Required

### 11.1 Source Material Upload

| Method | Endpoint | Roles | Description |
|---|---|---|---|
| POST | `/api/v1/content/upload` | teacher, org_admin | Upload PDF/image/text |
| GET | `/api/v1/content/materials` | teacher, org_admin | List uploaded materials |
| GET | `/api/v1/content/materials/:id` | teacher, org_admin | Get material detail |
| DELETE | `/api/v1/content/materials/:id` | teacher, org_admin | Delete source material |
| GET | `/api/v1/content/materials/:id/text` | teacher, org_admin | Get extracted text |

### 11.2 Transformation Jobs

| Method | Endpoint | Roles | Description |
|---|---|---|---|
| POST | `/api/v1/transform/start` | teacher, org_admin | Start transformation job |
| GET | `/api/v1/transform/jobs` | teacher, org_admin | List transformation jobs |
| GET | `/api/v1/transform/jobs/:id` | teacher, org_admin | Get job status and progress |
| GET | `/api/v1/transform/jobs/:id/stages` | teacher, org_admin | Get detailed stage logs |
| POST | `/api/v1/transform/jobs/:id/cancel` | teacher, org_admin | Cancel running job |
| POST | `/api/v1/transform/jobs/:id/retry` | teacher, org_admin | Retry failed job |
| POST | `/api/v1/transform/jobs/:id/regenerate/:nodeId` | teacher, org_admin | Regenerate a specific node |

### 11.3 Chapter Review

| Method | Endpoint | Roles | Description |
|---|---|---|---|
| GET | `/api/v1/transform/jobs/:id/review` | teacher, org_admin | Get generated chapter for review |
| PUT | `/api/v1/transform/jobs/:id/review/nodes/:nodeId` | teacher, org_admin | Edit a generated node |
| PUT | `/api/v1/transform/jobs/:id/review/questions/:questionId` | teacher, org_admin | Edit a generated question |
| POST | `/api/v1/transform/jobs/:id/preview` | teacher, org_admin | Generate preview HTML |
| POST | `/api/v1/transform/jobs/:id/publish` | teacher, org_admin | Approve and publish chapter |

### 11.4 Prompt Template Management

| Method | Endpoint | Roles | Description |
|---|---|---|---|
| GET | `/api/v1/admin/prompts` | super_admin | List prompt templates |
| PUT | `/api/v1/admin/prompts/:id` | super_admin | Update prompt template |

---

## 12. New Screens Required

(Summarized from Section 6 above)

| Screen | Purpose | Key Components |
|---|---|---|
| S45 Upload Source Material | Teacher uploads PDF/image/text | Drop zone, chapter info form, assessment config |
| S46 Transformation Progress | Live progress of AI job | Stepper, progress bar, stage list, estimated time |
| S47 Chapter Review — Node List | Review generated nodes | Node cards with edit/approve/warning status |
| S48 Chapter Review — Question Editor | Edit AI-generated questions | Question type tabs, editor forms, answer validation |
| S49 Chapter Preview | Preview rendered chapter | iframe with the child's view |
| S50 Chapter Publish | Approve and publish | Summary, validation results, publish button |

---

## 13. Revised Implementation Sequence

### 13.1 Corrected Critical Path

```
Phase 0 (Setup)
  │
  ├─> Phase 1 (Database — includes transformation tables)
  │     │
  │     ├─> Phase 2 (Auth)
  │     │     │
  │     │     └─> Phase 3 (Core API — includes upload/transformation endpoints)
  │     │           │
  │     │           ├─> Phase 4 (Chapter Mgmt & Validation — reduced)
  │     │           │
  │     │           └─> Phase 4B (AI Transformation Pipeline) ◄── NEW CRITICAL PHASE
  │     │                 │
  │     │                 ├─> Phase 5 (Sync)
  │     │                 │
  │     │                 └─> Phase 6 (Teacher Dashboard — includes upload/review)
  │     │                       │
  │     │                       └─> Phase 7 (Parent Dashboard)
  │     │
  │     └─> (parallel) Phase 8, 9, 10, 11
  │
  └─> (all) ─> Phase 12 (Testing — includes transformation tests)
                   │
                   └─> Phase 13 (Deployment — includes AI infrastructure)
                        │
                        └─> Phase 14 (Launch)
```

**Critical path:** Phase 0 → 1 → 2 → 3 → 4 → 4B → 6 → 12 → 13 → 14

Phase 4B is now on the critical path. It is the longest single phase (8 days) and the most technically complex. It should be the focus of the best engineer on the team.

### 13.2 What to Build FIRST (Answering the Original Question)

**Build order (first 5 things):**

1. **Database with transformation tables** — the schema must exist before anything else
2. **Auth system** — teachers need to log in before they can upload
3. **Upload API** — teachers need to upload PDFs before anything can be transformed
4. **AI transformation pipeline** — the core value: PDF → interactive HTML chapter with assessments
5. **Teacher review interface** — teachers need to review and approve before children see it

Everything else (sync, dashboards, gamification, Seva, economy) is built on top of this foundation.

---

**End of Gap Analysis & Corrections Document**

This document identifies 60+ gaps across six prior documents, provides the corrected content for each, specifies 6 new database tables, 15 new API endpoints, 6 new screens, a new implementation phase (Phase 4B — 8 days), and reverses the TRD Decision 8 (AI is now Phase 1, not Phase 3). The total effort estimate increases from 59 to 72 person-days. The critical path now includes the AI transformation pipeline as a first-class citizen.
