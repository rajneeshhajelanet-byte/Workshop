
UI (Banking/Insurance App)
       ↓
Chatbot API (Docker container on AWS EKS)
       ↓
Kubernetes Service Account (bound to IAM Role via OIDC)
       ↓
IAM OIDC Federation
   ├─ Pod requests token from OIDC provider
   ├─ Token exchanged for AWS STS temporary credentials
   └─ Credentials scoped to IAM Role permissions
       ↓
Orchestrator (Bedrock + Lambda inside EKS)
   ├─ Guardrails (LLM safety filters)
   ├─ RAG retrieval (Bedrock embeddings + OpenSearch/pgvector)
   └─ Routes to SageMaker endpoints
       ↓
SageMaker Endpoints (Fraud, Risk, Compliance, Claims APIs)
   ├─ Each endpoint = secure API
   ├─ Only callable by IAM role attached to pod
       ↓
S3 / DynamoDB / Audit Manager
   ├─ Store embeddings, logs, documents
   ├─ Encrypted with KMS
   └─ Access controlled by IAM role
       ↓
Reply returned to UI
