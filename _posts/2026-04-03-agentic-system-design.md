---
layout: post
title: "Agentic System Design: A Principled Framework"
date: 2026-04-03 09:00:00 -0700
categories: [agentic-systems, system-design]
---

## Core Principle

An agentic system should be designed as a **decision-making system under uncertainty**.

Its job is not to be "right" in a magical sense. Its job is to **choose actions that maximize expected utility under uncertainty**, and to **improve that decision policy over time through feedback and iteration**.

Expected utility is the average value of possible outcomes weighted by their likelihood. Uncertainty covers incomplete knowledge about the world, the user, the task, the tools, or the model's own correctness. Policy is the system's strategy for choosing the next action given the current state.

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

**Why:** a policy can only choose well if the state captures what matters. Gaps here propagate into every downstream decision.

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

**Why:** in production, intelligence is often less about raw reasoning and more about choosing the right next move — including knowing when not to act.

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

The system must estimate where it may be wrong or blind — and distinguish between two fundamentally different failure types:

* **Semantic failures:** the model has the capability to succeed but is unreliable — wrong intermediate reasoning, ambiguous retrieval, inconsistent tool use. These show up as high `pass@k` (succeeds sometimes) but low `pass^k` (fails under consistency pressure).
* **Hard failures:** structural gaps — missing tool, stale data, context that was never retrieved, action that has no grounding in real environment state. These persist regardless of how many attempts are made.

The distinction matters because the fix is different: semantic failures respond to better prompts, examples, and critics; hard failures require tool repair, retrieval expansion, or guardrails.

Sources of uncertainty include:

* ambiguous user intent
* missing context
* poor retrieval quality or retrieval that never ran
* tool unreliability or stale data
* reasoning errors that aren't anchored to retrieved evidence
* distribution shift from training to production

Signals can come from:

* model confidence
* disagreement across models or critics
* retrieval quality scores
* tool execution failures and trace anomalies
* historical failure patterns by issue type
* human feedback and escalation rates

**Why:** uncertainty determines whether the agent should act, gather more evidence, or defer. Misclassifying a hard failure as a semantic one wastes prompt iterations; the inverse misses structural fixes that would have eliminated entire failure classes.

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

A strong agentic system improves through closed-loop policy iteration: observe how the current policy behaves, identify where it underperforms, improve based on evidence, deploy carefully, and repeat.

Core loop:

1. collect trajectories, decisions, tool calls, outputs, and outcomes
2. evaluate success, failure, and quality of intermediate steps
3. cluster recurring failure modes
4. attribute likely root causes
5. update the policy — through better prompts, examples, tool routing, memory, retrieval, or eventually fine-tuning
6. re-evaluate on benchmarks and live traffic

Policy improvement does not require RL or weight updates. Prompt refinement, retrieval tuning, and deterministic guards around failure-prone regions are all valid policy updates.

A useful lens for calibrating the loop: distinguish `pass@k` (did the system ever succeed across k trials?) from `pass^k` (did it succeed every time?). `pass@k` diagnoses capability gaps; `pass^k` diagnoses reliability gaps. These require different interventions — more capable examples vs. tighter guardrails and routing.

**Why:** without a feedback loop, the system is static. With a weak loop, it learns the wrong lessons. The deeper goal is that the system accumulates operational knowledge about what actions work, when, and why — and develops specific, not generic, fixes for each failure class.

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

## Production Evidence: The Framework in Action

The evolution stages above are not abstract — they map directly to how real failure modes get discovered, diagnosed, and resolved. The table below traces six production failure patterns across domains, showing how each stage of the system's maturity changes what you can see, what you can measure, and how the policy adapts.

The columns correspond to Stages 2–5 of the evolution path.

