# Changelog

All notable changes to this dataset are documented in this file.

Versioning follows a **MAJOR.MINOR** convention adapted for datasets rather than software:
- **MAJOR** — row count changes, taxonomy changes, or schema/column changes.
- **MINOR** — corrections to existing rows (relabeling, severity fixes, rewritten low-quality rows) without changing row count or schema. Affected `task_id`s are always listed explicitly below, not just described in aggregate.

---

## [Unreleased]

Planned for v1.1.0 (see [`docs/final_quality_evaluation.md`](docs/final_quality_evaluation.md) for full rationale):
- Rewrite `notes` field for Coordination Failure rows (currently 3 unique rationales across 49 rows).
- Rewrite `notes` field for Memory Failure, Termination Failure, Knowledge Failure, and Safety & Alignment Failure rows (currently 7–9% unique).
- Increase `agent_answer` diversity for the highest-repeat strings, concentrated in Reasoning Failure rows.

---

## [1.0.0] — 2026-06-27

### Added
- Initial public release: 1,500 rows across 5 domains (Coding, Mathematics, RAG/QA, Planning, Customer Support), 300 rows each.
- 12 locked failure-taxonomy categories, 4 severity levels, exact 30/50/20 Easy/Medium/Hard difficulty split.
- Full methodology documentation: `schema.md`, `failure_taxonomy.md`, `annotation_guidelines.md`, `generation_templates.md`, `dataset_audit_report.md`, `balancing_report.md`, `final_quality_evaluation.md`.
- 50-row hand-audited golden reference subset (`golden_examples/golden_examples_50.csv`).

### Fixed (pre-release)
- Corrected 132 rows that had a literal forced-deduplication tag (`"(Case N)"`) left in the `prompt` field by the generation pipeline's collision-safety-net. Each was repaired with genuine content variation (a natural company-name insertion) rather than the placeholder tag, with zero exact duplicate prompts remaining afterward.
- Re-verified arithmetic on a sampled set of Mathematics rows (compound interest, sequential percentage discounts, division-with-remainder) — zero errors found.
- Confirmed zero rows where `expected_answer` and `agent_answer` are identical (every row contains a genuine, distinguishable failure).
- Confirmed zero non-approved `failure_type` values and zero invalid `failure_severity` values.

### Known issues at release (not blocking, tracked for v1.1.0)
- `notes` field diversity is ~11% unique (163 distinct strings across 1,500 rows); `agent_answer` diversity is ~26% unique. See [Known Limitations](README.md#known-limitations) in the README.
