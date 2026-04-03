---
layout: post
title: "Agentic System Design: A Principled One-Pager"
date: 2026-04-03 09:00:00 -0700
categories: [agentic-systems, system-design]
---

## Core Principle

An agentic system should be designed as a **decision-making system under uncertainty**.

Its job is not to be "right" in a magical sense. Its job is to **choose actions that maximize expected utility under uncertainty**, and to **improve that decision policy over time through feedback and iteration**.

In simple terms:

* **Utility** = how valuable an outcome is
* **Expected utility** = the average value of possible outcomes, weighted by how likely they are
* **Uncertainty** = incomplete knowledge about the world, the user, the task, the tools, or the model's own correctness
* **Policy** = the system's strategy for what to do next given the current state

So the design problem becomes:

> How do we build a system that repeatedly chooses the next best action, despite uncertainty, and becomes better at doing so over time?

---

## Why This Principle Matters

Most weak agent systems are built as prompt chains.
Most strong agent systems are built as **adaptive policies**.

A prompt chain says:

> "Given input X, follow these steps."

A principled agent says:

> "Given this state, these uncertainties, these possible actions, and these tradeoffs, what action is most likely to lead to the best outcome?"

This shift matters because real-world tasks are not deterministic:

* user intent may be ambiguous
* retrieved context may be incomplete
* tool outputs may fail or conflict
* reasoning may drift or hallucinate
* the cost of being wrong may vary by scenario

An agent must therefore reason not only about **what to do**, but also about:

* how sure it is
* what the downside risk is
* when to ask for more information
* when to use tools
* when to escalate to a human

---

## System Internals Mapped to the Principle

### 1. State Representation

The system needs a working view of the current situation.

This includes:

* user request and task type
* conversation history
* retrieved knowledge
* tool outputs
* prior failures or past outcomes
* confidence / uncertainty signals
* constraints such as latency, cost, and safety

**Why:** a policy can only choose well if the state captures what matters. Bad state representation leads to bad action selection.

---

### 2. Action Space

The agent must have a clear set of actions it can take.

Examples:

* answer directly
* retrieve more context
* call a tool
* ask a clarifying question
* decompose the task
* critique its own draft
* escalate to a human
* stop

**Why:** intelligence in production is often less about raw reasoning and more about choosing the right next move.

---

### 3. Utility Model

The system needs a notion of what counts as a good outcome.

Utility may combine:

* task success
* correctness
* customer satisfaction
* latency
* cost
* safety / policy compliance
* business impact such as refund reduction or resolution rate

In practice, there is rarely one perfect metric. Utility is often a weighted bundle of outcomes.

**Why:** if you do not define value, the system cannot optimize coherently. It will drift toward whatever proxy is easiest to improve.

---

### 4. Uncertainty Estimation

The system must estimate where it may be wrong or blind.

Sources of uncertainty include:

* ambiguous user intent
* missing context
* poor retrieval quality
* tool unreliability
* reasoning errors
* distribution shift

Signals can come from:

* model confidence
* disagreement across models or critics
* retrieval quality scores
* tool execution failures
* historical failure patterns
* human feedback

**Why:** uncertainty is what determines whether the agent should act boldly, gather more evidence, or defer.

---

### 5. Policy

The policy is the engine that maps state to action.

Examples:

* if confidence is high and task is simple, answer directly
* if knowledge is missing, retrieve
* if retrieval is conflicting, compare sources
* if correctness is high-stakes and uncertainty is high, escalate
* if tool choice is uncertain, run a critic or planner first

The policy may be encoded through:

* prompts
* routing logic
* heuristics
* learned policies
* memory-driven selection
* tool-use strategies

**Why:** the policy is the actual expression of intelligence in the system. The model alone is not the system.

---

### 6. Feedback and Learning Loop

A strong agentic system improves through closed-loop learning.

Core loop:

1. collect trajectories, decisions, tool calls, outputs, and outcomes
2. evaluate success, failure, and quality of intermediate steps
3. cluster recurring failure modes
4. attribute likely root causes
5. update the policy, prompts, tools, memory, retrieval, or model
6. re-evaluate on benchmarks and live traffic

**Why:** without a feedback loop, the system is static. With a weak feedback loop, it learns the wrong lessons. With a principled loop, it becomes progressively more robust.

---

## What "Policy Iteration" Means Here

Policy iteration means:

* observe how the current policy behaves
* identify where it underperforms
* improve the policy based on evidence
* deploy carefully
* repeat

This does not have to mean only RL or weight updates.
It can happen through:

* better prompts
* better examples
* improved tool routing
* stronger memory use
* retrieval improvements
* deterministic guards around failure-prone regions
* eventually, fine-tuning or policy learning when stable patterns emerge

The deeper idea is:

> The system should not merely execute. It should accumulate operational knowledge about what actions work, when, and why.

---

## Evolution Path of a Mature Agentic System

### Stage 1: Prompted workflow

The system is mostly prompt logic with basic tool use.

### Stage 2: Instrumented agent

The system captures traces, failures, and outcomes.

### Stage 3: Evaluated agent

The system has benchmarks, golden datasets, critics, and regression checks.

### Stage 4: Adaptive policy system

The system changes routing, prompts, and guardrails based on observed evidence.

### Stage 5: Learning system

The system continuously improves from live traffic, human feedback, and structured evaluation, while remaining stable under change.

---

## Design Rules That Usually Matter

* Prefer deterministic handling where the task can be encoded reliably.
* Use LLM reasoning where ambiguity or abstraction is irreducible.
* Separate metrics from true value; proxies can mislead.
* Distinguish noise from causal contributors in traces.
* Optimize not just for average performance, but for downside risk in important cases.
* Keep humans in the loop where uncertainty or stakes are high.
* Version and test every policy update to avoid regressions.
* Continuously expand the golden dataset with new, diverse, real failures.

---

## Final Takeaway

A principled agentic system is not just a model with tools.
It is a **policy-driven adaptive system** that:

* represents state
* acts through a policy
* estimates uncertainty
* optimizes utility
* learns from feedback
* evolves safely over time

The north star is:

> **maximize expected utility under uncertainty via iterative policy improvement**

That one sentence connects the theory and the engineering:

* expected utility explains what good performance means
* uncertainty explains why naive automation fails
* policy explains how the system acts
* iteration explains how the system evolves

That is the backbone of serious agentic system design.
