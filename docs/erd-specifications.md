# ERD Specifications

This document organizes the complete data model of the Validation Management Platform by business domain.

The complex overall schema is divided into 16 topics, with ERD images showing the main tables and relationships in each area. Each section includes a relationship diagram to help readers understand complex connections at a glance, a structural overview, and a summary of each table’s role.

> [!NOTE]
> This repository is public material intended to share examples of data modeling and system design.
>
> Organization names, user information, system names, project names, table names, column names, sample values, and other content in this document do not represent the actual operational data or internal system structure of any specific company. Names and examples have been generalized or created for public use.
>
> Before applying this model to a production environment, separately review the organization's business policies, security standards, personal data protection requirements, data retention policies, and applicable regulatory requirements.

> **For an overview of the entire structure**  
> Refer to the [Overall ERD Diagram](./erd-overview.md), which groups all 54 tables by business area.

> **For detailed column information**  
> For the columns, data types, PKs and FKs, nullability, default values, constraints, and business rules of each table, refer to the [Data Dictionary](./data-dictionary.md).

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

The diagrams below provide a simple view of the main table connections and business flows. For detailed relationships, refer to the ERD image and link in each area.

## Overall Structure at a Glance

```text
Organization and user setup
    ↓
System and equipment identification
    ↓
Validation project creation
    ↓
Project participant and activity setup
    ↓
QIA and vendor audit
    ↓
URS → FDS → DDS → DQ
    ↓
FRA risk assessment
    ↓
IQ → OQ → PQ qualification testing
    ↓
Traceability verification and RTM generation
    ↓
Deviation review and VSR preparation
    ↓
Project closure
```

The sequence above is an example to help explain the business process, rather than a fixed execution order. The actual activities to perform and their activation conditions are determined by `project_activity` and `activity_dependency`.

Most business data is connected around `validation_project`. Workflow, electronic signatures, evidence files, and Audit Trail are common control functions that apply throughout the process rather than belonging to a single stage.

The main document-based deliverables are managed with separate document headers and detail items.

```text
fds_spec       → fds_item / fds_interface
dds_spec       → dds_item
dq_assessment  → dq_item
fra_assessment → fra_item
iq_assessment  → iq_item
oq_assessment  → oq_item
pq_assessment  → pq_item
rtm_assessment → rtm_item
vsr_assessment → vsr_item
```

> [!IMPORTANT]
> Some relationships in `workflow_instance`, `electronic_signature`, `audit_trail`, `traceability_link`, `evidence_link`, `deviation`, and AI-related tables use polymorphic references that store the target type and target ID together. The validity of target entities not represented by physical FKs must be checked at the application or service layer.

---

## 1. Organization, User, and Global Role Management
<img width="2350" height="1379" alt="image" src="https://github.com/user-attachments/assets/fefb0bf9-5d05-45cb-972d-50fbaabd9745" />
Link : https://drawsql.app/teams/minho-kim/diagrams/01-organization-user-and-global-role-management

### Relationship Structure

```text
organization
  └─ app_user
       ├─ user_role ─ role
       └─ user_group_member ─ user_group
                                  └─ group_role ─ role
                                       └─ validation_project (PROJECT scope)
```

### Structure Overview

This structure manages user accounts by organization and assigns global permissions to individual users through the role master and user-to-role mappings.

`role` is the role master shared by global and project role assignments. `user_role` manages each user’s global roles, while `project_member` manages participation and responsibilities for each project. The scope of group assignments is distinguished by `group_role.scope_type` and `project_id`. `group_role.is_active` indicates whether the role assignment is active.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages basic information for customers or operating organizations and serves as the basis for distinguishing user affiliation |
| `app_user` | Manages user accounts, basic profiles, affiliated organizations, and account status |
| `role` | Manages role codes, names, and descriptions shared by global and project role assignments |
| `user_role` | Maps users to global roles in an N:M relationship and prevents duplicate assignment of the same role |
| `user_group` | Manages organization-specific user group information and active status, serving as the group master for assigning permissions to multiple users at once |
| `user_group_member` | Manages the N:M relationship between users and user groups, including membership status and membership start and end dates |
| `group_role` | Assigns roles to user groups and manages whether each role applies globally or to a specific project |

### Key Points

- `user_role` assigns roles directly to users, while `group_role` applies roles collectively to group members.
- For `GLOBAL` scope, `project_id` is NULL. For `PROJECT` scope, which applies to a specific project, `project_id` is required.
- `group_role` prevents duplicates among non-deleted mappings with `is_active=TRUE`. For `GLOBAL`, uniqueness is based on the group and role combination; for `PROJECT`, it is based on the group, role, and project combination.
- Global permissions and actual project participation roles are separate; project participants are managed in `project_member`.

---

## 2. System, Project, and Participant Management
<img width="2000" height="1491" alt="image" src="https://github.com/user-attachments/assets/b8625331-f875-41a2-9546-dc2ef32557cd" />
Link : https://drawsql.app/teams/minho-kim/diagrams/02-system-project-and-participant-management

### Relationship Structure

```text
organization
  └─ system_asset
       └─ validation_project
            └─ project_member
                 ├─ app_user
                 └─ role
```

### Structure Overview

This structure organizes Validation projects around systems and equipment belonging to an organization and manages participating users and their responsibilities for each project.

