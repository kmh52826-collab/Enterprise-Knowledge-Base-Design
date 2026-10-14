<a id="top"></a>

# ERD Specifications

This document describes the 76 tables of the Validation Management Platform, divided into 14 business areas. For each area, you can see the information it manages and how its main tables are connected.

To see the connections between business areas at a glance, refer to the **[Overall Data Structure](./erd-overview.md)**.

> [!NOTE]
> This document is intended to explain data modeling and system design. Example values are de-identified samples and do not represent actual operational data of any specific organization.

The relationship structures are simplified diagrams showing the main connections. Commonly used tables may appear in multiple ERDs. See the [Data Dictionary](./data-dictionary.md) for column definitions and business rules.

## Contents

1. [Organization, Users & Permissions](#erd-01)
2. [System Inventory & Project Management](#erd-02)
3. [Validation Planning & Pre-assessment](#erd-03)
4. [Requirements, Design & Risk Assessment](#erd-04)
5. [IQ Installation Qualification Testing](#erd-05)
6. [OQ Operational Qualification Testing](#erd-06)
7. [PQ Performance Qualification Testing](#erd-07)
8. [Deviation, Action & Re-execution](#erd-08)
9. [Requirement–Test Traceability](#erd-09)
10. [VSR Summary Report](#erd-10)
11. [Approval Workflow, E-signature & Audit Trail](#erd-11)
12. [Deliverables, Document Revisions & Files](#erd-12)
13. [Library & Regulatory Basis](#erd-13)
14. [AI Generation & Application](#erd-14)

---

<a id="erd-01"></a>

## 1. Organization, Users & Permissions

**Tables: 11**

### Relationship Structure

```text
organization
  ├─ app_user ─ user_role ─ role
  │    └─ user_signature_credential (electronic signature password)
  └─ user_group
       ├─ user_group_member ─ app_user
       └─ group_role ─ role

app_user / user_group
  ├─ access_permission_grant (screen/project access permissions)
  └─ inventory_role_grant (inventory business permissions)
```

### Overview

Users and groups are managed per organization, and business roles and access permissions are granted to individuals and groups. The structure separately manages roles such as author, reviewer and approver, access permissions for screens and projects, and inventory business permissions. The login method (SSO or platform password) is set per organization, and an electronic signature verifies the signer's identity with a signature-only password separate from the login. Accounts for automated system processing belong to the SYSTEM organization, and administrators from the developer company belong to the VENDOR organization.

### Table Roles

| Table | Role |
|---|---|
| `organization` | Organizations that users and groups belong to, with login method and AI usage settings |
| `app_user` | User accounts, their organization, account status and permission level |
| `role` | Definitions of business roles such as author, reviewer and approver |
| `user_role` | Business roles granted directly to users |
| `user_group` | User groups per organization |
| `user_group_member` | Users belonging to a group and their membership status |
| `group_role` | Common or project-specific business roles granted to groups |
| `access_permission_grant` | Screen and project access permissions for individuals and groups |
| `inventory_role_grant` | Inventory authoring, review, approval and disposal permissions for individuals and groups |
| `user_signature_credential` | Signature-only password used for electronic signature re-authentication, with lock and expiry status |
| `validation_project` | Basic information of validation projects and their progress and closure status |

### Key Points

- The permission level (SYSTEM_ADMIN, ADMIN, USER) and business roles are separate. An account's permission level alone does not make it a reviewer or approver.
- Permissions granted directly to a user and permissions received from the user's groups are applied together. Group permissions are determined based on active groups and members.
- Editing and disposal require both an access permission and a business role. Review and approval require being assigned as the approval assignee or substitute assignee.

[↑ Back to contents](#top)

---

<a id="erd-02"></a>

## 2. System Inventory & Project Management

**Tables: 14**

### Relationship Structure

```text
system_asset (system under validation)
  ├─ system_asset_revision (revision/approval history)
  └─ validation_project
       ├─ project_system_baseline (adopted approved revision)
       ├─ project_member (participants)
       ├─ project_activity ─ validation_activity
       └─ project_closure_request (closure request)

validation_activity ─ activity_dependency (activity dependency conditions)
```

### Overview

Manages the current information and revision history of systems under validation and runs projects based on them. Each project is linked to the approved revision adopted as its validation baseline, its participants, its activities and its closure requests.

### Table Roles

| Table | Role |
|---|---|
| `system_asset` | Current information and approval status of systems and equipment under validation |
| `system_asset_revision` | Change and approval history of a system, with the original content of its information at that time |
| `validation_project` | Basic information of validation projects and their progress and closure status |
| `project_system_baseline` | Approved system revision adopted by a project as its validation baseline |
| `project_member` | Project participants and their assigned roles |
| `validation_activity` | Common list of activities such as validation planning, assessment and testing |
| `project_activity` | Activities selected for a project and their progress status |
| `activity_dependency` | Predecessor/successor conditions between activities and the version of the condition set |
| `project_closure_request` | Project closure requests and links to the approval history of each round |
| `organization` | Organizations that users and groups belong to, with login method and AI usage settings |
| `app_user` | User accounts, their organization, account status and permission level |
| `role` | Definitions of business roles such as author, reviewer and approver |
| `workflow_instance` | Approval workflow instance for a specific business item |
| `electronic_signature` | Signer, signature target and the original content at the time of signing |

### Key Points

- The system's current information is kept separate from the validation baseline adopted by the project. Even if the system is revised later, the approved original content used by the project is preserved.
- A project proceeds according to its selected activities and the version of dependency conditions adopted when it was created. Common conditions are created as new versions, and the version adopted by an existing project is not changed.
- Closure requests are managed per round. Even if a request is made again after rejection, the reason and approval history of the earlier request can still be reviewed.

[↑ Back to contents](#top)

---

<a id="erd-03"></a>

## 3. Validation Planning & Pre-assessment

**Tables: 10**

### Relationship Structure

```text
validation_project
  ├─ vp_plan (validation plan)
  │    └─ vp_section (section/body)
  ├─ qia_assessment (quality impact assessment)
  │    └─ qia_module_item (module)
  │         └─ qia_process (process)
  └─ vendor_audit (vendor audit)
```

### Overview

Manages the project's validation plan (VP), quality impact assessment (QIA) and vendor audit (VA). A VP consists of sections and body text, and a QIA consists of a common assessment, modules and processes; a vendor audit records the auditor, date and assessment file.

### Table Roles

| Table | Role |
|---|---|
| `vp_plan` | Revisions and approval status of the validation plan |
| `vp_section` | Sections and body text included in the validation plan |
| `qia_assessment` | Project-wide Part 11 assessment and judgment criteria |
| `qia_module_item` | Quality impact assessment per module, with revision and approval status |
| `qia_process` | GxP assessment answers and judgment criteria for each process in a module |
| `vendor_audit` | Vendor audit revisions, auditor, date and assessment file |
| `validation_project` | Basic information of validation projects and their progress and closure status |
| `app_user` | User accounts, their organization, account status and permission level |
| `workflow_instance` | Approval workflow instance for a specific business item |
| `file_asset` | Information identifying a file's storage location and original |

### Key Points

- The VP, QIA modules and vendor audits each manage their own revisions and approvals. The body text and assessment answers at the time of approval are preserved.
- A QIA preserves not only the answers but also the questions and judgment criteria used at the time. Past results can still be interpreted even if the criteria change later.
- QIA answers may be left blank while in progress, but all of them must be entered at submission. A module with no processes or with unanswered questions is not judged (not evaluated) and cannot be submitted.

[↑ Back to contents](#top)

---

<a id="erd-04"></a>

## 4. Requirements, Design & Risk Assessment

**Tables: 15**

### Relationship Structure

```text
validation_project
  ├─ requirement (URS)
  ├─ fds_spec (FDS)
  ├─ dds_spec (DDS)
  ├─ dq_assessment ─ dq_item (design qualification assessment)
  └─ fra_assessment ─ fra_item (functional risk assessment)

dq_item ─ requirement / fds_spec / dds_spec
fra_item ─ requirement
```

### Overview

The URS manages the requirements to be validated, and the FDS and DDS manage the functional design and detailed design documents. DQ assesses the suitability of requirements and designs, and FRA records the risks and the rationale for their mitigation for each requirement. Regulatory clauses and the library are linked as the basis for authoring and assessment. URS entries are classified by major category, middle category and target, and all three values are required at submission.

### Table Roles

| Table | Role |
|---|---|
| `requirement` | Requirement content and acceptance criteria, with revision and approval status |
| `fds_spec` | Functional design document file, with revision and approval status |
| `dds_spec` | Detailed design document file, with revision and approval status |
| `dq_assessment` | Design qualification assessment information for a project |
| `dq_item` | Suitability judgments and review notes for requirements and design documents |
| `fra_assessment` | Functional risk assessment information for a project |
| `fra_item` | Risk scenarios, assessment results and SOP basis for each requirement |
| `validation_project` | Basic information of validation projects and their progress and closure status |
| `app_user` | User accounts, their organization, account status and permission level |
| `workflow_instance` | Approval workflow instance for a specific business item |
| `library_item` | Reusable items for authoring requirements, risk assessments and tests |
| `file_asset` | Information identifying a file's storage location and original |
| `regulatory_clause` | Individual clauses of regulatory documents and their location in the original text |
| `requirement_regulation` | Links between requirements and regulatory clauses, with the rationale for applying them |
| `regulatory_source` | Documents and editions of regulations, guidelines and internal SOPs |

### Key Points

- The FDS and DDS are each managed as independent documents. A draft can be created before the file scan is finished, but a file must be linked before submission. Replacing the file creates a new document revision, and the existing original is preserved.
- A DQ item links one URS revision to at most one FDS revision and at most one DDS revision. An approval request requires at least one valid approved design document from the same project.
- DQ and FRA preserve the exact requirement and design revisions used in the assessment. FRA severity, occurrence and detectability may be left blank while in progress but are all required at submission, and the judgment criteria and the risk assessment results at the time of approval are retained as well.

[↑ Back to contents](#top)

---

<a id="erd-05"></a>

## 5. IQ Installation Qualification Testing

**Tables: 13**

### Relationship Structure

```text
validation_project
  └─ iq_assessment (IQ assessment)
       └─ iq_item (test item/protocol)
            ├─ iq_step (test step)
            └─ iq_execution (execution attempt)
                 └─ iq_step_execution (result per step)
```

### Overview

IQ manages the test content for verifying installation qualification and the actual execution results. Test items and their detailed steps are organized under an assessment, and each execution attempt records a judgment and a completion record for each step.

### Table Roles

| Table | Role |
|---|---|
| `iq_assessment` | IQ assessment revisions, test composition and approval status summary |
| `iq_item` | Installation qualification test content and criteria, with protocol revisions |
| `iq_step` | Detailed steps of an IQ test item and their order |
| `iq_execution` | Judgment, result corrections and approval records for each IQ execution attempt |
| `iq_step_execution` | Completion status and processing records of each step in an IQ execution |
| `validation_project` | Basic information of validation projects and their progress and closure status |
| `project_system_baseline` | Approved system revision adopted by a project as its validation baseline |
| `app_user` | User accounts, their organization, account status and permission level |
| `library_item` | Reusable items for authoring requirements, risk assessments and tests |
| `workflow_instance` | Approval workflow instance for a specific business item |
| `electronic_signature` | Signer, signature target and the original content at the time of signing |
| `evidence_link` | Links between business records and evidence files |
| `file_asset` | Information identifying a file's storage location and original |

### Key Points

- The test composition of an assessment is kept separate from the revisions of individual tests. Even when the assessment is revised, tests whose content has not changed can be included again.
- Tests are executed with approved protocols. Submitted protocols and steps are not modified, and test judgments and the approval of execution results are managed separately.
- A re-execution is recorded as a new attempt, and a correction of an existing result as modification history of the same attempt. When an execution is stopped without a result, it is closed as aborted with a reason and an electronic signature, and past results, evidence and the system baseline at the time of execution are preserved.

[↑ Back to contents](#top)

---

<a id="erd-06"></a>

## 6. OQ Operational Qualification Testing

**Tables: 13**

### Relationship Structure

```text
validation_project
  └─ oq_assessment (OQ assessment)
       └─ oq_item (test item/protocol)
            ├─ oq_step (test step)
            └─ oq_execution (execution attempt)
                 └─ oq_step_execution (result per step)
```

### Overview

OQ manages the content, expected results and acceptance criteria of operational qualification tests, together with the actual execution results. Using the same structure as IQ, it separates assessments, test items, detailed steps and execution records, but manages OQ data and approval status separately.

### Table Roles

| Table | Role |
|---|---|
| `oq_assessment` | OQ assessment revisions, test composition and approval status summary |
| `oq_item` | Operational qualification test content and criteria, with protocol revisions |
| `oq_step` | Detailed steps of an OQ test item and their order |
| `oq_execution` | Judgment, result corrections and approval records for each OQ execution attempt |
| `oq_step_execution` | Completion status and processing records of each step in an OQ execution |
| `validation_project` | Basic information of validation projects and their progress and closure status |
| `project_system_baseline` | Approved system revision adopted by a project as its validation baseline |
| `app_user` | User accounts, their organization, account status and permission level |
| `library_item` | Reusable items for authoring requirements, risk assessments and tests |
| `workflow_instance` | Approval workflow instance for a specific business item |
| `electronic_signature` | Signer, signature target and the original content at the time of signing |
| `evidence_link` | Links between business records and evidence files |
| `file_asset` | Information identifying a file's storage location and original |

### Key Points

- When test content changes, a new protocol revision is created. A test with approval history is not deleted but withdrawn from use through a disposal approval workflow, and existing execution records continue to reference the protocol used at the time.
- Test judgment and result approval status are kept separate. Re-executions and result corrections are also recorded as distinct histories.
- The predecessor/successor order relative to IQ and PQ is managed in project activities. Links to requirements are covered in the traceability area, and actions on test failures in the deviation area.

[↑ Back to contents](#top)

---

<a id="erd-07"></a>

## 7. PQ Performance Qualification Testing

**Tables: 13**

### Relationship Structure

```text
validation_project
  └─ pq_assessment (PQ assessment)
       └─ pq_item (test item/protocol)
            ├─ pq_step (test step)
            └─ pq_execution (execution attempt)
                 └─ pq_step_execution (result per step)
```

### Overview

PQ manages the composition of performance qualification tests and the actual execution results. The test list of each assessment, the test content and steps, and the judgment and completion records of each attempt are preserved separately.

### Table Roles

| Table | Role |
|---|---|
| `pq_assessment` | PQ assessment revisions, test composition and approval status summary |
| `pq_item` | Performance qualification test content and criteria, with protocol revisions |
| `pq_step` | Detailed steps of a PQ test item and their order |
| `pq_execution` | Judgment, result corrections and approval records for each PQ execution attempt |
| `pq_step_execution` | Completion status and processing records of each step in a PQ execution |
| `validation_project` | Basic information of validation projects and their progress and closure status |
| `project_system_baseline` | Approved system revision adopted by a project as its validation baseline |
| `app_user` | User accounts, their organization, account status and permission level |
| `library_item` | Reusable items for authoring requirements, risk assessments and tests |
| `workflow_instance` | Approval workflow instance for a specific business item |
| `electronic_signature` | Signer, signature target and the original content at the time of signing |
| `evidence_link` | Links between business records and evidence files |
| `file_asset` | Information identifying a file's storage location and original |

### Key Points

- Assessment composition, test protocols and execution results are each managed separately. The same approved protocol can be executed in multiple attempts.
- Registering a result requires completed steps and an execution signature. A change after registration is made by finalizing a new correction record while preserving the original result and evidence, and cancelling a correction in progress leaves the existing result unchanged.
- Even if the project's current system baseline changes, the baseline of past executions is retained. Approved execution results serve as the basis for the VSR summary report.

[↑ Back to contents](#top)

---

<a id="erd-08"></a>

## 8. Deviation, Action & Re-execution

**Tables: 10**

### Relationship Structure

```text
iq_execution / oq_execution / pq_execution (failed test execution)
    ↓ deviation registration
deviation
  ├─ deviation_action_round (action/approval per round)
  │    ↓ re-execution
  │  iq_execution / oq_execution / pq_execution (new execution record)
  └─ deviation_completion_revision (completion report/closure report and approval)

On another failure, an action round is added
```

### Overview

Deviations arising from test failures are registered and managed from action approval through re-execution to final closure. `deviation` manages the overall status of the deviation, `deviation_action_round` the action details and approval/re-execution history of each round, and `deviation_completion_revision` the original content and approval of completion reports and closure reports.

### Table Roles

| Table | Role |
|---|---|
| `deviation` | Deviations arising from test failures and their overall processing status |
| `deviation_action_round` | Action details, approval history and re-execution links for each round |
| `deviation_completion_revision` | Original-content revisions and approval of completion reports and closure-without-re-execution reports |
| `validation_project` | Basic information of validation projects and their progress and closure status |
| `app_user` | User accounts, their organization, account status and permission level |
| `workflow_instance` | Approval workflow instance for a specific business item |
| `electronic_signature` | Signer, signature target and the original content at the time of signing |
| `iq_execution` | Judgment, result corrections and approval records for each IQ execution attempt |
| `oq_execution` | Judgment, result corrections and approval records for each OQ execution attempt |
| `pq_execution` | Judgment, result corrections and approval records for each PQ execution attempt |

### Key Points

- The initial failure record is preserved, and each action is linked to its approval basis and re-execution result. A test can have only one open deviation at a time; if the re-execution fails again, the next action round is added instead of a new deviation.
- When the same action is modified after rejection, it is managed as a new revision of that round. Approvals and signatures are linked to the original action content at that time.
- Two paths are distinguished: approving a completion report after a successful re-execution, and closing with a recorded reason and without re-execution after the action is approved. The final signature appropriate to each closure path is preserved, and completion reports and closure reports that are modified after rejection are also kept as new revisions.

[↑ Back to contents](#top)

---

<a id="erd-09"></a>

## 9. Requirement–Test Traceability

**Tables: 11**

### Relationship Structure

```text
requirement (URS)
  └─ traceability_link (trace relationship)
       ├─ fds_spec / dds_spec (design)
       ├─ dq_item / fra_item (design/risk assessment)
       └─ iq_item / oq_item / pq_item (test items)

Trace relationships and test results → RTM dashboard
```

### Overview

`traceability_link` connects each requirement to the designs, assessments and tests that it leads to. The RTM dashboard queries these relationships together with the actual test results to show the validation status of each requirement.

### Table Roles

| Table | Role |
|---|---|
| `traceability_link` | Trace relationships among requirements, designs, assessments and tests |
| `validation_project` | Basic information of validation projects and their progress and closure status |
| `app_user` | User accounts, their organization, account status and permission level |
| `requirement` | Requirement content and acceptance criteria, with revision and approval status |
| `fds_spec` | Functional design document file, with revision and approval status |
| `dds_spec` | Detailed design document file, with revision and approval status |
| `dq_item` | Suitability judgments and review notes for requirements and design documents |
| `fra_item` | Risk scenarios, assessment results and SOP basis for each requirement |
| `iq_item` | Installation qualification test content and criteria, with protocol revisions |
| `oq_item` | Operational qualification test content and criteria, with protocol revisions |
| `pq_item` | Performance qualification test content and criteria, with protocol revisions |

### Key Points

- A trace relationship points to the exact business revision at the time of linking. Even when a new revision is created, the links that served as the basis for past approvals are not changed automatically.
- Multiple requirements can be linked to a single test. Whether a requirement is linked to a test and whether the test actually passed and was approved are checked separately.
- The RTM operates as a dashboard query feature. It is not managed as an independent execution stage or an approval document, and it is not added as a separate item to the project progress rate or to the VSR activity list.

[↑ Back to contents](#top)

---

<a id="erd-10"></a>

## 10. VSR Summary Report

**Tables: 14**

### Relationship Structure

```text
validation_project
  └─ vsr_assessment (summary report)
       └─ vsr_item (result summary per activity)
            ├─ project_activity (activity)
            └─ deliverable_revision (supporting document)

Planning/assessment/test results → VSR aggregation/approval → project closure decision
```

### Overview

The project's planning, assessment and test results are gathered to compose the validation summary report (VSR). `vsr_assessment` manages the overall conclusion and confirmation status, and `vsr_item` the results of each activity and their basis. An approved report preserves the results and approval information at the time of aggregation.

### Table Roles

| Table | Role |
|---|---|
| `vsr_assessment` | Summary report revisions, conclusion and final confirmation status |
| `vsr_item` | Result summary for each activity and its basis at the time of aggregation |
| `validation_project` | Basic information of validation projects and their progress and closure status |
| `project_activity` | Activities selected for a project and their progress status |
| `deliverable_revision` | Content, basis and approval status of each document revision |
| `app_user` | User accounts, their organization, account status and permission level |
| `workflow_instance` | Approval workflow instance for a specific business item |
| `electronic_signature` | Signer, signature target and the original content at the time of signing |
| `vp_plan` | Revisions and approval status of the validation plan |
| `qia_module_item` | Quality impact assessment per module, with revision and approval status |
| `vendor_audit` | Vendor audit revisions, auditor, date and assessment file |
| `iq_execution` | Judgment, result corrections and approval records for each IQ execution attempt |
| `oq_execution` | Judgment, result corrections and approval records for each OQ execution attempt |
| `pq_execution` | Judgment, result corrections and approval records for each PQ execution attempt |

### Key Points

- The activities selected for the project are aggregated, and the RTM and the VSR itself are excluded from the per-activity detail list. The planning, assessment and test tables in this ERD are representative examples that show the basis of the aggregation.
- Business confirmation of the VSR and approval of its output document are managed separately. Even after business confirmation is completed, the output document may not have been generated yet.
- If the source results change after approval, they are aggregated and confirmed again in a new VSR revision. The results and basis of past reports remain unchanged.

[↑ Back to contents](#top)

---

<a id="erd-11"></a>

## 11. Approval Workflow, E-signature & Audit Trail

**Tables: 17**

### Relationship Structure

```text
project_workflow_config (approval route settings)
  └─ workflow_instance (approval workflow execution)
       └─ workflow_step (review/approval stage)
            ├─ workflow_step_assignee (assignee assignment)
            └─ approval_action (action history)
                 └─ electronic_signature (electronic signature)

audit_trail (audit trail records across all business operations)
user_signature_credential (password for verifying identity when signing)
```

### Overview

Review and approval procedures, assignees and processing results used by many business functions are managed in common. Individual approval workflows proceed based on the approval route settings, and the assignments and processing history of each stage are recorded. Electronic signatures preserve the original content of the signed target, and audit trail records preserve actions such as changes, access and exports.

### Table Roles

| Table | Role |
|---|---|
| `workflow_instance` | Approval workflow instance for a specific business item |
| `workflow_step` | Review and approval stages of an approval workflow and their order |
| `workflow_step_assignee` | Assignee and substitute assignee assignments for each approval stage |
| `approval_action` | Processing history of submission, review, approval, rejection and cancellation |
| `project_workflow_config` | Project-wide or activity-specific approval route settings |
| `electronic_signature` | Signer, signature target and the original content at the time of signing |
| `audit_trail` | History of user and system actions and data changes |
| `user_signature_credential` | Signature-only password used for electronic signature re-authentication, with lock and expiry status |
| `app_user` | User accounts, their organization, account status and permission level |
| `validation_project` | Basic information of validation projects and their progress and closure status |
| `project_activity` | Activities selected for a project and their progress status |
| `requirement` | Requirement content and acceptance criteria, with revision and approval status |
| `deliverable_revision` | Content, basis and approval status of each document revision |
| `deviation_action_round` | Action details, approval history and re-execution links for each round |
| `deviation_completion_revision` | Original-content revisions and approval of completion reports and closure-without-re-execution reports |
| `system_asset` | Current information and approval status of systems and equipment under validation |
| `project_closure_request` | Project closure requests and links to the approval history of each round |

### Key Points

- Approval route settings are kept separate from actual approval workflows. An approval workflow is created at submission, and review and approval history is recorded according to the route and assignments applied at that time. When the workflow is rejected or cancelled, the target revision is also closed with the same status, and any modification proceeds as a new revision.
- An electronic signature is linked to the exact target and revision, and the signed original content is preserved with it. Each signing verifies the signer's identity with a signature-only password separate from the login, and changing the body after signing requires a new revision and a new signature.
- Approval status and assignee actions are checked in the approval workflow records, and actions across all business operations in the audit trail. The audit trail adds new records without modifying existing content, and records made by the system or batch jobs rather than by people are distinguished by the SYSTEM_PROCESS menu.

[↑ Back to contents](#top)

---

<a id="erd-12"></a>

## 12. Deliverables, Document Revisions & Files

**Tables: 16**

### Relationship Structure

```text
deliverable_document (deliverable)
  └─ deliverable_revision (document revision)
       ├─ deliverable_section (section/body)
       └─ report_generation (PDF generation)
            └─ file_asset (output file)

User upload → file_scan_job (scan) → file_asset (registered after passing the scan)
     └─ file_link_request (link request) → automatically linked to evidence / design file / VA attachment when the scan passes
Business record ─ evidence_link (evidence link) ─ file_asset
```

### Overview

Project deliverables are managed as documents, revisions, and sections/body, and are linked to PDF generation and evidence files. Each document revision preserves the business content it was based on and the results at that time. This ERD uses IQ as an example to show how document bases and test evidence are linked.

For a user upload, the scan attempts and results of the quarantined original are recorded in `file_scan_job`, and the file is registered in `file_asset` after no threats are detected and its identity with the original is confirmed. `file_origin` distinguishes user uploads (`UPLOAD`) from trusted files generated internally by the server (`SYSTEM_GENERATED`). PDFs and export files generated by the server can be registered without an upload scan job, but any externally uploaded file used for generation must pass the scan first. Right after upload, the business target to link to (test step evidence, FDS/DDS file, VA attachment) is reserved in `file_link_request`, and the file is linked automatically once it passes the scan.

### Table Roles

| Table | Role |
|---|---|
| `deliverable_document` | Document number and type of a deliverable, and the activity it belongs to |
| `deliverable_revision` | Content, basis and approval status of each document revision |
| `deliverable_section` | Sections and body text included in a document revision |
| `report_generation` | PDF generation requests, progress status and output file |
| `file_scan_job` | Quarantined original of a user upload, scan attempts and results, and link to the registered file |
| `file_asset` | Origin, storage location and original-identification information of files used in business operations |
| `evidence_link` | Links between business records and evidence files |
| `file_link_request` | Reservations of the business targets to link an uploaded file to after it passes the scan, and their processing status |
| `validation_project` | Basic information of validation projects and their progress and closure status |
| `project_activity` | Activities selected for a project and their progress status |
| `project_system_baseline` | Approved system revision adopted by a project as its validation baseline |
| `app_user` | User accounts, their organization, account status and permission level |
| `workflow_instance` | Approval workflow instance for a specific business item |
| `iq_assessment` | IQ assessment revisions, test composition and approval status summary |
| `iq_item` | Installation qualification test content and criteria, with protocol revisions |
| `iq_execution` | Judgment, result corrections and approval records for each IQ execution attempt |

### Key Points

- Document revisions are managed separately from the revisions and approvals of the source business records. When only the document changes, unchanged source business records can be reused, and approval of the document does not replace approval of the source business records.
- The body, tables and basis of an approved document are preserved, and a new document revision is created when changes are needed. Even if the source or the validation baseline changes later, the baseline used by past documents is retained.
- A user upload can be used only when its latest scan attempt is `COMPLETED` with `NO_THREATS_FOUND`, and file registration and the `result_file_id` link have been completed. Scan completion alone does not allow use, and files not yet registered are blocked from attachment, download, preview and AI input. While a link request remains pending, blocked or failed, submission or result registration for that target cannot proceed.
- A scan retry is recorded as a new attempt for the same upload, and earlier judgments are preserved. Duplicate responses do not register the file twice, and no additional scan attempts are created for an upload whose registration is complete. The identity of the scanned original and the actually registered file is verified.
- Evidence files are linked to the relevant test execution or step record, and the original used for approval is preserved even if the file changes. Per-step evidence links can be found in the [IQ ERD](#erd-05). PDF generation jobs and the storage information of generated files are managed separately.

[↑ Back to contents](#top)

---

<a id="erd-13"></a>

## 13. Library & Regulatory Basis

**Tables: 12**

### Relationship Structure

```text
regulatory_source (regulatory document edition)
  └─ regulatory_clause (regulatory clause)
       ├─ requirement_regulation ─ requirement (URS)
       └─ library_item_regulation ─ library_item (reusable item)

library_item
  └─ used for authoring URS/FRA/IQ/OQ/PQ
```

### Overview

The library provides reusable items for authoring requirements, risk assessments and tests. The regulatory basis is managed as document editions and individual clauses, and the necessary clauses are linked to requirements and library items.

### Table Roles

| Table | Role |
|---|---|
| `library_item` | Reusable items for authoring requirements, risk assessments and tests |
| `regulatory_source` | Documents and editions of regulations, guidelines and internal SOPs |
| `regulatory_clause` | Individual clauses of regulatory documents and their location in the original text |
| `requirement_regulation` | Links between requirements and regulatory clauses, with the rationale for applying them |
| `library_item_regulation` | Links between library items and regulatory clauses, with the rationale for applying them |
| `file_asset` | Information identifying a file's storage location and original |
| `app_user` | User accounts, their organization, account status and permission level |
| `requirement` | Requirement content and acceptance criteria, with revision and approval status |
| `fra_item` | Risk scenarios, assessment results and SOP basis for each requirement |
| `iq_item` | Installation qualification test content and criteria, with protocol revisions |
| `oq_item` | Operational qualification test content and criteria, with protocol revisions |
| `pq_item` | Performance qualification test content and criteria, with protocol revisions |

### Key Points

- Content imported from the library is managed as independent business items of the project. Later library changes are not applied automatically, and when a URS is imported, its classification values and regulatory basis are copied along with it.
- When a regulation is revised, it is registered as a new edition. The edition, clause and citation information used for approval is preserved, and past links are retained even if a clause is later deactivated.
- Multiple regulatory clauses can be linked to a single requirement or library item. For the SOP basis of an FRA, the clause corresponding to the internal SOP is linked directly.

[↑ Back to contents](#top)

---

<a id="erd-14"></a>

## 14. AI Generation & Application

**Tables: 12**

### Relationship Structure

```text
ai_generation_job (generation request)
  └─ ai_generation_result (result set)
       └─ ai_result_item (individual draft)

User review/selection → applied to business items/document body → review/approval
```

### Overview

Manages the process from an AI generation request through user review to actual application in business work. Generation requests, result sets and individual drafts are kept separate, and the requirement, risk assessment, test or document body to which each adopted draft was applied is recorded. AI features are turned on and off in the organization settings.

### Table Roles

| Table | Role |
|---|---|
| `ai_generation_job` | AI generation requests, input conditions and progress status |
| `ai_generation_result` | Generated result sets and whether the user selected and applied them |
| `ai_result_item` | Individual drafts and the results of applying them to actual business work |
| `validation_project` | Basic information of validation projects and their progress and closure status |
| `app_user` | User accounts, their organization, account status and permission level |
| `requirement` | Requirement content and acceptance criteria, with revision and approval status |
| `fra_item` | Risk scenarios, assessment results and SOP basis for each requirement |
| `iq_item` | Installation qualification test content and criteria, with protocol revisions |
| `oq_item` | Operational qualification test content and criteria, with protocol revisions |
| `pq_item` | Performance qualification test content and criteria, with protocol revisions |
| `deliverable_revision` | Content, basis and approval status of each document revision |
| `deliverable_section` | Sections and body text included in a document revision |

### Key Points

- Generation completion, user selection and business application are separate states. The fact that the AI produced a result does not by itself mean that the result has been applied to business work or approved.
- New items can be reviewed in a preview before being applied, and are linked to the actual business items after application. Document generation is requested against a document revision, and the result is applied to the body of the corresponding section.
- AI results are applied only to drafts in progress or to new revisions. They do not overwrite original content after submission, and applied content also goes through the normal review and approval process.

[↑ Back to contents](#top)
