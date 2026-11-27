---
layout: default
title: "How a dozen agents share one desktop"
---

*November 2026. Omarchy makes Linux beautiful for humans; mAGI makes Linux a workstation for agents. This is how the workstation runs a dozen agents at once without them stepping on each other.*

mAGI's main interface is one terminal agent running in tmux. You speak or type to it, and it starts other agents behind it: workers in their own tmux windows, a browser on pages you are already signed in to, timers, a health checker. Everything below exists to keep that shape usable when the count goes from one agent to twelve. There are four pieces: bodies, a ledger, a nervous system, and hands.

## Bodies: tmux windows, one agent each

Every agent gets a body, and the body is a tmux window. The session is called `magi`. Window 1 is the orchestrator — the agent you talk to. Every other window is a worker, a monitor, or a service pane: a build agent, a browser window, a `bus` window tailing the event log, a status panel. A window holds exactly one agent process, started in a working directory, resumed by session id.

tmux is the common layer, and that choice is load-bearing. `send-keys`, `capture-pane`, window layout, and pane liveness work the same for every CLI. What differs per agent — where it keeps idle state, how to resume it, how to read its input box — lives in one adapter file per agent, in `agent/lib/magi_agents.py`. Claude Code keeps a session file, opencode answers on a local server API, Codex keeps a rollout file. The default orchestrator is opencode; Claude Code, Codex, or a custom command stay one adapter away, chosen with `AGENT` and `AGENT_CMD` in `~/.config/magi/agent.conf`.

Windows survive most things except a reboot. So the layout is saved every minute — window names, folders, and each agent's resume command — and rebuilt at login. Every restored pane runs its own resume command, so a restored window is the same conversation, not a picture of one. Restore is idempotent: a window is only created if its session does not already have one by that name. The snapshot and restore tools are `agent/bin/tmux-snapshot` and `agent/bin/tmux-restore`, wired as user systemd units.

A dozen idle sessions hold real memory — a few hundred megabytes each — so bodies are cheap to keep and cheap to close. Closing a window never deletes the session behind it; the resume command is logged with every close, and reopening is one command. That makes the policy simple: keep what is working, close what is done, and never pay memory for nostalgia.

## The ledger: rows as the source of truth

What the desktop looks like right now lives in tmux. What it means lives in the ledger: one SQLite file, `orch.db`, holding a row for everything the orchestrator runs or tracks. The ledger survives reboots, context compaction, and closed windows. On every fresh start the orchestrator reads the ledger first and resumes from it, not from memory.

The `agents` table holds every spawned agent window: name, pane, session id, directory, brief path, task, status, result, and resume command. `threads` holds open threads, mirrored in the daily note as a checklist. `watchers` holds standing watches with their last-seen-working timestamps. `decisions` holds the owner's decisions so nothing is asked twice. `deliverables` holds PRs, deploys, and releases. `events` mirrors the bus log; `nudges` holds timed messages typed into the orchestrator.

The CLI over the ledger is `orch`. `orch spawn NAME DIR BRIEF --task "one line"` starts an agent. `orch agent done NAME --result "…"` records its finish. `orch unfinished` lists spawned agents whose brief is not marked done. `orch thread add` opens a thread. `orch watch add` / `ok` / `end` tracks a watcher. `orch decide` logs a decision. `orch deliver` logs a deliverable. `orch sync` ingests new bus events, note ticks, and dead panes. `orch status` prints the whole picture on one screen. `orch up` starts everything. `orch doctor` checks every piece, fixes what is safe, and reports the rest on the bus.

The discipline is that everything durable goes through the ledger. Briefs are copied to dated files, never kept only in a scratchpad. Decisions are written down the moment they are made. Deliverables are logged when they land. A reboot can take the panes; it cannot take the rows.

## The lifecycle: spawn, start-check, done

Starting an agent is `orch spawn`, and it does four things at once: it copies the brief to a dated file and appends the finish footer, it opens a tmux window running the agent with a generated session id, it writes the ledger row with the pane, the session id, and the resume command, and it launches a background start-check.

The start-check exists because a first message typed too early is silently lost. The check waits for the TUI to boot, answers the folder-trust prompt if one appears, and delivers the brief exactly once. Delivery prefers the agent's own API where one exists and falls back to typed `send-keys` otherwise. Then it confirms via server truth and reports `spawn NAME: started OK` on the bus. If the TUI never reaches the input box, or the brief was accepted but cannot be confirmed, the report says so plainly instead of pretending.

Finishing is explicit. Every spawned brief ends with the same footer: when the brief is finished, the agent runs `orch done NAME "<one-line result>"`, which marks the ledger row done and emits `NAME "DONE: …"` on the bus. If the agent is blocked on the owner it emits `ASK OWNER: …` and keeps the rest going. Routine agents with no end — listeners, watchers — are spawned with `--standing` and stay untracked.

Between spawn and done, the ledger row is the contract. The `orch-agent-check` timer, every 30 minutes, puts one bus line listing every spawned agent whose brief is not marked done, and never types into any pane. `orch unfinished` shows the same list on demand. A task is finished only when its outcome is verified and recorded, never because the window went quiet.

Resuming a closed agent is the same path with history: `orch spawn NAME CWD "<follow-up>" --resume <session-id>` reopens the window on its ledger session id. The resume command is always at hand — in the ledger and in `orch agents --all`.

## Hands: tell, key, clear, model, restart

