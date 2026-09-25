# Work plans (PR and whole-plan scopes)

The Tralalero app can turn several cards into a work plan: a set of PR-sized units with requirements, a PASS checklist and a suggested order. The owner copies a long-running prompt for the whole plan or for one PR, and that prompt carries a `scopeRef`.

## Selector

`scopeRef` is only one of these, or the same path on the exact `https://tralalero.app` origin:

- `/work/{boardId}/plans/{planId}` for the whole plan
- `/work/{boardId}/plans/{planId}/units/{unitId}` for one PR

Keep the copied value unchanged. Never build it from a card, repository, PR title or current directory.

## Cycle

1. Call `get_work_plan_scope` with the `scopeRef`. Read the prompt text, then the structured units: their `workRefs`, requirements, dependencies and checklist criteria.
2. The requirements and checklist were drafted by AI from estimate-time material. Each card's `get_work_prompt` handoff carries the customer's own words; where they disagree, follow the customer and say so in that criterion's evidence.
3. For every returned `workRef`, call `get_work_prompt` and read it in full, including its attachments, before any edit.
4. Call `start_work_scope` immediately before the first edit. You may pass `requestFingerprints` (each card's `workRef` with the `expectedRequestFingerprint` from its handoff). A card listed in the response's `requestChanged` has new customer material; read its handoff again.
5. Implement the scope and verify every criterion. Do not mark a criterion PASS without concrete evidence.
6. Before submitting, call `get_work_prompt` again for every card. If a `requestFingerprint` changed, handle the new material first.
7. Call `submit_scope_for_review` with exactly one `criterionEvidence` entry (up to 500 characters) for every returned `criterionId` and exactly one `cardComments` entry for every returned card `workRef`. Each card comment follows the customer-text rules on its own; write a note that fits that card instead of copying one note across the scope. `gitSubmissions` may add, per card, the final request fingerprint, branch, full result commit SHA and PR number as records. Every field there is optional and nothing is verified.

## Rules the server enforces

- The first start locks the plan to either whole-plan or per-PR execution. The two modes cannot be mixed.
- The PR order is a suggestion. A preceding PR that has not started does not block a later one.
- Starts and submissions are all-or-nothing. A stale plan, one missing criterion, one missing card comment or one card in the wrong column rejects the whole call, and nothing moves.
- Retrying a successful `start_work_scope` or `submit_scope_for_review` is safe: it does not move cards twice or add duplicate comments.
