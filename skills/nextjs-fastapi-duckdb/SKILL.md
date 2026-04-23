---
name: nextjs-fastapi-duckdb
description: Guide for adding a FastAPI backend with DuckDB to an existing Next.js app. Use when setting up FastAPI, creating a Python venv, configuring the /api/query proxy route, querying Parquet files with DuckDB, or wiring up dev scripts to run both servers together.
---

# Next.js + FastAPI + DuckDB Setup

Assumes an existing Next.js app. Adds a FastAPI server that executes DuckDB queries against Parquet files, proxied through a Next.js API route.

## Quick Reference
- FastAPI server: `server/main.py`, runs on port 8000
- Venv: `venv/` at project root (shared across admin scripts)
- Parquet data: `server/data/*.parquet`
- Next.js proxy: `app/api/query/route.ts` → `POST /query` on FastAPI
- Dev script: `npm run dev` → `scripts/start.ps1` (starts both servers)
- Client utility: `app/utils/duckdb.ts` → `executeQuery(sql)`

---

## Step 1: Create the Python Virtual Environment

Run from the project root:

```bash
python -m venv venv
```

Activate and install dependencies:

```bash
# Windows (PowerShell)
venv\Scripts\Activate.ps1
pip install fastapi "uvicorn[standard]" duckdb

# Mac/Linux
source venv/bin/activate
pip install fastapi "uvicorn[standard]" duckdb
```

Save dependencies:

```bash
pip freeze > server/requirements.txt
```

Minimal `server/requirements.txt`:
```
fastapi
uvicorn[standard]
duckdb
```

---

## Step 2: FastAPI Server (`server/main.py`)

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import duckdb
import os
from typing import List, Dict, Any

app = FastAPI()

class QueryRequest(BaseModel):
    sql: str

class QueryResponse(BaseModel):
    data: List[Dict[str, Any]]
    columns: List[str]
    row_count: int

conn = duckdb.connect(':memory:')

PARQUET_PATH = os.path.join(os.path.dirname(__file__), 'data', 'claims.parquet')

@app.on_event("startup")
async def startup_event():
    if os.path.exists(PARQUET_PATH):
        conn.execute(f"CREATE VIEW claims AS SELECT * FROM '{PARQUET_PATH}'")

@app.get("/health")
async def health_check():
    return {"status": "healthy"}

@app.post("/query", response_model=QueryResponse)
async def execute_query(request: QueryRequest):
    sql = request.sql.strip()
    first_word = sql.upper().split()[0] if sql.split() else ""
    if first_word not in ('SELECT', 'WITH'):
        raise HTTPException(status_code=400, detail="Only SELECT queries are allowed")
    try:
        result = conn.execute(sql)
        rows = result.fetchall()
        columns = [desc[0] for desc in result.description]
        data = [dict(zip(columns, row)) for row in rows]
        return QueryResponse(data=data, columns=columns, row_count=len(data))
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=int(os.environ.get("PORT", 8000)))
```

**Multiple Parquet files:** Register each as its own view at startup:
```python
conn.execute(f"CREATE VIEW exposure AS SELECT * FROM '{EXPOSURE_PATH}'")
```

---

## Step 3: Run Both Servers Together

### Option A: PowerShell script (Windows) — `scripts/start.ps1`

```powershell
Write-Host "Starting FastAPI server..." -ForegroundColor Yellow
Start-Process -NoNewWindow powershell -ArgumentList "-Command", "cd server; ..\venv\Scripts\Activate.ps1; uvicorn main:app --reload --port 8000"

Start-Sleep -Seconds 3

$env:FASTAPI_URL = "http://localhost:8000"

Write-Host "Starting Next.js..." -ForegroundColor Yellow
next dev
```

Hook into `npm run dev` in `package.json`:

```json
"scripts": {
  "dev": "powershell -File scripts/start.ps1"
}
```

### Option B: `concurrently` (cross-platform)

```bash
npm install --save-dev concurrently
```

```json
"scripts": {
  "dev:api": "cd server && ../venv/bin/uvicorn main:app --reload --port 8000",
  "dev:next": "next dev",
  "dev": "concurrently \"npm run dev:api\" \"npm run dev:next\""
}
```

---

## Step 4: Next.js API Proxy Route (`app/api/query/route.ts`)

The Next.js app never calls FastAPI directly from the browser. All DuckDB queries go through a server-side proxy route that handles auth.

```typescript
import { NextRequest, NextResponse } from 'next/server';

const FASTAPI_URL = process.env.FASTAPI_URL || 'http://localhost:8000';

export async function POST(request: NextRequest) {
    const body = await request.json();
    const { sql } = body;

    if (!sql) {
        return NextResponse.json({ error: 'SQL query is required' }, { status: 400 });
    }

    const response = await fetch(`${FASTAPI_URL}/query`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ sql }),
    });

    if (!response.ok) {
        const error = await response.text();
        return NextResponse.json({ error }, { status: response.status });
    }

    return NextResponse.json(await response.json());
}
```

**With Firebase auth:** Verify the user's ID token before proxying:
```typescript
const user = await getAuthenticatedUser(request); // your Firebase auth helper
if (!user) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
```

**Single-container deployment:** When Next.js and FastAPI run in the same Docker container (recommended), `FASTAPI_URL` is always `http://localhost:8000` and no service-to-service auth is needed — FastAPI is never exposed externally.

