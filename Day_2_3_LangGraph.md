
markdown
# 🌐 LangGraph Example – Graph-Based Agent Orchestration

## 1. Why LangGraph?
- **LangChain** → linear orchestration (prompt → tool → response).  
- **LangGraph** → graph orchestration (nodes + edges).  
- Useful for:
  - Multi‑agent workflows  
  - Conditional branching  
  - Retry/fallback logic  
  - Parallel execution  

<img width="1024" height="1536" alt="Copilot_20260703_145515" src="https://github.com/user-attachments/assets/f8afe043-c73b-4184-a8ab-f67052769d79" />

---

## 2. Core Concepts
- **Nodes** → functions or agents (e.g., flight search, hotel booking).  
- **Edges** → transitions between nodes (e.g., after flights → go to hotels).  
- **Graph** → defines the workflow as a directed graph.  

---

## 3. Simple LangGraph Flow

### Install
```text
pip install langgraph langchain langchain-openai
Example Code
python
from langgraph.graph import StateGraph, END
from langchain.chat_models import AzureChatOpenAI

# 1. Define state
class State(dict):
    pass

# 2. Define nodes (agents)
def flight_node(state: State):
    state["flights"] = ["DEL-2026-06-30", "DEL-2026-07-01"]
    return state

def hotel_node(state: State):
    state["hotels"] = ["Hotel A", "Hotel B"]
    return state

def budget_node(state: State):
    # Simple check
    state["budget_ok"] = True if len(state["hotels"]) < 3 else False
    return state

# 3. Build graph
workflow = StateGraph(State)

workflow.add_node("flights", flight_node)
workflow.add_node("hotels", hotel_node)
workflow.add_node("budget", budget_node)

# Define edges
workflow.add_edge("flights", "hotels")
workflow.add_edge("hotels", "budget")
workflow.add_edge("budget", END)

# 4. Compile graph
app = workflow.compile()

# 5. Run workflow
result = app.invoke(State())
print(result)
4. Output
json
{
  "flights": ["DEL-2026-06-30", "DEL-2026-07-01"],
  "hotels": ["Hotel A", "Hotel B"],
  "budget_ok": true
}
5. Flow
Start with LangChain → show linear orchestration.

Introduce LangGraph → explain nodes, edges, graph structure.

Run simple example → flights → hotels → budget.

Extend → add retry/fallback, parallel nodes, or memory.

Compare → LangChain (linear) vs LangGraph (graph‑based).


==========================================================================
LangChain vs LangGraph – Workflow Comparison
🧩 LangChain (Linear Flow)
LangChain executes tasks step‑by‑step — each stage feeds directly into the next.
Ideal for simple pipelines like “prompt → tool → memory → output.”

text
+--------+      +---------+      +---------+      +---------+      +---------+
|  USER  | ---> |  AGENT  | ---> |  LOGIC  | ---> |  TOOLS  | ---> | MEMORY  |
|Request |      |Activates|      |Understands|    |Searches |      |Stores   |
+--------+      +---------+      +---------+      +---------+      +---------+
       1️⃣ Request     2️⃣ Activate     3️⃣ Understand     4️⃣ Search     5️⃣ Store
🧠 Key Idea:  
LangChain is linear orchestration — each component runs sequentially.
Useful for prompt chaining, tool calling, and memory‑based chatbots.

🌐 LangGraph (Branching Flow)
LangGraph models workflows as nodes and edges — multiple agents can run in parallel or conditionally.
Perfect for multi‑agent orchestration and goal‑based autonomy.

text
        +---------+
        |  USER   |
        | Request |
        +----+----+
             |
             v
      +------+------+
      | LangGraph   |
      | Orchestrator|
      +------+------+
             |
   -------------------------
   |           |           |
   v           v           v
+------+   +------+   +------+
|Flights|  |Hotels |  |Weather|
+---+--+   +---+--+   +---+--+
    |          |          |
    v          v          v
   -------------------------
             |
             v
        +---------+
        |  Budget |
        | Optimizer|
        +----+----+
             |
             v
        +---------+
        |   END   |
        +---------+
🧠 Key Idea:  
LangGraph is graph orchestration — nodes represent agents, edges define transitions.
It supports branching, parallel execution, retry/fallback, and memory persistence.


