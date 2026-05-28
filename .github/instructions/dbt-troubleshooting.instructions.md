---
applyTo: "**"
---

# Troubleshooting dbt Job Errors

## When to Use

- dbt Cloud / dbt platform job failed and you need to find the root cause
- Intermittent job failures that are hard to reproduce
- Error messages that don't clearly indicate the problem

**Not for:** Local dbt development errors — use analytics engineering skill instead.

## The Iron Rule

**Never modify a test to make it pass without understanding why it's failing.**

A failing test is evidence of a problem. Changing the test to pass hides the problem.

## Workflow

1. **Gather info**: Use MCP Admin API (`list_jobs_runs`, `get_job_run_error`) or ask user for logs + `run_results.json`
2. **Classify error**: Infrastructure / Code-Compilation / Data-Test Failure
3. **Investigate root cause**:
   - Check git history: `git log --oneline -20` and `git diff HEAD~5..HEAD -- models/ macros/`
   - Use `dbt compile --select failing_model` to isolate compilation issues
   - For test failures: get the test SQL, run with `dbt show --inline`, investigate actual data
4. **Fix or document**: If root cause found → fix + add regression test + PR. If NOT found → create findings document, don't guess.

## Error Classification

| Error Type | Indicators | Investigation |
|------------|-----------|---------------|
| **Infrastructure** | Connection timeout, warehouse error, permissions | Check warehouse, connections, timeouts |
| **Code/Compilation** | Undefined macro, syntax error, parsing error | Check git history, use `dbt parse` |
| **Data/Test Failure** | Test failed with N results, schema mismatch | Query actual failing data |

## Rationalizations That Mean STOP

| You're Thinking... | Reality |
|-------------------|---------|
| "Just make the test pass" | The test is telling you something is wrong |
| "It's probably just a flaky test" | "Flaky" means there's an underlying issue |
| "I'll just update the accepted values" | Are the new values valid business data or bugs? |

> Source: [dbt-labs/dbt-agent-skills](https://github.com/dbt-labs/dbt-agent-skills) — `troubleshooting-dbt-job-errors`
