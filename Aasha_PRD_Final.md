# Product Requirements Document — Aasha

**Product:** Aasha — Safe Learning. Real Impact.  
**Organisation:** Annanth Aasha Foundation  
**Document Type:** Product Requirements Document (PRD)  
**Version:** 2.0  
**Date:** August 2026  
**Status:** Approved for Implementation  

---

## 1. App Overview

Aasha is a child-centred learning platform that transforms a child's existing school-book content into interactive, gamified, bilingual learning experiences. It does not replace the school, teacher, curriculum, or textbook. It builds a learning layer around prescribed school-book content that helps the child move from reading to understanding to demonstrating mastery.

The product is delivered as self-contained HTML files — one per chapter — that work fully offline with zero internet dependency. Each chapter file contains the complete learning engine: concepts, visuals, interactive activities, worked examples, practice questions, assessments, gamification, and progress tracking.

Three chapters are currently built and tested:

| Chapter | Class | Subject | Nodes | Steps | Question Types |
|---|---|---|---|---|---|
| Understanding Quadrilaterals | 8 | NCERT Math | 22 | 201 | Quiz, TF, FB, WE, Solve, Rapid Fire, Memory Match |
| Comparing Quantities | 7 | NCERT Math | 11 | 134 | Quiz, TF, FB, WE, Solve |
| Fractions | 5–8 | NCERT Math | 20 | 193 | Quiz, TF, FB, WE, Solve |

The larger Aasha vision extends learning beyond the screen through Eco Seva, Jal Seva, and the Aasha Economy, allowing children to experience effort, responsibility, saving, patience, choice, and contribution.

### The Aasha Loop

```
📖 My School Book → 🧠 I Understand → 👀 I See It → 🖐️ I Explore It →
✏️ I Practise → 💡 I Learn From My Mistake → ✅ I Master It →
⭐ I Grow → 🪙 I Earn → 🌱 I Act → 💧 I Serve → 🐷 I Save →
🎁 I Choose → 🤝 I Contribute → 🌱 Another Child Gets an Opportunity →
∞ Infinite Hope
```

---

## 2. Product Vision

> **To make every child's own school book understandable, interactive, and meaningful — and to help learning become action, responsibility, and hope beyond the page.**

The child should not have to leave their curriculum to find a different curriculum. The book stays. The learning experience changes.

AI is an enabling capability, not the product definition. The child is the purpose.

---

## 3. Target Users

### 3.1 Primary User — The Child

School-going children aged 10–14 (Class 5–8) in Indian government schools and low-income private schools, using shared Android smartphones with intermittent or no internet.

Key characteristics:
- Shared device (multiple children, one phone)
- Limited storage (files must be small, under 500 KB each)
- No reliable internet (must work fully offline)
- Hindi/English bilingual (regional language support via LLE)
- Low digital literacy (UI must be intuitive)
- May not have access to continuous one-to-one tutoring
- Varying levels of conceptual understanding, language familiarity, confidence, and learning pace

### 3.2 Secondary User — Teacher / Facilitator

Supports children, observes progress, identifies difficulties, encourages learning, helps verify real-world activities, and intervenes when a child needs human support. Aasha should make the teacher's work more informed, not replace the teacher.

### 3.3 Secondary User — Parent / Guardian

Should understand what their child is learning, whether the child is progressing, where support is needed, and what meaningful activities the child has undertaken. The parent experience emphasises understanding and progress rather than surveillance.

### 3.4 Programme / Community Team

Needs to understand participation, learning progress, concept mastery, engagement, real-world activities, verification, reward behaviour, and cohort outcomes.

---

## 4. Problem Statement

### 4.1 The Education Problem

A child can read a textbook and still not understand it. A child can read every word, complete homework, memorise definitions, and prepare for an examination — and still fail to develop genuine understanding.

The problem becomes particularly serious when:
- The language is unfamiliar
- Terminology is difficult
- The concept is abstract
- The child needs a visual representation
- The child needs to see an example
- The child needs to manipulate something
- The child needs more practice
- The child makes a misconception and does not know why the answer is wrong
- There is nobody available to explain the same concept in another way

### 4.2 The Product Problem

> **How can we turn the child's existing school-book content from something they are expected to read into something they can actually understand, practise, apply, and remember?**

### 4.3 The Engagement Problem

> **How can learning become connected to the child's real life instead of ending when the textbook page ends?**

### 4.4 The Values Problem

> **How can children experience effort, responsibility, saving, and contribution without turning education into a transactional "study and get paid" system?**

