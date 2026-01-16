# Healthcare AI Platform

HIPAA-compliant Healthcare AI Platform featuring Clinical Documentation Copilot and Chart Review Agent.

## Quick Start (POC)

```bash
# Start all services
docker-compose up -d

# Access services:
# - Frontend: http://localhost:3000
# - API: http://localhost:8000/docs
# - FHIR Server: http://localhost:8080
# - Grafana: http://localhost:3001
# - RabbitMQ: http://localhost:15672
```

## Architecture

- **Backend**: Python 3.11 + FastAPI
- **Frontend**: Next.js 14 + React
- **FHIR**: HAPI FHIR Server (POC) → Healthcare API (MVP)
- **Database**: PostgreSQL 15
- **Cache**: Redis 7
- **Vector DB**: Qdrant (POC) → Vertex AI (MVP)
- **LLMs**: Claude 3.5 Sonnet + Gemini 1.5 Pro

## Documentation

- **[Developer Setup Guide](DEVELOPER_SETUP_GUIDE.md)** - Complete setup from scratch
- **[Master Plan](MASTER_PLAN.md)** - Full architecture and strategy
- **[Technology Stack](technology_stack.csv)** - All technologies and costs

## Cost

- **POC**: $0/month (local Docker) + ~$20-50 LLM API calls
- **MVP**: $400-600/month (GCP managed services)
- **Production**: $3,000-4,000/month

## Development

```bash
# View logs
docker-compose logs -f api

# Restart service
docker-compose restart api

# Stop all services
docker-compose down
```

## Status

🚧 **In Development** - POC Phase (Phase 0)

---

**License**: MIT  
**Version**: 0.1.0
