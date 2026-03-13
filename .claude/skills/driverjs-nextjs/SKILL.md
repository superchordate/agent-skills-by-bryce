---
name: driver-js
description: Guide for implementing product tours, walkthroughs, and element highlighting using driver.js. Use this when asked to create tours, add walkthroughs, highlight UI elements, or implement user onboarding flows.
---

# Driver.js Tour Implementation Guide

Guide for implementing product tours, feature walkthroughs, and element highlighting using driver.js in Next.js/React applications.

## Quick Reference

**What driver.js does:** Lightweight (5kb), zero-dependency library for creating product tours, feature walkthroughs, and highlighting UI elements with popovers.

**Installation:** `npm install driver.js`

**Imports:**
```typescript
import { driver } from "driver.js";
import "driver.js/dist/driver.css";
```

**Basic usage:**
```typescript
const driverObj = driver({
  showProgress: true,
  steps: [
    { element: '.selector', popover: { title: 'Title', description: 'Description' } }
  ]
});

driverObj.drive(); // Start the tour
```

**Key principles:**
- Use `data-tour` attributes for tour target elements (e.g., `data-tour="client-box"`)
- Store tour completion state in localStorage to avoid repeating tours
- Use sessionStorage for multi-page tour flows
- Add delays (500ms) before starting tours to ensure DOM is ready
- Use lifecycle hooks for custom behavior (`onDestroyed`, `onHighlighted`, etc.)
- Advance tours on element click using `onHighlighted` + event listeners

**Common patterns:**
- Multi-page tour flow: Store state in sessionStorage, check on next page
- Single element highlight: Use `driverObj.highlight()` instead of `drive()`
- Dynamic tours: Use functions for `element` property that return DOM elements
- Restart tour button: Clear localStorage flags and call `driverObj.drive()`

---

## Core Concepts

### Initialization

Create a driver instance with configuration:

```typescript
const driverObj = driver({
  // Global configuration
  showProgress: true,
  showButtons: ["next", "previous", "close"],
  animate: true,
  overlayColor: "black",
  overlayOpacity: 0.75,
  stagePadding: 10,
  stageRadius: 5,
  
  // Steps for product tour
  steps: [
    { element: '.step-1', popover: { title: 'Step 1', description: 'First step' } },
    { element: '.step-2', popover: { title: 'Step 2', description: 'Second step' } }
  ],
  
  // Lifecycle hooks
  onDestroyed: () => console.log('Tour ended'),
  onHighlighted: (element, step, options) => console.log('Element highlighted')
});
```

### Step Configuration

Each step in a tour requires:

```typescript
{
  // Target element (CSS selector, DOM element, or function returning element)
  element: '.my-element',  // or document.querySelector('.my-element')
  
  // Popover configuration
  popover: {
    title: 'Step Title',
    description: 'Step description with details',
    side: 'bottom',     // 'top' | 'right' | 'bottom' | 'left'
    align: 'center',    // 'start' | 'center' | 'end'
    showButtons: ['next', 'previous', 'close'],
    showProgress: true,
    progressText: '{{current}} of {{total}}',
  },
  
  // Step-specific options
  disableActiveInteraction: false,
  onDeselected: (element, step, options) => {},
  onHighlighted: (element, step, options) => {}
}
```

### Element Targeting

**Best practice:** Use `data-tour` attributes:

```tsx
// In JSX
<button data-tour="submit-button">Submit</button>

// In driver.js
element: "[data-tour='submit-button']"
```

**Dynamic elements:** Use functions:

```typescript
{
  element: () => document.querySelector('.dynamic-element'),
  popover: { title: 'Dynamic', description: 'Loaded dynamically' }
}
```

### Lifecycle Hooks

**Global hooks** (apply to all steps):
- `onHighlightStarted`: Before element is highlighted
- `onHighlighted`: After element is highlighted
- `onDeselected`: When moving away from a step
- `onDestroyStarted`: Before tour is destroyed
- `onDestroyed`: After tour is destroyed
- `onNextClick`: When next button is clicked
- `onPrevClick`: When previous button is clicked
- `onCloseClick`: When close button is clicked

**Step-level hooks** (override global):
```typescript
steps: [
  {
    element: '.step-1',
    popover: { /* ... */ },
    onHighlighted: (element, step, options) => {
      // Custom behavior for this step only
    }
  }
]
```

### API Methods

Once initialized, the driver object provides:

