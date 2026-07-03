
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
