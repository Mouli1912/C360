# CIBC Collections AI Platform

> Enterprise-grade system design for the CIBC Collections Hackathon.  
> **Unify the data. Ask it anything. Power the next best action.**

---

## 1. Executive Summary

The CIBC Collections problem is addressed through a three-layer architecture:

1. **Data Product Factory** — converts fragmented operational data into trusted, governed Collections 360 and feature-ready data products.
2. **Insight & NLP** — enables governed natural-language access to curated and golden data through NL-to-SQL, semantic definitions, RAG, and explainable answers.
3. **AI Decisioning** — uses customer, account, interaction, behavioural, and operational signals to support a Next Best Action workflow with explainability, constraints, and human review.

Governance is a cross-cutting capability across all three layers.

The design is intentionally modular so that ingestion, identity resolution, semantic querying, model inference, decisioning, agent experience, monitoring, and governance can evolve independently.

---

# 2. Problem Definition

Collections data is distributed across multiple operational domains:

- Customers
- Cards
- Loans
- Deposits
- Collections cases
- Contact history
- Promises to Pay
- Agent notes
- Call transcripts
- Transactions
- Statements
- Installments
- Bureau history
- Salary history
- Model scores
- Offers
- Agents and operational capacity
- Policy/reference documents

The core engineering problem is not simply building another dashboard or prediction model.

The platform must create a trustworthy path:

```text
Fragmented Data
      ↓
Trusted Data Products
      ↓
Collections 360
      ↓
Semantic / Insight Layer
      ↓
Decision Intelligence
      ↓
Human Review
      ↓
Customer / Agent Action
      ↓
Outcome Feedback
```

---

# 3. Architecture Principles

The system follows these principles:

### 3.1 Data products before AI

AI consumes governed data products rather than directly querying raw operational sources.

### 3.2 Semantic layer before NL-to-SQL

Natural-language questions are resolved against certified business metrics and approved source tables before SQL generation.

### 3.3 Decisioning is constrained optimisation

The NBA engine does not blindly select a model prediction. It combines:

- predictive signals
- customer state
- eligibility
- policy
- operational capacity
- channel constraints
- hardship/vulnerability indicators
- expected outcome

### 3.4 Human-in-the-loop

Customer-impacting actions require appropriate human review.

### 3.5 Explainability by construction

Every recommendation should have:

- decision
- reason
- supporting signals
- model/version
- relevant policy
- confidence/score where applicable
- audit trail

### 3.6 Governance is part of the architecture

Security, data contracts, access control, monitoring, protected-attribute controls, and auditability are not post-processing features.

---

# 4. High-Level Architecture

```text
                           ┌─────────────────────────────┐
                           │        SOURCE SYSTEMS       │
                           ├─────────────────────────────┤
                           │ Customers                   │
                           │ Cards / Loans / Deposits    │
                           │ Collections Cases           │
                           │ Contact History             │
                           │ PTP                         │
                           │ Transactions / Statements   │
                           │ Bureau / Salary History     │
                           │ Notes / Transcripts         │
                           │ Offers / Model Scores       │
                           │ Operations / Policies       │
                           └──────────────┬──────────────┘
                                          │
                                          ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                         LAYER 1 — DATA PRODUCT FACTORY                     │
│                                                                            │
│  Ingestion → Standardisation → Data Quality → Identity Resolution         │
│       → Curated/Silver → Golden Collections 360 → Feature Products        │
└────────────────────────────────┬───────────────────────────────────────────┘
                                 │
              ┌──────────────────┴──────────────────┐
              │                                     │
              ▼                                     ▼
┌─────────────────────────────┐       ┌────────────────────────────────────┐
│ LAYER 2 — INSIGHT & NLP     │       │ LAYER 3 — AI DECISIONING          │
│                             │       │                                    │
│ NL Question                 │       │ Customer/Case/Account Signals     │
│       ↓                     │       │            ↓                       │
│ Intent + Entity Detection  │       │ Predictive Models                 │
│       ↓                     │       │            ↓                       │
│ Authorization              │       │ Candidate Actions                 │
│       ↓                     │       │            ↓                       │
│ Semantic Layer             │       │ Eligibility / Policy               │
│       ↓                     │       │            ↓                       │
│ NL → SQL                   │       │ Operational Constraints            │
│       ↓                     │       │            ↓                       │
│ Read-only Execution        │       │ NBA Optimisation                   │
│       ↓                     │       │            ↓                       │
│ Answer + SQL + Sources     │       │ Explanation + Human Review         │
│                             │       │            ↓                       │
│ RAG over Notes/Policies    │       │        Agent / Customer             │
└─────────────────────────────┘       └────────────────────────────────────┘
              │                                     │
              └──────────────────┬──────────────────┘
                                 ▼
                    ┌─────────────────────────┐
                    │ MONITORING / GOVERNANCE │
                    │ Audit / Drift / Quality │
                    │ Access / HITL / Policy  │
                    └─────────────────────────┘
```

