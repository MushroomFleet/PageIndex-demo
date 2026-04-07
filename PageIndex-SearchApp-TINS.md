<!-- TINS Specification v1.0 -->
<!-- ZS:COMPLEXITY:HIGH -->
<!-- ZS:PRIORITY:HIGH -->
<!-- ZS:PLATFORM:WEB -->
<!-- ZS:LANGUAGE:TYPESCRIPT -->

# PageIndex SearchApp

## Description

PageIndex SearchApp is a fully operational React/Vite/TypeScript proof-of-concept application that demonstrates reasoning-based document retrieval (PageIndex) as both a standalone search system and a drop-in companion to traditional vector/RAG search. The app runs entirely client-side in the browser with no backend server. All storage uses IndexedDB. All LLM and embedding calls go directly from the browser to any OpenAI-compatible API endpoint.

The application is organised around two primary tabs — **Index** and **Search** — with Search containing three sub-tabs: **PageIndex (Tree Reasoning)**, **Vector RAG (Classic)**, and **Compare**. A user uploads a document in the Index tab, the system builds both a PageIndex tree and a vector embedding index, then the user switches to Search and runs queries against either or both systems. The Compare sub-tab runs the same query through both retrieval pipelines simultaneously and displays results side by side with latency, token cost, source references, and a manual accuracy toggle so the user can score which system gave the better answer.

This is a prototype built for demonstration, benchmarking, and development. The goal is a single self-contained app that a developer can clone, run `npm install && npm run dev`, configure an API key, upload a PDF, and immediately see reasoning-based retrieval working alongside traditional RAG — understanding viscerally how and where each approach succeeds or fails.

Target users: developers evaluating retrieval architectures, teams considering PageIndex as a replacement or supplement to their existing RAG pipeline, and researchers who need a controlled environment to compare retrieval strategies on their own documents.

## Functionality

### Core Features

- **PDF upload and text extraction**: Upload a PDF file via drag-and-drop or file picker. The system extracts text per page client-side using PDF.js. Supports text-based PDFs; scanned/image PDFs are detected and rejected with a clear message.
- **Markdown upload and paste**: Upload a `.md` file or paste markdown directly into a text area. The system parses heading structure immediately.
- **PageIndex tree construction**: An LLM-driven process that parses extracted text into a hierarchical JSON tree — sections, subsections, summaries, and page ranges. The tree is stored in IndexedDB. A live progress indicator shows nodes being built.
- **Vector index construction**: The same document text is chunked (configurable size/overlap), each chunk is embedded via the embedding API, and chunks plus vectors are stored in IndexedDB. A progress bar shows chunks embedded out of total.
- **Parallel indexing**: Both the tree and the vector index build concurrently after upload. Each has independent status indicators. The user can search with whichever finishes first — PageIndex search works as soon as the tree is ready; Vector search works as soon as embeddings are ready.
- **PageIndex search (Tree Reasoning)**: The LLM receives the document's tree as an in-context index and iteratively navigates it — selecting sections, retrieving raw content, following cross-references — until it produces an answer. The full reasoning trace is displayed step by step.
- **Vector RAG search (Classic)**: The query is embedded, cosine similarity ranks all stored chunks, the top-k chunks are injected into a prompt, and the LLM generates an answer. Retrieved chunks are displayed with their similarity scores.
- **Compare mode**: Both search pipelines run the same query in parallel. Results are displayed in a two-column layout showing: the answer, source references (tree nodes or chunk IDs with page numbers), reasoning trace (PageIndex only), retrieved chunks (Vector only), latency in milliseconds, estimated tokens consumed, and a thumbs-up/thumbs-down toggle per column for the user to score accuracy.
- **Tree inspector panel**: A collapsible sidebar showing the full tree hierarchy for the selected document. Clicking a node displays its raw content in a detail pane. Nodes visited during the most recent search are highlighted.
- **Conversation memory**: Follow-up queries within the same search session carry context. If the user asks "What about liabilities?" after asking about assets, both retrieval engines receive the conversation history, allowing contextual follow-up.
- **Document management**: View all indexed documents, see their tree/vector build status, inspect storage size, and delete documents (cascading delete of tree, content, chunks, embeddings, and conversations).
- **Settings panel**: Configure API endpoint URL, API key (stored in localStorage, never sent to any server other than the configured LLM endpoint), chat model, embedding model, chunk size, chunk overlap, max reasoning iterations, and temperature.

### User Interface Layout