Each project is linked to one target system. `project_member` maps users and roles at the project level, allowing the same user to perform different roles in different projects.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages reference information for the customer or operating organization to which systems and users belong |
| `system_asset` | Manages the asset number, type, responsible department, GAMP category, GxP classification, and identification status of a system or equipment subject to Validation |
| `validation_project` | Manages the scope, validation method, progress, status, and GAMP category of a Validation project for a target system |
| `project_member` | Connects participating users to their roles in each project and manages participation status and period |
| `app_user` | Manages user accounts and basic profiles for users who participate in or create and modify projects |
| `role` | Manages reference information for roles assigned to project participants, such as author, reviewer, and approver |

### Key Points

- Multiple Validation projects, such as initial validation, change validation, and revalidation, can be created for a single system or equipment asset.
- The same user can participate in different roles across projects. Duplicate project, user, and role combinations are not allowed among non-deleted membership records with `member_status=ACTIVE`.
- `app_user` and `role` are global reference data, while `project_member` represents the actual role assignment within a specific project.

---

## 3. Validation Activity and Dependency Management
<img width="2500" height="1729" alt="image" src="https://github.com/user-attachments/assets/3648a194-0317-43eb-aa29-fd262b715add" />
Link : https://drawsql.app/teams/minho-kim/diagrams/03-validation-activity-and-dependency-management

### Relationship Structure

```text
validation_project
  └─ project_activity ─ validation_activity

validation_activity
  └─ activity_dependency
       ├─ predecessor_activity_id (predecessor activity)
       └─ successor_activity_id   (successor activity)
```

### Structure Overview

This structure creates Validation projects for systems belonging to an organization and manages project-specific activities and their prerequisite conditions.

`validation_activity` defines the complete activity catalog. `project_activity` manages the activities selected for a project, whether they are mandatory, and their status. `activity_dependency` manages the rules used to decide whether a successor activity can be activated, based on predecessor status or business conditions.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages reference information for the customer or operating organization to which systems and users belong |
| `app_user` | Manages users who create or modify projects, activities, and dependency conditions |
| `system_asset` | Manages identifying information, GAMP category, GxP classification, and status of systems or equipment subject to Validation |
| `validation_project` | Manages the scope, validation method, progress, and status of a Validation project for a target system |
| `validation_activity` | Manages master data and the default display order for Validation activities, including SYSTEM_IDENTIFICATION, VP, QIA, VA, URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ, RTM, and VSR |
| `project_activity` | Manages the activities to be performed in each project, including whether they are mandatory, whether they are active, and their progress status |
| `activity_dependency` | Manages activation conditions for successor activities based on predecessor activities, required status, condition type, and evaluation order |

### Key Points

- `project_activity` manages activities assigned to a project and uses `is_selected` to identify those selected for execution. Duplicate project and activity combinations are not allowed among non-deleted records, regardless of whether they are selected.
- `activity_dependency` expresses business conditions beyond a simple sequence, including approval status, the existence of traceability links, test coverage for high-risk items, and whether unresolved deviations exist.
- `predecessor_activity_id` and `successor_activity_id` reference the same activity master in predecessor and successor roles, respectively. For `STATUS` conditions, the predecessor ID and `required_status` are required; for other conditions, `required_status` is NULL. The predecessor ID may be NULL for conditions without a specific predecessor activity, such as checking whether all activities have been approved.
- The application evaluates conditions based on the project’s valid current revision. `CREATED` means that at least one deliverable specified by `condition_value` exists; the existence of a `project_activity` row alone does not satisfy the condition. System identification and VP activities, which have no document table, use the applicable condition types, such as context confirmation or activity selection.
- `COMPLETED` is satisfied when the predecessor activity status is `COMPLETED` or `APPROVED` and the execution completion aggregation is valid. `APPROVED` is satisfied when the activity status is `APPROVED` and all current target deliverables have received final approval. Approval of a previous revision alone does not satisfy the approval condition for a new revision.
- Evaluate non-deleted conditions with `is_active=TRUE`. All `REQUIRED` conditions for the same successor activity must be satisfied. `RECOMMENDED` conditions are advisory. A `SKIPPED` or unselected state does not automatically satisfy the required status. `ALL_SELECTED_APPROVED` checks selected activities excluding the successor activity itself, and an empty target set does not automatically satisfy the condition.

---

## 4. Library, QIA, and Vendor Audit Management
<img width="2600" height="1866" alt="image" src="https://github.com/user-attachments/assets/0fa424a7-ec2f-4589-8b5e-75b20e7eefa0" />
Link : https://drawsql.app/teams/minho-kim/diagrams/04-library-qia-and-vendor-audit-management

### Relationship Structure

```text
library_item (independent master: standard URS, IQ, and OQ items)

organization
  └─ system_asset
       └─ validation_project
            ├─ qia_assessment
            │    └─ qia_module_item
            └─ vendor_audit
```

### Structure Overview

This structure organizes Validation projects around systems belonging to an organization and manages reusable standard library content, project-specific quality impact assessments, and vendor audit results.

