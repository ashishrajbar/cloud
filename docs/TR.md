# JTAPPS Technical Requirements

## 1. Document Info

Owner: Principal Architect
Version:  1.0
Approval Required From:

* Business Sponsor
* JTAPPS Lead
* Cloud Engineering
* Production Support

This document defines the non-negotiable technical characteristics of JTAPPS and governs all implementation decisions.

## 2. Purpose and Scope

### 2.1 Purpose

This document establishes the technical foundation for JTAPPS (modernized IPM platform), defining:

* Target architecture characteristics
* Technology constraints
* Integration patterns
* Performance expectations
* Operational standards
* Security & compliance requirements

This document ensures that JTAPPS is built as a cloud-native, resilient, auditable, and horizontally scalable trading platform.

### 2.2 Scope

* US Equities Trade ingestion pipelines (Executions, Blocks, Allocations)
* Event streaming architecture
* Microservices implementation
* Data storage and caching layers
* Observability and operational model
* CI/CD and Infrastructure as Code
* Disaster recovery and resilience
* Compliance alignment (auditability)

### 2.3 Out of Scope

* UI/UX design details
* Business rule logic specification

## 3. Architectural Principles

JTAPPS must adhere to the following architectural principles:

### 3.1 Cloud-Native by Design

* Containerized workloads (Kubernetes)
* Immutable infrastructure
* Auto-scaling at compute and messaging layers
* No manual configuration in production

### 3.2 Event-Driven Architecture

* Domain events are immutable
* Services communicate asynchronously where possible
* Kafka is used for decoupling producers and consumers
* No tight service-to-service synchronous dependencies for trade-critical flows

### 3.3 Stateless Application Services

* Application services must not maintain in-memory business state
* State must reside in authoritative data stores
* Enables horizontal scaling and rolling deployments

### 3.4 Clear System of Record Definition

* Kafka stores immutable event history
* Redis stores ephemeral acceleration state
* Databases store authoritative truth
* No ambiguity in ownership of data domains

### 3.5 Infrastructure as Code

* All infrastructure must be provisioned using Terraform
* Kubernetes manifests managed via Helm
* No console-based changes permitted in PROD

## 4. Environment Strategy

Please refer to the documenation under Environments

* UAT must mirror Prod topology
* Same deployment artifact across environments
* Configuration differences externalized via environment variables
* CI/CD pipeline enforces promotion workflow

## 5. Application Requirements

### 5.1 Application Architecture

* Microservices architecture aligned with domain boundaries
* Services deployed in Kubernetes (EKS)
* Event/Data streaming pipeline using Kafka
* Asynchronous event consumers
* APIs exposed via secure ingress

### 5.2 Service Design Requirements

Each service must:

* Expose health endpoints
* Emit structured logs
* Include correlation IDs
* Support graceful shutdown
* Implement idempotency for trade ingestion

## 6 Data & Messaging Requirements

### 6.1 Event Streaming

Kafka shall serve as the durable event backbone:

* At-least-once delivery semantics
* Ordered partition processing
* Schema enforcement via schema registry
* Consumer lag monitoring mandatory
* Enforced Schema registry (Avro / JSON Schema)

Topics must be:

* Partitioned based on business key
* Configured with retention policies aligned to business needs

### 6.2 Datastores

JTAPPS requires multiple data stores with clearly defined responsibilities:

| Data Type | Technology | Product | Role |
| --- | --- | --- | --- |
| Distributed Commit Log | Event / Log Streaming Store | Kafka | Replayable Event History |
| Transactional | RDBMS | RDS PostgreSQL | System of Record |
| Reference Data | Cache | Redis | Acceleration |
| Analytics | Data Ware House | Snowflake | Reporting & BI |
| Files | Object Storage | AWS S3 | File Storage |

### 6.3 Data Retention Policy (Expanded)

Data retention must satisfy regulatory, operational, and performance requirements.

#### 6.3.1 Trade Data