---

## 5. Product Philosophy

| Principle | Description |
|---|---|
| School-book first | The child's prescribed school book remains the educational foundation. Aasha is an enhancement layer. |
| Understanding before completion | Completing a lesson is not the outcome. The outcome is: the child understands the concept. |
| Different children need different explanations | A child may need a picture, an example, a simpler explanation, a worked solution, a practical interaction, another attempt, or a familiar language bridge. |
| Language is a bridge, not a replacement | Aasha helps children understand academic English through familiar language (Hindi). This is a bridge into English, not a replacement for English. |
| Learning should be active | Children should tap, drag, compare, manipulate, solve, predict, answer, correct, and discover. |
| Mistakes should become learning opportunities | A wrong answer should explain the misconception, not just say "Wrong." |
| Learning should eventually leave the page | Learning should connect with nature, water, community, responsibility, service, patience, saving, and contribution. |
| Gamification must support learning | Gamification reinforces effort and progress toward mastery. It must not become "click → collect points → forget the lesson." |
| Offline by design | Learning should not require continuous internet access. This is part of the product's accessibility philosophy. |

---

## 6. Core Features

### 6.1 Learning Engine

The fundamental content structure is: **Class → Subject → Chapter → Concept**.

Each concept is broken into a deliberate learning experience:

1. **Introduction** — What is this concept and why does it matter?
2. **Simple explanation** — Plain-language text with LLE word highlighting
3. **Visual representation** — Canvas-based diagrams (fraction bars, pizza models, geometric shapes, ratio bars, percentage grids, profit/loss bars, interest charts)
4. **Interactive discovery** — Drag vertices, tap pizza slices, move sliders, match equivalent fractions
5. **Worked example** — Step-by-step solved problem (Question → Step 1 → Step 2 → Answer)
6. **Guided practice** — Interactive solve problems where the child inputs each step
7. **Independent practice** — Quiz, fill-in-the-blank, true/false questions
8. **Conceptual feedback** — Wrong answers explain the misconception, not just the correct answer
9. **Concept check** — Assessment to verify understanding
10. **Progress milestone** — XP, coins, badges, level-up, celebration

### 6.2 Language Learning Engine (LLE)

A real-time bilingual text processor that:
- Highlights English words and reveals Hindi meanings on tap
- Shows inline Hindi for connective words (if, because, therefore, however, etc.)
- Processes fractions visually (½ renders as stacked numerator/denominator)
- Is integrated into every text element: explanations, questions, feedback, worked examples

The LLE word map currently contains 78+ English→Hindi translations. The language bridge is designed so children encounter academic English, understand it through familiar language, and then return to the original English — building language independence over time.

### 6.3 Question Types

| Type | Description | Implemented In |
|---|---|---|
| Quiz (MCQ) | Multiple choice with 4 options, targeted misconception feedback | All chapters |
| True/False | Binary with difficulty badges (easy/medium/hard) and explanation | All chapters |
| Fill-in-the-Blank | Typed answer with flexible matching (int, float, string, whitespace-insensitive) | All chapters |
| Worked Example | Step-by-step solved problem with progressive reveal | All chapters |
| Solve (Interactive) | Multi-step problem where child inputs each step (MCQ or text input) | All chapters |
| Interactive Canvas | Drag vertices, tap slices, move sliders | Quadrilaterals |
| Rapid Fire | 10 questions, 10-second timer each, auto-advancing, streak bonus | Quadrilaterals |
| Memory Match | 4×4 card-flipping game matching properties to names | Quadrilaterals |

### 6.4 Gamification System

| Element | Description |
|---|---|
| XP | Awarded for correct answers, speed bonuses, concept completion |
| Coins | Earned at concept milestones, spent in the Gem Shop |
| Levels | 6 levels: Beginner → Learner → Scholar → Expert → Master → Grandmaster |
| Badges | Achievement milestones (First Steps, Ratio Master, Profit Pro, etc.) |
| Streaks | Consecutive correct answers build streak; combo bonus XP |
| Speed Bonus | Answer quiz within 3s = +3 XP, within 6s = +2, within 10s = +1 |
| Confetti | Celebration animation on correct answers and milestones |
| Sound | Web Audio API tones for correct, wrong, level-up, completion |
| Progress Ring | SVG circular progress indicator showing chapter completion % |
| Level-Up Overlay | Full-screen celebration with particle burst |

### 6.5 Gem Shop (Economy)