---

# 5. Source Data Architecture

The data dictionary contains **31 tables, 1,421 column definitions, and 48 documented relationships**.

## 5.1 Core Tables

```text
customers
card_accounts
loan_accounts
deposit_accounts
collections_cases
contact_history
promises_to_pay
```

## 5.2 Unstructured Data

```text
agent_notes
call_transcripts
voice_samples
```

## 5.3 Historical / Behavioural Data

```text
loan_instalments
transactions
card_statements
account_monthly_snapshot
bureau_history
salary_credit_history
```

## 5.4 Decisioning Data

```text
model_scores
offers
```

## 5.5 Operations

```text
agents
case_assignment_history
agent_shifts
channel_capacity
ops_events
```

## 5.6 Semantic / Governance Data

```text
metric_definitions
benchmark_questions
reference_documents
hardship_programs
qa_checklist
```

## 5.7 Reference Data

```text
geo_reference
code_lookups
external
```

---

# 6. Layer 1 — Data Product Factory

## Objective

Transform fragmented source data into governed, reusable data products.

```text
Raw Sources
    ↓
Ingestion
    ↓
Schema Validation
    ↓
Standardisation
    ↓
Data Quality
    ↓
Identity Resolution
    ↓
Curated / Silver
    ↓
Golden Collections 360
    ↓
Feature Products
```

---

# 7. Ingestion

Each source is treated as a governed data product.

The ingestion layer should capture:

- source identifier
- ingestion timestamp
- source event timestamp
- schema version
- record count
- checksum where appropriate
- ingestion status
- data quality status
- lineage metadata

A source should not silently enter downstream processing if its schema violates the expected contract.

---

# 8. Schema Drift Detection

Schema drift is detected before data enters curated products.

Examples:

```text
Expected:
overdue_amount : decimal

Received:
overdue_amount : string
```

or:

```text
Expected:
snapshot_date

Received:
snapshot_dt
```

The platform should distinguish:

- additive changes
- breaking changes
- type changes
- missing columns
- unexpected columns
- relationship changes

AI assistance may be used for:

- source-to-target mapping suggestions
- transformation generation
- schema anomaly explanation

Final acceptance remains governed.

---

# 9. Data Quality Framework

Minimum quality controls include:

### Completeness

Required identifiers must not be unexpectedly null.

### Uniqueness

Primary-key constraints must hold.

### Referential Integrity

Foreign-key relationships must be valid.

### Validity

Values must conform to expected domains.

### Consistency

Dates, monetary values, statuses, and account relationships must remain internally consistent.

### Freshness

Data must arrive within its defined SLA.

### Distribution Monitoring

Important numeric and categorical fields should be monitored for unexpected distribution changes.

---

# 10. Identity Resolution

One of the key data-engineering challenges is that source systems do not consistently use the same customer identifier.

Examples from the data dictionary include:

```text
customers.crm_customer_id
card_accounts.src_customer_ref
loan_accounts.src_customer_ref
deposit_accounts.src_customer_ref
external.bank_subject_ref
```

Therefore:

```text
Source Customer Key
       ↓
Identity Mapping
       ↓
crm_customer_id
       ↓
Collections 360
```

The identity-resolution layer should maintain:

- source system
- source customer key
- canonical CRM customer ID
- match method
- confidence
- effective timestamps
- lineage

Identity matching should never be an opaque join.

---

# 11. Collections 360

The golden customer view combines relevant customer, account, collections, interaction, payment, behavioural, and decisioning information.

Conceptually:

```text
Customer
   │
   ├── Cards
   ├── Loans
   ├── Deposits
   ├── Collections Cases
   ├── Contacts
   ├── PTPs
   ├── Payments
   ├── Behaviour
   ├── Bureau Signals
   ├── Agent Interactions
   ├── Model Scores
   └── Offers
```

The golden layer should expose stable, business-consumable entities rather than forcing every consumer to understand source-system complexity.

---

# 12. Collections Case Model

`collections_cases` acts as a central operational object.

Relevant fields include:

