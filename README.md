# Tralalero workboard plugin

Work Tralalero customer change-request cards from an AI coding agent without leaving the workboard workflow.

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

The package also includes one `tralalero` skill that defines the safe end-to-end card cycle.

## Verify

After configuring the token, call `list_boards`. A successful response lists the workboards your Tralalero account can access.

This repository contains no secrets and no server code.

## License

MIT
