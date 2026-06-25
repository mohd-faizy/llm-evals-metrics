# 04. Application Evals

> **Purpose** — This is one of the most important sections in the repository. Most teams don't ship a model — they ship a **product built on a model**. This section teaches you how to evaluate the application, not just the underlying LLM.

---

## Table of Contents

- [Why Application Evals Matter](#why-application-evals-matter)
- [Benchmarks vs Application Evals](#benchmarks-vs-application-evals)
- [Example Applications & Their Eval Dimensions](#example-applications--their-eval-dimensions)
- [What to Measure](#what-to-measure)
- [Designing Application Eval Suites](#designing-application-eval-suites)
- [Evaluation Rubric Design](#evaluation-rubric-design)
- [Multi-Dimensional Scoring](#multi-dimensional-scoring)
- [Common Application Eval Patterns](#common-application-eval-patterns)
- [Case Study: Customer Support Bot](#case-study-customer-support-bot)
- [Common Mistakes](#common-mistakes)
- [Key Takeaways](#key-takeaways)

---

## Why Application Evals Matter

A model that scores 90% on MMLU can still produce a terrible customer support bot if:
- It hallucinates refund policies
- It can't follow your company's tone guidelines
- It escalates too aggressively (or not aggressively enough)
- It leaks internal information

**Application evals test the full system** — prompts, retrieval, post-processing, guardrails — not just the model in isolation.

> 💡 If the application fails to solve the user problem, the model benchmark score is secondary.

---

## Benchmarks vs Application Evals

| Dimension | Benchmarks | Application Evals |
|---|---|---|
| **What they test** | Raw model capabilities | Product behavior end-to-end |
| **Data source** | Public, standardized | Private, task-specific |
| **Distribution** | Academic / synthetic | Your actual user inputs |
| **Metrics** | Accuracy, BLEU, pass@k | Task success, CSAT, resolution rate |
| **Who designs them** | Researchers | Your product/engineering team |
| **When to use** | Model selection | Before every deployment |
| **Contamination risk** | High (public data) | Low (private data) |

---

## Example Applications & Their Eval Dimensions

| Application | Primary Dimension | Secondary Dimensions | Critical Failure Mode |
|---|---|---|---|
| **Chatbot** | Helpfulness, conversation quality | Latency, tone, safety | Hallucinating information |
| **Customer Support** | Resolution accuracy | Escalation appropriateness, empathy, policy compliance | Giving incorrect refund info |
| **Research Assistant** | Factual correctness | Citation accuracy, completeness | Citing non-existent papers |
| **Coding Copilot** | Code correctness (pass@1) | Code quality, security, explanation clarity | Introducing security vulnerabilities |
| **Medical QA** | Clinical accuracy | Appropriate disclaimers, completeness | Hallucinated treatment advice |
| **Legal QA** | Legal accuracy | Citation of relevant statutes, jurisdiction awareness | Misquoting case law |
| **Content Moderation** | Classification accuracy (F1) | False positive rate, latency | Missing harmful content |
| **Translation** | Meaning preservation | Fluency, cultural appropriateness | Inverting the meaning |
| **Summarization** | Faithfulness to source | Conciseness, coverage | Adding information not in source |

---

## What to Measure

### Core Quality Dimensions

| Dimension | Definition | How to Measure | Example Question |
|---|---|---|---|
| **Task Success** | Did the system accomplish the user's goal? | Binary pass/fail or task-specific metric | "Did the bot resolve the support ticket?" |
| **Correctness** | Is the information factually accurate? | LLM judge + ground truth comparison | "Are all cited facts verifiable?" |
| **Completeness** | Does the response address all parts of the query? | Checklist-based scoring | "Did it cover all 5 requested topics?" |
| **Relevance** | Is the response on-topic and appropriate? | LLM judge rubric | "Does this answer the asked question?" |
| **Helpfulness** | Would a user find this useful? | Human rating or preference test | "Would you use this answer?" |
| **Faithfulness** | Is the output grounded in the provided context? | Claim-level verification | "Does every claim trace to a source?" |

### Operational Dimensions

| Dimension | Definition | How to Measure | Target Example |
|---|---|---|---|
| **Latency** | Time to first token / total response time | System metrics | P95 < 2s |
| **Cost** | Token usage per request | API billing data | < $0.05 per interaction |
| **Consistency** | Same input → similar output across runs | Multi-run variance | σ < 0.1 on quality scores |
| **Safety** | Absence of harmful, biased, or toxic output | Safety classifier + red-team test | 0 critical safety failures |

### User Experience Dimensions

| Dimension | Definition | How to Measure |
|---|---|---|
| **Tone & Style** | Matches brand voice and user expectations | LLM judge with style rubric |
| **Formatting** | Proper structure (headers, bullets, code blocks) | Rule-based checks + LLM judge |
| **Conciseness** | Appropriate length — not too short, not too verbose | Length ratio + LLM judge |
| **Follow-up handling** | Multi-turn coherence and context retention | Conversation-level eval |

---

## Designing Application Eval Suites

### Step-by-Step Process

```
1. Define success criteria     → "What does a good output look like?"
2. Enumerate failure modes     → "What can go wrong?"
3. Build test cases            → "What inputs exercise each failure mode?"
4. Design scoring rubric       → "How do we score each dimension?"
5. Create golden dataset       → "What are the known-correct outputs?"
6. Implement automated scoring → "Can we run this in CI/CD?"
7. Validate against humans     → "Do automated scores match human judgment?"
8. Monitor in production       → "Are real-world inputs covered?"
```

### Test Case Categories

| Category | Purpose | Volume | Update Frequency |
|---|---|---|---|
| **Core regression set** | Ensure stable performance on known-good cases | 50–200 cases | Rarely (stable anchor) |
| **Failure-mode tests** | Target specific known weaknesses | 20–50 per mode | When new failures discovered |
| **Edge cases** | Test boundary conditions | 20–50 cases | Quarterly review |
| **Adversarial inputs** | Test robustness against attacks | 20–50 cases | Regular red-teaming cycles |
| **Production samples** | Reflect real-world distribution | 50–100 cases | Monthly refresh |
| **Slice-specific tests** | Test performance on subgroups | 10–30 per slice | As new segments emerge |

---

## Evaluation Rubric Design

A good rubric is the difference between noisy and reliable eval scores.

### Rubric Design Principles

1. **Be specific** — "The response is helpful" is too vague. "The response answers the user's question and provides actionable next steps" is better.
2. **Use anchored scales** — Define what each score level means with examples.
3. **Separate dimensions** — Don't conflate correctness and style in one score.
4. **Include failure examples** — Show what a 1/5 looks like, not just 5/5.

### Example: 5-Point Rubric for Customer Support

| Score | Label | Description |
|---|---|---|
| **5** | Excellent | Correctly resolves the issue, follows all policies, appropriate tone, offers proactive help |
| **4** | Good | Correctly resolves the issue, follows policies, minor tone/style improvements possible |
| **3** | Acceptable | Mostly correct but misses nuance, or correct but poor tone/formatting |
| **2** | Poor | Partially correct but includes errors, or misses key information |
| **1** | Failure | Incorrect information, policy violation, harmful content, or complete non-answer |

---

## Multi-Dimensional Scoring

For complex applications, use a **scorecard** that combines multiple dimensions:

### Example Scorecard: Research Assistant

| Dimension | Weight | Metric | Threshold | Scoring Method |
|---|---|---|---|---|
| Factual Correctness | 30% | Claim verification score | ≥ 0.90 | LLM judge |
| Completeness | 25% | Coverage of key points | ≥ 0.80 | Checklist |
| Citation Accuracy | 20% | % of claims with valid citations | ≥ 0.85 | Rule-based + LLM |
| Helpfulness | 15% | User rating (1–5) | ≥ 4.0 | Human / LLM judge |
| Conciseness | 10% | Length appropriateness (1–5) | ≥ 3.0 | LLM judge |

**Composite Score** = Σ (dimension_score × weight)

**Deployment Gate**: All thresholds must be met AND composite score ≥ 0.85

---

## Common Application Eval Patterns

### Pattern 1: Assertion-Based Testing

Test specific properties of the output with boolean assertions:

```python
# Pseudocode
assert "refund" in response.lower()           # Contains key term
assert len(response) < 500                     # Not too verbose
assert not contains_pii(response)              # No PII leakage
assert sentiment(response) > 0.3               # Not negative tone
assert factcheck(response, context) > 0.9      # Factually grounded
```

### Pattern 2: Golden Output Comparison

Compare against a known-good reference output:

```python
# Pseudocode
similarity = bert_score(response, golden_output)
assert similarity > 0.85
```

### Pattern 3: LLM-Judge Rubric Scoring

Use an LLM to score against a detailed rubric:

```python
# Pseudocode
score = llm_judge(
    input=user_query,
    output=response,
    rubric=rubric_text,
    scale="1-5"
)
assert score >= 4
```

### Pattern 4: Comparative (A/B) Testing

Compare two system variants head-to-head:

```python
# Pseudocode
winner = pairwise_judge(
    input=user_query,
    output_a=response_v1,
    output_b=response_v2,
    criteria="helpfulness"
)
# Track win rates across the eval set
```

---

## Case Study: Customer Support Bot

### Context
A customer support bot for an e-commerce platform handling returns, order status, and product questions.

### Eval Design

| Layer | What We Test | Method | Cases |
|---|---|---|---|
| **Correctness** | Policy-accurate responses | Compare against policy doc + LLM judge | 100 policy scenarios |
| **Tone** | Empathetic, professional | LLM judge with tone rubric | 50 emotional scenarios |
| **Escalation** | Knows when to hand off to human | Rule-based classification | 30 escalation triggers |
| **Safety** | No inappropriate content, no PII leakage | Safety classifier + red team | 40 adversarial inputs |
| **Resolution** | End-to-end task completion | Multi-turn simulation + LLM judge | 50 full conversations |

### Results Dashboard (Example)

```
╔══════════════════════════════════════════════════════╗
║  Customer Support Bot — Eval Report v2.3            ║
╠══════════════════════════════════════════════════════╣
║  Correctness:    92% (target: 90%) ✅               ║
║  Tone:           4.3/5 (target: 4.0) ✅             ║
║  Escalation:     88% (target: 85%) ✅               ║
║  Safety:         100% (target: 100%) ✅             ║
║  Resolution:     79% (target: 80%) ❌               ║
║                                                      ║
║  DECISION: Fix resolution regression before deploy   ║
╚══════════════════════════════════════════════════════╝
```

---

## Common Mistakes

| Mistake | Why It's a Problem | What to Do Instead |
|---|---|---|
| **Testing the model instead of the application** | Your prompts, RAG, and guardrails are part of the system | Test the full pipeline end-to-end |
| **Using only happy-path test cases** | Misses failures on edge cases and adversarial inputs | Include failure-mode and adversarial test cases |
| **Single-dimension scoring** | A "correct" response can still be unhelpful or unsafe | Use multi-dimensional scorecards |
| **No production feedback loop** | Eval set becomes stale and misses real-world failures | Sample production traces into eval sets |
| **Evaluating single turns in a multi-turn system** | Misses context retention and coherence issues | Evaluate full conversations |
| **Skipping cost/latency measurement** | A perfect-quality response that takes 30s is unusable | Include operational metrics |

---

## Key Takeaways

| Principle | Details |
|---|---|
| **Test the product, not the model** | Benchmarks test capability; application evals test fitness for your use case |
| **Multi-dimensional scoring** | One number can't capture quality, safety, cost, and latency |
| **Start with failure modes** | Design test cases around what can go wrong, not just what should go right |
| **Production feedback loop** | Continuously feed real failures back into your eval dataset |
| **Gate deployments** | Every eval dimension should have a threshold; don't deploy if any fails |

---

Move to [05. Dataset Engineering →](../05_dataset_engineering/README.md) to learn how to build the datasets that power your eval suites.
