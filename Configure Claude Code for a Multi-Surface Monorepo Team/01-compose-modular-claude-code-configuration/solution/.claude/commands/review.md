---
description: Review a change for correctness, regressions, and project-specific risks
argument-hint: "[files, PR, or change to review]"
allowed-tools:
  - Read
  - Glob
  - Grep
---

# Review

Review the requested change using the repository's standards and path-scoped rules.

## Must-report vs skip

**Must-report** findings that are concrete correctness problems, regressions, security issues, broken contracts, or behavior that is clearly inconsistent with the repository's standards.

**Skip** cosmetic preferences, speculative concerns without evidence, and issues that do not materially affect correctness, maintainability, or the requested behavior. Do not report something merely because you would personally implement it differently.

## Interacting vs independent issues

For **interacting** issues, bundle them into a single message when they describe one underlying failure or must be fixed together.

For **independent** issues, report them sequentially as separate findings so each can be understood and fixed independently.

## Input / Output examples

### Example 1

Input:
Review the order refund handler in `src/api/orders/refund.ts`.

Output:
Report a concrete defect if the handler can issue a refund without validating the order state; otherwise skip speculative concerns and state that no must-report finding was found.

### Example 2

Input:
Review the changes to `src/components/Cart/Cart.tsx` and its test.

Output:
Report any regression that makes the component behavior inconsistent with its test contract, and identify the exact file and behavior involved.

## Interview pattern

When reviewing an issue, explain the reasoning in an interview-style pattern: identify the observed behavior, explain why it matters, state the evidence, and give the smallest actionable recommendation.

Focus on evidence from the repository rather than speculation.
