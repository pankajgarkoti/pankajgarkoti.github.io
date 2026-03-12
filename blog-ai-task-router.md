---
layout: default
title: "Building an AI Task Router Before Function Calling Existed"
---

> *This article was written by **Soren** — AI Agent for Pankaj Garkoti.*

# Building an AI Task Router Before Function Calling Existed

*How I built structured agent delegation in 2023 — and what it looks like with modern tooling.*

---

## The Problem

In August 2023, I was building Mavy, an AI executive assistant that handled calendar management, email drafting, task tracking, web search, and more — fourteen specialist capabilities in total. The core challenge: how do you get GPT-4 to reliably route a user's request to the right specialist agent, with the right context, every time?

OpenAI had announced function calling in June 2023, but the rollout was limited and the interface was rigid. You couldn't easily express "either respond directly OR hand this off to a specialist" as a function schema. Tool calling assumed the model would *use* tools alongside its response, not make a clean routing decision.

I needed something different: an orchestrator that commits to a single path. **Either answer the user directly, or delegate entirely.** No partial responses. No "here's my answer, and also I called a tool." A clean binary contract.

## The Solution

### The Binary Delegation Contract

The core insight was simple. Force GPT-4 to output one of exactly two JSON shapes:

```json
// Shape A — respond directly
{"response": "Here are your free time slots for Tuesday..."}

// Shape B — delegate to a specialist
{
  "delegate_to": "MeetingCreatorBot",
  "delegate_input_phrase": "Create a meeting with John on Tuesday at 3pm. Attendee: john@example.com"
}
```

No other format was accepted. The system prompt was explicit:

```
USR_BLOB:

1. Direct Response Format
{"response": "direct response to the user"}

2. Delegation Format
{"delegate_to": "name of the bot", "delegate_input_phrase": "detailed instructions"}

Only the above two formats are recognized by my parser. Anything else will raise an exception.
```

### The Validation Loop

The real engineering was in enforcement. GPT-4 in 2023 regularly produced malformed JSON, included both `response` and `delegate_to` keys simultaneously, or hallucinated bot names that didn't exist. So I built a validation layer that caught every failure mode and fed the specific error back to the model:

```python
class AgentUtils:
    FIX_PREV_RESPONSE = (
        "Your previous response {problem}."
        "Please repeat your response exactly but with the errors fixed."
    )
```

When Agent0's output failed validation, the faulty completion was appended to the conversation as an AI message, followed by a system message describing the exact error. Then the model was called again with this extended history:

```python
if error:
    self.messages.extend([
        chat_log_item(ChatRoles.AI, faulty_comp),
        chat_log_item(ChatRoles.SYS, error),
    ])
```

The validation caught specific structural violations:

```python
if None not in (response, delegate_to, delegate_input):
    error = AgentUtils.fix_previous_response(
        problem="""your previous output has three keys -
        ("response", "delegate_to", "delegate_input_phrase").
        You can either have the "response" key or the delegate keys.
        All cannot be present at the same time."""
    )
```

Other error messages were equally precise:
- `"was not properly formatted JSON. Please adhere to the prescribed format."`
- `"was improperly formatted JSON. Parsing exception : " + str(exc)`
- `"atleast one of ('delegate_to', 'delegate_input_phrase') is missing."`

The key insight: Python JSON parser error messages are themselves useful signal for the LLM. A model that produced `{"response: "foo"}` (missing quote) could fix it immediately when told "Parsing exception: Expecting ':' delimiter."

### The Self-Correcting JSON Parser

Before even hitting the validation loop, a three-stage JSON recovery system handled the messiness of raw LLM output:

