# Plan Mode vs Direct Execution

## Plan-mode example

Use plan mode when a change crosses multiple files or requires investigation before implementation. For example, changing the order-refund behavior may require inspecting:

- `src/api/orders/refund.ts`
- `src/api/orders/handler.ts`
- `src/api/_schemas/orders.ts`
- `src/db/orders.ts`

A plan makes the intended changes and dependencies explicit before implementation. This helps prevent costly rework when several files must remain consistent.

## Direct-execution example

Use direct execution when the requested change is narrowly scoped to one well-understood function. For example, if `src/api/orders/refund.ts` contains one function whose validation condition needs to change, directly update that one function and run its focused tests.

The key signal is that the task targets one function or similarly small, well-understood unit rather than requiring architectural investigation.

## Explore example

Use an Explore subagent when discovery is verbose or uncertain. Its purpose is to isolate verbose discovery output from the main conversation while it searches the repository and returns the useful findings.

The Explore pattern should isolate verbose discovery and use a scratchpad to record intermediate findings, following the scratchpad pattern described in the Playbook. The main session should receive the distilled result rather than every exploratory step.

## Knight-Webb curriculum connection

The curriculum's Knight-Webb material, including the anchor talk **SWE Is Becoming Plan and Review**, presents planning and review as an important engineering workflow. This is covered in the curriculum / Module 8 material.

## Combined workflow

A combined workflow uses **plan-mode then direct execution**: first investigate and form a plan when the change spans several files, then switch to direct execution once the affected scope and implementation are understood.

For example, use plan mode to inspect the refund flow and identify the relevant files, then directly modify the one well-scoped function responsible for the validation behavior and run the focused tests.
