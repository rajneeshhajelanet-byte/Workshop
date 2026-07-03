It’s structured with clear phases, code blocks, and commands so students can follow step‑by‑step.

markdown
# 🐳 Day 2.10 – Deploy Agent Microservices with Docker + Kubernetes

Deploying AI agents via Docker and Kubernetes provides the **isolation, scalability, and self‑healing infrastructure** required to run autonomous workflows, manage multi‑agent orchestration, and handle variable LLM token processing loads.

This guide shows how to package an AI agent and orchestrate it at scale.

---

## Phase 1: Containerizing the AI Agent with Docker

### 1. Define the Dockerfile
Create a Dockerfile optimized for lightweight execution and security.

```dockerfile
# Use an official lightweight Python runtime
FROM python:3.11-slim

# Set working directory inside the container
WORKDIR /app

# Install system dependencies if required by your agent tools
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Copy and install dependencies first to leverage Docker layer caching
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Expose the API port (e.g., if using FastAPI to expose agent endpoints)
EXPOSE 8000

# Run the agent application
CMD ["python", "main.py"]
2. Build and Push to a Registry
Build the image locally and push it to a secure repository like Docker Hub, Amazon ECR, or Google Artifact Registry.

bash
# Build the image
docker build -t your-registry/ai-agent:v1 .

# Push to your container registry
docker push your-registry/ai-agent:v1
Phase 2: Orchestrating the Agent on Kubernetes
Kubernetes handles the lifecycle of your agent, managing configuration secrets (like OpenAI, Anthropic, or Hugging Face API keys) and exposing endpoints.

1. Manage API Keys and Secrets
Never hardcode sensitive LLM API keys. Store them using a Kubernetes Secret.

yaml
apiVersion: v1
kind: Secret
metadata:
  name: ai-agent-secrets
type: Opaque
stringData:
  OPENAI_API_KEY: "sk-proj-yourActualKeyGoesHere"
  VECTOR_DB_URL: "https://your-vector-database.com"
2. Create the Agent Deployment Manifest
This manifest defines how many replicas of the agent should run and specifies resource boundaries to prevent memory leaks during long reasoning loops.

yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-agent-deployment
  labels:
    app: ai-agent
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ai-agent
  template:
    metadata:
      labels:
        app: ai-agent
    spec:
      containers:
      - name: ai-agent-container
        image: your-registry/ai-agent:v1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8000
        envFrom:
        - secretRef:
            name: ai-agent-secrets
        resources:
          limits:
            cpu: "2"
            memory: 4Gi
          requests:
            cpu: "500m"
            memory: 1Gi
3. Expose the Agent via a Service
Create a service to expose the agent internally to other apps or externally via a load balancer.

yaml
apiVersion: v1
kind: Service
metadata:
  name: ai-agent-service
spec:
  type: ClusterIP  # Use LoadBalancer if exposing directly to the internet
  ports:
  - port: 80
    targetPort: 8000
  selector:
    app: ai-agent
4. Apply Configuration to the Cluster
Execute these commands to spin up your cloud architecture:

bash
kubectl apply -f secrets.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
✅ Key Takeaway
Docker → packages the agent into a portable container.

Kubernetes → orchestrates replicas, secrets, and services.

Secrets → secure API keys and configs.

Services → expose agents internally or externally.



markdown
# 🏦 Day 2.10 – Deploy Agent Microservices with Docker + Kubernetes
https://www.google.com/search?q=ai+agents+deployemnt++vi+docker+and+kubernetes+&safe=active&sca_esv=d8df4427cda4b17e&rlz=1C1GCCB_en___IN1180&sxsrf=APpeQnsOoLXtqe8ClqckwayteSeDadPhEw%3A1783084005914&ei=5bNHasq7N6yRnesPm5qy2QQ&biw=1280&bih=585&ved=0ahUKEwiKwpvGybaVAxWsSGcHHRuNLEsQ4dUDCBA&uact=5&oq=ai+agents+deployemnt++vi+docker+and+kubernetes+&gs_lp=Egxnd3Mtd2l6LXNlcnAiL2FpIGFnZW50cyBkZXBsb3llbW50ICB2aSBkb2NrZXIgYW5kIGt1YmVybmV0ZXMgMgUQIRifBTIFECEYnwVI72FQ_AJY0k9wBXgBkAECmAHAA6AB5yKqAQkwLjkuOS4xLjG4AQPIAQD4AQH4AQKYAhagApMdwgIKEAAYRxjWBBiwA8ICBBAhGBXCAgcQIRgKGKABwgIFECEYoAHCAgQQIxgnwgILEAAYgAQYigUYkQLCAgUQABiABMICBhAAGAcYHsICChAhGAoYoAEYwwTCAgQQIRgKwgIIEAAYgAQYogSYAwCIBgGQBgiSBwk1LjguOC4wLjGgB_VfsgcJMC44LjguMC4xuAeEHcIHBjMuMTQuNcgHLIAIAQ&sclient=gws-wiz-serp

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
