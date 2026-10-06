# Project-Harness Engineering with Claude and Claude Code — Reflection Brief

**Name:** Asheesh Dhamacharla  
**Date:** October 6, 2026

---

## Environment

- **Model(s):** Claude via the Anthropic Messages API, with Claude Code configuration, skills, commands, and orchestration patterns.
- **OS / Python:** Linux workspace / Python 3.x.
- **Approx. API spend:** Not captured in the available evidence.

---

# Part 1

## 1. Loop control

The Claims Intake Agent uses `response.stop_reason` as the loop-control mechanism. In `claims_intake/loop.py`, the loop continues only when `stop_reason == "tool_use"`, returns when it receives `"end_turn"`, and raises `UnexpectedStopReason` for any other value; the implementation deliberately uses `while True` rather than a fixed iteration count. The successful stolen-bike trace shows the sequence `tool_use → tool_use → tool_use → tool_use → end_turn` across five turns.

**Evidence:** `claims_intake/loop.py`; `runs/20260925_200251/traces/claim_02_stolen_bike.jsonl`.

## 2. Anti-pattern

One explicit anti-pattern test is `test_no_integer_literal_iteration_cap_in_loop`. Other checks include `test_no_string_membership_against_text_in_loop`, `test_stop_reason_is_loop_control`, and `test_no_claim_type_equality_branching_in_package`. These tests prevent the implementation from replacing semantic termination with brittle hard-coded assumptions such as `while turn < 5`, which could terminate a legitimate workflow prematurely.

**Evidence:** `tests/test_antipatterns.py`.

## 3. Tool design

`lookup_policy` and `record_claim_fact` both operate on claim information, but their descriptions establish different responsibilities. `lookup_policy` retrieves an existing coverage record, while `record_claim_fact` records one normalized fact into the case file. The terminal tools strengthen the boundary further: `route_to_adjuster` and `escalate_to_human` are explicitly marked as terminal tools and specify that the next response should stop with `end_turn`. The dispatcher also returns structured errors containing `is_error`, `error_category`, `is_retryable`, and `message`, so failures can be handled without raw exceptions escaping into the loop.

**Evidence:** `claims_intake/tools.py`.

## 4. Numbers

The successful stolen-bike run required five model turns: four `tool_use` turns followed by the final `end_turn`. The trace records 2,761 input / 81 output tokens on turn 1 and 4,060 input / 30 output tokens on turn 5. This shows how additional tool interactions expand conversational state and therefore affect inference cost. The available evidence does not contain the README sample numbers needed for an exact numerical comparison, so I am not inventing one.

**Evidence:** `runs/20260925_200251/traces/claim_02_stolen_bike.jsonl`.

## 5. Reduction

The retail-support context system reduced the baseline context from 38,708 tokens to 16,901 tokens, a 56.34% reduction. The dominant section was the active context at 15,789 tokens, while `case_facts`, `resolved_refund`, and `resolved_subscription` used 204, 416, and 510 tokens respectively. The compression experiment reduced the refund section from 12,334 input tokens to 403 output tokens and the subscription section from 11,475 to 497.

**Evidence:** System 2 `runs/20260925-201301/budget.json`.

## 6. Summarize vs preserve

The rule I would use is to summarize information useful for background reasoning while preserving structured information whose exact value or state must remain authoritative. The final context demonstrates this distinction because the structured sections remained very small while the active section retained the majority of the assembled context. This makes context engineering a selective information-preservation problem rather than a goal of compressing every piece of information equally.

**Evidence:** System 2 `budget.json`, including the per-section token counts and compression results.

## 7. Facts block

The evaluation/control comparison demonstrates that preserving structured state had a measurable effect. The assembled-context evaluation passed all six questions, including Q6 where the expected answer was `in_progress`, while the control variant failed Q6 and produced an unknown result. This shows that the structured status information was not merely an optimization for token usage; removing it caused a specific loss of information required to answer the evaluation correctly.