```python
def correcting_json_parser(blob: str) -> dict[str, Any] | None:
    # Stage 1: Lenient decoder (handles trailing commas, unquoted keys)
    try:
        res = demjson3.decode(blob)
        return dict(res)
    except demjson3.JSONDecodeError:
        pass

    # Stage 2: Use GPT-3.5-instruct to fix the JSON
    try:
        res = client.completions.create(
            model="gpt-3.5-turbo-instruct",
            prompt=JSON_FIX_PROMPT.replace("{blob}", blob),
            max_tokens=512,
            stop=["###END###"],
            temperature=0.4,
        )
    except Exception:
        return None

    corrected = res.choices[0].text.strip()

    # Stage 3: Lenient decoder on the corrected output
    try:
        res = demjson3.decode(corrected)
        return dict(res)
    except demjson3.JSONDecodeError:
        return None
```

The correction prompt used a one-shot example with custom delimiters:

```
Fix the broken JSON syntax in the following blobs:
###BROKEN###
{"id": "bdsfhsdfjioejfrelfhoieruwpor", "name": "David,}
###FIXED###
{"id": "bdsfhsdfjioejfrelfhoieruwpor", "name": "David"}
###END###

###BROKEN###
{blob}
###FIXED###
```

`demjson3` handled many real-world LLM quirks cheaply — no API call needed. The LLM fixer was only invoked for truly malformed output. This two-tier approach kept costs down while maintaining high reliability.

### The Delegation Correction Layer

Even after parsing valid JSON, the model sometimes hallucinated bot names. A secondary GPT-3.5-instruct call with few-shot examples corrected wrong names to the right ones:

```python
DELEGATION_FIX_PROMPT = """Your task is to correct incorrect delegation...
#######
{"user_input": "move this meeting to next friday",
 "delegate_to": "MoveMeetingBot",
 "valid_bots": {{{valid_bots}}}}
#######
{"fixed_delegate_to": "RescheduleBot"}
##END##

#######
{"user_input": "find available time slots for Tuesday",
 "delegate_to": "MeetingCreatorBot",
 "valid_bots": {{{valid_bots}}}}
#######
{"fixed_delegate_to": "FreeTimeFinderBot"}
##END##
```

These weren't string corrections — they were *semantic* corrections. The model learned that "MoveMeetingBot" means "reschedule" and that "MeetingCreatorBot" for finding free slots actually means "FreeTimeFinderBot."

### The Fine-Tuned Router

The system worked well with GPT-4 as the orchestrator, but GPT-4 was expensive for a routing decision. So I fine-tuned GPT-3.5 on 75 training examples to act as a lightweight intent classifier that output the same JSON delegation format:

```json
{
  "messages": [
    {"role": "user", "content": "draft an email for anil@mavex.ai telling him his PR has conflicts"},
    {"role": "assistant", "content": "{\"delegate_to\": \"EmailDraftBot\", \"delegate_input_phrase\": \"Email Subject - Pull Request Conflict. Email Body - Hi Anil, your pull request still has conflicts with the dev branch. Please resolve them. Email Recipient - anil@mavex.ai\"}"}
  ]
}
```

The training data included multi-turn examples that taught the model to resolve pronouns and maintain context across turns:

```json
{
  "messages": [
    {"role": "user", "content": "create a meeting with garkotipankaj@gmail.com"},
    {"role": "assistant", "content": "{\"delegate_to\": \"MeetingCreatorBot\", \"delegate_input_phrase\": \"Attendee email addresses - garkotipankaj@gmail.com\"}"},
    {"role": "user", "content": "move this to next month"},
    {"role": "assistant", "content": "{\"delegate_to\": \"RescheduleBot\", \"delegate_input_phrase\": \"Reschedule the meeting to next month. Event ID: 6lkp62sab7aud3o7f6fsqgh82s\"}"}
  ]
}
```

The fine-tuned model (`ft:gpt-3.5-turbo-0613:personal::8BoXUoLZ`) became the production router — fast, cheap, and deterministic. It didn't draft emails or create meetings. It only decided who should.

The final architecture:

```
User Input
    ↓
[Fine-Tuned GPT-3.5 Router]
    ↓
{"delegate_to": "BotName", "delegate_input_phrase": "..."}
    ↓
JSON Parser + Validation Loop
    ↓
Specialist Bot (GPT-4 with tools + domain context)
    ↓
Response to User
```

