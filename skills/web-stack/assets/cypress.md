# Cypress Best Practices for Next.js App Router

*Research completed: May 12, 2026*

This guide covers best practices for setting up and running Cypress tests with Next.js App Router (13+), focusing on performance optimization and Next.js-specific considerations.

## Table of Contents
- [Setup](#setup)
- [Adding Tests](#adding-tests)
- [Running Tests](#running-tests)
- [Performance Optimization](#performance-optimization)
- [Next.js Specific Considerations](#nextjs-specific-considerations)

---

## Setup

### Installation

Install Cypress as a dev dependency:

```bash
npm install -D cypress
# or
yarn add -D cypress
# or
pnpm add -D cypress
```

**System Requirements** (as of 2026):
- Node.js: 20.x, 22.x, >=24.x
- Browsers: Latest 3 major versions of Chrome, Edge, Firefox
- Cypress: 13.6.3+ (for TypeScript 5 support)

### Initial Configuration

Run Cypress for the first time to create configuration files:

```bash
npx cypress open
```

This creates `cypress.config.js` (or `.ts`) and the `cypress` folder structure.

### Next.js-Specific Configuration

**cypress.config.js (E2E Testing):**

```javascript
const { defineConfig } = require('cypress')

module.exports = defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    setupNodeEvents(on, config) {
      // implement node event listeners here
    },
    // Performance optimizations
    video: false,  // Disable video recording for faster tests
    screenshotOnRunFailure: true,
    // Reduce memory usage
    numTestsKeptInMemory: 0,  // Set to 0 during cypress run
  },
})
```

**cypress.config.js (Component Testing):**

```javascript
const { defineConfig } = require('cypress')

module.exports = defineConfig({
  component: {
    devServer: {
      framework: 'next',
      bundler: 'webpack', // or 'turbopack' for Next.js 16+
    },
    // Component tests don't require a running server
    specPattern: '**/*.cy.{js,jsx,ts,tsx}',
  },
})
```

**Key Configuration Options:**
- `baseUrl`: Set to your dev server URL (saves typing and enables automatic server verification)
- `video`: Set to `false` locally to speed up test runs
- `numTestsKeptInMemory`: Set to `0` during `cypress run` to reduce memory usage

### package.json Scripts

Add helpful scripts to your `package.json`:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    
    // Cypress E2E
    "cypress:open": "cypress open",
    "cypress:run": "cypress run",
    "cypress:run:headed": "cypress run --headed",
    
    // E2E with server (recommended)
    "test:e2e": "start-server-and-test dev http://localhost:3000 cypress:open",
    "test:e2e:ci": "start-server-and-test start http://localhost:3000 'cypress run'",
    
    // Component testing
    "cypress:component": "cypress open --component",
    "test:component": "cypress run --component"
  }
}
```

**Install start-server-and-test:**

```bash
npm install -D start-server-and-test
```

This package starts your Next.js server, waits for it to be ready, then runs Cypress tests.

---

## Adding Tests

### Folder Structure

```
project-root/
├── cypress/
│   ├── e2e/              # E2E test specs
│   │   └── home.cy.js
│   ├── component/        # Component test specs
│   │   └── Button.cy.tsx
│   ├── fixtures/         # Test data
│   │   └── users.json
│   ├── support/          # Custom commands, utilities
│   │   ├── commands.js
│   │   └── e2e.js
│   └── downloads/        # Downloaded files during tests
├── cypress.config.js
└── package.json
```

### Writing E2E Tests

**Basic E2E Test Example:**

```javascript
// cypress/e2e/home.cy.js
describe('Home Page', () => {
  beforeEach(() => {
    cy.visit('/')
  })

  it('displays the main heading', () => {
    cy.contains('h1', 'Welcome').should('be.visible')
  })

  it('navigates to about page', () => {
    cy.get('a[href*="about"]').click()
    cy.url().should('include', '/about')
    cy.contains('h1', 'About').should('be.visible')
  })
})
```

**Best Practices for E2E Tests:**
- Use `baseUrl` in config, then `cy.visit('/')` instead of full URLs
- Add `data-cy`, `data-testid`, or `data-test` attributes for reliable selectors
- Avoid coupling to CSS classes or IDs that may change
- Wait for elements to be visible before interacting: `.should('be.visible')`

### Writing Component Tests

**Next.js App Router Component Example:**

```typescript
// app/components/Button.tsx
export default function Button({ onClick, children }) {
  return (
    <button onClick={onClick} data-cy="button">
      {children}
    </button>
  )
}
```

**Component Test:**

```typescript
// cypress/component/Button.cy.tsx
import Button from '../../app/components/Button'

describe('<Button />', () => {
  it('mounts and displays text', () => {
    cy.mount(<Button>Click Me</Button>)
    cy.get('[data-cy="button"]').should('contain', 'Click Me')
  })

  it('calls onClick when clicked', () => {
    const onClickSpy = cy.spy().as('clickSpy')
    cy.mount(<Button onClick={onClickSpy}>Click Me</Button>)
    
    cy.get('[data-cy="button"]').click()
    cy.get('@clickSpy').should('have.been.calledOnce')
  })
})
```

**Important Notes for Component Testing:**
- Component tests do NOT support async Server Components (use E2E testing instead)
- No Next.js server needed - components mount in isolation
- Features like `<Image />` may not work (they rely on server)

### Selector Best Practices

**Priority Order:**

1. **Best:** `data-cy`, `data-testid`, or `data-test` attributes
   ```javascript
   cy.get('[data-cy="submit-button"]')
   ```

2. **Good:** Semantic selectors with specific context
   ```javascript
   cy.contains('form button', 'Submit')
   ```

3. **Acceptable:** Accessible roles and labels
   ```javascript
   cy.findByRole('button', { name: 'Submit' })
   ```

4. **Avoid:** CSS classes, IDs, or element tags alone
   ```javascript
   // ❌ Bad - coupled to styling
   cy.get('.btn-primary')
   // ❌ Bad - too generic
   cy.get('button')
   ```

---

## Running Tests

### Running Single Spec (Fastest)

**Run a single spec file** for rapid feedback during development:

```bash
# Run specific E2E spec
npx cypress run --spec "cypress/e2e/home.cy.js"

# Run specific component spec
npx cypress run --component --spec "cypress/component/Button.cy.tsx"

# Run all specs in a folder
npx cypress run --spec "cypress/e2e/auth/**/*"
```

**Why single spec is faster:**
- Cypress opens/closes browser per spec file
- Running one spec minimizes browser lifecycle overhead
- Faster feedback loop during development

### Headless vs Headed Mode

**Headless (default for `cypress run`):**
```bash
npx cypress run --headless
```
- Faster execution
- Better for CI/CD
- No visible browser window

**Headed (shows browser):**
```bash
npx cypress run --headed
```
- Useful for debugging
- See what's happening in real-time
- Slower than headless

### Interactive Mode (`cypress open`)

```bash
npx cypress open
```
- Best for writing and debugging tests
- Hot-reload on file changes
- Visual test runner with time-travel debugging
- Choose browser and spec interactively

### Running with Different Browsers

```bash
# Chrome (default)
npx cypress run --browser chrome

# Firefox
npx cypress run --browser firefox

# Edge
npx cypress run --browser edge

# Electron (bundled)
npx cypress run --browser electron
```

### CI/CD Optimized Commands

```bash
# Production build with all optimizations
npx cypress run \
  --browser chrome \
  --headless \
  --spec "cypress/e2e/**/*.cy.js"
```

**For Parallel Execution (Cypress Cloud):**

```bash
npx cypress run --record --parallel --group "E2E Tests"
```

---

## Performance Optimization

### 1. Disable Video Recording Locally

Videos slow down test execution significantly.

**cypress.config.js:**
```javascript
module.exports = defineConfig({
  e2e: {
    video: false,  // Disable for local dev
    // Keep videos only in CI
  },
})
```

**Or via command line:**
```bash
npx cypress run --config video=false
```

### 2. Reduce Memory Usage

**Set `numTestsKeptInMemory` to 0:**

```javascript
module.exports = defineConfig({
  e2e: {
    numTestsKeptInMemory: 0,  // During cypress run
  },
})
```

**Additional Memory Tips:**
- Run fewer specs per run (split into multiple jobs in CI)
- Use `--headed` flag if Electron keeps crashing
- Expose garbage collection: `ELECTRON_EXTRA_LAUNCH_ARGS=--js-flags=--expose_gc`

### 3. Optimize Test Execution Order

**Run failing tests first** with Spec Prioritization (Cypress Cloud):
- Faster feedback on failures
- Auto-cancel after N failures to save time

**Group tests logically:**
```bash
npx cypress run --group "smoke-tests" --spec "cypress/e2e/critical/**/*"
npx cypress run --group "regression" --spec "cypress/e2e/regression/**/*"
```

### 4. Avoid Unnecessary Waits

**❌ Bad - arbitrary wait:**
```javascript
cy.get('[data-cy="submit"]').click()
cy.wait(5000)  // Arbitrary wait
cy.get('[data-cy="success"]').should('be.visible')
```

**✅ Good - wait for specific condition:**
```javascript
cy.get('[data-cy="submit"]').click()
cy.intercept('POST', '/api/submit').as('submitRequest')
cy.wait('@submitRequest')
cy.get('[data-cy="success"]').should('be.visible')
```

### 5. Use Network Interception Wisely

**Stub external API calls** to speed up tests:

```javascript
beforeEach(() => {
  cy.intercept('GET', '/api/users', {
    fixture: 'users.json'
  })
})
```

### 6. Run Tests in Parallel (CI Only)

**GitHub Actions Example:**

```yaml
# .github/workflows/cypress.yml
name: Cypress Tests

on: [push]

jobs:
  cypress:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        # Run 4 parallel jobs
        containers: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v3
      - uses: cypress-io/github-action@v6
        with:
          build: npm run build
          start: npm start
          wait-on: 'http://localhost:3000'
          record: true
          parallel: true
          group: 'E2E Tests'
        env:
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
```

### 7. Component Tests for Faster Feedback

Component tests are significantly faster than E2E tests:
- No server startup required
- No page loads or navigation
- Isolated component mounting

**Use component tests for:**
- UI components in isolation
- Form validation logic
- Button interactions
- Visual states (loading, error, success)

**Use E2E tests for:**
- Full user workflows
- Navigation between pages
- Server-side data fetching
- Authentication flows

---

## Next.js Specific Considerations

### 1. Set `baseUrl` in Configuration

**cypress.config.js:**
```javascript
module.exports = defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
  },
})
```

**Benefits:**
- Shorter `cy.visit()` calls: `cy.visit('/')` instead of `cy.visit('http://localhost:3000/')`
- Cypress verifies server is running before tests start
- Automatic error if server is down

### 2. Start Server Before Tests

**Use `start-server-and-test` package:**

```json
{
  "scripts": {
    "test:e2e": "start-server-and-test dev 3000 'cypress open'",
    "test:e2e:ci": "start-server-and-test start 3000 'cypress run'"
  }
}
```

**This ensures:**
- Server starts automatically
- Cypress waits for server to be ready
- Server shuts down after tests complete

### 3. Test Against Production Build in CI

```json
{
  "scripts": {
    "test:e2e:ci": "npm run build && start-server-and-test start 3000 'cypress run'"
  }
}
```

**Why production build:**
- Tests behavior closer to what users experience
- Catches production-only issues
- Verifies optimizations don't break functionality

### 4. Handle Next.js App Router Features

#### Server Components

**Cannot test directly with Cypress Component Testing.**

**Solution:** Use E2E tests instead:

```javascript
// ✅ E2E test for async Server Component
describe('User Profile Page', () => {
  it('displays user data from server', () => {
    cy.visit('/user/123')
    cy.contains('h1', 'John Doe').should('be.visible')
    cy.get('[data-cy="email"]').should('contain', 'john@example.com')
  })
})
```

#### Client Components

**Can test with both E2E and Component Testing:**

```typescript
// ✅ Component test for Client Component
'use client'