- customer reference
- account identifiers
- primary product
- DPD
- delinquency bucket
- overdue amount
- exposure
- queue
- agent
- strategy
- test cell
- treatment
- next action
- contact counts
- PTP counts
- current PTP
- payment information
- cure/recidivism information
- hardship information

This makes the case a natural bridge between data products and decisioning.

---

# 13. Feature Engineering

Feature products should be generated from governed golden data.

Examples:

```text
Days Past Due
Overdue Exposure
Recent Payment Behaviour
PTP Broken Rate
Contact Success Rate
RPC Rate
Recent Contact Frequency
Historical Cure Rate
Recent Transaction Behaviour
Income / Salary Trends
Account-Level Delinquency
Product Mix
Prior Collections Outcomes
```

Features must have:

- definition
- source
- timestamp/as-of logic
- transformation
- owner
- data quality expectations

---

# 14. Point-in-Time Correctness

Decisioning features must respect the decision timestamp.

Bad:

```text
Decision at T
       ↓
Feature calculated using data after T
```

Correct:

```text
Decision at T
       ↓
Only information available at or before T
       ↓
Feature vector
       ↓
Prediction
```

This prevents target leakage and ensures offline evaluation is representative of production inference.

---

# 15. Layer 2 — Insight & NLP

The Insight layer provides governed natural-language access to trusted data.

Example:

```text
"Which customers are most likely to break a promise to pay?"
```

or:

```text
"Which strategy worked best for unsecured loans in the last 90 days?"
```

or:

```text
"What were the top drivers of roll-rate deterioration this week?"
```

The system must produce:

- answer
- SQL
- source tables / metrics
- relevant definitions
- refusal where the question is outside approved scope

---

# 16. Insight Architecture

```text
User Question
      ↓
Intent Classification
      ↓
Entity / Metric Extraction
      ↓
Authorization
      ↓
Semantic Layer
      ↓
SQL Generation
      ↓
SQL Validation
      ↓
Read-only Execution
      ↓
Result Validation
      ↓
Answer Generation
      ↓
Answer + SQL + Sources
```

---

# 17. Semantic Layer

The `metric_definitions` table is a key component.

It contains concepts such as:

- metric name
- business definition
- pseudo-SQL formula
- grain
- time basis
- filters
- unit
- good direction
- synonyms
- source tables
- example question
- owner
- certification status

This becomes the business semantic contract between humans, data, and AI.

---

# 18. NL-to-SQL

The NL-to-SQL system should not generate arbitrary SQL directly against every table.

Instead:

```text
Question
   ↓
Metric / Entity Mapping
   ↓
Approved Semantic Definitions
   ↓
Approved Tables / Columns
   ↓
SQL Generation
   ↓
SQL AST / Rule Validation
   ↓
Read-only Execution
```

Validation should check:

- allowed tables
- allowed columns
- read-only operations
- joins
- filters
- aggregation
- row limits
- time boundaries
- protected fields
- query cost

---

# 19. Query Safety

The Insight layer must prevent:

```sql
INSERT
UPDATE
DELETE
DROP
ALTER
TRUNCATE
```

It should also prevent access to restricted fields unless explicitly authorised.

For example, direct customer PII such as:

```text
name
phone
email
address
national_id_hash
```

should not become freely queryable merely because the model can generate SQL.

---

# 20. Refusal / Out-of-Scope Handling

The system should refuse unsupported questions.

The `benchmark_questions` table provides a useful evaluation mechanism through fields such as:

- question text
- question type
- difficulty
- expected tables
- expected metric
- should_refuse
- refusal reason
- tolerance
- grading rubric

Example:

```text
User:
"Give me the personal phone number of every customer."

System:
Refuse due to data-access and privacy policy.
```

The refusal should explain the permitted alternative when appropriate.

---

# 21. RAG for Unstructured Data

Structured data is not sufficient for questions involving:

- agent notes
- call transcripts
- procedures
- call scripts
- hardship programs
- regulatory summaries
- FAQs
- reference documents

RAG pipeline:

```text
Documents
   ↓
Parsing
   ↓
PII / PCI Redaction
   ↓
Chunking
   ↓
Embedding
   ↓
Vector Index
   ↓
Retrieval
   ↓
Policy / Access Filter
   ↓
LLM
   ↓
Answer + Citations
```

---

# 22. Layer 3 — AI Decisioning

For the hackathon, the selected Layer 3 use case is:

# Next Best Action

The objective is to support collection agents with a context-aware, explainable action recommendation.

The architecture is:

