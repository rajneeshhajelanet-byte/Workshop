
# 🔗 Why LangChain?

## 1. Motivation
Before LangChain, we built a **custom orchestration framework**:
- Manual keyword router  
- Custom PostgresClient for DB stats  
- Direct Azure OpenAI calls with strict JSON enforcement  
- Excel rules loaded manually  

This gave us **full control** and **lean dependencies**, but required writing a lot of boilerplate code.

LangChain solves this by providing:
- Pre‑built abstractions for **LLMs, tools, and agents**  
- Easy integration with **databases, APIs, and retrievers**  
- Faster prototyping with less custom code  

---

# ⚙️ LangChain Example – Database + Azure OpenAI Integration

## 2. Step‑by‑Step

### 🧠 Connect to Azure OpenAI
```python
from langchain.chat_models import AzureChatOpenAI

llm = AzureChatOpenAI(
    deployment_name="your-deployment",
    openai_api_version="2024-12-01-preview",
    temperature=0
)

🗄 Connect to Postgres via SQLDatabase
python
from langchain.sql_database import SQLDatabase

db = SQLDatabase.from_uri("postgresql+psycopg2://user:password@host:5432/dbname")
🛠 Wrap Database as a Tool
python
from langchain.agents import Tool

db_tool = Tool(
    name="DatabaseQuery",
    func=db.run,
    description="Run SQL queries and return results"
)

🤖 Initialize Agent with Tool
python
from langchain.agents import initialize_agent, AgentType

agent = initialize_agent(
    tools=[db_tool],
    llm=llm,
    agent_type=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)
▶️ Run a Query
python
response = agent.run("Find the top 5 customers by revenue")
print(response)
3. Dependencies
text
# Core LangChain framework
langchain>=0.2.0

# Azure OpenAI integration
langchain-openai>=0.1.0
openai>=1.0.0

# Database connectors
sqlalchemy>=2.0.0
psycopg2-binary>=2.9.9

# Environment variable loader
python-dotenv>=1.0.0

# Optional: community integrations
langchain-community>=0.0.30
4. Key Notes
LangChain → core orchestration framework

langchain-openai → Azure OpenAI connector

sqlalchemy + psycopg2 → required for SQLDatabase.from_uri

python-dotenv → loads secrets from .env

langchain-community → extra tools, retrievers, loaders

<img width="487" height="220" alt="image" src="https://github.com/user-attachments/assets/bc178f17-0e36-4ce6-a59d-86202386be09" />

