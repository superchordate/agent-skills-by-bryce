---
name: web-stack
description: Our preferred web application stack skill covering Next.js frontend development, Firestore database patterns, FastAPI+DuckDB analytics backend, and Jest testing. Use when building, modifying, or testing full-stack web applications with this cohesive technology stack.
---

# Web Stack Development Guide

Comprehensive guide for building modern web applications using a proven, cohesive technology stack: Next.js, Firestore, FastAPI+DuckDB, and Jest.

## What This Skill Covers

This skill covers the complete web development stack for building production-ready applications:

1. **Date Handling** - Working with dates, Firestore timestamps, and timezone-specific parsing
2. **Firestore Integration** - Reading, writing, and querying Firebase Firestore database
3. **FastAPI + DuckDB Backend** - Adding a Python analytics backend for data queries
4. **Testing with Jest** - Unit and integration testing for frontend components and utilities

## Decision Guide

Use this guide to quickly find the right resource for your task:

### 🗓️ Working with Dates?

**See: [Date Handling Guide](assets/dates.md)**

Use when you need to:
- Convert Firestore timestamps to JavaScript Dates
- Format dates for input fields (YYYY-MM-DD)
- Display dates in UI (MM/DD/YYYY)
- Parse user-input dates (birth dates, effective dates)
- Handle timezone-specific date parsing (Central Time)
- Calculate ages or date differences

**Quick concepts:**
- Five date format types (Firestore Timestamp, JS Date, Input, Display, Birth/Effective)
- All birth/effective dates use Central Time regardless of user location
- Conversion functions at `src/lib/firebase/dates.js`
- Call `timestampToDate()` in data-fetching functions, not UI components

---

### 🔥 Working with Firestore?

**See: [Firestore Integration Guide](assets/firestore.md)**

Use when you need to:
- Read or write Firestore documents
- Query documents with filters and ordering
- Create human-readable document IDs
- Set up Firestore indexes
- Handle client vs server-side Firestore operations
- Add required audit fields (createdAt, lastModified, etc.)

**Quick concepts:**
- Different imports for client vs server contexts
- Required fields: createdAt, lastModified, createdBy, lastModifiedBy
- Human-readable ID format: `{index-info}-{random-hash}`
- Deploy indexes: `firebase deploy --only firestore:indexes`
- Use usernames (not UIDs) for user references

---

### 📊 Adding Analytics/Data Backend?

**See: [FastAPI + DuckDB Setup Guide](assets/fastapi-duckdb.md)**

Use when you need to:
- Add a Python/FastAPI backend to Next.js
- Query Parquet files with DuckDB
- Set up a Python virtual environment
- Create a Next.js proxy API route
- Run both servers together in development
- Deploy Next.js + FastAPI in a single container

**Quick concepts:**
- FastAPI runs on port 8000 (internal only)
- Next.js proxies queries through `/api/query` route
- DuckDB queries Parquet files loaded as SQL views
- Single container deployment (no service-to-service auth needed)
- Dev script starts both servers: `npm run dev`

---

### 🧪 Writing Tests?

**See: [Testing Guide](assets/testing.md)**

Use when you need to:
- Set up Jest for Next.js
- Test React components (Server/Client)
- Mock Next.js features (navigation, Link, Image)
- Mock Firestore operations
- Test utility functions and plain JavaScript
- Write integration tests for user flows
- Add test command comments to source files

**Quick concepts:**
- Document test commands at top of tested files
- Use Jest alone for utilities, React Testing Library for components
- Mock `next/navigation`, `next/link`, `next/image`
- Query priority: `getByRole` > `getByLabelText` > `getByTestId`
- Run tests: `npm test -- ComponentName`

---

## Technology Stack Overview

**Frontend:** Next.js (React) with TypeScript  
**Database:** Firebase Firestore (NoSQL)  
**Analytics Backend:** Python FastAPI + DuckDB  
**Testing:** Jest + React Testing Library  
**Deployment:** Cloud Run (single container for Next.js + FastAPI)

This stack provides:
- **Fast development** - React/Next.js ecosystem + Firebase
- **Type safety** - TypeScript throughout
- **Scalable database** - Firestore auto-scales
- **Powerful analytics** - DuckDB for complex queries on Parquet files
- **Single deployment** - Both frontend and backend in one container

