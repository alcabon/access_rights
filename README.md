# access_rights

```mermaid

flowchart TD
    A[Org-Wide Default] 
      -->|Private| B[OWD <b>Private</b>]
    A 
      -->|Public Read/Write
or Read-Only| C[OWD <b>Public</b>]

    B --> D{Need to grant additional access?}
    D -->|Yes| E[Sharing Rules]
    D -->|No| S[Only owners & hierarchy see records]

    E --> F{Type of Rule}
    F --> G[Owner-Based]
    F --> H[Criteria-Based]
    E --> I[Manual Sharing or Apex-Managed Sharing]
    E --> J[Targets:
Roles, Territories,
Public Groups]

    C --> K{Need to restrict access?}
    K -->|Yes| L{Object supports <b>Restriction Rules</b>?}
    K -->|No| R[All records visible to users]

    L -->|Yes| M[Define Restriction Rules]
    M --> N[Filter by criteria
& apply to <b>Profiles/Perm Sets</b>]

    L -->|No| O{Options for restriction}
    O --> P[Change OWD to
<b>Private</b> + use
<b>Sharing Rules</b>]
    O --> Q[Build <b>Custom Apex REST</b>
with sharing enforced]

    style S fill:#f9f,stroke:#333,stroke-width:2px
    style R fill:#f9f,stroke:#333,stroke-width:2px


```