A spend-and-earn system where coins have utility:

| Item | Cost | Effect |
|---|---|---|
| Hint Token | 20 coins | Eliminates 2 wrong options in next quiz |
| Streak Freeze | 30 coins | Protects streak from 1 wrong answer |
| Ocean Theme | 50 coins | Changes app colour scheme to blue/teal |
| Forest Theme | 50 coins | Changes app colour scheme to green |
| Sunset Theme | 50 coins | Changes app colour scheme to orange/pink |
| Gold Avatar Frame | 100 coins | Decorative gold frame on profile |
| Star Avatar Frame | 150 coins | Decorative star frame on profile |

The economy teaches: effort → earning → saving → patience → choice.

### 6.6 Navigation

| Feature | Description |
|---|---|
| Back button | Navigate to previous step from any screen |
| Restart | Full chapter reset from completion screen |
| Profile picker | Multiple named profiles per device, each with saved progress |
| Resume | Continue from last position on next open |
| Chapter map | Visual overview of all concepts (locked/unlocked) |

### 6.7 Offline Architecture

Each chapter is a single self-contained HTML file with:
- All CSS inline (no external stylesheets)
- All JavaScript inline (no external scripts)
- All visuals via Canvas API or CSS/SVG (no image assets, except a base64 logo)
- All sounds via Web Audio API oscillators (no audio files)
- All state in `localStorage` and `sessionStorage`
- Zero network dependency — works on airplane mode
- File size under 500 KB

### 6.8 Worksheet Assessment

Each chapter includes worksheet-based assessments drawn from NCERT worksheets:

| Level | Description |
|---|---|
| Basic | Section A (MCQs + fill-blanks), Section B–E (step-by-step worked solutions) |
| Standard | Intermediate MCQs, fill-blanks, and multi-step problems |
| HOTS | Higher-order thinking problems with proof-style worked solutions |
| Final Mixed | Cross-level comprehensive assessment |

All worksheet problems include step-by-step worked solutions shown to the child.

### 6.9 Activity Verification (Vision — Phase 3)

Meaningful-action activities should not rely solely on self-reporting. The broader product direction includes:
- Photograph evidence
- Location/geotag
- Activity context
- Facilitator/community verification

Principle: **Reward meaningful verified action, not digital claims.**

### 6.10 Eco Seva & Jal Seva (Vision — Phase 3)

Learning connects with real-world responsibility:
- **Eco Seva**: Caring for plants, planting, community cleanliness, environmental activity
- **Jal Seva**: Responsible water-related action

Purpose: Teach children that what they learn can influence how they act.

---

## 7. User Stories

### 7.1 Child — Understanding

| ID | Story |
|---|---|
| US-01 | As a child, I want to start from the chapter I am studying in school so that Aasha helps me understand what I already need to learn. |
| US-02 | As a child, I want a difficult concept explained in a simpler way so that I don't have to memorise something I don't understand. |
| US-03 | As a child, I want to see a visual representation of a difficult idea so that I can understand something hard to imagine from text. |
| US-04 | As a child, I want to interact with a concept so that I can discover how it works. |
| US-05 | As a child, I want to see a worked example before solving a difficult problem myself. |

### 7.2 Child — Practice

| ID | Story |
|---|---|
| US-06 | As a child, I want to practise a concept immediately after learning it so that I can check whether I understood it. |
| US-07 | As a child, I want to know why my answer is wrong so that I can correct my thinking. |
| US-08 | As a child, I want another explanation or example when I do not understand the first one. |
| US-09 | As a child, I want to solve increasingly difficult questions so that I can build confidence. |

### 7.3 Child — Language

| ID | Story |
|---|---|
| US-10 | As a child, I want help with difficult English words so that language does not prevent me from understanding the concept. |
| US-11 | As a child, I want familiar-language support without losing exposure to the English I need to learn. |

### 7.4 Child — Progress

| ID | Story |
|---|---|
| US-12 | As a child, I want to know how far I have progressed through a chapter. |
| US-13 | As a child, I want my learning progress to remain associated with me so that I can continue where I stopped. |
| US-14 | As a child, I want to earn XP and achievements for meaningful learning progress. |

### 7.5 Child — Gamification

| ID | Story |
|---|---|
| US-14a | As a child, I want timed quiz questions with speed bonuses so that I feel urgency and excitement. |
| US-14b | As a child, I want rapid-fire rounds so that I can test my recall speed. |
| US-14c | As a child, I want memory match games so that I can learn through play. |
| US-14d | As a child, I want to spend my earned coins in a shop so that my effort has tangible value. |

