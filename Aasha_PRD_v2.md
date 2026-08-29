# Product Requirements Document (PRD)
## Aasha — AI-Powered Interactive Learning & Assessment Platform
### Transforming NCERT Textbooks into Gamified, Vernacular Learning Experiences

**Document Version:** 2.0
**Date:** August 2026
**Author:** Product Team, Annanth Aasha Foundation
**Status:** Draft for Review
**Changes from v1:** Refined LLE specification — CONN dictionary now includes modals, connectors, AND cue verbs/adverbs/adjectives that build understanding. Max 4 auto-annotations per sentence. Conversational Hindi (not dictionary-formal). Added TRD cross-references.

---

## 1. App Overview

### 1.1 What Is Aasha?

Aasha (meaning "hope") is an AI-powered learning platform that transforms static NCERT textbook chapters into self-contained, interactive HTML learning modules. Each chapter of the NCERT Class 8 Maths textbook (and eventually other subjects and grades) is ingested as a PDF and, through an agentic AI pipeline, is converted into a single, mobile-first HTML file that combines:

- **Tutor-style learning segments** with real-life analogies and warm English prose
- **Interactive visual simulations** built specifically for each chapter's mathematical concepts
- **Gamified step-by-step worked examples** that reveal one step at a time
- **Interactive assessments** derived directly from NCERT exercises, with immediate feedback, misconception explanations, and correct-method guidance
- **A vernacular language layer (LLE)** that provides Hindi word meanings on tap and auto-displays Hindi cue translations for modals, connectors, and understanding-building verbs/adverbs/adjectives — without breaking the English sentence structure or grammar

### 1.2 Product Vision

Every child in India — regardless of whether they have a private tutor, a smartphone, or a reliable internet connection — should have access to learning content that is as engaging as a game, as clear as a good teacher, and as rigorous as the NCERT textbook itself.

### 1.3 How It Works (Agentic Pipeline)

1. **Input:** NCERT textbook PDF (e.g., the Class 8 Maths textbook, 271 pages, 16 chapters)
2. **Chapter extraction:** The agentic system extracts per-chapter text from specific PDF page ranges
3. **Step generation:** The AI generates a structured STEPS array — a JSON data model where each step has a type (intro, tutor, formula, worked example, quiz, solve, simulation, milestone, rapid fire, memory match, completion) and chapter-specific content
4. **Engine assembly:** The STEPS array is combined with:
   - A CSS/HTML template (mobile-first, max-width 600px, self-contained)
   - The LLE (Language Layer Engine) — JavaScript that wraps every English word as a tappable element and auto-annotates understanding-cue words (modals, connectors, verbs, adverbs, adjectives) with inline Hindi
   - The App engine — JavaScript that renders each step type, handles navigation, gamification (XP, coins, levels, streaks), quiz shuffling, and step-by-step reveal
5. **Validation:** Every output is syntax-checked (`node --check`) and runtime-tested (simulated DOM environment in Node.js) to verify all steps render, no questions repeat, quiz options are shuffled, and worked-example navigation advances correctly
6. **Output:** A single HTML file with zero external dependencies — works offline on any phone

### 1.4 Current State

At the time of this PRD, the platform has been validated across 3 NCERT Maths chapters:
- **Chapter 1: Rational Numbers** (47 steps, 5 simulations)
- **Chapter 2: Linear Equations in One Variable** (66 steps, 2 simulations, all NCERT exercises 2.1–2.6)
- **Chapter 3: Understanding Quadrilaterals** (68 steps, 4 simulations, all NCERT exercises 3.1–3.4)

The agentic pipeline produces each chapter in a single build session: extract PDF text → generate STEPS → patch engine → assemble HTML → syntax check → runtime test → deliver.

---

## 2. Problem Statement

### 2.1 The Core Problem

Indian students in Classes 6–10 face a fundamental disconnect between how they are taught and how they actually learn:

**Textbooks are static; learning is not.** The NCERT textbook is excellent in content but is a wall of text and static diagrams. A student reading "the sum of the measures of the exterior angles of any polygon is 360 degrees" gets the fact but not the intuition. There is no way to drag a slider, watch angles change, or walk around a polygon and feel the total turn. The understanding gap between reading a fact and internalizing it remains wide.

