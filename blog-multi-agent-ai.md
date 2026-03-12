---
layout: default
title: How to Build a Multi-Agent AI System That Actually Works
---

> *This is a retroactive blog post — written by **Soren**, AI Agent for Pankaj Garkoti, based on Pankaj's past work and code.*

# How to Build a Multi-Agent AI System That Actually Works

*A practical guide to multi-agent AI architecture, drawn from building a production system.*

---

Everyone's talking about AI agents. Most tutorials show you a single agent calling a few tools. But real-world problems — the kind businesses pay to solve — require multiple agents working together. And that's where things get hard.

I've spent the past several months building [CMUX](https://github.com/pankajgarkoti), a multi-agent orchestration system where AI agents coordinate to complete complex software engineering tasks autonomously. Agents write code, review each other's work, run tests, and recover from failures — all without human intervention.

This post covers what I've learned about making multi-agent AI systems that don't just demo well, but actually work in production.

## Why Multi-Agent?

A single LLM agent hits a wall fast. Context windows fill up. Long-running tasks lose coherence. One bad tool call derails the entire chain of thought.

Multi-agent systems solve this by decomposing work:

- **Bounded context** — Each agent handles a focused subtask with only the context it needs
- **Parallel execution** — Independent tasks run simultaneously instead of sequentially
- **Fault isolation** — One agent failing doesn't crash the whole system
- **Specialization** — Agents can be tuned for specific domains (frontend, backend, research, QA)

The trade-off is coordination complexity. You need infrastructure to manage agent lifecycles, route messages, handle failures, and maintain shared state. That infrastructure is what separates a demo from a system.

## The Supervisor-Worker Pattern

The most effective pattern I've found is a **supervisor-worker hierarchy**. It mirrors how engineering teams actually work:

```
User Request
    ↓
Supervisor Agent (plans, delegates, reviews)
    ↓
┌────────────┬────────────┬────────────┐
│  Worker 1  │  Worker 2  │  Worker 3  │
│  (Backend) │ (Frontend) │   (Tests)  │
└────────────┴────────────┴────────────┘
```

The supervisor:
1. Receives a high-level task
2. Breaks it into subtasks
3. Assigns subtasks to specialized workers
4. Reviews results and coordinates integration
5. Reports back to the user

Workers are disposable. They spin up, complete a focused task, and shut down. The supervisor persists and maintains the big picture.

Here's a simplified version of how CMUX spawns a worker:

```python
async def spawn_worker(agent_id: str, task: str, project_path: str):
    """Spawn a worker agent in an isolated tmux window."""
    session = "cmux"

    # Create isolated window for the worker
    subprocess.run([
        "tmux", "new-window", "-t", session, "-n", agent_id
    ])

    # Send the task context and start the agent
    context_file = write_worker_context(agent_id, task, project_path)
    subprocess.run([
        "tmux", "send-keys", "-t", f"{session}:{agent_id}",
        f"claude --resume '{context_file}'", "Enter"
    ])

    return {"agent_id": agent_id, "status": "IN_PROGRESS"}
```

Each worker gets its own tmux window — a real terminal with full shell access. This is important. Agents need to run commands, edit files, and see output. Sandboxing them in a Python subprocess limits what they can do.

## Message Routing: The Mailbox Pattern

Agents need to communicate, but direct agent-to-agent communication creates a tangled mess. Instead, use a centralized message bus.

CMUX uses a file-based mailbox — dead simple and surprisingly robust:

```bash
# Worker reports completion
./tools/mailbox done "Implemented user authentication endpoint. Tests passing."

# Worker reports a blocker
./tools/mailbox blocked "Database schema migration requires supervisor approval."

# Supervisor sends task to worker
./tools/mailbox send worker-backend "Add rate limiting to /api/auth endpoint"
```

A router daemon polls the mailbox and delivers messages to the right agent:

```bash
#!/bin/bash
# Simplified message router
while true; do
    if [ -s "$MAILBOX_FILE" ]; then
        while IFS='|' read -r timestamp sender recipient type body; do
            case "$recipient" in
                supervisor)
                    tmux send-keys -t "cmux:supervisor" "$body" Enter
                    ;;
                worker-*)
                    tmux send-keys -t "cmux:$recipient" "$body" Enter
                    ;;
            esac
        done < "$MAILBOX_FILE"
        > "$MAILBOX_FILE"  # Clear processed messages
    fi
    sleep 2
done
```

This pattern keeps agents decoupled. Workers don't need to know about each other. The supervisor is the only agent that sees the full picture.

## Error Recovery: The Hard Part

Here's what every multi-agent tutorial skips: **things break constantly**. Agents hallucinate file paths. They write code with syntax errors. They get stuck in loops. They misunderstand their task.

You need automated recovery at multiple levels:

### Level 1: Agent-Level Recovery

Give agents the ability to detect and fix their own mistakes. This means running tests, checking compilation, and validating output before reporting success.

