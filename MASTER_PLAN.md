# Healthcare AI Platform - Master Plan

**Version**: 2.0 (Restructured for POC-First Approach)  
**Date**: January 14, 2026  
**Status**: POC Strategy Finalized - Ready for Development

---

## Executive Summary

### Vision
Build a HIPAA-compliant healthcare AI platform on Google Cloud Platform featuring **Copilots** (human-in-the-loop assistants) and **Agents** (autonomous automation) to reduce clinician burnout, improve diagnostic accuracy, and enhance patient care quality.

### Core Value Proposition
- **Save 10,000+ clinician hours** per month through AI-powered documentation and automated chart review
- **Reduce chart rejection rate by 40%** with AI-assisted quality assurance and compliance checking
- **Eliminate 70% of manual reporting time** with real-time analytics and AI-powered insights
- **Zero major safety incidents** through rigorous validation and human oversight

### Strategic Approach
- Start with **3 high-impact use cases** for MVP (Clinical Documentation, Chart Review Automation, Advanced Analytics)
- Deploy **incrementally** with clinical validation at each phase
- Build on **open-source technologies** where possible (OpenTofu, Docker, Python)
- Leverage **free public medical APIs** (RxNorm, PubMed, UMLS, FDA) to minimize costs
- Implement **hybrid knowledge strategy**: Vector database for custom content + public APIs for real-time data

---

## Development Strategy: POC-First Approach

### Why Start with a POC?

Building a HIPAA-compliant healthcare AI platform on GCP involves significant investment ($400-1,200/month for MVP). A **Proof of Concept (POC) validates technical feasibility and business value before committing resources**.

### Three-Phase Strategy

```
Phase 0: POC              Phase 1: Migration       Phase 2: MVP Pilot
(8 weeks)                 (8 weeks)                (12+ weeks)
┌─────────────┐          ┌──────────────┐         ┌──────────────┐
│ Local Docker│   ──→    │ GCP Managed  │   ──→   │ Production   │
│ Open Source │          │ Services     │         │ Scale        │
│             │          │              │         │              │
│ Cost: $0/mo │          │ $400-1,200/mo│         │ Scale up     │
│ 1-2 use cases│         │ Full features│         │ Multi-tenant │
└─────────────┘          └──────────────┘         └──────────────┘
```

### Phase 0: POC (8 Weeks, $0/month)

**Infrastructure**: 100% open source, local Docker Compose
- **HAPI FHIR** (replaces Healthcare API)
- **PostgreSQL** (replaces Cloud SQL)
- **Redis** (replaces Memorystore)
- **Qdrant** (replaces Vertex AI Vector Search)
- **Prometheus + Grafana** (monitoring)

**Scope**:
- Build 1-2 core use cases (Documentation, Chart Review)
- Validate LLM integration (Claude, Gemini)
- Test with synthetic FHIR data
- Demo to 5-10 clinicians
- Measure LLM token usage patterns

**Exit Criteria**:
- ✅ Technical feasibility proven
- ✅ Positive clinician feedback
- ✅ Cost model validated
- ✅ **Decision: Proceed to MVP or pivot**

**Cost**: **$0/month** (runs on laptop, only LLM API calls ~$20-50/month)

---

### Phase 1: Migration to MVP (8 Weeks)

**Infrastructure**: Migrate to managed GCP services
- **Healthcare API** (HIPAA-compliant FHIR)
- **Cloud SQL** (managed PostgreSQL)
- **Memorystore** (managed Redis)
- **Vertex AI Vector Search**
- **Cloud Run** (serverless containers)

**Scope**:
- Export data from POC (PostgreSQL, Qdrant, FHIR)
- Provision GCP infrastructure with OpenTofu
- Migrate application code to Cloud Run
- Implement HIPAA security controls
- Parallel testing (POC vs MVP)

**Migration Time**: 25-30 hours estimated

**Cost**: **$400-1,200/month** (with optimizations applied)

---

### Phase 2: MVP Pilot (12+ Weeks)

**Scope**:
- Pilot with 20-50 real clinicians
- Add UC-3 (Analytics Dashboard)
- Enhance security and compliance
- Implement SMART on FHIR for EHR integration
- Clinical validation and feedback
- Prepare for production scale

**Cost**: Scales based on usage, manage with cost optimization strategies

---

### Cost Trajectory Summary

| Phase | Duration | Infrastructure | Monthly Cost | Total Cost |
|-------|----------|----------------|--------------|------------|
| **POC** | 8 weeks | Local Docker Compose | **$0** | **$0** |
| **Migration** | 8 weeks | GCP (dev/staging) | $200-600 | $400-1,200 |
| **MVP Pilot** | 12 weeks | GCP (optimized) | $400-1,200 | $1,200-3,600 |
| **Production** | Ongoing | GCP (multi-region) | $1,500-4,000+ | Variable |

**Key Insight**: POC validates $50,000+ annual investment with $0 upfront infrastructure cost.

---

### Why This Approach Works

1. **De-risk Investment**: Validate before spending $5,000-15,000 on MVP
2. **Fast Learning**: 8 weeks to POC vs. 6 months to production-ready MVP
3. **Same Codebase**: Docker containers work locally and in Cloud Run
4. **Easy Migration**: Most components are drop-in replacements (Postgres→Cloud SQL, Redis→Memorystore)
5. **Cost Discovery**: Learn actual LLM usage before committing to managed services

---

## System Architecture

