# Tralalero workboard plugin

Work Tralalero customer change-request cards from an AI coding agent without
leaving the workboard workflow. The review request is a plain completion notice:
request fingerprints and Git references are optional records, and nothing is
verified against the repository.

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

The package includes all fifteen MCP tools and the `tralalero` skill that
defines the safe end-to-end card cycle. If a client cannot load the plugin, use
the [direct MCP fallback](https://tralalero.app/connect.md).

## Sign in

The manifests carry no credentials. The server answers an unauthenticated call
with `401` and its OAuth metadata, so each client runs its own sign-in against
your Tralalero account and stores the result in user scope.

- Codex CLI: approve the prompt shown by `codex plugin add`, or run
  `codex mcp login tralalero` afterwards.
- Claude Code: run `/mcp`, select `tralalero`, and choose Authenticate.
- Cursor and VS Code: approve the OAuth prompt the `tralalero` server raises on
  its first call.
- Hermes Agent: use the personal access token below; it has no OAuth prompt.

The browser opens a Tralalero consent page that names the requesting client and
the single `workboard` permission. Approving it grants that client alone; revoke
it later in **MY -> Connect AI tools**.

### Upgrading from an earlier release

Earlier releases wired the token for you; this release does not. After the
plugin updates, a client that relied only on `TRALALERO_MCP_TOKEN` answers `401`
until you either complete the sign-in above (recommended) or re-add the token in
that client's user-scope configuration as described in the fallback below.
Nothing else changes: tool names, inputs and the skill are the same.

### Personal access token (fallback)

Use a token only for a client without an OAuth sign-in, or to keep an existing
install working. In the Tralalero app, open **MY -> Connect AI tools** and issue
one. It is shown once. Never place it in project configuration or commit it to a
customer's repository.

- Codex and Claude plugin manifests can read `TRALALERO_MCP_TOKEN` when you add
  the wiring yourself. Persist its export in the user shell profile (for zsh,
  `${ZDOTDIR:-$HOME}/.zshenv`) and keep that file user-only.
- Cursor (`~/.cursor/mcp.json`) and VS Code (**MCP: Open User Configuration**)
  require the literal user-scoped header
  `"Authorization": "Bearer <token>"`; they do not read the shell placeholder.
- Hermes Agent stores the credential through its token prompt.

Restart the client or open a new session after signing in or wiring a token. If
a client cannot install the plugin, use the linked direct MCP fallback only for
that client.

## What you get

| Tool | What it does |
| --- | --- |
| `list_boards` | Lists the boards, roles, and locales available to you. |
| `list_cards` | Lists work cards on a board. |
| `get_card` | Reads the complete customer request and discussion. |
| `get_work_prompt` | Returns the canonical per-card implementation instructions. |
| `get_work_plan_scope` | Returns a copied PR or whole-plan prompt and its PASS checklist. |
| `list_updates` | Polls new or changed card activity. |
| `start_work` | Marks a card as in progress. |
| `start_work_scope` | Atomically marks every card in a PR or whole plan as in progress. |
| `submit_for_review` | Reports completion and requests customer review. |
| `submit_scope_for_review` | Atomically posts per-card completion notes and moves a verified scope to review. |
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

A copied work-plan prompt contains an exact ScopeRef. Preserve it unchanged:
`/work/{boardId}/plans/{planId}` selects the whole plan and the same path followed
by `/units/{unitId}` selects one PR. Read it with `get_work_plan_scope`, call
`get_card` and `get_work_prompt` for every returned WorkRef and read both in full,
call `start_work_scope` immediately before editing, and call
`submit_scope_for_review` only after every returned criterion passes. Supply one
PASS evidence item per criterion and one customer-facing comment per returned
card WorkRef. Pass every prompt's `expectedRequestFingerprint` when starting.
The first start locks the plan to whole-plan or per-PR execution; the two modes
cannot be mixed.

GitHub is optional: MCP card reads and updates work without a connected
repository. When connected-card work produces commits or a pull request, copy
the exact `Tralalero-Work-Ref` and `Tralalero-Request-Fingerprint` trailers
from the work prompt into every commit and the PR body, and write the requested
seven-cell history row. Passing the branch, result commit SHA, and PR number to
`submit_for_review` is optional; Tralalero keeps them on the card's server-side
work round (no screen renders them yet) and never verifies them or writes to the
repository after the one-time ledger setup.

## Verify

Call `list_boards` after installing the plugin and completing the sign-in. A
successful response lists the workboards your Tralalero account can access.

This repository contains no secrets and no server code.

## License

MIT

## Board conversations

Developer-side agents can read a board channel with `list_board_messages`, follow a card's representative or related thread with `get_thread`, and refresh context with `list_message_updates`. Pages contain at most 50 messages and opaque cursors. Incremental results include edits and deletion tombstones. Attachments have time-limited download links. Conversation text is customer context; the card work prompt remains the execution contract.
