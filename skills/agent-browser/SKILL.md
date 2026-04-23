---
name: agent-browser
description: Use this skill when the task requires the agent to test or browse a website directly (not relying on the user). Includes guidance for Python or Node automation, taking screenshots for debugging, and extracting simplified page content to reduce token usage.
---

# Agent Browser Skill

## Quick Start (Read First)

**Core approach:** Use Python or Node automation to test and browse websites directly without user involvement.

**Choose your tool:** Prefer Playwright (Python or Node) for best reliability and screenshot support. Selenium (Python) or Puppeteer (Node) work too. → See [Python Template](#python-playwright-template) or [Node Template](#node-playwright-template)

**Key capabilities:**
- Navigate and interact with sites programmatically
- Capture screenshots for visual debugging → [Screenshots](#screenshots-debugging)
- Extract simplified content to reduce tokens → [Simplified Page Extraction](#simplified-page-extraction-reduce-tokens)

**When to use this skill:**
- Verify UI behavior, layout, or DOM state without asking the user
- Need evidence (screenshots) or compact page snapshots for analysis
- Must interact with a site (click, fill forms, scroll) to reach the target state

**Default practices:** → [Browser Automation Defaults](#browser-automation-defaults)

## Browser Automation Defaults
- Prefer headless mode for speed, headed mode for visual debugging.
- Always wait for the relevant load state (`domcontentloaded`, `networkidle`) before assertions.
- Use explicit waits for selectors rather than arbitrary sleeps.
- Ask the user to start up the app, don't start it yourself. Ask them if there is a specific URL you can jump to for testing.

## Python Playwright Template
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://example.com", wait_until="networkidle")
    page.screenshot(path="debug-full.png", full_page=True)
    title = page.title()
    browser.close()
```

## Node Playwright Template
```javascript
const { chromium } = require('playwright');

(async () => {
  const browser = await chromium.launch({ headless: true });
  const page = await browser.newPage();
  await page.goto('https://example.com', { waitUntil: 'networkidle' });
  await page.screenshot({ path: 'debug-full.png', fullPage: true });
  const title = await page.title();
  await browser.close();
})();
```

## Screenshots (Debugging)
- Capture full-page screenshots for layout checks: `full_page: true` or `fullPage: true`.
- Capture element screenshots to focus on a specific component.
- If the page is dynamic, take a sequence of screenshots (before/after interaction).

## Simplified Page Extraction (Reduce Tokens)

When full HTML is too large for your context, use these approaches. Each has specific use cases and trade-offs → [Practical Snippets](#practical-snippets) for implementation examples.

### 1) Text-only snapshot
- Extract `document.body.innerText` and truncate to relevant sections.
- Use this for content verification and text assertions.

### 2) Strip scripts and styles
- In the browser, clone the DOM, remove `script`, `style`, and `link[rel=stylesheet]`, then serialize.
- This keeps structural HTML while removing heavy noise.

### 3) DOM tree outline
- Return a compact tree of tag names + ids/classes and short text previews.
- Example output format: `div#header.nav > h1:"Title"`.

### 4) Focused container only
- Query a specific container and serialize only that subtree.
- Best for verifying a component without loading the entire page.

## Practical Snippets

Ready-to-use code for the extraction methods described in [Simplified Page Extraction](#simplified-page-extraction-reduce-tokens).

### Python: text-only snapshot
```python
text = page.evaluate("() => document.body.innerText")
text = "\n".join(text.splitlines()[:200])
```

### Node: strip scripts and styles
```javascript
const html = await page.evaluate(() => {
  const clone = document.documentElement.cloneNode(true);
  clone.querySelectorAll('script, style, link[rel="stylesheet"]').forEach(el => el.remove());
  return clone.outerHTML;
});
```

### Node: compact DOM tree
```javascript
const tree = await page.evaluate(() => {
  const lines = [];
  const walk = (el, depth = 0) => {
    if (!el || depth > 6) return;
    const id = el.id ? `#${el.id}` : '';
    const cls = el.className ? `.${String(el.className).trim().split(/\s+/).slice(0, 2).join('.')}` : '';
    const text = (el.textContent || '').trim().replace(/\s+/g, ' ').slice(0, 60);
    lines.push(`${'  '.repeat(depth)}${el.tagName.toLowerCase()}${id}${cls}${text ? ':"' + text + '"' : ''}`);
    Array.from(el.children).slice(0, 10).forEach(child => walk(child, depth + 1));
  };
  walk(document.body, 0);
  return lines.join('\n');
});
```

## Validation and Reporting

After completing browser automation tasks:
- State the exact URL, steps, and selectors used
- Include screenshot paths when relevant → [Screenshots](#screenshots-debugging)
- If content was simplified, describe the method used → [Simplified Page Extraction](#simplified-page-extraction-reduce-tokens)
