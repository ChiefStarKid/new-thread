---
name: new-thread
description: Spin off a live parallel CC session as a chip. Claude synthesises the relevant context from the current conversation and calls spawn_task — no copy-paste, no context loss. Usage: /new-thread [objective]. No plan mode.
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

## Step 3 — Synthesise context and build the prompt

The spawned session has no memory of this conversation. Your job is to determine what context it actually needs to carry out the task — not to dump the whole conversation, but to extract and include only what is relevant to the new objective.

First, classify the brief:

**If the brief is a bug fix** (keywords: fix, broken, error, failing, not working, exception, crash, wrong output):

Extract from the current conversation:
- What the user was doing when the issue appeared
- What they were trying to achieve
- What the blocker or issue is (error, wrong output, unexpected behaviour)
- Whether the issue can be reproduced, and under what conditions
- Whether the root cause is technical (code/system) or behavioural (needs a change in logic, approach, or understanding)
- What has already been tried or ruled out

Construct the prompt:

```
Date: <today from currentDate system context>

Task: <the brief verbatim>

Bug report:
- Doing: <what the user was doing>
- Goal: <what they were trying to achieve>
- Issue: <the blocker or observed problem>
- Reproducible: <yes / no / unknown — and conditions if known>
- Root cause type: <technical | behavioural>
- Tried: <anything already attempted — omit if nothing>
```

**For all other tasks:**

Read the current conversation and identify:
- Decisions already made that bear on the new task
- Artefacts produced (files written, outputs generated) the new session should know about
- Constraints or ruling-outs that would otherwise be rediscovered the hard way
- Any framing or background the new session needs to start without false assumptions

Construct the prompt:

```
Date: <today from currentDate system context>

Task: <the brief verbatim>

Context: <synthesised relevant context from the current session — omit if nothing material applies>
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
