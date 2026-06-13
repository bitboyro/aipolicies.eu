---
name: eu-regulations
description: >
  Use this skill when a task requires primary-source answers about the EU AI Act (Regulation 2024/1689),
  GDPR (Regulation 2016/679), or the Data Act (Regulation 2023/2854) — article lookups, cross-regulatory
  analysis, compliance assessments, obligation inventories, or any question whose answer must be grounded
  in the legislative text rather than a model's training-time summary.
---

## When to use

Trigger this skill for any of the following:

- **Article lookup** — "What does Article 50 of the AI Act say about transparency?"
- **Obligation inventory** — "Which articles impose obligations on providers of high-risk AI systems?"
- **Cross-regulatory analysis** — "Where does the AI Act reference GDPR data-protection obligations?"
- **Compliance scoping** — "Does this use case fall under high-risk AI classification?"
- **Dimension-guided retrieval** — "Find all articles across the three regulations that relate to enforcement and liability."
- **Corpus overview** — "What regulations are indexed and how current is the data?"

Do not use this skill for questions about non-EU law, case law, soft law (guidelines, opinions), or content that falls outside the three indexed regulations.

---

## Connect

**Endpoint:** `https://mcp.aipolicies.eu/mcp`
Transport: Streamable HTTP. No authentication required.

### Claude Code
```
claude mcp add eu-regulations --transport http https://mcp.aipolicies.eu/mcp
```

### Claude Desktop
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

### GitHub Copilot / VS Code
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

## Tools

### Discovery

| Tool | Purpose | Key params |
|---|---|---|
| `list_dimensions` | List all dimensions tagged in the corpus with node counts. Run first to discover valid dimension names before calling `search_by_dimension`. | — |
| `get_dimensions` | Full schema for one or more dimensions: description, allowed values with counts, examples. | `names` (comma-separated) |
| `list_regulation_documents` | Paginated list of indexed documents with metadata. Use to discover `documentId` UUIDs. | `page`, `size`, `scope` |
| `list_regulation_nodes` | Paginated node listing. Use to discover `nodeId` strings when search is not appropriate. | `page`, `size`, `documentId`, `scope` |
| `get_regulation_toc` | Table of contents for one regulation: chapters, sections, articles with `nodeId`s. Fastest way to map structure before fetching nodes. | `regulation` (`euaiact`\|`gdpr`\|`data-act`), `includeArticles` |
| `get_knowledge_base_report` | Corpus-level health: document count, graph edge count, dimension coverage, source types. | — |

### Search

| Tool | Purpose | Key params |
|---|---|---|
| `search_regulations` | **Semantic (vector) search** — best for natural-language questions and conceptual queries. Returns `nodeId`, `documentId`, similarity score, and a snippet. | `queries` (up to 5, comma-separated), `scope`, `topK` |
| `search_regulations_text` | **Full-text (Lucene) search** — best for exact terms, article numbers, defined terms. | `queries` (up to 5, comma-separated), `scope`, `topK` |
| `search_by_dimension` | **Dimension-guided search** — retrieve all nodes tagged with a specific dimension value. Most token-efficient for set-style queries ("all transparency articles"). Use `cross-act-topic` to find articles sharing a regulatory theme across all three regulations. | `dimensionName`, `value`, `scope` |

### Retrieval

| Tool | Purpose | Key params |
|---|---|---|
| `get_regulation_nodes` | Batch-fetch node content and metadata by `nodeId`. Returns legal text, citation, graph context (cross-regulation edges), dimensions, AST children. | `nodeIds` (up to 20, comma-separated), `include` (`content,citation,dimensions,graphContext,astChildren`), `graphTypes` |
| `get_regulation_documents` | Batch-fetch documents by `documentId` UUID. Returns metadata, node list, dimensions. | `documentIds`, `includeContent` |

**`get_regulation_nodes` defaults:** the response includes `content` + `citation` only. Add `dimensions` and `graphContext` to `include` only when needed — they add significant token cost per node.

### Feedback

| Tool | Purpose | Key params |
|---|---|---|
| `report_error` | Report unexpected results, data quality issues, or tool failures to the server operators. | `errorMessage`, `toolName`, `context` |

---

## Scope values

All search and listing tools accept an optional `scope` parameter:

| Value | Content |
|---|---|
| `all` | Everything (default) |
| `euaiact` | EU AI Act only |
| `gdpr` | GDPR only |
| `data-act` | Data Act only |
| `legislation-news` | Legislative news items |

---

## Workflow

### Standard lookup (semantic → node)
1. `search_regulations(queries="...", scope="euaiact")` — identify candidate `nodeId`s.
2. `get_regulation_nodes(nodeIds="euaiact-article-50", include="content,citation,graphContext")` — read the full article text, citation metadata, and cross-regulation edges.

