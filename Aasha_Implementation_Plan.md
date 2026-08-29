# Aasha — Step-by-Step Implementation Plan

**Product:** Aasha — Safe Learning. Real Impact.  
**Organisation:** Annanth Aasha Foundation  
**Document Type:** Implementation Plan  
**Version:** 1.0  
**Date:** August 2026  
**Status:** Ready for Execution  
**Author:** Senior Full-Stack Engineer & Project Manager  

**Purpose:** This document is the single source of truth for building the Aasha platform from current prototype (3 offline HTML chapters) to full ecosystem. It is sequenced so that each phase's deliverables unblock the next phase. An AI coding agent or engineering team can follow this without guessing what to build first, what depends on what, or when something is "done."

**Guiding Rules:**
1. Each phase has a **definition of done** — if the deliverables aren't shipped, the phase isn't complete.
2. No phase skips testing. Every phase ships with automated tests.
3. The offline chapter files are never broken — they must always work as standalone HTML.
4. The backend is always optional from the child's perspective — the app works without it.
5. Every commit is deployable. No "integration week" at the end.

---

## Table of Contents

1. [Current State Assessment](#1-current-state-assessment)
2. [Architecture Decisions Recap](#2-architecture-decisions-recap)
3. [Phase 0 — Project Setup & Tooling](#phase-0--project-setup--tooling)
4. [Phase 1 — Database Foundation](#phase-1--database-foundation)
5. [Phase 2 — Authentication & Session Management](#phase-2--authentication--session-management)
6. [Phase 3 — Core API Layer](#phase-3--core-api-layer)
7. [Phase 4 — Chapter Content Pipeline](#phase-4--chapter-content-pipeline)
8. [Phase 5 — Sync Layer (Offline → Server)](#phase-5--sync-layer-offline--server)
9. [Phase 6 — Teacher Dashboard](#phase-6--teacher-dashboard)
10. [Phase 7 — Parent Dashboard](#phase-7--parent-dashboard)
11. [Phase 8 — Gamification Hardening](#phase-8--gamification-hardening)
12. [Phase 9 — Seva Activity System](#phase-9--seva-activity-system)
13. [Phase 10 — Aasha Economy (Wallet, Store, Saving)](#phase-10--aasha-economy-wallet-store-saving)
14. [Phase 11 — Chapter Enhancement (Rich Media)](#phase-11--chapter-enhancement-rich-media)
15. [Phase 12 — Testing & QA](#phase-12--testing--qa)
16. [Phase 13 — Deployment & CI/CD](#phase-13--deployment--cicd)
17. [Phase 14 — Final Polish & Launch](#phase-14--final-polish--launch)
18. [Dependency Graph](#18-dependency-graph)
19. [Effort Estimates](#19-effort-estimates)
20. [Risk Register](#20-risk-register)

---

## 1. Current State Assessment

### 1.1 What Is Already Built (Phase 1 Prototype — Shipped)

| Deliverable | Status | Artifact |
|---|---|---|
| Quadrilaterals chapter (gamified, 22 nodes, 201 steps) | ✅ Shipped | HTML file, 416 KB |
| Fractions chapter (20 nodes, 193 steps) | ✅ Shipped | HTML file, 387 KB |
| Comparing Quantities chapter (11 nodes, 134 steps) | ✅ Shipped | HTML file, 340 KB |
| Gamification features (timed quiz, rapid fire, memory match, gem shop, themes) | ✅ Shipped | In Quadrilaterals chapter |
| LLE (Language Learning Engine, 78+ words) | ✅ Shipped | In all chapters |
| localStorage state management | ✅ Shipped | Profile picker, progress, economy |
| Canvas-based interactive visuals | ✅ Shipped | Drag, slider, diagonal drawing |
| Web Audio API sound feedback | ✅ Shipped | Oscillator-based tones |
| PRD (comprehensive) | ✅ Shipped | 29 KB markdown |
| TRD (comprehensive) | ✅ Shipped | 60 KB markdown |
| App Flow Document | ✅ Shipped | 56 KB markdown, 44 screens |
| UI/UX Design Brief | ✅ Shipped | 50 KB markdown |
| Backend Schema | ✅ Shipped | 110 KB markdown, 43 tables |

### 1.2 What Is NOT Built (This Plan Addresses)

- Backend server (API, database, sync)
- Teacher dashboard (web app)
- Parent dashboard (web app)
- Sync layer (offline → online state synchronization)
- Seva activity system
- Aasha Economy (wallet, store, saving goals)
- Daily challenge system
- Content management / authoring tooling
- CI/CD pipeline
- Automated test suite (beyond chapter data validation)
- Monitoring and logging infrastructure
- Production deployment

---

## 2. Architecture Decisions Recap

| Decision | Choice | Rationale |
|---|---|---|
| Chapter files | Vanilla JS, self-contained HTML, ≤20 MB | Offline-first, zero dependencies |
| Backend framework | Fastify (Node.js 20) | Schema validation, 2× faster than Express |
| Database | PostgreSQL 16 | ACID, JSONB, pgvector for future AI |
| Cache | Redis 7 | Session revocation, rate limiting |
| File storage | MinIO (S3-compatible) | Chapter files, Seva photos |
| Reverse proxy | Caddy | Auto-HTTPS, 10-line config |
| Dashboards | React 18 + Vite + Tailwind | Complex UI needs component model |
| Auth | JWT RS256 + phone OTP | Stateless, mobile-friendly |
| Deployment | Docker Compose on Hetzner VPS | ₹500/month, NGO budget |
| CDN | Cloudflare free tier | Chapter file distribution |
| Monitoring | Uptime Kuma + Loki + Grafana | Open source, self-hosted |
| CI/CD | GitHub Actions | Free for repos, automated |

---

## Phase 0 — Project Setup & Tooling

**Goal:** Establish the monorepo, tooling, and development environment so every subsequent phase has a foundation.

**Duration:** 2 days

### Step 0.1 — Create Monorepo Structure

```
aasha/
├── chapters/                    # Offline HTML chapter files (the product)
│   ├── quadrilaterals/
│   ├── fractions/
│   └── comparing-quantities/
├── server/                      # Backend (Fastify API)
│   ├── src/
│   │   ├── plugins/            # Fastify plugins (auth, cors, rate-limit)
│   │   ├── routes/             # API route handlers
│   │   ├── services/           # Business logic
│   │   ├── db/                 # Database migrations, seeds, queries
│   │   ├── utils/              # Shared utilities
│   │   └── config/             # Environment config
│   ├── tests/
│   ├── package.json
│   └── tsconfig.json
├── dashboard/                   # Teacher/Parent dashboard (React)
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── vite.config.ts
├── packages/                    # Shared packages
│   ├── shared-types/           # TypeScript types shared between server & dashboard
│   ├── chapter-builder/        # Content authoring tools
│   └── test-harness/           # Chapter validation scripts
├── infra/                       # Infrastructure
│   ├── docker-compose.yml
│   ├── docker-compose.prod.yml
│   ├── Caddyfile
│   └── .env.example
├── .github/
│   └── workflows/
│       ├── test.yml
│       └── deploy.yml
├── package.json                 # Root workspace
└── README.md
```

### Step 0.2 — Initialize Node.js Workspace

```json
// Root package.json
{
  "name": "aasha",
  "private": true,
  "workspaces": ["server", "dashboard", "packages/*"],
  "scripts": {
    "dev:server": "cd server && npm run dev",
    "dev:dashboard": "cd dashboard && npm run dev",
    "test": "wsrun -t test",
    "lint": "wsrun -t lint",
    "build:all": "wsrun -t build"
  }
}
```

### Step 0.3 — Setup TypeScript, ESLint, Prettier

- TypeScript 5.4+ across server and dashboard
- ESLint with `@typescript-eslint` plugin
- Prettier for code formatting
- Shared `tsconfig.base.json` with strict mode

### Step 0.4 — Setup Docker Compose for Local Development

```yaml
# infra/docker-compose.yml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: aasha_dev
      POSTGRES_USER: aasha
      POSTGRES_PASSWORD: dev_password
    ports: ["5432:5432"]
    volumes: [pg_data:/var/lib/postgresql/data]
  
  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
  
  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    ports: ["9000:9000", "9001:9001"]
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: minio_password
    volumes: [minio_data:/data]

volumes:
  pg_data:
  minio_data:
```

### Step 0.5 — Setup Environment Configuration

```bash
# infra/.env.example
DATABASE_URL=postgres://aasha:dev_password@localhost:5432/aasha_dev
REDIS_URL=redis://localhost:6379
MINIO_ENDPOINT=localhost
MINIO_PORT=9000
MINIO_ACCESS_KEY=minio
MINIO_SECRET_KEY=minio_password
JWT_PRIVATE_KEY_PATH=./keys/private.pem
JWT_PUBLIC_KEY_PATH=./keys/public.pem
MSG91_AUTH_KEY=your_msg91_key
NODE_ENV=development
PORT=3000
```

### Step 0.6 — Generate JWT RSA Key Pair

```bash
openssl genrsa -out keys/private.pem 2048
openssl rsa -in keys/private.pem -pubout -out keys/public.pem
```

### Step 0.7 — Copy Existing Chapter Files

Copy the 3 shipped chapter HTML files into `chapters/` directory. These are the product — they must not break.

### Phase 0 Deliverables

- [ ] Monorepo with `server/`, `dashboard/`, `packages/`, `infra/`, `chapters/` directories
- [ ] Node.js workspace with npm workspaces
- [ ] TypeScript, ESLint, Prettier configured
- [ ] Docker Compose for local PostgreSQL, Redis, MinIO
- [ ] `.env.example` with all required variables
- [ ] RSA key pair for JWT signing
- [ ] 3 existing chapter files copied into `chapters/`
- [ ] README with setup instructions
- [ ] `docker compose up` starts DB, Redis, MinIO without errors

**Definition of Done:** A new developer can clone the repo, run `docker compose up`, `npm install`, and have a working development environment in under 15 minutes.

---

## Phase 1 — Database Foundation

**Goal:** Create the PostgreSQL database schema with all 43 tables, indexes, triggers, RLS policies, and seed data.

**Duration:** 3 days

### Step 1.1 — Create Migration System

Use `node-pg-migrate` or `db-migrate` for versioned SQL migrations:

```
server/src/db/migrations/
├── 001_create_extensions.sql          # uuid-ossp, pgcrypto
├── 002_create_enums.sql               # All 25 enum types
├── 003_create_org_tables.sql           # organisations, schools, classes, enrolments
├── 004_create_user_tables.sql          # users, user_roles
├── 005_create_auth_tables.sql          # user_sessions, otp_codes, password_resets, devices
├── 006_create_content_tables.sql       # subjects, chapters, nodes, steps, question_bank, LLE
├── 007_create_child_tables.sql         # child_profiles
├── 008_create_progress_tables.sql      # chapter_progress, step_attempts, concept_mastery
├── 009_create_assessment_tables.sql    # worksheet_scores, game_scores, assessment_attempts
├── 010_create_gamification_tables.sql  # levels, badges, child_badges, xp_ledger, streaks, daily_challenges, leaderboards
├── 011_create_economy_tables.sql       # coin_ledger, shop_items, shop_purchases, saving_goals, store_items, store_redemptions
├── 012_create_seva_tables.sql          # seva_activities, seva_types_catalog
├── 013_create_sync_tables.sql          # sync_queue, sync_conflicts
├── 014_create_ops_tables.sql           # audit_log, notifications, app_config
├── 015_create_indexes.sql             # All 75+ indexes
├── 016_create_triggers.sql             # All 6 trigger functions
├── 017_create_rls_policies.sql        # Row-Level Security on all child tables
├── 018_seed_levels.sql                # 5 levels
├── 019_seed_badges.sql                # 5 badges
├── 020_seed_shop_items.sql             # 7 shop items
├── 021_seed_subjects.sql              # 6 subjects
├── 022_seed_lle_connectives.sql        # 12 connectives
├── 023_seed_seva_types.sql             # 3 Seva types
├── 024_seed_app_config.sql            # 18 config defaults
└── 025_seed_organisation.sql          # Annanth Aasha Foundation
```

### Step 1.2 — Implement Migrations

Run each migration file in order against the local PostgreSQL. Verify:
- All 43 tables created
- All 25 enum types created
- All foreign keys and constraints valid
- All indexes created
- All triggers active
- RLS enabled on all child-related tables
- All seed data inserted

### Step 1.3 — Create Database Query Helpers

```typescript
// server/src/db/client.ts
import { Pool } from 'pg';

export const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
});

// Set session variables for RLS
export async function setUserContext(
  client: PoolClient,
  userId: string,
  role: string,
  schoolId?: string
) {
  await client.query('SET LOCAL app.user_id = $1', [userId]);
  await client.query('SET LOCAL app.user_role = $1', [role]);
  if (schoolId) {
    await client.query('SET LOCAL app.school_id = $1', [schoolId]);
  }
}
```

### Step 1.4 — Create TypeScript Types from Schema

Generate TypeScript interfaces for all 43 tables using `pg-to-typescript` or manually create in `packages/shared-types/`:

```typescript
// packages/shared-types/src/index.ts
export interface User { id: string; email?: string; phone?: string; ... }
export interface ChildProfile { id: string; displayName: string; ... }
export interface ChapterProgress { id: string; childId: string; ... }
// ... all 43 tables
```

### Phase 1 Deliverables

- [ ] 25 migration files covering all 43 tables, enums, indexes, triggers, RLS
- [ ] Database query client with connection pooling
- [ ] RLS session variable setter
- [ ] TypeScript types for all tables
- [ ] Seed data: 5 levels, 5 badges, 7 shop items, 6 subjects, 12 LLE connectives, 3 Seva types, 18 config values, 1 organisation
- [ ] Migration can be run from scratch: `npm run db:migrate` creates everything
- [ ] Migration can be rolled back: `npm run db:rollback`

**Definition of Done:** Running `npm run db:migrate` on a fresh database creates all 43 tables with correct schema, indexes, triggers, RLS, and seed data. `\dt` in psql shows all tables. Trigger tests pass.

---

## Phase 2 — Authentication & Session Management

**Goal:** Implement the complete authentication system: email/password, phone OTP, JWT issuance, refresh token rotation, session revocation.

**Duration:** 4 days

### Step 2.1 — Auth Plugin (Fastify)

```typescript
// server/src/plugins/auth.ts
import fp from 'fastify-plugin';
import jwt from 'fastify-jwt';
import { verifyAccessToken, verifyRefreshToken, issueTokens } from '../services/auth';

export default fp(async (fastify) => {
  fastify.register(jwt, {
    secret: { private: privateKey, public: publicKey },
    sign: { algorithm: 'RS256', expiresIn: '15m', issuer: 'aasha-platform' },
  });
  
  // Decorate request with user
  fastify.decorate('authenticate', async (request, reply) => {
    try {
      await request.jwtVerify();
      // Check session in Redis revocation list
      const revoked = await fastify.redis.sismember('revoked_jti', request.user.jti);
      if (revoked) throw new Error('Token revoked');
    } catch (err) {
      reply.code(401).send({ status: 'error', error: { code: 'UNAUTHORIZED' } });
    }
  });
});
```

### Step 2.2 — Auth Routes

Build these API endpoints:

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/v1/auth/register` | Email + password registration |
| POST | `/api/v1/auth/login` | Email + password login |
| POST | `/api/v1/auth/otp/send` | Send OTP to phone |
| POST | `/api/v1/auth/otp/verify` | Verify OTP, issue JWT |
| POST | `/api/v1/auth/refresh` | Refresh access token |
| POST | `/api/v1/auth/logout` | Revoke session |
| POST | `/api/v1/auth/password/forgot` | Send password reset email |
| POST | `/api/v1/auth/password/reset` | Reset password with token |
| GET | `/api/v1/auth/me` | Get current user profile |

### Step 2.3 — OTP Service

```typescript
// server/src/services/otp.ts
import bcrypt from 'bcrypt';
import crypto from 'crypto';
import { pool } from '../db/client';
import { sendSMS } from './sms';

export async function sendOTP(phone: string): Promise<void> {
  // Rate limit: 1 OTP per 60s per phone
  const recent = await pool.query(
    'SELECT created_at FROM otp_codes WHERE phone = $1 AND created_at > NOW() - INTERVAL \'60 seconds\' ORDER BY created_at DESC LIMIT 1',
    [phone]
  );
  if (recent.rows.length > 0) {
    throw new Error('RATE_LIMITED');
  }
  
  // Generate 6-digit code
  const code = String(crypto.randomInt(100000, 999999));
  const codeHash = await bcrypt.hash(code, 10);
  
  await pool.query(
    'INSERT INTO otp_codes (phone, code_hash, purpose, expires_at) VALUES ($1, $2, $3, NOW() + INTERVAL \'5 minutes\')',
    [phone, codeHash, 'login']
  );
  
  await sendSMS(phone, `Your Aasha verification code is ${code}. Valid for 5 minutes.`);
}
```

### Step 2.4 — JWT Issuance & Refresh

```typescript
// server/src/services/auth.ts
export async function issueTokens(userId: string, role: string, deviceId?: string) {
  const jti = uuidv4();
  const refreshToken = crypto.randomBytes(32).toString('base64url');
  const refreshTokenHash = crypto.createHash('sha256').update(refreshToken).digest('hex');
  
  // Store session
  await pool.query(
    'INSERT INTO user_sessions (user_id, refresh_token_hash, jwt_jti, ip_address, device_id, expires_at) VALUES ($1, $2, $3, $4, $5, NOW() + INTERVAL \'7 days\')',
    [userId, refreshTokenHash, jti, ipAddress, deviceId]
  );
  
  // Sign access token
  const accessToken = fastify.jwt.sign({ sub: userId, role, jti });
  
  return { accessToken, refreshToken };
}

export async function refreshTokens(oldRefreshToken: string) {
  const hash = crypto.createHash('sha256').update(oldRefreshToken).digest('hex');
  const session = await pool.query(
    'SELECT * FROM user_sessions WHERE refresh_token_hash = $1 AND status = $2 AND expires_at > NOW()',
    [hash, 'active']
  );
  
  if (session.rows.length === 0) throw new Error('INVALID_REFRESH_TOKEN');
  
  // Mark old session as replaced
  const newSession = await issueTokens(session.rows[0].user_id, ...);
  await pool.query(
    'UPDATE user_sessions SET status = $1, replaced_by = $2 WHERE id = $3',
    ['replaced', newSession.sessionId, session.rows[0].id]
  );
  
  return newSession;
}
```

### Step 2.5 — Role-Based Access Control Middleware

```typescript
// server/src/plugins/rbac.ts
export function requireRole(...roles: user_role[]) {
  return async (request: FastifyRequest, reply: FastifyReply) => {
    if (!roles.includes(request.user.role)) {
      reply.code(403).send({ status: 'error', error: { code: 'FORBIDDEN' } });
    }
  };
}

// Usage:
fastify.post('/api/v1/chapters', { preHandler: [fastify.authenticate, requireRole('super_admin', 'org_admin')] }, handler);
```

### Step 2.6 — Auth Tests

- Unit tests: bcrypt verification, JWT signing/verification, OTP generation
- Integration tests: register → login → me → refresh → logout flow
- Rate limit tests: 2nd OTP within 60s rejected, 6th login attempt rejected
- RLS tests: teacher cannot read another teacher's class data

### Phase 2 Deliverables

- [ ] Fastify auth plugin with JWT RS256 verification
- [ ] 9 auth API endpoints (register, login, OTP send/verify, refresh, logout, password reset, me)
- [ ] OTP service with rate limiting (1/min, 5/hour, 10/day per phone)
- [ ] Refresh token rotation (old session marked `replaced`, new session created)
- [ ] Redis-based JWT revocation list (for logout/password change)
- [ ] RBAC middleware with role checking
- [ ] Password hashing (bcrypt cost 12)
- [ ] Auth test suite: 30+ tests covering all flows and edge cases
- [ ] Postman collection or OpenAPI spec for all auth endpoints

**Definition of Done:** A teacher can register with email/password or phone OTP, receive JWT tokens, refresh them, and log out. Sessions are tracked in PostgreSQL and revocable via Redis. All auth tests pass.

---

## Phase 3 — Core API Layer

**Goal:** Build the CRUD and business-logic APIs for all non-auth entities: users, children, chapters, progress, content management.

**Duration:** 5 days

### Step 3.1 — User Management APIs

| Method | Endpoint | Roles | Description |
|---|---|---|---|
| GET | `/api/v1/users` | super_admin, org_admin | List users (filtered by scope) |
| GET | `/api/v1/users/:id` | super_admin, org_admin | Get user detail |
| PATCH | `/api/v1/users/:id` | self, super_admin | Update user |
| DELETE | `/api/v1/users/:id` | super_admin | Deactivate user |
| POST | `/api/v1/users/:id/roles` | super_admin, org_admin | Assign role |
| DELETE | `/api/v1/users/:id/roles/:roleId` | super_admin, org_admin | Revoke role |

### Step 3.2 — Organisation & School APIs

| Method | Endpoint | Roles | Description |
|---|---|---|---|
| GET | `/api/v1/organisations` | super_admin | List organisations |
| POST | `/api/v1/organisations` | super_admin | Create organisation |
| GET | `/api/v1/schools` | org_admin, school_admin | List schools in org |
| POST | `/api/v1/schools` | org_admin | Create school |
| GET | `/api/v1/schools/:id/classes` | school_admin, teacher | List classes in school |
| POST | `/api/v1/schools/:id/classes` | school_admin | Create class |
| POST | `/api/v1/classes/:id/enrol` | teacher, school_admin | Enrol child in class |

### Step 3.3 — Child Profile APIs

| Method | Endpoint | Roles | Description |
|---|---|---|---|
| POST | `/api/v1/children` | teacher, parent | Create child profile |
| GET | `/api/v1/children` | teacher (own class), parent (own child) | List children |
| GET | `/api/v1/children/:id` | teacher (own class), parent (own child) | Get child detail |
| PATCH | `/api/v1/children/:id` | teacher, parent (own) | Update child profile |
| DELETE | `/api/v1/children/:id` | school_admin | Deactivate child profile |
| GET | `/api/v1/children/:id/progress` | teacher, parent | Get all chapter progress |
| GET | `/api/v1/children/:id/progress/:chapterId` | teacher, parent | Get specific chapter progress |
| GET | `/api/v1/children/:id/mastery` | teacher, parent | Get concept mastery summary |
| GET | `/api/v1/children/:id/attempts` | teacher | Get step attempt history |
| GET | `/api/v1/children/:id/badges` | teacher, parent | Get earned badges |
| GET | `/api/v1/children/:id/economy` | parent | Get coin balance and transactions |

### Step 3.4 — Content Management APIs

| Method | Endpoint | Roles | Description |
|---|---|---|---|
| GET | `/api/v1/subjects` | all | List subjects |
| GET | `/api/v1/chapters` | all | List chapters (filtered by grade, board) |
| GET | `/api/v1/chapters/:id` | all | Get chapter metadata |
| GET | `/api/v1/chapters/:id/download` | all | Download chapter HTML file |
| POST | `/api/v1/chapters` | org_admin | Upload new chapter |
| PUT | `/api/v1/chapters/:id` | org_admin | Update chapter |
| POST | `/api/v1/chapters/:id/publish` | org_admin | Publish chapter |
| GET | `/api/v1/chapters/:id/nodes` | all | List nodes in chapter |
| GET | `/api/v1/chapters/:id/questions` | teacher | List questions in chapter |
| GET | `/api/v1/lle/words` | all | Search LLE word map |
| POST | `/api/v1/lle/words` | org_admin | Add LLE word |
| PUT | `/api/v1/lle/words/:id` | org_admin | Update LLE word |

### Step 3.5 — Gamification APIs

| Method | Endpoint | Roles | Description |
|---|---|---|---|
| GET | `/api/v1/levels` | all | List levels |
| GET | `/api/v1/badges` | all | List badges |
| GET | `/api/v1/shop/items` | all | List shop items |
| POST | `/api/v1/shop/purchase` | teacher (for child) | Purchase shop item |
| GET | `/api/v1/children/:id/xp` | teacher, parent | Get XP history |
| GET | `/api/v1/children/:id/coins` | parent | Get coin history |

### Step 3.6 — Analytics APIs

| Method | Endpoint | Roles | Description |
|---|---|---|---|
| GET | `/api/v1/analytics/class/:classId` | teacher | Class-level analytics |
| GET | `/api/v1/analytics/child/:childId` | teacher, parent | Individual analytics |
| GET | `/api/v1/analytics/misconceptions` | teacher | Common misconception patterns |
| GET | `/api/v1/analytics/cohort/:schoolId` | school_admin | School-level analytics |
| GET | `/api/v1/analytics/export` | teacher, admin | Export CSV report |

### Step 3.7 — Input Validation

All endpoints use Fastify JSON Schema validation:

```typescript
const createChildSchema = {
  body: {
    type: 'object',
    required: ['displayName', 'grade'],
    properties: {
      displayName: { type: 'string', minLength: 1, maxLength: 50 },
      grade: { type: 'integer', minimum: 1, maximum: 12 },
      board: { type: 'string', enum: ['cbse', 'icse', 'state', 'ib', 'cambridge', 'nios'] },
      avatar: { type: 'string' },
      schoolId: { type: 'string', format: 'uuid' },
      classId: { type: 'string', format: 'uuid' },
    }
  }
};

fastify.post('/api/v1/children', { schema: createChildSchema, preHandler: [fastify.authenticate, requireRole('teacher', 'parent')] }, handler);
```

### Step 3.8 — API Tests

- Integration tests for every endpoint
- RLS verification tests (teacher cannot access other school's data)
- Validation tests (invalid input rejected)
- Pagination tests (cursor-based)

### Phase 3 Deliverables

- [ ] 40+ API endpoints across users, children, content, gamification, analytics
- [ ] JSON Schema validation on all input
- [ ] RBAC enforcement on all endpoints
- [ ] RLS verification in tests
- [ ] OpenAPI/Swagger documentation auto-generated
- [ ] API test suite: 100+ tests
- [ ] Rate limiting (100 req/min per token for sync, 1000/min for read)

**Definition of Done:** All CRUD operations work. A teacher can create a child, view progress, assign chapters. A parent can view their child's progress. All endpoints are documented, validated, and tested. RLS prevents cross-scope data access.

---

## Phase 4 — Chapter Content Pipeline

**Goal:** Build the tooling to author, validate, and publish chapter HTML files programmatically.

**Duration:** 3 days

### Step 4.1 — Chapter Builder Script

Create a Node.js script that assembles a chapter HTML file from structured data:

```typescript
// packages/chapter-builder/src/index.ts
interface ChapterData {
  title: string;
  titleHindi: string;
  subject: subject_code;
  grade: number;
  board: board_code;
  nodes: NodeData[];
  wordMap: Record<string, string>;
  connectives: Record<string, string>;
  levels: LevelData[];
  badges: BadgeData[];
  shopItems: ShopItemData[];
}

export function buildChapter(data: ChapterData): string {
  const css = generateCSS();           // From design tokens
  const html = generateHTML(data);     // Shell, header, dialogs
  const js = generateJS(data);         // NODES, WE, SOLVE, App object
  return `${html}\n<style>${css}</style>\n<script>${js}</script>\n</body></html>`;
}
```

### Step 4.2 — Chapter Validation Harness

Extend the existing test harness (used to validate the 3 current chapters) into a reusable tool:

```typescript
// packages/test-harness/src/validate.ts
export async function validateChapter(htmlFilePath: string): Promise<ValidationResult> {
  // 1. Load HTML file
  // 2. Extract <script> content
  // 3. Parse NODES, WE, SOLVE, WM, CONN, LEVELS, BADGES
  // 4. Verify:
  //    - Every node has at least 1 step
  //    - Every quiz step has exactly 1 correct option
  //    - Every fill-blank has a valid answer
  //    - Every solve has all steps with answers
  //    - Every WE has progressive steps
  //    - No duplicate node indices
  //    - No duplicate step indices within a node
  //    - All WM words referenced in text exist in WM
  //    - File size < 20 MB
  //    - No external URLs
  //    - No eval()
  //    - No fetch()
  return { passed: boolean, errors: string[], warnings: string[] };
}
```

### Step 4.3 — Content Migration Tool

Script to migrate existing chapter data into the database:

```typescript
// packages/chapter-builder/src/migrate.ts
export async function migrateChapterToDB(htmlFilePath: string, chapterId: string) {
  // 1. Parse HTML file
  // 2. Extract NODES, WE, SOLVE, WM, CONN
  // 3. Insert into chapters table
  // 4. Insert each node into chapter_nodes
  // 5. Insert each step into node_steps
  // 6. Insert questions into question_bank
  // 7. Insert WM entries into lle_word_map
  // 8. Insert CONN entries into lle_connectives
}
```

### Step 4.4 — Upload Chapter to MinIO

```typescript
// server/src/services/storage.ts
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';

export async function uploadChapter(chapterId: string, html: Buffer) {
  const key = `chapters/${chapterId}/index.html`;
  await s3.send(new PutObjectCommand({
    Bucket: 'aasha',
    Key: key,
    Body: html,
    ContentType: 'text/html',
  }));
  
  const checksum = crypto.createHash('sha256').update(html).digest('hex');
  await pool.query('UPDATE chapters SET file_url = $1, file_checksum = $2 WHERE id = $3', [key, checksum, chapterId]);
}
```

### Phase 4 Deliverables

- [ ] Chapter builder script (assembles HTML from structured JSON data)
- [ ] Chapter validation harness (checks structural integrity, answer correctness, file size, no external deps)
- [ ] Content migration tool (parses existing 3 chapters into database)
- [ ] MinIO upload service for chapter files
- [ ] All 3 existing chapters migrated to database
- [ ] Validation passes on all 3 chapters

**Definition of Done:** A content author can write a chapter definition in JSON, run `npm run build:chapter`, get a validated HTML file, and upload it to the server. The existing 3 chapters are in the database and downloadable via API.

---

## Phase 5 — Sync Layer (Offline → Server)

**Goal:** Build the synchronization system that syncs localStorage state from chapter HTML files to the PostgreSQL database.

**Duration:** 4 days

### Step 5.1 — Sync API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/v1/sync/push` | Push localStorage state to server |
| GET | `/api/v1/sync/pull?childId=X&chapterId=Y` | Pull latest state from server |
| POST | `/api/v1/sync/merge` | Resolve conflicts |

### Step 5.2 — Push Handler

```typescript
// server/src/routes/sync.ts
fastify.post('/api/v1/sync/push', { preHandler: [fastify.authenticate] }, async (request, reply) => {
  const { childId, chapterId, clientState, clientVersion } = request.body;
  
  // 1. Verify caller has access to this child
  // 2. Get server version of chapter_progress
  const server = await pool.query('SELECT * FROM chapter_progress WHERE child_id = $1 AND chapter_id = $2', [childId, chapterId]);
  
  if (server.rows.length > 0 && server.rows[0].sync_version > clientVersion) {
    // Conflict detected
    return reply.code(409).send({ status: 'conflict', serverState: server.rows[0] });
  }
  
  // 3. Upsert chapter_progress
  await pool.query(`
    INSERT INTO chapter_progress (child_id, chapter_id, current_node_index, current_step_index, 
      xp_earned, coins_earned, correct_count, incorrect_count, sync_version, last_synced_at)
    VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, NOW())
    ON CONFLICT (child_id, chapter_id) DO UPDATE SET
      current_node_index = EXCLUDED.current_node_index,
      current_step_index = EXCLUDED.current_step_index,
      xp_earned = EXCLUDED.xp_earned,
      coins_earned = EXCLUDED.coins_earned,
      correct_count = EXCLUDED.correct_count,
      incorrect_count = EXCLUDED.incorrect_count,
      sync_version = chapter_progress.sync_version + 1,
      last_synced_at = NOW()
  `, [childId, chapterId, ...]);
  
  // 4. Insert step_attempts (batch)
  // 5. Update concept_mastery (triggers handle this)
  // 6. Update XP and coin ledgers
  // 7. Return new sync_version
  return { status: 'success', syncVersion: clientVersion + 1 };
});
```

### Step 5.3 — Conflict Resolution Strategy

| Field | Resolution | Rationale |
|---|---|---|
| `current_node_index` | MAX(client, server) | Never regress progress |
| `current_step_index` | MAX(client, server) | Never regress progress |
| `xp_earned` | MAX(client, server) | Never lose XP |
| `coins_earned` | MAX(client, server) | Never lose coins |
| `correct_count` | MAX(client, server) | Never lose correct count |
| `incorrect_count` | client wins (if higher) | Server may not have latest |
| `hints_used` | MAX(client, server) | Track all hint usage |
| `is_completed` | client OR server | If either says complete, it's complete |
| `completion_percentage` | MAX(client, server) | Never regress |

### Step 5.4 — Sync Queue Processor

Background worker that processes the `sync_queue` table:

```typescript
// server/src/services/sync-processor.ts
async function processSyncQueue() {
  const pending = await pool.query('SELECT * FROM sync_queue WHERE status = $1 ORDER BY created_at LIMIT 50', ['pending']);
  
  for (const item of pending.rows) {
    try {
      await pool.query('UPDATE sync_queue SET status = $1 WHERE id = $2', ['processing', item.id]);
      
      // Apply the operation
      switch (item.table_name) {
        case 'chapter_progress': await syncChapterProgress(item); break;
        case 'step_attempts': await syncStepAttempt(item); break;
        case 'game_scores': await syncGameScore(item); break;
        // ... etc
      }
      
      await pool.query('UPDATE sync_queue SET status = $1, completed_at = NOW() WHERE id = $2', ['completed', item.id]);
    } catch (err) {
      await pool.query('UPDATE sync_queue SET status = $1, error_message = $2, retry_count = retry_count + 1 WHERE id = $3', ['failed', err.message, item.id]);
    }
  }
}

// Run every 10 seconds
setInterval(processSyncQueue, 10000);
```

### Step 5.5 — Sync Client (in Chapter HTML)

Add a sync module to chapter HTML files that detects when the device is online and pushes state:

```javascript
// In chapter HTML <script>
var Sync = {
  isOnline: navigator.onLine,
  lastSync: null,
  
  init: function() {
    window.addEventListener('online', this.attemptSync);
    setInterval(this.attemptSync, 300000); // Every 5 minutes
  },
  
  attemptSync: function() {
    if (!navigator.onLine) return;
    var state = App.getState(); // Get localStorage state
    fetch('/api/v1/sync/push', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', 'Authorization': 'Bearer ' + Sync.getToken() },
      body: JSON.stringify({ childId: Sync.childId, chapterId: Sync.chapterId, clientState: state, clientVersion: state.syncVersion || 0 })
    })
    .then(r => r.json())
    .then(data => { Sync.lastSync = new Date(); })
    .catch(err => console.error('Sync failed:', err));
  },
  
  getToken: function() { return localStorage.getItem('aasha_sync_token'); }
};
```

### Phase 5 Deliverables

- [ ] Push, pull, merge API endpoints
- [ ] Conflict resolution with documented strategy
- [ ] Background sync queue processor (runs every 10s)
- [ ] Sync client module for chapter HTML files (detects online state, pushes every 5 min)
- [ ] Optimistic concurrency via `sync_version`
- [ ] Offline queue (localStorage) for submissions when offline
- [ ] Sync test suite: push/pull/conflict scenarios
- [ ] Chapter HTML files updated with sync module (graceful degradation — works without sync)

**Definition of Done:** A child can learn offline for hours. When the device comes online (and a teacher/parent has linked the profile), the localStorage state syncs to the server. Conflicts are resolved automatically (MAX strategy). The sync queue processes pending operations within 10 seconds.

---

## Phase 6 — Teacher Dashboard

**Goal:** Build the React web application for teachers to view class progress, identify struggling students, and review Seva activities.

**Duration:** 6 days

### Step 6.1 — Dashboard Project Setup

```bash
cd dashboard
npm create vite@latest . -- --template react-ts
npm install tailwindcss @tailwindcss/typography
npm install zustand react-router-dom
npm install recharts date-fns
npm install axios
npm install @tanstack/react-query
```

### Step 6.2 — Auth Flow (Login → OTP → Dashboard)

Build the login screen (S37 from App Flow Document):
- Phone + OTP or email/password
- OTP verification screen
- Token storage in httpOnly cookie (via API)
- Protected route wrapper

### Step 6.3 — Dashboard Layout

```
┌─────────────────────────────────────────────────────┐
│  TOP BAR: Aasha Logo | Teacher Name | [Logout]       │
├──────────┬──────────────────────────────────────────┤
│          │                                          │
│ SIDEBAR  │  MAIN CONTENT (router outlet)            │
│          │                                          │
│ - Home   │                                          │
│ - Students│                                         │
│ - Seva   │                                          │
│ - Reports│                                          │
│ - Settings│                                         │
│          │                                          │
└──────────┴──────────────────────────────────────────┘
```

- Sidebar: 240px fixed, collapsible on mobile
- Main: flexible, card-based layout
- Responsive: sidebar becomes hamburger menu below 768px

### Step 6.4 — Class Overview Page (S38)

- Class stats cards: students count, active this week, avg progress, avg mastery
- Weekly engagement bar chart (Recharts)
- Students needing help list (RED/YELLOW indicators)
- Common misconceptions ranked list
- Seva verification queue count

### Step 6.5 — Student Detail Page (S39)

- Child stats: XP, level, coins, streak, sessions
- Chapter progress bars
- Concept mastery list (per concept: GREEN/YELLOW/RED)
- Misconception patterns
- Assessment history table

### Step 6.6 — Analytics Page

- Misconception heatmap (which concepts are most missed)
- Engagement timeline (when are children active)
- Class comparison (if teacher has multiple classes)
- CSV export button

### Step 6.7 — Build & Test

- Component tests with React Testing Library
- E2E test: login → view class → click student → see mastery
- Responsive test: works on 768px, 1024px, 1280px

### Phase 6 Deliverables

- [ ] React dashboard with Tailwind CSS
- [ ] Login + OTP flow
- [ ] Class overview page with charts
- [ ] Student detail page with mastery breakdown
- [ ] Analytics page with misconception analysis
- [ ] CSV export
- [ ] Responsive layout (mobile, tablet, desktop)
- [ ] Protected routes (auth required)
- [ ] Dashboard test suite

**Definition of Done:** A teacher can log in, see their class overview, click into any student's detail, view concept mastery, identify struggling students, see common misconceptions, and export a CSV report. The dashboard is responsive and works on mobile.

---

## Phase 7 — Parent Dashboard

**Goal:** Build a simplified, mobile-first dashboard for parents to understand their child's learning.

**Duration:** 3 days

### Step 7.1 — Parent Login Flow

- Same auth system as teacher (phone OTP or email/password)
- Parent sees only their linked children

### Step 7.2 — Parent Dashboard Page (S42)

- Child's weekly summary: concepts learned, mastered, time spent, streak
- Plain-language assessment: "Rahul is doing well on X but finding Y challenging"
- Understanding indicators (GREEN/YELLOW/RED per concept)
- Seva activities list
- Suggestion card: "Try asking Rahul about shapes around the house"

### Step 7.3 — Notifications

- Badge earned, level up, Seva verified, saving goal reached
- In-app notification list
- Read/unread state

### Phase 7 Deliverables

- [ ] Parent login flow
- [ ] Parent dashboard with child summary
- [ ] Plain-language progress descriptions
- [ ] Understanding indicators (not marks)
- [ | Seva activities view
- [ ] Notification list
- [ ] Mobile-first responsive design
- [ ] Parent dashboard test suite

**Definition of Done:** A parent can log in, see their child's learning progress in plain language (not marks), understand where the child excels and where they need help, view Seva activities, and receive notifications.

---

## Phase 8 — Gamification Hardening

**Goal:** Port the gamification features (currently only in Quadrilaterals) to all chapters, add daily challenges, and ensure consistency.

**Duration:** 4 days

### Step 8.1 — Port Gamification to Fractions and Comparing Quantities

Extract the gamification code from the Quadrilaterals chapter and apply to:
- Fractions chapter (add timed quiz, rapid fire, memory match, gem shop, back button, restart)
- Comparing Quantities chapter (same features + back button + restart)

### Step 8.2 — Daily Challenge System

```javascript
// In chapter HTML
var DailyChallenge = {
  getSeed: function() {
    var d = new Date();
    return parseInt(d.getFullYear() + '' + (d.getMonth()+1) + '' + d.getDate());
  },
  
  getQuestions: function() {
    var seed = this.getSeed();
    // Deterministically select 5 questions from question bank based on seed
    // Same questions for all children on the same date
    return selectQuestionsBySeed(seed, 5);
  },
  
  render: function() {
    // Show 5 questions, untimed, award XP + coins
    // Track streak (consecutive days completed)
  }
};
```

### Step 8.3 — Leaderboard (Opt-In)

- Class leaderboard (weekly, monthly)
- School leaderboard (monthly)
- Children must explicitly opt in (no auto-enrollment)
- No global leaderboards (avoid unhealthy competition)
- Shows: rank, XP, level, badge count (no scores or wrong-answer counts)

### Step 8.4 — Gamification Consistency Check

- Verify all 3 chapters have: timed quiz, rapid fire, memory match, gem shop, back button, restart, hint system, streak tracking, level-up overlay, badge overlay, progress ring
- Run chapter validation harness on all 3 chapters

### Phase 8 Deliverables

- [ ] Fractions chapter updated with full gamification
- [ ] Comparing Quantities chapter updated with full gamification
- [ ] Daily challenge system (date-seeded, 5 questions, streak tracking)
- [ ] Opt-in leaderboard (class and school scope)
- [ ] All 3 chapters pass validation harness
- [ ] Gamification test suite

**Definition of Done:** All 3 chapters have identical gamification features. A child can play daily challenges. Leaderboards are opt-in and scoped to class/school. The validation harness passes on all chapters.

---

## Phase 9 — Seva Activity System

**Goal:** Build the Seva activity submission, verification, and reward system.

**Duration:** 4 days

### Step 9.1 — Seva APIs (Server)

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/v1/seva/types` | List available Seva activities |
| POST | `/api/v1/seva/submit` | Submit activity with photo + description |
| GET | `/api/v1/seva/my` | List child's Seva activities |
| GET | `/api/v1/seva/pending` | List pending activities (teacher) |
| POST | `/api/v1/seva/:id/verify` | Verify activity (teacher) |
| POST | `/api/v1/seva/:id/reject` | Reject activity with reason (teacher) |

### Step 9.2 — Photo Upload

```typescript
// server/src/services/storage.ts
export async function uploadSevaPhoto(activityId: string, photo: Buffer, mimeType: string) {
  // 1. Validate: max 5MB, image/jpeg or image/png
  // 2. Generate thumbnail (200x200)
  // 3. Upload both to MinIO
  // 4. Update seva_activities with URLs
}
```

### Step 9.3 — Seva Hub UI (in Chapter HTML or Dashboard)

Build the Seva Hub screen (S31), Seva Submit screen (S32), and Seva Status screen (S33) as described in the App Flow Document.

### Step 9.4 — Verification Flow (Teacher Dashboard)

Build the Seva Verification page (S40) in the teacher dashboard:
- List of pending submissions
- Photo viewer
- Verify/Reject buttons with reason input
- Auto-award coins on verify

### Step 9.5 — Offline Submission Queue

```javascript
// In chapter HTML
var SevaSync = {
  submitOffline: function(activity) {
    var queue = JSON.parse(localStorage.getItem('aasha_seva_queue') || '[]');
    queue.push(activity);
    localStorage.setItem('aasha_seva_queue', JSON.stringify(queue));
  },
  
  flushQueue: async function() {
    if (!navigator.onLine) return;
    var queue = JSON.parse(localStorage.getItem('aasha_seva_queue') || '[]');
    for (var item of queue) {
      try {
        await fetch('/api/v1/seva/submit', { /* ... */ });
        // Remove from queue on success
      } catch (e) {
        break; // Stop on first failure
      }
    }
  }
};
```

### Phase 9 Deliverables

- [ ] 6 Seva API endpoints
- [ ] Photo upload to MinIO with thumbnail generation
- [ ] Seva Hub, Submit, and Status screens
- [ | Teacher verification page in dashboard
- [ ] Offline submission queue with auto-flush
- [ ] Coin reward on verification (via coin_ledger trigger)
- [ ] Seva test suite

**Definition of Done:** A child can submit a Seva activity with a photo from the chapter app. The photo uploads to MinIO. A teacher sees the submission in their dashboard, verifies or rejects it. On verification, the child earns coins (recorded in coin_ledger). Offline submissions queue and auto-sync.

---

## Phase 10 — Aasha Economy (Wallet, Store, Saving)

**Goal:** Build the full economy system: wallet, saving goals, and the Aasha Store with real-world impact items.

**Duration:** 5 days

### Step 10.1 — Economy APIs

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/v1/economy/:childId` | Get wallet balance and transaction history |
| POST | `/api/v1/economy/deposit` | Move coins from spendable to savings |
| POST | `/api/v1/economy/withdraw` | Move coins from savings to spendable |
| POST | `/api/v1/saving-goals` | Create saving goal |
| GET | `/api/v1/saving-goals/:childId` | List saving goals |
| GET | `/api/v1/store/items` | List store items |
| POST | `/api/v1/store/redeem` | Redeem coins for store item |
| GET | `/api/v1/store/redemptions/:childId` | List redemption history |

### Step 10.2 — Wallet UI (S34)

- Large coin balance display
- Sub-stats: earned, spent, saved, investing
- Transaction history list
- Links to saving goals and store

### Step 10.3 — Saving Goal UI (S36)

- Goal selector (Plant-a-Tree, Book for a Child, Water Filter, Custom)
- Progress bar (current/target)
- Deposit button (moves coins from spendable to savings)
- Goal reached celebration

### Step 10.4 — Aasha Store UI (S35)

- Educational rewards (chapter unlock, theme pack)
- Real-world impact items (plant tree, book, water filter)
- Redemption flow with confirmation
- Fulfillment tracking (pending → fulfilled)

### Step 10.5 — Store Fulfillment (Admin)

- Admin dashboard page to view redemptions
- Mark as fulfilled (with notes and evidence URL)
- Notify child when fulfilled

### Phase 10 Deliverables

- [ ] 8 economy API endpoints
- [ ] Wallet screen with transaction history
- [ ] Saving goal creation and progress tracking
- [ ] Aasha Store with educational and real-world items
- [ ] Store redemption and fulfillment flow
- [ ] Coin ledger integrity (every transaction has balance_after)
- [ ] Economy test suite

**Definition of Done:** A child can view their wallet, set a saving goal, deposit coins into savings, redeem coins for store items (educational or real-world), and track fulfillment. Every coin transaction is logged in the immutable ledger with balance_after maintained by triggers.

---

## Phase 11 — Chapter Enhancement (Rich Media)

**Goal:** Enhance the chapter HTML files with richer content within the 20 MB budget — animated SVG diagrams, enhanced Canvas interactions, richer audio feedback, adaptive difficulty.

**Duration:** 5 days

### Step 11.1 — Animated SVG Diagrams

Replace static Canvas visuals with animated SVG for concepts that benefit from motion:
- Polygon angle sum discovery: animate the triangle-fitting-inside demonstration
- Fraction bar: animate slicing and equivalence
- Ratio bars: animate proportional scaling
- Profit/loss bars: animate growth/shrinkage

### Step 11.2 — Enhanced Canvas Interactions

- Multi-touch support for drag interactions (pinch to zoom on shapes)
- Smooth animations using `requestAnimationFrame` instead of instant redraws
- Visual guides (dashed lines showing where to drag)
- Haptic feedback via `navigator.vibrate()` on supported devices

### Step 11.3 — Richer Audio

- Multi-tone melodies for celebrations (not just 2-note)
- Different sound profiles for different question types
- Background ambient tone option (very low volume, toggleable)
- Sound for memory match card flip (different from correct/wrong)

### Step 11.4 — Adaptive Difficulty (Basic)

```javascript
// In chapter HTML
var AdaptiveDifficulty = {
  getDifficulty: function(childId, chapterId) {
    // Based on concept_mastery data:
    // If mastery > 80%: serve 'hard' or 'hots' questions
    // If mastery 40-80%: serve 'medium' questions
    // If mastery < 40%: serve 'easy' questions
    var mastery = App.getMastery();
    if (mastery > 80) return 'hard';
    if (mastery > 40) return 'medium';
    return 'easy';
  },
  
  selectQuestion: function(availableQuestions) {
    var difficulty = this.getDifficulty();
    var filtered = availableQuestions.filter(q => q.difficulty === difficulty);
    return filtered[Math.floor(Math.random() * filtered.length)];
  }
};
```

### Step 11.5 — Validate File Sizes

After enhancements, verify each chapter file is still under 20 MB:
- Run `wc -c` on each HTML file
- Run validation harness on each
- Test load time on a low-end Android emulator

### Phase 11 Deliverables

- [ ] Animated SVG diagrams for 5+ key concepts across 3 chapters
- [ ] Enhanced Canvas interactions with smooth animations and visual guides
- [ ] Richer audio feedback (multi-tone melodies, per-question-type sounds)
- [ ] Basic adaptive difficulty (serves questions based on mastery level)
- [ ] All chapters under 20 MB
- [ ] Validation harness passes on all chapters
- [ ] Performance test: load time < 2 seconds on low-end device

**Definition of Done:** Chapters are visually richer with animated diagrams, smoother interactions, and better audio. Adaptive difficulty serves easier or harder questions based on the child's mastery. All files remain under 20 MB and load in under 2 seconds.

---

## Phase 12 — Testing & QA

**Goal:** Comprehensive testing across all layers: chapter files, API, dashboard, sync, and end-to-end.

**Duration:** 5 days

### Step 12.1 — Chapter Test Suite (Automated)

```javascript
// packages/test-harness/src/index.ts
describe('Chapter Validation', () => {
  test('All 3 chapters pass structural validation', async () => {
    for (const chapter of ['quadrilaterals', 'fractions', 'comparing-quantities']) {
      const result = await validateChapter(`chapters/${chapter}/index.html`);
      expect(result.passed).toBe(true);
    }
  });
  
  test('Every quiz has exactly 1 correct answer', async () => { /* ... */ });
  test('Every fill-blank has a valid answer', async () => { /* ... */ });
  test('Every solve step has an answer', async () => { /* ... */ });
  test('No external URLs in any chapter', async () => { /* ... */ });
  test('No eval() in any chapter', async () => { /* ... */ });
  test('File size < 20 MB', async () => { /* ... */ });
  test('All LLE words referenced exist in WM', async () => { /* ... */ });
  test('All step types have render functions', async () => { /* ... */ });
});
```

### Step 12.2 — API Test Suite

```javascript
describe('Auth API', () => {
  test('Register with email/password', async () => { /* ... */ });
  test('Login with email/password', async () => { /* ... */ });
  test('Send OTP to phone', async () => { /* ... */ });
  test('Verify OTP', async () => { /* ... */ });
  test('Refresh token rotation', async () => { /* ... */ });
  test('Logout revokes session', async () => { /* ... */ });
  test('Rate limit: 2nd OTP within 60s rejected', async () => { /* ... */ });
});

describe('Sync API', () => {
  test('Push state to server', async () => { /* ... */ });
  test('Pull state from server', async () => { /* ... */ });
  test('Conflict detected when versions mismatch', async () => { /* ... */ });
  test('Conflict resolved with MAX strategy', async () => { /* ... */ });
  test('Offline queue flushes on reconnect', async () => { /* ... */ });
});

describe('RLS', () => {
  test('Teacher cannot read another teacher\'s class data', async () => { /* ... */ });
  test('Parent can only see own child', async () => { /* ... */ });
  test('School admin cannot see other school data', async () => { /* ... */ });
});
```

### Step 12.3 — Dashboard E2E Tests (Puppeteer)

```javascript
describe('Teacher Dashboard E2E', () => {
  test('Login → view class → click student → see mastery', async () => {
    await page.goto('http://localhost:5173/login');
    await page.type('input[name=email]', 'teacher@test.com');
    await page.type('input[name=password]', 'password');
    await page.click('button[type=submit]');
    await page.waitForSelector('.class-overview');
    await page.click('.student-card:first-child');
    await page.waitForSelector('.student-detail');
    expect(await page.$('.mastery-indicator')).toBeTruthy();
  });
});
```

### Step 12.4 — Chapter E2E Tests (Puppeteer)

```javascript
describe('Chapter Walkthrough', () => {
  test('Complete Quadrilaterals chapter without errors', async () => {
    await page.goto('file:///chapters/quadrilaterals/index.html');
    // Click through every step
    // Verify no console errors
    // Verify every answer is accepted correctly
    // Verify completion screen appears
  });
});
```

### Step 12.5 — Load Testing (k6)

```javascript
import http from 'k6/http';

export let options = {
  stages: [
    { duration: '30s', target: 50 },   // Ramp up to 50 users
    { duration: '1m', target: 50 },    // Hold at 50
    { duration: '30s', target: 100 },  // Ramp to 100
    { duration: '2m', target: 100 },   // Hold at 100
    { duration: '30s', target: 0 },    // Ramp down
  ],
};

export default function () {
  http.get('http://localhost:3000/api/v1/chapters');
  http.post('http://localhost:3000/api/v1/sync/push', { /* ... */ });
}
```

### Step 12.6 — Security Testing

- Run `npm audit` on all packages
- Run Trivy container scanner on Docker images
- Test SQL injection (all queries use parameterized inputs)
- Test XSS (DOMPurify on all user input rendered in dashboard)
- Test CSRF (SameSite cookies)
- Test rate limiting (auth endpoints)

### Step 12.7 — Manual QA Checklist

- [ ] Chapter files open on Chrome (Android 10+)
- [ ] Chapter files open on Firefox (Android)
- [ ] Chapter files work fully offline (airplane mode)
- [ ] Profile creation and switching works
- [ ] Progress saves and resumes correctly
- [ ] All gamification features work (timer, rapid fire, memory match, shop)
- [ ] Sync works when online (push and pull)
- [ ] Teacher dashboard loads and displays data
- [ ] Parent dashboard loads and displays data
- [ ] Seva submission and verification works
- [ ] Economy transactions are accurate
- [ ] All animations are smooth (no jank)
- [ ] All sounds play correctly
- [ ] No console errors in any chapter

### Phase 12 Deliverables

- [ ] Chapter test suite: 50+ automated tests
- [ ] API test suite: 100+ integration tests
- [ ] Dashboard E2E tests: 20+ Puppeteer tests
- [ ] Chapter E2E tests: 3 full walkthroughs
- [ ] Load test: 100 concurrent users, p95 < 200ms
- [ ] Security scan: no critical vulnerabilities
- [ ] Manual QA checklist: all items pass
- [ ] Test coverage > 80% on server code

**Definition of Done:** All automated tests pass. Load testing shows the server handles 100 concurrent users with p95 < 200ms. Security scan finds no critical vulnerabilities. Manual QA on Android devices passes all items.

---

## Phase 13 — Deployment & CI/CD

**Goal:** Set up the production deployment pipeline with Docker Compose, Caddy, and GitHub Actions.

**Duration:** 3 days

### Step 13.1 — Production Docker Compose

```yaml
# infra/docker-compose.prod.yml
services:
  caddy:
    image: caddy:2
    ports: ["80:80", "443:443"]
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
    depends_on: [api, dashboard]

  api:
    build: ./server
    environment:
      - DATABASE_URL=postgres://aasha:${DB_PASS}@db:5432/aasha
      - REDIS_URL=redis://cache:6379
      - JWT_SECRET=${JWT_SECRET}
      - NODE_ENV=production
    depends_on: [db, cache]
    restart: unless-stopped

  dashboard:
    build: ./dashboard
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_DB=aasha
      - POSTGRES_USER=aasha
      - POSTGRES_PASSWORD=${DB_PASS}
    volumes: [pg_data:/var/lib/postgresql/data]
    restart: unless-stopped

  cache:
    image: redis:7-alpine
    volumes: [redis_data:/data]
    restart: unless-stopped

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      - MINIO_ROOT_USER=${MINIO_USER}
      - MINIO_ROOT_PASSWORD=${MINIO_PASS}
    volumes: [minio_data:/data]
    restart: unless-stopped

  uptime-kuma:
    image: louislam/uptime-kuma:1
    volumes: [kuma_data:/app/data]
    restart: unless-stopped

volumes:
  caddy_data:
  pg_data:
  redis_data:
  minio_data:
  kuma_data:
```

### Step 13.2 — Caddyfile

```caddyfile
aasha.example.org {
    handle /api/* {
        reverse_proxy api:3000
        rate_limit { zone sync 100r/m }
    }
    handle /chapters/* {
        reverse_proxy minio:9000
        header Cache-Control "public, max-age=31536000, immutable"
    }
    handle {
        reverse_proxy dashboard:80
    }
}
```

### Step 13.3 — GitHub Actions CI/CD

```yaml
# .github/workflows/deploy.yml
name: Test & Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres: ...
      redis: ...
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run db:migrate
      - run: npm test
      - run: npm run test:e2e
      - run: npm audit --audit-level=high

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to VPS
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /opt/aasha
            git pull origin main
            docker compose -f infra/docker-compose.prod.yml build
            docker compose -f infra/docker-compose.prod.yml up -d
            docker image prune -f
```

### Step 13.4 — Provision VPS

```bash
# On Hetzner VPS (Debian 12)
apt update && apt install -y docker.io docker-compose git
git clone <repo> /opt/aasha
cd /opt/aasha
cp infra/.env.example infra/.env
# Edit .env with production secrets
docker compose -f infra/docker-compose.prod.yml up -d
```

### Step 13.5 — Setup Monitoring

- Uptime Kuma: Monitor API health endpoint, dashboard, and MinIO
- Loki + Grafana: Centralized logging from all containers
- Alert via Telegram/webhook on downtime

### Step 13.6 — DNS & CDN

- Point `aasha.example.org` to VPS IP
- Enable Cloudflare CDN (free tier)
- Set cache rules: chapter files = 1 year immutable, API = no cache

### Phase 13 Deliverables

- [ ] Production `docker-compose.prod.yml` with 7 services
- [ ] Caddyfile with auto-HTTPS and routing
- [ ] GitHub Actions workflow: test on PR, deploy on merge to main
- [ ] VPS provisioned with Docker, repo cloned, services running
- [ ] Uptime Kuma monitoring all endpoints
- [ ] Loki + Grafana log aggregation
- [ ] Cloudflare CDN configured
- [ ] HTTPS certificate auto-provisioned by Caddy
- [ ] Zero-downtime deployment (rolling restart)

**Definition of Done:** Pushing to `main` triggers automated tests. If tests pass, the code deploys to the VPS via SSH. All services (API, dashboard, DB, Redis, MinIO, Caddy, monitoring) start and are accessible via HTTPS. Uptime monitoring is active.

---

## Phase 14 — Final Polish & Launch

**Goal:** Final UX polish, documentation, and launch preparation.

**Duration:** 3 days

### Step 14.1 — UX Polish

- Review all screens against the UI/UX Design Brief
- Verify all color tokens, typography, spacing match the design system
- Test all empty states, error states, and success states
- Test all animations on a low-end Android device
- Verify `prefers-reduced-motion` disables animations
- Verify accessibility (contrast, tap targets, ARIA labels)

### Step 14.2 — Documentation

- API documentation (OpenAPI/Swagger)
- Deployment guide (step-by-step VPS setup)
- Content authoring guide (how to build a new chapter)
- Teacher onboarding guide
- Parent onboarding guide
- README with architecture overview

### Step 14.3 — Seed Production Data

- Create the Annanth Aasha Foundation organisation
- Create pilot schools
- Create teacher accounts
- Upload all 3 chapters to MinIO
- Verify chapters are downloadable via API

### Step 14.4 — Pilot Launch

- Onboard 1-2 pilot schools
- Train 2-3 teachers
- Distribute chapter files via WhatsApp/USB
- Monitor for 1 week
- Collect feedback

### Step 14.5 — Post-Launch Monitoring

- Daily check: Uptime Kuma alerts
- Weekly check: error logs in Loki
- Weekly check: sync queue backlog
- Bi-weekly: teacher feedback session
- Monthly: analytics review

### Phase 14 Deliverables

- [ ] All screens match UI/UX Design Brief
- [ ] All states (empty, error, success) tested
- [ ] Accessibility audit passes
- [ ] Complete documentation (API, deployment, authoring, onboarding)
- [ ] Production data seeded (organisation, schools, teachers, chapters)
- [ ] Pilot schools onboarded
- [ ] Monitoring dashboard active
- [ ] Feedback collection mechanism in place

**Definition of Done:** The platform is live, pilot schools are using it, teachers have been trained, chapters are distributed, monitoring is active, and feedback is being collected. The product is in the hands of real children.

---

## 18. Dependency Graph

```
Phase 0 (Setup)
  │
  ├─> Phase 1 (Database)
  │     │
  │     ├─> Phase 2 (Auth)
  │     │     │
  │     │     └─> Phase 3 (Core API)
  │     │           │
  │     │           ├─> Phase 4 (Content Pipeline) ──> Phase 5 (Sync)
  │     │           │                                    │
  │     │           ├─> Phase 6 (Teacher Dashboard) <────┘
  │     │           │         │
  │     │           │         └─> Phase 7 (Parent Dashboard)
  │     │           │
  │     │           ├─> Phase 8 (Gamification Hardening)
  │     │           │
  │     │           ├─> Phase 9 (Seva) ──> Phase 10 (Economy)
  │     │           │
  │     │           └─> Phase 11 (Chapter Enhancement)
  │     │
  │     └─> (all above) ─> Phase 12 (Testing)
  │                              │
  │                              └─> Phase 13 (Deployment)
  │                                      │
  │                                      └─> Phase 14 (Launch)
  │
  └─> (independent) Chapter files remain functional throughout
```

**Critical path:** Phase 0 → 1 → 2 → 3 → 5 → 12 → 13 → 14

**Parallel tracks (can be done concurrently after Phase 3):**
- Phase 4 (Content Pipeline) + Phase 6 (Teacher Dashboard) + Phase 8 (Gamification)
- Phase 9 (Seva) + Phase 10 (Economy) + Phase 11 (Chapter Enhancement)

---

## 19. Effort Estimates

| Phase | Duration | Effort (person-days) | Team |
|---|---|---|---|
| Phase 0 — Setup | 2 days | 2 | 1 full-stack |
| Phase 1 — Database | 3 days | 3 | 1 backend |
| Phase 2 — Auth | 4 days | 4 | 1 backend |
| Phase 3 — Core API | 5 days | 5 | 1 backend |
| Phase 4 — Content Pipeline | 3 days | 3 | 1 full-stack |
| Phase 5 — Sync Layer | 4 days | 4 | 1 backend |
| Phase 6 — Teacher Dashboard | 6 days | 6 | 1 frontend |
| Phase 7 — Parent Dashboard | 3 days | 3 | 1 frontend |
| Phase 8 — Gamification | 4 days | 4 | 1 full-stack |
| Phase 9 — Seva | 4 days | 4 | 1 full-stack |
| Phase 10 — Economy | 5 days | 5 | 1 full-stack |
| Phase 11 — Chapter Enhancement | 5 days | 5 | 1 frontend |
| Phase 12 — Testing | 5 days | 5 | 1 QA |
| Phase 13 — Deployment | 3 days | 3 | 1 DevOps |
| Phase 14 — Launch | 3 days | 3 | all |
| **Total** | **~59 calendar days** | **59 person-days** | |

**With 2 engineers (1 backend, 1 frontend):** ~35 calendar days  
**With 3 engineers (1 backend, 1 frontend, 1 full-stack):** ~25 calendar days  
**With 1 engineer (full-stack):** ~59 calendar days (sequential)

---

## 20. Risk Register

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| MSG91 SMS delivery delays | Medium | Medium | Fallback to email OTP; retry logic |
| Low-end Android crashes on enhanced Canvas | Medium | High | Test on real devices; degrade gracefully |
| Sync conflicts on shared devices | Medium | Medium | MAX strategy + manual resolution UI |
| localStorage quota exceeded | Low | Medium | Check `navigator.storage.estimate()`; warn before limit |
| Database migration failures in production | Low | High | Test on staging copy first; backup before migration |
| Teacher adoption resistance | Medium | High | Onboarding training; simple UI; WhatsApp support group |
| Internet connectivity at pilot schools | High | Medium | Chapter files work fully offline; sync is optional |
| Photo upload size issues (Seva) | Medium | Low | Client-side compression before upload; max 5MB |
| JWT key compromise | Low | Critical | Key rotation plan; Redis revocation; short TTL |
| Child data privacy concerns | Medium | High | PII encryption; no child PII in logs; DPDP compliance |

---

**End of Implementation Plan**

This plan takes the Aasha platform from its current state (3 offline chapter HTML files + comprehensive design documents) to a full production deployment with backend, dashboards, sync, Seva, and economy. Each phase has clear deliverables, a definition of done, and unblocks the next phase. The critical path is 59 person-days, reducible to ~25 calendar days with 3 engineers.
