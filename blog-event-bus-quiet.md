---
layout: default
title: "The event bus that made my agents stop shouting"
---

*November 2026. Omarchy makes Linux beautiful for humans; mAGI makes Linux a workstation for agents. This is the piece of the workstation I rewrote the most: the event bus.*

mAGI's main interface is one terminal agent running in tmux. You speak or type to it, and it starts other agents behind it: workers in their own tmux windows, a browser that watches pages you are already signed in to, timers, a health checker. The question that took me four years to answer properly is a small one: when one of those things finishes, how does the orchestrator find out?

## 2022: a chatbot people talked to

At Hear Us I built Joy, a conversational bot for wellbeing check-ins. It ran on Slack, then on WhatsApp: a state machine on top of Dialogflow, with Firestore for events and conversation memory. That was before ChatGPT. It was the first thing I built that people talked to instead of clicking through.

The same year I started keeping my dotfiles on GitHub. Neovim and the terminal. Everything below lives there.

Joy had the shape I keep rebuilding: one thing to talk to, with the work happening behind it. What it did not have was the reverse path. When something changed elsewhere, nobody told the conversation. The bot only knew what arrived in its input.

## 2023: one assistant, many specialists

At Mavex AI the product was Mavy, an executive assistant. The user talked to one thing; behind it a router sent each request to one of fourteen specialists: calendar, email, tasks, search and so on. Function calling was barely out, so the router was GPT-4 forced to return one of two JSON shapes: answer directly, or hand off. I wrote about it at the time, and about the search service behind it that picked its retrieval cutoff by clustering similarity scores instead of using a fixed threshold.

One assistant in front, specialists behind it. mAGI has the same shape.

The router pattern solves fan-out. It does not solve fan-in: fourteen specialists finishing at different times, and one conversation that needs to hear about each result exactly once, without being interrupted mid-sentence.

## 2024: the wishlist item

I pulled the agent loop out into AgentZero, a minimal tool-calling agent. Its README has a wishlist, and two items stayed with me:

- long-running processes where working memory is summarised as execution goes on
- a status check on a long task without interrupting it

Then PlanWeaver, a plan-and-execute agent, and ChainFactory: a small DSL for chaining LLM calls, with parallel steps and typed outputs, published on PyPI as `chainfactory-py`.

The second wishlist item is the event bus problem stated from the inside: an agent that is busy should be able to learn that something happened without losing its place. Polling from inside the session is the obvious answer and the wrong one. Every poll is a turn. Every turn spends tokens and fills the context with "nothing new". Interrupting is worse: text lands in the middle of a turn, or on top of something the user is typing.

## 2026: agents in tmux, and the shouting

Most of the agent work is from this year. In January, multiclaude: several Claude Code instances working on features in parallel, each in a tmux window, coordinated by a supervisor, in plain Bash. In May, Soren: a swarm of CLI agents, each in its own tmux window, with a shared mailbox, a router daemon, a health monitor and a journal. In November, bridge: you talk to one Claude Code session, by voice or text, and it runs the other agents in tmux.

By bridge, the pieces were all there: one agent in front, workers in tmux windows, voice, a browser agents can drive. They were spread across repos and each needed setting up.

mAGI puts them in one install. And the moment you put five agents in tmux windows behind one orchestrator, the notification problem gets loud. Every worker that finishes wants attention. Every watcher that fires wants attention. If each one types into the orchestrator whenever it feels like it, the orchestrator never finishes a thought. That is what I mean by shouting.

## The design: one log, delivery gated twice

The bus as shipped is described in `docs/event-bus.md` in the magi repo, and the shape has been stable since the 0.2.0 release on 15 November 2026:

1. **One append-only log.** `orch-bus emit SOURCE "text"` appends one JSON line (`ts`, `at`, `source`, `text`, optional dedupe `key`) to `~/.local/state/orchestrator/events.jsonl`. No broker, no port. Anything that can run a command can publish: workers, shell scripts, systemd timers, browser probes. Event text is capped at 300 characters, so an event is a pointer ("tests failed in build-2, log at …"), never the log itself.

2. **Idle-gated delivery.** `orch-bus dispatch`, running as the `orch-bus.service` user unit, watches the log's size. When there are undelivered events it waits 3 seconds so a burst coalesces, then types them into the orchestrator's pane as one `[event bus] …` line — but only if two checks pass: the agent's own state reports idle, and the input box is empty. If either check fails, the events stay queued and the read offset does not move. Delivery is `tmux send-keys -l` followed by a separate Enter.

3. **Idle means what the agent says it means.** There is one adapter file per agent, in `agent/lib/magi_agents.py`: where it keeps idle state, how to resume it, how to read the input box. Claude Code's session file, opencode's local server API, Codex's rollout file. Idle comes from the agent's own state, never from guessing at the pane. The input-box check is per agent too: for opencode it reads the run of box-drawing lines above the mode line, for Claude Code the line after its prompt character. A half-typed message is never overwritten.

4. **Typed sources and a pull mode.** Every event carries its source (`agent build-2`, `page-title`, `orch-doctor`, …). With `BUS_DELIVERY=off` in `~/.config/magi/agent.conf`, nothing is typed and the agent reads with `orch-bus unread`, which keeps its own offset. Read-only mode for cautious days.

5. **Worker events on the same path.** `magi-agent spawn` watches each worker it starts and delivers "finished", "waiting at a prompt" and "exited" through the same idle gate, then logs them on the bus marked `delivered`. The report names the window — `[magi] window fix-login (opencode) finished and is idle` — plus the command to read that window's screen. The orchestrator never polls.

