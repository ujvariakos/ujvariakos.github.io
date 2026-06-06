---
title: 'User-orchastrator-agent communication'
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
Cursor — AskQuestion tool
Claude Code — AskUserQuestion tool
...

# Solutions

## Bubble-up/exception pattern

## Message Bus

## Shared context