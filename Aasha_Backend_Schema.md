# Aasha — Complete Backend Schema

**Product:** Aasha — Safe Learning. Real Impact.  
**Organisation:** Annanth Aasha Foundation  
**Document Type:** Backend Database Schema  
**Version:** 1.0  
**Date:** August 2026  
**Status:** Ready for Implementation  
**Author:** Senior Backend Engineer  

**Database:** PostgreSQL 16  
**Character Set:** UTF-8 (full Unicode for Devanagari, emoji, all Indian scripts)  
**Timezone:** All timestamps stored as `TIMESTAMPTZ` in UTC; converted to IST (Asia/Kolkata) at presentation layer  

**Core Design Principles:**
1. Every table has a surrogate primary key (`UUID`, `gen_random_uuid()`) — never expose sequential integers as identifiers
2. Every table has `created_at` and `updated_at` audit columns
3. Every table has row-level ownership enforced via policy, not just application logic
4. All foreign keys are `ON DELETE RESTRICT` by default; `CASCADE` only when the child has no independent meaning
5. Soft deletes via `deleted_at TIMESTAMPTZ` — no hard deletes except via explicit admin purge job
6. JSON columns (`JSONB`) are used for flexible/nested data that doesn't need relational queries (badge maps, step configs, theme settings). Relational columns are used for anything that needs indexing, aggregation, or joins
7. All PII (child names, phone numbers, emails) is encrypted at rest via `pgcrypto`
8. The schema supports the offline-first architecture: the server is a sync target, never a runtime dependency. A child can learn for months without the server existing

---

## Table of Contents

