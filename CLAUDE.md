# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Feuilles de Caisse** — A French-language web and mobile application for automating monthly cash sheet generation from bank statement PDFs. Extracts transactions, categorizes them by accounting codes, and exports to Excel.

Stack: Python/FastAPI backend, React/TypeScript/Vite frontend, React Native mobile app, SQLite database.

## Commands

### Backend (from `github/feuilles-de-caisse/backend/`)
```
venv\Scripts\activate                              # Activate virtualenv (Windows)
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000  # Dev server with hot reload
pip install -r requirements.txt                    # Install dependencies
python recreate_database.py                        # Reset database
```

### Frontend (from `github/feuilles-de-caisse/frontend/`)
```
npm run dev       # Dev server on :5173 (proxies /api to backend)
npm run build     # Production build
npm run preview   # Preview production build
```

### Mobile (from `github/feuilles-de-caisse/mobile/`)
```
npm run android   # Build and run on Android
npm run start     # Start Metro bundler
npm test          # Run Jest tests
```

### Quick Start (from repo root)
- Windows: `.\START.ps1` / `.\STOP.ps1`
- Linux/Mac: `./start.sh` / `./stop.sh`

## Architecture

### Data Flow
1. User uploads PDF bank statement → `POST /api/uploads/bank-statement`
2. Backend parses PDF with pypdf (layout mode), extracts transactions via regex
3. `TransactionCategorizer` maps transactions to accounting codes using keyword matching
4. Frontend displays transactions for user review/editing
5. User confirms → `POST /api/cash-sheets` + `POST /api/transactions/batch`
6. `POST /api/cash-sheets/{id}/calculate` computes totals
7. `GET /api/cash-sheets/{id}/export` generates Excel via openpyxl

### Backend Structure (`backend/app/`)
- **main.py** — FastAPI entry point, CORS config (allows :5173 and :3000)
- **models.py** — SQLAlchemy ORM: `CashSheet`, `Transaction`
- **schemas.py** — Pydantic validation schemas
- **routers/** — REST endpoints: `cash_sheets.py`, `transactions.py`, `uploads.py`
- **services/** — Business logic: `pdf_parser.py`, `categorizer.py`, `excel_export.py`
- **database.py** — SQLAlchemy engine setup (SQLite file: `feuilles_caisse.db`)

### Frontend Structure (`frontend/src/`)
- **App.tsx** — Main app component
- **components/** — `UploadStatement`, `TransactionList`, `CashManagement`, `CashSheet`
- **services/api.ts** — Axios client (base URL: `http://localhost:8000/api`)
- **types/index.ts** — TypeScript interfaces matching backend schemas

### Mobile Structure (`mobile/src/`)
- **screens/** — `HomeScreen`, `UploadScreen`, `TransactionsScreen`, `CashManagementScreen`, `SummaryScreen`
- **navigation/AppNavigator.tsx** — React Navigation stack

## Key Domain Concepts

- **TransactionType**: `CB`, `VIREMENT`, `CHEQUE`, `PRELEVEMENT`, `ESPECES`, `AUTRE`
- **TransactionCategory**: 6-digit accounting codes (e.g., `707000` = recettes, `511300` = CB, `401000` = fournisseurs)
- **Soft delete**: Transactions use `deleted` + `deleted_at` fields; restore endpoint exists
- **CashSheet**: Monthly record with opening/closing balance and computed totals

## Conventions

- All UI text and most documentation is in **French**
- Python: snake_case; TypeScript: camelCase files, PascalCase components
- API routes: plural nouns (`/api/cash-sheets`, `/api/transactions`)
- No migrations system — SQLAlchemy creates tables from model definitions on startup
- TypeScript strict mode enabled (`noUnusedLocals`, `noUnusedParameters`)
- State management: React hooks only (no Redux/Zustand)
- API docs auto-generated at `http://localhost:8000/docs`