`library_item` manages standard items used to prepare URS, IQ, and OQ content. `qia_assessment` and `qia_module_item` evaluate the project's GxP and 21 CFR Part 11 scope, while `vendor_audit` manages the audit plan, results, and defect counts for the target system's vendor.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages reference information for the customer or operating organization to which systems and users belong |
| `app_user` | Manages user accounts and basic profiles for users who prepare and manage projects and assessment documents |
| `system_asset` | Manages identifying information, vendor, GAMP category, GxP classification, and status of systems or equipment subject to Validation |
| `validation_project` | Manages the scope, validation method, progress, and status of a Validation project for a target system |
| `library_item` | Manages standard requirements, test procedures, expected results, acceptance criteria, and regulatory bases for reuse in URS, IQ, and OQ |
| `qia_assessment` | Manages project-specific applicability of 21 CFR Part 11, GxP scope, document version, and assessment status |
| `qia_module_item` | Manages detailed GxP assessment items and results by module and process within a QIA document |
| `vendor_audit` | Manages vendor audit method, schedule, results, defect counts, document version, and progress status |

### Key Points

- `library_item` has no organization or project FK, and no other table in the current dictionary references `library_item` through an FK. Reusing standard items in business processes is separate from a direct database connection.
- QIA and vendor audits are assessment results associated with a specific project. QIA results are a key basis for determining the subsequent Validation scope and activities.
- A single QIA document can contain multiple module and process assessment items.

---

## 5. URS and FDS Management
<img width="4440" height="3366" alt="image" src="https://github.com/user-attachments/assets/a424ad13-1d60-4e0d-8f99-97d72d1926c1" />
Link : https://drawsql.app/teams/minho-kim/diagrams/05-urs-and-fds-management

### Relationship Structure

```text
validation_project
  ├─ requirement (URS)
  └─ fds_spec
       ├─ fds_item
       └─ fds_interface

requirement ─ traceability_link ─ fds_item (logical connection)
```

### Structure Overview

This structure organizes Validation projects around systems belonging to an organization and manages project-specific user requirements and functional design specifications.

`requirement` manages URS requirements and their revisions, while `fds_spec` manages FDS document versions and status. `fds_item` and `fds_interface` respectively manage the functional or screen items and system-to-system interface designs contained in an FDS document.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages reference information for the customer or operating organization to which systems and users belong |
| `app_user` | Manages users who create and modify projects, URS and FDS documents, and their detail items |
| `system_asset` | Manages identifying information, asset number, GAMP category, GxP classification, and status of systems or equipment subject to Validation |
| `validation_project` | Manages the scope, validation method, progress, and status of the Validation project to which URS and FDS belong |
| `requirement` | Manages item number, category, details, regulatory basis, revision version, and status of project-specific URS requirements |
| `fds_spec` | Manages the document number, title, version, revision number, and authoring, review, and approval status of each project-specific FDS document |
| `fds_item` | Manages classification, control number, function name, detailed description, and related screen for functions, screens, and interface items in an FDS document |
| `fds_interface` | Manages sending and receiving systems, integration data, transmission frequency, transmission method, and related FDS number for each system interface in an FDS document |

### Key Points

- URS defines the functionality users need, and FDS translates those requirements into functional, screen, and interface designs.
- FDS is divided into the document header `fds_spec`, functional details in `fds_item`, and interface details in `fds_interface`.
- Formal traceability between URS and FDS detail items is managed through polymorphic references in `traceability_link`. `fds_interface.source_system` and `target_system` are system-name strings, and `fds_mapping` is also a display value, so they should not be interpreted as FKs to `system_asset` or `fds_item`.

---

## 6. DDS and DQ Management
<img width="5840" height="4046" alt="image" src="https://github.com/user-attachments/assets/0389d811-f499-4e5f-9db0-530dfa497fce" />
Link : https://drawsql.app/teams/minho-kim/diagrams/06-dds-and-dq-management

### Relationship Structure

```text
validation_project
  ├─ requirement (URS)
  ├─ fds_spec
  │    └─ fds_item
  ├─ dds_spec
  │    └─ dds_item
  └─ dq_assessment
       └─ dq_item ─ requirement

URS / FDS / DDS / DQ detail items ─ traceability_link (logical connection)
```

### Structure Overview

This structure organizes Validation projects around systems belonging to an organization, develops detailed DDS designs based on URS and FDS, and then evaluates design qualification.

`dds_spec` and `dds_item` manage detailed designs for databases, components, interfaces, security, and batch processing. `dq_assessment` and `dq_item` evaluate whether URS requirements have been appropriately reflected in FDS and DDS designs and manage the results.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages reference information for the customer or operating organization to which systems and users belong |
| `app_user` | Manages users who create, modify, and review projects and FDS, DDS, and DQ documents and detail items |
| `system_asset` | Manages identifying information, asset number, GAMP category, GxP classification, and status of systems or equipment subject to Validation |
| `validation_project` | Manages the scope, validation method, progress, and status of the Validation project to which FDS, DDS, and DQ belong |
| `requirement` | Manages project-specific URS requirements, revisions, and status used as the basis for DQ evaluation |
| `fds_spec` | Manages the number, version, revision number, and status of the FDS document on which DDS preparation and DQ evaluation are based |
| `fds_item` | Manages detailed functions and designs for functional, screen, and interface items in an FDS document |
| `dds_spec` | Manages the document number, title, version, revision number, and approval status of each project-specific DDS document |
| `dds_item` | Manages detailed database, component, interface, security, and batch design items in a DDS document |
| `dq_assessment` | Manages the number, version, revision number, and status of each project-specific design qualification assessment document |
| `dq_item` | Manages links between URS requirements and FDS or DDS designs, qualification assessment results, reviewers, and notes |

