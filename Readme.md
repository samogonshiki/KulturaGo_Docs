# Высокоуровневая архитектура

```mermaid
%% KulturaGo — high-level deployment
flowchart LR
  subgraph Client Tier
    A[Mobile / Web<br>React + Flutter] -->|HTTPS| G(NGINX (API Gateway))
  end

  subgraph Observability
    P[Prometheus] -.-> Gf[Grafana]
  end

  Gf --- G          %% чтобы показать связь мониторинга с шлюзом

  subgraph Kubernetes Cluster
    direction TB
    G -->|gRPC&nbsp;/&nbsp;REST| Auth[auth-service]
    G --> Ticket[ticketing-service]
    G --> Event[event-service]
    G --> Notify[notification-service]
    G --> Gid[gid-service]
    G --> Search[search-service]
    G --> Rec[recommendation-service]
    G --> Pay[payment-service]
  end

  subgraph Messaging
    K[(Apache Kafka)]
  end
  K <-->|event-driven| Auth
  K <-->|event-driven| Ticket
  K <-->|event-driven| Event
  K <-->|event-driven| Notify
  K <-->|event-driven| Gid
  K <-->|event-driven| Search
  K <-->|event-driven| Rec
  K <-->|event-driven| Pay
```

```mermaid
@startuml
!theme spacelab

' ─── Nodes ──────────────────────────────────────────────────────────────
node "Mobile / Web\n(React • Flutter)" as Client
node "NGINX\n(API Gateway)" as Nginx
cloud "Prometheus" as Prom
database "Grafana" as Graf
cloud "Apache Kafka" as Kafka

package "Kubernetes Cluster" {
  [auth-service] as Auth
  [event-service] as Event
  [ticketing-service] as Ticket
  [notification-service] as Notify
  [gid-service] as Gid
  [search-service] as Search
  [recommendation-svc] as Rec
  [payment-service] as Pay
}

' ─── Edges ───────────────────────────────────────────────────────────────
Client --> Nginx : HTTPS
Nginx --> Auth    : gRPC / REST
Nginx --> Event
Nginx --> Ticket
Nginx --> Notify
Nginx --> Gid
Nginx --> Search
Nginx --> Rec
Nginx --> Pay

Prom ..> Graf : metrics ▶ dashboards
Graf ..> Nginx : latency, 5xx
Auth  --> Kafka
Event --> Kafka
Ticket --> Kafka
Notify --> Kafka
Gid   --> Kafka
Search --> Kafka
Rec   --> Kafka
Pay   --> Kafka
Kafka --> Auth
Kafka --> Event
Kafka --> Ticket
Kafka --> Notify
Kafka --> Gid
Kafka --> Search
Kafka --> Rec
Kafka --> Pay
@enduml
```