# Enterprise Service Operations Platform (`a2a-engine`)

The Core Orchestrator Engine is a modular, production-grade enterprise platform designed to dynamically coordinate specialized AI agents to automate business requests across various corporate domains. By leveraging **LangGraph state machines**, an **Agent-to-Agent (A2A) network layer**, and a **multi-tiered PostgreSQL memory ecosystem**, the architecture ensures that execution logic remains decoupled from specific agent code paths, enabling dynamic runtime scalability.

This architecture guide uses Alex’s request (*"Book Conference Room A for tomorrow and order 10 premium lunches..."*) to explain the underlying engineering concepts, the exact runtime data flow, and the reasoning behind these permanent technology choices.

---

## 🏗️ Technical Architecture Topology

```mermaid
graph TD
    %% Modern DevOps & AI Engineer Color Palette
    classDef client fill:#37474F,stroke:#546E7A,stroke-width:2px,color:#FFF;
    classDef gateway fill:#1A237E,stroke:#3F51B5,stroke-width:2px,color:#FFF;
    classDef state fill:#263238,stroke:#37474F,stroke-width:2px,color:#00E676;
    classDef orchestrator fill:#1B5E20,stroke:#2E7D32,stroke-width:2px,color:#FFF;
    classDef microservice fill:#E65100,stroke:#F57C00,stroke-width:2px,color:#FFF;
    classDef database fill:#4A148C,stroke:#6A1B9A,stroke-width:2px,color:#FFF;

    %% Ingress Trigger
    Client[Client Browser / Device<br/>Alex: Book Room A + 10 Lunches]:::client -->|HTTP POST + JWT Token| AuthFilter

    %% Tier 1: Security & Ingress Tier
    subgraph GatewayLayer [TIER 1: SECURITY & INGRESS TIER - FastAPI Gateway]
        AuthFilter[1. Ingress & Auth Filter<br/>- Decodes JWT Signature & exp<br/>- Extracts user identities]:::gateway -->|Verified Claims| RBACEngine[2. RBAC Enforcement Engine<br/>- Validates execute:room_booking<br/>- Validates execute:expense_procurement]:::gateway
    end

    %% Gateway to State Transition
    RBACEngine -->|Mounts context to runtime| StateObject

    %% Tier 2: Cognitive Orchestration Tier
    subgraph LangGraphLayer [TIER 2: COGNITIVE ORCHESTRATION TIER - LangGraph Engine]
        StateObject[(3. Central Graph State<br/>EnterpriseOrchestrationState)]:::state
        GuardrailsNode[4. Guardrails & Prompt Mgr<br/>- Scans for prompt injection<br/>- Applies LangChain template]:::orchestrator
        PlannerNode[5. Dynamic Planner Node<br/>- Generates Pydantic DAG Plan<br/>Task 1: Room A, Task 2: 10 Lunches]:::orchestrator
        DiscoveryNode[6. Discovery Node<br/>- Runtime Directory-Resolver<br/>- Resolves capability via SQLAlchemy]:::orchestrator
        DispatcherNode[7. A2A Dispatcher Node<br/>- Packages JSON-RPC 2.0 payloads<br/>- Fires Asynchronous HTTPX calls]:::orchestrator
        ReflectionNode[12. Reflection & Validation Node<br/>- Semantic Self-Auditing Gate<br/>- Checks Compliance Metrics]:::orchestrator
        
        %% Core Graph Logic Loop Pathways
        StateObject --> GuardrailsNode
        GuardrailsNode --> PlannerNode
        PlannerNode --> DiscoveryNode
        DiscoveryNode --> DispatcherNode
        DispatcherNode --> ReflectionNode
        
        ReflectionNode -->|Conditional Route: Validation Failed - Re-Plan Loop| PlannerNode
    end

    %% Tier 3: Distributed Execution Tier
    subgraph MicroservicesLayer [TIER 3: DISTRIBUTED EXECUTION TIER - Plug-and-Play Microservices]
        HRAgent[8. HR Agent Microservice<br/>Hosts: /.well-known/agent-card.json<br/>Status: Inactive for this loop]:::microservice
        ITAgent[8. Facilities Agent Microservice<br/>Locks Room A calendar slot<br/>Returns booking_id: BK-991A]:::microservice
        FinAgent[8. Finance Agent Microservice<br/>Processes catering payment code<br/>Clears vendor contract ledger]:::microservice
    end

    %% Outbound A2A Dispatches
    DispatcherNode -->|Async HTTP POST JSON-RPC 2.0| ITAgent
    DispatcherNode -->|Async HTTP POST JSON-RPC 2.0| FinAgent

    %% Tier 4: Enterprise Data Tier
    subgraph DataLayer [TIER 4: ENTERPRISE DATA TIER - Unified PostgreSQL Cluster]
        DBCheckpoint[(4a. Short-Term Memory Checkpoint<br/>- 6,000 INR cost hits 5k threshold<br/>- PostgresSaver snapshot hold<br/>- Awaits Manager Webhook Signal)]:::database
        DBRegistry[(10. Agent Registry Table<br/>- Indexes capability maps<br/>- Stores dynamic schema entries)]:::database
        DBRAG[(11. Long-Term Memory & RAG<br/>- pgvector database cluster<br/>- HNSW semantic search index)]:::database
    end

    %% Balanced Transactional Database Intersections
    StateObject <===>|Durable Checkpoint Tracking| DBCheckpoint
    DiscoveryNode ===>|Read Capability Index| DBRegistry
    ReflectionNode ===>|Semantic SOP Lookup| DBRAG

    %% Exit Strategy Flow
    ReflectionNode -->|Conditional Route: Validation Passed| EndNode([13. Resolution Exit Pipeline<br/>Returns Serialized Pydantic JSON]):::client
