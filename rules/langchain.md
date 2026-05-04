# LangChain Integration

`langchain-surrealdb` is the official LangChain integration package
maintained in collaboration between SurrealDB and the LangChain community.
It exposes SurrealDB as a first-class **vector store**, **document store**,
**chat message history backend**, and **retriever** so RAG, agent, and
conversational chains can run end-to-end on a single SurrealDB instance --
records, embeddings, edges, and chat memory live in the same database and
can be queried with one SurrealQL statement when needed.

> **Scope**: this rule covers Python (`langchain-surrealdb`) and JavaScript
> (`@langchain/surrealdb`). Both packages target LangChain 0.3+ and the v2
> SurrealDB SDKs.

---

## When to Reach for This Integration

| Goal | Why SurrealDB + LangChain fits |
|------|--------------------------------|
| RAG over a knowledge base where documents have structured metadata and graph relationships | Vector search + graph traversal in one query, no separate metadata store |
| Conversational agents with persistent multi-tenant chat history | `SurrealChatMessageHistory` is row-level-permissioned by `$auth.id` |
| Hybrid retrieval (vector + keyword + filter + graph) | One SurrealQL query joins all four; LangChain's hybrid retriever wraps it |
| Multi-tenant SaaS that needs per-tenant data isolation | DEFINE ACCESS / namespace-per-tenant + LangChain's session-keyed retrievers |
| Pipelines that already store domain data in SurrealDB | Avoid moving embeddings to a second database |

If you only need a pure vector store with no metadata, no permissions, and no
graph -- pick a dedicated vector DB. If your data already has structure, this
integration is the path of least resistance.

---

## Python Package

### Installation

```bash
pip install langchain-surrealdb

# Or, with embeddings provider extras
pip install "langchain-surrealdb[openai]"
pip install "langchain-surrealdb[huggingface]"
```

The package depends on `langchain-core>=0.3` and `surrealdb>=2.0` (the v2
Python SDK GA). It supports both the synchronous `Surreal` client and the
asynchronous `AsyncSurreal` client.

### Vector Store

```python
from langchain_surrealdb.vectorstores import SurrealDBVectorStore
from langchain_openai import OpenAIEmbeddings
from langchain_core.documents import Document

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")  # 1536 dims

# Connect over WebSocket; matches the SDK's connection string convention
store = SurrealDBVectorStore.from_endpoint(
    endpoint="ws://localhost:8000/rpc",
    namespace="myapp",
    database="prod",
    username="root",                # local dev only
    password="root",
    table="document",               # SurrealDB table that backs the store
    embeddings=embeddings,
    distance="cosine",              # cosine | euclidean | manhattan
    index_kind="hnsw",              # hnsw | mtree | bruteforce
    hnsw_m=16,
    hnsw_ef_construction=200,
)

# Add documents (creates records + computes embeddings)
docs = [
    Document(page_content="SurrealDB supports HNSW", metadata={"section": "vector"}),
    Document(page_content="RELATE creates graph edges", metadata={"section": "graph"}),
]
ids = store.add_documents(docs)

# Similarity search
results = store.similarity_search("How do I do graph traversal?", k=4)

# Similarity with metadata filter (compiled to SurrealQL WHERE clause)
results = store.similarity_search(
    "vector indexes",
    k=4,
    filter={"section": "vector"},
)

# Score in addition to documents
hits = store.similarity_search_with_score("vector indexes", k=4)
for doc, score in hits:
    print(score, doc.page_content)
```

The vector store creates the table, vector field, and index automatically the
first time it is used; subsequent runs are idempotent. Manage the schema via
SurrealKit if you prefer explicit DEFINE statements.

### Async Vector Store

```python
import asyncio
from langchain_surrealdb.vectorstores import AsyncSurrealDBVectorStore

async def main():
    store = await AsyncSurrealDBVectorStore.from_endpoint(
        endpoint="ws://localhost:8000/rpc",
        namespace="myapp", database="prod",
        username="root", password="root",
        embeddings=embeddings,
    )
    results = await store.asimilarity_search("graph traversal", k=4)

asyncio.run(main())
```