> **Note**: This diagram shows the **final MVP architecture** on GCP managed services (Phase 1+). The POC (Phase 0) starts much simpler with all services running locally via Docker Compose. See [POC Environment Setup](#poc-environment-setup) for the initial architecture.

### High-Level Architecture (MVP State)

```
┌─────────────────────────────────────────────────────────────────┐
│                       USER LAYER                                 │
│  Clinicians, Patients → Web Apps, Mobile Apps                   │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                    FRONTEND & API GATEWAY                        │
│  Cloud CDN → Load Balancer → Cloud Armor → API Gateway          │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│              APPLICATION LAYER (Cloud Run - Serverless)          │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────────┐  │
│  │   COPILOTS   │  │    AGENTS    │  │  BACKEND SERVICES   │  │
│  │ (Human-Loop) │  │ (Autonomous) │  │  (Auth, FHIR, Data) │  │
│  └──────────────┘  └──────────────┘  └─────────────────────┘  │
│           ↓                ↓                     ↓              │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │        ORCHESTRATION ENGINE (Google ADK/LangGraph)        │ │
│  └───────────────────────────────────────────────────────────┘ │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                      AI/ML LAYER                                 │
│                                                                  │
│  ┌──────────────────┐          ┌──────────────────────────┐    │
│  │   LLM GATEWAY    │          │   KNOWLEDGE LAYER        │    │
│  │                  │          │   (Hybrid Approach)      │    │
│  │  • Claude        │          │                          │    │
│  │  • Gemini        │          │  • Vector DB             │    │
│  │  • Model Router  │          │    (Custom Knowledge)    │    │
│  └──────────────────┘          │                          │    │
│                                │  • Public APIs (FREE):   │    │
│                                │    - RxNorm (Drugs)      │    │
│                                │    - PubMed (Literature) │    │
│                                │    - UMLS (Coding)       │    │
│                                │    - FDA (Warnings)      │    │
│                                └──────────────────────────┘    │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                       DATA LAYER                                 │
│                                                                  │
│  • Healthcare API (FHIR R4 Store) - PHI-compliant               │
│  • Cloud SQL (PostgreSQL) - Application data                    │
│  • Memorystore (Redis) - Cache + API responses                  │
│  • Cloud Storage - Documents, images                            │
│  • BigQuery - Analytics                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## Technology Stack

### Development Strategy: Open Source POC → Managed MVP

**Phase 1 (POC)**: 100% open source, local development, $0/month  
**Phase 2 (MVP)**: Migrate to managed GCP services, HIPAA-compliant, $400-1,200/month

---

### Technology Stack by Phase

| Layer | POC (Open Source) | MVP (Managed Services) | Migration Complexity |
|-------|-------------------|------------------------|----------------------|
| **Infrastructure** |
| IaC | OpenTofu | OpenTofu | ✅ Same |
| Container Runtime | Docker | Docker | ✅ Same |
| Hosting | Docker Compose (local) | **Cloud Run** | 🟡 Medium (containerized already) |
| Registry | Local | **Artifact Registry** | 🟢 Easy |
| **Backend** |
| Framework | FastAPI (Python 3.11+) | FastAPI | ✅ Same |
| Orchestration | LangGraph | LangGraph | ✅ Same |
| API Gateway | None (direct) | **Cloud Endpoints** | 🟢 Easy (add config) |
| **Frontend** |
| Framework | Next.js 14 | Next.js 14 | ✅ Same |
| Runtime | Node.js 18 | Node.js 18 | ✅ Same |
| CDN | None | **Cloud CDN** | 🟢 Easy |
| **AI/ML** |
| Primary LLM | Claude 3.5 Sonnet | Claude 3.5 Sonnet | ✅ Same |
| Secondary LLM | Gemini 1.5 Pro | Gemini 1.5 Pro | ✅ Same |
| Vector Database | **Qdrant** (open source) | **Vertex AI Vector Search** | 🟡 Medium (data migration) |
| Embeddings | text-embedding-3 | text-embedding-3 | ✅ Same |
| **Data Storage** |
| FHIR Store | **HAPI FHIR** (open source) | **Healthcare API** (FHIR R4) | 🟡 Medium (API compatible) |
| Database | **PostgreSQL** (Docker) | **Cloud SQL (PostgreSQL 15)** | 🟢 Easy (pg_dump/restore) |
| Cache | **Redis** (Docker) | **Memorystore (Redis 7)** | 🟢 Easy (Redis protocol) |
| Object Storage | Local volumes | **Cloud Storage** | 🟢 Easy (gsutil sync) |
| Analytics | **PostgreSQL** | **BigQuery** | 🟡 Medium (schema migration) |
| **Message Queue** |
| Async Processing | **RabbitMQ** (open source) | **RabbitMQ** (keep) | ✅ Same |
| **Public APIs (FREE)** |
| Drug Info | RxNorm API | RxNorm API | ✅ Same |
| Literature | PubMed E-utilities | PubMed E-utilities | ✅ Same |
| Coding | UMLS API | UMLS API | ✅ Same |
| Drug Warnings | FDA Drug Label API | FDA Drug Label API | ✅ Same |
| **Security** |
| Secrets | Environment variables | **Secret Manager** | 🟢 Easy (migrate secrets) |
| Encryption | N/A (dev only) | **Cloud KMS** | 🟢 Easy (new feature) |
| DLP | N/A | **Cloud DLP** | 🟢 Easy (new feature) |
| WAF | N/A | **Cloud Armor** | 🟢 Easy (enable) |
| **Observability** |
| Logging | Docker logs | **Cloud Logging** | 🟢 Easy (add agent) |
| Monitoring | **Prometheus + Grafana** | **Prometheus + Grafana** (keep) | ✅ Same |
| Tracing | **Jaeger** (optional) | **Cloud Trace** | 🟡 Medium (instrumentation) |
| **CI/CD** |
| Version Control | GitHub | GitHub | ✅ Same |
| Build Pipeline | Local builds | **Cloud Build** | 🟢 Easy (add config) |
| Testing | pytest + Jest | pytest + Jest | ✅ Same |

---

### Cost Comparison by Phase

| Phase | Infrastructure | Monthly Cost | Duration | Total Cost |
|-------|---------------|--------------|----------|------------|
| **POC (Open Source)** | Local Docker Compose | **$0** | 2 months | **$0** |
| **MVP (Managed - Baseline)** | GCP services 24/7 | $970-3,100 | 6 months | $5,820-18,600 |
| **MVP (Optimized)** | GCP + cost optimizations | $400-1,200 | 6 months | $2,400-7,200 |

**Key Cost Drivers (MVP)**: LLM tokens, Cloud Run compute, Cloud SQL

---

## POC Environment Setup

### Quick Start with Docker Compose

The POC runs entirely on your local machine using open source tools. No GCP account needed.

**Prerequisites**:
- Docker Desktop installed
- Claude and Gemini API keys (only costs: $20-50/month for testing)

**One-Command Setup**:
```bash
docker-compose up -d
```

**Services Included**:
| Service | Purpose | Replaces (in MVP) |
|---------|---------|-------------------|
| **HAPI FHIR** | FHIR R4 server | Healthcare API |
| **PostgreSQL** | Application database | Cloud SQL |
| **Redis** | Cache + sessions | Memorystore |
| **Qdrant** | Vector database | Vertex AI Vector Search |
| **RabbitMQ** | Message queue | Pub/Sub |
| **Prometheus + Grafana** | Monitoring | Cloud Monitoring |

**Access Points** (after `docker-compose up`):
- Frontend: `http://localhost:3000`
- API Docs: `http://localhost:8000/docs`
- FHIR Server: `http://localhost:8080`
- Grafana: `http://localhost:3001`
- RabbitMQ Management: `http://localhost:15672`

**Development Workflow**:
```bash
# Morning: Start environment
docker-compose up -d

# Develop all day (hot-reload enabled)
# Edit code in ./backend or ./frontend

# Evening: Stop to save laptop resources
docker-compose down

# Weekend: Stop everything, zero cost
docker-compose down
```

**Cost**: **$0/month** infrastructure (only LLM API calls ~$20-50/month)

> **📋 Full Docker Compose Configuration**: See [POC Open Source Stack Configuration](#poc-open-source-stack-configuration) for the complete `docker-compose.yml` file and all environment variables.

---

## Use Cases - POC & MVP Implementation

### UC-1: Clinical Documentation Assistant (Copilot) ⭐

**Target Users**: Physicians, Nurse Practitioners  
**Goal**: Reduce documentation time by 40% (save 48+ minutes daily per clinician)

**Workflow**:
1. Clinician sees patient, takes brief notes or voice memo
2. Copilot retrieves patient context from FHIR (demographics, allergies, meds, recent visits)
3. Claude generates structured SOAP note from brief input
4. Clinician reviews in Smart Sidecar UI, edits as needed
5. One-click approval inserts note into EHR via FHIR

**Technology Stack**:
- LLM: Claude (nuanced clinical reasoning)
- Vector DB: SOAP note templates, hospital-specific phrasing
- FHIR: Patient, Encounter, Condition, MedicationRequest

**Success Metrics**:
- 40% reduction in documentation time
- 90%+ clinician acceptance rate
- <10 seconds response time
- Zero PHI leaks

#### POC Scope (Phase 0)
- ✅ Basic SOAP note generation from text input
- ✅ FHIR patient context retrieval (HAPI FHIR)
- ✅ Claude integration for note generation
- ✅ Simple web form UI for input/output
- ✅ 5-10 SOAP note templates in Qdrant
- ❌ No voice input (text only)
- ❌ No EHR integration (standalone)
- ❌ No ghost text/autocomplete

#### MVP Enhancements (Phase 1)
- ✅ Smart Sidecar UI integrated with EHR
- ✅ Ghost text and autocomplete
- ✅ Voice dictation (Web Speech API)
- ✅ SMART on FHIR launch
- ✅ One-click note insertion to EHR
- ✅ Expanded template library (100+ templates)

---

### UC-2: Chart Review Agent ⭐

**Target Users**: Quality assurance teams, Compliance officers, Clinical directors  
**Goal**: Automate 80%+ of routine chart reviews, identify documentation gaps and compliance issues

**Workflow**:
1. Agent automatically scans patient charts at predetermined intervals (e.g., 24h post-visit)
2. Retrieves complete patient record from FHIR (Encounter, Conditions, Medications, Procedures, Notes)
3. AI analyzes chart for:
   - **Documentation completeness** (missing progress notes, unsigned orders)
   - **Coding accuracy** (ICD-10 alignment with documented conditions)
   - **Clinical quality indicators** (appropriate workup, guideline adherence)
   - **Billing compliance** (procedure codes match documentation)
4. Generates structured review report with:
   - Flagged issues categorized by severity (Critical/High/Medium/Low)
   - Specific documentation gaps with chart line references
   - Suggested corrections and missing elements
5. Routes critical issues to clinical supervisors via email/dashboard alerts
6. Tracks resolution and generates monthly quality reports

**Technology Stack**:
- LLM: Claude 3.5 Sonnet (complex medical documentation reasoning)
- Vector DB: Quality standards, coding guidelines, compliance rules
- Public APIs: UMLS for code validation, ICD-10 mappings
- FHIR: Patient, Encounter, Condition, Procedure, DocumentReference, DiagnosticReport
- Trigger: Cloud Scheduler (batch) or Pub/Sub (real-time on chart finalization)

**Success Metrics**:
- Process 80%+ charts autonomously without human review
- Identify 95%+ of documentation gaps vs. manual review
- <30 seconds per chart review
- Reduce chart rejection rate by 40%
- 85%+ accuracy on coding compliance flags

#### POC Scope (Phase 0)
- ✅ Basic chart completeness checks
- ✅ FHIR data retrieval for single patient
- ✅ Manual trigger (scan one chart on demand)
- ✅ Simple quality report output
- ✅ 10-20 quality rules in vector DB
- ❌ No automated scheduling
- ❌ No real-time alerts
- ❌ No ICD-10 validation (limited to UMLS API calls)

#### MVP Enhancements (Phase 1)
- ✅ Automated batch scanning (Cloud Scheduler)
- ✅ Real-time event triggers (Pub/Sub on chart finalization)
- ✅ Alert routing to clinical supervisors
- ✅ Comprehensive coding validation with UMLS API
- ✅ Monthly trend reports and analytics
- ✅ 100+ quality and compliance rules

---

### UC-3: Advanced Analytics Dashboard ⭐

**Target Users**: Healthcare administrators, Department heads, Quality improvement teams  
**Goal**: Provide real-time insights into clinical operations, AI performance, and quality metrics

**Workflow**:
1. Dashboard pulls data from multiple sources:
   - FHIR Store: Patient encounters, procedures, diagnoses
   - Cloud SQL: AI copilot/agent usage logs
   - BigQuery: Aggregated historical analytics
2. AI-powered insights engine analyzes trends:
   - Clinician time savings from documentation copilot
   - Chart review compliance trends
   - Common documentation gaps by department/provider
   - AI suggestion acceptance rates
3. Interactive visualizations display:
   - **Operational Metrics**: Patient volume, visit duration, documentation time
   - **Quality Metrics**: Chart completion rates, coding accuracy, compliance scores
   - **AI Performance**: Copilot usage, acceptance rates, time savings
   - **Financial Impact**: Cost per encounter, billing compliance, ROI calculations
4. Automated alerts for:
   - Declining quality scores
   - Unusually high chart rejection rates
   - AI performance anomalies
5. Natural language query interface: "Show me chart completion trends for cardiology last month"
6. Export reports to PDF/Excel for leadership meetings

**Technology Stack**:
- LLM: Gemini (natural language queries, insight summaries)
- Frontend: Next.js with charting libraries (Chart.js, Recharts)
- Data Sources: BigQuery (analytics warehouse), Cloud SQL, FHIR
- APIs: Healthcare API for FHIR queries, custom FastAPI for aggregations
- Caching: Memorystore for dashboard performance

**Success Metrics**:
- Dashboard load time <2 seconds
- 90%+ of queries answered without custom reports
- Real-time data updates (5-minute lag max)
- 4.5+ out of 5.0 user satisfaction for insights quality
- Reduce time spent on manual reporting by 70%

#### POC Scope (Phase 0)
- ❌ **Not implemented in POC**
- Focus POC on UC-1 and UC-2 core functionality
- Analytics deferred to MVP phase

#### MVP Implementation (Phase 1)
- ✅ Full dashboard with 4 metric categories:
  - Operational: Volume, duration, time savings
  - Quality: Chart completion, coding accuracy
  - AI Performance: Acceptance rates, latency
  - Financial: Cost per encounter, ROI
- ✅ Natural language query with Gemini
- ✅ BigQuery data warehouse
- ✅ Automated alerts for anomalies
- ✅ Export to PDF/Excel

---

## UI/UX Design - "Quiet Intelligence"

### Design Philosophy
The AI should be **visible when helpful** but **unobtrusive when not needed**. Avoid "AI for AI's sake" - every interaction must save time or improve quality.

### Copilot Interfaces

#### 1. Smart Sidecar (Contextual Sidebar)
**Use**: Diagnostic Support, Medication Checks

**Design**:
```
┌─────────────────────────┬──────────────────────────┐
│   EHR Patient Chart     │   Clinical Copilot       │
│                         │                          │
│  Patient: John Doe, 65M │  ⚠️ Sepsis Risk:         │
│                         │  Elevated (85%)          │
│  Vitals:                │  ┌────────────────────┐  │
│  • BP: 105/70           │  │ WBC: 18.5 ↑        │  │
│  • HR: 115 (Abnormal)   │  │ Temp: 102.4°F      │  │
│  • Temp: 102.4°F        │  │ Lactate pending    │  │
│  • SpO2: 94%            │  └────────────────────┘  │
│                         │  Recommend:              │
│  Labs:                  │  • Order serum lactate   │
│  • WBC: 18.5 ↑          │  • Blood cultures x2     │
│  • Bands: 12%           │  • Start empiric abx     │
│                         │                          │
│  History:               │  [View Guidelines]       │
│  • COPD                 │  [Order Labs]            │
│  • Diabetes             │                          │
│                         │  ────────────────────    │
│                         │                          │
│                         │  📝 SOAP Note Ready      │
│                         │  "65M with fever..."     │
│                         │  [Insert Note]           │
└─────────────────────────┴──────────────────────────┘
```

**Key Features**:
- Auto-updates based on current patient record
- Insight Cards (Sepsis Risk, Drug Interaction, etc.)
- Citations link to source data in EHR
- One-click Accept/Reject/Edit

---

#### 2. Ghost Text & Autocomplete
**Use**: Clinical Documentation

**Behavior**:
```
Clinician types: "Patient is a 45yo male with|"
                                            ↑
AI projects grey ghost text: " chest pain radiating to left arm..."
                              [Press Tab to accept]
                              [Press Esc to ignore]
```

**Voice Mode**:
- Real-time speech-to-text transcription
- AI structures unstructured dictation into SOAP format
- Floating transcription window shows live text

---

### Agent Interfaces

#### Command Center Dashboard
**Use**: Monitor autonomous agents (triage, scheduling, chart review)

**Design**:
```
╔════════════════════════════════════════════════════════╗
║            PATIENT TRIAGE DASHBOARD                    ║
╠════════════════════════════════════════════════════════╣
║                                                        ║
║  ┌─────────────┬─────────────┬─────────────┐         ║
║  │ IMMEDIATE   │   URGENT    │ SEMI-URGENT │         ║
║  │     3       │      12     │      45      │         ║
║  ├─────────────┼─────────────┼─────────────┤         ║
║  │ • Chest pain│ • Abdominal │ • Ankle sprain│        ║
║  │   Smith, J  │   pain      │   (multiple)  │        ║
║  │   [HIGH ⚠️] │   Lee, K    │              │        ║
║  │             │   [MED 🟡]  │              │        ║
║  │ • SOB       │             │              │        ║
║  │   Davis, M  │ • Fever     │              │        ║
║  │   [HIGH ⚠️] │   (multiple)│              │        ║
║  └─────────────┴─────────────┴─────────────┘         ║
║                                                        ║
║  Confidence Indicators:                               ║
║  🟢 >90% (Auto-approved)   🟡 70-90% (Review)        ║
║  🔴 <70% (Needs human triage)                        ║
║                                                        ║
║  Today: 248 triaged | 235 auto | 13 reviewed         ║
╚════════════════════════════════════════════════════════╝
```

---

## EHR Integration Strategy

### Primary Launch Method: SMART on FHIR Embedded

**User Experience**:
1. Clinician logs into Epic/Cerner
2. Opens patient chart
3. Clicks "AI Clinical Copilot" app tile
4. Platform launches in sidebar iframe with full patient context

**Authentication Flow**:
```
Epic/Cerner → SMART Launch Request (OAuth 2.0)
           ↓
Platform receives FHIR token with:
  • Patient ID
  • User identity
  • Scopes (read Patient, Observation, etc.)
  • Launch context
           ↓
Platform queries FHIR server for patient data
           ↓
Copilot interface loads with context
```

### Additional Launch Methods

| Method | Use Case | Context Passed |
|--------|----------|----------------|
| Browser Extension | Quick access without EHR navigation | Scrapes patient ID from URL |
| Standalone Web App | Care coordinators, admins | Manual patient search |
| Mobile App | On-call physicians, mobile rounds | Deep links from push notifications |
| Contextual Links | Email alerts, queue systems | URL parameters with patient ID |

---

## Data & Knowledge Strategy

### Hybrid Approach: Vector Database + Public APIs

#### Vector Database (Vertex AI Vector Search)
**Store**:
- ✅ Hospital-specific protocols and policies
- ✅ Approved formulary (drug list)
- ✅ Local treatment pathways
- ✅ SOAP note templates
- ✅ Curated clinical guidelines (AHA, ACC, etc.)
- ✅ Frequently used content (top 500 drugs)

**Why**: Fast semantic search, offline access, custom content, hospital-specific knowledge

**Cost**: ~$30-50/month for 100GB

---

#### Public APIs (All FREE)

**RxNorm API** (NIH):
- Drug name normalization: "Lipitor" → "Atorvastatin"
- Drug relationships and equivalencies
- RxCUI codes

**PubMed E-utilities** (NIH):
- 35+ million biomedical citations
- Search latest research in real-time
- Fetch abstracts on-demand

**UMLS API** (NIH):
- ICD-10, SNOMED CT, LOINC codes
- Code validation and mapping
- Medical terminology relationships

**FDA Drug Label API**:
- Drug labels and instructions
- Recent FDA warnings
- Known drug interactions

**Why**: Always current, authoritative sources, zero storage cost

**Cost**: $0 (FREE)

---

#### Caching Strategy (Redis)
All public API responses cached in Memorystore:
- Drug info: 7 days TTL
- FDA warnings: 1 day TTL
- PubMed searches: 1 hour TTL
- ICD codes: 30 days TTL

**Cost Comparison**:
- All Vector DB: $1,500/year
- All Public APIs (with caching): $600/year
- **Hybrid (Recommended)**: **$800/year**

---

## Deployment Architecture

### Environment Strategy

| Environment | Hosting | Database | FHIR Store | Purpose |
|-------------|---------|----------|------------|---------|
| **Dev** | Docker Compose (local/VM) | Cloud SQL (dev) | Healthcare API (synthetic) | Local development |
| **Staging** | Cloud Run | Cloud SQL | Healthcare API (test data) | Pre-production testing |
| **Production** | Cloud Run (multi-region) | Cloud SQL (HA + replicas) | Healthcare API (live PHI) | Clinical use |

### Container Deployment: Cloud Run

**Why Cloud Run?**
- ✅ Serverless - zero infrastructure management
- ✅ Auto-scaling from 0 to 1,000+ instances
- ✅ Pay-per-use - no cost when idle
- ✅ HIPAA-compliant
- ✅ Built-in HTTPS and load balancing
- ✅ Fast deployments (seconds)

**Trade-offs**:
- ❌ 60-minute max request timeout (acceptable for healthcare)
- ❌ Cold starts (1-3 seconds, mitigated with min instances)

**Future Migration Path**: If scale requires, can migrate to GKE with no code changes (same Docker containers)

---

### CI/CD Pipeline

```
GitHub Repository
      ↓
   [Push to main]
      ↓
Cloud Build (Trigger)
      ↓
┌─────────────────────┐
│ 1. Build Docker     │
│    Images           │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ 2. Run Tests        │
│    • pytest         │
│    • Jest           │
│    • Security scan  │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ 3. Push to          │
│    Artifact Registry│
└──────────┬──────────┘
           ↓
    ┌──────┴──────┐
    ↓             ↓
Deploy to     Deploy to
Staging       Production
(auto)        (manual approval)
```

---

## Security & Compliance

### HIPAA Compliance Checklist

#### Business Associate Agreements (BAAs)
- ✅ Google Cloud Platform
- ✅ Anthropic (Claude)
- ✅ Google (Gemini)

#### Encryption
- ✅ **At-rest**: Cloud KMS customer-managed encryption keys (CMEK)
- ✅ **In-transit**: TLS 1.3 for all communications
- ✅ **Database**: Cloud SQL automatic encryption
- ✅ **FHIR Store**: Healthcare API built-in encryption

#### Access Controls
- ✅ **Role-Based Access Control (RBAC)**: Cloud IAM policies
- ✅ **Attribute-Based Access Control (ABAC)**: Fine-grained permissions
- ✅ **Multi-Factor Authentication (MFA)**: Required for all users
- ✅ **Workload Identity**: Cloud Run → GCP services (no keys)

#### Audit Logging
- ✅ All PHI access logged to Cloud Audit Logs
- ✅ 7-year retention (HIPAA requirement)
- ✅ BigQuery for audit analytics
- ✅ Real-time alerts for suspicious activity

#### Data Loss Prevention
- ✅ DLP scanning of all AI outputs before logging
- ✅ Automatic PHI redaction in non-production environments
- ✅ Sensitive data detection in API requests

#### Network Security
- ✅ VPC with private service connect
- ✅ VPC Service Controls (security perimeter)
- ✅ Cloud Armor for DDoS + WAF
- ✅ Serverless VPC Access for Cloud Run

---

### Security Architecture Layers

1. **Network Security**: VPC, firewall rules, Serverless VPC Access
2. **Identity & Access**: Service accounts, IAM policies, MFA
3. **Data Protection**: CMEK, VPC Service Controls, DLP scanning
4. **Application Security**: Mutual TLS, secret management, container scanning
5. **Compliance**: PHI access controls, audit trails, breach detection

---

## Implementation Roadmap

### **Phase 0: POC with Open Source Stack (8 Weeks)**

#### Weeks 1-2: Local Development Setup
- [ ] Set up local development environment (Docker, Docker Compose)
- [ ] Create project structure (backend, frontend, infrastructure)
- [ ] Initialize Git repository
- [ ] Create Docker Compose configuration for all services
- [ ] Set up HAPI FHIR server with synthetic test data
- [ ] Configure Qdrant vector database
- [ ] Set up Prometheus + Grafana monitoring

#### Weeks 3-4: Core Backend Development
- [ ] Implement FastAPI backend with FHIR integration (HAPI FHIR)
- [ ] Build FHIR service layer for Patient, Observation, Condition resources
- [ ] Integrate Claude and Gemini APIs (with 80% mocking for cost savings)
- [ ] Implement LangGraph orchestration for multi-step workflows
- [ ] Set up Qdrant integration for vector search
- [ ] Build model router for Claude/Gemini selection

#### Weeks 5-6: Frontend & Use Cases
- [ ] Build Next.js frontend with basic UI components
- [ ] Implement UC-1: Clinical Documentation Copilot
  - SOAP note generation
  - Smart Sidecar UI component
- [ ] Implement UC-2: Chart Review Agent
  - Automated chart scanning
  - Quality issue detection
- [ ] Build simple analytics dashboard

#### Weeks 7-8: POC Validation & Demo
- [ ] Internal testing with synthetic data
- [ ] Demo to 5-10 test users (clinicians)
- [ ] Collect feedback on functionality and UX
- [ ] Document POC results and lessons learned
- [ ] **Decision Point**: Proceed to MVP or pivot?

**POC Exit Criteria**:
- ✅ 1-2 use cases functional end-to-end
- ✅ Positive feedback from test users
- ✅ Technical feasibility validated
- ✅ Cost model validated (LLM usage patterns)

**Total POC Cost**: **$0** (local development, LLM API calls only)

---

### **Phase 1: Migration to MVP (8 Weeks)**

#### Weeks 9-10: GCP Infrastructure Setup
- [ ] Set up GCP organization and billing
- [ ] Initialize OpenTofu project structure (persistent + ephemeral)
- [ ] Create GCP projects (dev, staging, prod)
- [ ] Provision managed services:
  - Cloud Run
  - Cloud SQL (PostgreSQL)
  - Healthcare API (FHIR Store)
  - Memorystore (Redis)
  - Vertex AI Vector Search
- [ ] Configure networking (VPC, firewall rules, Serverless VPC Access)
- [ ] Set up Cloud Build CI/CD pipeline
- [ ] Configure Artifact Registry

#### Weeks 11-12: Data Migration
- [ ] Export data from POC environment:
  - PostgreSQL database dump
  - Qdrant vectors export
  - HAPI FHIR data export
- [ ] Migrate PostgreSQL to Cloud SQL
- [ ] Migrate Qdrant vectors to Vertex AI Vector Search
- [ ] Migrate FHIR data to Healthcare API
- [ ] Migrate secrets to Secret Manager
- [ ] Validate data integrity post-migration

#### Weeks 13-14: Application Migration
- [ ] Update application code for GCP services:
  - Cloud SQL connection strings
  - Memorystore Redis URLs
  - Healthcare API endpoints
  - Vertex AI Vector Search integration
- [ ] Build Docker images and push to Artifact Registry
- [ ] Deploy backend to Cloud Run
- [ ] Deploy frontend to Cloud Run
- [ ] Configure Cloud CDN and Load Balancer
- [ ] Set up domain and SSL certificates

#### Weeks 15-16: Security & Compliance
- [ ] Implement security controls:
  - Cloud KMS encryption
  - DLP scanning
  - Cloud Armor WAF
  - IAM policies
- [ ] Configure audit logging (Cloud Audit Logs → BigQuery)
- [ ] Set up VPC Service Controls
- [ ] Security audit and penetration testing
- [ ] HIPAA compliance review
- [ ] Finalize BAAs with GCP, Anthropic, Google

**MVP Exit Criteria**:
- ✅ All POC functionality working on GCP
- ✅ HIPAA compliance validated
- ✅ Security controls in place
- ✅ Cost monitoring set up
- ✅ Ready for pilot with real users

**Total MVP Migration Cost**: $400-1,200/month (with optimizations)

---

### **Phase 2: MVP Pilot & Expansion (12 Weeks)**

#### Weeks 1-3: Infrastructure Foundation
- [ ] Set up GCP organization and projects (dev, staging, prod)
- [ ] Initialize OpenTofu project structure
- [ ] Provision Cloud Run, Cloud SQL, Healthcare API, Memorystore
- [ ] Configure networking (VPC, firewall rules)
- [ ] Set up CI/CD pipeline with Cloud Build
- [ ] Configure Artifact Registry for Docker images

#### Weeks 4-6: FHIR Integration
- [ ] Set up Healthcare API FHIR store
- [ ] Implement FHIR service layer (Python FastAPI)
- [ ] Test FHIR CRUD operations (Patient, Observation, etc.)
- [ ] Configure SMART on FHIR launch endpoint
- [ ] Test OAuth 2.0 flow with Epic/Cerner sandbox
- [ ] Implement FHIR patient context retrieval

#### Weeks 7-9: Orchestration & AI Layer
- [ ] Implement Google ADK/LangGraph orchestration engine
- [ ] Build workflow manager with state management (Redis)
- [ ] Integrate Claude API with retry logic and error handling
- [ ] Integrate Gemini API
- [ ] Build model router (load balance between Claude/Gemini)
- [ ] Test multi-step AI workflows end-to-end

#### Weeks 10-12: Public API Integration
- [ ] Integrate RxNorm API with Redis caching
- [ ] Integrate PubMed E-utilities API
- [ ] Integrate UMLS API for medical coding
- [ ] Integrate FDA Drug Label API
- [ ] Implement fallback mechanisms (API down → cache → vector DB)
- [ ] Test API response times and caching effectiveness

#### Weeks 13-15: Vector Database Setup
- [ ] Set up Vertex AI Vector Search index
- [ ] Implement embedding service (text-embedding-3)
- [ ] Load hospital protocols and policies into vector DB
- [ ] Load SOAP note templates
- [ ] Load curated clinical guidelines
- [ ] Test semantic search accuracy

#### Weeks 16-18: UC-1 - Clinical Documentation Copilot
- [ ] Build Copilot service (Python FastAPI)
- [ ] Implement Claude integration for SOAP note generation
- [ ] Build Smart Sidecar UI component (React)
- [ ] Implement ghost text autocomplete
- [ ] Integrate Web Speech API for voice dictation
- [ ] Test documentation workflow end-to-end

#### Weeks 19-21: UC-2 - Chart Review Agent
- [ ] Build Chart Review Agent service
- [ ] Implement Claude integration for chart analysis
- [ ] Build quality standards and compliance rules vector DB
- [ ] Integrate UMLS API for code validation
- [ ] Implement batch and real-time chart scanning triggers
- [ ] Build review report generation and severity classification
- [ ] Create alert routing system for critical issues
- [ ] Test autonomous chart review workflow

#### Weeks 22-24: UC-3 - Advanced Analytics Dashboard
- [ ] Design dashboard data architecture (BigQuery schema)
- [ ] Build ETL pipeline from FHIR and Cloud SQL to BigQuery
- [ ] Implement analytics API endpoints (FastAPI)
- [ ] Build dashboard frontend (Next.js with Chart.js/Recharts)
- [ ] Implement natural language query interface with Gemini
- [ ] Create operational, quality, AI performance, and financial metric views
- [ ] Implement automated alerts and reporting
- [ ] Test dashboard performance and data accuracy

#### Weeks 25-26: Security & Compliance Review
- [ ] Security audit and penetration testing
- [ ] HIPAA compliance review with legal/compliance team
- [ ] Finalize BAAs with Anthropic, Google
- [ ] Complete DLP scanning implementation
- [ ] Document audit procedures
- [ ] Prepare compliance documentation

#### Weeks 27-28: Clinical Validation & Pilot
- [ ] Pilot with 10-20 clinicians in controlled setting
- [ ] Collect feedback on UI/UX and accuracy
- [ ] Measure success metrics (time savings, chart review accuracy, dashboard usability)
- [ ] Iterate on AI prompts based on real-world usage
- [ ] Document clinical validation results
- [ ] Prepare for wider rollout

---

## Success Metrics

### Clinical Outcomes
- **Documentation Time**: Reduce by 40% (baseline → 40% reduction)
- **Chart Review Accuracy**: Identify 95%+ of documentation gaps vs. manual review
- **Chart Rejection Rate**: Reduce by 40% through improved documentation quality
- **Coding Compliance**: 85%+ accuracy on coding compliance flags
- **Adverse Events**: Zero major safety incidents attributed to AI

### User Adoption
- **Daily Active Users**: 60%+ of target clinician population
- **AI Acceptance Rate**: 90%+ of copilot suggestions accepted
- **User Satisfaction**: 4.0+ out of 5.0 rating
- **Time to Proficiency**: <1 hour training per clinician

### Operational Efficiency
- **Clinician Hours Saved**: 10,000+ hours/month organization-wide (documentation + chart review)
- **Chart Review Automation**: 80%+ autonomous processing
- **Dashboard Query Response**: 90%+ of queries answered without custom reports
- **System Uptime**: 99.9% availability
- **Response Time**: <10 seconds for copilots, <30 seconds for chart review, <2 seconds for dashboard

### Cost Efficiency
- **Infrastructure Cost**: <$5,000/month for 1,000 users
- **LLM Token Cost**: <$2,000/month
- **Total Cost per User**: <$8/user/month
- **ROI**: Positive return within 6 months

### Analytics & Insights
- **Dashboard Load Time**: <2 seconds
- **Data Freshness**: Real-time updates (5-minute lag max)
- **User Satisfaction with Insights**: 4.5+ out of 5.0
- **Reduction in Manual Reporting Time**: 70%

---

## Risk Mitigation

| Risk | Likelihood | Impact | Mitigation Strategy |
|------|------------|--------|---------------------|
| **AI Hallucinations** | High | Critical | RAG with citations, confidence scores, human-in-loop for copilots, comprehensive validation |
| **HIPAA Violations** | Medium | Critical | DLP scanning, audit logging, encryption, BAAs, quarterly compliance audits |
| **LLM API Downtime** | Medium | High | Multi-layer caching, graceful degradation, fallback to vector DB, 99.9% SLA with providers |
| **Clinical Errors** | Medium | Critical | Clinical validation studies, clinician override always available, comprehensive logging, incident response plan |
| **User Resistance** | Medium | Medium | Pilot program with champions, extensive training, emphasize time savings, collect continuous feedback |
| **EHR Integration Failures** | Low | High | SMART on FHIR standard, extensive testing with EHR sandboxes, fallback to manual entry |
| **Data Breaches** | Low | Critical | Zero-trust architecture, encryption, DLP, regular security audits, incident response plan |
| **Regulatory Changes** | Low | Medium | Legal/compliance team monitoring, flexible architecture, regular policy reviews |

---

## Cost Optimization Strategies

### Overview
LLM costs typically represent **50-70% of total infrastructure expenses**. Implementing cost optimization strategies can reduce overall platform costs by **50-70%** without sacrificing functionality.

**Baseline Cost**: $970-3,100/month for MVP (100 users)  
**Optimized Cost**: $400-1,200/month **(60-70% savings)**

---

### 1. LLM Token Usage Optimization (Highest Impact)

#### A. Smart Prompt Engineering
**Problem**: Verbose prompts waste tokens and increase latency

**Solutions**:
- ✅ Remove verbose instructions - use concise, directive language
- ✅ Avoid repetition - don't duplicate patient data already in context
- ✅ Set token budgets - max_tokens limits (500 for SOAP notes, 250 for triage)
- ✅ Use stop sequences to prevent unnecessary generation
- ✅ Remove examples from prompts after initial testing

**Savings**: 30-50% reduction in token usage  
**Monthly Impact**: $600-1,000/month saved

---

#### B. Caching Strategies
**Problem**: Identical or similar queries generate redundant LLM calls

**Solutions**:
```python
# 1. Prompt Caching (Claude supports this)
# Cache common system instructions/guidelines
system_prompt = "You are a clinical AI assistant..."  # Cached

# 2. Response Caching
# Identical queries return cached responses
cache_key = hash(f"{symptoms}:{patient_context}")
if cached_response := redis.get(cache_key):
    return cached_response

# 3. Semantic Caching
# Similar queries return cached responses
if similarity_score > 0.95:
    return cached_similar_response
```

**Savings**: 40-60% reduction for repeated queries  
**Monthly Impact**: $400-500/month saved

---

#### C. Model Tiering (Use Right Model for Right Task)
**Problem**: Using expensive models (Claude) for simple tasks

**Solution**: Implement intelligent model routing

| Task Complexity | Model | Cost/M Tokens | Use Cases |
|-----------------|-------|---------------|-----------|
| Simple | Gemini Flash | $0.35 | Triage classification, simple Q&A |
| Medium | Gemini Pro | $1.00 | Diagnostic reasoning, chart analysis |
| Complex | Claude Sonnet | $3.00 | SOAP notes, complex clinical reasoning |

**Implementation**:
```python
def route_to_model(task_type, complexity_score):
    if task_type == 'triage' or complexity_score < 3:
        return 'gemini-flash'  # 70% of requests
    elif complexity_score < 7:
        return 'gemini-pro'    # 20% of requests
    else:
        return 'claude-sonnet'  # 10% of requests
```

**Savings**: Use Gemini Flash for 70% of tasks  
**Monthly Impact**: $800-1,200/month saved

---

#### D. Batch Processing
**Problem**: Individual API calls have overhead

**Solutions**:
- ✅ Batch chart reviews (process 10-50 charts per API call)
- ✅ Use async processing for non-urgent tasks
- ✅ Schedule batch jobs during off-peak hours for lower priority items

**Savings**: 20-30% reduction through reduced API overhead  
**Monthly Impact**: $100-200/month saved

---

### 2. Infrastructure Optimization

#### A. Cloud Run Right-Sizing
**Current Configuration (Over-Provisioned)**:
- Min instances: 1 (always running)
- Max instances: 100
- CPU: 2 vCPU
- Memory: 2GB

**Optimized Configuration**:
```yaml
# Start lean, scale as needed
min_instances: 0  # Scale to zero when idle
max_instances: 10  # Prevent runaway costs during pilot
cpu: 1  # Sufficient for most tasks
memory: 512MB  # Start small, increase if needed
```

**Savings**: $100-300/month  
**When**: MVP launch, increase after validation

---

#### B. Cloud SQL Optimization
**Current**: db-n1-standard-1 ($50/month)  
**Optimized Development**:
```
Dev: db-f1-micro ($7/month)
Staging: db-g1-small ($25/month)
Production: db-custom-2-7680 ($40/month)
```

**Additional Optimizations**:
- ✅ Enable auto-scaling storage (start at 10GB, not 100GB)
- ✅ Set up automated backups with 7-day retention (not 30)
- ✅ Delete audit logs after 7 years (HIPAA minimum)

**Savings**: $15-25/month

---

#### C. Memorystore (Redis) Optimization
**Current**: Standard tier 5GB ($200/month)  
**Optimized**:
```
Dev: Basic tier 1GB ($15/month)
Staging: Basic tier 2GB ($30/month)
Production: Standard tier 5GB (only after pilot)
```

**Usage Guidelines**:
- ✅ Use Redis for hot data only (API responses, sessions)
- ✅ Store cold data in Cloud Storage ($0.02/GB)
- ✅ Set aggressive TTLs (7 days max for most data)

**Savings**: $25/month during development

---

### 3. Data Storage Optimization

#### A. Vector Database Cost Reduction
**Current Plan**: 100GB indexed ($30/month)  
**Optimized MVP**:
```
Start with 10-20GB of most critical content:
- Top 100 hospital protocols
- 500 most common drug monographs
- Core diagnostic criteria (Sepsis-3, CURB-65, etc.)
- SOAP templates for 5 specialties
```

**Incremental Loading Strategy**:
- Week 1-4: Core content only (10GB)
- Week 5-8: Add specialty-specific guidelines (20GB)
- Month 3+: Full content based on usage analytics (50-100GB)

**Savings**: $20-35/month  
**Future Cost**: Scale up as needed based on actual usage

---

#### B. Cloud Storage Lifecycle Policies
```yaml
# Automatically move old data to cheaper storage classes
lifecycle_rules:
  - condition:
      age: 30
    action:
      type: SetStorageClass
      storage_class: NEARLINE  # 50% cheaper
  
  - condition:
      age: 90
    action:
      type: SetStorageClass
      storage_class: COLDLINE  # 70% cheaper
  
  - condition:
      age: 365
    action:
      type: Delete  # Auto-delete temp files
```

**Savings**: 50-70% on storage costs  
**Monthly Impact**: $20-50/month

---

### 4. Development Environment Cost Savings

#### A. Local Development (Zero Cloud Cost)
```yaml
# docker-compose.yml for local dev
version: '3.8'
services:
  api:
    build: ./backend
    environment:
      - LLM_MOCK_MODE=true  # Mock LLM responses
      - FHIR_URL=http://hapi-fhir:8080  # Local FHIR server
  
  hapi-fhir:
    image: hapiproject/hapi:latest  # Open source FHIR server
  
  redis:
    image: redis:7-alpine  # Local cache
  
  postgres:
    image: postgres:15-alpine  # Local DB
```

**Development LLM Strategy**:
```python
# Mock expensive LLM calls during development
import os

if os.getenv('ENV') == 'development':
    def generate_soap_note(prompt):
        return "MOCK: Subjective: Patient reports..."
else:
    def generate_soap_note(prompt):
        return claude.complete(prompt)  # Real API call
```

**Savings**: $500-1,000/month during 6-month development phase  
**Total Savings**: $3,000-6,000 over development period

---

### 5. Observability Cost Optimization

#### A. Logging Cost Reduction
**Problem**: Verbose logging can cost $500+/month

**Solutions**:
```python
# 1. Set appropriate log retention
log_retention_days = 30  # Not forever

# 2. Filter noisy logs
if request.path == '/health':
    return  # Don't log health checks

# 3. Use sampling for success cases
if response.status == 200 and random.random() > 0.1:
    return  # Log only 10% of successful requests

# 4. Exclude debug logs in production
if os.getenv('ENV') == 'production':
    log_level = 'INFO'  # Not 'DEBUG'
```

**Savings**: 70-80% reduction in logging costs  
**Monthly Impact**: $100-200/month saved

---

#### B. Monitoring Optimization
- ✅ Use free tier metrics (first 150 metrics free)
- ✅ Only create custom metrics for critical business KPIs
- ✅ Use metric labels instead of separate metrics
- ✅ Set appropriate alert thresholds to avoid noise

**Savings**: $30-50/month

---

### 6. Cost Monitoring & Governance

#### A. Budget Alerts Setup
```bash
# Set up graduated budget alerts
gcloud billing budgets create \
  --billing-account=BILLING_ACCOUNT_ID \
  --display-name="Healthcare AI Budget" \
  --budget-amount=2000USD \
  --threshold-rule=percent=50 \
  --threshold-rule=percent=80 \
  --threshold-rule=percent=100 \
  --all-updates-rule-monitoring-notification-channels=CHANNEL_ID
```

---

#### B. Cost Attribution & Tracking
```python
# Tag every LLM call with metadata for cost tracking
import time
from datetime import datetime

def track_llm_usage(model, use_case, tokens_in, tokens_out):
    cost = calculate_cost(model, tokens_in, tokens_out)
    
    log_to_bigquery({
        'timestamp': datetime.utcnow(),
        'model': model,
        'use_case': use_case,
        'tokens_prompt': tokens_in,
        'tokens_completion': tokens_out,
        'cost_usd': cost,
        'user_id': current_user.id,
        'department': current_user.department
    })

# Example call
response = claude.complete(prompt)
track_llm_usage(
    model='claude-sonnet',
    use_case='clinical_documentation',
    tokens_in=response.usage.prompt_tokens,
    tokens_out=response.usage.completion_tokens
)
```

---

#### C. Weekly Cost Review Dashboard
Create BigQuery views for cost analysis:
```sql
-- Cost by use case (identify expensive features)
SELECT 
  use_case,
  COUNT(*) as request_count,
  SUM(cost_usd) as total_cost,
  AVG(cost_usd) as avg_cost_per_request
FROM llm_usage_log
WHERE DATE(timestamp) >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
GROUP BY use_case
ORDER BY total_cost DESC;

-- Cost by model (validate model tiering strategy)
SELECT 
  model,
  COUNT(*) as request_count,
  SUM(cost_usd) as total_cost,
  AVG(tokens_prompt + tokens_completion) as avg_tokens
FROM llm_usage_log
WHERE DATE(timestamp) >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
GROUP BY model;
```

---

### 7. Quick Wins (Implement First)

| Action | Effort | Savings/Month | Priority |
|--------|--------|---------------|----------|
| Use Gemini Flash for 70% of tasks | Low | $800-1,200 | 🔴 Critical |
| Set Cloud Run min instances = 0 | Low | $100-300 | 🔴 Critical |
| Implement prompt caching | Medium | $300-500 | 🟠 High |
| Start with 10GB Vector DB | Low | $20-35 | 🟠 High |
| Use basic Redis tier for dev | Low | $25 | 🟠 High |
| Set log retention to 30 days | Low | $50-100 | 🟡 Medium |
| Mock LLM calls in development | Low | $500-1,000 | 🟡 Medium |
| Filter health check logs | Low | $20-50 | 🟢 Low |

**Total Quick Wins**: **$1,815-3,210/month savings**

---

### 8. Cost Optimization Roadmap

#### Phase 1: MVP Launch (Implement Immediately)
- ✅ Model tiering (Gemini Flash for simple tasks)
- ✅ Cloud Run min instances = 0
- ✅ Smaller Cloud SQL and Redis instances
- ✅ 10GB Vector DB initial load
- ✅ Mock LLM calls in development
- ✅ Budget alerts at 50%/80%/100%

**Expected Savings**: 60-70% ($580-1,900/month)

#### Phase 2: Post-Pilot Optimization (Weeks 8-12)
- ✅ Implement prompt caching based on usage patterns
- ✅ Optimize prompts based on real-world performance
- ✅ A/B test model tiering accuracy vs. cost
- ✅ Set up cost attribution dashboard

**Expected Additional Savings**: 10-15% ($100-300/month)

#### Phase 3: Production Scale (Month 3+)
- ✅ Advanced caching strategies (semantic caching)
- ✅ Batch processing for background tasks
- ✅ Storage lifecycle optimization
- ✅ Fine-tune resource allocation based on 3 months of data

---

### 9. Cost Savings Summary

| Scenario | Before Optimization | After Optimization | Savings | % Reduction |
|----------|---------------------|-------------------|---------|-------------|
| **MVP (100 users)** | $970-3,100/month | $400-1,200/month | $570-1,900 | **60-70%** |
| **Production (1K users)** | $3,000-8,000/month | $1,500-4,000/month | $1,500-4,000 | **50%** |
| **Annual (1K users)** | $36,000-96,000/year | $18,000-48,000/year | $18,000-48,000 | **50%** |

---

### 10. Monitoring & Continuous Optimization

**Weekly Tasks**:
- Review BigQuery cost attribution dashboard
- Identify top 10 most expensive API calls
- Analyze prompt efficiency (tokens per request)
- Check for anomalous usage patterns

**Monthly Tasks**:
- Review GCP billing breakdown by service
- Compare actual vs. budgeted costs
- Identify optimization opportunities
- Update cost projections

**Quarterly Tasks**:
- Comprehensive cost audit
- Renegotiate LLM API contracts if volume justifies
- Evaluate new, cheaper model alternatives
- Update cost optimization strategy

---

**Key Principle**: Start lean, measure everything, scale what works. Don't over-provision based on theoretical maximums.

---

## POC Environment Cost & Teardown Strategy

### POC (Proof of Concept) Cost Breakdown

**POC Assumptions**:
- 5-10 test users for initial demos
- Synthetic FHIR data only (no real PHI)
- Single environment (no dev/staging separation)
- Limited usage: 4 hours/day, 5 days/week
- 2-month validation period

**Monthly Cost: $150-350/month (without teardown)**

| Component | POC Configuration | Monthly Cost |
|-----------|-------------------|--------------|
| **Infrastructure** |
| Cloud Run (3 services) | Min=0, Max=3, 512MB, 0.5 CPU | $20-50 |
| Cloud SQL | db-f1-micro (shared core) | $7 |
| Memorystore (Redis) | Basic tier 1GB | $15 |
| Healthcare API (FHIR) | 5GB synthetic data | $0.25 |
| Cloud Storage | 10GB | $0.20 |
| **AI/ML** |
| Claude API | 1.5M tokens/month (50K/day) | $5 |
| Gemini API | 3M tokens/month (100K/day) | $3 |
| Vector DB | 5GB minimal content | $1.50 |
| Embeddings | 500K tokens initial load | $0.07 |
| Public APIs | RxNorm, PubMed, UMLS, FDA | **FREE** |
| **Security & Observability** |
| Secret Manager, KMS | 10 secrets, 2 keys | $2.60 |
| Logging & Monitoring | Minimal usage | $5 |
| **Networking** |
| Load Balancer | Minimal traffic | $5-10 |
| **Total** | | **$150-350/month** |

---

### OpenTofu Teardown/Rebuild Strategy (80-90% Cost Savings)

**Problem**: Running POC environment 24/7 when only used 4 hours/day wastes money

**Solution**: Use OpenTofu to tear down environment when not in use

**With Teardown**: **$40-80/month (73% savings)**

---

### Implementation Strategy

#### 1. Separate Persistent vs. Ephemeral Infrastructure

**Persistent Infrastructure** (Never Destroy):
- ✅ Cloud Storage buckets (FHIR data, backups)
- ✅ Artifact Registry (Docker images)
- ✅ Secret Manager (API keys)
- ✅ OpenTofu state files

**Ephemeral Infrastructure** (Destroy Daily):
- ❌ Cloud Run services
- ❌ Memorystore Redis ($15/month → $2/month)
- ❌ Cloud SQL instance (stop, not destroy)
- ❌ Load Balancers

---

#### 2. Directory Structure

```
infrastructure/
├── persistent/          # Never destroyed
│   ├── main.tf         # Storage, secrets, registry
│   └── backend.tf      # State configuration
└── ephemeral/          # Destroyed when not in use
    ├── main.tf         # Cloud Run, Redis, SQL
    ├── variables.tf
    └── outputs.tf
```

---

#### 3. Daily Workflow Scripts

**File: `scripts/dev-start.sh`**
```bash
#!/bin/bash
set -e

echo "🚀 Starting development environment..."

# Apply ephemeral infrastructure
cd infrastructure/ephemeral
tofu apply -auto-approve

# Start Cloud SQL instance
gcloud sql instances patch healthcare-ai-db \
  --activation-policy=ALWAYS \
  --quiet

echo "✅ Environment ready! (5-10 minutes)"
```

**File: `scripts/dev-stop.sh`**
```bash
#!/bin/bash
set -e

echo "🛑 Stopping development environment..."

# Stop Cloud SQL (keeps data, stops billing)
gcloud sql instances patch healthcare-ai-db \
  --activation-policy=NEVER \
  --quiet

# Destroy ephemeral infrastructure
cd infrastructure/ephemeral
tofu destroy -auto-approve

echo "✅ Environment stopped. Saved ~$5-10 today!"
```

---

#### 4. Automated Weekend Shutdown (Optional)

**File: `.github/workflows/scheduled-teardown.yml`**
```yaml
name: Auto Environment Management

on:
  schedule:
    # Stop Friday 6pm EST
    - cron: '0 23 * * 5'
    # Start Monday 8am EST
    - cron: '0 13 * * 1'

jobs:
  manage-environment:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: opentofu/setup-opentofu@v1
      - uses: google-github-actions/auth@v1
        with:
          credentials_json: ${{ secrets.GCP_CREDENTIALS }}
      
      - name: Manage Infrastructure
        run: |
          cd infrastructure/ephemeral
          if [ "$(date +%u)" -eq 5 ]; then
            tofu destroy -auto-approve  # Friday
          else
            tofu apply -auto-approve    # Monday
          fi
```

**Result**: Automatic shutdown on weekends saves $30-40/month

---

### Cost Comparison: POC Scenarios

| Scenario | Duration | Cost | Notes |
|----------|----------|------|-------|
| **Always Running** | 2 months | $400-600 | Wasteful, 24/7 billing |
| **Manual Teardown** | 2 months | $80-160 | Daily start/stop (73% savings) |
| **Scheduled Teardown** | 2 months | $60-120 | Weekend auto-shutdown (80% savings) |
| **Ultra-Lean POC** | 2 months | $50-100 | Minimal config, 90% mocked LLMs |

---

### What Gets Preserved During Teardown

**✅ Preserved**:
- All FHIR data in Cloud Storage
- Docker images in Artifact Registry
- API keys in Secret Manager
- Database data (SQL stopped, not destroyed)
- OpenTofu state files
- Git repository

**❌ Lost** (Acceptable for Dev):
- Redis cache contents (easily repopulated)
- Running container instances
- Temporary logs (keep important ones in Cloud Storage)

**Rebuild Time**: 5-10 minutes (fully automated)

---

### POC Cost Summary

| Environment Type | Monthly Cost | 2-Month Total | vs. Baseline |
|------------------|--------------|---------------|--------------|
| **Always Running** | $200-300 | $400-600 | Baseline |
| **With Teardown** | $40-80 | **$80-160** | **73% savings** |
| **Ultra-Lean** | $25-50 | $50-100 | 83% savings |

**Recommended**: Use teardown strategy for POC to minimize risk and maximize learning per dollar spent.

---

### POC Open Source Stack Configuration

**Complete Docker Compose Setup** (`docker-compose.yml`):

```yaml
version: '3.8'

services:
  # Backend API
  api:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/healthcare_ai
      - REDIS_URL=redis://redis:6379
      - FHIR_URL=http://hapi-fhir:8080/fhir
      - QDRANT_URL=http://qdrant:6333
      - CLAUDE_API_KEY=${CLAUDE_API_KEY}
      - GEMINI_API_KEY=${GEMINI_API_KEY}
    depends_on:
      - postgres
      - redis
      - hapi-fhir
      - qdrant
    volumes:
      - ./backend:/app
  
  # Frontend
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:8000
    depends_on:
      - api
    volumes:
      - ./frontend:/app
  
  # FHIR Server (replaces Healthcare API)
  hapi-fhir:
    image: hapiproject/hapi:latest
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
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=healthcare_ai
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres-data:/var/lib/postgresql/data
  
  # Redis (replaces Memorystore)
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes
  
  # Vector Database (replaces Vertex AI Vector Search)
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
    volumes:
      - qdrant-data:/qdrant/storage
  
  # Message Queue
  rabbitmq:
    image: rabbitmq:3-management
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
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
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
```

**Start POC Environment**:
```bash
# Clone repository
git clone https://github.com/your-org/healthcare-ai
cd healthcare-ai

# Set environment variables
cp .env.example .env
# Edit .env with Claude and Gemini API keys

# Start all services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f api

# Access services:
# - Frontend: http://localhost:3000
# - API: http://localhost:8000/docs
# - FHIR Server: http://localhost:8080
# - Grafana: http://localhost:3001
# - RabbitMQ: http://localhost:15672
```

**Stop POC Environment**:
```bash
docker-compose down  # Preserves data volumes
# OR
docker-compose down -v  # Removes data volumes (fresh start)
```

**Total POC Cost**: **$0/month** (runs on laptop, only pays for LLM API calls)

---

### Migration Strategy: POC → MVP

#### Phase 1: Prepare for Migration (Week 1-2)

**1. Data Export from POC**
```bash
# Export PostgreSQL database
docker exec healthcare-ai-postgres-1 pg_dump -U postgres healthcare_ai > poc_data.sql

# Export Qdrant vectors
curl -X GET http://localhost:6333/collections/medical_knowledge/points/scroll \
  > qdrant_vectors.json

# Export FHIR data
curl http://localhost:8080/fhir/Patient?_format=json > fhir_patients.json
```

**2. Set Up GCP Infrastructure with OpenTofu**
```bash
cd infrastructure/persistent
tofu init
tofu apply  # Creates Cloud Storage, Artifact Registry, Secret Manager

cd ../ephemeral
tofu init
tofu apply  # Creates Cloud Run, Cloud SQL, Memorystore, Healthcare API
```

**3. Migrate Secrets**
```bash
# Upload API keys to Secret Manager
echo -n "${CLAUDE_API_KEY}" | gcloud secrets create claude-api-key --data-file=-
echo -n "${GEMINI_API_KEY}" | gcloud secrets create gemini-api-key --data-file=-
```

---

#### Phase 2: Data Migration (Week 3-4)

**1. Migrate PostgreSQL to Cloud SQL**
```bash
# Import to Cloud SQL
gcloud sql instances create healthcare-ai-db \
  --database-version=POSTGRES_15 \
  --tier=db-g1-small \
  --region=us-central1

# Import data
gcloud sql import sql healthcare-ai-db gs://your-bucket/poc_data.sql \
  --database=healthcare_ai
```

**2. Migrate Qdrant to Vertex AI Vector Search**
```python
# Migration script
from google.cloud import aiplatform
from qdrant_client import QdrantClient

# Read from Qdrant
qdrant = QdrantClient(host="localhost", port=6333)
points = qdrant.scroll(collection_name="medical_knowledge", limit=10000)

# Write to Vertex AI
aiplatform.init(project="your-project", location="us-central1")
index = aiplatform.MatchingEngineIndex("medical-knowledge-index")

# Batch upsert
for point in points:
    index.upsert_datapoints([{
        "id": str(point.id),
        "embedding": point.vector,
        "metadata": point.payload
    }])
```

**3. Migrate FHIR to Healthcare API**
```bash
# Create FHIR store
gcloud healthcare fhir-stores create healthcare-ai-fhir \
  --dataset=healthcare-ai-dataset \
  --location=us-central1

# Import FHIR data
gcloud healthcare fhir-stores import gcs healthcare-ai-fhir \
  --dataset=healthcare-ai-dataset \
  --location=us-central1 \
  --gcs-uri=gs://your-bucket/fhir/*.ndjson \
  --content-structure=BUNDLE
```

---

#### Phase 3: Application Migration (Week 5-6)

**1. Update Connection Strings**
```python
# Before (POC)
DATABASE_URL = "postgresql://postgres:password@localhost:5432/healthcare_ai"
REDIS_URL = "redis://localhost:6379"
FHIR_URL = "http://localhost:8080/fhir"
QDRANT_URL = "http://localhost:6333"

# After (MVP)
DATABASE_URL = os.getenv("CLOUD_SQL_CONNECTION_STRING")
REDIS_URL = os.getenv("MEMORYSTORE_REDIS_URL")
FHIR_URL = "https://healthcare.googleapis.com/v1/projects/PROJECT/locations/LOCATION/datasets/DATASET/fhirStores/STORE/fhir"
VERTEX_AI_INDEX = "projects/PROJECT/locations/LOCATION/indexes/INDEX"
```

**2. Deploy to Cloud Run**
```bash
# Build and push Docker images
docker build -t us-central1-docker.pkg.dev/PROJECT/healthcare-ai/api:latest ./backend
docker push us-central1-docker.pkg.dev/PROJECT/healthcare-ai/api:latest

# Deploy to Cloud Run
gcloud run deploy healthcare-ai-api \
  --image us-central1-docker.pkg.dev/PROJECT/healthcare-ai/api:latest \
  --platform managed \
  --region us-central1 \
  --set-secrets CLAUDE_API_KEY=claude-api-key:latest \
  --set-secrets GEMINI_API_KEY=gemini-api-key:latest
```

**3. Update Frontend API URL**
```javascript
// Before (POC)
const API_URL = 'http://localhost:8000';

// After (MVP)
const API_URL = process.env.NEXT_PUBLIC_API_URL || 'https://api.healthcare-ai.com';
```

---

#### Phase 4: Testing & Validation (Week 7-8)

**Parallel Testing**:
- Run POC and MVP simultaneously
- Compare outputs for accuracy
- Validate performance (latency, throughput)
- Test HIPAA compliance (encryption, audit logs)

**Cutover Checklist**:
- [ ] All data migrated successfully
- [ ] API endpoints returning correct responses
- [ ] FHIR queries working with Healthcare API
- [ ] Vector search accuracy matches POC
- [ ] Monitoring and alerts configured
- [ ] Security controls in place (DLP, KMS, IAM)
- [ ] Backup and disaster recovery tested
- [ ] Cost monitoring set up
- [ ] Documentation updated

**Once Validated**: Switch DNS to MVP, decommission POC

---

### Migration Complexity by Component

| Component | Complexity | Estimated Time | Risk |
|-----------|------------|----------------|------|
| PostgreSQL → Cloud SQL | 🟢 Easy | 2 hours | Low |
| Redis → Memorystore | 🟢 Easy | 1 hour | Low |
| HAPI FHIR → Healthcare API | 🟡 Medium | 8 hours | Medium |
| Qdrant → Vertex AI | 🟡 Medium | 8 hours | Medium |
| Local → Cloud Run | 🟡 Medium | 4 hours | Low |
| Docker Logs → Cloud Logging | 🟢 Easy | 2 hours | Low |
| **Total Migration Time** | | **25-30 hours** | |

**Recommended**: Plan 2-week migration window with parallel testing

---

## Immediate Next Steps

### This Week
1. ✅ Complete planning documentation
2. ⏭️ **Set up GCP organization and billing**
3. ⏭️ **Initialize OpenTofu project** with basic infrastructure
4. ⏭️ **Create GitHub repository** with project structure
5. ⏭️ **Set up development environment** (Docker, local FHIR server)

### Week 1-2 Goals
- Infrastructure provisioned via OpenTofu (Cloud Run, Cloud SQL, Healthcare API)
- Docker Compose working locally for all services
- Cloud Run deployment automated via Cloud Build
- FHIR store configured with synthetic test data

### Week 3-4 Goals
- FHIR integration working (CRUD operations)
- SMART on FHIR launch tested with sandbox
- Claude and Gemini APIs integrated
- First simple workflow end-to-end

---

## Decision Points & Open Questions

### Decisions Needed
- [ ] Confirm GCP budget and quotas ($5-10K/month for pilot)
- [ ] Select initial pilot department (ED, primary care, or specialty?)
- [ ] Determine go-live timeline (6 months realistic?)
- [ ] Finalize Epic/Cerner integration approach (which EHR first?)
- [ ] Select initial pilot users (10-20 clinicians)

### Open Questions
- How will we handle multi-tenant support (multiple hospitals)?
- What's the disaster recovery plan and RTO/RPO?
- How do we handle model versioning and A/B testing?
- What's the escalation path for AI errors?
- How do we measure and report clinical safety outcomes?

---

## Appendices

### A. Technology Selection Rationale

**OpenTofu vs. Pulumi/Terraform**:
- 100% open source (MPL 2.0), no vendor lock-in
- Terraform-compatible (huge ecosystem of modules)
- Community-governed by major companies

**Cloud Run vs. GKE**:
- MVP speed: Zero infrastructure management
- Cost: Pay only for actual usage (~50% cheaper for variable workloads)
- Scaling: Auto-scales from 0 to thousands automatically
- Easy migration path to GKE if needed later

**Claude + Gemini vs. single LLM**:
- Claude: Superior reasoning for clinical decisions, better safety alignment
- Gemini: Faster inference, multimodal (images), Google ecosystem integration
- Model router provides redundancy and cost optimization

**Hybrid Data Approach vs. all Vector DB**:
- Save ~$700/year with public APIs
- Always current data from authoritative sources
- Reduce storage and embedding costs
- Leverage free NIH/FDA resources

### B. FHIR Resources Used

Primary FHIR R4 Resources:
- **Patient**: Demographics, identifiers
- **Encounter**: Visit information
- **Observation**: Vitals, lab results
- **Condition**: Diagnoses, problems
- **MedicationRequest**: Current medications
- **AllergyIntolerance**: Drug allergies
- **DiagnosticReport**: Imaging, pathology
- **Procedure**: Surgical history

### C. Compliance References

- **HIPAA Security Rule**: 45 CFR Part 164, Subpart C
- **HIPAA Privacy Rule**: 45 CFR Part 164, Subpart E
- **HITECH Act**: Breach notification requirements
- **21st Century Cures Act**: Interoperability requirements
- **FDA Software as Medical Device**: Guidance documents

---

**Document Owner**: Marc Visocky  
**Last Updated**: January 12, 2026  
**Next Review**: Weekly during implementation  
**Status**: ✅ Architecture Finalized - Ready for Development