```text
Collections 360
      +
Behavioural Features
      +
Interaction Signals
      +
Model Scores
      +
Policy
      +
Operational Constraints
      ↓
Candidate Action Generation
      ↓
Eligibility Filtering
      ↓
Decision Optimisation
      ↓
Recommended NBA
      ↓
Explanation
      ↓
Human Review
      ↓
Execution
      ↓
Outcome
```

---

# 23. Decision Inputs

Potential inputs include:

### Customer State

- customer profile
- account relationships
- delinquency state
- historical collections outcomes

### Behaviour

- payment patterns
- transaction trends
- salary-credit trends
- prior PTP performance

### Interaction

- contact attempts
- RPC
- channel history
- call/transcript-derived signals
- agent notes

### Model Outputs

`model_scores` provides fields such as:

- score
- score band
- risk grade
- predicted value
- slot label
- slot rank
- answer-rate estimate
- top reasons
- recommended action
- model owner
- validation AUC
- PSI
- protected-attribute usage indicator

These outputs can be used as governed model signals rather than treated as unexplained predictions.

---

# 24. Candidate Action Generation

The system should generate a bounded action set.

For example:

```text
Candidate Actions
├── Contact via approved channel
├── Schedule follow-up
├── Payment arrangement workflow
├── Hardship assessment
├── Offer evaluation
└── Escalation / specialist review
```

The actual production action catalogue must be governed by policy and the available operational workflows.

---

# 25. Eligibility Layer

Before optimisation, candidates should be filtered through:

```text
Customer Eligibility
        +
Policy
        +
Consent
        +
Hardship / Vulnerability
        +
Account State
        +
Operational Availability
```

Invalid actions are removed before ranking.

This prevents the optimisation layer from selecting an action that is operationally or policy-wise impossible.

---

# 26. Constraint Engine

The constraint engine should enforce hard rules.

Examples:

```text
IF hardship_flag = true
    → apply hardship workflow

IF channel consent = false
    → remove channel

IF action unavailable
    → remove action

IF required human approval = true
    → route to reviewer

IF policy eligibility = false
    → remove offer/action
```

Hard constraints should be deterministic and independently testable.

---

# 27. Decision Optimisation

A generic decision objective can be represented as:

```text
Utility(action)
=
Expected Collection Value
× Probability of Success
− Operational Cost
− Customer Friction
− Risk Penalty
```

Subject to:

```text
Policy Constraints
Consent Constraints
Eligibility Constraints
Capacity Constraints
Fairness / Protected Attribute Constraints
Human Review Constraints
```

The exact objective and weights should be calibrated using validated business requirements and offline evaluation.

---

# 28. Explainability

Every NBA recommendation should return structured explanation metadata.

Example:

```json
{
  "action": "PAYMENT_ARRANGEMENT_REVIEW",
  "reason_codes": [
    "HIGH_PTP_BREAK_RATE",
    "RECENT_CONTACT_SUCCESS",
    "ELIGIBLE_FOR_PROGRAM"
  ],
  "model": "collections_nba_v1",
  "model_score": 0.81,
  "policy_checks": {
    "eligible": true,
    "consent": true
  },
  "human_review_required": true
}
```

The UI should translate this into concise agent-facing language.

---

# 29. Human-in-the-Loop

Customer-impacting decisions should not become fully autonomous by default.

Workflow:

```text
AI Recommendation
      ↓
Policy / Constraint Validation
      ↓
Human Review
      ↓
Approve / Modify / Reject
      ↓
Action
      ↓
Outcome
```

Human actions should be logged because they become valuable feedback for future evaluation.

---

# 30. Agent Experience

The agent UI should expose:

```text
┌───────────────────────────────────────────────┐
│ CUSTOMER / CASE                               │
├───────────────────────────────────────────────┤
│ Delinquency Status                            │
│ Exposure                                      │
│ Current PTP                                   │
│ Recent Contact History                        │
│ Payment Behaviour                             │
├───────────────────────────────────────────────┤
│ NEXT BEST ACTION                              │
│                                               │
│ Recommended Action                            │
│ Why                                           │
│ Supporting Signals                            │
│ Policy / Eligibility                          │
│ Confidence / Score                            │
├───────────────────────────────────────────────┤
│ [Approve] [Modify] [Reject]                   │
└───────────────────────────────────────────────┘
```

The agent should not need to understand the underlying ML implementation.

---

# 31. Governance

Governance spans the complete system.

```text
                    GOVERNANCE
┌────────────────────────────────────────────────────────┐
│ Identity & Access Control                              │
│ Data Contracts                                         │
│ Data Quality                                           │
│ Lineage                                                │
│ PII / PCI Protection                                   │
│ Protected Attribute Controls                           │
│ Model Monitoring                                       │
│ Decision Explanation                                   │
│ Human Review                                           │
│ Audit Logging                                          │
│ Retention                                              │
└────────────────────────────────────────────────────────┘
```