export default function Counter() {
  const [count, setCount] = useState(0)
  
  return (
    <button onClick={() => setCount(count + 1)} data-cy="counter">
      Count: {count}
    </button>
  )
}

// cypress/component/Counter.cy.tsx
import Counter from '@/app/components/Counter'

describe('<Counter />', () => {
  it('increments count on click', () => {
    cy.mount(<Counter />)
    cy.get('[data-cy="counter"]').should('contain', 'Count: 0')
    cy.get('[data-cy="counter"]').click()
    cy.get('[data-cy="counter"]').should('contain', 'Count: 1')
  })
})
```

#### Server Actions

**Test via E2E (not component tests):**

```javascript
describe('Form with Server Action', () => {
  it('submits form and shows success message', () => {
    cy.visit('/contact')
    cy.get('[data-cy="name"]').type('John Doe')
    cy.get('[data-cy="email"]').type('john@example.com')
    cy.get('[data-cy="submit"]').click()
    
    // Wait for server action to complete
    cy.contains('Thank you for your message').should('be.visible')
  })
})
```

### 5. Environment Variables

**Set Next.js environment variables for tests:**

```javascript
// cypress.config.js
module.exports = defineConfig({
  e2e: {
    env: {
      NEXT_PUBLIC_API_URL: 'http://localhost:3000/api',
    },
    setupNodeEvents(on, config) {
      // Access process.env and set as Cypress env
      config.env.API_KEY = process.env.API_KEY
      return config
    },
  },
})
```

**Use in tests:**
```javascript
cy.request({
  url: Cypress.env('NEXT_PUBLIC_API_URL') + '/users',
  headers: {
    'Authorization': `Bearer ${Cypress.env('API_KEY')}`
  }
})
```

### 6. Handling Next.js Image Optimization

**`<Image />` component may not work in component tests.**

**Solutions:**

1. **Mock the Next.js Image component:**
```javascript
// cypress/support/component.js
import { mount } from 'cypress/react18'

