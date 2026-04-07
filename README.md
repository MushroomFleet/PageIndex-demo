# RBR-PageIndex-dev

**Reasoning-Based Retrieval meets classic RAG — side by side, in your browser.**

RBR-PageIndex-dev is a client-side React/Vite/TypeScript application that implements [PageIndex](https://github.com/VectifyAI/PageIndex)-style reasoning-based document retrieval alongside traditional vector/RAG search. Upload a PDF or markdown file, and the app builds both a hierarchical PageIndex tree and a vector embedding index. Then query with either system — or both simultaneously — and see exactly where reasoning-based retrieval succeeds and where vector similarity falls short.

No backend server. No database to provision. Everything runs in the browser, persists in IndexedDB, and calls any OpenAI-compatible LLM endpoint directly.

---

## Why This Exists

The standard RAG pipeline — chunk, embed, search by cosine similarity — works well enough for simple documents. It breaks on the documents that matter most in professional settings: 200-page SEC filings, regulatory submissions, technical manuals, and legal contracts with tables, appendices, and cross-references.

When a document says *"see Appendix G for detailed statistical tables"*, a vector retriever has no mechanism to follow that reference. The answer lives in Appendix G, but nothing there is semantically similar to the original question. The retriever returns the wrong section with high confidence.

Reasoning-based retrieval takes a different approach. Instead of searching for similar chunks, it builds a hierarchical map of the document and lets the LLM navigate that map — scanning section summaries, drilling into promising areas, reading raw content, and following cross-references — the same way a human expert reads a long document.

This app puts both approaches in the same interface so you can see the difference on your own documents.

---

## Features

### Index Tab

- **PDF upload** with drag-and-drop or file picker — text extracted client-side via PDF.js
- **Markdown upload** or direct paste — heading levels parsed into tree structure automatically
- **Parallel index building** — the PageIndex tree and vector embeddings build concurrently, each with independent progress indicators
- **Tree inspector** — browse the full document hierarchy, read node summaries, click any node to view its raw content
- **Document management** — view build status, storage usage, and delete documents with full cascade cleanup

### Search Tab

Three sub-tabs, one query input, completely different retrieval architectures:

**🌳 PageIndex (Tree Reasoning)**
The LLM receives the document's hierarchical tree as an in-context index and iteratively navigates it. Each step is displayed in a live reasoning trace — you see the model select a section, read its content, follow a cross-reference, and arrive at an answer. Sources are reported as tree nodes with page ranges.

**🔍 Vector RAG (Classic)**
The query is embedded, cosine similarity ranks all stored chunks, the top-k chunks are injected into a prompt, and the LLM generates an answer. Retrieved chunks are displayed with their similarity scores. This is the standard pipeline most RAG systems use today.

**⚖ Compare**
Both pipelines run the same query in parallel. Results appear side by side with the answer, source references, latency, token count, and a thumbs-up/thumbs-down accuracy toggle. A session scoreboard tracks which system is winning across multiple queries.

### Additional Capabilities

- **Conversation memory** — follow-up questions carry context, so "What about liabilities?" resolves against the same document region as your previous question about assets
- **Configurable models** — works with any OpenAI-compatible API endpoint (OpenAI, Anthropic via proxy, local models via LM Studio/Ollama, etc.)
- **Adjustable parameters** — chunk size, overlap, max reasoning iterations, temperature, model selection for both chat and embeddings
- **Exportable trees** — download the PageIndex tree as JSON for use in external pipelines

---

## Quick Start

```bash
# Clone the repository
git clone https://github.com/MushroomFleet/RBR-PageIndex-dev.git
cd RBR-PageIndex-dev

# Install dependencies
npm install

# Start the dev server
npm run dev
```

Open `http://localhost:5173` in your browser. The Settings panel opens automatically on first launch — enter your API endpoint and key, then you're ready to upload documents.

### Requirements

- **Node.js** 18+ and npm
- An **OpenAI-compatible API key** (OpenAI, or any provider with a compatible `/v1/chat/completions` and `/v1/embeddings` endpoint)

### Configuration

All settings are accessible from the gear icon in the top-right corner:

| Setting | Default | Notes |
|---|---|---|
| API Endpoint | `https://api.openai.com/v1` | Any OpenAI-compatible base URL |
| Chat Model | `gpt-4o` | Used for tree construction and query reasoning |
| Embedding Model | `text-embedding-3-small` | Used for vector index and RAG search |
| Chunk Size | 512 tokens | For vector mode chunking |
| Chunk Overlap | 50 tokens | Sliding window overlap |
| Max Reasoning Steps | 6 | Tree reasoning iteration limit per query |
| Temperature | 0.1 | Low for retrieval precision |

Your API key is stored in `localStorage` and is only ever sent to your configured API endpoint — never to any other server.

---

## Project Structure

```
pageindex-searchapp/
├── index.html
├── package.json
├── tsconfig.json
├── vite.config.ts
├── src/
│   ├── main.tsx                    # React entry point
│   ├── App.tsx                     # Root component, tab routing, settings state
│   ├── types.ts                    # All shared TypeScript interfaces
│   ├── config.ts                   # Default configuration constants
│   │
│   ├── engine/                     # Retrieval engine (zero React dependency)
│   │   ├── llm-gateway.ts          # Chat completion + embedding API wrapper
│   │   ├── pdf-extract.ts          # PDF.js client-side text extraction
│   │   ├── md-parse.ts             # Markdown heading parser → tree
│   │   ├── tree-builder.ts         # LLM-driven PageIndex tree construction
│   │   ├── chunker.ts              # Sliding-window text chunking
│   │   ├── tree-reasoner.ts        # Agentic tree navigation retrieval
│   │   ├── vector-searcher.ts      # Cosine similarity search + RAG prompt
│   │   └── compare-runner.ts       # Parallel execution of both pipelines
│   │
│   ├── storage/                    # IndexedDB persistence layer
│   │   ├── db.ts                   # Schema init, typed CRUD helpers
│   │   ├── documents.ts            # Document records
│   │   ├── trees.ts                # Tree JSON storage
│   │   ├── node-content.ts         # Raw text per tree node
│   │   ├── chunks.ts               # Chunks + embedding vectors
│   │   └── conversations.ts        # Conversation history
│   │
│   ├── components/                 # React UI
│   │   ├── Layout.tsx              # Shell, tab bar, settings trigger
│   │   ├── SettingsModal.tsx        # Configuration panel
│   │   ├── IndexTab.tsx            # Upload + document list + inspector
│   │   ├── UploadArea.tsx          # Drag-and-drop + file picker + paste
│   │   ├── DocumentList.tsx        # Indexed documents with status
│   │   ├── DocumentCard.tsx        # Per-document status and actions
│   │   ├── TreeInspector.tsx       # Collapsible hierarchy browser
│   │   ├── TreeNode.tsx            # Recursive tree node component
│   │   ├── SearchTab.tsx           # Search sub-tab router + doc selector
│   │   ├── PageIndexSearch.tsx     # Tree reasoning query interface
│   │   ├── VectorSearch.tsx        # Vector RAG query interface
│   │   ├── CompareSearch.tsx       # Side-by-side with scoring
│   │   ├── ReasoningTrace.tsx      # Step-by-step navigation display
│   │   ├── ChunkList.tsx           # Retrieved chunks with scores
│   │   ├── AnswerPanel.tsx         # Answer + sources + metrics
│   │   └── QueryInput.tsx          # Shared query input component
│   │
│   └── styles/
│       └── global.css              # CSS variables, layout, component styles
```

The `engine/` directory has **zero React imports**. It can be extracted and used in a Node.js script, a Cloudflare Worker, or any JavaScript runtime — swap the storage layer and the retrieval engine works anywhere.

---

## How It Works

### PageIndex Tree Construction

When you upload a document, the system parses it into a hierarchical JSON tree — a machine-readable table of contents with section titles, page ranges, and LLM-generated summaries at every level.

For PDFs, the LLM analyses the first 20 pages for table-of-contents patterns, identifies top-level sections, then recursively splits large sections into subsections until each node fits within the token budget. For markdown, heading levels (`#`, `##`, `###`) define the hierarchy directly.

Each tree node maps to its raw page content stored in IndexedDB, so the LLM can retrieve the full text of any section on demand.

### Agentic Tree Reasoning

When you ask a question in PageIndex mode, the LLM receives the tree (titles + summaries, not raw content) and enters a reasoning loop:

1. **Read the tree** — identify which sections likely contain the answer
2. **Navigate to a node** — request its full content
3. **Extract information** — or spot a cross-reference ("see Appendix G")
4. **Follow the reference** — navigate to Appendix G, read its content
5. **Answer** — synthesise a response with precise source citations

Every step is visible in the reasoning trace. You can watch the model think through the document structure in real time.

### Vector RAG (Classic)

The same document is also chunked into overlapping 512-token windows, each embedded via the embedding API. At query time, the question is embedded, cosine similarity ranks all chunks, the top 5 are injected into a prompt, and the LLM answers based on those chunks alone.

This is the standard RAG pipeline. It's fast and cheap, but it can't follow cross-references, can't reason about document structure, and can't distinguish between semantically similar but contextually different passages.

### Compare Mode

Both pipelines run in parallel on the same query. You see exactly where each succeeds or fails — and on which types of questions reasoning-based retrieval justifies its higher token cost.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Browser Runtime                      │
│                                                         │
│  ┌────────────────────────────────────────────────────┐ │
│  │                   React UI                         │ │
│  │  ┌─────────────┐ ┌─────────────┐ ┌──────────────┐ │ │
│  │  │ IndexTab    │ │ SearchTab   │ │ SettingsModal│ │ │
│  │  │ UploadArea  │ │ PageIndex   │ │              │ │ │
│  │  │ DocList     │ │ VectorRAG   │ │              │ │ │
│  │  │ TreeInspect │ │ Compare     │ │              │ │ │
│  │  └─────────────┘ └─────────────┘ └──────────────┘ │ │
│  └──────────────────────┬─────────────────────────────┘ │
│                         │                               │
│  ┌──────────────────────▼─────────────────────────────┐ │
│  │              engine/ (no UI dependency)             │ │
│  │  ┌──────────────┐ ┌──────────────┐ ┌────────────┐ │ │
│  │  │ tree-builder │ │ tree-reasoner│ │ vector-    │ │ │
│  │  │              │ │              │ │ searcher   │ │ │
│  │  └──────────────┘ └──────────────┘ └────────────┘ │ │
│  │  ┌──────────────┐ ┌──────────────┐ ┌────────────┐ │ │
│  │  │ chunker      │ │compare-runner│ │ llm-gateway│ │ │
│  │  └──────────────┘ └──────────────┘ └────────────┘ │ │
│  └──────────────────────┬─────────────────────────────┘ │
│                         │                               │
│  ┌──────────────────────▼─────────────────────────────┐ │
│  │              storage/ (IndexedDB)                  │ │
│  │  documents | trees | node_content | chunks | convos│ │
│  └────────────────────────────────────────────────────┘ │
│                         │                               │
│                         ▼                               │
│               ┌──────────────────┐                      │
│               │ External LLM API │ (fetch)              │
│               └──────────────────┘                      │
└─────────────────────────────────────────────────────────┘
```

---

## Data Storage

All data lives in the browser's IndexedDB under the database `pageindex_searchapp`. Six object stores handle the full lifecycle:

| Store | Contents |
|---|---|
| `documents` | Document metadata, filenames, build status, storage size |
| `trees` | Hierarchical PageIndex tree JSON per document |
| `node_content` | Raw text for each tree node, keyed by `doc_id::node_id` |
| `chunks` | Text chunks with embedding vectors for vector search |
| `embeddings_meta` | Embedding model and dimension metadata per document |
| `conversations` | Query history with conversation turns and reasoning traces |

Data survives page reloads. Delete a document and all associated records cascade-delete across all stores.

---

## Supported Documents

| Format | Extraction Method | Tree Construction |
|---|---|---|
| **Text-based PDF** | PDF.js client-side extraction | LLM analyses structure, builds hierarchy |
| **Markdown (.md)** | Direct text read | Heading levels (`#`/`##`/`###`) define hierarchy |
| **Scanned/image PDF** | Not supported | Detected and rejected with a clear message |

For scanned PDFs, run OCR first (e.g., using the [PageIndex OCR tool](https://github.com/VectifyAI/PageIndex)) to produce a searchable PDF or markdown file, then upload that.

---

## Performance

| Operation | Typical Timing |
|---|---|
| PDF text extraction (100 pages) | < 3 seconds |
| PageIndex tree construction (50 pages) | 10–30 seconds (LLM-dependent) |
| Vector embedding (200 chunks) | < 60 seconds |
| PageIndex query (single document) | 3–10 seconds (3–5 reasoning steps) |
| Vector RAG query | 1–3 seconds |
| Cosine similarity search (5,000 chunks) | < 50ms |

Tree construction and embedding are one-time costs per document. Queries are where the latency-vs-accuracy tradeoff lives: vector search is faster and cheaper; PageIndex is slower but follows cross-references and navigates structure.

---

## Built With

- [React](https://react.dev/) 18 + [Vite](https://vitejs.dev/) 5 + [TypeScript](https://www.typescriptlang.org/) 5.5
- [PDF.js](https://mozilla.github.io/pdf.js/) for client-side PDF text extraction
- [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) for all persistent storage
- No UI framework beyond React. No Tailwind. No state management library. No routing library. Minimal dependency surface.

Inspired by [PageIndex](https://github.com/VectifyAI/PageIndex) (VectifyAI, MIT license), [Pankaj Pandey's article on reasoning-based retrieval](https://medium.com/@pankaj_pandey/9f593175c980), and the broader shift toward reasoning-based retrieval documented in the [Chroma "Context Rot" study](https://research.trychroma.com/context-rot) and [Claude Code's evolution from RAG to agentic search](https://www.latent.space/).

---

## TINS Specification

This project was built from a [TINS (There Is No Source)](https://thereisnosource.com) specification. The complete specification is available at [`PageIndex-SearchApp-TINS.md`](./PageIndex-SearchApp-TINS.md) in this repository. The TINS spec contains the full functional requirements, data models, algorithm pseudocode, implementation-ready TypeScript, UI layouts, and testing scenarios that were used to generate this codebase.

Additional planning documents are included for reference:

| Document | Description |
|---|---|
| `PageIndex-SearchApp-TINS.md` | Complete TINS spec for this application |
| `RBR-PageIndex-IndexedDB-TINS.md` | V1 spec — full IndexedDB client-side architecture |
| `RBR-PageIndex-D1-Cloudflare-TINS.md` | Cloud variant — Cloudflare Workers + D1 + R2 |
| `reasoning-based-document-retrieval-grounding.md` | Grounding document on the underlying methodology |

---

## License

MIT

---

## 📚 Citation

### Academic Citation

If you use this codebase in your research or project, please cite:

```bibtex
@software{rbr_pageindex_dev,
  title = {RBR-PageIndex-dev: Reasoning-Based Retrieval with PageIndex — Client-Side Search Application},
  author = {Drift Johnson},
  year = {2025},
  url = {https://github.com/MushroomFleet/RBR-PageIndex-dev},
  version = {1.0.0}
}
```

### Donate:

[![Ko-Fi](https://cdn.ko-fi.com/cdn/kofi3.png?v=3)](https://ko-fi.com/driftjohnson)