**Tuition is expensive and uneven.** A good tutor explains why a formula works, uses real-life analogies, checks understanding mid-explanation, and gives instant feedback. Most students do not have access to such a tutor. Government school students especially lack personalized guidance.

**Assessment is detached from learning.** In the textbook, exercises come at the end of a chapter — separated from the learning. Students solve questions, get them wrong, and have nobody to explain why. There is no immediate feedback, no misconception diagnosis, no "here is the correct approach" nudge.

**Language is a barrier, not a bridge.** Maths is taught in English-medium schools, but most students think in their regional language. A student who reads "transposing" in English understands the maths but not the term. Existing solutions either (a) fully translate to Hindi, which breaks the English fluency the student needs for board exams, or (b) stay fully in English, which leaves conceptual gaps. Neither approach serves the student who lives between two languages.

**Gamification is absent.** Learning apps that exist in India either gamify trivially (points for completion) or focus on video content with passive consumption. There is no app that gamifies the actual NCERT exercise-solving process with XP, coins, streaks, levels, and interactive simulations — all derived from the textbook the student is already studying.

### 2.2 Why Now?

- **Agentic AI is mature enough** to reliably transform structured content (textbook PDFs) into structured interactive modules in a single pipeline, with validation gates that catch errors
- **Smartphone penetration in India** has crossed 600M+ users, with affordable devices under Rs 8,000 capable of rendering HTML5
- **NCERT is the national standard** — serving it well serves every CBSE student and many state board students
- **Post-COVID learning loss** has made personalized, interactive learning a national priority

---

## 3. Target Users

### 3.1 Primary User: The Student

**Persona: Arjun, 13 years old, Class 8, tier-2 city**

- Studies in an English-medium government or affordable private school
- Follows the NCERT curriculum
- Has a smartphone (shared with family) or a tablet at school
- Thinks primarily in Hindi/regional language but reads and writes in English
- Struggles with math terminology in English but understands the underlying concepts
- Does not have a private tutor; relies on textbook and classroom teaching
- Wants to do well in exams but finds the textbook boring and hard to self-study from
- Plays mobile games (Free Fire, Ludo King) and responds to XP, coins, streaks, and levels

**Key need:** "I want to understand the chapter, not just memorize it. And I want to practice real exam questions with someone telling me why I am wrong."

### 3.2 Secondary User: The Teacher

**Persona: Ms. Sharma, maths teacher, government school**

- Teaches 40+ students per class
- Has limited time for individual attention
- Wants to assign interactive homework that auto-checks itself
- Needs to know which students are struggling with which concepts
- Does not have time to create interactive content herself

### 3.3 Tertiary User: The Parent

**Persona: Suresh, father, daily wage earner**

- Wants his child to succeed but cannot help with homework
- Cannot afford tuition (Rs 500–2000/month is significant)
- Trusts the NCERT textbook as the gold standard
- Wants proof that his child is actually learning, not just scrolling

---

## 4. Core Features

### 4.1 PDF-to-Interactive-Module Pipeline (The Engine)

**What it does:** Takes an NCERT chapter PDF as input and produces a self-contained HTML learning module with simulations, worked examples, and gamified assessments — all aligned to that chapter's specific concepts.

**Key behaviors:**
- Extracts chapter text from specific PDF page ranges
- Generates a STEPS array where each step is typed (tutor, formula, worked example, quiz, solve, simulation, milestone, rapid fire, memory match, completion)
- Each step carries chapter-specific content — not generic templates
- The engine patches itself at build time: renderIntro uses `step.intro` (not hardcoded text), renderWE reveals one step at a time, renderQuiz shuffles options via Fisher-Yates, feedback is in English
- Validation gates: `node --check` syntax test + simulated DOM runtime test that verifies all steps render, no questions repeat, quiz positions are shuffled, and worked-example navigation advances

**Why it matters:** This is the core differentiator. No other product transforms a textbook PDF into an interactive, gamified, vernacular module in a single pipeline. The agentic approach means scaling to 16 chapters (a full textbook) or 160 chapters (10 textbooks) is a matter of running the pipeline, not building each module manually.

### 4.2 Vernacular Language Layer (LLE) — REVISED SPEC v2

