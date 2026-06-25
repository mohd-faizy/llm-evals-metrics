# 18. Coding Agent Evals

> **Purpose** — Evaluate code generation and code editing systems: from single-function completion to repository-level autonomous coding agents. Coding evals are uniquely advantageous because **code can be objectively tested** — if the tests pass, the code works. This section covers benchmarks, metrics, evaluation strategies, and the specific challenges of evaluating agentic coding systems.

---

## Table of Contents

- [Why Coding Evals Are Special](#why-coding-evals-are-special)
- [Coding Eval Spectrum](#coding-eval-spectrum)
- [Metrics for Code Generation](#metrics-for-code-generation)
- [Function-Level Evaluation](#function-level-evaluation)
- [Repository-Level Evaluation](#repository-level-evaluation)
- [Coding Agent Evaluation](#coding-agent-evaluation)
- [Benchmarks](#benchmarks)
- [Code Quality Beyond Correctness](#code-quality-beyond-correctness)
- [Building Coding Eval Datasets](#building-coding-eval-datasets)
- [Evaluation Strategies](#evaluation-strategies)
- [Common Failure Modes](#common-failure-modes)
- [Common Mistakes](#common-mistakes)
- [Key Takeaways](#key-takeaways)

---

## Why Coding Evals Are Special

Coding evals have a unique advantage over other LLM evals:

| Advantage | Details |
|---|---|
| **Objective verification** | Code either passes tests or it doesn't — less subjective than text quality |
| **Automated testing** | Test suites provide free, scalable evaluation |
| **Functional correctness** | The output is verifiable by execution, not just similarity |
| **Rich failure signals** | Stack traces, test failures, and diffs pinpoint exactly what went wrong |

But they also have unique challenges:

| Challenge | Details |
|---|---|
| **Multiple valid solutions** | Many correct implementations for the same problem |
| **Beyond correctness** | Code can be correct but insecure, unreadable, or inefficient |
| **Repository context** | Real-world coding requires understanding entire codebases |
| **Environment dependencies** | Tests may need specific packages, databases, or services |

---

## Coding Eval Spectrum

![Coding Eval Spectrum](../assets/18_code_ag_evl.png)

---

## Metrics for Code Generation

### Core Metrics

| Metric | Definition | Formula | Range | Best For |
|---|---|---|---|---|
| **pass@1** | Probability of correct solution on first attempt | correct_solutions / total_attempts (1 sample) | 0–1 | Primary coding metric |
| **pass@k** | Probability that at least 1 of k samples is correct | 1 - C(n-c, k) / C(n, k) | 0–1 | Model capability ceiling |
| **% Resolved** | Percentage of tasks fully resolved | resolved_tasks / total_tasks | 0–1 | Repository-level tasks |
| **Exact Match** | Generated code matches reference exactly | Binary | 0 or 1 | Simple completion |
| **Test Pass Rate** | % of test cases that pass | passed_tests / total_tests | 0–1 | Partial credit |

### Why pass@1 Matters Most

```
pass@1:   Model gets it right on the first try (what users experience)
pass@10:  Model gets it right at least once in 10 tries (capability ceiling)
pass@100: Model can do it if you give it enough tries (inflated metric)
```

> Always report **pass@1** as the primary metric. pass@k with high k is misleading for production readiness.

### Beyond-Correctness Metrics

| Metric | What It Measures | How to Evaluate |
|---|---|---|
| **Code quality** | Readability, maintainability, style | Linter score, LLM judge |
| **Security** | Absence of vulnerabilities | SAST tools (Bandit, Semgrep) |
| **Efficiency** | Time/space complexity | Performance benchmarking |
| **Test quality** | Coverage and thoroughness of generated tests | Coverage tools, mutation testing |
| **Minimal diff** | Smallest change needed to fix the issue | Diff size analysis |
| **Idiomatic style** | Follows language conventions | Style checker + LLM judge |

---

## Function-Level Evaluation

### Benchmarks

| Benchmark | Language | Problems | Metric | Difficulty | Notes |
|---|---|---|---|---|---|
| **HumanEval** | Python | 164 | pass@k | Easy-Medium | Original coding benchmark |
| **HumanEval+** | Python | 164 | pass@k | Easy-Medium | Stricter tests (80× more test cases) |
| **MBPP** | Python | 974 | pass@k | Easy | Short Python programs |
| **MBPP+** | Python | 974 | pass@k | Easy | Enhanced test cases |
| **MultiPL-E** | 18 languages | 164×18 | pass@k | Easy-Medium | HumanEval translated to 18 languages |
| **CodeContests** | Multiple | 165 | pass@k | Hard | Competition-level problems |

### Function-Level Eval Process

```
1. Provide function signature + docstring
2. Model generates function body
3. Execute against test suite
4. Score: pass@1, pass@10
5. Optionally: evaluate code quality, security
```

### Example

```python
# Prompt (given to model):
def has_close_elements(numbers: list[float], threshold: float) -> bool:
    """Check if in given list of numbers, are any two numbers
    closer to each other than given threshold.
    >>> has_close_elements([1.0, 2.0, 3.0], 0.5)
    False
    >>> has_close_elements([1.0, 2.8, 3.0, 4.0, 5.0, 2.0], 0.3)
    True
    """

# Model generates:
    for i in range(len(numbers)):
        for j in range(i + 1, len(numbers)):
            if abs(numbers[i] - numbers[j]) < threshold:
                return True
    return False

# Evaluation: Run against hidden test cases → pass@1 = 1 (all tests pass)
```

---

## Repository-Level Evaluation

### What Changes at Repository Scale

| Dimension | Function-Level | Repository-Level |
|---|---|---|
| **Context** | Function signature + docstring | Entire codebase (thousands of files) |
| **Output** | Single function body | Multi-file patch (additions, deletions, modifications) |
| **Dependencies** | None/minimal | Complex import chains, build systems |
| **Testing** | Provided test suite | Must understand existing test infrastructure |
| **Navigation** | N/A | Must find relevant files in large repos |
| **Understanding** | Understand one function | Understand architecture, patterns, conventions |

### Repository-Level Metrics

| Metric | Definition | How to Measure |
|---|---|---|
| **% Resolved** | Task fully completed (all tests pass) | Run test suite |
| **Localization accuracy** | Found the right files to modify | Compare modified files vs ground truth |
| **Patch correctness** | Applied changes are correct | Test suite + diff analysis |
| **Minimal patch** | Changed only what's necessary | Diff size vs reference solution |
| **Build success** | Code compiles/builds after changes | Build system |
| **No regressions** | Existing tests still pass | Full test suite |

---

## Coding Agent Evaluation

Coding agents combine LLM capabilities with tool use (file search, code execution, testing) in an agentic loop.

### Agent Capabilities to Evaluate

| Capability | What to Test | How to Test |
|---|---|---|
| **Bug localization** | Can the agent find the buggy code? | Provide bug report, check if agent finds right file/line |
| **Root cause analysis** | Does the agent understand WHY it's broken? | Evaluate reasoning in agent traces |
| **Patch generation** | Does the fix actually work? | Run test suite |
| **Test generation** | Can the agent write tests for the fix? | Run generated tests, check coverage |
| **Multi-file editing** | Can the agent coordinate changes across files? | Tasks requiring 2+ file changes |
| **Dependency understanding** | Does the agent understand imports and dependencies? | Tasks involving dependency changes |
| **Iteration** | Does the agent fix issues when tests fail? | Observe retry behavior |

### Agent vs Non-Agent Comparison

| Approach | How It Works | Strengths | Weaknesses |
|---|---|---|---|
| **One-shot generation** | Single prompt → single response | Fast, cheap | No self-correction, limited context |
| **RAG-augmented** | Retrieve relevant code → generate | Better context | Limited to retrieved code |
| **Agentic** | Plan → Search → Edit → Test → Iterate | Self-correcting, full-repo context | Slow, expensive, may loop |

---

## Benchmarks

### Benchmark Comparison

| Benchmark | Scope | Tasks | Metric | Difficulty | Key Feature |
|---|---|---|---|---|---|
| **[HumanEval](https://github.com/openai/human-eval)** | Function | 164 | pass@k | Easy-Medium | Original, widely used |
| **[MBPP](https://github.com/google-research/google-research/tree/master/mbpp)** | Function | 974 | pass@k | Easy | Larger, simpler problems |
| **[SWE-Bench](https://www.swebench.com/)** | Repository | 2,294 | % Resolved | Hard | Real GitHub issues → patches |
| **[SWE-Bench Verified](https://www.swebench.com/)** | Repository | 500 | % Resolved | Hard | Human-validated subset, most reliable |
| **[SWE-Bench Lite](https://www.swebench.com/)** | Repository | 300 | % Resolved | Medium-Hard | Curated for faster evaluation |
| **[RepoBench](https://github.com/Leolty/repobench)** | Repository | Multi | Completion accuracy | Medium | Cross-file code completion |
| **[ClassEval](https://github.com/FudanSELab/ClassEval)** | Class | 100 | pass@k | Medium | Class-level generation |
| **[CodeContests](https://github.com/google-deepmind/code_contests)** | Algorithm | 165 | pass@k | Very Hard | Competition programming |
| **[LiveCodeBench](https://livecodebench.github.io/)** | Function | Rolling | pass@k | Medium-Hard | Contamination-free, refreshed |

### SWE-Bench Deep Dive

SWE-Bench is the **gold standard** for evaluating coding agents.

| Aspect | Details |
|---|---|
| **Task format** | Real GitHub issue → agent must produce a patch that fixes it |
| **Repositories** | 12 popular Python repos (Django, scikit-learn, matplotlib, etc.) |
| **Difficulty** | Requires reading issue, navigating repo, understanding codebase, writing patch |
| **Verification** | Pass/fail based on the project's existing test suite |
| **Key variant** | **SWE-Bench Verified** (500 tasks) — human-validated, most reliable |

### SWE-Bench Scoring

```
% Resolved = tasks_where_all_tests_pass / total_tasks × 100

Top agent scores (mid-2025):
├── >50% on Verified: Frontier agentic systems
├── 30-50% on Verified: Strong coding agents
├── 10-30% on Verified: Basic coding agents
└── <10%: Non-agentic or weak systems
```

---

## Code Quality Beyond Correctness

### Quality Dimensions

| Dimension | What to Measure | Tools |
|---|---|---|
| **Readability** | Is the code easy to understand? | LLM judge, complexity metrics |
| **Security** | Are there vulnerabilities? | Bandit, Semgrep, CodeQL |
| **Performance** | Is the code efficient? | Profiling, complexity analysis |
| **Style consistency** | Does it match the codebase's style? | Linters (ESLint, Black, Ruff) |
| **Test coverage** | How well are changes tested? | Coverage.py, Istanbul |
| **Documentation** | Are functions/classes documented? | Doc coverage tools |
| **Error handling** | Does it handle edge cases? | Manual review, fuzz testing |

### Security Eval Checklist

```
☐ No hardcoded secrets or credentials
☐ No SQL injection vulnerabilities
☐ No command injection
☐ No path traversal
☐ Input validation present
☐ No use of known-vulnerable functions
☐ Dependencies are not vulnerable (no known CVEs)
```

---

## Building Coding Eval Datasets

### Dataset Sources

| Source | Pros | Cons |
|---|---|---|
| **Real GitHub issues** | Realistic, diverse | Requires test suites, complex setup |
| **Competitive programming** | Clear specifications, test cases | Not representative of real-world coding |
| **Internal codebase issues** | Matches your actual use case | Not shareable, limited scale |
| **Synthetic generation** | Scalable, customizable | May not reflect real complexity |

### Custom Eval Set Construction

```
1. Select representative tasks from your codebase
2. Ensure each task has a clear specification
3. Write or verify test cases (aim for >90% coverage of the fix)
4. Record the ground truth solution
5. Categorize by difficulty, domain, and required skills
6. Version control the entire dataset
```

### Difficulty Levels

| Level | Description | Example | Typical pass@1 |
|---|---|---|---|
| **Easy** | Single function, clear specification | Implement `is_palindrome()` | 80–95% |
| **Medium** | Multi-function, requires context | Add caching to a service class | 50–80% |
| **Hard** | Multi-file, requires architectural understanding | Fix a race condition in a distributed system | 20–50% |
| **Expert** | Repository-level, ambiguous specification | Refactor authentication to support OAuth2 | 5–30% |

---

## Evaluation Strategies

### Strategy 1: Test-Based Verification

Run generated code against test suites.

```python
# Pseudocode
generated_code = model.generate(prompt)
write_file("solution.py", generated_code)
result = run_tests("test_solution.py")
score = result.passed / result.total  # pass rate
```

### Strategy 2: Multi-Dimensional Scoring

Score correctness + quality + security.

| Dimension | Weight | Method | Threshold |
|---|---|---|---|
| Test pass rate | 40% | Test execution | ≥ 100% (all tests pass) |
| Code quality | 20% | Linter + LLM judge | ≥ 7/10 |
| Security | 20% | SAST scan | 0 critical/high findings |
| Minimal diff | 10% | Diff size analysis | ≤ 1.5× reference |
| Documentation | 10% | Doc coverage check | All public APIs documented |

### Strategy 3: Trajectory Evaluation (for Agents)

Evaluate the agent's process, not just the final patch.

| Aspect | What to Evaluate |
|---|---|
| **File localization** | Did the agent find the right files to modify? |
| **Exploration efficiency** | How many files did it read before finding the issue? |
| **Failed attempts** | Did it try incorrect fixes before the correct one? |
| **Self-correction** | When tests failed, did it iterate effectively? |
| **Total cost** | How many tokens / API calls did it use? |

---

## Common Failure Modes

| Failure Mode | Description | Frequency | Impact |
|---|---|---|---|
| **Correct but fragile** | Code works for test cases but fails on edge cases | Common | Bugs in production |
| **Over-engineering** | Unnecessarily complex solution | Common | Hard to maintain |
| **Import errors** | Missing or wrong imports | Common | Code doesn't run |
| **Type errors** | Wrong types, especially in typed languages | Common | Runtime failures |
| **API misuse** | Using deprecated or non-existent API methods | Moderate | Runtime failures |
| **Incomplete fix** | Fixes the symptom but not the root cause | Moderate | Bug recurrence |
| **Regression** | Fixes one thing, breaks another | Moderate | New bugs |
| **Hallucinated APIs** | Uses functions/methods that don't exist | Common | Import/attribute errors |
| **Security vulnerabilities** | Introduces SQL injection, XSS, etc. | Rare but critical | Security breach |

---

## Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| **Only measuring pass@1 on HumanEval** | Overestimates real-world capability | Include SWE-Bench and repo-level tasks |
| **Ignoring code quality** | Correct but unmaintainable code | Add quality, security, and style metrics |
| **Not running tests in isolation** | Flaky tests contaminate results | Run in clean environments |
| **Using pass@100** | Inflates capability perception | Report pass@1 as primary metric |
| **No security scanning** | Vulnerabilities go undetected | Run SAST on generated code |
| **Only testing Python** | Many production systems use other languages | Include multi-language evaluation |
| **Static benchmarks only** | Contamination concerns | Use LiveCodeBench or custom eval sets |

---

## Key Takeaways

| Principle | Details |
|---|---|
| **Code can be objectively tested** | Coding evals have a huge advantage: test suites provide ground truth |
| **pass@1 is the metric that matters** | It's what users actually experience |
| **SWE-Bench Verified is the gold standard** | For evaluating coding agents on real-world tasks |
| **Correctness is necessary but not sufficient** | Also evaluate security, quality, and maintainability |
| **Repository context changes everything** | Function-level benchmarks don't predict repo-level performance |
| **Test with real-world complexity** | Competitive programming ≠ production software engineering |

---

← [17. Multimodal Evals](../17_multimodal_evals/README.md) | [Back to Index →](../README.md)
