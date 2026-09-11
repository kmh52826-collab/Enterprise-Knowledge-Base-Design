# ERD Specifications

This is an ERD specification document that organizes the overall data model of the Validation Management Platform by business domain.

The complex overall schema is divided into 16 topics, with the major tables and relationships in each domain provided as ERD images. Each section also summarizes an overview of the domain structure and the role of each table.

> [!NOTE]
> This repository is a public resource for sharing data modeling and system design examples.
>
> The organization names, user information, system names, project names, table names, column names, example values, and other information used in this document do not represent actual operational data or internal system structures of any specific company. Names and examples have been generalized or arbitrarily constructed for public disclosure purposes.
>
> When applying this model to an actual production environment, organizational business policies, security standards, personal information protection requirements, data retention policies, and applicable regulatory requirements should be reviewed separately.

> **When detailed column information is required**  
> Please refer to the [Data Dictionary](./data-dictionary.md) for the columns, data types, PK·FK, nullability, default values, and business rules of each table.

## Table of Contents

1. [Organization·User·Global Role Management](#1-organizationuserglobal-role-management)
2. [System·Project·Participant Management](#2-systemprojectparticipant-management)
3. [Validation Activity·Dependency Management](#3-validation-activitydependency-management)
4. [Library·QIA·Vendor Audit Management](#4-libraryqiavendor-audit-management)
5. [URS·FDS Management](#5-ursfds-management)
6. [DDS·DQ Management](#6-ddsdq-management)
7. [FRA Risk Assessment Management](#7-fra-risk-assessment-management)
8. [IQ·OQ·PQ Qualification Testing Management](#8-iqoqpq-qualification-testing-management)
9. [Design·Risk Deliverable Traceability Management](#9-designrisk-deliverable-traceability-management)
10. [Qualification Test Traceability·RTM Management](#10-qualification-test-traceabilityrtm-management)
11. [VSR·Deviation Management](#11-vsrdeviation-management)
12. [Workflow·Approval·Electronic Signature Management](#12-workflowapprovalelectronic-signature-management)
13. [File·Evidence·File Cleanup Management](#13-fileevidencefile-cleanup-management)
14. [Report·Notification·Backup Operations Management](#14-reportnotificationbackup-operations-management)
15. [AI Generation Job·Result Management](#15-ai-generation-jobresult-management)
16. [Audit Trail Management](#16-audit-trail-management)

---

## 1. Organization·User·Global Role Management
<img width="1650" height="1066" alt="image" src="https://github.com/user-attachments/assets/d1f9d215-dafa-46ae-b7ff-75f4bad5b622" />
Link : https://drawsql.app/teams/minho-kim/diagrams/01-organization-user-and-global-role-management

### Structure Overview

This structure manages user accounts based on organizations and grants global permissions to individual users through role masters and user-role mappings.

Global roles are permissions applied across the entire system, while project-specific participation and responsibilities are managed separately through the `project_member` table.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages basic information of the customer or operating organization and serves as the reference table for identifying user affiliation |
| `app_user` | Manages user accounts, basic profiles, affiliated organizations, and account status |
| `role` | Defines global roles applied across the system and their permission scope |
| `user_role` | Manages the many-to-many mapping between users and global roles and prevents duplicate assignment of the same role |

---

## 2. System·Project·Participant Management
<img width="2000" height="1491" alt="image" src="https://github.com/user-attachments/assets/b8625331-f875-41a2-9546-dc2ef32557cd" />
Link : https://drawsql.app/teams/minho-kim/diagrams/02-system-project-and-participant-management

### Structure Overview

This structure configures Validation projects based on systems and equipment belonging to an organization and manages project-specific users and their responsibilities.

Each project is connected to one target system, and `project_member` maps users and roles at the project level. This allows the same user to perform different roles across different projects.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages the reference information of the customer or operating organization to which systems and users belong |
| `system_asset` | Manages the management number, type, responsible department, GAMP category, GxP classification, and identification status of the system or equipment subject to Validation |
| `validation_project` | Manages the scope, validation approach, progress, status, and GAMP category of Validation projects for each target system |
| `project_member` | Connects project participants with their assigned roles and manages participation status and participation period |
| `app_user` | Manages user accounts and basic profiles of users who participate in, create, or modify projects |
| `role` | Manages reference information for roles such as author, reviewer, and approver assigned to project participants |

---

## 3. Validation Activity·Dependency Management
<img width="2500" height="1729" alt="image" src="https://github.com/user-attachments/assets/3648a194-0317-43eb-aa29-fd262b715add" />
Link : https://drawsql.app/teams/minho-kim/diagrams/03-validation-activity-and-dependency-management

### Structure Overview

This structure creates Validation projects based on systems belonging to an organization and manages project-specific activities and dependencies between activities.

`validation_activity` defines the overall Validation activity standards, while `project_activity` manages project-specific activities, required status, and progress status. `activity_dependency` manages the criteria for determining whether a subsequent activity can be activated based on the status or business conditions of a preceding activity.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages the reference information of the customer or operating organization to which systems and users belong |
| `app_user` | Manages user information for registering and modifying projects, activities, and activity dependencies |
| `system_asset` | Manages identification information, GAMP category, GxP classification, and status of the system or equipment subject to Validation |
| `validation_project` | Manages the scope, validation approach, progress, and status of Validation projects for each target system |
| `validation_activity` | Manages reference information and default display order for Validation activities such as VP, QIA, VA, URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ, RTM, and VSR |
| `project_activity` | Manages project-specific activities, required status, activation status, and progress status |
| `activity_dependency` | Manages activation conditions for subsequent activities based on preceding activities, required status, condition type, and evaluation order |

---

## 4. Library·QIA·Vendor Audit Management
<img width="2600" height="1866" alt="image" src="https://github.com/user-attachments/assets/0fa424a7-ec2f-4589-8b5e-75b20e7eefa0" />
Link : https://drawsql.app/teams/minho-kim/diagrams/04-library-qia-and-vendor-audit-management

### Structure Overview

This structure configures Validation projects based on systems belonging to an organization and manages reusable standard libraries, project-specific quality impact assessments, and vendor audit results.

`library_item` manages standard items used for authoring URS·IQ·OQ. `qia_assessment` and `qia_module_item` assess the GxP and 21 CFR Part 11 scope of the project, while `vendor_audit` manages the audit plan, results, and defect counts for the vendor of the target system.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages the reference information of the customer or operating organization to which systems and users belong |
| `app_user` | Manages user accounts and basic profiles of users who author and manage projects and assessment documents |
| `system_asset` | Manages identification information, vendor, GAMP category, GxP classification, and status of the system or equipment subject to Validation |
| `validation_project` | Manages the scope, validation approach, progress, and status of Validation projects for each target system |
| `library_item` | Manages standard requirements, test procedures, expected results, acceptance criteria, and supporting regulations to be reused in URS·IQ·OQ |
| `qia_assessment` | Manages project-specific 21 CFR Part 11 applicability, GxP scope, document version, and assessment status |
| `qia_module_item` | Manages detailed GxP assessment items and assessment results for each module or process in the QIA document |
| `vendor_audit` | Manages vendor audit methods, schedules, audit results, defect counts, document versions, and progress status |

---

## 5. URS·FDS Management
<img width="4440" height="3366" alt="image" src="https://github.com/user-attachments/assets/a424ad13-1d60-4e0d-8f99-97d72d1926c1" />
Link : https://drawsql.app/teams/minho-kim/diagrams/05-urs-and-fds-management

### Structure Overview

This structure configures Validation projects based on systems belonging to an organization and creates and manages project-specific user requirements and functional design specifications.

`requirement` manages URS requirements and revision versions, while `fds_spec` manages the versions and status of FDS documents. `fds_item` and `fds_interface` respectively manage the functional/screen items included in FDS documents and the design of interfaces between systems.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages the reference information of the customer or operating organization to which systems and users belong |
| `app_user` | Manages user information for authoring and modifying projects, URS and FDS documents, and detailed items |
| `system_asset` | Manages identification information, management number, GAMP category, GxP classification, and status of the system or equipment subject to Validation |
| `validation_project` | Manages the scope, validation approach, progress, and status of the Validation project to which URS and FDS belong |
| `requirement` | Manages item numbers, categories, detailed contents, supporting regulations, revision versions, and status of project-specific URS requirements |
| `fds_spec` | Manages document number, title, version, revision sequence, and authoring/review/approval status of project-specific FDS documents |
| `fds_item` | Manages the classification, management number, function name, detailed description, and related screens of functional, screen, and interface items included in FDS documents |
| `fds_interface` | Manages the transmitting and receiving systems, integrated data, transmission frequency, transmission method, and related FDS number for each system interface in the FDS document |

---

## 6. DDS·DQ Management
<img width="5840" height="4046" alt="image" src="https://github.com/user-attachments/assets/0389d811-f499-4e5f-9db0-530dfa497fce" />
Link : https://drawsql.app/teams/minho-kim/diagrams/06-dds-and-dq-management

### Structure Overview

This structure configures Validation projects based on systems belonging to an organization, creates detailed DDS designs based on URS and FDS, and evaluates design qualification.

`dds_spec` and `dds_item` manage detailed designs such as databases, components, interfaces, security, and batch processes. `dq_assessment` and `dq_item` evaluate whether URS requirements have been appropriately reflected in FDS and DDS designs and manage the resulting assessments.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages the reference information of the customer or operating organization to which systems and users belong |
| `app_user` | Manages user information for authoring, modifying, and reviewing projects, FDS·DDS·DQ documents, and detailed items |
| `system_asset` | Manages identification information, management number, GAMP category, GxP classification, and status of the system or equipment subject to Validation |
| `validation_project` | Manages the scope, validation approach, progress, and status of Validation projects to which FDS·DDS·DQ belong |
| `requirement` | Manages project-specific URS requirements, revision versions, and status used as the basis for DQ assessment |
| `fds_spec` | Manages the document number, version, revision sequence, and status of FDS documents serving as the basis for DDS authoring and DQ assessment |
| `fds_item` | Manages detailed functions and design contents for each function, screen, and interface in the FDS document |
| `dds_spec` | Manages the document number, title, version, revision sequence, and approval status of project-specific DDS documents |
| `dds_item` | Manages detailed design items for databases, components, interfaces, security, and batch processes included in DDS documents |
| `dq_assessment` | Manages the document number, version, revision sequence, and status of project-specific design qualification assessment documents |
| `dq_item` | Manages the linkage between URS requirements and FDS·DDS designs, qualification assessment results, reviewers, and remarks |

---

## 7. FRA Risk Assessment Management
<img width="2575" height="2129" alt="image" src="https://github.com/user-attachments/assets/077a700f-9e8b-483f-ac75-e73a29f8a150" />
Link : https://drawsql.app/teams/minho-kim/diagrams/07-fra-risk-assessment-management

### Structure Overview

This structure configures Validation projects based on systems belonging to an organization and evaluates functional risks in association with project-specific URS requirements.

`fra_assessment` manages the version and status of FRA documents, while `fra_item` manages functional risk scenarios, product impact, likelihood of occurrence, detectability, risk level, and mitigation strategies.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages the reference information of the customer or operating organization to which systems and users belong |
| `app_user` | Manages user information for authoring and modifying projects, FRA documents, and risk items |
| `system_asset` | Manages identification information, management number, GAMP category, GxP classification, and status of the system or equipment subject to Validation |
| `validation_project` | Manages the scope, validation approach, progress, and status of the Validation project to which FRA belongs |
| `requirement` | Manages project-specific URS requirements and revision versions that serve as the evaluation basis for FRA risk items |
| `fra_assessment` | Manages the number, title, version, revision sequence, and status of project-specific FRA documents |
| `fra_item` | Manages functional risk scenarios, PI·LL·DL assessment values, risk level, risk classification, mitigation strategies, and linked tests |

---

## 8. IQ·OQ·PQ Qualification Testing Management
<img width="3070" height="2543" alt="image" src="https://github.com/user-attachments/assets/dcbb8a59-325e-4e99-9be1-873c70aabc53" />
Link : https://drawsql.app/teams/minho-kim/diagrams/08-iq-oq-and-pq-qualification-testing-management

### Structure Overview

This structure creates IQ·OQ·PQ protocols and detailed test items based on systems and Validation projects belonging to an organization and manages test execution results and deviations.

The document headers and detailed test items for each qualification assessment are managed separately, while executors, actual results, judgments, and execution timestamps are recorded after protocol approval. Issues arising during test execution are managed through `deviation` with investigation, resolution, and closure statuses.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages the reference information of the customer or operating organization to which systems and users belong |
| `app_user` | Manages user information for authoring and modifying qualification documents and performing tests |
| `system_asset` | Manages identification information and status of the system or equipment subject to IQ·OQ·PQ |
| `validation_project` | Manages the scope, validation approach, and progress status of the Validation project to which IQ·OQ·PQ belong |
| `iq_assessment` | Manages the document number, version, revision information, protocol status, and execution record status of IQ documents |
| `iq_item` | Manages IQ installation verification procedures, expected results, actual results, judgments, executors, and execution timestamps |
| `oq_assessment` | Manages the document number, version, revision information, protocol status, and execution record status of OQ documents |
| `oq_item` | Manages OQ operational function test procedures, expected results, actual results, judgments, executors, and execution timestamps |
| `pq_assessment` | Manages PQ execution plans, schedules, execution methods, versions, protocol status, and execution record status |
| `pq_item` | Manages PQ performance test procedures, expected results, actual results, judgments, executors, and execution timestamps |
| `deviation` | Manages the contents, severity, investigation, resolution, and closure approval status of deviations occurring during document or test execution |

---

## 9. Design·Risk Deliverable Traceability Management
<img width="4360" height="5206" alt="image" src="https://github.com/user-attachments/assets/a1587eae-c5f3-47d7-9309-7495b8c0044d" />
Link : https://drawsql.app/teams/minho-kim/diagrams/09-design-and-risk-deliverable-traceability-management

### Structure Overview

This structure manages traceability relationships among URS requirements, FDS·DDS designs, DQ assessments, and FRA risk assessment deliverables within a Validation project.

Each document is structured together with its detailed items, while `traceability_link` connects implementation, assessment, and risk mitigation relationships between deliverable items using a polymorphic reference approach.

### Table Role Summary

| Table | Role |
|---|---|
| `app_user` | Manages user information for authoring and modifying design/risk deliverables and traceability relationships |
| `validation_project` | Manages the Validation project to which design/risk deliverables and traceability relationships belong |
| `requirement` | Manages project-specific URS requirements and revision versions that serve as the basis for traceability |
| `fds_spec` | Manages the number, version, revision information, and status of project-specific FDS documents |
| `fds_item` | Manages detailed FDS items that implement URS into functional, screen, and interface designs |
| `dds_spec` | Manages the number, version, revision information, and status of project-specific DDS documents |
| `dds_item` | Manages detailed design items for databases, components, interfaces, security, and batch processes |
| `dq_assessment` | Manages the version, revision information, and status of project-specific design qualification assessment documents |
| `dq_item` | Manages linkage contents between URS and FDS·DDS designs and design qualification assessment results |
| `fra_assessment` | Manages the version, revision information, and status of project-specific functional risk assessment documents |
| `fra_item` | Manages URS or function-specific risk scenarios, risk levels, risk classifications, and mitigation strategies |
| `traceability_link` | Manages implementation, assessment, and mitigation relationships among URS, FDS, DDS, DQ, and FRA items using a polymorphic reference approach |

---

## 10. Qualification Test Traceability·RTM Management
<img width="2840" height="2703" alt="image" src="https://github.com/user-attachments/assets/2173e7a8-503a-42f0-b909-66593b263fa5" />
Link : https://drawsql.app/teams/minho-kim/diagrams/10-qualification-test-traceability-and-rtm-management

### Structure Overview

This structure manages traceability relationships between URS requirements and IQ·OQ·PQ test items within a Validation project and manages requirements traceability matrices and deliverable coverage based on these relationships.

`traceability_link` manages verification relationships between URS and qualification test items, while `rtm_assessment` and `rtm_item` preserve project- and URS-specific traceability results and RTM snapshots at the time of approval.

### Table Role Summary

| Table | Role |
|---|---|
| `app_user` | Manages user information for authoring and modifying qualification tests, traceability relationships, and RTM documents |
| `validation_project` | Manages the Validation project to which qualification tests and RTM belong |
| `requirement` | Manages project-specific URS requirements that serve as the basis for test traceability and RTM |
| `iq_assessment` | Manages project-specific IQ document versions, protocol status, and execution record status |
| `iq_item` | Manages IQ installation verification procedures and test results for URS verification |
| `oq_assessment` | Manages project-specific OQ document versions, protocol status, and execution record status |
| `oq_item` | Manages OQ operational function test procedures and test results for URS verification |
| `pq_assessment` | Manages project-specific PQ execution plans, document versions, and test progress status |
| `pq_item` | Manages PQ performance test procedures and results performed under actual operating conditions |
| `traceability_link` | Manages verification relationships between URS and IQ·OQ·PQ test items using a polymorphic reference approach |
| `rtm_assessment` | Manages RTM snapshots of project-specific URS counts, FRA linkage rate, IQ·OQ coverage, and overall average coverage |
| `rtm_item` | Manages detailed snapshots of URS-specific FRA·FDS·DDS mappings, IQ·OQ·PQ results, and item-level coverage |

---

## 11. VSR·Deviation Management
<img width="4760" height="4306" alt="image" src="https://github.com/user-attachments/assets/d5c6ffd5-1702-4f1a-8de3-8585984fc5b2" />
Link : https://drawsql.app/teams/minho-kim/diagrams/11-vsr-and-deviation-management

### Structure Overview

This structure manages the final Validation conclusion and VSR by consolidating Validation activity execution results, RTM coverage, and deviation status within a Validation project.

`project_activity` manages project-specific activity progress status, while `deviation` is used to identify unresolved deviations. `vsr_assessment` and `vsr_item` summarize the project's final conclusion and document, test, deviation, and approval results by activity.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages the reference information of the customer or operating organization to which systems and users belong |
| `app_user` | Manages user information for authoring, modifying, and processing project activities, deviations, and VSR documents |
| `system_asset` | Manages identification information of the Validation system or equipment subject to VSR |
| `validation_project` | Manages the Validation project to which activities, deviations, RTM, and VSR belong |
| `validation_activity` | Manages the codes, names, and default sequence of Validation activities to be aggregated into VSR |
| `project_activity` | Manages selection status, required status, progress status, and approval timestamps of project-specific activities |
| `rtm_assessment` | Manages project-specific requirements traceability and coverage results referenced by VSR |
| `deviation` | Manages the severity, status, resolution details, and closure approval of deviations occurring during document and test execution |
| `vsr_assessment` | Manages the final Validation conclusion, conclusion details, version, and approval status for each project |
| `vsr_item` | Summarizes document numbers, revision sequences, result counts, deviations, and approval information for each Validation activity |

---

## 12. Workflow·Approval·Electronic Signature Management
<img width="4640" height="3766" alt="image" src="https://github.com/user-attachments/assets/55c8d805-4684-4d27-9288-ad118a65bc37" />
Link : https://drawsql.app/teams/minho-kim/diagrams/12-workflow-approval-and-electronic-signature-management

### Structure Overview

This structure manages the Workflow and electronic signature evidence from document submission through step-by-step review, approval, and rejection within a Validation project.

Step-specific assignees are designated based on project participants and roles, while `workflow_instance`, `workflow_step`, and `approval_action` manage the overall Workflow and actual processing history. Electronic signatures generated during approval or rejection preserve the target document version and content hash together.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages the reference information of the customer or operating organization to which systems and users belong |
| `app_user` | Manages information about document submitters, step assignees, actual processors, and electronic signers |
| `role` | Manages reference information for roles applied to project participants and Workflow assignees |
| `system_asset` | Manages information about systems or equipment in Validation projects subject to review and approval |
| `validation_project` | Manages the Validation project to which Workflow and project participants belong |
| `project_member` | Manages project-specific participating users, assigned roles, participation status, and participation period |
| `workflow_instance` | Manages submission information, current step, and progress status of the overall review and approval Workflow for each target document |
| `workflow_step` | Manages step order, step type, assignee, processing deadline, and step-specific status within the Workflow |
| `approval_action` | Manages actual processing details, processor, and processing timestamp for each step, including submission, review, approval, rejection, and cancellation |
| `electronic_signature` | Manages signer, signature meaning, target version, content hash, and re-authentication result during review, approval, or rejection |

---

## 13. File·Evidence·File Cleanup Management
<img width="2500" height="1791" alt="image" src="https://github.com/user-attachments/assets/be3ac81c-3cb4-46c1-b19e-80c1cc60e362" />
Link : https://drawsql.app/teams/minho-kim/diagrams/13-file-evidence-and-file-cleanup-management

### Structure Overview

This structure manages metadata for attachment files and test evidence used in Validation projects, their relationships with business objects, and cleanup operations for temporary and expired files.

`file_asset` manages metadata and cleanup status of actual stored files, while `evidence_link` manages relationships between documents, test items, deviations, and evidence files. `file_cleanup_execution` manages the history of cleanup target retrieval, deletion processing, failures, and retries.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages organization reference information to which users and systems related to files and projects belong |
| `app_user` | Manages information about file uploaders, file cleanup requesters, and processing users |
| `system_asset` | Manages information about the Validation systems or equipment from which files and evidence originate |
| `validation_project` | Manages the Validation project to which evidence file associations belong |
| `file_cleanup_execution` | Manages execution methods, status, processing counts, failures, and retry history of temporary, expired, and orphaned file cleanup operations |
| `file_asset` | Manages storage paths, sizes, formats, expiration, and cleanup status of attachments, evidence files, reports, and export files |
| `evidence_link` | Manages N:M evidence relationships between files and documents, test items, and deviations using a polymorphic reference approach |

---

## 14. Report·Notification·Backup Operations Management
<img width="2800" height="2423" alt="image" src="https://github.com/user-attachments/assets/5643c343-a8e0-4bf0-a773-d2411268b98b" />
Link : https://drawsql.app/teams/minho-kim/diagrams/14-report-notification-and-backup-operations-management

### Structure Overview

This structure manages scheduled and manual generation of operational reports, delivery of review and approval notifications, and backup execution of system data and files based on Validation projects and Workflows.

`report_schedule` and `report_generation` manage report schedules and generation jobs, while generated result files are linked to `file_asset`. `notification_delivery` manages delivery and retries of Workflow-related notifications, while `backup_execution` manages execution status and results of scheduled and manual backups.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages the reference information of the customer or operating organization to which operational reports and users belong |
| `app_user` | Manages information about report/backup requesters, notification recipients, and users who create or modify operational tasks |
| `system_asset` | Manages information about systems or equipment subject to operational reports and backups |
| `validation_project` | Manages the Validation project serving as the basis for project-level reports, notifications, and operational tasks |
| `workflow_instance` | Manages overall review and approval Workflow information for each document subject to notifications |
| `workflow_step` | Manages step-specific assignees and statuses serving as the basis for approval requests, processing deadline notifications, and delay notifications |
| `file_asset` | Manages storage paths, sizes, formats, and cleanup status of generated report result files |
| `report_schedule` | Manages report type, execution cycle, query period, output format, and next execution time for scheduled reports |
| `report_generation` | Manages report generation requests, query conditions, execution status, result files, failures, and retry history |
| `notification_delivery` | Manages recipients, channels, delivery status, failures, and retry history of notifications related to review/approval requests and processing deadlines |
| `backup_execution` | Manages backup type, target, status, storage location, size, and retry history for scheduled and manual database and file backups |

---

## 15. AI Generation Job·Result Management
<img width="2488" height="2229" alt="image" src="https://github.com/user-attachments/assets/52061c1a-be1e-4375-a75b-1bd04c5ad0d5" />
Link : https://drawsql.app/teams/minho-kim/diagrams/15-ai-generation-job-and-result-management

### Structure Overview

This structure manages AI generation requests for documents and items within Validation projects as asynchronous jobs and manages selection of generated results and whether they are applied to actual business data.

`ai_generation_job` manages AI models, input conditions, execution status, and retries. `ai_generation_result` and `ai_result_item` manage generated result sets and detailed items, including their selection, adoption, and target application information.

### Table Role Summary

| Table | Role |
|---|---|
| `organization` | Manages the reference information of the organization to which users using AI functions and target systems belong |
| `app_user` | Manages information about AI generation requesters, result selectors, and users who create or modify jobs |
| `system_asset` | Manages information about the Validation system or equipment to which AI-generated target documents belong |
| `validation_project` | Manages the Validation project to which AI generation jobs and target business data belong |
| `ai_generation_job` | Manages AI generation job types, target entities, models, input conditions, execution status, failures, and retry history |
| `ai_generation_result` | Manages the title, selection status, application status, selecting user, and selection timestamp of result sets generated by AI jobs |
| `ai_result_item` | Manages the contents and application targets of individual requirements, risk scenarios, test items, or document sections generated by AI |

---

## 16. Audit Trail Management
<img width="1563" height="904" alt="image" src="https://github.com/user-attachments/assets/60be2276-6f0d-407e-987f-4a1f4b0a6fdb" />
Link : https://drawsql.app/teams/minho-kim/diagrams/16-audit-trail-management

### Structure Overview

This structure preserves major data changes performed by users belonging to an organization and the processing history of system and batch operations for audit purposes.

`audit_trail` manages the operation type, target table and record, before/after values, reason for change, actor, request/session information, and target document version. Audit targets are identified using a polymorphic reference approach, and system or batch operations can be recorded without a user.
