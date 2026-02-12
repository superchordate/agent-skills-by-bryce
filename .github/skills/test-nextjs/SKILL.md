---
name: nextjs-testing
description: Guide for setting up and writing Jest tests for Next.js applications. Use when setting up testing infrastructure, creating test files, mocking Next.js features (next/navigation, next/link, next/image), writing unit and integration tests for Server/Client Components, API routes, hooks, or debugging test failures. Covers Jest setup, React Testing Library best practices, Next.js-specific testing patterns, and test command documentation in source files.
---

# Next.js Testing with Jest

Comprehensive guide for testing Next.js applications using Jest and React Testing Library.

## Quick Reference

**CRITICAL: Document test commands in covered files** - When creating or updating tests, add a comment at the top of each tested file indicating how to run its tests. → [See Test Command Comments](#test-command-comments-in-source-files)

**Navigation:**
- **Test Documentation** → [Test Command Comments](#test-command-comments-in-source-files)
- **Setup & Configuration** → [Jest Setup](#setup), [Configuration](#jest-configuration), [Module Aliases](#module-path-aliases)
- **Testing React Components** → [Server Components](#test-server-components), [Client Components](#test-client-components), [Async Components](#test-async-components)
- **Testing Utilities & JS** → [Utility Functions](#testing-utility-functions-and-plain-javascript), [Pure Functions](#testing-pure-functions), [Helper Modules](#testing-helper-modules)
- **Mocking** → [Next.js Features](#mocking-next-js-features), [Navigation](#mock-nextnavigation), [Link](#mock-nextlink), [Image](#mock-nextimage), [Firestore](#mock-firestore), [API Routes](#mock-api-routesserver-actions), [Context](#mock-context-providers)
- **Patterns** → [Multiple States](#test-multiple-states), [User Flows](#test-user-flows), [Accessibility](#test-accessibility), [Custom Render](#custom-render-function)
- **Best Practices** → [Query Priorities](#query-priorities-react-testing-library), [Text Matching](#flexible-text-matching), [File Organization](#test-file-organization)
- **Troubleshooting** → [Common Issues](#troubleshooting)

**Setup command:**
```bash
npm install -D jest jest-environment-jsdom @testing-library/react @testing-library/dom @testing-library/jest-dom ts-node @types/jest
npm init jest@latest
```

**Test file structure:**
```typescript
import '@testing-library/jest-dom'
import { render, screen } from '@testing-library/react'
import Page from '../app/page'
 
describe('Page', () => {
  it('renders a heading', () => {
    render(<Page />)
    const heading = screen.getByRole('heading', { level: 1 })
    expect(heading).toBeInTheDocument()
  })
})
```

**Running tests:**
```bash
npm test                    # Run all tests
npm test -- ComponentName   # Run specific test
npm test -- --watch         # Watch mode
npm test -- --coverage      # Coverage report
```

---

## Test Command Comments in Source Files

**CRITICAL:** Every file covered by tests must include a comment at the top documenting how to run its tests. This makes it immediately clear what tests cover a given file and how to execute them.

### Purpose

Test command comments provide:
- **Discoverability** - Developers know tests exist for this file
- **Execution clarity** - Exact command to run relevant tests
- **Maintenance** - Easy to find and update tests when modifying code
- **Documentation** - Self-documenting test coverage

### Format

Add a comment block at the top of tested files (after imports if necessary):

**For TypeScript/JavaScript files:**
```typescript
/**
 * Tests: npm test -- Page.test
 * Coverage: npm test -- --coverage Page.test
 */

import React from 'react'
// ... rest of file
```

**For components with specific test files:**
```typescript
/**
 * Tests: npm test -- Button
 * Test file: __tests__/components/Button.test.tsx
 */

export function Button({ onClick, children }) {
  // ... component code
}
```

**For files with multiple related tests:**
```typescript
/**
 * Tests: 
 *   - Unit: npm test -- useAuth.test
 *   - Integration: npm test -- auth-flow.test
 */

export function useAuth() {
  // ... hook code
}
```

### When to Add Comments

Add test command comments when:
- Creating a new test file for an existing source file
- Creating a new source file with tests
- Moving or renaming test files
- Changing test naming patterns

### Location

Place comments:
1. **Top of file** - Immediately after file header/license (if present)
2. **After imports** - If imports must come first for technical reasons
3. **Before main export** - For files with multiple exports

### Examples

**Page component:**
```typescript
/**
 * Tests: npm test -- page.test
 */

import { Metadata } from 'next'
import { PageContent } from './components/PageContent'

export const metadata: Metadata = {
  title: 'Home Page',
}

export default function Page() {
  return <PageContent />
}
```

**Custom hook:**
```typescript
/**
 * Tests: npm test -- useCustomHook.test
 * Coverage: npm test -- --coverage useCustomHook
 */

import { useState, useEffect } from 'react'

export function useCustomHook(initialValue: string) {
  const [value, setValue] = useState(initialValue)
  // ... hook implementation
}
```

**API route:**
```typescript
/**
 * Tests: npm test -- api/users
 * Integration tests: npm test -- api-integration
 */

import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  // ... API implementation
}
```

**Utility function:**
```typescript
/**
 * Tests: npm test -- formatters.test
 * Test file: __tests__/lib/formatters.test.ts
 */

export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(amount)
}
```

### Best Practices

1. **Keep commands copy-paste ready** - Use exact commands that work from project root
2. **Update when tests move** - If test file location changes, update comment
3. **Include test file path** - When test location is not obvious from command
4. **Document multiple test types** - Note unit, integration, and E2E tests separately
5. **Use consistent format** - Follow the same comment pattern across the project

### Automation Reminder

When creating or updating tests, always:
1. Create/update the test file
2. Add/update test command comment in source file
3. Verify command works from project root
4. Document any special test setup requirements

---

## Setup

### Jest Configuration

Create `jest.config.ts` with Next.js integration:

```typescript
import type { Config } from 'jest'
import nextJest from 'next/jest'
 
const createJestConfig = nextJest({
  // Provide the path to your Next.js app to load next.config.js and .env files
  dir: './',
})
 
const config: Config = {
  coverageProvider: 'v8',
  testEnvironment: 'jsdom',
  // Optional: Add setup file for custom matchers
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
}
 
export default createJestConfig(config)
```

### Setup File (Optional)

Create `jest.setup.ts` for custom matchers:

```typescript
import '@testing-library/jest-dom'
```

### Module Path Aliases

If using path aliases in `tsconfig.json`, configure Jest:

```typescript
// jest.config.ts
{
  moduleNameMapper: {
    '^@/components/(.*)$': '<rootDir>/components/$1',
    '^@/lib/(.*)$': '<rootDir>/lib/$1',
  }
}
```

### Package.json Scripts

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

---

## Mocking Next.js Features

### Mock next/navigation

```typescript
jest.mock('next/navigation', () => ({
  useRouter() {
    return {
      push: jest.fn(),
      replace: jest.fn(),
      prefetch: jest.fn(),
      back: jest.fn(),
      pathname: '/',
      query: {},
    }
  },
  usePathname() {
    return '/'
  },
  useSearchParams() {
    return new URLSearchParams()
  },
}))
```

### Mock next/link

```typescript
jest.mock('next/link', () => {
  return ({ children, href }: { children: React.ReactNode; href: string }) => {
    return <a href={href}>{children}</a>
  }
})
```

### Mock next/image

```typescript
jest.mock('next/image', () => ({
  __esModule: true,
  default: (props: any) => {
    // eslint-disable-next-line jsx-a11y/alt-text
    return <img {...props} />
  },
}))
```

### Mock Environment Variables

```typescript
// jest.setup.ts
process.env.NEXT_PUBLIC_API_URL = 'http://localhost:3000'
process.env.API_KEY = 'test-key'
```

### Mock API Routes/Server Actions

```typescript
// Mock fetch for API routes
global.fetch = jest.fn(() =>
  Promise.resolve({
    json: () => Promise.resolve({ data: 'test' }),
    ok: true,
    status: 200,
  })
) as jest.Mock

// Mock server action
jest.mock('@/app/actions', () => ({
  myServerAction: jest.fn(async () => ({ success: true })),
}))
```

### Mock Custom Hooks

```typescript
jest.mock('@/hooks/useCustomHook', () => ({
  useCustomHook: jest.fn(() => ({
    data: mockData,
    loading: false,
    error: null,
  })),
}))
```

### Mock Context Providers

```typescript
// Create a test wrapper
const MockProvider = ({ children }: { children: React.ReactNode }) => (
  <MyContext.Provider value={mockContextValue}>
    {children}
  </MyContext.Provider>
)

// Use in tests
render(<Component />, { wrapper: MockProvider })
```

### Mock Firestore

**Option 1: Mock firebase/firestore module**

```typescript
// jest.setup.ts or at top of test file
jest.mock('firebase/firestore', () => ({
  getFirestore: jest.fn(),
  collection: jest.fn(),
  doc: jest.fn(),
  getDoc: jest.fn(() => 
    Promise.resolve({
      exists: () => true,
      data: () => ({ id: '123', name: 'Test User' }),
      id: '123',
    })
  ),
  getDocs: jest.fn(() =>
    Promise.resolve({
      docs: [
        {
          id: '1',
          data: () => ({ name: 'Item 1' }),
        },
        {
          id: '2',
          data: () => ({ name: 'Item 2' }),
        },
      ],
      empty: false,
      size: 2,
    })
  ),
  setDoc: jest.fn(() => Promise.resolve()),
  updateDoc: jest.fn(() => Promise.resolve()),
  deleteDoc: jest.fn(() => Promise.resolve()),
  addDoc: jest.fn(() => Promise.resolve({ id: 'new-id' })),
  query: jest.fn((ref) => ref),
  where: jest.fn(),
  orderBy: jest.fn(),
  limit: jest.fn(),
}))
```

**Option 2: Mock Firestore service/helper**

If you have a Firestore service wrapper:

```typescript
// Mock the service
jest.mock('@/lib/firestore', () => ({
  getUserById: jest.fn(async (id: string) => ({
    id,
    name: 'Test User',
    email: 'test@example.com',
  })),
  createUser: jest.fn(async (data) => ({
    id: 'new-user-id',
    ...data,
  })),
  updateUser: jest.fn(async (id, data) => ({ id, ...data })),
  deleteUser: jest.fn(async () => true),
}))

// In tests
import { getUserById } from '@/lib/firestore'

it('fetches user data', async () => {
  const user = await getUserById('123')
  expect(user.name).toBe('Test User')
  expect(getUserById).toHaveBeenCalledWith('123')
})
```

**Option 3: Mock with custom return values per test**

```typescript
import { getDoc } from 'firebase/firestore'

// Mock at top of test file
jest.mock('firebase/firestore')

describe('User component', () => {
  it('shows user when found', async () => {
    // Custom mock for this test
    (getDoc as jest.Mock).mockResolvedValueOnce({
      exists: () => true,
      data: () => ({ name: 'John Doe', email: 'john@example.com' }),
      id: 'user-123',
    })

    render(<UserProfile userId="user-123" />)
    
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument()
    })
  })

  it('shows error when user not found', async () => {
    (getDoc as jest.Mock).mockResolvedValueOnce({
      exists: () => false,
      data: () => null,
    })

    render(<UserProfile userId="invalid" />)
    
    await waitFor(() => {
      expect(screen.getByText('User not found')).toBeInTheDocument()
    })
  })
})
```

**Option 4: Firebase Testing Library (for integration tests)**

For more comprehensive Firestore testing:

```bash
npm install -D @firebase/rules-unit-testing
```

```typescript
import {
  initializeTestEnvironment,
  assertFails,
  assertSucceeds,
} from '@firebase/rules-unit-testing'

let testEnv: any

beforeAll(async () => {
  testEnv = await initializeTestEnvironment({
    projectId: 'test-project',
    firestore: {
      host: 'localhost',
      port: 8080,
    },
  })
})

afterAll(async () => {
  await testEnv.cleanup()
})

it('allows authenticated user to read their data', async () => {
  const context = testEnv.authenticatedContext('user-123')
  const docRef = context.firestore().collection('users').doc('user-123')
  
  await assertSucceeds(docRef.get())
})
```

**Best practices for Firestore mocks:**
- **Mock at service layer** when possible for cleaner tests
- **Use type-safe mocks** - Cast to proper types to catch errors
- **Reset mocks** between tests with `jest.clearAllMocks()`
- **Test error cases** - Mock rejected promises for error scenarios
- **Consider integration tests** - Use Firebase emulator for complex queries

---

## Testing Patterns

### Test Server Components

```typescript
import { render, screen } from '@testing-library/react'
import ServerComponent from '@/app/components/ServerComponent'

describe('ServerComponent', () => {
  it('renders server-fetched data', () => {
    render(<ServerComponent />)
    expect(screen.getByText('Expected Text')).toBeInTheDocument()
  })
})
```

### Test Client Components

```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import ClientComponent from '@/app/components/ClientComponent'

describe('ClientComponent', () => {
  it('handles user interaction', () => {
    render(<ClientComponent />)
    const button = screen.getByRole('button')
    fireEvent.click(button)
    expect(screen.getByText('Clicked')).toBeInTheDocument()
  })
})
```

### Test Multiple States

```typescript
it('shows loading state', () => {
  const useData = require('@/hooks/useData').useData
  useData.mockReturnValueOnce({ data: null, loading: true, error: null })
  
  render(<Component />)
  expect(screen.getByText('Loading...')).toBeInTheDocument()
})

it('shows error state', () => {
  const useData = require('@/hooks/useData').useData
  useData.mockReturnValueOnce({ data: null, loading: false, error: 'Error' })
  
  render(<Component />)
  expect(screen.getByText('Error')).toBeInTheDocument()
})

it('shows data state', () => {
  const useData = require('@/hooks/useData').useData
  useData.mockReturnValueOnce({ data: mockData, loading: false, error: null })
  
  render(<Component />)
  expect(screen.getByText('Data loaded')).toBeInTheDocument()
})
```

### Test Async Components

```typescript
import { render, screen, waitFor } from '@testing-library/react'

describe('AsyncComponent', () => {
  it('renders after async operation', async () => {
    render(<AsyncComponent />)
    await waitFor(() => {
      expect(screen.getByText('Async Data')).toBeInTheDocument()
    })
  })
})
```

### Snapshot Testing

```typescript
it('renders unchanged', () => {
  const { container } = render(<Page />)
  expect(container).toMatchSnapshot()
})
```

---

## Testing Utility Functions and Plain JavaScript

**Key difference:** Utility functions and plain JavaScript don't need React Testing Library. Use Jest directly for cleaner, faster tests.

### Testing Pure Functions

Pure functions (no side effects, same input = same output) are the easiest to test:

```typescript
// lib/formatters.ts
/**
 * Tests: npm test -- formatters.test
 */

export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(amount)
}

export function slugify(text: string): string {
  return text
    .toLowerCase()
    .replace(/[^\w\s-]/g, '')
    .replace(/[\s_-]+/g, '-')
    .replace(/^-+|-+$/g, '')
}
```

```typescript
// __tests__/lib/formatters.test.ts
import { formatCurrency, slugify } from '@/lib/formatters'

describe('formatCurrency', () => {
  it('formats positive numbers', () => {
    expect(formatCurrency(1234.56)).toBe('$1,234.56')
  })

  it('formats negative numbers', () => {
    expect(formatCurrency(-100)).toBe('-$100.00')
  })

  it('formats zero', () => {
    expect(formatCurrency(0)).toBe('$0.00')
  })
})

describe('slugify', () => {
  it('converts text to slug', () => {
    expect(slugify('Hello World')).toBe('hello-world')
  })

  it('removes special characters', () => {
    expect(slugify('Hello! World?')).toBe('hello-world')
  })

  it('handles multiple spaces', () => {
    expect(slugify('Hello   World')).toBe('hello-world')
  })
})
```

### Testing Helper Modules

Helper modules that don't interact with DOM or React:

```typescript
// lib/validation.ts
/**
 * Tests: npm test -- validation.test
 */

export function isValidEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return emailRegex.test(email)
}

export function isStrongPassword(password: string): boolean {
  return (
    password.length >= 8 &&
    /[A-Z]/.test(password) &&
    /[a-z]/.test(password) &&
    /[0-9]/.test(password)
  )
}
```

```typescript
// __tests__/lib/validation.test.ts
import { isValidEmail, isStrongPassword } from '@/lib/validation'

describe('isValidEmail', () => {
  it('validates correct emails', () => {
    expect(isValidEmail('user@example.com')).toBe(true)
    expect(isValidEmail('test.email+tag@domain.co.uk')).toBe(true)
  })

  it('rejects invalid emails', () => {
    expect(isValidEmail('notanemail')).toBe(false)
    expect(isValidEmail('@example.com')).toBe(false)
    expect(isValidEmail('user@')).toBe(false)
  })
})

describe('isStrongPassword', () => {
  it('validates strong passwords', () => {
    expect(isStrongPassword('Password123')).toBe(true)
  })

  it('rejects weak passwords', () => {
    expect(isStrongPassword('short')).toBe(false)
    expect(isStrongPassword('alllowercase123')).toBe(false)
    expect(isStrongPassword('NoNumbers')).toBe(false)
  })
})
```

### Testing Classes

```typescript
// lib/Calculator.ts
/**
 * Tests: npm test -- Calculator.test
 */

export class Calculator {
  private result: number = 0

  add(value: number): Calculator {
    this.result += value
    return this
  }

  subtract(value: number): Calculator {
    this.result -= value
    return this
  }

  getResult(): number {
    return this.result
  }

  reset(): void {
    this.result = 0
  }
}
```

```typescript
// __tests__/lib/Calculator.test.ts
import { Calculator } from '@/lib/Calculator'

describe('Calculator', () => {
  let calculator: Calculator

  beforeEach(() => {
    calculator = new Calculator()
  })

  it('starts at zero', () => {
    expect(calculator.getResult()).toBe(0)
  })

  it('adds numbers', () => {
    calculator.add(5).add(3)
    expect(calculator.getResult()).toBe(8)
  })

  it('subtracts numbers', () => {
    calculator.add(10).subtract(3)
    expect(calculator.getResult()).toBe(7)
  })

  it('resets to zero', () => {
    calculator.add(5)
    calculator.reset()
    expect(calculator.getResult()).toBe(0)
  })
})
```

### Testing Async Utilities

```typescript
// lib/api.ts
/**
 * Tests: npm test -- api.test
 */

export async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`)
  if (!response.ok) {
    throw new Error('User not found')
  }
  return response.json()
}
```

```typescript
// __tests__/lib/api.test.ts
import { fetchUser } from '@/lib/api'

// Mock fetch globally
global.fetch = jest.fn()

describe('fetchUser', () => {
  beforeEach(() => {
    jest.clearAllMocks()
  })

  it('fetches user successfully', async () => {
    const mockUser = { id: '123', name: 'John Doe' }
    
    ;(global.fetch as jest.Mock).mockResolvedValueOnce({
      ok: true,
      json: async () => mockUser,
    })

    const user = await fetchUser('123')
    
    expect(user).toEqual(mockUser)
    expect(global.fetch).toHaveBeenCalledWith('/api/users/123')
  })

  it('throws error when user not found', async () => {
    ;(global.fetch as jest.Mock).mockResolvedValueOnce({
      ok: false,
    })

    await expect(fetchUser('invalid')).rejects.toThrow('User not found')
  })
})
```

### Testing Modules with Dependencies

Use Jest's mocking for external dependencies:

```typescript
// lib/storage.ts
/**
 * Tests: npm test -- storage.test
 */

import fs from 'fs/promises'

export async function saveToFile(filename: string, data: string): Promise<void> {
  await fs.writeFile(filename, data, 'utf-8')
}

export async function readFromFile(filename: string): Promise<string> {
  return await fs.readFile(filename, 'utf-8')
}
```

```typescript
// __tests__/lib/storage.test.ts
import { saveToFile, readFromFile } from '@/lib/storage'
import fs from 'fs/promises'

jest.mock('fs/promises')

describe('storage', () => {
  beforeEach(() => {
    jest.clearAllMocks()
  })

  describe('saveToFile', () => {
    it('writes data to file', async () => {
      await saveToFile('test.txt', 'Hello World')
      
      expect(fs.writeFile).toHaveBeenCalledWith(
        'test.txt',
        'Hello World',
        'utf-8'
      )
    })
  })

  describe('readFromFile', () => {
    it('reads data from file', async () => {
      ;(fs.readFile as jest.Mock).mockResolvedValue('File content')
      
      const content = await readFromFile('test.txt')
      
      expect(content).toBe('File content')
      expect(fs.readFile).toHaveBeenCalledWith('test.txt', 'utf-8')
    })
  })
})
```

### When to Use Jest vs React Testing Library

**Use Jest alone for:**
- ✅ Pure functions (formatters, validators, calculators)
- ✅ Utility classes
- ✅ API clients (non-React)
- ✅ Business logic
- ✅ Data transformations
- ✅ Node.js utilities

**Use React Testing Library for:**
- ✅ React components
- ✅ Custom hooks
- ✅ Components with user interaction
- ✅ Components with context/providers

**Benefits of testing utilities separately:**
- **Faster tests** - No React rendering overhead
- **Simpler setup** - No need for render, screen, etc.
- **Clearer intent** - Tests are about logic, not UI
- **Better isolation** - Test pure logic independently

---

## Best Practices

### Query Priorities (React Testing Library)

1. **Accessible queries (preferred):**
   - `getByRole` - Most robust
   - `getByLabelText` - Form elements
   - `getByPlaceholderText` - Form fallback
   - `getByText` - Non-interactive elements

2. **Semantic queries:**
   - `getByAltText` - Images
   - `getByTitle` - Tooltips

3. **Test IDs (last resort):**
   - `getByTestId` - Use when accessibility queries aren't possible

### Use data-testid Sparingly

```typescript
// Prefer this:
screen.getByRole('button', { name: /submit/i })

// Over this:
screen.getByTestId('submit-button')

// But use testId when needed:
<div data-testid="complex-component">
```

### Handle Duplicate Elements

```typescript
// Multiple elements with same text
const buttons = screen.getAllByRole('button')
expect(buttons).toHaveLength(3)

// Or be more specific
screen.getByRole('button', { name: 'Submit' })
```

### Flexible Text Matching

```typescript
// Case-insensitive regex
screen.getByText(/hello world/i)

// Partial match
screen.getByText(/hello/)

// Function matcher
screen.getByText((content, element) => {
  return element?.tagName.toLowerCase() === 'span' && content.startsWith('Hello')
})
```

### Test User Flows

```typescript
import userEvent from '@testing-library/user-event'

it('completes form submission flow', async () => {
  const user = userEvent.setup()
  render(<Form />)
  
  await user.type(screen.getByLabelText('Name'), 'John')
  await user.type(screen.getByLabelText('Email'), 'john@example.com')
  await user.click(screen.getByRole('button', { name: /submit/i }))
  
  await waitFor(() => {
    expect(screen.getByText('Success')).toBeInTheDocument()
  })
})
```

### Test Accessibility

```typescript
import { axe, toHaveNoViolations } from 'jest-axe'
expect.extend(toHaveNoViolations)

it('has no accessibility violations', async () => {
  const { container } = render(<Component />)
  const results = await axe(container)
  expect(results).toHaveNoViolations()
})
```

### Cleanup

React Testing Library automatically cleans up after each test. No manual cleanup needed.

---

## Common Patterns

### Test File Organization

**IMPORTANT:** When creating tests in any of these structures, always add test command comments to the source files being tested. → [See Test Command Comments](#test-command-comments-in-source-files)

```
__tests__/
  components/
    Button.test.tsx
    Header.test.tsx
  hooks/
    useAuth.test.ts
  pages/
    home.test.tsx
```

Or colocate with source:

```
components/
  Button.tsx
  Button.test.tsx
```

### Test Structure

```typescript
describe('ComponentName', () => {
  // Group related tests
  describe('rendering', () => {
    it('renders with default props', () => {})
    it('renders with custom props', () => {})
  })
  
  describe('interaction', () => {
    it('calls onClick when clicked', () => {})
  })
  
  describe('edge cases', () => {
    it('handles empty data', () => {})
    it('handles errors', () => {})
  })
})
```

### Custom Render Function

```typescript
// test-utils.tsx
import { render } from '@testing-library/react'
import { ThemeProvider } from '@/context/theme'

export function renderWithProviders(ui: React.ReactElement, options = {}) {
  return render(ui, {
    wrapper: ({ children }) => (
      <ThemeProvider>
        {children}
      </ThemeProvider>
    ),
    ...options,
  })
}

// In tests
import { renderWithProviders } from '@/test-utils'
renderWithProviders(<Component />)
```

---

## Troubleshooting

### "Cannot find module 'next/...' "

Ensure `next/jest` is properly configured in `jest.config.ts`.

### "TextEncoder is not defined"

Add to `jest.setup.ts`:
```typescript
import { TextEncoder, TextDecoder } from 'util'
global.TextEncoder = TextEncoder
global.TextDecoder = TextDecoder as any
```

### Async Server Components Not Supported

Use E2E testing (Playwright, Cypress) for async Server Components. Jest supports synchronous components only.

### CSS/Image Import Errors

`next/jest` automatically handles these. If issues persist, check your `jest.config.ts` setup.

---

## Resources

- [Next.js Testing Docs](https://nextjs.org/docs/app/building-your-application/testing/jest)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Testing Playground](https://testing-playground.com/)
