---
name: docs
description: Write or update project documentation. Produces clear, concise, developer-style docs. Use when the user asks to write docs, document a feature, create a README, or update existing documentation.
---

# Documentation

## Workflow

### Step 1: Understand the scope

Determine what needs to be documented:

- If the user specifies a file or feature, read the relevant code first.
- If the request is broad (e.g. "document this project"), scan the repo structure, key entry points, and any existing docs.
- If existing docs are being updated, read them to understand the current state and what changed.

### Step 2: Gather technical details

Before writing, collect the facts:

1. Read the source code that the docs will cover.
2. Check for existing READMEs, doc comments, or config files that provide context.
3. If the docs relate to a recent change, run `git diff` or `git log` to understand what changed.

### Step 3: Write the documentation

Write or update the docs following these rules:

**Tone and style**
- Write like a developer writing internal docs. Direct, flat, no filler.
- Do not address the reader as "you" or "your". Use imperative mood ("Run the command", "Set the flag") or third person.
- No emojis. No exclamation marks for emphasis.
- No AI-sounding phrases: skip "Let's", "Here's how to", "It's important to note", "Additionally", "Furthermore". Just state the information.

**Structure**
- Start with what the thing is and what it does. One or two sentences.
- Then cover how to use it: setup, configuration, commands, API.
- End with anything non-obvious: edge cases, limitations, gotchas.
- Use headers to organize sections. Keep the hierarchy flat (avoid nesting more than two levels deep).

**Length**
- Every sentence should add information. If removing a sentence loses nothing, remove it.
- Prefer short paragraphs (2-4 sentences). Walls of text do not get read.
- Code examples should be minimal and runnable. Do not over-comment them.

**Accuracy**
- Do not document behavior that does not exist in the code. Cross-check against the source.
- If something is unclear or undocumented in the code, call it out rather than guessing.

### Step 4: Place the output

- If updating existing docs, edit them in place.
- If creating new docs, place them where they belong (e.g. `README.md` in the repo root, `docs/` directory if one exists).
- Do not create files the user did not ask for.