---

# 32. Protected Attributes

The data dictionary contains protected/sensitive attributes.

Examples include:

```text
gender_code
marital_status
citizenship_status
ccb_deposit_flag
```

The architecture must ensure protected attributes are not used as decision features.

They may be retained in controlled governance datasets where required for monitoring or compliance, but must be separated from the decisioning feature path.

---

# 33. PII / PCI

Sensitive fields include direct or quasi-identifiers such as:

```text
name
DOB
phone
email
address
national ID hash
masked PAN
borrower information
phone hashes
SMS text
agent notes
call transcripts
```

Controls:

- encryption in transit
- encryption at rest
- role-based access
- field-level access where necessary
- masking
- redaction
- retention policies
- audit logging

---

# 34. Data Contract

Every important data product should have a contract containing:

```text
Owner
Schema
Primary Key
Relationships
Quality Rules
Allowed Uses
Refresh Frequency
PII Classification
Retention
SLA
```

Example:

```yaml
data_product: collections_360

owner: Collections Data Product Owner

grain:
  - customer
  - account
  - collections_case

quality_rules:
  - primary identifiers must be non-null
  - primary keys must be unique
  - foreign keys must resolve
  - monetary fields must satisfy domain constraints
  - data freshness must meet SLA

allowed_uses:
  - collections analytics
  - governed insight
  - approved decisioning
  - model evaluation

restricted_uses:
  - unrestricted customer profiling
  - unauthorised disclosure of PII
  - use of protected attributes for decisioning

refresh:
  - operational feeds according to source SLA
  - feature products according to inference requirements
```

---

# 35. Service Architecture

A modular service decomposition can be implemented as:

```text
                    API Gateway
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   C360 Service    Insight Service   Decision Service
        │                │                │
        ▼                ▼                ▼
 Data Products     Semantic Layer    Feature Store
                         │                │
                         ▼                ▼
                     SQL Engine       Model Runtime
                         │                │
                         └───────┬────────┘
                                 ▼
                           Audit Service
                                 │
                                 ▼
                         Observability Stack
```

---

# 36. Suggested Repository Structure

```text
cibc-collections/
│
├── README.md
│
├── docs/
│   ├── architecture/
│   │   ├── system-architecture.md
│   │   ├── data-architecture.md
│   │   ├── decision-architecture.md
│   │   └── governance.md
│   │
│   ├── contracts/
│   │   └── collections-360.yaml
│   │
│   └── decisions/
│       └── architecture-decision-records/
│
├── data/
│   ├── raw/
│   ├── curated/
│   ├── golden/
│   └── features/
│
├── pipelines/
│   ├── ingestion/
│   ├── quality/
│   ├── identity/
│   ├── c360/
│   └── features/
│
├── services/
│   ├── c360/
│   ├── insight/
│   ├── decisioning/
│   ├── agent/
│   └── governance/
│
├── models/
│   ├── training/
│   ├── inference/
│   ├── evaluation/
│   └── registry/
│
├── semantic/
│   ├── metrics/
│   ├── schemas/
│   └── prompts/
│
├── rag/
│   ├── ingestion/
│   ├── indexing/
│   └── retrieval/
│
├── frontend/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── data/
│   ├── model/
│   └── e2e/
│
├── infra/
│
└── .github/
    └── workflows/
```

---

# 37. API Design

## Collections 360

```http
GET /api/v1/customers/{customer_id}/360
GET /api/v1/cases/{case_id}
```

## Insight

```http
POST /api/v1/insights/query
```

Request:

```json
{
  "question": "Which strategy performed best in the last 90 days?"
}
```

Response:

```json
{
  "answer": "...",
  "sql": "...",
  "metrics": [],
  "sources": [],
  "confidence": 0.0
}
```

## Next Best Action

```http
POST /api/v1/decisioning/nba
```

Request:

```json
{
  "case_id": "CASE_ID"
}
```

Response:

```json
{
  "recommendation": "...",
  "reason_codes": [],
  "model_version": "...",
  "policy_checks": {},
  "human_review_required": true
}
```

---

# 38. Model Lifecycle

```text
Data
 ↓
Feature Generation
 ↓
Training Dataset
 ↓
Experimentation
 ↓
Offline Evaluation
 ↓
Bias / Leakage Checks
 ↓
Model Registry
 ↓
Approval
 ↓
Deployment
 ↓
Monitoring
 ↓
Drift Detection
 ↓
Retraining / Rollback
```

