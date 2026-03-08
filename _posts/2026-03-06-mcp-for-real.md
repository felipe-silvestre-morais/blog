---
title: "MCP: The USB-C Port That AI Was Missing"
date: 2026-03-06
categories: [ai, tech]
tags: [mcp, agent, llm]
---

I know you have been hearing about MCP everywhere lately. Maybe you saw it in a YouTube video, or someone in your team started talking about it, or you noticed it appearing in every AI tool you use. I was in this same position a few months ago — I kind of understood what it was, but I didn't *really* get it. Not until I built one myself.

So let me try to explain what MCP is, how it works, why it matters, and — most importantly — how you can build your own MCP server. We'll do that with a practical example: connecting an AI to a SQL database.

---

## Wait, What Problem Are We Solving?

Before MCP, connecting an AI model to an external tool was kind of a mess. Every integration was custom. You wanted Claude to read your files? Custom code. You wanted it to query your database? Custom code. You wanted to move that integration to a different AI tool? Start over.

This is what people call the **N×M integration problem**. You have N AI applications and M data sources, and you end up writing N×M custom connectors. It's not sustainable.

MCP — the **Model Context Protocol** — solves this by creating a standard. Think of it like USB-C. Before USB-C, every device had its own cable. After USB-C, one port works for everything. MCP does the same for AI integrations: one standard protocol that connects any AI application to any external tool or data source.

Anthropic introduced MCP in November 2024, and honestly, the adoption was faster than I expected. By March 2025, OpenAI also added support across their Agents SDK and the ChatGPT desktop app. Google DeepMind followed. Today, there are thousands of MCP servers available, and the ecosystem is growing every week.

---

## The Architecture (It's Simpler Than You Think)

MCP follows a client-server architecture. If you have ever built a web app, this will feel very familiar.

```
┌─────────────────────────────────────────┐
│           Your AI Application           │
│   (Claude Desktop, Cursor, your app)    │
│                                         │
│         ┌──────────────────┐            │
│         │   MCP Client     │            │
│         └────────┬─────────┘            │
└──────────────────┼──────────────────────┘
                   │ requests / responses
         ┌─────────▼──────────┐
         │    MCP Server      │
         │  (what you build)  │
         │                    │
         │  Tools | Resources │
         │      | Prompts     │
         └────────────────────┘
```

The **MCP Client** lives inside the AI application. When you use Claude Desktop or Cursor, there's already an MCP client built in — you don't need to build one. The client is responsible for discovering what the server can do, requesting data, and executing tools on behalf of the LLM.

The **MCP Server** is what you build. It's an independent process that listens for requests from clients and responds with data or actions. One client can connect to multiple servers simultaneously, so you can mix and match capabilities.

---

## The Three Primitives: Tools, Resources, and Prompts

This is the part I think most explanations skip too quickly, because understanding the difference between these three is actually very important.

### Tools — Model-Controlled Actions

Tools are functions that the AI model can call to **do things**. Think of them like POST endpoints in a REST API. The model decides when to use a tool based on the user's request. Examples: execute a SQL query, send an email, call an external API, create a file.

The key thing about tools is that they can have **side effects**. They can change state, write data, trigger processes. Because of this, the client usually asks for user confirmation before executing a tool.

```python
@mcp.tool()
def run_query(sql: str) -> str:
    """Execute a SQL query against the analytics database and return results."""
    # The docstring matters! The LLM reads it to decide when to use this tool.
    ...
```

### Resources — App-Controlled Data

Resources are **read-only data** that the application exposes to the AI. Think of them like GET endpoints. The difference from tools is that resources are *application-controlled* — meaning the host app (or user) decides when to load them, not the model itself.

Resources are meant for data that the model needs for context: schemas, configurations, static files, database contents. They should be **cheap to read** — not computationally expensive operations.

```python
@mcp.resource("db://schema/tables")
def get_schema() -> str:
    """Returns the full database schema for context."""
    ...
```

### Prompts — User-Controlled Templates