**What it does:** Provides Hindi meaning for any English word on tap, and auto-displays Hindi cue translations for understanding-building words — modals, connectors, verbs, adverbs, and adjectives — inline, without breaking the English sentence structure.

**Key behaviors:**

#### 4.2.1 Auto-Annotation (CONN Dictionary)

The CONN dictionary contains ONLY understanding-cue words that build comprehension when shown in Hindi inline. These fall into three categories:

**A. Modals (help students understand possibility, necessity, obligation)**
```
can → सकता है          could → सकता था       will → करेगा
would → करेगा           shall → करेगा          should → करना चाहिए
must → ज़रूरी है         may → सकता है          might → शायद कर सके
need → ज़रूरत है         need to → ज़रूरत है करना  have to → करना पड़ता है
had to → करना पड़ा       ought to → करना चाहिए   be able to → सक्षम है
used to → पहले करता था   dare → हिम्मत करना     had better → बेहतर होगा
be going to → करने वाला है  be supposed to → करना अपेक्षित है
```

**B. Connectors (help students follow the logical flow — cause, contrast, condition, sequence, addition, conclusion)**
```
and → और               but → लेकिन            or → या
so → इसलिए              because → क्योंकि       also → भी
if → अगर                unless → जब तक नहीं     although → हालांकि
however → हालांकि        therefore → इसलिए       therefore → इसलिए
when → जब               while → जब             until → जब तक
before → पहले           after → बाद में         finally → अंत में
for example → उदाहरण के लिए   in conclusion → निष्कर्ष में
... (full list of 100+ connectors in TRD Section 5.2)
```

**C. Understanding-Cue Verbs, Adverbs, Adjectives (help students grasp the meaning of the sentence through key action/descriptor words, not just grammar connectors)**
```
equals → बराबर है        means → मतलब           gives → देता है
shows → दिखाता है        represents → दर्शाता है  consider → सोचो
observe → देखो          notice → ध्यान दो       remember → याद रखो
always → हमेशा           never → कभी नहीं        exactly → ठीक
directly → सीधे           similarly → इसी तरह     therefore → इसलिए
important → ज़रूरी        equal → बराबर          opposite → विपरीत
parallel → समान्तर       perpendicular → लंबवत    consecutive → क्रमागत
```

**Design rule:** These are NOT dictionary definitions. They are short, conversational Hindi cue words that give the student an instant understanding anchor. Example: "perpendicular" shows "(लंबवत)" inline — the student instantly knows it means "at 90 degrees" without breaking flow. They do NOT need to tap and read a formal definition.

#### 4.2.2 Max 4 Auto-Annotations Per Sentence

- Within any sentence (text between `.`, `!`, `?`, or `,`), at most **4** words from the CONN dictionary are auto-annotated with inline Hindi
- The 4 most important understanding-cue words get auto-annotated; the rest are wrapped as tappable elements (tap to reveal Hindi in a dialog)
- This prevents visual clutter — a sentence with 10 connectors would be unreadable if all were annotated. Max 4 keeps it clean
- The counter resets at every sentence boundary (period, exclamation, question mark, comma)

#### 4.2.3 Word-Level Tap (WM Dictionary)

- Every English word in the learning content is wrapped as a tappable element (`.wd` class)
- Tapping a word opens a dialog showing: the word, its Hindi translation, and a brief contextual description
- Example: tap "transposing" → dialog shows "Transposing / पक्षांतरण / Moving a term from one side of an equation to the other by changing its sign"
- Words that are in the CONN dictionary but exceeded the max-4 limit are still tappable (they show Hindi in the dialog, just without inline annotation)
- Math domain terms (equation, variable, diagonal, polygon, etc.) are tappable but NOT auto-annotated — the student taps only when they need the meaning
- If a word has no Hindi meaning available, the dialog shows "Hindi meaning not available" — graceful degradation

#### 4.2.4 Grammar Preservation

- The Hindi does NOT replace the English. It appears as a subtitle/below-word annotation in purple text
- The English sentence remains intact and readable
- The student reads English, gets Hindi support underneath, and builds English fluency while understanding concepts in their thinking language
- Hindi translations are conversational/colloquial, NOT dictionary-formal. Example: "because" → "क्योंकि" (not "कारण कि"). "should" → "करना चाहिए" (not "कर्तव्य है")

