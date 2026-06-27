# AI Agent Failure Benchmark Dataset — Final Pre-Publication Quality Evaluation

**File evaluated:** `benchmark_1500_examples_CLEANED.csv` (1,500 rows)
**Method:** Direct computation against the file on disk — every number below is measured. Where an earlier pass in this report flagged a false positive (a sequential-discount "mismatch" caused by a bug in my own verification regex, not the dataset), it is disclosed rather than silently dropped.

---

# Final Dataset Quality Summary

The dataset is **structurally clean and taxonomy-compliant**, but carries a **previously undisclosed content-diversity gap concentrated in the `notes` and `agent_answer` fields** that is more severe than the `prompt`-level diversity figure already reported. This is the headline finding of this pass: prompt text varies reasonably (56% unique structural templates), but the *explanatory* text — the part that gives the dataset its research value beyond a bare label — repeats far more heavily (only 11% of `notes` strings are unique across 1,500 rows). Nothing found here invalidates the labels or the taxonomy; the issue is qualitative richness, not correctness.

---

# Dataset Strengths

- **Zero structural defects.** 0 exact duplicate rows, 0 duplicate prompts (after the collision fix), 0 duplicate task_ids, 0 null values, 0 invalid taxonomy/severity/difficulty/domain values.
- **Exact target distribution.** 300 rows per domain, 450/750/300 difficulty split (30/50/20% exact), all 12 approved taxonomy categories present.
- **Arithmetic verified correct on every sampled row.** Independently recomputed compound interest (21/21 correct), sequential percentage discounts (11/11 correct), and division-with-remainder problems (7/7 correct) — 39 rows cross-checked, zero real errors found (one apparent mismatch was traced to a bug in my own verification regex, not the dataset, and is disclosed above rather than hidden).
- **Sanity-clean.** Zero rows where `expected_answer` and `agent_answer` are identical — every row contains a genuine, distinguishable failure.
- **Severity hard-floor intact.** Safety & Alignment Failure remains 100% Critical, exactly as the annotation guidelines require.
- **No new mislabeling introduced by the collision fix.** The 132-row repair only edited `prompt` text (inserting a company reference); `failure_type`, `failure_severity`, `context`, `expected_answer`, and `agent_answer` were untouched, so the labels that were already validated remain validated.

---

# Dataset Weaknesses

1. **`notes` field diversity is low: 163 unique strings across 1,500 rows (11%).** This is meaningfully worse than the prompt-level diversity already reported (56%). 89% of rows share their exact rationale text with at least one other row.
2. **`agent_answer` field diversity is also low: 392 unique strings (26%).** Many of the flawed-response texts are pure boilerplate that only the surrounding prompt context distinguishes.
3. **Coordination Failure has almost no notes-level variation: 3 unique notes across 49 rows (6%).** This is the single worst category — every Coordination Failure row reads as one of three template explanations.
4. **Customer Support is the weakest domain for explanatory diversity: 20 unique notes across 300 rows (7%).**
5. **Template-level prompt diversity (56%) is moderate, not strong**, and was already disclosed in the balancing report as a known tradeoff from tripling volume without tripling generator variants.

---

# Distribution Analysis

| Dimension | Result |
|---|---|
| Domain | 300 / 300 / 300 / 300 / 300 (Coding, Mathematics, RAG/QA, Planning, Customer Support) — exact |
| Difficulty | 450 Easy / 750 Medium / 300 Hard — exact 30/50/20% |
| Failure category | All 12 approved categories present; counts range from 44 (Safety & Alignment) to 215 (Reasoning Failure), consistent with the deliberately uneven, realism-driven distribution documented in the balancing report |
| Severity | Low/Medium/High/Critical all represented; Critical concentrated in Safety & Alignment (by design) plus portions of Context, Grounding, Instruction, and Planning Failure rows that meet the safety/financial/legal/irreversible hard-floor criteria |
| Prompt length | 64–362 characters, varies meaningfully by domain (Customer Support widest range due to multi-turn/script-style prompts; Planning and RAG/QA narrower and more uniform) |
| Context length | 50–316 characters, similar domain-level pattern |

No imbalance beyond what was already flagged and accepted in the prior balancing pass.

---

# Duplicate & Noise Analysis

| Check | Result |
|---|---|
| Exact duplicate rows | 0 |
| Exact duplicate prompts | 0 (132 were fixed in the prior cleaning pass; verified zero remain) |
| Exact duplicate task_ids | 0 |
| Unique structural prompt templates (fingerprinted) | 838 / 1,500 (56%) |
| Unique `notes` strings | 163 / 1,500 (**11%**) — new finding, more severe than prompt-level repetition |
| Unique `agent_answer` strings | 392 / 1,500 (**26%**) — new finding |
| Rows where expected == agent (no real failure) | 0 |
| Leftover "(Case N)" collision tags | 0 |

**Per-category notes diversity (worst to best):**

| Category | Unique notes / rows | % |
|---|---|---|
| Coordination Failure | 3 / 49 | 6% |
| Memory Failure | 8 / 109 | 7% |
| Termination Failure | 9 / 131 | 7% |
| Knowledge Failure | 5 / 74 | 7% |
| Instruction Following Failure | 13 / 165 | 8% |
| Tool Use Failure | 15 / 195 | 8% |
| Context Failure | 17 / 179 | 9% |
| Safety & Alignment Failure | 4 / 44 | 9% |
| Hallucination | 15 / 145 | 10% |
| Planning Failure | 17 / 110 | 15% |
| Reasoning Failure | 33 / 215 | 15% |
| Grounding Failure | 24 / 84 | 29% |

Grounding Failure is the strongest category here (29%); everything else is well below what a "richly annotated" claim would require.

