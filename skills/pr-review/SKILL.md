---
name: pr-review
description: Review GitHub pull requests with actionable feedback. Posts code-specific comments on relevant lines and a summary as a general PR comment. Use when the user asks to review a PR, review a pull request, or give PR feedback.
---

# PR Review

## Prerequisites

- `gh` CLI authenticated and available
- The repo must be cloned locally or accessible via `gh`

## Workflow

### Step 1: Fetch PR data

Run these in parallel:
```bash
gh pr view <NUMBER> --json title,body,headRefName,baseRefName,files,state,url
gh pr diff <NUMBER>
gh api repos/<OWNER>/<REPO>/pulls/<NUMBER>/comments  # existing comments
gh api repos/<OWNER>/<REPO>/issues/<NUMBER>/comments  # PR-level conversation comments
```

### Step 2: Check the PR description and linked issues

- Read the PR description. It is the contract for what the PR claims to deliver.
- If the PR references an issue (e.g. "Fixes #123", "Closes #456"), fetch it:
  ```bash
  gh issue view <ISSUE_NUMBER> --repo <OWNER>/<REPO>
  ```
  Verify whether the code changes resolve the issue.
- Cross-check every claim in the PR description against the diff. Call out discrepancies.

### Step 3: Read existing PR comments for context

- Review existing comments (fetched in Step 1) for context from other reviewers.
- Use human reviewer insights to inform your analysis.

### Step 4: Resolve addressed review comments

Check all previous `[cursor-auto-review]` and Copilot review comments that are still unresolved. For each one, compare the flagged issue against the current diff. If the code has been updated to address the concern, resolve the thread on GitHub.

Fetch review threads using the GraphQL API:

```bash
gh api graphql -f query='
query {
  repository(owner: "<OWNER>", name: "<REPO>") {
    pullRequest(number: <NUMBER>) {
      reviewThreads(first: 100) {
        nodes {
          id
          isResolved
          comments(first: 5) {
            nodes {
              body
              author { login }
              path
              line
            }
          }
        }
      }
    }
  }
}'
```

For each unresolved thread where the comment was posted by the current bot or by Copilot:
1. Read the comment body to understand what was flagged.
2. Check the current diff at that file/line to see if the issue was addressed.
3. If resolved, mark the thread:

```bash
gh api graphql -f query='
mutation {
  resolveReviewThread(input: {threadId: "<THREAD_ID>"}) {
    thread { id isResolved }
  }
}'
```

Only resolve when the fix is clear. If the change is ambiguous or only partially addresses the concern, leave the thread open.

### Step 5: Analyze the diff

Read the full diff. For large diffs, read in chunks. Understand every changed file — do not skim.

Focus on:
- **Bugs**: logic errors, race conditions, missing error handling, off-by-one, null/undefined risks
- **Architecture**: wrong abstractions, coupling, misplaced responsibilities, unnecessary complexity
- **Design decisions**: naming that misleads, patterns that will cause maintenance pain
- **Security**: injection, secrets in code, auth gaps, unsafe defaults
- **Correctness**: does the code actually do what the PR description claims?
- **Completeness**: are all items promised in the PR description (or linked issue) delivered?

Do NOT comment on:
- Style, formatting, or linting issues
- Minor nits (typos in comments, import ordering, trailing whitespace)
- Things that are merely "not how I would do it" without a concrete downside

### Step 6: Form opinions

Before writing comments, decide:
- Is the overall approach sound? If not, say so directly.
- Are there naming or architectural decisions you disagree with? State your position.
- Will something bite the team in 3 months? Call it out.
- Does the PR deliver what the description (and linked issue) promises?

Do not soften with praise. Get to the point.

### Step 7: Post code-specific comments

For each finding tied to specific lines, post an inline review comment.

**Comment format**: every comment must be short and direct. Use `<details>` for expanded context.

```
[cursor-auto-review] Short, direct statement of the problem.

<details><summary>Details</summary>

Brief explanation — why this matters and a suggested fix. Keep it to a few sentences.

</details>
```

Post using `gh api`:

```bash
gh api repos/{owner}/{repo}/pulls/{number}/comments \
  -f body='[cursor-auto-review] Short problem statement.

<details><summary>Details</summary>

Brief context and suggested fix.

</details>' \
  -f commit_id='<HEAD_SHA>' \
  -f path='<file_path>' \
  -F line=<line_number> \
  -f side='RIGHT'
```

To get the HEAD commit SHA:
```bash
gh pr view <NUMBER> --json commits --jq '.commits[-1].oid'
```

Rules for inline comments:
- Every inline comment body MUST start with `[cursor-auto-review]`
- One comment per finding — do not batch unrelated issues
- The visible part (before `<details>`) must be 1-2 sentences max
- The `<details>` section provides context but is also kept brief — a few sentences, not paragraphs
- Use the line number from the RIGHT side of the diff

### Step 8: Post the summary comment

Post a single general review comment on the PR:

```bash
gh pr comment <NUMBER> --body '<summary>'
```

The summary MUST:
- Start with `[cursor-auto-review]`
- Be short — a few bullet points at most for the visible part
- Use `<details>` for any expanded context
- Contain ONLY issues not already covered by inline comments
- Be direct — no filler, no compliments
- Do NOT state whether the PR is ready to merge

If you have no general findings beyond inline comments:
```
[cursor-auto-review] N findings posted as inline comments.
```

## Comment format

Every comment (inline and summary) MUST:
- Be prefixed with `[cursor-auto-review]`
- Have the main finding visible in 1-2 short sentences
- Use `<details><summary>Details</summary>...</details>` for expanded explanation

## What to skip

- Do NOT post comments on things that are obviously intentional and reasonable
- Do NOT flag style or formatting — assume linters handle that
- Do NOT leave "nit:" comments
- Do NOT compliment or praise any part of the code
- Do NOT hedge — if something is wrong, say it is wrong
