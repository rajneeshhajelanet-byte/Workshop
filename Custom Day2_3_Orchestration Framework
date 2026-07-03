# ⚙️ Custom Orchestration Framework – Database + LLM Integration

## 1. Overview
Instead of relying on heavy frameworks like LangChain, we built our own **lightweight orchestration layer**.  
This gave us:
- Tighter control over agent routing  
- Leaner dependencies  
- Transparency in how agents interact  

---

## 2. Core Components

### 🔑 Routing Agent
- `route_query()` → classifies intent (e.g., `db_observability`, `query_optimization`, `out_of_domain`)  
- Keyword‑based dispatch, later extendable with Azure OpenAI classification  

### 📡 Streaming Responses
- `StreamingResponse` yields **status updates + final results**  
- Keeps the user informed while long queries run  

### 🤝 Agent Handoff
- Based on intent, calls:  
  - `handle_db_observability()`  
  - `handle_query_optimization()`  

### 🗄 Data Layer
- `PostgresClient` (psycopg2) executes queries  
- Supports `EXPLAIN ANALYZE` for live performance stats (RAM hits, disk reads, sequential scans)  

### 🧠 LLM Integration
- Direct call to **Azure OpenAI** with strict JSON enforcement  
- Prompts include **system context + DB stats + Excel rules**  

### 📚 Knowledge Base
- Excel rules loaded manually via `initialize_kb()`  
- Rules matched against queries for optimization hints  

---

## 3. Flow Diagram

```text
+-----------+        +----------------+        +------------------+
|  User     | -----> | Keyword Router | -----> | Assigned Agent   |
|  Query    |        | (Dispatch)     |        | (DB / Excel / LLM)|
+-----------+        +----------------+        +------------------+
       |                        |                        |
       v                        v                        v
+----------------+       +----------------+        +------------------+
| DB Agent       |       | Excel Agent    |        | Optimization Agent|
| (EXPLAIN stats)|       | (Rule matching)|        | (LLM prompt build)|
+----------------+       +----------------+        +------------------+
                                                         |
                                                         v
                                               +------------------+
                                               | JSON Optimized   |
                                               | SQL Response     |
                                               +------------------+