---

## Step 5: Client-Side Query Utility (`app/utils/duckdb.ts`)

```typescript
export interface QueryResult {
    data: any[];
    columns: string[];
    row_count: number;
}

export async function executeQuery(sql: string): Promise<QueryResult> {
    const response = await fetch('/api/query', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ sql }),
    });

    if (!response.ok) {
        const err = await response.json();
        throw new Error(err.error || 'Query failed');
    }

    return response.json();
}
```

Usage in a component or context:
```typescript
const { data } = await executeQuery(`
    SELECT Company, SUM("Total Incurred") as total
    FROM claims
    WHERE "Used" = 1
    GROUP BY Company
    ORDER BY total DESC
`);
```

---

## Step 6: Environment Configuration

Local dev (`.env.local` or set in start script):
```
FASTAPI_URL=http://localhost:8000
```

Next.js reads `FASTAPI_URL` server-side only (no `NEXT_PUBLIC_` prefix needed — keep FastAPI internal).

To expose a flag to the browser (e.g., to toggle DuckDB mode):
```
NEXT_PUBLIC_USE_DUCKDB=true
```

---

## Step 7: DuckDB SQL Patterns

DuckDB reads Parquet files as SQL tables via views registered at startup. Use standard SQL:

```sql
-- Filter and aggregate
SELECT "Job Classification", COUNT(*) as claims, SUM("Total Incurred") as total
FROM claims
WHERE "Company" = 'Acme Corp' AND "Used" = 1
GROUP BY "Job Classification"

-- CTE for proration
WITH base AS (
    SELECT period, payroll
    FROM exposure
    WHERE source = 'Jackson'
)
SELECT period, payroll / SUM(payroll) OVER () as pct
FROM base

-- Join across views
SELECT c.period, c.total_incurred, e.payroll
FROM claims c
FULL OUTER JOIN exposure e ON c.period = e.period
```

**Column names with spaces:** Always quote with double quotes: `"Job Classification"`, `"Total Incurred"`.

---

## Deployment (Single Docker Container)

Run both Next.js and FastAPI in one container — Next.js handles external traffic, FastAPI stays internal on `localhost:8000`. One Cloud Run service, no service-to-service auth needed.

### Architecture
```
Internet → Cloud Run (PORT) → Next.js :3000
                                  ↓ /api/query proxy
                             FastAPI :8000 (localhost only)
```

### `next.config.ts` — enable standalone output
```typescript
const nextConfig = {
  output: 'standalone',
};
export default nextConfig;
```

### `scripts/start.sh` — container entrypoint
```bash
#!/bin/sh
set -e

# Start FastAPI on localhost:8000 (internal only)
cd /app/server
uvicorn main:app --host 127.0.0.1 --port 8000 &

# Wait for FastAPI to be ready
until curl -sf http://localhost:8000/health > /dev/null; do
  sleep 1
done

# Start Next.js on $PORT (Cloud Run sets this, default 3000)
cd /app
HOSTNAME=0.0.0.0 node server.js
```

### `Dockerfile` (project root)
```dockerfile
FROM node:20-slim AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:20-slim AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:20-slim AS runner
WORKDIR /app

# Install Python and curl for healthcheck
RUN apt-get update && apt-get install -y python3 python3-pip curl && rm -rf /var/lib/apt/lists/*

# Install FastAPI dependencies
COPY server/requirements.txt ./server/requirements.txt
RUN pip3 install --no-cache-dir -r server/requirements.txt

# Copy Next.js standalone build
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public

# Copy FastAPI server and Parquet data
COPY server/ ./server/

# Copy entrypoint
COPY scripts/start.sh ./scripts/start.sh
RUN chmod +x scripts/start.sh

ENV FASTAPI_URL=http://localhost:8000
ENV NODE_ENV=production
ENV PORT=3000

EXPOSE 3000
CMD ["scripts/start.sh"]
```

### Cloud Run deployment
```bash
# Build and push
gcloud builds submit --tag gcr.io/PROJECT_ID/claims-analytics

# Deploy — only one service needed
gcloud run deploy claims-analytics \
  --image gcr.io/PROJECT_ID/claims-analytics \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --memory 1Gi
```

Cloud Run injects `PORT` automatically; Next.js standalone reads it via `node server.js`.

**Parquet data:** Bundle into the image with `COPY server/data/ ./server/data/` (shown above). For larger datasets, mount from Cloud Storage using the Cloud Run volume mount feature instead.

### `.dockerignore`
```
node_modules
.next
venv
__pycache__
*.pyc
.env*
data/*.xlsx
```
