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



