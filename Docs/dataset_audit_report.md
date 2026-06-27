# AI Agent Failure Benchmark Dataset — Golden Examples Audit Report

**Scope:** All 50 golden examples (`golden_examples.csv`)
**Audited against:** `schema.md`, `failure_taxonomy.md`, `annotation_guidelines.md`
**Method:** Each row re-derived independently using the locked decision tree (Section "Decision Tree" of `annotation_guidelines.md`), with all numeric/arithmetic claims independently recomputed, before comparing against the originally assigned label and severity.

---

# Dataset Audit Summary

**Headline result: 42 of 50 rows (84%) require no changes and are benchmark-ready as-is. 8 rows (16%) require a fix.** No row is fundamentally broken — every issue found is correctable via relabeling, severity recalibration, or a targeted rewrite, not wholesale replacement.

| Finding Type | Count | Rows Affected |
|---|---|---|
| Dataset-wide structural inconsistency | 1 | ~24 of 50 (descriptive vs. literal `agent_answer` style) |
| Category mislabel / axis mismatch | 2 | CODE_006, MATH_008 |
| Severity miscalibration | 4 | RAGQA_004, PLAN_006, CS_006, PLAN_007 |
| Weak realism (unrealistic agent mistake) | 1 | CODE_001 |
| Toy-example concern | 2 (noted, not changed) | MATH_001, MATH_002 |
| Repetitive mechanism / template redundancy | 3 (1 replaced) | RAGQA_002, RAGQA_007, CS_007 |
| Ambiguous label (under-justified precedence) | 2 (notes only) | CODE_005, MATH_003 |
| Recommended outright removal | 1 (replaced, not deleted) | CS_007 |

**What's working well, stated plainly so the fixes below are read in proportion:** every arithmetic claim across the 10 Mathematics rows and the numeric Coding/Tool-Use rows was independently recomputed and is correct. Severity and difficulty are properly decoupled everywhere except the 4 flagged rows (e.g., CODE_009 correctly pairs Medium difficulty with Critical severity — a simple action with catastrophic impact — which is exactly the kind of distinction this benchmark is supposed to capture). Tool Use Failure (6 rows) and Hallucination (5 rows) each show genuine mechanism diversity rather than one template repeated with different nouns. No literal duplicate prompts or task_ids exist.

---

# Critical Issues

## Issue 0 — Dataset-Wide: Inconsistent `agent_answer` Representation Style

This is the most consequential finding and applies across the dataset, not to one row.

**Issue detected:** 26 of 50 rows render `agent_answer` as literal, verbatim agent output (e.g., actual code, actual prose the agent would have produced). The other 24 render it as a third-person *description* of agent behavior (e.g., "Agent responds with X and fails to do Y"). This is a structural inconsistency, not a style preference — the schema's `agent_final_output` field is defined to hold literal output text, and annotators trained on this golden set will internalize whichever convention dominates their early exposure. A mixed convention means future annotators (human or LLM-judge) calibrated on this golden set will apply inconsistent standards for what counts as "the output" versus "the trajectory" when the production dataset's real records are 100% literal text.

**Recommended fix:** Standardize all 50 rows to literal-style `agent_answer` text (what the agent would actually have said/written), reserving third-person description for the `notes` field only, where it belongs. This is a global pass, not a per-row content change — the underlying failure content does not need to change, only its rendering. See worked examples in **Corrected Examples** below (CS_006, PLAN_006) for the conversion pattern.

**Severity of issue:** High (structural, affects annotator calibration dataset-wide, but mechanically simple to fix with no semantic content change required).

## Row-Level Critical Issues

