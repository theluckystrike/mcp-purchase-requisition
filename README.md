# mcp-purchase-requisition

<!-- mirror-seo:start -->

**MCP server for purchase requisitions you build once and approve per order, with the signed record of each approval.** Purchase requisitions you build once and approve per order, with the signed record of each approval.

Works with Claude Desktop, Claude Code, Cursor and any Model Context Protocol client. Runs on your own machine, or hosted with no install.

Product page: <https://mcp.zovo.one/s/purchase-requisition> — what it does, the tools it exposes, and a live token endpoint.

## Install

**Hosted, nothing to install.** Get a token from <https://mcp.zovo.one/mcp/connect> (the connect page) or <https://mcp.zovo.one/mcp/token> (the same token as JSON); a free anonymous one is issued on the spot and a Pro key works the same way. Then point an MCP client at `https://mcp.zovo.one/mcp/purchase-requisition` over streamable-http and send the token as `Authorization: Bearer <token>`.

If your client cannot set headers, put the token in the path instead: `https://mcp.zovo.one/mcp/purchase-requisition/t/<token>`. Both forms work. The bare URL with no token answers 401 on `tools/call`, so the token is not optional.

**Claude Desktop, one click.** Download `purchase-requisition.mcpb` from the [latest release](https://github.com/theluckystrike/mcp-servers/releases/latest) and double-click it.

**From source.** The mirror is self-contained: every `@theluckystrike/*` dependency is vendored, so a fresh clone builds with no extra setup.

```sh
git clone https://github.com/theluckystrike/mcp-purchase-requisition.git
cd mcp-purchase-requisition
npm install && npm run build
```

Then point your client at the built entry point:

```json
{
  "mcpServers": {
    "purchase-requisition": {
      "command": "node",
      "args": ["/absolute/path/to/mcp-purchase-requisition/dist/index.js"]
    }
  }
}
```

> `@theluckystrike/mcp-purchase-requisition` is **not published on npm yet**, so an `npx -y @theluckystrike/mcp-purchase-requisition` command will fail. The three paths above are the working ones and each is exercised by CI.

![purchase-requisition demo](https://raw.githubusercontent.com/theluckystrike/mcp-servers/main/assets/demo-purchase-requisition.gif)

Read-only mirror of [mcp-servers/servers/purchase-requisition](https://github.com/theluckystrike/mcp-servers/tree/main/servers/purchase-requisition). See [MIRROR.md](MIRROR.md).

<!-- mirror-seo:end -->

[![theluckystrike/mcp-purchase-requisition MCP server](https://glama.ai/mcp/servers/theluckystrike/mcp-purchase-requisition/badges/score.svg)](https://glama.ai/mcp/servers/theluckystrike/mcp-purchase-requisition)

**In the [official MCP Registry](https://registry.modelcontextprotocol.io/v0.1/servers/io.github.theluckystrike%2Fpurchase-requisition/versions/latest)** (`io.github.theluckystrike/purchase-requisition`).
Checklists you build once and run many times, and the dated record of each run that
somebody signs. A purchase-requisition is a named list of steps, optionally grouped into sections, each
one required or optional. A run is one pass of that purchase-requisition against a job: every step is
marked pass, fail or not applicable, with who marked it and on what day, and a note that
says what was found. `run_sign_off` then puts a name and a date on it and freezes it.

## The one rule that decides everything else

If somebody edits the purchase-requisition afterwards,
adds a step or deletes one, every run already in progress keeps the list it started with,
and the version it was copied from is recorded on the run.

That is not a caching convenience. A purchase-requisition somebody ticked and signed has to be the list
they actually saw. A run that read its steps live from the purchase-requisition would mean a signed
handover certificate for eleven checks when the person signing it saw ten, with no field in
the record showing that it had happened. It also means deleting a purchase-requisition leaves its runs
readable and complete, which is what you want the year afterwards when somebody asks what
was checked.

Two smaller rules follow from it:

- **Nothing derived is stored.** The pass, fail and outstanding counts, the percentage, and
  whether a run can be signed off are worked out on every call from the run's own steps. A
  stored "complete" flag is a fact about the afternoon somebody last looked, and `complete`
  here is a reading: it appears when the last step is answered and goes away again when one
  is put back to pending.
- **Not applicable is not a pass.** `na` counts as ANSWERED and never as passed. A step that
  was looked at and dismissed is a different fact from a step that passed, and merging the
  two is how a purchase-requisition reports full marks for a job where half the steps did not apply.

## What blocks a signature

A required step that is unanswered, a required step that failed, an unanswered optional
step, or a run with no steps. `force: true` signs anyway, and the exceptions stay on the
record and print on the report under "Signed with exceptions". They are not lost, and they
are not silent.

## Install

One-click (.mcpb): download `purchase-requisition.mcpb` from the latest release and double-click it
in Claude Desktop: https://github.com/theluckystrike/mcp-servers/releases/latest

npm publish for `@theluckystrike/mcp-purchase-requisition` is pending, so the `npx` line below returns
404 today. Build from source in the meantime; see `llms-install.md`.

### Claude Desktop

`~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or
`%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "purchase-requisition": {
      "command": "npx",
      "args": ["-y", "@theluckystrike/mcp-purchase-requisition"]
    }
  }
}
```

### Claude Code

```sh
claude mcp add purchase-requisition -- npx -y @theluckystrike/mcp-purchase-requisition
```

### Cursor

`~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (project), same entry as Claude Desktop.

## Tools

| Tool | What it does |
| --- | --- |
| `purchase-requisition_create` | Create a reusable purchase-requisition: a name, a category, a description |
| `purchase-requisition_item_add` | Add a step: the text, a section heading, and whether it is required |
| `purchase-requisition_item_remove` | Remove a step and bump the version. Runs already under way keep it |
| `purchase-requisition_show` | One purchase-requisition, grouped by section, with a blank printable copy on request |
| `purchase-requisition_list` | Every purchase-requisition with its version, step count and how many runs came from it |
| `purchase-requisition_delete` | Delete a purchase-requisition. Its runs stay readable, because each carries its own copy |
| `run_start` | Start a dated run against a job. The steps are copied into it at this point |
| `run_check` | Mark one step pass, fail or na, with who and when and what was found |
| `run_show` | The run: every step with its answer, the counts, the failures, and what blocks sign-off |
| `run_list` | Runs newest first, filtered by purchase-requisition, status, reference, or only those with failures |
| `run_sign_off` | Sign off with a name and a date, which freezes the run |
| `run_status` | Reopen a complete run, or abandon one when the job did not happen |
| `run_report` | The run as text on every tier. Pro also writes it to `out_path` as a .txt file |
| `run_delete` | Delete a run. A signed-off one is refused |
| `license_status` | Which tier this install is on and where the key came from |
| `license_activate` | Store a Pro key for this server |

There is also a resource, `purchase-requisition://contract`, carrying the snapshot rule, the item
states, the run status machine, what blocks a sign-off and where this server writes; and a
prompt, `run_the_purchase-requisition`, that walks the whole job in order.

## Free vs Pro

| | Free | Pro |
| --- | --- | --- |
| Checklists you keep | 3 | unlimited |
| Runs of them | unlimited | unlimited |
| Steps per purchase-requisition | up to 500 | up to 500 |
| `run_show`, `run_list`, the counts | yes | yes |
| The run report as text | yes | yes |
| Writing the report to a file with `out_path` | no | yes |

The meter is on how many DIFFERENT purchase-requisitions you keep, not on how many jobs you check. A
trade with one pre-delivery check, one handover sheet and one snag list runs its whole year
inside the free tier. Runs are never capped, because capping the running of a purchase-requisition would
cap the only thing a purchase-requisition is for. Deleting a purchase-requisition frees a slot.

Get Pro: https://mcp.zovo.one/buy/purchase-requisition (one-time), or all servers for one price at
https://mcp.zovo.one/buy/bundle

## Privacy

All data stays local, in `${XDG_DATA_HOME:-~/.local/share}/mcp-servers/purchase-requisition/`. There is
no network call anywhere in this server, no API key, and no account. The only file it reads
that it does not own is the shared business profile, for the name and address at the top of a
printed report, and it never writes to it.

Built by [theluckystrike](https://github.com/theluckystrike). Support: support@zovo.one

## Use these docs as an MCP server

Any MCP client (Claude, Cursor, Windsurf, VS Code) can read this repository's documentation directly via GitMCP — no install:

- Docs MCP URL: https://gitmcp.io/theluckystrike/mcp-purchase-requisition