Required monitoring includes:

- predictive performance
- data drift
- feature drift
- calibration
- PSI
- latency
- failure rate
- business outcome
- human override rate

---

# 39. Model Governance

`model_scores` contains useful governance fields:

```text
model
model_version
model_owner
auc_at_validation
psi_last_month
uses_protected_attributes
threshold_used
recommended_action
top_reason_1
top_reason_2
top_reason_3
```

These fields should be surfaced into the decisioning audit trail.

---

# 40. Observability

The platform should monitor four categories.

## Data

```text
Freshness
Completeness
Validity
Uniqueness
Distribution Drift
Schema Drift
```

## Model

```text
AUC / appropriate metric
Calibration
Drift
PSI
Feature Stability
```

## Application

```text
Latency
Throughput
Error Rate
Timeouts
Query Failures
```

## Business

```text
Collection Rate
Cure Rate
PTP Kept Rate
PTP Broken Rate
Contact Success
Offer Acceptance
Human Override Rate
```

---

# 41. Evaluation Strategy

## Layer 1

Evaluate:

- data completeness
- referential integrity
- identity-match accuracy
- pipeline latency
- freshness
- schema-drift detection

## Layer 2

Evaluate:

- SQL correctness
- metric correctness
- retrieval relevance
- refusal accuracy
- PII protection
- answer grounding

The `benchmark_questions` table should be used as a repeatable evaluation suite.

## Layer 3

Evaluate:

- recommendation quality
- business outcome
- constraint compliance
- explanation quality
- human override rate
- fairness monitoring
- stability over time

---

# 42. Testing Strategy

```text
Unit Tests
    ↓
Data Contract Tests
    ↓
Transformation Tests
    ↓
Identity Resolution Tests
    ↓
Semantic Query Tests
    ↓
NL-to-SQL Safety Tests
    ↓
RAG Grounding Tests
    ↓
Decision Constraint Tests
    ↓
Model Evaluation
    ↓
End-to-End Tests
```

Critical tests should include:

```text
No PII leakage
No protected attribute in decision features
No write SQL from Insight layer
No invalid action bypasses policy
No decision using post-outcome information
No recommendation without required explanation
```

---

# 43. Scalability

The system should scale independently by workload.

```text
Ingestion
  → horizontally scalable workers

Data Products
  → partitioned / incremental pipelines

Insight
  → stateless API + query execution pool

RAG
  → distributed vector index

Decisioning
  → stateless inference service

Monitoring
  → asynchronous event pipeline
```

Large historical datasets should be processed incrementally instead of repeatedly recomputing the full universe.

---

# 44. Reliability

Important reliability mechanisms:

### Idempotent ingestion

Reprocessing the same source batch should not create duplicates.

### Retry with backoff

Transient failures should be retried.

### Dead-letter handling

Malformed records should be isolated.

### Circuit breakers

Downstream failures should not cascade.

### Versioned contracts

Breaking schema changes require explicit handling.

### Rollback

Models and decision policies should support rollback to an approved version.

---

# 45. Security Model

Use role-based access control.

Example roles:

```text
DATA_ENGINEER
DATA_SCIENTIST
COLLECTIONS_AGENT
SUPERVISOR
RISK_REVIEWER
ADMIN
AUDITOR
```

Access should be granted according to least privilege.

Example:

```text
Agent
  → customer/case information required for work

Data Scientist
  → governed analytical datasets

Auditor
  → logs and governance metadata

Admin
  → platform configuration
```

---

# 46. AI Usage

AI should be used where it adds leverage rather than replacing deterministic engineering.

## AI-assisted Data Engineering

```text
Schema → AI mapping suggestion → validation → approval
```

## AI-assisted Insight

```text
Natural Language → Semantic Mapping → SQL → Validation → Execution
```

## AI-assisted Unstructured Understanding

```text
Notes / Transcript → Retrieval → Summarisation / Extraction
```

## AI-assisted Decisioning

```text
Signals → Model / Candidate Actions → Explainable Recommendation
```

Deterministic controls remain responsible for:

- access control
- policy enforcement
- PII controls
- SQL safety
- eligibility
- hard constraints
- audit logging

---

# 47. AI Guardrails

The LLM must not be the final authority for:

```text
Access Control
Policy Enforcement
Protected Attribute Decisions
PII Authorisation
SQL Execution
Eligibility
Hard Constraints
Audit Records
```

The LLM can propose.

The system validates.

The governed runtime executes.

---

# 48. Prompt Architecture

