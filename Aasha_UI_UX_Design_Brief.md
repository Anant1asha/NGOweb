# Aasha — UI/UX Design Brief

**Product:** Aasha — Safe Learning. Real Impact.  
**Organisation:** Annanth Aasha Foundation  
**Document Type:** UI/UX Design Brief  
**Version:** 1.0  
**Date:** August 2026  
**Status:** Ready for Implementation  
**Purpose:** This document defines the complete visual and interaction design system for the Aasha learning platform. It is detailed enough for an AI app builder to implement without guessing. It covers the full range of users: children from Class 1 to Class 10, all boards (CBSE, ICSE, State), any school.

---

## Table of Contents

1. [Design Style](#1-design-style)
2. [Design Tokens (Source of Truth)](#2-design-tokens-source-of-truth)
3. [Color Palette](#3-color-palette)
4. [Typography](#4-typography)
5. [Component Style](#5-component-style)
6. [Layout Rules](#6-layout-rules)
7. [Mobile / Desktop Behavior](#7-mobile--desktop-behavior)
8. [Button and Card Style](#8-button-and-card-style)
9. [Dashboard Design Direction](#9-dashboard-design-direction)
10. [Iconography and Visual Language](#10-iconography-and-visual-language)
11. [Animation and Motion Design](#11-animation-and-motion-design)
12. [Accessibility](#12-accessibility)
13. [Age-Adaptive Design (Class 1 to Class 10)](#13-age-adaptive-design-class-1-to-class-10)
14. [Inspiration References](#14-inspiration-references)
15. [Overall User Experience Principles](#15-overall-user-experience-principles)
16. [Implementation Checklist](#16-implementation-checklist)

---

## 1. Design Style

### 1.1 Style Definition

**Playful Minimalism with Educational Warmth.**

Aasha's design is clean, warm, and friendly without being childish. It uses generous whitespace, rounded corners, soft shadows, and a vibrant but controlled color palette. The visual language is approachable for a Class 1 child (age 6) while not feeling patronising to a Class 10 student (age 15).

The style sits at the intersection of:
- **Duolingo's** playful feedback and gamified progress (without the heavy character branding)
- **Khan Academy's** clean educational focus and mastery-based UI
- **Google Classroom's** structured, teacher-friendly layout
- **Indian textbook design** — familiar color coding (green for correct, red for wrong) and simple iconography that children already recognise from school materials

### 1.2 Style Keywords

| Keyword | What It Means in Practice |
|---|---|
| Warm | Cream/white backgrounds, warm accent colors, no harsh blacks |
| Rounded | All corners are rounded (minimum 8px, up to 20px for cards) |
| Spacious | Generous padding (14px minimum), no cramped layouts |
| Friendly | Emoji icons used naturally, conversational tone in copy |
| Focused | One primary action per screen, one concept per step |
| Responsive | Every tappable element is minimum 44x44px |
| Joyful | Confetti, sounds, and animations celebrate learning |
| Trustworthy | Consistent patterns, predictable navigation, no dark patterns |

### 1.3 What the Design Is NOT

- Not a flat, sterile Material Design clone
- Not a skeuomorphic, textbook-page recreation
- Not a cartoon-heavy kids' app (no mascots, no animated characters talking at the child)
- Not a dense, data-heavy dashboard for the child-facing screens
- Not a dark-themed developer tool

---

## 2. Design Tokens (Source of Truth)

All design decisions flow from CSS custom properties. These are the single source of truth — every component references these tokens, never hardcoded values.

### 2.1 Core Tokens (Default Theme)

```css
:root {
  /* Brand */
  --brand: #1e40af;          /* Deep blue — primary brand identity */
  --bl: #3b82f6;             /* Bright blue — links, highlights */
  
  /* Surfaces */
  --surface: #f8fafc;        /* Page background — cool off-white */
  --surface-2: #ffffff;      /* Card background — pure white */
  --border: #e2e8f0;        /* Borders — light grey-blue */
  
  /* Text */
  --text: #1e293b;          /* Primary text — dark slate */
  --muted: #64748b;         /* Secondary text — medium grey */
  
  /* Semantic */
  --green: #10b981;          /* Correct, success, continue */
  --green-lt: #d1fae5;       /* Correct background tint */
  --green-dk: #065f46;       /* Correct text on light green */
  --red: #ef4444;             /* Wrong, error, danger */
  --red-lt: #fee2e2;         /* Error background tint */
  --red-dk: #991b1b;         /* Error text on light red */
  --gold: #f59e0b;            /* Coins, badges, achievements */
  --gold-lt: #fef3c7;        /* Achievement background tint */
  --amber: #f59e0b;           /* Warning, hints (alias of gold) */
  
  /* Accents */
  --pi: #ec4899;             /* Pink — decorative, memory match */
  --purple: #a855f7;         /* Purple — decorative, themes */
  --amb: #fef3c7;            /* Amber light — hint backgrounds */
  
  /* Dynamic (theme-able) */
  --blue: #3b82f6;           /* Changes per theme */
  --green-dyn: #10b981;      /* Changes per theme */
  --green-lt-dyn: #d1fae5;   /* Changes per theme */
  
  /* Typography */
  --font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
  --font-size-base: 16px;
  --font-size-xs: 0.65rem;   /* 10.4px — labels, captions */
  --font-size-sm: 0.75rem;   /* 12px — secondary text */
  --font-size-md: 0.85rem;   /* 13.6px — body text */
  --font-size-lg: 1rem;      /* 16px — emphasis text */
  --font-size-xl: 1.2rem;    /* 19.2px — headings */
  --font-size-2xl: 1.5rem;   /* 24px — screen titles */
  --font-size-3xl: 2rem;     /* 32px — celebration text */
  --font-size-display: 3rem; /* 48px — level-up, milestone numbers */
  
  /* Spacing */
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 14px;
  --space-lg: 20px;
  --space-xl: 32px;
  
  /* Radius */
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 14px;
  --radius-xl: 20px;
  --radius-pill: 50%;
  
  /* Shadows */
  --shadow-card: 0 2px 8px rgba(0,0,0,0.06);
  --shadow-elevated: 0 8px 24px rgba(0,0,0,0.15);
  --shadow-header: 0 -1px 3px rgba(0,0,0,0.06);
  
  /* Transitions */
  --transition-fast: 150ms ease;
  --transition-med: 300ms ease;
  --transition-slow: 600ms cubic-bezier(0.34, 1.56, 0.64, 1);
  
  /* Z-Index Scale */
  --z-base: 1;
  --z-header: 10;
  --z-overlay: 50;
  --z-modal: 100;
  --z-toast: 200;
}
```

### 2.2 Why CSS Custom Properties

1. **Theme switching is instant** — changing `body.className` updates all colors app-wide with zero re-render
2. **No build step** — works in plain HTML without Sass/Less/PostCSS
3. **Cascade naturally** — child elements inherit from parents
4. **JavaScript can read them** — `getComputedStyle()` for canvas drawing

---

## 3. Color Palette

### 3.1 Default Palette

The default palette is a warm, accessible blue-green system. It is the primary identity for Aasha.

| Role | Token | Hex | Usage |
|---|---|---|---|
| Brand Primary | `--brand` | #1e40af | Deep blue — header bar, primary buttons |
| Brand Bright | `--bl` | #3b82f6 | Bright blue — links, hover states, selected items |
| Surface | `--surface` | #f8fafc | Page background — cool off-white |
| Card | `--surface-2` | #ffffff | Card, input, and button background |
| Border | `--border` | #e2e8f0 | All borders — light grey-blue |
| Text Primary | `--text` | #1e293b | All primary text — dark slate (not pure black) |
| Text Muted | `--muted` | #64748b | Secondary text, captions, timestamps |
| Success | `--green` | #10b981 | Correct answers, continue buttons, success states |
| Success Light | `--green-lt` | #d1fae5 | Correct answer background tint |
| Error | `--red` | #ef4444 | Wrong answers, error messages, danger actions |
| Error Light | `--red-lt` | #fee2e2 | Error background tint |
| Achievement | `--gold` | #f59e0b | Coins, badges, level-up, streaks |
| Achievement Light | `--gold-lt` | #fef3c7 | Hint buttons, achievement backgrounds |
| Pink | `--pi` | #ec4899 | Decorative — memory match cards, badges |
| Purple | `--purple` | #a855f7 | Decorative — themes, gradients |

### 3.2 Color Usage Rules

1. **Background is always `--surface`** (#f8fafc). Never pure white for the page — it is too harsh on low-end screens.
2. **Cards are always `--surface-2`** (#ffffff). White cards on off-white background create natural separation.
3. **Text is always `--text`** (#1e293b). Never pure black (#000) — it creates too much contrast on cheap screens.
4. **Green means go.** Correct answers, continue buttons, and success states are always green.
5. **Red means stop.** Wrong answers and errors are always red. Never use red for decorative purposes.
6. **Gold means reward.** Coins, badges, achievements, and level-up are always gold/amber.
7. **Blue means brand.** Navigation, headers, and primary actions use brand blue.
8. **Maximum 3 accent colors per screen.** Avoid rainbow layouts.

### 3.3 Contrast Ratios

All color combinations must meet WCAG AA standards (4.5:1 for normal text, 3:1 for large text):

| Foreground | Background | Ratio | Pass? |
|---|---|---|---|
| --text (#1e293b) | --surface (#f8fafc) | 14.8:1 | AAA |
| --muted (#64748b) | --surface (#f8fafc) | 4.8:1 | AA |
| White (#fff) | --brand (#1e40af) | 8.6:1 | AAA |
| White (#fff) | --green (#10b981) | 2.8:1 | AA (large only) |
| White (#fff) | --red (#ef4444) | 3.8:1 | AA (large only) |
| --green-dk (#065f46) | --green-lt (#d1fae5) | 8.9:1 | AAA |
| --red-dk (#991b1b) | --red-lt (#fee2e2) | 7.5:1 | AAA |

**Note:** Green and red button text uses white text. For buttons with small text (< 18px), use `--brand` (deep blue) instead of `--green` for primary actions where possible, as it has better contrast.

### 3.4 Theme System

Themes are implemented by overriding the dynamic CSS custom properties on the `body` class:

```css
/* Default theme (no class on body) */
body { --blue: #3b82f6; --purple: #a855f7; --green-dyn: #10b981; }

/* Ocean theme */
body.theme-ocean { --blue: #0ea5e9; --purple: #06b6d4; --green-dyn: #14b8a6; --green-lt-dyn: #ccfbf1; --amber: #f59e0b; }

/* Forest theme */
body.theme-forest { --blue: #16a34a; --purple: #15803d; --green-dyn: #22c55e; --green-lt-dyn: #dcfce7; --amber: #ca8a04; }

/* Sunset theme */
body.theme-sunset { --blue: #f97316; --purple: #ec4899; --green-dyn: #f59e0b; --green-lt-dyn: #fef3c7; --amber: #f97316; }
```

Themes only change the **accent colors** (--blue, --purple, --green-dyn, --amber). Structural colors (--surface, --text, --border, --brand) never change. This ensures the app remains readable across all themes.

### 3.5 Subject Color Coding

When the platform expands beyond mathematics, each subject gets a signature accent color:

| Subject | Accent Color | Hex | Usage |
|---|---|---|---|
| Mathematics | Blue | #3b82f6 | Chapter cards, icons, progress bars |
| Science | Green | #10b981 | Chapter cards, icons, progress bars |
| English | Purple | #a855f7 | Chapter cards, icons, progress bars |
| Hindi | Saffron | #f59e0b | Chapter cards, icons, progress bars |
| Social Studies | Teal | #0d9488 | Chapter cards, icons, progress bars |
| General Knowledge | Pink | #ec4899 | Chapter cards, icons, progress bars |

---

## 4. Typography

### 4.1 Font Family

```css
--font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
```

**Why system fonts?**
- Zero download (critical for offline HTML files)
- Native rendering on each platform (looks right on Android, iOS, Windows)
- No FOUT (Flash of Unstyled Text)
- Consistent with the OS the child is already using
- Works on all devices including old Android phones with limited font support

**For the teacher/parent dashboard (Phase 2+, online):** Use Inter or Noto Sans from Google Fonts. These are optimized for screen reading at small sizes and have excellent Devanagari support for Hindi text.

```css
/* Dashboard only */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=Noto+Sans+Devanagari:wght@400;500;600;700&display=swap');

--font-dashboard: 'Inter', 'Noto Sans Devanagari', system-ui, sans-serif;
```

### 4.2 Type Scale

The type scale uses `rem` units so it scales with the user's browser font-size setting. The base size is 16px.

| Token | Size | rem | Usage |
|---|---|---|---|
| Display | 48px | 3rem | Level-up numbers, milestone XP |
| 3XL | 32px | 2rem | Celebration headings, large stat values |
| 2XL | 24px | 1.5rem | Screen titles, chapter names |
| XL | 19px | 1.2rem | Section headings, concept titles |
| LG | 16px | 1rem | Button text, emphasis, nav labels |
| MD | 14px | 0.85rem | Body text, feedback, explanations |
| SM | 12px | 0.75rem | Captions, secondary text, timestamps |
| XS | 10px | 0.65rem | Labels, badges, micro-text |

### 4.3 Font Weights

| Weight | Value | Usage |
|---|---|---|
| Regular | 400 | Body text, explanations, questions |
| Medium | 500 | Emphasized body text, nav labels |
| Semibold | 600 | Button text, card titles |
| Bold | 700 | Stat values, quiz options, headings |
| Extrabold | 800 | Level names, badge names, achievement values |
| Black | 900 | Rapid fire stat values, display numbers |

### 4.4 Line Height

| Context | Line Height | Why |
|---|---|---|
| Body text | 1.5 | Comfortable reading for children |
| Headings | 1.2 | Tight, scannable |
| Buttons | 1 | Single-line, compact |
| LLE text | 1.6 | Extra space for tappable word highlighting |

### 4.5 Hindi / Devanagari Text

- Use `Noto Sans Devanagari` for the dashboard (loaded from Google Fonts)
- For offline chapter files, the system font fallback handles Devanagari on Android (Noto Sans is preinstalled)
- Hindi text in LLE word dialogs uses the same type scale but slightly larger (multiply by 1.1x) because Devanagari glyphs are visually smaller at the same font size
- Hindi connective text in parentheses is always `--muted` color and `--font-size-sm`

### 4.6 Text Color Rules

| Context | Color | Token |
|---|---|---|
| Primary text | Dark slate | --text |
| Secondary/caption | Medium grey | --muted |
| Correct feedback text | Dark green | --green-dk |
| Error feedback text | Dark red | --red-dk |
| Achievement text | Gold | --gold |
| Link/interactive | Bright blue | --bl |
| LLE highlighted word | Blue, underined | --bl with dotted underline |
| Disabled text | Light grey | #94a3b8 |

---

## 5. Component Style

### 5.1 Component Inventory

| Component | Variants | Used In |
|---|---|---|
| Button | primary, green, amber, secondary, danger, hint, disabled | All screens |
| Card | content, game, worked-example, shop-item, profile | All screens |
| Input | text (fill-blank), number (solve), OTP | Quiz, fill-blank, solve, login |
| Quiz Option | default, selected, correct, wrong, disabled | Quiz, worksheet |
| True/False Button | default, sel-true, sel-false, disabled | True/false |
| Feedback Box | ok (green), no (red), misconception (amber) | All answer screens |
| Progress Bar | linear, ring (SVG) | Chapter map, milestones |
| Timer Bar | green (>6s), amber (3-6s), red (<3s) | Quiz, rapid fire |
| Badge Chip | difficulty badge, stat chip, earned badge | Quiz, progress, dashboard |
| Dialog/Modal | word dialog, badge overlay, level-up overlay, confirmation | LLE, celebrations, settings |
| Memory Card | front (gradient), back (white/content), matched (green) | Memory match |
| Shop Item Card | purchasable, owned, insufficient | Gem shop |
| Stat Display | XP, coins, streak, level | Header, milestones, dashboard |
| Avatar | default, framed (gold, star) | Profile, header |
| Node Icon | completed, current, locked | Chapter map |
| Tooltip | info, mastery | Chapter map |

### 5.2 Component Design Rules

**Every component must:**
1. Use CSS custom properties (never hardcoded values)
2. Have clearly defined states (default, hover, active, disabled, error)
3. Have minimum 44x44px touch target
4. Use `touch-action: manipulation` to remove 300ms tap delay on mobile
5. Use `user-select: none` on interactive elements to prevent text selection on tap
6. Have `transition` on state changes (150-300ms, ease)

---

## 6. Layout Rules

### 6.1 Container

The app uses a single-column, centered layout:

```css
.container {
  max-width: 480px;      /* Mobile-first, readable line length */
  margin: 0 auto;         /* Centered */
  padding: 0 14px;        /* Horizontal padding */
  min-height: 100vh;      /* Full viewport height */
}
```

**Why 480px max-width?**
- Most Android phones in the target audience are 360-414px wide
- A 480px container fills the screen on phones and creates a readable column on tablets/desktops
- Content wider than 480px reduces reading comprehension for children

### 6.2 Vertical Structure

```
┌──────────────────────────────────┐
│  HEADER (fixed, 56px)             │  ← Back, title, stats, timer
├──────────────────────────────────┤
│                                   │
│  CONTENT (scrollable)             │  ← Step content, canvas, questions
│                                   │
│                                   │
├──────────────────────────────────┤
│  ACTION BAR (fixed, 64px)         │  ← Continue/Check button, Hint
└──────────────────────────────────┘
```

- **Header:** Fixed at top, 56px tall. Contains back button, step title, progress indicator, and stats (XP, coins, streak). Background is white with a bottom shadow.
- **Content:** Scrollable area between header and action bar. Padding: 14px. All content is centered with `max-width: 400px` for readability.
- **Action Bar:** Fixed at bottom, 64px tall. Contains the primary action button (Continue/Check). Background is white with a top shadow.

### 6.3 Spacing System

All spacing uses a 4px base unit:

| Token | Value | Usage |
|---|---|---|
| --space-xs | 4px | Tight gaps between small elements (icon + label) |
| --space-sm | 8px | Element spacing within a card |
| --space-md | 14px | Card padding, default element spacing |
| --space-lg | 20px | Section spacing, card-to-card gap |
| --space-xl | 32px | Major section breaks, screen-level padding |

### 6.4 Alignment Rules

1. **All text is left-aligned** by default. Center alignment only for:
   - Headings in intro/milestone screens
   - Stat values and numbers
   - Buttons (text is centered within button)
   - Celebration overlays
2. **All cards are centered** with `margin: 0 auto` and `max-width: 400px`
3. **All interactive options** (quiz options, T/F buttons) span full width within the card
4. **Canvas elements** are centered with `margin: 0 auto` and `max-width: 360px`

### 6.5 Grid for Chapter Map

The chapter map uses a horizontal scroll layout (not a grid):

```
Node0 — Node1 — Node2 — Node3 — ...
✅      ✅      🔓      🔒
```

- Each node is 60x60px (icon circle)
- Connected by a 2px dashed line (--border color)
- Completed nodes: green border, checkmark
- Current node: blue border, pulsing animation
- Locked nodes: grey, lock icon
- Horizontal scroll with `overflow-x: auto`
- On mobile, shows 3-4 nodes at a time

### 6.6 Grid for Memory Match

Memory match uses a 4x3 grid (for 12 cards / 6 pairs):

```css
.mm-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
  max-width: 320px;
  margin: 0 auto;
}
```

Each card uses `aspect-ratio: 1` to maintain squares.

### 6.7 Grid for Shop

Shop items use a 2-column grid:

```css
.shop-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}
```

Consumables, themes, and frames are grouped by category headers.

---

## 7. Mobile / Desktop Behavior

### 7.1 Mobile-First (Primary Target)

**Target devices:**
- Android phones (360px - 414px wide)
- Shared devices (multiple children, one phone)
- Chrome 90+ or Firefox 88+
- Touch input only (no mouse, no keyboard)
- Portrait orientation (landscape not optimised)

**Mobile behaviors:**
- All tap targets minimum 44x44px (Apple HIG standard)
- `touch-action: manipulation` on body (removes 300ms delay)
- `user-scalable=no` in viewport meta (prevent zoom on input focus)
- `overflow-x: hidden` on body (prevent horizontal scroll)
- No hover states (hover is meaningless on touch — use `:active` for tap feedback)
- Canvas elements use `touch-action: none` and custom touch handlers
- Long press (>500ms) for secondary actions (e.g., show tooltip on chapter map)
- Swipe left/right not used for navigation (children may accidentally swipe)

### 7.2 Tablet (Secondary Target)

- 768px - 1024px wide
- Same 480px max-width container, centered with more whitespace
- Touch input (no mouse expected)
- Can use slightly larger touch targets (48x48px) for comfort

### 7.3 Desktop (Teacher/Parent Dashboard Only)

- 1280px+ wide
- Multi-column layout for dashboard
- Mouse and keyboard input
- Hover states enabled
- Keyboard shortcuts (Enter to submit, Escape to close dialogs)
- The child-facing chapter files are NOT optimized for desktop — they work but the layout is centered and narrow

### 7.4 Responsive Breakpoints

The chapter HTML files use NO media queries (to keep file size small and behavior consistent). The 480px container with `margin: 0 auto` naturally adapts:

| Screen Width | Behavior |
|---|---|
| 320px (small Android) | Container fills screen, padding 10px, slightly tighter spacing |
| 360-414px (standard Android) | Container fills screen, padding 14px, default spacing |
| 768px+ (tablet/desktop) | Container is 480px centered, large margins on sides |

For the teacher/parent dashboard (Phase 2+), use standard breakpoints:

```css
/* Mobile first */
.dashboard { grid-template-columns: 1fr; }

@media (min-width: 768px) {
  .dashboard { grid-template-columns: 1fr 1fr; }
}

@media (min-width: 1024px) {
  .dashboard { grid-template-columns: 1fr 1fr 1fr; }
}
```

### 7.5 Safe Area Handling

For devices with notches or rounded corners:

```css
body {
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
}
```

---

## 8. Button and Card Style

### 8.1 Button System

All buttons share a base style:

```css
.btn {
  width: 100%;              /* Full width within container */
  padding: 13px;            /* Comfortable tap target */
  border: none;             /* No default borders */
  border-radius: 13px;     /* Rounded but not pill */
  font-size: 0.95rem;      /* ~15px — readable */
  font-weight: 600;         /* Semibold */
  font-family: inherit;     /* System font */
  cursor: pointer;
  transition: all 150ms ease;
  touch-action: manipulation;
  user-select: none;
  -webkit-tap-highlight-color: transparent;  /* No blue flash on tap */
}

.btn:active {
  transform: scale(0.97);   /* Slight press-down effect */
}

.btn:disabled {
  background: #cbd5e1;      /* Grey */
  color: #94a3b8;           /* Light grey text */
  cursor: not-allowed;
  transform: none;
}
```

**Button variants:**

| Variant | Class | Background | Text Color | Usage |
|---|---|---|---|---|
| Primary | `.btn-primary` | var(--brand) #1e40af | White | Default action, Check button |
| Success | `.btn-green` | var(--green) #10b981 | White | Continue after correct answer |
| Warning | `.btn-amber` | var(--gold) #f59e0b | White | Hint, streak freeze |
| Secondary | `.btn-secondary` | var(--surface) #f8fafc | var(--text) | Cancel, back |
| Danger | `.btn-danger` | var(--red) #ef4444 | White | Reset, delete |
| Back | `.btn-back` | #f1f5f9 | #475569 | Back navigation (smaller, with border) |
| Restart | `.btn-restart` | linear-gradient(135deg, #f59e0b, #ef4444) | White | Restart chapter (special gradient) |

**Button rules:**
1. Only ONE primary/green button per screen — it is the main action
2. Secondary buttons are always smaller and less visually prominent
3. Disabled buttons are grey — never hidden (child must see the button exists but isn't ready)
4. Button text is always imperative: "Check", "Continue", "Buy", "Submit" — not "OK" or "Click"
5. Button width is 100% on mobile, max 300px on tablet/desktop
6. The [Check] button starts disabled and becomes enabled when the child selects/enters an answer
7. The [Continue] button changes from blue (disabled state shows "Check") to green (enabled after correct answer)

### 8.2 Card System

All cards share a base style:

```css
.card {
  background: var(--surface-2);    /* White */
  border: 2px solid var(--border);  /* Light grey-blue border */
  border-radius: 14px;             /* Rounded */
  padding: 14px;                   /* Comfortable padding */
  max-width: 400px;                /* Readable width */
  margin: 0 auto 8px;              /* Centered with bottom gap */
  box-shadow: var(--shadow-card);  /* Subtle shadow */
}
```

**Card variants:**

| Variant | Class | Border | Shadow | Usage |
|---|---|---|---|---|
| Content card | `.card` | 2px solid --border | --shadow-card | Intro, text, visual explanations |
| Game card | `.game-card` | 2px solid --border | --shadow-card | Interactive canvas steps |
| Worked example | `.we-card` | 2px solid --border | --shadow-card | Worked examples, solve problems |
| Quiz option | `.quiz-opt` | 2px solid --border | none | Quiz answer options |
| Shop item | `.shop-item` | 2px solid --border | none | Gem shop items |
| Profile card | `.profile-card` | 2px solid --border | --shadow-card | Profile picker entries |
| Chapter card | `.chapter-card` | 2px solid --border | --shadow-card | Chapter hub entries |
| Milestone card | `.milestone-card` | none | --shadow-elevated | Progress milestone (elevated) |

**Card state modifiers:**

| State | Class | Visual |
|---|---|---|
| Selected | `.selected` | Border changes to --bl, light blue background |
| Correct | `.correct` | Border changes to --green, green background |
| Wrong | `.wrong` | Border changes to --red, red background |
| Locked | `.locked` | Greyed out, 50% opacity, lock icon |
| Completed | `.completed` | Green border, check icon |
| Current | `.current` | Blue border, pulsing animation |
| Disabled | `.disabled` | Grey background, no pointer events |

**Card rules:**
1. Always have a 2px border (not 1px — 1px is too thin on low-DPI screens)
2. Never use border without background color (borders alone are insufficient on bright screens)
3. Shadows are subtle (0.06 opacity) — they suggest depth without being heavy
4. Card max-width is 400px for readability (longer lines reduce comprehension)
5. Card-to-card gap is 8px (--space-sm)

### 8.3 Input Fields

```css
.fb-input {
  width: 100%;
  max-width: 380px;
  margin: 0 auto 8px;
  display: block;
  padding: 12px 16px;
  border: 2px solid var(--border);
  border-radius: 12px;
  font-size: 1rem;           /* 16px — prevents iOS zoom on focus */
  font-family: inherit;
  background: white;
  color: var(--text);
  transition: border-color 150ms ease;
}

.fb-input:focus {
  border-color: var(--brand);
  outline: none;
}

.fb-input:disabled {
  background: var(--surface);
  color: var(--muted);
}
```

**Input rules:**
1. Font size is always 16px (1rem) to prevent iOS auto-zoom on focus
2. Border is 2px (same as cards)
3. Focus state changes border to brand color (no outline — outline is ugly on mobile)
4. Placeholder text is --muted color
5. Input is disabled after answer is checked (prevent changes)

### 8.4 Feedback Boxes

```css
.feedback {
  padding: 12px;
  border-radius: 12px;
  margin: 8px auto;
  max-width: 400px;
  font-size: 0.85rem;
}

.feedback.ok {
  background: var(--green-lt);    /* #d1fae5 */
  border: 1px solid #86efac;
  color: var(--green-dk);         /* #065f46 */
}

.feedback.no {
  background: var(--red-lt);      /* #fee2e2 */
  border: 1px solid #fca5a5;
  color: var(--red-dk);           /* #991b1b */
}

.misconception {
  background: var(--red-lt);
  border: 1px solid #fca5a5;
  border-radius: 10px;
  padding: 10px;
  margin: 8px auto;
  font-size: 0.78rem;
}

.prop-box {
  background: #f0fdf4;
  border-left: 4px solid var(--green);
  padding: 8px 12px;
  border-radius: 0 8px 8px 0;
  font-size: 0.78rem;
}
```

---

## 9. Dashboard Design Direction

### 9.1 Teacher Dashboard (Phase 2+)

The teacher dashboard is a separate web application (not part of the chapter HTML). It uses a multi-column layout optimized for desktop.

**Layout:**

```
┌─────────────────────────────────────────────────────────┐
│  TOP BAR: Logo | Teacher Name | Class | [Logout]          │
├──────────┬────────────────────────────────────────────────┤
│          │                                                  │
│  SIDEBAR │  MAIN CONTENT AREA                               │
│          │                                                  │
│  - Home  │  ┌──────────────┐ ┌──────────────┐ ┌────────┐│
│  - Stud. │  │ Class Stats  │ │ Engagement   │ │ Misconc.││
│  - Seva  │  │ 32 students  │ │ [Bar Chart]  │ │ Top 5   ││
│  - Reports│ │ 45% avg      │ │              │ │         ││
│  - Settings│ └──────────────┘ └──────────────┘ └────────┘│
│          │                                                  │
│          │  ┌────────────────────────────────────────────┐│
│          │  │ Students Needing Help                     ││
│          │  │ RED Rahul — Stuck on Diagonals            ││
│          │  │ YELLOW Priya — No activity 4 days          ││
│          │  └────────────────────────────────────────────┘│
│          │                                                  │
└──────────┴────────────────────────────────────────────────┘
```

**Dashboard design principles:**
1. **Sidebar navigation** (240px wide, fixed) with icons + labels
2. **Card-based content** — each metric is a card with a chart or table
3. **Color-coded status** — RED (needs help), YELLOW (slow), GREEN (on track)
4. **Charts use Recharts** (React) — bar charts, line charts, donut charts
5. **Tables are striped** — alternating row backgrounds (--surface and white)
6. **No data visualization in the child-facing app** — all charts live in the dashboard
7. **Export to CSV** button on every report
8. **Print-friendly** — teachers may print progress reports for parent meetings

### 9.2 Parent Dashboard

Simpler than teacher dashboard — single column, mobile-optimized:

```
┌────────────────────────────────────┐
│  Parent: Rajesh          [Logout]   │
│  Child: Rahul                       │
├────────────────────────────────────┤
│                                     │
│  "Rahul's Learning This Week"       │
│                                     │
│  ┌──────────────────────────────┐  │
│  │ Concepts learned: 3          │  │
│  │ Concepts mastered: 2         │  │
│  │ Time spent: 1h 30m           │  │
│  │ Daily streak: 4 days          │  │
│  └──────────────────────────────┘  │
│                                     │
│  "Rahul is doing well! He's        │
│  strong on polygon properties      │
│  but finding angle sums            │
│  challenging."                     │
│                                     │
│  Understanding (not marks):         │
│  Polygon: GREEN  Diagonals: GREEN  │
│  Angles: YELLOW  Parallelogram: ?? │
│                                     │
│  [ View Detailed Report ]           │
│                                     │
└────────────────────────────────────┘
```

**Parent dashboard principles:**
1. **Plain language** — no jargon, no percentages without context
2. **Understanding over scores** — show what the child understands, not just marks
3. **Suggestions included** — "Try asking Rahul about shapes around the house"
4. **No surveillance** — no time-tracking charts, no screen-time shaming
5. **Mobile-first** — parents view on their phones

---

## 10. Iconography and Visual Language

### 10.1 Icon System

Aasha uses **emoji as primary icons**. This is a deliberate choice:

**Why emoji?**
- Zero bytes (no icon font, no SVG files, no image assets)
- Universally recognized by children (they use emoji daily)
- Cross-platform consistent (rendered by the OS)
- Colorful and engaging without design effort
- Work offline (part of Unicode, not loaded from a CDN)

**Where emoji are used:**

| Context | Emoji | Purpose |
|---|---|---|
| Concept nodes | Geometric shapes (⬢ ⬣ ◐ ⬡ ■ 🪁 🏆) | Visual concept identity |
| Navigation | ⚙ (settings), 🛒 (shop), ⬅ (back) | Universal actions |
| Feedback | ✓ (correct), ✗ (wrong), ⚠ (warning) | Answer feedback |
| Gamification | 🪙 (coins), ⭐ (XP), 🔥 (streak), 🏆 (trophy) | Progress and rewards |
| Levels | 🌱 📘 📚 🎓 🏆 | Level progression |
| Badges | 👣 📐 ⚖️ 🌳 🏆 | Achievement types |
| Shop items | 💡 🛡️ 🌊 🌲 🌅 🥇 ⭐ | Shop inventory |
| Subjects | 📐 (math), 🔬 (science), 📖 (English) | Subject identification |

**Rules for emoji use:**
1. Never use emoji that could be culturally insensitive
2. Use geometric/symbolic emoji for concepts (not faces/people)
3. Keep emoji usage consistent — the same concept always uses the same emoji
4. Limit to 1-2 emoji per heading (avoid emoji soup)
5. Use emoji at `font-size: 1.2rem` minimum for visibility

### 10.2 Canvas Visuals

All educational diagrams are drawn programmatically using Canvas 2D API:

**Drawing conventions:**
- Shape outlines: 2.5px stroke width, `strokeStyle` from CSS variable
- Fill: semi-transparent fills (rgba with 0.15-0.25 alpha)
- Labels: 14px font, centered, `fillStyle` from --text
- Vertices: 8px radius circles, filled with --bl
- Diagonals: 2px dashed lines, --purple color
- Angles: arc with 3px stroke, --gold color

### 10.3 Illustration Style

Aasha does NOT use:
- Photographs (too large for offline files, culturally specific)
- Cartoon characters or mascots (feels patronising to older children)
- Stock illustrations (too large, licensing concerns)
- Decorative backgrounds or patterns (wastes space, distracts)

Aasha DOES use:
- Programmatic Canvas drawings (shapes, diagrams, charts)
- Emoji as icons
- CSS gradients for decorative elements (memory card fronts, level-up overlay)
- CSS animations for feedback (confetti, pulse, shake)

---

## 11. Animation and Motion Design

### 11.1 Motion Principles

1. **Motion has meaning** — every animation communicates a state change, not decoration
2. **Fast feedback** — tap responses are instant (0ms delay), animations are 150-300ms
3. **Celebration is earned** — confetti and level-up animations only appear on genuine achievements
4. **Never block interaction** — animations are non-blocking; the child can always tap during an animation
5. **Respect reduced motion** — `@media (prefers-reduced-motion: reduce)` disables all non-essential animations

### 11.2 Animation Catalogue

| Name | Duration | Easing | Trigger | Effect |
|---|---|---|---|---|
| slideIn | 300ms | ease-out | New step renders | Content slides in from right (translateX) |
| stepReveal | 400ms | ease-out | Worked example step shown | Step fades in + slides up (translateY + opacity) |
| pop | 200ms | ease-out | Correct answer, badge earned | Element scales 1 → 1.1 → 1 |
| pulse | 500ms | ease-in-out | XP counter updates | Element scales 1 → 1.05 → 1 |
| tpulse | 1s loop | ease-in-out | Timer < 3s remaining | Timer bar pulses opacity 0.6 → 1.0 |
| bslide | 150ms | ease-out | Rapid fire answer | Option slides in from bottom |
| luscale | 600ms | cubic-bezier(0.34, 1.56, 0.64, 1) | Level-up overlay | Overlay scales 0.5 → 1 with bounce |
| confetti | 2000ms | linear | Milestone, badge, chapter complete | Particle burst from screen edges |
| shake | 300ms | ease-in-out | Wrong answer, error | Element translates X: -4, 4, -4, 4, 0 |
| flash-green | 200ms | ease-out | Correct canvas interaction | Canvas border flashes green opacity |
| flash-red | 200ms | ease-out | Wrong canvas interaction | Canvas border flashes red opacity |
| flip | 400ms | ease-out | Memory match card flip | Card rotates Y 0 → 180deg (3D transform) |

### 11.3 Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 12. Accessibility

### 12.1 Visual Accessibility

| Requirement | Implementation |
|---|---|
| WCAG AA contrast (4.5:1 text) | All color pairs verified (see Section 3.3) |
| Minimum tap target 44x44px | All buttons, options, and cards meet this |
| Focus visible | Focused elements show 2px --bl outline |
| Not color-dependent | Correct/wrong states use icons (✓/✗) in addition to colors |
| Font size | Base 16px, scalable via browser settings (rem units) |
| No text in images | All text is real HTML text (except canvas labels) |

### 12.2 Motor Accessibility

| Requirement | Implementation |
|---|---|
| Large tap targets | Minimum 44x44px, most are 48x48px+ |
| No precision gestures | Drag interactions have 20px tolerance |
| No time-critical UI (except rapid fire) | Rapid fire is optional, all other steps are untimed |
| No swipe navigation | All navigation via buttons (prevents accidental swipes) |
| Touch delay removed | `touch-action: manipulation` on body |

### 12.3 Cognitive Accessibility

| Requirement | Implementation |
|---|---|
| One task per screen | Each step has one concept, one question, one action |
| Clear next action | Continue/Check button is always the largest, most visible element |
| Predictable navigation | Back is always top-left, Continue is always bottom-center |
| Error recovery | Wrong answers explain why, not just "wrong" |
| Progress visible | Progress bar/ring on every screen |
| Simple language | LLE bridges English with Hindi; copy is written at Class 5 reading level |
| No information overload | Maximum 5 options per question, maximum 8 steps per concept |

### 12.4 Screen Reader Considerations

- Use semantic HTML (`<button>`, `<dialog>`, `<progress>`)
- Canvas elements have `aria-label` describing the visual
- Progress is announced via `aria-valuenow` and `aria-valuemax`
- Dialogs use `role="dialog"` and `aria-modal="true"`
- Form inputs have `aria-label`
- Dynamic content updates use `aria-live="polite"` for feedback messages

---

## 13. Age-Adaptive Design (Class 1 to Class 10)

### 13.1 The Challenge

Aasha serves children from age 6 (Class 1) to age 15 (Class 10). The design must be engaging for a 6-year-old while not feeling childish to a 15-year-old.

### 13.2 Approach: Same System, Different Content

The UI system (colors, typography, components, layout) is **identical across all classes**. What changes is the **content density and visual emphasis**:

| Aspect | Class 1-3 (Age 6-9) | Class 4-7 (Age 9-13) | Class 8-10 (Age 13-15) |
|---|---|---|---|
| Font size | 1.1x larger (base 18px) | Standard (base 16px) | Standard (base 16px) |
| Tap targets | Minimum 48x48px | Minimum 44x44px | Minimum 44x44px |
| Content per screen | 1 concept, 1 question | 1 concept, 2-3 questions | 1 concept, 3-5 questions |
| Visual ratio | More canvas, less text | Balanced | More text, fewer visuals |
| LLE depth | More Hindi words highlighted | Moderate | Key terms only |
| Gamification emphasis | High (sounds, confetti, badges) | High | Moderate (XP, levels, but less confetti) |
| Copy tone | Simple sentences, short words | Standard | Standard with technical terms |
| Emoji density | Higher (more playful) | Moderate | Lower (more professional) |
| Color vibrancy | More saturated | Standard | Standard |

### 13.3 Implementation

The age adaptation is driven by the **profile class level**:

```javascript
App.fontSizeMultiplier = (App.profile.class <= 3) ? 1.1 : 1.0;
App.minTapTarget = (App.profile.class <= 3) ? 48 : 44;
App.confettiEnabled = (App.profile.class <= 7);
```

CSS adapts via a body class:
```css
body.young-learner { font-size: 17.6px; }  /* 16px * 1.1 */
body.young-learner .btn { padding: 16px; }  /* Larger tap target */
```

### 13.4 Universal Design Rules (All Ages)

Regardless of class level, these rules are universal:
1. Green = correct, Red = wrong (culturally recognized in Indian schools)
2. One primary button per screen
3. Back button is always top-left
4. Progress is always visible
5. Wrong answers always explain why
6. No jargon without explanation (LLE covers this)
7. No timer on regular questions (only on rapid fire, which is optional)
8. Never punish a wrong answer (no losing coins, no losing XP)

---

## 14. Inspiration References

### 14.1 Direct Inspirations

| Reference | What We Take | What We Don't Take |
|---|---|---|
| **Duolingo** | Gamification feedback (confetti, sounds, streaks), progress bar, daily streak, shop economy | Heavy mascot branding, competitive leaderboards, push notifications |
| **Khan Academy** | Mastery-based learning, skill map, "practice until you get it", clean educational UI | Video-first approach, account-required, English-only |
| **Google Classroom** | Clean card layout, teacher/parent dashboard structure, assignment flow | Enterprise feel, Google account dependency |
| **BYJU'S** | Visual-first learning, interactive diagrams, Indian education context | Aggressive marketing, expensive, video-heavy (too large for offline) |
| **AglaSem Playground** | Timed quiz, rapid fire, memory match, gem shop, daily challenge, XP/gems economy | Competitive leaderboards, internet dependency, ads |
| **NCERT Textbooks** | Color coding (green/red), simple diagrams, bilingual approach, curriculum alignment | Dense text, no interactivity, static |

### 14.2 Design System References

| Reference | What We Take |
|---|---|
| **Tailwind CSS** | Color palette naming convention (slate, emerald, amber), utility-first approach for dashboard |
| **Material Design 3** | Touch target sizes (44dp), elevation system, ripple effect concept (simplified) |
| **Apple HIG** | Minimum 44x44pt tap targets, safe area insets, `touch-action: manipulation` |
| **Fluent Design (Microsoft)** | Acrylic/translucency for overlays (simplified to solid colors for performance) |

### 14.3 Indian Design Context

| Reference | What We Take |
|---|---|
| **Indian textbook design** | Green = correct, red = wrong, simple line diagrams, bilingual labels |
| **UPI / PhonePe UI** | Large tap targets, simple language, mobile-first, trust-building patterns |
| **DigiLocker** | Clean government-grade UI, simple forms, OTP flow |

---

## 15. Overall User Experience Principles

### 15.1 The 10 UX Commandments of Aasha

1. **The child is never stuck.** Every screen has a clear next action. If a child doesn't know the answer, the wrong-answer feedback explains it. If a child is lost, the Back button is always visible.

2. **The child is never punished.** Wrong answers reset the streak but never deduct coins or XP. Learning from mistakes is the design — not penalising them.

3. **The child always sees progress.** Every screen shows where the child is (header progress), where they've been (completed nodes), and where they're going (next concept).

4. **The child always gets feedback.** Every tap produces visual + audio feedback. No silent interactions. No wondering "did it register?"

5. **The child is never overwhelmed.** One concept, one question, one action per screen. If a concept is complex, it is broken into smaller steps.

6. **The child is respected.** No talking down, no excessive animation, no "baby" language. A Class 8 child is a young adult, not a toddler.

7. **The child is protected.** No external links, no ads, no tracking, no social features, no data collection. The child's attention is for learning, not for extraction.

8. **The child is motivated, not manipulated.** Gamification reinforces learning effort, not screen time. Coins are earned by understanding, not by tapping. There is no "spin the wheel" or random reward.

9. **The child can leave and return.** Progress saves automatically. The child can close the app at any step and resume exactly where they left off. No "you'll lose your progress" warnings.

10. **The child can learn in their language.** LLE bridges English with Hindi. Academic words are tappable for Hindi meanings. Connective words show inline Hindi. The child builds English competence through familiar-language support, not by replacing English.

### 15.2 Tone of Voice

| Context | Tone | Example |
|---|---|---|
| Concept introduction | Warm, clear, encouraging | "Let's discover what a polygon is!" |
| Explanation | Simple, direct, no jargon | "A polygon is a closed shape made of straight lines." |
| Correct feedback | Celebratory but brief | "Correct! +5 XP" |
| Wrong feedback | Gentle, explanatory, never negative | "Not quite. A rhombus has all sides equal, but its angles are not 90 degrees." |
| Empty state | Inviting, not shaming | "No chapters yet! Ask your teacher to share one." |
| Error | Helpful, actionable | "Something went wrong. Tap Restart Chapter to try again." |
| Achievement | Proud, specific | "Chapter Champion! You completed the entire chapter." |
| Navigation | Imperative, clear | "Continue", "Back", "Check", "Start" |

### 15.3 Copy Rules

1. **Maximum 15 words per instruction.** Children scan, they don't read paragraphs.
2. **Use "you" not "the student."** Direct, personal, not clinical.
3. **Use active voice.** "Tap the blue dot" not "The blue dot should be tapped."
4. **No exclamation marks in feedback.** "Correct! +5 XP" — the exclamation is in the green color and the XP, not the punctuation.
5. **Hindi connective words in parentheses.** "because (kyunki)", "therefore (isliye)" — inline, not separate.
6. **No abbreviations.** "Question 3 of 10" not "Q3/10". "Experience" not "XP" (in copy; "XP" is fine as a stat label).
7. **Numbers are always digits.** "4 sides" not "four sides". "360 degrees" not "three hundred sixty degrees".

---

## 16. Implementation Checklist

For an AI app builder implementing this design system, verify:

### 16.1 Setup

- [ ] CSS custom properties defined in `:root` (Section 2.1)
- [ ] Theme classes on `body` (`theme-ocean`, `theme-forest`, `theme-sunset`) (Section 3.4)
- [ ] Viewport meta tag: `width=device-width,initial-scale=1.0,user-scalable=no`
- [ ] Body: `font-family: system-ui, -apple-system, sans-serif; touch-action: manipulation;`
- [ ] Container: `max-width: 480px; margin: 0 auto;`

### 16.2 Colors

- [ ] All colors reference CSS variables, never hardcoded hex values
- [ ] Green/red/blue/gold tokens used per their semantic meaning (Section 3.2)
- [ ] Contrast ratios verified (Section 3.3)

### 16.3 Typography

- [ ] Type scale uses rem units (Section 4.2)
- [ ] Font weights: 400 (body), 600 (buttons), 700 (stats), 800 (headings), 900 (display)
- [ ] Line height: 1.5 (body), 1.2 (headings), 1.6 (LLE text)
- [ ] Input font-size: 16px (prevents iOS zoom)

### 16.4 Components

- [ ] Buttons: 100% width, 13px padding, 13px radius, 0.95rem font, scale(0.97) on active
- [ ] Cards: 2px border, 14px radius, 14px padding, max-width 400px, centered
- [ ] Inputs: 2px border, 12px radius, 16px font, border-color changes on focus
- [ ] Feedback: green/red background tints with matching text colors
- [ ] All tap targets minimum 44x44px

### 16.5 Layout

- [ ] Header: fixed top, 56px, white background, bottom shadow
- [ ] Action bar: fixed bottom, 64px, white background, top shadow
- [ ] Content: scrollable, 14px padding, centered
- [ ] No horizontal scroll (overflow-x: hidden on body)

### 16.6 Interactions

- [ ] `touch-action: manipulation` on body
- [ ] `-webkit-tap-highlight-color: transparent` on all interactive elements
- [ ] `user-select: none` on interactive elements
- [ ] Transitions: 150ms (fast), 300ms (medium), 600ms with bounce (celebration)
- [ ] Active state: `transform: scale(0.97)` on buttons

### 16.7 Accessibility

- [ ] Contrast ratios meet WCAG AA
- [ ] Tap targets minimum 44x44px
- [ ] Semantic HTML (`<button>`, `<dialog>`, `<progress>`)
- [ ] ARIA labels on canvas elements
- [ ] `prefers-reduced-motion` media query disables animations
- [ ] Not color-dependent (icons accompany color states)

### 16.8 Age Adaptation

- [ ] Body class `young-learner` for Class 1-3 (larger font, larger tap targets)
- [ ] Content density appropriate for class level
- [ ] LLE word map depth appropriate for class level

---

**End of UI/UX Design Brief**

This document defines the complete visual and interaction design system for the Aasha learning platform. An AI app builder can implement any screen, component, or interaction described here without guessing, because every color, font size, spacing value, animation duration, and interaction state is specified.