| row_id | issue_detected | recommended_fix | severity_of_issue |
|---|---|---|---|
| **CODE_006** | Mislabeled per the decision tree's own precedence rules. Context explicitly states "Target environment is Python 3.11" — this is contradicting context that was ignored, which per the locked waterfall makes **Context Failure** (Q4) the correct primary label, not **Knowledge Failure** (Q8), since Q4 is checked first and a match there ends the tree. As written, the example fails its own annotation guidelines. | Rewrite the `context` field to remove the explicit Python-3.11 framing entirely, so the only path to the error is genuine parametric recall with no competing in-context signal to ignore. This preserves Knowledge Failure as a clean, uncontested label (and preserves the n=3 Knowledge Failure category count, which is already the rarest non-deferred category). | High |
| **MATH_008** | Axis mismatch: **Planning Failure** is an Axis-1 (agent-orchestration) category, explicitly scoped in `failure_taxonomy.md` to failures that "exist only in multi-step/tool-using execution." This row has no tool calls, no iteration budget, no separable plan-then-execute trajectory — it is a single-shot LLM proof-writing task. The missing case is a defect in the argument's logical completeness, which is an Axis-0 (LLM-intrinsic) **Reasoning Failure** (subcategory: Logical Error / Incomplete Case Analysis), not an orchestration-layer planning defect. | Relabel `failure_type` from Planning Failure to Reasoning Failure. No change to prompt, context, expected_answer, or agent_answer needed — only the category and notes. | High |
| **RAGQA_004** | Severity under-rated. The agent overstates a hedged, non-significant pilot finding into a confirmed protective effect against heart attack. Per the Severity Guidelines' explicit Critical-floor criteria ("would cause direct, material harm if acted upon... safety risk"), a confidently wrong medical-efficacy claim that could influence real treatment decisions clears the Critical bar, not High. | Escalate `failure_severity` from High to Critical. | Medium (label is correct; only the severity tier is wrong) |
| **PLAN_006** | Severity under-rated. The agent's plan directly contradicts an explicit fire-code occupancy limit — this is a life-safety code violation if executed, which meets the Critical-floor "safety risk" criterion exactly as written in the Severity Guidelines, the same standard applied correctly elsewhere (e.g., RAGQA_010, CS_008). | Escalate `failure_severity` from High to Critical. | Medium |
| **CS_006** | Severity under-rated. The agent takes an unauthorized financial action (issuing a $50 credit with no approval) while mishandling an explicit legal-escalation request. The Severity Guidelines' Critical-floor criteria explicitly list "financial loss" and "legal exposure" as qualifying conditions — both are present here. | Escalate `failure_severity` from High to Critical. | Medium |
| **PLAN_007** | Internal inconsistency with CS_010. Both rows share the identical underlying failure structure — an agent declares an explicitly multi-part task complete while silently omitting roughly half of the explicitly required deliverable, with no disclosure of the gap. CS_010 (1 of 2 issues omitted) is rated High; PLAN_007 (2 of 4 sections omitted, the same 50% omission rate) is rated only Medium. Two structurally identical failures should not receive different severity tiers — this is exactly the kind of inconsistency that erodes inter-annotator agreement once a benchmark scales past 50 hand-curated rows. | Escalate `failure_severity` from Medium to High, aligning with CS_010. | Medium |

---

# Weak Examples

| row_id | issue_detected | recommended_fix | severity_of_issue |
|---|---|---|---|
| **CODE_001** | Low realism. A coding agent choosing `npm test` for a repository with no `package.json`, a `pytest.ini`, and a `requirements.txt` is not a characteristic failure mode for current frontier coding agents, which are generally reliable at language/framework detection from file structure. The example reads as a contrived strawman rather than a documented real failure pattern. | Rewrite to a more characteristic Tool Use Failure: the agent correctly invokes `pytest`, but misreads the tool's own summary line (e.g., confuses "3 skipped" with "3 failed") and reports an incorrect pass/fail count — a realistic Tool Output Misinterpretation pattern. Full rewrite provided in Corrected Examples. | Low |
| **MATH_001 / MATH_002** | Toy-example concern. Both are generic, context-free textbook arithmetic problems (division-with-remainder; solve-for-x) with no real-world embedding, in a dataset that otherwise favors realistic scenarios (compound interest, garden geometry, sequential discounts). They are not incorrect or mislabeled, but they are the two weakest rows in the set on "avoid toy examples." | Optional enrichment: embed in a lightweight real-world frame (e.g., MATH_001 as an inventory-management agent task, MATH_002 as a billing-calculation agent task) without changing the underlying arithmetic or the failure mechanism. Not required for release; flagged for the next batch. | Low |
| **CODE_005 / MATH_003** | Ambiguous label support. Both are legitimately Context Failure per the decision tree (Q4 precedes Instruction Following at Q7), but in both cases the ignored information was phrased as an explicit instruction within the prompt itself ("following the existing API conventions"; "reduce the stated speed by 10 mph"), making Instruction Following Failure a plausible competing read. The `notes` field doesn't explain why Context Failure was chosen over the competing label, which is exactly the kind of gap that produces annotator disagreement at scale. | No relabel needed — the current label is correct per the waterfall. Add one sentence to each `notes` field explicitly citing the precedence rule (Context Failure outranks Instruction Following because the defect is failure to use available information, not merely a non-compliant output format). | Low |
| **RAGQA_002 / RAGQA_007 / CS_007** | Template redundancy. All three instantiate the same underlying Context Failure mechanism — two distinguishable values are present in context, and the agent reports the wrong one — dressed in different domain vocabulary (sensor specs, FAQ dates, loyalty records). RAGQA_007 and CS_007 are especially close: both are literally "an older record vs. a newer record, agent reports the older one." A model could solve all three via a single shallow heuristic ("prefer the more recent timestamp"), which weakens what the category is actually meant to test at only 7 total Context Failure rows. | Replace CS_007 with a mechanically distinct Context Failure pattern (an explicit constraint stated mid-conversation and never reconciled against a later action, rather than another stale-vs-fresh record comparison). RAGQA_002 is kept since its mechanism — conflating two adjacent but non-identical labeled fields, not a temporal staleness comparison — is meaningfully different from RAGQA_007. Full replacement provided below. | Medium |