### Key Points

- DDS provides detailed design from an implementation perspective, while DQ assesses whether the design satisfies the URS.
- `dq_item` manages design qualification decisions and review results for each URS requirement. The stored values for `result_status` are `PASS`, `FAIL`, and `PENDING`; new items start as `PENDING` (Pending Review).
- Apart from the FDS and DDS mappings used for display in `dq_item`, formal, general-purpose traceability between deliverables is interpreted through `traceability_link`.

---

## 7. FRA Risk Assessment Management
<img width="2575" height="2129" alt="image" src="https://github.com/user-attachments/assets/077a700f-9e8b-483f-ac75-e73a29f8a150" />
Link : https://drawsql.app/teams/minho-kim/diagrams/07-fra-risk-assessment-management

### Relationship Structure

```text
validation_project
  ├─ requirement (URS)
  └─ fra_assessment
       └─ fra_item
            └─ requirement (when applicable)
```

### Structure Overview

This structure organizes Validation projects around systems belonging to an organization and evaluates functional risks in connection with project-specific URS requirements.

`fra_assessment` manages FRA document versions and status. `fra_item` manages functional risk scenarios, product impact, likelihood, detectability, risk score, risk level, and mitigation strategy.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages reference information for the customer or operating organization to which systems and users belong |
| `app_user` | Manages users who create and modify projects, FRA documents, and risk items |
| `system_asset` | Manages identifying information, asset number, GAMP category, GxP classification, and status of systems or equipment subject to Validation |
| `validation_project` | Manages the scope, validation method, progress, and status of the Validation project to which the FRA belongs |
| `requirement` | Manages project-specific URS requirements and revisions used as the basis for FRA risk items |
| `fra_assessment` | Manages the number, title, version, revision number, and status of each project-specific FRA document |
| `fra_item` | Manages functional risk scenarios, PI, LL, and DL ratings, risk score, risk level, mitigation strategy, and linked tests |

### Key Points

- FRA is an assessment for prioritizing testing and controls according to risk level, rather than testing every function with the same intensity.
- Risk items are linked to URS requirements when needed; risk items without a direct URS link are also allowed.
- Assessment results are used to determine the scope of subsequent IQ, OQ, and PQ testing and to evaluate RTM coverage.

---

## 8. IQ, OQ, and PQ Qualification Testing Management
<img width="3070" height="2543" alt="image" src="https://github.com/user-attachments/assets/dcbb8a59-325e-4e99-9be1-873c70aabc53" />
Link : https://drawsql.app/teams/minho-kim/diagrams/08-iq-oq-and-pq-qualification-testing-management

### Relationship Structure

```text
validation_project
  ├─ iq_assessment ─ iq_item
  ├─ oq_assessment ─ oq_item
  └─ pq_assessment ─ pq_item

iq_item / oq_item / pq_item
  └─ deviation (logical connection when a deviation occurs during testing)
```

### Structure Overview

This structure creates IQ, OQ, and PQ protocols and detailed test items for systems and Validation projects, and manages test execution results and deviations.

The document header and detailed test items for each qualification are managed separately. After protocol approval, the executor, actual result, qualification result, and execution time are recorded. Problems encountered during testing are managed in `deviation` through investigation, resolution, and closure.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages reference information for the customer or operating organization to which systems and users belong |
| `app_user` | Manages users who create and modify qualification documents and execute tests |
| `system_asset` | Manages identifying information and status for systems or equipment subject to IQ, OQ, and PQ |
| `validation_project` | Manages the scope, validation method, and progress status of the Validation project to which IQ, OQ, and PQ belong |
| `iq_assessment` | Manages IQ document number, version, revision information, protocol status, and execution record status |
| `iq_item` | Manages IQ installation verification procedures, expected and actual results, qualification result, executor, and execution time |
| `oq_assessment` | Manages OQ document number, version, revision information, protocol status, and execution record status |
| `oq_item` | Manages OQ operational function test procedures, expected and actual results, qualification result, executor, and execution time |
| `pq_assessment` | Manages PQ execution plan, schedule, execution method, version, protocol status, and execution record status |
| `pq_item` | Manages PQ performance test procedures, expected and actual results, qualification result, executor, and execution time |
| `deviation` | Manages the description, severity, investigation, resolution, and closure approval status of deviations arising during document or test execution |

### Key Points

- IQ verifies installation suitability, OQ verifies that functions meet specifications, and PQ verifies sustained performance under actual operating conditions.
- Each test distinguishes approval of the protocol, which defines the test procedure, from approval of the record, which contains the actual execution results. Final approval of IQ, OQ, and PQ activities requires both `protocol_status` and `record_status` in the parent document to be `APPROVED`.
- New IQ and OQ documents and items start as `DRAFT`. Item protocol and record statuses are synchronized with the corresponding parent document statuses within the same revision.
- Test execution is allowed based on protocol approval in the current parent document. IQ and OQ execution is not allowed based solely on item statuses. Adding, modifying, or deleting items in an approved protocol requires a new revision starting as `DRAFT`. Actual results, decisions, performers, and execution times are recorded, and discrepancies are managed in `deviation`.
- `pq_assessment.status` stores the execution progress states `수행 대기 중` (Awaiting Execution), `진행 중` (In Progress), and `완료` (Completed), with an initial value of `수행 대기 중` (Awaiting Execution). This is distinct from protocol and record approval statuses.

