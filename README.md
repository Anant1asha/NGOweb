# Aasha Safe Learning Platform

Aasha is a child-centred learning platform that transforms school-book content into interactive, gamified, bilingual offline learning chapters (Class 1-10, any board).

## Monorepo Structure

- `server/`: Fastify (Node.js) API for authentication, profile syncing, and the AI chapter transformation pipeline.
- `dashboard/`: React + Vite + Tailwind CSS dashboard for teachers (upload, progress tracking, and review) and admins.
- `chapters/`: Workspace for offline HTML chapters.
- `packages/`:
  - `@aasha/shared-types`: Type definitions shared between server, dashboard, and scripts.
  - `@aasha/chapter-builder`: CLI and library to compile chapters into standalone HTML files.
  - `@aasha/test-harness`: Reusable validation suite to structurally verify offline chapters.
- `infra/`: Local development infrastructure (Docker compose setup for Postgres, Redis, and MinIO).

## Development Setup

### Prerequisites

- **Node.js** (v20+ recommended)
- **Docker** and **Docker Compose**
- **Google Gemini API Key** (for content transformation)

### Installation & Run

1. Clone the repository and install dependencies:
   ```bash
   npm install
   ```

2. Copy the environment configuration and fill in variables:
   ```bash
   cp infra/.env.example infra/.env
   ```

3. Spin up local databases and storage:
   ```bash
   cd infra
   docker compose up -d
   ```

4. Run local development servers:
   - Server (Fastify API):
     ```bash
     npm run dev:server
     ```
   - Dashboard (React UI):
     ```bash
     npm run dev:dashboard
     ```
