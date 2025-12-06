# Smart Reorder Assistant (working title)

> Upload your sales and stock data, get a clear reorder plan that tells you **what to buy, when, and how much**.



## 1. What this project is

Small shops, cafés, and bakeries often live in **Excel and CSV files**.  
They know *roughly* what sells, but they don’t have a simple way to see:

- When they will **run out** of each item
- How many days of stock they have left
- What they **should reorder today**

This project is a small, focused web app that turns:

- `sales.csv` (past sales)
- `stock.csv` (current stock)

into a **reorder plan**.

The goal is to be:

- **Fast** to try (upload files, click one button)
- **Clear** to read (one table with priorities)
- **Simple** to run (Python backend + Vite React frontend)



## 2. Who this is for (v1)

Initial target users:

- Small bakeries and cafés
- Home-based food businesses
- Any small business that:
  - Tracks sales and stock in CSV/Excel
  - Has no proper inventory system
  - Wants a quick way to know *“what do I need to reorder?”*

Later, this can expand to:

- Small retail shops
- Small warehouses / distributors
- Any business with recurring sales + stock data



## 3. Core idea (top-level logic)

At a high level, the app does **one job**:

> **From recent sales + current stock → calculate how many days of stock are left and what to reorder, given a lead time and safety buffer.**

Inputs (per run):

- `sales.csv`:
  - `date`
  - `product`
  - `qty_sold`
- `stock.csv`:
  - `product`
  - `current_stock`
- Settings:
  - `leadTimeDays`
  - `safetyDays`
  - `windowDays` (how many past days to look at, e.g. 30)

Output (per product):

- `avg_daily_sales`
- `current_stock`
- `days_of_stock` (how many days until stock hits zero)
- `stockout_date` (estimated date)
- `reorder_qty` (how much to order now)
- `status`:
  - `RISK` – likely to run out before lead time
  - `WATCH` – okay but close to the edge
  - `OK` – safe for now



## 4. System at a glance (Level 1 view)

**Frontend**: Vite + React  
- Single-page app
- Lets user upload CSVs
- Sends data to backend as JSON
- Shows the reorder plan in a table

**Backend**: Python (FastAPI)  
- Receives sales + stock + settings as JSON
- Runs the calculation logic
- Returns a reorder plan as JSON

**Data storage (MVP)**  
- No database in the first version
- Everything is handled in-memory per request

Later, we can add:

- Postgres for persistent data
- Object storage for raw files
- Auth, tenants, scheduled jobs, etc.



## 5. Project stages (3 layers, going deeper over time)

### Stage 1 – MVP (single-user, file-based)

**Goal:**  
Prove the idea works end-to-end with simple CSV upload and a clean UI.

**What it does:**

- User uploads `sales.csv` and `stock.csv`
- User enters lead time and safety days
- Frontend parses files and sends structured JSON to the backend
- Backend returns a reorder plan
- Frontend shows:
  - Products sorted by urgency (days of stock ascending)
  - A simple risk status (OK / WATCH / RISK)
  - Optional “Download plan as CSV”

**What it does *not* do yet:**

- No login or accounts
- No database
- No saved history
- No integrations (Square, Shopify, etc.)



### Stage 2 – Multi-user + persistence

**Goal:**  
Turn the tool from a one-off calculator into a basic SaaS-style product.

**Key additions (high-level):**

- User accounts (email + password or OAuth)
- Each user/business has:
  - Saved products
  - Stored sales history
  - Latest stock snapshot
  - Default settings (lead time, safety days, window)
- Ability to:
  - Save and list previous reorder plans
  - Re-run a plan without re-uploading all files

**Tech changes:**

- Add a database (e.g. Postgres) for:
  - Users / tenants
  - Products
  - Sales
  - Stock snapshots
  - Reorder plans
- Add JWT or session-based auth in the backend
- Scope all data by tenant/user



### Stage 3 – Automation + integrations

**Goal:**  
Make the app useful **without manual CSV uploads** and reduce manual work.

**Planned features (high-level):**

- Daily or weekly **automatic plan generation**
  - e.g. “Every morning at 7am, create a new reorder plan”
- Email or in-app notifications:
  - “You have 3 products at risk this week”
- Integrations with existing tools:
  - POS / e-commerce (e.g. Square, Shopify, etc.) to pull sales
  - Simple webhooks or APIs for other systems

**Tech changes:**

- Background jobs (Celery/RQ + Redis) for heavy tasks
- Object storage (S3-like) for raw imports
- Monitoring, logging, and basic error dashboards



## 6. Current focus

Right now, the focus is **Stage 1 (MVP)**:

- [ ] Finalize input and output format  
- [ ] Implement the pure reorder calculation logic in Python  
- [ ] Expose a simple `POST /reorder-plan` endpoint (FastAPI)  
- [ ] Build a single Vite + React page for:
  - CSV upload
  - Settings inputs
  - Results table

Once Stage 1 is working and tested with sample data, the README will be extended with:

- Setup instructions (install, run backend, run frontend)
- Example CSV files
- Screenshots



## 7. High-level tech stack (for now)

- **Frontend:** Vite + React + TypeScript  
- **Backend:** Python + FastAPI  
- **Transport:** JSON over HTTP  
- **First deployment target (likely):**
  - Frontend → Vercel / Netlify
  - Backend → Render / Railway / Fly.io

This document is intentionally kept high-level so we can refine each section as the project grows and we go one level deeper.