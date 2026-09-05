---
name: tralalero
description: Use when working cards on a Tralalero workboard — reading a customer's
  change request, starting work, reporting back for review, or asking the customer a
  blocking question. Triggers on "Tralalero", "트랄랄레로", "workboard", "워크보드",
  "WB-1"/"WB-n", "카드 작업", "수정요청", "검수 요청", a canonical
  /work/{boardId}/cards/{cardId} path or tralalero.app work URL, and on any
  tralalero MCP tool (list_boards, list_cards, get_card, get_work_prompt,
  list_updates, start_work, submit_for_review, add_comment, ask_customer,
  get_work_plan_scope, start_work_scope, submit_scope_for_review).
version: 2.5.0
---

# Tralalero workboard

Non-technical customers submit change requests as cards on a Tralalero workboard. You implement one card or a generated PR/whole-plan scope and use the twelve MCP tools below to read the work, mark progress, communicate safely, and return completed work for review.

## The work prompt is the canon

`get_work_prompt` carries the repository instructions, memory-ledger procedure, Git commands, and attachment handling for that specific card. The first round is a full prompt. A rework round is a delta that carries only the customer evidence changed since the prior review submission and intentionally omits old request text, old comments, old attachments, and old AI guidance. This skill does not duplicate or summarize those instructions. If anything here conflicts with the work prompt, the work prompt wins; never skip reading it in full.

GitHub is optional: MCP card work is available even when the board has no GitHub connection. If the work prompt says a GitHub repository is connected and the work produces commits or a pull request, copy the exact `Tralalero-Work-Ref` and `Tralalero-Request-Fingerprint` trailers from the prompt's identity section into every commit and the PR body, and write the seven-cell history row the prompt requests. Do not invent, shorten, or derive a trailer yourself. Tralalero never verifies commits and never writes to the repository after the one-time ledger setup: the review request is a completion notice, and the branch, commit SHA, and PR number you pass to `submit_for_review` are optional records kept on the card's server-side work round (no screen renders them yet; the next rework prompt cites the SHA or PR as the prior result).

## Tools

| Tool | Purpose | Required selector |
| --- | --- | --- |
| `list_boards` | List accessible boards, roles, and locales. | None |
| `list_cards` | List active cards or poll a board view. | Exact `boardId` |
| `get_card` | Read the complete request, comments, rework reason, and attachments. | Exact `workRef` |
| `get_work_prompt` | Read the canonical instructions for one card. | Exact `workRef` |
| `get_work_plan_scope` | Read a generated PR or whole-plan prompt and PASS checklist. | Exact `scopeRef` |
| `list_updates` | Poll changes since an ISO 8601 cursor. | Exact `boardId` |
| `start_work` | Move a request into progress immediately before editing. | Exact `workRef` |
| `start_work_scope` | Atomically move every card in a PR or whole plan into progress. | Exact `scopeRef` |
| `submit_for_review` | Post the customer-facing completion note and request review. | Exact `workRef` |
| `submit_scope_for_review` | Atomically post per-card notes and move a fully verified scope to review. | Exact `scopeRef` + exact checklist/card coverage |
| `add_comment` | Add a customer-facing comment without moving the card. | Exact `workRef` |
| `ask_customer` | Ask one genuinely blocking customer question. | Exact `workRef` |

`workRef` is only the canonical path `/work/{boardId}/cards/{cardId}` or the same `https://tralalero.app/work/{boardId}/cards/{cardId}` URL. It is the sole card selector. Never infer a board or card from the current directory, Git remote, repository name, a header default, or a bare `WB-n` number.

## Entry modes

### Direct pasted card link

When the user pastes a canonical path or `https://tralalero.app/work/{boardId}/cards/{cardId}` URL, preserve that exact value as `workRef`. Reuse that unchanged `workRef` for all six card tools: `get_card`, `get_work_prompt`, `start_work`, `submit_for_review`, `add_comment`, and `ask_customer`. Do not call `list_boards` to reinterpret it and do not extract or reconstruct IDs from it. For later polling, retain `board.boardId` from either direct read; for customer-facing comments and questions, retain `get_work_prompt.structured.locale`.

### Autonomous board selection

1. Call `list_boards` and select a board by its returned name, role, and locale.
2. Call `list_cards` with that exact returned `boardId`.
3. Select a returned card and retain its `workRef` unchanged for `get_card`, `get_work_prompt`, `start_work`, `add_comment`, `ask_customer`, and `submit_for_review`.
4. For polling, call `list_updates` with the same explicit `boardId`; retain the returned `now` as the next `since` cursor.

WB numbers restart at 1 for each board. They are human references only, never input selectors.

`scopeRef` is only `/work/{boardId}/plans/{planId}` for a whole plan,
`/work/{boardId}/plans/{planId}/units/{unitId}` for one PR, or the same paths on
the exact `https://tralalero.app` origin. Preserve the copied value unchanged;
never infer it from a card, repository, PR title, or current directory.

## The work-plan cycle