---

## 9. Design and Risk Deliverable Traceability Management
<img width="4360" height="5206" alt="image" src="https://github.com/user-attachments/assets/a1587eae-c5f3-47d7-9309-7495b8c0044d" />
Link : https://drawsql.app/teams/minho-kim/diagrams/09-design-and-risk-deliverable-traceability-management

### Relationship Structure

```text
requirement (URS)
  ├─ IMPLEMENTED_BY → fds_item
  ├─ IMPLEMENTED_BY → dds_item
  ├─ ASSESSED_BY    → dq_item
  └─ ASSESSED_BY    → fra_item

The logical connections above are managed in traceability_link
```

### Structure Overview

This structure manages traceability among a Validation project's URS requirements and its FDS and DDS designs, DQ assessments, and FRA risk assessment deliverables.

Each document is composed of a header and detail items. `traceability_link` connects deliverable items through polymorphic references representing implementation, assessment, and risk mitigation relationships.

### Summary of Table Roles

| Table | Role |
|---|---|
| `app_user` | Manages users who create and modify design or risk deliverables and traceability relationships |
| `validation_project` | Manages the Validation project to which design or risk deliverables and traceability relationships belong |
| `requirement` | Manages project-specific URS requirements and revisions that serve as the basis for traceability |
| `fds_spec` | Manages the number, version, revision information, and status of each project-specific FDS document |
| `fds_item` | Manages FDS detail items that implement URS requirements as functions, screens, and interfaces |
| `dds_spec` | Manages the number, version, revision information, and status of each project-specific DDS document |
| `dds_item` | Manages detailed database, component, interface, security, and batch design items |
| `dq_assessment` | Manages the version, revision information, and status of each project-specific design qualification assessment document |
| `dq_item` | Manages links between URS and FDS or DDS designs and the corresponding design qualification assessment results |
| `fra_assessment` | Manages the version, revision information, and status of each project-specific functional risk assessment document |
| `fra_item` | Manages risk scenarios, risk scores, risk levels, and mitigation strategies by URS or function |
| `traceability_link` | Uses polymorphic references to manage implementation, assessment, and mitigation relationships among URS, FDS, DDS, DQ, and FRA items |

### Key Points

- `traceability_link` is a general-purpose relationship table connecting a source entity to a target entity.
- Typical relationship types are `IMPLEMENTED_BY`, `ASSESSED_BY`, `VERIFIED_BY`, and `MITIGATED_BY`.
- Because these are polymorphic references, `source_entity_id` and `target_entity_id` have no physical FKs to the target business tables. Validity for each type must be checked at the application layer.
- Duplicate combinations of project, source type/ID, target type/ID, and relationship type are not allowed among non-deleted traceability links.

---

## 10. Qualification Test Traceability and RTM Management
<img width="2840" height="2703" alt="image" src="https://github.com/user-attachments/assets/2173e7a8-503a-42f0-b909-66593b263fa5" />
Link : https://drawsql.app/teams/minho-kim/diagrams/10-qualification-test-traceability-and-rtm-management

### Relationship Structure

```text
requirement (URS)
  ├─ traceability_link → iq_item
  ├─ traceability_link → oq_item
  └─ traceability_link → pq_item

Aggregate source traceability relationships (business flow)
  └─ rtm_assessment
       └─ rtm_item (traceability items by URS)
```

### Structure Overview

This structure manages traceability between a Validation project's URS requirements and IQ, OQ, and PQ test items and uses those relationships to manage the Requirements Traceability Matrix and deliverable coverage.

`traceability_link` manages verification relationships between URS and qualification test items. `rtm_assessment` and `rtm_item` preserve project- and URS-level traceability results and an RTM snapshot at the time of approval.

### Summary of Table Roles

| Table | Role |
|---|---|
| `app_user` | Manages users who create and modify qualification tests, traceability relationships, and RTM documents |
| `validation_project` | Manages the Validation project to which qualification tests and RTM belong |
| `requirement` | Manages project-specific URS requirements that serve as the basis for test traceability and RTM |
| `iq_assessment` | Manages the version, protocol status, and execution record status of each project-specific IQ document |
| `iq_item` | Manages IQ installation verification procedures and test results used to verify URS requirements |
| `oq_assessment` | Manages the version, protocol status, and execution record status of each project-specific OQ document |
| `oq_item` | Manages OQ operational function test procedures and results used to verify URS requirements |
| `pq_assessment` | Manages project-specific PQ execution plans, document versions, and test progress status |
| `pq_item` | Manages PQ performance test procedures and results under actual operating conditions |
| `traceability_link` | Uses polymorphic references to manage verification relationships between URS and IQ, OQ, and PQ test items |
| `rtm_assessment` | Manages RTM snapshots of total URS count, FRA linkage rate, IQ and OQ coverage, and overall average coverage for each project |
| `rtm_item` | Manages detailed snapshots of FRA, FDS, and DDS mapping strings, IQ, OQ, and PQ test item reference strings, and item-level coverage for each URS requirement |

### Key Points