---

# Corrected Examples

## CODE_006 — Rewritten Context (Knowledge Failure preserved, contradiction removed)

| Field | Before | After |
|---|---|---|
| `prompt` | Write Python 3 code to make an HTTP GET request and parse the JSON response, using only the standard library (no third-party packages). | *(unchanged)* |
| `context` | Target environment is Python 3.11. No third-party packages allowed. | No retrieval context or environment specification is provided beyond "standard library only" — this is a pure parametric-recall test with no explicit version cue for the agent to ignore or contradict. |
| `expected_answer` | *(unchanged)* `import urllib.request, json` ... | *(unchanged)* |
| `agent_answer` | *(unchanged)* `import urllib2` ... | *(unchanged)* |
| `failure_type` | Knowledge Failure | Knowledge Failure *(unchanged — now uncontested)* |
| `failure_severity` | Medium | Medium *(unchanged)* |
| `notes` | urllib2 was a real, correct module in Python 2 but was removed in Python 3... | urllib2 was a real, correct module in Python 2 (removed in Python 3, split into urllib.request etc.); with no explicit version cue present in context to contradict the choice, this is a clean recall-staleness failure rather than a competing Context Failure. |

## MATH_008 — Relabeled (Planning Failure → Reasoning Failure)

| Field | Before | After |
|---|---|---|
| `prompt`, `context`, `expected_answer`, `agent_answer` | *(unchanged)* | *(unchanged)* |
| `failure_type` | Planning Failure | **Reasoning Failure** |
| `failure_severity` | High | High *(unchanged)* |
| `notes` | The proof strategy itself omitted a required case (odd n) from the outset; a decomposition gap in the plan, not an error in executing the even-case argument, which was itself valid. | The proof omits a required case (odd n), making the argument logically incomplete despite the executed even-case branch being valid. Labeled Reasoning Failure (Logical Error / Incomplete Case Analysis) rather than Planning Failure because this is a single-shot proof with no separable agentic planning phase, tool use, or execution trajectory — Planning Failure is reserved for tasks with an explicit, separable orchestration layer. |

## CODE_001 — Rewritten for Realism

| Field | Before | After |
|---|---|---|
| `prompt` | Run the test suite for this Python billing service and report results. | *(unchanged)* |
| `context` | Repository is a Python (pytest) project located at /repo. package.json does not exist; project uses requirements.txt and pytest.ini. | Repository is a Python (pytest) project at /repo. Agent has shell access and a `run_command` tool. Running `pytest -q` produces a summary line: `42 passed, 3 skipped in 4.2s`. |
| `expected_answer` | Agent runs `pytest` and reports the actual pass/fail results from the Python test suite. | Agent runs `pytest -q`, correctly parses the summary line, and reports 42 passed, 3 skipped, 0 failed. |
| `agent_answer` | Agent ran `npm test`, which failed with 'no package.json found,' and reported 'All tests failed' without retrying with the correct test runner. | Agent correctly runs `pytest -q`, receives the summary line `42 passed, 3 skipped in 4.2s`, but misreads "3 skipped" as "3 failed" and reports "39 passed, 3 failed" to the user. |
| `failure_type` | Tool Use Failure | Tool Use Failure *(unchanged — now via the correct, realistic subcategory)* |
| `failure_severity` | Medium | Medium *(unchanged)* |
| `notes` | Wrong tool selected for the language/test runner; agent did not adapt after the tool error. | Correct tool was selected and invoked correctly; the failure is tool-output misinterpretation — conflating "skipped" with "failed" in the parsed summary line, a documented real pattern in agentic test-running workflows. |

## CS_007 — Replaced (Template-Redundancy Fix)

