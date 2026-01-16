# Healthcare AI Platform - Developer Setup Guide

**For New Developers** | **Complete POC → MVP Implementation**

**Version**: 1.0  
**Last Updated**: January 16, 2026  
**Estimated Setup Time**: 4-6 hours (POC), 2-3 days (MVP)

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Local Environment Setup](#local-environment-setup)
3. [POC Phase 0: Local Development](#poc-phase-0-local-development)
4. [Development Workflow](#development-workflow)
5. [Testing Your POC](#testing-your-poc)
6. [MVP Phase 1: GCP Migration](#mvp-phase-1-gcp-migration)
7. [Deployment & Monitoring](#deployment--monitoring)
8. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Accounts

| Service | Purpose | Cost | Sign Up |
|---------|---------|------|---------|
| **GitHub** | Code repository | Free | https://github.com/signup |
| **Anthropic** | Claude API | $5 credit free | https://console.anthropic.com |
| **Google AI** | Gemini API | Free tier | https://makersuite.google.com/app/apikey |
| **Google Cloud** (MVP only) | Infrastructure | $300 credit | https://cloud.google.com/free |

### Required Software

| Software | Purpose | Installation |
|----------|---------|--------------|
| **Git** | Version control | https://git-scm.com/downloads |
| **Docker Desktop** | Run containers | https://www.docker.com/products/docker-desktop |
| **VS Code** | Code editor | https://code.visualstudio.com/ |
| **Node.js 18+** | JavaScript runtime | https://nodejs.org/ |
| **Python 3.11+** | Backend language | https://www.python.org/downloads/ |

---

## Local Environment Setup

### Step 1: Install Homebrew (macOS)

```bash
# Open Terminal and run:
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Verify installation
brew --version
```

### Step 2: Install Git

```bash
# Install Git
brew install git

# Configure Git (replace with your info)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Verify
git --version
```

### Step 3: Install Docker Desktop

1. Download from: https://www.docker.com/products/docker-desktop
2. Open the downloaded `.dmg` file
3. Drag Docker to Applications folder
4. Open Docker Desktop
5. Wait for Docker to start (whale icon in menu bar)
6. Verify:

```bash
docker --version
docker-compose --version
```

**Expected output**:
```
Docker version 24.0.x
Docker Compose version v2.x.x
```

### Step 4: Install Node.js

```bash
# Install Node.js via Homebrew
brew install node@18

# Verify installation
node --version  # Should show v18.x.x
npm --version   # Should show v9.x.x or higher
```

### Step 5: Install Python

```bash
# Install Python 3.11
brew install python@3.11

# Create alias (add to ~/.zshrc or ~/.bash_profile)
echo 'alias python=python3.11' >> ~/.zshrc
echo 'alias pip=pip3.11' >> ~/.zshrc
source ~/.zshrc

# Verify
python --version  # Should show Python 3.11.x
pip --version
```

### Step 6: Install VS Code

1. Download from: https://code.visualstudio.com/
2. Move to Applications folder
3. Open VS Code
4. Install recommended extensions:
   - Python (Microsoft)
   - Docker (Microsoft)
   - Prettier (Code formatter)
   - ES7+ React/Redux/React-Native snippets

**To install extensions**:
```bash
# From command line
code --install-extension ms-python.python
code --install-extension ms-azuretools.vscode-docker
code --install-extension esbenp.prettier-vscode
code --install-extension dsznajder.es7-react-js-snippets
```

---

## POC Phase 0: Local Development

### Step 1: Create Project Structure

```bash
# Create main project directory
cd ~/Documents/Projects
mkdir -p HealthcareAI
cd HealthcareAI

# Create directory structure
mkdir -p backend/app
mkdir -p frontend
mkdir -p infrastructure/{persistent,ephemeral}
mkdir -p monitoring
mkdir -p scripts
mkdir -p data/synthetic

# Verify structure
tree -L 2
```

**Expected structure**:
```
HealthcareAI/
├── backend/
│   └── app/
├── frontend/
├── infrastructure/
│   ├── persistent/
│   └── ephemeral/
├── monitoring/
├── scripts/
└── data/
    └── synthetic/
```

### Step 2: Initialize Git Repository

```bash
cd ~/Documents/Projects/HealthcareAI

# Initialize Git
git init

# Create .gitignore
cat > .gitignore << 'EOF'
# Environment variables
.env
.env.local

# Python
__pycache__/
*.py[cod]
*$py.class
venv/
.pytest_cache/

# Node
node_modules/
.next/
out/
build/

# Docker
.docker/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Data
data/real/
*.db
*.sqlite
EOF

# Create README
cat > README.md << 'EOF'
# Healthcare AI Platform

## Quick Start

```bash
# Start POC environment
docker-compose up -d

# Access services:
# - Frontend: http://localhost:3000
# - API: http://localhost:8000/docs
# - FHIR: http://localhost:8080
```

See [DEVELOPER_SETUP_GUIDE.md](DEVELOPER_SETUP_GUIDE.md) for detailed setup.
EOF

# Initial commit
git add .
git commit -m "Initial project structure"
```

### Step 3: Get API Keys

#### Claude API Key

1. Go to: https://console.anthropic.com
2. Sign up / Log in
3. Navigate to "API Keys"
4. Click "Create Key"
5. Copy the key (starts with `sk-ant-`)

#### Gemini API Key

1. Go to: https://makersuite.google.com/app/apikey
2. Sign in with Google account
3. Click "Create API Key"
4. Copy the key

#### Store API Keys

```bash
cd ~/Documents/Projects/HealthcareAI

# Create .env file
cat > .env << 'EOF'
# API Keys (NEVER commit this file to Git)
CLAUDE_API_KEY=your-claude-key-here
GEMINI_API_KEY=your-gemini-key-here

# Database
DATABASE_URL=postgresql://postgres:password@postgres:5432/healthcare_ai
REDIS_URL=redis://redis:6379
FHIR_URL=http://hapi-fhir:8080/fhir
QDRANT_URL=http://qdrant:6333

# Application
ENVIRONMENT=development
LOG_LEVEL=INFO
EOF

# Edit .env and replace with your actual keys
code .env
```

**⚠️ IMPORTANT**: Never commit `.env` to Git! It's already in `.gitignore`.

### Step 4: Create Docker Compose Configuration

```bash
cd ~/Documents/Projects/HealthcareAI

# Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  # Backend API
  api:
    build: ./backend
    container_name: healthcare-ai-api
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/healthcare_ai
      - REDIS_URL=redis://redis:6379
      - FHIR_URL=http://hapi-fhir:8080/fhir
      - QDRANT_URL=http://qdrant:6333
      - CLAUDE_API_KEY=${CLAUDE_API_KEY}
      - GEMINI_API_KEY=${GEMINI_API_KEY}
      - ENVIRONMENT=development
    depends_on:
      - postgres
      - redis
      - hapi-fhir
      - qdrant
    volumes:
      - ./backend:/app
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
  
  # Frontend
  frontend:
    build: ./frontend
    container_name: healthcare-ai-frontend
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:8000
    depends_on:
      - api
    volumes:
      - ./frontend:/app
      - /app/node_modules
  
  # FHIR Server (replaces Healthcare API)
  hapi-fhir:
    image: hapiproject/hapi:latest
    container_name: healthcare-ai-fhir
    ports:
      - "8080:8080"
    environment:
      - hapi.fhir.fhir_version=R4
      - spring.datasource.url=jdbc:postgresql://postgres:5432/hapi
      - spring.datasource.username=postgres
      - spring.datasource.password=password
      - spring.jpa.hibernate.ddl-auto=update
    depends_on:
      - postgres
  
  # PostgreSQL (replaces Cloud SQL)
  postgres:
    image: postgres:15-alpine
    container_name: healthcare-ai-postgres
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=healthcare_ai
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
  
  # Redis (replaces Memorystore)
  redis:
    image: redis:7-alpine
    container_name: healthcare-ai-redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes
  
  # Vector Database (replaces Vertex AI Vector Search)
  qdrant:
    image: qdrant/qdrant:latest
    container_name: healthcare-ai-qdrant
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant-data:/qdrant/storage
  
  # Message Queue
  rabbitmq:
    image: rabbitmq:3-management
    container_name: healthcare-ai-rabbitmq
    ports:
      - "5672:5672"   # AMQP
      - "15672:15672" # Management UI
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=password
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
  
  # Monitoring - Prometheus
  prometheus:
    image: prom/prometheus:latest
    container_name: healthcare-ai-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
  
  # Monitoring - Grafana
  grafana:
    image: grafana/grafana:latest
    container_name: healthcare-ai-grafana
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_SECURITY_ADMIN_USER=admin
    volumes:
      - grafana-data:/var/lib/grafana
      - ./monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards
    depends_on:
      - prometheus

volumes:
  postgres-data:
  redis-data:
  qdrant-data:
  rabbitmq-data:
  prometheus-data:
  grafana-data:
EOF

# Create monitoring config
mkdir -p monitoring/grafana/dashboards
cat > monitoring/prometheus.yml << 'EOF'
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'healthcare-ai-api'
    static_configs:
      - targets: ['api:8000']
EOF
```

### Step 5: Create Backend Application

```bash
cd ~/Documents/Projects/HealthcareAI/backend

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
EOF

# Create requirements.txt
cat > requirements.txt << 'EOF'
# Web Framework
fastapi==0.109.0
uvicorn[standard]==0.27.0
python-multipart==0.0.6

# Database
psycopg2-binary==2.9.9
sqlalchemy==2.0.25
alembic==1.13.1

# Redis
redis==5.0.1

# FHIR Client
fhirclient==4.1.0

# Vector DB
qdrant-client==1.7.0

# AI/ML
anthropic==0.18.1
google-generativeai==0.3.2
langgraph==0.0.20
langchain==0.1.0

# HTTP Client
httpx==0.26.0
requests==2.31.0

# Environment
python-dotenv==1.0.0

# Utilities
pydantic==2.5.3
pydantic-settings==2.1.0
EOF

# Create app structure
mkdir -p app/{api,models,services,utils}
touch app/__init__.py
touch app/api/__init__.py
touch app/models/__init__.py
touch app/services/__init__.py
touch app/utils/__init__.py

# Create main application
cat > app/main.py << 'EOF'
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
import os

app = FastAPI(
    title="Healthcare AI Platform",
    description="POC API for Clinical Documentation and Chart Review",
    version="0.1.0"
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
def read_root():
    return {
        "message": "Healthcare AI Platform API",
        "version": "0.1.0",
        "environment": os.getenv("ENVIRONMENT", "development")
    }

@app.get("/health")
def health_check():
    return {
        "status": "healthy",
        "database": "connected",  # TODO: actual DB check
        "redis": "connected",     # TODO: actual Redis check
        "fhir": "connected"       # TODO: actual FHIR check
    }
EOF

# Create database config
cat > app/database.py << 'EOF'
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://postgres:password@localhost:5432/healthcare_ai")

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
EOF
```

### Step 6: Create Frontend Application

```bash
cd ~/Documents/Projects/HealthcareAI/frontend

# Create Next.js app (this will take a few minutes)
npx create-next-app@latest . --typescript --tailwind --app --no-src-dir --import-alias "@/*"

# Answer prompts:
# ✔ Would you like to use TypeScript? … Yes
# ✔ Would you like to use ESLint? … Yes
# ✔ Would you like to use Tailwind CSS? … Yes
# ✔ Would you like to use `src/` directory? … No
# ✔ Would you like to use App Router? … Yes
# ✔ Would you like to customize the default import alias? … No

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev"]
EOF

# Update app/page.tsx for Healthcare AI
cat > app/page.tsx << 'EOF'
export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-center p-24">
      <div className="z-10 max-w-5xl w-full items-center justify-center font-mono text-sm">
        <h1 className="text-4xl font-bold mb-8 text-center">
          Healthcare AI Platform
        </h1>
        <p className="text-center mb-8">
          POC Environment - Clinical Documentation & Chart Review
        </p>
        
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mt-8">
          <div className="p-6 border rounded-lg">
            <h2 className="text-2xl font-semibold mb-4">Documentation Copilot</h2>
            <p className="text-gray-600">Generate SOAP notes from brief clinical notes</p>
            <button className="mt-4 px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700">
              Create Note
            </button>
          </div>
          
          <div className="p-6 border rounded-lg">
            <h2 className="text-2xl font-semibold mb-4">Chart Review Agent</h2>
            <p className="text-gray-600">Automated quality and compliance checking</p>
            <button className="mt-4 px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700">
              Review Chart
            </button>
          </div>
        </div>
        
        <div className="mt-12 text-center text-sm text-gray-500">
          <p>API Status: <span className="text-green-500">Connected</span></p>
        </div>
      </div>
    </main>
  );
}
EOF
```

### Step 7: Start POC Environment

```bash
cd ~/Documents/Projects/HealthcareAI

# Start all services (this will take 5-10 minutes first time)
docker-compose up -d

# Watch the logs
docker-compose logs -f
```

**What's happening**:
1. Docker downloads all images (HAPI FHIR, PostgreSQL, Redis, etc.)
2. Builds your backend and frontend containers
3. Starts all services
4. Initializes databases

**Expected output**:
```
✔ Container healthcare-ai-postgres    Started
✔ Container healthcare-ai-redis       Started
✔ Container healthcare-ai-qdrant      Started
✔ Container healthcare-ai-rabbitmq    Started
✔ Container healthcare-ai-fhir        Started
✔ Container healthcare-ai-api         Started
✔ Container healthcare-ai-frontend    Started
✔ Container healthcare-ai-prometheus  Started
✔ Container healthcare-ai-grafana     Started
```

### Step 8: Verify Everything Works

```bash
# Check all containers are running
docker-compose ps

# Should show all services as "Up"

# Open in browser:
open http://localhost:3000      # Frontend
open http://localhost:8000/docs # API Documentation
open http://localhost:8080      # FHIR Server
open http://localhost:3001      # Grafana (admin/admin)
open http://localhost:15672     # RabbitMQ (admin/password)
open http://localhost:6333/dashboard # Qdrant

# Test API
curl http://localhost:8000/health
```

**Expected response**:
```json
{
  "status": "healthy",
  "database": "connected",
  "redis": "connected",
  "fhir": "connected"
}
```

---

## Development Workflow

### Daily Workflow

```bash
# Morning: Start services
cd ~/Documents/Projects/HealthcareAI
docker-compose up -d

# Develop all day
# - Edit code in ./backend or ./frontend
# - Changes auto-reload (hot reload enabled)
# - Test in browser

# View logs
docker-compose logs -f api      # Backend logs
docker-compose logs -f frontend # Frontend logs

# Evening: Stop services
docker-compose down

# Weekend: Stop and free up disk space
docker-compose down -v  # Removes volumes (fresh start Monday)
```

### Making Code Changes

```bash
# Backend changes (Python)
cd ~/Documents/Projects/HealthcareAI/backend
code app/main.py
# Save file → API auto-reloads in ~2 seconds

# Frontend changes (React/Next.js)
cd ~/Documents/Projects/HealthcareAI/frontend
code app/page.tsx
# Save file → Browser auto-refreshes

# No need to restart Docker containers!
```

### Database Access

```bash
# Connect to PostgreSQL
docker exec -it healthcare-ai-postgres psql -U postgres -d healthcare_ai

# Run SQL queries
SELECT * FROM pg_tables;

# Exit
\q

# View Redis data
docker exec -it healthcare-ai-redis redis-cli
KEYS *
EXIT
```

### Viewing Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f api
docker-compose logs -f frontend
docker-compose logs -f hapi-fhir

# Last 100 lines
docker-compose logs --tail=100 api
```

---

## Testing Your POC

### Step 1: Create Synthetic FHIR Data

```bash
cd ~/Documents/Projects/HealthcareAI

# Create script to load test patients
cat > scripts/load_test_data.sh << 'EOF'
#!/bin/bash

# Load test patient into HAPI FHIR
curl -X POST http://localhost:8080/fhir/Patient \
  -H "Content-Type: application/fhir+json" \
  -d '{
  "resourceType": "Patient",
  "identifier": [{
    "system": "urn:oid:1.2.840.114350",
    "value": "TEST001"
  }],
  "name": [{
    "family": "Doe",
    "given": ["John"]
  }],
  "gender": "male",
  "birthDate": "1980-01-01"
}'

echo "\nTest patient loaded!"
EOF

chmod +x scripts/load_test_data.sh
./scripts/load_test_data.sh
```

### Step 2: Test Backend API

```bash
# Test health endpoint
curl http://localhost:8000/health | jq

# Test with Python
python3 << 'EOF'
import requests

# Health check
response = requests.get("http://localhost:8000/health")
print(response.json())

# TODO: Add more endpoints as you build them
EOF
```

### Step 3: Run Automated Tests

```bash
cd ~/Documents/Projects/HealthcareAI/backend

# Create pytest configuration
cat > pyproject.toml << 'EOF'
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
EOF

# Create test directory
mkdir -p tests
touch tests/__init__.py

# Create sample test
cat > tests/test_main.py << 'EOF'
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_read_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json()["message"] == "Healthcare AI Platform API"

def test_health_check():
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json()["status"] == "healthy"
EOF

# Run tests
pip install pytest
pytest -v
```

---

## MVP Phase 1: GCP Migration

### Prerequisites for MVP

1. **GCP Account with Billing Enabled**
2. **$300 Free Credit** (new accounts)
3. **API Keys** ready
4. **POC Working** and tested

### Step 1: Install GCP CLI

```bash
# Install gcloud CLI
brew install --cask google-cloud-sdk

# Initialize gcloud
gcloud init

# Follow prompts:
# - Sign in to your Google account
# - Select or create a project
# - Set default region: us-central1

# Verify
gcloud --version
gcloud config list
```

### Step 2: Install OpenTofu

```bash
# Install OpenTofu
brew install opentofu

# Verify
tofu --version
```

### Step 3: Enable Required GCP APIs

```bash
# Set your project ID
export PROJECT_ID="your-project-id"
gcloud config set project $PROJECT_ID

# Enable APIs
gcloud services enable \
  cloudbuild.googleapis.com \
  run.googleapis.com \
  sqladmin.googleapis.com \
  healthcare.googleapis.com \
  redis.googleapis.com \
  aiplatform.googleapis.com \
  secretmanager.googleapis.com \
  cloudkms.googleapis.com \
  dlp.googleapis.com \
  compute.googleapis.com

# This takes 2-3 minutes
```

### Step 4: Create Service Account

```bash
# Create service account
gcloud iam service-accounts create healthcare-ai-deployer \
  --display-name="Healthcare AI Deployer"

# Grant permissions
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:healthcare-ai-deployer@${PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/editor"

# Download key
gcloud iam service-accounts keys create ~/healthcare-ai-key.json \
  --iam-account=healthcare-ai-deployer@${PROJECT_ID}.iam.gserviceaccount.com

# Set environment variable
export GOOGLE_APPLICATION_CREDENTIALS=~/healthcare-ai-key.json
```

### Step 5: Export POC Data

```bash
cd ~/Documents/Projects/HealthcareAI

# Create export directory
mkdir -p data/exports

# Export PostgreSQL
docker exec healthcare-ai-postgres pg_dump -U postgres healthcare_ai > data/exports/poc_data.sql

# Export Qdrant vectors
curl http://localhost:6333/collections/medical_knowledge/points/scroll > data/exports/qdrant_vectors.json

# Export FHIR data
curl http://localhost:8080/fhir/Patient?_format=json > data/exports/fhir_patients.json

echo "✅ POC data exported to data/exports/"
```

### Step 6: Create GCP Infrastructure with OpenTofu

```bash
cd ~/Documents/Projects/HealthcareAI/infrastructure/persistent

# Create main.tf
cat > main.tf << 'EOF'
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
  
  backend "gcs" {
    bucket = "healthcare-ai-tfstate"
    prefix = "persistent"
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

# Cloud Storage for OpenTofu state
resource "google_storage_bucket" "tfstate" {
  name          = "healthcare-ai-tfstate"
  location      = var.region
  force_destroy = false
  
  versioning {
    enabled = true
  }
}

# Artifact Registry
resource "google_artifact_registry_repository" "docker_repo" {
  location      = var.region
  repository_id = "healthcare-ai"
  format        = "DOCKER"
}

# Secret Manager
resource "google_secret_manager_secret" "claude_api_key" {
  secret_id = "claude-api-key"
  
  replication {
    auto {}
  }
}

resource "google_secret_manager_secret" "gemini_api_key" {
  secret_id = "gemini-api-key"
  
  replication {
    auto {}
  }
}
EOF

# Create variables.tf
cat > variables.tf << 'EOF'
variable "project_id" {
  description = "GCP Project ID"
  type        = string
}

variable "region" {
  description = "GCP Region"
  type        = string
  default     = "us-central1"
}
EOF

# Initialize and apply
tofu init
tofu plan
tofu apply

# Upload secrets
echo -n "$CLAUDE_API_KEY" | gcloud secrets versions add claude-api-key --data-file=-
echo -n "$GEMINI_API_KEY" | gcloud secrets versions add gemini-api-key --data-file=-
```

### Step 7: Deploy to Cloud Run

```bash
cd ~/Documents/Projects/HealthcareAI

# Build and push backend
docker build -t us-central1-docker.pkg.dev/${PROJECT_ID}/healthcare-ai/api:latest ./backend
docker push us-central1-docker.pkg.dev/${PROJECT_ID}/healthcare-ai/api:latest

# Deploy to Cloud Run
gcloud run deploy healthcare-ai-api \
  --image us-central1-docker.pkg.dev/${PROJECT_ID}/healthcare-ai/api:latest \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars ENVIRONMENT=production \
  --set-secrets CLAUDE_API_KEY=claude-api-key:latest,GEMINI_API_KEY=gemini-api-key:latest

# Get URL
gcloud run services describe healthcare-ai-api --region us-central1 --format='value(status.url)'
```

---

## Deployment & Monitoring

### Access Your Services

**POC (Local)**:
```
Frontend:    http://localhost:3000
API:         http://localhost:8000
Grafana:     http://localhost:3001
```

**MVP (GCP)**:
```
Frontend:    https://app-[hash]-uc.a.run.app
API:         https://api-[hash]-uc.a.run.app
Console:     https://console.cloud.google.com
```

### Monitoring

```bash
# View Cloud Run logs
gcloud run services logs read healthcare-ai-api --region us-central1 --limit 100

# Monitor costs
gcloud billing accounts list
gcloud billing budgets list

# Check service health
curl https://api-[your-hash]-uc.a.run.app/health
```

---

## Troubleshooting

### Docker Issues

**Problem**: Docker containers won't start
```bash
# Check Docker is running
docker --version

# Restart Docker Desktop
# Click Docker icon → Restart

# Check logs
docker-compose logs
```

**Problem**: Port already in use
```bash
# Find what's using port 8000
lsof -i :8000

# Kill the process
kill -9 <PID>

# Or change port in docker-compose.yml
```

**Problem**: Out of disk space
```bash
# Clean up Docker
docker system prune -a --volumes

# Remove unused images
docker image prune -a
```

### Database Issues

**Problem**: PostgreSQL won't connect
```bash
# Check if container is running
docker ps | grep postgres

# Check logs
docker-compose logs postgres

# Restart database
docker-compose restart postgres
```

**Problem**: Database data corrupted
```bash
# Stop services
docker-compose down

# Remove volume
docker volume rm healthcareai_postgres-data

# Start fresh
docker-compose up -d
```

### API Issues

**Problem**: API returns 500 error
```bash
# Check API logs
docker-compose logs -f api

# Restart API
docker-compose restart api

# Check environment variables
docker exec healthcare-ai-api env | grep API_KEY
```

### Need Help?

1. **Check logs first**: `docker-compose logs -f`
2. **Search error message** on Stack Overflow
3. **GitHub Issues**: Create issue in your repo
4. **Documentation**: Re-read relevant sections

---

## Next Steps

### After POC Works

1. ✅ Validate with 5-10 clinicians
2. ✅ Measure LLM token usage
3. ✅ Document any issues
4. ✅ **Decision**: Proceed to MVP?

### Before MVP Migration

- [ ] POC validated successfully
- [ ] GCP account set up with billing
- [ ] API keys ready
- [ ] Team trained on GCP
- [ ] Budget approved ($400-600/month)

### MVP Checklist

- [ ] Infrastructure provisioned with OpenTofu
- [ ] Data migrated from POC
- [ ] Services deployed to Cloud Run
- [ ] HIPAA compliance review
- [ ] Security audit
- [ ] Monitoring configured
- [ ] Pilot with 20-50 users

---

**Congratulations!** You now have a complete guide to build and deploy the Healthcare AI Platform from POC to MVP!

## Quick Reference

```bash
# POC Commands
docker-compose up -d              # Start
docker-compose down               # Stop
docker-compose logs -f api        # View logs
docker-compose restart api        # Restart service

# MVP Commands
gcloud run deploy                 # Deploy to Cloud Run
gcloud run services list          # List services
gcloud run services logs read     # View logs
tofu apply                        # Update infrastructure
```

---

**Document Version**: 1.0  
**Last Updated**: January 16, 2026  
**Maintained By**: Development Team