| **Domain & Issue** | **Stage 2 — How the Instrumented Agent Catches It** | **Root Cause (Semantic / Hard)** | **Stage 3 — How the Evaluated Agent Measures It** | **Stage 4 — How the System Adapts** | **Stage 5 — How Learning Maximizes Expected Utility** |
|---|---|---|---|---|---|
| **Customer Support**<br>Agent approves or denies refunds inconsistently against policy | Traces show high refund approval variance across similar inputs; escalation rate spikes; CSAT drops in cohort analysis | **Semantic failure** — ambiguous policy language interpreted differently across calls; state missing order value and account history | Golden dataset of policy edge cases with compliance rubric; LLM judge scores against policy text; pass^k reveals inconsistency even when pass@k is high | Route high-value orders to human review; inject policy document into context; add a policy-lookup tool call before decision | Cluster denied-then-escalated cases; identify which policy clauses drive inconsistency; update few-shot examples with the hard cases; track policy compliance rate as a primary metric |
| **Code Generation**<br>Agent writes code that passes tests but introduces silent regressions | CI passes but prod monitoring catches regression; trace shows agent optimized for test pass, not broader invariants | **Semantic failure (proxy metric trap)** — agent learned to satisfy the test suite as a proxy for correctness; the proxy diverged from true utility | Extend eval suite with integration tests, static analysis, and type checks beyond the immediate task; evaluator-optimizer pattern: a critic LLM reviews for spec adherence, not just test pass | Add a critic step before accepting code; require explicit reasoning about edge cases; surface unwritten invariants via structured prompts | Collect prod regressions → add to golden regression suite; track mean time to regression as a signal; fine-tune critic on confirmed failure cases when volume justifies it |
| **Clinical/Healthcare**<br>Agent returns plausible-sounding but incorrect medication information | Trace shows reasoning chain generated conclusion before retrieval completed; groundedness check flags claim not found in retrieved documents | **Semantic failure (hallucination)** — CoT reasoning not anchored to retrieved evidence; model interpolated from parametric knowledge rather than grounding in source | Groundedness grader verifies every factual claim traces to a cited source; coverage grader checks required safety information is present; human pharmacist review on random sample | Require source citation before stating dosage; expand retrieval depth; add a verification step that re-retrieves after initial answer draft; hard guardrail on any answer with no grounded citation | Collect ungrounded outputs → curate authoritative medical KB; instrument retrieval quality scores over time; route queries with no high-confidence retrieval to human escalation |
| **Financial Services**<br>Agent recommends actions based on stale market data | Tool execution traces show cache hit with timestamp > 15 minutes old; outcome monitoring shows recommendation diverged from actual market close | **Hard failure** — tool returning cached data without freshness validation; no signal in state about data age | Eval tasks that inject known data-drift scenarios; grader checks that recommendation is consistent with data timestamp; time-sensitivity classification in task type | Add data freshness validation to tool schema; guardrail that blocks any recommendation if data is stale beyond threshold; route time-sensitive queries through live-data tool, not cache | Instrument all tool calls with freshness and latency metadata; use freshness violation rate as a policy signal; learn which query types are most sensitive to staleness and apply tighter routing rules to those |
| **RAG / Research Agent**<br>Agent gives incomplete answers, missing critical documents | Coverage trace shows required facts absent from output; retrieval log shows relevant documents existed but were not retrieved (low similarity score despite topical relevance) | **Hard failure (retrieval gap)** — dense embedding similarity missed conceptual relevance; query formulation did not surface the relevant shard | Coverage graders check for required facts against ground truth; multi-hop retrieval benchmarks test whether agent can reason across document boundaries; retrieval recall measured separately from answer quality | Hybrid retrieval (dense + sparse); query reformulation before retrieval; expand retrieval budget for low-confidence answers; add an explicit "check coverage" step that re-retrieves if answer confidence is low | Collect coverage failures → curate hard retrieval examples; fine-tune retriever on domain-specific concepts; track retrieval recall as a primary metric independent of answer quality; progressively tighten coverage thresholds |
| **Multi-Step Planning**<br>Agent enters repetitive action loops without making progress | Step count exceeds threshold; trace shows identical tool calls repeated consecutively; environment state unchanged across N steps | **Semantic failure (state not updating)** — agent not registering that prior action failed or returned null; policy has no explicit dead-end detection | Trajectory-level evals with known dead-end scenarios; grader checks for loop signatures (repeated actions, stagnant environment state); pass^k on multi-step tasks reveals loop risk under consistency pressure | Loop detection guardrail: force alternative action after k identical attempts; add explicit "progress check" step after each tool call; route to human if progress stalls after two alternatives | Identify loop entry conditions from traced failures → encode as explicit stop-or-escalate triggers in the policy; expand golden dataset with recovered-loop trajectories as positive examples; track loop rate by task type as a primary reliability metric |

*Semantic failure: the model has the capability but is not reliably using it — fixable through better prompts, examples, or critics. Hard failure: a structural gap — missing tool, stale data, retrieval that never ran — fixable only through system changes.*

