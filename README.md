# new-thread — Claude Code skill

If you're working in a long Claude Code session and a new task comes up — one that shouldn't interrupt what you're doing, and shouldn't inherit all the context you've built up — this skill handles the handoff.

`/new-thread [objective]` reads the current session, extracts only the context the new task needs, and calls `spawn_task` to launch a fresh parallel Claude session as a chip. You stay in your current thread. The new one starts focused, not burdened.

This is not a copy-paste workflow. It spawns a real parallel session — a separate Claude Code instance running concurrently, seeded with the minimum context it needs to do its job.

---

## The problem it solves

Long sessions accumulate context. That's useful until it isn't — when a new task threads into a window that was built for something else, token costs climb, reasoning gets polluted, and the original task loses its thread.

The old answer was to let the new task interrupt the current session anyway. Both suffered.

`/new-thread` separates them. Each session carries only the context it needs for its objective. The token budget goes further. The reasoning stays clean.

If you've ever wanted to run multiple Claude sessions at once, delegate a subtask without losing your place, or keep parallel Claude Code threads from bleeding into each other — this is the skill for that.

---

## Who this is for

- **Anyone managing complex multi-part work in Claude Code** — where tasks branch mid-session and context discipline matters
- **Developers and analysts running parallel experiments** — the hub-and-spoke pattern: one coordinating session, multiple spawns each holding only their slice of context
- **Anyone who has felt a long session get unwieldy** — and wanted to start fresh without losing the thread of what came before

---

## How it works

1. You type `/new-thread [objective]` — or just `/new-thread` and Claude asks
2. Claude reads the current session and synthesises the minimum context the new task needs: decisions made, artefacts produced, constraints, relevant framing
3. Claude calls `spawn_task` — this is a live parallel session, not a prompt to copy-paste
4. A chip appears in the Claude Code Desktop sidebar
5. You continue your current session or wait for the result — your call

The spawned session loads your `CLAUDE.md` on startup for identity and project context. The handoff prompt is intentionally lean: enough to complete the task accurately, no more.

---

## Usage patterns

**Fire and continue** — new task runs in parallel, you stay in the current session:
```
/new-thread draft the Q3 update based on what we just reviewed
```

**Wait for result** — spawn handles a subtask, you resume once it's done:
```
/new-thread check whether the auth fix we discussed breaks the refresh token flow
```

**Bug fix** — spotted an issue mid-session, don't want it derailing current work. Claude detects the bug fix intent and switches to a structured handoff: what you were doing, what you were trying to achieve, the symptom, whether it's reproducible, and whether the root cause is technical or behavioural:
```
/new-thread fix the token refresh error we just saw
```
The spawned session gets a proper bug report, not a vague brief.

**Hub-and-spoke** — current session coordinates, spawns each run an independent variant:
```
/new-thread test hypothesis A: momentum signal with 3-month lookback
/new-thread test hypothesis B: momentum signal with 6-month lookback
```
Each spawn gets the shared test structure plus only its hypothesis. Results come back to the hub.

**Mass dispatch** — a long session has accumulated multiple unrelated threads. Clear them all at once, each into its own focused context:
```
/new-thread investigate the heatmap pipeline lag
/new-thread draft the follow-up email to Alan
/new-thread check repo privacy settings
```
Unrelated tasks, dispatched in seconds, each starting clean. No cross-contamination.

**Clarification** — a specific ambiguity needs resolving but asking in the current session would pollute it. Spin off a focused thread to get the answer, then bring it back:
```
/new-thread clarify whether the API rate limit applies per user or per account
```
The spawned session gets the ambiguity, the options in play, and what resolving it looks like. You stay in the current session without the detour.

---

## Prerequisites

- **Claude Code Desktop** — requires `mcp__ccd_session__spawn_task`, available in Claude Code Desktop only
- **Windows** — paths use Windows conventions
- A `CLAUDE.md` at `~/CLAUDE.md` with your identity, behaviour preferences, and project context

---

## Setup

One-time:

1. Copy `new-thread.md` into your Claude Code commands folder:
   ```
   C:\Users\<your-username>\.claude\commands\new-thread.md
   ```

2. Ensure `~/CLAUDE.md` exists and contains your identity and any context you want spawned sessions to inherit automatically.

No API keys. No environment variables.

---

## Issues / feedback

Open an issue on this repo or email joseph.solomon@thirdsight.net.
