```mermaid
graph TD
    %% Modern DevOps & AI Engineer Color Palette
    classDef client fill:#37474F,stroke:#546E7A,stroke-width:2px,color:#FFF;
    classDef gateway fill:#1A237E,stroke:#3F51B5,stroke-width:2px,color:#FFF;
    classDef state fill:#263238,stroke:#37474F,stroke-width:2px,color:#00E676;
    classDef orchestrator fill:#1B5E20,stroke:#2E7D32,stroke-width:2px,color:#FFF;
    classDef microservice fill:#E65100,stroke:#F57C00,stroke-width:2px,color:#FFF;
    classDef database fill:#4A148C,stroke:#6A1B9A,stroke-width:2px,color:#FFF;

    %% Step 1: Client Ingress
    Client[Step 1: Alex's Browser<br/>'Provision AWS Sandbox & order a 75k Laptop']:::client -->|HTTPS POST Request + Alex's JWT Token| AuthFilter

    %% 1. Security Gateway Tier (Steps 2-3)
    subgraph GatewayLayer [1. SECURITY & INGRESS TIER: FastAPI Gateway]
        AuthFilter[Step 2: Authentication Filter<br/>- Decodes JWT Signature<br/>- Validates: usr_9921, AI_Eng]:::gateway -->|Valid Identity Claims| RBACEngine[Step 3: RBAC Enforcement Engine<br/>- Checks: execute:it_request<br/>- Checks: execute:finance_request]:::gateway
    end

    %% Step 4: Link Gateway to Central State
    RBACEngine -->|Step 4: Mounts profile to state| StateObject

    %% 2. Cognitive Orchestration Layer (Steps 5-8 & Step 11)
    subgraph LangGraphLayer [2. COGNITIVE ORCHESTRATION TIER: LangGraph Engine]
        StateObject[(Central Graph State<br/>raw_user_request + auth_context)]:::state
        GuardrailsNode[Step 5: Guardrails & Prompt Mgr<br/>- Scans for prompt injections<br/>- Appends corporate instructions]:::orchestrator
        PlannerNode[Step 6: Dynamic Planner Node<br/>- Generates Pydantic DAG Plan<br/>Task 1: AWS Sandbox | Task 2: 75k Laptop]:::orchestrator
        DiscoveryNode[Step 7: Discovery Node<br/>- Reads Task: 'cloud_provisioning'<br/>- Resolves matching microservice endpoint]:::orchestrator
        DispatcherNode[Step 8: A2A Dispatcher Node<br/>- Builds Pydantic JSON-RPC 2.0 payload<br/>- Executes asynchronous network post]:::orchestrator
        ReflectionNode[Step 11: Reflection & Validation Node<br/>- Evaluates output text structures<br/>- Assesses cost/hardware safety thresholds]:::orchestrator
        
        %% Core Graph Control Flow Sequence
        StateObject --> GuardrailsNode
        GuardrailsNode --> PlannerNode
        PlannerNode --> DiscoveryNode
        DiscoveryNode --> DispatcherNode
        DispatcherNode --> ReflectionNode
        
        ReflectionNode -->|Conditional Route: Task Execution Failed / Loop Back to Fix Plan| PlannerNode
    end

    %% 3. Distributed Microservices Tier (Steps 9-10 Execution Core)
    subgraph MicroservicesLayer [3. DISTRIBUTED EXECUTION TIER: Plug-and-Play Microservices]
        HRAgent[HR Agent Microservice<br/>Inactive for this workflow]:::microservice
        ITAgent[Step 9: IT Agent Microservice<br/>Provisions AWS Sandbox Container<br/>Returns Access Credentials]:::microservice
        FinAgent[Step 10b: Finance Agent Microservice<br/>Executes purchase layer after<br/>manager authorization webhook clears]:::microservice
    end

    %% Outbound A2A Tool Dispatches
    DispatcherNode -->|HTTP POST JSON-RPC: cloud_provisioning| ITAgent
    DispatcherNode -->|HTTP POST JSON-RPC: hardware_procurement| FinAgent

    %% 4. Multi-Tier Data & Memory Infrastructure
    subgraph DataLayer [4. ENTERPRISE DATA TIER: Unified PostgreSQL Cluster]
        DBCheckpoint[(Step 10a: Short-Term Memory<br/>- Hits 75k laptop expense limit<br/>- PostgresSaver freezes compute footprint<br/>- Awaits Manager Webhook Signal)]:::database
        DBRegistry[(Agent Capability Registry Table<br/>- Maps 'cloud_provisioning' to ITAgent URL<br/>- Validates Input/Output Pydantic Schemas)]:::database
        DBRAG[(Long-Term Memory / RAG DB<br/>- pgvector Knowledge Base<br/>- Cross-references corporate laptop asset policies)]:::database
    end

    %% Balanced Structural Database Intersections
    StateObject <===>|Durable State Snapshotting| DBCheckpoint
    DiscoveryNode ===>|Capability Lookup Query| DBRegistry
    ReflectionNode ===>|Semantic Policy Match via HNSW Index| DBRAG

    %% Step 12: Exit Strategy
    ReflectionNode -->|Conditional Route: Both Tasks Success + Verification Passed| EndNode([Step 12: END - Secure JSON Response to Client<br/>AWS URLs + Purchase Order Success Details]):::client
