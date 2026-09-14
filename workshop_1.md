# AI/ML & Agentic AI — Detailed Interview & Workshop Prep
### For Enterprise/Solution Architect & AI Leadership Roles

This expands the topics from your target JDs into full talking points: **what it is, why interviewers ask it, the answer structure, and how it maps to your PayerIQ project.** Use it to build 2–3 STAR stories, not to memorize definitions.

---

## PART A — ML & Deep Learning Foundations

### A1. ML/DL Frameworks: Scikit-learn, TensorFlow, PyTorch, Keras

| Framework | What it's for | Position it as |
|---|---|---|
| **Scikit-learn** | Classical ML — regression, classification, clustering, preprocessing pipelines | Your go-to for structured/tabular data, feature engineering, quick baselines before reaching for deep learning |
| **TensorFlow** | Production-grade DL, strong deployment story (TF Serving, TFLite, TF.js) | Enterprise-friendly, good for standardized MLOps pipelines |
| **PyTorch** | Research-friendly, dynamic computation graphs, dominant in LLM/agentic research | What almost all modern LLMs and fine-tuning libraries (HuggingFace, PEFT) are built on |
| **Keras** | High-level API, now the front-end for TensorFlow (and multi-backend: JAX/PyTorch) | Fastest way to prototype a neural net, good for teaching/workshop demos |

**Interview framing:** "As an architect, I care less about writing kernels in any one framework and more about knowing which one fits the problem, the team's skill set, and the deployment target. For classical tabular problems I'd default to scikit-learn; for anything LLM/agentic, the ecosystem is PyTorch-first (HuggingFace Transformers, PEFT, LangChain internals)."

**Likely question:** *"Have you trained models yourself, or mostly consumed APIs?"*
Answer honestly and pivot to strength: you're hands-on with the **consumption/integration/orchestration layer** (Azure OpenAI, Azure AI Search, RAG pipelines in PayerIQ) rather than training models from scratch — which is exactly what an *architect* role needs, versus a data scientist role.

---

### A2. Neural Network Architectures — CNNs, RNNs, LSTMs, Transformers

