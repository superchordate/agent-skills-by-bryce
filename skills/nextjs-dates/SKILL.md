---
name: nextjs-dates
description: Guide for handling dates in Next.js applications with Firebase Firestore. Use when working with date conversions, Firestore timestamps, date pickers, displaying dates in UI, or handling timezone-specific date parsing (especially Central Time for birth/effective dates). Covers conversion functions and Central Time assumptions.
---

# Next.js Date Handling Guide

## Quick Reference

**Key Concepts:**
- [Date Formats Overview](#date-formats-overview) - Five date format types and when to use each
- [Central Time Assumption](#central-time-assumption) - Birth/effective dates always use Central Time
- [Conversion Functions](#conversion-functions) - Quick reference for all date functions

**Common Tasks:**
- Converting Firestore timestamps → use `timestampToDate()` → [Details](#1-firestore-timestamps)
- Formatting dates for input fields → use `formatDateForInput()` → [Details](#3-input-dates)
- Displaying dates in UI → use `formatDateForDisplay()` → [Details](#4-display-dates)
- Parsing user dates → use `parseDateAsCentralTime()` → [Details](#5-birth-dates-and-effective-dates)

**Important Rules:**
- Call `timestampToDate()` in data-fetching functions, not UI components
- All user-input dates (birth/effective) are assumed Central Time
- Functions located at `src/lib/firebase/dates.js`

## Date Formats Overview

Dates in this application use **five different formats** depending on context:

### 1. Firestore Timestamps
**When:** Raw data from Firestore documents  
**Convert to:** JavaScript Date using `timestampToDate()`  
**Where to convert:** In data-fetching functions (not UI components)

### 2. JavaScript Dates
**When:** Working with dates in code logic  
**Source:** Result of `timestampToDate()` conversion  
**Usage:** Standard format used throughout the codebase

### 3. Input Dates
**When:** HTML date picker fields  
**Convert to:** Input format using `formatDateForInput()`  
**Format:** YYYY-MM-DD string

### 4. Display Dates
**When:** Showing dates to users in UI  
**Convert to:** Display format using `formatDateForDisplay()`  
**Format:** MM/DD/YYYY or similar human-readable format

### 5. Birth Dates and Effective Dates
**When:** Parsing dates from user input (forms, CSV uploads)  
**Convert to:** JavaScript Date using `parseDateAsCentralTime()`  
**Timezone:** Always Central Time (Chicago), regardless of user location

## Conversion Functions

All functions available at `src/lib/firebase/dates.js`:

| Function | Input | Output | Use Case |
|----------|-------|--------|----------|
| `timestampToDate()` | Firestore Timestamp | JavaScript Date | Converting Firestore documents |
| `formatDateForInput()` | JavaScript Date | YYYY-MM-DD string | Populating date picker fields |
| `formatDateForDisplay()` | JavaScript Date | MM/DD/YYYY string | Showing dates in UI |
| `parseDateAsCentralTime()` | Date string | JavaScript Date | Parsing user birth/effective dates |
| `createEffectiveDate()` | Form input | JavaScript Date | Creating effective dates from forms |
| `calculateAge()` | Birth date string | Number | Age calculations (uses Central Time) |

## Central Time Assumption

**Critical Rule:** All birth dates and effective dates from user inputs are assumed to be in **Central Time (Chicago timezone)**, regardless of where the user is located.

**Why:** Ensures consistent age calculations and date handling across the application.

**When to use Central Time parsing:**
- Bulk data imports containing birth dates
- User registration forms with birth dates
- Effective date inputs
- Any date that affects date-based calculations

**Functions that enforce Central Time:**
- `parseDateAsCentralTime()` - Explicitly parses as Central Time
- `createEffectiveDate()` - Creates dates in Central Time
- `calculateAge()` - Automatically uses Central Time parsing

## Best Practices

**✅ DO:**
- Convert Firestore timestamps immediately in data-fetching functions
- Use `parseDateAsCentralTime()` for all birth and effective dates
- Use appropriate format conversion before displaying dates
- Keep date logic in data layer, not UI components

**❌ DON'T:**
- Call `timestampToDate()` repeatedly in UI components
- Assume user's local timezone for birth/effective dates
- Display raw JavaScript Date objects in UI
- Format dates manually - use the helper functions