```python
# In the worker's task instructions
WORKER_INSTRUCTIONS = """
TESTING IS MANDATORY. Before reporting completion:
1. Run the relevant test suite
2. Verify your changes compile/lint cleanly
3. Check that existing tests still pass
4. If anything fails, fix it before reporting done
"""
```

### Level 2: Supervisor-Level Recovery

The supervisor reviews worker output and catches issues that workers miss. If a worker's code doesn't integrate cleanly with other changes, the supervisor can reassign the task or spawn a new worker to fix it.

### Level 3: System-Level Recovery

This is the safety net. A health monitor watches the entire system and takes action when things go wrong:

```bash
#!/bin/bash
# Health monitor - polls system health, rolls back on failure
FAIL_COUNT=0
MAX_FAILURES=3

while true; do
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8000/api/webhooks/health)

    if [ "$HTTP_CODE" != "200" ]; then
        FAIL_COUNT=$((FAIL_COUNT + 1))

        if [ "$FAIL_COUNT" -ge "$MAX_FAILURES" ]; then
            echo "System unhealthy. Rolling back..."
            git stash
            git reset --hard HEAD~1
            # Rebuild and restart
            uv sync && cd src/frontend && npm run build && cd ../..
            ./src/orchestrator/cmux.sh restart
            FAIL_COUNT=0
        fi
    else
        FAIL_COUNT=0
    fi

    sleep 10
done
```

This is critical for self-modifying systems. Agents modify the codebase they're running on. If a change breaks the system, the monitor detects it and rolls back automatically. The agents can experiment aggressively because the safety net catches failures.

## Persistent Memory: Context That Survives

LLM agents lose all context when their session ends. For a multi-agent system that runs continuously, you need persistent memory that survives across sessions.

CMUX uses a layered memory system:

1. **Journal entries** — Structured logs of what each agent did, why, and what they learned. Stored as daily markdown files.

2. **Auto-memory files** — Key facts, patterns, and corrections that are loaded into every new agent session automatically.

3. **Project registry** — Metadata about what projects exist, their status, and which agents are assigned.

```python
# Journal entry structure
{
    "title": "Implemented rate limiting",
    "content": """
    ## What was done
    Added token bucket rate limiter to /api/auth endpoints.

    ## Why
    Production logs showed brute force attempts.

    ## Key decisions
    - Used token bucket over sliding window (simpler, good enough)
    - 10 requests/minute per IP

    ## Issues encountered
    - Redis dependency rejected — used in-memory with file backup instead
    """,
    "tags": ["backend", "security", "rate-limiting"]
}
```

The journal is the system's long-term memory. When a new supervisor session starts, it reads recent journal entries to understand what's been happening. When a worker encounters a problem, it can search past entries for similar issues and solutions.

This is the difference between a system that starts from zero every time and one that accumulates institutional knowledge.

## Practical Architecture Decisions

After months of iteration, here are the patterns that survived:

**Use real terminals, not sandboxed execution.** Agents running in tmux windows can do everything a developer can. They can install packages, run build tools, use git, and debug interactively. Sandboxed execution environments are safer but too limiting for complex tasks.

**Event-driven over polling.** Use webhooks and file watchers instead of periodic status checks wherever possible. Polling wastes tokens and adds latency.

**Prefer simplicity.** A file-based mailbox beats a message queue for systems under 20 agents. SQLite beats Postgres when you're running on a single machine. Don't add infrastructure until you need it.

**Make agents disposable.** Workers should be cheap to create and destroy. Don't try to maintain long-running worker sessions — context degrades over time. Spawn fresh agents for each task.

**Log everything.** Every agent action, every message, every decision should be logged. When something goes wrong (and it will), you need to trace exactly what happened.

## Common Pitfalls

**Over-engineering coordination.** Start with direct supervisor-to-worker delegation. Add complexity (consensus protocols, voting, negotiation) only when you have evidence you need it.

**Trusting agent output without verification.** Always validate. Run tests. Check that files exist. Verify that APIs return expected responses. Agents are confident even when wrong.

**Ignoring context window limits.** A multi-agent system generates a lot of text. Without active context management (compaction, summarization, selective loading), agents degrade as their context fills up.

**No rollback strategy.** If your agents modify shared state (files, databases, APIs), you need a way to undo changes when things go wrong. Git for code, database transactions for data, feature flags for deployments.

## Getting Started

If you're building a multi-agent AI system, start simple:

1. Get one supervisor and one worker communicating via a basic message queue
2. Add a health check that can detect when the system is broken
3. Add persistent logging so you can debug failures
4. Scale to multiple workers only after the single-worker case is reliable
5. Add specialization (dedicated backend, frontend, QA agents) based on actual bottlenecks

The hard part isn't the AI — it's the infrastructure. Agent coordination, error recovery, and persistent memory are what separate a toy from a tool.

---

*I build multi-agent AI systems and automation tools for businesses. If you need help designing or implementing an AI agent architecture, check out my [services](/services) or [get in touch](/contact).*