```
+------------------------------------------------------------------------+
| PageIndex SearchApp                     [Settings ⚙]  [API Key 🔑]    |
+------------------------------------------------------------------------+
| [ Index ]  [ Search ]                                                  |
+------------------------------------------------------------------------+

=== INDEX TAB (active when [ Index ] is selected) =======================

+------------------------------------------------------------------------+
|                                                                        |
|  +------------------------------------------------------------------+  |
|  |                                                                  |  |
|  |   📄  Drop a PDF or Markdown file here, or click to browse      |  |
|  |       [Browse Files]     [Paste Markdown]                        |  |
|  |                                                                  |  |
|  +------------------------------------------------------------------+  |
|                                                                        |
|  INDEXED DOCUMENTS                                                     |
|  +------------------------------------------------------------------+  |
|  |                                                                  |  |
|  |  📑 annual-report-2023.pdf              42 pages                 |  |
|  |     Tree: ██████████ Ready (34 nodes)   3.2s                     |  |
|  |     Vectors: ████████░░ 80% (164/205 chunks)                    |  |
|  |     Storage: 2.4 MB                           [🔍 Inspect] [🗑] |  |
|  |                                                                  |  |
|  |  📑 contract-draft.md                   12 sections              |  |
|  |     Tree: ██████████ Ready (12 nodes)   0.8s                     |  |
|  |     Vectors: ██████████ Ready (48 chunks)                        |  |
|  |     Storage: 340 KB                           [🔍 Inspect] [🗑] |  |
|  |                                                                  |  |
|  +------------------------------------------------------------------+  |
|                                                                        |
|  TREE INSPECTOR (shown when [Inspect] clicked)                         |
|  +------------------------------------------------------------------+  |
|  |  📑 annual-report-2023.pdf                                       |  |
|  |  "Annual report covering fiscal year 2023 operations..."         |  |
|  |                                                                  |  |
|  |  ▼ 0001 Executive Summary (pp. 1-4)                              |  |
|  |      "Overview of key financial results and strategic..."        |  |
|  |  ▼ 0002 Financial Statements (pp. 5-38)                          |  |
|  |    ▶ 0003 Income Statement (pp. 5-12)                            |  |
|  |    ▶ 0004 Balance Sheet (pp. 13-22)                              |  |
|  |    ▼ 0005 Cash Flow (pp. 23-38)                                  |  |
|  |        "Cash flow from operations decreased by 12%..."           |  |
|  |  ▶ 0006 Notes to Financial Statements (pp. 39-78)                |  |
|  |  ▶ 0007 Appendices (pp. 79-102)                                  |  |
|  |                                                                  |  |
|  |  NODE CONTENT (click any node)                                   |  |
|  |  +--------------------------------------------------------------+|  |
|  |  | [Raw text content of selected node displayed here]            ||  |
|  |  | Limited to first 2000 chars with "Show full" toggle           ||  |
|  |  +--------------------------------------------------------------+|  |
|  +------------------------------------------------------------------+  |
+------------------------------------------------------------------------+

=== SEARCH TAB (active when [ Search ] is selected) ====================

+------------------------------------------------------------------------+
| Document: [ annual-report-2023.pdf ▼ ]                                 |
|                                                                        |
| [ PageIndex 🌳 ]  [ Vector RAG 🔍 ]  [ Compare ⚖ ]                   |
+------------------------------------------------------------------------+

--- PageIndex sub-tab ---------------------------------------------------

+------------------------------------------------------------------------+
|                                                                        |
|  +------------------------------------------------------------------+  |
|  | What is the total value of deferred tax assets?              [→] |  |
|  +------------------------------------------------------------------+  |
|                                                                        |
|  REASONING TRACE                                                       |
|  +------------------------------------------------------------------+  |
|  |  ● Step 1 — Read tree index                            0.3s     |  |
|  |    Identified 7 top-level sections. "Notes to Financial          |  |
|  |    Statements" (pp. 39-78) most likely contains tax data.        |  |
|  |                                                                  |  |
|  |  ● Step 2 — Navigate to node 0006                      1.1s     |  |
|  |    Retrieved content for "Notes to Financial Statements".        |  |
|  |    Found sub-section "Note 14: Income Taxes" (pp. 61-67).       |  |
|  |                                                                  |  |
|  |  ● Step 3 — Retrieve node 0019 (Note 14)               1.4s     |  |
|  |    Page 63 states deferred tax assets increased by $2.1M.       |  |
|  |    Page 64 references "See Appendix C, Table C-4 for            |  |
|  |    complete deferred tax asset schedule."                        |  |
|  |                                                                  |  |
|  |  ● Step 4 — Follow cross-reference to node 0031         0.9s    |  |
|  |    Retrieved Appendix C content. Table C-4 on page 89           |  |
|  |    lists total deferred tax assets: $14.7M.                     |  |
|  |                                                                  |  |
|  |  ✓ Step 5 — Answer                                      0.2s    |  |
|  +------------------------------------------------------------------+  |
|                                                                        |
|  ANSWER                                                                |
|  +------------------------------------------------------------------+  |
|  | The total value of deferred tax assets is $14.7 million, as     |  |
|  | detailed in Appendix C, Table C-4 (page 89). Note 14 on page   |  |
|  | 63 reports a $2.1M increase over the prior year.                |  |
|  |                                                                  |  |
|  | Sources: Node 0019 (pp. 61-67), Node 0031 (pp. 87-92)          |  |
|  | Latency: 3.9s  ·  Tokens: 4,280  ·  Steps: 5                   |  |
|  +------------------------------------------------------------------+  |
|                                                                        |
+------------------------------------------------------------------------+

--- Vector RAG sub-tab --------------------------------------------------

+------------------------------------------------------------------------+
|                                                                        |
|  +------------------------------------------------------------------+  |
|  | What is the total value of deferred tax assets?              [→] |  |
|  +------------------------------------------------------------------+  |
|                                                                        |
|  RETRIEVED CHUNKS (top 5 by cosine similarity)                         |
|  +------------------------------------------------------------------+  |
|  |  #1  Score: 0.891  ·  Chunk 47  ·  pp. 63-64                    |  |
|  |  "...deferred tax assets increased by $2.1M during the          |  |
|  |   fiscal year, primarily due to..."                              |  |
|  |                                                                  |  |
|  |  #2  Score: 0.874  ·  Chunk 48  ·  pp. 64-65                    |  |
|  |  "...valuation allowance for deferred tax assets was             |  |
|  |   reviewed and adjusted..."                                      |  |
|  |                                                                  |  |
|  |  #3  Score: 0.856  ·  Chunk 12  ·  pp. 14-15                    |  |
|  |  "...total assets including deferred items amounted to..."       |  |
|  |                                                                  |  |
|  |  #4  Score: 0.843  ·  Chunk 51  ·  pp. 67-68                    |  |
|  |  "...tax provision reconciliation shows effective rate..."       |  |
|  |                                                                  |  |
|  |  #5  Score: 0.831  ·  Chunk 22  ·  pp. 24-25                    |  |
|  |  "...deferred revenue recognition policy applies to..."          |  |
|  +------------------------------------------------------------------+  |
|                                                                        |
|  ANSWER                                                                |
|  +------------------------------------------------------------------+  |
|  | Based on the retrieved passages, deferred tax assets increased   |  |
|  | by $2.1M during the fiscal year. The exact total value is not    |  |
|  | stated in the retrieved chunks.                                  |  |
|  |                                                                  |  |
|  | Sources: Chunks 47, 48, 12, 51, 22                              |  |
|  | Latency: 1.8s  ·  Tokens: 2,140                                 |  |
|  +------------------------------------------------------------------+  |
|                                                                        |
+------------------------------------------------------------------------+

--- Compare sub-tab -----------------------------------------------------

+------------------------------------------------------------------------+
|                                                                        |
|  +------------------------------------------------------------------+  |
|  | What is the total value of deferred tax assets?              [→] |  |
|  +------------------------------------------------------------------+  |
|                                                                        |
|  +-------------------------------+  +-------------------------------+  |
|  | 🌳 PAGEINDEX                  |  | 🔍 VECTOR RAG                |  |
|  +-------------------------------+  +-------------------------------+  |
|  |                               |  |                               |  |
|  | The total value of deferred   |  | Deferred tax assets increased |  |
|  | tax assets is $14.7M, per     |  | by $2.1M during the fiscal    |  |
|  | Appendix C, Table C-4 (p.89). |  | year. The exact total is not  |  |
|  | Note 14 (p.63) reports a      |  | stated in the retrieved       |  |
|  | $2.1M increase YoY.           |  | chunks.                       |  |
|  |                               |  |                               |  |
|  | Sources:                      |  | Sources:                      |  |
|  |   Node 0019 (pp. 61-67)      |  |   Chunks 47,48,12,51,22       |  |
|  |   Node 0031 (pp. 87-92)      |  |   (pp. 14,24,63-68)           |  |
|  |                               |  |                               |  |
|  | ⏱ 3.9s · 📊 4,280 tok · 5 st |  | ⏱ 1.8s · 📊 2,140 tok        |  |
|  |                               |  |                               |  |
|  | Accurate?  [👍] [👎]          |  | Accurate?  [👍] [👎]          |  |
|  +-------------------------------+  +-------------------------------+  |
|                                                                        |
|  COMPARISON HISTORY (this session)                                     |
|  +------------------------------------------------------------------+  |
|  | Q1: "What is the total value..."  PageIndex 👍  Vector 👎        |  |
|  | Q2: "Who is the CEO?"             PageIndex 👍  Vector 👍        |  |
|  | Q3: "Summarise risk factors"       PageIndex 👍  Vector 👍        |  |
|  |                                                                  |  |
|  | Session score: PageIndex 3/3 · Vector 2/3                        |  |
|  +------------------------------------------------------------------+  |
+------------------------------------------------------------------------+
```

### Tab Structure

The app has exactly two top-level tabs and three search sub-tabs:

- **Index** — Document upload, index building, document list, tree inspector.
- **Search** — Query interface with three sub-tabs:
  - **PageIndex** — Tree reasoning search with full reasoning trace.
  - **Vector RAG** — Classic chunk-embed-search with retrieved chunks display.
  - **Compare** — Side-by-side execution with accuracy scoring.

### User Flows

**Flow 1: First-time setup**
1. User launches the app. The Settings panel opens automatically because no API key is configured.
2. User enters their API endpoint URL (defaults to `https://api.openai.com/v1`) and API key.
3. User selects a chat model (defaults to `gpt-4o`) and embedding model (defaults to `text-embedding-3-small`).
4. User clicks "Save". The key is stored in `localStorage`. The Settings panel closes.
5. The Index tab becomes active. The upload area is enabled.

**Flow 2: Document upload and indexing**
1. User drags a PDF onto the upload area (or clicks "Browse Files" and selects a PDF, or clicks "Paste Markdown" and enters text).
2. For PDF: extraction begins immediately. A progress step shows "Extracting text... page 12/42".
3. Extraction completes. The document appears in the "Indexed Documents" list with status "Building...".
4. Two parallel processes start:
   - **Tree construction**: The LLM analyses page content and builds the hierarchical tree. Progress shows "Building tree... 8/34 nodes". Each node is stored in IndexedDB as it completes.
   - **Vector indexing**: The text is chunked, then each chunk is embedded via the embedding API. Progress shows "Embedding chunks... 80/205".
5. Each process updates its status indicator independently: "Tree: Ready (34 nodes)" and "Vectors: Ready (205 chunks)".
6. User can click [🔍 Inspect] to open the tree inspector at any time after tree construction begins — already-built nodes are visible immediately.

**Flow 3: PageIndex search**
1. User switches to the Search tab. Selects "PageIndex 🌳" sub-tab.
2. User selects a document from the dropdown (only documents with tree_status = "ready" appear).
3. User types a question and presses Enter or clicks [→].
4. The reasoning trace area activates. Steps appear one at a time as the LLM returns each action:
   - Each step shows: a step icon (● for in-progress, ✓ for complete), the action taken, the LLM's reasoning, and the elapsed time for that step.
   - Steps animate in with a fade+slide as they arrive.
5. The final answer appears in the ANSWER panel with source node references, total latency, and token count.
6. The tree inspector (if open) highlights the nodes that were visited during this query.

**Flow 4: Vector RAG search**
1. User switches to "Vector RAG 🔍" sub-tab.
2. User selects a document (only documents with vector_status = "ready" appear).
3. User types a question and presses Enter or clicks [→].
4. The system embeds the query, performs cosine similarity search, retrieves the top-k chunks, injects them into a prompt, and calls the LLM.
5. The RETRIEVED CHUNKS panel shows each chunk with its similarity score, chunk ID, and page range. Chunk text is displayed truncated to 200 characters with a "Show more" toggle.
6. The ANSWER panel shows the LLM's response with source chunk references, latency, and token count.

