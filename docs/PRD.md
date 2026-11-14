## 7. Git Workflow & Branching Strategy

- **Repository:** `[https://github.com/qjoy-2462006/thi-kha.git](https://github.com/qjoy-2462006/thi-kha.git)`
- **Branch Structure:**
  - `main` — Stable production-ready branch.
  - `feat/docker-db` — Docker Compose, PostgreSQL 16 setup, and SQLAlchemy models.
  - `feat/backend-engine` — Python 3.14.6 FastAPI, Multi-provider Auto-Routing Failover, and PDF Layout Engine.
  - `feat/frontend-ui` — Next.js / Vue Dashboard, PDF Previewer, and Settings UI.
- **Commit Message Convention:**
  - `docs:` — Documentation and PRD updates (`docs: add complete PRD`)
  - `feat:` — New features (`feat: implement auto-routing failover engine`)
  - `fix:` — Bug fixes
  - `chore:` — Docker, dependency, and configuration updates
- **Security Rule:** `.env` and `data/` directories are strictly excluded via `.gitignore`. All user API keys are stored locally inside the PostgreSQL database and never committed to GitHub.

---

## 8. Non-Functional Requirements

- **Lightweight Resource Footprint:** Total Docker container memory usage (Frontend + Backend + PostgreSQL) must remain under `1GB RAM` to run smoothly and coolly on an Intel MacBook Pro.
- **Fault Tolerance:** If an API key hits a rate limit or fails mid-book, translation must resume from the exact failed page/chunk using the next active provider without restarting from page 1.
- **Data Persistence:** Database records and translated PDFs must persist across container restarts via local Docker volume mounts (`./data/postgres` and `./data/pdfs`).

---

## 9. AI Coding Agent Execution Guidelines

When an AI Agent is tasked with implementing this PRD, it must strictly adhere to the following workflow:
1. **Branch Isolation:** Never commit directly to `main`. Create and switch to the corresponding feature branch (`feat/docker-db`, `feat/backend-engine`, or `feat/frontend-ui`) before writing code.
2. **Incremental Commits:** Follow the Commit Message Convention (`feat:`, `chore:`, `fix:`, `docs:`) after completing each logical module.
3. **Environment Safety:** Never hardcode API keys or database passwords; always reference environment variables defined in `.env.example` or fetch keys dynamically from the PostgreSQL `providers` table.
4. **Python & Font Compliance:** Ensure all backend code targets **Python 3.14.6** async patterns and bundles Myanmar Unicode fonts (`NotoSansMyanmar` / `Padauk`) inside `backend/fonts/`.