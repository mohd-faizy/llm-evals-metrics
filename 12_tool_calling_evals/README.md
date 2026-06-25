# 12. Tool Calling Evals

> **Purpose** — Evaluate how well LLMs select and invoke tools (functions, APIs, external services). Tool calling is the foundation of agentic systems, and errors here — wrong function, bad arguments, missed invocations — cascade into downstream failures. This section covers how to measure tool precision, argument accuracy, execution success, and error recovery.

---

## Table of Contents

- [What Is Tool Calling?](#what-is-tool-calling)
- [Why Tool Calling Evals Matter](#why-tool-calling-evals-matter)
- [What to Evaluate](#what-to-evaluate)
- [Tool Calling Metrics](#tool-calling-metrics)
- [Evaluation Dimensions](#evaluation-dimensions)
- [Building Tool Calling Eval Datasets](#building-tool-calling-eval-datasets)
- [Common Failure Modes](#common-failure-modes)
- [Evaluation Strategies](#evaluation-strategies)
- [Multi-Tool Scenarios](#multi-tool-scenarios)
- [Benchmarks & Resources](#benchmarks--resources)
- [Common Mistakes](#common-mistakes)
- [Key Takeaways](#key-takeaways)

---

## What Is Tool Calling?

Tool calling (also called function calling) is when an LLM generates structured requests to invoke external functions or APIs instead of (or alongside) natural language text.

```
User: "What's the weather in Tokyo?"

LLM Decision:
├── Tool Selected:  get_weather
├── Arguments:      {"city": "Tokyo", "units": "celsius"}
└── Raw output:     Not generated (tool call instead)

Execution:
├── get_weather(city="Tokyo", units="celsius")
└── Returns: {"temp": 28, "condition": "partly_cloudy"}

LLM Final Response: "It's 28°C and partly cloudy in Tokyo."
```

---

## Why Tool Calling Evals Matter

| Problem | Impact |
|---|---|
| Wrong tool selected | Agent takes incorrect action (e.g., deletes instead of reads) |
| Wrong arguments | Tool call fails or returns wrong data |
| Missing tool call | Agent tries to answer from knowledge instead of looking it up |
| Unnecessary tool call | Wastes time and money on unnecessary API calls |
| Malformed output | Tool call parsing fails, breaking the pipeline |

> Tool calling errors are the **#1 source of agent failures** in production systems.

---

## What to Evaluate

### The Five Stages of Tool Calling

```
1. RECOGNITION   — Does the model recognize that a tool is needed?
2. SELECTION      — Does it choose the correct tool?
3. PARAMETERIZATION — Are the arguments correct and well-formed?
4. EXECUTION      — Does the tool call succeed?
5. INTEGRATION    — Does the model use the tool result correctly?
```

---

## Tool Calling Metrics

### Core Metrics

| Metric | Definition | Formula | Range |
|---|---|---|---|
| **Tool Call Detection** | Does the model call a tool when it should? | correct_detections / required_tool_calls | 0–1 |
| **Tool Precision** | When a tool is called, is it the right one? | correct_tool_selections / total_tool_calls | 0–1 |
| **Tool Recall** | When a tool should be called, is it called? | called_correctly / should_have_been_called | 0–1 |
| **Tool F1** | Harmonic mean of precision and recall | 2 × (P × R) / (P + R) | 0–1 |
| **Argument Accuracy** | Are all arguments correct? | correct_args / total_args | 0–1 |
| **Argument Completeness** | Are all required arguments provided? | provided_required / total_required | 0–1 |
| **Execution Success Rate** | % of tool calls that execute without error | successful / total | 0–1 |
| **End-to-End Success** | Does the task succeed (including tool result integration)? | successful_tasks / total_tasks | 0–1 |

### Advanced Metrics

| Metric | Definition | Why It Matters |
|---|---|---|
| **False positive rate** | Tool called when not needed | Wastes resources, adds latency |
| **Retry quality** | When a tool call fails, does the retry fix the issue? | Measures error recovery |
| **Schema adherence** | Does the generated call match the tool schema exactly? | Prevents parsing errors |
| **Ordering accuracy** | In multi-tool calls, correct execution order? | Prevents dependency errors |
| **Parallel call efficiency** | Independent tools called in parallel vs sequentially? | Affects latency |

---

## Evaluation Dimensions

### Dimension 1: Tool Selection

| Test Type | What It Tests | Example |
|---|---|---|
| **Single tool available** | Does the model use the only available tool correctly? | 1 tool, 10 queries |
| **Tool disambiguation** | Can it choose between similar tools? | `search_web` vs `search_internal_docs` |
| **No-tool-needed** | Does it avoid calling tools when unnecessary? | "What is 2+2?" (no tool needed) |
| **Multi-tool selection** | Does it choose the right combination? | "Find weather AND book a restaurant" |

### Dimension 2: Argument Quality

| Test Type | What It Tests | Example |
|---|---|---|
| **Required arguments** | Are all mandatory args provided? | `search(query=...)` — query is required |
| **Optional arguments** | Are optional args used correctly? | `search(query=..., limit=5)` |
| **Type correctness** | Are arg types correct (string, int, enum)? | `limit="five"` (should be `5`) |
| **Value accuracy** | Are values correct and relevant? | `city="New York"` not `city="new york city ny"` |
| **Complex arguments** | Handles nested objects, arrays? | `filter={"status": "active", "tags": ["urgent"]}` |
| **Edge cases** | Handles empty, null, special characters? | Empty string, Unicode, very long values |

### Dimension 3: Error Recovery

| Scenario | What to Evaluate |
|---|---|
| **Tool returns error** | Does the model understand the error and retry with fixed args? |
| **Tool not found** | Does the model fall back gracefully? |
| **Rate limited** | Does the model wait and retry? |
| **Partial result** | Does the model recognize incomplete data and handle appropriately? |
| **Contradictory result** | Does the model handle unexpected tool output? |

---

## Building Tool Calling Eval Datasets

### Dataset Schema

```json
{
  "id": "tc-001",
  "user_query": "What's the weather in Tokyo?",
  "available_tools": [
    {
      "name": "get_weather",
      "description": "Get current weather for a city",
      "parameters": {
        "city": {"type": "string", "required": true},
        "units": {"type": "string", "enum": ["celsius", "fahrenheit"], "default": "celsius"}
      }
    }
  ],
  "expected_tool_calls": [
    {
      "name": "get_weather",
      "arguments": {"city": "Tokyo", "units": "celsius"}
    }
  ],
  "expected_no_tool_call": false,
  "category": "single_tool",
  "difficulty": "easy"
}
```

### Test Case Categories

| Category | Cases to Include | Volume |
|---|---|---|
| **Single tool, clear intent** | Straightforward tool invocations | 50+ |
| **Tool disambiguation** | Similar tools, model must pick the right one | 30+ |
| **No tool needed** | Questions answerable without tools | 20+ |
| **Multi-tool required** | Tasks requiring multiple tool calls | 30+ |
| **Complex arguments** | Nested objects, arrays, enums | 20+ |
| **Error recovery** | Tool returns errors, model must adapt | 20+ |
| **Edge cases** | Empty input, special chars, very long args | 15+ |

---

## Common Failure Modes

| Failure Mode | Description | Example | Frequency |
|---|---|---|---|
| **Hallucinated tool** | Calls a tool that doesn't exist | `call: search_google(...)` when only `web_search` exists | Common |
| **Wrong tool** | Selects an incorrect but real tool | Uses `send_email` instead of `create_draft` | Common |
| **Missing argument** | Omits a required parameter | `get_weather()` without city | Common |
| **Wrong argument type** | Provides wrong data type | `limit="five"` instead of `limit=5` | Moderate |
| **Hallucinated argument** | Adds parameters that don't exist | `get_weather(city="Tokyo", detailed=true)` when `detailed` isn't a parameter | Moderate |
| **Unnecessary tool call** | Calls a tool when answer is obvious | `calculator(2+2)` | Moderate |
| **Missing tool call** | Answers from training data instead of using tool | "The weather in Tokyo is usually warm" (no actual lookup) | Common |
| **Schema violation** | Output doesn't match expected JSON structure | Missing closing brace, wrong nesting | Moderate |
| **Argument leakage** | Puts user PII into tool arguments | Sends full conversation as search query | Rare but critical |

---

## Evaluation Strategies

### Strategy 1: Exact Match Evaluation

Compare generated tool calls against expected tool calls exactly.

```python
# Pseudocode
assert generated.tool_name == expected.tool_name
assert generated.arguments == expected.arguments
```

**Pros**: Simple, deterministic
**Cons**: Penalizes valid alternatives (e.g., "New York" vs "New York City")

### Strategy 2: Semantic Argument Matching

Allow semantically equivalent arguments.

```python
# Pseudocode
assert generated.tool_name == expected.tool_name
assert semantic_similarity(generated.arguments["city"], expected.arguments["city"]) > 0.9
```

**Pros**: More forgiving of valid variations
**Cons**: Harder to implement, may allow errors

### Strategy 3: Execution-Based Evaluation

Run the generated tool call and check if the task succeeds.

```python
# Pseudocode
result = execute_tool_call(generated.tool_name, generated.arguments)
assert task_success(result, expected_outcome)
```

**Pros**: Tests real-world success, most realistic
**Cons**: Requires tool execution infrastructure, non-deterministic

### Strategy 4: Multi-Level Scoring

Score each aspect separately.

| Aspect | Score | Weight |
|---|---|---|
| Tool name correct? | 1/1 | 30% |
| All required args present? | 1/1 | 25% |
| Arg values correct? | 2/3 | 25% |
| Schema valid? | 1/1 | 10% |
| No extra hallucinated args? | 1/1 | 10% |
| **Weighted total** | **0.88** | |

---

## Multi-Tool Scenarios

### Types of Multi-Tool Calls

| Type | Description | Example | Eval Focus |
|---|---|---|---|
| **Sequential** | Tool B depends on Tool A's result | Search → then fetch details | Ordering, dependency handling |
| **Parallel** | Tools are independent | Get weather + get news | Parallel detection, efficiency |
| **Conditional** | Tool B only called based on Tool A's result | Check inventory → if in stock, place order | Conditional logic |
| **Iterative** | Same tool called multiple times with different args | Search with progressively refined queries | Refinement quality |

### Multi-Tool Metrics

| Metric | Definition |
|---|---|
| **Call set accuracy** | Are exactly the right tools called (no missing, no extra)? |
| **Ordering correctness** | For sequential calls, is the order correct? |
| **Parallel detection** | Does the model call independent tools in parallel? |
| **Dependency handling** | Does Tool B correctly use Tool A's output? |

---

## Benchmarks & Resources

| Benchmark | Focus | Scale | Key Feature |
|---|---|---|---|
| **[Berkeley Function Calling Leaderboard (BFCL)](https://gorilla.cs.berkeley.edu/leaderboard.html)** | Function calling accuracy | 2,000+ test cases | Multiple languages, complex schemas |
| **[ToolBench](https://github.com/OpenBMB/ToolBench)** | API tool use | 16,000+ APIs | Real-world APIs |
| **[API-Bank](https://github.com/AlibabaResearch/DAMO-ConvAI/tree/main/api-bank)** | API call planning | 314 APIs | Multi-step tool planning |
| **[Nexus Raven](https://github.com/nexusflowai/NexusRavenV2)** | Function calling | 1,000+ cases | Open-source function calling model |
| **[Τ-Bench](https://github.com/sierra-research/tau-bench)** | Tool-use in dialogues | 200+ tasks | Policy-following + tool use |

---

## Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| **Only testing happy path** | Misses error recovery failures | Include error scenarios in eval set |
| **Ignoring false positives** | Agent calls tools unnecessarily, wasting resources | Include "no tool needed" test cases |
| **Exact string match only** | Penalizes valid argument variations | Use semantic matching for free-text args |
| **Not testing multi-tool** | Misses ordering and dependency issues | Include sequential and parallel tool scenarios |
| **Ignoring schema validation** | Malformed calls crash in production | Validate against tool schema before scoring |
| **Not testing with many tools** | Performance degrades with large tool registries | Test with 5, 20, 50+ available tools |

---

## Key Takeaways

| Principle | Details |
|---|---|
| **Tool calling is the foundation of agency** | If tool calling fails, the entire agent fails |
| **Evaluate all five stages** | Recognition, selection, parameterization, execution, integration |
| **Include negative test cases** | Test "no tool needed" and "wrong tool" scenarios |
| **Test error recovery** | What happens when tool calls fail matters as much as when they succeed |
| **Score multi-dimensionally** | Tool name, argument presence, argument correctness, schema validity |
| **Scale testing** | Performance often degrades as the number of available tools grows |

---

Move to [13. Memory Evals →](../13_memory_evals/README.md) to learn how to evaluate memory systems in agents, assistants, and personalized applications.