1. Preserve the exact `scopeRef` embedded in the copied long-running prompt.
2. Call `get_work_plan_scope`, read its complete prompt, requirements, dependencies, and PASS checklist.
3. For every returned `workRef`, call both `get_card` and `get_work_prompt` and read both responses in full before starting. This hydrates the complete customer request, comments, rework reason, attachments, and card-specific repository instructions; follow each work prompt's attachment procedure before editing.
4. Call `start_work_scope` immediately before the first edit. Passing each card's exact `workRef` and `expectedRequestFingerprint` from the prompt responses is optional; when the response lists a card in `requestChanged`, re-read that card's prompt. The first start locks the plan to whole-plan or per-PR execution; never mix those modes.
5. Implement the scope and verify every criterion. The plan lists a suggested PR/dependency order; follow it when you can, but a preceding PR that has not started does not block a later one. Do not mark a criterion PASS without concrete evidence.
6. Before submitting, re-read every card's `get_work_prompt`; if a `requestFingerprint` differs from the one you started with, read and address the new evidence first. Then call `submit_scope_for_review` with exactly one evidence entry for every returned `criterionId` and exactly one customer-facing completion comment for every returned card `workRef`. When the work produced commits, you may add one `gitSubmissions` entry per card with the branch, result commit SHA, and PR number as a record; every field is optional and nothing is verified.

The scope write is all-or-zero: every grouped card enters progress/review together, or none do. A stale plan, one missing criterion, one missing card comment, or one invalid card state rejects the whole transition. Successful retries are idempotent and do not add duplicate comments.

## The work cycle

1. Enter through one of the two modes above and obtain the exact `workRef`.
2. Call `get_card` and read the entire request, all comments, the rework reason, and every attachment.
3. Call `get_work_prompt`, read it in full, and follow it exactly. When the card has attachments, the prompt ends with a download appendix of time-limited signed URLs — run its `curl` lines to save the files under `/tmp/workboard-att/`, then open them with the Read tool (read images and PDFs with vision). `get_card` also returns a signed `url` per attachment, valid until `attachmentUrlsExpireAt`; when a link has expired, call the tool again to reissue instead of reusing the old URL.
4. Call `start_work` immediately before touching the code. Passing `expectedRequestFingerprint` from step 3 is optional; if the response says `requestChanged: true`, re-read `get_work_prompt` before editing.
5. Implement and verify the requested change under the work prompt's repository rules.
6. Before submitting, re-read `get_work_prompt`; if its `requestFingerprint` differs from the one you started with, read and address the new evidence first. Then call `submit_for_review` with a clear customer-facing verification note. When the work produced commits, add the branch, full result commit SHA, and PR number as optional records; if the response still says `requestChanged: true`, the customer changed the request during submission — re-read `get_work_prompt` and follow up.

## Comments the guard accepts

Write two to four plain sentences in the board locale. Autonomous entry obtains it from `list_boards`; direct-link entry obtains it from `get_work_prompt.structured.locale`. Explain what changed and which screen or control the customer should use to verify it.

The server rejects code fences and inline backticks, diff hunks containing `@@`, stack traces, shell commands, file paths shaped like `a/b/c` or `name.ext`, code symbols in camelCase, snake_case, or SCREAMING_SNAKE, paired SQL keywords, GitHub, localhost, or `file://` links, and secret-shaped values. Ordinary preview links are allowed, as are human UI directions such as “Settings > Notifications.” Questions sent through `ask_customer` have stricter rules described below. If a comment is rejected, rewrite it as language a customer can understand instead of mechanically deleting punctuation.

The same rule applies independently to every `cardComments` item in `submit_scope_for_review`. Write a relevant 2–4 sentence note for each card rather than copying one generic note across the scope.

## ask_customer

Use `ask_customer` only for blocking ambiguity: you have read the card, work prompt, comments, and attachments, still cannot choose safely, and a wrong choice would cause rework. Ask one question at a time, on one line of at most 120 characters, in the board locale.

Questions are STRICTER than comments: no links, no symbols like `>`, no Latin-word(parenthetical) identifiers — write one plain sentence.

The question appears on the customer's lock screen. Use it only when the interruption costs less than the risk of discarding the card's work. A new question is rejected while another is unanswered; check the reply with `list_updates` using the explicit selected `boardId` and `get_card`, and distinguish your own question from the customer's reply by update kind. Autonomous entry gets that boardId from `list_boards`; direct-link entry gets it from `board.boardId` returned by `get_card` or `get_work_prompt`. If the result says `queued: false`, alert a person through the approved operational channel. Even when `queued: true`, a long silence can mean the notification never reached a registered device, so confirm through that operational channel.

Do not use it for a choice you can make yourself—make the choice and explain it in `submit_for_review`—or for progress reporting, which belongs in `add_comment`. A conflict between AGENTS.md or CLAUDE.md and repository memory is never a reason to ask the customer.

## When something fails

If an error includes archive guidance, retrying is pointless because older completed cards move to monthly archives. A 401 means the token is missing or revoked. `postedCommentId` is only for recovering a real comment ID returned by a pre-2.2 server; never invent one. If a comment is rejected, rewrite it for the customer under the guard rules above.