- `traceability_link` holds the current operational source traceability relationships, while `rtm_assessment` and `rtm_item` are snapshots at the time of RTM generation and approval.
- To preserve RTM results as they were at approval, the application must store snapshot values and enforce controls on changes after approval. The presence of FKs or version columns alone does not guarantee immutability.
- RTM is a deliverable used to identify unlinked URS requirements, missing designs, risk assessments or tests, mapping failures, and coverage. The descriptions of `rtm_item.iq_result`, `oq_result`, and `pq_result` currently define them as strings containing test item numbers, so they should not be interpreted as direct FKs or test PASS/FAIL decision values.

---

## 11. VSR and Deviation Management
<img width="4760" height="4306" alt="image" src="https://github.com/user-attachments/assets/d5c6ffd5-1702-4f1a-8de3-8585984fc5b2" />
Link : https://drawsql.app/teams/minho-kim/diagrams/11-vsr-and-deviation-management

### Relationship Structure

```text
project_activity approval status
        +
rtm_assessment traceability and coverage
        +
deviation resolution and closure status
        ↓ (business flow)
vsr_assessment
  └─ vsr_item (result summary by activity)
        ↓
Validation project closure decision
```

### Structure Overview

This structure consolidates Validation project activity results, RTM coverage, and deviation status to manage the final validation conclusion and VSR.

`project_activity` manages project-specific activity status, while `deviation` identifies unresolved deviations. `vsr_assessment` and `vsr_item` summarize the project's final conclusion and each activity's document, test, deviation, and approval results.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages reference information for the customer or operating organization to which systems and users belong |
| `app_user` | Manages users who create, modify, and process project activities, deviations, and VSR documents |
| `system_asset` | Manages identifying information for the Validation system or equipment covered by the VSR |
| `validation_project` | Manages the Validation project to which activities, deviations, RTM, and VSR belong |
| `validation_activity` | Manages codes, names, and default order for Validation activities aggregated in the VSR |
| `project_activity` | Manages selected activities, mandatory status, progress status, and approval time for each project |
| `rtm_assessment` | Manages project-specific requirement traceability and coverage results referenced by the VSR |
| `deviation` | Manages the severity, status, resolution details, and closure approval of deviations arising during document and test execution |
| `vsr_assessment` | Manages the final validation conclusion, conclusion details, version, and approval status for each project |
| `vsr_item` | Summarizes document number, revision, result counts, deviations, and approval information by Validation activity |

### Key Points

- VSR is a document that draws a final conclusion by consolidating activity approval status, test results, RTM coverage, and unresolved deviations. The activities summarized by `vsr_item.activity_code` include DDS. System identification is managed as project context, and the VSR itself is excluded from detailed aggregation.
- Normal closure is determined by considering approval of the selected activities, traceability, and deviation closure conditions together.
- `validation_project.closure_type` distinguishes normal closure (NORMAL) from forced closure (FORCED). `closure_reason` is required for forced closure, and `closure_requested_by` and `closure_requested_at` must match the signer and signing timestamp of the closure request signature. `closed_at` is the final closure timestamp, and the closure requester references `app_user` through an FK.
- Record the closure request signature as `SUBMIT`, targeting `validation_project` and the corresponding `project_id`. Assign a `CLOSE-n` version to each request and distinguish the signature meaning as `PROJECT_CLOSE_NORMAL` or `PROJECT_CLOSE_FORCED`. At final closure, compare the signature for that version, the preserved input, and the current request values. If the request details have changed, a new request version and a new signature are required.
- Fields such as `vsr_item.activity_code`, `doc_no`, and `deviation_info` are summary values. There are no direct FKs to `project_activity`, `rtm_assessment`, or `deviation`.

---

## 12. Workflow, Approval, and Electronic Signature Management
<img width="4640" height="3766" alt="image" src="https://github.com/user-attachments/assets/55c8d805-4684-4d27-9288-ad118a65bc37" />
Link : https://drawsql.app/teams/minho-kim/diagrams/12-workflow-approval-and-electronic-signature-management

### Relationship Structure

```text
validation_project
  └─ project_member ─ app_user / role

workflow_instance
  └─ workflow_step
       └─ approval_action ─ electronic_signature (when a signature exists)
```

### Structure Overview

This structure manages the Workflow from submission of a Validation project document through step-by-step review, approval, and rejection, together with electronic signature evidence.

Assign step assignees based on project participants and roles, and manage the overall Workflow and actual action history through `workflow_instance`, `workflow_step`, and `approval_action`. Electronic signatures linked to submission, review, approval, and rejection preserve both the target document version and its content hash. Project closure request signatures are managed using a separate `CLOSE-n` request version.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages reference information for the customer or operating organization to which systems and users belong |
| `app_user` | Manages document submitters, step assignees, actual processors, and electronic signers |
| `role` | Manages reference data for project participant roles; consulted by business logic when selecting Workflow assignees |
| `system_asset` | Manages information about the system or equipment associated with the Validation project subject to review and approval |
| `validation_project` | Manages the Validation project to which documents under review or approval and project participants belong |
| `project_member` | Manages participating users, assigned roles, participation status, and participation period for each project |
| `workflow_instance` | Manages submission information, current step, and overall status of the complete review and approval Workflow for each target document |
| `workflow_step` | Manages step order, step type, assignee, due date, and status for each Workflow step |
| `approval_action` | Manages actual actions such as submission, review, approval, rejection, and cancellation, including the processor and processing time |
| `electronic_signature` | Manages the signer, signature meaning, target version, content hash, and reauthentication result for document submission, review, approval, rejection, and project closure requests |

