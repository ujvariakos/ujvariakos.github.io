---
title: 'User-orchestrator-agent communication (In progress)'
date: '2026-09-18T11:08:40+02:00'
tags: []
featured_image: ""
description: ""
---

# The Problem
Agentic systems become more complex as they gain specialized agents, tools, and skills. An orchestrator commonly decomposes tasks, delegates work, and coordinates execution. Because subagents receive only the context relevant to their tasks, they may encounter decisions that require information or authorization from the user.

Subagents are optimized for reasoning and execution, not conversation management. Allowing them to interact with users directly would require them to maintain conversational state, handle ambiguity, and prioritize competing questions—effectively turning every subagent into a full orchestrator and weakening modularity. Restricting user interaction to the orchestrator creates a single control plane for conversation, making interactions more consistent, auditable, and easier to coordinate across concurrent tasks.

The orchestrator acts as a conversational governor: it aggregates partial signals from subagents, resolves conflicts, prioritizes information needs, and transforms internal uncertainty into structured user-facing queries.

An increasing number of agent products can pause execution to request structured user input.

For example:

- Claude Code — `AskUserQuestion` tool
- OpenAI Codex — `request_user_input` tool

Without an explicit escalation channel, a subagent may fail silently or make an unsafe assumption. Structured uncertainty signals allow missing information to be resolved before the resulting error propagates through the workflow.


# Proposed architecture

## Structured uncertainty signals

A structured uncertainty signal is the foundational data contract of the architecture. It defines what an unresolved request contains, but does not prescribe how that request is resolved, routed, or transported.

A subagent should not communicate with the user directly. When it cannot continue safely, it emits a structured uncertainty signal to the orchestrator. The signal describes why execution is blocked, what information is needed, and what the consequences of each possible response are.

A useful signal distinguishes between:

- `request_kind`: `clarification`, `decision`, `confirmation`, or `permission`
- `reason`: `missing_context`, `ambiguous_requirement`, `conflicting_constraints`, or `risk_threshold_exceeded`
- `blocking`: whether the subagent can continue without an answer
- `question`: the specific information required
- `options`: valid choices and their consequences, when applicable
- `recommendation`: the subagent's preferred choice and rationale
- `fallback`: the safe behavior if no answer is received
- `request_id`, `task_id`, and `agent_id`: identifiers used to route the answer back to the correct suspended task

For example:

```json
{
  "type": "uncertainty_signal",
  "request_id": "req-42",
  "task_id": "deploy-api",
  "agent_id": "deployment-agent",
  "request_kind": "decision",
  "reason": "ambiguous_requirement",
  "blocking": true,
  "question": "Which environment should receive the deployment?",
  "options": [
    {
      "id": "staging",
      "label": "Staging",
      "impact": "Deploys without affecting production users."
    },
    {
      "id": "production",
      "label": "Production",
      "impact": "Makes the release available to users."
    }
  ],
  "recommendation": {
    "option_id": "staging",
    "rationale": "The task does not explicitly authorize a production release."
  },
  "fallback": {
    "action": "pause",
    "reason": "No environment can be selected safely without user input."
  }
}
```

The signal should contain a concise rationale and relevant evidence, but not unrestricted chain-of-thought or private reasoning.

The orchestrator should track each request through a small lifecycle such as `open`, `resolved`, `declined`, `timed_out`, or `cancelled`. A response must reference the original `request_id`, allowing the orchestrator to resume the correct task or apply the declared fallback when no usable answer is available.

## Communication models

The signal contract does not require a particular communication mechanism. A system can exchange uncertainty signals through message passing, shared context, or a combination of both.

### Message passing

With message passing, a subagent explicitly sends an uncertainty signal to an orchestrator. Direct calls or in-memory queues may be sufficient for small systems, while a message bus can provide buffering, delivery guarantees, retries, and correlation for concurrent or distributed agents.

```text
Subagent --uncertainty signal--> Message bus --> Orchestrator
```

### Shared context

With shared context, a subagent publishes the uncertainty signal into shared state. An orchestrator observes or polls that state, claims the request, and writes the resolution back. This resembles a blackboard architecture and allows other authorized agents to contribute relevant information.

```text
Subagent --> Shared context <-- Orchestrator
```

### Hybrid model

A system can store the complete signal and its lifecycle state in shared context while using a message bus to notify the orchestrator that the state has changed. In this model, shared context is the source of truth and the message bus provides timely delivery.

## Escalation policy

### Bubble-up pattern

When a subagent encounters a situation it cannot resolve on its own, it should **propagate the problem up the hierarchy** rather than hallucinate an answer, fail silently, or make a risky assumption.

Each receiving orchestrator first attempts to resolve the signal within its own context and authority. If it cannot, the bubble-up policy forwards the same structured signal to the next level. The user is the final escalation target, not necessarily the first one.

The bubble-up pattern is independent of the communication model. A system can implement it by sending messages to parent orchestrators or by changing the ownership and visibility of a request stored in shared context.

## End-to-end flow

The structured signal defines the payload, the communication model carries it, and the bubble-up policy determines how far it must travel:

```text
+-------------------------------+
| Subagent creates structured   |
| uncertainty signal            |
+---------------+---------------+
                |
                v
+-------------------------------+
| Communication model           |
| - Message passing             |
| - Shared context              |
| - Hybrid                      |
+---------------+---------------+
                |
                v
+-------------------------------+
| Current orchestrator attempts |
| resolution                    |
+---------------+---------------+
                |
        +-------+--------+
        |                |
     resolved         unresolved
        |                |
        v                v
+---------------+  +--------------------+
| Return answer |  | Bubble up to parent|
| to subagent   |  | orchestrator       |
+---------------+  +---------+----------+
                            |
                            v
                  +---------------------+
                  | Resolve or continue |
                  | escalation to user  |
                  +---------------------+
```

In a complex agentic system, there can be multiple orchestrator and subagent layers. Regardless of the communication model or number of layers, the structured uncertainty signal remains the common contract throughout the flow.

This approach is related to structured exception-handling frameworks such as [SHIELDA: Structured Handling of Exceptions in LLM-Driven Agentic Workflows](https://arxiv.org/abs/2508.07935), which combines local handling, flow control, state recovery, and escalation for failures in agentic workflows.
