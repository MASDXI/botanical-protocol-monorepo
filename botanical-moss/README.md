# botanical-moss

## Overview Sequence Diagram

``` mermaid
sequenceDiagram
    autonumber
    participant HF as Hyperledger Fabric (garden)
    participant P as Producer (moss:listener)
    participant AP as Apache Pulsar (moss:broker)
    participant M as Go Worker (moss:worker)
    participant R as Redis (moss:cache)
    participant DB as PostgreSQL (moss:database)

    HF->>P: Emits ERC-8047 Compat. Transfer Event
    P->>AP: Publishes event payload to topic
    AP-->>P: ACK (Message safely persisted)
    AP->>M: Delivers to exactly 1 worker
    M->>R: Fetch current balance
    alt Cache Miss (Key expired or First Event)
        M->>DB: Fetch last known balance
        DB-->>M: Return balance (or 0)
    end
    M->>M: Consolidate (Current + Transfer Amount)
    par Dual Write
        M->>R: SET new balance (TTL `n` days)
        M->>DB: UPSERT new balance
    end
    M->>AP: ACK Message (Safe to delete)
```