### 7.6 Child — Navigation

| ID | Story |
|---|---|
| US-14e | As a child, I want a back button so that I can revisit a concept I did not fully understand. |
| US-14f | As a child, I want to restart a chapter after completing it so that I can practise again. |

### 7.7 Child — Aasha Economy

| ID | Story |
|---|---|
| US-15 | As a child, I want to earn Aasha Coins through meaningful effort so that I can experience the value of what I do. |
| US-16 | As a child, I want to save my coins so that I can make a larger or more meaningful choice later. |
| US-17 | As a child, I want to choose how to use my saved coins rather than simply receive a predetermined reward. |

### 7.8 Child — Seva

| ID | Story |
|---|---|
| US-18 | As a child, I want opportunities to use what I learn in real life. |
| US-19 | As a child, I want to participate in Eco Seva and Jal Seva activities. |
| US-20 | As a child, I want meaningful action to be recognised. |
| US-21 | As a child, I want my activity to be verified fairly so that recognition has meaning. |

### 7.9 Teacher / Facilitator

| ID | Story |
|---|---|
| US-22 | As a facilitator, I want to know which concepts children are struggling with so that I can provide support. |
| US-23 | As a facilitator, I want to see learning progress without having to manually track every activity. |
| US-24 | As a facilitator, I want to review submitted Seva activities so that meaningful actions can be verified. |

### 7.10 Parent

| ID | Story |
|---|---|
| US-25 | As a parent, I want to understand my child's learning progress rather than only seeing marks. |
| US-26 | As a parent, I want confidence that the platform encourages learning and responsibility rather than excessive screen consumption. |

### 7.11 Programme Team

| ID | Story |
|---|---|
| US-27 | As a programme team member, I want to measure whether children actually understand concepts after using Aasha. |
| US-28 | As a programme team member, I want evidence of meaningful activities rather than only participation claims. |
| US-29 | As a programme team member, I want to compare learning and behaviour before and after introducing Aasha. |

---

## 8. MVP Scope

The MVP proves the core hypothesis:

> **Can Aasha make prescribed school-book learning more understandable, interactive, and motivating for children?**

### 8.1 MVP — Must Have

| Component | Status |
|---|---|
| Class → Subject → Chapter → Concept structure | ✅ Built |
| Concept introduction with icon, title, subtitle | ✅ Built |
| Simple explanations with LLE word highlighting | ✅ Built |
| Visual explanations (Canvas: fraction bars, shapes, grids, charts) | ✅ Built |
| Interactive learning (drag, tap, slider, matching) | ✅ Built |
| Worked examples (step-by-step progressive reveal) | ✅ Built |
| Practice: quiz, fill-blank, true/false | ✅ Built |
| Assessment: final chapter assessment + worksheet levels | ✅ Built |
| Conceptual feedback (misconception explanations) | ✅ Built |
| Child profile with saved progress | ✅ Built |
| XP, levels, coins, badges, streaks | ✅ Built |
| Chapter progress bar/ring | ✅ Built |
| Offline self-contained HTML files | ✅ Built |
| Timed quiz with speed bonus XP | ✅ Built |
| Rapid fire round | ✅ Built |
| Memory match game | ✅ Built |
| Gem shop (hint tokens, streak freeze, themes) | ✅ Built |
| Back button + restart | ✅ Built |
| English content with Hindi word-level support | ✅ Built |
| 3 chapters: Fractions, Comparing Quantities, Quadrilaterals | ✅ Built |

### 8.2 MVP — Should Have (Next Sprint)

| Component | Status |
|---|---|
| Daily challenge (date-based seed) | Designed — TRD ready |
| Daily streak tracking | Designed |
| Chapter map with lock/unlock | Not built |
| Concept mastery visualization | Not built |
| Multiple child profiles on one device | ✅ Built (profile picker) |
| Basic facilitator view | Not built |
| Basic learning analytics | Not built |
| Initial Eco Seva activity | Not built |
| Initial Jal Seva activity | Not built |
| Basic activity evidence | Not built |
| Additional chapters (more NCERT topics) | Not built |

### 8.3 MVP — Later / Phase 2+

| Component |
|---|
| Full Aasha Store |
| Full saving experience |
| Rich Seva system |
| Advanced activity verification (photo, geotag) |
| Parent dashboard |
| Teacher dashboard |
| Institution dashboard |
| AI-powered personalised learning |
| AI misconception detection |
| AI-generated alternative explanations |
| Broader curriculum coverage |
| Extensive regional language support |

