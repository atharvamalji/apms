# Automated Port Management System (APMS)

The **Automated Port Management System (APMS)** is a high-performance, multi-tenant industrial logistics platform engineered to automate vehicle operations, bulk cargo tracking, and marine timelines in deep-water port terminals.

By utilizing a high-concurrency **Go** ingestion layer for real-time edge processing and a robust **Spring Boot** core for complex domain logic, APMS acts as an intelligent automation bridge between physical edge hardware—such as rugged Zebra mobile computers, weighbridge scale indicators, and loop controllers—and legacy enterprise port mainframes.

---

## Key Features

* **Operator-Managed Automation:** Shifting checkpoint workflows from manual text-entry data forms to rapid, one-click Zebra handheld scanner verification loops.
* **State-Machine State Tracking:** Eliminating operational sequence errors via an automated workflow validation engine that prevents vehicles from bypassing critical tracking milestones (e.g., bypassing the weighbridge scale before port exiting).
* **High-Throughput Telemetry Ingestion:** Sub-second edge parsing capable of debouncing continuous physical weight indicators and data telemetry packets without impacting transactional database nodes.
* **Multi-Tenant Identity Federation:** Dual-auth integration infrastructure natively supporting both enterprise **SAML 2.0 Single Sign-On (SSO)** for port authorities and **OAuth 2.0 + PKCE** for active on-field mobile operations.
* **Extensible Legacy Migration Hub:** Dynamic database schema structures coupled with an asynchronous **Kafka** pipeline to seamlessly ingest historical data and safely mirror active operational transactions directly into legacy Oracle DB backends.

---

## System Microservices

The platform is architected around a strict **Database-Per-Service** model to guarantee maximum service decoupling, structural fault tolerance, and isolated horizontal scalability within a **Kubernetes** cluster environment.

| Runtime | Microservice | Database | Primary Responsibility |
| --- | --- | --- | --- |
| **Go** | `apms-telemetry-ingest` | Ephemeral (Redis Cache) | Exposes high-speed REST endpoints for Zebra handheld scans; manages persistent TCP socket loops to capture real-time weighbridge indicator data. |
| **Go** | `apms-device-manager` | PostgreSQL (`apms_device_health`) | Tracks active ping heartbeats of physical edge computers and field components to generate instant network connection loss alerts. |
| **Java** | `apms-auth-broker` | PostgreSQL (`apms_identity_broker`) | Central security gateway that brokers incoming corporate SAML assertions and translates them into uniform, cryptographically signed JWT access tokens. |
| **Java** | `apms-infrastructure-service` | PostgreSQL (`apms_infrastructure_core`) | Serves as the core relational directory for the physical multi-tenant layout hierarchy (Tenant $\rightarrow$ Port $\rightarrow$ Terminal $\rightarrow$ Sector). |
| **Java** | `apms-workflow-engine` | PostgreSQL (`apms_workflow_runtime`) | Evaluates current vehicle milestone sequences upon operator trigger-scans to coordinate gate interlocks and authorize barrier actions. |
| **Java** | `apms-cargo-ledger` | PostgreSQL (`apms_cargo_inventory`) | Executes continuous ACID inventory computations (Gross - Tare mass variables) to adjust real-time volumetric balances inside active yard stockpiles. |
| **Java** | `apms-marine-service` | PostgreSQL (`apms_marine_ops`) | Tracks deep-water vessel diaries, berth booking layouts, and automates Statement of Facts logs to audit maritime demurrage contracts. |
| **Java** | `apms-data-migration-hub` | PostgreSQL (`apms_migration_metadata`) | Manages high-throughput historical ETL migration batches and routes client-extended metadata fields dynamically using PostgreSQL JSONB profiles. |
| **Java** | `apms-iportman-sync` | Database-less (Oracle Client) | Asynchronously consumes finalized terminal event streams from Kafka and safely pushes data records into legacy iPortman Oracle systems using exponential backoff retry logic. |