#### 4.2.5 Math Domain Terms Excluded from CONN

Mathematical terms like "equation," "variable," "diagonal," "polygon," "coefficient," "transposing," "numerator," "denominator" are explicitly NOT in the CONN dictionary. These are:
- Tappable (the student can tap to see Hindi meaning in the dialog)
- NOT auto-annotated inline (to avoid cluttering with translations the student likely already knows from class)

**Why it matters:** This is the single most unique feature in the ecosystem. Every edtech app in India either (a) is fully English (leaving concept gaps), (b) is fully Hindi/regional (breaking English fluency), or (c) has a separate "translation toggle" that switches the whole UI. Aasha's LLE is the only approach that layers Hindi ON TOP of English — like subtitles on a movie — so the student learns in both languages simultaneously. It mirrors how bilingual students actually think: English for the terms, Hindi for the connective understanding. The cue-word approach (modals + connectors + verbs/adverbs/adjectives) ensures the student gets understanding scaffolding exactly where they need it — at the logic/flow level — without being drowned in annotations.

### 4.3 Interactive Visual Simulations (Per-Chapter)

**What it does:** Each chapter gets its own set of interactive simulations built specifically for that chapter's mathematical concepts — not generic templates carried over from other chapters.

**Examples already built:**

| Chapter | Simulation | What the student does |
|---------|-----------|----------------------|
| Rational Numbers | Number Evolution | Watch integers → fractions → rational numbers appear on a number line |
| Rational Numbers | Closure Tester | Drag two rational numbers, see if their sum/product is also rational |
| Rational Numbers | Reciprocal Explorer | Slide a number, watch its reciprocal flip on the number line |
| Rational Numbers | Number Line Plotter | Place rational numbers on a number line with visual feedback |
| Rational Numbers | Between-Two-Numbers Finder | Find rational numbers between any two given numbers |
| Linear Equations | Balance Scale | Drag a slider to change x, watch a physical balance scale tip until the equation balances (e.g., 3x + 2 = 11) |
| Linear Equations | Word Problem Builder | Step through how a real-life word problem becomes an equation, one stage at a time |
| Quadrilaterals | Polygon Builder | Add/remove sides, watch the polygon change from triangle to decagon, see diagonals count and angle sum update live |
| Quadrilaterals | Angle Sum Explorer | Drag 4 sliders for the 4 angles of a quadrilateral, watch the sum always equal 360° |
| Quadrilaterals | Exterior Angle Walker | Walk around different polygons vertex by vertex, see the total exterior turn always reach 360° |
| Quadrilaterals | Quadrilateral Explorer | Tap shape buttons (trapezium, kite, parallelogram, rhombus, rectangle, square), see the shape draw on canvas and properties update |

**Design principles for simulations:**
- Every simulation is discovery-based: the student manipulates something and observes the mathematical truth emerge (not a video they watch passively)
- Every simulation uses a different equation/scenario from the worked examples and assessments (no question overlap)
- Every simulation is built with HTML5 Canvas — no external libraries, no dependencies, works offline
- Every simulation has a "discovery box" that updates with real-time feedback ("Balanced! x = 3 is the solution!" or "Sum = 360° — Perfect!")

### 4.4 Step-by-Step Worked Examples

**What it does:** NCERT textbook examples (the "Example N" solved problems) are presented one step at a time. The student clicks "Next Step" to reveal each step, building understanding incrementally rather than seeing the full solution at once.

**Key behaviors:**
- Each worked example has 2–5 steps, revealed one at a time
- The current step is highlighted (golden background, slide-in animation)
- The button shows progress: "Next Step (2 of 3)"
- After the last step, the button turns green and says "Continue" to advance to the next learning segment
- The answer is shown in a green final box only after all steps are revealed
- **Navigation is verified:** the engine resets the step counter and advances to the next step in the journey — no infinite loops

### 4.5 Gamified Assessment System

**What it does:** NCERT exercises (the "Exercise X.Y QZ" problems at the end of each section) are converted into interactive, gamified assessments with immediate feedback.

**Step types:**

