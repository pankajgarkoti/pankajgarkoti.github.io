---
layout: default
title: "Teaching the browser to the agent"
---

# Teaching the browser to the agent

The browser was the last thing on my machine that only a person could operate. Everything else — shells, files, tmux windows, git — had a command line the agent could already use. The browser had pixels. Getting the agent past the pixels meant teaching it the difference between exploring once and replaying forever, and that distinction now lives in a small SQLite file full of recorded flows.

mAGI gives the agent a Chromium with DevTools on a local port, a profile of its own, on workspace 9. The agent gets a browser tool pointed at it, so it uses the browser you are signed in to. That sentence hides a year's worth of trouble: "uses the browser" meant driving it like a person — screenshots, coordinates, model-in-the-loop clicking, one expensive turn per button. It worked the way a toddler works a phone: eventually, and at great cost.

## March: exploring once, replaying forever

The earlier attempt was [FlowPilot](https://github.com/computer-reinvention/flowpilot), from March this year: browser automation that explores a site once with an agent, compiles scripts to replay it without LLM calls, and hands the live browser to a human over CDP when it hits a login or a CAPTCHA.

That sentence contains the whole thesis. Exploration is the expensive part and it only needs to happen once. Replay should cost nothing. And the handoff matters more than the automation: the moment the page asks for a human, the human gets the live browser, not a screenshot of it.

magi-reflex, merged in [PR #8](https://github.com/pankajgarkoti/magi/pull/8), is that thesis moved inside mAGI and given a memory. The senses (browser-bus probes) watch pages; reflexes are the muscle that acts in them. A reflex is a small JSON document — a flow recorded once and replayed forever. Scripts drive semantic CDP actions, not synthetic mouse physics. No screenshots, no model-in-the-loop clicking. The full contract is written down in [docs/magi-reflex.md](https://github.com/pankajgarkoti/magi/blob/main/docs/magi-reflex.md), which is the document I point people at when they ask what a reflex actually guarantees.

## What a reflex is

A reflex document names its domain, its entry URL, its inputs, and its phases. One phase is one semantic step, and every phase carries the same five things: a `target_assert` script that proves the exact intended target exists before anything acts (no target, no action); a `reflex_script` in plain JavaScript that does the step, with helpers like `h.click`, `h.clickAt`, `h.focus`, `h.text`, `h.key`, `h.eval`, `h.files`, `h.sleep`, `h.now` and `ARGS` for the inputs; a `verify_script` that proves the step worked; a `steady_rule` for when the page has settled enough to trust the verify; and timeouts, a per-phase budget plus a verify poll (`interval_ms`, `timeout_ms`).

The three practice reflexes in [`agent/reflexes/`](https://github.com/pankajgarkoti/magi/tree/main/agent/reflexes) show the range: `local-fill-form` serves a tiny local test page from the repo's own [testpage directory](https://github.com/pankajgarkoti/magi/tree/main/agent/lib/reflex/testpage), fills name, email and message, uploads one small file, submits, and verifies the echoed values. `httpbin-submit` fills the public form at `https://httpbin.org/forms/post`, submits, and verifies the echoed JSON contains our values. `theinternet-login` logs in with the public demo credentials printed on the page (`tomsmith` / `SuperSecretPassword!`), submits, and verifies the success flash.

Nothing in those three touches a real account. That is deliberate, and it is enforced, not advised.

## The store: candidate, active, stale, broken

Reflexes live in a local SQLite store with two tables. The `reflexes` table is keyed by `(domain, name)` and every reflex has a lifecycle: `candidate` means new, never replayed successfully; `active` means proven by at least one successful replay, and only active reflexes are offered for use; `stale` means the network signature drifted, so replay refuses and lookup stops offering it until it is re-sealed; `broken` means demoted after consecutive failures, never offered.

A regenerated script for an active reflex waits in `candidate` as a next-version document and never clobbers the serving one; the next successful replay promotes it. `owner_registered` arms a reflex outside the harmless-surface allowlist. `lease_until` and `lease_by` provide a learn-exclusive lease so two agents do not learn the same page at once.

The `runs` table is a bounded audit: one row per phase per replay plus a summary row, pruned to the last 100 rows per reflex name. Every row records `zero_model_calls`. Traces go to JSONL files under the state directory. The `--trace` flag on a run logs every CDP frame sent or received — the complete activity record of the process. Replay is provably model-free because the run path is one process opening one websocket to `127.0.0.1:9222`, evaluating stored scripts, reading local files, and emitting one bus line. There is no model on that path to call.

## The run loop: its own tab, or nothing

`magi-reflex run NAME --args '{…}'` opens its own tab — never another one — navigates to the entry URL, and per phase enforces the entry-domain lock, runs target assertion, then the reflex script, then steady, then verify. Then it closes its tab on every exit path.

Any failure stops the run and emits one bus line: `magi-reflex <name> FAILED at <phase>: <why>`. Success emits one line with the elapsed milliseconds. The store records per-phase and run audit rows. `magi-reflex suggest --url URL` prints the available fast-paths for a page with an anti-bias header, verbatim: using one is optional, hand-driving is always fine, and if you do something new and repeatable, consider recording it. `ls` lists the registry, `show` prints a full document, `install` refreshes one, `seal` captures signatures, `lease` coordinates learners, `dogma` re-measures steady-state detectors.

The domain lock is checked after navigation and during each phase: if the tab's origin leaves the reflex domain, the run aborts. Reflexes never act on messaging or social surfaces. That is not a suggestion in the docs; it is a check in the runner, and it is the same instinct as the event bus guard from [PR #1](https://github.com/pankajgarkoti/magi/pull/1): the dangerous action is refused rather than detected.

## Safety gates, written as law

The safety section of [docs/magi-reflex.md](https://github.com/pankajgarkoti/magi/blob/main/docs/magi-reflex.md) is dated 25 November and reads like incident law, because that is what it is. Four rules, all enforced in code.

Harmless-surface allowlist. Before a run opens a tab, the reflex domain is checked against `127.0.0.1, localhost, example.com, httpbin.org, the-internet.herokuapp.com`. Outside the allowlist with `owner_registered=0`, the run is REFUSED with one bus line: `magi-reflex <name> REFUSED: domain <d> not owner-registered`. With `owner_registered=1`, the owner has explicitly armed it and the exact entry URL is still required.

Pre-action target assertion. Every phase must carry `target_assert`; the runner refuses a reflex missing it. The assertion runs before the phase types or clicks anything.

Entry-domain lock. After navigation and during each phase, the runner aborts if the tab's origin leaves the reflex domain.

No credentials, no codes, no challenges. Reflexes never learn or run flows that type credentials, OTPs, or solve challenges. Such phases end with an owner handoff, the same way FlowPilot handed the live browser to a human at a login or a CAPTCHA. The agent's hands stop where the owner's identity starts.

The parallel with the bus guard is exact: untrusted page text is data, the frame is what holds, and the dangerous action is refused rather than detected. Detection scores drift; refusals hold.

## Drift: pages change, so reflexes go STALE

A recorded script rots the day the site redesigns. Most replay tools rot silently: they click where the button was and report success. Reflexes rot loudly.

A phase may declare a `net_filter`: a positive list of URL substrings or globs for the API calls that actually matter (`Document`, `XHR`, and `Fetch` requests only). While the phase runs, the runner records those requests from the action through steady state, drops the query string, and normalises opaque path segments — numbers, UUIDs, long hex or base64-ish ids — to `:id`. The ordered `METHOD /path` lines are hashed with sha256. `net_filter: []` means "this phase should make no matching call".

The signature stays null until `magi-reflex seal NAME` captures it from a verified replay. Once sealed, every replay compares the observed signature before verify is trusted. A mismatch fails the phase as `STALE` and emits one bus line with the expected and observed hashes. The reflex becomes `stale`: `run` refuses it and `suggest` no longer offers it. `seal` runs the flow again in capture mode — verify still gates it — writes the fresh signatures, and returns the reflex to `active`.

Re-sealing is signature re-learning only. An agentic re-learn of a changed script remains a later slice, deliberately: the machine tells you the page changed and waits. It never flails at a redesign, clicking hopefully. And filters must stay narrow — the doc says so plainly — because an unfiltered hash of a chatty app's traffic is noise, not a signature.

## Steady state: measure, don't assume

After acting, when has the page settled enough to verify? Three candidates were measured on real pages with the `magi-reflex dogma` filter ([the measurements are in the repo](https://github.com/pankajgarkoti/magi/blob/main/tests/agent/reflex/dogma-2026-11-25.json)): DOM-stable (no mutation for `quiet_ms`), network-quiesce, and fixed delay. DOM-stable at `quiet_ms=700` was safe on the measured loads. Network quiesce measured false-settle on chatty single-page apps; fixed delays fired before content existed. The default steady rule is DOM-stable — and verify is the real authority anyway. Steady-timeout proceeds to verify; verify decides.

I like this section of the doc because it reports a measurement with the losers named. Someone ran the candidates, wrote down which one lied, and set the default to the one that did not. That is the whole difference between a constant and a finding.

## Zero model calls, provably

Replay is agent-assisted only at author time. The run path has no model on it: one process, one websocket, stored scripts, local files, one bus line. The audit rows say `zero_model_calls=1` because the alternative — trusting the architecture diagram — is how "deterministic" systems quietly grow a model dependency. Here the claim is in the database, per phase, per replay.

The remaining slices are named in the doc, honestly: the agentic learn listener, script re-learn, and arming on bus events come later. What ships is record once, replay forever, refuse loudly, on harmless surfaces, with the owner's browser and the owner's explicit arming as the boundary.

Four years ago I built a bot people talked to. Three years ago, one assistant with fourteen specialists behind a router. Two years ago, the agent loop and the wishlist item about checking a long task without interrupting it. This year, the browser the agent can use. The senses watch; the muscle acts; the bus carries the news; the ledger remembers. Each piece took the same shape as the last one, and each one finally does only what it can prove.
