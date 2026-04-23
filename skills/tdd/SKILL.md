---
name: tdd-red-green-refactor
description: >
  Enforces a disciplined Red-Green-Refactor (TDD) workflow in TypeScript/Node.js.
  Use this whenever creating new features, fixing bugs, or migrating logic to ensure
  high-quality, verifiable implementations.
---

# Red-Green-Refactor (TDD) Skill: TypeScript Edition

This skill implements a structural framework for AI-assisted programming so every change is test-verified, typed, and incremental.

## Quick Reference

- Use for new features, bug fixes, and logic migrations.
- Mandatory loop: Write 1 test -> See it fail -> Write minimal fix -> See it pass -> Refactor -> Re-run tests.
- Never batch-write many tests before implementation.
- Never change tests just to make broken code pass.
- For bug fixes, first confirm root cause with focused debug logging (use JSON.stringify for nested objects).

## The Three-Phase Cycle

### Phase 1: Red (Establish Failure)
You must prove the feature does not exist and that your test is valid.

1. Write one test for the next smallest behavior slice.
2. Execute the test and confirm it fails.
3. Verify failure quality:
	 - Failure must come from missing/incorrect logic.
	 - Failure must not be caused by setup/config errors.
4. For bug fixes, gather root-cause evidence before implementing:
	 - Add minimal debug logs when behavior is unclear.
	 - Prefer JSON.stringify-based logs for complex objects.

### Phase 2: Green (Minimal Pass)
Make the test pass as quickly and simply as possible.

1. Write the smallest implementation that satisfies only the current Red test.
2. Run tests:
	 - Run the focused test first.
	 - Then run the broader impacted test set.
3. Treat Red -> Green as the proof-of-work checkpoint.

### Phase 3: Refactor (Clean Up)
Improve structure while preserving Green.

1. Refactor for naming, duplication, and clarity.
2. Re-run tests after each meaningful refactor step.
3. If tests go Red, revert or correct immediately, then re-run.

## Core Operational Rules

### 1) No Horizontal Splurging
Do not write a large batch of tests upfront.

- Required cadence: Write 1 Test -> Fail -> Write 1 Fix -> Pass.
- Repeat loop for each sub-behavior.

### 2) Impose Backpressure
Use assertions and strong typing to prevent speculative coding.

- Prefer explicit TypeScript types over any.
- Let type errors and failing tests guide each next step.

### 3) Verification of Integrity
Do not weaken tests to fit flawed code.

- Only change tests when requirements changed.
- If a test is updated, document the requirement change in your reasoning.

### 4) Fail Loudly
Do not hide real errors during debugging.

- Avoid try/catch wrappers and silent fallbacks for core logic under test.
- Prefer direct failures so the Red phase reveals the true defect.

## Workflow Template

Use this exact loop during implementation:

1. Red: add one focused failing test.
2. Green: implement the smallest passing change.
3. Refactor: clean code while staying green.
4. Repeat for the next behavior.

## Example Workflow (TypeScript + Vitest)

Step 1: Red

```typescript
// math.test.ts
import { describe, it, expect } from 'vitest';
import { add } from './math';

describe('add', () => {
	it('should sum two numbers', () => {
		expect(add(2, 2)).toBe(4); // Fails while implementation is missing/incorrect
	});
});
```

Step 2: Green

```typescript
// math.ts
export const add = (a: unknown, b: unknown) => {
	return 4; // Minimal code to pass only the current test
};
```

Step 3: Refactor

```typescript
// math.ts
export const add = (a: number, b: number): number => {
	return a + b;
};
```

## Execution Notes

- If possible, run targeted tests first to keep the cycle tight.
- If a failing test is ambiguous, improve test specificity before implementation.
- If implementation does not turn Green, return to Red diagnostics and tighten understanding before writing more code.

