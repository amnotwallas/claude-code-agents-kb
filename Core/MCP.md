---
tags: [claude-code, core, mcp, tools, integration]
type: core
related:
  - "[[CLAUDE-md]]"
  - "[[Anatomy-of-a-Skill]]"
  - "[[Security]]"
  - "[[Agent-Architecture]]"
created: 2026-08-02
status: stable
---

# MCP (Model Context Protocol)

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

## What Is MCP?

Model Context Protocol is an open, standard protocol for connecting an AI model like Claude to external tools, data sources, and services — a common interface so any MCP-compliant server can be plugged into any MCP-compliant client (Claude Code among them) without custom integration code per pair.

Before MCP, connecting Claude to, say, a company's internal database or a project management tool meant writing bespoke integration glue for that specific combination. MCP standardizes the interface: build an MCP server once for a data source, and any MCP client can use it.

## Why It Matters for Claude Code

Claude Code's tool surface — file access, bash, web search, and whatever else it has natively — is necessarily generic. MCP is how it extends into project-specific or organization-specific systems: a company's internal API, a specific database, a design tool like Figma, a ticketing system. Without MCP, that capability either doesn't exist or has to be custom-built per project.

## Core Concepts

### MCP Server
A process that exposes capabilities to an MCP client following the protocol — it declares what it offers and handles the calls. Can run locally (a subprocess Claude Code launches) or remotely (a hosted service Claude Code connects to over the network).

### MCP Client
The application consuming those capabilities — Claude Code acts as an MCP client when it's configured with one or more MCP servers.

### The Three Capability Types
- **Tools**: actions the model can invoke, with defined inputs and outputs — e.g., "create a ticket," "run a query," "post a message"
- **Resources**: data the model can read — e.g., a file, a database record, a document — conceptually similar to a GET endpoint
- **Prompts**: predefined prompt templates the server exposes for common operations against it — less commonly used than tools and resources in practice

## Configuring MCP Servers in Claude Code

Servers get declared in project or user configuration, and referenced in [[CLAUDE-md]] so Claude knows what's available and what each is for:

```markdown
## MCP Servers
- `postgres` (staging only, read-only) — data debugging
- `github` — PR and issue management
- `figma` — reading design files for UI implementation
```

Documenting *why* each server is there and what scope it has (read-only vs read-write, staging vs production) is as important as the technical connection itself — see [[CLAUDE-md]] for how this fits into the broader anatomy of project memory.

## MCP vs Skills: When to Use Each

| | MCP Server | Skill |
|---|---|---|
| **What it provides** | Live access to external systems (data, actions) | Procedural knowledge for a task Claude already has the tools for |
| **State** | Connects to real, current external state | Static instructions, no live connection |
| **Example** | Query the production database for the current inventory count | How to structure a new FastAPI endpoint |

They compose: a skill's step-by-step process (see [[Anatomy-of-a-Skill]]) can reference calling an MCP tool as one of its steps — MCP provides the capability, the skill provides the procedure for using it well in a specific context.

## Security Considerations

MCP servers are a direct extension of what Claude can read, write, and execute — they deserve the same scrutiny as any other tool permission. See [[Security]] for the full threat model, but specifically for MCP:

- **Verify before installing**: review the server's source when possible, don't install based on description alone
- **Pin versions**: avoid `@latest` — a compromised update under an existing package name is a real supply chain vector
- **Scope permissions tightly**: a server that only needs to read shouldn't be configured with write access "for convenience"
- **Treat servers with internet access as higher risk**: content they retrieve from the network is external input and should be treated as untrusted (see [[Security]], indirect prompt injection)

## Example: A Minimal MCP Server

Illustrative shape of what an MCP server exposes — the protocol itself is JSON-RPC based; this shows the conceptual tool definition a server would register:

```python
# Conceptual sketch of what an MCP server declares — actual
# implementation depends on the MCP SDK in use
from mcp_sdk import Server, Tool

server = Server(name="internal-tickets")

@server.tool(
    name="create_ticket",
    description="Create a support ticket in the internal tracker",
)
async def create_ticket(title: str, description: str, priority: str) -> dict:
    ticket = await tickets_api.create(
        title=title, description=description, priority=priority,
    )
    return {"ticket_id": ticket.id, "url": ticket.url}
```

Once registered and connected, Claude Code can call `create_ticket` as a tool the same way it would call any built-in tool — the model doesn't need to know or care that it's talking to an external system via MCP versus a native capability.

## Anti-patterns

- Installing an MCP server without reviewing what it can access — same risk as granting any tool broad permission blindly (see [[Security]])
- Not documenting configured MCP servers in [[CLAUDE-md]] — Claude and the team both lose visibility into what's available and why
- Giving a server write/execute access when the actual need is read-only
- Treating MCP as a replacement for skills — a server provides capability, not the judgment on when and how to use it well; that's still the job of a good prompt or skill

## Related Notes

- [[CLAUDE-md]] — where MCP servers get documented for project-level visibility
- [[Anatomy-of-a-Skill]] — how a skill's procedure can incorporate MCP tool calls
- [[Security]] — the threat model and defenses that apply directly to MCP servers
- [[Agent-Architecture]] — tool surface (including MCP-provided tools) as a design dimension
