# Product Requirements Document (PRD): Thi Kha

## 1. Product Overview & Repository Info
- **Product Name:** Thi Kha
- **App Icon:** Minimalist Book Icon (SVG / Favicon)
- **Version:** 1.0 (MVP)
- **GitHub Repository:** `[https://github.com/qjoy-2462006/thi-kha.git](https://github.com/qjoy-2462006/thi-kha.git)`
- **Local Workspace Path:** `/Users/qjoy/Q joy Learning Map/Programming Learning/thi-kha`
- **Deployment Target:** Local Self-Hosted Web Application via Docker (Optimized for Intel MacBook Pro & Cross-Platform)
- **Core Objective:** Translate text-based English PDF books directly into Myanmar (Myanmar Unicode) using free AI APIs while preserving the original document layout (page structure, images, headings, tables).

---

## 2. System Architecture & Tech Stack
- **Containerization:** Docker & Docker Compose (Lightweight footprint, no heavy local LLM models)
- **Backend Runtime:** **Python 3.14.6** (`python:3.14.6-slim`)
- **Backend Framework:** FastAPI + Uvicorn (Async/Await task handling & SSE/WebSocket live progress streaming)
- **Frontend Framework:** Next.js (React) or Vue 3 + Tailwind CSS (Responsive single-page dashboard)
- **PDF & Burmese Shaping Engine:**
  - `PyMuPDF` (`fitz`) — Exact bounding box coordinates, font size extraction, and vector/image preservation.
  - `uharfbuzz` + `ReportLab` / `WeasyPrint` — Accurate Myanmar Unicode complex text shaping (stacking, ligatures, diacritics).
- **Database:** PostgreSQL 16 (Exposed port `5432` for direct connection & inspection via **DBngin** or **TablePlus**)
- **Network & VPN:** Relies on the Host Machine's external VPN connection directly via Docker bridge/DNS pass-through so all global AI providers work without geographical restrictions.

---

## 3. Core Functional Requirements

### 3.1. Dynamic Provider & API Key Management
- **Custom Provider Setup:** Users can manually add and save Provider names (e.g., `Google AI Studio`, `Groq`, `OpenRouter`, `Cloudflare Workers AI`), API Keys, and optional Base URLs / Account IDs via the Settings UI.
- **Auto-Model Discovery:** No manual per-model selection required; the system automatically fetches and utilizes available/free models supported by the configured API key.
- **Smart Auto-Routing & Failover Engine:**
  - If an active Provider/Key hits a Rate Limit (`HTTP 429`), Quota Exceeded, or API Error mid-translation, the system must automatically route/failover to the next available active Provider/Key/Model in the database and resume seamlessly from the current page without halting.
  - Supports priority ordering and round-robin load balancing across configured keys.

### 3.2. PDF Processing & Layout Preservation Engine
- **Input Scope:** Accepts text-based PDFs only (OCR for scanned/image-only PDFs is out of scope).
- **Element Separation:**
  - Images, vector graphics, tables, and page borders must remain untouched in their original positions.
  - Extracts exact bounding box coordinates (`x0, y0, x1, y1`), font sizes, and alignments for all text blocks.
- **Myanmar Unicode & Auto Font-Scaling:**
  - All translated output must conform to **Myanmar Unicode Standard** (Default Font: `Noto Sans Myanmar` or `Padauk`).
  - Because Burmese text typically occupies more space than English, the engine must perform **Auto Font-Scaling** and smart line-wrapping so translated text fits cleanly inside the original bounding boxes without overlapping.

### 3.3. Real-Time Progress & Single-Pane PDF Preview
- **Live Progress Tracking:** Displays real-time page-by-page translation progress (e.g., `Translating Page 12 of 150... | Active Provider: Groq`), percentage bar, and failover event logs in the UI.
- **Built-in Myanmar PDF Viewer:** Embedded single-pane PDF previewer allowing users to read and inspect the translated Burmese PDF directly inside the app.
- **Export & Download:** Allows downloading the final translated file named `[Original_File_Name]_mm.pdf`.

### 3.4. Translation History & Persistence
- Stores metadata and file paths of all translated books in PostgreSQL.
- Users can browse, preview, re-download, or delete past translations from the **History** tab at any time.

---

## 4. Database Schema Design (PostgreSQL 16)

Designed for easy inspection and management via **DBngin** or **TablePlus**:

| Table Name | Columns & Data Types | Description |
| :--- | :--- | :--- |
| **`providers`** | `id` (UUID, PK), `name` (VARCHAR), `api_key` (TEXT), `base_url` (VARCHAR, Nullable), `account_id` (VARCHAR, Nullable), `priority` (INT), `is_active` (BOOLEAN), `status` (VARCHAR: `active`/`rate_limited`/`error`), `created_at` (TIMESTAMP) | User-configured API providers, keys, and priority routing order |
| **`books_history`** | `id` (UUID, PK), `original_filename` (VARCHAR), `translated_filename` (VARCHAR), `file_path` (TEXT), `total_pages` (INT), `completed_pages` (INT), `status` (VARCHAR: `processing`/`completed`/`failed`), `used_providers` (JSONB), `created_at` (TIMESTAMP) | History and status of all translated PDF books |
| **`translation_logs`** | `id` (SERIAL, PK), `book_id` (UUID, FK), `page_number` (INT), `provider_id` (UUID, FK), `model_used` (VARCHAR), `status` (VARCHAR), `error_message` (TEXT, Nullable), `timestamp` (TIMESTAMP) | Audit logs for auto-routing failovers, rate limits, and page-level status |

