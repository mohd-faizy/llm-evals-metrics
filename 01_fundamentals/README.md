# 01. Evaluation Fundamentals

> **Purpose** — Learn the building blocks of LLM evaluation: what an eval is, what it measures, the types of metrics available, and how to choose the right evaluation method for your task. This section gives you the vocabulary and mental models used throughout the rest of the repository.

---

## Table of Contents

- [What Is an Eval?](#what-is-an-eval)
- [Anatomy of an Eval](#anatomy-of-an-eval)
- [Metric Categories](#metric-categories)
- [Reference-Based vs Reference-Free Evaluation](#reference-based-vs-reference-free-evaluation)
- [Human vs Automated Scoring](#human-vs-automated-scoring)
- [Pointwise vs Pairwise Methods](#pointwise-vs-pairwise-methods)
- [Taxonomy of Eval Types](#taxonomy-of-eval-types)
- [Common Metrics Reference](#common-metrics-reference)
- [Common Mistakes](#common-mistakes)
- [Outcome](#outcome)

---

## What Is an Eval?

An **eval** (evaluation) is a structured test that measures how well an LLM system performs on a specific task under defined conditions.

At its core, every eval answers one question:

> *"Given this input, did the system produce an acceptable output?"*

An eval is **not** a benchmark (which is a standardized collection of evals), and it is **not** a metric (which is how you score the output). It is the full pipeline:

```
Input → System Under Test → Output → Scoring → Result
```

---

## Anatomy of an Eval

Every eval has five components:

| Component | Description | Example |
|---|---|---|
| **Input** | The prompt, query, or context given to the system | "Summarize this 3-page contract" |
| **System Under Test** | The LLM, pipeline, agent, or application being evaluated | GPT-4o + RAG pipeline |
| **Output** | The system's response | A 200-word summary |
| **Expected Output** (optional) | Ground truth or reference answer for comparison | Human-written summary |
| **Scoring Function** | The method used to evaluate the output | ROUGE-L score, LLM judge, human rating |

### With vs Without Ground Truth

```
WITH ground truth:     Input → System → Output → Compare(Output, Reference) → Score
WITHOUT ground truth:  Input → System → Output → Judge(Output, Rubric) → Score
```

Not all evals have ground truth. Open-ended tasks (creative writing, conversational chat) often use rubric-based or preference-based evaluation instead.

---

## Metric Categories

Metrics fall into three broad families:

### 1. Lexical Metrics (String-Based)

Compare output text directly against a reference using token overlap.

| Metric | What It Measures | Formula Intuition | Best For |
|---|---|---|---|
| **Exact Match** | Perfect string equality | output == reference | Classification, entity extraction |
| **BLEU** | n-gram precision | How much of the output appears in the reference | Translation (historically) |
| **ROUGE-1** | Unigram recall | How much of the reference appears in the output | Summarization |
| **ROUGE-L** | Longest common subsequence | Structural similarity | Summarization |
| **F1 (token)** | Harmonic mean of precision/recall | Balance between coverage and accuracy | QA, extraction |

> ⚠️ **Limitation**: Lexical metrics penalize valid paraphrases. "The cat sat on the mat" and "A feline was resting on the rug" score poorly despite being semantically equivalent.

### 2. Semantic Metrics (Embedding-Based)

Compare meaning rather than exact words using vector similarity.

| Metric | What It Measures | How It Works | Best For |
|---|---|---|---|
| **BERTScore** | Semantic similarity | Token-level cosine similarity using BERT embeddings | Any text generation task |
| **Embedding Cosine Similarity** | Overall meaning overlap | Sentence-level embedding comparison | Retrieval, semantic search |
| **METEOR** | Semantic + lexical match | Combines synonyms, stemming, and word order | Translation, summarization |

### 3. Model-Based Metrics (LLM-as-a-Judge)

Use another LLM to evaluate the output against a rubric.

| Metric | What It Measures | How It Works | Best For |
|---|---|---|---|
| **Rubric Score (1–5)** | Overall quality on defined criteria | LLM rates output on a scale | General quality assessment |
| **Pairwise Preference** | Which of two outputs is better | LLM picks A or B (or tie) | Model comparison |
| **Faithfulness** | Factual consistency with source | LLM checks claims against context | RAG, summarization |
| **Helpfulness** | Whether the response solves the user's problem | LLM evaluates against user intent | Chatbots, assistants |

---

## Reference-Based vs Reference-Free Evaluation

| Dimension | Reference-Based | Reference-Free |
|---|---|---|
| **Definition** | Compares output to a known-correct answer | Evaluates output based on rubric/criteria alone |
| **Requires** | Ground truth dataset | Scoring rubric or judge model |
| **Precision** | High (if references are high-quality) | Variable (depends on rubric quality) |
| **Scalability** | Limited by annotation cost | Highly scalable |
| **Best for** | Factual QA, classification, extraction | Open-ended generation, creativity, style |
| **Examples** | Exact match, ROUGE, BERTScore | LLM judge, human rating, preference ranking |

### Decision Matrix

```
Can you define a single correct answer?
├── YES → Reference-based (Exact Match, ROUGE, F1)
│         ├── Is lexical similarity sufficient?
│         │   ├── YES → Lexical metrics (BLEU, ROUGE)
│         │   └── NO  → Semantic metrics (BERTScore)
│         └── Do you need to measure multiple dimensions?
│             └── YES → LLM judge with rubric + reference
└── NO  → Reference-free
          ├── Can you define clear quality criteria?
          │   ├── YES → LLM judge with rubric
          │   └── NO  → Human evaluation / preference ranking
          └── Is relative comparison more useful?
              └── YES → Pairwise evaluation
```

---

## Human vs Automated Scoring

### Comparison

| Aspect | Human Scoring | Automated Scoring |
|---|---|---|
| **Cost** | $5–50 per example | $0.001–0.05 per example |
| **Speed** | Hours to days | Seconds |
| **Consistency** | Variable (inter-annotator agreement ~70-85%) | Perfectly consistent (given same inputs) |
| **Nuance** | Captures subjective quality, cultural context | Misses subtle quality differences |
| **Scale** | 100s of examples | 10,000s of examples |
| **Bias** | Annotator bias, fatigue | Model bias, prompt sensitivity |

### When to Use Each

| Scenario | Recommended Approach |
|---|---|
| Building initial eval dataset | Human |
| Calibrating LLM judge prompts | Human (gold standard) → validate automated |
| Daily regression testing | Automated |
| CI/CD gating | Automated |
| Safety review of edge cases | Human |
| New task or domain | Human first, then automate |
| High-stakes deployment decision | Human + automated |

---

## Pointwise vs Pairwise Methods

### Pointwise Evaluation

Each output is scored independently on an absolute scale.

```
Input: "What is the capital of France?"
Output: "The capital of France is Paris, a city known for the Eiffel Tower."
Score: 5/5 (Correct, complete, well-formatted)
```

**Strengths**: Produces absolute scores, easy to track over time, works at scale
**Weaknesses**: Calibration drift, annotators may interpret scales differently

### Pairwise Evaluation

Two outputs are compared head-to-head, and the better one is selected.

```
Input: "What is the capital of France?"
Output A: "Paris"
Output B: "The capital of France is Paris."
Winner: B (more complete)
```

**Strengths**: More reliable relative judgments, eliminates scale ambiguity
**Weaknesses**: No absolute scores, O(n²) comparisons, can't threshold directly

### Side-by-Side Comparison

| Feature | Pointwise | Pairwise |
|---|---|---|
| Output | Absolute score (e.g., 4/5) | Relative preference (A > B) |
| Scale needed | 1–5 or 1–10 rubric | Binary or ternary (A / B / Tie) |
| Comparison count | O(n) | O(n²) |
| Use case | Regression testing, CI gates | Model selection, RLHF |
| Score stability | Subject to calibration drift | More stable across annotators |

---

## Taxonomy of Eval Types

| Eval Type | What It Tests | Scope | Example |
|---|---|---|---|
| **Unit Eval** | Single capability in isolation | One input → one output | "Does the model extract dates correctly?" |
| **Integration Eval** | End-to-end system behavior | Full pipeline | "Does the RAG system answer correctly?" |
| **Regression Eval** | Whether a change broke something | Before/after comparison | "Did the prompt update degrade summarization?" |
| **Stress Eval** | Behavior under extreme conditions | Edge cases, adversarial inputs | "What happens with 100K token context?" |
| **Slice Eval** | Performance on specific subgroups | Filtered data segments | "How does it perform on medical vs legal questions?" |
| **A/B Eval** | Which variant performs better | Two system versions | "Does version 2 get higher user ratings?" |
| **Safety Eval** | Harmful or risky behavior | Adversarial and edge cases | "Can the model be jailbroken?" |
| **Consistency Eval** | Stability across runs | Same input, multiple runs | "Does the model give different answers each time?" |

---

## Common Metrics Reference

### Quick-Reference Table

| Metric | Type | Range | Higher = Better? | Use Case |
|---|---|---|---|---|
| Exact Match | Lexical | 0 or 1 | ✅ | Classification, extraction |
| BLEU | Lexical | 0–1 | ✅ | Translation |
| ROUGE-1 | Lexical | 0–1 | ✅ | Summarization |
| ROUGE-L | Lexical | 0–1 | ✅ | Summarization |
| BERTScore | Semantic | -1 to 1 | ✅ | General text generation |
| Cosine Similarity | Semantic | -1 to 1 | ✅ | Retrieval, semantic search |
| Perplexity | Probabilistic | 1 to ∞ | ❌ | Language modeling |
| pass@k | Functional | 0–1 | ✅ | Code generation |
| Faithfulness | Model-based | 0–1 | ✅ | RAG, summarization |
| Win Rate | Preference | 0–1 | ✅ | Model comparison |
| Cohen's Kappa | Agreement | -1 to 1 | ✅ | Inter-annotator reliability |

---

## Common Mistakes

| Mistake | Why It's a Problem | What to Do Instead |
|---|---|---|
| **Using BLEU/ROUGE for open-ended tasks** | Penalizes valid paraphrases | Use semantic metrics or LLM judge |
| **Measuring accuracy without defining "correct"** | Ambiguous ground truth → noisy scores | Write explicit rubrics first |
| **Using a single metric** | Hides multi-dimensional tradeoffs | Use a metric suite (correctness + fluency + safety) |
| **Not versioning eval datasets** | Can't reproduce past results | Git-track datasets alongside code |
| **Optimizing for the metric instead of the goal** | Goodhart's Law — the metric stops measuring what you care about | Regularly validate metrics against human judgment |
| **Skipping inter-annotator agreement** | You don't know if your labels are reliable | Compute Cohen's Kappa or Krippendorff's Alpha |

---

## Outcome

By the end of this section, you should be able to:

1. ✅ Decompose any evaluation task into its five components (input, system, output, reference, scoring)
2. ✅ Choose between lexical, semantic, and model-based metrics for your use case
3. ✅ Decide when to use reference-based vs reference-free evaluation
4. ✅ Select pointwise vs pairwise methods based on your goal
5. ✅ Define a testable evaluation problem **before** selecting any tooling

---

Move to [02. Landscape →](../02_landscape/README.md) to explore the evaluation ecosystem — frameworks, platforms, and tools available.