### Retriever

```python
retriever = store.as_retriever(
    search_type="similarity",       # "similarity" | "mmr" | "similarity_score_threshold"
    search_kwargs={"k": 6, "score_threshold": 0.7, "filter": {"published": True}},
)

# Plug into any LangChain chain
docs = retriever.invoke("explain HNSW")
```

For maximum marginal relevance:

```python
retriever = store.as_retriever(
    search_type="mmr",
    search_kwargs={"k": 6, "fetch_k": 30, "lambda_mult": 0.5},
)
```

### Chat Message History

```python
from langchain_surrealdb.chat_history import SurrealChatMessageHistory

history = SurrealChatMessageHistory(
    endpoint="ws://localhost:8000/rpc",
    namespace="myapp", database="prod",
    username="root", password="root",
    table="chat_message",
    session_id="user_alice_session_42",
)

# LangChain runnable wiring
from langchain_core.runnables.history import RunnableWithMessageHistory
chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: SurrealChatMessageHistory(..., session_id=session_id),
    input_messages_key="input",
    history_messages_key="history",
)
```

Pair `session_id` with a DEFINE ACCESS record-level rule so messages are
queryable only by their owning user (`rules/security.md`).

### Hybrid Retriever (Vector + Keyword + Graph)

```python
from langchain_surrealdb.retrievers import SurrealHybridRetriever

retriever = SurrealHybridRetriever(
    vector_store=store,
    keyword_field="page_content",
    graph_traversal=(
        "->referenced_by->document"   # one-hop expansion through graph edges
    ),
    weights={"vector": 0.6, "keyword": 0.3, "graph": 0.1},
)

docs = retriever.invoke("What does RELATE do?")
```

Internally this compiles to a single SurrealQL statement that joins vector
similarity, full-text `MATCHES`, and graph traversal -- no per-component
round-trip.

---

## JavaScript / TypeScript Package

### Installation

```bash
npm install @langchain/surrealdb @langchain/core surrealdb
```

The package supports the v2 JS SDK GA. WASM and Node.js embedded engines work
out of the box.

### Vector Store

```typescript
import { SurrealVectorStore } from "@langchain/surrealdb";
import { OpenAIEmbeddings } from "@langchain/openai";
import { Document } from "@langchain/core/documents";

const embeddings = new OpenAIEmbeddings({ model: "text-embedding-3-small" });

const store = await SurrealVectorStore.fromEndpoint(
  {
    endpoint: "ws://localhost:8000/rpc",
    namespace: "myapp",
    database: "prod",
    auth: { username: "root", password: "root" },
    table: "document",
    distance: "cosine",
    indexKind: "hnsw",
  },
  embeddings,
);

await store.addDocuments([
  new Document({ pageContent: "SurrealDB supports HNSW", metadata: { section: "vector" } }),
  new Document({ pageContent: "RELATE creates edges", metadata: { section: "graph" } }),
]);

const results = await store.similaritySearch("graph traversal", 4);
const filtered = await store.similaritySearch("vector indexes", 4, { section: "vector" });
```

### Retriever

```typescript
const retriever = store.asRetriever({
  k: 6,
  searchType: "similarity",
  filter: { published: true },
});

const docs = await retriever.invoke("explain HNSW");
```

### Chat History

```typescript
import { SurrealChatMessageHistory } from "@langchain/surrealdb";
import { RunnableWithMessageHistory } from "@langchain/core/runnables";

const history = new SurrealChatMessageHistory({
  endpoint: "ws://localhost:8000/rpc",
  namespace: "myapp",
  database: "prod",
  auth: { username: "root", password: "root" },
  table: "chat_message",
  sessionId: "user_alice_session_42",
});
```

---

