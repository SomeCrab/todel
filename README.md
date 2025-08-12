```mermaid
graph TD
  U[User (HTTPS)] -->|GET https://api.wiru.site| DNS[DNS]
  DNS -->|A record to server IP| ING[NGINX Ingress]
  ING -->|host api.wiru.site| SVC_BE[Service backend (8000)]
  SVC_BE --> POD_BE[Pod backend (gunicorn)]
  POD_BE -->|/static| WN[WhiteNoise]
  POD_BE -->|ORM| SVC_PG[Service postgres (5432)]
  SVC_PG --> PG[Pod Postgres]

  %% Outbound mail
  POD_BE -->|SMTP 25| SVC_PF[Service postfix]
  SVC_PF --> PF[Pod Postfix]
  PF -->|DKIM + deliver| MX[Recipient MX (Internet)]
  MX -->|bounces (DSN)| STRATO[MX STRATO for wiru.site]

  %% Probes
  KUBE[Kubelet probes] -.->|GET /health (Host api.wiru.site)| POD_BE

  %% Cluster boundary
  subgraph K8s [Kubernetes (MicroK8s)]
    ING
    SVC_BE
    POD_BE
    SVC_PG
    PG
    SVC_PF
    PF
  end
```

