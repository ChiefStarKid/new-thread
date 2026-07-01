# new-thread — Claude Code skill

> **Requires Claude Code Desktop.** The skill calls `spawn_task`, which creates the suggested-task chip — a Desktop-only feature. It won't work in the terminal CLI or web. Examples use Windows paths.

**[▶ Try the interactive demo](https://chiefstarkid.github.io/new-thread/)** — click through the full `/new-thread` + `/loop-back` flow in your browser.

If you're working in a long Claude Code session and a new task comes up — one that shouldn't interrupt what you're doing, and shouldn't inherit all the context you've built up — this skill handles the handoff.

`/new-thread [objective]` reads the current session, extracts only the context the new task needs, and calls `spawn_task` to launch a fresh parallel Claude session as a chip. You stay in your current thread. The new one starts focused, not burdened.

This is not a copy-paste workflow. It stages a real parallel session — a separate Claude Code instance, seeded with the minimum context it needs — that you launch with one click on the chip.

---

## Quick start

1. Copy `new-thread.md` into `C:\Users\<your-username>\.claude\commands\`
2. In any Claude Code Desktop session, type `/new-thread [what you want the new thread to do]`
3. A chip appears in the sidebar — click to open the spawned session, or dismiss it

`/new-thread` is a Claude Code custom command — a `.md` file that Claude reads and executes as a skill. No installation script, no configuration beyond setup.

When the spawned session finishes, use `/loop-back` from within it to send the findings back as a message-turn directly into the root session.

---

## What you actually get

Strip away the theory and here is what the skill does for you, concretely:

1. **You stay in your current session.** Firing `/new-thread` does not move you. It stages the off-shoot as a chip; you start it with a click and it runs elsewhere, while your live conversation continues uninterrupted. (This is the opposite of `/fork`, which makes *you* leave for the branch.)

2. **Claude writes the hand-off brief for you.** Without the skill, spinning off a related task means either opening a blank chat and re-explaining everything, or dragging the entire history along. `/new-thread` reads the current conversation and assembles the brief itself — you type one line, Claude works out what the spawn needs to know. You skip the re-explaining.

3. **It runs in parallel.** Once started, the chip is a genuinely separate, concurrent Claude session. Stage three in ten seconds, start them, and they all work at once while you carry on. `/compact` and `/fork` are both single-threaded — one conversation at a time.

4. **`/loop-back` carries the answer home automatically.** When the spawn is done, you don't copy-paste its conclusion back by hand. `/loop-back` sends the findings as a message-turn directly into the root session — it lands as an actionable turn in the root's conversation, ready to continue from.

That is the proven part — the workflow ergonomics. Stay put, auto-written brief, parallel execution, automatic return. None of it depends on a theory being right.

---

## Why it might also be cheaper (a working hypothesis)

There is a second, bolder claim — that off-shooting is also more *token-efficient* than the usual alternative. This part is a **working hypothesis**, not a measured result. Treat it as a reasoned bet, separate from the ergonomic wins above.

The reasoning: Claude reads the whole conversation on every turn. That's by design and it's a strength — Claude reasons over everything you've said. The cost is that every turn pays for the entire history in tokens, and real work is seldom linear. A long session sprouts off-shoots — a bug noticed mid-flow, a tangent, a question — and each one pursued in the same window both inflates the context every future turn must carry and bleeds its framing into a conversation that was about something else.

The usual remedy is compaction: summarise and continue from the summary. The hypothesis is that **compaction is too blunt an instrument**. It flattens a branching conversation into a lossy précis, discards the distinctions between threads, and may cost more tokens in the long run as Claude re-derives context the summary dropped. `/new-thread` instead keeps each conversation focused by construction — every off-shoot carries only the slice it needs, and `/loop-back` returns just the conclusion, not the transcript.

**This has not been quantified.** The next action is a study: take one long, branching session, run it two ways — compacted in place versus split via `/new-thread` + `/loop-back` — and compare total tokens to reach the same end state. Until then, the efficiency claim is a bet, not evidence. If you run the numbers, open an issue — results welcome, including ones that disprove it.

---

## How it compares to /compact and /fork

Claude Code already ships two ways to manage a session that has grown too big or branched off-topic. `/new-thread` is a third option with a different trade-off. If you're deciding between them:

| | `/compact` | `/fork` | `/new-thread` |
|---|---|---|---|
| **Result** | Same session continues, shorter | New session, branched at a point | New session, runs in parallel |
| **Context carried over** | Lossy summary of *everything* | Full history up to the fork point | Synthesised *minimum* the task needs |
| **Original session** | Replaced by its summary | Untouched, but you leave it | Untouched, you stay in it |
| **Runs alongside?** | No — you continue in one thread | No — you switch to the fork | Yes — a chip runs concurrently |
| **Off-topic pollution** | Reduced, not removed — the summary still carries every thread | Inherited in full — the fork starts with all the noise | Eliminated — the spawn starts clean on one task |
| **Returns findings?** | N/A — never left | Manual — you carry them back yourself | `/loop-back` delivers a message-turn into the root session |

**The short version:**

- **`/compact`** fixes *length*. It shrinks the current conversation but keeps you in it, and the summary still blends every thread together. Good when you want to keep going on the *same* task and just need headroom.
- **`/fork`** branches the *whole* context. The new session inherits everything up to the fork point — fast to start, but it carries all the accumulated noise with it, and you've left your original thread behind. Good when the new work genuinely needs the full history.
- **`/new-thread`** extracts *only what the off-shoot needs* and runs it in parallel. Slower to set up (Claude has to synthesise the brief), but the spawn starts clean, your original thread stays live and untouched, and `/loop-back` brings just the conclusion back. Good when the off-shoot is a *different* task that shouldn't drag the whole conversation with it.

Put differently: `/fork` asks "what if I copied this entire conversation and kept going?" `/new-thread` asks "what's the smallest brief that lets a fresh session do this one thing, while I carry on here?"

![Three ways to handle an off-shoot: /compact summarises one thread in place; /fork branches the whole context and you leave the original behind; /new-thread keeps your main thread unbroken while a long independent spawn does the heavy work and only the answer returns.](assets/workflow-comparison-dark.svg)

---

## Who this is for

- **Anyone managing complex multi-part work in Claude Code** — where tasks branch mid-session and context discipline matters
- **Developers and analysts running parallel experiments** — the hub-and-spoke pattern: one coordinating session, multiple spawns each holding only their slice of context
- **Anyone hitting Claude's context limits in long sessions** — and wanting to split work across focused threads rather than fight the context window

---

## How it works

1. You type `/new-thread [objective]` — or just `/new-thread` and Claude asks
2. Claude classifies your brief by intent (bug fix, ideation, research, clarification, or task) and extracts the relevant context from the current session — not a dump, an intelligent synthesis
3. Claude calls `spawn_task` — staging a parallel Claude session, not a prompt to copy-paste
4. A **Suggested task** chip appears at the top-right of the conversation — click **Start locally** to spin it off into its own session, or dismiss it
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

`/loop-back` closes the loop. Run it from inside a spawned session when the work is done — it synthesises the findings and delivers them as a message-turn directly into the root session via `send_message`. The findings land as an actionable turn inside the root's conversation, not as a separate chip to click.

**Note:** `/loop-back` cannot place a chip into another session — `spawn_task` only creates chips in the *current* session. The message-turn mechanism is the correct design: the root gets the answer inline and can act on it immediately.

The message is shaped by what the root needs to do:

- **Findings** — question answered; root needs to act on the answer
- **Deliverable ready** — artefact produced; root needs to review or deploy
- **Blocked** — hit a wall; root needs to decide or unblock
- **Decision needed** — options surfaced; root needs to choose

**`/loop-back` only works in sessions spawned by `/new-thread`.** Each spawned prompt includes a `Thread title:` field that `/loop-back` uses as a stable search anchor — if the root session has been renamed since spawning, the transcript search finds it anyway. If fired from a rootless session it will error and offer a manual override.

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

**Note on session ID lookup:** each child prompt now includes a `Thread title:` field — the title baked in at spawn time. `/loop-back` first tries to match the root by title via `list_sessions`. If the root has been renamed since spawning (titles drift as conversations evolve), it falls back to searching transcripts for `Thread chip created: "<thread title>" (root: <id>, <date>)` — a phrase logged by the root at spawn time that is immutable. This makes the return path robust to title drift without requiring the root session to know its own `local_` ID.

**Known failure mode (fixed):** earlier versions of `new-thread.md` could write explanatory placeholder text (e.g. "unknown — could not confirm title") into `Root session title`/`Root session ID` instead of omitting the field when the runtime title wasn't visible, and the confirmation line in `new-thread.md` Step 7 was itself a verbatim match for the search query `loop-back.md` uses — so the transcript search could return the skill doc's own instructions instead of a real spawn event. Both are fixed: unconfirmable fields are now omitted rather than hedged, and the confirmation line is stamped with the root ID and date so it can't collide with the doc's own template text.

---

## Issues / feedback

Open an issue on this repo. For other enquiries, reach me on [LinkedIn](https://www.linkedin.com/in/joseph-solomon-%E6%88%B4%E4%BC%81%E5%BA%86-376156160/) or at joseph@kainosis.com.
