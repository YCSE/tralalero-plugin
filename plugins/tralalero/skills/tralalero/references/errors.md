# Failures and what to do

| Signal | Meaning | Next step |
| --- | --- | --- |
| HTTP `401` | This client's Tralalero sign-in is missing or expired, or the personal access token fallback (`TRALALERO_MCP_TOKEN`) is missing or revoked. | Complete the OAuth sign-in in the client (Codex `codex mcp login tralalero`; Claude Code `/mcp`, then Authenticate), or re-wire the token. |
| `work_prompt_pending` | The card's AI estimate is still running. | Wait, then call `get_work_prompt` again. |
| `work_prompt_changed` | The card changed while the handoff was being assembled. | Call `get_work_prompt` again. |
| `requestChanged: true` | The customer's request changed since the fingerprint you passed. The move or submission still happened. | Read the handoff again and handle the new material. |
| Not found, with archive guidance | Completed cards from earlier months move to monthly archives. | Do not retry; the result will not change. |
| `start_work` refused for the column | Request, Urgent and Review cards can start; a Review card goes back to In progress. A Done card was confirmed by the customer. | Leave a Done card where it is; take further changes as a new request. |
| Comment or question refused | The customer-text guard found technical content. | Rewrite it for the customer; see `references/customer-text.md`. |
| `ask_customer` refused | A question is already unanswered, an AI clarification is open, you are the requester, or the card is done. | Wait for the reply, or use `add_comment`. |
| `delete_comment` refused | A rework reason or an automatic record stays. Not found means that `commentId` is not on the card. | Leave a protected record, or correct it with `add_comment`. For not found, take the `commentId` from `get_card` again. |
| `delete_comment` returns `requestChanged: true` | You withdrew a customer's comment, so the request changed. `alreadyWithdrawn: true` instead means nothing changed. | Read the handoff again. A work plan that includes the card may need a new plan. |
| `submit_for_review` on a Review card: `commentWithdrawn: true`, or refused | The card is already in review. Its note with this text was withdrawn, or no note matches; nothing was posted. | Post the corrected note with `add_comment`; the card stays in review. |
| Scope start or submit refused | Nothing moved: a criterion, a card comment or a card state did not match. | Fix the input from `get_work_plan_scope` and retry; see `references/work-plan.md`. If a submitted scope has a card the customer confirmed as Done, it cannot be reopened; do not retry. |

## Attachment links

- `get_card` returns a signed `url` per attachment, valid until `attachmentUrlsExpireAt`. After that, call `get_card` again for new links instead of reusing old ones.
- In `get_card`, a `url` of `null` while `attachmentUrlsExpireAt` is set means that one file failed to sign. If `attachmentUrlsExpireAt` is `null` and `attachmentUrlsUnavailableReason` is set, the whole batch failed; call again to retry (`attachmentUrlsNote` says the same in one sentence). With neither set, this environment does not issue links, and retrying will not help.
- In `get_work_prompt`, the download appendix at the end of the handoff carries the links (valid for one hour). `structured.attachmentUrls.issued` says whether links were actually included. `issued: false` with an `unavailableReason` (and a one-sentence `note`) means issuing failed, so call again. Without a reason, this environment does not issue links; download from the workboard card instead.

## postedCommentId

`postedCommentId` only recovers a partial failure from a server before 2.2, using the real comment ID that server returned. Never invent one, and leave it out on a normal first submission.
