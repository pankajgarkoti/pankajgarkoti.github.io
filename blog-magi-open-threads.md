---
layout: default
title: "Open threads: how the mAGI agent keeps track of what is still open"
---

# Open threads: how the mAGI agent keeps track of what is still open

![mAGI](https://raw.githubusercontent.com/pankajgarkoti/magi/v0.3.1/docs/logo/logo-colour-light.svg)

*mAGI 0.3.1 · November 2026 · [source](https://github.com/pankajgarkoti/magi) · [release](https://github.com/pankajgarkoti/magi/releases/tag/v0.3.1)*

In mAGI you talk to one AI agent in a terminal, and it starts other agents for separate jobs. Over a day that adds up to a lot of loose ends: a question it asked that you never answered, a job waiting on someone else, an agent that stopped halfway. An agent that only remembers the current conversation loses most of these. Open threads are how mAGI keeps them.

## What an open thread is

One line with a checkbox in today's note:

```
## Open threads
- [ ] 14:10 · release notes: waiting on you to pick the download host
- [ ] 16:42 · docs-fix (Claude Code) was closed after 61m idle · resume with `cd ~/code/docs && claude --resume 3f9c…`
```

The notes are plain Markdown files in `~/notes`, one per day. The agent adds a line whenever something is left open: a decision you haven't made, work that is parked, something it is waiting for. Longer context (what was on the screen, what was tried) goes into a second file next to the note, linked from the line, so the list stays short.

To close a thread, tick the box. You can do that in any editor. Nothing else needs to happen.

## Agents that were closed while idle

Each agent window holds a few hundred MB of memory, so mAGI closes agent windows that have been idle for an hour (sooner when memory is low). It never closes one that is asking you something or has text typed into it. Before 0.3.1, a closed window was written to a log and that was it. If the agent had stopped halfway through something, nobody would notice.

Now every closed window becomes an open thread, with the command that reopens that exact conversation and the last lines of its screen. The window is gone, but the work is on the list until you decide what to do with it.

## The reminder

Every three hours (00:30, 03:30, and so on, local time) the agent gets a reminder with the number of threads still open. It reads them and asks you about each one in plain words: resume it, drop it, or wait. When you answer, it ticks the box. If the computer was off at reminder time, the reminder runs at the next boot.

The reminder goes through mAGI's event bus, which only types into the agent's terminal when the agent is idle and you aren't typing, so it never interrupts a turn or a half-written message.

## Where it is stored

The checkboxes in the daily notes are the record you see. The agent also keeps a small SQLite database in `~/notes/orchestrator/` with each thread, when it was raised, and when it was closed; ticking a box closes it there too. The notes folder is a local git repository, committed every fifteen minutes, and nothing is uploaded anywhere.

From a terminal:

```
orch-thread add "waiting on the hosting quote"     # add one (details on stdin)
orch-thread scan                                   # everything still open
```

## Also in 0.3.1

- **Status panel.** ALT + M shows a read-only panel with the state of everything mAGI runs: the orchestrator and its agents, the listener, the event bus, the agent browser, the timers, the notes, memory and disk. If the panel window or a terminal for it doesn't exist, ALT + M creates it.

Download, checksum and release notes: [github.com/pankajgarkoti/magi/releases/tag/v0.3.1](https://github.com/pankajgarkoti/magi/releases/tag/v0.3.1)