| Architecture | Core idea | Best for | One-line "why it lost relevance / why it matters now" |
|---|---|---|---|
| **CNN** (Convolutional NN) | Sliding filters detect local spatial patterns, pooled hierarchically | Images, spatial data, some 1D signal/text tasks | Still dominant in vision; largely superseded by Transformers (ViT) at scale |
| **RNN** (Recurrent NN) | Processes sequences step-by-step, carrying a hidden state forward | Early sequence modeling (time series, simple NLP) | Suffers from vanishing gradients over long sequences |
| **LSTM** (Long Short-Term Memory) | RNN variant with gates (input/forget/output) to preserve long-range dependencies | Sequence tasks needing longer memory than vanilla RNN | Solved vanishing gradients but still sequential (slow to train, can't parallelize) |
| **Transformer** | Self-attention lets every token attend to every other token in parallel | Virtually all modern LLMs, and increasingly vision (ViT) | Parallelizable training, scales with data/compute — the architecture behind GPT/Claude/Llama/Gemini |

**The one deep-dive worth being fluent in: Self-Attention**
- Every token is projected into **Query, Key, Value** vectors.
- Attention score = similarity (dot product) of Query with every Key → softmax → weighted sum of Values.
- **Multi-head attention** runs this in parallel across several "heads" so the model can attend to different relationship types (syntax, coreference, etc.) simultaneously.
- **Positional encoding** is added because attention itself has no notion of word order.

**Interview-ready one-liner:** *"A transformer replaced sequential recurrence with parallel self-attention — every token directly attends to every other token, weighted by relevance — which is what made training on internet-scale data computationally feasible and gave rise to LLMs."* You don't need to derive the math — just explain it conceptually and confidently (this is explicitly flagged in your study plan as "enough to explain, not derive").

---

### A3. ML Workflows — Preprocessing, Training, Evaluation, Optimization

Structure any answer around this pipeline (works for classic ML *and* as an analogy for LLM/RAG pipelines):

1. **Data preprocessing** — cleaning, handling missing values, encoding categoricals, normalization/scaling, train/val/test split, handling class imbalance.
2. **Feature engineering / selection** — domain-driven features, dimensionality reduction (PCA), feature importance.
3. **Model training** — algorithm selection, hyperparameter tuning (grid/random search, Bayesian optimization), cross-validation.
4. **Evaluation** — accuracy/precision/recall/F1 for classification, RMSE/MAE for regression, ROC-AUC, confusion matrix; for LLMs: groundedness, hallucination rate, relevance scoring (see Part B).
5. **Optimization & deployment** — model compression, quantization, monitoring for drift, retraining triggers.

**Architect-level framing:** Don't get pulled into being a data scientist in the interview — say you understand the *full lifecycle* well enough to design the platform/pipeline around it (MLOps), even if a data science team owns steps 2–3 day-to-day.

---

## PART B — LLMs, Prompting & RAG (Tier 1 — non-negotiable)

### B1. LLM Landscape

| Model family | Maker | Notable trait to mention |
|---|---|---|
| **GPT (GPT-4/4o/5-series)** | OpenAI | Broadest tooling ecosystem, Assistants/Agents SDK, strong function-calling |
| **Claude (Sonnet/Opus/Haiku)** | Anthropic | Strong at long-context reasoning, agentic tool-use, constitutional-AI safety approach, native MCP support |
| **Llama** | Meta | Open-weights, self-hostable — relevant when data residency/compliance blocks external APIs (a real BFSI/healthcare concern) |
| **Mistral** | Mistral AI | Efficient open-weight models, good cost/performance for smaller deployments |
| **Gemini** | Google | Deep GCP/Workspace integration, strong multimodal (native video/audio) |

**Key architectural point to make:** licensing/deployment model matters more than benchmark scores in enterprise conversations — **API-hosted (OpenAI/Anthropic/Azure OpenAI)** vs. **self-hosted open-weights (Llama/Mistral on your own infra)** is a governance, cost, and data-residency decision, not just a capability decision. This is exactly the kind of trade-off an *architect* is expected to own.

---

### B2. Prompt Engineering

| Technique | What it is | When to use |
|---|---|---|
| **Zero-shot** | Ask directly, no examples | Simple, well-defined tasks the model already "knows" |
| **Few-shot** | Give 2–5 examples in the prompt | Format-sensitive or domain-specific output |
| **Chain-of-thought (CoT)** | Ask the model to "think step by step" before answering | Multi-step reasoning, math, complex decisions |
| **Prompt chaining** | Break a task into sequential prompts, each feeding the next | Complex workflows (e.g., extract → validate → format) |
| **System vs. user prompts** | System prompt sets persistent behavior/role/guardrails; user prompt is the per-turn ask | Every production LLM app — this is literally how you set guardrails |

**Advanced patterns worth naming:** ReAct (reasoning + acting interleaved — the backbone of most agent loops), self-consistency (sample multiple CoT paths, vote), tree-of-thought (explore multiple reasoning branches).

**Tie to PayerIQ:** your `openai_service.py` almost certainly separates a system prompt (role: "you are a business analyst generating STTM/FRD documents") from user prompts (the actual requirements ask) — call this out explicitly as a real example of prompt engineering in production.

---

### B3. RAG Architecture (you have hands-on experience — lead with this)

**Pipeline:** `Ingestion → Chunking → Embedding → Vector Store → Retrieval → Generation`

1. **Ingestion** — pull source docs (PDF, SharePoint, DB) into a processing pipeline.
2. **Chunking** — split into retrievable units.
   - *Fixed-size chunking*: simple, fast, but can cut sentences/ideas mid-thought.
   - *Semantic chunking*: split on meaning boundaries (paragraphs, headers, topic shifts) — better retrieval quality, more compute.
   - *Overlap*: carry a few sentences between chunks so context isn't lost at boundaries.
3. **Embedding** — convert each chunk into a vector via an embedding model (e.g., Azure OpenAI `text-embedding-3`).
4. **Vector store** — persist vectors for similarity search.
5. **Retrieval** — at query time, embed the query, do similarity search (often **hybrid**: vector + keyword/BM25) to fetch top-k relevant chunks.
6. **Generation** — feed retrieved chunks + query into the LLM as context, generate grounded answer.

**Vector DB trade-offs (this is a favorite architect question):**

| DB | Type | Trade-off |
|---|---|---|
| **Azure AI Search** | Managed, hybrid search built-in | Your real experience — tightly integrated with Azure OpenAI/Blob/SharePoint; less portable outside Azure |
| **Pinecone** | Fully managed SaaS | Very easy ops, but vendor lock-in and cost at scale |
| **FAISS** | Self-hosted library (Meta) | Free, fast, no managed service — you own scaling/HA |
| **Chroma** | Self-hosted, dev-friendly | Great for prototyping/local dev, less battle-tested at enterprise scale |
| **Weaviate** | Self-hosted or managed, GraphQL API | Rich filtering/hybrid search, more operational overhead |

**Your answer:** *"On PayerIQ I used Azure AI Search for both the vector index and hybrid (vector + keyword) retrieval, since it's natively integrated with Azure OpenAI and our document sources were already in SharePoint/Blob via Graph API. For a client without an Azure commitment, I'd evaluate FAISS or Weaviate for self-hosted control, or Pinecone if the team wants zero ops overhead."*

**Hands-on lab (do this if you haven't):** build a tiny semantic search engine — load a handful of docs → chunk → embed → store in Chroma/FAISS → query. Even 30 minutes of doing this makes your RAG answers noticeably more concrete in interviews.

---

### B4. Fine-Tuning — LoRA, QLoRA, PEFT, RLHF

| Technique | What it does | Why it matters |
|---|---|---|
| **Full fine-tuning** | Update all model weights | Expensive, needs huge data/compute — rarely justified for enterprise use cases |
| **PEFT** (Parameter-Efficient Fine-Tuning) | Umbrella term: only update a small subset/adapter of parameters | The practical default — cheap, fast, avoids catastrophic forgetting |
| **LoRA** (Low-Rank Adaptation) | Freezes base model, injects small trainable low-rank matrices into attention layers | Most common PEFT method — small file size, easy to swap "adapters" per task |
| **QLoRA** | LoRA + 4-bit quantization of the base model | Lets you fine-tune large models on a single consumer/enterprise GPU |
| **RLHF** (Reinforcement Learning from Human Feedback) | Train a reward model from human preference rankings, then use RL (PPO) to align the LLM to it | How base models become "helpful, harmless" chat models (what OpenAI/Anthropic do at the foundation-model layer — not something you'd typically do at the application layer) |

**The single most important architect-level point:** know **when fine-tuning beats RAG/prompting** — and the honest answer is: rarely, first.

- Use **prompting** when the task is about instruction-following / format.
- Use **RAG** when the task needs up-to-date or proprietary *knowledge* the model wasn't trained on.
- Use **fine-tuning** when the task needs a new *skill/style/behavior* (a specific tone, a structured output format learned from thousands of examples, domain-specific jargon) that prompting/RAG can't reliably produce.
- In practice: **try prompting → then RAG → then fine-tuning**, in that order of cost/complexity.

---

### B5. Model Evaluation

- **Hallucination detection**: does the answer contain claims not supported by the retrieved context?
- **Groundedness**: can every claim in the output be traced back to a source chunk? (Directly relevant to PayerIQ — a generated STTM/FRD *must* be grounded in the actual source documents, not invented.)
- **Evaluation harnesses**: automated test sets of (question, expected-answer-properties) pairs run against the pipeline on every change.
- **LLM-as-judge**: use a strong LLM to score another LLM's output against a rubric (relevance, correctness, tone) — cheap, scalable, but has known biases (favors verbose answers, its own family's style) worth naming if asked.

---

## PART C — Agentic AI (Tier 2 — heavily weighted across every JD)

### C1. AI Agents vs. Agentic AI

- **Single tool-calling agent**: LLM decides to call *one* tool based on the user's request, gets a result, responds. Essentially a smarter API router.
- **Agentic AI**: multi-step **autonomous planning and reasoning loops** — the system breaks a goal into subtasks, decides which tools/agents to invoke in what order, evaluates intermediate results, and adapts the plan — with much less human intervention per step.

**Simple interview line:** *"A tool-calling agent answers one question and picks one tool. Agentic AI pursues a goal — it plans, acts, observes the result, and re-plans, potentially across many steps and multiple tools or sub-agents, before returning to the human."*

### C2. Tool Use / Function Calling

- The LLM is given a **schema** (name, description, parameters — usually JSON Schema) for each available tool.
- Based on the user's request, the model decides *whether* to call a tool, *which* one, and *with what arguments* — it emits a structured call rather than free text.
- The orchestrating application executes the actual function/API call and feeds the result back into the conversation for the model to use in its next step.
- **Good tool schema design**: clear, unambiguous names and descriptions (the model chooses tools based on the description text — vague descriptions cause wrong tool selection), tightly typed parameters, and idempotent/safe operations wherever possible.

### C3. MCP — Model Context Protocol (you have real hands-on experience — lead with this)

- MCP standardizes **how an LLM application discovers and calls external tools/data sources** — think of it as "a universal adapter" between models and tools, instead of every app writing bespoke integration code per tool per model provider.
- **MCP vs. plain API integration**: a hand-rolled integration hardcodes one tool to one model's function-calling format. MCP defines a standard client/server protocol — an MCP *server* exposes tools/resources, and any MCP-*compliant* client (any model that supports it) can discover and call them without custom glue code per model.
- **MCP vs. OpenAI/Anthropic native tool-calling**: native function-calling is the *mechanism* a model uses to invoke a tool in a single conversation; MCP is the *transport/discovery layer* that can sit underneath that mechanism, standardizing how the tool is described, authenticated, and reached — so the same MCP server can serve multiple model providers' agents.

**Your answer:** *"On PayerIQ I built an MCP server/client pair so the FastAPI backend could expose its document-processing and template-filling capabilities as MCP tools, rather than hardcoding a single integration. That meant the same tool surface could be called by different agent front-ends without re-writing the integration layer each time."* (Adjust to your actual implementation, but this is the kind of framing interviewers want — a *why*, not just a *what*.)

### C4. Agentic AI Frameworks (know 2+ well)

| Framework | Maker/origin | Strength | Position it as |
|---|---|---|---|
| **LangChain / LangGraph** | LangChain | LangChain = building blocks (prompts, chains, retrievers); LangGraph = explicit state-machine/graph orchestration for multi-step agents | Most widely adopted, huge ecosystem/integrations |
| **CrewAI** | CrewAI | Role-based multi-agent orchestration (define agents as "roles" with goals) | Fast to prototype role-based multi-agent workflows |
| **AutoGen** | Microsoft | Multi-agent conversation framework, strong Azure/enterprise alignment | Natural fit if the org is Microsoft-aligned |
| **Semantic Kernel** | Microsoft | Enterprise-grade SDK (C#/.NET/Python) for orchestrating LLMs + "plugins" (their term for tools) with planning | Best answer for Microsoft-track interviews — pairs naturally with Copilot Studio/Azure AI Foundry |
| **LlamaIndex** | LlamaIndex | Started as a data-framework specializing in indexing/retrieval, now has agent workflows too | Strong when RAG-heavy indexing is the core need |
| **OpenAI Agents SDK** | OpenAI | Lightweight, native to OpenAI's tool-calling/handoff model | Simple hand-off patterns between agents |
| **Anthropic Claude (Agent) SDK** | Anthropic | Native MCP support, strong for building tool-using Claude agents | Pairs directly with your MCP experience |

**Pick-2 recommendation for your Microsoft-aligned target roles:** **LangGraph** (industry-standard mental model for graph-based agent orchestration) + **Semantic Kernel** (directly relevant to the Microsoft ecosystem tier of your target JDs).

### C5. Multi-Agent System Design

- **Orchestrator pattern**: one "manager" agent decomposes the goal and delegates subtasks to specialist agents, then synthesizes their outputs.
- **Agent-to-agent handoff**: one agent recognizes a request is outside its scope and explicitly passes control (with context) to a better-suited agent.
- **Shared memory/state**: how agents in a crew share intermediate results — a shared scratchpad/blackboard, a shared vector store, or explicit message-passing.
- **Failure handling**: what happens when a sub-agent errors, times out, or returns a low-confidence result — retry, escalate to a human, or fall back to a simpler deterministic path. (This is a favorite senior-level question: *"what happens when your agent is wrong?"* — always have an answer beyond "it works.")

### C6. Agent Memory & Planning

- **Short-term memory**: the current conversation/task context window.
- **Long-term memory**: persisted facts/preferences/history retrieved across sessions (often itself a RAG pipeline over past interactions).
- **ReAct pattern** (Reason + Act): the agent alternates between a reasoning step ("thought") and an action step (tool call), observing the result before the next thought — the foundational loop underneath most modern agents.

### C7. Workflow Automation Tools

- **n8n**: open-source, self-hostable visual workflow automation, increasingly used to orchestrate LLM/agent steps alongside traditional integrations.
- **Power Automate**: Microsoft's equivalent, deeply integrated with M365/Power Platform — relevant to your Microsoft-ecosystem tier (pairs with Copilot Studio, AI Builder).

---

## PART D — LLMOps & Governance

### D1. LLMOps vs. classic MLOps
Classic MLOps = model lifecycle (train → deploy → monitor → retrain on drift). **LLMOps** adds concerns specific to prompt-and-API-driven systems: **prompt versioning** (treat prompts like code — version-controlled, tested), **evaluation pipelines** (automated regression tests for prompt/RAG changes), and **cost/latency monitoring** (token spend and response time per request, since you're often billed per token against a third-party API rather than owning fixed training compute).

### D2. AI Governance & Responsible AI
- **Frameworks to name as reference points**: NIST AI Risk Management Framework (RMF), ISO 42001 (AI management systems).
- **Responsible AI pillars**: bias/fairness (does the model perform equitably across groups), explainability (can you justify an output to a regulator/auditor), hallucination mitigation (grounding, citations, confidence thresholds, human-in-the-loop for high-stakes outputs).
- **Security layers for agentic systems**: IAM/OIDC for tool access control, TLS everywhere, output guardrails, and **prompt injection defense** (treating any retrieved/external content as untrusted input that could try to hijack the agent's instructions — a genuinely hot topic right now, worth having an opinion on).
- **Data privacy for AI**: PII handling in the pipeline — directly relevant to PayerIQ's document-processing work (healthcare payer documents almost certainly touch PII/PHI-adjacent data — be ready to talk about how you handle that, e.g., redaction, access controls, data residency).

---

## PART E — Turning This Into Interview Stories

Your study plan is right that **PayerIQ is your strongest single asset** — it's real, hands-on evidence spanning Tier 1 (RAG), Tier 2 (MCP/agentic), Tier 3 (Azure AI Search/OpenAI), and Tier 7 (document intelligence/BPO-style automation). Don't describe it as one big project — break it into **2–3 STAR-format stories**, each anchored to a different tier:

1. **RAG/architecture story** — the ingestion → chunking → embedding → Azure AI Search → generation pipeline you built for STTM/FRD generation. Situation (BAs spend hours drafting requirements docs), Task (automate grounded document generation), Action (RAG design decisions — why Azure AI Search, why hybrid search, chunking strategy), Result (time saved, accuracy/groundedness achieved).
2. **Agentic/MCP story** — why you built an MCP server/client instead of point-to-point API integration, and what that bought you (reusability across agent front-ends, cleaner separation of concerns).
3. **Governance/data-handling story** — how you handled PII in payer documents, or a security/access-control decision — ties to Tier 6 and shows you think like an *architect*, not just an implementer.

**For a workshop**, this same document doubles as a 5-day curriculum outline:
- **Day 1**: Tier 1 fundamentals + build the semantic search lab.
- **Day 2**: Prompting deep-dive + fine-tuning concepts (LoRA/QLoRA/RLHF) — conceptual, not hands-on training.
- **Day 3**: Agentic AI — ReAct, tool calling, pick 2 frameworks, build a simple agent.
- **Day 4**: MCP deep-dive + multi-agent orchestration lab.
- **Day 5**: Governance/Responsible AI + present participants' own architecture using this framework.

---

## PART F — Active Community Presence

JDs increasingly list this explicitly. It doesn't need to be heavy — consistency beats volume:
- **LinkedIn**: share a short post whenever you resolve a real design decision (e.g., "why I chose hybrid search over pure vector search for a payer-document RAG system") — this doubles as interview material.
- **GitHub**: even a clean, well-documented repo of your PayerIQ MCP server/client (sanitized of any client-confidential specifics) is strong signal — architects are expected to *show*, not just describe.
- **Hugging Face**: not mandatory for an architect role, but starring/following spaces relevant to your stack (embedding models, evaluation tooling) and occasionally commenting shows engagement without requiring you to publish your own models.
