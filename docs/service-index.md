# Service Index

Complete index of all services in the Voyager architecture with quick reference information.

## Actors/Users

| Actor | Domain | Description | Primary Responsibilities |
|-------|--------|-------------|-------------------------|
| [Automation Designer](actors/automation-designer.md) | Default | Process automation architect | Design and architect automation workflows |
| [Automation Troubleshooter](actors/automation-troubleshooter.md) | Default | Process diagnostics specialist | Diagnose and resolve automation issues |
| [New Customer](actors/new-customer.md) | Default | Solution adopter and end user | Use and adopt Voyager automation solutions |

## Execution Services

| Service | Namespace | Technology | Purpose | APIs |
|---------|-----------|------------|---------|------|
| [Jarvis](execution-services/jarvis.md) | `jarvis` | Rust, KEDA | SPy program execution engine (Python subset) | Kafka (`spy_control`) |
| [Jeeves](execution-services/jeeves.md) | `jeeves` | Rust | Run orchestration and metadata | gRPC, REST |
| [Guidance Center](execution-services/guidance.md) | `guidance` | Rust, LLM | Complex exception escalation hub | gRPC, REST |

## Data Services

| Service | Namespace | Technology | Purpose | APIs |
|---------|-----------|------------|---------|------|
| [Business Journal](data-services/journal.md) | `journal` | Rust, InfluxDB, Telegraf | Fact persistence and dynamic querying | gRPC, REST |

## Process Management

| Service | Namespace | Technology | Purpose | APIs |
|---------|-----------|------------|---------|------|
| [Grimoire](process-management/grimoire.md) | `grimoire` | Rust, PostgreSQL, LLM | English process specifications and AI design | gRPC, REST, WebSocket |

## Book Services

| Service | Namespace | Technology | Purpose | APIs |
|---------|-----------|------------|---------|------|
| [Books](book-services/books.md) | `books` | Python, K8s CRDs | Python skill collections for 3rd party integration | Internal |

## Chat & Agent Services

| Service | Namespace | Technology | Purpose | APIs |
|---------|-----------|------------|---------|------|
| [Threads](chat-agent-services/threads.md) | `threads` | Rust, PostgreSQL, AI Agents | AI-powered chat orchestration | gRPC, REST, WebSocket |
| [Process Designer](chat-agent-services/process-designer.md) | `threads` | Python, AI/ML | Process design | gRPC, REST |
| [Spy Mapper](chat-agent-services/spy-mapper.md) | `threads` | Python, AI/ML | SPy program (Python subset) analysis | gRPC, REST |
| [Spy Writer](chat-agent-services/spy-writer.md) | `threads` | Python, AI/ML | SPy program (Python subset) generation | gRPC, REST |

## Integration Services

| Service | Namespace | Technology | Purpose | APIs |
|---------|-----------|------------|---------|------|
| [Triage](integration-services/triage.md) | `triage` | Rust | First responder for execution exceptions | gRPC (internal) |

## Platform Services

| Service | Namespace | Technology | Purpose | APIs |
|---------|-----------|------------|---------|------|
| [Envoy Gateway](platform-services/envoy-gateway.md) | `envoy-gateway-system` | Envoy | Public edge and routing | HTTP, WebSocket |
| [User & Org Manager](platform-services/user-org-manager.md) | `user` | Rust, PostgreSQL | Identity and authorization | gRPC, REST |
| [BDK](platform-services/bdk.md) | `bdk` | Rust, K8s Controllers | Python Book skill collection management | gRPC, REST |
| [LLM Services](platform-services/llm-services.md) | `llm` | Rust, LiteLLM, Phoenix | Model and prompt management | gRPC, REST |
| [Bifrost](platform-services/bifrost.md) | `bifrost` | Rust, Tailscale | Network management | gRPC, REST |
| [Object Service](platform-services/object-service.md) | `objects` | Rust | Artifact storage and signed URLs | gRPC, REST |

## Customer Deployments

| Service | Namespace | Technology | Purpose | APIs |
|---------|-----------|------------|---------|------|
| [Customer Books](platform-services/customer-deployments.md) | `customer-*` | Python, Tailscale | Customer-specific Book skill deployments | Internal |

## Quick Reference

### By Technology Stack

#### Rust Services
- Jarvis (SPy Execution - Python subset)
- Jeeves (Run Orchestration)
- Business Journal (Fact Persistence)
- Grimoire (English Process Specifications)
- Threads (AI Chat Orchestration)
- Guidance Center (Complex Exception Escalation Hub)
- Triage (First Responder for All Exceptions)
- UOM (Identity)
- BDK (Python Book Skill Collections)
- Bifrost (Network Management)
- LLM Services (AI Models)
- Object Service (Artifact Storage)

#### Python Services
- Books (Shared skill collections for 3rd party integration)
- Customer Book Deployments (Customer-specific skill collections)
- Process Designer (Process design)
- Spy Mapper (SPy program analysis - Python subset)
- Spy Writer (SPy program generation - Python subset)

#### Database Services
- PostgreSQL: Jeeves, Grimoire, Threads, UOM
- InfluxDB: Business Journal (fact persistence)
- Redis: Caching across services

#### Infrastructure Services
- Envoy Gateway (Routing)
- Kafka (Event streaming)
- KEDA (Autoscaling)
- Object Service (Artifact storage)
- Tailscale (Networking)

### By Data Flow

#### Execution Path
1. Client → Envoy Gateway → UOM (auth) → Jeeves → Jarvis → Kafka → Business Journal

#### Exception Handling Path
1. Jarvis → Kafka → Guidance Center → Automation Troubleshooter

#### Chat Path
1. Client → Envoy Gateway → Agents → Threads → WebSocket stream

#### Process Management Path
1. Client → Envoy Gateway → Grimoire → Process Database (English specifications)

#### External Integration Path
1. External System → BDK → Jeeves → Jarvis

#### Exception Handling Path
1. Execution Exception → Triage (Automated Recovery) → Guidance (Human Intervention)

### API Patterns

#### Standard Pattern
- **gRPC**: Core service API
- **Envoy Transcoder**: gRPC ↔ REST translation
- **REST via Gateway**: External HTTP API
- **WebSocket**: Real-time features (where applicable)

#### Internal Only
- Kafka message consumption
- Internal gRPC calls
- Database access

### Security Levels

#### Public APIs (via Gateway)
- Authentication required (JWT)
- Authorization via UOM
- Rate limiting applied

#### Internal APIs
- Service mesh communication
- No external access
- Inter-service auth

#### Customer Isolated
- Namespace-level isolation
- Dedicated credentials
- Tailscale network access
