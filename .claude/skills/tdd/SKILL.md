---
name: tdd
description: Strategy for debugging with Test Driven Development. Use whenever the user asks for help debugging code or to fix something in the app.
---

# Test-Driven Development (TDD)

Use these steps to fix bugs, only skip them if the nature of the bug makes writing a test very difficult. 
1. Identify the fix for the bug, but don't implement it yet. Use debug logging (with JSON stringify so we don't need to expand objects in the console) or other techniques to understand the root cause of the bug.
2. Write a test that fails because of the bug. This test should be as specific as possible to the bug, and should fail because of the bug. Use the test-nextjs and analytics-app-tests skills for guidance on writing tests. DO NOT FIX THE BUG YET. Run the test to ensure it fails in the manner expected.
3. Implement the fix for the bug.
4. Run the test to ensure it passes. If it doesn't, go back to step 1 and repeat the process until the test passes.

