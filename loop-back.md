---
name: loop-back
description: Synthesise this session's findings and fire a chip back to the root session that spawned it. Only works in sessions created by /new-thread.
---

# /loop-back — Return findings to the root session

Closes the loop opened by `/new-thread`: synthesises what this session produced and fires a `spawn_task` chip addressed to the root session.

**Do NOT enter plan mode. Execute immediately.**

---

## Step 1 — Detect root session

Scan the conversation context for the two fields injected by `/new-thread`:

```
Root session ID: <UUID>
Root session title: <title>
```

**If either field is missing → hard error:**

```
/loop-back only works in sessions spawned by /new-thread.
This session has no root session ID in its prompt header.

Fire /new-thread from your main session to create a thread,
then use /loop-back from inside it.
```

Then call `AskUserQuestion` with one question:
- Question: "Do you want to supply a root session ID manually instead?"
- Header: "Manual root"
- Options: `["Yes — I'll paste the ID", "No — cancel"]`

If the user selects **Yes**: ask a follow-up `AskUserQuestion` for the root session ID (free text) and root session title (free text), then continue with those values.

If the user selects **No**: exit cleanly with no further output.

---

## Step 2 — Load the spawn_task schema

Call `ToolSearch` with `{ "query": "select:mcp__ccd_session__spawn_task", "max_results": 1 }`.

---

## Step 3 — Synthesise the return payload

Read the current conversation and extract:

- **What was produced** — files written, decisions made, findings confirmed, outputs generated
- **What the root session should do next** — the clearest next action, if determinable
- **Open questions or blockers** — anything unresolved that the root needs to handle
- **What was ruled out** — approaches tried and discarded, so the root doesn't re-tread them

Keep it tight. The root session has its own context; this is a handoff note, not a transcript dump.

---

## Step 4 — Build the chip prompt

Classify the return by what the root needs to do with it:

| Return type | Use when |
|---|---|
| **Findings** | This session answered a question or completed research; root needs to act on the answer |
| **Deliverable ready** | This session produced an artefact (file, draft, script); root needs to review or deploy it |
| **Blocked** | This session hit a wall; root needs to decide or unblock before work can continue |
| **Decision needed** | This session surfaced options; root needs to choose |

Build the prompt using the matching template:

### Findings / Research complete

```
Date: <today from currentDate system context>
Root session ID: <root session ID>
Root session title: <root session title>

Task: <one-line summary of what this session was asked to do>

Return type: Findings

Summary:
<what was discovered — 3–6 bullets>

Recommended next action:
<what the root session should do now>

Ruled out:
<approaches tried and discarded — omit if none>

Open questions:
<anything unresolved — omit if none>
```

### Deliverable ready

```
Date: <today from currentDate system context>
Root session ID: <root session ID>
Root session title: <root session title>

Task: <one-line summary of what this session was asked to do>

Return type: Deliverable ready

Deliverable:
<what was produced and where it lives — file path(s), sheet name, etc.>

What to do with it:
<review / deploy / merge / send — whatever the obvious next step is>

Open questions:
<anything the root needs to decide — omit if none>
```

### Blocked

```
Date: <today from currentDate system context>
Root session ID: <root session ID>
Root session title: <root session title>

Task: <one-line summary of what this session was asked to do>

Return type: Blocked

Blocker:
<what stopped progress>

What was tried:
<approaches attempted before hitting the wall>

What the root needs to do:
<decision or action needed to unblock>
```

### Decision needed

```
Date: <today from currentDate system context>
Root session ID: <root session ID>
Root session title: <root session title>

Task: <one-line summary of what this session was asked to do>

Return type: Decision needed

Options:
- <Option A>: <one-line case for it>
- <Option B>: <one-line case for it>
<add more if material>

Recommendation:
<which option and the one-sentence reason>

Stakes:
<what changes downstream depending on the choice — omit if low>
```

---

Close every prompt with:

```
Follow CLAUDE.md before doing anything.
```

---

## Step 5 — Derive title and tldr

- **`title`**: "← <root session title truncated to 45 chars>" — the arrow signals this is a return chip
- **`tldr`**: 1–2 sentences describing what came back and what the root needs to do. No file paths.

---

## Step 6 — Call spawn_task

Call `mcp__ccd_session__spawn_task` with:
- `title`: from Step 5
- `tldr`: from Step 5
- `prompt`: from Step 4

---

## Step 7 — Confirm

Output one line in chat:

```
Return chip fired to "<root session title>".
```

If the tool fails, report the error. Do not ask the user to retry manually — troubleshoot first.
