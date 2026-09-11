# Validation Management Platform & Hybrid Knowledge Base Design
> **AI-Powered Validation Management Platform for Regulated Environments**

**Status:** Active / In Progress (Start: Aug 2026)  
**Role:** Data Architect & Knowledge Engineer (Relational Data Modeling & AI Knowledge Architecture)  
**Current Progress:** Phase 1 - Relational Data Modeling (Completed) / Phase 2 - AI Pipeline & Vector/Graph DB Architecture (In Progress)

> ⚠️ **Notice & Disclaimer:**  
> - **WIP Project Notice:** This repository contains documentation for an enterprise project that is currently under development (Work-in-Progress). The system architecture and detailed module implementations are continuously being updated, and some documentation or code sections may still be under construction.  
> - **Security & Dummy Data Disclaimer:** All example data, figures, identifiers, source code samples, and other materials included in this repository and technical documentation are **dummy data** that have been modified or replaced for information security and confidentiality purposes. No actual trade secrets, security data, or real customer information from the enterprise's internal systems are included.

---

## 📌 Project Executive Summary

This project is an initiative to build an **AI-powered Validation Management Platform (DVT)** that digitizes the entire fragmented Validation workflow while complying with GMP and CSV (Computerized System Validation) regulations in the pharmaceutical and biotechnology industries.

The platform is designed to centrally manage the **end-to-end Validation lifecycle**, from project initiation, URS, RA, RTM, qualification (IQ/OQ/PQ), and test execution to VSR, operations, and change management within a single platform.

This portfolio focuses on two core pillars that serve as the foundation of the platform:
- **Regulatory-compliant relational data modeling (RDBMS ERD)** `[Completed]`
- **Hybrid knowledge DB (Vector DB & Graph DB) pipeline implementation for AI-powered in-depth audit response and end-to-end traceability** `[In Progress]`

---

## 🛠 Tech Stack

| Category | Technologies |
| :--- | :--- |
| **Relational Database** | Amazon RDS (PostgreSQL) |
| **AI & Knowledge Databases** | Vector DB, Graph DB (Neo4j / Amazon Neptune) |
| **Object Storage & AI Pipeline** | Amazon S3, Amazon Bedrock API |
| **Languages** | Python, SQL |

---

## 🏗️ System Overview & Architecture

[![System Overview & Context](https://github.com/user-attachments/assets/26fe8971-9772-43d5-a1f0-de782b84f96c)](https://github.com/user-attachments/assets/26fe8971-9772-43d5-a1f0-de782b84f96c)

> 💡 **Tip:** Click the image above to view the full diagram in a larger and clearer original resolution.

### 🎯 Architecture Summary
- **Validation Lifecycle** : Supports the entire regulatory environment (CSV) process end-to-end, from Project Initiation to DQ/FRA, IQ/OQ/PQ, RTM, and VSR
- **Core AI Audit & Traceability** : Designed so that, in response to in-depth queries from regulatory auditors, AI can search for and provide relevant documents based on the hybrid knowledge DB, while tracking End-to-End Lineage and approval history
- **Compliance & Governance** : Database-level Audit Trail and history management in compliance with 21 CFR Part 11 and ALCOA++ principles
- **Hybrid Knowledge Architecture** : 
  - **RDBMS (PostgreSQL)** : Core Data, Audit Trail, Source of Truth Management
  - **Vector DB** : Semantic Search and AI Context Retrieval
  - **Graph DB** : End-to-End Lineage and Impact/Dependency Analysis
  - **Object Storage (S3)** : Document Repository & Evidence Storage

---

## 🚀 Key Technical Contributions & Engineering Design

### 1. End-to-End Relational Data Modeling & ERD Design (Completed)
Reflecting the complexity of a highly regulated domain, I directly designed and implemented the entire PostgreSQL schema, from conceptual modeling through logical/physical ERD design.

- **Regulatory Compliance Schema Design:** Designed dedicated Audit Trail and Versioning table structures and a history-tracking architecture to strictly enforce 21 CFR Part 11 (electronic signatures and audit trails) regulations and ALCOA++ principles at the database level.
- **Complex Traceability Mapping & Normalization:** Normalized the complex many-to-many (N:M) dependencies among core entities in the Validation process (Project -> URS -> RA -> RTM -> Test Protocol) and defined a schema capable of precisely tracking the impact relationships (Impact Analysis) between entities.

#### 🔗 [View ERD Data Dictionary & Schema Design Document](docs/erd-specifications.md)
*(The detailed ERD design document and specifications for core tables can be found at the link above.)*

---

### 2. AI Deliverable Draft Generation Pipeline (Vector & Graph-Augmented Retrieval) (In Progress)
I am building an AI pipeline that leverages fragmented internal regulations, guidelines, and original source data from previous project deliverables stored as files in S3 to automatically generate document drafts that comply with regulatory guidelines.

- **Document Ingestion & Vector Embedding:** Designing a pipeline that parses and embeds unstructured Validation documents and templates stored as files in Amazon S3 on a **Vector DB** chunk-by-chunk basis. This is designed to enable precise context retrieval based on semantics (Semantic Search), going beyond simple keyword-based search.
- **Graph-Augmented Context Integration:** Beyond simple text similarity, I am implementing a pipeline that integrates the **hierarchical relationships and approval history between documents (Graph DB)** across Validation stages such as URS and RA with **similarity search from the Vector DB (Semantic Search)**, and uses LLM integration to derive high-quality deliverable drafts (Multi-candidate Drafts) that reflect regulatory consistency and interconnectivity.

---

### 3. Hybrid Database Architecture for Advanced Audit Search & Traceability (In Progress)
Beyond simple document draft generation, I am designing and implementing a database structure that enables **AI to directly locate relevant documents and trace approval histories through a hybrid knowledge DB** when in-depth queries and verification requirements are raised by regulatory reviewers (Auditors).

- **Graph DB Modeling (Knowledge Graph & Ontology):** To overcome the limitations of simple similarity-based searches between documents, I designed a conceptual object relationship and approval history graph model (Ontology) around the question, **"Through which risk assessment (RA) was a specific requirement (URS) ultimately approved?"** Through this, I defined a DB schema capable of precisely tracking approval relationship networks (Lineage Tracking) during auditor queries.
- **Hybrid Retrieval Pipeline:** Building a hybrid search pipeline that combines **Semantic Search (meaning-based document retrieval)** from the Vector DB with **Structural Lineage (structural history tracking)** from the Graph DB, enabling **AI to immediately derive and provide supporting evidence documents and approval processes** in response to complex auditor queries.