6. **Probes from the browser.** `browser-bus` attaches to the agent browser over CDP (`127.0.0.1:9222`) and runs small probe scripts inside matching tabs, in an isolated world. A probe declares `@match`, the `@source` names it may emit under, and a purpose, and calls `emit({text, key})`. The shipped example, `agent/browser-bus/examples/page-title.js`, emits when a local dev server's page title changes. The page you are already signed in to becomes an event source, with no API key and no scraping from the agent's side.

7. **A ledger.** `orch` keeps agents, open threads, watchers, decisions and deliverables in SQLite, so the state survives reboots and context compaction. `orch status` prints the whole picture on one screen.

8. **A guard and a breaker.** Once probes relay messages written by other people, bus text is untrusted input to an LLM. The guard (PR #1, merged) gives every event a trust level (owner, internal, external), normalises the text, scores it for injection patterns, and presents external text only inside a fixed frame marked as data, not instructions. High-risk events are delivered as a one-line summary and kept in full in a quarantine file; only allow-listed probe sources can emit external source names. A circuit breaker stops all typing after repeated high-risk events until the owner runs `orch-bus resume`. You can read the corpus and the results: `tests/agent/bus_guard/` holds the classifier sets and the dispatch and end-to-end tests.

That is the whole bus: a file, a gate with two locks, a fallback reader, and a guard at the door.

## The 2s-to-15s lesson

For a while the dispatcher polled every 2 seconds. The change to 15 seconds is one line in `agent/bin/orch-bus` — `time.sleep(15)` where the `2` was — and it is the kind of change that looks like nothing and teaches everything.

The reasoning, recorded in the commit and in PR #11: delivery is gated on the agent's own idle state and on input-box emptiness, and emitters only fire on change. So most 2-second wakeups found nothing to do. They were pure churn — the dispatcher waking up, checking the offset against the file size, going back to sleep. Fifteen seconds keeps the same behaviour when something actually lands, because delivery tolerates delay by design: events queue, the 3-second burst window coalesces them, and the offset only moves after delivery. Worst case the orchestrator hears about a finished worker a dozen seconds later. It was never waiting with a stopwatch; it was busy.

A companion commit fixed the comment above the sleep, which had claimed the loop was event-gated when it is plainly poll-based. I like that commit. Comments that lie about the mechanism are how the next person tunes the wrong knob.

## --urgent, and who gets told first

Someone asked who gets told first when something matters: me, or the orchestrator. The honest answer had been the orchestrator, always — everything queued for the next idle window. That is right for CI results and page changes. It is wrong for the thing that cannot wait.

PR #10 added `orch-bus emit --urgent`. The event still queues for idle-gated delivery to the agent, exactly like a normal event, but it additionally raises a desktop notification via `notify-send`. Backward compatible by default, guarded when `notify-send` is unavailable, and dedupe still applies: a re-emit with an already-seen `--key` does not raise a second notification. The end-to-end test in `tests/agent/bus_guard/test_e2e.py` covers exactly that: the urgent flag lands in the JSON, `notify-send` fires once with a sane title, and a deduped re-emit stays silent.

Normal events wait for idle; urgent ones also tap you on the shoulder. The distinction is in one flag, and the test pins it down.

## The prompt-API path, and logging the failures

PR #9 changed how events reach an opencode orchestrator. Typing with `tmux send-keys` works for every agent, but it can garble or interleave when the pane is busy redrawing. For opencode, the dispatcher now prefers `POST /api/session/{id}/prompt` on the agent's local server: the prompt queues server-side instead of being typed into the box. The tmux path stays as the fallback for other agents and for when the API is unreachable.

PR #12 is the follow-up I am glad got written down. The API path used to swallow failures in a bare `except`: if the POST failed, nothing said so, and nobody could tell which path had delivered. Now it logs the return code and the stderr snippet through the module logger, so the operator can see `prompt API failed … falling back to tmux` in the log. And it documents why the API path needs no draft guard: a server-side queued prompt cannot clobber a half-typed line, because nothing is typed. The guard exists for the mechanism that needs it, and only there.

Small fixes, each with a test or a log line to prove it. That is the whole engineering culture I want: the mechanism states its assumption, and the assumption is checked where it matters.

## What the bus does not do

It does not prioritise. Everything pending arrives as one line — `[event bus] 14:28 build-2: tests passed | 14:29 page-title: Build OK` — and the agent reads it like typed input. Urgent is the only priority, and urgent only changes who else is told.

It does not guarantee latency. Fifteen seconds plus the 3-second coalesce window is the contract. If you need faster, you are standing next to the machine anyway; read the log with `tail -f ~/.local/state/orchestrator/events.jsonl` or `orch-bus live`.

It does not trust the network. Anything from outside arrives framed as data, quarantined when it looks hostile, and the breaker stops the typing until a human resumes it. Detection on the blind set was 55% — the number is in the repo's test results — and the design assumes the detector misses. The frame is what holds, not the score.

## Timeline

| When | What | Where to check |
|---|---|---|
| 11 Nov 2026 | Workers report finished / asking / exited, idle-gated | commit `c64be03` |
| 15 Nov 2026 | Bus, browser-bus probes and agent browser ship as defaults | commit `aa752e9`, tag `v0.2.0` |
| 16 Nov 2026 | Prompt-injection guard and circuit breaker | PR #1 |
| 23 Nov 2026 | opencode becomes the default agent; tmux stays the common layer | PR #7 |
| 26 Nov 2026 | opencode prompt-API delivery; `--urgent` notifications | PR #9, PR #10 |
| 27 Nov 2026 | Dispatch poll 2s → 15s; API failure logging + no-draft-guard note | PR #11, PR #12 |

The pattern I keep finding: one thing to talk to, other things doing the work behind it, and something that remembers what is still open. The bus is the hearing for that shape. It took four years of building the mouth before I built the ears properly: wait for idle, check the input box, deliver once, write down the offset, and never shout.
