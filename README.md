# new-thread — Claude Code skill

If you're working in a long Claude Code session and a new task comes up — one that shouldn't interrupt what you're doing, and shouldn't inherit all the context you've built up — this skill handles the handoff.

`/new-thread [objective]` reads the current session, extracts only the context the new task needs, and calls `spawn_task` to launch a fresh parallel Claude session as a chip. You stay in your current thread. The new one starts focused, not burdened.

This is not a copy-paste workflow. It spawns a real parallel session — a separate Claude Code instance running concurrently, seeded with the minimum context it needs to do its job.

---

## Quick start

1. Copy `new-thread.md` into `C:\Users\<your-username>\.claude\commands\`
2. In any Claude Code Desktop session, type `/new-thread [what you want the new thread to do]`
3. A chip appears in the sidebar — click to open the spawned session, or dismiss it

`/new-thread` is a Claude Code custom command — a `.md` file that Claude reads and executes as a skill. No installation script, no configuration beyond setup.

When the spawned session finishes, use `/loop-back` from within it to fire the findings back as a context-loaded chip.

---

## The problem it solves

Long Claude Code sessions accumulate context. That's useful until it isn't — when a new task enters a session built for something else, token costs climb, reasoning gets polluted, and the original task loses its thread.

The standard fix was to let the new task interrupt the current session anyway. Both suffered: the context window fills faster, the original task stalls, and the new task inherits context it doesn't need.

`/new-thread` is a context window management tool. Each session carries only what it needs for its objective. Token costs stay low. Reasoning stays clean. The typical pattern: after a long session has built up significant context, fire `/new-thread` for anything that branches off — the new prompt is inherently lean because it only carries the relevant slice.

If you've ever wanted to manage Claude's context limits proactively, run multiple Claude sessions at once, or keep parallel Claude Code threads from bleeding into each other — this is the skill for that.

---

## Who this is for

- **Anyone managing complex multi-part work in Claude Code** — where tasks branch mid-session and context discipline matters
- **Developers and analysts running parallel experiments** — the hub-and-spoke pattern: one coordinating session, multiple spawns each holding only their slice of context
- **Anyone hitting Claude's context limits in long sessions** — and wanting to split work across focused threads rather than fight the context window

---

## How it works

1. You type `/new-thread [objective]` — or just `/new-thread` and Claude asks
2. Claude classifies your brief by intent (bug fix, ideation, research, clarification, or task) and extracts the relevant context from the current session — not a dump, an intelligent synthesis
3. Claude calls `spawn_task` — a live parallel Claude session, not a prompt to copy-paste
4. A chip appears in the Claude Code Desktop sidebar
5. You continue your current session or wait for the result — your call

The spawned session loads your `CLAUDE.md` on startup for identity and project context. The handoff prompt is intentionally lean: enough to complete the task accurately, no more. Every spawn also receives the root session ID and title so it knows where it came from.

---

## Intelligent brief classification

The skill doesn't treat all tasks the same. Before building the handoff prompt, Claude classifies your brief by intent and applies a matching template:

| Mode | When it applies | What the spawn gets |
|---|---|---|
| **Bug / defect** | Something is broken or throwing an error | Structured bug report: what you were doing, goal, symptom, reproducibility, root cause type |
| **Ideation** | Exploring options, no single right answer yet | Problem space, constraints, options already considered, expected output format |
| **Research / explain** | Answering a specific question | The question, available sources, how the answer feeds back |
| **Clarification** | Resolving an ambiguity without polluting this session | The ambiguity, options in play, what resolving it looks like |
| **Task** | Build, write, update, draft, run | Relevant decisions, artefacts, constraints, framing |

If intent is ambiguous, Claude asks — but only if resolving it here is cheaper than letting the spawn figure it out itself.

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

**Bug fix** — spotted an issue mid-session, don't want it derailing current work. Claude detects bug fix intent and switches to a structured handoff:
```
/new-thread fix the token refresh error we just saw
```
The spawned session gets a proper bug report — what you were doing, what you were trying to achieve, the symptom, reproducibility, and whether it's a technical or behavioural root cause.

**Hub-and-spoke** — current session coordinates, spawns each run an independent variant. Useful for parallel hypothesis testing where each spawn needs the shared structure but only its own hypothesis:
```
/new-thread test hypothesis A: momentum signal with 3-month lookback
/new-thread test hypothesis B: momentum signal with 6-month lookback
```

**Mass dispatch** — a long session has accumulated multiple unrelated threads. Clear them all at once, each into its own focused context:
```
/new-thread investigate the pipeline lag
/new-thread draft the follow-up email
/new-thread check repo privacy settings
```
Unrelated tasks, dispatched in seconds, each starting clean. No cross-contamination.

**Clarification** — a specific ambiguity needs resolving without polluting the current session:
```
/new-thread clarify whether the API rate limit applies per user or per account
```
The spawned session gets the ambiguity, the options in play, and what resolving it looks like. Use `/loop-back` when done to return the answer.

---

## Companion command: /loop-back

`/loop-back` closes the loop. Run it from inside a spawned session when the work is done — it synthesises the findings and fires a chip back to the root session, seeded with everything the root needs to continue.

The return chip is classified by what the root needs to do:

- **Findings** — question answered; root needs to act on the answer
- **Deliverable ready** — artefact produced; root needs to review or deploy
- **Blocked** — hit a wall; root needs to decide or unblock
- **Decision needed** — options surfaced; root needs to choose

**`/loop-back` only works in sessions spawned by `/new-thread`.** If fired from a rootless session it will error and offer a manual override.

Install: copy `loop-back.md` into the same `~/.claude/commands/` folder as `new-thread.md`.

---

## Prerequisites

- **Claude Code Desktop** — requires `mcp__ccd_session__spawn_task`, available in Claude Code Desktop only
- **Windows** — paths use Windows conventions
- A `CLAUDE.md` at `~/CLAUDE.md` with your identity, behaviour preferences, and project context

---

## Setup

One-time:

1. Copy `new-thread.md` (and optionally `loop-back.md`) into your Claude Code commands folder:
   ```
   C:\Users\<your-username>\.claude\commands\
   ```

2. Ensure `~/CLAUDE.md` exists and contains your identity and any context you want spawned sessions to inherit automatically. This is the only configuration the skill needs — no PII is hardcoded in the skill files.

No API keys. No environment variables.

**Note on session ID lookup:** the skill identifies the current session by finding the most recently modified JSONL file under `~/.claude/projects/`. It uses `$USERNAME` to locate the path — no hardcoding required. This works across different users and working directories on Windows.

---

## Issues / feedback

Open an issue on this repo or email joseph.solomon@thirdsight.net.
