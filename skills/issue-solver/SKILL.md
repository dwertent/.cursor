---
name: issue-solver
description: Analyze a GitHub issue (or any issue description), research linked resources, and produce a solution plan as a markdown file. Use when the user asks to solve an issue, investigate an issue, plan a fix for an issue, or create a solution plan from a GitHub issue URL or description.
---

# Issue Solver

## Prerequisites

- `gh` CLI authenticated and available (for GitHub issues)
- The repo should be cloned locally if the issue relates to a specific project

## Workflow

### Step 1: Parse the issue

Determine what the user provided:

- **GitHub issue URL or number** — fetch it with `gh`
- **Plain text description** — use it directly

For GitHub issues:

```bash
gh issue view <NUMBER> --json title,body,labels,assignees,comments,state,url
```

If only a URL is provided (e.g. `https://github.com/owner/repo/issues/123`), extract the owner, repo, and number from it.

### Step 2: Extract and follow linked resources

Scan the issue body and comments for:

- **Referenced issues/PRs** (`#123`, `owner/repo#456`) — fetch each one
- **External links** (docs, RFCs, Stack Overflow, blog posts, etc.) — fetch with `WebFetch` to understand context
- **Code references** (file paths, line numbers, stack traces) — read the referenced files locally
- **Error messages or logs** — note them for diagnosis

Prioritize links that provide context about root cause, expected behavior, or prior discussion. Skip links that are clearly irrelevant (badges, CI status, unrelated docs).

### Step 3: Understand the codebase context

If the issue relates to a local repo:

1. Search for relevant code using `Grep` or `SemanticSearch` based on symbols, error messages, or concepts from the issue
2. Read the relevant files to understand the current behavior
3. Check recent git history for related changes if useful:
   ```bash
   git log --oneline -20 -- <relevant_files>
   ```

### Step 4: Analyze and form a solution

Based on everything gathered:

1. Identify the **root cause** or core problem
2. Consider multiple approaches if applicable
3. Pick the best approach — justify briefly why
4. Break the solution into concrete, ordered steps
5. Note any risks, trade-offs, or open questions

### Step 5: Write the solution plan

Write the plan to `issue-solution.md` in the repo root (or current working directory if no repo). Use this structure:

```markdown
# Issue Solution — <title or short description>

**Source**: <issue URL or "User-provided description">
**Date**: <today's date>

## Overview

<2-4 sentence summary: what the issue is, why it happens, and the high-level approach to fix it.>

## Root Cause

<Concise explanation of why the issue occurs. Reference specific files/lines when possible.>

## Solution

### Approach

<Brief description of the chosen approach and why it was selected over alternatives, if applicable.>

### Steps

1. **<Action>** — `path/to/file`
   <What to change and why. Be specific enough that a developer can execute without guessing.>

2. **<Action>** — `path/to/file`
   <What to change and why.>

3. ...

### Testing

<How to verify the fix works. Specific commands, test cases, or manual steps.>

## Open Questions

<Anything unresolved — ambiguities in the issue, decisions that need team input, risks.
Remove this section if there are none.>

## References

- <Link 1 — brief description>
- <Link 2 — brief description>
```

### Rules

- **Be specific**. "Update the handler" is useless. "Add a nil check before accessing `resp.Body` in `pkg/client/http.go:142`" is actionable.
- **Steps must be ordered**. A developer should be able to follow them top-to-bottom.
- **Don't implement the fix**. This skill produces a plan, not code changes. The user will decide whether to proceed with implementation.
- **Surface unknowns**. If the issue is vague or missing context, say so in Open Questions rather than guessing.
- **Keep it concise**. The plan should be thorough but not verbose. Aim for clarity, not length.