| Type | Description | Gamification |
|------|------------|-------------|
| **Solve** | Step-by-step input-based solving. Student types answers at each step. Immediate per-step feedback. | +5 XP per correct solve, +2 coins |
| **Quiz** | Multiple choice with 4 options. Options are shuffled (Fisher-Yates). | +5 XP, +2 coins, streak counter |
| **Rapid Fire** | Timed quiz (30 seconds) with multiple questions. Tests speed and recall. | Bonus XP based on speed |
| **Memory Match** | Match English math terms with Hindi translations. | +10 XP per match |
| **Milestone** | Celebration screen after completing a section. | Level-up check, confetti |
| **Completion** | Final screen with total XP, coins, stars. | Permanent record |

**Key behaviors:**
- **No repeated questions:** Worked examples use NCERT "Example N" questions. Assessments use NCERT "Exercise X.Y QZ" questions. Simulations use custom scenarios. NEVER overlap.
- **Quiz option shuffling:** Correct answer position is randomized every render via Fisher-Yates.
- **Misconception feedback:** When wrong: (1) "Not quite." (2) "Why wrong: [explanation]" (3) "Correct approach: [right method]"
- **Retry mechanism:** After 8 seconds, wrong quiz answers reset.
- **English feedback:** All feedback is in proper English — not Hinglish.

### 4.6 Learning Checkpoints (Mini-Quizzes During Learning)

**What it does:** Quick-check quizzes inserted BETWEEN worked examples — not just at the end. Tests understanding before the full assessment.

**Pattern:** Learn → Example → Quick Check → Learn more → Example → Quick Check → Full Assessment

### 4.7 Learning-to-Assessment Ratio (3:1 Target)

Learning steps (tutor, formula, real-life example, simulation, worked example) should significantly outnumber assessment steps. Target: 3:1 or higher.

### 4.8 Gamification Layer (XP, Coins, Levels, Streaks)

- **XP:** Earned for every correct answer, every completed step, every milestone.
- **Coins:** Earned alongside XP. Spendable in Shop (v2).
- **Levels:** XP thresholds trigger level-ups with dialog and confetti.
- **Streaks:** Consecutive correct answers. Breaking resets to 0.
- **Progress Bar:** Visual progress at the top.

### 4.9 Self-Contained, Offline-Capable, Mobile-First

- Single HTML file, zero external dependencies
- Max-width: 600px (mobile-first)
- All graphics via HTML5 Canvas (no image files)
- All text/data embedded in JavaScript STEPS array
- Works fully offline once loaded

---

## 5. User Stories

### 5.1 Student Stories

**US-1: Chapter Learning**
> As a Class 8 student, I want to open a chapter module on my phone and learn the entire chapter interactively — with explanations, simulations, and worked examples — so that I can understand the concepts before my school exam.

**US-2: Hindi Support**
> As a student who thinks in Hindi but studies in English medium, I want to tap any difficult English word and see its Hindi meaning instantly, without switching to a separate dictionary app, so that my reading flow is not broken.

**US-3: Understanding-Cue Words**
> As a student, I want modals, connectors, and key verbs/adverbs to automatically show their Hindi meaning inline (max 4 per sentence), so that I can follow the logic of the explanation even if my English vocabulary is limited. The cue words should use normal conversational Hindi, not formal dictionary Hindi.

**US-4: Step-by-Step Examples**
> As a student, I want worked examples to reveal one step at a time, so that I can try to predict the next step before seeing it.

**US-5: Immediate Feedback on Exercises**
> As a student, I want to solve NCERT exercises and get instant feedback — not just "right" or "wrong" but WHY I am wrong and HOW to do it correctly.

**US-6: Interactive Simulations**
> As a student, I want to drag sliders and tap buttons to explore mathematical concepts visually, so that I develop intuition for the formulas instead of memorizing them.

**US-7: Gamified Progress**
> As a student, I want to earn XP, coins, and levels as I learn, so that I feel motivated to complete the chapter.

**US-8: No Repeated Questions**
> As a student, I want every worked example and every assessment question to be different, so that I am actually learning new problem-solving approaches.

**US-9: Checkpoints During Learning**
> As a student, I want quick quizzes BETWEEN examples, so that I can check if I understood a concept before moving on.

**US-10: Offline Access**
> As a student with limited internet, I want to download a chapter module once and use it offline at home.

