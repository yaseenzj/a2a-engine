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
    Client[Step 1: Alex's Portal App<br/>'Book Conference Room A and order 10 lunches']:::client -->|HTTPS POST Request + Alex's JWT Token| AuthFilter

    %% 1. Security Gateway Tier (Steps 2-3)
    subgraph GatewayLayer [1. SECURITY & INGRESS TIER: FastAPI Gateway]
        AuthFilter[Step 2: Authentication Filter<br/>- Decodes JWT Digital Token<br/>- Validates Identity: Alex, Sales Team]:::gateway -->|Valid Identity Approved| RBACEngine[Step 3: RBAC Enforcement Engine<br/>- Checks: permission to book rooms<br/>- Checks: permission to spend budget]:::gateway
    end

    %% Step 4: Link Gateway to Central State
    RBACEngine -->|Step 4: Mounts Alex's profile to state| StateObject

    %% 2. Cognitive Orchestration Layer (Steps 5-8 & Step 11)
    subgraph LangGraphLayer [2. COGNITIVE ORCHESTRATION TIER: LangGraph Engine]
        StateObject[(Central Graph State<br/>raw_user_request + auth_context)]:::state
        GuardrailsNode[Step 5: Guardrails & Prompt Mgr<br/>- Screens out profanity or malicious text<br/>- Attaches strict execution instructions]:::orchestrator
        PlannerNode[Step 6: Dynamic Planner Node<br/>- Generates Pydantic DAG Plan<br/>Task 1: Book Room A + Task 2: Order 10 Lunches]:::orchestrator
        DiscoveryNode[Step 7: Discovery Node<br/>- Reads Task Requirement: 'room_booking'<br/>- Looks up target microservice web address]:::orchestrator
        DispatcherNode[Step 8: A2A Dispatcher Node<br/>- Formats clean JSON-RPC 2.0 message payload<br/>- Fires async network call to the worker]:::orchestrator
        ReflectionNode[Step 11: Reflection & Validation Node<br/>- Double-checks confirmation codes text<br/>- Assesses expense compliance rules]:::orchestrator
        
        %% Core Graph Control Flow Sequence
        StateObject --> GuardrailsNode
        GuardrailsNode --> PlannerNode
        PlannerNode --> DiscoveryNode
        DiscoveryNode --> DispatcherNode
        DispatcherNode --> ReflectionNode
        
        ReflectionNode -->|Conditional Route: Booking Failed / Loop Back to Choose Room B| PlannerNode
    end

    %% 3. Distributed Microservices Tier (Steps 9-10 Execution Core)
    subgraph MicroservicesLayer [3. DISTRIBUTED EXECUTION TIER: Plug-and-Play Microservices]
        HRAgent[HR Agent Microservice<br/>Left completely idle for this request]:::microservice
        ITAgent[Step 9: Facilities Agent Microservice<br/>Locks Room A calendar slot<br/>Generates digital calendar invite code]:::microservice
        FinAgent[Step 10b: Finance Agent Microservice<br/>Processes lunch payment to caterer<br/>after manager clicks approval link]:::microservice
    end

    %% Outbound A2A Tool Dispatches
    DispatcherNode -->|HTTP POST JSON-RPC: room_booking| ITAgent
    DispatcherNode -->|HTTP POST JSON-RPC: expense_procurement| FinAgent

    %% 4. Multi-Tier Data & Memory Infrastructure
    subgraph DataLayer [4. ENTERPRISE DATA TIER: Unified PostgreSQL Cluster]
        DBCheckpoint[(Step 10a: Short-Term Memory<br/>- Lunch bill exceeds 5000 INR automatic cap<br/>- PostgresSaver pauses and saves graph state<br/>- Pauses compute until manager approves expense)]:::database
        DBRegistry[(Agent Capability Registry Table<br/>- Links 'room_booking' to Facilities Agent URL<br/>- Houses validation input/output schemas)]:::database
        DBRAG[(Long-Term Memory / RAG DB<br/>- pgvector database store<br/>- Cross-references corporate lunch budget policies)]:::database
    end

    %% Balanced Structural Database Intersections
    StateObject <===>|Saves active thread checkpoints| DBCheckpoint
    DiscoveryNode ===>|Queries API endpoint address directory| DBRegistry
    ReflectionNode ===>|Checks catering policy text via HNSW index| DBRAG

    %% Step 12: Exit Strategy
    ReflectionNode -->|Conditional Route: Room Locked + Lunch Paid + Quality Checked| EndNode([Step 12: END - Clean JSON Summary Response<br/>Room A Booking ID + Catering Receipt Details]):::client
