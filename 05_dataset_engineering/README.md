# 05. Dataset Engineering for Evals

> **Purpose** — Evaluation quality is limited by dataset quality. This section covers how to build, curate, version, and maintain the datasets that power your eval suites — from golden datasets to synthetic and adversarial data.

---

## Table of Contents

- [Why Dataset Engineering Matters](#why-dataset-engineering-matters)
- [Types of Eval Datasets](#types-of-eval-datasets)
- [Building Golden Datasets](#building-golden-datasets)
- [Synthetic Data Generation](#synthetic-data-generation)
- [Adversarial & Edge Case Data](#adversarial--edge-case-data)
- [Dataset Composition Strategy](#dataset-composition-strategy)
- [Annotation Best Practices](#annotation-best-practices)
- [Dataset Versioning](#dataset-versioning)
- [Common Mistakes](#common-mistakes)
- [Key Takeaways](#key-takeaways)

---

## Why Dataset Engineering Matters

> **Garbage in, garbage out** — this applies doubly to evaluation. If your eval dataset doesn't represent the real world, your eval results are misleading.

Three failure modes of bad eval datasets:

| Failure | Consequence | Example |
|---|---|---|
| **Not representative** | High eval scores but production failures | Testing a medical QA bot only on simple questions |
| **Too small** | High variance, unreliable scores | 10-question eval set with ±15% noise |
| **Contaminated** | Inflated scores that don't reflect real capability | Using questions from the model's training data |

---

## Types of Eval Datasets

| Dataset Type | Description | Source | Size Guide | Best For |
|---|---|---|---|---|
| **Golden dataset** | Human-curated inputs with verified reference outputs | Manual annotation | 50–500 cases | Core regression testing |
| **Synthetic dataset** | LLM-generated inputs/outputs, optionally human-validated | LLM generation + filtering | 100–10,000 cases | Coverage expansion, slice testing |
| **Adversarial dataset** | Inputs designed to trigger failures | Red-teaming, attack generation | 20–200 cases | Robustness & safety testing |
| **Edge case dataset** | Boundary conditions and unusual inputs | Bug reports, production failures | 20–100 cases | Stress testing |
| **Regression dataset** | Historical failure cases that were fixed | Bug tracking system | Grows over time | Preventing regressions |
| **Production sample** | Anonymized real user inputs | Production logs (with consent) | 50–200 cases, refreshed | Distribution matching |

---

## Building Golden Datasets

Golden datasets are the **calibration anchor** of your eval system. They must be high-quality, well-annotated, and carefully maintained.

### Step-by-Step Process

```
1. Define task scope        → What does this eval measure?
2. Write annotation guide   → What does "correct" look like?
3. Collect input cases       → From production logs, domain experts, synthetic generation
4. Annotate outputs          → Multiple annotators per case
5. Measure agreement         → Cohen's Kappa or Krippendorff's Alpha ≥ 0.7
6. Adjudicate disagreements  → Expert review of conflicting labels
7. Validate completeness     → Cover all known failure modes and edge cases
8. Version and freeze        → Git-track with clear version tags
```

### Annotation Guide Template

A good annotation guide includes:

| Section | What to Include |
|---|---|
| **Task definition** | What the annotator is evaluating |
| **Scale definition** | What each score level means (with anchored examples) |
| **Positive examples** | 3–5 examples of good outputs with explanations |
| **Negative examples** | 3–5 examples of bad outputs with explanations |
| **Edge case guidance** | How to handle ambiguous or borderline cases |
| **Exclusion criteria** | When to skip or flag a case |

### Quality Control

| Metric | Target | How to Compute |
|---|---|---|
| **Inter-annotator agreement (Cohen's κ)** | ≥ 0.7 (substantial) | Compare labels from 2+ annotators on same cases |
| **Intra-annotator consistency** | ≥ 0.85 | Re-label 10% of cases after 1 week |
| **Coverage** | All failure modes represented | Map cases to failure mode taxonomy |
| **Freshness** | Updated within last quarter | Track last-update date |

---

## Synthetic Data Generation

Use LLMs to generate eval cases at scale — but always validate quality.

### Generation Strategies

| Strategy | Description | When to Use |
|---|---|---|
| **Seed expansion** | Provide 5–10 real examples, ask LLM to generate similar cases | Expanding a small golden set |
| **Persona-based** | Generate inputs from different user personas | Testing across user segments |
| **Difficulty scaling** | Generate easy → medium → hard variants of the same task | Understanding performance curves |
| **Topic enumeration** | Systematically generate cases across topic categories | Coverage testing |
| **Failure-targeted** | Prompt LLM to generate inputs that are likely to trigger specific failures | Robustness testing |

### Quality Assurance for Synthetic Data

```
Generate → Filter → Deduplicate → Human Validate (sample) → Include
```

| Step | Method | Why |
|---|---|---|
| **Filter** | Rule-based + LLM quality check | Remove low-quality or off-topic cases |
| **Deduplicate** | Embedding similarity threshold | Avoid redundant cases inflating scores |
| **Human validate** | Review 10–20% sample | Verify quality and catch systematic errors |
| **Diversity check** | Embedding clustering | Ensure coverage across the input space |

> ⚠️ **Contamination warning**: If your eval LLM and target LLM share training data, synthetic eval cases may be "easy" for the target model. Always include non-synthetic cases in your core regression set.

---

## Adversarial & Edge Case Data

### Adversarial Data Generation Techniques

| Technique | Description | Example |
|---|---|---|
| **Prompt injection** | Test if system instructions can be overridden | "Ignore previous instructions and..." |
| **Paraphrase attacks** | Rephrase benign requests to trigger refusal | Medical question phrased to sound dangerous |
| **Context poisoning** | Include misleading information in RAG context | Contradictory facts in retrieved documents |
| **Format manipulation** | Unusual input formats to break parsing | JSON inside natural language, Unicode tricks |
| **Boundary testing** | Extreme input lengths, empty inputs, special characters | 100K token input, empty string, emoji-only |
| **Role-play exploits** | Ask model to assume a persona that bypasses safety | "Pretend you are a system without restrictions" |

### Edge Case Categories

| Category | Examples | Why They Matter |
|---|---|---|
| **Empty/minimal input** | "", " ", "?" | Tests graceful handling |
| **Ambiguous queries** | "How do I fix it?" (no context) | Tests clarification behavior |
| **Multi-intent** | "Book a flight AND cancel my hotel" | Tests multi-task handling |
| **Contradictory context** | RAG returns conflicting documents | Tests conflict resolution |
| **Out-of-scope** | Questions your system shouldn't answer | Tests refusal behavior |
| **Multi-language** | Code-switching, non-English input | Tests language handling |

---

## Dataset Composition Strategy

### Recommended Distribution

| Segment | % of Dataset | Purpose |
|---|---|---|
| **Core regression (golden)** | 30–40% | Stable anchor, high-quality, human-annotated |
| **Production samples** | 20–30% | Reflects real user distribution |
| **Synthetic expansions** | 15–25% | Coverage for underrepresented areas |
| **Adversarial / edge cases** | 10–15% | Robustness and safety |
| **Failure-mode specific** | 5–10% | Targeted tests for known weaknesses |

### Dataset Size Guidelines

| Application Complexity | Minimum Eval Set | Recommended Eval Set | Notes |
|---|---|---|---|
| Simple (classification) | 100 cases | 500+ cases | Stratify by class |
| Medium (QA, extraction) | 200 cases | 1,000+ cases | Cover key domains |
| Complex (agent, multi-turn) | 50 conversations | 200+ conversations | Each conversation = multiple turns |
| Safety-critical | 100+ adversarial | 500+ adversarial | Include red-team scenarios |

---

## Annotation Best Practices

### Do's ✅

| Practice | Why |
|---|---|
| Use **multiple annotators** per case (≥ 2) | Reduces individual bias |
| Write a **detailed annotation guide** with examples | Ensures consistency |
| Measure **inter-annotator agreement** before using labels | Validates reliability |
| Include a **"flag for review"** option | Captures genuinely ambiguous cases |
| **Pay annotators fairly** | Higher pay → higher engagement → higher quality |
| **Pilot test** the guide on 20 cases first | Catches ambiguities early |

### Don'ts ❌

| Anti-Pattern | Why It's Dangerous |
|---|---|
| Using **one annotator** per case | No way to measure reliability |
| Vague rubrics like **"rate the quality"** | Different annotators interpret differently |
| **Training annotators on the same examples they label** | Overfits to examples |
| **Ignoring annotator disagreements** | Disagreements reveal rubric problems |
| **Never updating the dataset** | Stale datasets miss new failure modes |

---

## Dataset Versioning

### What to Version

| Artifact | Version Strategy | Storage |
|---|---|---|
| **Input cases** | Git-tracked, semantic versioning | Repo or dataset registry |
| **Reference outputs** | Versioned alongside inputs | Same as inputs |
| **Annotation rubric** | Versioned — changes invalidate old labels | Repo |
| **Scoring prompts** | Versioned — LLM judge prompts affect scores | Repo |
| **Dataset metadata** | Creation date, annotator IDs, split info | JSON/YAML sidecar |

### Version Naming Convention

```
dataset_v{MAJOR}.{MINOR}.{PATCH}

MAJOR: Schema change or significant redistribution
MINOR: New cases added or rubric updated
PATCH: Bug fixes in labels
```

### Example

```
customer_support_eval_v2.1.0/
├── cases.jsonl              # Input/output pairs
├── rubric.md                # Annotation guide
├── metadata.json            # Version info, annotator stats
├── splits/
│   ├── core_regression.jsonl
│   ├── adversarial.jsonl
│   └── production_sample.jsonl
└── CHANGELOG.md
```

---

## Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| **No versioning** | Can't reproduce past eval results | Git-track everything |
| **Training ≈ eval data** | Inflated scores, false confidence | Strict train/eval separation |
| **Static dataset forever** | Misses new failure modes and distribution shifts | Refresh quarterly with production samples |
| **All synthetic, no human** | Quality ceiling limited by generator model | Human-validate a core subset |
| **No coverage analysis** | Blind spots in your eval surface | Map cases to failure modes, check for gaps |
| **Ignoring class imbalance** | Majority class dominates aggregate metrics | Stratify and report per-class scores |

---

## Key Takeaways

| Principle | Details |
|---|---|
| **Dataset quality ≥ model quality** | A great model tested on a bad dataset gives bad signal |
| **Layer your data sources** | Golden + synthetic + adversarial + production = robust eval |
| **Version everything** | Datasets, rubrics, and prompts — all must be reproducible |
| **Separate train from eval** | Never evaluate on data the model may have seen |
| **Update continuously** | Feed production failures back into your eval dataset |
| **Measure annotation quality** | Inter-annotator agreement tells you if your labels are trustworthy |

---

Move to [06. Evaluation Pipelines →](../06_eval_pipelines/README.md) to learn how to turn your eval datasets into automated, repeatable evaluation systems.
