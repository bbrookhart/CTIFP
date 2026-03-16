<div align="center">

<br/>

```
 ██████╗████████╗██╗███████╗██████╗
██╔════╝╚══██╔══╝██║██╔════╝██╔══██╗
██║        ██║   ██║█████╗  ██████╔╝
██║        ██║   ██║██╔══╝  ██╔═══╝
╚██████╗   ██║   ██║██║     ██║
 ╚═════╝   ╚═╝   ╚═╝╚═╝     ╚═╝
```

### Cognitive Threat Intelligence Fusion Platform

*Ontology-first cyber risk intelligence. Evidence-backed. Analyst-ready.*

<br/>

[![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Next.js](https://img.shields.io/badge/Next.js-14-000000?style=flat-square&logo=nextdotjs&logoColor=white)](https://nextjs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.111-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Neo4j](https://img.shields.io/badge/Neo4j-5.x-008CC1?style=flat-square&logo=neo4j&logoColor=white)](https://neo4j.com/)
[![Apache Kafka](https://img.shields.io/badge/Kafka-3.7-231F20?style=flat-square&logo=apachekafka&logoColor=white)](https://kafka.apache.org/)

[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![OpenSearch](https://img.shields.io/badge/OpenSearch-2.x-005EB8?style=flat-square&logo=opensearch&logoColor=white)](https://opensearch.org/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com/)
[![OpenTelemetry](https://img.shields.io/badge/OTel-Instrumented-425CC7?style=flat-square&logo=opentelemetry&logoColor=white)](https://opentelemetry.io/)
[![License](https://img.shields.io/badge/License-Apache%202.0-D22128?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/Status-v1%20Alpha-F97316?style=flat-square)]()

<br/>

[![CI](https://img.shields.io/github/actions/workflow/status/your-org/ctifp/ci.yml?branch=main&style=flat-square&logo=githubactions&logoColor=white&label=CI)](https://github.com/your-org/ctifp/actions)
[![Security Scan](https://img.shields.io/github/actions/workflow/status/your-org/ctifp/security-scan.yml?branch=main&style=flat-square&logo=githubactions&logoColor=white&label=Security)](https://github.com/your-org/ctifp/actions)
[![Coverage](https://img.shields.io/badge/Coverage-87%25-22C55E?style=flat-square)](https://github.com/your-org/ctifp)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE%20ATT%26CK-v14-E31B23?style=flat-square)](https://attack.mitre.org/)
[![STIX 2.1](https://img.shields.io/badge/STIX-2.1-8B5CF6?style=flat-square)](https://oasis-open.github.io/cti-documentation/)
[![OCSF](https://img.shields.io/badge/OCSF-1.1-0EA5E9?style=flat-square)](https://schema.ocsf.io/)

</div>

---

## What is CTIFP?

CTIFP is an **ontology-first cyber risk platform** that ingests internal security telemetry and external open-source intelligence, fuses them into a unified semantic graph, and surfaces ranked, evidence-backed risk findings for analyst review.

It answers three operational questions:

| # | Question | How CTIFP Answers It |
|---|----------|----------------------|
| **1** | What internal assets, identities, and systems matter most right now? | Asset criticality graph seeded from live telemetry, enriched with vulnerability and control coverage data |
| **2** | What external threats, campaigns, and vulnerabilities are relevant to this environment? | LLM extraction pipeline maps threat reports, news, and CTI feeds into typed graph entities |
| **3** | What risk pathways should analysts prioritize first — and why? | Transparent scoring model with full evidence chains, ATT&CK mapping, and EPSS/KEV prioritization |

> **Design philosophy:** The graph is the intelligence layer. The LLM is an enrichment component, not the system of record. No extraction writes to production state without human review.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│  L1 · INGESTION                                                      │
│  Splunk · CloudTrail · CISA KEV · MITRE ATT&CK · News RSS · GDELT   │
│  Schema validation → MinIO raw lake → Kafka topics                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │ raw.events  raw.documents
┌────────────────────────────▼────────────────────────────────────────┐
│  L2 · DOCUMENT INTELLIGENCE                                          │
│  Parse → Chunk → Embed → OpenSearch                                  │
│  LLM Extraction (Instructor + litellm) → Pydantic schemas            │
│  → Postgres extraction_queue → Human review queue                    │
└────────────────────────────┬────────────────────────────────────────┘
                             │ approved extractions
┌────────────────────────────▼────────────────────────────────────────┐
│  L3 · FUSION & GRAPH                                                 │
│  Entity resolution (exact + fuzzy + embedding) → Entity registry     │
│  Graph-sync-service → Neo4j MERGE (node + edge + provenance)         │
│  Full mutation audit log in Postgres                                 │
└────────────────────────────┬────────────────────────────────────────┘
                             │ graph state
┌────────────────────────────▼────────────────────────────────────────┐
│  L4 · RISK ENGINE                                                    │
│  Feature extraction → Weighted linear scorer → ATT&CK gap analysis  │
│  KEV / EPSS prioritization → RiskFindings with rationale + evidence  │
└────────────────────────────┬────────────────────────────────────────┘
                             │ findings
┌────────────────────────────▼────────────────────────────────────────┐
│  L5 · ANALYST EXPERIENCE  (Next.js 14)                               │
│  Graph explorer · Search · Risk board · Evidence panel               │
│  Review queue · Case management · ATT&CK Navigator export            │
└─────────────────────────────────────────────────────────────────────┘
```

### Services

| Service | Language | Responsibility |
|---------|----------|----------------|
| `ingestion-service` | Python | Source connectors, schema validation, Kafka publish |
| `normalization-service` | Python | OCSF field mapping, canonical event model |
| `document-processing-service` | Python | Parse, chunk, embed, index documents |
| `entity-extraction-service` | Python | LLM extraction workflows (LangGraph + Instructor) |
| `entity-resolution-service` | Python | Dedup, fingerprint match, entity registry |
| `graph-sync-service` | Python | Validated MERGE writes to Neo4j |
| `risk-scoring-service` | Python | Risk features, scoring, ATT&CK/KEV mapping, findings |
| `graph-api` | Python / FastAPI | Entity detail, path finding, blast radius |
| `search-api` | Python / FastAPI | Hybrid BM25 + kNN retrieval |
| `review-api` | Python / FastAPI | Human-in-the-loop queue management |
| `analyst-ui` | Next.js 14 / TypeScript | Full analyst interface |

---

## Ontology

CTIFP models 30+ entity types across three domains. Every node and edge carries universal provenance properties enforced at the graph-sync boundary.

<details>
<summary><strong>Internal Domain</strong> — click to expand</summary>

| Entity | Description |
|--------|-------------|
| `Asset` | Physical or virtual system (criticality 1–5) |
| `Host` | OS-level host with IP/FQDN |
| `Service` | Network-exposed service (port/protocol/version) |
| `Application` | Software application with vendor and version |
| `Identity` | Human, service, or machine identity (sensitivity 1–5) |
| `Account` | Credential-bearing account linked to an Identity |
| `Role` | Permission set assigned to an Identity |
| `Credential` | Authentication credential (key, cert, secret) |
| `NetworkSession` | Observed network flow between assets |
| `Alert` | Security detection event with tactic/technique mapping |
| `Vulnerability` | CVE-linked weakness on an asset or service |
| `Control` | Defensive control with coverage scope and effectiveness |
| `BusinessUnit` | Organizational unit owning assets and identities |

</details>

<details>
<summary><strong>External Domain</strong> — click to expand</summary>

| Entity | Description |
|--------|-------------|
| `ThreatActor` | Named adversary with motivation and sophistication |
| `Malware` | Malware family with capabilities and YARA references |
| `Campaign` | Coordinated intrusion campaign with sector/region targets |
| `Indicator` | IP, domain, hash, or URL with TLP classification |
| `TTP` | MITRE ATT&CK technique (technique_id, tactic, sub-technique) |
| `CVE` | Vulnerability record with CVSS, EPSS, and KEV flag |
| `NewsEvent` | Extracted intelligence event from news or reports |
| `FilingEntity` | SEC/regulatory filing with risk factor extraction |
| `SourceDocument` | Parent document with provenance metadata |

</details>

<details>
<summary><strong>Operational Domain</strong> — click to expand</summary>

| Entity | Description |
|--------|-------------|
| `Observation` | Normalized observation linking raw events to entities |
| `RiskFinding` | Scored, evidence-backed risk with rationale |
| `Hypothesis` | Analyst-authored investigative hypothesis |
| `Recommendation` | Actionable remediation with priority and owner |
| `Case` | Investigation container grouping findings and notes |
| `AnalystNote` | Free-text annotation on any entity |
| `SourceRecord` | Provenance record for every extraction |

</details>

<details>
<summary><strong>Universal Node Properties</strong> — click to expand</summary>

Every node in the graph carries these properties, enforced by Pydantic at the graph-sync boundary:

```python
class BaseNode(BaseModel):
    id: UUID
    external_ids: list[str] = []
    source: str
    source_type: Literal["INTERNAL", "EXTERNAL", "DERIVED"]
    confidence: float = Field(ge=0.0, le=1.0)
    first_seen: datetime
    last_seen: datetime
    provenance_pointer: str          # source_record_id in Postgres
    review_status: Literal["PENDING", "APPROVED", "REJECTED", "AUTO_APPROVED"]
    created_at: datetime
    updated_at: datetime
    labels: list[str] = []
```

</details>

<details>
<summary><strong>Key Relationships</strong> — click to expand</summary>

```
(Asset)          -[:HAS_VULNERABILITY]->  (Vulnerability)
(Asset)          -[:COMMUNICATES_WITH]->  (Asset)
(Identity)       -[:HAS_ACCOUNT]->        (Account)
(Account)        -[:AUTHENTICATES_TO]->   (Application)
(ThreatActor)    -[:USES_TTP]->           (TTP)
(Campaign)       -[:ATTRIBUTED_TO]->      (ThreatActor)
(Campaign)       -[:TARGETS]->            (Asset | BusinessUnit)
(Malware)        -[:EXPLOITS]->           (CVE)
(Alert)          -[:MATCHES_INDICATOR]->  (Indicator)
(Vulnerability)  -[:RAISES_RISK_FOR]->    (Asset)       <- computed
(TTP)            -[:MITIGATED_BY]->       (Control)
(NewsEvent)      -[:MENTIONS]->           (ThreatActor | Campaign | CVE)
```

All edges carry: `confidence`, `first_seen`, `last_seen`, `source`, `provenance_pointer`

</details>

---

## Risk Scoring

Risk scores are computed by a **transparent weighted linear model** — no black boxes. Every score is accompanied by a full weight breakdown and evidence reference list.

```
risk_score = sum(wi * feature_i) * recency_decay

Features:
  asset_criticality       (w=0.25)  <- business-defined 1-5 scale
  vuln_exploitability     (w=0.20)  <- CVSS x EPSS, KEV bonus
  threat_proximity        (w=0.20)  <- BFS depth to nearest ThreatActor/Campaign
  adversary_relevance     (w=0.15)  <- sector/tech targeting overlap
  control_coverage_gap    (w=0.12)  <- ATT&CK techniques without mitigating control
  identity_sensitivity    (w=0.08)  <- linked identity sensitivity rating
  signal_recency          (w=0.05)  <- volume and freshness of external signals
```

All weights are versioned in config. Score rationale serializes to human-readable text in every `RiskFinding`.

---

## Standards & Integrations

| Standard | Usage |
|----------|-------|
| **MITRE ATT&CK v14** | TTP entity mapping, coverage gap analysis, Navigator export |
| **STIX 2.1** | External CTI entity modeling (ThreatActor, Campaign, Indicator, Malware) |
| **OCSF 1.1** | Canonical normalization model for all internal security events |
| **CISA KEV** | Vulnerability prioritization (daily sync, KEV flag on CVE nodes) |
| **NVD / CVE** | CVE metadata, CVSS v3, EPSS scores |
| **TAXII** | Planned v2: inbound and outbound STIX bundle exchange |

---

## Getting Started

### Prerequisites

- Docker and Docker Compose v2.20+
- Node.js 20+ (for analyst-ui)
- Python 3.12+
- 16 GB RAM recommended for full local stack

### Bootstrap

```bash
# Clone the repository
git clone https://github.com/your-org/ctifp.git
cd ctifp

# Copy environment config
cp .env.example .env

# Start all infrastructure services
# (Kafka, Postgres, Neo4j, OpenSearch, Redis, MinIO, Vault, Keycloak)
docker compose -f docker-compose.infra.yml up -d

# Wait for services to be healthy
make wait-healthy

# Run database migrations and seed base data
make bootstrap

# Seed MITRE ATT&CK and CISA KEV
make seed-attck
make seed-kev

# Start all application services
docker compose up -d

# Verify everything is running
make status
```

### Access

| Interface | URL | Default Credentials |
|-----------|-----|---------------------|
| Analyst UI | http://localhost:3000 | `analyst / changeme` |
| graph-api (Swagger) | http://localhost:8001/docs | Bearer token from Keycloak |
| search-api (Swagger) | http://localhost:8002/docs | Bearer token from Keycloak |
| review-api (Swagger) | http://localhost:8003/docs | Bearer token from Keycloak |
| Neo4j Browser | http://localhost:7474 | `neo4j / changeme` |
| OpenSearch Dashboards | http://localhost:5601 | `admin / changeme` |
| Keycloak Admin | http://localhost:8080 | `admin / changeme` |
| Grafana | http://localhost:3001 | `admin / changeme` |
| Jaeger | http://localhost:16686 | — |

> **Warning:** Change all default credentials before any network-exposed deployment.

---

## Development

### Project Structure

```
ctifp/
├── services/
│   ├── ingestion-service/
│   ├── normalization-service/
│   ├── document-processing-service/
│   ├── entity-extraction-service/
│   ├── entity-resolution-service/
│   ├── graph-sync-service/
│   ├── risk-scoring-service/
│   ├── graph-api/
│   ├── search-api/
│   ├── review-api/
│   └── analyst-ui/
├── libs/
│   ├── ctifp-models/          # Shared Pydantic ontology models
│   ├── ctifp-kafka/           # Producer / consumer base classes
│   ├── ctifp-auth/            # JWT validation middleware
│   ├── ctifp-observability/   # OTel setup and metric registration
│   └── ctifp-testing/         # Shared fixtures and test factories
├── infra/
│   ├── postgres/migrations/
│   ├── neo4j/
│   ├── opensearch/
│   ├── kafka/
│   ├── keycloak/
│   └── vault/
├── scripts/
└── docs/
```

### Common Tasks

```bash
# Run tests for all services
make test

# Run tests for a specific service
make test SERVICE=graph-api

# Lint and type-check everything
make lint

# Build all Docker images
make build

# Tail logs for a specific service
make logs SERVICE=entity-extraction-service

# Open a Cypher shell against local Neo4j
make cypher

# Run the end-to-end smoke test (ingest -> extract -> graph -> finding)
make e2e

# Generate OpenAPI specs for all services
make openapi
```

### Adding a New Ingestion Connector

1. Create `services/ingestion-service/src/connectors/your_source.py` extending `BaseConnector`
2. Implement `fetch() -> AsyncIterator[RawEvent | RawDocument]`
3. Register the connector in `connectors/__init__.py`
4. Add schema mapping in `normalization-service/src/ocsf/` if it is an internal event source
5. Add a Kafka topic entry in `infra/kafka/topic-configs/`
6. Write unit tests in `services/ingestion-service/tests/`

See [`docs/adding-a-connector.md`](docs/adding-a-connector.md) for the complete guide.

---

## LLM Extraction Pipeline

The extraction pipeline uses [Instructor](https://github.com/jxnl/instructor) on top of [litellm](https://github.com/BerriAI/litellm) for model-agnostic structured output. Extractions are schema-constrained, retried on validation failure, and **never write directly to Neo4j**.

```
document chunk
      |
      v
LangGraph extraction workflow
      |
      +-- ThreatActor extractor  -+
      +-- Campaign extractor      |
      +-- TTP extractor           +---> Pydantic validation --> confidence score
      +-- Indicator extractor     |
      +-- CVE extractor          -+
                |
                v
         extraction_queue (Postgres)
                |
         analyst review  (approve / reject / edit)
                |
                v
         entity resolution
                |
                v
          Neo4j MERGE  <- graph-sync-service only
```

**Supported models:** Any OpenAI-compatible API endpoint (GPT-4o, Claude, Mistral, local Ollama). Configure via `LLM_MODEL` in `.env`.

---

## Security

CTIFP is designed secure-by-default. Key controls:

- **Zero-trust service mesh** — all inter-service calls use short-lived mTLS certs or OIDC client-credentials tokens
- **LLM write gate** — no LLM output touches Neo4j without schema validation and human approval
- **Vault-managed secrets** — dynamic Postgres credentials, LLM API key rotation, no secrets in env vars or source control
- **Strict schema validation** — Pydantic v2 strict mode at every Kafka consumer and API boundary; unknown fields rejected
- **Fine-grained RBAC** — `analyst`, `senior_analyst`, `admin` roles with per-endpoint enforcement via JWT claims
- **Audit trail** — every API call, queue decision, graph mutation, and login logged to Postgres `audit_log`
- **Parameterized Cypher only** — no string interpolation in graph queries; pass-through Cypher restricted to read-only analyst role
- **TLS everywhere** — TLS 1.3 on HTTP, SASL/SCRAM + TLS on Kafka, bolt+s on Neo4j, HTTPS on OpenSearch

To report a security vulnerability, see [`SECURITY.md`](SECURITY.md).

---

## Observability

Every service emits structured logs, Prometheus metrics, and OpenTelemetry traces. Trace IDs propagate through Kafka message headers for full distributed visibility.

| Signal | What | Tooling |
|--------|------|---------|
| Structured logs | Request in/out, queue events, DB queries, LLM calls, graph writes | structlog → Kafka → OpenSearch |
| Metrics | Ingestion rate, DLQ rate, extraction confidence histogram, scoring latency, API p50/p99 | Prometheus → Grafana |
| Traces | Full distributed trace: HTTP → Kafka → DB → Neo4j | OTel SDK → Jaeger |
| Pipeline status | Per-connector last-success, lag, messages/min | Prometheus gauge → Grafana |
| Extraction quality | Precision/recall on labeled set (weekly eval run) | Python eval job → Grafana |
| Graph mutations | Every MERGE/DELETE logged with before/after state hash | Postgres `graph_mutations` |

---

## Build Phases

| Phase | Scope | Weeks |
|-------|-------|-------|
| **1 — Foundation** | Repo, infra, containers, auth, schemas, CI | 1–3 |
| **2 — Ingestion** | 2 internal + 3 external connectors, OCSF normalization | 4–6 |
| **3 — Document Intelligence** | Parse, chunk, embed, LLM extraction, review queue | 6–9 |
| **4 — Fusion & Graph** | Entity resolution, graph upsert, cross-domain linking | 9–12 |
| **5 — Risk Engine** | Scoring model, ATT&CK/KEV mapping, ranked findings | 12–15 |
| **6 — Analyst Experience** | Graph UI, search, evidence panel, risk board, cases | 15–19 |

---

## Contributing

Contributions are welcome. Please read [`CONTRIBUTING.md`](CONTRIBUTING.md) before opening a pull request.

- All new entity types require updates to `libs/ctifp-models/` and `infra/neo4j/constraints.cypher`
- All new API endpoints require OpenAPI spec updates in `docs/api/`
- All graph-write paths must go through `graph-sync-service` — no direct Neo4j writes from other services
- Every PR must pass `make lint`, `make test`, and `make security-scan` in CI

---

## Roadmap

- [ ] Multi-tenant graph isolation and query enforcement
- [ ] Auto-approve for high-confidence entity types (CVE IDs, ATT&CK technique IDs)
- [ ] ML-trained risk scoring model using labeled finding history
- [ ] TAXII server for inbound/outbound STIX bundle exchange
- [ ] Dark web monitoring connector
- [ ] Graph neural network for entity resolution and relationship inference
- [ ] Real-time SIEM/SOAR webhook integration
- [ ] Active learning loop for extraction model fine-tuning
- [ ] ATT&CK Evaluations-based threat emulation scenario mapping

---

## License

Apache License 2.0 — see [`LICENSE`](LICENSE) for details.

---

<div align="center">

Built by security engineers, for security engineers.<br/>
Evidence-based cyber risk forecasting — no magic, no black boxes.

<br/>

[![Star](https://img.shields.io/github/stars/your-org/ctifp?style=flat-square&logo=github&color=F97316)](https://github.com/your-org/ctifp)
[![Issues](https://img.shields.io/github/issues/your-org/ctifp?style=flat-square&logo=github&color=0EA5E9)](https://github.com/your-org/ctifp/issues)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-22C55E?style=flat-square)](CONTRIBUTING.md)

</div>