```typescript
// Tour navigation
driverObj.drive();         // Start tour at step 0
driverObj.drive(2);        // Start tour at step 2
driverObj.moveNext();      // Move to next step
driverObj.movePrevious();  // Move to previous step
driverObj.moveTo(3);       // Move to specific step

// State queries
driverObj.hasNextStep();      // Boolean
driverObj.hasPreviousStep();  // Boolean
driverObj.isFirstStep();      // Boolean
driverObj.isLastStep();       // Boolean
driverObj.isActive();         // Is tour/highlight active

// Getters
driverObj.getActiveIndex();    // Current step index
driverObj.getActiveStep();     // Current step config
driverObj.getActiveElement();  // Current DOM element
driverObj.getConfig();         // Current configuration
driverObj.getState();          // Current state object

// Setters
driverObj.setConfig({ /* ... */ });  // Update configuration
driverObj.setSteps([ /* ... */ ]);   // Update steps

// Single element highlighting
driverObj.highlight({
  element: '.element',
  popover: { title: 'Title', description: 'Description' }
});

// Control
driverObj.refresh();  // Recalculate and redraw
driverObj.destroy();  // End tour/highlight
```

---

## Implementation Patterns

### Pattern 1: React Hook for Auto-Start Tour

Create a custom hook that automatically starts the tour:

```typescript
"use client";

import { useEffect } from "react";
import { driver } from "driver.js";
import "driver.js/dist/driver.css";

export function useWelcomeTour() {
  useEffect(() => {
    // Check if tour has been completed
    const tourCompleted = localStorage.getItem("tourCompleted");
    if (tourCompleted) return;

    const driverObj = driver({
      showProgress: true,
      steps: [
        {
          element: "[data-tour='first-step']",
          popover: {
            title: "Welcome! 👋",
            description: "Let's take a quick tour of the features.",
            side: "bottom",
            align: "center"
          },
          onHighlighted: (element) => {
            if (element) {
              const handleClick = () => {
                // Advance tour when element is clicked
                if (driverObj.hasNextStep()) {
                  driverObj.moveNext();
                } else {
                  driverObj.destroy();
                }
              };
              element.addEventListener('click', handleClick, { once: true });
            }
          }
        }
      ],
      onDestroyed: () => {
        localStorage.setItem("tourCompleted", "true");
      }
    });

    // Delay to ensure DOM is ready
    const timeout = setTimeout(() => {
      driverObj.drive();
    }, 500);

    return () => clearTimeout(timeout);
  }, []);
}

// Usage in component
export default function HomePage() {
  useWelcomeTour();
  return <div data-tour="first-step">Content</div>;
}
```

### Pattern 2: Multi-Page Tour Flow

Use sessionStorage to coordinate tours across page navigations:

```typescript
// Page 1: Home
export function useHomeTour() {
  useEffect(() => {
    const tourCompleted = localStorage.getItem("tourCompleted");
    if (tourCompleted) return;

    const driverObj = driver({
      steps: [
        {
          element: "[data-tour='client-box']",
          popover: {
            title: "Select a Client",
            description: "Click here to view the dashboard."
          },
          onHighlighted: (element) => {
            if (element) {
              const handleClick = () => {
                // Let navigation happen, then end this step
                setTimeout(() => driverObj.destroy(), 100);
              };
              element.addEventListener('click', handleClick, { once: true });
            }
          }
        }
      ]
    });

    setTimeout(() => driverObj.drive(), 500);
  }, []);
}

// In component, set flag on navigation
<Link 
  href="/dashboard"
  data-tour="client-box"
  onClick={() => {
    if (!localStorage.getItem("tourCompleted")) {
      sessionStorage.setItem("showDashboardTour", "true");
    }
  }}
>
  Dashboard
</Link>

// Page 2: Dashboard
export function useDashboardTour() {
  useEffect(() => {
    const showTour = sessionStorage.getItem("showDashboardTour");
    if (!showTour) return;
    
    sessionStorage.removeItem("showDashboardTour");

    const driverObj = driver({
      steps: [
        {
          element: "[data-tour='dashboard-widget']",
          popover: {
            title: "Dashboard Overview",
            description: "Here's your data."
          },
          onHighlighted: (element) => {
            if (element) {
              const handleClick = () => {
                setTimeout(() => driverObj.destroy(), 100);
              };
              element.addEventListener('click', handleClick, { once: true });
            }
          }
        }
      ],
      onDestroyed: () => {
        localStorage.setItem("tourCompleted", "true");
      }
    });

    setTimeout(() => driverObj.drive(), 500);
  }, []);
}
```

### Pattern 3: Manual Tour Button

Provide a button to restart the tour:

```typescript
"use client";

import { driver } from "driver.js";
import "driver.js/dist/driver.css";

export default function TourButton() {
  const startTour = () => {
    // Clear tour completion flags
    localStorage.removeItem("tourCompleted");
    sessionStorage.clear();

    const driverObj = driver({
      showProgress: true,
      steps: [
        { element: '.step-1', popover: { title: 'Step 1', description: 'First' } },
        { element: '.step-2', popover: { title: 'Step 2', description: 'Second' } }
      ],
      onDestroyed: () => {
        localStorage.setItem("tourCompleted", "true");
      }
    });

    driverObj.drive();
  };

  return (
    <button 
      onClick={startTour}
      className="fixed bottom-6 right-6 z-50 bg-blue-600 text-white px-4 py-2 rounded-lg"
    >
      Start Tour
    </button>
  );
}
```

### Pattern 4: Contextual Help (Single Element)

Highlight a single element without a full tour:

```typescript
function showHelp() {
  const driverObj = driver();
  
  driverObj.highlight({
    element: '#complex-feature',
    popover: {
      title: 'Need Help?',
      description: 'This feature allows you to...',
      side: 'left',
      align: 'start'
    }
  });
}

<button onClick={showHelp}>Show Help</button>
```

### Pattern 5: Async/Dynamic Content

Handle dynamically loaded content:

```typescript
const driverObj = driver({
  steps: [
    {
      element: () => {
        // Wait for element to exist
        return document.querySelector('.dynamic-element') || document.body;
      },
      popover: {
        title: 'Dynamic Content',
        description: 'This loaded dynamically.'
      }
    }
  ]
});
```

### Pattern 6: Advance Tour on Element Click

Make the tour advance automatically when users click highlighted elements:

```typescript
export function useInteractiveTour() {
  useEffect(() => {
    const driverObj = driver({
      steps: [
        {
          element: "[data-tour='step-1']",
          popover: {
            title: "Click to Continue",
            description: "Click this element to move to the next step."
          },
          onHighlighted: (element) => {
            if (element) {
              const handleClick = () => {
                if (driverObj.hasNextStep()) {
                  driverObj.moveNext();
                } else {
                  driverObj.destroy();
                }
              };
              // Use { once: true } to automatically remove listener after one click
              element.addEventListener('click', handleClick, { once: true });
            }
          }
        },
        {
          element: "[data-tour='step-2']",
          popover: {
            title: "Second Step",
            description: "Click here to continue."
          },
          onHighlighted: (element) => {
            if (element) {
              const handleClick = () => {
                driverObj.destroy();
              };
              element.addEventListener('click', handleClick, { once: true });
            }
          }
        }
      ]
    });

    setTimeout(() => driverObj.drive(), 500);
  }, []);
}
```

**Note:** For elements that navigate to another page (like links), add a small delay before advancing:

```typescript
onHighlighted: (element) => {
  if (element) {
    const handleClick = () => {
      // Let navigation happen first
      setTimeout(() => driverObj.destroy(), 100);
    };
    element.addEventListener('click', handleClick, { once: true });
  }
}
```

---

## Styling

### CSS Customization

Override default styles in your global CSS:

```css
/* Popover styling */
.driver-popover {
  background: #ffffff !important;
  color: #000000 !important;
}

.driver-popover-title {
  font-family: 'Your Font', sans-serif !important;
  font-size: 1.25rem !important;
  color: #0A2240 !important;
}

.driver-popover-description {
  color: #4a5568 !important;
}

/* Button styling */
.driver-popover-next-btn,
.driver-popover-prev-btn,
.driver-popover-close-btn {
  background: #0099A8 !important;
  color: #ffffff !important;
}

.driver-popover-next-btn:hover,
.driver-popover-prev-btn:hover {
  background: #007a86 !important;
}

/* Progress text */
.driver-popover-progress-text {
  color: #0099A8 !important;
}

/* Overlay */
.driver-overlay {
  background: rgba(0, 0, 0, 0.75) !important;
}
```

### Configuration-Based Styling

```typescript
const driverObj = driver({
  popoverClass: 'custom-tour-popover',
  overlayColor: '#1a202c',
  overlayOpacity: 0.8,
  stagePadding: 15,
  stageRadius: 8
});
```

---

## Best Practices

### 1. DOM Readiness

Always add a delay before starting tours in client components:

```typescript
useEffect(() => {
  const timeout = setTimeout(() => {
    driverObj.drive();
  }, 500);  // Ensure DOM is ready
  
  return () => clearTimeout(timeout);
}, []);
```

### 2. Tour Target Attributes

Use semantic `data-tour` attributes:

```tsx
// Good
<button data-tour="submit-form">Submit</button>

// Avoid
<button className="btn-primary-submit-form">Submit</button>
```

### 3. Tour State Management