### Structural navigation
1. `get_regulation_toc(regulation="gdpr")` — get the chapter/section/article map with `nodeId`s.
2. `get_regulation_nodes(nodeIds="gdpr-article-22")` — fetch specific articles by position.

### Cross-regulatory analysis
1. `search_by_dimension(dimensionName="cross-act-topic", value="data-protection", scope="all")` — find all nodes tagged with the theme across all three regulations.
2. Or: fetch a node with `include="graphContext"` and inspect `regulation-overlap` edges to follow authoritative cross-references.

`graphContext` contains multiple edge types — `regulation-overlap` is the authoritative cross-regulation link; `containment` is structural (chapter → article); `topical` is broader thematic linking. Use `graphTypes="regulation-overlap"` to filter to cross-regulation edges only.

### Dimension-guided retrieval (most token-efficient for set queries)
1. `list_dimensions()` — confirm the dimension name and coverage count.
2. `get_dimensions(names="cross-act-topic")` — inspect allowed values.
3. `search_by_dimension(dimensionName="cross-act-topic", value="transparency-accountability")` — retrieve the matching node set.
4. `get_regulation_nodes(nodeIds="<comma-separated ids from step 3>", include="content,citation")`.

### Corpus orientation (start here for unknown-corpus tasks)
1. `get_knowledge_base_report()` — check what is indexed and how current.
2. `list_dimensions()` — see which dimensions are tagged and how densely.

---

## Citing sources (mandatory)

**Every regulatory claim must include a source citation.** Regulatory text changes; unanchored summaries mislead. Cite before or immediately after the claim.

### How to cite

Each node returned by `get_regulation_nodes` includes citation fields when `include` contains `citation`:

```
citation_short=Art. 50 AI Act
citation_full=Article 50 of Regulation (EU) 2024/1689 (Artificial Intelligence Act)
citation_url=https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32024R1689
```

Use the EUR-Lex `citation_url` as the authoritative source link in your output.

**Citation format:**

> [claim text] — *[citation_short]*, [citation_full] ([citation_url])

**Example:**
> Providers of AI systems that interact with natural persons must disclose that the person is interacting with an AI system — *Art. 50 AI Act*, Article 50 of Regulation (EU) 2024/1689 (Artificial Intelligence Act) (https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32024R1689)

If `citation_url` is absent (e.g. for nodes without a parsed `sourceRef`), use the regulation-level EUR-Lex CELEX link as fallback:
- AI Act: `https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32024R1689`
- GDPR: `https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32016R0679`
- Data Act: `https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32023R2854`

---

## Examples

### 1. Transparency obligations for limited-risk AI
**Prompt:** "What are the transparency obligations for AI systems that interact with people under the EU AI Act?"

**Tool path:**
```
search_regulations(queries="transparency obligations AI systems interacting with people", scope="euaiact", topK=5)
→ get_regulation_nodes(nodeIds="euaiact-article-50", include="content,citation")
```

**What to expect:** The semantic search surfaces Article 50 (and related recitals). Fetching the node returns the full text plus `citation_url` for inline attribution.

---

### 2. GDPR cross-references in the AI Act
**Prompt:** "Which GDPR articles does the AI Act reference in the context of automated decision-making?"

**Tool path:**
```
get_regulation_nodes(nodeIds="euaiact-article-22", include="content,citation,graphContext", graphTypes="regulation-overlap")
```

**What to expect:** The `graphContext` block lists `regulation-overlap` edges from the AI Act node to GDPR nodes (e.g. `gdpr-article-22`), giving you the authoritative cross-references without a search step.

---

### 3. Obligation inventory across all regulations
**Prompt:** "Find all articles related to fundamental rights across the AI Act, GDPR, and Data Act."

**Tool path:**
```
list_dimensions()
→ search_by_dimension(dimensionName="cross-act-topic", value="fundamental-rights", scope="all")
→ get_regulation_nodes(nodeIds="<returned ids>", include="content,citation")
```

**What to expect:** The dimension search returns a node set spanning all three regulations. Fetching nodes gives full text and citations for each.

---

### 4. Structural navigation — full GDPR chapter overview
**Prompt:** "Show me the structure of GDPR Chapter III and give me node IDs for the rights articles."

**Tool path:**
```
get_regulation_toc(regulation="gdpr", includeArticles=true)
```

**What to expect:** The TOC returns chapter/section/article structure with `nodeId` values (e.g. `gdpr-article-15`, `gdpr-article-17`) ready for direct retrieval.

---

### 5. Corpus health check before a broad analysis
**Prompt:** "What is indexed in the knowledge base right now?"

**Tool path:**
```
get_knowledge_base_report()
→ list_regulation_documents(scope="all")
```

**What to expect:** The report gives document count, graph edge count, dimension coverage, and source types. Document listing confirms which regulations are present and their indexing status.