**Flow 5: Compare mode**
1. User switches to "Compare ⚖" sub-tab.
2. User selects a document (must have both tree and vectors ready — the dropdown only shows such documents).
3. User types a question and presses Enter.
4. Both pipelines fire simultaneously. Each column shows a loading skeleton until its result arrives. Whichever finishes first renders immediately; the other continues loading.
5. Results appear side by side. The user clicks 👍 or 👎 under each column to score accuracy.
6. The "Comparison History" section at the bottom logs all scored queries for this session with a running tally.

**Flow 6: Follow-up question (conversation memory)**
1. After receiving an answer (in any search sub-tab), the query input retains focus.
2. User types a follow-up: "What about the prior year comparison?"
3. The system sends the previous question+answer as conversation history alongside the new query.
4. For PageIndex: the LLM receives the tree plus the conversation context, and may start its navigation from the same region.
5. For Vector RAG: the LLM receives the previous chunks plus the new top-k chunks.
6. Conversation resets when: the user switches documents, switches sub-tabs, or clicks a "New conversation" button.

### Edge Cases and Error Handling

- **No API key**: All upload areas and query inputs are disabled. A persistent banner reads "Set your API key in Settings to get started." The Settings icon pulses gently to draw attention.
- **PDF with no extractable text (scanned images)**: After extraction, if total extracted text is under 50 characters for a multi-page PDF, display: "This PDF appears to be image-based. PageIndex SearchApp requires text-based PDFs. Try running OCR first." The document is not added.
- **LLM API error during tree construction**: The current progress is saved. The document shows "Tree: Error — click to retry." Clicking retries from the last successfully built node, not from the start.
- **LLM API error during query**: Display the partial reasoning trace (PageIndex) or a clear error message (Vector RAG): "LLM request failed after 3 retries. Check your API key and endpoint." Offer a "Retry" button.
- **LLM returns invalid JSON during tree construction or reasoning**: Attempt one automatic retry with a prompt addendum: "Your previous response was not valid JSON. Please respond ONLY with a valid JSON object." If that fails, fall back to treating the response as a plain-text answer (for queries) or skip the current node (for tree construction, logging a warning).
- **Empty document**: If the PDF has 0 pages or the markdown is empty, show "This document is empty" and do not create any records.
- **Very large document (200+ pages)**: Show an estimate before processing: "This document has ~200 pages. Tree construction will make approximately 15-25 LLM calls and cost an estimated $0.30-0.80 in API tokens. Proceed?" with Confirm/Cancel buttons.
- **IndexedDB quota exceeded**: If a write fails with a QuotaExceededError, display: "Browser storage is full. Delete some documents to free space." Show per-document storage sizes to help the user choose what to delete.
- **Embedding API returns different dimensions than stored embeddings**: If `embeddings_meta.dimensions` for a document doesn't match the current embedding model's output, display: "Embeddings were built with a different model. Rebuild vectors to search." and offer a one-click "Rebuild Vectors" button.
- **Switching documents mid-query**: If the user switches the document dropdown while a query is in progress, cancel the in-flight LLM calls (using AbortController) and clear the results area.
- **No results in vector search**: If all cosine similarity scores are below 0.5, display: "No sufficiently similar chunks found. Try rephrasing your question or using PageIndex search, which can navigate document structure even when phrasing doesn't match."

## Technical Implementation

### Architecture and File Structure

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
│   ├── engine/                     # Retrieval engine (UI-independent)
│   │   ├── llm-gateway.ts          # LLM chat + embedding API wrapper
│   │   ├── pdf-extract.ts          # PDF.js text extraction
│   │   ├── md-parse.ts             # Markdown heading parser
│   │   ├── tree-builder.ts         # PageIndex tree construction algorithm
│   │   ├── chunker.ts              # Text chunking for vector mode
│   │   ├── tree-reasoner.ts        # Agentic tree reasoning retrieval
│   │   ├── vector-searcher.ts      # Cosine similarity vector search
│   │   └── compare-runner.ts       # Parallel execution of both pipelines
│   │
│   ├── storage/                    # IndexedDB layer
│   │   ├── db.ts                   # Database open, upgrade, typed helpers
│   │   ├── documents.ts            # CRUD for documents store
│   │   ├── trees.ts                # CRUD for trees store
│   │   ├── node-content.ts         # CRUD for node_content store
│   │   ├── chunks.ts               # CRUD for chunks + embeddings stores
│   │   └── conversations.ts        # CRUD for conversations store
│   │
│   ├── components/                 # React UI components
│   │   ├── Layout.tsx              # Top-level layout, tab bar, settings trigger
│   │   ├── SettingsModal.tsx        # API key, model selection, parameters
│   │   ├── IndexTab.tsx            # Upload area + document list + inspector
│   │   ├── UploadArea.tsx          # Drag-and-drop + file picker + paste
│   │   ├── DocumentList.tsx        # List of indexed docs with status
│   │   ├── DocumentCard.tsx        # Single document: status bars, actions
│   │   ├── TreeInspector.tsx       # Collapsible tree view + node content
│   │   ├── TreeNode.tsx            # Single tree node (recursive)
│   │   ├── SearchTab.tsx           # Search sub-tab container + doc selector
│   │   ├── PageIndexSearch.tsx     # Query input + reasoning trace + answer
│   │   ├── VectorSearch.tsx        # Query input + chunks + answer
│   │   ├── CompareSearch.tsx       # Side-by-side + scoring + history
│   │   ├── ReasoningTrace.tsx      # Step-by-step trace display
│   │   ├── ChunkList.tsx           # Retrieved chunks with scores
│   │   ├── AnswerPanel.tsx         # Answer + sources + metrics (shared)
│   │   └── QueryInput.tsx          # Text input + submit (shared)
│   │
│   └── styles/
│       └── global.css              # CSS variables, base styles, component styles
```

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

The `engine/` directory has zero React imports. It can be extracted and used in a Node.js process, a Cloudflare Worker, or any other JavaScript runtime with IndexedDB (or with the storage layer swapped). This is the same design principle from the V1 specification.

### Dependencies

```json
{
  "name": "pageindex-searchapp",
  "private": true,
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "pdfjs-dist": "^4.8.0",
    "uuid": "^10.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.3.0",
    "@types/react-dom": "^18.3.0",
    "@types/uuid": "^10.0.0",
    "@vitejs/plugin-react": "^4.3.0",
    "typescript": "^5.5.0",
    "vite": "^5.4.0"
  }
}
```

No UI framework beyond React. No Tailwind (styles are hand-written CSS with variables). No state management library (React `useState`/`useReducer`/`useContext` only). No routing library (tabs are state-driven, not URL-driven). Minimal dependency surface.

### Vite Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  optimizeDeps: {
    include: ['pdfjs-dist']
  },
  build: {
    target: 'es2022'
  }
});
```

### Data Model (TypeScript Interfaces)

All types live in `src/types.ts`:

```typescript
// ─── Configuration ───────────────────────────────────────

export interface AppConfig {
  api_url: string;                // Default: "https://api.openai.com/v1"
  api_key: string;                // User-provided, stored in localStorage
  chat_model: string;             // Default: "gpt-4o"
  embedding_model: string;        // Default: "text-embedding-3-small"
  chunk_size: number;             // Default: 512 (tokens, approximate)
  chunk_overlap: number;          // Default: 50 (tokens)
  max_reasoning_iterations: number; // Default: 6
  temperature: number;            // Default: 0.1
  max_tokens: number;             // Default: 4096
  toc_check_pages: number;        // Default: 20
  max_tokens_per_node: number;    // Default: 20000
}

// ─── Document ────────────────────────────────────────────

export interface DocumentRecord {
  doc_id: string;                 // UUID, primary key
  filename: string;
  file_type: "pdf" | "markdown";
  page_count: number;
  total_tokens: number;
  tree_status: "pending" | "building" | "ready" | "error";
  vector_status: "pending" | "building" | "ready" | "skipped";
  tree_node_count: number;        // Updated as tree builds
  chunk_count: number;            // Updated as chunks are created
  chunks_embedded: number;        // Updated as embeddings complete
  storage_bytes: number;          // Estimated total IndexedDB usage
  error_message: string | null;
  created_at: string;             // ISO 8601
  updated_at: string;
}

// ─── Tree ────────────────────────────────────────────────

export interface TreeRecord {
  doc_id: string;                 // Primary key
  tree: TreeNode;
  doc_description: string;
  total_nodes: number;
  max_depth: number;
  built_with_model: string;
  built_at: string;
}

export interface TreeNode {
  node_id: string;                // "0001", "0002", etc.
  title: string;
  start_page: number;             // 0-indexed
  end_page: number;               // Inclusive
  summary: string;
  depth: number;                  // 0 = root
  parent_id: string | null;
  sub_nodes: TreeNode[];
}

// ─── Node Content ────────────────────────────────────────

export interface NodeContent {
  id: string;                     // `${doc_id}::${node_id}`
  doc_id: string;
  node_id: string;
  content: string;
  content_type: "text" | "table" | "mixed";
  token_count: number;
  page_numbers: number[];
}

// ─── Chunks and Embeddings ───────────────────────────────

export interface ChunkRecord {
  chunk_id: string;               // UUID
  doc_id: string;
  node_id: string | null;
  content: string;
  token_count: number;
  start_page: number;
  end_page: number;
  position_in_doc: number;
  embedding: number[] | null;     // Float32 vector, null until embedded
}

export interface EmbeddingsMeta {
  doc_id: string;
  model: string;
  dimensions: number;
  total_chunks: number;
  embedded_at: string;
}

// ─── Conversation ────────────────────────────────────────

export interface Conversation {
  conversation_id: string;
  doc_ids: string[];
  turns: ConversationTurn[];
  created_at: string;
  updated_at: string;
}

export interface ConversationTurn {
  role: "user" | "assistant" | "system";
  content: string;
  search_mode: "tree" | "vector" | "compare";
  sources: SourceRef[];
  reasoning_trace: ReasoningStep[];
  tokens_used: number;
  latency_ms: number;
  timestamp: string;
}

export interface SourceRef {
  doc_id: string;
  node_id: string | null;         // null for vector-only results
  chunk_ids: string[];             // empty for tree-only results
  page_numbers: number[];
}

// ─── Reasoning ───────────────────────────────────────────

export interface ReasoningStep {
  step_number: number;
  action: "read_tree" | "select_node" | "retrieve_content" | "follow_reference" | "answer";
  node_id: string | null;
  reasoning: string;
  content_excerpt: string | null;  // First 200 chars of retrieved content
  elapsed_ms: number;
}

// ─── Search Results ──────────────────────────────────────

export interface TreeSearchResult {
  answer: string;
  sources: SourceRef[];
  reasoning_trace: ReasoningStep[];
  tokens_used: number;
  latency_ms: number;
}

export interface VectorSearchResult {
  answer: string;
  sources: SourceRef[];
  retrieved_chunks: {
    chunk_id: string;
    content: string;
    score: number;
    page_numbers: number[];
  }[];
  tokens_used: number;
  latency_ms: number;
}

export interface CompareResult {
  tree: TreeSearchResult;
  vector: VectorSearchResult;
}

// ─── Scoring ─────────────────────────────────────────────

export interface ComparisonEntry {
  query: string;
  tree_accurate: boolean | null;  // null = not yet scored
  vector_accurate: boolean | null;
  timestamp: string;
}
```

### IndexedDB Schema

```typescript
// src/storage/db.ts

const DB_NAME = "pageindex_searchapp";
const DB_VERSION = 1;

let dbInstance: IDBDatabase | null = null;

export async function getDB(): Promise<IDBDatabase> {
  if (dbInstance) return dbInstance;

  return new Promise((resolve, reject) => {
    const request = indexedDB.open(DB_NAME, DB_VERSION);

    request.onupgradeneeded = (event) => {
      const db = (event.target as IDBOpenDBRequest).result;

      // Documents
      if (!db.objectStoreNames.contains("documents")) {
        const store = db.createObjectStore("documents", { keyPath: "doc_id" });
        store.createIndex("by_created", "created_at", { unique: false });
      }

      // Trees
      if (!db.objectStoreNames.contains("trees")) {
        db.createObjectStore("trees", { keyPath: "doc_id" });
      }

      // Node content
      if (!db.objectStoreNames.contains("node_content")) {
        const store = db.createObjectStore("node_content", { keyPath: "id" });
        store.createIndex("by_doc", "doc_id", { unique: false });
      }

      // Chunks
      if (!db.objectStoreNames.contains("chunks")) {
        const store = db.createObjectStore("chunks", { keyPath: "chunk_id" });
        store.createIndex("by_doc", "doc_id", { unique: false });
        store.createIndex("by_position", ["doc_id", "position_in_doc"], { unique: true });
      }

      // Embeddings metadata
      if (!db.objectStoreNames.contains("embeddings_meta")) {
        db.createObjectStore("embeddings_meta", { keyPath: "doc_id" });
      }

      // Conversations
      if (!db.objectStoreNames.contains("conversations")) {
        const store = db.createObjectStore("conversations", { keyPath: "conversation_id" });
        store.createIndex("by_updated", "updated_at", { unique: false });
      }
    };

    request.onsuccess = () => {
      dbInstance = request.result;
      resolve(dbInstance);
    };

    request.onerror = () => reject(request.error);
  });
}

// Generic typed helpers
export async function dbPut<T>(storeName: string, value: T): Promise<void> {
  const db = await getDB();
  return new Promise((resolve, reject) => {
    const tx = db.transaction(storeName, "readwrite");
    tx.objectStore(storeName).put(value);
    tx.oncomplete = () => resolve();
    tx.onerror = () => reject(tx.error);
  });
}

export async function dbGet<T>(storeName: string, key: string): Promise<T | undefined> {
  const db = await getDB();
  return new Promise((resolve, reject) => {
    const tx = db.transaction(storeName, "readonly");
    const request = tx.objectStore(storeName).get(key);
    request.onsuccess = () => resolve(request.result as T | undefined);
    request.onerror = () => reject(request.error);
  });
}

export async function dbGetAll<T>(storeName: string): Promise<T[]> {
  const db = await getDB();
  return new Promise((resolve, reject) => {
    const tx = db.transaction(storeName, "readonly");
    const request = tx.objectStore(storeName).getAll();
    request.onsuccess = () => resolve(request.result as T[]);
    request.onerror = () => reject(request.error);
  });
}

export async function dbGetAllByIndex<T>(
  storeName: string, indexName: string, key: string
): Promise<T[]> {
  const db = await getDB();
  return new Promise((resolve, reject) => {
    const tx = db.transaction(storeName, "readonly");
    const index = tx.objectStore(storeName).index(indexName);
    const request = index.getAll(key);
    request.onsuccess = () => resolve(request.result as T[]);
    request.onerror = () => reject(request.error);
  });
}

export async function dbDelete(storeName: string, key: string): Promise<void> {
  const db = await getDB();
  return new Promise((resolve, reject) => {
    const tx = db.transaction(storeName, "readwrite");
    tx.objectStore(storeName).delete(key);
    tx.oncomplete = () => resolve();
    tx.onerror = () => reject(tx.error);
  });
}

export async function dbDeleteByIndex(
  storeName: string, indexName: string, key: string
): Promise<void> {
  const db = await getDB();
  return new Promise((resolve, reject) => {
    const tx = db.transaction(storeName, "readwrite");
    const index = tx.objectStore(storeName).index(indexName);
    const cursorRequest = index.openCursor(key);
    cursorRequest.onsuccess = () => {
      const cursor = cursorRequest.result;
      if (cursor) {
        cursor.delete();
        cursor.continue();
      }
    };
    tx.oncomplete = () => resolve();
    tx.onerror = () => reject(tx.error);
  });
}
```

### Key Algorithms

All algorithms are carried forward from the V1 IndexedDB TINS specification. The following provides the complete implementation-ready form for each.

#### LLM Gateway (`src/engine/llm-gateway.ts`)

```typescript
import type { AppConfig } from "../types";

export async function chatCompletion(
  messages: { role: string; content: string }[],
  config: AppConfig,
  options?: { json_mode?: boolean; temperature?: number; signal?: AbortSignal }
): Promise<{ content: string; usage: { total_tokens: number } }> {
  const body: Record<string, unknown> = {
    model: config.chat_model,
    messages,
    temperature: options?.temperature ?? config.temperature,
    max_tokens: config.max_tokens
  };
  if (options?.json_mode) {
    body.response_format = { type: "json_object" };
  }

  let lastError: Error | null = null;
  for (let attempt = 0; attempt < 3; attempt++) {
    try {
      const res = await fetch(`${config.api_url}/chat/completions`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${config.api_key}`
        },
        body: JSON.stringify(body),
        signal: options?.signal
      });

      if (!res.ok) {
        const errText = await res.text();
        throw new Error(`LLM API error ${res.status}: ${errText}`);
      }

      const data = await res.json();
      return {
        content: data.choices[0].message.content,
        usage: data.usage ?? { total_tokens: 0 }
      };
    } catch (err) {
      if ((err as Error).name === "AbortError") throw err;
      lastError = err as Error;
      if (attempt < 2) {
        await new Promise(r => setTimeout(r, 1000 * Math.pow(2, attempt)));
      }
    }
  }
  throw lastError!;
}