### Key Points

- `workflow_instance` manages overall progress, `workflow_step` manages individual review and approval steps, and `approval_action` manages the actual processing actions.
- `electronic_signature` provides identity verification and signature evidence for document submission, review, approval, rejection, and project closure requests. When linking it to a Workflow action, verify that the actual actor matches the signer, that the target table, record, and version match, and that the action type matches the signature action.
- `approval_action.signature_id` references an electronic signature and allows NULL. For `CANCEL`, record the cancellation reason and Audit Trail without a signature, set the Workflow to `CANCELLED`, and set the remaining incomplete steps to `SKIPPED`.
- Create Workflow steps before submission and link `SUBMIT` to the first non-deleted step. Link `CANCEL` during execution to the current step, and cancellation before execution starts to the first step. Do not cancel a Workflow that has ended. These links do not mean that review or approval of the associated step is complete.
- `workflow_step.step_type` is `REVIEW` or `APPROVE`; record `SUBMIT` and `CANCEL` in `approval_action.action_type`. Among non-deleted steps, `step_order` must be unique within the same Workflow.
- `workflow_instance` has no `project_id` FK, and `workflow_step` has no FK to `project_member` or `role`. The assignee directly references a user through `assignee_id`, and suitability for the project role must be checked by business logic.
- Separately from these concepts, Audit Trail records the history of data changes that occur during processing.

---

## 13. File, Evidence, and File Cleanup Management
<img width="2500" height="1791" alt="image" src="https://github.com/user-attachments/assets/be3ac81c-3cb4-46c1-b19e-80c1cc60e362" />
Link : https://drawsql.app/teams/minho-kim/diagrams/13-file-evidence-and-file-cleanup-management

### Relationship Structure

```text
file_asset
  ├─ file_cleanup_execution (last cleanup operation)
  └─ evidence_link
       └─ business documents and test items (logical connection)
```

### Structure Overview

This structure manages metadata for attachments and test evidence used in a Validation project, their relationships to business objects, and cleanup jobs for temporary and expired files.

`file_asset` manages stored-file metadata and cleanup status, while `evidence_link` manages relationships between evidence files and documents, test items, or deviations. `file_cleanup_execution` manages cleanup target discovery, deletion processing, failures, and retry history.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages organization reference data for users and systems associated with files and projects |
| `app_user` | Manages file uploaders and users who request or process file cleanup jobs |
| `system_asset` | Manages information about the Validation system or equipment from which files and evidence originate |
| `validation_project` | Manages the Validation project to which evidence-file links belong |
| `file_cleanup_execution` | Manages execution type, status, processing counts, failures, and retry history for temporary, expired, and orphaned file cleanup jobs |
| `file_asset` | Manages storage path, size, format, expiration, and cleanup status for attachments, evidence files, reports, and export files |
| `evidence_link` | Uses polymorphic references to manage N:M evidence relationships between files and documents, test items, or deviations |

### Key Points

- `file_asset` holds metadata about the file itself, while `evidence_link` identifies the business item for which that file serves as evidence.
- Separating these two areas allows one file to be linked to multiple business items, or multiple files to be linked to one business item. Duplicate combinations of project, file, target type/ID, and evidence type are not allowed among non-deleted evidence links.
- `file_asset.cleanup_execution_id` points only to the last cleanup execution and allows NULL. A cleanup execution is not a mandatory parent for every file, and the current definition does not include a separate N:M history table that preserves the full cleanup history for each file.
- File cleanup is an operational function that cleans up temporary, expired, and orphaned files through controlled procedures and records the results; it is not a function for arbitrarily deleting evidence that must be retained under regulations.

---

## 14. Report, Notification, and Backup Operations Management
<img width="2800" height="2423" alt="image" src="https://github.com/user-attachments/assets/5643c343-a8e0-4bf0-a773-d2411268b98b" />
Link : https://drawsql.app/teams/minho-kim/diagrams/14-report-notification-and-backup-operations-management

### Relationship Structure

```text
report_schedule
  └─ report_generation
       └─ file_asset (generated output file)

workflow_instance / workflow_step
  └─ notification_delivery

backup_execution (independent operational execution history)
```

### Structure Overview

This structure manages scheduled and manual generation of operational reports, delivery of review and approval notifications, and backup execution for system data and files around Validation projects and Workflows.

`report_schedule` and `report_generation` manage report schedules and generation jobs, and generated output files are linked to `file_asset`. `notification_delivery` manages Workflow-related notification delivery and retries, while `backup_execution` manages the status and results of scheduled and manual backups.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages reference information for the customer or operating organization to which operational reports and users belong |
| `app_user` | Manages report and backup requesters, notification recipients, and users who create or modify operational jobs |
| `system_asset` | Manages reference data for systems and equipment linked to projects; has no direct FK relationship with the backup execution table |
| `validation_project` | Provides the basis for project-level reports, notifications, and operational jobs |
| `workflow_instance` | Manages complete review and approval Workflow information for documents that trigger notifications |
| `workflow_step` | Manages step assignees and status used for approval-request, due-date, and overdue notifications |
| `file_asset` | Manages storage path, size, format, and cleanup status of generated report files |
| `report_schedule` | Manages recurring report type, frequency, query period, output format, and next execution time |
| `report_generation` | Manages report generation requests, query criteria, execution status, output file, failures, and retry history |
| `notification_delivery` | Manages recipient, channel, delivery status, failures, and retry history for review and approval requests and due-date notifications |
| `backup_execution` | Manages scheduled and manual backup type, target, status, storage location, size, and retry history for databases and files |

