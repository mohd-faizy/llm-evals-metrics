# 13. Memory Evals

> **Purpose** — Evaluate memory systems in LLM agents, assistants, and personalized applications. Memory — short-term, long-term, semantic, episodic — enables continuity across conversations, personalization, and learning from past interactions. When memory fails, agents forget, hallucinate past events, or surface irrelevant information. This section covers how to test for all of these.

---

## Table of Contents

- [Why Memory Matters](#why-memory-matters)
- [Types of Memory in LLM Systems](#types-of-memory-in-llm-systems)
- [What to Evaluate](#what-to-evaluate)
- [Memory Metrics](#memory-metrics)
- [Evaluation Strategies](#evaluation-strategies)
- [Building Memory Eval Datasets](#building-memory-eval-datasets)
- [Common Failure Modes](#common-failure-modes)
- [Testing Patterns](#testing-patterns)
- [Memory vs Context Window](#memory-vs-context-window)
- [Common Mistakes](#common-mistakes)
- [Key Takeaways](#key-takeaways)

---

## Why Memory Matters

Without memory, every conversation starts from scratch. Memory enables:

| Capability | Example | Without Memory |
|---|---|---|
| **Continuity** | "As we discussed last week..." | Every session is a blank slate |
| **Personalization** | "You prefer dark mode, here's the config" | Generic responses every time |
| **Learning** | "Last time this approach didn't work, let's try X" | Repeats the same mistakes |
| **Context retention** | Referencing facts from turn 5 in turn 50 | Loses track of earlier context |
| **Deduplication** | "We already covered this topic" | Repeats information needlessly |

---

## Types of Memory in LLM Systems

| Memory Type | Scope | Persistence | Storage | Example |
|---|---|---|---|---|
| **In-context (short-term)** | Current conversation | Session only | Context window | Previous turns in a chat |
| **Working memory** | Current task | Task duration | Scratchpad / state | Agent's current plan and observations |
| **Episodic memory** | Past interactions | Long-term | Database / vector store | "User asked about X on March 5th" |
| **Semantic memory** | Learned facts | Long-term | Knowledge base / embeddings | "User is a Python developer who prefers Flask" |
| **Procedural memory** | Learned skills/patterns | Long-term | Fine-tuning / prompt library | "When asked about deployment, check k8s config first" |

### Memory Architecture

```
┌──────────────────────────────────────────────────────────┐
│                    MEMORY SYSTEM                          │
│                                                          │
│  ┌──────────────┐   ┌────────────────┐   ┌───────────┐  │
│  │   In-Context  │   │    Episodic     │   │  Semantic │  │
│  │   Memory      │   │    Memory       │   │  Memory   │  │
│  │               │   │                │   │           │  │
│  │  Recent turns │   │  Past sessions │   │  User     │  │
│  │  Current task │   │  Interactions  │   │  profile  │  │
│  │  state        │   │  Events        │   │  Facts    │  │
│  └──────┬───────┘   └───────┬────────┘   └─────┬─────┘  │
│         └──────────────┬─────┘─────────────────┘         │
│                        v                                  │
│              ┌──────────────────┐                         │
│              │   Memory         │                         │
│              │   Manager        │──> Retrieve, Store,     │
│              │                  │    Update, Forget       │
│              └──────────────────┘                         │
└──────────────────────────────────────────────────────────┘
```

---

## What to Evaluate

### Evaluation Dimensions

| Dimension | Question | Example Test |
|---|---|---|
| **Recall** | Can the system retrieve relevant past information? | "What did I tell you my name was?" |
| **Precision** | Is only relevant information retrieved? | Doesn't surface unrelated past conversations |
| **Accuracy** | Is the recalled information correct? | Correctly states the user's preference |
| **Temporal ordering** | Does the system maintain chronological order? | "Which project did we discuss first?" |
| **Update handling** | When information changes, does memory update? | "I've moved to Berlin" → updates location |
| **Forgetting (appropriate)** | Does the system forget what it should? | Doesn't recall deleted/private data |
| **Forgetting (inappropriate)** | Does the system forget what it shouldn't? | Loses important context over time |
| **Contamination** | Does memory from one user leak to another? | No cross-user information bleeding |
| **Relevance filtering** | Does it retrieve the most relevant memories? | Surfaces recent preferences over old ones |
| **Capacity** | How much can the system remember? | Test with 10, 100, 1000 past interactions |

---

## Memory Metrics

| Metric | Definition | How to Measure | Range |
|---|---|---|---|
| **Recall Accuracy** | % of relevant memories correctly retrieved | correct_retrievals / relevant_memories | 0–1 |
| **Recall Precision** | % of retrieved memories that are actually relevant | relevant_retrieved / total_retrieved | 0–1 |
| **Memory F1** | Harmonic mean of recall accuracy and precision | 2 × (P × R) / (P + R) | 0–1 |
| **Temporal Accuracy** | % of time-based queries answered correctly | correct_temporal / total_temporal | 0–1 |
| **Update Accuracy** | % of updates correctly reflected in future queries | correct_updates / total_updates | 0–1 |
| **Decay Resilience** | Performance after N interactions (memory stability) | score_at_N / score_at_1 | 0–1 |
| **Cross-Session Recall** | Can information from session 1 be recalled in session 5? | correct_cross_session / total_queries | 0–1 |
| **Contamination Rate** | % of queries returning information from wrong user/context | contaminated / total_queries | 0 = best |
| **Forgetting Compliance** | % of deleted information no longer retrievable | properly_forgotten / deletion_requests | 0–1 |

---

## Evaluation Strategies

### Strategy 1: Probing Questions

Ask specific questions about past interactions.

```
Session 1: User says "My favorite color is blue"
Session 2: User says "I work at Google"
Session 5: Ask "What's my favorite color?" → Expected: "Blue"
Session 5: Ask "Where do I work?" → Expected: "Google"
```

### Strategy 2: Conversation Continuation

Test if the system maintains context across sessions.

```
Session 1: "Help me plan a trip to Japan"
Session 3: "What were the flights we found?" → Should reference Session 1 findings
```

### Strategy 3: Contradiction Testing

Update information and verify the memory system handles it.

```
Turn 5:  "I live in San Francisco"
Turn 20: "I just moved to New York"
Turn 25: "Where do I live?" → Expected: "New York" (not San Francisco)
```

### Strategy 4: Capacity Testing

Gradually increase the amount of information and test retrieval.

| Load Level | Information Stored | Expected Recall |
|---|---|---|
| Light (10 facts) | 10 user preferences | ≥ 95% recall |
| Medium (50 facts) | 50 facts across 10 sessions | ≥ 85% recall |
| Heavy (200 facts) | 200 facts across 50 sessions | ≥ 70% recall |
| Stress (1000+ facts) | 1000+ facts across 100 sessions | Measure degradation curve |

### Strategy 5: Temporal Reasoning

Test the system's ability to reason about when things happened.

```
"When did I first mention the React project?" → "In our March 12th conversation"
"What changed since our last discussion?" → "You decided to switch from Flask to FastAPI"
```

---

## Building Memory Eval Datasets

### Dataset Structure

```json
{
  "eval_id": "mem-001",
  "sessions": [
    {
      "session_id": "s1",
      "timestamp": "2025-03-01",
      "turns": [
        {"role": "user", "content": "I'm a backend developer at Stripe"},
        {"role": "assistant", "content": "Got it! What are you working on?"},
        {"role": "user", "content": "Migrating our services to Kubernetes"}
      ]
    }
  ],
  "probe_questions": [
    {
      "question": "Where do I work?",
      "expected_answer": "Stripe",
      "memory_type": "semantic",
      "sessions_gap": 3
    }
  ]
}
```

### Test Case Categories

| Category | What It Tests | Example |
|---|---|---|
| **Basic recall** | Simple fact retrieval | "What's my name?" |
| **Temporal recall** | When something was discussed | "When did we last talk about X?" |
| **Update handling** | Information changes over time | Old preference → new preference |
| **Cross-session** | Recall across multiple sessions | Session 1 fact queried in session 5 |
| **Interference** | Similar information doesn't get confused | Two projects with similar names |
| **Forgetting** | Deleted info is actually deleted | "Forget my credit card number" → verify gone |
| **Relevance** | Most relevant memory surfaces first | Recent preference over old one |
| **Capacity limits** | Behavior under high memory load | 100+ stored facts |

---

## Common Failure Modes

| Failure Mode | Description | Impact | Detection |
|---|---|---|---|
| **Total amnesia** | System forgets everything between sessions | Breaks continuity completely | Cross-session recall test |
| **Partial recall** | Remembers some facts, forgets others | Inconsistent user experience | Comprehensive probing |
| **Hallucinated memory** | "Remembers" things that never happened | Severe trust issues | Probe for non-existent facts |
| **Cross-user contamination** | Information from User A appears for User B | Privacy violation 🔴 | Multi-user isolation test |
| **Stale memory** | Old information persists after updates | Gives outdated advice | Contradiction/update test |
| **Recency bias** | Only recalls recent interactions, forgets old ones | Loses long-term context | Test recall of early facts |
| **Irrelevant retrieval** | Surfaces unrelated past context | Confusing responses | Relevance scoring |
| **Memory hallucination** | Confabulates past interactions | User distrust | Probe for fabricated events |
| **Context window overflow** | Too much memory injected, model performance degrades | Quality drop | Test with varying memory injection sizes |

---

## Testing Patterns

### Pattern 1: Plant-and-Probe

Plant a fact, wait N turns/sessions, probe for recall.

```
PLANT:  Turn 3 → "My dog's name is Max"
WAIT:   20 turns of unrelated conversation
PROBE:  Turn 23 → "What's my dog's name?"
ASSERT: Response contains "Max"
```

### Pattern 2: Update-and-Verify

Plant a fact, update it, verify the latest version is recalled.

```
PLANT:   Turn 3 → "My phone is an iPhone 14"
UPDATE:  Turn 15 → "I just got an iPhone 16"
VERIFY:  Turn 20 → "What phone do I use?"
ASSERT:  Response mentions iPhone 16, NOT iPhone 14
```

### Pattern 3: Multi-User Isolation

Run parallel sessions with different users, verify no cross-contamination.

```
User A: "I work at Google"
User B: "I work at Meta"
User A probe: "Where do I work?" → Must say "Google", NOT "Meta"
```

### Pattern 4: Decay Curve

Measure recall accuracy as the gap between plant and probe increases.

| Gap (sessions) | Expected Recall |
|---|---|
| 1 session | ≥ 95% |
| 5 sessions | ≥ 90% |
| 20 sessions | ≥ 80% |
| 100 sessions | Measure and report |

---

## Memory vs Context Window

| Feature | Context Window | External Memory |
|---|---|---|
| **Capacity** | 4K–2M tokens (model-specific) | Unlimited (database-backed) |
| **Persistence** | Session only | Across sessions |
| **Speed** | Instant (already in context) | Requires retrieval (latency) |
| **Accuracy** | Perfect (it's right there) | Depends on retrieval quality |
| **Cost** | Increases per-request cost (more tokens) | Storage + retrieval cost |
| **Eval focus** | In-context recall, long-context handling | Retrieval quality, update handling |

---

## Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| **Only testing within a single session** | Misses cross-session memory failures | Test across multiple sessions |
| **Not testing updates/contradictions** | Stale information goes undetected | Include update + contradiction tests |
| **Ignoring privacy/contamination** | Cross-user data leaks | Test multi-user isolation |
| **No capacity testing** | Performance degrades silently under load | Test with increasing memory sizes |
| **Treating context window as memory** | Conflating two different mechanisms | Test both separately |
| **Not testing appropriate forgetting** | System can't comply with deletion requests | Include forgetting/deletion tests |

---

## Key Takeaways

| Principle | Details |
|---|---|
| **Memory is a retrieval problem** | Evaluate it like retrieval: precision, recall, relevance |
| **Test across sessions** | Single-session tests miss the most important memory failures |
| **Update handling is critical** | Information changes; memory must keep up |
| **Privacy is non-negotiable** | Cross-user contamination is a dealbreaker |
| **Measure decay** | How does recall degrade over time and distance? |
| **Hallucinated memories are dangerous** | The system must not "remember" things that never happened |

---

Move to [14. Safety Evals →](../14_safety_evals/README.md) to learn how to evaluate harmful behavior, jailbreak resistance, bias, and safety in LLM systems.