### 8.4 Features to Avoid in Version 1

| Feature | Why Avoid |
|---|---|
| Generic "Ask Anything" AI chatbot | Start with curriculum, not open chat |
| Open internet browser | Protect children from uncontrolled exposure |
| Social network (profiles, messaging, feeds) | Not required to validate learning |
| Competitive public leaderboards | Encourage personal progress, not ranking |
| Cash-for-study mechanics | Contradicts Aasha Economy values |
| Excessive gamification (gambling, random rewards) | Must support learning, not exploit |
| Full e-commerce marketplace | Store is educational, not retail |
| School management system (attendance, fees, HR) | Aasha is a learning layer, not an ERP |
| Full parent surveillance platform | Emphasise understanding, not policing |
| AI avatar as a product goal | A talking avatar ≠ better learning |
| Entire curriculum at once | Depth before breadth — "the chapters we have actually work" |
| Complex hardware requirements | Must work on devices children actually have |

---

## 9. Product Layers

| Layer | Purpose | Status |
|---|---|---|
| Layer 1 — Learning | Understand: school book, concepts, explanation, visualisation, language bridge, interaction | ✅ Built |
| Layer 2 — Mastery | Demonstrate: practice, mistake feedback, assessment, progress, mastery, XP, levels, achievements | ✅ Built |
| Layer 3 — Life | Act: Eco Seva, Jal Seva, Aasha Coins, saving, choice, contribution, enable the next child | Vision — Phase 3+ |

---

## 10. Success Metrics

### 10.1 North Star Metric

> **Percentage of participating children who demonstrate measurable improvement in understanding of the concepts they study through Aasha.**

This is more important than downloads, screen time, lessons opened, or coins earned.

### 10.2 Learning Metrics

| Metric | Description |
|---|---|
| Concept Understanding | % of children demonstrating understanding after completing a concept |
| Pre/Post Improvement | Difference between baseline and post-Aasha understanding |
| First-Attempt Success | How often children solve a practice problem correctly on first try |
| Learning Gain After Feedback | Whether a child improves after receiving an explanation for a mistake |
| Retention | Whether understanding remains after a delay |
| Mastery Rate | % of concepts for which the child demonstrates sufficient understanding |

### 10.3 Engagement Metrics

| Metric | Target |
|---|---|
| Chapter completion rate | >80% (from ~60% baseline) |
| Average questions attempted per session | 30+ (from 15 baseline) |
| Repeat sessions per week | 3+ (from 1.2 baseline) |
| Weekly active learners | Growing |
| Chapter return rate | Increasing |
| Practice participation | High |
| Repeat attempts after mistakes | High (indicates willingness to try again) |

Engagement is a supporting metric, not the definition of success. A child spending 90 minutes without learning is not success.

### 10.4 Learning Experience Metrics

| Metric | Purpose |
|---|---|
| Which explanations children use | Identify most effective explanation formats |
| Which interactions they complete | Identify engaging interaction types |
| Where they abandon | Identify friction points |
| Which questions generate repeated mistakes | Identify difficult concepts |
| Which misconceptions are common | Build misconception library for future AI |
| Whether visual interaction improves performance | Validate visual learning approach |
| Whether worked examples improve subsequent solving | Validate worked example approach |
| Whether timed quizzes improve engagement vs. anxiety | Validate timer feature |
| Whether memory match improves concept retention | Validate game-based learning |

### 10.5 Aasha Economy Metrics (Phase 4)

| Metric | Purpose |
|---|---|
| Coins earned vs. coins saved | Are children learning to save? |
| Saving duration | Are children practising patience? |
| Redemption rate | When do children choose to spend? |
| Reward selection | What do children value? |
| Hint token usage | Do children use help strategically? |

Critical behavioural measure: **Are children learning to save and make choices, or simply trying to maximise points?**

### 10.6 Seva Metrics (Phase 3)

| Metric |
|---|
| Children participating in Seva |
| Activities submitted |
| Activities verified |
| Repeat participation |
| Quality of evidence |
| Learning-to-action conversion |
| Activities per participating child |

A high number of submissions without meaningful verification is not success.

### 10.7 Impact Metrics (Long-term)

| Dimension | Question |
|---|---|
| Learning | Did the child understand better? |
| Behaviour | Did learning lead to meaningful action? |
| Agency | Did the child develop a sense of effort and choice? |
| Responsibility | Did the child practise saving and contribution? |
| Continuity | Did the child continue participating? |
| Contribution | Can the cycle eventually help create opportunity for another child? |

