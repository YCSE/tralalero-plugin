# Tralalero workboard plugin

Work Tralalero customer change-request cards from an AI coding agent without
leaving the workboard workflow. Version 2.1 adds native Codex packaging while
continuing to install the remote MCP tools and canonical WorkRef skill together.

## Install

For Codex CLI 0.147.0 or newer, use the native marketplace installer:

```sh
codex plugin marketplace add YCSE/tralalero-plugin
codex plugin add tralalero@tralalero
```

Claude Code can use its native user-scoped marketplace installer:

```sh
claude plugin marketplace add YCSE/tralalero-plugin --scope user
claude plugin install tralalero@tralalero --scope user
```

Cursor and VS Code can use the universal installer while targeting only the
selected client:

```sh
npx plugins add YCSE/tralalero-plugin --target cursor --scope user
npx plugins add YCSE/tralalero-plugin --target vscode --scope user
```

To install into every supported client detected on the current machine:

```sh
npx plugins add YCSE/tralalero-plugin --scope user
```

The package includes both the nine MCP tools and the `tralalero` skill that
defines the safe end-to-end card cycle. If a client cannot load the plugin, use
the [direct MCP fallback](https://tralalero.app/connect.md).

## Token

In the Tralalero app, open **MY -> Connect AI tools** and issue a token. It is
shown once. Never place it in project configuration or commit it to a
customer's repository.

- Codex and Claude plugin manifests can read `TRALALERO_MCP_TOKEN`. Persist its
  export in the user shell profile (for zsh, `${ZDOTDIR:-$HOME}/.zshenv`) and
  keep that file user-only. The app's copied setup does this without
  overwriting the rest of the profile.
- Cursor (`~/.cursor/mcp.json`) and VS Code (**MCP: Open User Configuration**)
  require the literal user-scoped header
  `"Authorization": "Bearer <token>"`; they do not read the shell placeholder.
- Hermes Agent stores the credential through its token prompt.

Restart the client or open a new session after wiring the token. If a client
cannot install the plugin, use the linked direct MCP fallback only for that
client.

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

`list_cards` and `list_updates` require an explicit board ID. Every card tool
uses the exact returned WorkRef (`/work/{boardId}/cards/{cardId}`); a pasted
canonical Tralalero card URL is also a WorkRef. Preserve that value unchanged
for `get_card`, `get_work_prompt`, `start_work`, `submit_for_review`,
`add_comment`, and `ask_customer`. Its direct reads return `board.boardId` for
polling, while `get_work_prompt.structured.locale` is the customer-comment
language. Never select a card from a current repository, Git remote, or `WB-n`
number.

GitHub is optional: MCP card reads and updates work without a connected
repository. When a connected-card work prompt results in commits or a pull
request, copy the exact
`Tralalero-Work-Ref: /work/{boardId}/cards/{cardId}` trailer from that prompt
into every commit and the PR body.

## Verify

Call `list_boards` after installing the plugin and configuring the token. A
successful response lists the workboards your Tralalero account can access.

This repository contains no secrets and no server code.

## License

MIT