Cypress.Commands.add('mount', (component, options) => {
  // Mock next/image
  const Image = ({ src, alt, ...props }) => (
    <img src={src} alt={alt} {...props} />
  )
  
  return mount(component, {
    ...options,
    // Add mocks here
  })
})
```

2. **Use E2E tests for components with `<Image />`:**
```javascript
describe('Hero Component with Image', () => {
  it('displays optimized image', () => {
    cy.visit('/')
    cy.get('img[alt="Hero image"]').should('be.visible')
    cy.get('img[alt="Hero image"]').should('have.attr', 'src')
      .and('include', '/_next/image')
  })
})
```

### 7. Testing Dynamic Routes

```javascript
// Test dynamic route: /blog/[slug]
describe('Blog Post Page', () => {
  it('displays blog post with slug', () => {
    cy.visit('/blog/my-first-post')
    cy.get('h1').should('contain', 'My First Post')
    cy.get('article').should('be.visible')
  })
  
  it('handles 404 for invalid slug', () => {
    cy.visit('/blog/non-existent', { failOnStatusCode: false })
    cy.contains('404').should('be.visible')
  })
})
```

---

## Quick Reference

### Common Commands

```bash
# Open Cypress Test Runner
npx cypress open

# Run all E2E tests headlessly
npx cypress run