---

## 11. Strategic Product Hypotheses

### Hypothesis 1 — Learning
> If a child's existing school-book content is transformed into an interactive, language-accessible, concept-focused learning experience, children who previously read without fully understanding can develop stronger conceptual understanding and greater confidence.

### Hypothesis 2 — Values
> If learning is connected to meaningful real-world responsibility and an intentionally designed economy of effort, saving, and choice, children can begin to experience learning as something that has value beyond examination marks.

### Hypothesis 3 — Scale
> If these experiences are designed as reusable learning assets rather than one-time interventions, the same underlying product can eventually serve many children and schools.

---

## 12. Product Roadmap

### Phase 1 — Prove Learning (Current)
**Focus:** School Book → Concept → Understand → Practise → Master  
**Success question:** Does Aasha help children understand better?  
**Status:** 3 chapters built, tested, delivered. Gamification (timer, rapid fire, memory match, gem shop) implemented.

### Phase 2 — Personalise Learning
**Add:** Deeper learner profiles, misconception patterns, adaptive difficulty, alternative explanations, personalised practice, stronger language support.  
**Success question:** Can Aasha increasingly adapt the experience to the individual child?

### Phase 3 — Connect Learning to Action
**Add:** Eco Seva, Jal Seva, activity evidence, verification, meaningful-action tracking.  
**Success question:** Does learning translate into responsible action?

### Phase 4 — Build the Aasha Economy
**Add:** Aasha Coins (full), saving, choices, Aasha Store, responsible rewards.  
**Success question:** Can children experience effort, patience, saving, and responsible choice?

### Phase 5 — Build the Infinite Hope Loop
**Connect:** Understand → Learn → Act → Earn → Save → Contribute → Enable Another Child  
**Success question:** Can one child's learning and responsibility become part of an ecosystem that creates opportunity for another child?

---

## 13. Product Principles for Feature Decisions

Every future feature must pass these questions:

1. Does this help the child understand?
2. Does this help the child practise?
3. Does this help the child become more independent?
4. Does this connect learning with meaningful action?
5. Does this encourage responsibility rather than dependency?
6. Does this work for the child we are actually trying to serve?

If the answer is consistently no, the feature should not be prioritised.

---

## 14. Current State vs Full Vision

| Area | Current | Full Vision |
|---|---|---|
| School chapters | ✅ 3 chapters | Full NCERT coverage |
| Concept learning | ✅ | ✅ |
| Visual explanations | ✅ Canvas-based | ✅ |
| Interactive learning | ✅ Drag, tap, slider | ✅ |
| Worked examples | ✅ Step-by-step | ✅ |
| Practice (all types) | ✅ Quiz, FB, TF, solve | ✅ |
| Conceptual feedback | ✅ Misconception explanations | ✅ |
| Final assessment | ✅ Worksheet levels | ✅ |
| Child profiles | ✅ Named, saved | ✅ |
| Progress | ✅ Bar + ring | ✅ |
| XP, levels, coins, badges | ✅ | ✅ |
| Streaks | ✅ | ✅ |
| Language bridge (LLE) | ✅ 78+ words | Expanded |
| Offline learning | ✅ Zero dependency | ✅ |
| Timed quizzes | ✅ Speed bonus | ✅ |
| Rapid fire | ✅ | ✅ |
| Memory match | ✅ | ✅ |
| Gem shop | ✅ Hints, themes, frames | ✅ |
| Back/restart | ✅ | ✅ |
| Daily challenge | Designed | ✅ |
| AI adaptation | Future | ✅ |
| Eco Seva | Vision | ✅ |
| Jal Seva | Vision | ✅ |
| Activity verification | Vision | ✅ |
| Saving economy | Vision | ✅ |
| Aasha Store | Vision | ✅ |
| Parent experience | Future | ✅ |
| Teacher tools | Future | ✅ |
| Infinite Hope loop | Vision | Long-term |

---

## 15. The One Thing We Must Protect

As Aasha grows, the product must never lose its original purpose.

It should never become:
> "Another app trying to keep children on a screen."

It should remain:
> "A tool that helps a child understand the book in front of them, become more capable because of it, and eventually use what they learn to do something meaningful in the world."

**Aasha is not building a better screen.  
Aasha is building a better path from the page to the child — and eventually from the child to the world.**
