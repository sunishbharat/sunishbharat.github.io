---
title: AtlasMind - Production AI Assistant
description: Live natural language to JQL assistant using RAG, pgvector, and 5 runtime-switchable LLM backends. Deployed solo on Oracle Cloud A1.
---

# AtlasMind: Production AI Assistant

!!! abstract "Summary"
    **Type**: Personal project - live production system
    **Stack**: Python, FastAPI, pgvector, vLLM, Groq, AWS Bedrock, Oracle Cloud A1

    **Key outcomes:**

    - Live at [atlasmind.de](https://atlasmind.de){ target="_blank" } - fully deployed and running
    - 5 runtime-switchable LLM backends (Ollama, vLLM, Groq, AWS Bedrock, OpenAI-compatible)
    - Self-healing query retry loop: up to 4 automatic correction cycles per failed JQL query
    - Two-stage router that bypasses the JQL parser for general conversational queries
    - GPU inference on OCI A1 Flex (Ampere ARM) at zero egress cost

## Challenge

Jira's JQL query language is powerful but inaccessible to most stakeholders. Writing correct JQL requires knowing field names, operators, function syntax, and project-specific custom fields - knowledge that resides with a handful of engineers and not with the people who need reports.

The goal was to build a system that converts plain-English requests ("show me all open FPGA blockers assigned to Alice since last sprint") into valid, executable JQL - and recovers automatically when it gets the query wrong.

## Approach

Built a two-stage routing architecture:

1. **Intent classifier** - determines whether the user's query needs JQL or is a general question (documentation lookup, Jira how-to). General queries bypass the JQL pipeline entirely and go straight to the LLM with RAG context.
2. **JQL generation + self-healing loop** - for query requests, the LLM generates JQL from the user's prompt. If Jira's API returns a validation error, the error message is fed back to the LLM as additional context, and the query is regenerated. Up to 4 retry cycles before the loop exits with a graceful error.

The RAG layer uses pgvector to store and retrieve Jira field definitions and project metadata, keeping the context window small and the queries accurate.

LLM routing is runtime-switchable across 5 backends with no restart required, enabling cost-performance tradeoffs on the fly: local GPU inference for low-latency, Groq for speed, AWS Bedrock for compliance-sensitive queries.

Deployed solo on an Oracle Cloud A1 Flex instance (Ampere ARM architecture) using Docker Compose, with Tailscale for secure tunneling and zero-cost ingress.

## Demo

![AtlasMind dashboard demo](../../assets/Atlasmind_tutorial-1.gif)

## Tech Stack

| Layer | Technology |
|---|---|
| API | Python, FastAPI |
| Vector store | PostgreSQL + pgvector |
| Local inference | Ollama, vLLM (GPU) |
| Cloud inference | Groq, AWS Bedrock |
| Infrastructure | Oracle Cloud A1 Flex, Docker, Tailscale |

## Links

[View Code (GitHub) :fontawesome-brands-github:](https://github.com/sunishbharat/atlasMind-Lite){ .md-button target="_blank" }
[Live Site :material-arrow-top-right:](https://atlasmind.de){ .md-button .md-button--primary target="_blank" }
