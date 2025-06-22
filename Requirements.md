# NeuraSQL MVP — Language-Agnostic Requirements

## 1. Functional Requirements

### 1.1 SQL Frontend  
- **Query Types**  
  - **CREATE TABLE**: define tables and column schemas  
  - **INSERT**: add new rows of data  
  - **SELECT**: read rows, support `WHERE` filters and column projections  
- **Extended Function**  
  - **SIMILARITY(col, text)**: compute semantic similarity between a text column and a query string  

### 1.2 Storage & Durability  
- **In-Memory Store**  
  - Maintain tables as collections of rows keyed by table name  
  - Support basic CRUD operations in memory  
- **On-Disk Persistence**  
  - Write every mutating operation (e.g. CREATE, INSERT) to a write-ahead log (WAL)  
  - On startup, replay the WAL to reconstruct in-memory state  
  - Optionally snapshot full table state periodically for faster recovery  

### 1.3 Vector Search Integration  
- **Embeddings**  
  - Compute a fixed-length numeric embedding for each text row at insert time  
- **Approximate Nearest Neighbor (k-NN)**  
  - Index embeddings in a high-dimensional search structure  
  - Expose a matching operator that returns top-K most similar rows  

### 1.4 Interactive Shell & API  
- **REPL / CLI**  
  - Read–Eval–Print Loop for issuing SQL and similarity queries interactively  
  - Subcommands for auxiliary tasks (e.g. `advisor suggest`)  
- **Programmatic Interface**  
  - Expose a minimal RPC/REST or library API for embedding your engine in other apps  

### 1.5 Index Advisor Prototype  
- **Query Logging**  
  - Record each query’s text, timestamp, execution time, and basic statistics (table sizes, filter complexity)  
- **Offline Model Training**  
  - Train a simple classifier or decision-tree to predict when adding a particular index would significantly reduce latency  
- **Online Suggestions**  
  - Provide a command or API endpoint that returns SQL `CREATE INDEX` statements based on current workload  

## 2. Non-Functional Requirements

### 2.1 Correctness & Reliability  
- Queries must return correct results according to SQL semantics (with the added similarity function).  
- WAL and replay mechanism must guarantee no data loss under normal shutdown or crash recovery.

### 2.2 Performance  
- **Baseline**: Handle at least 1,000 INSERTs/sec and 100 SELECTs/sec on moderate hardware.  
- **Vector Search**: Return top-10 nearest neighbors in under 50 ms for datasets up to 100,000 embeddings.

### 2.3 Scalability & Extensibility  
- Architecture must cleanly separate parsing, execution, storage, and AI components.  
- Storage engine should be pluggable (e.g. row-store vs. columnar vs. LSM style).  
- Ability to add replicas or shards in future versions without major rewrites.

### 2.4 Observability & Testing  
- **Logging**: Structured logs for all queries, errors, and advisor actions.  
- **Metrics**: Expose counters and histograms for request rates, latencies, and resource usage.  
- **Automated Tests**:  
  - Unit tests for parser, storage, vector index, and advisor logic  
  - Integration tests that exercise full SQL → storage → query → result pipeline  
  - Benchmark tests to detect performance regressions

### 2.5 Documentation  
- **User Guide**: Quickstart instructions for installation, REPL usage, and embedding.  
- **Developer Guide**: Architecture overview, module responsibilities, and extension points.  
- **API Reference**: Spec for CLI commands, RPC/REST endpoints, and configuration options.

## 3. Advanced (“Stretch”) Features

1. **MVCC & Time-Travel**  
   - Multi-version concurrency control allowing queries “as of” a past timestamp  
2. **Self-Tuning Storage**  
   - Automatic selection or tuning of on-disk format (e.g. B-Tree vs. LSM) based on workload  
3. **Fully Automated Index Creation**  
   - Background service that not only suggests but also creates and validates new indexes without manual approval  
4. **Sharding & Replication**  
   - Horizontal partitioning of tables and synchronous/asynchronous replicas for high availability  
5. **REST Dashboard**  
   - Web UI showing query plans, latency graphs, embedding space visualizations, and advisor recommendations  
6. **Blockchain-Anchored Audit Logs**  
   - Cryptographically chain WAL snapshots and anchor Merkle roots on a public ledger for tamper-proof auditing  
7. **Plugin Ecosystem**  
   - Define extension points so community plugins can add new functions, storage engines, or machine-learning modules  
