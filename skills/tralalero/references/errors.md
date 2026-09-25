# Failures and what to do

| Signal | Meaning | Next step |
| --- | --- | --- |
| HTTP `401` | This client's Tralalero sign-in is missing or expired, or the personal access token fallback (`TRALALERO_MCP_TOKEN`) is missing or revoked. | Complete the OAuth sign-in in the client (Codex `codex mcp login tralalero`; Claude Code `/mcp`, then Authenticate), or re-wire the token. |
| `work_prompt_pending` | The card's AI estimate is still running. | Wait, then call `get_work_prompt` again. |
| `work_prompt_changed` | The card changed while the handoff was being assembled. | Call `get_work_prompt` again. |
| `requestChanged: true` | The customer's request changed since the fingerprint you passed. The move or submission still happened. | Read the handoff again and handle the new material. |
| Not found, with archive guidance | Completed cards from earlier months move to monthly archives. | Do not retry; the result will not change. |
| `start_work` refused for the column | Only Request and Urgent cards can start. A card in Review returns only when the customer asks for rework. | Leave the card where it is. |
| Comment or question refused | The customer-text guard found technical content. | Rewrite it for the customer; see `references/customer-text.md`. |
| `ask_customer` refused | A question is already unanswered, an AI clarification is open, you are the requester, or the card is done. | Wait for the reply, or use `add_comment`. |
| Scope start or submit refused | Nothing moved: a criterion, a card comment or a card state did not match. | Fix the input from `get_work_plan_scope` and retry; see `references/work-plan.md`. |

## Attachment links

- `get_card` returns a signed `url` per attachment, valid until `attachmentUrlsExpireAt`. After that, call `get_card` again for new links instead of reusing old ones.
- In `get_card`, a `url` of `null` while `attachmentUrlsExpireAt` is set means that one file failed to sign. If `attachmentUrlsExpireAt` is `null` and `attachmentUrlsUnavailableReason` is set, the whole batch failed; call again to retry (`attachmentUrlsNote` says the same in one sentence). With neither set, this environment does not issue links, and retrying will not help.
- In `get_work_prompt`, the download appendix at the end of the handoff carries the links (valid for one hour). `structured.attachmentUrls.issued` says whether links were actually included. `issued: false` with an `unavailableReason` (and a one-sentence `note`) means issuing failed, so call again. Without a reason, this environment does not issue links; download from the workboard card instead.

## postedCommentId

`postedCommentId` only recovers a partial failure from a server before 2.2, using the real comment ID that server returned. Never invent one, and leave it out on a normal first submission.
