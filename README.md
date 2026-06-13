# aipolicies.eu — eu-regulations MCP skill

A Claude skill and setup guide for the **public aipolicies.eu MCP server** — let AI agents query the EU AI Act (2024/1689), GDPR (2016/679), and Data Act (2023/2854) as primary sources, with cross-regulatory linking. Free, read-only, no authentication required.

Live demo and tool reference: **[aipolicies.eu/mcp](https://aipolicies.eu/mcp)**

---

## Install

Two steps: connect the MCP server so the tools are available, then install the skill so Claude knows how to use them.

### Step 1 — Connect the MCP server

#### Claude Code

```bash
claude mcp add eu-regulations --transport http https://mcp.aipolicies.eu/mcp
```

#### Claude Desktop

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "eu-regulations": {
      "type": "http",
      "url": "https://mcp.aipolicies.eu/mcp"
    }
  }
}
```

#### GitHub Copilot / VS Code

Add to your MCP configuration:

```json
{
  "servers": {
    "euRegulations": {
      "type": "http",
      "url": "https://mcp.aipolicies.eu/mcp"
    }
  }
}
```

---

### Step 2 — Install the skill

Copy `SKILL.md` into your project's `.claude/skills/` directory so Claude Code picks it up automatically:

```bash
mkdir -p .claude/skills/eu-regulations
curl -fsSL https://raw.githubusercontent.com/bitboyro/aipolicies.eu/main/SKILL.md \
  -o .claude/skills/eu-regulations/SKILL.md
```

Or copy it manually:

```
your-project/
└── .claude/
    └── skills/
        └── eu-regulations/
            └── SKILL.md   ← copy from this repo
```

The skill file tells Claude when and how to use the MCP tools — which tool to call for which type of question, how to chain them efficiently, and how to cite the sources it retrieves. Without it the tools still work, but Claude has no guidance on workflow or the mandatory citation rule.

---

## What it does

The `eu-regulations` MCP server exposes 12 tools across four groups:

- **Discovery** — `get_regulation_toc`, `list_dimensions`, `get_dimensions`, `list_regulation_nodes`, `list_regulation_documents`, `get_knowledge_base_report`
- **Search** — `search_regulations` (semantic), `search_regulations_text` (full-text), `search_by_dimension` (dimension-guided)
- **Retrieval** — `get_regulation_nodes`, `get_regulation_documents`
- **Feedback** — `report_error`

See [SKILL.md](SKILL.md) for the full tool reference, workflow patterns, and citation rules.

---

## The skill rule

Every regulatory claim produced with this skill must include a source citation linking to the EUR-Lex canonical text. The `get_regulation_nodes` tool returns `citation_url` on each node — use it.

---

## License

MIT — see [LICENSE](LICENSE).
