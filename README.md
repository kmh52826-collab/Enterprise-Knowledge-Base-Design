# Validation Management Platform & Hybrid Knowledge Base Design
> **AI-Powered Validation Management Platform for Regulated Environments**

**Status:** Active / In Progress (Start: Aug 2026)  
**Role:** Data Architect (Relational Data Modeling & AI Knowledge Base Data Design)  
**Current Progress:** Phase 1 - Relational Data Modeling (Completed) / Phase 2 - AI Knowledge Base & Vector/Graph DB (In Progress)

> ⚠️ **Notice & Disclaimer:**  
> - **WIP Project Notice:** This repository contains documentation for a corporate project that is currently under development (Work-in-Progress). The system architecture and detailed module implementations are continuously being updated, and some documents or code sections may still be under construction.  
> - **Security & Dummy Data Disclaimer:** All example data, figures, identifiers, and source code samples in this repository and its technical documents are **dummy data** that have been processed or substituted for information security and confidentiality. No actual internal trade secrets, security data, or real customer information of the company are included.

---

## 📌 Project Executive Summary

This project builds an **AI-based Validation Management Platform** that complies with GMP and CSV (Computerized System Validation) regulations in the pharmaceutical and biotech field and digitalizes the entire Validation workflow, which had been fragmented.

It is designed to manage the **end-to-end Validation lifecycle** on a single platform, from validation planning and pre-assessment through URS, design (FDS/DDS) and design qualification (DQ), risk assessment (FRA), qualification testing (IQ/OQ/PQ), deviation management, and the summary report (VSR).

External AI engineers develop the AI features with a vector DB and LLMs, and I am responsible for designing the entire platform and the data that the AI reads and writes. This portfolio focuses on the two areas I am responsible for:
- **Relational Data Modeling for Regulatory Compliance (RDBMS ERD)** `[Completed]`
- **Data Design and Preparation of a Hybrid Knowledge Base (Vector DB & Graph DB) for AI Audit Support and End-to-End Traceability** `[In Progress]`

---

## 🛠 Tech Stack

| Category | Technologies |
| :--- | :--- |
| **Relational Database** | Amazon RDS (PostgreSQL) |
| **AI & Knowledge Databases** | pgvector, Graph DB (Neo4j) |
| **Object Storage & AI Pipeline** | Amazon S3, Amazon Bedrock API |
| **Languages** | Python, SQL |

---

## 🏗️ System Overview & Architecture

[![System Overview & Context](https://github.com/user-attachments/assets/26fe8971-9772-43d5-a1f0-de782b84f96c)](https://github.com/user-attachments/assets/26fe8971-9772-43d5-a1f0-de782b84f96c)

> 💡 **Tip:** Click the image above to view the full diagram at its original resolution, larger and sharper.

### 🎯 Architecture Summary
- **Validation Lifecycle** : End-to-End support for the entire process in a regulated environment (CSV), from validation planning through URS, design, DQ, FRA, IQ/OQ/PQ, deviation management, and VSR
- **Core AI Audit & Traceability** : Designed so that, for in-depth inquiries from regulatory auditors (Auditor), the AI searches for and provides relevant documents through the hybrid knowledge base and can trace End-to-End Lineage and approval history
- **Compliance & Governance** : Audit Trail and history management based on database constraints and server-side validation, compliant with 21 CFR Part 11 and ALCOA++ principles
- **Hybrid Knowledge Architecture** : 
  - **RDBMS (PostgreSQL)** : Management of Core Data, Audit Trail, and Source of Truth
  - **Vector DB (pgvector)** : Semantic Search and AI Context Retrieval
  - **Graph DB (Neo4j)** : End-to-End Lineage and Impact/Dependency Analysis
  - **Object Storage (S3)** : Document Repository & Evidence Storage

---

## 🚀 Key Technical Contributions & Engineering Design

### 1. End-to-End Relational Data Modeling & ERD Design (Completed)
Reflecting the complexity of a Highly Regulated Domain, I personally designed the entire PostgreSQL schema of 80 tables across 14 business areas, from conceptual modeling to logical/physical ERD design. Each time the requirements changed, I reported the design to the project members and the PM, incorporated their feedback, and revised it several times.

- **Regulatory Compliance Schema Design:** Designed to enforce 21 CFR Part 11 (electronic signatures and audit trails) and ALCOA++ principles through both database constraints (PK/FK/CHECK/UNIQUE) and server-side validation rules. An electronic signature preserves the signed original content together with the target revision, and audit trail records are never modified; only new records are added.
- **Revision-Pinned Traceability:** Designed so that the trace links between requirements (URS), design, risk assessment (FRA), and test items (IQ/OQ/PQ) point to the exact revision at the time of linking. Past approval evidence does not change even when a new revision is created, and each requirement is linked to the regulatory clause it is based on, together with that clause's edition.
- **AI Output Provenance:** Drafts generated by AI do not immediately become official deliverables. Generation, user selection, and application to business records are managed as separate states, and the origin of each item, whether written directly or taken from an AI draft, is recorded.
- **Design Trade-off:** Polymorphic references were used to link revisions of many kinds of deliverables to each other. As a result, referential integrity must be validated in the application layer rather than by database FKs. To compensate, I defined rules that check the target table, PK, and revision on the server at every save, submission, and approval.

#### 🔗 Data Model Documents
- [Data Dictionary](docs/data-dictionary.md) — columns, constraints, and business rules of the 80 tables
- [ERD Specifications](docs/erd-specifications.md) — table relationships for each of the 14 business areas
- [Overall Data Structure](docs/erd-overview.md) — overall structure across areas

---

### 2. AI Knowledge Base Preparation (In Progress)
I am preparing the knowledge base that the AI will use to search and to generate document drafts.

- **Document De-identification:** I am building processing logic that de-identifies, in bulk, the deliverable documents the in-house GMP/CSV team wrote across about 160 Validation projects, so that external AI engineers can use them.
- **Vector Indexing (with AI engineers):** Together with the AI engineers, I am parsing and embedding the de-identified documents in chunks and loading them into pgvector. This enables context retrieval based on meaning (Semantic), beyond simple keyword search.

---

### 3. Hybrid Retrieval for Audit Search & Traceability (In Progress)
Together with the AI engineers, I am designing a hybrid retrieval structure connected to the relational source data so that, when an in-depth inquiry comes from a regulatory reviewer (Auditor), **the AI can find the relevant documents and trace the approval history**.

- **Graph DB Modeling (Knowledge Graph & Ontology):** To overcome the limits of simple similarity search between documents, I am conceiving an approval-lineage graph model (Ontology) that can answer **"Through which risk assessment (FRA) was a specific requirement (URS) finally approved?"**
- **Validity-Aware Retrieval:** Disposed old versions or unapproved drafts can be very similar to a question, but they cannot be used as audit evidence. I am designing an approach that combines semantic search results with the revision and approval information in the relational schema so that only currently valid evidence is provided to the AI.