# Run single spec file
npx cypress run --spec "cypress/e2e/home.cy.js"

# Run with specific browser
npx cypress run --browser chrome

# Run with headed browser (visible)
npx cypress run --headed

# Component testing
npx cypress open --component
npx cypress run --component

# Start server and run tests
npm run build && start-server-and-test start 3000 'cypress run'
```

### Configuration Checklist

- [ ] Set `baseUrl` to your dev server URL
- [ ] Disable `video` for local development
- [ ] Set `numTestsKeptInMemory: 0` for CI
- [ ] Add `start-server-and-test` to package.json scripts
- [ ] Configure both E2E and Component testing (if needed)
- [ ] Set up proper environment variables
- [ ] Add `.gitignore` entries for Cypress artifacts

### .gitignore for Cypress

```gitignore
# Cypress
cypress/videos
cypress/screenshots
cypress/downloads
cypress.env.json
```

---

## Resources

- **Official Docs:** https://docs.cypress.io
- **Next.js Testing Guide:** https://nextjs.org/docs/app/building-your-application/testing/cypress
- **Cypress Best Practices:** https://docs.cypress.io/app/core-concepts/best-practices
- **Real World App Example:** https://github.com/cypress-io/cypress-realworld-app
- **Cypress Tips & Tricks:** https://glebbahmutov.com/blog/cypress-tips-and-tricks/

---

*Last updated: May 12, 2026 | Cypress 13.6.3+ | Next.js 13-16*
