---
name: ci-fix
description: Diagnose and fix CI workflow failures from a GitHub Actions run link. Fetches logs, identifies the root cause, and applies fixes locally. Use when the user provides a CI run URL, asks to fix CI, or asks why a workflow failed.
---

# CI Fix

## Prerequisites

- `gh` CLI authenticated and available
- The repo must be cloned locally
- The local branch should match the branch that failed in CI

## Workflow

### Step 1: Parse the run URL

Extract the run ID from the provided URL. GitHub Actions URLs follow this pattern:

```
https://github.com/<owner>/<repo>/actions/runs/<run_id>
```

If the user provides a job URL (`/runs/<run_id>/job/<job_id>`), extract both the run ID and job ID.

### Step 2: Fetch run details

Get an overview of the run and identify which jobs failed:

```bash
gh run view <run_id> --repo <owner>/<repo>
```

This shows the run status, branch, commit, and a list of jobs with their pass/fail status.

### Step 3: Fetch the failure logs

Download the logs for the failed job(s):

```bash
gh run view <run_id> --repo <owner>/<repo> --log-failed
```

If that output is too large or truncated, target a specific failed job:

```bash
gh run view <run_id> --repo <owner>/<repo> --job <job_id> --log
```

To list jobs and find the failed job ID:

```bash
gh api repos/<owner>/<repo>/actions/runs/<run_id>/jobs --jq '.jobs[] | select(.conclusion == "failure") | {id, name, conclusion}'
```

### Step 4: Identify the root cause

Read the logs and classify the failure:

- **Build error**: compilation failure, missing dependency, syntax error
- **Test failure**: assertion failed, timeout, unexpected output
- **Lint/format error**: style violation, formatting mismatch
- **Infrastructure issue**: runner out of disk, network timeout, service unavailable (not fixable locally -- report it to the user)
- **Flaky test**: test passes locally but fails intermittently in CI (flag it, do not just re-run)

Look for the first error in the log. Later errors are often cascading failures caused by the first one.

### Step 5: Find the relevant code

Based on the error, locate the files that need to change:

1. If the error includes a file path and line number, go there directly.
2. If it is a test failure, read both the test file and the code under test.
3. If it is a build or config issue, check the CI workflow file (`.github/workflows/`) and any build config files.

### Step 6: Apply the fix

Make changes to the local files. Rules:

- Fix the actual problem. Do not paper over it by skipping tests or loosening checks.
- If the CI failure is due to an infrastructure issue or something outside the codebase, report it to the user instead of attempting a code fix.
- Do NOT commit, push, or re-trigger the workflow unless the user explicitly asks.

### Step 7: Verify locally (if possible)

If the failing check can be reproduced locally, run it:

- Test failure: run the specific test suite or test case
- Lint error: run the linter
- Build error: run the build

Report the result. If it cannot be verified locally (e.g. integration tests that need CI infrastructure), note that.

### Step 8: Report

Tell the user:
- What failed and why (root cause, not just the symptom)
- What was changed to fix it
- Whether the fix was verified locally
- Anything that needs manual attention (infrastructure issues, flaky tests, config changes that may need review)
