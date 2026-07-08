```mermaid
graph TD
    %% Modern DevOps & AI Engineer Color Palette
    classDef client fill:#37474F,stroke:#546E7A,stroke-width:2px,color:#FFF;
    classDef gateway fill:#1A237E,stroke:#3F51B5,stroke-width:2px,color:#FFF;
    classDef state fill:#263238,stroke:#37474F,stroke-width:2px,color:#00E676;
    classDef orchestrator fill:#1B5E20,stroke:#2E7D32,stroke-width:2px,color:#FFF;
    classDef microservice fill:#E65100,stroke:#F57C00,stroke-width:2px,color:#FFF;
    classDef database fill:#4A148C,stroke:#6A1B9A,stroke-width:2px,color:#FFF;

    %% Ingress Tier
    Client[Customer Device / Browser]:::client -->|1. HTTPS Request + JWT Token| AuthFilter

    %% 1. Security Gateway Tier
    subgraph GatewayLayer [1. SECURITY & INGRESS TIER: FastAPI Gateway]
        AuthFilter[Authentication Filter<br/>- Decodes JWT Claims<br/>- Validates Signature & 'exp']:::gateway -->|Verified Claims| RBACEngine[RBAC Enforcement Engine<br/>- Assesses Roles & Scopes<br/>- Enforces Permission Vectors]:::gateway
    end

    %% Link Gateway to Central State
    RBACEngine -->|2. Injects Validated Context| StateObject

    %% 2. Cognitive Orchestration Layer (LangGraph ReAct Pipeline)
    subgraph LangGraphLayer [2. COGNITIVE ORCHESTRATION TIER: LangGraph Engine]
        StateObject[(Central Graph State<br/>EnterpriseOrchestrationState)]:::state
        GuardrailsNode[Guardrails & Prompt Mgr<br/>- Prompt Management Templates<br/>- Input Validation Filtering]:::orchestrator
        PlannerNode[Dynamic Planner Node<br/>- Parses Intent via LLM<br/>- Builds Structured Pydantic DAG]:::orchestrator
        DiscoveryNode[Discovery Node<br/>- Extracts Task Capabilities<br/>- Performs Live Registry Query]:::orchestrator
        DispatcherNode[A2A Dispatcher Node<br/>- Packages Payloads into JSON-RPC 2.0<br/>- Network Execution Engine]:::orchestrator
        ReflectionNode[Reflection & Validation Node<br/>- Quality Evaluation Spans<br/>- Inspects Output Formats & Conf]:::orchestrator
        
        %% Core Graph Control Flow Loops
        StateObject --> GuardrailsNode
        GuardrailsNode --> PlannerNode
        PlannerNode --> DiscoveryNode
        DiscoveryNode --> DispatcherNode
        DispatcherNode --> ReflectionNode
        
        ReflectionNode -->|Conditional Route: Verification Failed / Re-Plan Loop| PlannerNode
    end

    %% 3. Distributed Microservices Tier
    subgraph MicroservicesLayer [3. DISTRIBUTED EXECUTION TIER: Plug-and-Play Microservices]
        HRAgent[HR Agent Microservice<br/>Hosts: /.well-known/agent-card.json]:::microservice
        ITAgent[IT Support Microservice<br/>Hosts: /.well-known/agent-card.json]:::microservice
        FinAgent[Finance Agent Microservice<br/>Hosts: /.well-known/agent-card.json]:::microservice
    end

    %% Outbound A2A Dispatches
    DispatcherNode -->|HTTP POST JSON-RPC 2.0| HRAgent
    DispatcherNode -->|HTTP POST JSON-RPC 2.0| ITAgent
    DispatcherNode -->|HTTP POST JSON-RPC 2.0| FinAgent

    %% 4. Multi-Tier Data & Memory Infrastructure
    subgraph DataLayer [4. ENTERPRISE DATA TIER: Unified PostgreSQL Cluster]
        DBCheckpoint[(Short-Term Memory<br/>- LangGraph PostgresSaver<br/>- Human-in-the-Loop Checkpoints)]:::database
        DBRegistry[(Agent Registry Table<br/>- Tracks Capability Mappings<br/>- Stores Active JSON Schemas)]:::database
        DBRAG[(Long-Term Memory<br/>- pgvector Knowledge Base<br/>- Profile Preferences & HNSW)]:::database
    end

    %% Balanced Structural Database Intersections
    StateObject <===>|Atomic State Tracking| DBCheckpoint
    DiscoveryNode ===>|Read Capability Index| DBRegistry
    ReflectionNode ===>|Semantic Context Lookup| DBRAG

    %% Exit Strategy
    ReflectionNode -->|Conditional Route: Verification Passed| EndNode([END: Secure JSON Response to Gateway]):::client
