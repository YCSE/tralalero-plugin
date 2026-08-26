---
name: tralalero
description: Use when working cards on a Tralalero workboard — reading a customer's
  change request, starting work, reporting back for review, or asking the customer a
  blocking question. Triggers on "Tralalero", "트랄랄레로", "workboard", "워크보드",
  "WB-1"/"WB-n", "카드 작업", "수정요청", "검수 요청", and on any tralalero MCP tool
  (list_boards, list_cards, get_card, get_work_prompt, list_updates, start_work,
  submit_for_review, add_comment, ask_customer).
version: 1.0.0
---

# Tralalero workboard

Non-technical customers submit change requests as cards on a Tralalero workboard. You implement those requests and use the nine MCP tools below to read the board, mark progress, communicate safely, and return completed work for review.

## The work prompt is the canon

`get_work_prompt` carries the repository instructions, memory-ledger procedure, Git commands, and attachment handling for that specific card. This skill does not duplicate or summarize those instructions. If anything here conflicts with the work prompt, the work prompt wins; never skip reading it in full.

## Tools

| Tool | Purpose | `boardId` |
| --- | --- | --- |
| `list_boards` | List accessible boards, roles, and locales. | No |
| `list_cards` | List active cards or poll a board view. | Optional |
| `get_card` | Read the complete request, comments, rework reason, and attachments. | Optional |
| `get_work_prompt` | Read the canonical instructions for one card. | Required |
| `list_updates` | Poll changes since an ISO 8601 cursor. | Optional |
| `start_work` | Move a request into progress immediately before editing. | Required |
| `submit_for_review` | Post the customer-facing completion note and request review. | Required |
| `add_comment` | Add a customer-facing comment without moving the card. | Required |
| `ask_customer` | Ask one genuinely blocking customer question. | Required |

Always pass an explicit `boardId` to `get_work_prompt` and all four write tools. Header defaults are only a convenience for the other read tools.

## The cycle

1. Call `list_boards` and identify the board by its returned name, role, and locale.
2. Call `list_cards`, or use `list_updates` with the saved cursor when polling.
3. Call `get_card` and read the entire request, all comments, the rework reason, and every attachment.
4. Call `get_work_prompt`, read it in full, and follow it exactly.
5. Call `start_work` immediately before touching the code.
6. Implement and verify the requested change under the work prompt's repository rules.
7. Call `submit_for_review` with a clear customer-facing verification note.

## boardId and WB-n

WB numbers restart at 1 on every board, so `WB-1` is not globally unique. Never infer a board from a card number or the current repository. Always call `list_boards`, choose the verified board, and pass its exact `boardId` wherever required.

## Comments the guard accepts

Write two to four plain sentences in the board locale returned by `list_boards`. Explain what changed and which screen or control the customer should use to verify it.

The server rejects code fences and inline backticks, diff hunks containing `@@`, stack traces, shell commands, file paths shaped like `a/b/c` or `name.ext`, code symbols in camelCase, snake_case, or SCREAMING_SNAKE, paired SQL keywords, GitHub, localhost, or `file://` links, and secret-shaped values. Ordinary preview links are allowed, as are human UI directions such as “Settings > Notifications.” Questions sent through `ask_customer` have stricter rules described below. If a comment is rejected, rewrite it as language a customer can understand instead of mechanically deleting punctuation.

## ask_customer

Use `ask_customer` only for blocking ambiguity: you have read the card, work prompt, comments, and attachments, still cannot choose safely, and a wrong choice would cause rework. Ask one question at a time, on one line of at most 120 characters, in the board locale.

Questions are STRICTER than comments: no links, no symbols like `>`, no Latin-word(parenthetical) identifiers — write one plain sentence.

The question appears on the customer's lock screen. Use it only when the interruption costs less than the risk of discarding the card's work. A new question is rejected while another is unanswered; check the reply with `list_updates` and `get_card`, and distinguish your own question from the customer's reply by update kind. If the result says `queued: false`, alert a person through the approved operational channel. Even when `queued: true`, a long silence can mean the notification never reached a registered device, so confirm through that operational channel.

Do not use it for a choice you can make yourself—make the choice and explain it in `submit_for_review`—or for progress reporting, which belongs in `add_comment`. A conflict between AGENTS.md or CLAUDE.md and repository memory is never a reason to ask the customer.

## When something fails

If an error includes archive guidance, retrying is pointless because older completed cards move to monthly archives. A 401 means the token is missing or revoked. If `submit_for_review` posts the comment but fails to move the card, pass the returned `postedCommentId` back exactly as received. If a comment is rejected, rewrite it for the customer under the guard rules above.