---

## Design Rules That Usually Matter

**Use deterministic logic where the task is encodable; reserve LLM reasoning for where it is not.**
Route well-defined intents — order status lookup, password reset, structured data extraction — with code, not a model. LLM reasoning belongs where ambiguity or abstraction is irreducible: interpreting customer frustration, synthesizing conflicting policies, handling novel edge cases. Routing everything through a model adds latency and variance where neither is needed.

**Design tool interfaces to eliminate failure classes, not just reduce their frequency.**
A tool that accepts relative file paths will produce path errors; one that requires absolute paths structurally cannot. A tool that returns raw HTML will confuse the model; one that returns structured JSON with named fields largely won't. The interface shapes the failure modes. An hour of ACI design prevents weeks of prompt debugging.

**Do not add architectural complexity without a measurement showing it helps.**
Before adding an orchestrator-worker layer, verify that a single-agent loop with better prompts does not already solve the problem. Before adding a critic, measure whether it reduces error rate on your eval set. Complexity added without measurement tends to obscure root causes and make the system harder to improve later.

**Define utility at the level of real outcomes, not the metrics that are easy to instrument.**
A code agent optimizing for test-pass rate will eventually write tests that pass trivially. A support agent optimizing for CSAT will learn to apologize its way to five stars without resolving anything. Define value at the outcome level — fewer production regressions, fewer escalations, lower refund rate — and treat instrumented metrics as proxies that need to be audited.

**Diagnose the failure type before deciding on the fix.**
If the agent succeeds 3 times out of 10 (high `pass@k`, low `pass^k`), the model has the capability but is inconsistent — better prompts, few-shot examples, or a critic will help. If it fails every time regardless of phrasing, it is a hard failure — the tool is missing, the context was never retrieved, or the data is stale. Prompt engineering a hard failure wastes time and misleads you about what the real problem is.

**Weight utility for downside scenarios, not just average performance.**
A 95%-accurate agent that gives wrong medical dosage or submits incorrect financial transactions 5% of the time is not a 95%-correct system — it is a liability. Define separate utility weights for high-stakes output classes. Optimize those separately from the aggregate.

**Make human escalation an explicit, instrumented action — not an emergency exit.**
Define confidence thresholds below which the agent surfaces to a human rather than acting. For irreversible actions — transaction submission, file deletion, sending external communications — require confirmation regardless of confidence. If escalation is not a named action in the policy with its own trace and outcome tracking, it is invisible to the feedback loop.

**Treat every prompt change as a policy change — diff, test, and version it.**
A prompt edit changes the policy. Run it against the regression suite; compare `pass^k` before and after. Silent prompt edits are one of the most common sources of gradual performance degradation, because the change is invisible in deployment but visible only later in outcome metrics.

**Build a pipeline from production failure to golden dataset.**
Every escalation is a test case the eval suite missed. Every novel failure in production is a gap. The pipeline should be: prod failure → triage → classify root cause → add to golden set → verify fix → regression guard. Teams that do this continuously have eval suites that stay current with production; teams that do not find their evals saturating while prod failures keep changing.

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

---

## References

* Russell, S. & Norvig, P. *Artificial Intelligence: A Modern Approach* (4th ed., 2020) — foundational treatment of utility-based agents and rational decision-making under uncertainty.
* Sutton, R. & Barto, A. *Reinforcement Learning: An Introduction* (2nd ed., 2018) — policy iteration, generalized policy improvement, and the MDP framework underlying adaptive policies.
* Yao, S. et al. "ReAct: Synergizing Reasoning and Acting in Language Models." *ICLR 2023.* [arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629) — demonstrates interleaved reasoning and action selection in LLM agents.
* Wei, J. et al. "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models." *NeurIPS 2022.* [arxiv.org/abs/2201.11903](https://arxiv.org/abs/2201.11903) — shows the role of explicit reasoning traces in agent reliability.
* Anthropic. "Building Effective Agents." [anthropic.com/research/building-effective-agents](https://www.anthropic.com/research/building-effective-agents) — practical architecture patterns for production agentic systems.
* Anthropic. "Demystifying Evals for AI Agents." [anthropic.com/engineering/demystifying-evals-for-ai-agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — evaluation and feedback loop design for agentic workflows.
