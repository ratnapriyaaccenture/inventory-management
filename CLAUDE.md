# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Factory Inventory Management System - Full-stack application with Vue 3 frontend, Python FastAPI backend, and in-memory mock data (no database).

## Stack
- **Frontend**: Vue 3 + Composition API + Vite (port 3000)
- **Backend**: Python FastAPI (port 8001)
- **Data**: JSON files in `server/data/` loaded into memory at startup via `server/mock_data.py`

## Commands

### Backend
```bash
# Preferred (if uv is installed)
cd server && uv run python main.py

# Fallback (if uv not in PATH)
cd server && python -m uvicorn main:app --host 0.0.0.0 --port 8001
```

### Frontend
```bash
cd client && npm install && npm run dev
```

### Tests
```bash
# All backend tests
cd tests && uv run pytest -v

# Single test file
cd tests && uv run pytest backend/test_inventory.py -v

# Single test
cd tests && uv run pytest backend/test_inventory.py::TestInventoryEndpoints::test_get_all_inventory -v

# With coverage
cd tests && uv run pytest --cov=../server --cov-report=html
```

## Architecture

### Filter System (Global State)
`client/src/composables/useFilters.js` is a **singleton** — refs are declared at module scope, so all views share the same filter state. The 4 filters (period, location/warehouse, category, status) are applied globally via query params on every API call. `FilterBar.vue` mutates this shared state; each view calls `getCurrentFilters()` from `useFilters()` to pass params to `api.js`.

### Data Flow
```
FilterBar.vue → useFilters (singleton refs) → api.js → FastAPI query params
→ apply_filters() / filter_by_month() → Pydantic model → computed properties in view
```

### Composables
- `useFilters.js` — global filter state (singleton, shared across all views)
- `useAuth.js` — mock current user; user profile fields are locale-aware (en/ja)
- `useI18n.js` — i18n singleton; supports `en` and `ja` locales, persisted to `localStorage`; currency follows locale (USD/JPY)

### API Endpoints
- `GET /api/inventory` — filters: warehouse, category
- `GET /api/inventory/{id}` — single item
- `GET /api/orders` — filters: warehouse, category, status, month
- `GET /api/orders/{id}` — single order
- `GET /api/dashboard/summary` — all filters
- `GET /api/demand` — no filters
- `GET /api/backlog` — no filters
- `GET /api/spending/summary|monthly|categories|transactions` — no filters
- `GET/POST /api/tasks` — task management
- `DELETE /api/tasks/{id}`, `PATCH /api/tasks/{id}` — toggle task
- `POST /api/purchase-orders` — create PO from backlog item
- `GET /api/purchase-orders/{backlog_item_id}`

### Backend Filtering
`server/main.py` has two reusable helpers: `apply_filters()` (warehouse, category, status) and `filter_by_month()` (supports direct month `2025-01` and quarter `Q1-2025` formats). All filtering operates on copies of in-memory lists — **data changes do not persist** across restarts.

### Frontend Views → API mapping
| View | API calls |
|------|-----------|
| Dashboard.vue | `/api/dashboard/summary` |
| Inventory.vue | `/api/inventory` |
| Orders.vue | `/api/orders` |
| Demand.vue | `/api/demand` |
| Backlog.vue | `/api/backlog`, `/api/purchase-orders` |
| Spending.vue | `/api/spending/*` |
| Reports.vue | multiple endpoints |

## Code Style
- Always document non-obvious logic changes with comments

## Key Constraints
- Inventory filters do **not** support month (no time dimension on inventory data)
- Revenue goals: $800K/month (single warehouse), $9.6M YTD (all months)
- Use unique keys in `v-for` — use `sku`, `id`, `month`, never array index
- Validate dates before calling `.getMonth()` — check `!isNaN(date.getTime())`
- When changing JSON data structure in `server/data/`, update the corresponding Pydantic model in `server/main.py`

## Design System
- Colors: Slate/gray (`#0f172a`, `#64748b`, `#e2e8f0`)
- Status colors: green/blue/yellow/red
- Charts: Custom SVG + CSS Grid (no charting library)
- No emojis in UI

## Critical Tool Usage Rules

### Subagents
- **vue-expert**: MANDATORY for creating or significantly modifying any `.vue` file
- **code-reviewer**: Use after writing significant code
- **Explore**: Use for codebase exploration and pattern searches
- **backend-api-test** skill: Use when writing/modifying tests in `tests/backend/`

### MCP Tools
- **Always** use `mcp__github__*` tools for GitHub operations (exception: local branch creation — use `git checkout -b`)
- **Always** use `mcp__playwright__*` for browser testing against `http://localhost:3000` (frontend) and `http://localhost:8001` (API)
