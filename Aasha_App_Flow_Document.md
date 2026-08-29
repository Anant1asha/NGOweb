# Aasha — Complete App Flow Document

**Product:** Aasha — Safe Learning. Real Impact.  
**Organisation:** Annanth Aasha Foundation  
**Document Type:** App Flow Document (UX Strategy)  
**Version:** 1.0  
**Date:** August 2026  
**Status:** Ready for Implementation  
**Purpose:** This document specifies every screen, user action, button behaviour, navigation path, success state, error state, and empty state in the Aasha ecosystem. It is detailed enough for an AI coding agent to build without guessing.

---

## Table of Contents

1. [Ecosystem Overview](#1-ecosystem-overview)
2. [Screen Inventory](#2-screen-inventory)
3. [Login / Signup / Profile Flow](#3-login--signup--profile-flow)
4. [Chapter Hub Flow](#4-chapter-hub-flow)
5. [Learning Engine Flow](#5-learning-engine-flow)
6. [Assessment Flows](#6-assessment-flows)
7. [Gamification Flows](#7-gamification-flows)
8. [Gem Shop / Economy Flow](#8-gem-shop--economy-flow)
9. [Seva Activity Flow (Phase 3)](#9-seva-activity-flow-phase-3)
10. [Aasha Economy — Saving & Store Flow (Phase 4)](#10-aasha-economy--saving--store-flow-phase-4)
11. [Teacher Dashboard Flow (Phase 2)](#11-teacher-dashboard-flow-phase-2)
12. [Parent Dashboard Flow (Phase 2)](#12-parent-dashboard-flow-phase-2)
13. [Navigation Map](#13-navigation-map)
14. [Empty States Catalogue](#14-empty-states-catalogue)
15. [Error States Catalogue](#15-error-states-catalogue)
16. [Success States Catalogue](#16-success-states-catalogue)
17. [Animation & Feedback Reference](#17-animation--feedback-reference)
18. [Button Behaviour Reference](#18-button-behaviour-reference)

---

## 1. Ecosystem Overview

### 1.1 Architecture Evolution

The current prototype is a set of self-contained HTML files (one per chapter). The ecosystem goal is to evolve this into a connected platform while preserving the offline-first learning core.

### 1.2 Design Principles for the Ecosystem

1. **Learning chapters remain offline HTML files** — each up to 20 MB (current: ~416 KB), self-contained
2. **The ecosystem wraps around the chapters** — a thin launcher/sync layer that is optional
3. **Children never need to log in to learn** — the chapter files work independently
4. **Login is only for teachers, parents, and sync** — children use local named profiles
5. **No real-money payments ever** — all economy is virtual coins earned through learning
6. **Every screen must work on a 5-inch Android phone** — mobile-first, touch-first

---

## 2. Screen Inventory

### 2.1 Complete Screen List

| # | Screen | Phase | User | Offline? |
|---|---|---|---|---|
| S01 | Splash / Brand Screen | 1 | Child | Yes |
| S02 | Profile Picker | 1 | Child | Yes |
| S03 | Create New Profile | 1 | Child | Yes |
| S04 | Chapter Hub (subject to chapter list) | 1 | Child | Yes |
| S05 | Chapter Map (concepts overview) | 1 | Child | Yes |
| S06 | Concept Introduction | 1 | Child | Yes |
| S07 | Text Explanation (LLE) | 1 | Child | Yes |
| S08 | Canvas Visual Explanation | 1 | Child | Yes |
| S09 | Interactive Canvas (drag) | 1 | Child | Yes |
| S10 | Interactive Canvas (slider) | 1 | Child | Yes |
| S11 | Interactive Canvas (diagonal) | 1 | Child | Yes |
| S12 | Worked Example (step-by-step) | 1 | Child | Yes |
| S13 | Solve Problem (guided) | 1 | Child | Yes |
| S14 | Quiz (MCQ, timed) | 1 | Child | Yes |
| S15 | True/False | 1 | Child | Yes |
| S16 | Fill-in-the-Blank | 1 | Child | Yes |
| S17 | Progress Milestone | 1 | Child | Yes |
| S18 | Rapid Fire Round | 1 | Child | Yes |
| S19 | Memory Match Game | 1 | Child | Yes |
| S20 | Worksheet Assessment (Basic) | 1 | Child | Yes |
| S21 | Worksheet Assessment (Standard) | 1 | Child | Yes |
| S22 | Worksheet Assessment (HOTS) | 1 | Child | Yes |
| S23 | Worksheet Assessment (Final Mixed) | 1 | Child | Yes |
| S24 | Chapter Complete / Celebration | 1 | Child | Yes |
| S25 | Gem Shop | 1 | Child | Yes |
| S26 | Level-Up Overlay | 1 | Child | Yes |
| S27 | Badge Earned Overlay | 1 | Child | Yes |
| S28 | Word Dialog (LLE) | 1 | Child | Yes |
| S29 | Daily Challenge | 2 | Child | Yes |
| S30 | Leaderboard (cohort, opt-in) | 2 | Child | Sync |
| S31 | Seva Hub (activity list) | 3 | Child | Sync |
| S32 | Seva Submit (photo + description) | 3 | Child | Sync |
| S33 | Seva Pending / Verified | 3 | Child | Sync |
| S34 | Aasha Wallet (coin balance) | 4 | Child | Sync |
| S35 | Aasha Store (real rewards) | 4 | Child | Sync |
| S36 | Saving Goal | 4 | Child | Sync |
| S37 | Teacher Login | 2 | Teacher | Online |
| S38 | Teacher Dashboard (class overview) | 2 | Teacher | Online |
| S39 | Teacher Student Detail | 2 | Teacher | Online |
| S40 | Teacher Seva Verification | 3 | Teacher | Online |
| S41 | Parent Login | 2 | Parent | Online |
| S42 | Parent Dashboard (child progress) | 2 | Parent | Online |
| S43 | Admin Content Management | 2 | Admin | Online |
| S44 | Settings (theme, sound, language) | 1 | Child | Yes |

---

## 3. Login / Signup / Profile Flow

### 3.1 Design Decision

Aasha uses **local profiles for children** (no server account needed) and **server accounts for teachers/parents**. This preserves the offline-first principle: a child can learn with zero internet, while teachers and parents get sync and analytics when online.

### 3.2 Child Profile Flow

**S01 — Splash / Brand Screen**

- Shows Aasha logo, tagline "Safe Learning. Real Impact."
- Brand animation plays for 1.5 seconds
- Auto-advances to S02 Profile Picker after 1.5s or on tap

**S02 — Profile Picker**

- Header: "Who's learning today?"
- Lists all existing profiles as cards. Each card shows: avatar, name, class, progress percentage bar, XP, coins, streak, level
- "+ New Learner" button at bottom

**EMPTY STATE (no profiles):**
- "Welcome to Aasha!"
- "Let's start your learning journey."
- [ + Create Your Profile ] button (goes to S03)

**Navigation:**
- Tap existing profile card -> S04 Chapter Hub
- Tap "+ New Learner" -> S03 Create New Profile

**S03 — Create New Profile**

Fields:
- Name (text input, required, max 20 characters)
- Class (radio buttons: 5, 6, 7, 8; default: 8)
- Avatar (6 emoji choices, select one; default: first)

Buttons:
- [ Cancel ] -> return to S02
- [ Start Learning ] -> validate, create profile, go to S04

**VALIDATION:**
- Name empty -> error: "Please enter your name" (focus input)
- Name > 20 chars -> error: "Name too long (max 20 characters)" (truncate)
- Name already exists -> error: "This name is already in use. Try a different one." (clear input)

**ON SUCCESS:**
- `localStorage.setItem('aasha_profiles', {...})` updated with new profile
- Navigate to S04 Chapter Hub

### 3.3 Profile Data Model

```javascript
// localStorage key: aasha_profiles
{
  profiles: [
    {
      name: "Rahul",
      class: 8,
      avatar: "fox",
      createdAt: "2026-08-24T10:00:00Z",
      chaptersCompleted: ["quadrilaterals"],
      totalXP: 320,
      totalCoins: 145,
      level: 3,
      streak: 4
    }
  ]
}

// Per-chapter progress: aasha_quad_<childName>
{
  nIdx: 5,      // current node index
  sIdx: 3,      // current step index
  coins: 145,
  xp: 320,
  level: 3,
  streak: 4,
  badges: { first: true, angles: true },
  conceptsMastered: { 0: true, 1: true, 2: true }
}
```

### 3.4 Teacher / Parent Login Flow (Phase 2+)

**S37 — Teacher Login**

Two login methods on one screen:
1. **Phone OTP**: Enter phone (+91 field), tap [Send OTP], enter 6-digit code, tap [Verify]
2. **Email/Password**: Enter email, enter password, tap [Login]

Additional links:
- [Forgot Password] -> password reset flow
- [Register] -> registration form (name, email, phone, password, school)

**OTP Verification sub-screen:**
- 6 input boxes for digits
- [Verify] button (disabled until 6 digits entered)
- "Resend in 30s" countdown (rate limiting)

**BUTTON BEHAVIOURS:**

| Button | Success | Error |
|---|---|---|
| [Send OTP] | "OTP sent to +91....1234", show OTP input | "Invalid phone number" / "Rate limited. Try in 60s" |
| [Verify] | JWT issued, navigate to S38 | "Invalid OTP. Try again." (shake) / "OTP expired. Resend." |
| [Login] | JWT issued, navigate to S38 | "Invalid email or password" / "Account not found. Register?" |
| [Register] | JWT issued, navigate to S38 | "Email already registered. Login?" |

**OTP states:**
- EMPTY: OTP field empty, Verify disabled
- SUCCESS: Green checkmark animation, auto-advance to dashboard
- ERROR: Red shake animation on OTP boxes, clear inputs

---

## 4. Chapter Hub Flow

### 4.1 Chapter Hub Screen (S04)

**Header bar:** Avatar + Name + Level icon + Level name + XP + Coins + Streak. Settings gear icon and Shop icon.

**Body:** "What are we learning today?"

Chapters grouped by subject. Each chapter card shows:
- Icon, title, class level
- Progress bar with percentage
- Number of concepts
- Button: [ Continue -> ] (if in progress) or [ Start -> ] (if not started) or "Completed" badge (if 100%)

Footer: [ Back to Profiles ]

### 4.2 Chapter Hub Behaviours

| Action | Trigger | Behaviour |
|---|---|---|
| Tap chapter card | Click on chapter card | Navigate to S05 Chapter Map |
| Tap "Continue" | Click Continue button | Navigate to S05, resume at last position |
| Tap "Start" | First time opening chapter | Navigate to S05, start at node 0 |
| Tap Settings | Click gear icon | Navigate to S44 Settings |
| Tap Shop | Click shop icon | Navigate to S25 Gem Shop |
| Tap "Back to Profiles" | Click back button | Navigate to S02 Profile Picker |

### 4.3 Chapter Hub States

**Empty state (no chapters downloaded):**
- "No chapters yet!"
- "Ask your teacher to share a chapter file"
- "or tap below to see available chapters."
- [ Browse Available Chapters ] button

**Success state (chapter completed):**
- Chapter card shows "Completed! 100% 22/22 concepts"
- "Chapter Champion badge earned"
- Buttons: [ Review Chapter ] [ Restart ]

---

## 5. Learning Engine Flow

### 5.1 Chapter Map Screen (S05)

**Header:** Back button, chapter title, child name + progress percentage + level + streak

**Body:** Visual node graph showing all concepts as connected icons. Each node is one of three states:
- **Completed** (green check): can be tapped to review
- **Current** (blue, pulsing): can be tapped to continue
- **Locked** (grey, lock icon): tapping shows shake animation + "Complete previous concepts first"

**Current node card:** Shows icon, title, subtitle/description, [ Start -> ] button

**Footer:** [Settings] [Shop] [Progress]

### 5.2 Chapter Map Behaviours

| Action | Trigger | Behaviour |
|---|---|---|
| Tap completed node | Click on completed concept | Navigate to that concept (review mode) |
| Tap current node | Click on current concept | Navigate to S06 Concept Introduction |
| Tap locked node | Click on locked concept | Shake animation + "Complete previous concepts first" |
| Tap Progress | Click progress icon | Navigate to detailed progress view |
| Tap Back | Click back | Navigate to S04 Chapter Hub |
| Long press completed node | Long press | Show tooltip: "Mastered on Aug 24" |

### 5.3 Learning Step Flow (Within a Concept)

Each concept node contains a sequence of steps. The learner progresses through them linearly:

```
Node N: "What is a Polygon?"
  Step 0: intro   -> S06 Concept Introduction
  Step 1: text    -> S07 Text + LLE
  Step 2: visual  -> S08 Canvas Visual
  Step 3: ix_drag -> S09 Interactive (drag)
  Step 4: we      -> S12 Worked Example
  Step 5: quiz    -> S14 Quiz (timed)
  Step 6: tf      -> S15 True/False
  Step 7: fb      -> S16 Fill-in-the-Blank
  Step 8: progress-> S17 Progress Milestone

Each step: [Back]  content  [Continue ->]
Step 8 (progress) -> unlock next node -> advance
```

### 5.4 Concept Introduction Screen (S06)

**Header:** Back button, "Node X of N", progress percentage
**Body:** Node icon, concept title, subtitle, introductory text (LLE active)
**Footer:** [ Continue -> ] button

**Button behaviours:**
- [Continue ->] -> `App.next()` -> advance to next step (sIdx++)
- [Back] -> `App.back()` -> go to previous step (sIdx-- or nIdx-- if at step 0)

**Success state:** Step renders, LLE processes text within 50ms, highlighted words are interactive.
**Error state (LLE fails):** Fallback to plain text (no highlighting). Console error logged.

### 5.5 Text Explanation Screen with LLE (S07)

**Header:** Back, concept title, step number (e.g., 3/8)
**Body:** Text content with LLE-processed words. Highlighted words (blue) are tappable. Connective words show inline Hindi in parentheses.

**Word Dialog (S28) — triggered by tapping a highlighted word:**
- Modal overlay (not a full screen)
- Shows: English word, Hindi translation, brief definition
- [ Hear Word ] -> playTone() with word rhythm
- [ Got It ] -> close dialog, return to text
- [x] or tap outside -> close dialog

**Footer:** [ Continue -> ]

### 5.6 Canvas Visual Explanation Screen (S08)

**Header:** Back, concept title, step number
**Body:** Canvas element (drawn with getContext('2d')), caption text below
**Footer:** [ Continue -> ]

**Canvas rendering:** Uses `canvas.getContext('2d')`, draws shapes with `moveTo()`, `lineTo()`, `stroke()`, `fill()`. Colours from CSS custom properties.

**Success state:** Canvas renders within 3ms, shapes are visible with labels.
**Error state (canvas unsupported):** Show fallback SVG image with `<polygon>` elements.

### 5.7 Interactive Canvas Screens (S09, S10, S11)

#### S09 — Drag Interaction

- Prompt: "Drag the blue dot to draw a diagonal"
- Canvas shows a polygon with draggable vertex
- User drags blue dot to a non-adjacent vertex
- BEHAVIOURS:
  - Touch/drag on blue dot -> draw line to target
  - On valid diagonal drawn -> green flash + correct tone
  - On invalid line (adjacent vertices) -> red flash + low tone
  - [Reset] -> clear canvas, reset count
  - [Continue] -> enabled after correct interaction
- SUCCESS: Correct diagonal drawn -> green + tone
- ERROR: Wrong vertices selected -> red + low tone
- EMPTY: No interaction yet -> [Continue] disabled

#### S10 — Slider Interaction

- Prompt: "Move the slider to change the shape"
- Canvas shows polygon that updates in real-time as slider moves
- Slider range: 3-20 (number of sides)
- Formula updates in real-time: (n-2) x 180
- [Continue] -> always enabled (exploratory interaction)
- SUCCESS: Slider moves -> shape updates smoothly
- ERROR: Slider out of range -> clamped to 3-20

#### S11 — Diagonal Drawing Interaction

- Prompt: "Tap two non-adjacent vertices to draw a diagonal"
- BEHAVIOURS:
  - Tap vertex 1 -> blue highlight
  - Tap vertex 2 (non-adjacent) -> draw diagonal
  - Tap vertex 2 (adjacent) -> red flash, "not a diagonal"
  - Count updates: "Diagonals: 1/2"
  - [Continue] enabled after target count reached
- SUCCESS: All diagonals drawn -> green + tone
- ERROR: Adjacent vertices tapped -> red + message
- EMPTY: No vertices tapped -> hint: "Tap a corner"

### 5.8 Worked Example Screen (S12)

**Header:** Back, "Worked Example", step number
**Body:**
- Question text at top
- Steps revealed one at a time (hidden initially)
- Progress dots showing how many steps revealed
- [ Previous Step ] and [ Next Step -> ] buttons
- [ Continue to Practice -> ] button (only enabled after all steps revealed)

**BEHAVIOURS:**
- [Next Step] -> reveal next step with slideIn animation
- [Previous Step] -> hide last step
- [Continue] -> only enabled after all steps revealed

**SUCCESS:** Final step revealed -> green border + check mark
**EMPTY:** Only question visible, all steps hidden
**ERROR:** N/A (worked examples are read-only)

### 5.9 Solve Problem Screen (S13)

**Header:** Back, "Solve", step number
**Body:**
- Problem text at top
- Multiple steps, each with a text input and [Check] button
- Each step is locked until the previous step is answered correctly
- Progress dots

**BEHAVIOURS:**
- [Check] -> validate answer -> show feedback
  - Correct: green feedback, +2 XP, unlock next step
  - Wrong: red feedback, show correct answer, allow retry (clear input)
- [Continue] -> enabled when all steps are correct

**SUCCESS:** All steps correct -> green border, XP+5
**ERROR:** Wrong answer -> red feedback, retry allowed
**EMPTY:** Input blank, [Check] disabled

### 5.10 Answer Checking Logic (Detailed)

**Quiz (MCQ) — `checkQuiz(step, cb)`:**

```
User selects option A -> App.selOpt = 0
User taps [Check] -> checkQuiz()
  IF option.c is true:
    - Mark option with .correct class (green background)
    - playTone(523, 0.12, "sine") + playTone(659, 0.12, "sine") after 120ms
    - Show "Correct! +5 XP" (+ speed bonus if applicable)
    - XP += 5 + bonus, streak++, updateStats(), checkLevelUp()
    - Change button to "Continue ->" (green)
    - Stop quiz timer
  IF option.c is false:
    - Mark selected option with .wrong class (red background)
    - Mark correct option with .correct class (green background)
    - playTone(200, 0.15, "sawtooth")
    - Show "Not quite." + misconception text (if opt.m exists)
    - streak = 0, updateStats()
    - Reset selection, re-enable options for retry
    - Button stays as "Check" (disabled until new selection)
```

**True/False — `checkTrueFalse(step, cb)`:**

```
User taps True or False -> App.selTf = true/false
User taps [Check] -> checkTrueFalse()
  IF selTf === step.q.answer:
    - Mark selected button with green
    - playTone(523, 0.12, "sine")
    - Show "Correct! +5 XP" + explanation (if step.q.exp)
    - XP += 5, streak++, updateStats(), checkLevelUp()
    - Change button to "Continue ->" (green)
  IF selTf !== step.q.answer:
    - playTone(200, 0.15, "sawtooth")
    - Show "Not quite." + explanation
    - streak = 0, updateStats()
    - Reset selection (remove .sel-true / .sel-false)
    - Button stays as "Check" (disabled)
```

**Fill-in-the-Blank — `checkFillBlank(step, cb)`:**

```
User types answer in input field
User taps [Check] -> checkFillBlank()
  val = input.value.trim().toLowerCase()
  ans = String(step.q.answer).toLowerCase()
  
  correct = (val === ans)                           // exact string match
        OR (parseInt(val) === parseInt(ans))         // integer match
        OR (parseFloat(val) === parseFloat(ans))     // float match (37.5, 62.5)
        OR (val === ans.replace(/\s/g, ""))           // whitespace-insensitive
  
  IF correct:
    - Fill the blank with user's answer
    - playTone(523, 0.12, "sine")
    - Show "Correct! +5 XP"
    - XP += 5, streak++, updateStats(), checkLevelUp()
    - Change button to "Continue ->" (green)
  IF wrong:
    - playTone(200, 0.15, "sawtooth")
    - Show "Not quite. Answer: [correct answer]"
    - Show explanation (if step.q.exp)
    - streak = 0, updateStats()
    - Change button to "Continue ->" (green)
    - NOTE: Fill-blank allows continuing even after wrong answer
      (the correct answer is shown, so the child learns)
```

**Solve — `checkSolve(step, cb)`:**

```
Multi-step problem. Each step has its own input.
For each step:
  User types answer -> taps [Check] for that step
  
  IF correct:
    - green feedback, +2 XP per step
    - Unlock next step input
  IF wrong:
    - red feedback, show correct answer
    - Allow retry (clear input)
    - After retry, if still wrong, show answer and allow continue

  When all steps are correct:
    - Show "Problem Solved! +5 XP"
    - Enable [Continue ->] button
```

---

## 6. Assessment Flows

### 6.1 Progress Milestone Screen (S17)

After completing all learning steps in a node, the child reaches a progress milestone.

**Body:**
- "Concept Complete!" heading
- Concept name with check mark
- Three stat cards: +XP, +Coins, Streak
- Progress ring showing chapter completion percentage
- [ Continue to Next Concept -> ] button

**BEHAVIOURS on render:**
- `coins += step.tokens` (e.g., 10)
- `XP += step.xp` (e.g., 20)
- `conceptsMastered[nIdx] = true`
- `save()` to localStorage
- `checkBadges()` -> may trigger badge overlay (S27)
- `checkLevelUp()` -> may trigger level-up overlay (S26)
- `fireConfetti()` animation
- `playTone()` celebration sound

**[Continue]** -> `App.next()` -> advance to next node

**SUCCESS:** Confetti + sound + green border + stats up
**IF level up:** Level-Up Overlay (S26) appears
**IF badge earned:** Badge Overlay (S27) appears

### 6.2 Worksheet Assessment Screens (S20-S23)

**Header:** Back, worksheet level name, question number (Q 3/10), progress bar
**Body:**
- Question text
- Answer options (MCQ, fill-blank, true/false, or solve step)
- [Check] button
- After answering: feedback (correct/wrong) + worked solution shown below
- [ Next Question -> ] button

**BEHAVIOURS:**
- Same checking logic as quiz/fill-blank/true-false/solve
- Each worksheet has 10-15 questions
- Progress bar at top shows Q X/Y
- Cannot go back to previous worksheet question
- Must answer before proceeding

**TYPES within worksheet:**
- Section A: MCQ (quiz type)
- Section B: Fill-in-the-blank
- Section C: True/False
- Section D-E: Step-by-step worked solutions

**COMPLETION:** After last question -> show summary
- "Worksheet Complete! 8/10 correct +40 XP +20 Coins"
- [ Continue to Standard Worksheet -> ]

**Worksheet levels progression:** Basic (S20) -> Standard (S21) -> HOTS (S22) -> Final Mixed (S23)

---

## 7. Gamification Flows

### 7.1 Level-Up Overlay (S26)

Full-screen overlay (not a separate page). Shows when `checkLevelUp()` detects XP crossed a threshold.

**Content:** "LEVEL UP!" heading, new level name, level icon
**Animation:** `luscale` animation (scales up from centre with bounce), confetti burst
**Audio:** Ascending arpeggio (523 -> 659 -> 784 Hz, sine wave)

**Levels:**

| XP Threshold | Level Name | Icon |
|---|---|---|
| 0 | Beginner | seedling |
| 50 | Learner | book |
| 120 | Scholar | books |
| 200 | Expert | graduation cap |
| 350 | Master | trophy |

**[Continue]** -> closes overlay, returns to current step

### 7.2 Badge Earned Overlay (S27)

Modal dialog (not a full page). Shows when `checkBadges()` detects a badge condition is met.

**Content:** "Badge Earned!" heading, badge icon, badge name, badge description
**Animation:** `pop` animation (badge scales from 0 to 1), confetti burst

**Badges:**

| Badge ID | Name | Condition |
|---|---|---|
| first | First Steps | Complete your first concept |
| angles | Angle Master | Discover angle sum property |
| para | Parallel Thinker | Master parallelograms |
| family | Family Tree | Master the quadrilateral hierarchy |
| champ | Chapter Champion | Complete the chapter |

**[Collect Badge]** -> close dialog, badge saved to profile

### 7.3 Rapid Fire Round (S18)

**Header:** "RAPID FIRE", question number (Q 3/10), streak counter, timer bar (depleting from 10s to 0s)
**Body:** Question text, 4 answer option buttons (large tappable areas)

**BEHAVIOURS:**
- 10 questions, 10 seconds each
- Timer bar depletes from 10s to 0s
- Tap answer -> immediate feedback (no Check button)
- Correct: green flash + correct tone -> auto-advance after 0.5s
- Wrong: red flash + low tone -> auto-advance after 0.5s
- Timeout: show correct answer -> auto-advance after 1s
- No back button during rapid fire
- Streak bonus: +2 XP per consecutive correct

**AT END (10 questions done):**
- "Rapid Fire Complete!"
- Score: 7/10, Best Streak: 6
- XP Earned: +35, Coins Earned: +15
- Best Score: 8/10 (previous best)
- [ Continue -> ]

**SUCCESS:** Correct answer -> green flash, auto-advance
**ERROR:** Wrong answer -> red flash, correct shown
**TIMEOUT:** Timer hits 0 -> show correct -> advance

### 7.4 Memory Match Game (S19)

**Header:** "Memory Match", Moves counter, Pairs counter (e.g., 2/6)
**Body:** 4x3 grid of face-down cards (12 cards total, 6 pairs)

**BEHAVIOURS:**
- Tap card 1 -> flip (show front with term/property)
- Tap card 2 -> flip
- IF match: green border + correct tone -> cards stay open
- IF no match: red border + low tone -> cards flip back after 1s
- Moves counter increments per pair attempt
- Pairs counter increments on match

**CARD CONTENT (pairs match a shape name to its property):**
- "Rhombus" <-> "All sides equal"
- "Rectangle" <-> "All angles 90"
- "Parallelogram" <-> "Opposite sides parallel"
- "Square" <-> "Rhombus + Rectangle"
- "Trapezium" <-> "One pair parallel"
- "Kite" <-> "Adjacent sides equal"

**AT END (all 6 pairs matched):**
- "Memory Match Complete!"
- Moves: 8, Pairs: 6/6
- XP: +25, Coins: +10
- [ Continue -> ]

**SUCCESS:** Match found -> green border, cards stay open
**ERROR:** No match -> red border, cards flip back
**EMPTY:** All cards face-down at start

### 7.5 Timed Quiz Behaviour (Within S14)

The quiz screen has a timer that adds urgency:

**Header:** Back, "Quiz", step number, timer display (e.g., "7s"), timer bar (depleting left to right)

**TIMER BEHAVIOUR:**
- Timer starts at 10s when question renders
- Timer bar depletes left to right
- Speed bonus:
  - Answer within 3s -> +3 XP bonus
  - Answer within 6s -> +2 XP bonus
  - Answer within 10s -> +1 XP bonus
  - Timeout -> no bonus, show correct answer

**HINT BEHAVIOUR:**
- [Hint] -> `useHint()` -> removes 2 wrong options (greys them out)
- Costs 1 hint token (from shop inventory)
- If no hint tokens: "No hint tokens. Visit shop."
- Hint button greys out after use

**TIMEOUT BEHAVIOUR:**
- Timer hits 0 -> `quizTimeout()` fires
- All options become non-clickable
- Correct answer highlighted in green
- "Time's up! The answer was: [correct]"
- [Continue] button appears (green)
- streak = 0

---

## 8. Gem Shop / Economy Flow

### 8.1 Gem Shop Screen (S25)

**Header:** Back, "Gem Shop", coin balance
**Body:** Items grouped by category (Consumables, Themes, Avatar Frames)

**Shop Items:**

| Item | Cost | Type | Description |
|---|---|---|---|
| Hint Token | 20 coins | consumable | Remove 2 wrong options in next quiz |
| Streak Freeze | 30 coins | consumable | Protect streak from 1 wrong answer |
| Ocean Theme | 50 coins | permanent | Blue/teal colours |
| Forest Theme | 50 coins | permanent | Green colours |
| Sunset Theme | 50 coins | permanent | Orange/pink colours |
| Gold Frame | 100 coins | permanent | Gold border on profile avatar |
| Star Frame | 150 coins | permanent | Star border on profile avatar |

Each item card shows: icon, name, cost, description, [Buy] button (or [Apply] if already owned for themes)

### 8.2 Shop Button Behaviours

| Button | State | Behaviour |
|---|---|---|
| [Buy] (consumable) | Enough coins | Deduct cost, increment owned count, play tone |
| [Buy] (consumable) | Not enough coins | Shake button, "Not enough coins. Earn more!" |
| [Buy] (theme) | Enough coins | Deduct cost, apply theme immediately, play tone |
| [Buy] (theme) | Already owned | Button shows "Owned [Apply]" |
| [Buy] (frame) | Enough coins | Deduct cost, apply frame, play tone |
| [Preview] (theme) | Any | Show 3s preview of colour scheme, then revert |
| [Apply] (theme) | Owned | `applyTheme()` -> `document.body.className = 'theme-X'` |
| [Back] | Any | Close shop, return to previous screen |

### 8.3 Shop Economy States

**Success state (purchase):**
- "Purchased: Hint Token"
- "-20 coins  Balance: 125 coins"
- "Hint tokens owned: 3"

**Error state (insufficient coins):**
- "Not enough coins!"
- "Need 50 coins, you have 25 coins"
- "Complete more concepts to earn coins."
- [ Earn Coins -> ] (navigate back to chapter)

**Empty state (no items owned):**
- "You haven't bought anything yet!"
- "Earn coins by completing concepts, then spend them here."
- "Current balance: 45 coins"

### 8.4 Economy Data Model

```javascript
// localStorage key: aasha_economy
{
  coins: 145,           // current spendable balance
  totalEarned: 320,     // lifetime earnings (for analytics)
  hintTokens: 2,        // consumable count
  streakFreezes: 0,     // consumable count
  theme: "default",      // "default" | "ocean" | "forest" | "sunset"
  avatarFrame: "none",  // "none" | "gold" | "star"
  purchasedItems: ["theme_ocean", "frame_gold"]  // permanent items
}
```

### 8.5 Coin Economy Flow (No Real Money)

**EARN (learning actions)**

| Action | XP | Coins |
|---|---|---|
| Quiz correct | +5 | 0 |
| True/False correct | +5 | 0 |
| Fill-blank correct | +5 | 0 |
| Solve step correct | +2 | 0 |
| Speed bonus (within 3s) | +3 | 0 |
| Concept milestone | +20 | +10 |
| Rapid fire complete | +25-35 | +15 |
| Memory match complete | +25 | +10 |
| Worksheet complete | +40 | +20 |
| Chapter complete | +100 | +50 |

**SPEND (shop purchases)**

| Item | Cost |
|---|---|
| Hint Token | -20 coins |
| Streak Freeze | -30 coins |
| Theme (permanent) | -50 coins |
| Avatar Frame | -100-150 coins |

**NO REAL MONEY:** No payment gateway, no in-app purchases, no cash-out. Coins are educational currency only.

**There is no payment or upgrade flow.** All economy is virtual, earned through learning. This is a deliberate design decision from the PRD: "The economy teaches effort -> earning -> saving -> patience -> choice."

---

## 9. Seva Activity Flow (Phase 3)

### 9.1 Seva Hub Screen (S31)

**Header:** Back, "Seva Activities"
**Body:** "Learning becomes action"

Activities grouped by type:
- **Eco Seva (Environment):** Plant a Seed, Community Cleanliness
- **Jal Seva (Water):** Water Conservation

Each activity card shows: icon, title, description, status (Not started / X done), [Start Activity ->] button

**Summary section:** Total activities, Verified count, Pending count, Coins earned from Seva

**EMPTY STATE (no activities done):**
- "No Seva activities yet. Start your first one!"

### 9.2 Seva Submit Flow (S32)

**Fields:**
- "What did you do?" — text area (max 200 chars, min 10 chars)
- "Add Photo Evidence" — [Take Photo] / [Choose File] (optional, earns +5 bonus coins)
- Location (auto-detected if online, optional, privacy-respected)

**Buttons:**
- [Cancel] -> return to S31
- [Submit Activity] -> validate and submit

**VALIDATION:**
- Description required (min 10 chars) -> error: "Please describe your activity (at least 10 characters)"
- Photo optional (but earns +5 bonus coins)
- Location optional
- Photo too large (>5MB) -> error: "Photo too large. Please use a smaller image."

**ON SUBMIT (online):**
- POST /api/v1/seva/submit
- Status: "pending verification"
- Navigate to S33

**ON SUBMIT (offline):**
- Store in localStorage queue (`aasha_seva_queue`)
- Status: "queued (will sync when online)"
- Navigate to S33

**SUCCESS:** "Activity submitted! A facilitator will verify it soon. You'll earn coins once verified."
**ERROR:** "Submission failed. Saved locally. Will retry when online."

### 9.3 Seva Pending / Verified Screen (S33)

**Header:** Back, "My Seva Activities"
**Body:** List of submitted activities, each showing:

**PENDING STATE:** Yellow clock icon, "Pending"
- Activity type, submitted date, description, photo thumbnail
- "Waiting for facilitator verification"

**VERIFIED STATE:** Green check, "Verified"
- Activity type, submitted date, verified by, description
- "+30 coins earned"

**REJECTED STATE:** Red cross, "Rejected"
- Activity type, submitted date, description
- "Activity could not be verified. Reason: [text]"
- [Submit Again] button

**EMPTY STATE (no activities):**
- "You haven't submitted any Seva activities yet."
- "Start one from the Seva Hub!"
- [Go to Seva Hub ->]

---

## 10. Aasha Economy — Saving & Store Flow (Phase 4)

### 10.1 Aasha Wallet Screen (S34)

**Header:** Back, "Aasha Wallet"
**Body:**
- Large coin balance display
- Sub-stats: Earned, Spent, Saved, Investing
- [Set Saving Goal] button -> S36
- [Visit Store] button -> S35

**Transaction history list:** Date, description, amount (+/-), balance after

**EMPTY STATE (no transactions):**
- "No transactions yet. Earn coins by learning!"

### 10.2 Saving Goal Flow (S36)

**Fields:**
- "What are you saving for?" — radio options:
  - Aasha Plant-a-Tree (200 coins)
  - Aasha Book for a Child (500 coins)
  - Aasha Water Filter (1000 coins)
  - Custom goal: name + target amount
- Current balance, target, remaining, progress bar
- Deposit input: [__] coins + [Deposit] button (moves coins from spendable to savings, irreversible)

**Buttons:**
- [Cancel] -> return to S34
- [Set Goal] -> lock in target, start tracking

**SUCCESS (goal reached):**
- "Goal Reached!"
- "You saved 200 coins for Plant-a-Tree"
- "A tree will be planted in your name!"
- [Collect Reward]

**ERROR:** Deposit > balance -> "Not enough coins to deposit"
**EMPTY:** No goal set -> prompt to choose one

### 10.3 Aasha Store Flow (S35)

**Header:** Back, "Aasha Store", coin balance
**Body:** Items in two categories:

**Educamental Rewards:**
- New Chapter Unlock (100 coins)
- Premium Theme Pack (200 coins)

**Real-World Impact:**
- Plant a Tree (200 coins) — "A tree will be planted in your name"
- Book for a Child (500 coins) — "Your coins fund a book for another child"
- Water Filter (1000 coins) — "Fund a water filter for a school"

**NOTE:** No real-money purchases. Coins are earned through learning and Seva. Store items create real-world impact managed by the foundation.

**SUCCESS:** "Thank you! Your contribution will create real impact. You'll receive a confirmation when the action is completed."
**ERROR:** Not enough coins -> same as shop error
**EMPTY:** No items affordable -> "Keep earning!"

---

## 11. Teacher Dashboard Flow (Phase 2)

### 11.1 Teacher Dashboard Screen (S38)

Web-based dashboard (not part of chapter HTML). Requires login (S37).

**Header:** Teacher name, class, [Logout]
**Body sections:**

**Class Overview:**
- Students count, active this week, avg. progress, avg. mastery, chapters completed
- Weekly engagement bar chart

**Students Needing Help:**
- Red/yellow indicators per student with specific stuck concept
- [View All Students ->]

**Common Misconceptions:**
- Ranked list of most common misconceptions (e.g., "45% confuse rhombus with rectangle")
- [View Detailed Report ->]

**Seva Verification Queue:**
- Count of pending activities
- [Review Seva Submissions ->] -> S40

**EMPTY STATE (no students synced):**
- "No student data yet. Share chapter files with your students and ask them to sync when online."

### 11.2 Teacher — Student Detail (S39)

**Header:** Back, student name, class
**Body:**
- Total XP, level, coins, streak, sessions this week, avg. session time
- Chapter progress bars (per chapter)
- Concept mastery list (per concept: DONE/WARN/LOCKED + mastery %)
- Misconception pattern (frequently missed questions, confused pairs, recommended remediation)
- Assessment history (worksheet scores, rapid fire scores)
- [Provide Feedback] [Assign Remedial]

### 11.3 Teacher — Seva Verification (S40)

**Header:** Back, "Seva Verification (N pending)"
**Body:** List of pending submissions, each showing:
- Activity type, student name, class, submitted date/time
- Description text
- Photo thumbnail
- Location (if provided)
- [Verify] [Reject] [Skip] buttons
- If [Reject] tapped: reason text input appears, [Confirm Reject]

**EMPTY STATE (no pending):**
- "All caught up! No activities pending verification."

**SUCCESS:** Verified -> student earns coins, notification sent
**ERROR:** Network error -> "Verification saved locally. Will sync."

---

## 12. Parent Dashboard Flow (Phase 2)

### 12.1 Parent Dashboard (S42)

Web-based dashboard. Requires login (S41).

**Header:** Parent name, child name, [Logout]
**Body:**

**"Child's Learning This Week":**
- Concepts learned, concepts mastered, time spent, daily streak, progress %
- Plain-language summary: "Rahul is doing well! He's strong on polygon properties but finding angle sums challenging."
- Suggestion: "Try asking Rahul about shapes around the house — real examples help!"

**Seva Activities:**
- List of child's Seva activities with verification status

**Understanding (not marks):**
- Explanation: "This shows what Rahul understands, not just his scores."
- Per-concept status: GREEN (mastered), YELLOW (learning), RED (needs help)

**EMPTY STATE (no data):**
- "Rahul hasn't started learning yet. Share a chapter file with him to begin!"

---

## 13. Navigation Map

### 13.1 Complete Navigation Graph

```
App Launch
  -> S01 Splash (1.5s)
    -> S02 Profile Picker
      -> S03 Create Profile (on success -> S04)
      -> S04 Chapter Hub (tap existing profile)
        -> S25 Gem Shop ([Back] -> S04)
        -> S44 Settings ([Back] -> S04)
        -> S02 Profile Picker ([Back to Profiles])
        -> S05 Chapter Map (tap chapter card)
          -> S06 Concept Intro
            -> S07 Text + LLE
              -> S28 Word Dialog ([Got It] -> return)
            -> S08 Canvas Visual
            -> S09/S10/S11 Interactive Canvas
            -> S12 Worked Example
            -> S13 Solve Problem
            -> S14 Quiz (timed)
              -> S26 Level-Up Overlay ([Continue] -> return)
              -> S27 Badge Overlay ([Collect] -> return)
            -> S15 True/False
            -> S16 Fill-blank
            -> S17 Progress Milestone
              -> next node (S06) OR
              -> S18 Rapid Fire
              -> S19 Memory Match
              -> S20-S23 Worksheets
                -> S24 Chapter Complete
                  -> [Restart] -> S05 (reset)
                  -> [Back to Hub] -> S04
```

### 13.2 Settings Flow (S44)

**Header:** Back, "Settings"

**Sections:**
- **Appearance:** Theme selector (Default / Ocean / Forest / Sunset) — tap to apply immediately
- **Sound:** Sound Effects toggle (ON/OFF) — toggles playTone globally
- **Language:** LLE Hindi toggle (ON/OFF) — toggles word highlighting
- **Progress:** Profile name, class, total XP, level, badges earned/total
- **Reset Chapter Progress:** Button with confirmation dialog ("This will reset your progress for this chapter. Are you sure?" [Yes] [No])
- **About:** Aasha version, organisation, tagline

**[Back to Chapter]** -> return to previous screen

---

## 14. Empty States Catalogue

| Screen | Empty State | CTA |
|---|---|---|
| S02 Profile Picker | "Welcome to Aasha! Let's start your learning journey." | [ + Create Your Profile ] |
| S04 Chapter Hub (no chapters) | "No chapters yet! Ask your teacher to share a chapter file." | [ Browse Available ] |
| S05 Chapter Map (no progress) | "Ready to begin? Tap the first concept to start!" | [ Start First Concept -> ] |
| S14 Quiz (no selection) | Hint text: "Select an answer above" | (Check button disabled) |
| S15 True/False (no selection) | Hint text: "Tap True or False" | (Check button disabled) |
| S16 Fill-blank (empty input) | Placeholder: "Type your answer here" | (Check button disabled) |
| S19 Memory Match | All cards face-down | (No CTA — just start tapping) |
| S25 Gem Shop (no purchases) | "You haven't bought anything yet! Earn coins by completing concepts." | (No CTA) |
| S25 Gem Shop (no coins) | "You have 0 coins. Complete concepts to earn coins!" | [ Earn Coins -> ] |
| S31 Seva Hub (no activities) | "No Seva activities yet. Start your first one!" | (Browse activities) |
| S33 Seva Status (no submissions) | "You haven't submitted any Seva activities yet." | [ Go to Seva Hub -> ] |
| S34 Wallet (no transactions) | "No transactions yet. Earn coins by learning!" | [ Go to Chapter Hub -> ] |
| S38 Teacher Dashboard (no data) | "No student data yet. Share chapter files with your students." | (No CTA) |
| S40 Seva Verification (empty queue) | "All caught up! No activities pending verification." | (No CTA) |
| S42 Parent Dashboard (no data) | "Rahul hasn't started learning yet. Share a chapter file!" | (No CTA) |

---

## 15. Error States Catalogue

| Screen | Error Condition | Error Message | Recovery Action |
|---|---|---|---|
| S03 Create Profile | Name empty | "Please enter your name" | Focus name input |
| S03 Create Profile | Name > 20 chars | "Name too long (max 20 characters)" | Truncate input |
| S03 Create Profile | Name already exists | "This name is already in use. Try a different one." | Clear input |
| S09 Interactive Drag | Wrong vertices | Red flash + "That's not a diagonal — try non-adjacent corners" | Allow retry |
| S10 Interactive Slider | Value out of range | (Clamped to 3-20 automatically) | Auto-clamp |
| S11 Interactive Diagonal | Adjacent vertices | "These are adjacent — diagonals connect non-adjacent corners" | Allow retry |
| S13 Solve Problem | Wrong answer | "Not quite. Answer: [correct]" + explanation | Allow retry or continue |
| S14 Quiz | Wrong answer | "Not quite." + misconception text (if available) | Reset selection, allow retry |
| S14 Quiz | Timeout (10s) | "Time's up! The answer was: [correct option]" | Show correct, allow continue |
| S14 Quiz | Hint with no tokens | "No hint tokens available. Visit the shop to buy more." | Link to shop |
| S15 True/False | Wrong answer | "Not quite." + explanation | Reset selection, allow retry |
| S16 Fill-blank | Wrong answer | "Not quite. Answer: [correct answer]" + explanation | Continue (correct shown) |
| S18 Rapid Fire | Wrong answer | Red flash on selected option + low tone | Auto-advance after 0.5s |
| S18 Rapid Fire | Timeout (10s) | Show correct answer in green | Auto-advance after 1s |
| S19 Memory Match | No match | Red border on both cards + low tone | Cards flip back after 1s |
| S25 Gem Shop | Insufficient coins | "Not enough coins! Need 50 coins, you have 25 coins" | [ Earn Coins -> ] |
| S25 Gem Shop | Already owned (theme) | "Already owned" | [ Apply ] replaces [ Buy ] |
| S28 Word Dialog | Word not in WM map | Word renders as plain text (no highlight) | (No error shown) |
| S32 Seva Submit | Description < 10 chars | "Please describe your activity (at least 10 characters)" | Focus text area |
| S32 Seva Submit | Network error | "Submission failed. Saved locally. Will retry when online." | Auto-retry on reconnect |
| S32 Seva Submit | Photo too large (>5MB) | "Photo too large. Please use a smaller image." | Clear photo input |
| S36 Saving Goal | Deposit > balance | "Not enough coins to deposit" | Clear input |
| S37 Teacher Login | Invalid email/password | "Invalid email or password" | Clear password field |
| S37 Teacher Login | Rate limited | "Too many attempts. Try again in 60s." | Disable button 60s |
| S37 Teacher Login | Account not found | "Account not found. Would you like to register?" | [ Register ] link |
| OTP Verification | Wrong OTP | "Invalid OTP. Please try again." | Clear OTP boxes, shake |
| OTP Verification | OTP expired | "OTP expired. Please resend." | [ Resend OTP ] button |
| S38 Teacher Dashboard | Sync failure | "Could not load latest data. Showing cached data." | [ Retry ] button |
| S40 Seva Verify | Network error on verify | "Verification saved locally. Will sync." | Auto-retry |
| Any screen | JS crash | Fallback: "Something went wrong. [ Restart Chapter ]" | Reload page |
| Any screen | localStorage full | "Storage full. Please clear old profiles." | Link to profile management |

---

## 16. Success States Catalogue

| Screen | Success Condition | Visual Feedback | Audio | Animation |
|---|---|---|---|---|
| S06 Concept Intro | Step renders | LLE highlights process | (none) | slideIn |
| S09 Interactive Drag | Correct diagonal | Green flash on canvas | Correct tone (523+659Hz) | Canvas glow |
| S12 Worked Example | Step revealed | Green left border | (none) | stepReveal |
| S13 Solve Step | Correct answer | Green feedback box | Correct tone (523Hz) | pop |
| S13 Solve Complete | All steps correct | "Problem Solved! +5 XP" | Correct tone | pulse |
| S14 Quiz | Correct answer | Green feedback + correct option highlighted | Correct tone (523->659Hz) | correct class |
| S14 Quiz | Speed bonus earned | "+3 speed" in feedback | (included in tone) | pulse |
| S15 True/False | Correct answer | Green feedback + explanation | Correct tone (523Hz) | correct class |
| S16 Fill-blank | Correct answer | Green + blank filled with answer | Correct tone (523Hz) | pop |
| S17 Progress Milestone | Concept complete | Celebration + XP + coins + ring update | Celebration tone | confetti + slideIn |
| S18 Rapid Fire | Correct answer | Green flash + streak counter | Correct tone | bslide |
| S18 Rapid Fire Complete | Round complete | Summary with score + best | Celebration tone | confetti |
| S19 Memory Match | Pair matched | Green border, cards stay open | Correct tone | pop |
| S19 Memory Match Complete | All pairs matched | Summary with moves + XP | Celebration tone | confetti |
| S20-S23 Worksheet | Question correct | Feedback + worked solution | Correct tone | pop |
| S20-S23 Worksheet Complete | All questions done | Summary with score + XP + coins | Celebration tone | confetti |
| S24 Chapter Complete | All nodes done | Full-screen celebration | Ascending arpeggio | confetti + luscale |
| S25 Gem Shop | Purchase success | "Purchased: [item]" + balance update | Correct tone | pop |
| S26 Level-Up | XP threshold crossed | Full-screen overlay with new level | Ascending arpeggio | luscale + confetti |
| S27 Badge Earned | Badge condition met | Badge shown with description | Celebration tone | pop + confetti |
| S32 Seva Submit | Activity submitted | "Submitted! Pending verification." | Correct tone | slideIn |
| S33 Seva Verified | Activity verified | Green check + coins credited | Correct tone | pop |
| S36 Saving Goal | Goal reached | "Goal Reached!" + reward | Celebration tone | confetti + luscale |
| S37 Teacher Login | Login success | Redirect to dashboard | (none) | slideIn |
| OTP Verification | OTP correct | Green checkmarks | Correct tone | pop |
| S40 Seva Verified | Activity verified | "Verified — student notified" | (none) | slideIn |

---

## 17. Animation & Feedback Reference

### 17.1 CSS Animations

| Animation | Trigger | Duration | Description |
|---|---|---|---|
| `slideIn` | New step renders | 300ms | Content slides in from right |
| `stepReveal` | Worked example step shown | 400ms | Step fades + slides up |
| `pop` | Correct answer, badge earned | 200ms | Element scales 1->1.1->1 |
| `pulse` | XP counter updates | 500ms | Element pulses subtly |
| `tpulse` | Timer bar | 1s loop | Timer bar pulses when < 3s |
| `bslide` | Rapid fire answer | 150ms | Option slides in |
| `luscale` | Level-up overlay | 600ms | Overlay scales from 0.5->1 with bounce |
| `confetti` | Milestone, badge, chapter complete | 2s | Particle burst from edges |
| `shake` | Error state (wrong answer) | 300ms | Element shakes left-right 3 times |
| `flash-green` | Correct interaction on canvas | 200ms | Canvas border flashes green |
| `flash-red` | Wrong interaction on canvas | 200ms | Canvas border flashes red |

### 17.2 Audio Feedback

| Event | Frequency | Duration | Waveform |
|---|---|---|---|
| Correct answer | 523Hz -> 659Hz | 0.12s each | sine |
| Wrong answer | 200Hz | 0.15s | sawtooth |
| Level-up | 523->659->784Hz | 0.4s | sine |
| Chapter complete | 523->659->784->1047Hz | 0.8s | sine |
| Badge earned | 659->784Hz | 0.3s | sine |
| Concept milestone | 523->659Hz | 0.12s each | sine |
| Rapid fire correct | 800Hz | 0.08s | sine |
| Rapid fire wrong | 200Hz | 0.1s | sawtooth |
| Memory match | 440Hz | 0.15s | sine |
| Shop purchase | 659->880Hz | 0.2s | sine |

### 17.3 Visual State Classes

| CSS Class | Element | Visual Effect |
|---|---|---|
| `.correct` | Quiz option, T/F button | Green background (#22c55e), white text |
| `.wrong` | Quiz option, T/F button | Red background (#ef4444), white text |
| `.selected` | Quiz option | Blue border (--blue), light blue bg |
| `.sel-true` | True button | Green border + tint |
| `.sel-false` | False button | Red border + tint |
| `.feedback.ok` | Feedback div | Green background, white text, check icon |
| `.feedback.no` | Feedback div | Red background, white text, cross icon |
| `.misconception` | Feedback sub-div | Amber background, warning icon |
| `.prop-box` | Explanation box | Light blue background, info icon |
| `.locked` | Chapter map node | Greyed out, lock icon |
| `.completed` | Chapter map node | Green border, check icon |
| `.current` | Chapter map node | Blue border, pulsing animation |
| `.btn-green` | Continue button | Green background, white text |
| `.btn-primary` | Default button | Blue background, white text |
| `.disabled` | Disabled button | Grey background, pointer: none |

---

## 18. Button Behaviour Reference

### 18.1 Global Buttons

| Button | Label | Function | Appears On | Disabled When |
|---|---|---|---|---|
| Back | Back | `App.back()` | All learning steps | At first step of first node |
| Continue | Continue -> | `App.next()` | All learning steps | Before question answered |
| Check | Check | `App.checkQuiz()` etc. | Quiz, T/F, Fill-blank, Solve | No selection/input made |
| Restart | Restart | `App.restart()` | Chapter complete screen | Never |
| Shop | Shop icon | `App.renderShop()` | Chapter hub, chapter map | Never |
| Settings | Gear icon | Navigate to S44 | Chapter hub, chapter map | Never |
| Hint | Hint | `App.useHint()` | Quiz screen | No hint tokens, or already used |

### 18.2 Continue Button State Machine

```
Step renders
  Button: disabled
  Label: "Continue ->"
    |
    v
Is this a question step (quiz/tf/fb)?
  | Yes                              | No
  v                                  v
Button: "Check"            Button: "Continue ->"
disabled                    enabled
    |                           |
User answers                  |
    |                          |
    v                          |
Button: "Check"               |
enabled                       |
    |                          |
User taps Check               |
    |                          |
    v                          |
Correct?                      |
  | Yes    | No               |
  v         v                  |
Button:     Button:            |
"Continue"  "Check"             |
green       disabled            |
enabled     (retry)             |
    |                           |
    v                           v
  App.next() -> advance to next step
```

### 18.3 Button Colours

| State | CSS Class | Background | Text |
|---|---|---|---|
| Default/Primary | `.btn-primary` | var(--blue) #3b82f6 | White |
| Success/Continue | `.btn-green` | var(--green) #22c55e | White |
| Cancel/Back | `.btn-secondary` | var(--surface) #f8fafc | var(--text) #1e293b |
| Disabled | `.btn:disabled` | #cbd5e1 (grey) | #94a3b8 (light grey) |
| Danger/Reset | `.btn-danger` | var(--red) #ef4444 | White |
| Hint | `.btn-hint` | var(--amber) #f59e0b | White |

---

## Appendix A: Screen-to-Step-Type Mapping

| Step Type (`step.t`) | Screen | Render Function | Check Function | Has Timer? | Has Input? |
|---|---|---|---|---|---|
| `intro` | S06 | `renderIntro()` | N/A | No | No |
| `text` | S07 | `render()` -> `rt()` | N/A | No | No (LLE taps) |
| `canvas_visual` | S08 | Canvas draw | N/A | No | No |
| `ix_drag` | S09 | `renderIxDrag()` | Canvas event | No | Touch/drag |
| `ix_slider` | S10 | `renderIxSlider()` | Slider input | No | Slider |
| `ix_diagonal` | S11 | `renderIxDiagonal()` | Tap vertices | No | Tap |
| `we` | S12 | `renderWE()` + `showWE()` | N/A | No | Next Step btn |
| `solve` | S13 | `renderSolve()` + `showSolveStep()` | `checkSolve()` | No | Text input |
| `quiz` | S14 | `renderQuiz()` | `checkQuiz()` | Yes (10s) | Option tap |
| `truefalse` | S15 | `renderTrueFalse()` | `checkTrueFalse()` | No | T/F tap |
| `fillblank` | S16 | `renderFillBlank()` | `checkFillBlank()` | No | Text input |
| `progress` | S17 | `renderProgress()` | N/A | No | No |
| `rapid_fire` | S18 | `renderRapidFire()` + `showRFQuestion()` | `rfAnswer()` | Yes (10s/Q) | Option tap |
| `memory_match` | S19 | `renderMemoryMatch()` | `mmCheck()` | No | Card tap |

## Appendix B: localStorage Key Map

| Key | Purpose | Written By | Read By |
|---|---|---|---|
| `aasha_profiles` | List of all child profiles | S03 Create Profile | S02 Profile Picker |
| `aasha_quad_<name>` | Per-child per-chapter progress | `App.save()` | `App.loadProgress()` |
| `aasha_economy` | Cross-chapter economy (coins, themes, items) | `App.saveEconomy()` | `App.loadEconomy()` |
| `aasha_rf_<name>` | Rapid fire best scores | `showRFSummary()` | `renderRapidFire()` |
| `aasha_daily_<name>` | Daily challenge state (Phase 2) | Daily challenge | Daily challenge |
| `aasha_seva_queue` | Offline Seva submission queue (Phase 3) | S32 Seva Submit | Sync layer |
| `aasha_settings` | User settings (sound, theme, language) | S44 Settings | `App.init()` |

## Appendix C: User Journey — Complete First-Time Experience

1. Child receives HTML file via WhatsApp/USB
2. Opens file in Chrome -> S01 Splash (1.5s)
3. -> S02 Profile Picker (empty state)
4. -> S03 Create Profile (enters name "Rahul", class 8, avatar fox)
5. -> S04 Chapter Hub (sees Quadrilaterals, Fractions, Comparing Quantities)
6. Taps "Understanding Quadrilaterals" -> S05 Chapter Map
7. All nodes locked except Node 0 -> taps Node 0
8. -> S06 Concept Intro: "What is a Polygon?"
9. [Continue] -> S07 Text with LLE (taps "polygon" -> S28 Word Dialog -> "bahubhuj")
10. [Continue] -> S08 Canvas Visual (sees shapes drawn)
11. [Continue] -> S09 Interactive Drag (drags to make a diagonal)
12. [Continue] -> S12 Worked Example (steps through problem)
13. [Continue] -> S14 Quiz (10s timer, selects answer, [Check])
    - Correct -> +5 XP + speed bonus
    - Wrong -> misconception explanation -> retry
14. [Continue] -> S15 True/False -> answer -> correct/wrong
15. [Continue] -> S16 Fill-blank -> types answer -> correct/wrong
16. [Continue] -> S17 Progress Milestone
    - +20 XP, +10 coins, confetti, celebration tone
    - Badge earned -> S27 Badge Overlay -> [Collect Badge]
    - Level up (50 XP) -> S26 Level-Up Overlay -> [Continue]
17. [Continue] -> Node 1: "Classifying Polygons" -> S06 (repeat learning cycle)
18. ... progression through all 22 nodes ...
19. Node 38: Memory Match game -> S19 (match 6 pairs)
20. Node 40: Rapid Fire -> S18 (10 questions, 10s each)
21. Nodes 32-43: Worksheet assessments (Basic -> Standard -> HOTS -> Final)
22. -> S24 Chapter Complete: full celebration, 100% progress, Chapter Champion badge
23. [Restart] to practise again, or [Back to Hub] -> S04
24. Next day: opens file -> S02 -> taps "Rahul" -> S04
    - Progress saved, resumes from where left off
25. Taps Shop -> S25 -> buys Hint Token (-20 coins)
26. Returns to chapter, uses hint in next quiz
27. Earning continues, levels rise, badges accumulate

## Appendix D: File Size Budget for Ecosystem

| Component | Size | Budget |
|---|---|---|
| Current chapter (Quadrilaterals) | 416 KB | 20 MB |
| Available for enhancement | 19.6 MB | — |
| Enhanced Canvas animations | ~500 KB | within budget |
| Enhanced CSS animations | ~50 KB | within budget |
| Inline SVG illustrations | ~200 KB | within budget |
| Web Audio soundscapes (if enhanced) | ~100 KB | within budget |
| Additional question types | ~200 KB | within budget |
| Daily challenge system | ~20 KB | within budget |
| Adaptive difficulty logic | ~50 KB | within budget |
| **Total projected (enhanced chapter)** | **~1.6 MB** | **20 MB** |

The 20 MB budget allows for significantly richer content per chapter — animated SVG diagrams, more complex Canvas interactions, richer audio feedback, adaptive learning logic — while remaining fully self-contained and offline-capable. Files should only grow as content genuinely requires it, not with irrelevant padding.

---

**End of App Flow Document**

This document specifies every screen, user action, button behaviour, navigation path, success state, error state, and empty state in the Aasha ecosystem. An AI coding agent can build any screen described here without guessing, because each interaction is specified with its trigger, behaviour, visual feedback, audio feedback, and recovery action.
