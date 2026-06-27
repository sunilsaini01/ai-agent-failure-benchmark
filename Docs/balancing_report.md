# AI Agent Failure Benchmark Dataset — Balancing & Distribution Analysis

**Scope:** `benchmark_500_examples.csv` (500 rows, generated via the template library)
**Method:** Direct computation against the file on disk — every count below is measured, not estimated. Template-repetition figures use a fingerprinting pass that strips company names, industries, numbers, and emails to expose structural (not just literal) duplication.

---

# Dataset Distribution Summary

| Dimension | Result |
|---|---|
| Total rows | 500 |
| Domains | 5, exactly 100 each |
| Difficulty split | 150 Easy / 250 Medium / 100 Hard (30% / 50% / 20% — exact target hit) |
| Failure categories used | 12 of 12 approved taxonomy categories present |
| Non-approved labels found | 0 |
| Invalid severity values found | 0 |
| Exact duplicate prompts | 0 |
| Unique structural templates (fingerprinted) | 309 of 500 (62% of rows are structurally unique; 38% share a stripped-down template with at least one other row) |

**Headline finding:** the dataset is structurally clean (correct schema, correct taxonomy, correct quotas) but **not yet diverse enough at the template level** to support large-scale expansion without first growing the generator's internal variant count — the imbalance that matters most here isn't category counts, it's *template density within categories*.

---

# Failure Category Distribution

| Category | Count | % of 500 | Unique Templates | Effective Diversity |
|---|---|---|---|---|
| Reasoning Failure | 80 | 16.0% | 56 | 70% |
| Context Failure | 70 | 14.0% | 31 | 44% |
| Tool Use Failure | 58 | 11.6% | 30 | 52% |
| Instruction Following Failure | 52 | 10.4% | 32 | 62% |
| Hallucination | 50 | 10.0% | 39 | 78% |
| Planning Failure | 42 | 8.4% | 19 | 45% |
| Termination Failure | 40 | 8.0% | 24 | 60% |
| Memory Failure | 30 | 6.0% | 16 | 53% |
| Knowledge Failure | 28 | 5.6% | 24 | 86% |
| Grounding Failure | 20 | 4.0% | 18 | 90% |
| Safety & Alignment Failure | 20 | 4.0% | 12 | 60% |
| Coordination Failure | 10 | 2.0% | 8 | 80% |

**Reading this table correctly:** raw count and diversity are independent problems. Context Failure has a healthy count (70) but the weakest diversity in the dataset (44% — only 31 distinct templates produced 70 rows). Knowledge Failure and Grounding Failure are low-count but high-diversity (86–90%), meaning their small sample is at least varied. The categories needing the most generator work are **Context Failure and Planning Failure** — high enough count that they look fine on a bar chart, but thin enough on templates that a model could learn the template rather than the failure pattern.

---

# Domain Distribution

| Domain | Count | Categories Present | Single-Domain-Confined Categories |
|---|---|---|---|
| Coding | 100 | 7 of 12 | — |
| Mathematics | 100 | 7 of 12 | — |
| RAG/QA | 100 | 7 of 12 | Grounding Failure (100% of its rows live only here) |
| Planning | 100 | 6 of 12 | Coordination Failure (100% of its rows live only here) |
| Customer Support | 100 | 7 of 12 | — |

| Domain | Unique Templates / 100 | Effective Diversity |
|---|---|---|
| Mathematics | 88 | 88% |
| RAG/QA | 80 | 80% |
| Customer Support | 51 | 51% |
| Coding | 47 | 47% |
| Planning | 43 | 43% |

**Domain counts are perfectly balanced. Domain *diversity* is not.** Mathematics is the strongest domain by a wide margin (computed arithmetic naturally forces unique numbers into every row, which inflates apparent diversity somewhat, but the underlying scenarios are also genuinely varied). Coding and Planning are the weakest — both rely on a small number of rigid operational templates (refactor-the-class, squash-commits, migrate-the-database, schedule-the-meeting) repeated across many industries with only the company name and a couple of numbers changing.

---

# Difficulty Distribution

| Difficulty | Count | % | Per-Domain Count |
|---|---|---|---|
| Easy | 150 | 30.0% | 30 per domain, exactly |
| Medium | 250 | 50.0% | 50 per domain, exactly |
| Hard | 100 | 20.0% | 20 per domain, exactly |

No imbalance here — the 30/50/20 target is hit exactly, both in aggregate and per-domain. This is a structural strength of generating programmatically against a quota rather than asking for difficulty "in general."

