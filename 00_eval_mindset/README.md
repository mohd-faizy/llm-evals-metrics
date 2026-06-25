# 00. Evaluation Mindset

> **Purpose** — Establish the philosophy and mental models that guide every evaluation decision throughout this repository. Before choosing tools or metrics, you need to know *what you're protecting against* and *what quality means for your users*.

---

## Table of Contents

- [Why This Section Comes First](#why-this-section-comes-first)
- [Core Philosophy](#core-philosophy)
- [The Four Foundational Questions](#the-four-foundational-questions)
- [Failure Mode Thinking](#failure-mode-thinking)
- [Human vs Automated Evaluation](#human-vs-automated-evaluation)
- [Offline vs Online Evaluation](#offline-vs-online-evaluation)
- [Pointwise vs Pairwise Evaluation](#pointwise-vs-pairwise-evaluation)
- [LLM-as-a-Judge: Tool, Not Truth](#llm-as-a-judge-tool-not-truth)
- [Building Your Evaluation Philosophy](#building-your-evaluation-philosophy)
- [Key Takeaways](#key-takeaways)

---

## Why This Section Comes First

Most evaluation failures are not technical — they are **philosophical**. Teams pick a metric before defining what "good" means. They benchmark a model before identifying the failures that matter. They automate scoring before checking whether the scores correlate with user outcomes.

This section ensures you start with the right questions, not the right tools.

---

## Core Philosophy

Modern LLM evaluation is not just about scoring text output. It is about:

1. **Identifying user-facing failure modes** that matter in production
2. **Building a repeatable system** to detect those failures
3. **Making risk-aware tradeoff decisions** between quality, latency, cost, and safety

> ⚠️ An eval system that measures the wrong thing precisely is worse than no eval system at all — it creates false confidence.

---

## The Four Foundational Questions

Before writing a single eval, answer these questions for your application:

| # | Question | What It Determines |
|---|---|---|
| 1 | **What failure are we trying to prevent?** | The scope and focus of your eval suite |
| 2 | **Who is affected when the system is wrong?** | The severity weighting and safety requirements |
| 3 | **What is the acceptable tradeoff between quality, latency, cost, and safety?** | Your multi-dimensional success criteria |
| 4 | **Do we need offline evaluation, online monitoring, or both?** | Your eval architecture and infrastructure |

### Worked Example

Consider a **medical QA assistant**:

| Question | Answer |
|---|---|
| What failure are we trying to prevent? | Hallucinated medical advice that contradicts clinical evidence |
| Who is affected? | Patients who may act on incorrect information — **high severity** |
| Acceptable tradeoffs? | Latency is secondary; safety and correctness are non-negotiable |
| Offline, online, or both? | Both — offline for regression testing, online for monitoring real-world queries |

Compare this with a **creative writing assistant**:

| Question | Answer |
|---|---|
| What failure are we trying to prevent? | Repetitive, boring, or off-tone outputs |
| Who is affected? | Writers who waste time — **moderate severity** |
| Acceptable tradeoffs? | Some factual looseness is fine; creativity and engagement matter more |
| Offline, online, or both? | Primarily offline; online A/B testing for engagement |

The same model might power both systems, but the evaluation strategies are fundamentally different.

---

## Failure Mode Thinking

The most productive way to start an eval project is to **enumerate failure modes**, not metrics.

### Common LLM Failure Modes

| Failure Mode | Description | Risk Level | Example |
|---|---|---|---|
| **Hallucination** | Generating plausible but factually incorrect content | 🔴 High | Citing a paper that doesn't exist |
| **Refusal** | Declining to answer a legitimate request | 🟡 Medium | Refusing to explain a medical concept |
| **Instruction Violation** | Ignoring formatting, length, or style constraints | 🟡 Medium | Returning JSON when asked for YAML |
| **Toxicity** | Producing harmful, offensive, or biased content | 🔴 High | Generating slurs in response to benign input |
| **Repetition** | Producing redundant or looping output | 🟢 Low | Repeating the same paragraph three times |
| **Incompleteness** | Answering partially or missing key information | 🟡 Medium | Listing 3 of 7 requested items |
| **Latent Bias** | Systematically favoring one demographic or viewpoint | 🔴 High | Recommending male candidates more often |
| **Confidentiality Breach** | Leaking private data from context or training | 🔴 High | Revealing PII from a retrieved document |

### From Failure Modes to Eval Criteria

```
Failure Mode → Eval Criterion → Metric → Dataset → Scoring Method
```

**Example:**
```
Hallucination → Faithfulness to source → Faithfulness score (0-1) → 200 QA pairs with ground truth → LLM judge + human audit
```

> 💡 **Tip**: Start with 3–5 critical failure modes. You can always expand later. Trying to measure everything from day one leads to an eval suite that measures nothing well.

---

## Human vs Automated Evaluation

Neither human nor automated evaluation is universally better. They serve different roles:

| Dimension | Human Evaluation | Automated Evaluation |
|---|---|---|
| **Strength** | Captures nuance, subjective quality, edge cases | Fast, cheap, reproducible, scalable |
| **Weakness** | Slow, expensive, inconsistent across annotators | Misses subtle quality differences |
| **Best for** | Calibration, rubric validation, safety review | Regression testing, CI/CD, high-volume scoring |
| **Scale** | 100s of examples | 10,000s of examples |
| **Cost** | $5–50 per example | $0.001–0.05 per example |
| **Turnaround** | Hours to days | Seconds to minutes |

### The Hybrid Approach (Recommended)

```
┌─────────────────────────────────────────────────────────┐
│  1. HUMAN EVAL → Build gold-standard reference set      │
│  2. CALIBRATE  → Validate that automated scores align   │
│  3. AUTOMATE   → Use automated scoring for CI/CD        │
│  4. AUDIT      → Periodically re-validate with humans   │
└─────────────────────────────────────────────────────────┘
```

Start with human evaluation to establish ground truth. Use those human-labeled examples to calibrate automated metrics. Run automated evals daily. Circle back to human review when automated scores diverge or new failure modes emerge.

---

## Offline vs Online Evaluation

| Aspect | Offline Evaluation | Online Evaluation |
|---|---|---|
| **When** | Before deployment (dev / staging) | After deployment (production) |
| **Data** | Curated test sets, synthetic data | Real user traffic |
| **Purpose** | Catch regressions, validate changes | Detect drift, measure real impact |
| **Feedback loop** | Developer → eval → iterate | Users → monitor → alert → iterate |
| **Examples** | Benchmark runs, regression suites | A/B tests, shadow scoring, user feedback |

### When You Need Both

- **Offline only** works for simple, low-risk applications with stable inputs
- **Online only** is dangerous — you're using production users as test subjects
- **Both** is required for any system where failures have real consequences

---

## Pointwise vs Pairwise Evaluation

Two fundamental paradigms for scoring:

| Approach | How It Works | Pros | Cons |
|---|---|---|---|
| **Pointwise** | Score each output independently on a rubric (e.g., 1–5) | Simple, absolute scores, easy to track over time | Calibration drift, scale ambiguity |
| **Pairwise** | Compare two outputs and pick the better one | More reliable relative judgments, reduces scale bias | Doesn't produce absolute scores, O(n²) comparisons |

### When to Use Each

- **Pointwise**: Regression testing (need absolute thresholds), CI/CD gates, large-scale automated scoring
- **Pairwise**: Model comparison, preference tuning, judge calibration

### Combining Both

Use pairwise comparisons to **calibrate** your pointwise rubric. If your 4/5 ratings consistently lose to a competitor in pairwise tests, your rubric may be inflated.

---

## LLM-as-a-Judge: Tool, Not Truth

Using an LLM to evaluate another LLM's output is powerful but comes with critical caveats:

### Known Biases

| Bias | Description | Mitigation |
|---|---|---|
| **Position bias** | Prefers the first (or last) option in a comparison | Randomize order, run both orderings |
| **Verbosity bias** | Rates longer answers higher regardless of quality | Include length-penalizing rubric criteria |
| **Self-preference bias** | A model rates its own outputs higher | Use a different model family as judge |
| **Sycophancy bias** | Agrees with the prompt framing rather than evaluating objectively | Use neutral, balanced prompt framing |
| **Authority bias** | Defers to confident-sounding answers | Include factual verification in rubric |

### Rules of Thumb

1. **Never use a judge without a calibration set** — Test your judge prompt against human-labeled examples first
2. **Disclose the judge model** — Results are only reproducible if the judge model is specified
3. **Cross-validate with humans periodically** — Judge quality can degrade as tasks or models change
4. **Use structured rubrics** — Vague instructions like "rate the quality" produce noisy scores
5. **Monitor judge agreement** — Track inter-rater reliability (judge vs human, judge vs judge)

---

## Building Your Evaluation Philosophy

Use this template to document your team's evaluation philosophy before building anything:

```markdown
## Our Evaluation Philosophy

### Application: [name]
### Critical failure modes:
1. ...
2. ...
3. ...

### Quality definition:
- Must: [non-negotiable requirements]
- Should: [important but negotiable]
- Nice-to-have: [aspirational goals]

### Tradeoff priorities (rank 1-4):
- [ ] Quality
- [ ] Latency
- [ ] Cost
- [ ] Safety

### Evaluation approach:
- Offline: [what we test before deploy]
- Online: [what we monitor after deploy]

### Scoring method:
- Primary: [human / automated / hybrid]
- Judge model: [if applicable]
- Calibration plan: [how we validate scores]
```

---

## Key Takeaways

| Principle | Details |
|---|---|
| **Start with failures, not metrics** | Enumerate what can go wrong before deciding how to measure it |
| **Define quality for your context** | A medical QA system and a creative writing tool need entirely different eval criteria |
| **Use the hybrid approach** | Human labels for calibration, automated scoring for scale |
| **Treat LLM judges as tools** | Calibrate them, monitor them, don't blindly trust them |
| **Evaluation is continuous** | Offline evals catch regressions; online evals catch drift |

---

Move to [01. Fundamentals →](../01_fundamentals/README.md) to learn the building blocks of evaluation: metrics, methods, and taxonomies.