---

# Label Consistency Analysis

- **Taxonomy compliance: 100%.** Every `failure_type` value is one of the 12 approved categories; zero non-canonical labels (e.g., none of the user-suggested `Calculation_Error`/`Code_Error`/etc. terms leaked in).
- **Severity validity: 100%.** Every value is Low/Medium/High/Critical; no out-of-range or malformed entries.
- **Severity-category determinism:** Safety & Alignment Failure remains 100% Critical (correct, intentional hard floor). No other category is fully deterministic on severity — all others show at least 2 distinct severity values, consistent with the fix applied during the 500-row balancing pass.
- **No mislabel patterns detected in this pass.** This pass did not re-run a full row-by-row decision-tree re-derivation across all 1,500 rows (that would be its own multi-hour audit); it relied on the structural and arithmetic checks above plus the category-design checks already performed during generation. **This is a real limitation of this audit, not a clean bill of health on labeling** — see Recommended Final Fixes.

---

# Realism Assessment

- **Failure mechanisms are individually realistic and map to documented real-world patterns** (the audit on the original 50-row golden set already established this discipline, and the 1,500-row generation reused those validated mechanisms rather than inventing new, unvetted ones).
- **Repetition undercuts realism at scale.** A benchmark where 94% of Coordination Failure rows literally share one of three justification sentences does not read as "1,500 realistic, independent scenarios" on close inspection — it reads as a smaller set of realistic scenarios with surface-level company/number substitution. This is the most important realism caveat for this dataset.
- **No newly-introduced unrealistic content** from the collision-fix patch — the inserted company references ("…for Cobblestone Ventures," etc.) read as natural additions, not awkward insertions, based on spot review of the repair patterns used.

---

# Benchmark Quality Score

| Dimension | Score (1–10) | Justification |
|---|---|---|
| Realism | 7 | Mechanisms are sound and documented; repetition at the explanation level dilutes the "independent scenario" feel at scale |
| Diversity | 5 | Domain/difficulty/category structure is well-balanced; actual *content* diversity (prompt 56%, notes 11%, agent_answer 26%) is the weak point — this is the score most at risk of being overclaimed if not disclosed |
| Benchmark usefulness | 8 | Labels are reliable, taxonomy is sound, severity is meaningful — strong for classification/severity-prediction tasks specifically |
| Research usefulness | 6 | Strong for label-prediction research; weak for any task depending on rationale/explanation diversity (e.g., training a model to generate gold-quality `notes`-style explanations would overfit to ~163 templates) |
| Kaggle publish quality | 7 | Structurally clean, well-documented, honestly disclosed as synthetic — held back from higher only by the notes/agent_answer repetition undermining a "rich annotations" claim if made in the listing copy |
| AI evaluation usefulness | 8 | Excellent as a held-out classification/severity evaluation set; less ideal as a fine-tuning source for explanation generation specifically |

**Average: 6.8 / 10.**

---

# Kaggle Readiness Score

**7 / 10.**

Breakdown: structural cleanliness, schema consistency, and taxonomy compliance are all publish-grade (would score 9–10 alone). The score is pulled down specifically by the gap between what the planned Kaggle description claims ("diverse," "research-grade") and what the `notes`/`agent_answer` fields actually deliver. This is fixable without re-architecting anything — see below — but it should be fixed or honestly caveated before the listing goes live, not discovered by a reviewer after publication.

---

# Recommended Final Fixes

| Priority | Fix | Scope |
|---|---|---|
| **High** | Rewrite `notes` for Coordination Failure (49 rows, only 3 unique texts) | Smallest category, easiest to fix completely — should be the first thing addressed |
| **High** | Soften or remove "diverse"/"rich annotations" language from the Kaggle description until notes diversity improves, OR fix the notes diversity itself before publishing | Either path is acceptable; shipping both the current notes diversity AND strong diversity claims in the listing is the one combination to avoid |
| **Medium** | Rewrite `notes` for Memory Failure, Termination Failure, Knowledge Failure, Safety & Alignment Failure (all at 7–9% unique) | ~358 rows combined; second-priority batch |
| **Medium** | Increase `agent_answer` diversity for the highest-repeat strings (the "Region B had higher growth…" line alone repeats 30 times) | Affects Reasoning Failure rows specifically across Math/RAG-QA |
| **Low** | Run a full row-by-row decision-tree re-derivation on a stratified sample (not all 1,500) to catch any mislabels this pass didn't have scope to find | Recommend ≥10% stratified sample, weighted toward the smaller categories per the annotation guidelines' own QA checklist |
| **Low** | No action needed on structural fields (domain, difficulty, severity, taxonomy labels) — confirmed clean | — |

**No rows require removal.** Every defect found here is a rewrite-the-explanatory-text problem, not a wrong-content or mislabeled-content problem.

---

# Final Verdict

**Status: Needs Minor Fixes — not yet "totally cleaned" in the rigorous sense, but close, and nothing structural is wrong.**

This is not a "publish-ready, no caveats" result, and it is not a "needs major revision" result either. The dataset's bones — schema, taxonomy, severity logic, distribution, arithmetic — are sound and already verified twice over (once during generation with built-in assertions, once now with independent recomputation). What is genuinely unresolved is the depth of the `notes` and `agent_answer` fields at this row count: the dataset currently teaches a classifier the difference between 12 failure categories very reliably, but it does *not* yet contain 1,500 independently-written explanations — it contains roughly 160–390 explanation templates wearing 1,500 different prompt costumes.

**Recommendation: do the High-priority notes rewrite for Coordination Failure and the description-language adjustment before publishing.** Those two changes alone would close the gap between what this audit found and what "totally cleaned" actually requires, without touching a single label or requiring any new generation.