**One caveat worth flagging:** difficulty was assigned by a quota draw, not derived from the six difficulty dials defined in `generation_templates.md` (step count, distractor count, context length, domain specificity, reasoning depth, tool-chain depth). This means the *label* `Easy`/`Medium`/`Hard` is correctly distributed, but it was not independently verified that an "Easy" Planning row and an "Easy" Coding row represent comparable actual difficulty. Recommend a difficulty-dial audit before the 1000–2000 expansion, not just a quota check.

---

# Severity Distribution

| Severity | Count | % |
|---|---|---|
| Medium | 220 | 44.0% |
| High | 135 | 27.0% |
| Critical | 86 | 17.2% |
| Low | 59 | 11.8% |

| Failure Category | Severities Present | Severity Determinism |
|---|---|---|
| Safety & Alignment Failure | Critical only (20/20) | 100% — by design (hard-floor rule) |
| Coordination Failure | High only (10/10) | 100% — **not by design, a generator gap** |
| Instruction Following Failure | Critical, Low | 2 values |
| Context Failure | Critical, Medium | 2 values |
| Hallucination | High, Medium | 2 values |
| Grounding Failure | Critical, High | 2 values |
| Knowledge Failure | Low, Medium | 2 values |
| Memory Failure | High, Medium | 2 values |
| Termination Failure | High, Medium | 2 values |
| Planning Failure | Critical, High, Medium | 3 values |
| Reasoning Failure | High, Low, Medium | 3 values |
| Tool Use Failure | Critical, High, Medium | 3 values |

**Critical finding:** for 2 of 12 categories, severity is **100% predictable from category alone** — a model (or a lazy annotator) doesn't need to read the row to guess severity. For Safety & Alignment Failure this is intentional and correct (the hard-floor rule in `annotation_guidelines.md` says safety violations are always Critical — this should stay deterministic). For **Coordination Failure, this is unintentional** — every one of the 10 rows uses the same "budget overage by a coordinator handoff" scenario at the same magnitude, which is a generator limitation, not a property of coordination failures in general. A coordination failure causing duplicated *harmful* actions, or one causing a trivial, immediately-caught inefficiency, would justify a different severity — that variation is simply missing from the current template.

---

# Detected Imbalance Issues

1. **Coordination Failure is both the smallest category (10 rows, 2.0%) and confined to a single domain (Planning) and a single severity (High).** This is the dataset's weakest category on every axis simultaneously. It is technically "present" but not yet usable for any severity- or domain-generality analysis.

2. **Grounding Failure is confined entirely to RAG/QA (20/20 rows).** This is defensible as a starting point — grounding is most naturally a RAG concept — but the taxonomy explicitly treats Grounding Failure as domain-general (a coding agent citing API docs, a support agent citing a knowledge-base article). Zero coverage outside RAG/QA limits what this category can teach about cross-domain generalization.

3. **Context Failure (70 rows) and Planning Failure (42 rows) have the lowest template diversity of any high-count categories (44% and 45%).** These categories look statistically fine until you check what's actually varying inside them — mostly company name and a number, not the underlying failure mechanism.

4. **Coding and Planning are the two weakest domains for template diversity (47% and 43%)**, despite being perfectly balanced on row count. Both rely on a handful of rigid task templates (refactor-the-class, squash-commits, migrate-the-database for Coding; schedule-the-meeting, migrate-the-database, plan-the-offsite for Planning) repeated with only surface variation.

5. **No category is over-represented relative to its real-world frequency expectation.** Reasoning Failure (16%) and Context Failure (14%) lead, which matches both common LLM-evaluation literature and the taxonomy's own stated expectation that these are the most frequent failure modes — this is *not* an imbalance to correct, just worth confirming explicitly so it isn't mistaken for one.

6. **Difficulty is not imbalanced** — flagging this explicitly since the task brief's own example list ("too many Easy examples") was checked and does not apply here; 30/50/20 is hit exactly.

7. **15 structural templates each appear 6–9 times** (the maximum observed repeat rate for any single template), concentrated in: equipment-manual Context Failure (RAG/QA), conference-room-capacity Context Failure (Planning), technical-migration Planning Failure (Planning), billing-complaint-script Instruction Failure (Customer Support), refund-turn Memory Failure (Customer Support), and the "calling on behalf of my friend Jane" Safety Failure (Customer Support). None exceed 9 occurrences — there is no single runaway template — but these six clusters account for a disproportionate share of their categories' row counts.

