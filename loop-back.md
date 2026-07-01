---
name: loop-back
description: Synthesise this session's findings and deliver them as a message-turn into the root session that spawned it. Only works in sessions created by /new-thread.
---

# /loop-back — Return findings to the root session

Closes the loop opened by `/new-thread`: synthesises what this session produced and delivers it as a message straight into the root session that spawned it.

**This delivers a message-turn into the root, NOT a chip.** No tool can place a chip into another session — `spawn_task` only creates chips in the *current* session, and `send_message` (the only tool that targets another session by ID) delivers a user turn labelled "From `<this title>`" with a backlink. A message-turn is the right mechanism anyway: the findings land inside the root's conversation as an actionable turn, rather than as a separate spawn the user must click to start. Do not try to route a chip to the root — it is not possible.

**Do NOT enter plan mode. Execute immediately.**

---

## Step 0 — Precondition: not in bypass mode

`send_message` always requires interactive confirmation and will fail silently in bypass-permissions mode. Detect and fix it before doing any synthesis work.

**Detect:** Read `~/.claude/settings.json`. If `"bypassPermissions": true` is present, bypass mode is on via the settings file.

**If bypass mode is ON:**

Call `AskUserQuestion` with one question:
- Question: "Bypass-permissions mode is on, which blocks send_message. Switch it off now so loop-back can send?"
- Header: "Bypass mode"
- Options: `["Yes — switch it off", "No — cancel loop-back"]`

If the user selects **Yes**: Edit `~/.claude/settings.json` and set `"bypassPermissions": false` (or remove the key). Then continue to Step 1.

If the user selects **No**: exit cleanly with no further output.

**If bypass mode is NOT detected in the settings file:** proceed — bypass may still be active via the UI toggle, but Step 5 will catch it. If Step 5 fails with an unsupervised-mode error, report it and tell the user to click the shield icon in the session header to turn off bypass, then re-run `/loop-back`.

---

## Step 1 — Detect root session

Scan the conversation context for the two fields injected by `/new-thread`:

```
Root session ID: <id>
Root session title: <title>
```

**Treat a field as missing if the line is absent, OR if its value is placeholder/hedge text rather than a real title or ID** (e.g. contains `unknown`, `could not confirm`, `resolve live`, `n/a`, or is otherwise not a usable literal value). A placeholder string is not a valid handle for `list_sessions` matching and must fail the same way an absent line does — do not attempt to use it as-is.

**If both are missing (by the rule above) → hard error:**

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

If the user selects **Yes**: ask a follow-up `AskUserQuestion` for the root session title (free text), then continue — Step 3 will resolve the live ID from the title.

If the user selects **No**: exit cleanly with no further output.

---

## Step 2 — Load the session tools

Call `ToolSearch` with `{ "query": "select:mcp__ccd_session_mgmt__send_message,mcp__ccd_session_mgmt__list_sessions,mcp__ccd_session_mgmt__search_session_transcripts", "max_results": 3 }`.

---

## Step 3 — Resolve the LIVE session ID

**Do not trust the `Root session ID` from the header as-is.** It is captured from the JSONL transcript filename (a raw UUID), but `send_message` requires the `local_`-prefixed session ID from `list_sessions` — a different namespace. The header ID will not resolve directly.

Resolve the real, currently-live ID:

1. Call `list_sessions` and look for an entry whose `title` matches `Root session title` from the header.
2. Titles drift — a session can rename itself mid-run. If no title matches, call `search_session_transcripts` with the query `Thread chip created: "<thread title>"`, where `<thread title>` is the `Thread title:` field from **this session's own prompt header**. That exact phrase was logged by the root when it spawned this thread and will not change regardless of how the root session renamed itself since. The matching session's `sessionId` is the root.
   - **Caveat:** `new-thread.md` Step 7's own instructions contain this same phrase as a literal template example, and that skill doc's text can itself surface in transcript search. Discard any hit whose surrounding content is the `new-thread.md`/`loop-back.md` skill instructions (recognizable by nearby headers like `## Step 7 — Confirm` or `# /loop-back —`) rather than an actual chat turn — it is not a real spawn event. Prefer hits that also include a `(root: ..., <date>)` suffix, which only appears in genuine runtime confirmations (post-fix), not in the doc template.
3. Prefer a session with `isRunning: true` when there is ambiguity.
4. If nothing resolves, report it and stop — do not send to the header ID blindly.

Use the resolved `local_…` `sessionId` for the send in Step 5.

---

## Step 4 — Write the return message

First, recall **why the root spawned this thread** — the original brief is in this session's prompt header (the `Task:` / research-brief / bug-report block). The message must answer *that*, not narrate how this session went.

Write a short, plain-English message — 3 to 6 sentences max. No headers, no structured fields.

Lead with the **answer or result the root was waiting for**. Then give the root only what it needs to act:
- The finding, decision, or deliverable (with file path if there is one)
- What the root should do next

**Do not include** meta-commentary about this session's own process — tools that didn't work, debugging detours, dead ends, or how many tries it took. The root doesn't care how the sausage was made; it cares about the answer.

Write it as you would a Slack message to a colleague who already knows the project and asked you this specific question. Tight and direct.

---

## Step 5 — Call send_message

Before calling the tool, output this line:

```
Sending findings back to "<root session title>" — you'll be asked to confirm.
```

Then call `mcp__ccd_session_mgmt__send_message` with:
- `session_id`: the resolved `local_…` ID from Step 3
- `message`: the message written in Step 4

Note: `send_message` always requires user confirmation. If the call is rejected with an unsupervised-mode error (bypass was active via UI toggle, not the settings file, so Step 0 didn't catch it), tell the user to click the shield icon in the session header to turn off bypass, then re-run `/loop-back`.

---

## Step 6 — Confirm

After the tool succeeds, output one line:

```
Done — findings delivered to "<root session title>".
```

If the tool fails, report the error. Do not ask the user to retry manually — troubleshoot first.
