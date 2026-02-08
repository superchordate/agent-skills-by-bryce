---
name: nextjs-testing
description: Guide for setting up and writing Jest tests for Next.js applications. Use when setting up testing infrastructure, creating test files, mocking Next.js features, or writing unit and integration tests for Server/Client Components, API routes, and hooks. Covers Jest setup, React Testing Library best practices, and Next.js-specific testing patterns.
---

# Next.js Testing with Jest

Comprehensive guide for testing Next.js applications using Jest and React Testing Library.

## Quick Reference

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

**Jest config template:** → [See Setup](#jest-configuration)  
**Mocking strategies:** → [See Mocking](#mocking-next-js-features)  
**Testing patterns:** → [See Patterns](#testing-patterns)  
**Best practices:** → [See Best Practices](#best-practices)

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