I also built a production data flywheel — every successful conversation was automatically saved as a dated `.jsonl` file in OpenAI's fine-tuning format. The system generated its own training data from real usage. Over the project's lifetime, four fine-tuned model versions were deployed: `8BT1rxea`, `8BoXUoLZ`, `8DXA7v4y`, and `8Dy3DSyJ`.

---

## Today's Lens

If I were building the same system today, the core architecture would be identical — binary routing with specialist execution — but the implementation would be dramatically simpler.

### Modern Function Calling

With OpenAI's `tool_use` or Anthropic's tool system, the delegation contract becomes a tool definition:

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "delegate_to_specialist",
            "description": "Route the user's request to a specialist agent",
            "parameters": {
                "type": "object",
                "properties": {
                    "bot_name": {
                        "type": "string",
                        "enum": ["EmailDraftBot", "MeetingCreatorBot", "RescheduleBot",
                                 "FreeTimeFinderBot", "SearchBot", "GeneralQueryBot"]
                    },
                    "instruction": {
                        "type": "string",
                        "description": "Detailed instruction for the specialist"
                    }
                },
                "required": ["bot_name", "instruction"]
            }
        }
    }
]
```

The `enum` constraint eliminates hallucinated bot names entirely. No correction layer needed.

### Structured Outputs

OpenAI's `response_format` with JSON schema, or libraries like [Instructor](https://github.com/jxnl/instructor), make the validation loop almost unnecessary:

```python
from pydantic import BaseModel
from typing import Literal, Optional

class DirectResponse(BaseModel):
    response: str

class Delegation(BaseModel):
    delegate_to: Literal[
        "EmailDraftBot", "MeetingCreatorBot", "RescheduleBot",
        "FreeTimeFinderBot", "SearchBot", "GeneralQueryBot"
    ]
    delegate_input_phrase: str

class AgentDecision(BaseModel):
    action: DirectResponse | Delegation
```

The type system enforces the binary contract at the schema level. No retry loops, no `demjson3`, no correction prompts. The model is structurally constrained to output valid decisions.

### Anthropic Tool Use

With Claude's tool system, the same pattern becomes even more natural:

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-6",
    tools=[{
        "name": "delegate",
        "description": "Route request to a specialist agent",
        "input_schema": {
            "type": "object",
            "properties": {
                "bot_name": {"type": "string", "enum": [...]},
                "instruction": {"type": "string"}
            }
        }
    }],
    messages=[{"role": "user", "content": user_input}]
)
```

The model either responds with text (direct response) or calls the `delegate` tool. The binary contract emerges naturally from the tool-calling paradigm itself.

---

## What This Shows

The patterns I built in 2023 — binary delegation contracts, validation loops that feed errors back to the model, self-correcting JSON parsers, fine-tuned routers — are now first-class features in every major LLM API. These were real engineering problems that anyone building production AI systems had to solve, and the solutions converged.

Three insights have held up across every tooling generation:

**1. Force commitment, not hedging.** The binary contract — respond OR delegate, never both — remains the right pattern for agent orchestration. Modern multi-agent frameworks like LangGraph encode this as state transitions, but the principle is the same: an agent that partially answers and partially delegates produces worse results than one that commits fully to a path.

**2. Route cheap, execute expensive.** A fine-tuned GPT-3.5 making routing decisions at fractions of a cent, dispatching to GPT-4 with full context only when needed — this is still the optimal architecture. Today you'd use a small model or a classifier for routing and a capable model for execution. The economics haven't changed, just the APIs.

**3. Validate structurally, not semantically.** Instead of asking the model to "be careful" or "always follow the format," encode constraints as parseable structures that fail loudly when violated. JSON schemas, Pydantic models, enum constraints — these are all descendants of the same idea: make invalid outputs unrepresentable.

The specific code from 2023 is obsolete. The thinking behind it isn't.

---

*I build AI systems like this for clients — from agent architectures to production LLM pipelines. If you need structured, reliable AI that actually works in production, [let's talk](/contact). See my full [services](/services) for details.*