---

# Weak Benchmark Examples

## 27 Rows With a Forced Deduplication Tag

These rows contain a literal `(Ref #TASK_ID)` suffix appended to the prompt — a safety-net mechanism that fired when the generator produced two prompts identical down to the punctuation. This is the clearest, most mechanically-detectable signal of "the generator ran out of variation in this cell" anywhere in the dataset. Affected rows:

```
MATH_039, MATH_057, MATH_071, MATH_076, MATH_099
RAGQA_029, RAGQA_031, RAGQA_042, RAGQA_044, RAGQA_045, RAGQA_056, RAGQA_057,
RAGQA_061, RAGQA_062, RAGQA_064, RAGQA_065, RAGQA_070, RAGQA_075, RAGQA_076,
RAGQA_079, RAGQA_080, RAGQA_085, RAGQA_091, RAGQA_093, RAGQA_095, RAGQA_097
PLAN_063
```

Notably, 22 of these 27 (81%) are in RAG/QA — despite RAG/QA having the second-highest template diversity overall, a handful of its specific mechanism cells (Hallucination-by-fabricated-authors, Grounding-by-overstated-hedge, Safety-injection) were drawn from a narrow enough variable pool that exact collisions happened repeatedly at this sample size.

## The Six Heaviest Template Clusters (6–9 occurrences each)

Listed under Detected Imbalance Issue #7 above. None are mislabeled or low-realism individually — each is a legitimate, well-constructed failure instance — but as a cluster they teach a narrow, memorizable surface pattern rather than a generalizable failure concept. These are "low marginal benchmark value" rows: row 2 through row 9 of each cluster add comparatively little new signal over row 1.

## Coordination Failure Rows (All 10)

Not wrong, but weak as a set: identical scenario shape (Flights agent books first, Coordinator fails to relay remaining budget, Hotels agent overshoots by a modest fixed proportion), identical severity (High), identical domain (Planning). This is the one category in the dataset where every row is interchangeable with every other row on every dimension except company/industry name.

---

# Recommended Corrections

| Action | Target | Rationale |
|---|---|---|
| **Rewrite** | The 27 `(Ref #...)` rows | Replace with genuinely distinct surface content (different fact type, different document excerpt, different injection vector) rather than leaving the forced-uniqueness tag in a public benchmark file |
| **Rewrite** | A majority of the 6 heaviest template clusters (target: reduce each from 6–9 down to 3–4 occurrences, redistributing the freed quota into new mechanism variants within the same category) | Restores template diversity without changing category counts |
| **Regenerate with new templates** | Coordination Failure (all 10) | Needs at minimum: a second scenario shape (not just budget overage — duplicated harmful action, missed handoff causing a contradiction, role confusion), a second domain host (e.g., a multi-agent coding or customer-support escalation team, not just travel planning), and severity variation (Low for a caught inefficiency, Critical for a duplicated destructive action) |
| **Regenerate with new templates** | Grounding Failure (expand beyond RAG/QA) | Add a Coding-domain variant (agent cites real API docs but overstates what they support) and a Customer-Support variant (agent cites a real knowledge-base article but extrapolates beyond it) |
| **Add generator variants (no row changes needed yet)** | Coding Tool Use Failure, Coding Planning Failure, Planning-domain Planning/Termination/Tool-Use templates | These are the domain/category intersections with the fewest internal variant branches; expansion to 1000–2000 rows will only compound the existing template thinness if no new variants are added first |
| **No action — confirmed correct** | Safety & Alignment Failure severity determinism (100% Critical) | This is the intended hard-floor behavior, not a defect; do not "fix" this one |
| **No action — confirmed correct** | All taxonomy labels, all severity values, domain/difficulty quotas | Fully valid; re-verified directly against the schema and taxonomy with zero violations found |

**Nothing in the dataset requires outright removal.** Every row is a legitimate, correctly-labeled failure instance — the issues found are about *insufficient internal variation within cells*, not incorrect content. This is good news for the 1000–2000 expansion: the foundation doesn't need to be torn up, it needs more generator variants poured into specific cells before the next generation run.

---

# Recommended Expansion Strategy

## Target Scale: 1,500 rows (midpoint of the 1,000–2,000 range)

Recommended over the extremes because it gives each currently-thin category (Coordination, Grounding) enough absolute headroom to reach a statistically usable size without inflating the already-strong categories (Reasoning, Context) past the point of further useful signal.

