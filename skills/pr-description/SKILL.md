---
name: pr-description
description: Write a PR description summarizing the conversation and code changes. Outputs to a file, does not update GitHub. Use when the user asks to write a PR description, summarize changes for a PR, or draft a PR body.
---

# PR Description

## Prerequisites

- The repo must be cloned locally
- A branch with changes ready for PR (or a diff to summarize)

## Workflow

### Step 1: Gather context

Collect the information needed to write an accurate description:

1. Review the conversation history to understand the problem and the approach taken.
2. Run `git diff` against the base branch to see the actual changes:
   ```bash
   git diff main...HEAD
   ```
   If the base branch is not `main`, adjust accordingly (check with `git log --oneline -1 origin/HEAD` or ask the user).
3. If the conversation references issues or other PRs, note them for linking.

### Step 2: Write the description

Write the PR description to `pr-description.md` in the repo root.

Structure:

```
## Overview

<A few sentences explaining the problem and how this PR solves it. If there is a related issue or PR, link it here inline (e.g. "Resolves #123" or "Follow-up to #456").>

## Changes

<Describe the main changes in plain prose. Focus on the important parts: what was added, removed, or restructured and why. Skip trivial stuff like formatting, minor renames, or import reordering unless that is the point of the PR.

Write in short paragraphs. Do not use bullet lists. Do not use bold text. Write the way you would explain the change to a teammate over chat.>
```

### Rules

- **Do NOT update the PR on GitHub**. Only write to `pr-description.md`.
- **Be accurate**. Cross-check your summary against the diff. Do not claim changes that are not in the diff.
- **Keep it short**. A few sentences for the overview, a short paragraph or two for the changes. Developers do not read walls of text.
- **Write like a developer**. No bold, no `-` bullet lists, no headers beyond Overview and Changes. Plain, direct language.
- **No emojis**. Anywhere.
- **Link relevant resources**. If the conversation mentions an issue number, PR, or doc link, include it in the overview.
- **Skip the small stuff**. Do not describe every line changed. Focus on the decisions and the meaningful parts of the diff.
