# 07. LLM-as-a-Judge

> **Purpose** — Master model-based evaluation: how to use LLMs to score other LLMs' outputs reliably. Covers rubric design, prompt engineering for judges, calibration techniques, known biases, and when to trust (and distrust) automated judgments.

---

## Table of Contents

- [What Is LLM-as-a-Judge?](#what-is-llm-as-a-judge)
- [Why Use LLM Judges?](#why-use-llm-judges)
- [Judge Architectures](#judge-architectures)
- [Rubric Design](#rubric-design)
- [Prompt Engineering for Judges](#prompt-engineering-for-judges)
- [Calibration](#calibration)
- [Known Biases](#known-biases)
- [Judge Model Selection](#judge-model-selection)
- [Self-Consistency & Agreement](#self-consistency--agreement)
- [Pairwise Ranking](#pairwise-ranking)
- [Advanced Techniques](#advanced-techniques)
- [Common Mistakes](#common-mistakes)
- [Key Takeaways](#key-takeaways)

---

## What Is LLM-as-a-Judge?

LLM-as-a-Judge uses a language model to evaluate the output of another language model (or the same model) against a defined rubric or set of criteria.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  User Query  │────>│  Target LLM  │────>│   Output     │
└──────────────┘     └──────────────┘     └──────┬───────┘
                                                  │
                     ┌──────────────┐              │
                     │  Judge LLM   │<─────────────┘
                     │  + Rubric    │
                     └──────┬───────┘
                            │
                     ┌──────v───────┐
                     │    Score     │
                     │  + Reasoning │
                     └──────────────┘
```

---

## Why Use LLM Judges?

| Advantage | Details |
|---|---|
| **Scale** | Score thousands of examples in minutes vs days for human review |
| **Cost** | ~$0.01–0.10 per judgment vs $5–50 for human evaluation |
| **Consistency** | No fatigue, no mood variation (but has its own biases) |
| **Multi-dimensional** | Can score correctness, style, safety, and more in one pass |
| **Nuance** | Captures quality differences that rule-based metrics miss |

| Limitation | Details |
|---|---|
| **Bias** | Position, verbosity, self-preference, sycophancy biases |
| **Calibration drift** | Judge quality changes with model updates |
| **Domain gaps** | May not understand specialized domains |
| **Not ground truth** | A judge score is an opinion, not a fact |
| **Cost at scale** | Still more expensive than rule-based metrics |

---

## Judge Architectures

### Architecture Comparison

| Architecture | Description | Pros | Cons | Best For |
|---|---|---|---|---|
| **Pointwise** | Score a single output on a rubric (1–5) | Simple, absolute scores | Calibration-sensitive | Regression testing, CI gates |
| **Pairwise** | Compare two outputs, pick the better one | More reliable relative judgments | No absolute score, O(n²) | Model comparison, RLHF |
| **Reference-based** | Compare output against a golden reference | Grounds judgment in truth | Requires reference answers | Factual QA, extraction |
| **Reference-free** | Evaluate output quality without a reference | Works for open-ended tasks | Less precise | Creative writing, chat |
| **Multi-aspect** | Score multiple dimensions separately | Detailed, actionable feedback | More expensive (more tokens) | Complex applications |
| **Chain-of-thought** | Judge explains reasoning before scoring | Better calibrated scores | Slower, more expensive | High-stakes decisions |

---

## Rubric Design

The rubric is the single most important factor in judge quality. A vague rubric produces noisy scores; a precise rubric produces reliable scores.

### Rubric Design Principles

| Principle | Bad Example | Good Example |
|---|---|---|
| **Be specific** | "Rate the quality" | "Rate the factual accuracy of the response against the provided context" |
| **Anchor the scale** | "Score 1–5" | "1 = Major factual errors, 3 = Mostly correct with minor issues, 5 = Fully accurate with all claims supported" |
| **Define dimensions separately** | "Rate overall quality (includes correctness, style, and helpfulness)" | Score correctness, style, and helpfulness as separate 1–5 dimensions |
| **Provide examples** | No examples | Include 2–3 examples per score level |
| **Handle edge cases** | Ignore ambiguity | "If the response is partially correct, score 3 and explain which parts are incorrect" |

### Example: 5-Point Factual Accuracy Rubric

| Score | Label | Definition | Example Behavior |
|---|---|---|---|
| **5** | Fully accurate | All claims are factually correct and verifiable against the source | Every statement matches the source document |
| **4** | Mostly accurate | Minor inaccuracies that don't affect the core answer | Correct answer with a slightly wrong date |
| **3** | Partially accurate | Mix of correct and incorrect information | Right conclusion from wrong reasoning |
| **2** | Mostly inaccurate | Major factual errors in the core answer | Wrong answer with some correct details |
| **1** | Completely wrong | Fabricated or contradicts the source | Hallucinated information not in any source |

### Multi-Aspect Rubric Template

```markdown
## Evaluation Rubric

Evaluate the response on the following dimensions independently.
For each dimension, provide a score from 1-5 and a brief justification.

### Correctness (1-5)
- 5: All information is factually accurate
- 3: Mostly correct with minor errors
- 1: Contains major factual errors or hallucinations

### Completeness (1-5)
- 5: Addresses all parts of the question thoroughly
- 3: Addresses the main question but misses secondary aspects
- 1: Fails to address the core question

### Helpfulness (1-5)
- 5: Directly actionable, provides clear next steps
- 3: Somewhat useful but requires additional effort from the user
- 1: Not useful for the user's purpose

### Safety (Pass/Fail)
- Pass: No harmful, biased, or inappropriate content
- Fail: Contains harmful content, PII, or policy violations
```

---

## Prompt Engineering for Judges

### Judge Prompt Structure

A well-structured judge prompt has five sections:

```
1. ROLE         → "You are an expert evaluator..."
2. TASK         → "Evaluate the following response..."
3. RUBRIC       → Detailed scoring criteria with examples
4. INPUT        → The user query + system output (+ reference if applicable)
5. OUTPUT FORMAT → Structured output (JSON preferred)
```

### Example Judge Prompt

```markdown
You are an expert evaluator assessing the quality of AI-generated responses.

## Task
Evaluate the following response to a user query. Score it on Correctness
and Helpfulness using the rubrics below.

## Rubric

### Correctness (1-5)
5 = All claims are factually accurate and verifiable
4 = Mostly accurate, minor issues only
3 = Mix of accurate and inaccurate information
2 = Major factual errors present
1 = Completely incorrect or fabricated

### Helpfulness (1-5)
5 = Directly answers the question with actionable guidance
4 = Answers the question well, minor improvements possible
3 = Partially addresses the question
2 = Tangentially related but doesn't answer the question
1 = Irrelevant or unhelpful

## Input
**User Query**: {{query}}
**Response**: {{response}}

## Output Format
Respond in JSON format:
{
  "correctness_reasoning": "...",
  "correctness_score": <1-5>,
  "helpfulness_reasoning": "...",
  "helpfulness_score": <1-5>
}
```

### Prompt Engineering Tips

| Tip | Why |
|---|---|
| **Ask for reasoning before the score** | Chain-of-thought improves calibration (by 10-15% in studies) |
| **Use structured output (JSON)** | Easier to parse, reduces format variance |
| **Include negative examples** | Anchors the low end of the scale |
| **Specify what to ignore** | "Do not penalize for response length" prevents verbosity bias |
| **Set temperature to 0** | Maximizes reproducibility |

---

## Calibration

Calibration ensures your LLM judge scores align with human judgment.

### Calibration Process

```
1. Create a calibration set     → 30–50 cases with human scores
2. Run judge on calibration set → Get automated scores
3. Measure agreement            → Cohen's Kappa, Spearman correlation
4. Analyze disagreements        → Where and why does the judge diverge?
5. Refine rubric/prompt         → Adjust based on disagreement patterns
6. Re-run and re-measure        → Iterate until agreement ≥ target
```

### Agreement Metrics

| Metric | What It Measures | Target | Interpretation |
|---|---|---|---|
| **Cohen's Kappa (κ)** | Agreement beyond chance (categorical) | ≥ 0.6 | < 0.4 poor, 0.4–0.6 moderate, 0.6–0.8 substantial, > 0.8 near-perfect |
| **Spearman Correlation (ρ)** | Rank-order agreement (ordinal) | ≥ 0.7 | How well the judge preserves the ranking of outputs |
| **Mean Absolute Error** | Average score difference | ≤ 0.5 | On a 1–5 scale, judge scores within 0.5 of human |
| **Exact Agreement %** | Percentage of identical scores | ≥ 60% | Strict but informative |

### When Calibration Fails

| Symptom | Likely Cause | Fix |
|---|---|---|
| Judge always scores high (4–5) | Rubric too lenient, no negative anchors | Add failure examples, tighten rubric |
| Judge always scores low (1–2) | Rubric too strict, unrealistic standards | Adjust expectations, add positive anchors |
| High variance across runs | Temperature too high, rubric ambiguous | Set temperature=0, clarify rubric |
| Disagrees on specific categories | Domain knowledge gap | Add domain context to prompt, or use domain-specific judge |

---

## Known Biases

| Bias | Description | Detection | Mitigation |
|---|---|---|---|
| **Position bias** | Prefers the first (or last) option in pairwise comparison | Run A-vs-B and B-vs-A, check for asymmetry | Randomize order, average both orderings |
| **Verbosity bias** | Rates longer, more detailed answers higher | Correlate scores with response length | Add "do not favor length" to rubric, control for length |
| **Self-preference bias** | A model rates its own outputs higher than competitors | Compare self-eval vs cross-model eval scores | Use a different model family as judge |
| **Sycophancy bias** | Agrees with the framing in the prompt | Test with misleading prompts | Use neutral, unbiased prompt framing |
| **Authority bias** | Favors confident-sounding answers over uncertain ones | Compare scores for hedged vs confident wrong answers | Include "penalize confident errors" in rubric |
| **Format bias** | Prefers well-formatted (markdown, bullets) even if content is wrong | Test identical content with different formatting | Separate content and format scoring |
| **Recency bias** | In multi-turn, over-weights later context | Test with different context orderings | Explicitly reference relevant context |

---

## Judge Model Selection

| Scenario | Recommended Judge | Rationale |
|---|---|---|
| General quality assessment | GPT-4o, Claude 3.5 Sonnet | Best overall calibration |
| Evaluating OpenAI models | Claude or Gemini | Avoids self-preference bias |
| Evaluating Anthropic models | GPT-4o or Gemini | Avoids self-preference bias |
| Privacy-sensitive data | Llama 3.1 70B+ (self-hosted) | Data stays on your infrastructure |
| High-volume, cost-sensitive | Gemini Flash, Claude Haiku, GPT-4o-mini | 10–50x cheaper per judgment |
| Standardized rubric scoring | Prometheus 2 (open-source) | Purpose-built for eval |
| Multi-modal evaluation | GPT-4o, Gemini Pro | Native vision capabilities |

### Cost Comparison (Approximate per 1,000 Judgments)

| Judge Model | Input Cost | Output Cost | Total (est.) |
|---|---|---|---|
| GPT-4o | ~$2.50 | ~$7.50 | ~$10 |
| Claude 3.5 Sonnet | ~$1.50 | ~$7.50 | ~$9 |
| Gemini 1.5 Pro | ~$0.63 | ~$2.50 | ~$3 |
| GPT-4o-mini | ~$0.08 | ~$0.30 | ~$0.40 |
| Llama 3.1 70B (self-hosted) | Infra cost | Infra cost | ~$0.50–2 |

---

## Self-Consistency & Agreement

### Measuring Judge Reliability

Run the same judgment multiple times and measure consistency:

| Test | Method | Healthy Range |
|---|---|---|
| **Self-agreement** | Run same case 5x, check score variance | σ ≤ 0.3 on 1–5 scale |
| **Inter-judge agreement** | Compare two different judge models | κ ≥ 0.5 |
| **Human-judge agreement** | Compare judge scores to human labels | κ ≥ 0.6 |
| **Temporal stability** | Re-run after model update | Δ ≤ 0.2 mean shift |

### Boosting Consistency

| Technique | How It Helps |
|---|---|
| **Temperature = 0** | Eliminates sampling randomness |
| **Structured output (JSON)** | Reduces format-related variance |
| **Chain-of-thought** | Forces explicit reasoning before scoring |
| **Majority voting** | Run 3x, take median score |
| **Ensemble judging** | Average scores from 2+ judge models |

---

## Pairwise Ranking

### When to Use Pairwise

- Comparing two model versions (A/B testing)
- Evaluating prompt variants
- Building preference datasets for RLHF
- When absolute scoring is unreliable

### Pairwise Prompt Template

```markdown
You are comparing two responses to the same user query.

## User Query
{{query}}

## Response A
{{response_a}}

## Response B
{{response_b}}

## Task
Which response better addresses the user's query?
Consider: correctness, completeness, helpfulness, and clarity.

Respond in JSON:
{
  "reasoning": "...",
  "winner": "A" | "B" | "tie",
  "confidence": "high" | "medium" | "low"
}
```

### Mitigating Position Bias in Pairwise

```
Run 1: A first, B second → Result
Run 2: B first, A second → Result
Final: Agree → use that result; Disagree → mark as "tie" or "uncertain"
```

---

## Advanced Techniques

### 1. Critique-then-Score

Have the judge first critique the response, then score it:

```
Step 1: "List all issues with this response"
Step 2: "Based on your critique, assign a score from 1-5"
```

This typically produces better-calibrated scores than direct scoring.

### 2. Decomposed Scoring

Break complex evaluation into simpler sub-tasks:

```
Overall quality → Factual accuracy score
                → Completeness score
                → Style score
                → Safety check
                → Weighted composite
```

Each sub-judge can be optimized independently.

### 3. Calibration Anchoring

Include known-scored examples in the judge prompt:

```
"Here are examples of each score level for reference:
Score 5 example: [example]
Score 3 example: [example]
Score 1 example: [example]

Now evaluate the following response:"
```

### 4. Meta-Judging

Use a second judge to evaluate the first judge's reasoning:

```
Judge 1: Scores the output → Score + Reasoning
Judge 2: Evaluates Judge 1's reasoning → Valid / Invalid
If Invalid: Re-score with refined prompt
```

---

## Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| **No calibration set** | No way to know if scores are meaningful | Build 30–50 human-labeled calibration cases |
| **Vague rubric** | Noisy, inconsistent scores | Anchor every score level with examples |
| **Same model as judge and target** | Self-preference bias | Use a different model family |
| **Ignoring bias** | Systematically wrong scores | Test for position, verbosity, and format bias |
| **Not tracking judge model version** | Scores become unreproducible after model update | Log exact judge model version |
| **Single run, no consistency check** | High variance goes undetected | Run multiple times, measure agreement |

---

## Key Takeaways

| Principle | Details |
|---|---|
| **The rubric is everything** | Invest more time in rubric design than in prompt tricks |
| **Calibrate against humans** | A judge without calibration is just guessing with extra steps |
| **Know the biases** | Position, verbosity, self-preference — test for all of them |
| **Use chain-of-thought** | Reasoning before scoring improves calibration significantly |
| **Choose the right judge model** | Avoid self-evaluation; match judge capability to task complexity |
| **Judge models are tools, not truth** | Never treat a judge score as ground truth without validation |

---

Move to [08. RAG Evals →](../08_rag_evals/README.md) to learn how to evaluate retrieval-augmented generation systems — measuring both retrieval quality and generation faithfulness.
