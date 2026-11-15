---
layout: default
title: "mAGI 0.2: an event bus and a shared browser for a terminal-agent desktop"
---

# mAGI 0.2: an event bus and a shared browser for a terminal-agent desktop

![mAGI](https://raw.githubusercontent.com/pankajgarkoti/magi/v0.2.0/docs/logo/logo-colour-light.svg)

*mAGI 0.2.0 · November 2026 · [source](https://github.com/pankajgarkoti/magi) · [release](https://github.com/pankajgarkoti/magi/releases/tag/v0.2.0)*

## Abstract

mAGI is a Fedora Atomic remix whose primary interface is an orchestrator agent in tmux. Version 0.2.0 adds three default components: an append-only event bus for notifying the orchestrator; a daemon that runs read-only probes in tabs of a dedicated Chromium instance over the Chrome DevTools Protocol and publishes their output to the bus; and a SQLite ledger with a periodic health check. Events reach the agent only when it is idle with an empty input. Apart from Chromium's loopback debugging port, no component listens on a socket. In a virtual machine, events reached the agent in 5.1–6.2 s, mostly a deliberate 3 s coalescing window; page changes reached the bus in a median of 96 ms.

## 1 Introduction

In mAGI the user addresses a single terminal agent (Claude Code, opencode, Codex or another command) in window 1 of a tmux session; that agent starts other agents in further windows and operates the desktop. Version 0.1 delivered one kind of asynchronous signal to the orchestrator: a line typed into its input when a worker it had started finished, stalled on a prompt or exited.

Other signals had no path. A build finishing in a script, a page in the browser changing, a timer firing, or a check failing could reach the orchestrator only if the orchestrator polled for them. Polling from inside an LLM session costs a model turn per check and fills the context with negative results. Browser access had a second problem: agents that drove a browser started a fresh headless profile, without the user's sessions.

Version 0.2 addresses both with components that run by default in the image. Section 2 describes the design, Section 3 the implementation, Section 4 the measurements taken on a test installation, and Section 5 the limitations.

## 2 System design

**2.1 Event bus.** The bus is a single append-only file of JSON lines. A publisher runs `orch-bus emit SOURCE TEXT [--key K]`; the optional key drops repeated events. Text is normalised to one line of at most 300 characters. There is no broker process on the publish path: emitting is a file append. A separate delivery loop reads new lines and types them into the orchestrator as one batched line, `[event bus] 14:02 source: text | ...`, under two conditions: the orchestrator's agent reports itself idle, and its input box contains no text typed by the user. Bursts are coalesced for 3 s. Events stay in the log either way, and the orchestrator can read them on demand with `orch-bus unread`, which keeps its own offset. Delivery can be switched off.

**2.2 Browser-to-bus.** A dedicated Chromium instance runs with its own profile and a fixed DevTools port on 127.0.0.1. Two consumers attach to it. Agents use it through the chrome-devtools MCP server, configured with `--browserUrl` so that they share the user's signed-in sessions instead of launching a new browser. The `browser-bus` daemon attaches as a CDP client and maintains a registry of probes. A probe is a JavaScript file with a userscript-style header (`@match`, `@source`, `@purpose`); its body runs in each matching top-level tab and calls `emit({text, key})`. Each emitted object is validated, rate-limited per probe, deduplicated and published to the bus.

**2.3 Ledger and health check.** `orch` stores agents started by the orchestrator, open threads, standing watchers, user decisions, deliverables and a mirror of the bus in one SQLite database. Open threads are mirrored to a Markdown checklist that the user edits directly; ticking an item closes it in the ledger. A timer runs `orch doctor` every 15 minutes. It checks the user units, the core tmux windows, the debugging port, the browser-bus connection, database integrity and watcher expiry; restarts what can be restarted; and reports the remainder on the bus only when the set of problems changes.

**2.4 Startup.** On login a oneshot user unit builds the tmux session. If a layout snapshot exists, saved every minute by a timer, it is restored first: agent panes are reopened with their agent's resume command and other panes with their start command. The orchestrator is pinned to window 1, and two core windows are created if absent: `bus`, which follows the event log, and `chrome`, which runs the browser launcher. The bus delivery service, browser-bus and both timers are enabled globally in the image and do not depend on the orchestrator being enabled.

## 3 Implementation

**3.1 CDP bindings.** browser-bus calls `Target.setDiscoverTargets` and attaches with flattened sessions only to page targets whose URL matches at least one probe; it detaches when none match. For each attached tab it registers `Runtime.addBinding` restricted to a named isolated world, and installs each probe with `Page.addScriptToEvaluateOnNewDocument` in that world, followed by an immediate `Runtime.evaluate` for the already-loaded document. The isolated world shares the DOM with the page but not its JavaScript globals, so page scripts can neither read probe state nor call the binding. Probe calls arrive as `Runtime.bindingCalled` events. Payloads above 8 KiB, malformed JSON, unknown probe names, and bus sources outside the probe's declared allow-list are discarded. Each probe has a token bucket of 30 events refilled at one per 4 s. Injection is idempotent per probe content hash; editing a probe file triggers a hot reload through `fs.watch`.

**3.2 No network listener.** Neither the bus nor browser-bus opens a socket. Publishing is a file append; the CLI talks to the browser-bus daemon by writing request files into a state directory and reading the response files it writes back. The only listening port added by the release is Chromium's DevTools endpoint, bound with `--remote-debugging-address=127.0.0.1`. Chromium accepts a fixed debugging port only with a non-default profile directory, so the launcher uses `~/.config/chromium-agent`, seeded once from an existing Chromium profile if one exists.

**3.3 Agent-agnostic delivery.** Whether the orchestrator is idle, and whether its input box holds a draft, is read through the adapter layer introduced for multiple agents: from Claude Code's per-process session files, opencode's local API, Codex's rollout files, or the tmux pane for other commands. The bus uses the same function that delivers worker notifications, and worker notifications are also written to the bus marked as delivered so they are not typed twice.

**3.4 Launcher.** `chrome-debug` detects a Wayland socket under `$XDG_RUNTIME_DIR` and the Hyprland instance signature when started from tmux or systemd before the compositor, waiting up to 60 s at boot. A request to `/json/version` on the port acts as the single-instance check. tmux records the window's start command, so the browser returns after a reboot with the rest of the layout.

**3.5 systemd units and configuration.** Four user units are enabled globally in the image with `systemctl --global enable`: `orch-bus.service` (delivery), `browser-bus.service`, `orch-doctor.timer` and `orch-open-threads.timer`. Every tool lives in `/usr/share/magi` and is symlinked into `/usr/bin`, so the user's PATH at login is irrelevant. Before the orchestrator starts, the chrome-devtools MCP entry is added to the agent's own configuration file if missing: through `claude mcp add` for Claude Code, and by editing `opencode.json` or Codex's `config.toml`. No agent, account or key is part of the image; the first-login setup installs the chosen agent with its publisher's installer and the user signs in.

## 4 Evaluation

All measurements were taken on 15 November 2026 in one headless QEMU/KVM guest (UEFI, 4 vCPU, 5 GB RAM) on a laptop host. The guest was installed from a test ISO of the release candidate and then moved to the final 0.2.0 image with `bootc upgrade`. A stub program stood in for the agent: it reports idle/busy state through the same files as Claude Code and logs every line it receives. No real model was involved.

**4.1 Tests.** The adapter suite (`tests/agent/test-adapters.sh`) passed 148 of 148 checks across the four agent adapters. The VM suites `test-agent.sh` (setup, reboot) and `test-bus.sh` (units, core windows, loopback binding, MCP entry, bus round trip and deduplication, delivery, probe event, ledger, doctor; and the same checks after a reboot) passed on the final image.

**4.2 Latency.** Table 1 lists the two paths measured.

| Path | n | Result |
|---|---|---|
| `orch-bus emit` → line in the agent's input | 5 | 5.1–6.2 s |
| page `document.title` change → bus event | 6 | median 96 ms, max 131 ms |

*Table 1.* The first path includes the 3 s coalescing delay and the 2–3 s polling intervals of the delivery loop and the idle check; it is bounded by design rather than by computation. The second was measured from a timestamp written into the title by the page to the event's own timestamp.

**4.3 Resources.** Resident memory after the test: browser-bus 62 MB, the delivery loop 24 MB, Chromium with one tab 1.4 GB across its processes. Installed size added to the image: Chromium 359 MB and Node.js 22 with npm 88 MB; the image grew by 0.67 GB and the ISO is 5.42 GB. Time to `graphical.target` ranged from 27.0 s to 41.4 s over three boots, excluding one boot that ran `ldconfig` after an image upgrade (101 s).

**4.4 Defects found.** The VM runs exposed five defects, fixed before release: a race between the daemon and the session script that skipped installing the default probe; a race in which a tab created and navigated in one step loaded its page before the probe hook was registered; the agent browser opening inside the console's scratchpad workspace; no recovery when the browser launcher gave up before the compositor started; and a pre-existing hang in which the voice listener waited on a password prompt for a sleep inhibitor.

## 5 Limitations and security

**Shared browser.** The debugging port is reachable only from the loopback interface, but any local process of any user can connect to it, and a CDP client has full control of the browser: it can read pages, cookies and storage of every signed-in site and act on them. This exposure is the mechanism by which agents use the user's sessions; it cannot be removed without removing that capability. Mitigations available to the user are to sign in only where agent access is intended, to keep untrusted software off the machine, or to disable the browser (`BROWSER=0`). Chromium's per-connection approval prompt, which applies to its built-in remote-debugging toggle, does not apply to a fixed port, by design.

**Data flow to the provider.** Probe output and bus events become input to the orchestrator agent and are therefore sent to its model provider. The default probe emits only a host name and an unread count; probes written for messaging sites will forward message text.

**Probe fragility.** Probes depend on page markup and break when sites change it. The shipped WhatsApp example is not installed by default for this reason and because it forwards message text.

**Delivery semantics.** Delivery is at-least-once from the log's perspective but best-effort into the agent: if the orchestrator remains busy, events wait in the log. Coalescing adds up to 3 s of latency by design. Typing into a terminal agent relies on screen heuristics for draft detection when the agent exposes no state file.

**Autologin.** With automatic login the GNOME keyring remains locked and Chromium blocks page loads on a keyring prompt; the VM tests used `--password-store=basic`. On the guest's first boot Hyprland failed once to create its DRM backend and started on the next login; the cause was not determined.

**Scope of the evaluation.** All measurements in Section 4 come from one virtual machine with a stub agent standing in for the real ones; no measurements were taken with real model latency or on physical hardware.

## 6 Related work

The bus is a minimal instance of the blackboard and tuple-space patterns [1, 2], reduced to an append-only log in the manner of event sourcing [3]. Probe headers follow the userscript conventions of Greasemonkey [4]. Browser control uses the Chrome DevTools Protocol [5]; agent access uses the Model Context Protocol [6] through the chrome-devtools MCP server [7], which can also launch its own browser, as can Playwright MCP [8]. Supervision uses systemd user units and timers [9]; the agent layer and layout persistence use tmux [10].

## References

1. L. D. Erman, F. Hayes-Roth, V. R. Lesser, D. R. Reddy. "The Hearsay-II speech-understanding system: integrating knowledge to resolve uncertainty." *ACM Computing Surveys* 12(2), 1980.
2. D. Gelernter. "Generative communication in Linda." *ACM TOPLAS* 7(1), 1985.
3. M. Fowler. "Event Sourcing." 2005. <https://martinfowler.com/eaaDev/EventSourcing.html>
4. Greasemonkey. <https://www.greasespot.net/>
5. Chrome DevTools Protocol. <https://chromedevtools.github.io/devtools-protocol/>
6. Model Context Protocol specification. <https://modelcontextprotocol.io/>
7. chrome-devtools-mcp. <https://github.com/ChromeDevTools/chrome-devtools-mcp>
8. Playwright MCP. <https://github.com/microsoft/playwright-mcp>
9. systemd.unit(5), systemd.timer(5). <https://www.freedesktop.org/software/systemd/man/>
10. tmux. <https://github.com/tmux/tmux>
11. mAGI source. <https://github.com/pankajgarkoti/magi>
