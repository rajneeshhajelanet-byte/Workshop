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



4. Example Code Snippets
🔌 Postgres Client
python
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
🧠 Prompt for Azure OpenAI
python
prompt = f"""
SYSTEM CONTEXT:
You are a PostgreSQL expert optimizing queries using LIVE DB stats + Excel rules.

USER QUERY: {user_query}

LIVE DATABASE PERFORMANCE:
- Execution Time: {exec_time}
- Sequential Scan Active: {is_scan}
- Stats: {perf_data}

KNOWLEDGE BASE RULE:
- Rule ID: {rule_id}
- Recommended Action: {rule_action}

TASK:
Return ONLY JSON with keys:
1. optimized_sql
2. explanation
3. validation_checklist
4. confidence_score
5. reasoning
"""
⚡ LLM Call
python
def llm_generate(prompt: str) -> str:
    response = client.chat.completions.create(
        model=DEPLOYMENT_NAME,
        messages=[
            {"role": "system", "content": "You are a PostgreSQL Performance Engineer. Return strict JSON."},
            {"role": "user", "content": prompt}
        ],
        temperature=0
    )
    return response.choices[0].message.content.strip()
5. Example Flow
User Query → PostgresClient → DB Stats → Prompt → Azure OpenAI → JSON Optimization Result

6. Why Custom Framework?
Lightweight compared to LangChain

Transparent routing logic

Direct control over agents and connectors

Enterprise‑ready: integrates DB stats, Excel rules, and LLM optimization seamlessly

Code

---

✅ This Markdown file now clearly documents your **Custom Framework** with:  
- Overview  
- Components  
- Flow diagram  
- Code snippets  
- Prompt design  
- Why you chose custom orchestration  

Would you like me to also prepare a **side‑by‑side comparison table** (Custom Framework vs L