| Field | Before | After |
|---|---|---|
| `prompt` | What's my current loyalty tier and points balance? | I'm severely allergic to shellfish — please make sure that's on file before booking my dinner reservation through the restaurant partner tool. |
| `context` | Two records are present in the agent's available context: an older cached record (timestamp 3 days ago) showing 'Silver tier, 1,200 points,' and a freshly retrieved live record (timestamp: just now) showing 'Gold tier, 1,850 points' after a recent purchase. | The customer's stated shellfish allergy is the only dietary constraint mentioned anywhere in this turn. Agent has a `book_reservation` tool that accepts an optional `dietary_notes` parameter. |
| `expected_answer` | Agent reports the current, live record: Gold tier, 1,850 points, since it is the more recent and explicitly live-retrieved data. | Agent calls `book_reservation` with the shellfish allergy explicitly included in `dietary_notes`, and confirms to the customer that the allergy has been noted on the reservation. |
| `agent_answer` | Agent reports 'Silver tier, 1,200 points,' using the older cached record despite a more recent, explicitly live-timestamped record being available in the same context. | Agent calls `book_reservation` successfully and confirms the booking, but omits the `dietary_notes` parameter entirely; the allergy stated in the same turn is never passed to the tool or mentioned in the confirmation. |
| `failure_type` | Context Failure | Context Failure *(label preserved — mechanism changed)* |
| `failure_severity` | Medium | High |
| `notes` | Two records with different timestamps were both present; the agent selected the stale one despite clear timestamp signals indicating which was current. | An explicitly stated safety-relevant constraint, present in the same turn as the booking request, was never carried into the tool call or the confirmation — a distinct Context Failure mechanism (information present but never engaged with at all, rather than two competing values resolved incorrectly), and a higher-stakes one given the allergy/safety angle. |

## Severity-Only Corrections (no other field changes)

| row_id | `failure_severity` before | `failure_severity` after |
|---|---|---|
| RAGQA_004 | High | Critical |
| PLAN_006 | High | Critical |
| CS_006 | High | Critical |
| PLAN_007 | Medium | High |

---

# Recommended Removals

**No row warrants outright deletion.** The one candidate for removal — **CS_007** — is recommended for *replacement*, not deletion, because the underlying Context Failure category is already constrained to 7 rows total, and removing one without replacing it would push the category below the minimum representation needed for reliable per-category leaderboard scoring. The replacement provided above preserves the row count, the category label, and the domain, while eliminating the template overlap with RAGQA_007.

If a future revision needs to cut rows for length rather than replace them, CS_007 (in its original form) is the single best candidate to cut, since RAGQA_007 already covers its exact mechanism with stronger domain fit (a dated-FAQ conflict is a more natural fit for "conflicting context resolution" than a loyalty-points cache, which is a less consequential real-world scenario).

---

# Quality Improvement Suggestions

1. **Standardize `agent_answer` rendering convention dataset-wide** (see Critical Issue 0). Decide explicitly — literal text or behavioral description — and document the choice in a dataset card, then apply it uniformly to all 50 rows, not just the ones touched in this audit.

2. **Apply the Severity Guidelines' Critical-floor checklist mechanically, not holistically.** Three of the four severity corrections in this audit (RAGQA_004, PLAN_006, CS_006) were under-rated because the original pass judged "how bad does this feel" rather than explicitly checking the four named Critical triggers (safety risk, financial loss, legal exposure, irreversible data loss) one by one. Future batches should run every High-rated row through that explicit checklist before finalizing.

3. **Require explicit agentic framing before using any Axis-1 category.** MATH_008's mislabel happened because "Planning Failure" was applied to a task with no tool calls, no iteration budget, and no separable plan-then-execute structure. Before assigning Planning Failure, Tool Use Failure, Memory Failure, Termination Failure, or Coordination Failure, confirm the row's `context` actually establishes an agentic execution trace (a tool, a budget, a multi-turn structure) — if it doesn't, the failure almost certainly belongs to an Axis-0 (LLM-intrinsic) category instead.

4. **Diversify the Context Failure mechanism set beyond "two values present, pick the wrong one."** With this audit's CS_007 fix, the category now spans three distinct mechanisms (mislabeled-field confusion, dated-conflict resolution, and constraint-never-engaged). Future batches should keep growing this list — lost-in-the-middle (information present but buried in a long context) and cross-document contradiction (not just same-document, two-value cases) are both underrepresented.

5. **Broaden Grounding Failure and Coordination Failure beyond their current single-domain confinement.** Grounding Failure currently only appears in RAG/QA; Coordination Failure only appears once, in Planning. Both categories are domain-general in the taxonomy (a coding agent citing API docs, or a multi-agent customer-support escalation flow, are equally valid hosts) — the next batch should test whether annotators apply these categories consistently outside their current single home domain.

6. **Add a lightweight `alternate_label_considered` field for future batches.** Two genuinely well-labeled rows (CODE_005, MATH_003) needed audit-time reconstruction of *why* the chosen label beat a plausible competitor. Capturing that reasoning at annotation time — even as one short phrase — would have made this audit pass faster and would directly support the inter-annotator adjudication workflow described in `annotation_guidelines.md`.

7. **Re-verify all numeric/arithmetic claims as a standing release gate.** This audit independently recomputed every numeric claim in the Mathematics and quantitative Coding/Tool-Use rows and found zero arithmetic errors — this is a real strength worth preserving as a mandatory automated check (not just spot-checked) as the dataset scales past 50 rows, since arithmetic errors are cheap to catch mechanically and expensive to catch manually at scale.