---

## 5. User Interface (UI) Specification

1. **Top Navigation Bar:**
   - **Thi Kha** Logo (Minimalist Book Icon) + App Title
   - Active Providers Count Badge & Network/VPN Status Indicator
2. **Workspace Views (3 Main Tabs):**
   - **Translate (Home):** Drag & Drop PDF Upload Zone -> Page Range Selector (`All` or Custom e.g., `1-25`) -> "Start Translation" Button -> Live Progress Bar & Provider Failover Status -> Built-in Myanmar PDF Previewer & Download Button.
   - **History:** Searchable list/cards of translated books with Preview, Download, and Delete actions.
   - **Settings (Providers & Keys):** Add New Provider Form (`Provider Name`, `API Key`, `Optional Base URL / Account ID`), Priority Sorting, Active/Inactive Toggle, and Test Connection Button.

---

## 6. Project Folder Structure

```text
thi-kha/
├── .gitignore                       # Ignores .env, data/, *.pdf, node_modules, __pycache__
├── docker-compose.yml               # Orchestrates frontend, backend, and postgres containers
├── .env.example                     # Environment variables template (DB credentials, ports)
├── README.md                        # Project overview & Docker setup guide
│
├── docs/
│   └── PRD.md                       # Complete Product Requirements Document
│
├── data/                            # Persistent local volumes (Git-ignored)
│   ├── postgres/                    # PostgreSQL 16 data directory (TablePlus/DBngin compatible)
│   └── pdfs/
│       ├── uploads/                 # Temporary original English PDFs
│       └── translated/              # Final generated Myanmar Unicode PDFs (*_mm.pdf)
│
├── backend/                         # Python 3.14.6 FastAPI Service
│   ├── Dockerfile                   # python:3.14.6-slim + PyMuPDF/HarfBuzz build dependencies
│   ├── requirements.txt             # fastapi, uvicorn, pymupdf, uharfbuzz, sqlalchemy, asyncpg, httpx
│   ├── fonts/                       # Bundled Myanmar Unicode fonts
│   │   ├── NotoSansMyanmar-Regular.ttf
│   │   ├── NotoSansMyanmar-Bold.ttf
│   │   └── Padauk-Regular.ttf
│   └── app/
│       ├── main.py                  # FastAPI entry point, CORS, SSE progress endpoints
│       ├── config.py                # App & DB configuration settings
│       ├── database.py              # Async SQLAlchemy engine & session setup
│       ├── models/                  # PostgreSQL ORM models
│       │   ├── provider.py
│       │   ├── book.py
│       │   └── log.py
│       ├── schemas/                 # Pydantic request/response validation schemas
│       │   ├── provider_schema.py
│       │   └── book_schema.py
│       ├── api/                     # REST & Streaming routes
│       │   ├── routes_providers.py  # CRUD & auto-fetch models for API keys
│       │   ├── routes_translate.py  # PDF upload, start job, SSE live progress
│       │   └── routes_history.py    # List, preview stream, download, delete books
│       └── services/                # Core business logic engines
│           ├── router_engine.py     # Multi-provider auto-routing & failover manager
│           ├── model_fetcher.py     # Auto-discovers available models per provider key
│           ├── pdf_extractor.py     # Extracts text blocks & bounding boxes via PyMuPDF
│           ├── pdf_builder.py       # Renders Myanmar Unicode with HarfBuzz & auto-scaling
│           └── translator_job.py    # Async background worker coordinating page translation
│
└── frontend/                        # Next.js or Vue 3 Web Application
    ├── Dockerfile                   # Node 22 Alpine lightweight build
    ├── package.json
    ├── public/
    │   └── book-icon.svg            # Thi Kha minimalist book logo & favicon
    └── src/
        ├── app/ (or views/)
        │   ├── page.tsx             # Translate Workspace (Upload, Progress, PDF Preview)
        │   ├── history/page.tsx     # Translated Books History view
        │   └── settings/page.tsx    # Add Provider & API Key Setup view
        ├── components/
        │   ├── Navbar.tsx           # Header with Thi Kha Book Icon & status badge
        │   ├── UploadDropzone.tsx   # PDF drag-and-drop & page range selector
        │   ├── ProgressTracker.tsx  # Live progress bar & active provider failover status
        │   ├── PdfPreviewer.tsx     # Embedded single-pane Myanmar PDF viewer
        │   └── ProviderForm.tsx     # Dynamic provider name & API key setup modal/form
        └── lib/
            └── api.ts               # Backend API client & SSE event listener