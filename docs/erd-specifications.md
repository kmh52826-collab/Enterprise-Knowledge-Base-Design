<a id="top"></a>

# ERD Specifications

This document describes the 73 tables of the Validation Management Platform across 14 business areas. Each area shows the information it manages and the connections among its main tables.

For an at-a-glance view of the connections between business areas, see the [Overall Data Structure Diagram](./erd-overview.md).

> [!NOTE]
> This document explains data modeling and system design. Example values are de-identified samples and do not represent actual operational data from any specific organization.

The relationship diagrams provide simplified views of the main connections. Shared tables may appear in multiple ERDs. For detailed structures, follow the ERD link in each area; for column definitions and business rules, see the [Data Dictionary](./data-dictionary.md).

## Table of Contents

1. [Organizations, Users, and Permissions](#erd-01)
2. [System Inventory and Project Management](#erd-02)
3. [Validation Planning and Preliminary Assessments](#erd-03)
4. [Requirements, Design, and Risk Assessment](#erd-04)
5. [IQ Installation Qualification Testing](#erd-05)
6. [OQ Operational Qualification Testing](#erd-06)
7. [PQ Performance Qualification Testing](#erd-07)
8. [Deviations, Actions, and Reruns](#erd-08)
9. [Requirements and Test Traceability](#erd-09)
10. [VSR Summary Reporting](#erd-10)
11. [Approval Workflows, Electronic Signatures, and Audit Records](#erd-11)
12. [Deliverables, Document Revisions, and Files](#erd-12)
13. [Library and Regulatory References](#erd-13)
14. [AI Generation and Application](#erd-14)

---

<a id="erd-01"></a>

## 1. Organizations, Users, and Permissions

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/01-organization-security>

**Table Count: 10**

### Relationship Structure

```text
organization
  ├─ app_user ─ user_role ─ role
  └─ user_group
       ├─ user_group_member ─ app_user
       └─ group_role ─ role

app_user / user_group
  ├─ access_permission_grant (Screen and project access permissions)
  └─ inventory_role_grant (Inventory business permissions)
```

### Structural Overview

Users and groups are managed by organization, and business roles and access permissions are assigned to individuals and groups. This structure separately manages roles such as author, reviewer, and approver; access permissions for screens and projects; and inventory business permissions.

### Table Roles

| Table | Role |
|---|---|
| `organization` | Information about the organization to which users and groups belong |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `role` | Definitions of business roles such as author, reviewer, and approver |
| `user_role` | Business roles assigned directly to users |
| `user_group` | User groups by organization |
| `user_group_member` | Users belonging to groups and their membership status |
| `group_role` | Global or project-specific business roles assigned to groups |
| `access_permission_grant` | Screen and project access permissions for individuals and groups |
| `inventory_role_grant` | Inventory authoring, review, approval, and disposal permissions for individuals and groups |
| `validation_project` | Basic validation project information and progress/closure status |

### Key Points

- Administrative permission levels and business roles are separate. An account's administrative permissions alone do not make the user a reviewer or approver.
- Permissions assigned directly to users and those inherited from their groups are applied together. Group permissions are evaluated based on active groups and memberships.
- Editing and disposal require both access permissions and business roles. Review and approval require assignment as an approval workflow assignee or substitute.

[↑ Back to Table of Contents](#top)

---

<a id="erd-02"></a>

## 2. System Inventory and Project Management

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/02-inventory-project>

**Table Count: 14**

### Relationship Structure

```text
system_asset (System subject to validation)
  ├─ system_asset_revision (Revision and approval history)
  └─ validation_project
       ├─ project_system_baseline (Adopted approved revision)
       ├─ project_member (Participants)
       ├─ project_activity ─ validation_activity
       └─ project_closure_request (Closure request)

validation_activity ─ activity_dependency (Activity prerequisites and dependencies)
```

### Structural Overview

Current information and revision history for the system subject to validation are managed as the basis for project operations. The project is linked to the approved revision adopted as its validation baseline, participants, execution activities, and closure requests.

### Table Roles

| Table | Role |
|---|---|
| `system_asset` | Current information and approval status of systems and equipment subject to validation |
| `system_asset_revision` | System change and approval history, with the original information as it existed at the time |
| `validation_project` | Basic validation project information and progress/closure status |
| `project_system_baseline` | Approved system revision adopted by the project as its validation baseline |
| `project_member` | Project participants and their assigned roles |
| `validation_activity` | Common list of execution activities, including validation planning, assessments, and testing |
| `project_activity` | Activities selected for the project and their progress status |
| `activity_dependency` | Prerequisite and dependency conditions between activities |
| `project_closure_request` | Project closure requests and links to approval history for each request round |
| `organization` | Information about the organization to which users and groups belong |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `role` | Definitions of business roles such as author, reviewer, and approver |
| `workflow_instance` | Approval workflow instance for a specific business operation |
| `electronic_signature` | Signer, signature target, and original content at the time of signing |

### Key Points

- The system's current information is distinguished from the validation baseline adopted by the project. Even if the system is revised later, the approved original content used by the project is preserved.
- Projects proceed according to their selected activities and the prerequisites and dependencies adopted at creation. Changes to shared conditions are not automatically applied to existing projects.
- Closure requests are managed by round. Even when a request is resubmitted after rejection, the reason and approval history of the previous request remain available.

[↑ Back to Table of Contents](#top)

---

<a id="erd-03"></a>

## 3. Validation Planning and Preliminary Assessments

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/03-planning-assessment>

**Table Count: 10**

### Relationship Structure

```text
validation_project
  ├─ vp_plan (Validation plan)
  │    └─ vp_section (Table of contents and body)
  ├─ qia_assessment (Quality impact assessment)
  │    └─ qia_module_item (Module)
  │         └─ qia_process (Process)
  └─ vendor_audit (Vendor audit)
```

### Structural Overview

This area manages the project's Validation Plan (VP), Quality Impact Assessment (QIA), and Vendor Audit (VA). The VP consists of a table of contents and body, while QIA consists of a shared assessment, modules, and processes. Vendor audits record the auditor, audit date, and assessment file.

### Table Roles

| Table | Role |
|---|---|
| `vp_plan` | Validation plan revisions and approval status |
| `vp_section` | Table of contents and body included in the validation plan |
| `qia_assessment` | Project-wide Part 11 assessment and evaluation criteria |
| `qia_module_item` | Quality impact assessment by module, with revision and approval status |
| `qia_process` | GxP assessment responses and evaluation criteria for each process within a module |
| `vendor_audit` | Vendor audit revisions, auditor, audit date, and assessment file |
| `validation_project` | Basic validation project information and progress/closure status |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `workflow_instance` | Approval workflow instance for a specific business operation |
| `file_asset` | File storage location and information identifying the original file |

### Key Points

- The VP, QIA modules, and vendor audits each manage their own revisions and approvals. Content and assessment responses as they existed at approval are preserved.
- QIA preserves not only responses but also the questions and evaluation criteria used at the time. Historical results remain interpretable even after the criteria change.
- QIA modules without processes are displayed as NON_GXP and may be approved. This label does not mean that process assessment has been completed.

[↑ Back to Table of Contents](#top)

---

<a id="erd-04"></a>

## 4. Requirements, Design, and Risk Assessment

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/04-requirements-design-risk>

**Table Count: 15**

### Relationship Structure

```text
validation_project
  ├─ requirement (URS)
  ├─ fds_spec (FDS)
  ├─ dds_spec (DDS)
  ├─ dq_assessment ─ dq_item (Design qualification assessment)
  └─ fra_assessment ─ fra_item (Functional risk assessment)

dq_item ─ requirement / fds_spec / dds_spec
fra_item ─ requirement
```

### Structural Overview

URS manages the requirements to be validated, while FDS and DDS manage functional and detailed design documents. DQ evaluates conformity between requirements and design, and FRA records risks and the rationale for responses to each requirement. Regulatory clauses and library items are linked as supporting references for authoring and assessment.

### Table Roles

| Table | Role |
|---|---|
| `requirement` | Requirement content, acceptance criteria, and revision/approval status |
| `fds_spec` | Functional design document files and revision/approval status |
| `dds_spec` | Detailed design document files and revision/approval status |
| `dq_assessment` | Project design qualification assessment information |
| `dq_item` | Conformity determinations and review details for requirements and design documents |
| `fra_assessment` | Project functional risk assessment information |
| `fra_item` | Risk scenarios, assessment results, and supporting SOP references for each requirement |
| `validation_project` | Basic validation project information and progress/closure status |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `workflow_instance` | Approval workflow instance for a specific business operation |
| `library_item` | Reusable items for authoring requirements, risk assessments, and tests |
| `file_asset` | File storage location and information identifying the original file |
| `regulatory_clause` | Individual clauses in regulatory documents and their locations in the source text |
| `requirement_regulation` | Links between requirements and regulatory clauses, with applicability rationale |
| `regulatory_source` | Documents and editions of regulations, guidelines, and internal SOPs |

### Key Points

- FDS and DDS are managed as independent documents. Replacing a file creates a new document revision while preserving the existing original.
- A DQ item links one URS revision to at most one FDS revision and at most one DDS revision. An approval request requires at least one valid approved design document from the same project.
- DQ and FRA preserve the exact requirement and design revisions used in the assessment. FRA also retains the evaluation criteria and risk assessment results as they existed at approval.

[↑ Back to Table of Contents](#top)

---

<a id="erd-05"></a>

## 5. IQ Installation Qualification Testing

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/05-iq-testing>

**Table Count: 13**

### Relationship Structure

```text
validation_project
  └─ iq_assessment (IQ assessment)
       └─ iq_item (Test item and protocol)
            ├─ iq_step (Test step)
            └─ iq_execution (Execution attempt)
                 └─ iq_step_execution (Results by step)
```

### Structural Overview

IQ manages test content for verifying installation qualification and the actual execution results. Test items and detailed steps are organized under an assessment, with outcomes and step completion records maintained for each execution attempt.

### Table Roles

| Table | Role |
|---|---|
| `iq_assessment` | IQ assessment revisions, test composition, and approval status summaries |
| `iq_item` | Installation qualification test content, criteria, and protocol revisions |
| `iq_step` | Detailed steps and their order within an IQ test item |
| `iq_execution` | Outcomes, result corrections, and approval records for each IQ execution attempt |
| `iq_step_execution` | Completion status and processing records for each step of an IQ execution |
| `validation_project` | Basic validation project information and progress/closure status |
| `project_system_baseline` | Approved system revision adopted by the project as its validation baseline |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `library_item` | Reusable items for authoring requirements, risk assessments, and tests |
| `workflow_instance` | Approval workflow instance for a specific business operation |
| `electronic_signature` | Signer, signature target, and original content at the time of signing |
| `evidence_link` | Links between business records and evidence files |
| `file_asset` | File storage location and information identifying the original file |

### Key Points

- The test composition of an assessment is distinguished from revisions of individual tests. When an assessment is revised, tests whose content has not changed may be included again.
- Tests are performed using approved protocols. Test outcomes and approval of execution results are managed separately.
- Reruns are recorded as new attempts, while corrections to existing results are recorded as revision history within the same attempt. Historical results, evidence, and the system baseline at the time of execution are preserved.

[↑ Back to Table of Contents](#top)

---

<a id="erd-06"></a>

## 6. OQ Operational Qualification Testing

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/06-oq-testing>

**Table Count: 13**

### Relationship Structure

```text
validation_project
  └─ oq_assessment (OQ assessment)
       └─ oq_item (Test item and protocol)
            ├─ oq_step (Test step)
            └─ oq_execution (Execution attempt)
                 └─ oq_step_execution (Results by step)
```

### Structural Overview

OQ manages the content, expected results, and acceptance criteria of operational qualification tests, together with actual execution results. Following the same structure as IQ, it separates assessments, test items, detailed steps, and execution records, while managing OQ data and approval statuses independently.

### Table Roles

| Table | Role |
|---|---|
| `oq_assessment` | OQ assessment revisions, test composition, and approval status summaries |
| `oq_item` | Operational qualification test content, criteria, and protocol revisions |
| `oq_step` | Detailed steps and their order within an OQ test item |
| `oq_execution` | Outcomes, result corrections, and approval records for each OQ execution attempt |
| `oq_step_execution` | Completion status and processing records for each step of an OQ execution |
| `validation_project` | Basic validation project information and progress/closure status |
| `project_system_baseline` | Approved system revision adopted by the project as its validation baseline |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `library_item` | Reusable items for authoring requirements, risk assessments, and tests |
| `workflow_instance` | Approval workflow instance for a specific business operation |
| `electronic_signature` | Signer, signature target, and original content at the time of signing |
| `evidence_link` | Links between business records and evidence files |
| `file_asset` | File storage location and information identifying the original file |

### Key Points

- A change in test content requires a new protocol revision. Existing execution records continue to reference the protocol used at the time.
- Test outcomes are distinguished from result approval status. Reruns and result corrections are also recorded in separate histories.
- Prerequisite and dependency relationships with IQ and PQ are managed through project activities. Links to requirements are covered in the traceability area, and actions addressing test failures are covered in the deviation area.

[↑ Back to Table of Contents](#top)

---

<a id="erd-07"></a>

## 7. PQ Performance Qualification Testing

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/07-pq-testing>

**Table Count: 13**

### Relationship Structure

```text
validation_project
  └─ pq_assessment (PQ assessment)
       └─ pq_item (Test item and protocol)
            ├─ pq_step (Test step)
            └─ pq_execution (Execution attempt)
                 └─ pq_step_execution (Results by step)
```

### Structural Overview

PQ manages the composition of performance qualification tests and their actual execution results. Test lists for each assessment, test content and steps, and outcomes and completion records for each attempt are preserved separately.

### Table Roles

| Table | Role |
|---|---|
| `pq_assessment` | PQ assessment revisions, test composition, and approval status summaries |
| `pq_item` | Performance qualification test content, criteria, and protocol revisions |
| `pq_step` | Detailed steps and their order within a PQ test item |
| `pq_execution` | Outcomes, result corrections, and approval records for each PQ execution attempt |
| `pq_step_execution` | Completion status and processing records for each step of a PQ execution |
| `validation_project` | Basic validation project information and progress/closure status |
| `project_system_baseline` | Approved system revision adopted by the project as its validation baseline |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `library_item` | Reusable items for authoring requirements, risk assessments, and tests |
| `workflow_instance` | Approval workflow instance for a specific business operation |
| `electronic_signature` | Signer, signature target, and original content at the time of signing |
| `evidence_link` | Links between business records and evidence files |
| `file_asset` | File storage location and information identifying the original file |

### Key Points

- Assessment composition, test protocols, and execution results are managed separately. The same approved protocol may be executed across multiple attempts.
- Result registration requires step completion and an execution signature. Subsequent changes follow a correction procedure that preserves the original results and evidence.
- Historical executions retain their baselines even if the project's current system baseline changes. Approved execution results support the VSR summary report.

[↑ Back to Table of Contents](#top)

---

<a id="erd-08"></a>

## 8. Deviations, Actions, and Reruns

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/08-deviation-rerun>

**Table Count: 9**

### Relationship Structure

```text
iq_execution / oq_execution / pq_execution (Failed test execution)
    ↓ Register deviation
deviation
  └─ deviation_action_round (Actions and approvals by round)
       ↓ Rerun
iq_execution / oq_execution / pq_execution (New execution record)

Add an action round if the rerun fails again
```

### Structural Overview

Deviations arising from test failures are registered and managed from action approval through reruns to final closure. `deviation` manages the overall deviation status, while `deviation_action_round` manages the action content and approval/rerun history for each round.

### Table Roles

| Table | Role |
|---|---|
| `deviation` | Deviations arising from test failures and their overall processing status |
| `deviation_action_round` | Action content, approval history, and rerun links for each round |
| `validation_project` | Basic validation project information and progress/closure status |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `workflow_instance` | Approval workflow instance for a specific business operation |
| `electronic_signature` | Signer, signature target, and original content at the time of signing |
| `iq_execution` | Outcomes, result corrections, and approval records for each IQ execution attempt |
| `oq_execution` | Outcomes, result corrections, and approval records for each OQ execution attempt |
| `pq_execution` | Outcomes, result corrections, and approval records for each PQ execution attempt |

### Key Points

- The initial failure record is preserved, and each action is linked to its supporting approval records and rerun results. If a rerun fails again, the next action round is added.
- When the same action is modified after rejection, it is managed as a new revision within that round. Approvals and signatures are linked to the original action content as it existed at the time.
- The path that approves a completion report after a successful rerun is distinguished from the path that closes with a recorded reason after action approval, without a rerun. The final signature appropriate to each closure path is preserved.

[↑ Back to Table of Contents](#top)

---

<a id="erd-09"></a>

## 9. Requirements and Test Traceability

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/09-traceability>

**Table Count: 11**

### Relationship Structure

```text
requirement (URS)
  └─ traceability_link (Traceability relationship)
       ├─ fds_spec / dds_spec (Design)
       ├─ dq_item / fra_item (Design and risk assessments)
       └─ iq_item / oq_item / pq_item (Test items)

Traceability relationships and test results → RTM dashboard
```

### Structural Overview

`traceability_link` connects requirements to the designs, assessments, and tests that address them. The RTM dashboard retrieves these relationships together with actual test results to show validation status for each requirement.

### Table Roles

| Table | Role |
|---|---|
| `traceability_link` | Traceability relationships among requirements, designs, assessments, and tests |
| `validation_project` | Basic validation project information and progress/closure status |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `requirement` | Requirement content, acceptance criteria, and revision/approval status |
| `fds_spec` | Functional design document files and revision/approval status |
| `dds_spec` | Detailed design document files and revision/approval status |
| `dq_item` | Conformity determinations and review details for requirements and design documents |
| `fra_item` | Risk scenarios, assessment results, and supporting SOP references for each requirement |
| `iq_item` | Installation qualification test content, criteria, and protocol revisions |
| `oq_item` | Operational qualification test content, criteria, and protocol revisions |
| `pq_item` | Performance qualification test content, criteria, and protocol revisions |

### Key Points

- Traceability relationships point to the exact business revisions used when the links were established. New revisions do not automatically change the links supporting historical approvals.
- Multiple requirements may be linked to a single test. Whether a requirement is linked to a test and whether that test has actually passed and been approved are checked separately.
- RTM operates as a dashboard query function. It is not managed as an independent execution stage or approval document, and it is not included as a separate item in project progress or the VSR activity list.

[↑ Back to Table of Contents](#top)

---

<a id="erd-10"></a>

## 10. VSR Summary Reporting

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/10-vsr-summary>

**Table Count: 14**

### Relationship Structure

```text
validation_project
  └─ vsr_assessment (Summary report)
       └─ vsr_item (Result summaries by activity)
            ├─ project_activity (Execution activity)
            └─ deliverable_revision (Supporting document)

Planning, assessment, and test results → VSR aggregation and approval → Project closure decision
```

### Structural Overview

The project's planning, assessment, and test results are collected to form the Validation Summary Report (VSR). `vsr_assessment` manages the overall conclusion and confirmation status, while `vsr_item` manages results and supporting references for each activity. Approved reports preserve the results and approval information as they existed at aggregation time.

### Table Roles

| Table | Role |
|---|---|
| `vsr_assessment` | Summary report revisions, conclusions, and final confirmation status |
| `vsr_item` | Result summaries for each execution activity and supporting references at aggregation time |
| `validation_project` | Basic validation project information and progress/closure status |
| `project_activity` | Activities selected for the project and their progress status |
| `deliverable_revision` | Content, supporting references, and approval status for each document revision |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `workflow_instance` | Approval workflow instance for a specific business operation |
| `electronic_signature` | Signer, signature target, and original content at the time of signing |
| `vp_plan` | Validation plan revisions and approval status |
| `qia_module_item` | Quality impact assessment by module, with revision and approval status |
| `vendor_audit` | Vendor audit revisions, auditor, audit date, and assessment file |
| `iq_execution` | Outcomes, result corrections, and approval records for each IQ execution attempt |
| `oq_execution` | Outcomes, result corrections, and approval records for each OQ execution attempt |
| `pq_execution` | Outcomes, result corrections, and approval records for each PQ execution attempt |

### Key Points

- Execution activities selected for the project are aggregated, while RTM and the VSR itself are excluded from the detailed activity list. The planning, assessment, and test tables in the ERD are representative examples of aggregation sources.
- VSR business confirmation and approval of the output document are managed separately. Business confirmation may be complete even when the output document has not yet been generated.
- If source results change after approval, they are aggregated again and confirmed in a new VSR revision. The results and supporting references of historical reports remain unchanged.

[↑ Back to Table of Contents](#top)

---

<a id="erd-11"></a>

## 11. Approval Workflows, Electronic Signatures, and Audit Records

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/11-workflow-signature-audit>

**Table Count: 15**

### Relationship Structure

```text
project_workflow_config (Approval route configuration)
  └─ workflow_instance (Approval workflow execution)
       └─ workflow_step (Review and approval steps)
            ├─ workflow_step_assignee (Assignee allocation)
            └─ approval_action (Action history)
                 └─ electronic_signature (Electronic signature)

audit_trail (Audit records across business operations)
```

### Structural Overview

Review and approval procedures, assignees, and processing results used across business operations are managed centrally. Individual approval workflows follow the configured approval routes, with assignments and action history recorded for each step. Electronic signatures preserve the original signed content, while audit records preserve actions such as changes, access, and exports.

### Table Roles

| Table | Role |
|---|---|
| `workflow_instance` | Approval workflow instance for a specific business operation |
| `workflow_step` | Review and approval steps and their processing order within a workflow instance |
| `workflow_step_assignee` | Primary and substitute assignee allocations for each approval workflow step |
| `approval_action` | Action history for submission, review, approval, rejection, and cancellation |
| `project_workflow_config` | Project-wide or activity-specific approval route configuration |
| `electronic_signature` | Signer, signature target, and original content at the time of signing |
| `audit_trail` | User and system actions and data change history |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `validation_project` | Basic validation project information and progress/closure status |
| `project_activity` | Activities selected for the project and their progress status |
| `requirement` | Requirement content, acceptance criteria, and revision/approval status |
| `deliverable_revision` | Content, supporting references, and approval status for each document revision |
| `deviation_action_round` | Action content, approval history, and rerun links for each round |
| `system_asset` | Current information and approval status of systems and equipment subject to validation |
| `project_closure_request` | Project closure requests and links to approval history for each request round |

### Key Points

- Approval route configuration is distinguished from actual workflow instances. Each submission records review and approval history according to the route and assignments applied at the time.
- Electronic signatures are linked to exact targets and revisions, and the original signed content is preserved with them. Changing the body after signing requires a new revision and a new signature.
- Approval status and assignee actions are available in approval records, while actions across business operations are available in audit records. Audit records are appended without modifying existing entries.

[↑ Back to Table of Contents](#top)

---

<a id="erd-12"></a>

## 12. Deliverables, Document Revisions, and Files

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/12-documents-files>

**Table Count: 15**

### Relationship Structure

```text
deliverable_document (Deliverable)
  └─ deliverable_revision (Document revision)
       ├─ deliverable_section (Table of contents and body)
       └─ report_generation (PDF generation)
            └─ file_asset (Result file)

User upload → file_scan_job (Scanning) → file_asset (Registration after passing the scan)
Business record ─ evidence_link (Evidence link) ─ file_asset
```

### Structural Overview

Project deliverables are managed as documents, revisions, and tables of contents/body sections, with links to PDF generation and evidence files. Each document revision preserves the business content on which it was based and the results at that time. This ERD uses IQ as an example to show the links to supporting document sources and test evidence.

For user uploads, scan attempts and results for the quarantined original are recorded in `file_scan_job`. The file is registered in `file_asset` after no threats are found and its identity with the original is verified. `file_origin` distinguishes user uploads (`UPLOAD`) from files generated internally by a trusted server (`SYSTEM_GENERATED`). Server-generated PDF and export files may be registered without an upload scanning job; external uploaded files used in generation must first pass scanning.

### Table Roles

| Table | Role |
|---|---|
| `deliverable_document` | Deliverable document number, type, and associated activity |
| `deliverable_revision` | Content, supporting references, and approval status for each document revision |
| `deliverable_section` | Table of contents and body included in a document revision |
| `report_generation` | PDF generation requests, progress status, and result files |
| `file_scan_job` | Quarantined originals of user uploads, scan attempts/results, and links to registered files |
| `file_asset` | Generation origin, storage location, and original file identification for files used in business operations |
| `evidence_link` | Links between business records and evidence files |
| `validation_project` | Basic validation project information and progress/closure status |
| `project_activity` | Activities selected for the project and their progress status |
| `project_system_baseline` | Approved system revision adopted by the project as its validation baseline |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `workflow_instance` | Approval workflow instance for a specific business operation |
| `iq_assessment` | IQ assessment revisions, test composition, and approval status summaries |
| `iq_item` | Installation qualification test content, criteria, and protocol revisions |
| `iq_execution` | Outcomes, result corrections, and approval records for each IQ execution attempt |

### Key Points

- Document revisions and the revisions/approvals of source business records are managed separately. When only a document changes, unchanged source business records may be reused; document approval does not replace approval of those source records.
- The body, tables, and supporting references of approved documents are preserved. Changes require a new document revision. The baseline used by a historical document is retained even if source data or the validation target baseline subsequently changes.
- User uploads may be used only when the latest scan attempt is `COMPLETED` with `NO_THREATS_FOUND`, and file registration and the `result_file_id` link are complete. Scan completion alone does not permit use. Before registration, files are blocked from attachments, downloads, previews, and AI input.
- Scan retries are recorded as new attempts for the same upload, preserving existing verdicts. Duplicate responses do not register a file twice, and no additional scan attempts are created for registered uploads. Identity between the scanned original and the file actually registered is verified.
- Evidence files are linked to the corresponding test execution or step record, and originals used for approval are preserved even when files change. Step-level evidence links are shown in the [IQ ERD](#erd-05). PDF generation jobs and storage information for generated files are managed separately.

[↑ Back to Table of Contents](#top)

---

<a id="erd-13"></a>

## 13. Library and Regulatory References

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/13-library-regulation>

**Table Count: 12**

### Relationship Structure

```text
regulatory_source (Regulatory document edition)
  └─ regulatory_clause (Regulatory clause)
       ├─ requirement_regulation ─ requirement (URS)
       └─ library_item_regulation ─ library_item (Reusable item)

library_item
  └─ Used to author URS·FRA·IQ·OQ·PQ
```

### Structural Overview

The library provides reusable items for authoring requirements, risk assessments, and tests. Regulatory references are managed as document editions and individual clauses, with relevant clauses linked to requirements and library items.

### Table Roles

| Table | Role |
|---|---|
| `library_item` | Reusable items for authoring requirements, risk assessments, and tests |
| `regulatory_source` | Documents and editions of regulations, guidelines, and internal SOPs |
| `regulatory_clause` | Individual clauses in regulatory documents and their locations in the source text |
| `requirement_regulation` | Links between requirements and regulatory clauses, with applicability rationale |
| `library_item_regulation` | Links between library items and regulatory clauses, with applicability rationale |
| `file_asset` | File storage location and information identifying the original file |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `requirement` | Requirement content, acceptance criteria, and revision/approval status |
| `fra_item` | Risk scenarios, assessment results, and supporting SOP references for each requirement |
| `iq_item` | Installation qualification test content, criteria, and protocol revisions |
| `oq_item` | Operational qualification test content, criteria, and protocol revisions |
| `pq_item` | Performance qualification test content, criteria, and protocol revisions |

### Key Points

- Content imported from the library is managed as independent business items within the project. Subsequent library changes are not applied automatically, and regulatory references are copied along with imported URS content.
- Revised regulations are registered as new editions. Editions, clauses, and citation information used for approval are preserved, and historical links remain intact even if a clause is later deactivated.
- Multiple regulatory clauses may be linked to a requirement or library item. FRA SOP references link directly to clauses belonging to internal SOPs.

[↑ Back to Table of Contents](#top)

---

<a id="erd-14"></a>

## 14. AI Generation and Application

ERD Link: <https://drawsql.app/teams/minho-kim/diagrams/14-ai-generation>

**Table Count: 12**

### Relationship Structure

```text
ai_generation_job (Generation request)
  └─ ai_generation_result (Result collection)
       └─ ai_result_item (Individual draft)

User review and selection → Application to business items and document bodies → Review and approval
```

### Structural Overview

This area manages the process from AI generation requests through user review to application in actual business records. Generation requests, result collections, and individual drafts are distinguished, and the requirements, risk assessments, tests, or document bodies to which adopted drafts were applied are recorded.

### Table Roles

| Table | Role |
|---|---|
| `ai_generation_job` | AI generation requests, input conditions, and progress status |
| `ai_generation_result` | Generated result collections and whether users selected and applied them |
| `ai_result_item` | Individual drafts and the results of applying them to actual business records |
| `validation_project` | Basic validation project information and progress/closure status |
| `app_user` | User accounts, organizational affiliation, account status, and administrative permissions |
| `requirement` | Requirement content, acceptance criteria, and revision/approval status |
| `fra_item` | Risk scenarios, assessment results, and supporting SOP references for each requirement |
| `iq_item` | Installation qualification test content, criteria, and protocol revisions |
| `oq_item` | Operational qualification test content, criteria, and protocol revisions |
| `pq_item` | Performance qualification test content, criteria, and protocol revisions |
| `deliverable_revision` | Content, supporting references, and approval status for each document revision |
| `deliverable_section` | Table of contents and body included in a document revision |

### Key Points

- Generation completion, user selection, and application to business records are separate states. AI generation of a result alone does not mean that it has been applied or approved.
- New items can be reviewed in a preview before application and are linked to actual business items after application. Document generation requests target a document revision, and results are applied to the body of the corresponding table-of-contents section.
- AI results are applied to editable drafts or new revisions. Approved original content is not overwritten, and applied content follows the normal review and approval procedures.

[↑ Back to Table of Contents](#top)
