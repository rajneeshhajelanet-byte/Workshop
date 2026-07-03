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



Example
class PostgresClient:
    def __init__(self, config):
        self.config = config

    def execute_query(self, sql):
        conn = psycopg2.connect(**self.config, sslmode='require')
        cur = conn.cursor(cursor_factory=extras.RealDictCursor)
        cur.execute(sql)
        res = cur.fetchall() if cur.description else [{"message": "Success"}]
        conn.close()
        return res

    def investigate(self, sql):
        wrapped = f"EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) {sql.strip().rstrip(';')}"
        raw_output = self.execute_query(wrapped)
        plan_root = raw_output[0]['QUERY PLAN'][0]
        plan_details = plan_root.get('Plan', {})
        return {
            "exec_time": plan_root.get('Execution Time', 0),
            "hits": plan_details.get('Shared Hit Blocks', 0),
            "reads": plan_details.get('Shared Read Blocks', 0),
            "is_scan": "Seq Scan" in str(plan_details),
            "total_cost": plan_details.get('Total Cost', 0)
        }

