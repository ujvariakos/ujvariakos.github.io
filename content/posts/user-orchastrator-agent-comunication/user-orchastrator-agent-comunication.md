---
title: 'User-orchastrator-agent communication (In progress)'
date: '2026-06-06T11:08:40+02:00'
tags: []
featured_image: ""
description: ""
---

# The Problem
Our agentic system has become more and more complex. A similar pattern can be seen in coding-agent-based systems, where capabilities are progressively expanded through the introduction of specialized agents, tools, and skills. Most complex coding-agent systems include an orchestrator that decomposes tasks, delegates work to specialized agents, and manages their execution flow. In a coding-agent-based system, subagents often do not have all the necessary context, so it may happen that a decision requires user input or information provided by the user.

Subagents are optimized for reasoning and execution, not conversation management. Allowing them to directly interact with users would require them to maintain conversational state, ambiguity handling, and prioritization logic — effectively turning every subagent into a full orchestrator, which defeats modularity. The restriction on subagent-user interaction enforces a single control plane for dialogue management, ensuring that user interaction remains deterministic, auditable, and globally consistent across the system.

The orchestrator acts as a conversational governor: it aggregates partial signals from subagents, resolves conflicts, prioritizes information needs, and transforms internal uncertainty into structured user-facing queries.

More and more coding agents have the "user-interruptable model".

For example:
- Cursor — AskQuestion tool
- Claude Code — AskUserQuestion tool
- ...

In practice, limiting subagents to silent failure until orchestration can increase the risk of error propagation. A structured mechanism for subagents to request clarification improves system reliability by ensuring that missing context is resolved at the earliest possible stage in the reasoning pipeline, rather than being inferred downstream.


# Solutions

## Structured uncertanity signal

The subagent does not communicate with the user. Instead, it sends a typed event to the orchestrator. The orchestrator collects the information and responds to the subagent.
- need_user_input
- missing_context
- confirmation_required

## Bubble-up pattern
When a subagent encounters a situation it cannot resolve on its own, it should **propagate the problem up the hierarchy** rather than hallucinate an answer, fail silently, or make a risky assumption.

Tipical workflow


```goat
+------------------+
| Local handling   |
+--------+---------+
         |
         v unresolved
+------------------+
| Escalate to      |
| orchestrator     |
+--------+---------+
         |
         v
+------------------+
| Orchestrator     |
| decides          |
+--------+---------+
         |
         v needs user input
+------------------+
| Bubble up to     |
| user             |
+------------------+
```
In a complex agentic system, there can be additional orchastrator and subagent layers.


[SHIELDA: Structured Handling of Exceptions in LLM-Driven Agentic Workflows](https://arxiv.org/abs/2508.07935)

## Message Bus

## Shared context