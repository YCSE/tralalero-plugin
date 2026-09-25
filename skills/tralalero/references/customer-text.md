# Writing to the customer

Customers are not developers. Everything sent through `add_comment`, `submit_for_review`, `submit_scope_for_review` (each `cardComments` entry) and `ask_customer` is shown to them as written.

## Language

Use the board locale. Autonomous entry gets it from `list_boards`. Direct-link entry gets it from `get_work_prompt` `structured.locale`.

## Comments and completion notes

Write two to four plain sentences: what changed, and which screen or control the customer should use to check it.

The server refuses a comment that contains:

- code fences or inline backticks
- diff hunks containing `@@`, stack traces or shell commands
- file paths shaped like `a/b/c` or `name.ext`
- code symbols in camelCase, snake_case or SCREAMING_SNAKE
- paired SQL keywords
- GitHub, localhost or `file://` links
- secret-shaped values

Ordinary preview links are allowed, and so are UI directions such as "Settings > Notifications". When a comment is refused, rewrite it in language the customer understands; do not just strip punctuation until it passes.

## Questions (`ask_customer`)

Questions are stricter than comments because they appear on the customer's lock screen:

- one question, on one line, of at most 120 characters, in the board locale
- at most two options, written into the sentence
- no links, no symbols such as `>`, and no Latin identifiers in parentheses

Ask only when you have read the whole handoff, including comments and attachments, still cannot choose safely, and a wrong choice would mean rework. Make choices you can make yourself and explain them in `submit_for_review`. Progress reports belong in `add_comment`. A conflict between repository documents is never a reason to ask the customer.

Only one unanswered question per card is allowed, and a question is refused while an AI clarification question on the card is still open. After asking, stop work on that card. Watch for the reply with `list_updates` (same `boardId`) and `get_card`; your own question also appears there as a new comment, so tell the two apart by the comment kind and author role.

`queued: false` in the result means no notification could be queued; tell a person through your approved operational channel. Even with `queued: true`, a long silence can mean the notification never reached a device, so check through that channel as well.