- **localStorage**: For persistent tour completion (across sessions)
- **sessionStorage**: For multi-page tour flows (same session only)

```typescript
// Mark tour as completed
localStorage.setItem("tourCompleted", "true");

// Check if tour should show
const shouldShow = !localStorage.getItem("tourCompleted");

// Clear for testing
localStorage.removeItem("tourCompleted");
```

### 4. Popover Positioning

Choose appropriate `side` and `align` based on element position:

```typescript
// Top of page elements
{ side: 'bottom', align: 'start' }

// Bottom of page elements
{ side: 'top', align: 'start' }

// Centered elements
{ side: 'bottom', align: 'center' }

// Side navigation
{ side: 'right', align: 'start' }
```

### 5. Progress Indication

Always show progress for multi-step tours:

```typescript
const driverObj = driver({
  showProgress: true,
  progressText: 'Step {{current}} of {{total}}',
  steps: [ /* ... */ ]
});
```

### 6. Keyboard Navigation

Driver.js supports keyboard navigation by default:
- **Arrow keys**: Navigate between steps
- **Escape**: Close tour
- **Tab**: Navigate through popover buttons

Keep this enabled unless you have a specific reason:

```typescript
allowKeyboardControl: true  // default
```

### 7. Mobile Considerations

Test tours on mobile devices and adjust positioning:

```typescript
const isMobile = window.innerWidth < 768;

const driverObj = driver({
  stagePadding: isMobile ? 5 : 10,
  steps: [
    {
      element: '.element',
      popover: {
        side: isMobile ? 'bottom' : 'right',
        // Adjust descriptions for mobile
        description: isMobile ? 'Short text' : 'Detailed explanation'
      }
    }
  ]
});
```

---

## Common Issues & Solutions

### Issue: Tour starts before DOM is ready

**Solution:** Add delay in useEffect:

```typescript
useEffect(() => {
  const timeout = setTimeout(() => {
    driverObj.drive();
  }, 500);
  return () => clearTimeout(timeout);
}, []);
```

### Issue: Element not found

**Solution:** Use function-based element targeting:

```typescript
{
  element: () => document.querySelector('.my-element'),
  popover: { /* ... */ }
}
```

### Issue: Tour shows every time

**Solution:** Check localStorage before showing:

```typescript
const tourCompleted = localStorage.getItem("tourCompleted");
if (tourCompleted) return;
```

### Issue: Multi-page tour loses state

**Solution:** Use sessionStorage to pass state between pages:

```typescript
// Page 1
sessionStorage.setItem("showNextTour", "true");

// Page 2
const shouldShow = sessionStorage.getItem("showNextTour");
if (shouldShow) {
  sessionStorage.removeItem("showNextTour");
  // Start tour
}
```

### Issue: Tour conflicts with dark mode

**Solution:** Customize overlay and popover colors:

```css
.driver-popover {
  background: var(--your-bg-color) !important;
  color: var(--your-text-color) !important;
}
```

---

## Testing Tours

### Manual Testing Checklist

- [ ] Clear localStorage and sessionStorage
- [ ] Navigate to starting page
- [ ] Verify tour starts automatically (if configured)
- [ ] Click through all steps
- [ ] Test "previous" button
- [ ] Test "close" button (verify state saves)
- [ ] Reload page (verify tour doesn't repeat)
- [ ] Test manual "Start Tour" button
- [ ] Test on mobile viewport
- [ ] Test with keyboard navigation

### Reset Tour for Testing

```typescript
// Clear all tour state
localStorage.removeItem("tourCompleted");
sessionStorage.clear();

// Or create a dev-only reset button
if (process.env.NODE_ENV === 'development') {
  <button onClick={() => {
    localStorage.clear();
    sessionStorage.clear();
    window.location.reload();
  }}>
    Reset Tour
  </button>
}
```

---

## Resources

- **Official docs**: https://driverjs.com/docs/
- **GitHub**: https://github.com/kamranahmedse/driver.js
- **NPM**: https://www.npmjs.com/package/driver.js
- **Examples**: https://driverjs.com/docs/examples

---

## Summary

**Key takeaways:**
1. Use `data-tour` attributes for tour targets
2. Import both the library and CSS
3. Add 500ms delay before starting tours in React
4. Use localStorage for completion state, sessionStorage for multi-page flows
5. Customize styling via CSS or configuration
6. Test thoroughly with cleared storage
7. Provide manual tour restart button for users
8. Make tours interactive: advance on element click using `onHighlighted` hook

**When to use driver.js:**
- Product onboarding flows
- Feature walkthroughs
- Contextual help tooltips
- UI element highlighting
- Step-by-step tutorials
