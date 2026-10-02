# CIBC Collections Intelligence & Next Best Action Platform

> **CIBC Collections Hackathon — L6-Level System Design**
>
> A governed AI decision-support platform that transforms fragmented collections data into a trusted **Collections 360**, enables **natural-language analytics**, and provides **explainable Next Best Action (NBA)** recommendations to collections agents.

---

## Table of Contents

- [1. Executive Summary](#1-executive-summary)
- [2. Problem Definition](#2-problem-definition)
- [3. Solution Overview](#3-solution-overview)
- [4. Design Principles](#4-design-principles)
- [5. High-Level Architecture](#5-high-level-architecture)
- [6. Data Architecture](#6-data-architecture)
- [7. Layer 1 — Data Product Factory](#7-layer-1--data-product-factory)
- [8. Identity Resolution](#8-identity-resolution)
- [9. Collections 360](#9-collections-360)
- [10. Feature Engineering](#10-feature-engineering)
- [11. Layer 2 — Insight & NLP](#11-layer-2--insight--nlp)
- [12. Semantic Layer](#12-semantic-layer)
- [13. Natural Language to SQL](#13-natural-language-to-sql)
- [14. Retrieval-Augmented Generation](#14-retrieval-augmented-generation)
- [15. Layer 3 — Next Best Action](#15-layer-3--next-best-action)
- [16. Candidate Action Generation](#16-candidate-action-generation)
- [17. Decision Optimisation](#17-decision-optimisation)
- [18. Explainability](#18-explainability)
- [19. Human-in-the-Loop](#19-human-in-the-loop)
- [20. Responsible AI](#20-responsible-ai)
- [21. Security Architecture](#21-security-architecture)
- [22. Data Contract](#22-data-contract)
- [23. Service Architecture](#23-service-architecture)
- [24. API Design](#24-api-design)
- [25. Repository Structure](#25-repository-structure)
- [26. Testing Strategy](#26-testing-strategy)
- [27. Observability](#27-observability)
- [28. Scalability & Reliability](#28-scalability--reliability)
- [29. Model Lifecycle](#29-model-lifecycle)
- [30. Technology Stack](#30-technology-stack)
- [31. Implementation Plan](#31-implementation-plan)
- [32. 36-Hour Hackathon Execution Plan](#32-36-hour-hackathon-execution-plan)
- [33. Demo Flow](#33-demo-flow)
- [34. Engineering Decisions](#34-engineering-decisions)
- [35. Future Extensions](#35-future-extensions)
- [36. Final Architecture Thesis](#36-final-architecture-thesis)

---

# 1. Executive Summary

The CIBC Collections Hackathon provides a multi-domain collections ecosystem containing structured and unstructured data across customers, accounts, collections cases, contact history, promises to pay, transactions, statements, bureau history, model scores, offers, agents, channel capacity, policies and other operational sources.

The central challenge is not simply to build a predictive model.

The platform must establish a complete decision-support lifecycle:

```text
Fragmented Data
      ↓
Trusted Data Products
      ↓
Collections 360
      ↓
Certified Business Intelligence
      ↓
Predictive Signals
      ↓
Candidate Actions
      ↓
Policy + Operational Constraints
      ↓
Next Best Action
      ↓
Explanation
      ↓
Human Review
      ↓
Customer Action
      ↓
Outcome
      ↓
Monitoring & Learning