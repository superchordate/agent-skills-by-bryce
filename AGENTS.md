# General Design Notes

- Don't use try/catch or || fallbacks. Erorrs should fail loudly to allow debugging as they arise. 

- Top-of-file metadata: Always check the first few lines of files you work with in case there are special comments there. Add special comments there as you identify gotchas. If there is a test referenced and you change the code, make sure to run the test.

- If you add a component, add a Jest/Pytest test to cover it. 

- When running npm test, use npm test 2>&1 | Where-Object { $_ -match "FAIL|Summary of all failing|● .*›|Test Suites: .* failed|Tests: .* failed" } or similar to just get the key information from results. Then run the specific tests to see error messages. Fix all failing tests at once, then re-run them to confirm.

- If the code doesn't make it clear what is wrong, add minimal debug logs and I'll report them back to you to see what the data looks like. Debug logs will collapse objects so use stringify with truncation to view data. 

- DuckDB SQL should simply pull necessary data at the summary level. It should NOT do complex processing of the results. Pass the data to JS for any detailed processing of the output.

- You can ask for help! If there is clearly important context missing, stop what you are doing and ask the user for help. If the user gets annoyed, that's OK it just means you are asking the correct amount of questions. 

- Write temp python files and run them with venv instead of trying to write Python in PowerShell. 
