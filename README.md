# Cloud Run GPU Hackathon AI Agent Sample Repo: Deploy Your ADK Agent to Cloud Run with GPU

 In this sample repo, you'll complete the prototype-to-production journey by taking a working ADK agent and deploying it as a scalable, robust application on Google Cloud Run with GPU support.


## 🏗️ What You'll Build

You'll deploy a **Production Gemma3 Agent** with conversational capabilities:

**Gemma Agent** (GPU-Accelerated):

- General conversations and Q&A
- Creative writing assistance
- Production-ready deployment on Cloud Run

## 📋 Prerequisites

- Google Cloud Project with billing enabled. (Please follow the Hackathon handbook manual instruction to apply the credit coupon to your project for the hackathon - DO NOT use your personal credit card)
- Google Cloud SDK installed and configured
- Basic understanding of containers and cloud deployment

## 🚀 Lab Overview

### Part 1: Understanding the Production Agent (10 minutes)

Let's first explore the agent we'll be deploying:

#### Agent Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   User Request  │ -> │   ADK Agent     │ -> │  Gemma Backend  │
│                 │    │  (Cloud Run)    │    │ (Cloud Run+GPU) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              v
                       ┌─────────────────┐
                       │ FastAPI Server  │
                       │ Health Checks   │
                       │─────────────────┘
```

#### Key Components

## Prerequisites 

```bash
# Set your Google Cloud project
export PROJECT_ID="your-project-id"
gcloud config set project $PROJECT_ID
gcloud config set run/region europe-west1

# Enable APIs
gcloud services enable run.googleapis.com cloudbuild.googleapis.com aiplatform.googleapis.com
```

## Deploy Gemma Backend

```bash
cd hackathon-cloudrun/ollama-backend

gcloud run deploy ollama-gemma3-4b-gpu \
  --source . \
  --concurrency 4 \
  --cpu 8 \
  --set-env-vars OLLAMA_NUM_PARALLEL=4 \
  --gpu 1 \
  --gpu-type nvidia-l4 \
  --max-instances 1 \
  --memory 32Gi \
  --allow-unauthenticated \
  --no-cpu-throttling \
  --no-gpu-zonal-redundancy \
  --timeout=600


## download ollama utility and test the Cloud Run GPU service that is created
curl -fsSL https://ollama.com/install.sh
OLLAMA_HOST=<Cloud Run SERVICE URL generated above> ollama run gemma3:4b
```

## Deploy ADK Cloud Run Agent that calls the Gemma Backend

```bash
# go to the ADK agent directory
cd hackathon-cloudrun/adk-agent

export OLLAMA_URL=$(gcloud run services describe ollama-gemma3-4b-gpu \
  --region europe-west1 \
  --format='value(status.url)')

# Create environment file

cat > .env << EOF
GOOGLE_CLOUD_PROJECT=$PROJECT_ID
GOOGLE_CLOUD_LOCATION=europe-west1
GEMMA_MODEL_NAME=gemma3:4b
OLLAMA_API_BASE=$OLLAMA_URL
EOF

# Deploy the ADK based AI agent to Cloud Run with ADK webUI 

gcloud run deploy production-adk-agent \
    --source . \
    --region europe-west1 \
    --allow-unauthenticated \
    --memory 4Gi \
    --cpu 2 \
    --max-instances 1 \
    --concurrency 50 \
    --timeout 300 \
    --set-env-vars GOOGLE_CLOUD_PROJECT=$PROJECT_ID \
    --set-env-vars GOOGLE_CLOUD_LOCATION=europe-west1 \
    --set-env-vars GEMMA_MODEL_NAME=gemma3:4b \
    --set-env-vars OLLAMA_API_BASE=$OLLAMA_URL
```

## Test Your Agent's health

```bash
# Get service URL
export SERVICE_URL=$(gcloud run services describe production-adk-agent \
    --region=europe-west1 \
    --format='value(status.url)')

# Test health endpoint
curl $SERVICE_URL/health

```

## 🎉 Test your Agent with the ADK WebUI

Your production ADK agent is now running on Cloud Run with GPU acceleration!

Interact with your agent by entering the SERVICE_URL above for your production-adk-agent into a new browser tab. You should see the ADK web interface.

**Try these queries:**

**Gemma Agent** (Conversational):

- "Tell me about artificial intelligence"
- "What are some creative writing tips?"
- "Explain quantum computing in simple terms"
- "Can you help me brainstorm ideas for a blog post?"