### Key Points

- This area manages operational job requests, processing results, failures, and retry history as data, rather than the infrastructure configuration itself.
- The success status for reports and backups is `COMPLETED`, while the success status for notification delivery is `SENT`. Notifications follow the `PENDING → PROCESSING → SENT` flow, with separate states for failure, retry, and cancellation.
- A single recurring schedule can produce multiple report generation jobs, and the generated output is linked to `file_asset`.

---

## 15. AI Generation Job and Result Management
<img width="2488" height="2229" alt="image" src="https://github.com/user-attachments/assets/52061c1a-be1e-4375-a75b-1bd04c5ad0d5" />
Link : https://drawsql.app/teams/minho-kim/diagrams/15-ai-generation-job-and-result-management

### Relationship Structure

```text
ai_generation_job
  └─ ai_generation_result
       └─ ai_result_item

Business flow: user review and selection → application to business data → formal review and approval
```

### Structure Overview

This structure manages AI generation requests for Validation project documents and items as asynchronous jobs and tracks result selection and whether results have been applied to actual business data.

`ai_generation_job` manages the AI model, input conditions, execution status, and retries. `ai_generation_result` and `ai_result_item` manage generated result sets and their detail items, including selection, adoption, and application targets.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages organization reference data for users of AI features and target systems |
| `app_user` | Manages AI generation requesters, result selectors, and users who create or modify jobs |
| `system_asset` | Manages the Validation system or equipment to which AI-generated target documents belong |
| `validation_project` | Manages the Validation project to which AI generation jobs and target business data belong |
| `ai_generation_job` | Manages AI generation job type, target entity, model, input conditions, execution status, failures, and retry history |
| `ai_generation_result` | Manages the title, selection status, application status, selecting user, and selection time of a result set generated by an AI job |
| `ai_result_item` | Manages the content and application target of individual AI-generated requirements, risk scenarios, test items, or document sections |

### Key Points

- AI results do not become official deliverables immediately upon generation. They are applied to actual business tables after user review and selection.
- Original generated content is kept separate from the actual application result, and the model used, input conditions, selecting user, and application status are recorded.
- The business principle is that applied results follow the same formal review and approval procedures as other business data. There is no direct FK from the AI result tables to Workflow.
- `ai_generation_job.project_id` allows NULL, so not every generation job is required to be linked to a specific project.

---

## 16. Audit Trail Management
<img width="1563" height="904" alt="image" src="https://github.com/user-attachments/assets/60be2276-6f0d-407e-987f-4a1f4b0a6fdb" />
Link : https://drawsql.app/teams/minho-kim/diagrams/16-audit-trail-management

### Relationship Structure

```text
organization
  └─ app_user
       └─ audit_trail (user actions)

audit_trail (also records system and batch actions)
  └─ target business record (logical connection)
```

### Structure Overview

This structure preserves major data changes made by users belonging to an organization and the processing history of system and batch jobs for audit purposes.

`audit_trail` manages the action type, target table and record, before and after values, reason for change, actor, request and session information, and target document or project closure request version. Audit targets are identified through polymorphic references, and system or batch actions can be recorded without a user.

### Summary of Table Roles

| Table | Role |
|---|---|
| `organization` | Manages reference information for the customer or operating organization to which audited users belong |
| `app_user` | Manages user accounts and organizational affiliation for users who make major data changes |
| `audit_trail` | Manages the actor, target, before and after values, reason for change, request and session information, and document or closure request version for data creation, modification, deletion, and operational actions |

### Key Points

```text
Who             actor_id / actor_type
When            created_at
What            target_table_name / target_record_id
How             action_type
Before change   old_values
After change    new_values
Why             reason_for_change
Where           client_ip / request_uri / user_agent
Which request   request_id / session_id
Which version   target_version / target_revision_number
```

- In addition to user-made changes, system or batch actions can also be recorded; in those cases, the user ID may be absent.
- Audit Trail is the history of data changes and important system actions, while electronic signatures provide signature evidence for actions such as submission, review, approval, rejection, and project closure requests.
- Record project closure requests as `UPDATE`, final normal closure as `PROJECT_CLOSE`, and forced closure as `PROJECT_FORCE_CLOSE`, storing the same `CLOSE-n` version as the electronic signature. Preserve the signature input string `signed_payload` and `signature_id` in `new_values` of the closure request record, and calculate the electronic signature’s SHA-256 hash from the UTF-8 bytes of that string.
- Manage `client_ip` and `old_values`/`new_values`, which may contain personal information, as sensitive data. `password_hash` is sensitive information but has `Audit=N`; do not include the hash value in audit logs.
- The history of changes can be traced if the implementation stores existing values in Audit Trail upon deletion. Because `old_values` currently allows NULL, preservation of values before deletion must be ensured through business logic and recording policies.
- `audit_trail` has no direct organization ID. The organizational scope of system and batch records without a user must be determined through another path, such as the target record.
