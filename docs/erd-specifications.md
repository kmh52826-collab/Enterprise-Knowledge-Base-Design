# ERD Specifications

This document presents the ERD specifications for the complete data model of the Validation Management Platform, organized by business domain.

The comprehensive schema is divided into 16 topics, with ERD images illustrating the key tables and relationships in each domain. Each section also includes a structural overview and a summary of each table's role.

> [!NOTE]
> This repository provides public reference materials for sharing examples of data modeling and system design.
>
> The organization names, user information, system names, project names, table names, column names, sample values, and other details used in this document do not represent any specific company's actual production data or internal system architecture. Generalized or fictitious names and examples have been used for public sharing.
>
> Before applying this model in a production environment, separately review your organization's business policies, security standards, personal data protection requirements, data retention policies, and applicable regulatory requirements.

> **For detailed column information**  
> Refer to the [Data Dictionary](./data-dictionary.md) for each table's columns, data types, primary and foreign keys, nullability, default values, and business rules.

## Table of Contents

1. [Organization, User, and Global Role Management](#1-organization-user-and-global-role-management)
2. [System, Project, and Participant Management](#2-system-project-and-participant-management)
3. [Validation Activity and Dependency Management](#3-validation-activity-and-dependency-management)
4. [Library, QIA, and Vendor Audit Management](#4-library-qia-and-vendor-audit-management)
5. [URS and FDS Management](#5-urs-and-fds-management)
6. [DDS and DQ Management](#6-dds-and-dq-management)
7. [FRA Risk Assessment Management](#7-fra-risk-assessment-management)
8. [IQ, OQ, and PQ Qualification Testing Management](#8-iq-oq-and-pq-qualification-testing-management)
9. [Design and Risk Deliverable Traceability Management](#9-design-and-risk-deliverable-traceability-management)
10. [Qualification Test Traceability and RTM Management](#10-qualification-test-traceability-and-rtm-management)
11. [VSR and Deviation Management](#11-vsr-and-deviation-management)
12. [Workflow, Approval, and Electronic Signature Management](#12-workflow-approval-and-electronic-signature-management)
13. [File, Evidence, and File Cleanup Management](#13-file-evidence-and-file-cleanup-management)
14. [Report, Notification, and Backup Operations Management](#14-report-notification-and-backup-operations-management)
15. [AI Generation Job and Result Management](#15-ai-generation-job-and-result-management)
16. [Audit Trail Management](#16-audit-trail-management)

---

## 1. Organization, User, and Global Role Management
<img width="1650" height="1066" alt="image" src="https://github.com/user-attachments/assets/d1f9d215-dafa-46ae-b7ff-75f4bad5b622" />
Link : https://drawsql.app/teams/minho-kim/diagrams/01-organization-user-and-global-role-management

### Structural Overview

This structure manages user accounts by organization and assigns global permissions to individual users through role master data and user-to-role mappings.

Global roles define permissions that apply across the entire system. Project participation and project-specific roles are managed separately in the `project_member` table.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains basic information about client companies or operating organizations and serves as the reference table for user organizational affiliations |
| `app_user` | Manages user accounts, basic profiles, organizational affiliations, and account statuses |
| `role` | Defines global roles and permission scopes that apply across the entire system |
| `user_role` | Manages many-to-many mappings between users and global roles and prevents duplicate assignment of the same role |

---

## 2. System, Project, and Participant Management
<img width="2000" height="1491" alt="image" src="https://github.com/user-attachments/assets/b8625331-f875-41a2-9546-dc2ef32557cd" />
Link : https://drawsql.app/teams/minho-kim/diagrams/02-system-project-and-participant-management

### Structural Overview

This structure organizes validation projects around systems and equipment belonging to an organization and manages participating users and their assigned roles for each project.

Each project is linked to a single target system, and `project_member` maps users and roles at the project level. This allows the same user to perform different roles in different projects.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about the client companies or operating organizations to which systems and users belong |
| `system_asset` | Manages asset management numbers, types, responsible departments, GAMP categories, GxP classifications, and identification statuses for systems or equipment subject to validation |
| `validation_project` | Manages the scope, validation approach, progress percentage, status, and GAMP category of validation projects for each target system |
| `project_member` | Links participating users to their assigned roles for each project and manages participation statuses and periods |
| `app_user` | Manages accounts and basic profiles of users who participate in, create, or modify projects |
| `role` | Maintains reference definitions for roles assigned to project participants, such as author, reviewer, and approver |

---

## 3. Validation Activity and Dependency Management
<img width="2500" height="1729" alt="image" src="https://github.com/user-attachments/assets/3648a194-0317-43eb-aa29-fd262b715add" />
Link : https://drawsql.app/teams/minho-kim/diagrams/03-validation-activity-and-dependency-management

### Structural Overview

This structure creates validation projects for systems belonging to an organization and manages project activities and the dependencies between them.

`validation_activity` defines the master list of validation activities, while `project_activity` manages which activities are included in each project, whether they are mandatory, and their progress statuses. `activity_dependency` manages the criteria used to determine whether a successor activity can be enabled based on the status of predecessor activities or business conditions.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about the client companies or operating organizations to which systems and users belong |
| `app_user` | Manages information about users who create or modify projects, activities, and dependency conditions |
| `system_asset` | Manages identification information, GAMP categories, GxP classifications, and statuses for systems or equipment subject to validation |
| `validation_project` | Manages the scope, validation approach, progress percentage, and status of validation projects for each target system |
| `validation_activity` | Maintains reference information and the default display order for validation activities such as VP, QIA, VA, URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ, RTM, and VSR |
| `project_activity` | Manages the activities included in each project, whether they are mandatory, whether they are enabled, and their progress statuses |
| `activity_dependency` | Manages conditions for enabling successor activities based on predecessor activities, required statuses, condition types, and evaluation order |

---

## 4. Library, QIA, and Vendor Audit Management
<img width="2600" height="1866" alt="image" src="https://github.com/user-attachments/assets/0fa424a7-ec2f-4589-8b5e-75b20e7eefa0" />
Link : https://drawsql.app/teams/minho-kim/diagrams/04-library-qia-and-vendor-audit-management

### Structural Overview

This structure organizes validation projects around systems belonging to an organization and manages a reusable standard library, project-specific quality impact assessments, and vendor audit results.

`library_item` manages standard items used to prepare URS, IQ, and OQ documents. `qia_assessment` and `qia_module_item` assess the applicability and scope of GxP and 21 CFR Part 11 for each project, while `vendor_audit` manages audit plans, results, and deficiency counts for vendors supplying the target systems.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about the client companies or operating organizations to which systems and users belong |
| `app_user` | Manages accounts and basic profiles of users who create and manage projects and assessment documents |
| `system_asset` | Manages identification information, vendors, GAMP categories, GxP classifications, and statuses for systems or equipment subject to validation |
| `validation_project` | Manages the scope, validation approach, progress percentage, and status of validation projects for each target system |
| `library_item` | Manages standard requirements, test procedures, expected results, acceptance criteria, and regulatory references for reuse in URS, IQ, and OQ |
| `qia_assessment` | Manages the applicability of 21 CFR Part 11, GxP scope, document version, and assessment status for each project |
| `qia_module_item` | Manages detailed GxP assessment items and results for each module or process in a QIA document |
| `vendor_audit` | Manages vendor audit methods, schedules, results, deficiency counts, document versions, and progress statuses |

---

## 5. URS and FDS Management
<img width="4440" height="3366" alt="image" src="https://github.com/user-attachments/assets/a424ad13-1d60-4e0d-8f99-97d72d1926c1" />
Link : https://drawsql.app/teams/minho-kim/diagrams/05-urs-and-fds-management

### Structural Overview

This structure organizes validation projects around systems belonging to an organization and supports the creation and management of user requirements and functional design specifications for each project.

`requirement` manages URS requirements and their revision versions, while `fds_spec` manages FDS document versions and statuses. `fds_item` and `fds_interface` respectively manage the functional and screen items included in FDS documents and the designs of interfaces between systems.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about the client companies or operating organizations to which systems and users belong |
| `app_user` | Manages information about users who create or modify projects, URS and FDS documents, and their detailed items |
| `system_asset` | Manages identification information, asset management numbers, GAMP categories, GxP classifications, and statuses for systems or equipment subject to validation |
| `validation_project` | Manages the scope, validation approach, progress percentage, and status of validation projects containing URS and FDS documents |
| `requirement` | Manages item numbers, categories, detailed descriptions, regulatory references, revision versions, and statuses of URS requirements for each project |
| `fds_spec` | Manages document numbers, titles, versions, revision sequence numbers, and authoring, review, and approval statuses of FDS documents for each project |
| `fds_item` | Manages item classifications, management numbers, function names, detailed descriptions, and related screens for functional, screen, and interface items included in FDS documents |
| `fds_interface` | Manages sending and receiving systems, exchanged data, transmission frequencies, transmission methods, and related FDS numbers for each inter-system interface in an FDS document |

---

## 6. DDS and DQ Management
<img width="5840" height="4046" alt="image" src="https://github.com/user-attachments/assets/0389d811-f499-4e5f-9db0-530dfa497fce" />
Link : https://drawsql.app/teams/minho-kim/diagrams/06-dds-and-dq-management

### Structural Overview

This structure organizes validation projects around systems belonging to an organization, supports the creation of DDS detailed designs based on URS and FDS, and subsequently assesses design qualification.

`dds_spec` and `dds_item` manage detailed designs for databases, components, interfaces, security, batch processing, and other areas. `dq_assessment` and `dq_item` assess whether URS requirements are appropriately reflected in the FDS and DDS designs and manage the assessment results.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about the client companies or operating organizations to which systems and users belong |
| `app_user` | Manages information about users who create, modify, or review projects, FDS, DDS, and DQ documents, and their detailed items |
| `system_asset` | Manages identification information, asset management numbers, GAMP categories, GxP classifications, and statuses for systems or equipment subject to validation |
| `validation_project` | Manages the scope, validation approach, progress percentage, and status of validation projects containing FDS, DDS, and DQ documents |
| `requirement` | Manages project-specific URS requirements, revision versions, and statuses that serve as the basis for DQ assessments |
| `fds_spec` | Manages the numbers, versions, revision sequence numbers, and statuses of FDS documents that serve as the basis for DDS preparation and DQ assessments |
| `fds_item` | Manages detailed functionality and design content for each function, screen, and interface in an FDS document |
| `dds_spec` | Manages document numbers, titles, versions, revision sequence numbers, and approval statuses of DDS documents for each project |
| `dds_item` | Manages detailed design items for databases, components, interfaces, security, and batch processing included in DDS documents |
| `dq_assessment` | Manages document numbers, versions, revision sequence numbers, and statuses of design qualification assessment documents for each project |
| `dq_item` | Manages the relationships between URS requirements and FDS and DDS designs, qualification assessment results, reviewers, and remarks |

---

## 7. FRA Risk Assessment Management
<img width="2575" height="2129" alt="image" src="https://github.com/user-attachments/assets/077a700f-9e8b-483f-ac75-e73a29f8a150" />
Link : https://drawsql.app/teams/minho-kim/diagrams/07-fra-risk-assessment-management

### Structural Overview

This structure organizes validation projects around systems belonging to an organization and assesses functional risks in relation to each project's URS requirements.

`fra_assessment` manages FRA document versions and statuses, while `fra_item` manages risk scenarios, product impact, likelihood of occurrence, detectability, risk scores, and mitigation strategies for each function.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about the client companies or operating organizations to which systems and users belong |
| `app_user` | Manages information about users who create or modify projects, FRA documents, and risk items |
| `system_asset` | Manages identification information, asset management numbers, GAMP categories, GxP classifications, and statuses for systems or equipment subject to validation |
| `validation_project` | Manages the scope, validation approach, progress percentage, and status of validation projects containing FRA documents |
| `requirement` | Manages project-specific URS requirements and revision versions that serve as the basis for assessing FRA risk items |
| `fra_assessment` | Manages document numbers, titles, versions, revision sequence numbers, and statuses of FRA documents for each project |
| `fra_item` | Manages risk scenarios, PI, LL, and DL assessment values, risk scores, risk ratings, mitigation strategies, and linked tests for each function |

---

## 8. IQ, OQ, and PQ Qualification Testing Management
<img width="3070" height="2543" alt="image" src="https://github.com/user-attachments/assets/dcbb8a59-325e-4e99-9be1-873c70aabc53" />
Link : https://drawsql.app/teams/minho-kim/diagrams/08-iq-oq-and-pq-qualification-testing-management

### Structural Overview

This structure supports the creation of IQ, OQ, and PQ protocols and detailed test items for systems belonging to an organization and their validation projects, and manages test execution results and deviations.

Document headers and detailed test items are managed separately for each qualification assessment. Following protocol approval, the executor, actual results, outcome, and execution timestamp are recorded. Issues arising during test execution are managed in `deviation` through investigation, resolution, and closure statuses.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about the client companies or operating organizations to which systems and users belong |
| `app_user` | Manages information about users who create or modify qualification documents and execute tests |
| `system_asset` | Manages identification information and statuses for systems or equipment subject to IQ, OQ, and PQ |
| `validation_project` | Manages the scope, validation approach, and progress status of validation projects containing IQ, OQ, and PQ documents |
| `iq_assessment` | Manages document numbers, versions, revision information, protocol statuses, and execution record statuses for IQ documents |
| `iq_item` | Manages IQ installation verification procedures, expected results, actual results, outcomes, executors, and execution timestamps |
| `oq_assessment` | Manages document numbers, versions, revision information, protocol statuses, and execution record statuses for OQ documents |
| `oq_item` | Manages OQ operational function test procedures, expected results, actual results, outcomes, executors, and execution timestamps |
| `pq_assessment` | Manages execution plans, schedules, execution methods, versions, protocol statuses, and execution record statuses for PQ documents |
| `pq_item` | Manages PQ performance test procedures, expected results, actual results, outcomes, executors, and execution timestamps |
| `deviation` | Manages descriptions, severity levels, investigation and resolution statuses, and closure approval statuses of deviations arising in documents or during test execution |

---

## 9. Design and Risk Deliverable Traceability Management
<img width="4360" height="5206" alt="image" src="https://github.com/user-attachments/assets/a1587eae-c5f3-47d7-9309-7495b8c0044d" />
Link : https://drawsql.app/teams/minho-kim/diagrams/09-design-and-risk-deliverable-traceability-management

### Structural Overview

This structure manages traceability relationships among a validation project's URS requirements, FDS and DDS designs, DQ assessments, and FRA risk assessment deliverables.

The structure includes both document headers and detailed items, while `traceability_link` connects deliverable items through implementation, assessment, and risk mitigation relationships using polymorphic references.

### Table Role Summary

| Table | Role |
|---|---|
| `app_user` | Manages information about users who create or modify design and risk deliverables and traceability relationships |
| `validation_project` | Manages validation projects containing design and risk deliverables and traceability relationships |
| `requirement` | Manages project-specific URS requirements and revision versions that serve as the basis for traceability |
| `fds_spec` | Manages document numbers, versions, revision information, and statuses of FDS documents for each project |
| `fds_item` | Manages detailed FDS items that implement URS requirements through functional, screen, and interface designs |
| `dds_spec` | Manages document numbers, versions, revision information, and statuses of DDS documents for each project |
| `dds_item` | Manages detailed design items for databases, components, interfaces, security, and batch processing |
| `dq_assessment` | Manages versions, revision information, and statuses of design qualification assessment documents for each project |
| `dq_item` | Manages relationships between URS requirements and FDS and DDS designs, along with design qualification assessment results |
| `fra_assessment` | Manages versions, revision information, and statuses of functional risk assessment documents for each project |
| `fra_item` | Manages risk scenarios, risk scores, risk ratings, and mitigation strategies for each URS requirement or function |
| `traceability_link` | Manages implementation, assessment, and mitigation relationships among URS, FDS, DDS, DQ, and FRA items using polymorphic references |

---

## 10. Qualification Test Traceability and RTM Management
<img width="2840" height="2703" alt="image" src="https://github.com/user-attachments/assets/2173e7a8-503a-42f0-b909-66593b263fa5" />
Link : https://drawsql.app/teams/minho-kim/diagrams/10-qualification-test-traceability-and-rtm-management

### Structural Overview

This structure manages traceability relationships between a validation project's URS requirements and IQ, OQ, and PQ test items, and uses these relationships to manage the requirements traceability matrix and deliverable coverage.

`traceability_link` manages verification relationships between URS requirements and qualification test items, while `rtm_assessment` and `rtm_item` preserve traceability results at the project and individual URS requirement levels, along with RTM snapshots captured at the time of approval.

### Table Role Summary

| Table | Role |
|---|---|
| `app_user` | Manages information about users who create or modify qualification tests, traceability relationships, and RTM documents |
| `validation_project` | Manages validation projects containing qualification tests and RTM documents |
| `requirement` | Manages project-specific URS requirements that serve as the basis for test traceability and the RTM |
| `iq_assessment` | Manages versions, protocol statuses, and execution record statuses of IQ documents for each project |
| `iq_item` | Manages IQ installation verification procedures and test results used to verify URS requirements |
| `oq_assessment` | Manages versions, protocol statuses, and execution record statuses of OQ documents for each project |
| `oq_item` | Manages OQ operational function test procedures and test results used to verify URS requirements |
| `pq_assessment` | Manages PQ execution plans, document versions, and test progress statuses for each project |
| `pq_item` | Manages PQ performance test procedures and results under actual operating conditions |
| `traceability_link` | Manages verification relationships between URS requirements and IQ, OQ, and PQ test items using polymorphic references |
| `rtm_assessment` | Manages project-level RTM snapshots of URS requirement counts, FRA linkage rates, IQ and OQ coverage, and overall average coverage |
| `rtm_item` | Manages detailed snapshots of FRA, FDS, and DDS mappings, IQ, OQ, and PQ results, and item-level coverage for each URS requirement |

---

## 11. VSR and Deviation Management
<img width="4760" height="4306" alt="image" src="https://github.com/user-attachments/assets/d5c6ffd5-1702-4f1a-8de3-8585984fc5b2" />
Link : https://drawsql.app/teams/minho-kim/diagrams/11-vsr-and-deviation-management

### Structural Overview

This structure consolidates activity execution results, RTM coverage, and deviation statuses for a validation project to manage the final validation conclusion and VSR.

`project_activity` manages the progress status of each project's activities, and `deviation` identifies whether any unresolved deviations remain. `vsr_assessment` and `vsr_item` summarize the project's final conclusion and the document, test, deviation, and approval results for each activity.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about the client companies or operating organizations to which systems and users belong |
| `app_user` | Manages information about users who create, modify, or process project activities, deviations, and VSR documents |
| `system_asset` | Manages identification information for systems or equipment subject to validation and covered by a VSR |
| `validation_project` | Manages validation projects containing activities, deviations, RTM documents, and VSR documents |
| `validation_activity` | Manages codes, names, and the default order of validation activities summarized in the VSR |
| `project_activity` | Manages whether each project activity is selected, whether it is mandatory, its progress status, and its approval timestamp |
| `rtm_assessment` | Manages project-specific requirements traceability and coverage results referenced by the VSR |
| `deviation` | Manages severity levels, statuses, resolution details, and closure approvals for deviations arising in documents or during test execution |
| `vsr_assessment` | Manages the final validation conclusion, detailed conclusion, version, and approval status for each project |
| `vsr_item` | Manages summaries of document numbers, revision numbers, result counts, deviations, and approval information for each validation activity |

---

## 12. Workflow, Approval, and Electronic Signature Management
<img width="4640" height="3766" alt="image" src="https://github.com/user-attachments/assets/55c8d805-4684-4d27-9288-ad118a65bc37" />
Link : https://drawsql.app/teams/minho-kim/diagrams/12-workflow-approval-and-electronic-signature-management

### Structural Overview

This structure manages workflows and electronic signature evidence for validation project documents, from submission through review, approval, or rejection at each step.

Responsible users are assigned to each step based on project participants and roles. `workflow_instance`, `workflow_step`, and `approval_action` manage the overall workflow and the history of actions actually performed. Electronic signatures generated upon approval or rejection are preserved together with the target document version and content hash.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about the client companies or operating organizations to which systems and users belong |
| `app_user` | Manages information about document submitters, step assignees, users who perform actions, and electronic signers |
| `role` | Maintains reference definitions for roles applied to project participants and workflow assignees |
| `system_asset` | Manages information about systems or equipment associated with validation projects subject to review and approval |
| `validation_project` | Manages validation projects to which workflows and project participants belong |
| `project_member` | Manages participating users, assigned roles, participation statuses, and participation periods for each project |
| `workflow_instance` | Manages submission information, the current step, and progress status of the overall review and approval workflow for each target document |
| `workflow_step` | Manages step order, step types, assignees, due dates, and individual step statuses within a workflow |
| `approval_action` | Manages details of actions performed at each step, such as submission, review, approval, rejection, and cancellation, along with the users who performed them and their timestamps |
| `electronic_signature` | Manages signers, signature meanings, target versions, content hashes, and reauthentication results for review, approval, and rejection actions |

---

## 13. File, Evidence, and File Cleanup Management
<img width="2500" height="1791" alt="image" src="https://github.com/user-attachments/assets/be3ac81c-3cb4-46c1-b19e-80c1cc60e362" />
Link : https://drawsql.app/teams/minho-kim/diagrams/13-file-evidence-and-file-cleanup-management

### Structural Overview

This structure manages metadata for attachments and test evidence used in validation projects, their relationships with business entities, and cleanup operations for temporary and expired files.

`file_asset` manages metadata and cleanup statuses for stored files, while `evidence_link` manages relationships between evidence files and documents, test items, or deviations. `file_cleanup_execution` manages the history of cleanup candidate searches, deletion processing, failures, and retries.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about organizations to which users and systems associated with files and projects belong |
| `app_user` | Manages information about file uploaders, users requesting file cleanup operations, and users performing those operations |
| `system_asset` | Manages information about systems or equipment subject to validation for which files and evidence are generated |
| `validation_project` | Manages validation projects to which evidence file links belong |
| `file_cleanup_execution` | Manages execution modes, statuses, processed item counts, failures, and retry history for cleanup operations involving temporary, expired, and orphaned files |
| `file_asset` | Manages storage paths, sizes, formats, expiration information, and cleanup statuses for attachments, evidence files, reports, and exported files |
| `evidence_link` | Manages N:M evidence relationships between files and documents, test items, or deviations using polymorphic references |

---

## 14. Report, Notification, and Backup Operations Management
<img width="2800" height="2423" alt="image" src="https://github.com/user-attachments/assets/5643c343-a8e0-4bf0-a773-d2411268b98b" />
Link : https://drawsql.app/teams/minho-kim/diagrams/14-report-notification-and-backup-operations-management

### Structural Overview

This structure manages scheduled and manual generation of operational reports, delivery of review and approval notifications, and execution of system data and file backups in the context of validation projects and workflows.

`report_schedule` and `report_generation` manage report schedules and generation jobs, with generated output files linked to `file_asset`. `notification_delivery` manages delivery and retries of workflow-related notifications, while `backup_execution` manages the execution statuses and results of scheduled and manual backups.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about the client companies or operating organizations to which operational reports and users belong |
| `app_user` | Manages information about report and backup requesters, notification recipients, and users who create or modify operational jobs |
| `system_asset` | Manages information about systems or equipment covered by operational reports and backups |
| `validation_project` | Manages validation projects that serve as the basis for project-level reports, notifications, and operational jobs |
| `workflow_instance` | Manages information about the overall review and approval workflow for each document subject to notifications |
| `workflow_step` | Manages step assignees and statuses used as the basis for approval request, upcoming deadline, and overdue notifications |
| `file_asset` | Manages storage paths, sizes, formats, and cleanup statuses of generated report output files |
| `report_schedule` | Manages report types, execution frequencies, reporting periods, output formats, and next execution timestamps for scheduled reports |
| `report_generation` | Manages report generation requests, query criteria, execution statuses, output files, failures, and retry history |
| `notification_delivery` | Manages recipients, channels, delivery statuses, failures, and retry history for review and approval requests and deadline-related notifications |
| `backup_execution` | Manages backup types, targets, statuses, storage locations, sizes, and retry history for scheduled and manual database and file backups |

---

## 15. AI Generation Job and Result Management
<img width="2488" height="2229" alt="image" src="https://github.com/user-attachments/assets/52061c1a-be1e-4375-a75b-1bd04c5ad0d5" />
Link : https://drawsql.app/teams/minho-kim/diagrams/15-ai-generation-job-and-result-management

### Structural Overview

This structure manages AI generation requests for validation project documents and items as asynchronous jobs, and tracks the selection of generated results and whether they have been applied to actual business data.

`ai_generation_job` manages AI models, input parameters, execution statuses, and retries. `ai_generation_result` and `ai_result_item` manage the selection and adoption of generated result sets and detailed items, along with information about their application targets.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about organizations to which users of AI features and target systems belong |
| `app_user` | Manages information about AI generation requesters, users selecting results, and users who create or modify jobs |
| `system_asset` | Manages information about systems or equipment subject to validation with which documents targeted for AI generation are associated |
| `validation_project` | Manages validation projects containing AI generation jobs and the target business data |
| `ai_generation_job` | Manages AI generation job types, target entities, models, input parameters, execution statuses, failures, and retry history |
| `ai_generation_result` | Manages titles, selection statuses, application statuses, selecting users, and selection timestamps for result sets generated by AI jobs |
| `ai_result_item` | Manages the content and application targets of individual AI-generated requirements, risk scenarios, test items, or document sections |

---

## 16. Audit Trail Management
<img width="1563" height="904" alt="image" src="https://github.com/user-attachments/assets/60be2276-6f0d-407e-987f-4a1f4b0a6fdb" />
Link : https://drawsql.app/teams/minho-kim/diagrams/16-audit-trail-management

### Structural Overview

This structure preserves significant data changes made by users belonging to an organization, along with the execution history of system and batch operations, for auditing purposes.

`audit_trail` manages operation types, target tables and records, before and after values, reasons for changes, actors, request and session information, and target document versions. Audited entities are identified using polymorphic references, and system or batch operations can be recorded without an associated user.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Maintains reference information about the client companies or operating organizations to which audited users belong |
| `app_user` | Manages account and organizational affiliation information for users who make significant data changes |
| `audit_trail` | Manages actors, targets, before and after values, reasons for changes, request and session information, and document versions for data creation, modification, deletion, and operational activities |
