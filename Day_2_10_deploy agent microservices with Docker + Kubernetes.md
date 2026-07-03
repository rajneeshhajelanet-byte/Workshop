markdown
# 🏦 Day 2.10 – Deploy Agent Microservices with Docker + Kubernetes

## 1. High-Level Architecture
```text
UI (Banking/Insurance App)
       ↓
Chatbot API (FastAPI/Uvicorn in Docker)
       ↓
Orchestrator (Bedrock + Lambda / AKS Functions)
       ↓
AI Agents (SageMaker Endpoints)
       ↓
Memory Layer (pgvector / OpenSearch / S3 / DynamoDB)
       ↓
Compliance & Audit (IAM OIDC, CloudWatch, Audit Manager)
2. Security Layers
🔹 Docker + AKS
Package Chatbot API in Docker.

Deploy to Azure Kubernetes Service (AKS) or Amazon EKS.

Use network policies + service mesh (Istio/Linkerd) for secure communication.

TLS termination at ingress controller.

🔹 IAM OIDC
Use IAM roles with OIDC federation for API pods in AKS/EKS.

Each microservice gets a scoped IAM role (least privilege).

Example: Chatbot API can only invoke-endpoint on SageMaker, not access S3 directly.

🔹 S3 Buckets
Store embeddings, logs, and documents securely.

Enable bucket policies + KMS encryption.

Access controlled via IAM roles, not static keys.

🔹 Service Lifecycle Management
Automate deployment, scaling, and patching of APIs.

Use CI/CD pipelines with security scans.

Rotate secrets automatically.

🔹 Payload Security
All payloads (UI → Chatbot → SageMaker) go over HTTPS/TLS.

Use JWT tokens or Cognito/OAuth2 for user authentication.

Validate payload schema (Pydantic in FastAPI).

Guardrail APIs filter unsafe inputs before hitting Bedrock/SageMaker.

3. Secure Orchestration Call Example
python
import boto3
import json

# IAM OIDC role ensures this pod can only call SageMaker
sagemaker = boto3.client("sagemaker-runtime", region_name="us-east-1")

def call_sagemaker_secure(endpoint_name, payload):
    response = sagemaker.invoke_endpoint(
        EndpointName=endpoint_name,
        ContentType="application/json",
        Body=json.dumps(payload)
    )
    return json.loads(response['Body'].read().decode())
4. AWS EKS with IAM OIDC Federation
🏗️ How It Works
Service Pod in EKS

Each microservice (e.g., Chatbot API) runs inside a Kubernetes pod.

Pod is associated with a Kubernetes Service Account.

IAM OIDC Federation

EKS cluster is linked to AWS IAM via OIDC provider.

Service account annotated with IAM role ARN.

Pod receives temporary AWS credentials via STS.

Auth Flow

Pod retrieves JWT token from OIDC provider.

Token exchanged for AWS temporary credentials.

Credentials scoped to IAM Role permissions.

Payload Flow

UI → Chatbot API (Docker/EKS).

Chatbot API attaches auth token when calling SageMaker/S3.

AWS validates token via IAM OIDC → grants access only if role policy allows.

5. Security Benefits
No static AWS keys in code/config.

Least privilege → each service account maps to a specific IAM role.

Automatic rotation → STS temporary credentials expire and refresh.

Auditability → CloudTrail logs every API call with IAM role identity.

6. Example Pod → SageMaker Secure Call
Kubernetes Service Account with IAM Role:

yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: chatbot-api-sa
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/ChatbotApiRole
Inside Chatbot API pod (Python):

python
import boto3, json

# IAM OIDC automatically provides temporary creds
sagemaker = boto3.client("sagemaker-runtime", region_name="us-east-1")

def call_sagemaker(endpoint, payload):
    response = sagemaker.invoke_endpoint(
        EndpointName=endpoint,
        ContentType="application/json",
        Body=json.dumps(payload)
    )
    return json.loads(response['Body'].read().decode())
7. End-to-End Flow
text
UI (Banking/Insurance App)
       ↓
Chatbot API (Docker container on AWS EKS)
       ↓
Kubernetes Service Account → IAM Role via OIDC
       ↓
IAM OIDC Federation → STS temporary credentials
       ↓
Orchestrator (Bedrock + Lambda inside EKS)
       ↓
SageMaker Endpoints (Fraud, Risk, Compliance, Claims APIs)
       ↓
S3 / DynamoDB / Audit Manager (secure storage + compliance logging)
       ↓
Reply returned to UI
8. Key Takeaway
EKS pods authenticate via IAM OIDC → STS temporary credentials.

SageMaker endpoints = secure APIs (fraud, risk, compliance, claims).

S3/DynamoDB/Audit Manager → encrypted storage + compliance logging.

Bedrock + Lambda → orchestrates intent parsing, RAG retrieval, and routes to the right agent.

👉 This ensures your UI → Chatbot API → SageMaker APIs → S3/Audit flow is secure, compliant, and auditable.
