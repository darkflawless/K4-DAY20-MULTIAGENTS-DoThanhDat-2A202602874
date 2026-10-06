---
name: satisfy-house-rules-not-just-tests
description: Use on code-fix tasks where passing tests is necessary but not sufficient, to ensure changelog, regression tests, type hints, and no-modify rules are all met.
---
Passing the existing test suite is the floor, not the goal. House rules are graded separately and fail independently.

## Rules to satisfy on every code-fix task
1. Do not modify existing tests. New test files are allowed; editing files under `tests/` that already exist is not.
2. Add regression tests: create the required file (e.g. `tests/test_regressions.py`) with one test function per bug fixed, meeting the minimum count. Run it and confirm it passes.
3. Add type annotations to every public function (name not starting with `_`): all parameters and the return value.
4. Update the changelog: add one bullet per fix under the required heading (e.g. `## Unreleased`) in the exact format (e.g. `- fix(<function name>): <short description>`), meeting the minimum bullet count.

## Workflow
1. Fix the source so behavior matches docstrings/spec.
2. Write the regression tests and run them.
3. Add type hints and re-run the full suite.
4. Update the changelog.
5. Run the task's own checker if present; otherwise re-read the instruction and tick each rule.

## Anti-patterns
- Verifying only with a throwaway `_check.py` and the pre-existing suite.
- Fixing behavior but leaving functions untyped.
- Forgetting the changelog or writing bullets in the wrong format.
- Touching existing test files to make them pass.