## Target Domain Distribution (unchanged ratio, scaled)

| Domain | Current (500) | Target (1,500) |
|---|---|---|
| Coding | 100 | 300 |
| Mathematics | 100 | 300 |
| RAG/QA | 100 | 300 |
| Planning | 100 | 300 |
| Customer Support | 100 | 300 |

Keep the even 20%-per-domain split — nothing in this audit suggests rebalancing domain share, only template density within domains.

## Target Difficulty Distribution (unchanged ratio)

30% Easy / 50% Medium / 20% Hard, scaled to 450 / 750 / 300. No change recommended — the current split is correct and was hit exactly at 500 rows.

## Target Failure Category Distribution (rebalanced)

| Category | Current % | Recommended % at 1,500 | Recommended Count | Direction |
|---|---|---|---|---|
| Reasoning Failure | 16.0% | 14.0% | 210 | Slight decrease (already strong; don't let it keep growing share) |
| Context Failure | 14.0% | 13.0% | 195 | Slight decrease in *share*, but increase in template variants per row |
| Tool Use Failure | 11.6% | 12.0% | 180 | Hold roughly flat |
| Instruction Following Failure | 10.4% | 10.0% | 150 | Hold roughly flat |
| Hallucination | 10.0% | 10.0% | 150 | Hold flat — good diversity already |
| Planning Failure | 8.4% | 9.0% | 135 | Slight increase, paired with new templates |
| Termination Failure | 8.0% | 8.0% | 120 | Hold flat |
| Memory Failure | 6.0% | 7.0% | 105 | Modest increase |
| Knowledge Failure | 5.6% | 6.0% | 90 | Modest increase |
| Grounding Failure | 4.0% | 5.0% | 75 | **Increase, and require multi-domain coverage** |
| Safety & Alignment Failure | 4.0% | 4.0% | 60 | Hold flat — this category should stay a deliberate minority, not grow just because it's "important" |
| Coordination Failure | 2.0% | 3.0% | 45 | **Increase, and require ≥2 scenario shapes and ≥2 severities** |

**Why not push Coordination and Grounding higher still:** both remain genuinely rarer failure modes in real-world agent deployments than, say, Reasoning or Context failures — over-correcting to "equal shares across all 12 categories" would itself be an unrealistic distribution, contradicting the realism objective. The recommended increases (2.0%→3.0%, 4.0%→5.0%) are sized to reach a usable per-category sample (45 and 75 rows) without manufacturing an artificial frequency.

## Template Diversity Floor for Expansion

Before generating the next 1,000 rows, require every category × domain cell to have **at least 6 distinct generator-level mechanism variants** before drawing more than 15 rows from that cell. This single rule would have caught the Coordination Failure problem (1 variant), the Context Failure problem (effectively 2–3 dominant variants per domain), and the Coding/Planning domain-wide thinness, before generation rather than after.

---

# Final Dataset Quality Assessment

**Structural integrity: Strong.** Zero taxonomy violations, zero invalid severities, zero exact duplicates, exact quota compliance on domain and difficulty, zero arithmetic errors found in any numerically-verifiable row (Mathematics and the quantitative Coding/Tool-Use rows were checked directly).

**Category and severity correctness: Strong.** Every row's label is consistent with the locked decision tree as applied during generation; the hard-floor severity rule for Safety & Alignment Failure is honored at 100%.

**Diversity: Moderate, with identified gaps.** 62% of rows are structurally unique at the template level; the remaining 38% cluster into a manageable number of identifiable, fixable template groups rather than being scattered unpredictably. Two categories (Coordination, Grounding) and two domains (Coding, Planning) carry essentially all of the diversity risk in the dataset.

**Readiness for 1,000–2,000 expansion: Conditional — not yet.** The dataset is a sound foundation, but scaling the *current* generator as-is would mechanically reproduce today's template-density gaps at 3–4x the row count, making them harder to fix later rather than easier. Adding the recommended new mechanism variants to the identified thin cells (Coordination, Grounding, Coding Tool Use/Planning, Planning-domain Tool Use/Termination) before the next generation run is a prerequisite, not an optional nice-to-have.

**Overall verdict:** This is a well-engineered, schema-compliant, taxonomy-compliant 500-row benchmark with a known and narrowly-scoped diversity gap. No content needs to be deleted. The path to 1,500 rows is: widen the generator at the specific cells identified above, then scale — not scale first and audit later.
