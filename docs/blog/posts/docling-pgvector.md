---
date: 2026-06-05
categories:
  - AI
  - Engineering
authors:
  - sunish
description: How I built a production RAG pipeline using Docling and pgvector without LangChain - PDF parsing, table extraction, and vector search from scratch.
---

# Building a RAG Pipeline with Docling and pgvector: Without LangChain

Every RAG tutorial I found started the same way: `pip install langchain`, followed by 15 lines of boilerplate that somehow parsed a PDF, embedded it, stored it, and retrieved it - all without explaining what any of it was actually doing. I kept staring at those tutorials thinking: I know the output, but I have no idea what's happening between input and search result.

<!-- more -->

So I built [docling-pgvector](https://github.com/sunishbharat/docling-pgvector){ target="_blank" } - a clean RAG pipeline with no orchestration framework involved. Just IBM's Docling for parsing, SentenceTransformers for embeddings, and pgvector on PostgreSQL as the vector store. The whole thing fits in a few hundred lines of Python and I understand every single one of them.

## Why Docling?

I had a specific problem with most PDF parsers: tables. Feed a standard RAG pipeline a paper or a technical report, and any comparison table gets extracted as garbled flat text, with column headers mixed with row values, all meaning lost. If the table happens to contain the exact answer to your query, you're not getting it back in any useful form.

Docling does something different. It identifies tables in a PDF and exports them as Markdown: structured, readable, and searchable. That means a complexity comparison table from a research paper gets stored as a proper Markdown table in your vector database, not as a string of tokens that a model has to reconstruct from scratch.

This turned out to matter a lot in practice. When I tested the pipeline against the *Attention Is All You Need* paper, the top retrieved chunk for a table-specific query was the actual Markdown table, with a distance score of 0.55, correctly ranked above the surrounding prose. Here's what that looks like from a real test run:

```
query = "Maximum path lengths, per-layer complexity and minimum \
         number of sequential operations Table"

INFO:root:id_=43, dist=0.550322916862681
 Table 1: Maximum path lengths, per-layer complexity and minimum
 number of sequential operations for different layer types.

 | Layer Type     | Complexity per Layer | Sequential Ops | Max Path Length |
 |----------------|----------------------|----------------|-----------------|
 | Self-Attention | O(n² · d)            | O(1)           | O(1)            |
 | Recurrent      | O(n · d²)            | O(n)           | O(n)            |
 | Convolutional  | O(k · n · d²)        | O(1)           | O(log_k(n))     |

INFO:root:id_=16, dist=0.672102512607041
 4 Why Self-Attention
 In this section we compare various aspects of self-attention layers...
```

The table comes back ranked above the relevant prose section. If I had used a standard text-based PDF parser, that table would have been noise.

## The Pipeline

The architecture is deliberately straightforward. A PDF goes in, similarity-ranked chunks come out:

```
PDF File
   │
   ▼
Docling (PDF parser)          ← GPU auto-detected
   │  page-batched conversion
   ▼
HybridChunker + TableItem     ← semantic text chunks + Markdown tables
   │  unique content
   ▼
SentenceTransformer            ← BAAI/bge-base-en-v1.5 (768-dim)
   │  vector embeddings
   ▼
PostgreSQL + pgvector          ← similarity search (L2 distance)
```

The embedding model is `BAAI/bge-base-en-v1.5`, a solid general-purpose model at 768 dimensions. It's not the flashiest choice but it's well understood, well tested, and you can swap it out for any HuggingFace SentenceTransformer by changing a single config parameter. I specifically did not want a hard dependency on one model. The config is validated against HuggingFace Hub before downloading, so you get a clear error if you mistype a model name rather than a confusing runtime failure.

The vector store is pgvector on PostgreSQL 17. I chose pgvector over a dedicated vector database because I already had PostgreSQL in my infrastructure for other things, and pgvector's L2 distance search is more than adequate for the scale I needed. Adding another database to your stack just for vector search is a trade-off that only makes sense at large scale: for a project like this, it would have been unnecessary complexity.

## What it looks like in code

There's no magic. You call `DocumentProcessor` to parse and embed, then write directly to PostgreSQL:

```python
from document_processor import DocumentProcessor
from dconfig import EmbeddingsConfig

config = EmbeddingsConfig(model_name="BAAI/bge-base-en-v1.5")
processor = DocumentProcessor(embedconfig=config)

content_list, model = processor.embeddings_generate(path="./data/test.pdf")
embeddings = model.encode(content_list)
```

Then store and search:

```python
from pgvector_client import PGVectorClient, PGVectorConfig

pg_config = PGVectorConfig(host="localhost", database="vectordb")

# Store
with PGVectorClient(pg_config) as client:
    for chunk, embed in zip(content_list, embeddings):
        with client.cursor() as cur:
            cur.execute(
                "INSERT INTO items (text, embedding) VALUES (%s, %s)",
                (chunk, embed)
            )

# Search
query_vec = model.encode("your search query", normalize_embeddings=True)
with PGVectorClient(pg_config) as client:
    with client.cursor() as cur:
        cur.execute("""
            SELECT id, text, embedding <-> %s AS distance
            FROM items ORDER BY distance LIMIT 2
        """, (query_vec,))
        results = cur.fetchall()
```

That's the whole retrieval loop. No abstractions hiding the database call. No framework deciding when to re-rank or how to format the output. You get back rows with distance scores and you decide what to do with them.

## Getting it running

I wanted this to be usable in three different ways: from a quick pull-and-run all the way to a full dev setup with tests. So the repo has three options. If you just want to try it, the fastest path is pulling the pre-built Docker image:

```bash
docker pull ghcr.io/sunishbharat/docling-pgvector:cpu-dev

# Start PostgreSQL + pgvector on a shared network
docker network create devnet
docker run --name pgvector-container \
  --network devnet \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=<password> \
  -e POSTGRES_DB=vectordb \
  -p 5432:5432 \
  -d pgvector/pgvector:pg17

# Enable the extension
docker exec pgvector-container psql -U postgres -d vectordb \
  -c "CREATE EXTENSION IF NOT EXISTS vector;"

# Run the project container
docker run --rm -it \
  --network devnet \
  -e DATABASE_URL=postgresql://postgres:<password>@pgvector-container:5432/vectordb \
  -v $(pwd):/workspace/docling-pgvector \
  -w /workspace/docling-pgvector \
  ghcr.io/sunishbharat/docling-pgvector:cpu-dev \
  bash
```

If you use VS Code, the Dev Container option is even simpler: open the repo, click "Reopen in Container", and everything configures itself - PostgreSQL, the vector extension, the test PDF. No manual setup at all. For anyone who just wants to run it locally with Python 3.12+ and [uv](https://docs.astral.sh/uv/){ target="_blank" }, there's Option C in the README.

- Source: [github.com/sunishbharat/docling-pgvector](https://github.com/sunishbharat/docling-pgvector){ target="_blank" }
- Docker image: [ghcr.io/sunishbharat/docling-pgvector:cpu-dev](https://github.com/sunishbharat/docling-pgvector/pkgs/container/docling-pgvector){ target="_blank" }

## What I actually learned

**Table extraction is a real problem, not a nice-to-have.** I underestimated this going in. A significant fraction of the useful information in technical documents lives in tables: comparison grids, benchmark results, configuration options. If your RAG pipeline can't retrieve that content in a structured form, you're building on a gap. Docling's HybridChunker handling of `TableItem` objects is the feature that made it the right tool here.

**Not using a framework forces clarity.** When I wrote the pgvector client from scratch, I had to make explicit decisions: connection pooling, cursor lifecycle, transaction boundaries, how to handle the vector dimension mismatch if someone changes models mid-session. LangChain handles some of this transparently, which is convenient right until something goes wrong and you have no idea which abstraction layer is misbehaving. I now have a much clearer mental model of what a RAG pipeline is actually doing, and that has made me a better architect of the systems that use them, including AtlasMind.

**GPU auto-detection is worth the effort to wire in.** Docling supports GPU-accelerated PDF conversion automatically if CUDA is available. On the first run on a GPU machine the difference in conversion time is significant, especially on long documents. The pipeline falls back to CPU transparently, but if you have the hardware it's worth knowing about.

**Pydantic v2 validation on config objects saves debugging time.** I used Pydantic v2 for `EmbeddingsConfig` and `PGVectorConfig`. Every config error that would otherwise show up as a mysterious runtime failure at step three of the pipeline now shows up immediately, with a clear message, at the point of construction. Small thing, but it made iterating significantly less frustrating.

The next thing I want to add is support for non-PDF formats: CSV and web pages are both obvious additions. The Docling team is actively developing the library and new converters are arriving regularly. For now, the PDF path is solid, tested, and documented. If you're building a RAG pipeline and want to understand what you're actually building, this is a cleaner starting point than most tutorials.

---

- Source code: [github.com/sunishbharat/docling-pgvector](https://github.com/sunishbharat/docling-pgvector){ target="_blank" }
