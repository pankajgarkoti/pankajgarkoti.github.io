---
layout: default
title: Engineering Evolution Timeline
---

<style>
.timeline-intro {
  max-width: 700px;
  margin: 0 auto 2rem;
  text-align: center;
  color: #555;
  font-size: 1.05rem;
  line-height: 1.6;
}
.timeline-intro h1 {
  font-size: 1.8rem;
  color: #333;
  margin-bottom: 0.5rem;
}
.timeline-legend {
  display: flex;
  justify-content: center;
  gap: 1.5rem;
  margin: 1rem 0 2rem;
  flex-wrap: wrap;
}
.legend-item {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  font-size: 0.85rem;
  color: #666;
}
.legend-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  display: inline-block;
}
.legend-dot.ahead { background: #16a34a; }
.legend-dot.concurrent { background: #2563eb; }

/* Timeline container */
.tl {
  position: relative;
  max-width: 900px;
  margin: 0 auto;
  padding: 1rem 0;
}
/* Vertical center line */
.tl::before {
  content: '';
  position: absolute;
  left: 50%;
  top: 0;
  bottom: 0;
  width: 3px;
  background: #d1d5db;
  transform: translateX(-50%);
}

/* ---- MAJOR ENTRY (full treatment) ---- */
.tl-entry {
  position: relative;
  width: 45%;
  padding: 1rem 1.25rem;
  margin-bottom: 2rem;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.10);
  border-left: 4px solid #16a34a;
}
.tl-entry.concurrent {
  border-left-color: #2563eb;
}
.tl-entry.left { left: 0; }
.tl-entry.right { left: 55%; }
/* Connector dot — major */
.tl-entry::after {
  content: '';
  position: absolute;
  top: 1.2rem;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #16a34a;
  border: 3px solid #fff;
  box-shadow: 0 0 0 2px #16a34a;
}
.tl-entry.concurrent::after {
  background: #2563eb;
  box-shadow: 0 0 0 2px #2563eb;
}
.tl-entry.left::after { right: -32px; }
.tl-entry.right::after { left: -32px; }
/* Date badge */
.tl-date {
  display: inline-block;
  font-size: 0.78rem;
  font-weight: 600;
  padding: 0.15rem 0.5rem;
  border-radius: 4px;
  background: #f0fdf4;
  color: #16a34a;
  margin-bottom: 0.4rem;
}
.tl-entry.concurrent .tl-date {
  background: #eff6ff;
  color: #2563eb;
}
.tl-project {
  font-size: 1.1rem;
  font-weight: 700;
  color: #1a1a1a;
  margin: 0.2rem 0;
}
.tl-desc {
  font-size: 0.88rem;
  color: #444;
  line-height: 1.5;
  margin: 0.3rem 0;
}
.tl-pattern {
  font-size: 0.82rem;
  color: #16a34a;
  font-weight: 600;
  margin: 0.4rem 0 0.15rem;
}
.tl-entry.concurrent .tl-pattern {
  color: #2563eb;
}
.tl-industry {
  font-size: 0.8rem;
  color: #888;
  font-style: italic;
}

/* ---- MINOR ENTRY (muted, compact) ---- */
.tl-minor {
  position: relative;
  width: 45%;
  padding: 0.45rem 1rem;
  margin-bottom: 1rem;
  background: #fafafa;
  border-radius: 6px;
  border-left: 3px solid #d1d5db;
}
.tl-minor.left { left: 0; }
.tl-minor.right { left: 55%; }
.tl-minor::after {
  content: '';
  position: absolute;
  top: 0.7rem;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #d1d5db;
  border: 2px solid #fff;
  box-shadow: 0 0 0 1px #d1d5db;
}
.tl-minor.left::after { right: -29px; }
.tl-minor.right::after { left: -29px; }
.tl-minor .tl-date {
  font-size: 0.7rem;
  background: #f5f5f5;
  color: #999;
  padding: 0.1rem 0.4rem;
  margin-bottom: 0;
  margin-right: 0.4rem;
}
.tl-minor .tl-inline {
  font-size: 0.8rem;
  color: #888;
  display: inline;
}

/* Mobile: stack vertically */
@media (max-width: 700px) {
  .tl::before { left: 20px; }
  .tl-entry, .tl-minor {
    width: calc(100% - 50px);
    left: 45px !important;
  }
  .tl-entry.left::after, .tl-entry.right::after,
  .tl-minor.left::after, .tl-minor.right::after {
    left: -32px;
    right: auto;
  }
}
</style>

<div class="timeline-intro">
<h1>Engineering Evolution</h1>
<p>A month-by-month map of systems I built — many before the industry shipped similar ideas. Not claiming invention, just showing the trajectory.</p>
</div>

<div class="timeline-legend">
  <span class="legend-item"><span class="legend-dot ahead"></span> Built before industry equivalent</span>
  <span class="legend-item"><span class="legend-dot concurrent"></span> Built concurrently with industry</span>
</div>

<div class="tl">

<!-- ═══════════ 2022 ═══════════ -->

<!-- Jun 2022 — MAJOR -->
<div class="tl-entry left">
  <div class="tl-date">Jun 2022</div>
  <div class="tl-project">Joy / HearUs Slack Bot</div>
  <div class="tl-desc">Pre-ChatGPT conversational AI with event-driven Firestore listeners, mood tracking, and reactive architecture for workplace wellbeing.</div>
  <div class="tl-pattern">Pattern: Event-driven conversational AI</div>
  <div class="tl-industry">Industry: ChatGPT launched Nov 2022; LangChain Oct 2022</div>
</div>

<!-- Jul 2022 — MAJOR -->
<div class="tl-entry right">
  <div class="tl-date">Jul 2022</div>
  <div class="tl-project">WhatsApp Bot (Joy)</div>
  <div class="tl-desc">State-machine conversational AI with Dialogflow NLU and multi-tenant Firestore backend. Production WhatsApp integration.</div>
  <div class="tl-pattern">Pattern: State-machine dialogue + multi-tenant AI</div>
  <div class="tl-industry">Industry: Most chatbot frameworks emerged 2023+</div>
</div>

<!-- Aug–Oct 2022 -->
<div class="tl-minor left">
  <div class="tl-date">Aug–Oct 2022</div>
  <div class="tl-inline">Continued Joy/HearUs iteration — Firestore event patterns, conversation memory refinement</div>
</div>

<!-- Nov 2022 -->
<div class="tl-minor right">
  <div class="tl-date">Nov 2022</div>
  <div class="tl-inline">ChatGPT launches publicly — validates the conversational AI direction already in production</div>
</div>

<!-- Dec 2022 -->
<div class="tl-minor left">
  <div class="tl-date">Dec 2022</div>
  <div class="tl-inline">WhatsApp Bot in production — state machine handling real users, multi-tenant architecture stable</div>
</div>

<!-- ═══════════ 2023 ═══════════ -->

<!-- Jan 2023 — MAJOR -->
<div class="tl-entry right">
  <div class="tl-date">Jan 2023</div>
  <div class="tl-project">oauth_server (Mavy v1)</div>
  <div class="tl-desc">Hierarchical agent orchestration — AgentZero router delegating to specialist sub-agents via structured JSON contracts. Multi-agent before multi-agent was a thing.</div>
  <div class="tl-pattern">Pattern: Hierarchical multi-agent orchestration</div>
  <div class="tl-industry">Industry: AutoGen (Oct 2023), CrewAI (Dec 2023)</div>
</div>

<!-- Feb–Apr 2023 -->
<div class="tl-minor left">
  <div class="tl-date">Feb–Apr 2023</div>
  <div class="tl-inline">Mavy v1 evolution — refining agent delegation patterns, JSON contract schema iterations</div>
</div>

<!-- May–Jun 2023 -->
<div class="tl-minor right">
  <div class="tl-date">May–Jun 2023</div>
  <div class="tl-inline">Mavy sub-agent specialization — adding domain-specific agents under AgentZero router</div>
</div>

<!-- Jul 2023 -->
<div class="tl-minor left">
  <div class="tl-date">Jul 2023</div>
  <div class="tl-inline">Mavy v1 stabilized — orchestration patterns proven in production use</div>
</div>

<!-- Aug 2023 — MAJOR (two projects) -->
<div class="tl-entry right">
  <div class="tl-date">Aug 2023</div>
  <div class="tl-project">MavyBackend (chainsmoke)</div>
  <div class="tl-desc">Binary Delegation Contract — GPT-4 commander + GPT-3.5 executor with typed Context object. Cost-optimized two-tier agent architecture.</div>
  <div class="tl-pattern">Pattern: Commander/Executor delegation with typed contracts</div>
  <div class="tl-industry">Industry: OpenAI function calling limited (Jun 2023); tool_use widespread 2024</div>
</div>

<!-- Sep 2023 — MAJOR -->
<div class="tl-entry left">
  <div class="tl-date">Sep 2023</div>
  <div class="tl-project">MavySearch</div>
  <div class="tl-desc">Adaptive K-Means Retrieval Cutoff — clusters similarity scores dynamically instead of using a fixed threshold for RAG retrieval.</div>
  <div class="tl-pattern">Pattern: Adaptive retrieval cutoff via clustering</div>
  <div class="tl-industry">Industry: Similar approaches in LlamaIndex/LangChain mid-2024</div>
</div>

<!-- Oct 2023 — MAJOR -->
<div class="tl-entry right">
  <div class="tl-date">Oct 2023</div>
  <div class="tl-project">FineTuning</div>
  <div class="tl-desc">Fine-tuned GPT-3.5 as a JSON router for intent classification and task delegation. Lightweight, fast, cost-effective routing layer.</div>
  <div class="tl-pattern">Pattern: Fine-tuned model as structured router</div>
  <div class="tl-industry">Industry: OpenAI fine-tuning for function calling came late 2024</div>
</div>

<!-- Nov 2023 — MAJOR -->
<div class="tl-entry left concurrent">
  <div class="tl-date">Nov 2023</div>
  <div class="tl-project">HQ Dashboard</div>
  <div class="tl-desc">Multi-assistant management platform with SSE streaming, prompt libraries, and brand voice synthesis. Full-stack AI operations dashboard.</div>
  <div class="tl-pattern">Pattern: Multi-assistant management + prompt orchestration</div>
  <div class="tl-industry">Industry: OpenAI Assistants API (Nov 2023) — built independently, concurrent</div>
</div>

<!-- Dec 2023 -->
<div class="tl-minor right">
  <div class="tl-date">Dec 2023</div>
  <div class="tl-inline">HQ Dashboard expansion — adding prompt library features, SSE streaming refinement</div>
</div>

<!-- ═══════════ 2024 ═══════════ -->

<!-- Jan 2024 -->
<div class="tl-minor left">
  <div class="tl-date">Jan 2024</div>
  <div class="tl-inline">HQ multi-assistant iteration — brand voice synthesis, user management</div>
</div>

<!-- Feb 2024 — MAJOR (two projects) -->
<div class="tl-entry right">
  <div class="tl-date">Feb 2024</div>
  <div class="tl-project">AgentZero</div>
  <div class="tl-desc">Extracted the core agent loop into a reusable library — Thought-Action-Observation cycle with a pluggable tool registry.</div>
  <div class="tl-pattern">Pattern: Reusable agent loop abstraction</div>
  <div class="tl-industry">Industry: LangGraph agents and CrewAI adopted similar patterns mid-2024</div>
</div>

<div class="tl-entry left">
  <div class="tl-date">Feb 2024</div>
  <div class="tl-project">PlanWeaver</div>
  <div class="tl-desc">Planner-Executor agent framework with a plan state machine — decompose, execute steps, replan on failure.</div>
  <div class="tl-pattern">Pattern: Plan-and-execute with state machine</div>
  <div class="tl-industry">Industry: Plan-and-execute patterns appeared in LangGraph mid-2024</div>
</div>

<!-- Mar–May 2024 -->
<div class="tl-minor right">
  <div class="tl-date">Mar–May 2024</div>
  <div class="tl-inline">HQ Dashboard continued — SSE optimizations, prompt template management</div>
</div>

<!-- Jun 2024 -->
<div class="tl-minor left">
  <div class="tl-date">Jun 2024</div>
  <div class="tl-inline">HQ wrapping up — stabilizing multi-assistant workflows for handoff</div>
</div>

<!-- Jul 2024 — MAJOR -->
<div class="tl-entry right">
  <div class="tl-date">Jul 2024</div>
  <div class="tl-project">ChainFactory</div>
  <div class="tl-desc">Declarative <code>.fctr</code> DSL for LLM pipeline orchestration. Config-driven, not code-driven. Published on PyPI.</div>
  <div class="tl-pattern">Pattern: Declarative DSL for LLM pipelines</div>
  <div class="tl-industry">Industry: DSPy (Stanford, 2024) — ChainFactory's file-driven approach is distinct</div>
</div>

<!-- Aug–Dec 2024 -->
<div class="tl-minor left">
  <div class="tl-date">Aug–Oct 2024</div>
  <div class="tl-inline">ChainFactory iteration — .fctr DSL refinement, PyPI releases, documentation</div>
</div>

<div class="tl-minor right">
  <div class="tl-date">Nov–Dec 2024</div>
  <div class="tl-inline">ChainFactory stabilization — production usage patterns, spec drafts (v001–v004)</div>
</div>

<!-- ═══════════ 2025 ═══════════ -->

<!-- Jan–Jun 2025 -->
<div class="tl-minor left">
  <div class="tl-date">Jan–Jun 2025</div>
  <div class="tl-inline">ChainFactory continued development — spec iterations, community feedback integration</div>
</div>

<!-- Jul 2025 -->
<div class="tl-minor right">
  <div class="tl-date">Jul 2025</div>
  <div class="tl-inline">ChainFactory v1 stable — DSL finalized, PyPI package mature</div>
</div>

<!-- Sep 2025 — MAJOR -->
<div class="tl-entry left">
  <div class="tl-date">Sep 2025</div>
  <div class="tl-project">RelayBackend</div>
  <div class="tl-desc">Bidirectional semantic intersection matching for professional networking — finds mutual value, not just one-way similarity.</div>
  <div class="tl-pattern">Pattern: Bidirectional semantic matching</div>
  <div class="tl-industry">Industry: No direct equivalent yet</div>
</div>

<!-- Oct–Dec 2025 -->
<div class="tl-minor right">
  <div class="tl-date">Oct–Dec 2025</div>
  <div class="tl-inline">RelayBackend refinement — matching algorithm tuning, networking feature development</div>
</div>

<!-- ═══════════ 2026 ═══════════ -->

<!-- Jan 2026 — MAJOR -->
<div class="tl-entry left">
  <div class="tl-date">Jan 2026</div>
  <div class="tl-project">Multiclaude</div>
  <div class="tl-desc">Pure Bash multi-agent orchestration with mailbox protocol, agent lifecycle management, and supervisor coordination.</div>
  <div class="tl-pattern">Pattern: Shell-native multi-agent orchestration</div>
  <div class="tl-industry">Industry: Most multi-agent systems require heavy frameworks</div>
</div>

<!-- Feb 2026 — MAJOR -->
<div class="tl-entry right">
  <div class="tl-date">Feb 2026</div>
  <div class="tl-project">CMUX</div>
  <div class="tl-desc">Self-improving multi-agent system with supervisor hierarchy, automatic rollback, persistent memory, health monitoring. Running autonomously.</div>
  <div class="tl-pattern">Pattern: Self-modifying AI system with safety guarantees</div>
  <div class="tl-industry">Industry: Emerging research area — no production equivalent</div>
</div>

<!-- Mar 2026 -->
<div class="tl-minor left">
  <div class="tl-date">Mar 2026</div>
  <div class="tl-inline">CMUX autonomous operation — self-improvement cycles, revenue work delivery, agent team coordination</div>
</div>

</div>

<p style="text-align:center; margin-top:2rem; color:#888; font-size:0.85rem;">
  <a href="/">← Back to Home</a>
</p>