export async function embedTexts(
  texts: string[],
  config: AppConfig
): Promise<{ embeddings: number[][]; dimensions: number }> {
  const res = await fetch(`${config.api_url}/embeddings`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${config.api_key}`
    },
    body: JSON.stringify({
      model: config.embedding_model,
      input: texts
    })
  });

  if (!res.ok) {
    const errText = await res.text();
    throw new Error(`Embedding API error ${res.status}: ${errText}`);
  }

  const data = await res.json();
  const embeddings = data.data
    .sort((a: { index: number }, b: { index: number }) => a.index - b.index)
    .map((d: { embedding: number[] }) => d.embedding);
  const dimensions = embeddings[0]?.length ?? 0;
  return { embeddings, dimensions };
}
```

#### PDF Text Extraction (`src/engine/pdf-extract.ts`)

```typescript
import * as pdfjsLib from "pdfjs-dist";

// Set worker source — use CDN to avoid bundling issues
pdfjsLib.GlobalWorkerOptions.workerSrc =
  `https://cdnjs.cloudflare.com/ajax/libs/pdf.js/${pdfjsLib.version}/pdf.worker.min.mjs`;

export interface ExtractedPage {
  page_number: number;   // 0-indexed
  text: string;
}

export interface ExtractionResult {
  pages: ExtractedPage[];
  page_count: number;
  total_chars: number;
}

export async function extractPDFText(
  file: File,
  onProgress?: (current: number, total: number) => void
): Promise<ExtractionResult> {
  const arrayBuffer = await file.arrayBuffer();
  const pdf = await pdfjsLib.getDocument({ data: arrayBuffer }).promise;
  const pages: ExtractedPage[] = [];
  let totalChars = 0;

  for (let i = 1; i <= pdf.numPages; i++) {
    const page = await pdf.getPage(i);
    const textContent = await page.getTextContent();
    const text = textContent.items
      .map((item: { str?: string }) => item.str ?? "")
      .join(" ")
      .trim();
    pages.push({ page_number: i - 1, text });
    totalChars += text.length;
    onProgress?.(i, pdf.numPages);
  }

  return { pages, page_count: pdf.numPages, total_chars: totalChars };
}
```

#### Markdown Parser (`src/engine/md-parse.ts`)

```typescript
import type { TreeNode } from "../types";

interface ParsedSection {
  level: number;
  title: string;
  content: string;
  start_line: number;
  end_line: number;
}

export function parseMarkdownToSections(md: string): ParsedSection[] {
  const lines = md.split("\n");
  const sections: ParsedSection[] = [];
  let current: ParsedSection | null = null;

  for (let i = 0; i < lines.length; i++) {
    const match = lines[i].match(/^(#{1,6})\s+(.+)$/);
    if (match) {
      if (current) {
        current.end_line = i - 1;
        current.content = current.content.trim();
        sections.push(current);
      }
      current = {
        level: match[1].length,
        title: match[2].trim(),
        content: "",
        start_line: i,
        end_line: i
      };
    } else if (current) {
      current.content += lines[i] + "\n";
    }
  }
  if (current) {
    current.end_line = lines.length - 1;
    current.content = current.content.trim();
    sections.push(current);
  }
  return sections;
}

export function sectionsToTree(sections: ParsedSection[]): {
  tree: TreeNode;
  contentMap: Map<string, string>;
} {
  // Build a tree from heading levels.
  // The root is a virtual node. Level 1 headings are direct children.
  // Level 2 headings nest under the most recent level 1, etc.

  let nodeCounter = 0;
  function nextId(): string {
    nodeCounter++;
    return String(nodeCounter).padStart(4, "0");
  }

  const root: TreeNode = {
    node_id: nextId(),
    title: "Document Root",
    start_page: 0,
    end_page: sections.length > 0 ? sections[sections.length - 1].end_line : 0,
    summary: "",
    depth: 0,
    parent_id: null,
    sub_nodes: []
  };

  const contentMap = new Map<string, string>();
  const stack: TreeNode[] = [root];

  for (const section of sections) {
    const node: TreeNode = {
      node_id: nextId(),
      title: section.title,
      start_page: section.start_line,
      end_page: section.end_line,
      summary: "",  // Will be filled by LLM
      depth: section.level,
      parent_id: null,
      sub_nodes: []
    };

    // Find the correct parent: walk up the stack until depth < section.level
    while (stack.length > 1 && stack[stack.length - 1].depth >= section.level) {
      stack.pop();
    }
    const parent = stack[stack.length - 1];
    node.parent_id = parent.node_id;
    parent.sub_nodes.push(node);
    stack.push(node);

    contentMap.set(node.node_id, section.content);
  }

  return { tree: root, contentMap };
}
```

#### Tree Builder (`src/engine/tree-builder.ts`)

```typescript
import type { AppConfig, TreeNode, NodeContent, TreeRecord } from "../types";
import { chatCompletion } from "./llm-gateway";
import type { ExtractedPage } from "./pdf-extract";
import { dbPut } from "../storage/db";

export interface TreeBuildProgress {
  phase: "scanning" | "building" | "summarising" | "complete" | "error";
  nodes_built: number;
  nodes_estimated: number;
  current_section: string;
}

export async function buildTreeFromPDF(
  docId: string,
  pages: ExtractedPage[],
  config: AppConfig,
  onProgress?: (progress: TreeBuildProgress) => void
): Promise<TreeRecord> {
  // Phase 1: Scan for structure
  onProgress?.({ phase: "scanning", nodes_built: 0, nodes_estimated: 0, current_section: "Table of Contents" });

  const tocPages = pages.slice(0, config.toc_check_pages).map(p => p.text).join("\n\n---PAGE BREAK---\n\n");

  const structureResponse = await chatCompletion(
    [
      {
        role: "system",
        content: `You are a document structure analyser. Given the following pages from the beginning of a document, identify the top-level sections. If a table of contents is present, use it. Otherwise infer sections from headings, topic shifts, and formatting cues. Return ONLY a JSON array of objects with fields: "title" (string), "start_page" (0-indexed integer), "end_page" (0-indexed integer, inclusive). Cover ALL pages in the document (total pages: ${pages.length}). Do not skip any page ranges.`
      },
      { role: "user", content: tocPages }
    ],
    config,
    { json_mode: true, temperature: 0.0 }
  );

  let topSections: { title: string; start_page: number; end_page: number }[];
  try {
    const parsed = JSON.parse(structureResponse.content);
    topSections = Array.isArray(parsed) ? parsed : parsed.sections ?? [parsed];
  } catch {
    // Fallback: one node per 10 pages
    topSections = [];
    for (let i = 0; i < pages.length; i += 10) {
      topSections.push({
        title: `Pages ${i + 1}-${Math.min(i + 10, pages.length)}`,
        start_page: i,
        end_page: Math.min(i + 9, pages.length - 1)
      });
    }
  }

  const estimatedNodes = topSections.length * 3; // rough estimate
  onProgress?.({ phase: "building", nodes_built: 0, nodes_estimated: estimatedNodes, current_section: "" });

  // Phase 2: Recursively build tree
  let nodeCounter = 0;
  function nextId(): string {
    nodeCounter++;
    return String(nodeCounter).padStart(4, "0");
  }

  async function buildNode(
    title: string, startPage: number, endPage: number,
    depth: number, parentId: string | null
  ): Promise<TreeNode> {
    const nodeId = nextId();
    const sectionText = pages
      .slice(startPage, endPage + 1)
      .map(p => p.text)
      .join("\n\n");
    const tokenEstimate = Math.ceil(sectionText.split(/\s+/).length * 1.33);

    // Store raw content
    const contentRecord: NodeContent = {
      id: `${docId}::${nodeId}`,
      doc_id: docId,
      node_id: nodeId,
      content: sectionText,
      content_type: "text",
      token_count: tokenEstimate,
      page_numbers: Array.from({ length: endPage - startPage + 1 }, (_, i) => startPage + i)
    };
    await dbPut("node_content", contentRecord);

    onProgress?.({
      phase: "building",
      nodes_built: nodeCounter,
      nodes_estimated: Math.max(estimatedNodes, nodeCounter + 1),
      current_section: title
    });

    // Decide: split further or summarise
    if (tokenEstimate > config.max_tokens_per_node && (endPage - startPage) >= 2) {
      // Ask LLM to split into subsections
      const splitResponse = await chatCompletion(
        [
          {
            role: "system",
            content: `You are a document structure analyser. The following text is from a section titled "${title}" spanning pages ${startPage + 1} to ${endPage + 1}. Break it into 2-6 logical subsections. Return ONLY a JSON array of objects with fields: "title" (string), "start_page" (0-indexed integer), "end_page" (0-indexed integer, inclusive). Every page from ${startPage} to ${endPage} must be covered.`
          },
          { role: "user", content: sectionText.slice(0, 15000) } // Limit to avoid overflow
        ],
        config,
        { json_mode: true, temperature: 0.0 }
      );

      let subSections: { title: string; start_page: number; end_page: number }[];
      try {
        const parsed = JSON.parse(splitResponse.content);
        subSections = Array.isArray(parsed) ? parsed : [parsed];
      } catch {
        subSections = [];
      }

      const subNodes: TreeNode[] = [];
      for (const sub of subSections) {
        const child = await buildNode(sub.title, sub.start_page, sub.end_page, depth + 1, nodeId);
        subNodes.push(child);
      }

      // Summarise based on children
      const childSummaries = subNodes.map(n => `- ${n.title}: ${n.summary}`).join("\n");
      const summaryResponse = await chatCompletion(
        [
          {
            role: "system",
            content: "Summarise this section in 1-3 sentences based on its subsection summaries."
          },
          { role: "user", content: `Section: ${title}\nSubsections:\n${childSummaries}` }
        ],
        config,
        { temperature: 0.3 }
      );

      return {
        node_id: nodeId, title, start_page: startPage, end_page: endPage,
        summary: summaryResponse.content.trim(),
        depth, parent_id: parentId, sub_nodes: subNodes
      };
    } else {
      // Leaf node: summarise directly
      const summaryResponse = await chatCompletion(
        [
          {
            role: "system",
            content: "Summarise the following document section in 1-3 sentences. Be specific about what data, topics, or conclusions it contains."
          },
          { role: "user", content: sectionText.slice(0, 10000) }
        ],
        config,
        { temperature: 0.3 }
      );

      return {
        node_id: nodeId, title, start_page: startPage, end_page: endPage,
        summary: summaryResponse.content.trim(),
        depth, parent_id: parentId, sub_nodes: []
      };
    }
  }

  // Build all top-level sections
  const rootChildren: TreeNode[] = [];
  for (const section of topSections) {
    const node = await buildNode(section.title, section.start_page, section.end_page, 1, "0000");
    rootChildren.push(node);
  }

  // Generate document description
  const docDescResponse = await chatCompletion(
    [
      {
        role: "system",
        content: "Describe this document in one paragraph based on its section structure."
      },
      {
        role: "user",
        content: rootChildren.map(n => `${n.title} (pp. ${n.start_page + 1}-${n.end_page + 1}): ${n.summary}`).join("\n")
      }
    ],
    config,
    { temperature: 0.3 }
  );

  const root: TreeNode = {
    node_id: "0000",
    title: "Document Root",
    start_page: 0,
    end_page: pages.length - 1,
    summary: docDescResponse.content.trim(),
    depth: 0,
    parent_id: null,
    sub_nodes: rootChildren
  };

  function countNodes(n: TreeNode): number {
    return 1 + n.sub_nodes.reduce((sum, c) => sum + countNodes(c), 0);
  }
  function maxDepth(n: TreeNode): number {
    return n.sub_nodes.length === 0 ? n.depth : Math.max(...n.sub_nodes.map(maxDepth));
  }

  const treeRecord: TreeRecord = {
    doc_id: docId,
    tree: root,
    doc_description: docDescResponse.content.trim(),
    total_nodes: countNodes(root),
    max_depth: maxDepth(root),
    built_with_model: config.chat_model,
    built_at: new Date().toISOString()
  };

  await dbPut("trees", treeRecord);

  onProgress?.({
    phase: "complete",
    nodes_built: countNodes(root),
    nodes_estimated: countNodes(root),
    current_section: ""
  });

  return treeRecord;
}
```

#### Chunker (`src/engine/chunker.ts`)

```typescript
import type { AppConfig, ChunkRecord } from "../types";
import type { ExtractedPage } from "./pdf-extract";
import { v4 as uuid } from "uuid";

export function chunkDocument(
  docId: string,
  pages: ExtractedPage[],
  config: AppConfig
): ChunkRecord[] {
  const fullText = pages.map(p => p.text).join("\n\n");
  const words = fullText.split(/\s+/);
  const tokensPerWord = 1.33; // approximate
  const chunkSizeWords = Math.floor(config.chunk_size / tokensPerWord);
  const overlapWords = Math.floor(config.chunk_overlap / tokensPerWord);
  const stepSize = Math.max(1, chunkSizeWords - overlapWords);

  // Build a character-offset-to-page-number map
  let charOffset = 0;
  const pageBreaks: { offset: number; page: number }[] = [];
  for (const page of pages) {
    pageBreaks.push({ offset: charOffset, page: page.page_number });
    charOffset += page.text.length + 2; // +2 for \n\n joiner
  }

  function pageForOffset(offset: number): number {
    let page = 0;
    for (const pb of pageBreaks) {
      if (pb.offset <= offset) page = pb.page;
      else break;
    }
    return page;
  }

  const chunks: ChunkRecord[] = [];
  let wordIndex = 0;
  let position = 0;

  while (wordIndex < words.length) {
    const windowWords = words.slice(wordIndex, wordIndex + chunkSizeWords);
    const content = windowWords.join(" ");
    const tokenCount = Math.ceil(windowWords.length * tokensPerWord);

    // Approximate character offset for page mapping
    const charsBeforeChunk = words.slice(0, wordIndex).join(" ").length;
    const charsEndChunk = charsBeforeChunk + content.length;
    const startPage = pageForOffset(charsBeforeChunk);
    const endPage = pageForOffset(charsEndChunk);

    chunks.push({
      chunk_id: uuid(),
      doc_id: docId,
      node_id: null,  // Will be backfilled if tree exists
      content,
      token_count: tokenCount,
      start_page: startPage,
      end_page: endPage,
      position_in_doc: position,
      embedding: null
    });

    position++;
    wordIndex += stepSize;
  }

  return chunks;
}
```

#### Tree Reasoner (`src/engine/tree-reasoner.ts`)

```typescript
import type { AppConfig, TreeNode, ReasoningStep, TreeSearchResult, SourceRef } from "../types";
import { chatCompletion } from "./llm-gateway";
import { dbGet } from "../storage/db";
import type { NodeContent } from "../types";

export async function treeReasoningSearch(
  query: string,
  docId: string,
  tree: TreeNode,
  config: AppConfig,
  conversationHistory: { role: string; content: string }[],
  onStep?: (step: ReasoningStep) => void,
  signal?: AbortSignal
): Promise<TreeSearchResult> {
  const startTime = performance.now();
  let totalTokens = 0;
  const trace: ReasoningStep[] = [];
  const visitedNodes: string[] = [];

  // Serialize tree (without raw content — just structure + summaries)
  function serializeTree(node: TreeNode, indent: number = 0): string {
    const pad = "  ".repeat(indent);
    let out = `${pad}[${node.node_id}] ${node.title} (pp. ${node.start_page + 1}-${node.end_page + 1})\n`;
    out += `${pad}  Summary: ${node.summary}\n`;
    for (const child of node.sub_nodes) {
      out += serializeTree(child, indent + 1);
    }
    return out;
  }

  const treeText = serializeTree(tree);

  const systemPrompt = `You are a document retrieval agent. You have access to a hierarchical index of a document. The index shows section IDs, titles, page ranges, and summaries.

DOCUMENT INDEX:
${treeText}

At each step, respond with ONLY a JSON object (no markdown, no explanation outside JSON) with one of these formats:

To retrieve a section's full content:
{ "action": "retrieve_node", "node_id": "XXXX", "reasoning": "why this section likely contains the answer" }

To provide the final answer:
{ "action": "answer", "content": "your answer based on retrieved content", "sources": [{ "node_id": "XXXX", "page_numbers": [N] }], "reasoning": "how you derived this answer" }

RULES:
- Navigate the tree systematically. Start with the most promising top-level section.
- Read node content when you need details. Summaries are not sufficient for precise answers.
- Follow cross-references: if content says "see Appendix X" or "refer to Table Y", navigate there.
- You may retrieve up to ${config.max_reasoning_iterations} nodes before you must answer.
- If you are unsure, say so in your answer rather than guessing.`;

  const messages: { role: string; content: string }[] = [
    { role: "system", content: systemPrompt },
    ...conversationHistory,
    { role: "user", content: query }
  ];

  for (let iteration = 0; iteration < config.max_reasoning_iterations; iteration++) {
    const stepStart = performance.now();

    const response = await chatCompletion(messages, config, { json_mode: true, signal });
    totalTokens += response.usage.total_tokens;

    let parsed: Record<string, unknown>;
    try {
      parsed = JSON.parse(response.content);
    } catch {
      // Retry with hint
      messages.push({ role: "assistant", content: response.content });
      messages.push({ role: "system", content: "Your response was not valid JSON. Please respond ONLY with a JSON object." });
      continue;
    }

    const elapsed = Math.round(performance.now() - stepStart);

    if (parsed.action === "retrieve_node") {
      const nodeId = parsed.node_id as string;
      visitedNodes.push(nodeId);

      // Load content from IndexedDB
      const content = await dbGet<NodeContent>("node_content", `${docId}::${nodeId}`);
      const contentText = content?.content ?? "(Content not found for this node)";

      const step: ReasoningStep = {
        step_number: iteration + 1,
        action: "retrieve_content",
        node_id: nodeId,
        reasoning: parsed.reasoning as string,
        content_excerpt: contentText.slice(0, 200),
        elapsed_ms: elapsed
      };
      trace.push(step);
      onStep?.(step);

      // Add to messages for next iteration
      messages.push({ role: "assistant", content: response.content });
      messages.push({
        role: "system",
        content: `Content for node ${nodeId} (${content?.page_numbers?.map(p => p + 1).join(", ") ?? "?"} pages):\n\n${contentText}`
      });

    } else if (parsed.action === "answer") {
      const step: ReasoningStep = {
        step_number: iteration + 1,
        action: "answer",
        node_id: null,
        reasoning: parsed.reasoning as string,
        content_excerpt: null,
        elapsed_ms: elapsed
      };
      trace.push(step);
      onStep?.(step);

      const sources: SourceRef[] = ((parsed.sources as { node_id: string; page_numbers?: number[] }[]) ?? []).map(s => ({
        doc_id: docId,
        node_id: s.node_id,
        chunk_ids: [],
        page_numbers: s.page_numbers ?? []
      }));

      return {
        answer: parsed.content as string,
        sources,
        reasoning_trace: trace,
        tokens_used: totalTokens,
        latency_ms: Math.round(performance.now() - startTime)
      };
    }
  }

  // Force answer after max iterations
  messages.push({
    role: "system",
    content: "You have reached the maximum number of retrieval steps. Provide your best answer NOW based on the content you have retrieved."
  });

  const finalResponse = await chatCompletion(messages, config, { json_mode: true, signal });
  totalTokens += finalResponse.usage.total_tokens;

  let finalParsed: Record<string, unknown>;
  try {
    finalParsed = JSON.parse(finalResponse.content);
  } catch {
    finalParsed = { content: finalResponse.content, sources: [], reasoning: "Forced answer after max iterations" };
  }

  const finalStep: ReasoningStep = {
    step_number: config.max_reasoning_iterations + 1,
    action: "answer",
    node_id: null,
    reasoning: "Forced answer after reaching maximum reasoning steps.",
    content_excerpt: null,
    elapsed_ms: 0
  };
  trace.push(finalStep);
  onStep?.(finalStep);

  return {
    answer: (finalParsed.content ?? finalParsed.answer ?? finalResponse.content) as string,
    sources: ((finalParsed.sources as SourceRef[]) ?? []).map(s => ({
      doc_id: docId, node_id: s.node_id, chunk_ids: [], page_numbers: s.page_numbers ?? []
    })),
    reasoning_trace: trace,
    tokens_used: totalTokens,
    latency_ms: Math.round(performance.now() - startTime)
  };
}
```

#### Vector Searcher (`src/engine/vector-searcher.ts`)

```typescript
import type { AppConfig, ChunkRecord, VectorSearchResult, SourceRef } from "../types";
import { chatCompletion, embedTexts } from "./llm-gateway";
import { dbGetAllByIndex } from "../storage/db";

function cosineSimilarity(a: number[], b: number[]): number {
  let dot = 0, normA = 0, normB = 0;
  for (let i = 0; i < a.length; i++) {
    dot += a[i] * b[i];
    normA += a[i] * a[i];
    normB += b[i] * b[i];
  }
  const denom = Math.sqrt(normA) * Math.sqrt(normB);
  return denom === 0 ? 0 : dot / denom;
}

export async function vectorSearch(
  query: string,
  docId: string,
  config: AppConfig,
  conversationHistory: { role: string; content: string }[],
  topK: number = 5,
  signal?: AbortSignal
): Promise<VectorSearchResult> {
  const startTime = performance.now();

  // Embed the query
  const { embeddings } = await embedTexts([query], config);
  const queryEmbedding = embeddings[0];

  // Load all chunks for this document
  const chunks = await dbGetAllByIndex<ChunkRecord>("chunks", "by_doc", docId);
  const embeddedChunks = chunks.filter(c => c.embedding !== null);

  if (embeddedChunks.length === 0) {
    return {
      answer: "No embedded chunks found for this document. Please wait for vector indexing to complete.",
      sources: [],
      retrieved_chunks: [],
      tokens_used: 0,
      latency_ms: Math.round(performance.now() - startTime)
    };
  }

  // Score and rank
  const scored = embeddedChunks.map(chunk => ({
    chunk,
    score: cosineSimilarity(queryEmbedding, chunk.embedding!)
  }));
  scored.sort((a, b) => b.score - a.score);
  const topChunks = scored.slice(0, topK);

  // Build context for LLM
  const contextText = topChunks
    .map((item, i) => `[Chunk ${i + 1}, pp. ${item.chunk.start_page + 1}-${item.chunk.end_page + 1}, score: ${item.score.toFixed(3)}]\n${item.chunk.content}`)
    .join("\n\n---\n\n");

  const messages: { role: string; content: string }[] = [
    {
      role: "system",
      content: `You are a question-answering assistant. Answer the user's question based ONLY on the provided document chunks. If the chunks do not contain enough information to answer, say so explicitly. Cite the chunk numbers and page numbers you use.

RETRIEVED CHUNKS:
${contextText}`
    },
    ...conversationHistory,
    { role: "user", content: query }
  ];

  const response = await chatCompletion(messages, config, { signal });

  const sources: SourceRef[] = [{
    doc_id: docId,
    node_id: null,
    chunk_ids: topChunks.map(c => c.chunk.chunk_id),
    page_numbers: [...new Set(topChunks.flatMap(c => [c.chunk.start_page, c.chunk.end_page]))]
  }];

  return {
    answer: response.content,
    sources,
    retrieved_chunks: topChunks.map(item => ({
      chunk_id: item.chunk.chunk_id,
      content: item.chunk.content,
      score: item.score,
      page_numbers: Array.from(
        { length: item.chunk.end_page - item.chunk.start_page + 1 },
        (_, i) => item.chunk.start_page + i
      )
    })),
    tokens_used: response.usage.total_tokens,
    latency_ms: Math.round(performance.now() - startTime)
  };
}
```

#### Compare Runner (`src/engine/compare-runner.ts`)

```typescript
import type { AppConfig, TreeNode, CompareResult } from "../types";
import type { ReasoningStep } from "../types";
import { treeReasoningSearch } from "./tree-reasoner";
import { vectorSearch } from "./vector-searcher";

export async function compareSearch(
  query: string,
  docId: string,
  tree: TreeNode,
  config: AppConfig,
  conversationHistory: { role: string; content: string }[],
  onTreeStep?: (step: ReasoningStep) => void,
  signal?: AbortSignal
): Promise<CompareResult> {
  // Run both in parallel
  const [treeResult, vectorResult] = await Promise.all([
    treeReasoningSearch(query, docId, tree, config, conversationHistory, onTreeStep, signal),
    vectorSearch(query, docId, config, conversationHistory, 5, signal)
  ]);

  return { tree: treeResult, vector: vectorResult };
}
```

### Application State Management

The app uses React Context for global state, avoiding external state libraries:

```typescript
// src/App.tsx — simplified state structure

interface AppState {
  activeTab: "index" | "search";
  activeSearchTab: "pageindex" | "vector" | "compare";
  selectedDocId: string | null;
  config: AppConfig;
  configReady: boolean;           // true once API key is set
  documents: DocumentRecord[];    // Loaded from IndexedDB on mount
  inspectingDocId: string | null; // Which doc's tree is open in inspector
  conversationHistory: { role: string; content: string }[];
  comparisonHistory: ComparisonEntry[];
}
```

State flows:
- `config` is loaded from `localStorage` on mount. If `api_key` is empty, `configReady = false` and the Settings modal opens.
- `documents` is loaded from IndexedDB on mount and updated via polling or callbacks from ingestion processes.
- `conversationHistory` resets when `selectedDocId` changes or `activeSearchTab` changes.
- `comparisonHistory` persists for the browser session only (in React state, not IndexedDB).

## Style Guide

- **Aesthetic direction**: Developer-tool / code-editor aesthetic. Dark by default. Think VS Code meets a data dashboard — utilitarian, information-dense, but refined.
- **Background**: `#0d1117` (GitHub dark). Surface panels: `#161b22`. Borders: `#30363d`.
- **Primary text**: `#e6edf3`. Muted text: `#8b949e`. Links/accents: `#58a6ff`.
- **Success green**: `#3fb950`. Warning amber: `#d29922`. Error red: `#f85149`.
- **PageIndex accent**: `#a371f7` (purple — distinguishes tree reasoning results from vector).
- **Vector accent**: `#58a6ff` (blue — distinguishes vector results).
- **Typography**: `"JetBrains Mono", "Fira Code", monospace` for reasoning traces, code, tree inspector, and metrics. `"IBM Plex Sans", -apple-system, sans-serif` for body text, headings, and UI labels. Load JetBrains Mono and IBM Plex Sans from Google Fonts.
- **Tab bar**: Horizontal pill-style tabs with a subtle background highlight on the active tab. No underlines. Active tab uses primary text color; inactive uses muted.
- **Search sub-tabs**: Smaller pill tabs with emoji prefixes (🌳 PageIndex, 🔍 Vector RAG, ⚖ Compare). Active tab gets a 2px bottom border in the respective accent colour (purple for PageIndex, blue for Vector, split gradient for Compare).
- **Cards**: Document cards in the Index tab use `surface` background with `border` on all sides, 8px border-radius, 16px padding. Hover: subtle border colour shift to `#58a6ff`.
- **Progress bars**: Thin (4px height), rounded, green fill on dark track. Animate width smoothly.
- **Reasoning trace**: Vertical timeline with a thin left border. Step indicators are small filled circles (● in-progress = amber, ✓ complete = green). Each step fades in with a 200ms slide-down animation as it arrives.
- **Chunk list**: Each chunk in a bordered card with the similarity score as a coloured badge (>0.9 green, 0.8-0.9 blue, <0.8 muted). Chunk text in monospace, truncated with "Show more".
- **Compare columns**: Equal-width columns separated by a 1px dashed border. Column headers use their respective accent colours. The accuracy buttons (👍/👎) use hover highlights in green/red.
- **Spacing**: 8px base unit. Padding: 16px (cards, panels), 24px (major sections). Gap between items: 12px.
- **Transitions**: 150ms ease for all interactive state changes (hover, tab switch, expand/collapse). Reasoning steps: 200ms fade+translateY on entry.
- **Settings modal**: Dark overlay at 50% opacity. Modal centered, 480px max-width, same surface/border styling.
- **Responsive**: Below 768px, the Index tab's tree inspector moves to a full-screen slide-over. The Compare tab stacks columns vertically.

## Testing Scenarios

1. **Basic PDF round-trip**: Upload a 10-page PDF. Verify text extraction completes, tree builds with multiple nodes, vector index builds with chunks. Switch to Search, run a query in PageIndex mode, verify an answer with sources is returned.
2. **Markdown round-trip**: Paste markdown with 3 heading levels. Verify tree matches heading structure exactly. Query in PageIndex mode. Verify answer uses correct section.
3. **Cross-reference navigation**: Upload a PDF where a section says "see Appendix X for details." Query the topic covered in Appendix X. Verify the PageIndex reasoning trace shows it navigated from the reference to the appendix.
4. **Vector search sanity**: Upload a document. Run a simple factual query in Vector RAG mode. Verify top chunks are relevant (high cosine scores) and the answer is reasonable.
5. **Compare mode — PageIndex wins**: Upload a document where the answer requires following a cross-reference. Run in Compare mode. Verify PageIndex finds the correct answer while Vector returns only the referencing section (not the appendix).
6. **Compare mode — both succeed**: Upload a document. Ask a straightforward question whose answer lives in a single paragraph. Verify both modes return correct answers.
7. **Conversation follow-up**: In PageIndex mode, ask about Topic A. Then ask "What about Topic B in the same section?" Verify the second answer uses context from the first turn.
8. **No API key state**: Clear localStorage. Reload. Verify the Settings modal opens, upload/query controls are disabled, and the banner is visible.
9. **Large document warning**: Upload a 200+ page PDF. Verify the cost estimate modal appears before processing begins.
10. **Tree inspector interaction**: Build a tree. Open the tree inspector. Click various nodes. Verify raw content displays. Run a query. Verify visited nodes are highlighted in the tree.
11. **Error recovery**: Start a tree build, then disconnect the network mid-way (or use an invalid API key). Verify the document shows "Error — click to retry" and retry works.
12. **Delete document cascade**: Upload a document, build both indexes. Delete the document. Verify all related records (tree, node_content, chunks, conversations) are removed from IndexedDB.

## Performance Goals

- **PDF text extraction**: Under 3 seconds for a 100-page PDF.
- **Tree construction**: 10-30 seconds for a 50-page document (dominated by LLM latency). UI must remain responsive throughout.
- **Vector index construction**: Under 60 seconds for 200 chunks (embedding API batching).
- **PageIndex query**: 3-10 seconds for a typical single-document query (3-5 reasoning steps).
- **Vector query**: 1-3 seconds (one embedding call + brute-force cosine + one LLM call).
- **Compare query**: Maximum of the two individual latencies (parallel execution).
- **Brute-force cosine similarity**: Under 50ms for 5,000 chunks.
- **IndexedDB operations**: Under 20ms for single-record reads. Under 200ms for loading all chunks of a large document.
- **Initial app load**: Under 2 seconds (Vite dev server). Under 1 second (production build).
- **All UI interactions**: Response within 100ms. No layout shift on tab switches.

## Accessibility Requirements

- Full keyboard navigation: Tab cycles through all interactive elements. Enter/Space activates buttons and tabs. Arrow keys navigate within the tree inspector.
- ARIA roles: `role="tablist"` on tab bars, `role="tab"` on tabs, `role="tabpanel"` on content areas, `role="tree"` on tree inspector, `role="treeitem"` on tree nodes, `role="status"` on progress indicators and answer panels.
- Screen reader: Upload status announced ("Document uploaded, building tree..."), search results announced ("Answer ready, 5 reasoning steps"), comparison scores announced.
- Focus management: After upload, focus moves to the new document card. After search, focus moves to the answer panel. After opening Settings, focus traps within the modal.
- Colour contrast: All text meets WCAG AA (minimum 4.5:1 against background). Accent colours on dark backgrounds exceed this.
- Reduced motion: Respect `prefers-reduced-motion` — disable slide-in animations, use instant transitions.

## Extended Features (Optional)

- **Export/import tree JSON**: Download the PageIndex tree as a `.json` file. Import a pre-built tree to skip construction.
- **Streaming LLM answers**: Use Server-Sent Events or ReadableStream to display answer tokens as they arrive.
- **Token cost estimator**: Display estimated API cost per operation (tree construction, vector indexing, query) based on model pricing and token counts.
- **Query history**: Persist all queries and results in IndexedDB. Provide a "History" tab showing past queries with their mode, answer, and accuracy score.
- **Batch benchmark**: Upload a set of question-answer pairs as a JSON file. Run all queries through both modes automatically. Export a CSV with per-query metrics (latency, tokens, accuracy).
- **Dark/light theme toggle**: Light theme with the same accent system. Default dark, toggle in Settings.
- **Chunk-to-node mapping visualisation**: In Hybrid mode, show which tree nodes the top vector chunks map to, visualising the overlap between vector similarity and structural navigation.
