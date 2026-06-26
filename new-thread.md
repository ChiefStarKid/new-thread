---
name: new-thread
description: Spin off a new CC session as a one-click chip. Usage: /new-thread [brief description of what the new thread should do]. Calls mcp__ccd_session__spawn_task immediately — no plan mode.
---

# /new-thread — Spin off a new CC session

Wraps `mcp__ccd_session__spawn_task` to create a chip without the user having to open a new chat manually.

**Do NOT enter plan mode. Execute immediately.**

---

## Step 1 — Get the brief

- If the user provided an argument after `/new-thread`, use it as the brief.
- If no argument was given, call `AskUserQuestion` with a single question: "What should the new thread work on?" (header: "Brief", options: leave open-ended — use the Other/free-text path).

---

## Step 2 — Load the spawn_task schema

Call `ToolSearch` with `{ "query": "select:mcp__ccd_session__spawn_task", "max_results": 1 }` to load the tool schema before calling it.

---

## Step 3 — Build the prompt

Construct the `prompt` string for the spawned session. It must be **self-contained** — the spawned session has no memory of this conversation.

```
Date: <today from currentDate system context>

Task: <the brief verbatim>
```

Close the prompt with:

```
Follow CLAUDE.md before doing anything.
```

---

## Step 4 — Derive title and tldr

- **`title`**: Imperative phrase from the brief, ≤60 chars. Start with a verb (e.g. "Fix auth bug in login flow", "Draft Q3 update email", "Build momentum report").
- **`tldr`**: 1–2 plain-English sentences describing what the new session will do and why. No file paths.

---

## Step 5 — Call spawn_task

Call `mcp__ccd_session__spawn_task` with:
- `title`: from Step 4
- `tldr`: from Step 4
- `prompt`: from Step 3
- `cwd`: omit (defaults to `~`)

---

## Step 6 — Confirm

Output one line in chat:

```
Thread chip created: "<title>"
```

If the tool fails, report the error and suggest the user call `/new-thread` again from a main CC window (not a worktree session).
