# Kalpana One agent plugin

Let Claude, ChatGPT, Cursor or any MCP client fill your [Kalpana One](https://kalpana.one) templates from a list or spreadsheet and render personalized banners, ads and listing images in bulk.

This repository holds the plugin, the `create-creative-batch` skill and the client configuration for the hosted Kalpana One MCP server at `https://mcp.kalpana.one/mcp`. It contains no credentials: every user signs in with their own Kalpana account.

## Install

**Claude Code**

```
/plugin marketplace add shreekaramtech/kalpana-one-agent
/plugin install kalpana-one@kalpana-one
```

Or connect the server alone:

```
claude mcp add --transport http kalpana https://mcp.kalpana.one/mcp
```

**Claude, ChatGPT and other clients**

Add a custom connector with the URL `https://mcp.kalpana.one/mcp`. The client opens Kalpana's sign-in page the first time it connects.

**Any MCP client (JSON config)**

```json
{
  "mcpServers": {
    "kalpana": {
      "type": "http",
      "url": "https://mcp.kalpana.one/mcp"
    }
  }
}
```

## Setup

1. Create a free account at [app.kalpana.one](https://app.kalpana.one/signup). New accounts get free render credits.
2. Add a template to a workspace (the Kalpana app explains how).
3. Connect the plugin. Sign-in is OAuth: you choose which workspaces the agent may see, and your role in each workspace decides what it can do. No API key is pasted anywhere.

## What the agent can do

The server exposes 15 tools:

| Tool | What it does |
| --- | --- |
| `list_workspaces` | List the workspaces you connected |
| `search_templates` | Find templates by name or size, or browse the latest |
| `get_template_inputs` | Describe a template's fillable text and image inputs |
| `list_asset_collections` | List image collections |
| `search_assets` | Find images in the workspace asset library |
| `inspect_image` | View a template preview, asset or rendered image |
| `validate_batch` | Check rows against the template before anything is created |
| `create_batch` | Create a draft batch (renders nothing, spends nothing) |
| `update_batch` | Edit a draft batch: rename it, or add, change or remove rows |
| `update_job` | Edit one pending row |
| `run_batch` | Render a batch (spends render credits) |
| `get_batch` | Check a batch's progress |
| `get_batch_jobs` | List a batch's rows with their status |
| `list_batches` | List recent batches |
| `get_batch_outputs` | Get download links for rendered images |

The `create-creative-batch` skill walks an agent through the whole flow: find a template, map the user's data to its inputs, validate, create a draft, run it after confirming the credit cost, and share the results.

Still images only (PNG, JPEG, WebP) through this connection. Animated templates and design edits stay in the Kalpana app.

## Layout

```
.claude-plugin/plugin.json        Claude plugin manifest
.claude-plugin/marketplace.json   lets this repository act as its own Claude marketplace
.codex-plugin/plugin.json         Codex and ChatGPT plugin manifest
.mcp.json                         Claude Code server config (hosted endpoint, OAuth)
plugin.json, mcp.json             agent-plugins.org manifest and config (Cursor)
skills/create-creative-batch/     the agent skill
assets/                           logos
```

## Links

- Product: https://kalpana.one
- MCP server: https://kalpana.one/mcp
- Setup docs: https://docs.kalpana.one/mcp/quickstart
- Support: support@kalpana.one
- Privacy: https://kalpana.one/privacy

## License

MIT