Prompts should be version controlled.

Example:

```text
System Prompt
    ↓
Policy Context
    ↓
Semantic Schema
    ↓
Metric Definitions
    ↓
User Question
    ↓
Retrieved Context
    ↓
Generated Output
```

Prompts should not contain uncontrolled sensitive data.

---

# 49. Demo Flow

The complete demonstration should follow one coherent customer journey.

## Step 1 — Data

Show:

```text
Multiple Source Tables
        ↓
Collections 360
```

## Step 2 — Insight

Ask:

```text
"Which strategies have the highest kept-PTP rate over the last 90 days?"
```

Show:

```text
Answer
SQL
Metric Definition
Source Tables
```

## Step 3 — Customer Context

Open a collection case.

Show:

```text
Delinquency
Exposure
Contact History
PTP History
Behaviour
Model Signals
```

## Step 4 — NBA

Generate:

```text
Next Best Action
```

Show:

```text
Recommendation
Reasons
Eligibility
Policy
Human Review
```

## Step 5 — Feedback

Approve / modify / reject.

Record outcome.

---

# 50. Build Plan

## Phase 1 — Foundation

```text
Repository
Environment
Database
Data ingestion
Core schemas
```

## Phase 2 — Layer 1

```text
Data quality
Identity resolution
Collections 360
Feature generation
```

## Phase 3 — Layer 2

```text
Metric definitions
Semantic mapping
NL-to-SQL
Query validation
RAG
```

## Phase 4 — Layer 3

```text
Feature vector
Model integration
Candidate actions
Constraint engine
NBA
Explanation
```

## Phase 5 — Governance

```text
RBAC
PII controls
Audit
Monitoring
Evaluation
```

## Phase 6 — Demo

```text
End-to-end workflow
UI polish
Architecture validation
Failure-path testing
Final documentation
```

---

# 51. 36-Hour Hackathon Execution Plan

## Oct 2 — System Design

```text
Architecture
Data mapping
Layer responsibilities
Decisioning design
Governance design
Repo skeleton
```

## Oct 3 — 9:00 AM

```text
Dataset integration
Layer 1 implementation
Collections 360
```

## Oct 3 — Afternoon

```text
Semantic layer
NL-to-SQL
RAG
```

## Oct 3 — Evening

```text
NBA engine
Constraints
Explainability
```

## Oct 4 — Morning

```text
Integration
Frontend
Observability
Testing
```

## Oct 4 — Afternoon

```text
End-to-end demo
Failure testing
Documentation
Architecture review
```

## Oct 4 — Before 9:00 PM

```text
Final submission
```

---

# 52. Team Ownership

For a four-person team, a practical split is:

```text
Engineer 1
Data Platform / Layer 1

Engineer 2
Insight / NLP / RAG

Engineer 3
AI Decisioning / NBA

Engineer 4
Frontend / Integration / Governance
```

Cross-review should be used for:

```text
Security
Data Contracts
Decision Logic
Demo Integration
```

---

# 53. Engineering Decision Records

Important architectural decisions should be documented.

Example:

```text
ADR-001
Use Collections 360 as the canonical decisioning data product.

ADR-002
Use metric_definitions as the semantic source of truth.

ADR-003
Restrict NL-to-SQL execution to read-only queries.

ADR-004
Keep protected attributes outside the decision feature path.

ADR-005
Require human review for defined customer-impacting actions.

ADR-006
Version models, prompts, policies and data contracts.
```

---

# 54. Non-Functional Requirements

## Performance

```text
C360 lookup → low-latency interactive
Insight query → bounded execution
NBA inference → interactive agent experience
```

## Reliability

```text
No silent data corruption
No uncontrolled model execution
No policy bypass
```

## Security

```text
Least privilege
PII controls
Auditability
```

## Explainability

```text
Every recommendation must be explainable.
```

## Reproducibility

```text
Same input
+
Same feature snapshot
+
Same model version
+
Same policy version
=
Reproducible decision
```

---

# 55. End-to-End Data Flow

