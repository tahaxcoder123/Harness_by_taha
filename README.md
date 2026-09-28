# Harness-Eval — Coding Agent Harness Comparison & Evaluation Engine

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![Evaluation Engine](https://img.shields.io/badge/eval-multi--dimensional-emerald.svg)]()
[![License](https://img.shields.io/badge/license-MIT-purple.svg)]()

> **"Did changing the coding-agent harness make the agent better, worse, or inconclusive?"**

`harness-eval` is an evaluation engine designed to answer this question with empirical, auditable evidence. Rather than judging harness changes by reading a few anecdotal outputs or collapsing evaluation into an unexplained single number, `harness-eval` runs a baseline harness and a candidate harness over identical benchmark tasks, collects multidimensional signals, and renders transparent verdicts (`POSITIVE`, `NEGATIVE`, or `INCONCLUSIVE`).

> [!NOTE]
> **Detailed Technical Documentation**: For the complete architectural specification, multi-dimensional methodology, decision engine rules matrix, and client-facing technical breakdown, see [`docs/Project_Documentation.md`](file:///d:/Task/chat/docs/Project_Documentation.md).

---

## 1. Core Concepts

| Concept | Definition |
| :--- | :--- |
| **Harness** | The entire system perimeter surrounding the coding agent: model choice, system prompts, `AGENTS.md`, skill libraries, tool permissions, hooks, and temperature. |
| **Baseline** | The current stable harness configuration in production. |
| **Candidate** | The proposed or modified harness configuration under test (e.g. adding a new skill, updating prompts, or switching models). |
| **Benchmark** | A curated collection of engineering tasks with explicit acceptance criteria, target files, unit tests, and holdout tests. |
| **Evaluation Run** | An isolated execution of a benchmark task under a specific harness, recording execution traces, patch diffs, test logs, tokens, and runtime. |
| **Holdout Tests** | Private verification tests run during evaluation that the agent was not shown during development, preventing test gaming and overfitting. |
| **Metrics** | Multi-dimensional measurements: Correctness, Requirement satisfaction, Convention compliance, Cost/tokens, Latency, and Reliability. |
| **Evidence** | Granular, auditable artifacts: unified diffs, failure tracebacks, criterion check results, and per-task comparisons. |
| **Decision** | The verdict rendered by a transparent rules engine: `POSITIVE` (unambiguous improvement), `NEGATIVE` (measurable degradation), or `INCONCLUSIVE` (trade-offs or insufficient sample size). |
| **Inconclusive Result** | A primary first-class outcome returned when accuracy gains are paired with high regressions, extreme cost inflation, or small-sample statistical uncertainty. |

---

## 2. Architecture

```text
                  +-----------------------------------+
                  |      Benchmark Tasks (YAML)       |
                  +-----------------+-----------------+
                                    |
                                    v
                         +--------------------+
                         |    Agent Runner    |
                         | (Mock / Real API)  |
                         +----+----------+----+
                              |          |
               +--------------+          +---------------+
               v                                         v
     +-------------------+                     +-------------------+
     | Baseline Harness  |                     | Candidate Harness |
     | (config, prompts, |                     | (config, prompts, |
     |  skills, tools)   |                     |  skills, tools)   |
     +---------+---------+                     +---------+---------+
               |                                         |
               +--------------+          +---------------+
                              |          |
                              v          v
                         +--------------------+
                         | Multi-Dimensional  |
                         |  Evaluator Engine  |
                         +---------+----------+
                                   |
           +-----------------------+-----------------------+
           |                       |                       |
           v                       v                       v
     [Correctness]           [Requirements]          [Code Quality]
     - Unit tests            - Acceptance Criteria   - Type annotations
     - Holdout tests         - Weighted scoring      - Lint / Formatting
           |                       |                       |
           +-----------------------+-----------------------+
                                   |
                                   v
                         +--------------------+
                         |     Comparator     |
                         |   (Task Deltas)    |
                         +---------+----------+
                                   |
                                   v
                         +--------------------+
                         |  Decision Engine   |
                         | (Rules & Tradeoffs)|
                         +---------+----------+
                                   |
     +-----------------------------+-----------------------------+
     |                             |                             |
     v                             v                             v
[Terminal Summary]         [Interactive HTML]            [report.json]
(Rich tables & banner)    (Self-contained report)     (Raw runs/ artifacts)
```

---

## 3. Installation

Requires Python 3.10+.

```bash
# Clone the repository
git clone https://github.com/example/harness-eval.git
cd harness-eval

# Create and activate virtual environment
python -m venv .venv
# On Linux/macOS:
source .venv/bin/activate
# On Windows:
.venv\Scripts\activate

# Install the package in editable mode with development dependencies
pip install -e .
```

---

## 4. Quick Start

Run an evaluation comparing the baseline harness against the candidate harness on the sample project benchmark:

```bash
harness-eval evaluate \
  --baseline harnesses/baseline \
  --candidate harnesses/candidate \
  --tasks benchmarks/sample/tasks.yaml \
  --output reports/sample
```

Inspect the raw diffs, logs, and evidence for a specific task:

```bash
harness-eval inspect task-01 --output reports/sample
```

Show configuration differences between the two harnesses:

```bash
harness-eval diff \
  --baseline harnesses/baseline \
  --candidate harnesses/candidate
```

---

## 5. Sample Terminal Output

```text
-------------------------- HARNESS EVALUATION REPORT --------------------------
+------------------------------ Harness Changes ------------------------------+
| Baseline : baseline (claude-3-5-haiku-20241022)                             |
| Candidate: candidate (claude-3-5-sonnet-20241022)                           |
| Skills Added  : skills/python-testing.md, skills/strict-typing.md           |
| Skills Removed: skills/basic-python.md                                      |
| Hooks Added   : hooks/pre_commit_lint.sh                                    |
+-----------------------------------------------------------------------------+
                 Evaluation Overview (6 Tasks, Repetitions=1)                  
+-----------------------------------------------------------------------------+
| Evaluation Dimension    |   Baseline |   Candidate |      Delta |  Relative |
|-------------------------+------------+-------------+------------+-----------|
| Correctness Rate        |      36.7% |      100.0% |    +63.3pp |   +172.7% |
| Requirement             |      31.4% |      100.0% |    +68.6pp |   +218.2% |
| Satisfaction            |            |             |            |           |
| Convention Compliance   |      75.0% |      100.0% |    +25.0pp |    +33.3% |
| Pass Rate               |       0.0% |      100.0% |   +100.0pp |         - |
| Average Cost ($)        |    $0.0091 |     $0.0328 |   $+0.0237 |   +260.4% |
| Average Runtime (s)     |      16.0s |       20.8s |      +4.7s |    +29.6% |
| Total Tokens            |      27784 |       36799 |      +9015 |    +32.5% |
+-----------------------------------------------------------------------------+
                          Per-Task Evidence Breakdown                          
+-----------------------------------------------------------------------------+
| Task ID | Title          | Baseline | Candidate |   Outcome    | Cost Delta |
|---------+----------------+----------+-----------+--------------+------------|
| task-01 | Add pagination |  FAILED  |  SUCCESS  | IMPROVED (+) |   $+0.0246 |
|         | to users API   |          |           |              |            |
| task-02 | Fix order      |  FAILED  |  SUCCESS  | IMPROVED (+) |   $+0.0248 |
|         | total bug      |          |           |              |            |
| task-03 | Add duplicate  |  FAILED  |  SUCCESS  | IMPROVED (+) |   $+0.0210 |
|         | email check    |          |           |              |            |
| task-04 | Add auth unit  |  FAILED  |  SUCCESS  | IMPROVED (+) |   $+0.0243 |
|         | tests          |          |           |              |            |
| task-05 | Refactor db    |  FAILED  |  SUCCESS  | IMPROVED (+) |   $+0.0232 |
|         | transaction    |          |           |              |            |
| task-06 | Add phone field|  FAILED  |  SUCCESS  | IMPROVED (+) |   $+0.0248 |
+-----------------------------------------------------------------------------+
+--------------------------- Evaluator Conclusion ----------------------------+
| DECISION: INCONCLUSIVE                                                      |
|                                                                             |
| Verdict: Promising correctness gain, but inconclusive due to resource       |
| trade-offs or sample size.                                                  |
| Reason: Candidate improved correctness by +63.3 percentage points (36.7% -> |
| 100.0%) and had 0 regressions. However, the verdict is INCONCLUSIVE due to  |
| higher operational cost (+260.4%), and limited sample size (N=6).           |
| Engineering teams should verify whether the accuracy gain justifies the     |
| added resource cost.                                                        |
|                                                                             |
| Documented Trade-offs / Limitations:                                        |
|  - Cost increased by +260.4% ($0.0091 -> $0.0328/task).                     |
|  - Sample size (N=6) is below threshold of 10 tasks for statistical         |
| certainty.                                                                  |
|                                                                             |
| Statistical Reliability: Sample size (N=6, 6 tasks x 1 reps) is too small   |
| to establish formal statistical significance. Observed differences are      |
| directional.                                                                |
+-----------------------------------------------------------------------------+

Report files generated:
  HTML Report : reports/sample/report.html
  JSON Report : reports/sample/report.json
  Summary JSON: reports/sample/summary.json
```

---

## 6. Report Structure & Artifacts

All evaluation outputs are saved to the designated `--output` directory:

```text
reports/sample/
├── report.html        # Interactive, self-contained HTML evaluation report
├── report.json        # Complete, versioned JSON schema dump
├── summary.json       # Compact executive summary with deltas and trade-offs
└── runs/              # Granular task-level evidence
    ├── task-01/
    │   ├── baseline_run.json    # Exact tokens, timings, and test results
    │   ├── candidate_run.json
    │   ├── baseline.patch       # Unified diff generated by baseline
    │   ├── candidate.patch      # Unified diff generated by candidate
    │   └── comparison.json      # Structured task-level delta and evidence points
    ├── task-02/
    └── ...
```

---

## 7. Configuration Guide

### Adding a New Harness
Create a new directory under `harnesses/<harness_name>/`:
```yaml
# harnesses/my_harness/config.yaml
name: my_harness
version: "1.0.0"
model: "claude-3-5-sonnet-20241022"
temperature: 0.0
system_prompt: "prompts/system_prompt.txt"
agents_file: "AGENTS.md"
skills:
  - "skills/python-testing.md"
tools:
  - "filesystem"
  - "git"
hooks:
  - "hooks/pre_commit_lint.sh"
cost_per_1k_input: 0.003
cost_per_1k_output: 0.015
```

### Adding a New Benchmark Task
Add an entry to `benchmarks/sample/tasks.yaml`:
```yaml
  - id: task-07
    title: Add rate limiting middleware
    description: |
      Implement a TokenBucket rate limiter in app/rate_limit.py.
      Limit requests to 60 per minute per IP address.
    target_files:
      - app/rate_limit.py
    acceptance_criteria:
      - id: AC-07-1
        description: "Rejects requests exceeding 60 req/min with HTTP 429"
        weight: 1.0
    evaluation:
      unit_tests:
        - "tests/test_rate_limit.py"
      holdout_tests:
        - "eval_tests/test_rate_limit_holdout.py"
      lint_checks:
        require_type_hints: true
      timeout_seconds: 30
```

### Adding a New Runner
Subclass `AgentRunner` in `src/harness_eval/runners/base.py`:
```python
from harness_eval.runners.base import AgentRunner
from harness_eval.models import RunResult, BenchmarkTask, HarnessConfig
from pathlib import Path

class CustomRunner(AgentRunner):
    def run(self, task: BenchmarkTask, harness: HarnessConfig, project_dir: Path, iteration: int = 1, seed = None) -> RunResult:
        # Custom execution logic here
        ...
```

---

## 8. Real vs. Mock Runner

- **Mock Runner (`--runner mock`, default)**:
  - Fully deterministic and seedable (`--seed 42`).
  - Zero external dependencies or API keys required.
  - Generates realistic unified diffs, pytest logs, holdout test execution, and token counters.
  - Enables immediate local execution and CI testing.
- **Real Runner (`--runner real`)**:
  - Intended for execution against actual coding agent CLI commands or LLM providers.
  - Requires `AGENT_RUNNER_CMD`, `OPENAI_API_KEY`, or `ANTHROPIC_API_KEY` in the environment.
  - Executes task commands in an isolated subprocess, capturing stdout/stderr and real execution times.

---

## 9. Repeated Runs & Variance Analysis

LLM coding agents exhibit non-deterministic behavior. To evaluate consistency, use `--repetitions`:

```bash
harness-eval evaluate --repetitions 3 --output reports/reps_3
```

When `--repetitions >= 3` and total sample size $N \ge 15$, the statistics engine aggregates results across iterations, computing:
- Pass rate and failure distributions
- Runtime and cost mean, median, min, and max
- Variance consistency across repetitions

---

## 10. Design Decisions & Limitations

### Deliberate Design Decisions
1. **First-Class Inconclusive Verdicts**: Unlike conventional benchmarks that force binary pass/fail, `harness-eval` renders `INCONCLUSIVE` whenever accuracy gains require disproportionate cost inflation ($>60\%$) or whenever regressions are detected.
2. **Holdout Test Isolation**: Evaluates models against hidden tests they were never instructed to pass, filtering out agents that overfit to visible assertions.
3. **No Unexplained Composite Number**: Metrics are kept in their native dimensional units (accuracy in percentage points, cost in dollars, runtime in seconds).

### Current Limitations
1. **Static Analysis Heuristics**: Convention compliance checks check type hints and AST patterns; dynamic runtime security scanning is not currently integrated.
2. **Subprocess Isolation**: While the real runner limits directory execution, executing arbitrary untrusted agent code in production should be performed within Docker or microVM containers.

---

## 11. Automated Test Suite

Run the full pytest test suite:

```bash
pytest -v tests
```

Output:
```text
tests/test_benchmark_loader.py::test_load_sample_benchmark PASSED
tests/test_comparator.py::test_compare_task_outcome_improved PASSED
tests/test_decision_logic.py::test_decision_inconclusive_due_to_cost_tradeoff PASSED
tests/test_e2e_cli.py::test_cli_evaluate_and_inspect_command PASSED
tests/test_reporting.py::test_export_reports PASSED
...
============================= 28 passed in 4.79s ==============================
```
"# Harness_by_taha" 
#   H a r n e s s _ b y _ t a h a  
 "# Harness_by_taha" 
#   H a r n e s s _ b y _ t a h a  
 