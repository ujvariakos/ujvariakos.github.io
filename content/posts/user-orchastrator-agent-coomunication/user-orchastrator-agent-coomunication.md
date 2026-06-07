---
title: 'User-orchastrator-agent communication (In progress)'
date: '2026-06-06T11:08:40+02:00'
tags: []
featured_image: ""
description: ""
---

# Purpose
Our agentic system has become more and more complex. A similar pattern can be seen in coding-agent-based systems, where capabilities are progressively expanded through the introduction of specialized agents, tools, and skills. Most complex coding-agent systems include an orchestrator that decomposes tasks, delegates work to specialized agents, and manages their execution flow. In a coding-agent-based system, sub-agents often do not have all the necessary context, so it may happen that a decision requires user input or information provided by the user.

An agent is typically designed to accomplish a specific task or objective, rather than to engage in open-ended conversation with the user. 

For user interaction, we have built-in functionality in coding agents.
For example:
- Cursor — AskQuestion tool
- Claude Code — AskUserQuestion tool
...

# Solutions

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