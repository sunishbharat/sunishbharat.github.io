---
date: 2026-06-28
categories:
  - AI
  - Engineering
authors:
  - sunish
description: How I built AtlasMind-Netra - the AI agent and MCP server layer of AtlasMind that clarifies ambiguous Jira queries through conversation before dispatching to the JQL pipeline.
---

# Building AtlasMind-Netra: An AI Agent and MCP Server for Jira

AtlasMind started as a plain-English to JQL query engine. The core loop was straightforward: you type a question, the system embeds it, retrieves the nearest Jira fields from pgvector, constructs a JQL query, and returns a chart. It works. But I kept running into the same class of problem - ambiguous queries.

"Show me the open issues" - open in which project? "What's blocked?" - blocked by whom, and since when? "List the critical tickets" - critical meaning Priority: Critical, or critical to a specific sprint goal? Every one of these is a valid question. None of them has a single valid answer without more context. The original AtlasMind loop handled this by doing its best and returning something, which sometimes meant returning the wrong thing confidently.

Netra is the fix. It is the AI agent and MCP server layer of the AtlasMind platform. Before anything hits the JQL pipeline, Netra talks back.

<!-- more -->

**Links:**

- AtlasMind live: [atlasmind.de](https://atlasmind.de){ target="_blank" }
- Frontend: [github.com/sunishbharat/AtlasMind-frontendUI](https://github.com/sunishbharat/AtlasMind-frontendUI){ target="_blank" }
- Backend: [github.com/sunishbharat/atlasMind-Lite](https://github.com/sunishbharat/atlasMind-Lite){ target="_blank" }

## Why a Conversational Layer at All

The problem with most Jira query interfaces is that they demand precision from the user upfront. You either know the JQL and write it yourself, or you write a vague natural-language question and hope the system interprets it correctly. There is no middle ground.

What I actually wanted was something closer to how I work with a junior analyst: I give a rough question, they ask one clarifying question if needed, and then they go off and do the work. Not three clarifying questions. One, if it is genuinely necessary.

Netra implements exactly this. It classifies whether the incoming query is specific enough to dispatch immediately, or ambiguous enough to warrant a single targeted question back to the user. The clarification is not open-ended. It identifies the specific dimension that is underspecified - project scope, time window, status filter, assignee - and asks about that one thing.

## Architecture

The system has two distinct layers that work in sequence.

```
User Query
   │
   ▼
Netra AI Agent           ← conversational disambiguation
   │  clarify or dispatch
   ▼
AtlasMind MCP Server     ← tool execution layer
   │  JQL pipeline + pgvector + Jira REST API
   ▼
Structured Response      ← charts, tables, summaries
```

The agent layer is responsible for everything before a Jira API call is made. The MCP server layer is responsible for everything after.

### The Netra Agent

The agent sits at the entry point of the platform. Every query comes through it. Its job is to decide one of three things:

1. The query is specific enough. Dispatch it immediately to the MCP toolchain.
2. The query is ambiguous in one identifiable way. Ask one targeted question, then dispatch.
3. The query is out of scope. Return a clear explanation without touching the Jira API.

The classification happens in a single LLM call against a structured prompt that encodes the known ambiguity categories: missing project context, missing time range, missing status scope, missing assignee filter, missing issuetype. If the classifier fires on one of these, it generates a concise question targeting exactly that gap.

This matters because it keeps the interaction tight. An agent that asks multiple questions before doing anything is not useful in practice. The moment you have to answer three questions before getting a chart, you might as well have written the JQL yourself.

### The MCP Server

The MCP server is the execution layer. Once Netra has established that the query is ready to dispatch, the MCP server takes over. It exposes a set of tools that the agent can invoke: query execution, field lookup, sprint context retrieval, issue detail fetching.

The MCP protocol makes this composable. Coding assistants like Claude Code or Cursor can connect to the server directly and use the same tools programmatically. The same toolchain that powers the conversational interface is also accessible as structured function calls from any MCP-compatible client.

Under the hood, the tool execution follows the same architecture as the original AtlasMind backend: pgvector field retrieval, LLM-based JQL generation with the 7-pass sanitizer, the self-correcting feedback loop against the Jira REST API. Netra adds the conversational envelope around it.

## What the Clarification Loop Actually Looks Like

Here is a concrete example of the flow with an ambiguous query:

```
User   : Show me all the open issues

Netra  : Which project should I search in?
         (or type 'all' to search across all accessible projects)

User   : KAFKA

Netra  → dispatches: project = KAFKA AND status not in ('Closed', 'Resolved')
         Found 3847 result(s); showing 1000.
```

And here is the same flow with a specific query that goes straight through:

```
User   : Show me all open P1 bugs in project KAFKA assigned to unresolved

Netra  → dispatches immediately:
         project = KAFKA AND issuetype = Bug
         AND priority = Highest
         AND status not in ('Closed', 'Resolved')
         Found 12 result(s).
```

The agent fires a question only when it has to. When the query contains enough structure, it disappears entirely and you interact directly with the result.

## Why MCP as the Transport Layer

I chose MCP deliberately rather than a custom API. The reason is composability.

A custom REST API works fine for the AtlasMind web interface but it is a dead end for anything else. If a developer is using Claude Code and wants to query their Jira backlog mid-session without leaving their editor, a custom API requires a separate integration effort for every client. MCP is already the integration layer that those clients understand.

This means Netra's toolset is immediately usable in any MCP-compatible context: Claude Desktop, VS Code with Copilot, Cursor, terminal-based agents. You connect the server once and the tools are available everywhere that speaks the protocol.

The tradeoff is that MCP is a younger protocol with a smaller debugging ecosystem than REST. Error surfacing from MCP tool calls is less standardized than a JSON response with an HTTP status code. I spent more time than expected on error handling edge cases that would have been trivial over REST. That cost is worth it for the distribution upside, but it is worth naming.

## What I Learned

**Disambiguation is a product decision, not just an engineering one.** The instinct when building a conversational agent is to ask as many clarifying questions as needed to get to a confident answer. The correct instinct is the opposite: ask the minimum number of questions that gets you to a useful answer. One question per turn, targeted at the most significant underspecified dimension. More than that and the interaction model breaks down. Users stop using it.

**The MCP server boundary forces you to think clearly about what a tool is.** When you expose something as an MCP tool, you have to give it a name, a description, and a typed input schema. That constraint is useful. It makes you articulate exactly what the tool does, what it expects, and what it returns. I found that writing the tool definitions first and the implementation second produced cleaner code than building in the other direction.

**Stateless agents are harder to debug than they look.** The Netra agent holds no session state between turns. The entire conversation history is passed back in each request. This is the correct architecture for composability and it makes individual calls inspectable, but it means bugs that depend on conversation history across multiple turns take longer to reproduce. A conversation log that seemed fine step by step was failing at turn three because the agent was receiving a malformed history from the client. Structured logging from turn one is not optional.

**Conversational latency is noticed more than query latency.** In the non-conversational AtlasMind loop, a 2-3 second round trip to generate JQL, query Jira, and render a chart reads as normal. In a conversational loop where Netra is asking a question, the same 2-3 seconds on the clarification response feels slow because the user is waiting for a question, not a result. The disambiguation response needs to be fast. I ended up routing clarification generation to a smaller, faster model and reserving the higher-quality models for JQL generation.

---

- AtlasMind live: [atlasmind.de](https://atlasmind.de){ target="_blank" }
- Frontend: [github.com/sunishbharat/AtlasMind-frontendUI](https://github.com/sunishbharat/AtlasMind-frontendUI){ target="_blank" }
- Backend: [github.com/sunishbharat/atlasMind-Lite](https://github.com/sunishbharat/atlasMind-Lite){ target="_blank" }
