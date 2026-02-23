# cursor-config

Shared Cursor rules and skills for code review, issue solving, and day-to-day development workflows.

## What's in here

### Rules (`rules/`)

Rules are instructions that Cursor follows automatically or when triggered by context.

| File | Always on | Description |
|------|-----------|-------------|
| `coding.mdc` | Yes | General coding standards: keep docs in sync, write like a developer, no emojis, follow language conventions. |
| `pr-review.mdc` | No | How to review a PR: post inline comments via `gh api`, be concise, skip nits, focus on bugs and architecture. |
| `pr-fix.mdc` | No | How to address PR review comments: evaluate each one independently, apply fixes locally, generate a report. |
| `pr-description.mdc` | No | How to write a PR description: summarize conversation and diff, write to a file, keep it short and developer-friendly. |
| `issue-solver.mdc` | No | How to analyze an issue and produce a solution plan without implementing anything. |

### Skills (`skills/`)

Skills are step-by-step workflows that Cursor can execute when asked.

| Directory | Description |
|-----------|-------------|
| `pr-review/` | Fetches PR data, analyzes the diff, posts inline review comments and a summary on GitHub. |
| `pr-fix/` | Fetches review comments on a PR, evaluates and applies fixes locally, writes a report to `pr-fix-report.md`. |
| `pr-description/` | Gathers conversation context and `git diff`, writes a PR description to `pr-description.md`. |
| `issue-solver/` | Parses an issue, follows linked resources, reads relevant code, writes a solution plan to `issue-solution.md`. |

## Setup

Clone this repo and symlink:

```bash
ln -s  /path/to/your-repo/.cursor/rules ~/.cursor/rules
ln -s /path/to/your-repo/.cursor/skills ~/.cursor/skills
```

## Prerequisites

Some skills require the GitHub CLI (`gh`) to be installed and authenticated. Install it from https://cli.github.com/ and run `gh auth login`.

## Usage

Rules with `alwaysApply: true` (like `coding.mdc`) are active in every conversation. The rest activate when Cursor detects a matching context (e.g. you ask it to review a PR).

Skills are triggered by asking Cursor to do something that matches the skill description, for example:

- "Review PR #42" triggers the `pr-review` skill
- "Fix the PR comments" triggers the `pr-fix` skill
- "Write a PR description" triggers the `pr-description` skill
- "Solve issue #15" triggers the `issue-solver` skill
