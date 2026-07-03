
# 👥 CrewAI – Collaborative Agent Framework
<img width="1274" height="619" alt="image" src="https://github.com/user-attachments/assets/d398f904-b10d-4c01-9dec-17ddd3cdd318" />
<img width="1364" height="568" alt="image" src="https://github.com/user-attachments/assets/92674be4-0ce0-4ff5-aacf-7b4089ecffb9" />

## 1. Why CrewAI?
- **LangChain** → linear orchestration (prompt → tool → memory).  
- **LangGraph** → graph orchestration (nodes + edges).  
- **CrewAI** → collaborative orchestration (multiple agents working together).  

CrewAI enables **specialized agents** to collaborate on a shared goal.  
Each agent can:
- Use its own tools (LLM, APIs, databases).  
- Communicate with other agents.  
- Pass intermediate results.  
- Coordinate through a central **crew orchestrator**.  

---

## 2. Conceptual Flow (Student View)

```text
+---------+
|  USER   |
| Request |
+----+----+
     |
     v
+------------+
|   CrewAI   |
| Orchestrator|
+------+------+
       |
  -------------------------
  |           |           |
  v           v           v
+------+   +------+   +------+
|Research| |Writer | |Reviewer|
| Agent  | | Agent | | Agent  |
+---+----+ +---+----+ +---+----+
     |         |          |
     v         v          v
  -------------------------
       |
       v
+-------------+
| Final Output|
| (Report/Plan)|
+-------------+
3. Simple Code Example
python
from crewai import Agent, Crew

# Define agents
researcher = Agent(name="Researcher", role="Collect data", goal="Find insights")
writer = Agent(name="Writer", role="Draft report", goal="Summarize findings")
reviewer = Agent(name="Reviewer", role="Validate content", goal="Ensure accuracy")

# Create crew
crew = Crew(agents=[researcher, writer, reviewer])

# Run workflow
result = crew.run("Prepare a summary of AI orchestration frameworks")
print(result)
4. Teaching Flow
Start with LangChain → single agent, linear tasks.

Move to LangGraph → multiple agents, graph orchestration.

Introduce CrewAI → collaborative multi‑agent teamwork.

Show example → Researcher → Writer → Reviewer → Output.

Discuss enterprise use cases → report generation, compliance automation, AI project planning.

5. Key Takeaway
LangChain → linear pipelines.

LangGraph → branching workflows.

CrewAI → collaborative teamwork among agents.
