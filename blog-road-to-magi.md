---
layout: default
title: "How my projects ended up at mAGI"
---

# How my projects ended up at mAGI

*November 2026*

mAGI is a Fedora/Hyprland setup where the main interface is one LLM agent in tmux. Looking back at the repos, most of what I built over the last four years points at it. This post goes through them in order.

## 2022: a chatbot people talked to

At Hear Us I built Joy, a conversational bot for wellbeing check-ins. It ran on Slack, then on WhatsApp: a state machine on top of Dialogflow, with Firestore for events and conversation memory. This was before ChatGPT. It was the first thing I built that people talked to instead of clicking through.

The same year I started keeping my [dotfiles](https://github.com/pankajgarkoti/dotfiles) on GitHub. Neovim and the terminal. Everything below lives there.

## 2023: one assistant, many specialists

At Mavex AI the product was Mavy, an executive assistant. The user talked to one thing; behind it a router sent each request to one of fourteen specialists: calendar, email, tasks, search and so on. Function calling was barely out, so the router was GPT-4 forced to return one of two JSON shapes: answer directly, or hand off. I wrote about it [here](/blog-ai-task-router). The search service behind it picked its retrieval cutoff by clustering similarity scores instead of using a fixed threshold ([post](/blog-adaptive-rag-cutoff)).

One assistant in front, specialists behind it. mAGI has the same shape.

## 2024: agent loops and pipelines

I pulled the agent loop out into [AgentZero](https://github.com/computer-reinvention/AgentZero), a minimal tool-calling agent. Its README has a wishlist. Two items:

- long-running processes where working memory is summarised as execution goes on
- a status check on a long task without interrupting it

Then [PlanWeaver](https://github.com/computer-reinvention/PlanWeaver), a plan-and-execute agent, and [ChainFactory](https://github.com/computer-reinvention/ChainFactory): a small DSL for chaining LLM calls, with parallel steps and typed outputs, published on PyPI as `chainfactory-py`. The spec drafts are on this site.

## 2025: notes, and LLMs on the side of the code

[daily-notes.nvim](https://github.com/pankajgarkoti/daily-notes.nvim) is a daily notes plugin for Neovim: one file per day, year/month folders, headers carried over from the previous day. [doctr](https://github.com/pankajgarkoti/doctr) was a daemon that read git diffs and kept documentation in sync with an LLM.

Neither is an agent. Both are about not losing track of things, which is most of what mAGI's ledger and notes do now.

## 2026: agents in tmux

Most of the agent work is from this year.

- **January.** [multiclaude](https://github.com/computer-reinvention/multiclaude): several Claude Code instances working on features in parallel, each in a tmux window, coordinated by a supervisor. Plain Bash.
- **March.** [FlowPilot](https://github.com/computer-reinvention/flowpilot): browser automation that explores a site once with an agent, compiles scripts to replay it without LLM calls, and hands the live browser to a human over CDP when it hits a login or a CAPTCHA.
- **May.** [trie](https://github.com/computer-reinvention/trie): an in-repo index of what each symbol does and why it changed, enforced at commit time and exposed to coding agents. [Soren](https://github.com/computer-reinvention/soren): a swarm of CLI agents, each in its own tmux window, with a shared mailbox, a router daemon, a health monitor and a journal. Its README says to keep your copy private because the memories and journals get personal.
- **November 4.** [bridge](https://github.com/pankajgarkoti/bridge): you talk to one Claude Code session, by voice or text, and it runs the other agents in tmux. Speech-to-text and spoken replies run locally.

By bridge, the pieces were all there: one agent in front, workers in tmux windows, voice, a browser agents can drive. They were spread across repos and each needed setting up.

## mAGI

[mAGI](https://github.com/pankajgarkoti/magi) puts them in one install. You log in and the orchestrator is already running in tmux. You speak or type to it. It starts other agents, operates the desktop through `hyprctl`, and asks you only for decisions.

Some parts map straight back:

- The event bus delivers events to the orchestrator only when it is idle and its input box is empty ([how it works](https://github.com/pankajgarkoti/magi/blob/main/docs/event-bus.md)). That's the AgentZero wishlist item about checking on a long task without interrupting it, from the other side.
- The agent browser is a Chromium with DevTools on a local port, shared between you and the agent. FlowPilot's CDP handoff, without the Docker.
- Probes in that browser can watch a messaging tab. The shipped example probe (off by default) is for WhatsApp, where Joy ran in 2022.
- Daily notes are written by you and by the agent ([0.3](/blog-magi-0-3-session-notes)). daily-notes.nvim with a second author.
- Voice and the orchestrator come from bridge.

The pattern I see: I keep building one thing to talk to, with other things doing the work behind it, and something that remembers what's still open. It moved from a chat app, to a product backend, to my own desktop.

## Timeline

| When | Project | What it was |
|---|---|---|
| 2022 | Joy (Hear Us) | Wellbeing chatbot on Slack and WhatsApp, with conversation memory |
| 2022 | [dotfiles](https://github.com/pankajgarkoti/dotfiles) | Neovim and terminal config |
| 2023 | Mavy (Mavex AI) | One assistant, fourteen specialists behind a router |
| Jan 2024 | [AgentZero](https://github.com/computer-reinvention/AgentZero) | Minimal tool-calling agent |
| Feb 2024 | [PlanWeaver](https://github.com/computer-reinvention/PlanWeaver) | Plan-and-execute agent |
| Jul 2024 | [ChainFactory](https://github.com/computer-reinvention/ChainFactory) | DSL for LLM pipelines, on PyPI |
| Jun 2025 | [daily-notes.nvim](https://github.com/pankajgarkoti/daily-notes.nvim) | Daily notes in Neovim |
| Aug 2025 | [doctr](https://github.com/pankajgarkoti/doctr) | Docs daemon driven by git diffs |
| Jan 2026 | [multiclaude](https://github.com/computer-reinvention/multiclaude) | Parallel Claude Code agents in tmux |
| Mar 2026 | [FlowPilot](https://github.com/computer-reinvention/flowpilot) | Browser automation with human handoff over CDP |
| May 2026 | [trie](https://github.com/computer-reinvention/trie) | Index of meaning and intent for a codebase |
| May 2026 | [Soren](https://github.com/computer-reinvention/soren) | Agent swarm in tmux with mailbox and journal |
| Nov 2026 | [bridge](https://github.com/pankajgarkoti/bridge) | Talk to one Claude Code session; it runs the rest |
| Nov 2026 | [mAGI](https://github.com/pankajgarkoti/magi) | A Fedora/Hyprland setup built around LLM agents |

mAGI is on GitHub at [pankajgarkoti/magi](https://github.com/pankajgarkoti/magi); a star helps other people find it.
