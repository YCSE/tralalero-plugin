# Tralalero workboard plugin

Work Tralalero customer change-request cards from an AI coding agent without leaving the workboard workflow. Version 2 uses a canonical WorkRef for every card action.

## Install

For agents that support Agent Plugins 1.0:

```sh
npx plugins add YCSE/tralalero-plugin
```

For Claude Code:

```sh
claude plugin marketplace add YCSE/tralalero-plugin
claude plugin install tralalero@tralalero
```

(Inside an interactive Claude Code session, the equivalent slash commands are `/plugin marketplace add ...` and `/plugin install ...`.)

## Token

In the Tralalero app, open Settings > MCP connection and issue a token. It is shown once. Store it as the user-scoped environment variable `TRALALERO_MCP_TOKEN`; never place it in project configuration or commit it to a customer's repository.

## What you get

| Tool | What it does |
| --- | --- |
| `list_boards` | Lists the boards, roles, and locales available to you. |
| `list_cards` | Lists work cards on a board. |
| `get_card` | Reads the complete customer request and discussion. |
| `get_work_prompt` | Returns the canonical per-card implementation instructions. |
| `list_updates` | Polls new or changed card activity. |
| `start_work` | Marks a card as in progress. |
| `submit_for_review` | Reports completion and requests customer review. |
| `add_comment` | Adds a customer-facing comment. |
| `ask_customer` | Sends one blocking question to the customer. |

`list_cards` and `list_updates` require an explicit board ID. Every card tool uses the exact returned WorkRef (`/work/{boardId}/cards/{cardId}`); a pasted canonical Tralalero card URL is also a WorkRef. Preserve that pasted value unchanged for all six card tools: `get_card`, `get_work_prompt`, `start_work`, `submit_for_review`, `add_comment`, and `ask_customer`. Its direct reads return `board.boardId` for polling, while `get_work_prompt.structured.locale` is the customer-comment language. Never select a card from a current repository, Git remote, or `WB-n` number.

The package also includes one `tralalero` skill that defines the safe end-to-end card cycle. GitHub is optional: you can read and update MCP cards without a connected repository. When a connected-card work prompt results in commits or a pull request, copy the exact `Tralalero-Work-Ref: /work/{boardId}/cards/{cardId}` trailer from that prompt into every commit and the PR body.

## Verify

After configuring the token, call `list_boards`. A successful response lists the workboards your Tralalero account can access.

This repository contains no secrets and no server code.

## License

MIT
