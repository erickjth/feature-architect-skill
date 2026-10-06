# Diagram snippets (Mermaid)

Use `<pre class="mermaid">` in HTML. Keep labels short; max ~12 nodes per diagram; one caption line under each. Color code: new = green, changed = amber, existing = gray via `classDef`.

## Use cases (actors -> UC)
```mermaid
flowchart LR
  U([Customer]) --> UC1["UC-1 Book appointment"]
  U --> UC2["UC-2 Cancel appointment"]
  P([Provider]) --> UC3["UC-3 Confirm booking"]
  UC1 -.includes.-> UC4["Check availability"]
```

## Context
```mermaid
flowchart LR
  User([User]) --> App["Feature / App"]
  App --> Pay[(Payment API)]
  App --> Mail[(Email provider)]
  CRM[(Upstream CRM)] --> App
```

## Component map with change markers
```mermaid
flowchart LR
  FE["Frontend"] --> API["API"]
  API --> Core["Core module"]
  Core --> DB[(Database)]
  Core --> Q[[Queue]]
  classDef new fill:#d1fae5,stroke:#059669,color:#064e3b
  classDef changed fill:#fef3c7,stroke:#d97706,color:#78350f
  classDef existing fill:#f3f4f6,stroke:#9ca3af,color:#374151
  class Q new
  class Core changed
  class FE,API,DB existing
```

## Building blocks - ports and adapters
```mermaid
flowchart LR
  subgraph Driving["Driving adapters"]
    REST["REST view"]
    Job["Queue consumer"]
  end
  subgraph Core["Domain core"]
    UC["BookAppointment (use case)"]
    Ent["Appointment (entity)"]
    PR{{"AppointmentRepository (port)"}}
  end
  subgraph Driven["Driven adapters"]
    ORM["ORM repo"]
    Fake["In-memory fake"]
  end
  REST --> UC
  Job --> UC
  UC --> Ent
  UC --> PR
  ORM -.implements.-> PR
  Fake -.implements.-> PR
```

## Runtime sequence (with failure path)
```mermaid
sequenceDiagram
  actor U as User
  participant A as API
  participant S as UseCase
  participant R as Repo
  participant X as Payment
  U->>A: POST /bookings
  A->>S: execute(cmd)
  S->>R: save(pending)
  S->>X: charge (timeout 3s, idempotency key)
  alt ok
    X-->>S: paid
    S->>R: mark confirmed
    S-->>A: 201
  else timeout
    S->>R: mark pending-retry
    S-->>A: 202
  end
```

## State
```mermaid
stateDiagram-v2
  [*] --> Pending
  Pending --> Confirmed: payment ok
  Pending --> Failed: retries exhausted
  Confirmed --> Cancelled: user cancels
```

## Data (ER)
```mermaid
erDiagram
  CLIENT ||--o{ BOOKING : makes
  BOOKING }o--|| SERVICE : for
  BOOKING { uuid id PK
    uuid client_id FK
    string status
    timestamp starts_at }
```

## Deployment
```mermaid
flowchart TB
  CDN["CDN"] --> LB["Load balancer"]
  subgraph VPC["VPC"]
    LB --> W1["App container x N"]
    W1 --> DB[("Postgres primary")]
    DB --> RO[("Read replica")]
    W1 --> C[("Redis cache")]
    W1 --> WK["Worker container"]
  end
```

## Phase plan
```mermaid
gantt
  dateFormat YYYY-MM-DD
  section MVP
  Core flow (UC-1..2) :a1, 2026-01-05, 7d
  section Post-launch
  Refactor + debt     :a2, after a1, 5d
```
