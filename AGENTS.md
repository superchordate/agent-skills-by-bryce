# General Design Notes

- Don't use try/catch or || fallbacks. Erorrs should fail loudly to allow debugging as they arise. 

- Top-of-file metadata: Always check the first few lines of files you work with in case there are special comments there. Add special comments there as you identify gotchas. If there is a test referenced and you change the code, make sure to run the test.

- If you add a component, add a Jest/Pytest test to cover it. 