**Evidence:** System 2 `eval.jsonl` and `eval_control.jsonl`.

## 8. Path-scoped rules

The Claude Code configuration uses path-scoped frontmatter rather than applying every rule globally. The React rule targets `src/components/**/*` and `src/pages/**/*`, while the test rule targets `**/*.test.tsx` and `**/*.test.ts`. This is preferable to putting every convention into a broad directory-level `CLAUDE.md` because the relevant instruction is activated according to the artifact being edited.

**Evidence:** `.claude/rules/react.md` and `.claude/rules/tests.md`.

## 9. Forked skill

The `deploy-check` skill uses `context: fork` and limits itself to `Read`, `Glob`, and `Grep`. This makes deployment checking an isolated, read-only activity instead of allowing its exploratory context to accumulate in the main working session. The separation is useful because deployment checking is diagnostic work, while the main session may be performing implementation work.

**Evidence:** `.claude/skills/deploy-check/SKILL.md`.

## 10. Scope

The configuration distinguishes project-level and user-level Claude configuration. Project-level configuration includes `./CLAUDE.md`, `.claude/standards/`, and `.claude/rules/`, while user-level configuration includes `~/.claude/CLAUDE.md`, `~/.claude/commands/`, and `~/.claude/skills/`. This separation allows team-shared behavior to remain in the repository while personal configuration remains outside version control.

**Evidence:** project `CLAUDE.md` and the configuration scope documentation.

## 11. Push work down

System 4 pushes historical defect retrieval into the warm data tier rather than asking Claude to reason over the entire defect history. `pipeline.py` calls `gather_new_defects()`, which delegates to `warm.defects_since()`. The warm tier uses `SELECT * FROM defects WHERE ts > ? ORDER BY ts DESC LIMIT ?`, and the defect store has timestamp and shift indexes. This means deterministic filtering happens in SQLite before data reaches Claude, so the model receives only the relevant recent defect window.

**Evidence:** `shift_monitor/pipeline.py` (`gather_new_defects`); `shift_monitor/warm.py` (`defects_since` and indexes).

## 12. Crash recovery

The recovery design distinguishes between `resume` and `fresh` execution based on checkpoint validity and staleness. The implementation uses a 30-minute threshold: a sufficiently recent checkpoint can be resumed, while a stale checkpoint causes a fresh run. This matters because blindly resuming stale state can propagate obsolete assumptions into a new shift. A fresh run combined with an injected summary can therefore be more reliable than continuing from an old intermediate state after the freshness boundary has been crossed.

**Evidence:** `shift_monitor/recovery.py`.

## 13. Small state

The multi-shift system's `data/hot_state.json` is 643 bytes. It contains recent defect hashes, the current shift summary, active alerts, and threshold statuses. The important design property is that hot state is bounded rather than becoming an append-only representation of the entire operational history. For an indefinitely running system, uncontrolled state growth eventually becomes a reliability and cost problem because later operations inherit more state.

**Evidence:** `data/hot_state.json`; file size verified as 643 bytes.

---

# Part 2

## 14. Three layers

The projects exposed three useful engineering layers. The file/artifact layer contains persistent things such as traces, SQLite data, configuration files, rules, summaries, and hot state. The harness layer contains deterministic code that controls loops, validates configuration, manages budgets, executes tools, and enforces boundaries. The orchestration layer coordinates what information should be retrieved, compressed, summarized, or passed to Claude.

The key lesson is that not every reliability problem belongs in the prompt: artifacts and code should enforce hard guarantees, while Claude should handle reasoning where ambiguity genuinely exists.

**Evidence:** `claims_intake/loop.py`, System 2 `budget.json` and evaluation artifacts, System 3 configuration files, and System 4 `pipeline.py` and `warm.py`.

## 15. Deterministic vs prompt