1. [Schema Overview & ERD](#1-schema-overview--erd)
2. [Enum Types](#2-enum-types)
3. [User Management Tables](#3-user-management-tables)
4. [Authentication & Session Tables](#4-authentication--session-tables)
5. [Organisation & School Tables](#5-organisation--school-tables)
6. [Child Profile Tables](#6-child-profile-tables)
7. [Content Management Tables](#7-content-management-tables)
8. [Learning Progress Tables](#8-learning-progress-tables)
9. [Assessment & Analytics Tables](#9-assessment--analytics-tables)
10. [Gamification Tables](#10-gamification-tables)
11. [Economy Tables](#11-economy-tables)
12. [Seva Activity Tables](#12-seva-activity-tables)
13. [Sync & Device Tables](#13-sync--device-tables)
14. [Audit & Notification Tables](#14-audit--notification-tables)
15. [Indexes](#15-indexes)
16. [Relationships Summary](#16-relationships-summary)
17. [Authentication & Session Handling](#17-authentication--session-handling)
18. [Permissions Matrix](#18-permissions-matrix)
19. [Data Ownership Rules](#19-data-ownership-rules)
20. [Row-Level Security Policies](#20-row-level-security-policies)
21. [Triggers & Functions](#21-triggers--functions)
22. [Migration & Seed Data](#22-migration--seed-data)

---

## 1. Schema Overview & ERD

### 1.1 Domain Map

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           AASHA BACKEND SCHEMA                            │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐   │
│  │  IDENTITY LAYER  │     │  CONTENT LAYER   │     │  LEARNING LAYER  │   │
│  │                  │     │                  │     │                  │   │
│  │  users           │     │  subjects        │     │  child_profiles  │   │
│  │  user_roles      │     │  chapters        │     │  chapter_progress│   │
│  │  user_devices    │     │  chapter_nodes   │     │  step_attempts   │   │
│  │  user_sessions   │     │  node_steps      │     │  concept_mastery  │   │
│  │  otp_codes       │     │  question_bank   │     │  worksheet_scores│   │
│  │  password_resets │     │  lle_word_map    │     │                  │   │
│  │                  │     │  lle_connectives │     │                  │   │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘   │
│                                                                          │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐   │
│  │  GAMIFICATION    │     │  ECONOMY LAYER   │     │  SEVA LAYER      │   │
│  │  LAYER            │     │                  │     │                  │   │
│  │                  │     │                  │     │                  │   │
│  │  xp_ledger       │     │  coin_ledger    │     │  seva_activities  │   │
│  │  levels          │     │  shop_items     │     │  seva_verifications│   │
│  │  badges          │     │  shop_purchases  │     │                  │   │
│  │  child_badges    │     │  saving_goals    │     │                  │   │
│  │  streaks         │     │  store_items    │     │                  │   │
│  │  daily_challenges│     │  store_redemptions│    │                  │   │
│  │  leaderboards    │     │                  │     │                  │   │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘   │
│                                                                          │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐   │
│  │  ORG LAYER       │     │  SYNC LAYER      │     │  OPS LAYER       │   │
│  │                  │     │                  │     │                  │   │
│  │  organisations   │     │  sync_queue      │     │  audit_log       │   │
│  │  schools         │     │  sync_conflicts  │     │  notifications   │   │
│  │  classes         │     │  devices         │     │  app_config      │   │
│  │  class_enrolment │     │                  │     │                  │   │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘   │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Table Count Summary

| Layer | Tables | Purpose |
|---|---|---|
| Identity | 6 | Users, roles, sessions, OTP, devices, password resets |
| Organisation | 4 | Organisations (NGO/foundation), schools, classes, enrolments |
| Content | 7 | Subjects, chapters, nodes, steps, questions, LLE word maps |
| Learning | 5 | Child profiles, chapter progress, step attempts, mastery, worksheet scores |
| Gamification | 7 | XP ledger, levels, badges, child badges, streaks, daily challenges, leaderboards |
| Economy | 6 | Coin ledger, shop items, purchases, saving goals, store items, redemptions |
| Seva | 2 | Seva activities, verifications |
| Sync | 3 | Sync queue, conflicts, devices |
| Operations | 3 | Audit log, notifications, app config |
| **Total** | **43** | |

---

## 2. Enum Types

```sql
-- User roles
CREATE TYPE user_role AS ENUM (
    'super_admin',     -- Platform-wide admin (foundation staff)
    'org_admin',       -- Organisation-level admin (NGO coordinator)
    'school_admin',    -- School-level admin (principal)
    'teacher',         -- Teacher / facilitator
    'parent',          -- Parent / guardian
    'child'            -- Child (server-side record only; children don't log in)
);

-- Account status
CREATE TYPE account_status AS ENUM (
    'pending',         -- Registered but not verified (email/phone)
    'active',          -- Fully active
    'suspended',       -- Temporarily suspended (admin action)
    'deactivated'      -- User deactivated; data retained per policy
);

-- Auth method
CREATE TYPE auth_method AS ENUM (
    'email_password',
    'phone_otp',
    'google_oauth',    -- Future
    'microsoft_oauth'  -- Future
);

-- Session status
CREATE TYPE session_status AS ENUM (
    'active',
    'expired',
    'revoked',         -- User logged out or admin revoked
    'replaced'         -- New session issued, old replaced
);

-- Subject codes (extensible)
CREATE TYPE subject_code AS ENUM (
    'mathematics',
    'science',
    'english',
    'hindi',
    'social_studies',
    'general_knowledge',
    'sanskrit',
    'computer_science',
    'environmental_science'
);

-- Board codes (Indian education boards)
CREATE TYPE board_code AS ENUM (
    'cbse',
    'icse',
    'state',
    'ib',
    'cambridge',
    'nios'
);

-- Step types (from the App Flow Document)
CREATE TYPE step_type AS ENUM (
    'intro',
    'text',
    'canvas_visual',
    'ix_drag',
    'ix_slider',
    'ix_diagonal',
    'we',
    'solve',
    'quiz',
    'truefalse',
    'fillblank',
    'progress',
    'rapid_fire',
    'memory_match',
    'worksheet_mcq',
    'worksheet_fb',
    'worksheet_tf',
    'worksheet_solve'
);

-- Question types
CREATE TYPE question_type AS ENUM (
    'mcq',             -- Multiple choice (quiz)
    'true_false',
    'fill_blank',
    'solve_step',      -- Multi-step solve problem
    'drag_interactive',
    'slider_interactive',
    'diagonal_interactive',
    'rapid_fire'
);

-- Difficulty levels
CREATE TYPE difficulty_level AS ENUM (
    'easy',
    'medium',
    'hard',
    'hots'             -- Higher-order thinking skills
);

-- Assessment result
CREATE TYPE assessment_result AS ENUM (
    'correct',
    'incorrect',
    'partial',         -- Multi-step solve: some steps correct
    'timeout',         -- Timed quiz: timer expired
    'skipped'          -- Child moved on without answering
);

-- Badge category
CREATE TYPE badge_category AS ENUM (
    'milestone',       -- First concept, chapter complete
    'skill',           -- Angle master, parallel thinker
    'streak',          -- 7-day streak, 30-day streak
    'special'          -- Event-based, seasonal
);

-- Shop item type
CREATE TYPE shop_item_type AS ENUM (
    'consumable',      -- Hint token, streak freeze
    'theme',           -- Ocean, forest, sunset
    'cosmetic',        -- Avatar frame
    'permanent_upgrade' -- Future: extra slots, etc.
);

-- Economy transaction type
CREATE TYPE economy_tx_type AS ENUM (
    'earn',            -- Earned from learning
    'spend',           -- Spent in shop
    'save',            -- Moved to savings
    'withdraw',        -- Withdrawn from savings back to spendable
    'reward',          -- Seva or store redemption reward
    'admin_adjust'     -- Manual correction by admin
);

-- Seva activity type
CREATE TYPE seva_type AS ENUM (
    'eco_seva',        -- Environmental: planting, cleanliness
    'jal_seva'         -- Water-related: conservation, responsible use
);

-- Seva status
CREATE TYPE seva_status AS ENUM (
    'pending',         -- Submitted, awaiting verification
    'verified',        -- Facilitator verified
    'rejected',        -- Facilitator rejected (with reason)
    'expired'          -- No verification within 30 days
);

-- Sync operation type
CREATE TYPE sync_op_type AS ENUM (
    'push',            -- Client pushing state to server
    'pull',            -- Client pulling state from server
    'conflict',        -- Conflict detected
    'resolve'          -- Conflict resolved
);

-- Sync conflict resolution
CREATE TYPE conflict_resolution AS ENUM (
    'server_wins',     -- Server state takes precedence
    'client_wins',     -- Client state takes precedence
    'merged',          -- Automatic merge (e.g., take max XP)
    'manual'           -- Requires manual resolution
);

-- Notification type
CREATE TYPE notification_type AS ENUM (
    'badge_earned',
    'level_up',
    'seva_verified',
    'seva_rejected',
    'saving_goal_reached',
    'store_redemption',
    'chapter_assigned',
    'system_announcement'
);

-- Notification channel
CREATE TYPE notification_channel AS ENUM (
    'in_app',          -- Shown in the app/dashboard
    'push',            -- Push notification (Phase 2+)
    'sms',             -- SMS (for parents without smartphones)
    'email'
);

-- Device platform
CREATE TYPE device_platform AS ENUM (
    'android',
    'ios',
    'web',
    'unknown'
);
```

---

## 3. User Management Tables

### 3.1 `users` — All authenticated users (teachers, parents, admins)

```sql
CREATE TABLE users (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Identity
    email               VARCHAR(255) UNIQUE,          -- Nullable (phone-only users)
    email_verified      BOOLEAN NOT NULL DEFAULT FALSE,
    phone               VARCHAR(20) UNIQUE,           -- E.164 format (+91...)
    phone_verified      BOOLEAN NOT NULL DEFAULT FALSE,
    name                VARCHAR(100) NOT NULL,
    
    -- Password (nullable for OTP-only users)
    password_hash       VARCHAR(255),                 -- bcrypt cost factor 12
    password_changed_at TIMESTAMPTZ,
    
    -- Profile
    avatar              VARCHAR(50) DEFAULT 'person',  -- Emoji or identifier
    preferred_language  VARCHAR(10) DEFAULT 'en-IN',  -- BCP-47 code
    
    -- Status
    role                user_role NOT NULL DEFAULT 'teacher',
    account_status      account_status NOT NULL DEFAULT 'pending',
    
    -- Organisation link
    organisation_id     UUID REFERENCES organisations(id) ON DELETE SET NULL,
    school_id           UUID REFERENCES schools(id) ON DELETE SET NULL,
    
    -- Metadata
    last_login_at       TIMESTAMPTZ,
    last_active_at      TIMESTAMPTZ,
    login_count         INTEGER NOT NULL DEFAULT 0,
    metadata            JSONB NOT NULL DEFAULT '{}',  -- Flexible extra fields
    
    -- Audit
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at         TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at          TIMESTAMPTZ,                  -- Soft delete
    
    -- Constraints
    CONSTRAINT chk_email_or_phone CHECK (
        email IS NOT NULL OR phone IS NOT NULL
    ),
    CONSTRAINT chk_password_if_email CHECK (
        email IS NULL OR password_hash IS NOT NULL OR phone_verified = TRUE
    )
);

COMMENT ON TABLE users IS
    'All authenticated platform users: teachers, parents, admins. Children do NOT have accounts in this table — they use local profiles (child_profiles) that are optionally linked to a parent/teacher account.';
```

### 3.2 `user_roles` — Multi-role support (a teacher can also be a parent)

```sql
CREATE TABLE user_roles (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role        user_role NOT NULL,
    scope_type  VARCHAR(20) NOT NULL DEFAULT 'global', -- 'global', 'school', 'class'
    scope_id    UUID,                                    -- school_id or class_id if scoped
    granted_by  UUID REFERENCES users(id) ON DELETE SET NULL,
    granted_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at  TIMESTAMPTZ,
    
    UNIQUE(user_id, role, scope_type, scope_id),
    
    CONSTRAINT chk_scope CHECK (
        (scope_type = 'global' AND scope_id IS NULL) OR
        (scope_type IN ('school', 'class') AND scope_id IS NOT NULL)
    )
);

COMMENT ON TABLE user_roles IS
    'A user can hold multiple roles. E.g., a teacher at School A and a parent of a child at School B. Scope limits the role to a specific school or class.';
```

---

## 4. Authentication & Session Tables

### 4.1 `user_sessions` — JWT refresh token sessions

```sql
CREATE TABLE user_sessions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    -- Token identifiers
    refresh_token_hash  VARCHAR(255) NOT NULL UNIQUE,  -- SHA-256 hash of refresh token
    jwt_jti             UUID NOT NULL UNIQUE,            -- JWT ID for access token revocation
    
    -- Session metadata
    ip_address      INET NOT NULL,
    user_agent      TEXT,
    device_id       UUID REFERENCES devices(id) ON DELETE SET NULL,
    platform        device_platform NOT NULL DEFAULT 'web',
    
    -- Token lifecycle
    issued_at       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at      TIMESTAMPTZ NOT NULL,                -- 7 days from issue
    revoked_at      TIMESTAMPTZ,                          -- When user logs out
    replaced_by     UUID REFERENCES user_sessions(id),   -- Token rotation chain
    
    status          session_status NOT NULL DEFAULT 'active',
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    -- Auto-expire check
    CONSTRAINT chk_expiry CHECK (expires_at > issued_at)
);

COMMENT ON TABLE user_sessions IS
    'JWT-based session management. Access tokens are stateless (15-min TTL, signed RS256). Refresh tokens are stored here (hashed) for revocation. On refresh, old session is marked replaced_by the new one.';
```

### 4.2 `otp_codes` — One-time passwords for phone OTP login

```sql
CREATE TABLE otp_codes (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    phone           VARCHAR(20) NOT NULL,               -- E.164 format
    code_hash       VARCHAR(255) NOT NULL,               -- bcrypt hash of 6-digit code
    purpose         VARCHAR(20) NOT NULL DEFAULT 'login', -- 'login', 'register', 'reset'
    
    -- Rate limiting
    attempt_count   INTEGER NOT NULL DEFAULT 0,
    max_attempts    INTEGER NOT NULL DEFAULT 5,
    
    -- Lifecycle
    expires_at      TIMESTAMPTZ NOT NULL,                -- 5 minutes from creation
    consumed_at     TIMESTAMPTZ,                         -- When successfully verified
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    CONSTRAINT chk_not_expired CHECK (consumed_at IS NULL OR consumed_at < expires_at)
);

COMMENT ON TABLE otp_codes IS
    'OTP codes for phone-based authentication. Codes are hashed (never stored in plaintext). 5-minute expiry, max 5 attempts. Rate-limited: 1 OTP per 60s per phone.';
```

### 4.3 `password_resets` — Password reset tokens

```sql
CREATE TABLE password_resets (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token_hash      VARCHAR(255) NOT NULL UNIQUE,        -- SHA-256 hash of reset token
    expires_at      TIMESTAMPTZ NOT NULL,                -- 30 minutes from creation
    consumed_at     TIMESTAMPTZ,
    ip_address      INET,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 4.4 `devices` — Registered devices (for sync and analytics)

```sql
CREATE TABLE devices (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    user_id         UUID REFERENCES users(id) ON DELETE CASCADE,  -- Nullable (shared devices)
    device_uuid     VARCHAR(255) NOT NULL UNIQUE,                  -- Browser-generated UUID
    device_name     VARCHAR(100),                                  -- "Rahul's phone", "School tablet"
    platform        device_platform NOT NULL DEFAULT 'android',
    app_version     VARCHAR(20),                                   -- Chapter file version
    
    -- Sync tracking
    last_sync_at    TIMESTAMPTZ,
    sync_enabled    BOOLEAN NOT NULL DEFAULT TRUE,
    
    -- Metadata
    metadata        JSONB NOT NULL DEFAULT '{}',                  -- Screen size, browser, etc.
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_seen_at    TIMESTAMPTZ
);

COMMENT ON TABLE devices IS
    'Tracks physical devices for sync management. A shared Android phone may have multiple child profiles but one device record. The device_uuid is generated client-side and stored in localStorage.';
```

---

## 5. Organisation & School Tables

### 5.1 `organisations` — Foundation / NGO entities

```sql
CREATE TABLE organisations (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(200) NOT NULL,
    slug            VARCHAR(100) NOT NULL UNIQUE,        -- URL-safe identifier
    description     TEXT,
    
    -- Contact
    email           VARCHAR(255),
    phone           VARCHAR(20),
    address         TEXT,
    
    -- Branding
    logo_url        VARCHAR(500),
    primary_color   VARCHAR(7) DEFAULT '#1e40af',        -- Hex color
    
    -- Status
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    
    -- Settings
    settings        JSONB NOT NULL DEFAULT '{}',          -- Feature flags, limits, etc.
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at      TIMESTAMPTZ
);
```

### 5.2 `schools` — Schools participating in the programme

```sql
CREATE TABLE schools (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    organisation_id UUID NOT NULL REFERENCES organisations(id) ON DELETE RESTRICT,
    
    name            VARCHAR(200) NOT NULL,
    slug            VARCHAR(100) NOT NULL UNIQUE,
    
    -- Location
    address         TEXT,
    city            VARCHAR(100),
    state           VARCHAR(50),                        -- Indian state
    pincode         VARCHAR(6),
    geolocation     POINT,                               -- (latitude, longitude)
    
    -- School details
    board           board_code NOT NULL DEFAULT 'cbse',
    medium          VARCHAR(20) DEFAULT 'english',       -- Medium of instruction
    total_students  INTEGER,
    
    -- Contact
    principal_name  VARCHAR(100),
    principal_phone VARCHAR(20),
    coordinator_name VARCHAR(100),
    coordinator_phone VARCHAR(20),
    
    -- Status
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    enrolled_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    -- Metadata
    metadata        JSONB NOT NULL DEFAULT '{}',
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at      TIMESTAMPTZ
);
```

### 5.3 `classes` — Class/division within a school

```sql
CREATE TABLE classes (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    school_id       UUID NOT NULL REFERENCES schools(id) ON DELETE CASCADE,
    
    grade           INTEGER NOT NULL CHECK (grade BETWEEN 1 AND 12),
    section         VARCHAR(10) NOT NULL DEFAULT 'A',    -- A, B, C, etc.
    board           board_code NOT NULL DEFAULT 'cbse',
    medium          VARCHAR(20) DEFAULT 'english',
    
    -- Academic year
    academic_year   VARCHAR(9) NOT NULL,                 -- e.g., '2026-2027'
    
    -- Teacher
    class_teacher_id UUID REFERENCES users(id) ON DELETE SET NULL,
    
    -- Capacity
    capacity        INTEGER DEFAULT 40,
    
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    UNIQUE(school_id, grade, section, academic_year)
);

COMMENT ON TABLE classes IS
    'A class is a grade + section + academic year combination. E.g., Class 8-B for 2026-2027.';
```

### 5.4 `class_enrolments` — Links children to classes

```sql
CREATE TABLE class_enrolments (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    class_id        UUID NOT NULL REFERENCES classes(id) ON DELETE CASCADE,
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    
    -- Enrollment details
    roll_number     INTEGER,
    enrolled_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    unenrolled_at   TIMESTAMPTZ,
    
    -- Guardian
    parent_user_id  UUID REFERENCES users(id) ON DELETE SET NULL,
    
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    UNIQUE(class_id, child_id)
);
```

---

## 6. Child Profile Tables

### 6.1 `child_profiles` — Child identity (no login; managed by teacher/parent)

```sql
CREATE TABLE child_profiles (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Identity (PII — encrypted at rest)
    name_encrypted  BYTEA NOT NULL,                       -- pgcrypto: encrypt(name, key)
    name_search_hash VARCHAR(64),                         -- HMAC-SHA256 for search without decrypt
    display_name     VARCHAR(50) NOT NULL,                -- First name only, for UI display
    avatar          VARCHAR(50) NOT NULL DEFAULT 'person', -- Emoji identifier
    
    -- Demographics
    grade           INTEGER NOT NULL CHECK (grade BETWEEN 1 AND 12),
    board           board_code NOT NULL DEFAULT 'cbse',
    date_of_birth   DATE,                                  -- Optional, for age-adaptive features
    
    -- Links
    school_id       UUID REFERENCES schools(id) ON DELETE SET NULL,
    
    -- Guardian links
    primary_parent_id   UUID REFERENCES users(id) ON DELETE SET NULL,
    
    -- Settings (per child)
    preferred_language  VARCHAR(10) DEFAULT 'en-IN',
    sound_enabled       BOOLEAN NOT NULL DEFAULT TRUE,
    lle_enabled         BOOLEAN NOT NULL DEFAULT TRUE,
    reduced_motion      BOOLEAN NOT NULL DEFAULT FALSE,
    
    -- Aggregate stats (denormalized for performance; maintained by triggers)
    total_xp           INTEGER NOT NULL DEFAULT 0,
    total_coins        INTEGER NOT NULL DEFAULT 0,
    current_level_id   UUID REFERENCES levels(id) ON DELETE SET NULL,
    current_streak     INTEGER NOT NULL DEFAULT 0,
    longest_streak      INTEGER NOT NULL DEFAULT 0,
    last_active_at     TIMESTAMPTZ,
    
    -- Sync
    device_id           UUID REFERENCES devices(id) ON DELETE SET NULL,
    
    -- Status
    is_active           BOOLEAN NOT NULL DEFAULT TRUE,
    
    -- Audit
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at          TIMESTAMPTZ,
    
    created_by          UUID REFERENCES users(id) ON DELETE SET NULL  -- Teacher/parent who created
);

COMMENT ON TABLE child_profiles IS
    'Children do NOT have user accounts. They have profiles created by a teacher or parent. The profile stores identity (encrypted), learning settings, and denormalized aggregate stats. The full name is encrypted; only the display_name (first name) is stored in plaintext for UI rendering. name_search_hash allows searching by name without decrypting.';
```

---

## 7. Content Management Tables

### 7.1 `subjects` — Academic subjects

```sql
CREATE TABLE subjects (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code            subject_code NOT NULL UNIQUE,
    name            VARCHAR(50) NOT NULL,               -- "Mathematics", "Science"
    name_hindi      VARCHAR(50),                        -- "गणित", "विज्ञान"
    icon            VARCHAR(10) NOT NULL,               -- Emoji
    color_hex       VARCHAR(7) NOT NULL DEFAULT '#3b82f6',
    sort_order      INTEGER NOT NULL DEFAULT 0,
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 7.2 `chapters` — Chapter metadata (one per HTML file)

```sql
CREATE TABLE chapters (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    subject_id     UUID NOT NULL REFERENCES subjects(id) ON DELETE RESTRICT,
    
    -- Identity
    title           VARCHAR(200) NOT NULL,
    title_hindi     VARCHAR(200),
    slug            VARCHAR(100) NOT NULL UNIQUE,       -- 'understanding-quadrilaterals'
    description     TEXT,
    
    -- Curriculum mapping
    grade           INTEGER NOT NULL CHECK (grade BETWEEN 1 AND 10),
    board           board_code NOT NULL DEFAULT 'cbse',
    chapter_number  INTEGER NOT NULL,                   -- Chapter number in the textbook
    
    -- Content metadata
    node_count     INTEGER NOT NULL DEFAULT 0,          -- Number of concept nodes
    step_count     INTEGER NOT NULL DEFAULT 0,          -- Total learning steps
    file_size_bytes BIGINT,                              -- Size of the HTML file
    
    -- Distribution
    file_url        VARCHAR(500),                        -- MinIO/S3 URL for the HTML file
    file_checksum   VARCHAR(64),                        -- SHA-256 checksum for integrity
    version         VARCHAR(20) NOT NULL DEFAULT '1.0.0',
    
    -- Status
    is_published    BOOLEAN NOT NULL DEFAULT FALSE,
    published_at    TIMESTAMPTZ,
    
    -- Metadata
    metadata        JSONB NOT NULL DEFAULT '{}',        -- LLE word count, question count, etc.
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at      TIMESTAMPTZ,
    
    UNIQUE(subject_id, grade, board, chapter_number)
);

COMMENT ON TABLE chapters IS
    'Each chapter corresponds to one self-contained HTML file. The file_url points to the HTML file in MinIO. file_checksum allows clients to verify download integrity. version tracks content updates.';
```

### 7.3 `chapter_nodes` — Concept nodes within a chapter

```sql
CREATE TABLE chapter_nodes (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    chapter_id      UUID NOT NULL REFERENCES chapters(id) ON DELETE CASCADE,
    
    node_index      INTEGER NOT NULL,                   -- Position in the chapter (0-based)
    title           VARCHAR(200) NOT NULL,
    title_hindi     VARCHAR(200),
    subtitle        VARCHAR(300),
    icon            VARCHAR(10) NOT NULL,               -- Emoji for this node
    node_type       VARCHAR(30) NOT NULL DEFAULT 'concept', -- 'concept', 'assessment', 'game', 'worksheet'
    
    -- Step count for this node
    step_count      INTEGER NOT NULL DEFAULT 0,
    
    -- Unlock rule
    requires_node_index INTEGER,                        -- Must complete this node index first
    
    -- Metadata
    metadata        JSONB NOT NULL DEFAULT '{}',
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    UNIQUE(chapter_id, node_index)
);
```

### 7.4 `node_steps` — Individual steps within a node

```sql
CREATE TABLE node_steps (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    node_id         UUID NOT NULL REFERENCES chapter_nodes(id) ON DELETE CASCADE,
    
    step_index      INTEGER NOT NULL,                   -- Position within the node (0-based)
    step_type       step_type NOT NULL,
    
    -- Content reference
    content_json    JSONB NOT NULL DEFAULT '{}',        -- Full step content (question, options, etc.)
    
    -- Assessment metadata
    question_id     UUID REFERENCES question_bank(id) ON DELETE SET NULL,  -- If this step uses a bank question
    
    -- XP and rewards
    xp_reward       INTEGER NOT NULL DEFAULT 5,
    coin_reward     INTEGER NOT NULL DEFAULT 0,         -- Non-zero only for progress milestones
    
    -- Timing
    has_timer       BOOLEAN NOT NULL DEFAULT FALSE,
    timer_seconds   INTEGER,                            -- Duration if has_timer
    
    -- Metadata
    metadata        JSONB NOT NULL DEFAULT '{}',
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    UNIQUE(node_id, step_index)
);

COMMENT ON TABLE node_steps IS
    'Each step within a node. The content_json field contains the full step content (question text, options, worked example steps, canvas drawing instructions, etc.) in the same JSON structure used by the JavaScript App object. This allows the content authoring tool to write directly to this table.';
```

### 7.5 `question_bank` — Reusable question definitions

```sql
CREATE TABLE question_bank (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    chapter_id      UUID REFERENCES chapters(id) ON DELETE SET NULL,
    
    question_type   question_type NOT NULL,
    difficulty     difficulty_level NOT NULL DEFAULT 'medium',
    
    -- Question content
    question_text   TEXT NOT NULL,
    question_text_hindi TEXT,                            -- Hindi translation
    
    -- Answer data (structure varies by type)
    -- MCQ:    {"options": [{"text":"4","is_correct":true,"misconception":"..."}, ...]}
    -- TF:     {"answer": true, "explanation": "..."}
    -- FB:     {"answer": "360", "acceptable": ["360","360.0"], "unit": "degrees"}
    -- Solve:  {"steps": [{"prompt":"Sum = ___","answer":"360"}, ...]}
    answer_data     JSONB NOT NULL,
    
    -- Educational metadata
    concept_tags    TEXT[] DEFAULT '{}',                -- Array of concept identifiers
    misconception_tags TEXT[] DEFAULT '{}',             -- Common misconceptions this question tests
    blooms_level    VARCHAR(20) DEFAULT 'understand',  -- Bloom's taxonomy level
    
    -- LLE
    lle_word_tags   TEXT[] DEFAULT '{}',               -- Words in this question that have LLE translations
    
    -- Status
    is_published    BOOLEAN NOT NULL DEFAULT TRUE,
    usage_count     INTEGER NOT NULL DEFAULT 0,         -- How many times attempted
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by      UUID REFERENCES users(id) ON DELETE SET NULL
);

COMMENT ON TABLE question_bank IS
    'Reusable question definitions. The answer_data JSONB stores the full answer structure. For MCQ, it includes option text, correctness, and misconception feedback. For fill-blank, it includes the answer and acceptable variants. For solve, it includes all step prompts and answers.';
```

### 7.6 `lle_word_map` — Language Learning Engine translations

```sql
CREATE TABLE lle_word_map (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    english_word    VARCHAR(100) NOT NULL,
    hindi_translation VARCHAR(100) NOT NULL,
    hindi_transliteration VARCHAR(100),                 -- Romanized Hindi
    definition      TEXT,                                -- Brief definition in English
    definition_hindi TEXT,                                -- Brief definition in Hindi
    
    -- Context
    subject_id      UUID REFERENCES subjects(id) ON DELETE SET NULL,
    grade_level_min INTEGER DEFAULT 1,
    grade_level_max INTEGER DEFAULT 10,
    
    -- Metadata
    part_of_speech  VARCHAR(30),                        -- noun, verb, adjective, etc.
    difficulty      difficulty_level DEFAULT 'medium',
    
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    UNIQUE(english_word, subject_id)
);

COMMENT ON TABLE lle_word_map IS
    'The LLE word map. Each entry maps an English academic word to its Hindi translation, transliteration, and brief definition. Words can be subject-specific (e.g., "polygon" for mathematics) or general. The grade_level range controls which words appear for which age groups.';
```

### 7.7 `lle_connectives` — Connective words with inline Hindi

```sql
CREATE TABLE lle_connectives (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    english_word    VARCHAR(50) NOT NULL UNIQUE,        -- "because", "therefore", "however"
    hindi_translation VARCHAR(50) NOT NULL,              -- "क्योंकि", "इसलिए", "हालांकि"
    hindi_transliteration VARCHAR(50),
    
    -- Display
    display_inline  BOOLEAN NOT NULL DEFAULT TRUE,      -- Show Hindi inline in parentheses
    
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 8. Learning Progress Tables

### 8.1 `chapter_progress` — Child's progress through a chapter

```sql
CREATE TABLE chapter_progress (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    chapter_id      UUID NOT NULL REFERENCES chapters(id) ON DELETE CASCADE,
    
    -- Position
    current_node_index  INTEGER NOT NULL DEFAULT 0,
    current_step_index  INTEGER NOT NULL DEFAULT 0,
    
    -- Stats
    xp_earned          INTEGER NOT NULL DEFAULT 0,
    coins_earned       INTEGER NOT NULL DEFAULT 0,
    correct_count      INTEGER NOT NULL DEFAULT 0,
    incorrect_count   INTEGER NOT NULL DEFAULT 0,
    timeout_count      INTEGER NOT NULL DEFAULT 0,
    hints_used         INTEGER NOT NULL DEFAULT 0,
    
    -- Completion
    is_completed       BOOLEAN NOT NULL DEFAULT FALSE,
    completed_at       TIMESTAMPTZ,
    completion_percentage NUMERIC(5,2) NOT NULL DEFAULT 0.00, -- 0.00 to 100.00
    
    -- Sync
    last_synced_at     TIMESTAMPTZ,
    sync_version       INTEGER NOT NULL DEFAULT 0,      -- Optimistic concurrency version
    
    -- Time tracking
    time_spent_seconds INTEGER NOT NULL DEFAULT 0,
    last_active_at     TIMESTAMPTZ,
    
    created_at         TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at         TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    UNIQUE(child_id, chapter_id)
);

COMMENT ON TABLE chapter_progress IS
    'The core progress table. Each row represents a child\'s journey through one chapter. This is the table that gets synced from localStorage. sync_version is used for optimistic concurrency — the client sends its version, the server checks and increments.';
```

### 8.2 `step_attempts` — Every answer attempt (for analytics)

```sql
CREATE TABLE step_attempts (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    chapter_id      UUID NOT NULL REFERENCES chapters(id) ON DELETE CASCADE,
    node_id         UUID REFERENCES chapter_nodes(id) ON DELETE SET NULL,
    step_id         UUID REFERENCES node_steps(id) ON DELETE SET NULL,
    question_id     UUID REFERENCES question_bank(id) ON DELETE SET NULL,
    
    -- Attempt data
    step_type       step_type NOT NULL,
    attempt_number  INTEGER NOT NULL DEFAULT 1,         -- 1st try, 2nd try, etc.
    
    -- Answer
    selected_answer TEXT,                               -- What the child selected/typed
    correct_answer  TEXT,                               -- The correct answer (for comparison)
    result          assessment_result NOT NULL,
    
    -- Timing
    time_taken_ms   INTEGER,                             -- Milliseconds from render to answer
    timed_out       BOOLEAN NOT NULL DEFAULT FALSE,
    
    -- Context
    hint_used       BOOLEAN NOT NULL DEFAULT FALSE,
    speed_bonus_xp  INTEGER NOT NULL DEFAULT 0,
    
    -- Telemetry
    device_id       UUID REFERENCES devices(id) ON DELETE SET NULL,
    
    attempted_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

COMMENT ON TABLE step_attempts IS
    'Every single answer attempt is logged here for analytics. This drives the misconception detection, learning analytics, and teacher dashboard. If a child answers the same question 3 times, there are 3 rows. This table will be large — partitioned by month for performance.';
```

### 8.3 `concept_mastery` — Per-concept mastery tracking

```sql
CREATE TABLE concept_mastery (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    node_id         UUID NOT NULL REFERENCES chapter_nodes(id) ON DELETE CASCADE,
    chapter_id      UUID NOT NULL REFERENCES chapters(id) ON DELETE CASCADE,
    
    -- Mastery metrics
    mastery_percentage NUMERIC(5,2) NOT NULL DEFAULT 0.00, -- 0.00 to 100.00
    mastery_level   VARCHAR(20) NOT NULL DEFAULT 'not_started', -- not_started, learning, practicing, mastered
    
    -- Attempt stats for this concept
    total_attempts  INTEGER NOT NULL DEFAULT 0,
    correct_attempts INTEGER NOT NULL DEFAULT 0,
    first_try_correct INTEGER NOT NULL DEFAULT 0,      -- Correct on first attempt
    
    -- Assessment
    is_mastered     BOOLEAN NOT NULL DEFAULT FALSE,
    mastered_at     TIMESTAMPTZ,
    
    -- Misconception tracking
    common_errors   JSONB NOT NULL DEFAULT '[]',        -- Array of error patterns
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    UNIQUE(child_id, node_id)
);

COMMENT ON TABLE concept_mastery IS
    'Tracks mastery per concept (node). mastery_percentage is calculated from correct_attempts/total_attempts. mastery_level transitions: not_started -> learning (<40%) -> practicing (40-80%) -> mastered (>80%). A node is "mastered" when mastery_percentage >= 80 AND at least 3 correct attempts.';
```

### 8.4 `worksheet_scores` — Worksheet assessment results

```sql
CREATE TABLE worksheet_scores (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    chapter_id      UUID NOT NULL REFERENCES chapters(id) ON DELETE CASCADE,
    
    worksheet_level difficulty_level NOT NULL,          -- easy (Basic), medium (Standard), hard (HOTS), hots (Final Mixed)
    
    -- Scores
    total_questions INTEGER NOT NULL,
    correct_answers INTEGER NOT NULL,
    incorrect_answers INTEGER NOT NULL,
    score_percentage NUMERIC(5,2) NOT NULL,
    
    -- Rewards
    xp_earned      INTEGER NOT NULL DEFAULT 0,
    coins_earned   INTEGER NOT NULL DEFAULT 0,
    
    -- Timing
    time_taken_seconds INTEGER,
    
    started_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    completed_at   TIMESTAMPTZ,
    
    created_at     TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 8.5 `game_scores` — Rapid fire and memory match scores

```sql
CREATE TABLE game_scores (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    chapter_id      UUID REFERENCES chapters(id) ON DELETE SET NULL,
    
    game_type       VARCHAR(20) NOT NULL,              -- 'rapid_fire', 'memory_match'
    
    -- Score data
    score           INTEGER NOT NULL,                   -- Correct count (rapid fire) or moves (memory match)
    max_possible    INTEGER NOT NULL,                   -- 10 for rapid fire, 6 for memory match
    best_streak     INTEGER NOT NULL DEFAULT 0,        -- For rapid fire
    time_taken_seconds INTEGER,
    
    -- Rewards
    xp_earned      INTEGER NOT NULL DEFAULT 0,
    coins_earned   INTEGER NOT NULL DEFAULT 0,
    
    -- Is this a new personal best?
    is_personal_best BOOLEAN NOT NULL DEFAULT FALSE,
    
    played_at       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 9. Assessment & Analytics Tables

### 9.1 `assessment_attempts` — Aggregated assessment view (for dashboard queries)

```sql
CREATE TABLE assessment_attempts (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    chapter_id      UUID NOT NULL REFERENCES chapters(id) ON DELETE CASCADE,
    
    -- Assessment context
    assessment_type VARCHAR(30) NOT NULL,               -- 'quiz', 'worksheet', 'rapid_fire', 'memory_match'
    difficulty      difficulty_level,
    
    -- Results
    score           NUMERIC(5,2) NOT NULL,              -- Percentage
    correct_count   INTEGER NOT NULL,
    total_count     INTEGER NOT NULL,
    
    -- Analytics
    time_taken_seconds INTEGER,
    misconception_tags TEXT[] DEFAULT '{}',
    
    attempted_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

COMMENT ON TABLE assessment_attempts IS
    'Aggregated assessment records for fast dashboard queries. While step_attempts stores every individual attempt (high volume, partitioned), this table stores summarized assessment results. Updated by triggers or application logic when an assessment is completed.';
```

---

## 10. Gamification Tables

### 10.1 `levels` — Level definitions

```sql
CREATE TABLE levels (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    level_number    INTEGER NOT NULL UNIQUE,            -- 1, 2, 3, 4, 5
    name            VARCHAR(50) NOT NULL,               -- "Beginner", "Learner", "Scholar", "Expert", "Master"
    icon            VARCHAR(10) NOT NULL,               -- Emoji
    xp_threshold    INTEGER NOT NULL,                    -- Minimum XP to reach this level
    color_hex       VARCHAR(7) NOT NULL DEFAULT '#3b82f6',
    
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    sort_order      INTEGER NOT NULL,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Seed data
INSERT INTO levels (level_number, name, icon, xp_threshold, color_hex, sort_order) VALUES
(1, 'Beginner', '🌱', 0, '#3b82f6', 1),
(2, 'Learner', '📘', 50, '#3b82f6', 2),
(3, 'Scholar', '📚', 120, '#3b82f6', 3),
(4, 'Expert', '🎓', 200, '#3b82f6', 4),
(5, 'Master', '🏆', 350, '#f59e0b', 5);
```

### 10.2 `badges` — Badge definitions

```sql
CREATE TABLE badges (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    badge_id        VARCHAR(50) NOT NULL UNIQUE,         -- 'first', 'angles', 'para', 'family', 'champ'
    name            VARCHAR(100) NOT NULL,
    description     TEXT NOT NULL,
    icon            VARCHAR(10) NOT NULL,               -- Emoji
    category        badge_category NOT NULL DEFAULT 'milestone',
    
    -- Earn condition (evaluated by application logic)
    condition_type  VARCHAR(50) NOT NULL,                -- 'first_concept', 'chapter_complete', 'streak_7', etc.
    condition_value JSONB NOT NULL DEFAULT '{}',        -- Parameters for the condition
    
    -- Scope
    chapter_id      UUID REFERENCES chapters(id) ON DELETE CASCADE,  -- NULL = platform-wide badge
    
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    sort_order      INTEGER NOT NULL DEFAULT 0,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Seed data
INSERT INTO badges (badge_id, name, description, icon, category, condition_type, condition_value, sort_order) VALUES
('first', 'First Steps', 'Complete your first concept', '👣', 'milestone', 'first_concept', '{}', 1),
('angles', 'Angle Master', 'Discover angle sum property', '📐', 'skill', 'node_mastery', '{"node_index": 5}', 2),
('para', 'Parallel Thinker', 'Master parallelograms', '⚖️', 'skill', 'node_mastery', '{"node_index": 12}', 3),
('family', 'Family Tree', 'Master the quadrilateral hierarchy', '🌳', 'skill', 'multi_node_mastery', '{"node_indices": [14, 15, 16]}', 4),
('champ', 'Chapter Champion', 'Complete the chapter', '🏆', 'milestone', 'chapter_complete', '{}', 5);
```

### 10.3 `child_badges` — Badges earned by children

```sql
CREATE TABLE child_badges (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    badge_id        UUID NOT NULL REFERENCES badges(id) ON DELETE RESTRICT,
    
    earned_at       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    -- Context
    chapter_id      UUID REFERENCES chapters(id) ON DELETE SET NULL,
    trigger_data    JSONB NOT NULL DEFAULT '{}',        -- What exactly triggered the badge
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    UNIQUE(child_id, badge_id)
);
```

### 10.4 `xp_ledger` — XP transaction log

```sql
CREATE TABLE xp_ledger (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    
    -- Transaction
    amount          INTEGER NOT NULL,                   -- Positive (earned) or negative (adjustment)
    reason          VARCHAR(200) NOT NULL,              -- "Quiz correct", "Speed bonus", "Concept milestone"
    reason_code     VARCHAR(50) NOT NULL,              -- 'quiz_correct', 'speed_bonus', 'concept_milestone', 'chapter_complete', etc.
    
    -- Context
    chapter_id      UUID REFERENCES chapters(id) ON DELETE SET NULL,
    node_id         UUID REFERENCES chapter_nodes(id) ON DELETE SET NULL,
    step_id         UUID REFERENCES node_steps(id) ON DELETE SET NULL,
    
    -- Balance after this transaction
    balance_after   INTEGER NOT NULL,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

COMMENT ON TABLE xp_ledger IS
    'Immutable ledger of all XP transactions. Each row records the amount, reason, and resulting balance. This is an append-only table — no updates, no deletes (except admin purge). balance_after is maintained by trigger for audit trail.';
```

### 10.5 `streaks` — Streak tracking

```sql
CREATE TABLE streaks (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    
    -- Current streak
    current_streak  INTEGER NOT NULL DEFAULT 0,
    longest_streak  INTEGER NOT NULL DEFAULT 0,
    
    -- Streak protection
    freezes_available INTEGER NOT NULL DEFAULT 0,       -- From shop purchases
    freezes_used    INTEGER NOT NULL DEFAULT 0,
    
    -- Dates
    last_correct_date DATE,                             -- Date of last correct answer
    streak_started_at TIMESTAMPTZ,
    
    -- Daily challenge streak
    daily_challenge_streak INTEGER NOT NULL DEFAULT 0,
    last_daily_challenge_date DATE,
    
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    UNIQUE(child_id)
);
```

### 10.6 `daily_challenges` — Daily challenge records

```sql
CREATE TABLE daily_challenges (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    
    challenge_date  DATE NOT NULL,
    
    -- Challenge content (deterministic based on date seed)
    seed            INTEGER NOT NULL,                   -- Date-based seed for question selection
    question_ids    UUID[] NOT NULL DEFAULT '{}',       -- Array of question_bank IDs for this challenge
    
    -- Results
    questions_total INTEGER NOT NULL DEFAULT 5,
    questions_correct INTEGER NOT NULL DEFAULT 0,
    score_percentage NUMERIC(5,2) DEFAULT 0.00,
    
    -- Rewards
    xp_earned      INTEGER NOT NULL DEFAULT 0,
    coins_earned   INTEGER NOT NULL DEFAULT 0,
    
    -- Status
    is_completed    BOOLEAN NOT NULL DEFAULT FALSE,
    completed_at    TIMESTAMPTZ,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    UNIQUE(child_id, challenge_date)
);

COMMENT ON TABLE daily_challenges IS
    'One daily challenge per child per day. The seed is derived from the date (YYYYMMDD as integer), ensuring all children get the same challenge on the same day. Questions are selected deterministically from the question bank based on the seed.';
```

### 10.7 `leaderboards` — Cohort leaderboard snapshots (Phase 2, opt-in only)

```sql
CREATE TABLE leaderboards (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Scope
    scope_type      VARCHAR(20) NOT NULL,               -- 'class', 'school', 'global'
    scope_id       UUID,                                 -- class_id, school_id, or NULL for global
    period         VARCHAR(20) NOT NULL,                 -- 'weekly', 'monthly', 'all_time'
    
    -- Entry
    child_id       UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    rank           INTEGER NOT NULL,
    score          INTEGER NOT NULL,                     -- XP for this period
    
    -- Snapshot
    snapshot_date  DATE NOT NULL,
    
    -- Opt-in (children must explicitly join leaderboards)
    is_opted_in    BOOLEAN NOT NULL DEFAULT FALSE,
    
    created_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    UNIQUE(scope_type, scope_id, period, child_id, snapshot_date)
);

COMMENT ON TABLE leaderboards IS
    'Leaderboard snapshots. Children must explicitly opt in — no child is ranked without consent. Leaderboards are scoped to class or school, never global without explicit opt-in. Updated by a nightly job.';
```

---

## 11. Economy Tables

### 11.1 `coin_ledger` — Immutable coin transaction log

```sql
CREATE TABLE coin_ledger (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    
    -- Transaction
    type            economy_tx_type NOT NULL,            -- 'earn', 'spend', 'save', 'withdraw', 'reward', 'admin_adjust'
    amount          INTEGER NOT NULL,                   -- Positive for earn/save_in, negative for spend/withdraw
    reason          VARCHAR(200) NOT NULL,              -- "Concept milestone", "Hint token purchase", "Seva reward"
    reason_code     VARCHAR(50) NOT NULL,              -- 'concept_milestone', 'shop_purchase', 'seva_verified', etc.
    
    -- Context
    chapter_id      UUID REFERENCES chapters(id) ON DELETE SET NULL,
    shop_item_id    UUID REFERENCES shop_items(id) ON DELETE SET NULL,
    seva_activity_id UUID REFERENCES seva_activities(id) ON DELETE SET NULL,
    
    -- Balance after this transaction
    spendable_balance_after INTEGER NOT NULL,
    savings_balance_after   INTEGER NOT NULL DEFAULT 0,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

COMMENT ON TABLE coin_ledger IS
    'Immutable ledger of all coin transactions. Append-only — no updates or deletes. balance_after fields are maintained by trigger for audit trail. This is the source of truth for the Aasha Economy.';
```

### 11.2 `shop_items` — Gem shop item definitions

```sql
CREATE TABLE shop_items (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    item_id         VARCHAR(50) NOT NULL UNIQUE,         -- 'hint', 'freeze', 'theme_ocean', etc.
    name            VARCHAR(100) NOT NULL,
    description     TEXT NOT NULL,
    icon            VARCHAR(10) NOT NULL,               -- Emoji
    cost            INTEGER NOT NULL CHECK (cost >= 0), -- Cost in coins
    
    item_type       shop_item_type NOT NULL,             -- 'consumable', 'theme', 'cosmetic'
    
    -- For themes
    theme_class     VARCHAR(50),                         -- 'theme-ocean', 'theme-forest', 'theme-sunset'
    theme_css_overrides JSONB,                           -- CSS custom property overrides
    
    -- For consumables
    effect_code     VARCHAR(50),                        -- 'remove_2_wrong', 'protect_streak'
    effect_params   JSONB NOT NULL DEFAULT '{}',
    
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    sort_order      INTEGER NOT NULL DEFAULT 0,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Seed data
INSERT INTO shop_items (item_id, name, description, icon, cost, item_type, effect_code, sort_order) VALUES
('hint', 'Hint Token', 'Remove 2 wrong options', '💡', 20, 'consumable', 'remove_2_wrong', 1),
('freeze', 'Streak Freeze', 'Protect streak from 1 wrong answer', '🛡️', 30, 'consumable', 'protect_streak', 2);

INSERT INTO shop_items (item_id, name, description, icon, cost, item_type, theme_class, sort_order) VALUES
('theme_ocean', 'Ocean Theme', 'Blue/teal colours', '🌊', 50, 'theme', 'theme-ocean', 3),
('theme_forest', 'Forest Theme', 'Green colours', '🌲', 50, 'theme', 'theme-forest', 4),
('theme_sunset', 'Sunset Theme', 'Orange/pink colours', '🌅', 50, 'theme', 'theme-sunset', 5);

INSERT INTO shop_items (item_id, name, description, icon, cost, item_type, sort_order) VALUES
('frame_gold', 'Gold Frame', 'Gold border on avatar', '🥇', 100, 'cosmetic', 6),
('frame_star', 'Star Frame', 'Star border on avatar', '⭐', 150, 'cosmetic', 7);
```

### 11.3 `shop_purchases` — Purchase records

```sql
CREATE TABLE shop_purchases (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    shop_item_id    UUID NOT NULL REFERENCES shop_items(id) ON DELETE RESTRICT,
    
    quantity        INTEGER NOT NULL DEFAULT 1 CHECK (quantity > 0),
    unit_cost       INTEGER NOT NULL,                   -- Cost at time of purchase (for audit)
    total_cost      INTEGER NOT NULL,                  -- unit_cost * quantity
    
    -- For consumables: tracks usage
    remaining_uses  INTEGER,                            -- NULL for permanent items
    
    purchased_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 11.4 `saving_goals` — Child's saving goals

```sql
CREATE TABLE saving_goals (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    
    goal_name       VARCHAR(200) NOT NULL,              -- "Plant-a-Tree", "Book for a Child"
    target_amount   INTEGER NOT NULL CHECK (target_amount > 0),
    
    -- Progress
    current_amount  INTEGER NOT NULL DEFAULT 0,
    is_completed    BOOLEAN NOT NULL DEFAULT FALSE,
    completed_at    TIMESTAMPTZ,
    
    -- Linked store item (if saving for a specific store reward)
    store_item_id   UUID REFERENCES store_items(id) ON DELETE SET NULL,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at      TIMESTAMPTZ
);
```

### 11.5 `store_items` — Aasha Store real-world impact items

```sql
CREATE TABLE store_items (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    name            VARCHAR(200) NOT NULL,
    description     TEXT NOT NULL,
    icon            VARCHAR(10) NOT NULL,
    cost            INTEGER NOT NULL CHECK (cost > 0), -- Cost in coins
    
    -- Category
    category        VARCHAR(30) NOT NULL,               -- 'educational', 'real_world_impact'
    
    -- Real-world fulfillment
    is_fulfillable  BOOLEAN NOT NULL DEFAULT FALSE,    -- Requires physical action by foundation
    fulfillment_type VARCHAR(50),                       -- 'plant_tree', 'donate_book', 'water_filter'
    fulfillment_cost NUMERIC(10,2),                     -- Real cost in INR (for foundation budgeting)
    
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    sort_order      INTEGER NOT NULL DEFAULT 0,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 11.6 `store_redemptions` — Store redemption records

```sql
CREATE TABLE store_redemptions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    store_item_id   UUID NOT NULL REFERENCES store_items(id) ON DELETE RESTRICT,
    
    amount_paid     INTEGER NOT NULL,                   -- Coins paid
    
    -- Fulfillment status
    status          VARCHAR(20) NOT NULL DEFAULT 'pending', -- 'pending', 'fulfilled', 'cancelled'
    fulfilled_at    TIMESTAMPTZ,
    fulfilled_by    UUID REFERENCES users(id) ON DELETE SET NULL, -- Admin who fulfilled
    
    -- Evidence
    fulfillment_notes TEXT,
    evidence_url    VARCHAR(500),                      -- Photo/proof URL
    
    redeemed_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 12. Seva Activity Tables

### 12.1 `seva_activities` — Real-world activity submissions

```sql
CREATE TABLE seva_activities (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    
    -- Activity details
    seva_type       seva_type NOT NULL,                 -- 'eco_seva', 'jal_seva'
    activity_name   VARCHAR(200) NOT NULL,              -- "Plant a Seed", "Community Cleanliness"
    description     TEXT NOT NULL,
    
    -- Evidence
    photo_url       VARCHAR(500),                       -- MinIO URL for uploaded photo
    photo_thumbnail_url VARCHAR(500),
    
    -- Location
    geolocation     POINT,                               -- (latitude, longitude)
    location_text   VARCHAR(200),                       -- "Park near my house"
    
    -- Verification
    status          seva_status NOT NULL DEFAULT 'pending',
    verified_at     TIMESTAMPTZ,
    verified_by     UUID REFERENCES users(id) ON DELETE SET NULL,
    rejection_reason TEXT,
    
    -- Reward
    coins_earned    INTEGER NOT NULL DEFAULT 0,
    bonus_coins     INTEGER NOT NULL DEFAULT 0,         -- Extra for photo evidence
    coin_ledger_id  UUID REFERENCES coin_ledger(id) ON DELETE SET NULL,
    
    -- Sync (for offline submissions)
    device_id       UUID REFERENCES devices(id) ON DELETE SET NULL,
    is_offline_submission BOOLEAN NOT NULL DEFAULT FALSE,
    
    submitted_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 12.2 `seva_types_catalog` — Available Seva activities

```sql
CREATE TABLE seva_types_catalog (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    seva_type       seva_type NOT NULL,
    activity_name   VARCHAR(200) NOT NULL,
    description     TEXT NOT NULL,
    icon            VARCHAR(10) NOT NULL,
    
    -- Reward
    base_coins      INTEGER NOT NULL DEFAULT 20,
    photo_bonus_coins INTEGER NOT NULL DEFAULT 5,
    
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    sort_order      INTEGER NOT NULL DEFAULT 0,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    UNIQUE(seva_type, activity_name)
);
```

---

## 13. Sync & Device Tables

### 13.1 `sync_queue` — Offline sync operations queue

```sql
CREATE TABLE sync_queue (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    device_id       UUID REFERENCES devices(id) ON DELETE SET NULL,
    
    -- Operation
    operation_type  sync_op_type NOT NULL,               -- 'push', 'pull'
    table_name      VARCHAR(50) NOT NULL,               -- 'chapter_progress', 'step_attempts', etc.
    record_id       UUID,                                -- ID of the affected record
    
    -- Data
    payload         JSONB NOT NULL,                      -- Full data payload
    client_version  INTEGER NOT NULL DEFAULT 0,
    server_version  INTEGER NOT NULL DEFAULT 0,
    
    -- Status
    status          VARCHAR(20) NOT NULL DEFAULT 'pending', -- 'pending', 'processing', 'completed', 'failed', 'conflict'
    error_message   TEXT,
    retry_count     INTEGER NOT NULL DEFAULT 0,
    max_retries     INTEGER NOT NULL DEFAULT 3,
    
    -- Conflict
    conflict_resolution conflict_resolution,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_at    TIMESTAMPTZ,
    completed_at    TIMESTAMPTZ
);

COMMENT ON TABLE sync_queue IS
    'Queue of sync operations from offline devices. Each row represents one state change that needs to be applied to the server. Processed by a background worker. If a conflict is detected (client_version != server_version), the row is marked conflict and requires resolution.';
```

### 13.2 `sync_conflicts` — Detailed conflict records

```sql
CREATE TABLE sync_conflicts (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    sync_queue_id   UUID NOT NULL REFERENCES sync_queue(id) ON DELETE CASCADE,
    
    child_id        UUID NOT NULL REFERENCES child_profiles(id) ON DELETE CASCADE,
    
    -- Conflict data
    table_name      VARCHAR(50) NOT NULL,
    record_id       UUID NOT NULL,
    
    client_state    JSONB NOT NULL,                      -- What the client sent
    server_state    JSONB NOT NULL,                      -- What the server has
    client_version  INTEGER NOT NULL,
    server_version  INTEGER NOT NULL,
    
    -- Resolution
    resolution      conflict_resolution NOT NULL DEFAULT 'manual',
    resolved_state  JSONB,                                -- Final merged state
    resolved_by     UUID REFERENCES users(id) ON DELETE SET NULL,
    resolved_at     TIMESTAMPTZ,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 14. Audit & Notification Tables

### 14.1 `audit_log` — Immutable audit trail

```sql
CREATE TABLE audit_log (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    actor_id        UUID REFERENCES users(id) ON DELETE SET NULL, -- Who performed the action
    actor_role      user_role,
    
    -- Action
    action          VARCHAR(50) NOT NULL,               -- 'login', 'create_child', 'verify_seva', 'purchase_item', etc.
    entity_type     VARCHAR(50) NOT NULL,               -- 'child_profile', 'seva_activity', 'shop_purchase'
    entity_id       UUID,
    
    -- Details
    old_values      JSONB,                                -- State before change
    new_values      JSONB,                                -- State after change
    
    -- Context
    ip_address      INET,
    user_agent      TEXT,
    device_id       UUID REFERENCES devices(id) ON DELETE SET NULL,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

COMMENT ON TABLE audit_log IS
    'Immutable audit trail of all significant actions. Every create, update, delete, verify, purchase, and login is logged here. This table is append-only — no updates, no deletes. Used for compliance, debugging, and analytics.';
```

### 14.2 `notifications` — User-facing notifications

```sql
CREATE TABLE notifications (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    recipient_id    UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE, -- Teacher/parent/admin
    child_id        UUID REFERENCES child_profiles(id) ON DELETE CASCADE,  -- Related child (if any)
    
    type            notification_type NOT NULL,
    title           VARCHAR(200) NOT NULL,
    body            TEXT,
    
    -- Context
    entity_type     VARCHAR(50),
    entity_id       UUID,
    
    -- Delivery
    channel         notification_channel NOT NULL DEFAULT 'in_app',
    is_read         BOOLEAN NOT NULL DEFAULT FALSE,
    read_at         TIMESTAMPTZ,
    
    -- Action
    action_url      VARCHAR(500),                       -- Deep link for the notification
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at      TIMESTAMPTZ                         -- Optional expiry
);
```

### 14.3 `app_config` — Application configuration (key-value)

```sql
CREATE TABLE app_config (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    config_key      VARCHAR(100) NOT NULL UNIQUE,
    config_value    JSONB NOT NULL,
    description     TEXT,
    
    -- Scope
    scope_type      VARCHAR(20) NOT NULL DEFAULT 'global', -- 'global', 'organisation', 'school'
    scope_id       UUID,
    
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_by      UUID REFERENCES users(id) ON DELETE SET NULL
);
```

---

## 15. Indexes

### 15.1 Performance-Critical Indexes

```sql
-- Users
CREATE INDEX idx_users_email ON users(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_phone ON users(phone) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_role ON users(role) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_organisation ON users(organisation_id) WHERE deleted_at IS NULL;

-- User roles
CREATE INDEX idx_user_roles_user ON user_roles(user_id) WHERE revoked_at IS NULL;
CREATE INDEX idx_user_roles_scope ON user_roles(scope_type, scope_id) WHERE revoked_at IS NULL;

-- Sessions
CREATE INDEX idx_sessions_user ON user_sessions(user_id) WHERE status = 'active';
CREATE INDEX idx_sessions_expires ON user_sessions(expires_at) WHERE status = 'active';
CREATE INDEX idx_sessions_jti ON user_sessions(jwt_jti);

-- OTP
CREATE INDEX idx_otp_phone ON otp_codes(phone) WHERE consumed_at IS NULL;
CREATE INDEX idx_otp_expires ON otp_codes(expires_at) WHERE consumed_at IS NULL;

-- Child profiles
CREATE INDEX idx_child_school ON child_profiles(school_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_child_parent ON child_profiles(primary_parent_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_child_grade ON child_profiles(grade) WHERE deleted_at IS NULL;
CREATE INDEX idx_child_active ON child_profiles(is_active) WHERE deleted_at IS NULL;

-- Chapter progress
CREATE INDEX idx_progress_child ON chapter_progress(child_id);
CREATE INDEX idx_progress_chapter ON chapter_progress(chapter_id);
CREATE INDEX idx_progress_child_chapter ON chapter_progress(child_id, chapter_id);
CREATE INDEX idx_progress_completed ON chapter_progress(is_completed) WHERE is_completed = TRUE;
CREATE INDEX idx_progress_sync ON chapter_progress(last_synced_at) WHERE last_synced_at IS NOT NULL;

-- Step attempts (high-volume table — needs careful indexing)
CREATE INDEX idx_attempts_child ON step_attempts(child_id);
CREATE INDEX idx_attempts_child_chapter ON step_attempts(child_id, chapter_id);
CREATE INDEX idx_attempts_chapter_node ON step_attempts(chapter_id, node_id);
CREATE INDEX idx_attempts_result ON step_attempts(result);
CREATE INDEX idx_attempts_attempted ON step_attempts(attempted_at);
CREATE INDEX idx_attempts_question ON step_attempts(question_id) WHERE question_id IS NOT NULL;
CREATE INDEX idx_attempts_child_date ON step_attempts(child_id, attempted_at);

-- Concept mastery
CREATE INDEX idx_mastery_child ON concept_mastery(child_id);
CREATE INDEX idx_mastery_child_chapter ON concept_mastery(child_id, chapter_id);
CREATE INDEX idx_mastery_mastered ON concept_mastery(is_mastered) WHERE is_mastered = TRUE;

-- XP ledger
CREATE INDEX idx_xp_child ON xp_ledger(child_id);
CREATE INDEX idx_xp_child_date ON xp_ledger(child_id, created_at);

-- Coin ledger
CREATE INDEX idx_coin_child ON coin_ledger(child_id);
CREATE INDEX idx_coin_child_date ON coin_ledger(child_id, created_at);
CREATE INDEX idx_coin_type ON coin_ledger(type);

-- Child badges
CREATE INDEX idx_badges_child ON child_badges(child_id);
CREATE INDEX idx_badges_badge ON child_badges(badge_id);

-- Shop purchases
CREATE INDEX idx_purchases_child ON shop_purchases(child_id);
CREATE INDEX idx_purchases_item ON shop_purchases(shop_item_id);

-- Seva
CREATE INDEX idx_seva_child ON seva_activities(child_id);
CREATE INDEX idx_seva_status ON seva_activities(status);
CREATE INDEX idx_seva_verified_by ON seva_activities(verified_by) WHERE verified_by IS NOT NULL;
CREATE INDEX idx_seva_submitted ON seva_activities(submitted_at);

-- Sync queue
CREATE INDEX idx_sync_child ON sync_queue(child_id);
CREATE INDEX idx_sync_status ON sync_queue(status) WHERE status = 'pending';
CREATE INDEX idx_sync_created ON sync_queue(created_at) WHERE status = 'pending';

-- Audit log
CREATE INDEX idx_audit_actor ON audit_log(actor_id);
CREATE INDEX idx_audit_entity ON audit_log(entity_type, entity_id);
CREATE INDEX idx_audit_created ON audit_log(created_at);

-- Notifications
CREATE INDEX idx_notif_recipient ON notifications(recipient_id) WHERE is_read = FALSE;
CREATE INDEX idx_notif_child ON notifications(child_id);

-- Worksheet scores
CREATE INDEX idx_worksheet_child ON worksheet_scores(child_id);
CREATE INDEX idx_worksheet_chapter ON worksheet_scores(chapter_id, worksheet_level);

-- Game scores
CREATE INDEX idx_game_child ON game_scores(child_id);
CREATE INDEX idx_game_child_type ON game_scores(child_id, game_type);
CREATE INDEX idx_game_best ON game_scores(child_id, game_type, is_personal_best);

-- Daily challenges
CREATE INDEX idx_daily_child_date ON daily_challenges(child_id, challenge_date);

-- Leaderboards
CREATE INDEX idx_leaderboard_scope ON leaderboards(scope_type, scope_id, period, snapshot_date);

-- Classes
CREATE INDEX idx_class_school ON classes(school_id) WHERE is_active = TRUE;
CREATE INDEX idx_class_teacher ON classes(class_teacher_id) WHERE is_active = TRUE;

-- Enrolments
CREATE INDEX idx_enrolment_class ON class_enrolments(class_id) WHERE is_active = TRUE;
CREATE INDEX idx_enrolment_child ON class_enrolments(child_id) WHERE is_active = TRUE;

-- Chapters
CREATE INDEX idx_chapter_subject_grade ON chapters(subject_id, grade) WHERE deleted_at IS NULL AND is_published = TRUE;
CREATE INDEX idx_chapter_slug ON chapters(slug);

-- Question bank
CREATE INDEX idx_question_chapter ON question_bank(chapter_id) WHERE is_published = TRUE;
CREATE INDEX idx_question_type ON question_bank(question_type, difficulty);
CREATE INDEX idx_question_tags ON question_bank USING GIN(concept_tags);
CREATE INDEX idx_question_misconception ON question_bank USING GIN(misconception_tags);

-- LLE word map
CREATE INDEX idx_lle_word ON lle_word_map(english_word) WHERE is_active = TRUE;
CREATE INDEX idx_lle_subject ON lle_word_map(subject_id) WHERE subject_id IS NOT NULL;
```

### 15.2 Partitioning Strategy

```sql
-- step_attempts is the highest-volume table. Partition by month.
-- (Requires PostgreSQL declarative partitioning)
CREATE TABLE step_attempts (
    -- same columns as above
) PARTITION BY RANGE (attempted_at);

-- Monthly partitions
CREATE TABLE step_attempts_2026_08 PARTITION OF step_attempts
    FOR VALUES FROM ('2026-08-01') TO ('2026-09-01');
CREATE TABLE step_attempts_2026_09 PARTITION OF step_attempts
    FOR VALUES FROM ('2026-09-01') TO ('2026-10-01');
-- ... etc

-- audit_log and coin_ledger are also candidates for partitioning at scale.
```

---

## 16. Relationships Summary

### 16.1 Entity Relationship Diagram (Text)

```
organisations (1) ──< schools (1) ──< classes (1) ──< class_enrolments (1) >── child_profiles (1)
                                              │
                                              └── class_teacher >── users (1)

users (1) ──< user_roles (N)
users (1) ──< user_sessions (N)
users (1) ──< devices (N)
users (1) ──< audit_log (N) [as actor]
users (1) ──< notifications (N) [as recipient]
users (1) ──< password_resets (N)

child_profiles (1) ──< chapter_progress (N) >── chapters (1)
child_profiles (1) ──< step_attempts (N) >── node_steps (1) >── chapter_nodes (1) >── chapters (1)
child_profiles (1) ──< concept_mastery (N) >── chapter_nodes (1)
child_profiles (1) ──< child_badges (N) >── badges (1)
child_profiles (1) ──< xp_ledger (N)
child_profiles (1) ──< coin_ledger (N)
child_profiles (1) ──< streaks (1)
child_profiles (1) ──< daily_challenges (N)
child_profiles (1) ──< shop_purchases (N) >── shop_items (1)
child_profiles (1) ──< saving_goals (N) >── store_items (1)
child_profiles (1) ──< store_redemptions (N) >── store_items (1)
child_profiles (1) ──< seva_activities (N)
child_profiles (1) ──< game_scores (N)
child_profiles (1) ──< worksheet_scores (N)
child_profiles (1) ──< leaderboards (N)
child_profiles (1) ──< sync_queue (N)
child_profiles (1) ──< sync_conflicts (N)

chapters (1) ──< chapter_nodes (N)
chapter_nodes (1) ──< node_steps (N)
node_steps (1) ──< question_bank (0..1) [via question_id]
chapters (1) ──< question_bank (N)

subjects (1) ──< chapters (N)
chapters (1) ──< chapter_progress (N)
chapters (1) ──< step_attempts (N)
chapters (1) ──< worksheet_scores (N)

seva_activities (1) ──< seva_verifications (0..1) [via verified_by]
shop_items (1) ──< shop_purchases (N)
store_items (1) ──< store_redemptions (N)
store_items (1) ──< saving_goals (N)

sync_queue (1) ──< sync_conflicts (0..1)
```

### 16.2 Cardinality Summary

| Parent | Child | Cardinality | FK Action |
|---|---|---|---|
| organisations | schools | 1:N | RESTRICT |
| schools | classes | 1:N | CASCADE |
| classes | class_enrolments | 1:N | CASCADE |
| child_profiles | class_enrolments | 1:N | CASCADE |
| users | user_roles | 1:N | CASCADE |
| users | user_sessions | 1:N | CASCADE |
| users | devices | 1:N | CASCADE |
| users | child_profiles | 1:N | SET NULL |
| users | audit_log | 1:N | SET NULL |
| child_profiles | chapter_progress | 1:N | CASCADE |
| child_profiles | step_attempts | 1:N | CASCADE |
| child_profiles | concept_mastery | 1:N | CASCADE |
| child_profiles | child_badges | 1:N | CASCADE |
| child_profiles | xp_ledger | 1:N | CASCADE |
| child_profiles | coin_ledger | 1:N | CASCADE |
| child_profiles | streaks | 1:1 | CASCADE |
| child_profiles | daily_challenges | 1:N | CASCADE |
| child_profiles | shop_purchases | 1:N | CASCADE |
| child_profiles | saving_goals | 1:N | CASCADE |
| child_profiles | seva_activities | 1:N | CASCADE |
| child_profiles | game_scores | 1:N | CASCADE |
| child_profiles | sync_queue | 1:N | CASCADE |
| chapters | chapter_nodes | 1:N | CASCADE |
| chapter_nodes | node_steps | 1:N | CASCADE |
| node_steps | question_bank | N:1 | SET NULL |
| shop_items | shop_purchases | 1:N | RESTRICT |
| store_items | store_redemptions | 1:N | RESTRICT |
| badges | child_badges | 1:N | RESTRICT |

---

## 17. Authentication & Session Handling

### 17.1 Authentication Flow

```
┌────────────────────────────────────────────────────────────────┐
│                  AUTHENTICATION ARCHITECTURE                      │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  TEACHER/PARENT LOGIN                                          │
│  ┌──────────────┐                                              │
│  │ Email + Pass │──> bcrypt verify ──> JWT (access + refresh) │
│  └──────────────┘                              │               │
│                                                │               │
│  ┌──────────────┐                              │               │
│  │ Phone + OTP   │──> verify OTP hash ──> JWT  │               │
│  └──────────────┘                              │               │
│                                                ▼               │
│  ┌──────────────────────────────────────────────────────┐     │
│  │  JWT (RS256)                                          │     │
│  │  Access token: 15 min TTL                             │     │
│  │    Claims: { sub, role, jti, exp, iat }               │     │
│  │  Refresh token: 7 day TTL                             │     │
│  │    Stored as SHA-256 hash in user_sessions             │     │
│  └──────────────────────────┬───────────────────────────┘     │
│                              │                                 │
│                              ▼                                 │
│  ┌──────────────────────────────────────────────────────┐     │
│  │  SESSION VALIDATION                                   │     │
│  │  1. Verify JWT signature (RS256 public key)           │     │
│  │  2. Check jti not in revocation list (Redis)          │     │
│  │  3. Check session status = 'active' (PostgreSQL)     │     │
│  │  4. Check user account_status = 'active'              │     │
│  └──────────────────────────────────────────────────────┘     │
│                                                                │
│  TOKEN REFRESH                                                 │
│  ┌──────────────────────────────────────────────────────┐     │
│  │  1. Client sends refresh token                         │     │
│  │  2. Server hashes token, looks up in user_sessions     │     │
│  │  3. If found and active:                               │     │
│  │     a. Mark old session as 'replaced'                  │     │
│  │     b. Create new session with new tokens              │     │
│  │     c. Return new access + refresh tokens              │     │
│  │  4. If not found or expired: return 401                │     │
│  └──────────────────────────────────────────────────────┘     │
│                                                                │
│  LOGOUT                                                        │
│  ┌──────────────────────────────────────────────────────┐     │
│  │  1. Mark session status = 'revoked'                    │     │
│  │  2. Add jti to Redis revocation set with TTL = 900s   │     │
│  │     (matches access token TTL)                        │     │
│  └──────────────────────────────────────────────────────┘     │
│                                                                │
│  CHILD AUTHENTICATION: NONE                                    │
│  Children use local named profiles (localStorage).            │
│  No server authentication. Sync is initiated by               │
│  a teacher/parent who links the child profile.                 │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 17.2 JWT Structure

```json
{
  "sub": "550e8400-e29b-41d4-a716-446655440000",  // user_id
  "role": "teacher",
  "scope": {
    "school_id": "...",
    "class_ids": ["..."]
  },
  "jti": "660e8400-e29b-41d4-a716-446655440000",  // unique token ID
  "iat": 1724457600,
  "exp": 1724458500,  // 15 minutes from iat
  "iss": "aasha-platform",
  "aud": "aasha-api"
}
```

### 17.3 OTP Flow

```
1. Client sends phone number -> POST /api/v1/auth/otp/send
   - Rate limit: 1 OTP per 60s per phone
   - Generate 6-digit code
   - bcrypt hash the code (cost factor 10 — lower than password because 5-min expiry)
   - Store in otp_codes table
   - Send via MSG91 SMS gateway
   - Return: { status: "sent", expires_in: 300 }

2. Client sends OTP -> POST /api/v1/auth/otp/verify
   - Look up latest unconsumed OTP for this phone
   - Check expiry (5 min)
   - Check attempt_count < max_attempts (5)
   - bcrypt verify the code
   - If correct:
     - Mark OTP as consumed
     - Find or create user with this phone
     - Mark phone_verified = true
     - Issue JWT (access + refresh)
     - Return: { access_token, refresh_token, user }
   - If incorrect:
     - Increment attempt_count
     - If attempt_count >= max_attempts: mark as expired
     - Return: { status: "error", message: "Invalid OTP", attempts_remaining: N }

3. Rate limiting:
   - Per phone: 1 OTP / 60s, 5 OTPs / hour, 10 OTPs / day
   - Per IP: 10 OTP requests / hour
```

### 17.4 Password Security

```
Password storage:
  - bcrypt with cost factor 12 (~250ms hash time)
  - Minimum 8 characters
  - No maximum (passphrases encouraged)
  - No complexity requirements (NIST SP 800-63B: length over complexity)
  - Check against HaveIBeenPwned API (online) or local breach list (offline)

Password reset:
  - Token: 32 bytes random, base64url encoded
  - Stored as SHA-256 hash in password_resets table
  - 30-minute expiry
  - Single-use (consumed_at set on use)
  - Rate limit: 3 reset requests per email per hour
```

---

## 18. Permissions Matrix

### 18.1 Role-Permission Matrix

| Action | super_admin | org_admin | school_admin | teacher | parent | child |
|---|---|---|---|---|---|---|
| **Users** | | | | | | |
| Create user (any role) | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Create teacher | ✅ | ✅ | ✅ (own school) | ❌ | ❌ | ❌ |
| View user list | ✅ | ✅ (own org) | ✅ (own school) | ❌ | ❌ | ❌ |
| Deactivate user | ✅ | ✅ (own org) | ✅ (own school) | ❌ | ❌ | ❌ |
| **Child Profiles** | | | | | | |
| Create child profile | ✅ | ✅ | ✅ | ✅ (own class) | ✅ (own child) | ❌ |
| View child profile | ✅ | ✅ (own org) | ✅ (own school) | ✅ (own class) | ✅ (own child) | ❌ |
| Edit child profile | ✅ | ✅ | ✅ | ✅ (own class) | ✅ (own child) | ❌ |
| Delete child profile | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| **Learning Data** | | | | | | |
| View child progress | ✅ | ✅ (own org) | ✅ (own school) | ✅ (own class) | ✅ (own child) | ❌ |
| View step attempts | ✅ | ✅ (own org) | ✅ (own school) | ✅ (own class) | ❌ (summary only) | ❌ |
| View concept mastery | ✅ | ✅ | ✅ | ✅ (own class) | ✅ (own child) | ❌ |
| Reset child progress | ✅ | ✅ | ✅ | ✅ (own class) | ❌ | ❌ |
| **Content** | | | | | | |
| Create chapter | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Publish chapter | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Edit question bank | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Edit LLE word map | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Assign chapter to class | ✅ | ✅ | ✅ | ✅ (own class) | ❌ | ❌ |
| **Gamification** | | | | | | |
| Award badge manually | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Adjust XP (admin) | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Adjust coins (admin) | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Economy** | | | | | | |
| View coin ledger | ✅ | ✅ (own org) | ✅ (own school) | ❌ | ✅ (own child) | ❌ |
| View shop purchases | ✅ | ✅ | ✅ | ❌ | ✅ (own child) | ❌ |
| Fulfill store redemption | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Seva** | | | | | | |
| Submit Seva activity | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ (via child profile) |
| Verify Seva activity | ✅ | ✅ | ✅ (own school) | ✅ (own class) | ❌ | ❌ |
| Reject Seva activity | ✅ | ✅ | ✅ (own school) | ✅ (own class) | ❌ | ❌ |
| **Analytics** | | | | | | |
| View class analytics | ✅ | ✅ | ✅ | ✅ (own class) | ❌ | ❌ |
| View school analytics | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| View org analytics | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Export analytics CSV | ✅ | ✅ | ✅ | ✅ (own class) | ❌ | ❌ |
| **System** | | | | | | |
| Manage app config | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| View audit log | ✅ | ✅ (own org) | ❌ | ❌ | ❌ | ❌ |
| Manage levels/badges | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Manage shop items | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Sync** | | | | | | |
| Push child state | ❌ | ❌ | ❌ | ✅ (own class) | ✅ (own child) | ❌ |
| Pull child state | ❌ | ❌ | ❌ | ✅ (own class) | ✅ (own child) | ❌ |

### 18.2 Permission Enforcement

Permissions are enforced at three layers:

1. **API Layer (Fastify middleware):** Checks the JWT role and scope. Returns 403 if the role is not permitted for the endpoint.

2. **Database Layer (Row-Level Security):** PostgreSQL RLS policies filter rows based on the requesting user's scope. Even if the API is bypassed, the database enforces ownership.

3. **Application Layer (Service logic):** Business rules like "a teacher can only verify Seva activities for children in their class" are enforced in the service layer.

---

## 19. Data Ownership Rules

### 19.1 Ownership Model

| Data | Owner | Who Can Read | Who Can Write |
|---|---|---|---|
| User account | The user themselves | User, admins above them | User (own profile), admins |
| Child profile | The teacher/parent who created it | Linked teacher, parent, admins | Teacher/parent (creator), admins |
| Child learning data (progress, attempts, mastery) | The child (via their profile) | Teacher (class), parent, admins | Child (via sync), admins (correction) |
| Child gamification data (XP, coins, badges) | The child | Teacher (class), parent, admins | System (automatic), admins (adjustment) |
| Child economy data (purchases, savings) | The child | Parent, admins | Child (via shop), admins (correction) |
| Seva activity | The child (submitted via profile) | Child's teacher, parent, admins | Child (submit), teacher (verify/reject) |
| Chapter content | The organisation | All users (read) | org_admin, super_admin |
| Question bank | The organisation | All users (read) | org_admin, super_admin |
| Analytics | The organisation | Aggregated: teachers, admins. Individual: teacher (own class), parent (own child), admins | System (generated) |
| Audit log | The platform | super_admin, org_admin | System only (append) |

### 19.2 Data Retention Rules

| Data Type | Retention Period | After Retention |
|---|---|---|
| Child learning data (step_attempts) | 3 years | Aggregated, then raw deleted |
| Child profile (active) | While enrolled + 1 year | Anonymized after 1 year |
| Child profile (after leaving school) | 1 year | Soft deleted, name encrypted removed |
| Audit log | 7 years | Archived to cold storage |
| Seva activity data | 3 years | Photo deleted, metadata retained |
| Session data | 7 days after expiry | Hard deleted |
| OTP codes | 24 hours after expiry/consumption | Hard deleted |
| Sync queue (completed) | 30 days | Hard deleted |
| Notifications (read) | 90 days | Hard deleted |
| Notifications (unread) | 180 days | Hard deleted |

### 19.3 PII Handling

| PII Field | Storage | Encryption | Access |
|---|---|---|---|
| User email | `users.email` | At rest (pgcrypto) | User, admins |
| User phone | `users.phone` | At rest (pgcrypto) | User, admins |
| User password_hash | `users.password_hash` | bcrypt (one-way) | Nobody (verified only) |
| Child full name | `child_profiles.name_encrypted` | pgcrypto AES-256 | Nobody in plaintext (searched via HMAC hash) |
| Child display_name | `child_profiles.display_name` | Plaintext (first name only) | Teachers, parents (linked) |
| Child date_of_birth | `child_profiles.date_of_birth` | At rest (pgcrypto) | Teachers, parents (linked) |
| Seva photo | MinIO/S3 | Server-side encryption | Verifying teacher, admins |
| OTP code | `otp_codes.code_hash` | bcrypt (one-way) | Nobody (verified only) |
| Session tokens | `user_sessions.refresh_token_hash` | SHA-256 (one-way) | Nobody (verified only) |
| IP address (audit) | `audit_log.ip_address` | Plaintext (operational need) | super_admin, org_admin |
| Geolocation (Seva) | `seva_activities.geolocation` | Plaintext (operational need) | Verifying teacher, admins |

---

## 20. Row-Level Security Policies

PostgreSQL Row-Level Security (RLS) ensures that even if the API layer is bypassed, the database enforces data access rules.

### 20.1 Enable RLS

```sql
-- Enable RLS on all child-related tables
ALTER TABLE child_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE chapter_progress ENABLE ROW LEVEL SECURITY;
ALTER TABLE step_attempts ENABLE ROW LEVEL SECURITY;
ALTER TABLE concept_mastery ENABLE ROW LEVEL SECURITY;
ALTER TABLE child_badges ENABLE ROW LEVEL SECURITY;
ALTER TABLE xp_ledger ENABLE ROW LEVEL SECURITY;
ALTER TABLE coin_ledger ENABLE ROW LEVEL SECURITY;
ALTER TABLE streaks ENABLE ROW LEVEL SECURITY;
ALTER TABLE daily_challenges ENABLE ROW LEVEL SECURITY;
ALTER TABLE shop_purchases ENABLE ROW LEVEL SECURITY;
ALTER TABLE saving_goals ENABLE ROW LEVEL SECURITY;
ALTER TABLE seva_activities ENABLE ROW LEVEL SECURITY;
ALTER TABLE game_scores ENABLE ROW LEVEL SECURITY;
ALTER TABLE worksheet_scores ENABLE ROW LEVEL SECURITY;
ALTER TABLE leaderboards ENABLE ROW LEVEL SECURITY;
ALTER TABLE sync_queue ENABLE ROW LEVEL SECURITY;
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;

-- super_admin bypasses RLS
ALTER TABLE child_profiles FORCE ROW LEVEL SECURITY;
-- (repeat for all above tables)
```

### 20.2 Policy Examples

```sql
-- Child profiles: a user can see children they are linked to
CREATE POLICY child_profile_read ON child_profiles FOR SELECT
    USING (
        -- super_admin and org_admin see all in their org
        current_setting('app.user_role') IN ('super_admin', 'org_admin')
        -- school_admin sees children in their school
        OR (current_setting('app.user_role') = 'school_admin'
            AND school_id::text = current_setting('app.school_id'))
        -- teacher sees children in their classes
        OR (current_setting('app.user_role') = 'teacher'
            AND id IN (
                SELECT child_id FROM class_enrolments
                WHERE class_id IN (
                    SELECT id FROM classes
                    WHERE class_teacher_id::text = current_setting('app.user_id')
                    AND is_active = TRUE
                ) AND is_active = TRUE
            ))
        -- parent sees their own children
        OR (current_setting('app.user_role') = 'parent'
            AND primary_parent_id::text = current_setting('app.user_id'))
    );

-- Chapter progress: same access rules as child_profiles
CREATE POLICY progress_read ON chapter_progress FOR SELECT
    USING (
        child_id IN (
            SELECT id FROM child_profiles
            WHERE -- same conditions as child_profile_read
                current_setting('app.user_role') IN ('super_admin', 'org_admin')
                OR (current_setting('app.user_role') = 'school_admin'
                    AND school_id::text = current_setting('app.school_id'))
                OR (current_setting('app.user_role') = 'parent'
                    AND primary_parent_id::text = current_setting('app.user_id'))
        )
    );

-- Sync queue: only the linked teacher/parent can push/pull
CREATE POLICY sync_write ON sync_queue FOR ALL
    USING (
        child_id IN (
            SELECT id FROM child_profiles
            WHERE primary_parent_id::text = current_setting('app.user_id')
            OR id IN (
                SELECT child_id FROM class_enrolments
                WHERE class_id IN (
                    SELECT id FROM classes
                    WHERE class_teacher_id::text = current_setting('app.user_id')
                )
            )
        )
    );

-- Notifications: only the recipient can read
CREATE POLICY notif_read ON notifications FOR SELECT
    USING (recipient_id::text = current_setting('app.user_id'));

-- Audit log: only super_admin and org_admin
CREATE POLICY audit_read ON audit_log FOR SELECT
    USING (
        current_setting('app.user_role') IN ('super_admin', 'org_admin')
    );
```

### 20.3 Session Variable Setting

The API sets session variables on each request so RLS policies can use them:

```sql
-- Executed at the start of each API request (in a transaction)
SET LOCAL app.user_id = '550e8400-e29b-41d4-a716-446655440000';
SET LOCAL app.user_role = 'teacher';
SET LOCAL app.school_id = '660e8400-e29b-41d4-a716-446655440000';
```

---

## 21. Triggers & Functions

### 21.1 `updated_at` Auto-Update Trigger

```sql
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to all tables with updated_at
CREATE TRIGGER trg_users_updated BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER trg_child_profiles_updated BEFORE UPDATE ON child_profiles
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER trg_chapter_progress_updated BEFORE UPDATE ON chapter_progress
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER trg_concept_mastery_updated BEFORE UPDATE ON concept_mastery
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER trg_streaks_updated BEFORE UPDATE ON streaks
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER trg_seva_activities_updated BEFORE UPDATE ON seva_activities
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER trg_sync_queue_updated BEFORE UPDATE ON sync_queue
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
-- (repeat for all tables with updated_at)
```

### 21.2 XP Balance Trigger (Maintains `child_profiles.total_xp`)

```sql
CREATE OR REPLACE FUNCTION update_child_xp_balance()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE child_profiles
    SET total_xp = total_xp + NEW.amount,
        updated_at = NOW()
    WHERE id = NEW.child_id;
    
    -- Also update current_level_id if XP crossed a threshold
    UPDATE child_profiles
    SET current_level_id = (
        SELECT id FROM levels
        WHERE xp_threshold <= (SELECT total_xp FROM child_profiles WHERE id = NEW.child_id)
        ORDER BY xp_threshold DESC
        LIMIT 1
    )
    WHERE id = NEW.child_id;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_xp_ledger_insert AFTER INSERT ON xp_ledger
    FOR EACH ROW EXECUTE FUNCTION update_child_xp_balance();
```

### 21.3 Coin Balance Trigger (Maintains `child_profiles.total_coins`)

```sql
CREATE OR REPLACE FUNCTION update_child_coin_balance()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.type IN ('earn', 'reward', 'withdraw') THEN
        UPDATE child_profiles
        SET total_coins = total_coins + NEW.amount,
            updated_at = NOW()
        WHERE id = NEW.child_id;
    ELSIF NEW.type IN ('spend', 'save') THEN
        UPDATE child_profiles
        SET total_coins = total_coins + NEW.amount,  -- amount is negative for spend/save
            updated_at = NOW()
        WHERE id = NEW.child_id;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_coin_ledger_insert AFTER INSERT ON coin_ledger
    FOR EACH ROW EXECUTE FUNCTION update_child_coin_balance();
```

### 21.4 Concept Mastery Auto-Update Trigger

```sql
CREATE OR REPLACE FUNCTION update_concept_mastery()
RETURNS TRIGGER AS $$
DECLARE
    mastery_pct NUMERIC(5,2);
    total INTEGER;
    correct INTEGER;
    first_try INTEGER;
BEGIN
    -- Recalculate mastery for this child + node
    SELECT 
        COUNT(*),
        COUNT(*) FILTER (WHERE result = 'correct'),
        COUNT(*) FILTER (WHERE result = 'correct' AND attempt_number = 1)
    INTO total, correct, first_try
    FROM step_attempts
    WHERE child_id = NEW.child_id
    AND node_id = NEW.node_id;
    
    mastery_pct := CASE WHEN total > 0 THEN (correct::NUMERIC / total) * 100 ELSE 0 END;
    
    -- Upsert into concept_mastery
    INSERT INTO concept_mastery (child_id, node_id, chapter_id, mastery_percentage, mastery_level,
                                   total_attempts, correct_attempts, first_try_correct, is_mastered, mastered_at, updated_at)
    VALUES (NEW.child_id, NEW.node_id, NEW.chapter_id, mastery_pct,
            CASE 
                WHEN mastery_pct >= 80 THEN 'mastered'
                WHEN mastery_pct >= 40 THEN 'practicing'
                WHEN total > 0 THEN 'learning'
                ELSE 'not_started'
            END,
            total, correct, first_try,
            mastery_pct >= 80 AND correct >= 3,
            CASE WHEN mastery_pct >= 80 AND correct >= 3 THEN NOW() ELSE NULL END,
            NOW())
    ON CONFLICT (child_id, node_id) DO UPDATE
    SET mastery_percentage = EXCLUDED.mastery_percentage,
        mastery_level = EXCLUDED.mastery_level,
        total_attempts = EXCLUDED.total_attempts,
        correct_attempts = EXCLUDED.correct_attempts,
        first_try_correct = EXCLUDED.first_try_correct,
        is_mastered = EXCLUDED.is_mastered,
        mastered_at = COALESCE(concept_mastery.mastered_at, EXCLUDED.mastered_at),
        updated_at = NOW();
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_step_attempt_insert AFTER INSERT ON step_attempts
    FOR EACH ROW EXECUTE FUNCTION update_concept_mastery();
```

### 21.5 Streak Update Trigger

```sql
CREATE OR REPLACE FUNCTION update_streak()
RETURNS TRIGGER AS $$
DECLARE
    today DATE := CURRENT_DATE;
    streak_row RECORD;
BEGIN
    SELECT * INTO streak_row FROM streaks WHERE child_id = NEW.child_id;
    
    IF NOT FOUND THEN
        INSERT INTO streaks (child_id, current_streak, longest_streak, last_correct_date, streak_started_at)
        VALUES (NEW.child_id, 1, 1, today, NOW());
    ELSIF NEW.result = 'correct' THEN
        IF streak_row.last_correct_date = today THEN
            -- Already answered correctly today, no change
            RETURN NEW;
        ELSIF streak_row.last_correct_date = today - 1 THEN
            -- Continue streak
            UPDATE streaks SET 
                current_streak = current_streak + 1,
                longest_streak = GREATEST(longest_streak, current_streak + 1),
                last_correct_date = today,
                updated_at = NOW()
            WHERE child_id = NEW.child_id;
        ELSE
            -- Streak broken (or first ever)
            UPDATE streaks SET 
                current_streak = 1,
                last_correct_date = today,
                streak_started_at = NOW(),
                updated_at = NOW()
            WHERE child_id = NEW.child_id;
        END IF;
        
        -- Update child_profiles denormalized streak
        UPDATE child_profiles SET current_streak = current_streak + 1, updated_at = NOW()
        WHERE id = NEW.child_id;
    ELSIF NEW.result = 'incorrect' THEN
        -- Only break streak if no streak freeze available
        IF streak_row.freezes_available > 0 THEN
            UPDATE streaks SET 
                freezes_available = freezes_available - 1,
                freezes_used = freezes_used + 1,
                updated_at = NOW()
            WHERE child_id = NEW.child_id;
            -- Streak preserved
        ELSE
            UPDATE streaks SET 
                current_streak = 0,
                updated_at = NOW()
            WHERE child_id = NEW.child_id;
            UPDATE child_profiles SET current_streak = 0, updated_at = NOW()
            WHERE id = NEW.child_id;
        END IF;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_streak_update AFTER INSERT ON step_attempts
    FOR EACH ROW EXECUTE FUNCTION update_streak();
```

### 21.6 Audit Log Trigger

```sql
CREATE OR REPLACE FUNCTION log_audit_event()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (actor_id, action, entity_type, entity_id, old_values, new_values)
        VALUES (
            NULLIF(current_setting('app.user_id', true), '')::uuid,
            'update_' || TG_TABLE_NAME,
            TG_TABLE_NAME,
            NEW.id,
            to_jsonb(OLD),
            to_jsonb(NEW)
        );
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (actor_id, action, entity_type, entity_id, old_values)
        VALUES (
            NULLIF(current_setting('app.user_id', true), '')::uuid,
            'delete_' || TG_TABLE_NAME,
            TG_TABLE_NAME,
            OLD.id,
            to_jsonb(OLD)
        );
    END IF;
    RETURN NULL; -- After trigger
END;
$$ LANGUAGE plpgsql;

-- Apply to sensitive tables
CREATE TRIGGER trg_audit_child_profiles AFTER UPDATE OR DELETE ON child_profiles
    FOR EACH ROW EXECUTE FUNCTION log_audit_event();
CREATE TRIGGER trg_audit_users AFTER UPDATE OR DELETE ON users
    FOR EACH ROW EXECUTE FUNCTION log_audit_event();
CREATE TRIGGER trg_audit_shop_purchases AFTER INSERT ON shop_purchases
    FOR EACH ROW EXECUTE FUNCTION log_audit_event();
CREATE TRIGGER trg_audit_store_redemptions AFTER INSERT ON store_redemptions
    FOR EACH ROW EXECUTE FUNCTION log_audit_event();
CREATE TRIGGER trg_audit_seva AFTER UPDATE ON seva_activities
    FOR EACH ROW EXECUTE FUNCTION log_audit_event();
```

---

## 22. Migration & Seed Data

### 22.1 Migration Order

```sql
-- 1. Extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- 2. Enum types (Section 2)
-- 3. Organisation tables (Section 5)
-- 4. User tables (Section 3, 4)
-- 5. Device tables (Section 4.4)
-- 6. Content tables (Section 7)
-- 7. Child profile tables (Section 6)
-- 8. Class/enrolment tables (Section 5.3, 5.4)
-- 9. Learning progress tables (Section 8)
-- 10. Assessment tables (Section 9)
-- 11. Gamification tables (Section 10)
-- 12. Economy tables (Section 11)
-- 13. Seva tables (Section 12)
-- 14. Sync tables (Section 13)
-- 15. Operations tables (Section 14)
-- 16. Indexes (Section 15)
-- 17. Triggers (Section 21)
-- 18. RLS policies (Section 20)
-- 19. Seed data (below)
```

### 22.2 Seed Data

```sql
-- Organisations
INSERT INTO organisations (name, slug) VALUES
('Annanth Aasha Foundation', 'annanth-aasha');

-- Subjects
INSERT INTO subjects (code, name, name_hindi, icon, color_hex, sort_order) VALUES
('mathematics', 'Mathematics', 'गणित', '📐', '#3b82f6', 1),
('science', 'Science', 'विज्ञान', '🔬', '#10b981', 2),
('english', 'English', 'अंग्रेजी', '📖', '#a855f7', 3),
('hindi', 'Hindi', 'हिन्दी', '📚', '#f59e0b', 4),
('social_studies', 'Social Studies', 'सामाजिक विज्ञान', '🗺️', '#0d9488', 5),
('general_knowledge', 'General Knowledge', 'सामान्य ज्ञान', '🌍', '#ec4899', 6);

-- Levels (already seeded in Section 10.1)
-- Badges (already seeded in Section 10.2)
-- Shop items (already seeded in Section 11.2)

-- LLE connectives
INSERT INTO lle_connectives (english_word, hindi_translation, hindi_transliteration) VALUES
('because', 'क्योंकि', 'kyunki'),
('therefore', 'इसलिए', 'isliye'),
('however', 'हालांकि', 'halanki'),
('although', 'यद्यपि', 'yadyapi'),
('since', 'चूंकि', 'chunki'),
('so that', 'ताकि', 'taaki'),
('if', 'यदि', 'yadi'),
('unless', 'जब तक', 'jab tak'),
('while', 'जबकि', 'jabki'),
('whereas', 'जबकि', 'jabki'),
('hence', 'अतः', 'atah'),
('thus', 'इस प्रकार', 'is prakaar');

-- Seva types catalog
INSERT INTO seva_types_catalog (seva_type, activity_name, description, icon, base_coins, photo_bonus_coins, sort_order) VALUES
('eco_seva', 'Plant a Seed', 'Plant a seed and care for it daily', '🌱', 20, 5, 1),
('eco_seva', 'Community Cleanliness', 'Help clean your surroundings', '🧹', 20, 5, 2),
('jal_seva', 'Water Conservation', 'Use water responsibly for a day', '💧', 20, 5, 3);

-- App config defaults
INSERT INTO app_config (config_key, config_value, description) VALUES
('max_child_profiles_per_device', '5', 'Maximum child profiles on one device'),
('sync_interval_seconds', '300', 'How often to sync (5 minutes)'),
('daily_challenge_question_count', '5', 'Questions per daily challenge'),
('rapid_fire_question_count', '10', 'Questions per rapid fire round'),
('rapid_fire_timer_seconds', '10', 'Seconds per rapid fire question'),
('quiz_timer_seconds', '10', 'Seconds per timed quiz question'),
('speed_bonus_threshold_3s', '3', 'Answer within 3s for +3 XP'),
('speed_bonus_threshold_6s', '6', 'Answer within 6s for +2 XP'),
('speed_bonus_threshold_10s', '10', 'Answer within 10s for +1 XP'),
('mastery_threshold_percentage', '80', 'Mastery percentage to mark concept mastered'),
('mastery_minimum_correct', '3', 'Minimum correct answers for mastery'),
('seva_verification_timeout_days', '30', 'Days before Seva expires if unverified'),
('otp_expiry_minutes', '5', 'OTP validity in minutes'),
('otp_max_attempts', '5', 'Maximum OTP attempts'),
('session_refresh_ttl_days', '7', 'Refresh token TTL in days'),
('session_access_ttl_minutes', '15', 'Access token TTL in minutes'),
('password_bcrypt_cost', '12', 'bcrypt cost factor'),
('max_file_size_chapter_mb', '20', 'Maximum chapter HTML file size in MB'),
('seva_photo_max_size_mb', '5', 'Maximum Seva photo size in MB');
```

---

**End of Backend Schema Document**

This document defines the complete database schema for the Aasha learning platform. It contains 43 tables across 9 domain layers, with full column definitions, data types, primary keys, foreign keys, relationships, indexes, enum types, triggers, row-level security policies, authentication/session handling, a permissions matrix, data ownership rules, and seed data. An AI coding agent or backend engineer can implement this schema without ambiguity.