The orchestrator does not just start agents; it drives them. Five small commands cover nearly all of it, each one adapter-aware so the same verb works across CLIs.

`orch tell NAME TEXT` delivers text to an agent's input: for opencode through the prompt API with a typed fallback, for Claude Code as a paste. `orch key NAME KEY` sends a control key — `c-c`, `enter`, `up`, `down`, `escape` — without literal typing, for the moments when the pane needs a keystroke rather than words. `orch clear NAME` interrupts with `C-c` until the input box is empty, so a stale or jammed draft stops blocking later commands. `orch model NAME MODEL` switches the agent's model through a display-name filter, verified against the API. `orch restart NAME` respawns the pane on its ledger session id, which re-binds a stranded TUI to its server session; `--fresh` mints a new session id when a clean start is wanted, and `--model` queues a model switch after boot.

These are deliberately small. They do not edit files, they do not decide anything, and none of them blocks for more than a few seconds. The rule underneath is that driving is explicit and observable — one command, one effect, logged in the ledger or on the bus.

## The nervous system: the event bus

Twelve agents that all want attention are a noise problem. If each one types into the orchestrator whenever it finishes, the orchestrator never finishes a thought. The event bus is the answer: one append-only log, with delivery gated twice.

`orch-bus emit SOURCE "text"` appends one JSON line — timestamp, source, text, optional dedupe key — to `~/.local/state/orchestrator/events.jsonl`. No broker, no port. Anything that can run a command can publish: workers, shell scripts, systemd timers, browser probes. Event text is capped at 300 characters, so an event is a pointer ("build-2: tests failed, log at …"), never the log itself.

`orch-bus dispatch`, running as a user unit, watches the log. When undelivered events exist it waits a few seconds so a burst coalesces, then types them into the orchestrator's pane as one `[event bus] …` line — but only if two checks pass: the agent's own state reports idle, and the input box is empty. If either check fails the events stay queued and the read offset does not move. Worker events travel the same path: each spawned worker is watched, and "finished", "waiting at a prompt", and "exited" arrive through the same idle gate, naming the window plus the command to read its screen.

Every event carries its source — `agent build-2`, `page-title`, `orch-doctor`, `stall-watch`, `agent-finish` — and dedupe keys keep repeats quiet: a re-emit with an already-seen key does not deliver twice. With `BUS_DELIVERY=off` nothing is typed at all and the agent reads with `orch-bus unread`, which keeps its own offset. Urgent events additionally raise a desktop notification, so the owner is tapped on the shoulder while the agent still waits for idle. Browser probes attach over CDP to the agent browser and emit when watched pages change, so a page the owner is already signed in to becomes an event source with no scraping from the agent's side.

Because probes relay text written by other people, bus text is treated as untrusted input. Every event gets a trust level — owner, internal, external — and external text is presented only inside a fixed frame marked as data, never as instructions. High-risk events arrive as a one-line summary with the full text quarantined to a file. A circuit breaker stops all typing after repeated high-risk events until the owner resumes the bus.

## Watchers: stall, idle, and the ones that never close

Some agents stall: the pane is alive, the ledger says running, but nothing advances. Two watchers watch for that, and they divide the job so reports never duplicate.

`agent-stall-watch`, every 10 minutes, is the liveness checker. For each running ledger agent it reads server truth by session id, never the pane. A dead or missing pane is reported. A session the server calls running is working by definition — long builds go quiet — so its baseline refreshes and nothing fires. Otherwise, when the session is idle with output tokens and the server timestamp both static past twenty minutes, the agent is reported STALLED. A faster path covers the finished-but-unrecorded case: a turn with output settled over ninety seconds ago while the ledger row is still running is reported as IDLE, and the orchestrator decides whether to continue, close, or reap. Rows the reaper already flagged are skipped here. Every report is one change-only bus line, and the watcher never touches a pane.

`orch-agent-check`, every 30 minutes, is the finish checker. It lists every spawned agent whose brief is not marked done, with the last bus line and the resume path. It emits one bus line and never types into any pane.

The reaper, `tmux-reap-stale`, every 10 minutes, is the memory checker. It closes sessions idle over an hour — twenty minutes when memory is short — but never an unfinished agent: tracked agents that go idle without finishing are woken with the brief up to three times, then reported STALLED and left open. It never closes exempt windows, panes waiting on a prompt, or panes holding a draft. Every close is logged with its resume command and becomes an open thread in the daily note, so a reaped agent is a thread to resolve, not a task silently dropped.

Alongside them run the standing watchers in the `watchers` table: the open-threads nudge every three hours, the voice digest, the doctor, the browser and listener guards. `orch watches` lists them; the ledger remembers which ones are owed attention.

## TUI-first identity

Underneath all of this is an identity choice: the desktop boots into the agent's seat. The orchestrator starts at login and its console drops open over whatever is on screen, with every other agent window resumed around it. The window is a view onto a tmux session, not the session itself — close the terminal and nothing stops; open another and it attaches to the same running conversation.

The control centre's AGENTS page and the first-login setup carry the same stance: pick an agent, run its official installer, sign in with your own account. No agent is bundled. The default state of the machine is a running conversation, not an app launcher.

That is the whole system as it runs today: tmux windows for bodies, one adapter file per agent, a SQLite ledger for meaning, a spawn path that confirms delivery, explicit finish markers, five small driving commands, one quiet bus with two locks on delivery, watchers for stall and idleness and memory, and a desktop that boots into the conversation.