A deterministic guarantee from System 1 is loop termination behavior: `loop.py` explicitly continues on `tool_use`, returns on `end_turn`, and rejects unexpected stop reasons. A prompt-guided behavior is tool selection: the tool descriptions tell Claude when to use `lookup_policy`, when to record facts, and when classification should occur. I would use deterministic code whenever violating the rule could corrupt state, cause an unsafe action, or make the system uncontrollable, and model reasoning where the problem is genuinely semantic.

**Evidence:** `claims_intake/loop.py` and `claims_intake/tools.py`.

## 16. Context two faces

System 2 demonstrates intra-session context engineering: 38,708 baseline tokens were reduced to 16,901 assembled tokens, a 56.34% reduction. System 4 demonstrates cross-session context engineering by using compact hot state together with warm SQLite storage and historical data rather than retaining the entire history in the active state. The hot state itself is only 643 bytes. The deeper pattern is the same: the model should receive the smallest representation that preserves the information necessary for the current decision.

**Evidence:** System 2 `budget.json`; System 4 `data/hot_state.json`, `pipeline.py`, and `warm.py`.

## 17. Reliability invisible in one run

The strongest example is the deterministic test suite. The systems were not judged only by successful demonstrations; anti-pattern tests explicitly checked properties such as avoiding fixed iteration caps and text-based branching. System 3 reached 35 passing tests and System 4 reached 33 passing tests. This provides evidence beyond a single successful Claude execution and shows that important architectural properties can be enforced independently of model behavior.

**Evidence:** `tests/test_antipatterns.py`; System 3 `pytest_outputs.txt`; System 4 `system4_pytest_output.txt`.

## 18. Blast radius

The claims intake loop has a clear blast-radius boundary. An uncontrolled loop could continue consuming model calls and expanding context, so the system separates semantic termination from a `Budget` safety mechanism rather than relying on a fixed iteration cap. The tool layer also returns structured failures instead of allowing raw exceptions to escape into the loop. The resulting kill switch is therefore implemented at the harness and budget layer rather than depending on Claude voluntarily behaving correctly.

**Evidence:** `claims_intake/loop.py` and `claims_intake/tools.py`.

---

# Part 3

## 19. What broke?

The clearest failure during development was at the engineering/evidence boundary rather than simply being a model-reasoning failure. An attempted Claims Intake end-to-end execution initially failed because the runtime credentials were not valid, while the later successful run produced the complete set of claim traces, queues, escalation output, and summary artifacts. The important lesson was to distinguish between “the code exists” and “there is valid evidence that the end-to-end system actually executed successfully.” The final successful run provides that evidence through its persistent trace and output artifacts.

**Evidence:** `runs/20260925_200251/` and its traces, queues, `escalations.jsonl`, and `summary.md`.

## 20. What would I change?

I would design the evidence layer at the same time as the agent architecture rather than treating evidence collection as a final submission step. The strongest systems naturally exposed useful artifacts such as stop-reason traces, token budgets, evaluation/control comparisons, path-scoped configuration, warm-tier queries, checkpoints, and bounded hot state. My architectural change would therefore be to make observability and evaluation first-class outputs of every agentic component: important control decisions should leave machine-readable traces, important invariants should have deterministic tests, and important state transitions should have bounded artifacts that can be inspected independently of the model.

**Evidence:** successful Claims traces; System 2 `budget.json` and evaluation artifacts; System 3 configuration/test artifacts; System 4 warm/recovery/hot-state artifacts.

---

# Final Takeaway

The biggest lesson from the four systems is:

> **Do not ask Claude to be the system. Build the system around Claude.**

The model should handle the parts that genuinely require semantic reasoning. The harness should own control flow, budgets, validation, state boundaries, failure semantics, and safety. The orchestration layer should decide what information Claude actually needs.

The four projects converge on the same architecture:

**Model reasoning → Harness enforcement → Orchestrated context → Persistent evidence**

That is the pattern I would carry forward into larger agentic and AI-infrastructure systems.