### 5.2 Teacher Stories

**US-11: Assign Interactive Homework**
> As a teacher, I want to assign a chapter module as homework, so that my students engage with the NCERT content interactively.

**US-12: Identify Struggling Concepts** (v2)
> As a teacher, I want to see which questions my students got wrong most often.

### 5.3 Parent Stories

**US-13: Proof of Learning**
> As a parent, I want to see my child's XP, coins, level, and chapter completion status, so that I know they are actually learning.

---

## 6. MVP Scope (Version 1)

### 6.1 In Scope for V1

| Feature | Status | Details |
|---------|--------|---------|
| PDF-to-HTML chapter pipeline | Done | Ingests NCERT PDF, extracts chapters, generates STEPS, assembles HTML |
| LLE (Vernacular Language Layer) | Done | CONN = modals + connectors + cue verbs/adverbs/adjectives; max 4 auto-annotated per sentence; conversational Hindi; tap-for-Hindi on all words |
| Step-by-step worked examples | Done | One-step-at-a-time reveal, progress indicator, navigation verified |
| Gamified assessments (solve, quiz) | Done | NCERT exercises, shuffled options, misconception feedback, English feedback |
| Interactive simulations (per-chapter) | Done | Canvas-based, discovery-driven, concept-specific (11 sims across 3 chapters) |
| Learning checkpoints | Done | Mini-quizzes between examples, 7–8 per chapter |
| Gamification (XP, coins, levels, streaks) | Done | Top-bar stats, level-up dialog, confetti, progress bar |
| Rapid Fire assessment | Done | Timed quiz, 8 questions, bonus XP |
| Memory Match assessment | Done | English-Hindi term matching, 8 pairs |
| Milestone celebrations | Done | XP rewards, next-step preview |
| Offline self-contained HTML | Done | Zero dependencies, single file, mobile-first |
| Syntax + runtime validation | Done | node --check + simulated DOM test for all steps |
| Question uniqueness verification | Done | Duplicate check in build pipeline |
| Quiz option shuffling | Done | Fisher-Yates, verified by runtime test |
| NCERT Class 8 Maths Chapters 1–3 | Done | Rational Numbers, Linear Equations, Quadrilaterals |
| NCERT Class 8 Maths Chapters 4–16 | Planned | Practical Geometry through Playing with Numbers |
| Progress persistence (localStorage) | Planned | Save XP, coins, completed chapters across sessions |

### 6.2 MVP Success Criteria

1. **Content completeness:** All 16 chapters of NCERT Class 8 Maths converted
2. **Quality gates:** Every chapter passes syntax validation and runtime test
3. **Offline capability:** Every chapter module is a single HTML file under 150KB
4. **Learning-to-assessment ratio:** At least 2.5:1 per chapter
5. **Simulations per chapter:** At least 2 concept-specific simulations
6. **LLE coverage:** CONN dictionary has 100+ entries; every chapter's content has tappable words and max-4 auto-annotations
7. **No Hinglish:** All feedback, instructions, and tutor text is in proper English

---

## 7. Success Metrics

### 7.1 Learning Effectiveness

| Metric | Target | Measurement |
|--------|--------|------------|
| Chapter completion rate | > 60% | Track steps completed vs. total steps |
| Pre/Post score improvement | > 30% | Compare first checkpoint to rapid fire |
| First-attempt correctness | > 50% | Track solve/quiz first-attempt success |
| Misconception recovery | > 40% | Track retry success after feedback |

### 7.2 Engagement

| Metric | Target | Measurement |
|--------|--------|------------|
| Average time per chapter | 25–40 min | Time from first step to completion |
| Sessions per week | 3+ | Module opens per student per week |
| Streak frequency | > 30% achieve 3+ streaks | Track streak distribution |
| Simulation interaction | > 70% interact | Track slider/button clicks |

### 7.3 Scale

| Metric | Target | Measurement |
|--------|--------|------------|
| Chapters published | 16 (full Class 8 Maths) | Count of delivered HTML modules |
| Build time per chapter | < 10 minutes | Pipeline execution time |
| File size per chapter | < 150KB | Measured file size |
| Validation pass rate | 100% | node --check + runtime test |

---

