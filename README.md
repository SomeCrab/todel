<details>
    <summary>Graph</summary>
```mermaid
graph TD
    U[User (HTTPS)] -->|GET https://api.wiru.site| DNS[(DNS)]
    DNS -->|A → server IP| ING[NGINX Ingress Controller]

    TLS[[TLS: wildcard *.wiru.site]] --> ING
    ING -->|host=api.wiru.site| SVC_BE[Service: backend :8000]
    SVC_BE --> POD_BE[(Pod: backend / gunicorn)]
    POD_BE -->|/static| WN[WhiteNoise (staticfiles)]
    POD_BE -->|ORM| SVC_PG[Service: postgres :5432]
    SVC_PG --> PG[(Pod: Postgres)]

    %% Outbound mail
    POD_BE -->|SMTP :25| SVC_PF[Service: postfix]
    SVC_PF --> PF[(Pod: Postfix)]
    PF -->|DKIM sign + deliver| MX[(Recipient MX on the Internet)]
    MX -->|Bounce (DSN)| STRATO[MX: STRATO for wiru.site]

    %% Probes
    KUBE[Kubelet probes] -.->|GET /health<br/>Host: api.wiru.site| POD_BE

    %% Cluster boundary
    subgraph K8s[ Kubernetes (MicroK8s) ]
      ING
      SVC_BE
      POD_BE
      SVC_PG
      PG
      SVC_PF
      PF
    end

    %% Styles
    classDef db fill:#99f,stroke:#333;
    class PG,SVC_PG db;
    classDef ingress fill:#bff,stroke:#333;
    class ING ingress;
    classDef svc fill:#fff,stroke:#333;
    class SVC_BE,SVC_PF svc;
    classDef pod fill:#eee,stroke:#333;
    class POD_BE,PF pod;
    classDef ext fill:#ffe,stroke:#333;
    class STRATO,MX ext;
    classDef note fill:#f6f6f6,stroke:#999,stroke-dasharray: 3 3;
    class TLS,WN,KUBE note;

```
</details>
