                             ┌────────────────────┐
Client (Post /migrate) ────>│   Load Balancer     │
                             └────────┬───────────┘
                                      │
         ┌────────────────────────────┼────────────────────────────┐
         ↓                            ↓                            ↓
   ┌────────────┐              ┌────────────┐              ┌────────────┐
   │   Pod 1    │              │   Pod 2    │              │   Pod 3    │
   │ Spring Boot│              │ Spring Boot│              │ Spring Boot│
   │ API Layer  │              │ API Layer  │              │ API Layer  │
   │            │              │            │              │            │
   │ Async: 10  │              │ Async: 10  │              │ Async: 10  │
   │ Kafka: 3   │              │ Kafka: 3   │              │ Kafka: 3   │
   └─────┬──────┘              └─────┬──────┘              └─────┬──────┘
         │                           │                             │
         │ Publishes                 │ Publishes                   │ Publishes
         │ to Kafka                  │ to Kafka                    │ to Kafka
         │ topic                     │ topic                       │ topic
         ▼                           ▼                             ▼
                    ┌────────────────────────────────────┐
                    │ Kafka Topic: "migration-requests"  │
                    └──────────────┬─────────────────────┘
                                   │
                            Consumed by Workers
                                   ↓
               ┌────────────────────────────────────┐
               │ Spring Boot Worker (with Batch Job)│
               │ - Listens to Kafka Topic           │
               │ - Runs ETL / Migration Logic       │
               │ - Updates status in Redis / Kafka  │
               └────────────────────────────────────┘
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │  Redis / Kafka (Job Status)  │
                    └──────────────────────────────┘
                                   │
                                   ▼
             Client polls /status endpoint to check job status