* Retained in primary RDBMS for regulatory retention period (e.g., 7+ years)
* Archived to cost-effective object storage after operational window
* Must support audit reconstruction

#### 6.3.2 Kafka Topics

Time based retention

* Operational topics: 7–14 days
* Critical replay topics: extended retention

#### 6.3.3 Logs & Metrics

* Application logs retained 90 days
* Application logs archived based on organizational policies
* Audit logs retained per compliance policy
* WORM-compliant storage where required

#### 6.3.4 Backups

* Automated database backups
* Cross-region replication (where required)
* Periodic restore validation tests

## 7 Performance & Scalability

### 7.1 Throughput Requirements

System must support X hundred thousand messages per minute (* TBD - Open Question)

### 7.2 Latency Targets

* API P95 latency ≤ defined threshold in ms (* TBD)
* End-to-end trade processing SLA (* TBD)

### 7.3 Scaling Strategies

T.B.D.

* Kubernetes
* Kafka
* Database scaling (vertical + read replicas)
* Load testing mandatory before production release

## Reliability & Availability

### 8.1 High Availability

* Multi-AZ deployment
* No single points of failure
* Automatic failover mechanisms

### 8.2 Graceful Degradation

* Non-critical services degrade before trade ingestion fails
* Backpressure handling and management

### 8.3 Availability Target

* 99.9% minimum for PROD
* Defined maintenance windows

## 9. Security Requirements

### 9.1 Identity & Access Management

* IAM-based service roles
* Role-based Kubernetes access
* No hard-coded credentials
* Hashi Vault as the Secrets Manager

### 9.2 Data Protection

* TLS 1.2+ for all network traffic
* Encryption at rest for all data stores
* Key management via AWS KMS

## 10.  DevOps & CI/CD

### 10.1 CI/CD Pipeline

Pipeline must enforce:

* Build
* Unit tests
* Integration tests
* Security scans
* Artifact versioning
* Automated deployment

### 10.2 Deployment Strategy

* Blue/Green or Canary
* Rollback capability within defined SLA
* No manual production hotfixes

## 11. Observability & Operations

### 11.1 Logging

* Logging for microservices based on containerization best practices 
* Correlation ID across services
* Centralized log aggregation using JEM Platform

### 11.2 Metrics

* Application metrics
* Kafka consumer lag
* DB connection pool monitoring
* JVM metrics (where applicable)

### 11.3 Alerting

* Alerts tied to SLOs
* No noisy alerts
* Runbooks mandatory

## 12. Resilience & Disaster Recovery

### 12.1 RTO / RPO

* Defined Recovery Time Objective
* Defined Recovery Point Objective

### 12.2 DR Strategy

* Cross-region replication for critical components
* DR simulation exercises every six months

## 13. Deployment & Release Management

* Release notes mandatory
* Version tagging in Git
* Approval gates before PROD deployment
* Production deployment freeze during critical trading periods

## 14. Dependencies & Constraints

* External trade feeds
* Settlement systems
* Enterprise cloud guardrails
* Compliance and Regulatory deadlines

## 15. Risks & Open Items

Formal risk log required including:

* Technology risk
* Performance risk
* Organizational risk
* Vendor risk

## 16. Acceptance Criteria

System is considered production-ready when:

* Performance benchmarks validated
* Security review approved
* DR test passed
* Observability verified
* Business sign-off obtained

## 17 Data Ownership & Domain Boundaries

Each domain must have:

* Named technical owner
* Named business owner
* Clear definition of data authority

## 18 Cost & Capacity Planning

* Cloud cost monitoring mandatory
* Autoscaling thresholds aligned with budget
* Reserved capacity where justified

## 19 Technical Debt Management

* Defined process for documenting tech debt
* Remediation timelines
* Architectural review board governance

## 20 Architecture Review & Change Control

* All major changes require architecture review
* ADR (Architecture Decision Records) required
* No ad-hoc design deviations
