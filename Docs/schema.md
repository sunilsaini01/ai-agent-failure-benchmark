# AI Agent Failure Benchmark Dataset — Schema

**Version:** v2.0  
**Last Updated:** 2025  
**Maintainer:** AI Evaluation Research  
**License:** CC BY 4.0  
**Kaggle-Ready:** Yes  

---

## Table of Contents

1. [Overview](#overview)
2. [Failure Taxonomy](#failure-taxonomy)
3. [Full Schema Reference](#full-schema-reference)
   - [Section A: Record Identification & Metadata](#section-a--record-identification--metadata)
   - [Section B: Task Definition — Input Features](#section-b--task-definition--input-features)
   - [Section C: Agent Configuration — Metadata](#section-c--agent-configuration--metadata)
   - [Section D: Agent Execution Trace — Input Features](#section-d--agent-execution-trace--input-features)
   - [Section E: Failure Labels — Target Labels](#section-e--failure-labels--target-labels)
   - [Section F: Evaluation Metrics](#section-f--evaluation-metrics)
   - [Section G: Annotation Quality Metadata](#section-g--annotation-quality-metadata)
4. [Column Classification Summary](#column-classification-summary)
5. [Minimum Viable Schema — v1](#minimum-viable-schema--v1)
6. [Advanced Research Schema — v2](#advanced-research-schema--v2)
7. [Enum Definitions](#enum-definitions)
8. [JSON Sub-Schema Definitions](#json-sub-schema-definitions)
9. [Derived Columns — Do Not Store](#derived-columns--do-not-store)
10. [Common Mistakes to Avoid](#common-mistakes-to-avoid)
11. [Design Principles](#design-principles)

---

## Overview

### Core Research Question

> *When AI agents fail — why do they fail, at what stage, under what conditions, and how severely?*

### Primary Use Cases

| Use Case | Target Column(s) |
|---|---|
| Binary classification | `failure_occurred` |
| Multi-class classification | `failure_category`, `failure_subcategory` |
| Severity regression | `failure_severity`, `task_success_score` |
| Efficiency benchmarking | `token_efficiency_ratio`, `step_efficiency_ratio` |
| Sequence failure detection | `full_trajectory`, `failure_step_index` |
| Model vs. model comparison | `underlying_model`, `task_success_score` |
| Framework benchmarking | `agent_framework`, `failure_occurred` |
| Safety evaluation | `harmful_output_detected`, `hallucination_detected` |

### Critical Distinction

| Term | Definition | Remediation |
|---|---|---|
| **LLM Failure** | The base model produced wrong, unsafe, or incoherent output | Change model, fine-tune, or adjust sampling |
| **Agent Failure** | Orchestration, planning, tool routing, or multi-step logic broke | Change framework, prompts, or agent architecture |
| **Co-occurrence** | Both can fail simultaneously — the schema must capture each independently | Captured via `is_llm_failure` + `is_agent_framework_failure` |

---

## Failure Taxonomy

> Lock this taxonomy before annotation begins. Treat as an immutable enum in production.

```
L1: failure_category              L2: failure_subcategory
─────────────────────────────     ──────────────────────────────────────────
REASONING_FAILURE                 hallucination
                                  logical_error
                                  math_error
                                  false_premise
                                  invalid_inference

PLANNING_FAILURE                  wrong_decomposition
                                  missing_step
                                  circular_plan
                                  over_planning
                                  under_planning

TOOL_USE_FAILURE                  wrong_tool_selected
                                  wrong_parameters
                                  tool_not_called
                                  redundant_tool_calls
                                  tool_output_misinterpreted

CONTEXT_MANAGEMENT_FAILURE        context_window_exceeded
                                  context_ignored
                                  lost_prior_state
                                  conflicting_context

INSTRUCTION_FOLLOWING_FAILURE     constraint_violated
                                  output_format_wrong
                                  task_scope_ignored
                                  partial_completion

TERMINATION_FAILURE               premature_stop
                                  infinite_loop
                                  max_iterations_exceeded
                                  goal_not_checked

MEMORY_FAILURE                    retrieval_failure
                                  incorrect_retrieval
                                  missing_update
                                  stale_memory

SAFETY_ALIGNMENT_FAILURE          policy_violation
                                  unsafe_output
                                  prompt_injection_exploited
                                  jailbreak_success

COORDINATION_FAILURE              agent_conflict
  (multi-agent tasks only)        role_confusion
                                  duplicate_work
                                  missing_handoff

GROUNDING_FAILURE                 confabulation
                                  source_ignored
                                  out_of_context_claim

NO_FAILURE                        task_succeeded
```

---

## Full Schema Reference

### Section A — Record Identification & Metadata

| Column | Type | Required | Description | Example |
|---|---|---|---|---|
| `record_id` | `string (UUID v4)` | ✅ | Globally unique row identifier. Never reuse across versions. | `"a3f7c2b1-4e8d-4f1a-b2c3-9d0e1f2a3b4c"` |
| `dataset_version` | `string` | ✅ | Schema version for reproducibility. Increment on any structural change. | `"v2.0"` |
| `split` | `enum` | ✅ | Dataset partition. Required for Kaggle competitions and fair benchmarking. | `"train"` |
| `source` | `string` | ✅ | Data origin: how the agent run was generated or collected. | `"synthetic_gpt4o"`, `"red_team_human"`, `"live_api_capture"` |
| `collection_timestamp` | `string (ISO 8601)` | ✅ | UTC timestamp of when the agent run was captured. Enables temporal drift analysis. | `"2024-11-15T10:22:00Z"` |
| `annotation_method` | `enum` | ✅ | How failure labels were assigned. | `"human"`, `"llm_judge"`, `"hybrid"`, `"automated_rule"` |

---

### Section B — Task Definition — Input Features

| Column | Type | Required | Description | Example |
|---|---|---|---|---|
| `task_id` | `string` | ✅ | Unique task identifier. Multiple agent runs on the same task share this ID. Enables cross-agent comparison on identical tasks. | `"task_0042"` |
| `task_description` | `string (text)` | ✅ | The exact prompt or instruction given to the agent verbatim. No paraphrasing. | `"Find the top 3 papers on RAG published in 2024 and summarize each in under 100 words."` |
| `task_category` | `enum` | ✅ | High-level task type for stratified evaluation. | `"multi_hop_reasoning"` |
| `task_domain` | `enum` | ✅ | Subject domain. Enables domain-specific failure analysis. | `"software_engineering"` |
| `task_complexity` | `enum` | ✅ | Difficulty level. Annotator-assigned or rubric-derived. | `"hard"` |
| `requires_multi_step` | `boolean` | ✅ | True if the task requires more than one reasoning or action step to complete. | `true` |
| `requires_tool_use` | `boolean` | ✅ | True if the task requires at least one external tool call. | `true` |
| `requires_memory` | `boolean` | ✅ | True if the task requires the agent to maintain state across multiple turns or steps. | `false` |
| `is_multi_agent` | `boolean` | ✅ | True if the task involves multiple cooperating agents. Unlocks COORDINATION_FAILURE taxonomy. | `false` |
| `input_context` | `string (text) \| null` | Optional | Documents, data, or background context provided to the agent as input. Null if no context given. | `"[contents of a 5-page PDF report]"` |
| `available_tools` | `JSON list<string> \| null` | Optional | Tool names the agent had access to. Null if no tools. | `["web_search", "calculator", "python_repl", "file_reader"]` |
| `max_steps_allowed` | `integer \| null` | Optional | Step budget set for the agent. Null if unlimited. | `10` |
| `expected_output` | `string (text)` | ✅ | Ground truth or reference answer. Must be paired with a scoring rubric in documentation. | `"Paper 1: (name) — summary. Paper 2: ..."` |
| `output_type` | `enum` | ✅ | Format of the expected output. Used to select the correct evaluation metric. | `"text"` |

---

### Section C — Agent Configuration — Metadata

| Column | Type | Required | Description | Example |
|---|---|---|---|---|
| `agent_id` | `string` | ✅ | Unique identifier for this agent configuration. Enables reproducible benchmarking. | `"react-gpt4o-v3"` |
| `agent_framework` | `string` | ✅ | Orchestration framework powering the agent. | `"LangChain"`, `"AutoGPT"`, `"CrewAI"`, `"LlamaIndex"`, `"custom"` |
| `agent_type` | `enum` | ✅ | Agent architectural pattern. Directly affects failure mode distribution. | `"ReAct"` |
| `underlying_model` | `string` | ✅ | The LLM powering the agent. Core variable for model benchmarking. | `"gpt-4o"`, `"claude-3-5-sonnet"`, `"llama-3-70b"` |
| `model_version` | `string` | ✅ | Exact model version or checkpoint. General model name alone is insufficient for reproducibility. | `"gpt-4o-2024-11-20"` |
| `temperature` | `float` | ✅ | Sampling temperature used during the run. Affects stochasticity of failure modes. | `0.7` |
| `system_prompt_hash` | `string (SHA-256) \| null` | Optional | SHA-256 hash of the system prompt. Preserves privacy while enabling deduplication. | `"3f8a9c2b1d..."` |
| `memory_type` | `enum \| null` | Optional | Memory mechanism used. Relevant for MEMORY_FAILURE analysis. | `"vector_store"` |
| `context_window_size` | `integer \| null` | Optional | Maximum token context window of the underlying model. | `128000` |

---

### Section D — Agent Execution Trace — Input Features

| Column | Type | Required | Description | Example |
|---|---|---|---|---|
| `agent_final_output` | `string (text)` | ✅ | The final response or output the agent returned. The primary artifact being evaluated. | `"I found 3 relevant papers: Paper 1..."` |
| `num_steps_taken` | `integer` | ✅ | Total reasoning or action steps executed. Baseline efficiency signal. | `7` |
| `num_tool_calls` | `integer` | ✅ | Total external tool calls made during the run. | `3` |
| `tool_call_sequence` | `JSON list<ToolCall> \| null` | Optional | Ordered list of tool invocations with inputs. See JSON sub-schema. Null if no tools used. | See JSON sub-schema |
| `tool_call_results` | `JSON list<ToolResult> \| null` | Optional | Results returned from each tool call. Required for TOOL_USE_FAILURE subcategory annotation. | See JSON sub-schema |
| `full_trajectory` | `JSON list<Step> \| null` | Optional | Complete chain-of-thought and action trace. Required for `failure_step_index` and RCA research. | See JSON sub-schema |
| `terminated_early` | `boolean` | ✅ | True if agent stopped before reaching the goal without hitting max iterations. | `false` |
| `hit_max_iterations` | `boolean` | ✅ | True if agent exhausted its step budget without completing the task. | `false` |
| `execution_time_ms` | `integer \| null` | Optional | Wall-clock time in milliseconds for the full agent run. Latency benchmarking. | `4823` |
| `input_token_count` | `integer \| null` | Optional | Total input tokens consumed across all LLM API calls in this run. | `5102` |
| `output_token_count` | `integer \| null` | Optional | Total output tokens generated across all LLM API calls. | `1204` |
| `api_calls_made` | `integer \| null` | Optional | Total LLM API calls made during the agent run. Differs from `num_steps_taken` in multi-call steps. | `8` |

---

### Section E — Failure Labels — Target Labels

| Column | Type | Required | Description | Example |
|---|---|---|---|---|
| `failure_occurred` | `boolean` | ✅ | **Primary binary label.** True if the agent failed to complete the task correctly. | `true` |
| `failure_category` | `enum \| null` | ✅ | **Primary multi-class label.** L1 failure type from the locked taxonomy. Null only when `failure_occurred = false`. | `"TOOL_USE_FAILURE"` |
| `failure_subcategory` | `enum \| null` | ✅ | **Secondary multi-class label.** L2 granular failure type. Null when no failure. | `"wrong_tool_selected"` |
| `failure_stage` | `enum \| null` | ✅ | Pipeline stage where failure first occurred. Null when no failure. | `"execution"` |
| `failure_severity` | `enum \| null` | ✅ | Impact level of the failure on task outcome. Null when no failure. | `"critical"` |
| `failure_step_index` | `integer \| null` | Optional | Step number (0-indexed) at which the failure first appeared. Requires `full_trajectory`. | `3` |
| `is_llm_failure` | `boolean` | ✅ | True if the root cause lies in the LLM's own output (hallucination, reasoning error, etc.). | `true` |
| `is_agent_framework_failure` | `boolean` | ✅ | True if the root cause lies in orchestration logic, tool routing, or planning architecture. | `false` |
| `is_recoverable` | `boolean \| null` | Optional | True if the agent could self-correct with one additional reflection or retry step. Used in self-healing agent research. | `false` |
| `root_cause_description` | `string (text) \| null` | Optional | Free-text annotator description of root cause. Unstructured. Goes in `annotation_notes` level of detail. | `"Agent selected calculator tool when web search was needed to retrieve current data."` |

---

### Section F — Evaluation Metrics

| Column | Type | Required | Description | Example |
|---|---|---|---|---|
| `task_success_binary` | `boolean` | ✅ | True if agent completed the task correctly. Strict pass/fail. | `false` |
| `task_success_score` | `float [0.0, 1.0]` | ✅ | Continuous success score. Allows partial credit. Scoring rubric must be documented separately. | `0.4` |
| `output_quality_score` | `float [0.0, 1.0] \| null` | Optional | Quality of the agent's output independent of task success. Captures well-written failures. | `0.6` |
| `factual_accuracy_score` | `float [0.0, 1.0] \| null` | Optional | Fraction of factual claims in output that are verifiably accurate. | `0.75` |
| `instruction_adherence_score` | `float [0.0, 1.0] \| null` | Optional | Proportion of stated constraints and instructions correctly followed. | `0.5` |
| `hallucination_detected` | `boolean` | ✅ | True if at least one hallucination was detected in the agent's output. | `true` |
| `harmful_output_detected` | `boolean` | ✅ | True if output contained harmful, unsafe, or policy-violating content. | `false` |
| `token_efficiency_ratio` | `float \| null` | Optional | `task_success_score / (input_token_count + output_token_count)`. Quality-per-cost signal. Derived — compute at load time. | `0.000078` |
| `step_efficiency_ratio` | `float \| null` | Optional | `task_success_score / num_steps_taken`. Planning efficiency signal. Derived — compute at load time. | `0.057` |
| `human_eval_score` | `float [0.0, 1.0] \| null` | Optional | Human annotator rating of overall output quality. Null if annotation method was not human. | `0.35` |
| `llm_judge_score` | `float [0.0, 1.0] \| null` | Optional | LLM-as-judge evaluation score. Scalable but has known biases (positional, verbosity). | `0.40` |
| `llm_judge_model` | `string \| null` | Optional | The model used as evaluator judge. Required when `llm_judge_score` is not null. | `"gpt-4o-2024-11-20"` |

---

### Section G — Annotation Quality Metadata

| Column | Type | Required | Description | Example |
|---|---|---|---|---|
| `annotator_id` | `string (hashed) \| null` | Optional | Anonymized annotator identifier. Never store PII. Used for inter-annotator agreement computation. | `"ann_017"` |
| `annotation_confidence` | `enum \| null` | Optional | Annotator's self-rated confidence in their failure label. | `"high"` |
| `inter_annotator_agreement` | `float \| null` | Optional | Cohen's κ or Krippendorff's α for this specific record where multiple annotators labeled it. | `0.78` |
| `annotation_notes` | `string (text) \| null` | Optional | Free-text edge case notes. For borderline cases, disagreements, or unusual behavior. | `"Borderline case — agent produced partially correct output but wrong format."` |
| `is_ambiguous_case` | `boolean` | ✅ | True for records where annotators disagreed or the failure classification is contested. Filter this from test sets. | `false` |

---

## Column Classification Summary

### Input Features

```
task_description          task_category             task_domain
task_complexity           requires_multi_step       requires_tool_use
requires_memory           is_multi_agent            input_context
available_tools           max_steps_allowed         expected_output
output_type               agent_type                underlying_model
temperature               system_prompt_hash        memory_type
context_window_size       agent_final_output        num_steps_taken
num_tool_calls            tool_call_sequence        tool_call_results
full_trajectory           terminated_early          hit_max_iterations
```

### Target Labels

```
failure_occurred          failure_category          failure_subcategory
failure_stage             failure_severity          is_llm_failure
is_agent_framework_failure                          task_success_binary
```

### Evaluation Metrics

```
task_success_score        output_quality_score      factual_accuracy_score
instruction_adherence_score                         hallucination_detected
harmful_output_detected   token_efficiency_ratio    step_efficiency_ratio
human_eval_score          llm_judge_score
```

### Metadata

```
record_id                 task_id                   dataset_version
split                     source                    collection_timestamp
annotation_method         agent_id                  agent_framework
model_version             execution_time_ms         input_token_count
output_token_count        api_calls_made            annotator_id
annotation_confidence     inter_annotator_agreement annotation_notes
is_ambiguous_case         llm_judge_model           failure_step_index
is_recoverable            root_cause_description
```

---

## Minimum Viable Schema — v1

> **Goal:** 18 columns. Enables binary + multi-class classification, basic benchmarking, and publication-level research. Immediately Kaggle-ready.

| # | Column | Type | Role |
|---|---|---|---|
| 1 | `record_id` | UUID | Metadata |
| 2 | `task_id` | string | Metadata |
| 3 | `split` | enum | Metadata |
| 4 | `task_description` | text | Input Feature |
| 5 | `task_category` | enum | Input Feature |
| 6 | `task_complexity` | enum | Input Feature |
| 7 | `requires_multi_step` | boolean | Input Feature |
| 8 | `requires_tool_use` | boolean | Input Feature |
| 9 | `underlying_model` | string | Input Feature |
| 10 | `agent_type` | enum | Input Feature |
| 11 | `available_tools` | JSON list | Input Feature |
| 12 | `expected_output` | text | Input Feature |
| 13 | `agent_final_output` | text | Input Feature |
| 14 | `num_steps_taken` | integer | Input Feature |
| 15 | `failure_occurred` | boolean | **Target Label** |
| 16 | `failure_category` | enum | **Target Label** |
| 17 | `failure_severity` | enum | **Target Label** |
| 18 | `task_success_score` | float | Eval Metric |

---

## Advanced Research Schema — v2

> **Goal:** Full v1 + deep execution trace + multi-judge evaluation + annotation quality + efficiency metrics. Suitable for NeurIPS/ICML benchmark papers.

**v2 = All columns in v1 PLUS:**

| Column | Research Value |
|---|---|
| `task_domain` | Domain-specific failure rate analysis |
| `is_multi_agent` | Multi-agent coordination research |
| `requires_memory` | Memory-augmented agent ablation studies |
| `input_context` | Retrieval-augmented agent evaluation |
| `max_steps_allowed` | Step budget impact on failure rate |
| `agent_framework` | Framework benchmarking |
| `model_version` | Exact version reproducibility |
| `system_prompt_hash` | Prompt sensitivity studies |
| `memory_type` | Memory mechanism ablation |
| `context_window_size` | Context scaling experiments |
| `tool_call_sequence` | Tool selection pattern mining |
| `tool_call_results` | Tool output misinterpretation research |
| `full_trajectory` | Step-level failure localization |
| `terminated_early` | Premature termination analysis |
| `hit_max_iterations` | Budget constraint impact research |
| `execution_time_ms` | Latency benchmarking |
| `input_token_count` | Cost modeling |
| `output_token_count` | Cost modeling |
| `api_calls_made` | Multi-call agent cost analysis |
| `failure_subcategory` | Fine-grained failure taxonomy research |
| `failure_stage` | Pipeline stage failure analysis |
| `failure_step_index` | Failure point detection in trajectories |
| `is_llm_failure` | LLM vs. framework failure attribution |
| `is_agent_framework_failure` | Framework failure attribution |
| `is_recoverable` | Self-healing and reflection agent research |
| `root_cause_description` | NLP-based RCA modeling |
| `task_success_binary` | Strict pass/fail alongside continuous score |
| `output_quality_score` | Output quality independent of success |
| `factual_accuracy_score` | Grounding and attribution research |
| `instruction_adherence_score` | Alignment research |
| `hallucination_detected` | LLM reliability benchmarking |
| `harmful_output_detected` | AI safety evaluation |
| `token_efficiency_ratio` | Cost-quality Pareto analysis |
| `step_efficiency_ratio` | Planning efficiency research |
| `human_eval_score` | Human baseline for LLM-judge calibration |
| `llm_judge_score` | Scalable evaluation at dataset scale |
| `llm_judge_model` | Judge model comparison research |
| `source` | Data provenance and bias analysis |
| `collection_timestamp` | Temporal model drift analysis |
| `annotation_method` | Annotation pipeline reproducibility |
| `annotator_id` | Per-annotator bias analysis |
| `annotation_confidence` | Label uncertainty modeling |
| `inter_annotator_agreement` | Dataset quality quantification |
| `annotation_notes` | Edge case documentation |
| `is_ambiguous_case` | Test set quality filtering |

---

## Enum Definitions

### `split`
```
"train" | "val" | "test"
```

### `task_category`
```
"information_retrieval"   | "code_generation"      | "code_debugging"
"multi_hop_reasoning"     | "math_problem_solving" | "planning"
"tool_use"                | "question_answering"   | "summarization"
"agentic_workflow"        | "creative_generation"  | "data_analysis"
"classification"          | "decision_making"      | "other"
```

### `task_domain`
```
"software_engineering"  | "data_science"    | "healthcare"
"finance"               | "legal"           | "science_research"
"education"             | "customer_service"| "cybersecurity"
"general"               | "other"
```

### `task_complexity`
```
"trivial" | "easy" | "medium" | "hard" | "expert"
```

### `agent_type`
```
"ReAct"              | "Plan-and-Execute"  | "MRKL"
"Reflexion"          | "Tree-of-Thought"   | "Self-Ask"
"BabyAGI"            | "OpenAI-Functions"  | "custom"
```

### `output_type`
```
"text"           | "code"           | "json"
"action_sequence"| "decision"       | "structured_data"
"mixed"          | "other"
```

### `failure_stage`
```
"input_parsing"  | "planning"    | "tool_selection"
"tool_execution" | "synthesis"   | "output_formatting"
"termination"    | "coordination"
```

### `failure_severity`
```
"critical"  → Task completely failed; output is wrong or harmful
"high"      → Task mostly failed; output is significantly flawed
"medium"    → Task partially succeeded; output has notable issues
"low"       → Task nearly succeeded; minor errors only
```

### `annotation_method`
```
"human"            | "llm_judge"       | "hybrid"
"automated_rule"   | "crowd_sourced"
```

### `annotation_confidence`
```
"high" | "medium" | "low"
```

### `memory_type`
```
"sliding_window" | "vector_store" | "summarization"
"full_history"   | "episodic"     | "none"
```

---

## JSON Sub-Schema Definitions

### `tool_call_sequence` — `list<ToolCall>`

```json
[
  {
    "step_index": 2,
    "tool_name": "web_search",
    "tool_input": {
      "query": "RAG papers 2024 arxiv"
    },
    "call_timestamp_ms": 1230
  },
  {
    "step_index": 4,
    "tool_name": "calculator",
    "tool_input": {
      "expression": "3 * 7"
    },
    "call_timestamp_ms": 2810
  }
]
```

### `tool_call_results` — `list<ToolResult>`

```json
[
  {
    "step_index": 2,
    "tool_name": "web_search",
    "tool_output": "1. RAG-fusion (2024)... 2. HippoRAG...",
    "tool_success": true,
    "error_message": null
  },
  {
    "step_index": 4,
    "tool_name": "calculator",
    "tool_output": null,
    "tool_success": false,
    "error_message": "TypeError: expression parse failed"
  }
]
```

### `full_trajectory` — `list<Step>`

```json
[
  {
    "step_index": 0,
    "step_type": "thought",
    "content": "I need to search for recent RAG papers from 2024.",
    "timestamp_ms": 102
  },
  {
    "step_index": 1,
    "step_type": "action",
    "content": "web_search",
    "action_input": {"query": "RAG 2024 arxiv"},
    "timestamp_ms": 540
  },
  {
    "step_index": 2,
    "step_type": "observation",
    "content": "Found: RAG-Fusion, HippoRAG, GraphRAG...",
    "timestamp_ms": 1250
  },
  {
    "step_index": 3,
    "step_type": "thought",
    "content": "I have enough results. I will now summarize.",
    "timestamp_ms": 1310
  },
  {
    "step_index": 4,
    "step_type": "final_answer",
    "content": "Paper 1: RAG-Fusion — ...",
    "timestamp_ms": 2980
  }
]
```

---

## Derived Columns — Do Not Store

> Compute these at load time or in feature engineering. Storing them wastes space and risks staleness.

| Column | Derivation Formula |
|---|---|
| `total_token_count` | `input_token_count + output_token_count` |
| `token_efficiency_ratio` | `task_success_score / total_token_count` |
| `step_efficiency_ratio` | `task_success_score / num_steps_taken` |
| `failure_occurred` (if inferred) | `task_success_binary == false` ← **Do not infer this.** Annotate directly. Partial success exists. |

---

## Common Mistakes to Avoid

### 1. No locked taxonomy before annotation
> Annotating `failure_category` as free-text produces a useless column. Fix: lock the enum before any data is collected.

### 2. Conflating `failure_occurred` and `task_success_binary`
> These are NOT always inverses. An agent can partially succeed (`task_success_score = 0.4`) while still failing (`failure_occurred = true`). Annotate each independently.

### 3. Missing LLM vs. framework failure distinction
> Without `is_llm_failure` and `is_agent_framework_failure`, your dataset cannot answer the most important diagnostic question. Every benchmark paper will ask this first.

### 4. `expected_output` without a scoring rubric
> Storing a reference answer with no documented scoring method makes `task_success_score` non-reproducible. The rubric belongs in a companion `SCORING_RUBRIC.md` file.

### 5. Not recording `full_trajectory`
> Labeling the final output only is shallow. Failure often manifests at step 7 but originates at step 2. Without the trajectory, root cause analysis is impossible.

### 6. Skipping inter-annotator agreement
> If `inter_annotator_agreement` (κ) < 0.60 on failure labels, your target variable is noise. Measure it. Report it. Filter the test set to κ > 0.80.

### 7. Storing system prompts in plaintext
> System prompts are proprietary IP. Store `system_prompt_hash` (SHA-256) only. If reproducibility requires the prompt, release it in a separate access-controlled file.

### 8. Using free-text where enums belong
> `failure_category = "the agent got confused and went in circles"` is uncategorizable. All structured labels must be enums. Free-text goes in `annotation_notes` only.

### 9. Missing `split` column
> Without explicit train/val/test assignment, every user splits differently. Benchmark results become incomparable. This is non-negotiable.

### 10. Storing derived columns
> `total_token_count`, `token_efficiency_ratio`, `step_efficiency_ratio` are all derivable. Storing them risks inconsistency when source values are corrected.

---

## Design Principles

```
Benchmark Quality = Label Reliability × Schema Completeness × Reproducibility

Label Reliability    → Locked taxonomy + IAA measurement + ambiguity flagging
Schema Completeness  → Task + Agent Config + Execution Trace + Failure Labels + Metrics
Reproducibility      → Schema version + Source + Timestamp + Split + Model version
```

### The Three Audiences

| Audience | Primary Columns |
|---|---|
| **NLP / AI Evaluation Researchers** | `full_trajectory`, `failure_category`, `hallucination_detected`, `factual_accuracy_score`, `inter_annotator_agreement` |
| **Agent Systems Engineers** | `agent_framework`, `agent_type`, `tool_call_sequence`, `failure_stage`, `is_agent_framework_failure`, `step_efficiency_ratio` |
| **AI Safety Researchers** | `harmful_output_detected`, `safety_alignment_failure` subcategories, `is_recoverable`, `prompt_injection_exploited` |

Design the schema to serve all three. That is what makes a benchmark cited rather than merely downloaded.