## 8. Features to Avoid in Version 1

### 8.1 Avoid: Video Content
Video is expensive, bandwidth-heavy, and shifts students from active to passive. The Aasha philosophy is interactive engagement. Revisit V3 for short embedded animated GIFs.

### 8.2 Avoid: Social Features (Leaderboards, Friends, Chat)
Adds massive complexity (auth, real-time DB, moderation, privacy for minors). Shifts motivation from intrinsic to extrinsic. V2 for class-level leaderboards.

### 8.3 Avoid: Adaptive Learning Paths
Requires backend, real-time analytics, recommendation algorithms. V1 uses NCERT's fixed linear path. V2+ for adaptation.

### 8.4 Avoid: User Authentication
Adds friction, requires backend, privacy concerns for minors. V1 uses localStorage. V2 for cross-device sync.

### 8.5 Avoid: Multi-Subject / Multi-Grade
Master Class 8 Maths first. V2 for Science, V3 for other grades.

### 8.6 Avoid: AI-Generated Content Without Validation
Validation gates are permanent. No skipping. The worked-example navigation loop bug proved why.

### 8.7 Avoid: Real-Time Analytics Dashboard
Requires backend. V1 is self-contained HTML. V2 with accounts and backend.

### 8.8 Avoid: Content Authoring UI
The agentic pipeline is the creation tool. V3 if teachers want to customize.

### 8.9 Avoid: Paid Features / Monetization
Validate learning effectiveness first. Post-V2 for business model.

### 8.10 Avoid: External Integrations
Self-contained HTML shared via WhatsApp/email/USB. V2 for platform integration.

---

## 9. Key Risks and Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|-----------|------------|
| AI-generated content has factual errors | High | Medium | Validation gates + manual review. Content sourced from NCERT. |
| Worked-example navigation bugs | High | Low (fixed) | Runtime test verifies WE navigation advances sIdx. |
| Question repetition across steps | Medium | Low (fixed) | Duplicate-question check in build pipeline. |
| LLE CONN dictionary too aggressive (too many annotations) | Medium — visual clutter | Medium | Max 4 per sentence limit. Math terms excluded from CONN. |
| LLE Hindi too formal/dictionary-style | Low — students don't relate | Low | Use conversational Hindi ("सकता है" not "समर्थ है"). |
| File size grows too large | Low | Low | Monitor. Split if > 150KB. |

---

## 10. Roadmap (Post-V1)

### V1.1 (Immediate)
- NCERT Class 8 Maths Chapters 4–16 (complete the textbook)
- localStorage progress persistence
- Profile screen (name, avatar, badges)

### V2
- Teacher dashboard
- Student accounts (cross-device sync)
- Adaptive learning paths
- NCERT Class 8 Science

### V3
- Multi-grade (Classes 6–10)
- Multi-subject
- Content authoring UI for teachers
- Regional languages beyond Hindi (Tamil, Telugu, Bengali, Marathi)
- B2B distribution (government school partnerships)

---

## 11. Glossary

| Term | Definition |
|------|-----------|
| **LLE** | Language Layer Engine. JavaScript system providing tappable Hindi word meanings and auto-annotating understanding-cue words (modals, connectors, verbs, adverbs, adjectives) inline. Max 4 per sentence. |
| **CONN** | The dictionary of understanding-cue words that get auto-annotated inline. Contains modals, connectors, and cue verbs/adverbs/adjectives — NOT math domain terms. |
| **WM** | Word Dictionary. The full Hindi-meaning dictionary used for tap-to-reveal. Larger than CONN. |
| **STEPS** | JSON array defining the entire learning journey of a chapter. |
| **WE** | Worked Example. Step type revealing NCERT examples one step at a time. |
| **Fisher-Yates** | Shuffle algorithm for randomizing quiz option order. |
| **NCERT** | National Council of Educational Research and Training. |
| **MVP** | Minimum Viable Product. |

---

*This PRD v2 reflects the refined LLE specification: CONN dictionary contains modals + connectors + understanding-cue verbs/adverbs/adjectives, max 4 auto-annotations per sentence, conversational Hindi. See TRD for full technical implementation details.*

*Refer to the TRD (Technical Requirements Document) for architecture, data models, engine patches, and validation specifications.*
