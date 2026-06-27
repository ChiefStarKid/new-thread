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

## Step 2 — Capture root session identity

Before loading the schema, identify the current session so the spawn can reference it.

**The captured ID must be the `local_`-prefixed session ID that `list_sessions` / `send_message` use** — NOT the raw UUID from the JSONL transcript filename. They are different namespaces; an ID in the wrong namespace cannot be resolved by `/loop-back` later.

1. Load the lookup tool: `ToolSearch` with `{ "query": "select:mcp__ccd_session_mgmt__list_sessions", "max_results": 1 }`.
2. Note the current session title from the conversation context.
3. The current session is **excluded** from `list_sessions`, so it cannot return its own ID directly. Capture instead:
   - `Root session title` — the current session title (the primary handle `/loop-back` resolves against).
   - `Root session ID` — best-effort. If the runtime exposes the current `local_…` session ID, use it. Otherwise record the raw transcript UUID and mark it `(transcript UUID — resolve live via title)` so `/loop-back` knows not to trust it directly.

`/loop-back` resolves the live ID from `Root session title` at send time (titles can drift, IDs can go stale), so the title is the field that matters most here.

---

## Step 3 — Load the spawn_task schema

Call `ToolSearch` with `{ "query": "select:mcp__ccd_session__spawn_task", "max_results": 1 }` to load the tool schema before calling it.

---

## Step 4 — Synthesise context and build the prompt

The spawned session has no memory of this conversation. Your job is to determine what context it actually needs to carry out the task — not to dump the whole conversation, but to extract and include only what is relevant to the new objective.

First, classify the brief by intent — not by keywords. Read the brief and the current conversation to determine which mode fits:

| Mode | Intent signals |
|---|---|
| **Bug / defect** | Something is broken, behaving wrongly, or throwing an error. The goal is to diagnose and fix a specific failure. |
| **Ideation** | The goal is to generate, explore, or evaluate options. No single right answer exists yet. Output is ideas, not a deliverable. |
| **Research / explain** | The goal is to answer a specific question or explain something. Output is understanding or a summary, not an action. |
| **Clarification** | A specific ambiguity in the current session needs resolving without polluting this context. Output is a decision or answer to bring back. |
| **Task** | Everything else — build, write, update, draft, run. A concrete deliverable is expected. |

If the intent is genuinely ambiguous after reading the brief and context, use `AskUserQuestion` to ask — but only if resolving it here is cheaper than letting the spawned session figure it out itself.

Also check: **is this session acting as a hub?** If the user has fired or is about to fire multiple chips on related variants (e.g. testing hypotheses, running parallel experiments), note the hub pattern in the prompt and suggest a naming convention (e.g. H1/H2/H3, or Variant A/B/C) so results can be correlated when they come back.

---

### Bug / defect

Extract from the current conversation:
- What the user was doing when the issue appeared
- What they were trying to achieve
- What the blocker or issue is (error, wrong output, unexpected behaviour)
- Whether the issue can be reproduced, and under what conditions
- Whether the root cause is technical (code/system) or behavioural (needs a change in logic, approach, or understanding)
- What has already been tried or ruled out

```
Date: <today from currentDate system context>
Root session ID: <UUID from Step 2>
Root session title: <title from Step 2>

Task: <the brief verbatim>

Bug report:
- Doing: <what the user was doing>
- Goal: <what they were trying to achieve>
- Issue: <the blocker or observed problem>
- Reproducible: <yes / no / unknown — and conditions if known>
- Root cause type: <technical | behavioural>
- Tried: <anything already attempted — omit if nothing>
```

---

### Ideation

Extract from the current conversation:
- The problem space being explored
- Constraints or requirements already established
- Options or directions already considered and their status (live, ruled out, preferred)
- What a good output looks like (volume of ideas? ranked options? a single recommendation?)

```
Date: <today from currentDate system context>

Task: <the brief verbatim>

Ideation brief:
- Problem: <the space being explored>
- Constraints: <what must be true of any valid idea — omit if none>
- Already considered: <options already on the table and their status — omit if none>
- Output expected: <what the user wants back>
```

---

### Research / explain

Extract from the current conversation:
- The specific question to be answered
- Relevant data, files, or sources already identified
- What the answer will be used for (so the spawn calibrates depth and format)

```
Date: <today from currentDate system context>

Task: <the brief verbatim>

Research brief:
- Question: <the specific question>
- Sources / data: <what's available to draw on — omit if none identified>
- Used for: <how the answer feeds back into the current work>
```

---

### Clarification

Extract from the current conversation:
- The specific ambiguity or decision point
- The options or interpretations in play
- What the user needs back (a recommendation, a decision, a fact)

```
Date: <today from currentDate system context>

Task: <the brief verbatim>

Clarification needed:
- Ambiguity: <what is unclear>
- Options: <interpretations or choices in play>
- Output: <what resolving this looks like>
```

---

### Task

Read the current conversation and identify:
- Decisions already made that bear on the new task
- Artefacts produced (files written, outputs generated) the new session should know about
- Constraints or ruling-outs that would otherwise be rediscovered the hard way
- Any framing or background the new session needs to start without false assumptions

```
Date: <today from currentDate system context>

Task: <the brief verbatim>

Context: <synthesised relevant context from the current session — omit if nothing material applies>
```

---

Close every prompt with:

```
Follow CLAUDE.md before doing anything.
```

---

## Step 5 — Derive title and tldr

- **`title`**: Imperative phrase from the brief, ≤60 chars. Start with a verb (e.g. "Fix auth bug in login flow", "Draft Q3 update email", "Build momentum report").
- **`tldr`**: 1–2 plain-English sentences describing what the new session will do and why. No file paths.

---

## Step 6 — Call spawn_task

Before calling, prepend the following as the **first line** of the prompt (before `Date:`):

```
Thread title: <title from Step 5>
```

This gives the child a stable, searchable anchor it can use in `/loop-back` to find this root session, even if this session's title drifts after spawning.

Call `mcp__ccd_session__spawn_task` with:
- `title`: from Step 5
- `tldr`: from Step 5
- `prompt`: from Step 4 (with `Thread title:` prepended)
- `cwd`: omit (defaults to `~`)

---

## Step 7 — Confirm

Output one line in chat:

```
Thread chip created: "<title>"
```

If the tool fails, report the error and suggest the user call `/new-thread` again from a main CC window (not a worktree session).