Prompts are reusable templates that the *user* can invoke. They're not about giving the model context (that's resources) or letting the model do things (that's tools). They're about giving the user a quick way to start a specific type of interaction.

Imagine you have a prompt template like "analyze this table for anomalies" — instead of typing that every time, you surface it as a prompt in the UI. The user selects it, fills in the arguments, and the conversation starts with a perfect, structured message.

```python
@mcp.prompt()
def analyze_table(table_name: str, metric: str = "row_count") -> str:
    """Generate a prompt for analyzing a database table."""
    return f"""Please analyze the table '{table_name}' focusing on {metric}.
    Use the available tools to query the data and provide insights about:
    - Data distribution
    - Potential anomalies or outliers
    - Trends over time if applicable"""
```

**Quick summary of the difference:**
- **Tool** → The model calls it automatically when it thinks it's useful
- **Resource** → The host app or user loads it to inject context  
- **Prompt** → The user picks it from the UI to start a structured task

---

## Let's Build Something Real: A SQL Database MCP Server

Let me show you a practical MCP server that connects an AI to a SQL database. I'll use SQLite here to keep it simple, but the same pattern works for PostgreSQL, MySQL, or any other database.

### Setup

First, install the dependencies:

```bash
uv init sql-mcp-server
cd sql-mcp-server
uv add "mcp[cli]" aiosqlite
```

### The Server

```python
# server.py
import json
import aiosqlite
from mcp.server.fastmcp import FastMCP

DB_PATH = "analytics.db"

mcp = FastMCP(
    "SQL Analytics Server",
    description="MCP server for querying and analyzing our analytics database"
)


# ── RESOURCE: expose the database schema ────────────────────────────────────

@mcp.resource("db://schema")
async def get_database_schema() -> str:
    """
    Returns the complete database schema including all tables, 
    columns, and their types. Use this to understand what data 
    is available before writing queries.
    """
    async with aiosqlite.connect(DB_PATH) as db:
        cursor = await db.execute(
            "SELECT name FROM sqlite_master WHERE type='table'"
        )
        tables = await cursor.fetchall()

        schema_parts = []
        for (table_name,) in tables:
            cursor = await db.execute(f"PRAGMA table_info({table_name})")
            columns = await cursor.fetchall()
            col_defs = ", ".join(
                f"{col[1]} {col[2]}" for col in columns
            )
            schema_parts.append(f"TABLE {table_name} ({col_defs})")

        return "\n".join(schema_parts)


@mcp.resource("db://tables/{table_name}/sample")
async def get_table_sample(table_name: str) -> str:
    """
    Returns the first 5 rows of a table as a preview.
    Useful for understanding the data format before running analysis.
    """
    async with aiosqlite.connect(DB_PATH) as db:
        cursor = await db.execute(
            f"SELECT * FROM {table_name} LIMIT 5"
        )
        rows = await cursor.fetchall()
        columns = [desc[0] for desc in cursor.description]

        result = [",".join(columns)]
        for row in rows:
            result.append(",".join(str(v) for v in row))

        return "\n".join(result)


# ── TOOLS: execute queries and modify data ───────────────────────────────────

@mcp.tool()
async def run_select_query(sql: str) -> str:
    """
    Execute a read-only SELECT query against the analytics database.
    Returns results as JSON. Use this to fetch data, run aggregations,
    or explore specific records. Do NOT use for INSERT/UPDATE/DELETE operations.
    
    Args:
        sql: A valid SELECT SQL statement
    
    Returns:
        JSON string with query results and row count
    """
    sql = sql.strip()
    if not sql.upper().startswith("SELECT"):
        return "Error: Only SELECT queries are allowed with this tool."

    async with aiosqlite.connect(DB_PATH) as db:
        cursor = await db.execute(sql)
        rows = await cursor.fetchall()
        columns = [desc[0] for desc in cursor.description]

        results = [dict(zip(columns, row)) for row in rows]
        return json.dumps({
            "row_count": len(results),
            "columns": columns,
            "data": results[:100]  # limit to avoid context overflow
        }, indent=2, default=str)


@mcp.tool()
async def run_write_query(sql: str, confirm: bool = False) -> str:
    """
    Execute an INSERT, UPDATE, or DELETE query.
    Requires explicit confirmation (confirm=True) to run.
    Always describe to the user what you are about to do before calling this tool.
    
    Args:
        sql: A valid INSERT, UPDATE, or DELETE SQL statement
        confirm: Must be True to actually execute. Set to False to just preview.
    
    Returns:
        Number of rows affected or a preview of the operation
    """
    if not confirm:
        return f"Preview mode: would execute:\n{sql}\n\nCall again with confirm=True to run."

    async with aiosqlite.connect(DB_PATH) as db:
        cursor = await db.execute(sql)
        await db.commit()
        return f"Success. Rows affected: {cursor.rowcount}"


@mcp.tool()
async def get_table_stats(table_name: str) -> str:
    """
    Get basic statistics for a table: row count, column count,
    and for numeric columns: min, max, and average values.
    
    Args:
        table_name: Name of the table to analyze
    
    Returns:
        JSON string with table statistics
    """
    async with aiosqlite.connect(DB_PATH) as db:
        cursor = await db.execute(f"SELECT COUNT(*) FROM {table_name}")
        row_count = (await cursor.fetchone())[0]

        cursor = await db.execute(f"PRAGMA table_info({table_name})")
        columns = await cursor.fetchall()

        stats = {"table": table_name, "row_count": row_count, "columns": {}}

        for col in columns:
            col_name, col_type = col[1], col[2]
            if any(t in col_type.upper() for t in ["INT", "REAL", "FLOAT", "NUMERIC"]):
                cursor = await db.execute(
                    f"SELECT MIN({col_name}), MAX({col_name}), AVG({col_name}) "
                    f"FROM {table_name}"
                )
                min_v, max_v, avg_v = await cursor.fetchone()
                stats["columns"][col_name] = {
                    "type": col_type,
                    "min": min_v,
                    "max": max_v,
                    "avg": round(avg_v, 2) if avg_v else None
                }
            else:
                stats["columns"][col_name] = {"type": col_type}

        return json.dumps(stats, indent=2, default=str)


# ── PROMPTS: reusable interaction templates ──────────────────────────────────

@mcp.prompt()
def analyze_table(table_name: str) -> str:
    """Start a full analysis of a database table."""
    return f"""I need you to do a complete analysis of the '{table_name}' table.

Please:
1. First, load the database schema (db://schema) to understand the full context
2. Get a sample of the data (db://tables/{table_name}/sample)  
3. Run get_table_stats for detailed statistics
4. Identify any patterns, anomalies, or insights
5. Suggest 2-3 specific queries that would be useful for this data

Format your response with clear sections and include example SQL queries."""


@mcp.prompt()
def write_query_for(goal: str) -> str:
    """Generate a SQL query for a specific business goal."""
    return f"""I need a SQL query to accomplish this goal: {goal}

Please:
1. Check the database schema (db://schema) first
2. Write the query step by step, explaining your logic
3. Run it with run_select_query to verify it works
4. Explain the results and suggest any refinements"""


# ── TRANSPORT ────────────────────────────────────────────────────────────────

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### Testing It

Before connecting to any AI application, you can test everything with the MCP Inspector:

```bash
mcp dev server.py
```

This opens a browser UI where you can call tools, inspect resources, and test prompts interactively. It's really useful for debugging.

### Connecting to Claude Desktop

Add this to your Claude Desktop config file (`~/Library/Application Support/Claude/claude_desktop_config.json` on Mac):

```json
{
  "mcpServers": {
    "sql-analytics": {
      "command": "uv",
      "args": [
        "--directory",
        "/absolute/path/to/sql-mcp-server",
        "run",
        "server.py"
      ]
    }
  }
}
```

Restart Claude Desktop and you'll see the MCP tools available. Now you can literally say "what tables do we have?" and Claude will load the schema resource. Or "how many users signed up last week?" and it will write and execute the query automatically.

---

## Transport Mechanisms: Local vs. Remote

One thing that is easy to overlook is how the client and server actually communicate. MCP supports multiple **transport** mechanisms:

**stdio (Standard I/O)** — The simplest option. The server runs as a local process and communicates via stdin/stdout. Perfect for local development and local AI apps like Claude Desktop or Cursor. This is what we used in the example above.

**SSE (Server-Sent Events)** — The server runs as an HTTP service, and the client connects over the internet. This is the option for production scenarios where you want your MCP server deployed in the cloud and accessible by multiple users.

**Streamable HTTP** — A newer transport introduced in the 2025-11-25 spec that is more flexible than SSE. It supports both streaming and non-streaming responses and is becoming the recommended option for remote servers.

For development, start with stdio. When you're ready to share your server with a team or deploy it to production, move to Streamable HTTP.

---

## When Should You Build an MCP Server?

This is the question I get the most. Not every integration needs an MCP server. Here are some scenarios where it really makes sense:

**Internal data access** — If your team uses Claude or Cursor and needs to query internal databases, APIs, or file systems, an MCP server is the cleanest way to do it. One server, every team member benefits immediately.

**Repetitive AI workflows** — If you find yourself copying the same context or running the same type of queries over and over, package that into an MCP server with good prompt templates. It saves time every single day.

**Multi-tool AI applications** — If you're building your own AI-powered product, you don't need to reinvent every integration. Create an MCP client in your app and immediately gain access to the ecosystem of existing MCP servers.

**Developer tooling** — Custom code generation rules, project-specific documentation, linting tools, test generation — all of these can be MCP servers attached to your IDE.

**Domain-specific agents** — A customer support bot that can check order status, a data analyst bot that can run reports, a DevOps assistant that can check service health — all of these are great MCP server candidates.

---

## What's New: MCP in 2025 and Beyond

The MCP ecosystem moved incredibly fast this past year. Some things worth knowing:

**Industry-wide adoption** — OpenAI adopted MCP in March 2025, and Google DeepMind confirmed support soon after. This isn't an Anthropic-only thing anymore. It's becoming the universal standard for AI integrations.

**The Linux Foundation governance** — In December 2025, Anthropic donated MCP to the newly formed Agentic AI Foundation (AAIF), a directed fund under the Linux Foundation. This was a huge deal. It means MCP is now vendor-neutral and governed by the community, similar to how Kubernetes or PyTorch are maintained.

**New spec features (2025-11-25)** — The latest spec version added async task tracking (so long-running operations don't block), improvements to server identity, and made the protocol more stateless — important for scaling to production scenarios.

**MCP Registry** — There's now an official registry for discovering MCP servers. Notion, Stripe, GitHub, Hugging Face, and Postman have all built their own servers. There are thousands available, and the number keeps growing.

**FastMCP 3.0 (January 2026)** — The Python library got a major update with component versioning, granular authorization, OpenTelemetry support, and multiple provider types. Building production-grade MCP servers is much easier now.

**Security considerations** — One thing I have to mention: security researchers found some real issues in 2025 — prompt injection, tool permission combinations that could exfiltrate data, and lookalike tools. These don't mean you shouldn't use MCP, but they mean you should be careful about which servers you trust, especially third-party ones. Always review what a server can do before connecting it to your AI applications.

---

## Final Thoughts

MCP feels to me like one of those things that you don't fully appreciate until you build something with it. The moment Claude started querying my database and explaining the results in plain English, I had this feeling of "okay, this changes things." 

The abstraction is genuinely good. The separation between tools (model does things), resources (model gets context), and prompts (user starts workflows) is clean and makes sense. And the fact that you build it once and it works with Claude, Cursor, or any MCP-compatible application is really powerful.

If you're a software engineer and you haven't tried it yet, I highly recommend spending a few hours building a small server for something you already do manually. The SQL example I showed here is a great starting point. Or maybe you have some internal APIs you wish Claude could access — that's another perfect use case.

The tools are there, the ecosystem is there, and honestly it's a lot of fun to build with.

---

*If you want to explore more, the official docs are at [modelcontextprotocol.io](https://modelcontextprotocol.io) and Anthropic has a great course at [anthropic.skilljar.com](https://anthropic.skilljar.com). The full Python SDK is on GitHub under the `modelcontextprotocol` organization.*
