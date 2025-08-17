# Voyager Architecture Documentation

This documentation provides a comprehensive overview of the Voyager microservices architecture deployed across Kubernetes namespaces.

## 📁 Documentation Structure

- **[Architecture Overview](architecture-overview.md)** - High-level system architecture and service organization
- **[Service Index](service-index.md)** - Complete index of all services with quick references

### Service Categories

#### 🚀 [Execution Services](execution-services/)
Core execution engine and orchestration
- [Jarvis](execution-services/jarvis.md) - SPy program execution engine (Python subset)
- [Jeeves](execution-services/jeeves.md) - Run orchestration and execution metadata
- [Guidance Center](execution-services/guidance.md) - Complex exception escalation hub for human troubleshooters

#### 💾 [Data Services](data-services/)
Fact processing, storage, and querying
- [Business Journal](data-services/journal.md) - Fact persistence in InfluxDB and dynamic querying

#### 📋 [Process Management](process-management/)
Process definitions and management
- [Grimoire](process-management/grimoire.md) - English process specifications and AI-powered design

#### 📚 [Book Services](book-services/)
Python skill collections for third-party integrations
- [Books](book-services/books.md) - Shared Python skill collections for 3rd party integration

#### 💬 [Chat & Agent Services](chat-agent-services/)
Conversational interfaces and agent management
- [Threads](chat-agent-services/threads.md) - Chat conversation persistence and Process management
- [Process Designer](chat-agent-services/process-designer.md) - Process design
- [SPy Mapper](chat-agent-services/spy-mapper.md) - SPy program (Python subset) analysis and mapping
- [SPy Writer](chat-agent-services/spy-writer.md) - SPy program (Python subset) generation

#### 🔗 [Integration Services](integration-services/)
External event processing and exception handling
- [Triage](integration-services/triage.md) - First responder for all execution exceptions with automated recovery

#### 🏗️ [Platform Services](platform-services/)
Authentication, authorization, and system utilities
- [Envoy Gateway](platform-services/envoy-gateway.md) - Public edge with authentication and routing
- [User & Organization Manager](platform-services/user-org-manager.md) - Identity and authorization
- [BDK](platform-services/bdk.md) - Python Book skill collection management
- [LLM Services](platform-services/llm-services.md) - Model and prompt management
- [Bifrost/Network Manager](platform-services/bifrost.md) - Secure customer network access
- [Object Service](platform-services/object-service.md) - Artifact storage and signed URLs

#### 🚢 [Deployment](deployment/)
Infrastructure and deployment documentation
- [Kubernetes Deployment](deployment/kubernetes.md) - Namespace organization and deployment patterns
- [Infrastructure](deployment/infrastructure.md) - AWS, networking, and external systems
- [Security](deployment/security.md) - Security patterns and considerations

## 🎯 Quick Navigation

### By Namespace
- `jarvis` - SPy program execution
- `jeeves` - Run orchestration
- `journal` - Fact querying and storage
- `grimoire` - Process management
- `books` - Shared book implementations
- `threads` - Chat persistence
- `objects` - Artifact storage and signed URLs
- `triage` - Exception handling
- `envoy-gateway-system` - Public edge
- `user` - Identity management
- `bdk` - Book Development Kit
- `llm` - Model/prompt management
- `bifrost` - Network management
- `vault` - Secret management
- `customer-*` - Customer deployments

### By Technology
- **Rust Services**: Jarvis, Jeeves, Grimoire, BDK
- **PostgreSQL Databases**: Jeeves, Grimoire, Threads, UOM
- **InfluxDB**: Journal time-series storage
- **Kafka**: Event streaming backbone
- **Envoy**: Gateway and protocol translation
- **Python Books**: Customer book deployments

### Key Patterns
- **gRPC + REST**: All services expose gRPC with Envoy transcoding to REST
- **WebSocket Support**: Real-time features via gateway WebSocket routing
- **CRD-Based**: Book definitions, model definitions, and evaluations
- **Controller Pattern**: Automated lifecycle management
- **Service Mesh**: Inter-service communication via Kubernetes networking

## 🔍 Finding Information

Use the service index for quick lookups, or browse by category for detailed documentation. Each service document includes:
- Purpose and responsibilities
- API specifications (gRPC and REST)
- Data flow diagrams where applicable
- Infrastructure requirements
- Integration points
- Security considerations

## 🏛️ Architecture Principles

- **Microservices**: Each service has a single responsibility
- **Event-Driven**: Kafka-based event streaming for loose coupling
- **Kubernetes-Native**: CRDs, controllers, and cloud-native patterns
- **Security-First**: Authentication, authorization, and encrypted communication
- **Observability**: Comprehensive logging, metrics, and tracing