---

## Next.js Best Practices

### Project Structure

```
app/
  api/              # API routes (server-side)
  components/       # Reusable components
  (routes)/         # Page routes
lib/
  firebase/         # Firebase config and utilities
  utils/            # Utility functions
__tests__/          # Test files
  components/
  lib/
  pages/
server/             # FastAPI backend (if using)
  data/             # Parquet files
  main.py           # FastAPI app
```

### Client vs Server Components

**Server Components (default):**
- No "use client" directive
- Can use async/await directly
- Direct database access
- No browser APIs
- Better performance, smaller bundles

**Client Components:**
- Require "use client" directive
- Use hooks (useState, useEffect, etc.)
- Browser APIs available
- Event handlers (onClick, etc.)
- Interactive UI

**Key rule:** Keep components Server Components unless you need interactivity or hooks.

### Environment Variables

**Server-side only:**
```env
DATABASE_URL=...
API_SECRET=...
FASTAPI_URL=http://localhost:8000
```

**Client-side (exposed to browser):**
```env
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_RECAPTCHA_KEY=...
```

### Data Fetching

**Server Components (preferred):**
```typescript
async function getData() {
  const res = await fetch('https://api.example.com/data')
  return res.json()
}

export default async function Page() {
  const data = await getData()
  return <div>{data.title}</div>
}
```

**Client Components (when needed):**
```typescript
'use client'
import { useEffect, useState } from 'react'

export default function Page() {
  const [data, setData] = useState(null)
  
  useEffect(() => {
    fetch('/api/data')
      .then(res => res.json())
      .then(setData)
  }, [])
  
  return <div>{data?.title}</div>
}
```

### API Routes

**Location:** `app/api/[route]/route.ts`

**Basic structure:**
```typescript
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  return NextResponse.json({ data: 'value' })
}

export async function POST(request: NextRequest) {
  const body = await request.json()
  // Process request
  return NextResponse.json({ success: true })
}
```

**With authentication:**
```typescript
export async function GET(request: NextRequest) {
  const user = await getAuthenticatedUser(request)
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }
  
  // Handle authenticated request
  return NextResponse.json({ data: 'value' })
}
```

### TypeScript Best Practices

**Props interfaces:**
```typescript
interface ButtonProps {
  onClick: () => void
  children: React.ReactNode
  variant?: 'primary' | 'secondary'
}

export function Button({ onClick, children, variant = 'primary' }: ButtonProps) {
  return <button onClick={onClick}>{children}</button>
}
```

**Page props:**
```typescript
interface PageProps {
  params: { id: string }
  searchParams: { [key: string]: string | string[] | undefined }
}

export default function Page({ params, searchParams }: PageProps) {
  return <div>ID: {params.id}</div>
}
```

### Error Handling

**Error boundaries (client components):**
```typescript
'use client'
import { useEffect } from 'react'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    console.error(error)
  }, [error])

  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}
```

**Loading states:**
```typescript
// loading.tsx in route folder
export default function Loading() {
  return <div>Loading...</div>
}
```

---

## Quick Command Reference

```bash
# Development
npm run dev                    # Start dev server
npm run build                  # Build for production
npm start                      # Start production server

# Testing
npm test                       # Run all tests
npm test -- ComponentName      # Run specific test
npm test -- --watch            # Watch mode
npm test -- --coverage         # Coverage report

# Deployment
firebase deploy                # Deploy to Firebase Hosting
firebase deploy --only firestore:indexes  # Deploy Firestore indexes

# Docker (if using FastAPI backend)
docker build -t myapp .
docker run -p 3000:3000 myapp
```

---

## Resources

For detailed information on each topic, see the specific guides:

- **[Date Handling](assets/dates.md)** - Comprehensive date conversion and formatting guide
- **[Firestore Integration](assets/firestore.md)** - Complete Firestore patterns for Next.js
- **[FastAPI + DuckDB Setup](assets/fastapi-duckdb.md)** - Full backend integration guide
- **[Testing with Jest](assets/testing.md)** - Complete testing reference

### External Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [React Documentation](https://react.dev)
- [TypeScript Documentation](https://www.typescriptlang.org/docs)
- [Firebase Documentation](https://firebase.google.com/docs)