## Schema Footprint

The integration creates the following tables (names configurable):

```surql
-- Vector store
DEFINE TABLE document SCHEMAFULL;
DEFINE FIELD page_content ON TABLE document TYPE string;
DEFINE FIELD metadata     ON TABLE document FLEXIBLE TYPE object;
DEFINE FIELD embedding    ON TABLE document TYPE array<float>;
DEFINE INDEX idx_document_embedding ON TABLE document FIELDS embedding
  HNSW DIMENSION 1536 DIST COSINE;

-- Chat history
DEFINE TABLE chat_message SCHEMAFULL;
DEFINE FIELD session_id ON TABLE chat_message TYPE string;
DEFINE FIELD role       ON TABLE chat_message TYPE string ASSERT $value INSIDE ['system','user','assistant','tool'];
DEFINE FIELD content    ON TABLE chat_message TYPE string;
DEFINE FIELD created_at ON TABLE chat_message TYPE datetime DEFAULT time::now();
DEFINE INDEX idx_chat_message_session ON TABLE chat_message FIELDS session_id, created_at;
```

Pin these definitions in your `database/schema/` directory under SurrealKit
control if you want explicit, reviewed schema evolution rather than the
auto-create-on-first-write behavior.

---

## Multi-Tenant + Permissioned Stores

Combine the integration with `DEFINE ACCESS` / `DEFINE TABLE ... PERMISSIONS`
to enforce per-tenant isolation server-side rather than only filtering in
Python:

```surql
DEFINE TABLE document SCHEMAFULL
  PERMISSIONS
    FOR select WHERE tenant_id = $auth.tenant_id
    FOR create, update, delete WHERE tenant_id = $auth.tenant_id;

DEFINE FIELD tenant_id ON TABLE document TYPE string;
```

Then signin with the tenant-scoped record user before constructing the vector
store:

```python
import surrealdb

db = surrealdb.Surreal()
db.connect("ws://localhost:8000/rpc")
db.signin({
    "namespace": "myapp", "database": "prod",
    "access": "tenant_user",
    "variables": {"email": "alice@acme.com", "password": "..."},
})
db.use("myapp", "prod")

store = SurrealDBVectorStore.from_client(db, table="document", embeddings=embeddings)
```

Row-level permissions filter both writes and similarity search reads, so even
if application code constructs the wrong filter, the database will not return
another tenant's documents.

---

## Performance Tips

- **Pick the right index**: `hnsw` for >100k vectors, `mtree` for medium
  cardinality with frequent updates, `bruteforce` for <10k vectors or testing.
  See `rules/vector-search.md` for trade-offs.
- **Pre-create indexes**: auto-create-on-first-write incurs index build cost on
  the first query. Use SurrealKit rollouts for production schema.
- **Use record IDs as document IDs**: pass `RecordId("document", uuid)` when
  inserting; this avoids a secondary unique index on a `doc_id` field.
- **Co-locate metadata**: filter fields used in `WHERE` should be indexed
  (`DEFINE INDEX ... FIELDS section`) to keep similarity-with-filter on the
  fast path.
- **Async over sync** for I/O-bound RAG pipelines; the v2 Python SDK's
  `AsyncSurreal` is reentrant-safe.
- **Embedding batching**: pass lists of documents to `add_documents`; the
  integration batches embedding calls and SurrealDB inserts.

---

## Cross-References

- `rules/vector-search.md` -- HNSW, MTree, distance metrics, RAG patterns
- `rules/data-modeling.md` -- record IDs, schemafull definitions, graph edges
- `rules/security.md` -- DEFINE ACCESS, row-level permissions for multi-tenancy
- `rules/sdks.md` -- Python and JS v2 SDK details that LangChain wraps
- `rules/surrealmcp.md` -- agent-side access (complements LangChain's app-side use)
- `rules/surrealml.md` -- when you want the model itself stored in SurrealDB,
  not just the embeddings
