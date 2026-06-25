# 10. Agent Evals

> **Purpose** — Evaluate autonomous LLM agents — systems that plan, act, observe, and iterate over multiple steps to accomplish goals. Agent evaluation is fundamentally harder than single-turn eval because agents make sequential decisions with compounding consequences. This section covers trajectory evaluation, planning quality, tool use, self-correction, and the benchmarks used to measure agent capabilities.

---

## Table of Contents

- [What Are LLM Agents?](#what-are-llm-agents)
- [Why Agent Evals Are Hard](#why-agent-evals-are-hard)
- [What to Evaluate](#what-to-evaluate)
- [Agent Evaluation Dimensions](#agent-evaluation-dimensions)
- [Trajectory Evaluation](#trajectory-evaluation)
- [Planning Quality](#planning-quality)
- [Tool Use Evaluation](#tool-use-evaluation)
- [Self-Correction & Reflection](#self-correction--reflection)
- [Agent Benchmarks](#agent-benchmarks)
- [Evaluation Strategies](#evaluation-strategies)
- [Designing Agent Eval Suites](#designing-agent-eval-suites)
- [Common Failure Modes](#common-failure-modes)
- [Common Mistakes](#common-mistakes)
- [Key Takeaways](#key-takeaways)

---

## What Are LLM Agents?

An **agent** is a system that uses an LLM to autonomously plan and execute multi-step actions to achieve a goal.

```
                    ┌──────────────────────────────────┐
                    │           AGENT LOOP              │
                    │                                    │
    Goal ──────────>│  Plan → Act → Observe → Reflect   │──────> Result
                    │    ↑                      │        │
                    │    └──────────────────────┘        │
                    │         (iterate)                  │
                    └──────────────────────────────────┘
                              │
                         Uses Tools:
                    - Search, Browse, Code
                    - APIs, Databases, Files
                    - Other agents
```

### Agent vs Workflow

| Dimension | Workflow | Agent |
|---|---|---|
| **Control flow** | Developer defines the path | Model decides the path |
| **Steps** | Fixed (2–10) | Variable (could be 1 or 50) |
| **Tool selection** | Pre-assigned per step | Model chooses which tool to use |
| **Planning** | None (hard-coded) | Model generates and revises plan |
| **Error handling** | Developer codes fallbacks | Model self-corrects |
| **Eval complexity** | Medium | High |

---

## Why Agent Evals Are Hard

| Challenge | Description |
|---|---|
| **Non-deterministic paths** | Same goal can be achieved via many different action sequences |
| **Compounding errors** | A wrong action at step 3 can make steps 4–10 irrelevant |
| **Credit assignment** | When the agent fails, which action caused the failure? |
| **Long horizons** | Agents may take 10–50+ steps, making evaluation expensive |
| **Outcome ≠ process** | The agent may get the right answer via a terrible process (or vice versa) |
| **No single "correct" trajectory** | Multiple valid paths to the same goal |
| **Environment dependence** | Agent performance depends on tool reliability, API responses, etc. |

---

## What to Evaluate

### The Three Levels of Agent Evaluation

```
Level 1: OUTCOME    — Did the agent achieve the goal?
Level 2: PROCESS    — Did the agent take reasonable steps?
Level 3: EFFICIENCY — Did the agent achieve the goal efficiently?
```

All three matter. An agent that gets the right answer after 50 random steps is worse than one that gets it in 5 targeted steps.

---

## Agent Evaluation Dimensions

| Dimension | Definition | How to Measure | Example |
|---|---|---|---|
| **Task Completion** | Did the agent achieve the specified goal? | Binary pass/fail or partial credit | "Was the bug fixed?" |
| **Plan Quality** | Was the plan logical and efficient? | LLM judge rates the plan | "Did the plan identify the right files to edit?" |
| **Tool Selection** | Did the agent choose the right tools? | Compare selected tools vs optimal tools | "Used search instead of code execution" |
| **Tool Argument Quality** | Were tool inputs correct and well-formed? | Validate arguments against schema | "Searched for correct query, not gibberish" |
| **Tool Success Rate** | What fraction of tool calls succeeded? | success_count / total_calls | 85% of tool calls returned useful results |
| **Trajectory Quality** | Was the overall sequence of actions reasonable? | Trace evaluation by LLM judge or human | "No loops, no irrelevant actions" |
| **Self-Correction** | Did the agent recover from mistakes? | Track error → recovery sequences | "After a failed API call, tried alternative" |
| **Recovery Rate** | When errors occur, how often does the agent recover? | recovered / total_errors | 70% of errors led to successful recovery |
| **Efficiency** | Steps taken vs minimum steps needed | actual_steps / optimal_steps | 8 steps vs optimal 5 |
| **Cost** | Total tokens and API calls | Sum of all LLM + tool costs | $0.45 per task |
| **Latency** | Total time to complete the task | Wall-clock time | 45 seconds |

---

## Trajectory Evaluation

A **trajectory** is the full sequence of actions the agent took to (attempt to) complete a task.

### Example Trajectory

```
Step 1: [THINK] "I need to find the bug in the login function"
Step 2: [TOOL]  search_code("login function") → found auth.py:L45-L80
Step 3: [THINK] "The password comparison uses == instead of constant-time compare"
Step 4: [TOOL]  edit_file("auth.py", line=52, old="==", new="hmac.compare_digest")
Step 5: [TOOL]  run_tests() → 14/14 passed ✅
Step 6: [DONE]  "Fixed timing attack vulnerability in login comparison"
```

### Trajectory Scoring Approaches

| Approach | How It Works | Pros | Cons |
|---|---|---|---|
| **Outcome-only** | Only check if the final result is correct | Simple, objective | Misses process quality |
| **Step-level scoring** | Score each action independently | Detailed, identifies specific failures | Expensive, subjective |
| **Trajectory comparison** | Compare against expert reference trajectory | Captures process quality | Multiple valid trajectories exist |
| **LLM trajectory judge** | Have a judge LLM evaluate the full trajectory | Captures reasoning quality | Judge may not understand agent actions |
| **Checkpoint evaluation** | Define intermediate milestones and check if the agent hits them | Objective, partial credit | Requires defining checkpoints per task |

### Trajectory Metrics

| Metric | Formula Intuition | What It Captures |
|---|---|---|
| **Step accuracy** | correct_actions / total_actions | Quality of individual decisions |
| **Unnecessary action rate** | (total - optimal) / total | Wasted effort |
| **Loop detection** | repeated_actions / total_actions | Agent stuck in loops |
| **Goal progress** | checkpoints_hit / total_checkpoints | Partial completion |
| **Reasoning quality** | LLM judge score on think steps | Quality of the agent's reasoning |

---

## Planning Quality

| Aspect | What to Evaluate | Measurement |
|---|---|---|
| **Decomposition** | Does the agent break the goal into reasonable sub-tasks? | LLM judge or human review |
| **Ordering** | Are sub-tasks in a logical order? | Check dependency ordering |
| **Completeness** | Does the plan cover all necessary steps? | Compare against reference plan |
| **Feasibility** | Are the planned actions achievable with available tools? | Validate tools exist for each step |
| **Adaptation** | Does the agent revise the plan when something goes wrong? | Check for plan updates after errors |

---

## Tool Use Evaluation

| Metric | Definition | How to Measure |
|---|---|---|
| **Tool Precision** | When a tool is called, is it the right tool? | correct_tool_calls / total_tool_calls |
| **Tool Recall** | When a tool should be called, does the agent call it? | tools_correctly_used / tools_that_should_be_used |
| **Argument Correctness** | Are tool arguments correct? | Validate against expected arguments |
| **Execution Success** | Does the tool call succeed? | success_count / total_calls |
| **Retry Quality** | When a tool call fails, does the agent retry sensibly? | Check retry behavior (new args? different tool?) |

> See [12. Tool Calling Evals →](../12_tool_calling_evals/README.md) for a deeper dive on evaluating tool use specifically.

---

## Self-Correction & Reflection

Self-correction is a critical agent capability. Evaluate it explicitly:

| Behavior | What to Check | Good Sign | Bad Sign |
|---|---|---|---|
| **Error detection** | Does the agent recognize when something went wrong? | "That result doesn't match expectations, let me try again" | Ignores error and continues |
| **Strategy change** | Does the agent try a different approach after failure? | Switches from API search to web search | Retries same failing action |
| **Backtracking** | Does the agent undo incorrect actions? | Reverts a bad file edit | Compounds errors |
| **Verification** | Does the agent verify its work before declaring done? | Runs tests after code changes | Submits without checking |
| **Over-correction** | Does the agent fix things that aren't broken? | N/A | Changes working code, adds unnecessary steps |

---

## Agent Benchmarks

| Benchmark | Domain | Task Type | Metric | Scale | Key Feature |
|---|---|---|---|---|---|
| **[SWE-Bench](https://www.swebench.com/)** | Coding | Fix real GitHub issues | % resolved | 2,294 tasks | Real-world software engineering |
| **[SWE-Bench Verified](https://www.swebench.com/)** | Coding | Fix validated GitHub issues | % resolved | 500 tasks | Human-verified, most reliable |
| **[WebArena](https://webarena.dev/)** | Web | Complete web tasks | Task success | 812 tasks | Real websites, realistic tasks |
| **[GAIA](https://gaia-benchmark.github.io/)** | General | Multi-step reasoning + tools | Accuracy | 466 tasks | Tests real-world assistant tasks |
| **[Τ-Bench](https://github.com/sierra-research/tau-bench)** | Customer support | Multi-turn tool-use dialogues | Task success | 200+ tasks | Tests policy-following + tool use |
| **[ToolBench](https://github.com/OpenBMB/ToolBench)** | Tool use | API selection + execution | Pass rate | 16k+ APIs | Large-scale tool use |
| **[AgentBench](https://github.com/THUDM/AgentBench)** | Multi-domain | Code, web, games, DB | Composite | 8 environments | Diverse agent capabilities |
| **[MINT](https://github.com/xingyaoww/mint-bench)** | Multi-turn | Tool-augmented interaction | Task success | 586 tasks | Multi-turn tool use |

---

## Evaluation Strategies

### Strategy 1: Outcome-Based Evaluation

Simplest approach — only check the final result.

```python
result = agent.run(task)
assert result.success == True
assert result.output == expected_output
```

**Pros**: Simple, objective, cheap
**Cons**: No visibility into how the agent got there

### Strategy 2: Trajectory + Outcome Evaluation

Evaluate both the process and the result.

```python
result = agent.run(task)
trajectory = agent.get_trajectory()

# Outcome
assert result.success == True

# Process
assert len(trajectory.steps) <= max_steps
assert trajectory.unnecessary_actions == 0
assert trajectory.loop_count == 0
assert all(step.tool_args_valid for step in trajectory.steps)
```

### Strategy 3: Checkpoint-Based Evaluation

Define intermediate milestones and award partial credit.

```
Task: "Fix the login bug and add a test"

Checkpoints:
☐ Found the correct file          (+20%)
☐ Identified the bug              (+20%)
☐ Fixed the bug correctly         (+30%)
☐ Added a test                    (+20%)
☐ All tests pass                  (+10%)
```

### Strategy 4: Comparative Agent Evaluation

Compare two agent implementations on the same task set.

| Task | Agent A Result | Agent A Steps | Agent B Result | Agent B Steps | Winner |
|---|---|---|---|---|---|
| Task 001 | ✅ Success | 6 steps | ✅ Success | 4 steps | B (more efficient) |
| Task 002 | ❌ Fail | 12 steps | ✅ Success | 8 steps | B |
| Task 003 | ✅ Success | 5 steps | ❌ Fail | 15 steps | A |

---

## Designing Agent Eval Suites

### Test Case Design

| Category | Purpose | Example |
|---|---|---|
| **Standard tasks** | Baseline capability | "Fetch weather for New York" |
| **Multi-step tasks** | Planning + execution | "Research competitors and create comparison table" |
| **Error recovery** | Self-correction ability | Task where first tool call will fail |
| **Ambiguous goals** | Clarification ability | "Make it better" (should ask for clarification) |
| **Resource-constrained** | Efficiency under limits | "Complete in ≤5 tool calls" |
| **Adversarial** | Robustness | Misleading tool outputs, contradictory information |

### Recommended Eval Matrix

| Dimension | Weight | Metric | Threshold |
|---|---|---|---|
| Task Completion | 40% | Pass rate | ≥ 80% |
| Tool Use Quality | 20% | Tool precision + argument accuracy | ≥ 85% |
| Efficiency | 15% | Steps vs optimal | ≤ 1.5x optimal |
| Self-Correction | 15% | Recovery rate | ≥ 60% |
| Safety | 10% | No unsafe actions | 100% |

---

## Common Failure Modes

| Failure Mode | Description | Detection | Mitigation |
|---|---|---|---|
| **Infinite loops** | Agent repeats the same action indefinitely | Step count monitoring, repetition detection | Set max steps, detect repetition |
| **Hallucinated tools** | Agent tries to use tools that don't exist | Validate tool names against registry | Provide clear tool documentation |
| **Wrong tool selection** | Uses search when it should use code execution | Compare selected vs optimal tools | Improve tool descriptions |
| **Argument hallucination** | Tool arguments are made up or malformed | Schema validation on arguments | Provide examples in tool docs |
| **Premature termination** | Agent declares "done" before completing the task | Check task completion criteria | Add verification step |
| **Catastrophic forgetting** | Agent forgets earlier context in long trajectories | Check consistency across steps | Summarize progress periodically |
| **Over-exploration** | Agent explores irrelevant paths | Track relevance of each action | Improve planning, add time budgets |

---

## Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| **Outcome-only evaluation** | Misses process quality and efficiency | Evaluate trajectories alongside outcomes |
| **Not testing error recovery** | Agent works on happy path but fails on errors | Include tasks designed to trigger recoverable errors |
| **Ignoring efficiency** | Agent gets right answers but wastes resources | Set step/cost budgets and track efficiency |
| **No trajectory logging** | Can't debug failures | Log full trajectories with reasoning |
| **Testing only simple tasks** | Overestimates agent capability | Include multi-step, multi-tool, and ambiguous tasks |
| **Deterministic expectations** | Penalizing valid alternative paths | Use checkpoint-based or outcome-based eval, not path-exact matching |

---

## Key Takeaways

| Principle | Details |
|---|---|
| **Evaluate outcome AND process** | A right answer via a terrible process is fragile and unreliable |
| **Trajectory quality matters** | Loops, wrong tools, and inefficiency are all agent failures |
| **Test self-correction explicitly** | Agent recovery from errors is a critical capability |
| **Use checkpoints for partial credit** | Not all agent tasks are binary pass/fail |
| **Log everything** | Full trajectory traces are essential for debugging |
| **Set efficiency budgets** | An agent that takes 50 steps for a 5-step task is too expensive |

---

Move to [11. Multi-Agent Evals →](../11_multi_agent_evals/README.md) to learn how to evaluate systems where multiple agents coordinate, delegate, and collaborate.
