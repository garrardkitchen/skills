# Topic Relationship Map

Use this file when the user wants a conceptual map of how the `.NET` security, architecture, and operational topics relate.

```mermaid
%%{init: {'theme':'base','themeVariables':{
  'primaryColor':'#0f172a',
  'primaryTextColor':'#e2e8f0',
  'primaryBorderColor':'#38bdf8',
  'lineColor':'#94a3b8',
  'secondaryColor':'#1e293b',
  'tertiaryColor':'#111827'
}}}%%
flowchart LR
  classDef core fill:#2563eb,stroke:#1d4ed8,color:#ffffff,stroke-width:2px;
  classDef security fill:#dc2626,stroke:#991b1b,color:#ffffff,stroke-width:2px;
  classDef architecture fill:#7c3aed,stroke:#5b21b6,color:#ffffff,stroke-width:2px;
  classDef data fill:#059669,stroke:#047857,color:#ffffff,stroke-width:2px;
  classDef runtime fill:#ea580c,stroke:#c2410c,color:#ffffff,stroke-width:2px;
  classDef ops fill:#0f766e,stroke:#115e59,color:#ffffff,stroke-width:2px;
  classDef testing fill:#be185d,stroke:#9d174d,color:#ffffff,stroke-width:2px;

  subgraph Core["Core Guidance"]
    Skill["dotnet-agentic-security"]
    OwaspAgentic["OWASP Agentic Top 10"]
    OwaspWeb["OWASP Web Top 10"]
    OwaspApi["OWASP API Security Top 10 2023"]
    Asvs["OWASP ASVS v5.0.0"]
    HttpApis["HTTP API Best Practices"]
  end

  subgraph Security["Identity and Security"]
    Identity["MFA / RBAC / App Roles"]
    AzureIdentity["Azure.Identity"]
    Secrets["Secrets / Key Vault / Rotation"]
    Mcp["MCP Server Security"]
    Headers["CORS / Headers / ProblemDetails"]
  end

  subgraph Architecture["Architecture and Workflows"]
    Clean["Clean Architecture"]
    Solid["SOLID / GoF"]
    Eda["EDA / Outbox / Events"]
    Maf["Agent Framework Workflows"]
    Tenant["Multitenancy / Isolation"]
  end

  subgraph Data["Data and Storage"]
    Ef["EF Core / Code First"]
    Providers["SQLite / Azure SQL / Cosmos"]
    Concurrency["Concurrency Tokens / Transactions"]
    Cache["Redis / Distributed Cache"]
  end

  subgraph Runtime["Runtime and Transport"]
    Workers["Workers / Channels / SemaphoreSlim"]
    Sse["SSE / Streaming"]
    Realtime["SignalR / WebSockets / gRPC"]
    Bus["Service Bus / Webhooks"]
    Containers["Containers / Passwordless Hosting"]
  end

  subgraph Ops["Operations"]
    OTel["OpenTelemetry / Exporters"]
    Resilience["Resilience / Rate Limiting"]
    Health["Health Checks"]
    Docs["README / Changelog"]
  end

  subgraph Testing["Verification"]
    Unit["Unit / Theory / MemberData"]
    Integration["WebApplicationFactory / Testcontainers"]
    SecurityTests["Security Integration Tests"]
  end

  Skill --> OwaspAgentic
  Skill --> OwaspWeb
  Skill --> OwaspApi
  Skill --> Asvs
  Skill --> HttpApis
  OwaspAgentic --> Maf
  OwaspAgentic --> Mcp
  OwaspWeb --> Headers
  OwaspWeb --> Identity
  OwaspApi --> Identity
  OwaspApi --> Tenant
  Asvs --> SecurityTests
  HttpApis --> Headers
  HttpApis --> Identity
  Identity --> AzureIdentity
  AzureIdentity --> Secrets
  Secrets --> Containers
  Clean --> Solid
  Clean --> Eda
  Eda --> Bus
  Maf --> Workers
  Maf --> Sse
  Maf --> Realtime
  Tenant --> Ef
  Ef --> Providers
  Ef --> Concurrency
  Concurrency --> Cache
  Workers --> Resilience
  Bus --> OTel
  Realtime --> OTel
  Containers --> Health
  OTel --> Docs
  Unit --> Integration
  Integration --> SecurityTests
  SecurityTests --> Identity
  SecurityTests --> Mcp
  SecurityTests --> HttpApis
```

## Reading hints

- security topics reinforce both API and agentic guidance
- architecture topics shape how security and testing are applied
- runtime topics describe transport and orchestration surfaces
- operations and testing prove the system works safely in practice
- OWASP API Security Top 10 2023 and OWASP ASVS v5.0.0 should drive concrete implementation checks, not just awareness