```text
┌───────────────────────────────────────────────────────────────────┐
│                         SOURCE SYSTEMS                            │
└──────────────────────────────┬────────────────────────────────────┘
                               │
                               ▼
                     ┌────────────────────┐
                     │ INGESTION          │
                     └─────────┬──────────┘
                               │
                               ▼
                     ┌────────────────────┐
                     │ DATA QUALITY       │
                     └─────────┬──────────┘
                               │
                               ▼
                     ┌────────────────────┐
                     │ IDENTITY RESOLUTION│
                     └─────────┬──────────┘
                               │
                               ▼
                     ┌────────────────────┐
                     │ CURATED / SILVER   │
                     └─────────┬──────────┘
                               │
                               ▼
                     ┌────────────────────┐
                     │ COLLECTIONS 360    │
                     └─────────┬──────────┘
                               │
               ┌───────────────┴────────────────┐
               │                                │
               ▼                                ▼
      ┌────────────────────┐          ┌────────────────────┐
      │ INSIGHT & NLP      │          │ AI DECISIONING     │
      │                    │          │                    │
      │ Semantic Layer     │          │ Features           │
      │ NL-to-SQL          │          │ Models             │
      │ RAG                │          │ Constraints        │
      │ SQL Validation     │          │ NBA                │
      └─────────┬──────────┘          └─────────┬──────────┘
                │                               │
                └───────────────┬───────────────┘
                                ▼
                       ┌──────────────────┐
                       │ HUMAN REVIEW     │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │ ACTION           │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │ OUTCOME          │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │ MONITORING       │
                       │ & FEEDBACK       │
                       └──────────────────┘
```

---

# 56. Final Architecture Thesis

The platform is not three independent applications.

It is one governed decision-intelligence platform:

```text
             TRUSTED DATA
                  │
                  ▼
          COLLECTIONS 360
             /         \
            /           \
           ▼             ▼
      INSIGHT          DECISION
         │                │
         ▼                ▼
    "WHAT IS             "WHAT
     HAPPENING?"          SHOULD WE DO?"
         │                │
         └───────┬────────┘
                 ▼
           HUMAN + AI
                 │
                 ▼
             OUTCOME
                 │
                 ▼
             FEEDBACK
```

The key architectural property is that all three layers share:

- governed data products
- semantic definitions
- identity resolution
- security
- auditability
- monitoring
- model/prompt/version governance

This creates a closed-loop Collections intelligence platform rather than a collection of disconnected AI features.

---

# 57. Source of Truth

The following components should be treated as authoritative:

### Data Structure

`CIBC Hackathon - Data Dictionary.xlsx`

### Problem Definition

`CIBC Collections Hackathon Problem Statement.pdf`

### Hackathon Submission Requirements

The official problem statement and submission instructions.

### Implementation

This repository.

The README describes the intended system architecture. Code, configuration, schemas, model artifacts, and deployment manifests are the implementation source of truth once built.

---

# 58. Implementation Status

Update this section continuously during the hackathon.

```text
[ ] Repository setup
[ ] Database setup
[ ] Raw ingestion
[ ] Data quality framework
[ ] Identity resolution
[ ] Collections 360
[ ] Feature pipeline
[ ] Semantic layer
[ ] NL-to-SQL
[ ] Query guardrails
[ ] RAG
[ ] Model integration
[ ] Candidate action engine
[ ] Constraint engine
[ ] NBA
[ ] Explainability
[ ] Human review
[ ] Agent UI
[ ] Monitoring
[ ] Governance
[ ] End-to-end testing
[ ] Demo
[ ] Final documentation
```

---

# 59. Quick Start

Replace the commands below with the project's actual runtime commands as implementation is finalised.

```bash
git clone <repository-url>

cd cibc-collections

# Create environment
python -m venv .venv

# Activate on Windows
.venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
copy .env.example .env

# Start services
docker compose up -d

# Run application
python -m app
```

---

# 60. Development Rules

## Do

- Keep business definitions in the semantic layer.
- Keep policies deterministic.
- Version models and prompts.
- Test data contracts.
- Log decisions.
- Keep decision features separate from protected attributes.
- Validate every AI-generated SQL statement.
- Preserve lineage.
- Make every NBA explainable.

## Do Not

- Query raw tables directly from the UI.
- Allow LLM-generated write SQL.
- Put unrestricted PII into prompts.
- Use protected attributes for decisioning.
- Let an LLM bypass policy.
- Hide model decisions behind a single opaque score.
- Use post-outcome data for historical decision reconstruction.
- Treat generated explanations as evidence unless grounded in actual model/data outputs.

---

# 61. Final System Principle

```text
AI proposes.
Data validates.
Policy constrains.
Humans review.
Systems execute.
Outcomes teach.
Governance controls everything.
```

---

## Project Objective

Build a governed, scalable Collections intelligence platform that can:

1. **Unify fragmented data**
2. **Expose trusted business insights**
3. **Support explainable AI decisioning**
4. **Keep humans in control of customer-impacting actions**
5. **Create an auditable feedback loop**

**Unify the data. Ask it anything. Power the next best action.**
