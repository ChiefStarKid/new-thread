# new-thread — Claude Code skill

If you're trying to spin off a background task from within a Claude Code session without opening a new chat manually, this skill does it in one command.

`/new-thread` creates a chip — a background session card — that the user can launch, monitor, or dismiss, without losing the context of the current session.

---

## Who this is for

Claude Code Desktop users (Windows) who work across multiple parallel tasks and want to delegate work to a new session without context-switching.

---

## What it does

1. Asks for a brief (or reads it from the command argument)
2. Constructs a self-contained prompt for the spawned session, including today's date and the task
3. Calls `spawn_task` to create the chip
4. Confirms with one line in chat

The spawned session picks up identity and project context from your `CLAUDE.md` automatically — no duplication needed.

---

## Prerequisites

- **Claude Code Desktop** — requires the `mcp__ccd_session__spawn_task` tool, which is only available in Claude Code Desktop
- **Windows** — skill and memory paths use Windows conventions
- A `CLAUDE.md` at `~/CLAUDE.md` with your identity and project context (the spawned session reads this on startup)

---

## Setup

**One-time:**

1. Copy `new-thread.md` into your Claude Code commands folder:
   ```
   C:\Users\<your-username>\.claude\commands\new-thread.md
   ```

2. Ensure `~/CLAUDE.md` exists and contains your identity, preferred behaviour, and any project context you want spawned sessions to inherit.

That's it. No API keys, no environment variables.

---

## Usage

```
/new-thread fix the auth bug in the login flow
```

or just:

```
/new-thread
```

Claude will ask what the new thread should work on, then create the chip.

**Output:**
```
Thread chip created: "Fix auth bug in login flow"
```

The chip appears in the Claude Code Desktop sidebar. Click to open the session, or dismiss it if you change your mind.

---

## Example chip prompt (what the spawned session receives)

```
Date: 2026-06-26

Task: fix the auth bug in the login flow

Follow CLAUDE.md before doing anything.
```

The spawned session loads `CLAUDE.md`, picks up your identity and project context, and gets to work.

---

## Issues / feedback

Open an issue on this repo or email joseph.solomon@thirdsight.net.
