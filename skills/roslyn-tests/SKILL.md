---
name: roslyn-tests
description: "Run, monitor, and analyze xUnit/MSTest/NUnit tests via VS Test Explorer. Full TDD cycle.
TRIGGER when: user says 'run tests', 'check tests', 'test results', 'what tests failed', or any testing-related task.
DO NOT TRIGGER when: talking about workflows (wf_*) or non-test tasks."
---

# Test Runner — VS Test Explorer Integration

> Tests run via VS DTE API (Test Explorer). Non-blocking — results available via polling.

## Run tests

```json
vs { "action": "run_tests" }
// Run all tests in solution. Returns immediately, tests run in background.

vs { "action": "run_tests", "options": {"failedOnly": true} }
// Re-run only previously failed tests.

vs { "action": "stop_tests" }
// Cancel running tests.
```

## Check results

```json
vs_query { "what": "test_results" }
// → { status, passed, failed, total, results: [{name, outcome, message}], summary }
// status: "idle" | "running" | "completed" | "failed"

vs_query { "what": "test_results", "options": {"failedOnly": true} }
// → only failed tests with error messages, file paths, line numbers
```

## Discover tests

```json
vs_query { "what": "tests" }
// → { testCount, tests: [{name, class, method, file}], status }
// Uses Roslyn to find [Fact], [Test], [TestMethod], [Theory] attributes

find_tests_for_type { "targetType": "MyService" }
// → Find unit tests targeting a specific type. Searches test projects for classes/methods referencing the type.
```

## TDD cycle

```
1. vs { "action": "run_tests" }           → start tests
2. vs_query { "what": "test_results" }    → check results
3. If failed:
   - Read error message (name, message, file:line)
   - Fix the code
   - vs { "action": "run_tests", "options": {"failedOnly": true} }  → re-run failed
4. Repeat until all green
```

## Important

- "run tests" = xUnit/MSTest via Test Explorer (`vs run_tests`)
- "run workflow" = AI step-by-step automation (`wf_run`)
- These are DIFFERENT things. Never confuse them.
