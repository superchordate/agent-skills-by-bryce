---
name: ui-ux
description: User interface and user experience standards for this application. Use when creating or modifying UI components, buttons, links, modals, forms, or interactive elements. Covers Tailwind CSS conventions, accessibility requirements, and modal behavior.
---

# UI/UX Standards Guide

## Quick Reference

**Interactive Elements:**
- [Buttons and Links](#buttons-and-links) - Cursor pointer requirement
- [Modals](#modals) - Background blur, close behavior

**Best Practices:**
- [Accessibility](#accessibility) - ARIA labels, keyboard navigation
- [Loading States](#loading-states) - User feedback during operations
- [Form Validation](#form-validation) - Error messages and inline feedback
- [Color Schemes](#color-schemes) - Light mode enforcement

## Buttons and Links

**Required:** All clickable elements must use the `cursor-pointer` Tailwind class.

**Purpose:** Makes it visually clear to users that an element is interactive.

**Examples:**

```jsx
// Button
<button className="cursor-pointer bg-blue-500 text-white px-4 py-2 rounded">
  Click Me
</button>

// Link
<a href="/dashboard" className="cursor-pointer text-blue-600 hover:underline">
  Go to Dashboard
</a>

// Custom clickable div
<div onClick={handleClick} className="cursor-pointer p-4 hover:bg-gray-100">
  Clickable Card
</div>
```

**Don'ts:**
- ❌ `<button className="bg-blue-500">Click</button>` - Missing cursor-pointer
- ❌ `<div onClick={handler}>Click</div>` - Missing cursor-pointer

## Modals

**Required features:**
1. **Blurred background** - Use backdrop-blur Tailwind class
2. **Close on Escape key** - Keyboard accessibility
3. **Close on outside click** - Click overlay to dismiss

**Example implementation:**

```jsx
'use client';

import { useEffect } from 'react';

export default function Modal({ isOpen, onClose, children }) {
  // Close on Escape key
  useEffect(() => {
    const handleEscape = (e) => {
      if (e.key === 'Escape') onClose();
    };
    
    if (isOpen) {
      document.addEventListener('keydown', handleEscape);
      return () => document.removeEventListener('keydown', handleEscape);
    }
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div
      className="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center z-50"
      onClick={onClose} // Close on outside click
    >
      <div
        className="bg-white rounded-lg p-6 max-w-lg w-full mx-4"
        onClick={(e) => e.stopPropagation()} // Prevent close when clicking modal content
      >
        {children}
      </div>
    </div>
  );
}
```

**Key Tailwind classes:**
- `backdrop-blur-sm` - Blurs background content
- `bg-black/50` - Semi-transparent overlay
- `fixed inset-0` - Full-screen coverage
- `z-50` - Ensures modal appears above other content

## Accessibility

**Keyboard navigation:**
- All interactive elements should be keyboard-accessible
- Modals close on Escape key
- Tab order should be logical

**ARIA labels:**
- Add descriptive labels for screen readers
- Use `aria-label` or `aria-labelledby` for icon-only buttons

**Example:**
```jsx
<button
  aria-label="Close modal"
  className="cursor-pointer"
  onClick={onClose}
>
  <XIcon className="h-6 w-6" />
</button>
```

## Loading States

**Always provide feedback during async operations:**

```jsx
<button
  disabled={isLoading}
  className="cursor-pointer bg-blue-500 text-white px-4 py-2 rounded disabled:opacity-50 disabled:cursor-not-allowed"
>
  {isLoading ? 'Saving...' : 'Save'}
</button>
```

**Spinner component example:**
```jsx
{isLoading && (
  <div className="flex items-center justify-center">
    <div className="animate-spin h-8 w-8 border-4 border-blue-500 border-t-transparent rounded-full"></div>
  </div>
)}
```

## Form Validation

**Inline error messages:**
- Show errors below the relevant input field
- Use red text and borders for error states
- Clear errors when user corrects the input

**Example:**
```jsx
<div>
  <input
    type="email"
    className={`px-3 py-2 border rounded ${
      error ? 'border-red-500' : 'border-gray-300'
    }`}
    value={email}
    onChange={(e) => setEmail(e.target.value)}
  />
  {error && <p className="text-red-500 text-sm mt-1">{error}</p>}
</div>
```

## Color Schemes

**Required:** Use CSS to enforce light-mode styling even when users have dark mode enabled in Chrome/browser.

**Purpose:** Ensures consistent visual appearance regardless of user's system preferences.

**Implementation:**

```css
/* In your global CSS or Tailwind config */
@media (prefers-color-scheme: dark) {
  :root {
    color-scheme: light;
  }
}
```

**Or with Tailwind:**
```jsx
// Wrap your app or layout
<div className="bg-white text-black">
  {/* Your app content */}
</div>
```

**Next.js global CSS example:**
```css
/* globals.css */
html,
body {
  color-scheme: light;
  background-color: white;
  color: black;
}

/* Override system dark mode */
@media (prefers-color-scheme: dark) {
  html,
  body {
    color-scheme: light;
    background-color: white;
    color: black;
  }
}
```

## Best Practices Summary

**✅ DO:**
- Use `cursor-pointer` on all clickable elements
- Implement proper modal close behavior (Escape, outside click)
- Blur modal backgrounds for visual hierarchy
- Provide loading states for async operations
- Show inline validation errors
- Follow keyboard navigation standards
- Enforce light-mode styling even when users have dark mode enabled

**❌ DON'T:**
- Create clickable elements without cursor-pointer class
- Build modals that can't be dismissed with Escape
- Leave users wondering if their action is processing
- Hide error messages or show them in unclear locations
