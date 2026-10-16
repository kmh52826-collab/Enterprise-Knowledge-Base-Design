<a id="top"></a>

# Data Dictionary

> Validation Management Platform data model document  
> **Example values**: personal and sensitive information is shown as de-identified sample values

For table relationships by business area, see **[ERD Specifications](./erd-specifications.md)**; for the overall structure across areas, see **[Overall Data Structure](./erd-overview.md)**.

## Contents

- [1. Document Overview](#1-document-overview)
- [2. Notation](#2-notation)
- [3. Table List](#3-table-list)
- [4. Shortcuts by Domain](#4-shortcuts-by-domain)
- [5. Table and Column Details](#5-table-and-column-details)

## 1. Document Overview

- Total tables: **77**
- Total columns: **1,263**
- Domains: **26**
- Naming convention: tables and columns use `snake_case`; the PK is an identifier column per entity
- Date/time convention: stored in UTC in the DB; the user's or site's Timezone is applied on screens
- GxP principle: approved records are not modified directly but are managed through Revisions and status history, and key changes are recorded in the Audit Trail

## 2. Notation

| Notation | Meaning |
|---|---|
| `Y` | Applied or required |
| `N` | Not applied or optional |
| `-` | Not applicable or unspecified |
| PK | Primary Key |
| FK | Foreign Key |
| Not Null | The inverse of whether NULL is allowed |
| Audit | Whether the column is subject to Audit Trail recording |
| UQ | Whether the individual column is unique. Composite/conditional uniqueness is defined separately in the constraints of the table |
| IDX | Whether an index is defined. Indexes created by the PK or an individual UNIQUE are also marked Y; for composite/conditional indexes, also check the constraints |
| Sensitive | Personal / sensitive information classification |
| Default | SQL expression. Strings are written in single quotes, Booleans as TRUE/FALSE, and functions as the function expression itself; `-` means no default value |

`date` examples are written as `YYYY-MM-DD`, and `timestamptz` examples as `YYYY-MM-DDTHH:mm:ssZ` in UTC.

The component columns of a composite/conditional uniqueness are not individually marked `UQ=Y`. Which data counts as active is determined by the `WHERE` condition of the corresponding constraint.

Values with Sensitive=Y are viewed and exported only by permitted users and for permitted processing purposes. Real names, email addresses, IP addresses, names of auditors and approvers, and Audit JSON that may contain personal information are included in this classification. password_hash/previous_password_hash and user_signature_credential.signature_password_hash/previous_signature_password_hash are excluded from responses, general exports, and Audit JSON, and keep Audit=N.

NN=N does not mean that the value is always optional, even at approval or completion. Conditionally required values follow the constraints and business rules of the table. Type+ID pairs that point to multiple tables and references inside JSON are distinguished from actual FKs.

PK, FK, CHECK, and UNIQUE in the DB are constraints enforced on save. Each row of a constraint table defines one DB constraint or business validation. A conditional UNIQUE applies only to rows that meet its WHERE condition. Business validation is performed on the server and is not replaced solely by selection restrictions in the UI.

When a reference is saved, submitted, or approved, the existence of the target, the match of the parent, project, and protocol revision, and the current usable/approvable status are checked. For polymorphic references and references inside JSON, the allowed actual table, PK, and version and the object format are checked. Links between project business records require the same owning project, and for non-project materials such as shared regulations and libraries, the view and use permissions on that material are checked. New business links may not point to deleted targets or to business records of another project. Viewing a past approved reference keeps the original content of that time and distinguishes it from any later disposal or loss-of-validity status.

Concurrent requests are controlled by server transactions together with target row locks or saved-version comparison. For duplicate approval, number issuance, switching the current revision, and creating an in-progress closure request, checking and saving are not separated. A request whose saved version has changed is treated as a conflict, and the latest data is not overwritten with values from an earlier screen. Status transitions, signatures, and audit trail records are committed together or rolled back together, and audit change fields are not arbitrarily omitted.

Versioned question and judgment rule definitions (QIA Part 11/GxP questions, FRA risk judgment, meaning of activity dependency judgments) are not kept in a separate master table but are managed as server deployment settings identified by version name. A different definition is never deployed under the same version name, and the original content copied into each assessment, document, and project (question_set_snapshot, rule_snapshot, evaluation_definition_snapshot, dependency_snapshot) serves as the audit basis.

For business revisions that have an approval status, the body, sub-steps, composition, and attachments are locked after submission (REVIEW). Saving a DRAFT before submission updates the same row and records the before and after values in the Audit Trail. Rejection (REJECTED) and cancellation (CANCELLED) are terminal states of the revision and do not revert to DRAFT. Edits after rejection or cancellation and changes after approval are handled as a new revision (DRAFT) of the same logical key, and the terminated revision and its Workflow and electronic signatures are retained (for inventory, revision_number is incremented; for test execution results, record_revision is incremented under the same attempt_no). Screens may show this as 'edit', but a new revision row is created in the DB.

## 3. Table List

| No | Domain | Logical Table Name | Physical Table Name | PK | Main References (FK) | GxP | Audit |
|---:|---|---|---|---|---|:---:|:---:|
| 1 | Organization | Organization / Customer | [`organization`](#table-organization) | `organization_id` | - | High | Y |
| 2 | Security | User | [`app_user`](#table-app_user) | `user_id` | `organization_id` | High | Y |
| 3 | Security | Role | [`role`](#table-role) | `role_id` | - | High | Y |
| 4 | Security | User Role | [`user_role`](#table-user_role) | `user_role_id` | `user_id, role_id, created_by, updated_by` | High | Y |
| 5 | Security | User Group | [`user_group`](#table-user_group) | `user_group_id` | `organization_id, created_by, updated_by` | High | Y |
| 6 | Security | User Group Member | [`user_group_member`](#table-user_group_member) | `user_group_member_id` | `user_group_id, user_id, created_by, updated_by` | High | Y |
| 7 | Security | User Group Role | [`group_role`](#table-group_role) | `group_role_id` | `user_group_id, role_id, project_id, created_by, updated_by` | High | Y |
| 8 | Security | Menu/Project Access Permission | [`access_permission_grant`](#table-access_permission_grant) | `grant_id` | `user_id, user_group_id, project_id, created_by, updated_by` | High | Y |
| 9 | Security | Inventory Business Role | [`inventory_role_grant`](#table-inventory_role_grant) | `grant_id` | `user_id, user_group_id, created_by, updated_by` | High | Y |
| 10 | Security | Electronic Signature Password | [`user_signature_credential`](#table-user_signature_credential) | `credential_id` | `user_id, reset_requested_by, created_by, updated_by` | High | Y |
| 11 | Compliance | Electronic Signature | [`electronic_signature`](#table-electronic_signature) | `signature_id` | `signer_id` | Critical | Y |
| 12 | Compliance | Audit Trail | [`audit_trail`](#table-audit_trail) | `audit_id` | `actor_id` | Critical | Y |
| 13 | File | File Asset | [`file_asset`](#table-file_asset) | `file_id` | `uploader_id` | High | Y |
| 14 | File | File Malware Scan Job | [`file_scan_job`](#table-file_scan_job) | `scan_job_id` | `previous_scan_job_id, project_id, uploaded_by, requested_by, result_file_id` | High | Y |
| 15 | File | Evidence File Link | [`evidence_link`](#table-evidence_link) | `evidence_link_id` | `project_id, file_id, created_by, updated_by` | High | Y |
| 16 | File | File Link Request | [`file_link_request`](#table-file_link_request) | `link_request_id` | `project_id, file_id, evidence_link_id, created_by, updated_by` | High | Y |
| 17 | System | System/Equipment Identification | [`system_asset`](#table-system_asset) | `system_id` | `organization_id` | High | Y |
| 18 | System | System Inventory Revision History | [`system_asset_revision`](#table-system_asset_revision) | `system_revision_id` | `system_id, processed_by, signature_id` | High | Y |
| 19 | Library | Library Item Master | [`library_item`](#table-library_item) | `library_id` | - | High | Y |
| 20 | Validation | Validation Project | [`validation_project`](#table-validation_project) | `project_id` | `system_id, created_by, updated_by, closure_requested_by, current_closure_request_id, current_system_baseline_id` | High | Y |
| 21 | Validation | Project Validation Target Revision | [`project_system_baseline`](#table-project_system_baseline) | `baseline_id` | `project_id, system_revision_id, adopted_by` | Critical | Y |
| 22 | Validation | Project Member | [`project_member`](#table-project_member) | `project_member_id` | `project_id, user_id, role_id, created_by, updated_by` | High | Y |
| 23 | Validation | Validation Activity Master | [`validation_activity`](#table-validation_activity) | `activity_id` | `created_by, updated_by` | High | Y |
| 24 | Validation | Project Activity | [`project_activity`](#table-project_activity) | `project_activity_id` | `project_id, activity_id, created_by, updated_by` | Critical | Y |
| 25 | Validation | Activity Dependency Condition | [`activity_dependency`](#table-activity_dependency) | `activity_dependency_id` | `successor_activity_id, predecessor_activity_id, created_by, updated_by` | Critical | Y |
| 26 | Validation | Project Closure Request | [`project_closure_request`](#table-project_closure_request) | `closure_request_id` | `project_id, requested_by, workflow_instance_id, created_by, updated_by` | High | Y |
| 27 | VP | Validation Plan | [`vp_plan`](#table-vp_plan) | `vp_id` | `project_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` | High | Y |
| 28 | VP | Validation Plan Section | [`vp_section`](#table-vp_section) | `vp_section_id` | `vp_id, created_by, updated_by` | High | Y |
| 29 | QIA | Quality Impact Assessment Header | [`qia_assessment`](#table-qia_assessment) | `qia_id` | `project_id, created_by, updated_by` | High | Y |
| 30 | QIA | QIA Module Detailed Assessment | [`qia_module_item`](#table-qia_module_item) | `qia_module_item_id` | `qia_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` | High | Y |
| 31 | QIA | QIA Process Assessment | [`qia_process`](#table-qia_process) | `qia_process_id` | `qia_module_item_id, created_by, updated_by` | High | Y |
| 32 | VA | Vendor Audit Assessment | [`vendor_audit`](#table-vendor_audit) | `audit_id` | `project_id, auditor_user_id, file_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` | High | Y |
| 33 | URS | User Requirements Specification | [`requirement`](#table-requirement) | `requirement_id` | `project_id, created_by, workflow_instance_id, updated_by, disposal_workflow_id, source_library_id` | High | Y |
| 34 | FDS | Functional Design Specification | [`fds_spec`](#table-fds_spec) | `fds_id` | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` | High | Y |
| 35 | DDS | Detailed Design Specification | [`dds_spec`](#table-dds_spec) | `dds_id` | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` | High | Y |
| 36 | DQ | Design Qualification Assessment | [`dq_assessment`](#table-dq_assessment) | `dq_id` | `project_id, created_by, updated_by` | High | Y |
| 37 | DQ | DQ Detailed Assessment Item | [`dq_item`](#table-dq_item) | `dq_item_id` | `dq_id, requirement_id, created_by, updated_by, fds_revision_id, dds_revision_id, executed_by, workflow_instance_id, disposal_workflow_id` | High | Y |
| 38 | FRA | Functional Risk Assessment | [`fra_assessment`](#table-fra_assessment) | `fra_id` | `project_id, created_by, updated_by` | High | Y |
| 39 | FRA | FRA Risk Detail Item | [`fra_item`](#table-fra_item) | `fra_item_id` | `fra_id, requirement_id, created_by, updated_by, sop_clause_id, workflow_instance_id, disposal_workflow_id, source_library_id` | High | Y |
| 40 | IQ | Installation Qualification Assessment | [`iq_assessment`](#table-iq_assessment) | `iq_id` | `project_id, created_by, updated_by` | High | Y |
| 41 | IQ | IQ Detailed Test Item | [`iq_item`](#table-iq_item) | `iq_item_id` | `iq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_workflow_id, disposal_signature_id, disposed_by` | High | Y |
| 42 | IQ | IQ Test Step | [`iq_step`](#table-iq_step) | `step_id` | `iq_item_id, created_by, updated_by` | High | Y |
| 43 | IQ | IQ Test Execution | [`iq_execution`](#table-iq_execution) | `execution_id` | `iq_item_id, executed_by, execution_signature_id, abort_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` | High | Y |
| 44 | IQ | IQ Step Execution | [`iq_step_execution`](#table-iq_step_execution) | `step_execution_id` | `execution_id, step_id, confirmed_by, created_by, updated_by` | High | Y |
| 45 | OQ | Operational Qualification Assessment | [`oq_assessment`](#table-oq_assessment) | `oq_id` | `project_id, created_by, updated_by` | High | Y |
| 46 | OQ | OQ Detailed Test Item | [`oq_item`](#table-oq_item) | `oq_item_id` | `oq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_workflow_id, disposal_signature_id, disposed_by` | High | Y |
| 47 | OQ | OQ Test Step | [`oq_step`](#table-oq_step) | `step_id` | `oq_item_id, created_by, updated_by` | High | Y |
| 48 | OQ | OQ Test Execution | [`oq_execution`](#table-oq_execution) | `execution_id` | `oq_item_id, executed_by, execution_signature_id, abort_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` | High | Y |
| 49 | OQ | OQ Step Execution | [`oq_step_execution`](#table-oq_step_execution) | `step_execution_id` | `execution_id, step_id, confirmed_by, created_by, updated_by` | High | Y |
| 50 | PQ | Performance Qualification Assessment | [`pq_assessment`](#table-pq_assessment) | `pq_id` | `project_id, created_by, updated_by` | High | Y |
| 51 | PQ | PQ Detailed Test Item | [`pq_item`](#table-pq_item) | `pq_item_id` | `pq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_workflow_id, disposal_signature_id, disposed_by` | High | Y |
| 52 | PQ | PQ Test Step | [`pq_step`](#table-pq_step) | `step_id` | `pq_item_id, created_by, updated_by` | High | Y |
| 53 | PQ | PQ Test Execution | [`pq_execution`](#table-pq_execution) | `execution_id` | `pq_item_id, executed_by, execution_signature_id, abort_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` | High | Y |
| 54 | PQ | PQ Step Execution | [`pq_step_execution`](#table-pq_step_execution) | `step_execution_id` | `execution_id, step_id, confirmed_by, created_by, updated_by` | High | Y |
| 55 | VSR | Validation Summary Report | [`vsr_assessment`](#table-vsr_assessment) | `vsr_id` | `project_id, created_by, updated_by, workflow_instance_id, approval_signature_id` | Critical | Y |
| 56 | VSR | VSR Activity Summary Item | [`vsr_item`](#table-vsr_item) | `vsr_item_id` | `vsr_id, created_by, updated_by, project_activity_id, document_revision_id` | Critical | Y |
| 57 | Workflow | Workflow Instance | [`workflow_instance`](#table-workflow_instance) | `workflow_instance_id` | `requested_by, created_by, updated_by, project_id, workflow_config_id` | Critical | Y |
| 58 | Workflow | Workflow Step | [`workflow_step`](#table-workflow_step) | `workflow_step_id` | `workflow_instance_id, created_by, updated_by` | Critical | Y |
| 59 | Workflow | Approval Workflow Step Assignee | [`workflow_step_assignee`](#table-workflow_step_assignee) | `assignment_id` | `workflow_step_id, assignee_id, substitute_user_id, created_by, updated_by` | Critical | Y |
| 60 | Workflow | Approval Action History | [`approval_action`](#table-approval_action) | `approval_action_id` | `workflow_step_id, workflow_step_assignee_id, actor_id, signature_id` | Critical | Y |
| 61 | Workflow | Project Approval Route Configuration | [`project_workflow_config`](#table-project_workflow_config) | `config_id` | `project_id, project_activity_id, applied_signature_id, created_by, updated_by` | Critical | Y |
| 62 | Traceability | Common Traceability Link | [`traceability_link`](#table-traceability_link) | `traceability_link_id` | `project_id, created_by, updated_by` | Critical | Y |
| 63 | Deviation | Deviation Management | [`deviation`](#table-deviation) | `deviation_id` | `project_id, approved_by, created_by, updated_by, action_workflow_id, action_signature_id, completion_signature_id, closure_signature_id, current_action_round_id, current_completion_revision_id` | Critical | Y |
| 64 | Deviation | Deviation Action Round | [`deviation_action_round`](#table-deviation_action_round) | `action_round_id` | `deviation_id, workflow_instance_id, signature_id, created_by, updated_by` | Critical | Y |
| 65 | Deviation | Deviation Closure Report Revision | [`deviation_completion_revision`](#table-deviation_completion_revision) | `completion_revision_id` | `deviation_id, action_round_id, workflow_instance_id, signature_id, created_by, updated_by` | Critical | Y |
| 66 | Document | Deliverable Document | [`deliverable_document`](#table-deliverable_document) | `document_id` | `project_id, project_activity_id, created_by, updated_by` | Critical | Y |
| 67 | Document | Deliverable Revision | [`deliverable_revision`](#table-deliverable_revision) | `document_revision_id` | `document_id, workflow_instance_id, approved_by, file_id, created_by, updated_by, system_baseline_id` | Critical | Y |
| 68 | Document | Deliverable Section/Body | [`deliverable_section`](#table-deliverable_section) | `section_id` | `document_revision_id, created_by, updated_by` | Critical | Y |
| 69 | Report | Report Generation Job | [`report_generation`](#table-report_generation) | `report_generation_id` | `project_id, requested_by, result_file_id, created_by, updated_by, document_revision_id` | High | Y |
| 70 | AI | AI Generation Job | [`ai_generation_job`](#table-ai_generation_job) | `ai_job_id` | `project_id, requested_by, created_by, updated_by` | High | Y |
| 71 | AI | AI Generation Result | [`ai_generation_result`](#table-ai_generation_result) | `ai_result_id` | `ai_job_id, selected_by, created_by, updated_by` | High | Y |
| 72 | AI | AI Generation Result Item | [`ai_result_item`](#table-ai_result_item) | `ai_result_item_id` | `ai_result_id, created_by, updated_by` | High | Y |
| 73 | AI | AI Outbound Data Sensitive-Information Detection Result | [`ai_sensitive_detection`](#table-ai_sensitive_detection) | `detection_id` | `ai_job_id, created_by` | High | Y |
| 74 | Regulation | Regulatory Basis Document | [`regulatory_source`](#table-regulatory_source) | `regulatory_source_id` | `file_id, verified_by, created_by, updated_by` | High | Y |
| 75 | Regulation | Regulatory Clause | [`regulatory_clause`](#table-regulatory_clause) | `regulatory_clause_id` | `regulatory_source_id, created_by, updated_by` | High | Y |
| 76 | Regulation | Requirement Regulatory Basis | [`requirement_regulation`](#table-requirement_regulation) | `requirement_regulation_id` | `requirement_id, regulatory_clause_id, created_by, updated_by` | High | Y |
| 77 | Regulation | Library Regulatory Basis | [`library_item_regulation`](#table-library_item_regulation) | `library_item_regulation_id` | `library_id, regulatory_clause_id, created_by, updated_by` | High | Y |

## 4. Shortcuts by Domain

- **Organization**: [`organization`](#table-organization)
- **Security**: [`app_user`](#table-app_user), [`role`](#table-role), [`user_role`](#table-user_role), [`user_group`](#table-user_group), [`user_group_member`](#table-user_group_member), [`group_role`](#table-group_role), [`access_permission_grant`](#table-access_permission_grant), [`inventory_role_grant`](#table-inventory_role_grant), [`user_signature_credential`](#table-user_signature_credential)
- **Compliance**: [`electronic_signature`](#table-electronic_signature), [`audit_trail`](#table-audit_trail)
- **File**: [`file_asset`](#table-file_asset), [`file_scan_job`](#table-file_scan_job), [`evidence_link`](#table-evidence_link), [`file_link_request`](#table-file_link_request)
- **System**: [`system_asset`](#table-system_asset), [`system_asset_revision`](#table-system_asset_revision)
- **Library**: [`library_item`](#table-library_item)
- **Validation**: [`validation_project`](#table-validation_project), [`project_system_baseline`](#table-project_system_baseline), [`project_member`](#table-project_member), [`validation_activity`](#table-validation_activity), [`project_activity`](#table-project_activity), [`activity_dependency`](#table-activity_dependency), [`project_closure_request`](#table-project_closure_request)
- **VP**: [`vp_plan`](#table-vp_plan), [`vp_section`](#table-vp_section)
- **QIA**: [`qia_assessment`](#table-qia_assessment), [`qia_module_item`](#table-qia_module_item), [`qia_process`](#table-qia_process)
- **VA**: [`vendor_audit`](#table-vendor_audit)
- **URS**: [`requirement`](#table-requirement)
- **FDS**: [`fds_spec`](#table-fds_spec)
- **DDS**: [`dds_spec`](#table-dds_spec)
- **DQ**: [`dq_assessment`](#table-dq_assessment), [`dq_item`](#table-dq_item)
- **FRA**: [`fra_assessment`](#table-fra_assessment), [`fra_item`](#table-fra_item)
- **IQ**: [`iq_assessment`](#table-iq_assessment), [`iq_item`](#table-iq_item), [`iq_step`](#table-iq_step), [`iq_execution`](#table-iq_execution), [`iq_step_execution`](#table-iq_step_execution)
- **OQ**: [`oq_assessment`](#table-oq_assessment), [`oq_item`](#table-oq_item), [`oq_step`](#table-oq_step), [`oq_execution`](#table-oq_execution), [`oq_step_execution`](#table-oq_step_execution)
- **PQ**: [`pq_assessment`](#table-pq_assessment), [`pq_item`](#table-pq_item), [`pq_step`](#table-pq_step), [`pq_execution`](#table-pq_execution), [`pq_step_execution`](#table-pq_step_execution)
- **VSR**: [`vsr_assessment`](#table-vsr_assessment), [`vsr_item`](#table-vsr_item)
- **Workflow**: [`workflow_instance`](#table-workflow_instance), [`workflow_step`](#table-workflow_step), [`workflow_step_assignee`](#table-workflow_step_assignee), [`approval_action`](#table-approval_action), [`project_workflow_config`](#table-project_workflow_config)
- **Traceability**: [`traceability_link`](#table-traceability_link)
- **Deviation**: [`deviation`](#table-deviation), [`deviation_action_round`](#table-deviation_action_round), [`deviation_completion_revision`](#table-deviation_completion_revision)
- **Document**: [`deliverable_document`](#table-deliverable_document), [`deliverable_revision`](#table-deliverable_revision), [`deliverable_section`](#table-deliverable_section)
- **Report**: [`report_generation`](#table-report_generation)
- **AI**: [`ai_generation_job`](#table-ai_generation_job), [`ai_generation_result`](#table-ai_generation_result), [`ai_result_item`](#table-ai_result_item), [`ai_sensitive_detection`](#table-ai_sensitive_detection)
- **Regulation**: [`regulatory_source`](#table-regulatory_source), [`regulatory_clause`](#table-regulatory_clause), [`requirement_regulation`](#table-requirement_regulation), [`library_item_regulation`](#table-library_item_regulation)

---

# 5. Table and Column Details

## Organization

<a id="table-organization"></a>
### 1. Organization / Customer (`organization`)

| Item | Definition |
|---|---|
| Description | Basic information on a customer or operating organization |
| Primary Key | `organization_id` |
| Main References (FK) | - |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period. Rows are not physically deleted and are managed only by status |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1 | Organization ID | `organization_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique organization identifier | `00000000-0000-0000-0000-000000000001` |
| 2 | Organization Code | `organization_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Organization/company identification code | `ORG-SAMPLE-01` |
| 3 | Organization Name | `organization_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Company/site name | `Sample Co., Ltd.` |
| 4 | Organization Type | `organization_type` | `varchar(50)` | N | N | - | Y | `'HEADQUARTERS'` | N | N | N | Y | HEADQUARTERS=headquarters, PLANT=plant, LAB=laboratory, OVERSEAS=overseas subsidiary, SYSTEM=system-only organization, VENDOR=platform developer operations organization. SYSTEM is an internal-only organization to which the SYSTEM account belongs; there is 1 per DB, and it is excluded from organization lists and selection options on screens. Screen display names are mapped to a Korean label per code | `HEADQUARTERS` |
| 5 | Status | `status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE | `ACTIVE` |
| 6 | AI Features Enabled | `ai_enabled` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Organization-level switch that turns AI draft generation and the audit-response bot on or off together. If FALSE, the entry points of both features are hidden on screens, and the server rejects new AI job creation and bot queries. Fixed to FALSE for the SYSTEM organization | `TRUE` |
| 7 | SSO Enabled | `sso_enabled` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Login method of the organization. If TRUE, HUMAN accounts belonging to it log in through the company IdP (SSO); if FALSE, they log in with the platform's own password (LOCAL). Because SSO adoption differs by customer, it is set per organization. IdP connection information (issuer, certificate, client settings) is managed in server settings, not in this table. FALSE for the SYSTEM organization | `TRUE` |
| 8 | Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Organization description | `Corporate IT and data operations organization` |
| 9 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-01T00:00:00Z` |
| 10 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T16:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `ck_organization_1` | CHECK | `status` | CHECK (status IN ('ACTIVE','INACTIVE')) |
| `ck_organization_2` | CHECK | `organization_type` | CHECK (organization_type IN ('HEADQUARTERS','PLANT','LAB','OVERSEAS','SYSTEM','VENDOR')) |
| `ck_organization_3` | CHECK | `organization_type, ai_enabled` | CHECK (organization_type <> 'SYSTEM' OR ai_enabled = FALSE) |
| `rule_organization_4` | Business validation | `organization_type` | There is 1 organization with organization_type='SYSTEM' per DB, and only account_type=SYSTEM accounts belong to it. It cannot own projects, system assets, or user groups, and it is not shown in organization lists, selection options, or reports on screens. It is not deleted, and an inactive (INACTIVE) organization also counts toward the limit of 1 per DB (uq_organization_system_vendor). When it is needed again, a new one is not created; the same row is reactivated. |
| `rule_organization_5` | Business validation | `ai_enabled` | Only administrator levels (SYSTEM_ADMIN or ADMIN of that organization) may change ai_enabled, and the before and after values are recorded in the Audit Trail (menu_code=ACCOUNTS). |
| `rule_organization_6` | Business validation | `sso_enabled` | Only administrator levels may change sso_enabled, and the change is recorded in the Audit Trail (menu_code=ACCOUNTS). On change, the auth_provider, SSO identifiers, and password_hash of the HUMAN accounts belonging to the organization are switched in bulk for the whole organization (excluding emergency (break-glass) access accounts). Detailed procedures, such as when existing passwords are disposed of, are defined in operating procedures. |
| `ck_organization_7` | CHECK | `organization_type, sso_enabled` | CHECK (organization_type <> 'SYSTEM' OR sso_enabled = FALSE) |
| `rule_organization_8` | Business validation | `organization_type` | There is 1 organization with organization_type='VENDOR' per DB, and only the platform developer's operations staff (permission_level=SYSTEM_ADMIN) belong to it. It cannot own projects, system assets, or user groups, and it is excluded from organization lists, selection options, and statistics on customer screens. sso_enabled is set according to whether the developer's IdP is used. It is not deleted, and an inactive (INACTIVE) organization also counts toward the limit of 1 per DB (uq_organization_system_vendor). When it is needed again, a new one is not created; the same row is reactivated. |
| `uq_organization_system_vendor` | UNIQUE | `organization_type` | UNIQUE (organization_type) WHERE organization_type IN ('SYSTEM','VENDOR') |

#### Business Rules

Manages the organization to which users, groups, and systems belong.

ai_enabled is the organization-level AI usage policy. If TRUE, both AI draft generation (ai_generation_job) and the audit-response bot can be used; if FALSE, both features are blocked together. There are no separate per-feature switches. After switching to FALSE, no new AI jobs are created, but jobs already PENDING/PROCESSING at the time of the switch are allowed to run to completion.

The SYSTEM organization is an internal organization that represents the affiliation of the SYSTEM account used for batch/automated processing. It does not own business data and is excluded from customer screens and statistics.

The VENDOR organization is the organization to which the platform developer's operations staff (SYSTEM_ADMIN) belong. It is kept separate from customer organizations, and its staff connect through the developer's IdP (SSO) or emergency access (LOCAL) regardless of any customer IdP failure. It does not own business data and is excluded from customer screens and statistics.

sso_enabled determines the login method of the organization. Human accounts in an organization where it is TRUE log in only through SSO; as the only exception, emergency (break-glass) access accounts kept for IdP failures use LOCAL. All human accounts in an organization where it is FALSE log in with LOCAL. Under both methods, electronic signatures re-authenticate with the signature-only password in user_signature_credential.

[↑ Back to top](#top)

---

## Security

<a id="table-app_user"></a>
### 2. User (`app_user`)

| Item | Definition |
|---|---|
| Description | User account and basic profile information |
| Primary Key | `user_id` |
| Main References (FK) | `organization_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period. Rows are not physically deleted and are managed only by status |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 11 | User ID | `user_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique user account identifier | `00000000-0000-0000-0000-000000000001` |
| 12 | Organization ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | References organization.organization_id | `00000000-0000-0000-0000-000000000001` |
| 13 | Password Hash | `password_hash` | `varchar(255)` | N | N | - | N | - | N | N | Y | N | Hash value used to verify the password for the platform's own login (auth_provider=LOCAL). Required for auth_provider=LOCAL accounts (users of organizations not using SSO, and emergency (break-glass) access accounts of organizations using SSO); SSO accounts and the SYSTEM account keep it NULL. Not used for electronic signature re-authentication (the signature password is in user_signature_credential). The raw hash is not included in responses, general exports, or audit logs. Audit=N is maintained. | `[REDACTED_PASSWORD_HASH]` |
| 14 | Previous Password Hash | `previous_password_hash` | `varchar(255)` | N | N | - | N | - | N | N | Y | N | Hash of the immediately previous login password (password_hash). Used to reject the same value as the previous password when a LOCAL account changes or resets its password (requirement 1.3). NULL for SSO and SYSTEM accounts. Not included in responses, general exports, or audit logs; Audit=N is maintained. | `[REDACTED_PASSWORD_HASH]` |
| 15 | User Full Name | `full_name` | `varchar(100)` | N | N | - | Y | - | N | Y | Y | Y | User name | `Gildong Hong` |
| 16 | Email | `email` | `varchar(254)` | N | N | - | Y | - | Y | Y | Y | Y | Email used for account invitation and login. Stored as a value normalized by trimming leading/trailing whitespace and converting to lowercase. It is the account value shown on screens; no separate login ID is kept. | `user@example.com` |
| 17 | Authentication Method | `auth_provider` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | How this account logs in. SSO=the company IdP verifies identity, LOCAL=login with the password_hash stored by the platform. The server sets it according to the sso_enabled of the account's organization: SSO for organizations using SSO (exception: only SYSTEM_ADMIN accounts for emergency access in case of IdP failure are LOCAL), LOCAL for organizations not using SSO. The SYSTEM account does not log in, so the value is irrelevant | `SSO` |
| 18 | SSO Identifier | `sso_subject` | `varchar(255)` | N | N | - | N | - | N | Y | Y | Y | User identifier issued by the IdP (OIDC sub / SAML NameID). Because it is unique only within its issuer (sso_issuer), accounts are matched by the (sso_issuer, sso_subject) combination, and email is used only for secondary confirmation. Required for HUMAN accounts with auth_provider=SSO; NULL for LOCAL and SYSTEM accounts | `[REDACTED_SSO_SUBJECT]` |
| 19 | SSO Issuer | `sso_issuer` | `varchar(255)` | N | N | - | N | - | N | Y | N | Y | Identifier of the IdP that issued sso_subject (OIDC issuer URL / SAML entityID). Even within the same customer, different legal entities or sites may use different IdPs, so it is stored together with sso_subject to distinguish accounts. Required for HUMAN accounts with auth_provider=SSO; NULL for LOCAL and SYSTEM accounts | `https://login.example.com/tenant-sample` |
| 20 | Account Type | `account_type` | `varchar(20)` | N | N | - | Y | `'HUMAN'` | N | Y | N | Y | HUMAN=human user, SYSTEM=account that acts as the performer of batch/automated processing. There is 1 SYSTEM account per DB; it is not used for login, electronic signatures, or approval workflow assignments, and is used only as an FK value such as created_by/updated_by/requested_by in seed and batch processing | `HUMAN` |
| 21 | Permission Level | `permission_level` | `varchar(20)` | N | N | - | Y | `'USER'` | N | N | N | Y | SYSTEM_ADMIN=platform developer administrator (manages all customers, organizations, and operating settings), ADMIN=customer administrator (manages accounts, permissions, and settings of its own organization; formerly GLOBAL_ADMIN), USER=general user. The former PROJECT_ADMIN is removed because the screens have no such concept, and project-level roles are managed in project_member. Distinct from business roles (role) such as author and reviewer; administrator levels are not stored in role | `USER` |
| 22 | Department Name | `department_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Name of the department the user belongs to | `Information Strategy Team` |
| 23 | Rank/Position | `position_title` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Rank information | `Senior` |
| 24 | Account Status | `status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE \| LOCKED | `ACTIVE` |
| 25 | Last Login At | `last_login_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Timestamp of the last system access | `2026-08-26T16:00:00Z` |
| 26 | Consecutive Login Failure Count | `failed_login_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of consecutive LOCAL login password failures. Reset to 0 on success; after 5 consecutive failures, status switches to LOCKED (requirement 1.3). The lock is not released automatically over time; reset to 0 when an administrator resets the account. Not used for SSO accounts because the IdP manages failures | `0` |
| 27 | Password Change Date | `password_changed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the login password (password_hash) was last changed. Used only for LOCAL accounts; NULL for SSO and SYSTEM accounts. The signature password change time is user_signature_credential.changed_at | `2026-08-01T09:00:00Z` |
| 28 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-01T00:00:00Z` |
| 29 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T16:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_app_user_1` | Business validation | `email` | Email is stored as a value normalized by trimming leading/trailing whitespace and converting to lowercase (ck_app_user_email), and duplicates are not allowed based on the normalized value (email UQ). |
| `ck_app_user_2` | CHECK | `permission_level` | CHECK (permission_level IN ('SYSTEM_ADMIN','ADMIN','USER')) |
| `ck_app_user_3` | CHECK | `status` | CHECK (status IN ('ACTIVE','INACTIVE','LOCKED')) |
| `ck_app_user_4` | CHECK | `auth_provider` | CHECK (auth_provider IN ('SSO','LOCAL')) |
| `ck_app_user_5` | CHECK | `account_type` | CHECK (account_type IN ('HUMAN','SYSTEM')) |
| `ck_app_user_6` | CHECK | `account_type, auth_provider, sso_issuer, sso_subject, password_hash` | CHECK ((account_type='SYSTEM' AND password_hash IS NULL AND sso_issuer IS NULL AND sso_subject IS NULL) OR (account_type='HUMAN' AND auth_provider='SSO' AND sso_issuer IS NOT NULL AND sso_subject IS NOT NULL AND password_hash IS NULL) OR (account_type='HUMAN' AND auth_provider='LOCAL' AND password_hash IS NOT NULL AND sso_issuer IS NULL AND sso_subject IS NULL)) |
| `uq_app_user_7` | UNIQUE | `sso_issuer, sso_subject` | UNIQUE (sso_issuer, sso_subject) WHERE sso_subject IS NOT NULL |
| `rule_app_user_8` | Business validation | `auth_provider, organization_id, permission_level` | The auth_provider of a HUMAN account must match the sso_enabled of its organization (SSO if TRUE, LOCAL if FALSE). As an exception, in organizations with sso_enabled=TRUE, LOCAL is allowed only for emergency (break-glass) access accounts with permission_level=SYSTEM_ADMIN. |
| `rule_app_user_9` | Business validation | `account_type` | There is 1 account_type=SYSTEM account per DB, and it belongs to the organization_type=SYSTEM organization. It keeps permission_level=USER and status=ACTIVE and is excluded from login, electronic signatures, approval workflow assignments, and user_signature_credential creation. It is not deleted or deactivated. The limit of 1 per DB is enforced by uq_app_user_system_account. |
| `uq_app_user_system_account` | UNIQUE | `account_type` | UNIQUE (account_type) WHERE account_type = 'SYSTEM' |
| `rule_app_user_10` | Business validation | `permission_level` | SYSTEM_ADMIN is granted only to the platform developer's operations staff; customer administrators use ADMIN. SYSTEM_ADMIN accounts belong only to the organization_type=VENDOR organization, and all HUMAN accounts of the VENDOR organization are SYSTEM_ADMIN. Administrator levels also do not bypass the rules for electronic signature identity verification, assignment, and author/reviewer/approver separation. |
| `rule_app_user_11` | Business validation | `auth_provider` | Emergency (break-glass) access accounts (LOCAL accounts in organizations with sso_enabled=TRUE) cannot be used for electronic signatures or approval workflow assignments, and their login and usage history is recorded in the Audit Trail. The allowed number and the custody/usage procedures are defined by operating policy (SOP) (pending decision). LOCAL accounts in organizations not using SSO can sign and perform approval workflow actions as general users. |
| `rule_app_user_12` | Business validation | `password_hash, previous_password_hash, status, failed_login_count` | For LOCAL accounts, 5 consecutive login password failures switch status to LOCKED; the lock is not released automatically over time and is released only by an administrator reset. When the password is changed or reset, the same value as the immediately previous password (previous_password_hash) is not allowed (requirement 1.3). The password must be at least 10 characters long and contain at least 1 each of uppercase letters, lowercase letters, digits, and special characters, and it must be changed once 180 days have passed since password_changed_at (requirement 1.3). The length, composition, and expiry days are managed as operating settings. Consecutive failures are counted in failed_login_count; when the account is locked, ACCOUNT_LOCKED is recorded in the Audit Trail, and when the password is changed, PASSWORD_CHANGE is recorded in the Audit Trail. |
| `ck_app_user_failed_login` | CHECK | `failed_login_count` | CHECK (failed_login_count >= 0) |
| `ck_app_user_previous_password` | CHECK | `auth_provider, previous_password_hash` | CHECK (previous_password_hash IS NULL OR auth_provider = 'LOCAL') |
| `ck_app_user_email` | CHECK | `email` | CHECK (email = lower(trim(email))) |

#### Business Rules

Multiple business roles are managed in user_role, and membership in multiple groups in user_group_member. Active/inactive on screens corresponds to ACTIVE/INACTIVE. Manages account lock status, password information, and access times.

The login method is determined by the sso_enabled of the account's organization. HUMAN accounts in organizations using SSO have auth_provider=SSO, and the account is matched by comparing the (issuer, identifier) of the IdP authentication result with (sso_issuer, sso_subject). The platform does not hold login passwords for SSO accounts, and password_hash stays NULL. HUMAN accounts in organizations not using SSO have auth_provider=LOCAL and log in with email and password_hash. The identifier for screen display and invitations is the email; in SSO matching, email is used only for secondary confirmation. If there is no matching account, the password does not match, the account is INACTIVE or LOCKED, or it is the SYSTEM account, login is rejected and recorded in audit_trail as LOGIN_FAILED.

In organizations using SSO, auth_provider=LOCAL is allowed only for SYSTEM_ADMIN accounts for emergency access (break-glass), such as during an IdP failure. Emergency (break-glass) access accounts log in with password_hash and cannot be used for electronic signatures or approval workflow assignments, and all of their login and usage history is recorded in the Audit Trail. They are not used in normal operation, and their existence is reviewed periodically. LOCAL accounts in organizations not using SSO are not emergency (break-glass) access accounts and are used for regular business work and signatures.

Electronic signature re-authentication uses the signature-only password in user_signature_credential, regardless of the login method. The login password and the signature password are not set to the same value.

account_type=SYSTEM is the account that represents the performer of batch/automated processing. Only 1 exists per DB, it belongs to the SYSTEM organization, and it is used as an FK value such as created_by/updated_by/requested_by in seed and batch processing. In the Audit Trail, this account's user_id is recorded as actor_id together with actor_type=SYSTEM or BATCH.

Permission levels (SYSTEM_ADMIN=platform developer administrator, ADMIN=customer administrator, USER=general user) and per-menu/per-project access permissions are managed separately. Administrator levels are managed only in permission_level, not in the role table.

[↑ Back to top](#top)

---

<a id="table-role"></a>
### 3. Role (`role`)

| Item | Definition |
|---|---|
| Description | Master of permission roles such as author, reviewer, and approver |
| Primary Key | `role_id` |
| Main References (FK) | - |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 30 | Role ID | `role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique role/permission identifier | `00000000-0000-0000-0000-000000000001` |
| 31 | Role Code | `role_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | AUTHOR=author, REVIEWER=reviewer, APPROVER=approver, VIEWER=viewer. Administrator levels (SYSTEM_ADMIN/ADMIN) are not business roles, so they are not stored in role but managed in app_user.permission_level | `AUTHOR` |
| 32 | Role Name | `role_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Display name of the business role on the account screen. Multiple roles can be granted to 1 user. | `Author` |
| 33 | Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Description of the detailed permission scope of the role | `Permission to author and submit business items` |
| 34 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-01T00:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `ck_role_1` | CHECK | `role_code` | CHECK (role_code IN ('AUTHOR','REVIEWER','APPROVER','VIEWER')) |

#### Business Rules

Menu/project view, edit, and dispose permissions are stored in access_permission_grant, and inventory business roles in inventory_role_grant.

[↑ Back to top](#top)

---

<a id="table-user_role"></a>
### 4. User Role (`user_role`)

| Item | Definition |
|---|---|
| Description | User-role mapping information |
| Primary Key | `user_role_id` |
| Main References (FK) | `user_id, role_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 35 | Mapping ID | `user_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique user-role mapping identifier. Duplicate (user_id, role_id) is not allowed among active (is_active=TRUE) rows. Role revocation is handled with is_active=FALSE, and the row is not deleted. | `00000000-0000-0000-0000-000000000001` |
| 36 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 37 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | References role.role_id | `00000000-0000-0000-0000-000000000001` |
| 38 | Active | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the role grant is active. Changed to FALSE when the role is revoked; the row is not deleted | `TRUE` |
| 39 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-01T00:00:00Z` |
| 40 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who granted the role | `00000000-0000-0000-0000-000000000001` |
| 41 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-01T00:00:00Z` |
| 42 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified or revoked the role grant | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_user_role_1` | UNIQUE | `user_id, role_id, is_active` | UNIQUE (user_id, role_id) WHERE is_active=TRUE |

[↑ Back to top](#top)

---

<a id="table-user_group"></a>
### 5. User Group (`user_group`)

| Item | Definition |
|---|---|
| Description | Manages basic information and active status of user groups per organization |
| Primary Key | `user_group_id` |
| Main References (FK) | `organization_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 43 | User Group ID | `user_group_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique user group identifier | `00000000-0000-0000-0000-000000000001` |
| 44 | Organization ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | Organization to which the user group belongs | `00000000-0000-0000-0000-000000000001` |
| 45 | Group Code | `group_code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Code identifying the user group within the organization. Duplicate (organization_id, group_code) is not allowed among rows where deleted_at IS NULL and is_active is TRUE. | `QA_REVIEWER_GROUP` |
| 46 | Group Name | `group_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Group name displayed to users | `QA Reviewer Group` |
| 47 | Group Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Description of the purpose and permission scope of the group | `Group of QA review assignees for Validation documents` |
| 48 | In Use | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the user group is active | `TRUE` |
| 49 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | User group creation time (UTC) | `2026-09-14T09:00:00Z` |
| 50 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who created the user group | `00000000-0000-0000-0000-000000000001` |
| 51 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | User group last update time (UTC) | `2026-09-14T09:00:00Z` |
| 52 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last updated the user group | `00000000-0000-0000-0000-000000000001` |
| 53 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | User group soft-deletion time | `2026-09-15T09:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_user_group_1` | UNIQUE | `organization_id, group_code, deleted_at, is_active` | UNIQUE (organization_id, group_code) WHERE deleted_at IS NULL AND is_active=TRUE |

[↑ Back to top](#top)

---

<a id="table-user_group_member"></a>
### 6. User Group Member (`user_group_member`)

| Item | Definition |
|---|---|
| Description | Manages the N:M relationship between users and user groups, and the status and period of group membership |
| Primary Key | `user_group_member_id` |
| Main References (FK) | `user_group_id, user_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 54 | Group Member ID | `user_group_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique user group member mapping identifier | `00000000-0000-0000-0000-000000000001` |
| 55 | User Group ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | Y | - | N | Y | N | Y | User group to which the member belongs | `00000000-0000-0000-0000-000000000001` |
| 56 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User included in the group. Duplicate (user_group_id, user_id) is not allowed among rows where deleted_at IS NULL and member_status is ACTIVE. | `00000000-0000-0000-0000-000000000001` |
| 57 | Member Status | `member_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | Y | N | Y | Group member status. ACTIVE, INACTIVE, WITHDRAWN | `ACTIVE` |
| 58 | Joined At | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time the user was added to the group | `2026-09-14T09:00:00Z` |
| 59 | Left At | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the user's group membership ended. NULL if the user is currently a member | `2026-09-15T09:00:00Z` |
| 60 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Group member mapping creation time (UTC) | `2026-09-14T09:00:00Z` |
| 61 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the group member | `00000000-0000-0000-0000-000000000001` |
| 62 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Group member mapping last update time (UTC) | `2026-09-14T09:00:00Z` |
| 63 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last updated the group member mapping | `00000000-0000-0000-0000-000000000001` |
| 64 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Group member mapping soft-deletion time | `2026-09-15T09:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_user_group_member_1` | UNIQUE | `user_group_id, user_id, deleted_at, member_status` | UNIQUE (user_group_id, user_id) WHERE deleted_at IS NULL AND member_status='ACTIVE' |
| `ck_user_group_member_status` | CHECK | `member_status` | CHECK (member_status IN ('ACTIVE','INACTIVE','WITHDRAWN')) |

#### Business Rules

A user may join multiple groups. Group-based permissions are aggregated from the active groups the user belongs to.

[↑ Back to top](#top)

---

<a id="table-group_role"></a>
### 7. User Group Role (`group_role`)

| Item | Definition |
|---|---|
| Description | Grants roles to user groups and manages their global or per-project scope |
| Primary Key | `group_role_id` |
| Main References (FK) | `user_group_id, role_id, project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 65 | Group Role ID | `group_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the permission mapping between a user group and a role | `00000000-0000-0000-0000-000000000001` |
| 66 | User Group ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | Y | - | N | Y | N | Y | User group that receives the role | `00000000-0000-0000-0000-000000000001` |
| 67 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | Role granted to the group | `00000000-0000-0000-0000-000000000001` |
| 68 | Permission Scope Type | `scope_type` | `varchar(20)` | N | N | - | Y | `'GLOBAL'` | N | Y | N | Y | Scope of the group role. GLOBAL, PROJECT | `PROJECT` |
| 69 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Required when scope_type is PROJECT. NULL when GLOBAL. The correspondence between scope and NULL is validated by a CHECK constraint | `00000000-0000-0000-0000-000000000001` |
| 70 | In Use | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the group role mapping is in use. The per-scope duplicate prohibition applies to rows where deleted_at IS NULL and is_active is TRUE. GLOBAL is based on (user_group_id, role_id), and PROJECT on (user_group_id, role_id, project_id). | `TRUE` |
| 71 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Group role mapping creation time (UTC) | `2026-09-14T09:00:00Z` |
| 72 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who granted the group role | `00000000-0000-0000-0000-000000000001` |
| 73 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Group role mapping last update time (UTC) | `2026-09-14T09:00:00Z` |
| 74 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last updated the group role mapping | `00000000-0000-0000-0000-000000000001` |
| 75 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Group role mapping soft-deletion time | `2026-09-15T09:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `ck_group_role_1` | CHECK | `scope_type, project_id` | CHECK ((scope_type='GLOBAL' AND project_id IS NULL) OR (scope_type='PROJECT' AND project_id IS NOT NULL)) |
| `uq_group_role_global` | UNIQUE | `user_group_id, role_id, scope_type, is_active, deleted_at` | UNIQUE (user_group_id, role_id) WHERE scope_type='GLOBAL' AND is_active=TRUE AND deleted_at IS NULL |
| `uq_group_role_project` | UNIQUE | `user_group_id, role_id, project_id, scope_type, is_active, deleted_at` | UNIQUE (user_group_id, role_id, project_id) WHERE scope_type='PROJECT' AND is_active=TRUE AND deleted_at IS NULL |

#### Business Rules

Business role mapping for groups. The per-menu and per-project permission checkboxes on screens are managed in access_permission_grant.

[↑ Back to top](#top)

---

<a id="table-access_permission_grant"></a>
### 8. Menu/Project Access Permission (`access_permission_grant`)

| Item | Definition |
|---|---|
| Description | Screen view/edit/dispose permissions granted to an individual or a group |
| Primary Key | `grant_id` |
| Main References (FK) | `user_id, user_group_id, project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 76 | Permission ID | `grant_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 77 | Individual User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 78 | User Group ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | N | - | N | Y | N | Y | References user_group.user_group_id | `00000000-0000-0000-0000-000000000001` |
| 79 | Permission Scope | `scope_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | MENU=menu, PROJECT=project | `MENU` |
| 80 | Menu Code | `menu_code` | `varchar(40)` | N | N | - | N | - | N | N | N | Y | INVENTORY / PROJECT / LIBRARY / AUDIT_TRAIL / ACCOUNTS. A subset of the same code system as audit_trail.menu_code, used only for screen menu-level permissions. Regulatory basis management is included in the LIBRARY menu. Required for MENU scope | `LIBRARY` |
| 81 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Required for PROJECT scope. NULL for MENU scope | `00000000-0000-0000-0000-000000000001` |
| 82 | Can View | `can_view` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | View checkbox on the screen | `FALSE` |
| 83 | Can Edit | `can_edit` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Edit checkbox on the screen | `FALSE` |
| 84 | Can Dispose | `can_dispose` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Dispose checkbox on the project permission screen. FALSE for menu scope | `FALSE` |
| 85 | Active | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Changed to FALSE when the permission is revoked | `TRUE` |
| 86 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation time value | `2026-09-01T00:00:00Z` |
| 87 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 88 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Update time value | `2026-09-01T00:00:00Z` |
| 89 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `ck_access_permission_grant_1` | CHECK | `user_id, user_group_id` | CHECK ((user_id IS NOT NULL AND user_group_id IS NULL) OR (user_id IS NULL AND user_group_id IS NOT NULL)) |
| `ck_access_permission_grant_2` | CHECK | `scope_type, menu_code, project_id, can_dispose` | CHECK ((scope_type='MENU' AND menu_code IS NOT NULL AND project_id IS NULL AND can_dispose=FALSE) OR (scope_type='PROJECT' AND project_id IS NOT NULL AND menu_code IS NULL)) |
| `rule_access_permission_grant_3` | Business validation | `can_edit` | If can_edit or can_dispose is TRUE, can_view must also be TRUE. can_edit for the Audit Trail menu (menu_code=AUDIT_TRAIL) is FALSE. |
| `rule_access_permission_grant_4` | Business validation | - | Duplicate active permissions are not allowed within each combination: individual+menu, group+menu, individual+project, and group+project. |
| `uq_access_user_id_menu` | UNIQUE | `user_id, menu_code, is_active, scope_type` | UNIQUE (user_id, menu_code) WHERE is_active=TRUE AND user_id IS NOT NULL AND scope_type='MENU' |
| `uq_access_user_id_project` | UNIQUE | `user_id, project_id, is_active, scope_type` | UNIQUE (user_id, project_id) WHERE is_active=TRUE AND user_id IS NOT NULL AND scope_type='PROJECT' |
| `uq_access_user_group_id_menu` | UNIQUE | `user_group_id, menu_code, is_active, scope_type` | UNIQUE (user_group_id, menu_code) WHERE is_active=TRUE AND user_group_id IS NOT NULL AND scope_type='MENU' |
| `uq_access_user_group_id_project` | UNIQUE | `user_group_id, project_id, is_active, scope_type` | UNIQUE (user_group_id, project_id) WHERE is_active=TRUE AND user_group_id IS NOT NULL AND scope_type='PROJECT' |
| `ck_access_view_required` | CHECK | `can_edit, can_dispose, can_view` | CHECK ((NOT can_edit AND NOT can_dispose) OR can_view) |
| `ck_access_permission_grant_menu` | CHECK | `menu_code` | CHECK (menu_code IS NULL OR menu_code IN ('INVENTORY','PROJECT','LIBRARY','AUDIT_TRAIL','ACCOUNTS')) |

#### Business Rules

Effective permissions on screens are calculated by combining direct individual grants with grants from the active groups the user belongs to. Results inherited from groups are not stored again as direct per-user permissions.

Final permissions are computed by first confirming that the account is active, then combining with OR, per function, the direct individual permissions in the relevant menu/project scope and the permissions granted to active groups and active members. Anything not granted is not allowed. Edit and dispose require both the corresponding access permission and the business role, and review and approval additionally check for an assignment in the current Workflow or a valid substitute assignment. Access permissions do not automatically grant approval roles, and business roles do not automatically grant access permissions either.

The permission level determines the scope of administrative functions; global roles are computed from user_role and GLOBAL group_role, and project roles from project_member and the PROJECT group_role of the relevant project. For inventory work, inventory_role_grant applies. Administrators also do not bypass the rules for electronic signature identity verification, assignment, author/reviewer/approver separation, and locking of approved original content. Permission changes and group withdrawal/deactivation are reflected in effective permissions from the next request onward, and the server makes the final determination.

[↑ Back to top](#top)

---

<a id="table-inventory_role_grant"></a>
### 9. Inventory Business Role (`inventory_role_grant`)

| Item | Definition |
|---|---|
| Description | Inventory author / reviewer / approver / disposal authority holder selections for individuals and groups |
| Primary Key | `grant_id` |
| Main References (FK) | `user_id, user_group_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 90 | Inventory Role ID | `grant_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 91 | Individual User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 92 | User Group ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | N | - | N | Y | N | Y | References user_group.user_group_id | `00000000-0000-0000-0000-000000000001` |
| 93 | Inventory Role | `role_code` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | AUTHOR=author, REVIEWER=reviewer, APPROVER=approver, DISPOSER=disposal authority holder | `APPROVER` |
| 94 | Active | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Active flag value | `TRUE` |
| 95 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation time value | `2026-09-01T00:00:00Z` |
| 96 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 97 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Update time value | `2026-09-01T00:00:00Z` |
| 98 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `ck_inventory_role_grant_1` | CHECK | `user_id, user_group_id` | CHECK ((user_id IS NOT NULL AND user_group_id IS NULL) OR (user_id IS NULL AND user_group_id IS NOT NULL)) |
| `ck_inventory_role_grant_2` | CHECK | `role_code` | CHECK (role_code IN ('AUTHOR','REVIEWER','APPROVER','DISPOSER')) |
| `rule_inventory_role_grant_3` | Business validation | - | Duplicates are not allowed for each combination of active individual + role and active group + role. |
| `uq_inventory_user_id` | UNIQUE | `user_id, role_code, is_active` | UNIQUE (user_id, role_code) WHERE is_active=TRUE AND user_id IS NOT NULL |
| `uq_inventory_user_group_id` | UNIQUE | `user_group_id, role_code, is_active` | UNIQUE (user_group_id, role_code) WHERE is_active=TRUE AND user_group_id IS NOT NULL |

#### Business Rules

These are inventory permissions assigned on screen, separate from the account's general business roles. Individual grants and inheritance from active groups are combined.

[↑ Back to top](#top)

---

<a id="table-user_signature_credential"></a>
### 10. Electronic Signature Password (`user_signature_credential`)

| Item | Definition |
|---|---|
| Description | Manages the hash, status, expiry, and lock of the password dedicated to electronic signature re-authentication. Kept separate from the login password so that identity verification at signing is performed even in SSO environments |
| Primary Key | `credential_id` |
| Main References (FK) | `user_id, reset_requested_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period. The row is not deleted even after the account is deactivated. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 99 | Signature Credential ID | `credential_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 100 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | Y | Y | N | Y | User who owns the signature password. 1 row per user (1:1); only account_type=HUMAN accounts are allowed | `00000000-0000-0000-0000-000000000001` |
| 101 | Signature Password Hash | `signature_password_hash` | `varchar(255)` | N | N | - | N | - | N | N | Y | N | Hash of the password dedicated to electronic signature re-authentication. Separate from the login password (app_user.password_hash), and the same value is not allowed. NULL in NOT_SET status. Not included in responses, exports, Audit JSON, or signed_payload; Audit=N is maintained | `[REDACTED_PASSWORD_HASH]` |
| 102 | Previous Signature Password Hash | `previous_signature_password_hash` | `varchar(255)` | N | N | - | N | - | N | N | Y | N | Hash of the immediately previous signature password. Used to reject the same value as the previous password when changing or resetting it (requirement 1.3). NULL if it has never been changed or reset. Not included in responses, exports, Audit JSON, or signed_payload; Audit=N is maintained | `[REDACTED_PASSWORD_HASH]` |
| 103 | Credential Status | `credential_status` | `varchar(20)` | N | N | - | Y | `'NOT_SET'` | N | Y | N | Y | NOT_SET=not set (cannot sign), ACTIVE=usable, LOCKED=locked after 5 consecutive failures (released only by an administrator reset), RESET_REQUIRED=awaiting reset by the user after an administrator reset request | `ACTIVE` |
| 104 | Initial Set At | `set_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the user first set the signature password | `2026-09-01T09:00:00Z` |
| 105 | Last Changed At | `changed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the signature password was last changed. Basis for expiry calculation | `2026-09-01T09:00:00Z` |
| 106 | Expires At | `expires_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | changed_at plus the expiry period (180 days, requirement 1.3, an operating setting). Once passed, signing is not possible even if ACTIVE; recalculated after a change | `2026-12-01T09:00:00Z` |
| 107 | Consecutive Failure Count | `failed_attempt_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of consecutive signature password re-authentication failures. Reset to 0 on success; after 5 consecutive failures, switches to LOCKED (requirement 1.3). Reset to 0 when a new signature password is set after an administrator reset | `0` |
| 108 | Reset Requested By ID | `reset_requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Administrator who switched the status to RESET_REQUIRED. Administrators can only request a reset and cannot set the password value | `00000000-0000-0000-0000-000000000001` |
| 109 | Reset Requested At | `reset_requested_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the status was switched to RESET_REQUIRED | `2026-09-01T09:00:00Z` |
| 110 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 111 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 112 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 113 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_user_signature_credential_1` | UNIQUE | `user_id` | UNIQUE (user_id) |
| `ck_user_signature_credential_2` | CHECK | `credential_status` | CHECK (credential_status IN ('NOT_SET','ACTIVE','LOCKED','RESET_REQUIRED')) |
| `ck_user_signature_credential_3` | CHECK | `credential_status, signature_password_hash` | CHECK ((credential_status='NOT_SET' AND signature_password_hash IS NULL) OR (credential_status<>'NOT_SET' AND signature_password_hash IS NOT NULL)) |
| `ck_user_signature_credential_4` | CHECK | `failed_attempt_count` | CHECK (failed_attempt_count >= 0) |
| `ck_user_signature_credential_6` | CHECK | `credential_status, reset_requested_by, reset_requested_at` | CHECK (credential_status <> 'RESET_REQUIRED' OR (reset_requested_by IS NOT NULL AND reset_requested_at IS NOT NULL)) |
| `rule_user_signature_credential_7` | Business validation | `user_id` | user_id allows only account_type=HUMAN accounts. No row is created for SYSTEM accounts or emergency (break-glass) access accounts (LOCAL accounts in organizations that use SSO). A row is created for LOCAL accounts in organizations that do not use SSO. |
| `rule_user_signature_credential_8` | Business validation | `signature_password_hash, previous_signature_password_hash` | Setting and changing are performed only by the users themselves, who verify their identity with the current signature password. For initial setup (NOT_SET) and reset (RESET_REQUIRED), the current signature password cannot be used, so identity is verified by login re-authentication (SSO for SSO accounts, the login password for LOCAL accounts). Administrators can only switch the status to RESET_REQUIRED; a NOT_SET status is not switched and remains as is. Administrators cannot set or view the value. The new password cannot be the same as the immediately previous password (previous_signature_password_hash) (requirement 1.3). The new password must be at least 10 characters long and contain at least 1 each of uppercase letters, lowercase letters, digits, and special characters (requirement 1.3). The length and composition are managed as operating settings. |
| `rule_user_signature_credential_9` | Business validation | `credential_status, failed_attempt_count, expires_at` | On re-authentication failure, failed_attempt_count is incremented, and after 5 consecutive failures the status switches to LOCKED (requirement 1.3). The lock is not released automatically over time; when an administrator resets it (switch to RESET_REQUIRED), the user sets a new signature password after login re-authentication, the status returns to ACTIVE, and failed_attempt_count is reset to 0. On success, it is reset to 0. When expires_at passes, signing is refused and a change is required. The expiry period is 180 days (requirement 1.3) and is managed as an operating setting. |
| `rule_user_signature_credential_10` | Business validation | - | Setting/changing is recorded as SIGNATURE_PASSWORD_SET, administrator reset requests as SIGNATURE_PASSWORD_RESET, re-authentication failures as SIGNATURE_FAILED, and locking/unlocking as an UPDATE that changes credential_status, in the Audit Trail (menu_code=AUTH). Hashes, plaintext, and input values are not recorded. |

#### Business Rules

Electronic signature re-authentication is performed only with the signature-only password in this table. In organizations that use SSO, the platform does not hold the login password, so identity verification for electronic signatures (the two components of identification code + password under 21 CFR Part 11 §11.200(a)) is satisfied with a signature password separate from login. Organizations that do not use SSO also apply the same procedure and keep the login password and the signature password separate. The same procedure applies regardless of the login method (SSO/LOCAL).

Users whose credential_status is NOT_SET, LOCKED, or RESET_REQUIRED, or whose expires_at has passed, cannot perform submission, review, approval, disposal, or test execution confirmation that requires an electronic signature. The screen requires setting or changing the password first, before signing.

Re-authentication processing order: on a signature request, lock the row for the signer's user_id, confirm that it is ACTIVE, not expired, and not locked, and then compare the input value with the hash. On success, reset failed_attempt_count to 0 and, in the same transaction, commit the electronic_signature row (authentication_method=SIGNATURE_PASSWORD, authentication_result=SUCCESS) together with the target status transition and audit trail record. On failure, no signature row is created; only the failure count increment and the SIGNATURE_FAILED audit trail record are saved.

The plaintext password, hash, and input values are not included in electronic_signature.signed_payload, the old_values/new_values of audit_trail, or API response bodies. When the account becomes INACTIVE or LOCKED, this row is kept but signing is refused. On account reactivation, if credential_status is NOT_SET, it remains NOT_SET (initial setup required); any other status is switched to RESET_REQUIRED to require a reset. In this case, reset_requested_by and reset_requested_at record the administrator who performed the reactivation and the time.

[↑ Back to top](#top)

---

## Compliance

<a id="table-electronic_signature"></a>
### 11. Electronic Signature (`electronic_signature`)

| Item | Definition |
|---|---|
| Description | Electronic signature records for submission, review, approval, disposal, execution confirmation, approval route application, and closure requests |
| Primary Key | `signature_id` |
| Main References (FK) | `signer_id` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 114 | Electronic Signature ID | `signature_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the electronic signature record | `00000000-0000-0000-0000-000000000001` |
| 115 | Signer ID | `signer_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 116 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Actual target table name. Distinguishes business item revisions, deliverable revisions, and closure requests. Inventory is identified by the system_asset PK and revision_number | `requirement` |
| 117 | Target Record ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | PK value of the record to which the signature was applied | `00000000-0000-0000-0000-000000000001` |
| 118 | Signature Step | `signature_action` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | SUBMIT / REVIEW / APPROVE / REJECT / DISPOSE / CONFIG_APPLY / EXECUTE / ABORT. Records the signature meaning together with the target task. ABORT is an abort of IQ/OQ/PQ test execution or the cancellation of an in-progress result correction | `APPROVE` |
| 119 | Signature Purpose | `signature_meaning` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Business meaning of submission / review / approval / disposal / test execution confirmation / test abort or correction cancellation. For closure requests, PROJECT_CLOSE_NORMAL or PROJECT_CLOSE_FORCED | `Final document approval` |
| 120 | Signature Timestamp | `signed_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp at which the electronic signature was performed | `2026-08-26T16:30:00Z` |
| 121 | Target Version | `target_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Revision display version or revision number. Closure request: CLOSE-{request_version}; deviation action: ACTION-{round_number}-{revision_number}; completion report: COMPLETE-{action round}-{revision_number}; closure without re-execution: CLOSE-{action round}-{revision_number} (deviation_completion_revision). For test execution, the attempt and the result revision are identified together. | `v1.0` |
| 122 | Signed Content Hash | `content_hash` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | Lowercase 64-character SHA-256 of the entire normalized UTF-8 JSON of signed_payload, including payload_schema_version. Must be recalculable identically from the stored original content. | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |
| 123 | Re-authentication Method | `authentication_method` | `varchar(50)` | N | N | - | Y | `'SIGNATURE_PASSWORD'` | N | N | N | Y | Re-authentication method used to verify the signer's identity when performing the electronic signature. Fixed to SIGNATURE_PASSWORD (the signature-only password in user_signature_credential), regardless of the login method (SSO/LOCAL) | `SIGNATURE_PASSWORD` |
| 124 | Re-authentication Result | `authentication_result` | `varchar(20)` | N | N | - | Y | `'SUCCESS'` | N | N | N | Y | Result of re-authentication processing when performing the electronic signature. Fixed to SUCCESS because only successful signatures are stored in this table; failures are recorded only in audit_trail (SIGNATURE_FAILED) and user_signature_credential.failed_attempt_count | `SUCCESS` |
| 125 | Signer Display Name | `signer_name_snapshot` | `varchar(100)` | N | N | - | Y | - | N | N | Y | Y | User name displayed on screen at the time of signing | `Gildong Hong` |
| 126 | Signer Role | `signer_role_snapshot` | `varchar(100)` | N | N | - | Y | - | N | N | Y | Y | Role at the time of signing, such as author/reviewer/approver | `Approver` |
| 127 | Signed Original Content Format Version | `payload_schema_version` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Version identifying the included fields and normalization rules per target type and signature purpose | `SIGN-1` |
| 128 | Signed Original Content | `signed_payload` | `jsonb` | N | N | - | Y | - | N | N | Y | Y | {schema_version,target,baseline_id,body,children,source_refs,files,context} as finalized by the server. Areas that do not exist for a given target are included as an empty array or NULL. Passwords and authentication secrets are excluded. | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_electronic_signature_1` | Business validation | - | target_table_name and target_record_id must together point to the actual PK and revision of the target. This is a polymorphic reference to multiple tables, not a physical FK. |
| `rule_electronic_signature_2` | Business validation | - | The signer and the processor must be the same person, and a signature row is created only when signature password re-authentication succeeds while the signer's user_signature_credential is ACTIVE, not expired, and not locked. Failed re-authentication is not stored in this table but is recorded in audit_trail as SIGNATURE_FAILED. |
| `rule_electronic_signature_3` | Business validation | - | Content is not modified after signing. Modification of the target is handled with a new revision and a new signature. |
| `ck_signature_hash` | CHECK | `content_hash` | CHECK (content_hash ~ '^[0-9a-f]{64}$') |
| `ck_signature_auth_method` | CHECK | `authentication_method` | CHECK (authentication_method = 'SIGNATURE_PASSWORD') |
| `ck_signature_auth_result` | CHECK | `authentication_result` | CHECK (authentication_result = 'SUCCESS') |
| `rule_electronic_signature_4` | Business validation | `signer_id` | signer_id must be an active account with account_type=HUMAN and must not be an emergency (break-glass) access account (a LOCAL account in an organization that uses SSO). SYSTEM accounts and emergency access accounts cannot sign; LOCAL accounts in organizations that do not use SSO can sign. |
| `ck_signature_action` | CHECK | `signature_action` | CHECK (signature_action IN ('SUBMIT','REVIEW','APPROVE','REJECT','DISPOSE','CONFIG_APPLY','EXECUTE','ABORT')) |

#### Business Rules

Disposal, test execution confirmation, and approval route application are linked via the signature FK of the corresponding business record. General approval workflow processing uses workflow and approval_action.

Identity verification at signing is performed every time, independently of the login session; regardless of SSO login, the signature-only password from user_signature_credential is entered for re-authentication. Re-authentication and signature row creation, target status transition, and the audit trail record are committed in the same transaction.

Closure signatures are identified by the project_closure_request PK and CLOSE-{request_version}. Past signed original content and hashes are retained.

signed_payload includes the body confirmed by the signer, ordered child steps, the table, PK, version, and content hash of the source targets, the file_id, content_hash, and size of the original attachments that already exist at signing, and the project validation baseline baseline_id. PDF output files generated later from the approved original content are not retroactively added to this original attachment list. context includes the signature purpose and processing reason; in addition, for a completion report it includes the corrective and preventive actions and the completion report original content; for closure without re-execution, the closure reason; for disposal, the disposal reason and the identifier of the originally approved target; and for test abort or correction cancellation (ABORT), the abort reason, the attempt_no and record_revision of the target execution, and the step completion and result values entered up to the point of the abort. source_refs reference only targets whose exact revision original content has been preserved. Original content changed by resubmission after rejection is also kept separately in the signed_payload of each signature.

Normalization sorts object keys by character code and serializes JSON without whitespace in UTF-8. Arrays with a business order keep their order, and set-type references and file lists are sorted by identifier. Dates are standard UTC strings, UUIDs are lowercase, and NULL is kept explicitly. Number and string conversion rules preserve the definition for each payload_schema_version. After the server re-reads and verifies the target, its child data, and the file hashes, it commits the storage of the signed original content, the status transition, and the audit trail record in the same transaction.

Management fields such as approval status, current revision flag, updated_at, approval workflow progress summary, subsequent disposal status, and invalidated_at are excluded from the original business approval body. Changes to management fields are also recorded in the audit trail, and the original signed_payload and content_hash are not updated. For separate signatures such as disposal or closure, the corresponding processing reason is included in context. The final approved original content for the same target, purpose, and version is not overwritten or re-approved; when the business original content changes, a new version or a new action revision is created.

[↑ Back to top](#top)

---

<a id="table-audit_trail"></a>
### 12. Audit Trail (`audit_trail`)

| Item | Definition |
|---|---|
| Description | Audit trail records of before/after values of key data changes and of the actor |
| Primary Key | `audit_id` |
| Main References (FK) | `actor_id` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 129 | Audit Trail ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the audit trail record | `00000000-0000-0000-0000-000000000001` |
| 130 | Actor ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Required for USER actors. For SYSTEM/BATCH actors, stores the user_id of the SYSTEM account (account_type=SYSTEM); NULL is allowed only for the initial seed before the SYSTEM account is created. NULL for ANONYMOUS | `00000000-0000-0000-0000-000000000001` |
| 131 | Action Type | `action_type` | `varchar(40)` | N | N | - | Y | - | N | Y | N | Y | CREATE, UPDATE, DELETE, EXPORT, LOGIN, LOGOUT, LOGIN_FAILED, ACCOUNT_LOCKED, PASSWORD_CHANGE, SIGNATURE_PASSWORD_SET, SIGNATURE_PASSWORD_RESET, SIGNATURE_FAILED, REPORT_GENERATE, REPORT_DOWNLOAD, REPORT_CANCEL, PROJECT_CLOSE, PROJECT_FORCE_CLOSE, SUBMIT, REVIEW, APPROVE, REJECT, CANCEL, DISPOSE, PERMISSION_CHANGE, FILE_DOWNLOAD. The recording criteria for approval workflow, disposal, and permission changes are in rule_audit_trail_11, and those for file downloads are in rule_audit_trail_12 | `REPORT_GENERATE` |
| 132 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | Actual table name of the target. NULL for events without a target row | `requirement` |
| 133 | Target Record ID | `target_record_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Target PK. NULL for events without a target row, such as login and logout | `00000000-0000-0000-0000-000000000001` |
| 134 | Data Before Change | `old_values` | `jsonb` | N | N | - | N | - | N | N | Y | Y | Business values before/after the change. Excludes passwords, tokens, and password_hash | `{"status": "in progress"}` |
| 135 | Data After Change | `new_values` | `jsonb` | N | N | - | N | - | N | N | Y | Y | Business values before/after the change. Excludes passwords, tokens, and password_hash | `{"status": "approved"}` |
| 136 | Reason for Change | `reason_for_change` | `text` | N | N | - | N | - | N | N | N | Y | Reason for data change under 21 CFR Part 11 | `Corrected typos in requirement and supplemented regulation items` |
| 137 | Client IP Address | `client_ip` | `varchar(45)` | N | N | - | N | - | N | N | Y | Y | User client IP | `192.0.2.10` |
| 138 | Occurred At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Audit trail log creation time (UTC) | `2026-08-26T16:30:00Z` |
| 139 | Actor Type | `actor_type` | `varchar(20)` | N | N | - | Y | `'USER'` | N | Y | N | Y | Type of actor that performed the change. Classified as USER, SYSTEM, BATCH, or ANONYMOUS; for USER, actor_id must be stored. ANONYMOUS is for events in which the actor cannot be identified, such as a login failure where the IdP identifier or the entered email matches no account | `USER` |
| 140 | Request ID | `request_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Identifier for grouping multiple audit trail records generated by a single screen/API request as the same request | `00000000-0000-0000-0000-000000000001` |
| 141 | Session ID | `session_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Identifier of the user login session in which the change occurred. NULL allowed for system/batch processing or requests without a session | `00000000-0000-0000-0000-000000000001` |
| 142 | Target Document Version | `target_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | For revision-managed documents, stores the display version at the time of the change; NULL allowed for general tables. For project closure requests and closure records, stores the same CLOSE-n version as the electronic signature | `v1.0` |
| 143 | Target Revision Number | `target_revision_number` | `integer` | N | N | - | N | - | N | N | N | Y | For revision-managed documents, stores the numeric revision number at the time of the change; NULL allowed for general tables | `1` |
| 144 | Request Path | `request_uri` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | Screen or API request path that triggered the change. NULL allowed when there is no path, such as system/batch processing | `/api/fds/approve` |
| 145 | Client Information | `user_agent` | `text` | N | N | - | N | - | N | N | Y | Y | Browser, operating system, or client application information used for the change request | `Sample-Client/1.0` |
| 146 | Menu Code | `menu_code` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | INVENTORY / PROJECT / LIBRARY / ACCOUNTS / AUDIT_TRAIL / VP / VA / QIA / URS / FDS_GROUP / FRA / DQ / IQ / OQ / PQ / VSR / DASHBOARD / AUTH / SYSTEM_PROCESS. Same code system as menu permissions (access_permission_grant.menu_code); the code recorded for each table follows the area mapping in the business rules. AUDIT_TRAIL is the audit trail record view/export screen. SYSTEM_PROCESS is used only for records by non-human actors (actor_type=SYSTEM/BATCH) and covers the initial seed, scheduled batches, the file scan Worker, file link request processing, and system maintenance. The target business area is identified by target_table_name | `URS` |
| 147 | Actor Role | `actor_role_snapshot` | `varchar(100)` | N | N | - | N | - | N | N | Y | Y | For the role filter on the audit screen and display of the role at the time | `Author` |
| 148 | Actor Display Name | `actor_display_snapshot` | `varchar(100)` | N | N | - | N | - | N | N | Y | Y | User name at the time, displayed on the audit screen | `Gildong Hong` |
| 149 | Changed Field | `field_path` | `text` | N | N | - | N | - | N | N | N | Y | Column/item path to display on the per-field change screen. NULL for whole-record events | `requirement_text` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_audit_trail_1` | Business validation | - | The target table and ID are either both present or both NULL. |
| `rule_audit_trail_2` | Business validation | `actor_id, actor_type` | actor_id is required for USER actors. SYSTEM/BATCH actors store the user_id of the SYSTEM account; NULL is allowed only for the initial seed before the SYSTEM account is created. ANONYMOUS has actor_id NULL and is used only for events in which the actor cannot be identified, such as LOGIN_FAILED. |
| `rule_audit_trail_3` | Business validation | - | Audit records are append-only and are not modified or deleted on screen. |
| `ck_audit_trail_4` | CHECK | `action_type` | CHECK (action_type IN ('CREATE','UPDATE','DELETE','EXPORT','LOGIN','LOGOUT','LOGIN_FAILED','ACCOUNT_LOCKED','PASSWORD_CHANGE','SIGNATURE_PASSWORD_SET','SIGNATURE_PASSWORD_RESET','SIGNATURE_FAILED','REPORT_GENERATE','REPORT_DOWNLOAD','REPORT_CANCEL','PROJECT_CLOSE','PROJECT_FORCE_CLOSE','SUBMIT','REVIEW','APPROVE','REJECT','CANCEL','DISPOSE','PERMISSION_CHANGE','FILE_DOWNLOAD')) |
| `ck_audit_trail_5` | CHECK | `actor_type` | CHECK (actor_type IN ('USER','SYSTEM','BATCH','ANONYMOUS')) |
| `ck_audit_trail_6` | CHECK | `menu_code` | CHECK (menu_code IN ('INVENTORY','PROJECT','LIBRARY','ACCOUNTS','AUDIT_TRAIL','VP','VA','QIA','URS','FDS_GROUP','FRA','DQ','IQ','OQ','PQ','VSR','DASHBOARD','AUTH','SYSTEM_PROCESS')) |
| `ck_audit_trail_7` | CHECK | `actor_type, actor_id` | CHECK ((actor_type='USER' AND actor_id IS NOT NULL) OR (actor_type='ANONYMOUS' AND actor_id IS NULL) OR actor_type IN ('SYSTEM','BATCH')) |
| `rule_audit_trail_8` | Business validation | `action_type, menu_code` | LOGIN/LOGOUT/LOGIN_FAILED/ACCOUNT_LOCKED/PASSWORD_CHANGE/SIGNATURE_PASSWORD_SET/SIGNATURE_PASSWORD_RESET/SIGNATURE_FAILED are recorded with menu_code=AUTH. However, automatic processing performed by SYSTEM/BATCH actors is recorded as SYSTEM_PROCESS (ck_audit_trail_10). LOGIN_FAILED stores the attempted identifier (sso_issuer/sso_subject or email) and the rejection reason code in new_values, and does not store the plaintext password or hash. |
| `rule_audit_trail_9` | Business validation | `menu_code` | The server assigns menu_code according to the area mapping in the business rules for USER/ANONYMOUS actions and as SYSTEM_PROCESS for SYSTEM/BATCH actions; it is not determined by request values. When adding a table that is not in the mapping, define its mapping as well. |
| `ck_audit_trail_10` | CHECK | `actor_type, menu_code` | CHECK ((actor_type IN ('SYSTEM','BATCH')) = (menu_code = 'SYSTEM_PROCESS')) |
| `rule_audit_trail_11` | Business validation | `action_type, target_table_name, target_record_id` | When an approval workflow action (approval_action) occurs, the action_type of the same name (SUBMIT/REVIEW/APPROVE/REJECT/CANCEL; recall is CANCEL) is recorded with the business row subject to approval as the target. When disposal is completed (transition to lifecycle_status=DISPOSED or is_disposed=TRUE), DISPOSE is recorded on the target row. Grants, changes, and revocations of roles and permissions (user_role, group_role, user_group_member, access_permission_grant, inventory_role_grant) are recorded as PERMISSION_CHANGE instead of CREATE/UPDATE. |
| `rule_audit_trail_12` | Business validation | `action_type, target_table_name, target_record_id, menu_code` | When a user downloads a stored file (file_asset, such as attachments, evidence, FDS/DDS files, VA attachments, or regulation originals), FILE_DOWNLOAD is recorded with target_table_name=file_asset and target_record_id=file_id. menu_code follows the area mapping (for files, the code of the linked business area). Downloads of report generation results (report_generation) are recorded as REPORT_DOWNLOAD and data exports as EXPORT; previews are not recorded. |

#### Business Rules

LOGIN_FAILED is an event in which the platform rejected a login. For SSO, it is the case where IdP authentication passed but the platform rejected the login; for LOCAL, it is a password mismatch or a rejection by the platform. If no account matches, it is recorded with actor_type=ANONYMOUS and actor_id NULL; if the account exists but there is a password mismatch or the account is INACTIVE, LOCKED, or a SYSTEM account, it is recorded with actor_type=USER and the corresponding actor_id. ACCOUNT_LOCKED is recorded when app_user.status changes to LOCKED, and PASSWORD_CHANGE is recorded when the login password (password_hash) of a LOCAL account is changed.

menu_code=SYSTEM_PROCESS is used for all records left by non-human actors (actor_type=SYSTEM/BATCH). This covers the initial data seed, scheduled batches, scan status transitions by the file scan Worker, automatic linking based on file link requests (file_link_request), and automatic retries. In this case, the business area is identified by target_table_name/target_record_id, and the original screen the user started from is traced through the user audit record for the same target (e.g., the upload completion record). SYSTEM_PROCESS is not used for changes performed by users on screen (USER) or for ANONYMOUS events (ck_audit_trail_10).

Area mapping (for USER/ANONYMOUS actions; SYSTEM/BATCH actions use SYSTEM_PROCESS): accounts, groups, and permissions (app_user, role, user_role, user_group, user_group_member, group_role, access_permission_grant, inventory_role_grant) and organization are recorded as ACCOUNTS; user_signature_credential and login/signature authentication events as AUTH; system_asset/system_asset_revision as INVENTORY; library_item/regulatory_source/regulatory_clause/library_item_regulation as LIBRARY; and validation_project/project_system_baseline/project_member/project_activity/project_closure_request/project_workflow_config and the activity master (validation_activity/activity_dependency) as PROJECT. Business items and documents use the corresponding activity code (VP/VA/QIA/URS/FDS_GROUP/FRA/DQ/IQ/OQ/PQ/VSR); requirement_regulation is URS, and fds_spec/dds_spec are FDS_GROUP. Approval workflow (workflow_instance/workflow_step/workflow_step_assignee/approval_action) follows the area code of the business subject to approval (INVENTORY for inventory, PROJECT for project closure requests). Deviations (deviation/deviation_action_round/deviation_completion_revision) use the IQ/OQ/PQ of the original execution; evidence (evidence_link) and files (file_asset/file_link_request) use the code of the linked business target (LIBRARY for uploads outside a project); upload completion records (file_scan_job), made before a link target is set, use PROJECT if file_scan_job.project_id is present and LIBRARY if it is not (uploads outside a project, such as regulation originals); and traceability_link uses the code of the screen where the link was saved (DASHBOARD for the RTM screen). deliverable_document/deliverable_revision/deliverable_section use the corresponding activity code; report_generation uses the activity code of the target deliverable if DELIVERABLE, or AUDIT_TRAIL if AUDIT_TRAIL; and AI jobs, results, and detections (ai_generation_job/ai_generation_result/ai_result_item/ai_sensitive_detection) use the activity code of the business for which content is generated. electronic_signature follows the area code of the signature target (target_table_name) (e.g., PROJECT for approval route application, INVENTORY for inventory, and the IQ/OQ/PQ of the original execution for deviation completion/closure). Audit trail record view/export is AUDIT_TRAIL.

Events related to the electronic signature password are recorded with user_signature_credential as the target table and the corresponding credential_id as the target record, and the hash and plaintext are not included in old_values/new_values.

[↑ Back to top](#top)

---

## File

<a id="table-file_asset"></a>
### 13. File Asset (`file_asset`)

| Item | Definition |
|---|---|
| Description | Manages storage metadata for user attachments, evidence files, generated reports, and data export files |
| Primary Key | `file_id` |
| Main References (FK) | `uploader_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period according to the retention policy of the file type and the linked target |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 150 | File ID | `file_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the attachment metadata | `00000000-0000-0000-0000-000000000001` |
| 151 | Uploader ID | `uploader_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Stores the uploader for user-uploaded files, the user who requested generation for server-generated files, and the user_id of the SYSTEM account for files generated by scheduled batches | `00000000-0000-0000-0000-000000000001` |
| 152 | Original File Name | `original_file_name` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | File name at the time of upload | `sample_document.pdf` |
| 153 | Stored File Path | `stored_file_path` | `text` | N | N | - | Y | - | N | N | N | Y | Storage path/S3 key | `documents/2026/08/sample_001.pdf` |
| 154 | File Category | `file_category` | `varchar(30)` | N | N | - | Y | `'ATTACHMENT'` | N | Y | N | Y | Business category of the file. ATTACHMENT, EVIDENCE, REPORT, EXPORT | `REPORT` |
| 155 | File Size | `file_size_bytes` | `bigint` | N | N | - | Y | `0` | N | N | N | Y | File size (bytes) | `1048576` |
| 156 | MIME Type | `mime_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Attachment file format. NULL allowed if unidentified | `application/pdf` |
| 157 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T16:30:00Z` |
| 158 | File Content Hash | `content_hash` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | Lowercase 64-character SHA-256 of the original bytes of the stored file. Calculated and verified by the server. | - |
| 159 | Storage Object Version | `storage_version_id` | `varchar(255)` | N | N | - | N | - | N | N | N | Y | Exact object version ID in versioned storage. NULL if versioning is not available, in which case overwriting stored_file_path is prohibited. | - |
| 160 | File Origin | `file_origin` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | UPLOAD / SYSTEM_GENERATED. Assigned by the server based on the actual generation path; user uploads must pass the scan and be linked to the scan job. Distinct from file_category | `UPLOAD` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `ck_file_hash` | CHECK | `content_hash` | CHECK (content_hash ~ '^[0-9a-f]{64}$') |
| `ck_file_size` | CHECK | `file_size_bytes` | CHECK (file_size_bytes >= 0) |
| `ck_file_origin` | CHECK | `file_origin` | CHECK (file_origin IN ('UPLOAD','SYSTEM_GENERATED')) |
| `rule_file_asset_scan` | Business validation | `file_origin, file_id` | UPLOAD files are registered by linking to the result_file_id of a file_scan_job completed with no threats found, and the hash, size, uploader, file name, and file category are verified to match. |
| `rule_file_asset_origin` | Business validation | `file_origin` | SYSTEM_GENERATED is allowed only when a server-internal generation path is confirmed; skipping the scan is not decided based on the origin or file category supplied by user input. |
| `ck_file_category` | CHECK | `file_category` | CHECK (file_category IN ('ATTACHMENT','EVIDENCE','REPORT','EXPORT')) |

#### Business Rules

VA/FDS/DDS attachments and test evidence are linked by file ID. File replacement history is retained in the revision/execution attempt of the linked target.

Attachments as of approval are not overwritten with other files.

A usable file_asset is created only when file storage and server hash verification are complete. User uploads are recorded as file_origin=UPLOAD, and in addition the latest attempt of file_scan_job must be COMPLETED and NO_THREATS_FOUND, and the bytes of the scanned original and the stored file must be identical. File asset creation and linking of the scan job's result_file_id are committed in the same transaction. Files that are pending scan, being scanned, failed, unscannable, or with threats found are not registered in this table as usable files.

file_origin is assigned by the server after confirming the actual file generation path. PDF and export files generated by trusted server-internal processing are recorded as SYSTEM_GENERATED and can be registered without a user upload scan job. Externally uploaded files used for generation must pass the scan first. User-uploaded files are UPLOAD and subject to scanning even if file_category is REPORT or EXPORT.

Uploads for which only metadata exists or whose scan/registration is not complete are not used for business attachments, download/preview, document parsing/AI input, or evidence/approval/report completion. File references for VA/FDS/DDS, original regulation content, evidence, and documents link only file_ids that meet these registration conditions.

Replacement is handled with a new file_id and a new storage object. During the retention period, the original bytes and the path, version, and hash are kept, and overwriting or deletion of the object is blocked. If a storage version exists, that version is specified when retrieving. Different attachments with the same content are allowed, so content_hash is not UNIQUE. File downloads check view permission on the linked business target, and being the uploader alone does not grant access permission to other business data.

[↑ Back to top](#top)

---

<a id="table-file_scan_job"></a>
### 14. File Malware Scan Job (`file_scan_job`)

| Item | Definition |
|---|---|
| Description | Quarantine storage information for user-uploaded files, malware scan attempts and results, and the link to file asset registration after passing the scan |
| Primary Key | `scan_job_id` |
| Main References (FK) | `previous_scan_job_id, project_id, uploaded_by, requested_by, result_file_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Scan attempts, judgments, and file registration history are retained for the auditable period. Quarantined originals are managed under a quarantine file retention/deletion policy separate from the scan results |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 161 | Scan Job ID | `scan_job_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of one scan attempt. Basis for preventing duplicate processing of queue delivery and result receipt | `00000000-0000-0000-0000-000000000001` |
| 162 | Upload ID | `upload_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Logical identifier of one upload, issued by the server. Retry attempts for the same original keep the same value; not an FK to a separate table | `00000000-0000-0000-0000-000000000001` |
| 163 | Scan Attempt | `attempt_no` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | 1 for the initial scan. Retries and rescans are added as the next attempt of the same upload_id; not incremented merely by receiving a duplicate message | `1` |
| 164 | Previous Scan Job ID | `previous_scan_job_id` | `uuid` | N | Y | `file_scan_job.scan_job_id` | N | - | Y | Y | N | Y | Immediately preceding attempt for a retry or rescan. NULL for the first attempt; multiple subsequent attempts do not branch from the preceding job | `00000000-0000-0000-0000-000000000001` |
| 165 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | The project, if it is a project attachment. NULL is allowed for uploads outside a project, such as original regulation content, in which case separate menu/data access permissions are verified | `00000000-0000-0000-0000-000000000001` |
| 166 | Uploader ID | `uploaded_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who performed the original upload. The initial uploader is kept even on automatic retries | `00000000-0000-0000-0000-000000000001` |
| 167 | Original File Name | `original_file_name` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | File name at the time of upload. The same value is used when registering the file asset | `design_document.pdf` |
| 168 | Quarantine Storage Name | `storage_bucket` | `varchar(255)` | N | N | - | Y | - | N | Y | N | Y | Identifier of the bucket or storage where the original to be scanned is stored. Connection secrets and signed URLs are not stored | `upload-quarantine-example` |
| 169 | Quarantine File Path | `stored_file_path` | `text` | N | N | - | Y | - | N | Y | N | Y | Quarantine object path unique to the upload. Not exposed as a business download path | `uploads/sample-upload/design_document.pdf` |
| 170 | Quarantine Object Version | `storage_version_id` | `varchar(255)` | N | N | - | N | - | N | Y | N | Y | Exact object version that was scanned. NULL for storage without version support, in which case overwriting the original path is prohibited | - |
| 171 | Original Content Hash | `content_hash` | `varchar(64)` | N | N | - | N | - | N | N | N | Y | Lowercase 64-character SHA-256 of the original bytes as confirmed by the server or trusted scan processing. NULL before confirmation; required upon completion with no threats found | - |
| 172 | File Size | `file_size_bytes` | `bigint` | N | N | - | Y | - | N | N | N | Y | Actual byte size of the stored original | `1048576` |
| 173 | MIME Type | `mime_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Confirmed file format of the original. The file type is not trusted based on user input alone. application/octet-stream if the file was scanned but its format could not be determined; NULL if not yet checked | `application/pdf` |
| 174 | File Category | `file_category` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | ATTACHMENT / EVIDENCE / REPORT / EXPORT. Business purpose category; not used as grounds for skipping the scan | `ATTACHMENT` |
| 175 | Job Status | `job_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | PENDING / PROCESSING / COMPLETED / FAILED / CANCELLED. COMPLETED means the scan has finished; whether use is allowed is determined by scan_result | `PENDING` |
| 176 | Scan Result | `scan_result` | `varchar(30)` | N | N | - | N | - | N | Y | N | Y | NO_THREATS_FOUND / THREATS_FOUND / UNSCANNABLE. Distinguishes no threats found, threats found, and unscannable; NULL before completion or on technical failure or cancellation | `NO_THREATS_FOUND` |
| 177 | Scan Service/Engine Name | `scanner_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Identifier of the scan service or engine designated for the attempt. Not tied to a specific product | `MALWARE_SCANNER` |
| 178 | Scan Engine Version | `scanner_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Engine version actually used. NULL is allowed if the scan service does not provide it | - |
| 179 | Detection Signature Version | `signature_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Version of the malware detection information used for the scan. NULL is allowed if not provided | - |
| 180 | Scan Service Request ID | `scanner_request_id` | `varchar(255)` | N | N | - | N | - | N | Y | N | Y | Execution/request identifier issued by the scan service. Used to match results against the job and the original object | - |
| 181 | Scan Details | `scan_details` | `jsonb` | N | N | - | N | - | N | N | Y | Y | Verifiable grounds for the scan, such as provider_result (the clamd raw response with the leading stream: label removed, e.g. OK, <malware name> FOUND, ERROR; the platform judgment is in scan_result), reason_code (detailed reason for UNSCANNABLE: FORMAT_MISMATCH, ENCRYPTED, UNSUPPORTED_EXTENSION, SOURCE_MISMATCH, ENGINE_UNSCANNABLE, SIZE_LIMIT_EXCEEDED; carried over as the block reason in file_link_request.failure_code), format_detail (description of the format mismatch), engine_reason_code (the original reason returned by the scan engine), threat_names, and report_reference. Excludes passwords, tokens, signed URLs, and file contents | `{"provider_result":"OK","threat_names":[]}` |
| 182 | Scan Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Party that requested the scan attempt. Stores the uploader (uploaded_by) for the initial attempt automatically registered right after upload, the requesting user for manual rescans, and the user_id of the SYSTEM account for automatic retries | `00000000-0000-0000-0000-000000000001` |
| 183 | Scan Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time the scan attempt was registered | `2026-09-18T01:00:00Z` |
| 184 | Scan Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the scan for the attempt started. May be NULL on failure or cancellation before execution | `2026-09-18T01:00:05Z` |
| 185 | Scan Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time completion, failure, or cancellation was finalized. Distinct from the file asset registration time | `2026-09-18T01:00:20Z` |
| 186 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Last change time of the job status, scan result, or result file link | `2026-09-18T01:00:21Z` |
| 187 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Technical error code for FAILED. Values in use (a reference list, not restricted by a CHECK): SOURCE_NOT_FOUND (quarantined source not found), SOURCE_READ_FAILED (failed to read the source), SCANNER_UNAVAILABLE (could not connect to the scan engine), SCAN_TIMEOUT (scan response timed out), SCANNER_ERROR (the scan engine responded with an error), WORKER_IO_FAILED (Worker temporary file handling failed), PROCESSING_STALLED (an attempt that stayed in processing beyond the time limit (processing policy), recorded as failed by the check job). Retried automatically; if all attempts up to the maximum attempt count (processing policy) fail, the link request is blocked with file_link_request.failure_code=SCAN_FAILED. Threats found is not a technical error and is recorded in scan_result | `SCAN_TIMEOUT` |
| 188 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | Y | Y | Detailed reason for the technical failure. Does not include secrets, signed URLs, or file contents | `Scan response timeout` |
| 189 | Registered File ID | `result_file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | Y | Y | N | Y | File asset registered after no threats were found and the identity with the original was confirmed. NULL before registration; only one attempt is linked per upload | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_file_scan_attempt` | UNIQUE | `upload_id, attempt_no` | UNIQUE (upload_id, attempt_no) |
| `uq_file_scan_active` | UNIQUE | `upload_id` | UNIQUE (upload_id) WHERE job_status IN ('PENDING','PROCESSING') |
| `uq_file_scan_published` | UNIQUE | `upload_id` | UNIQUE (upload_id) WHERE result_file_id IS NOT NULL |
| `uq_file_scan_source` | UNIQUE | `storage_bucket, stored_file_path, storage_version_id` | UNIQUE (storage_bucket, stored_file_path, COALESCE(storage_version_id, '')) WHERE attempt_no=1 |
| `ck_file_scan_attempt` | CHECK | `attempt_no, previous_scan_job_id, scan_job_id` | CHECK ((attempt_no=1 AND previous_scan_job_id IS NULL) OR (attempt_no>1 AND previous_scan_job_id IS NOT NULL AND previous_scan_job_id<>scan_job_id)) |
| `ck_file_scan_status` | CHECK | `job_status` | CHECK (job_status IN ('PENDING','PROCESSING','COMPLETED','FAILED','CANCELLED')) |
| `ck_file_scan_result` | CHECK | `scan_result` | CHECK (scan_result IS NULL OR scan_result IN ('NO_THREATS_FOUND','THREATS_FOUND','UNSCANNABLE')) |
| `ck_file_scan_result_state` | CHECK | `job_status, scan_result, completed_at` | CHECK ((job_status IN ('PENDING','PROCESSING') AND scan_result IS NULL AND completed_at IS NULL) OR (job_status='COMPLETED' AND scan_result IS NOT NULL AND completed_at IS NOT NULL) OR (job_status IN ('FAILED','CANCELLED') AND scan_result IS NULL AND completed_at IS NOT NULL)) |
| `ck_file_scan_started` | CHECK | `job_status, started_at` | CHECK (job_status NOT IN ('PROCESSING','COMPLETED') OR started_at IS NOT NULL) |
| `ck_file_scan_time` | CHECK | `requested_at, started_at, completed_at` | CHECK ((started_at IS NULL OR started_at>=requested_at) AND (completed_at IS NULL OR completed_at>=requested_at) AND (started_at IS NULL OR completed_at IS NULL OR completed_at>=started_at)) |
| `ck_file_scan_size` | CHECK | `file_size_bytes` | CHECK (file_size_bytes>=0) |
| `ck_file_scan_hash` | CHECK | `content_hash` | CHECK (content_hash IS NULL OR content_hash ~ '^[0-9a-f]{64}$') |
| `ck_file_scan_clean_hash` | CHECK | `scan_result, content_hash` | CHECK (scan_result IS NULL OR scan_result<>'NO_THREATS_FOUND' OR content_hash IS NOT NULL) |
| `ck_file_scan_version` | CHECK | `storage_version_id` | CHECK (storage_version_id IS NULL OR length(storage_version_id)>0) |
| `ck_file_scan_category` | CHECK | `file_category` | CHECK (file_category IN ('ATTACHMENT','EVIDENCE','REPORT','EXPORT')) |
| `ck_file_scan_failed` | CHECK | `job_status, error_code` | CHECK (job_status<>'FAILED' OR error_code IS NOT NULL) |
| `ck_file_scan_publish` | CHECK | `result_file_id, job_status, scan_result, content_hash` | CHECK (result_file_id IS NULL OR (job_status='COMPLETED' AND scan_result IS NOT NULL AND scan_result='NO_THREATS_FOUND' AND content_hash IS NOT NULL)) |
| `rule_file_scan_chain` | Business validation | `upload_id, attempt_no, previous_scan_job_id` | The previous attempt must be a finished row of the same upload_id with attempt_no-1. The storage, path, version, size, uploader, project, and file category of the same upload are kept, and a confirmed hash is not changed. |
| `rule_file_scan_registration` | Business validation | `result_file_id, content_hash` | Registration is performed only when the latest attempt of the upload has no threats found and has not yet been registered. The file_asset must have file_origin=UPLOAD, and its uploader, file name, category, size, and hash must match the scanned original. |
| `rule_file_scan_access` | Business validation | `project_id, uploaded_by, requested_by` | Verifies the linkage of the DB, storage, user, and project belonging to the authenticated customer, and the file registration permission for the relevant business task. The customer DB or access permission is not determined solely by the organization ID or storage path passed in the request. |

#### Business Rules

**Scan targets and quarantine.** For user-uploaded files, the initial scan attempt is registered in this table after storage completes. The file type, size, and upload permission are checked, and read access to the quarantined original is allowed only to the scan processing party. Before passing the scan and being registered as a file asset, the file cannot be used for download/preview by general users, business attachments, document parsing/AI input, or approval/evidence/report completion. The scan is not skipped even if the file_category of a user-uploaded file is REPORT or EXPORT.

**Jobs and judgment.** One row is one scan attempt; it transitions from PENDING to PROCESSING and then ends as COMPLETED or FAILED. Failures and cancellations before start can also be recorded. Only NO_THREATS_FOUND under COMPLETED is a candidate for file registration; THREATS_FOUND and UNSCANNABLE remain blocked. If the entire original cannot be scanned due to encryption, an unsupported format, scan scope limits, or the like, it is recorded as UNSCANNABLE and the reason is left in scan_details. Access errors, timeouts, and scan service outages are recorded as FAILED. Unknown service responses or missing results are not converted to no threats found.

**Retries and concurrency.** Requests and results for the queue and the scan service use scan_job_id and verified customer connection information, and the original storage, path, and object version are cross-checked. Status transitions are updated conditionally on the current status, and duplicate deliveries or duplicate results do not cause repeated scans or duplicate file asset registration. The status, original, judgment, and scan grounds of a finished attempt are not overwritten. When re-running after a technical failure, the next attempt of the same upload_id is created. A rescan after unscannable or threats found records an authorized request and a reason; if the file content changes, it is registered with a new upload_id. A delayed response from a previous attempt does not revert a failure or cancellation to success, nor does it replace the judgment of the latest attempt.

**Job delivery and recovery.** Pending jobs registered in the DB must be redeliverable even after a queue send failure. In-progress jobs with no result within the time limit are recovered by confirming the actual scan execution and then finalizing the failure and registering the next attempt. The maximum number of attempts and the operational review procedure for failed jobs are managed by the processing policy, and unlimited retries are not allowed. If the service's internal retries are not provided as individual attempts, only the verifiable request identifier and the final result are recorded, and no execution history is created for executions that were not received.

**Scanned original and registered file.** A scan result is valid only for the specified object version or for an original protected from overwriting. When copying to business storage after completion with no threats found, confirm that the hash and size of the final stored object match the scanned original, and record the final path and version in file_asset. A file converted or edited after the scan is not registered as the same scanned original. The location and version of the initial scan target are kept in this table.

**File registration consistency.** file_asset creation, linking of this attempt's result_file_id, and the audit trail record are committed in the same DB transaction. Because external storage operations are separate from the DB transaction, the file is not exposed as usable before final storage is confirmed. Lock the first attempt of the same upload or apply equivalent concurrency control so that the creation of the next attempt and file registration are not committed at the same time. Even with no threats found, if file registration has not finished, result_file_id is NULL and use is not allowed. On a storage/registration failure, the identity with the original can be re-verified to resume registration of the same attempt; in this case, the finished scan result is not changed. For an upload whose registration is complete, no additional scan attempts are created in this table.

**Permissions and result trust.** The Worker or the result-receiving process selects the relevant DB and storage using customer connection information managed by the server. The sender, execution identification, and original object of a scan result are verified, and the judgment is not finalized based on user input. Passing the scan does not mean file access permission or completion of business approval; existing business permissions are checked again at actual linking or download.

**History and retention.** Scan requests, status transitions, final judgments, and file registrations are recorded in the Audit Trail. Scan results linked to a file subject to retention are kept for the auditable period of that file. Whether and when to delete unregistered, malicious, or unscannable originals is managed by the quarantine file policy, and even if an original is cleaned up, the scan result and processing reason are retained. The long-term lock policy for approval evidence is not applied wholesale to quarantined originals.

[↑ Back to top](#top)

---

<a id="table-evidence_link"></a>
### 15. Evidence File Link (`evidence_link`)

| Item | Definition |
|---|---|
| Description | Manages N:M links between documents/test items and evidence files |
| Primary Key | `evidence_link_id` |
| Main References (FK) | `project_id, file_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 190 | Evidence Link ID | `evidence_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of a link between a document/test item and an evidence file. Duplicates of (project_id, file_id, target_entity_type, target_entity_id, evidence_type) are not allowed among rows where deleted_at IS NULL. | `00000000-0000-0000-0000-000000000001` |
| 191 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | ID of the project to which the evidence link belongs | `00000000-0000-0000-0000-000000000001` |
| 192 | File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | Y | - | N | Y | N | Y | ID of the linked evidence file | `00000000-0000-0000-0000-000000000001` |
| 193 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT / FDS_SPEC / DDS_SPEC / VP_PLAN / QIA_MODULE_ITEM / VENDOR_AUDIT / DQ_ITEM / FRA_ITEM / IQ_ITEM / OQ_ITEM / PQ_ITEM / IQ_EXECUTION / OQ_EXECUTION / PQ_EXECUTION / IQ_STEP_EXECUTION / OQ_STEP_EXECUTION / PQ_STEP_EXECUTION / VSR_ASSESSMENT / DEVIATION / DELIVERABLE_REVISION | `IQ_STEP_EXECUTION` |
| 194 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | PK value of the table corresponding to target_entity_type. No physical FK is set because this is a polymorphic reference | `00000000-0000-0000-0000-000000000001` |
| 195 | Evidence Type | `evidence_type` | `varchar(50)` | N | N | - | Y | `'TEST_RESULT'` | N | Y | N | Y | TEST_RESULT, SCREENSHOT, LOG, REPORT, APPROVAL_DOCUMENT | `TEST_RESULT` |
| 196 | Evidence Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Content of the evidence file and purpose of the link | `Screen capture of IQ execution results` |
| 197 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Evidence link creation time (UTC) | `2026-09-01T10:00:00Z` |
| 198 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who created the evidence link | `00000000-0000-0000-0000-000000000001` |
| 199 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Evidence link last update time (UTC) | `2026-09-01T10:00:00Z` |
| 200 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who last updated the evidence link | `00000000-0000-0000-0000-000000000001` |
| 201 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time of the evidence link | `2026-09-02T10:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_evidence_link_1` | UNIQUE | `project_id, file_id, target_entity_type, target_entity_id, evidence_type, deleted_at` | UNIQUE (project_id,file_id,target_entity_type,target_entity_id,evidence_type) WHERE deleted_at IS NULL |
| `rule_evidence_link_2` | Business validation | - | Checks the actual PK, same project, and exact revision for each target type. The polymorphic reference is not shown as a physical FK. |
| `rule_evidence_link_3` | Business validation | - | Per-step attachments are linked to each IQ/OQ/PQ_STEP_EXECUTION, and attachments for the whole attempt to each EXECUTION. Attachments of approved targets are not overwritten. |
| `rule_evidence_link_4` | Business validation | `file_id` | Only files that meet the file_asset registration conditions are linked. For UPLOAD, the no-threats-found scan and the identity of the registered file with the original are confirmed, and the quarantine path or scan job ID is not used in place of file_id. |
| `ck_evidence_link_target_type` | CHECK | `target_entity_type` | CHECK (target_entity_type IN ('REQUIREMENT','FDS_SPEC','DDS_SPEC','VP_PLAN','QIA_MODULE_ITEM','VENDOR_AUDIT','DQ_ITEM','FRA_ITEM','IQ_ITEM','OQ_ITEM','PQ_ITEM','IQ_EXECUTION','OQ_EXECUTION','PQ_EXECUTION','IQ_STEP_EXECUTION','OQ_STEP_EXECUTION','PQ_STEP_EXECUTION','VSR_ASSESSMENT','DEVIATION','DELIVERABLE_REVISION')) |
| `ck_evidence_link_type` | CHECK | `evidence_type` | CHECK (evidence_type IN ('TEST_RESULT','SCREENSHOT','LOG','REPORT','APPROVAL_DOCUMENT')) |

#### Business Rules

For the primary attachment of VA/FDS/DDS, the file_id in the respective table is the original. The same primary attachment is not duplicated and edited here as a separate original.

When IQ/OQ/PQ step evidence is newly uploaded, it is first requested via file_link_request (EVIDENCE), and once the file passes the scan, a row in this table is created according to that request. When carrying existing evidence over to a new revision during a revision, or attaching an already registered file to another target, link it directly with the existing file_id.

[↑ Back to top](#top)

---

<a id="table-file_link_request"></a>
### 16. File Link Request (`file_link_request`)

| Item | Definition |
|---|---|
| Description | A request that records in advance which business target an uploaded file will be linked to after it passes the scan. The file scan Worker links it automatically when it passes |
| Primary Key | `link_request_id` |
| Main References (FK) | `project_id, file_id, evidence_link_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 202 | Link Request ID | `link_request_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the file link request | `00000000-0000-0000-0000-000000000001` |
| 203 | Upload ID | `upload_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | file_scan_job.upload_id. Because scan_job_id changes on rescan, the request is made per upload rather than per scan attempt. It is not a table key, so no physical FK is defined | `00000000-0000-0000-0000-000000000001` |
| 204 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Project to which the link target belongs | `00000000-0000-0000-0000-000000000001` |
| 205 | Link Kind | `link_kind` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | EVIDENCE=create an evidence_link row, PRIMARY_FILE=fill in the file_id of the target row | `EVIDENCE` |
| 206 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | EVIDENCE: IQ_STEP_EXECUTION / OQ_STEP_EXECUTION / PQ_STEP_EXECUTION. PRIMARY_FILE: FDS_SPEC / DDS_SPEC / VENDOR_AUDIT | `IQ_STEP_EXECUTION` |
| 207 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | PK of the target row. No physical FK because this is a polymorphic reference (same approach as evidence_link) | `00000000-0000-0000-0000-000000000001` |
| 208 | Evidence Type | `evidence_type` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Required for EVIDENCE; same value as evidence_link.evidence_type. NULL for PRIMARY_FILE | `TEST_RESULT` |
| 209 | Evidence Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Used only for EVIDENCE and moved to evidence_link.description when linked. NULL for PRIMARY_FILE | `Screen capture of IQ step 3 results` |
| 210 | Request Status | `request_status` | `varchar(20)` | N | N | - | Y | `'WAITING'` | N | Y | N | Y | WAITING=awaiting scan, LINKED=link completed, BLOCKED=blocked by the scan (no file_asset), FAILED=the file passed but linking failed (target locked, etc.), CANCELLED=cancelled by the user or target draft deleted | `WAITING` |
| 211 | Linked File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | File that passed the scan and was linked. Required for LINKED | `00000000-0000-0000-0000-000000000001` |
| 212 | Created Evidence Link ID | `evidence_link_id` | `uuid` | N | Y | `evidence_link.evidence_link_id` | N | - | N | Y | N | Y | evidence_link row created by linking as EVIDENCE. Required when EVIDENCE and LINKED | `00000000-0000-0000-0000-000000000001` |
| 213 | Linked At | `linked_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the link to the business target was completed. Required for LINKED | `2026-09-01T10:05:00Z` |
| 214 | Failure Reason Code | `failure_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Block/failure reason code. BLOCKED (the file did not pass the scan and must be uploaded again): THREATS_FOUND, ENCRYPTED, FORMAT_MISMATCH, UNSUPPORTED_EXTENSION, SOURCE_MISMATCH, ENGINE_UNSCANNABLE, SIZE_LIMIT_EXCEEDED, SCAN_FAILED, SCAN_CANCELLED. FAILED (the file is valid but could not be linked to the business target): TARGET_NOT_FOUND, TARGET_LOCKED, FILE_NOT_FOUND, FILE_PROJECT_MISMATCH, FILE_CATEGORY_MISMATCH. Required for BLOCKED/FAILED and used as the reason shown to the user on screen (ck_file_link_failure_code) | - |
| 215 | Failure Reason | `failure_message` | `text` | N | N | - | N | - | N | N | N | Y | Description of the block/failure reason | - |
| 216 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Request creation time (UTC) | `2026-09-01T10:00:00Z` |
| 217 | Requester ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who made the request. Must be the same as the user who uploaded the file | `00000000-0000-0000-0000-000000000001` |
| 218 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Request last update time (UTC) | `2026-09-01T10:05:00Z` |
| 219 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Party that last changed the request status. The user_id of the SYSTEM account if changed by the Worker | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_file_link_request_upload` | UNIQUE | `upload_id, request_status` | UNIQUE (upload_id) WHERE request_status <> 'CANCELLED' |
| `ck_file_link_kind` | CHECK | `link_kind` | CHECK (link_kind IN ('EVIDENCE','PRIMARY_FILE')) |
| `ck_file_link_target` | CHECK | `link_kind, target_entity_type` | CHECK ((link_kind='EVIDENCE' AND target_entity_type IN ('IQ_STEP_EXECUTION','OQ_STEP_EXECUTION','PQ_STEP_EXECUTION')) OR (link_kind='PRIMARY_FILE' AND target_entity_type IN ('FDS_SPEC','DDS_SPEC','VENDOR_AUDIT'))) |
| `ck_file_link_evidence_type` | CHECK | `link_kind, evidence_type, description` | CHECK ((link_kind='EVIDENCE' AND evidence_type IN ('TEST_RESULT','SCREENSHOT','LOG','REPORT','APPROVAL_DOCUMENT')) OR (link_kind='PRIMARY_FILE' AND evidence_type IS NULL AND description IS NULL)) |
| `ck_file_link_status` | CHECK | `request_status` | CHECK (request_status IN ('WAITING','LINKED','BLOCKED','FAILED','CANCELLED')) |
| `ck_file_link_status_state` | CHECK | `request_status, link_kind, file_id, evidence_link_id, linked_at, failure_code` | CHECK ((request_status='WAITING' AND file_id IS NULL AND linked_at IS NULL AND failure_code IS NULL) OR (request_status='LINKED' AND file_id IS NOT NULL AND linked_at IS NOT NULL AND (link_kind<>'EVIDENCE' OR evidence_link_id IS NOT NULL)) OR (request_status='BLOCKED' AND file_id IS NULL AND failure_code IS NOT NULL) OR (request_status='FAILED' AND failure_code IS NOT NULL) OR (request_status='CANCELLED' AND linked_at IS NULL)) |
| `ck_file_link_evidence_link` | CHECK | `link_kind, evidence_link_id` | CHECK (evidence_link_id IS NULL OR link_kind='EVIDENCE') |
| `ck_file_link_failure_code` | CHECK | `request_status, failure_code` | CHECK ((failure_code IS NULL OR failure_code IN ('THREATS_FOUND','ENCRYPTED','FORMAT_MISMATCH','UNSUPPORTED_EXTENSION','SOURCE_MISMATCH','ENGINE_UNSCANNABLE','SIZE_LIMIT_EXCEEDED','SCAN_FAILED','SCAN_CANCELLED','TARGET_NOT_FOUND','TARGET_LOCKED','FILE_NOT_FOUND','FILE_PROJECT_MISMATCH','FILE_CATEGORY_MISMATCH')) AND (request_status <> 'BLOCKED' OR failure_code IN ('THREATS_FOUND','ENCRYPTED','FORMAT_MISMATCH','UNSUPPORTED_EXTENSION','SOURCE_MISMATCH','ENGINE_UNSCANNABLE','SIZE_LIMIT_EXCEEDED','SCAN_FAILED','SCAN_CANCELLED')) AND (request_status <> 'FAILED' OR failure_code IN ('TARGET_NOT_FOUND','TARGET_LOCKED','FILE_NOT_FOUND','FILE_PROJECT_MISMATCH','FILE_CATEGORY_MISMATCH'))) |
| `rule_file_link_request_1` | Business validation | `created_by, upload_id, target_entity_type, target_entity_id` | The requester must be the same as the user who uploaded the file (file_scan_job.uploaded_by). The link target must exist, belong to the same project, and be in an editable state, and the requester must have edit permission on the target. The editable state is: for FDS/DDS/VA, before submission (REVIEW); for IQ/OQ/PQ step executions, when the execution_status of the parent execution (*_execution) is IN_PROGRESS (before results are registered). |
| `rule_file_link_request_2` | Business validation | `request_status` | Link processing locks the request row and then changes its status. If the file has already passed the scan at the time of the request, it is linked immediately; otherwise, the Worker links it after the scan passes. File registration (file_asset) and business linking are in separate transactions, so even if linking fails, the file that passed the scan is kept, and FAILED and the reason are recorded. |
| `rule_file_link_request_3` | Business validation | `request_status` | When the user cancels, WAITING/BLOCKED/FAILED requests are changed to CANCELLED. When an FDS/DDS draft is deleted (deleted_at) or the parent execution of an IQ/OQ/PQ step execution is aborted (ABORTED), the unlinked (WAITING/BLOCKED/FAILED) requests for that target are changed to CANCELLED in the same transaction. LINKED requests are not cancelled. |
| `rule_file_link_request_4` | Business validation | `target_entity_type, target_entity_id, request_status` | Before submission or result registration for the target, check whether any WAITING/BLOCKED/FAILED requests exist; if so, proceeding is not allowed. BLOCKED/FAILED are resolved by the user cancelling or re-uploading. |

#### Business Rules

The user waits only until the file upload completes, and the Worker handles malware and format scanning separately. To uphold the principle that only files that passed the scan are registered as file_asset (rule_file_asset_scan), the upload and the link target are first recorded in this table, and when the scan passes, the Worker creates an evidence_link row for EVIDENCE (created_by is the requester) and fills in the file_id of fds_spec/dds_spec/vendor_audit for PRIMARY_FILE.

Audit records for request registration and cancellation are recorded with the screen's menu_code (USER), and the Worker's status transitions and linking with SYSTEM_PROCESS (SYSTEM/BATCH). When vendor_audit.file_id is empty, 'no attachment' and 'scan in progress' are distinguished by the request status in this table.

When carrying existing evidence over to a new revision during a revision, or attaching an already registered file to another target (such as a deviation), there is no upload, so this table is not used and the link is made directly with the existing file_id. How to clean up files that were uploaded but not requested is determined separately.

[↑ Back to top](#top)

---

## System

<a id="table-system_asset"></a>
### 17. System/Equipment Identification (`system_asset`)

| Item | Definition |
|---|---|
| Description | Baseline information for systems/equipment subject to validation (management number, GAMP category) |
| Primary Key | `system_id` |
| Main References (FK) | `organization_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 220 | System ID | `system_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique system identifier | `00000000-0000-0000-0000-000000000001` |
| 221 | Organization ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | Customer company or operating organization to which the system/equipment belongs | `00000000-0000-0000-0000-000000000001` |
| 222 | Management Number | `management_number` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Asset/equipment management number | `EQ-MES-2024-001` |
| 223 | System Name | `system_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | System/equipment name | `Sample System` |
| 224 | Major Category | `major_category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Computerized system / equipment / facility & utility | `Computerized system` |
| 225 | Middle Category | `middle_category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Middle category belonging to the selected major category | `Quality system` |
| 226 | Target | `target_type` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Target belonging to the selected major and middle categories | `LIMS` |
| 227 | Responsible Department | `department_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Department responsible for management/operation | `Production Engineering Team` |
| 228 | Installation Location | `location` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Physical/logical installation location | `Server room, Building A, 3F` |
| 229 | Vendor | `vendor` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Equipment/system vendor name | `Sample Vendor` |
| 230 | Model Name | `model_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Equipment/system model name | `FillMaster 500` |
| 231 | System Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Description of the system's purpose and operating scope | `Production process data collection and control` |
| 232 | CS Included | `is_cs_included` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether it is included as a Computerized System | `TRUE` |
| 233 | Software Version | `software_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Software version of the computerized system. Distinct from the inventory revision number. | `4.8.2` |
| 234 | Inventory Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Integer version incremented by the revision function. Separate from the software version | `2` |
| 235 | GAMP Category | `gamp_category` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Screen selection value CATEGORY_1/CATEGORY_3/CATEGORY_4/CATEGORY_5 (GAMP 5 software categories: 1=infrastructure software, 3=non-configured products, 4=configured products, 5=custom applications). Category 2 was retired in GAMP 5 and is not used. NULL if not selected | `CATEGORY_4` |
| 236 | GxP Applicability | `gxp_applicability` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | APPLICABLE=applicable, NOT_APPLICABLE=not applicable. NULL if not selected | `APPLICABLE` |
| 237 | Part 11 Applicability | `part11_applicability` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | APPLICABLE=applicable, NOT_APPLICABLE=not applicable. NULL if not selected | `APPLICABLE` |
| 238 | Approval Status | `approval_status` | `varchar(30)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT=in progress, REVIEW=under review, APPROVAL=under approval, APPROVED=approved, REAPPROVAL_REQUIRED=re-approval required, REJECTED=rejected, CANCELLED=cancelled. Modifications after REJECTED or CANCELLED are handled as a new revision with an incremented revision_number | `DRAFT` |
| 239 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-25T00:00:00Z` |
| 240 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-25T00:00:00Z` |
| 241 | In-use/Disposal Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE=in use, DISPOSED=disposed. Separate from the approval status | `ACTIVE` |
| 242 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the disposal electronic signature was completed | `2026-09-01T00:00:00Z` |
| 243 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason entered on the disposal screen | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_system_asset_1` | UNIQUE | `organization_id, management_number` | UNIQUE (organization_id, management_number) |
| `ck_system_asset_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `ck_system_asset_3` | CHECK | `gxp_applicability` | CHECK (gxp_applicability IS NULL OR gxp_applicability IN ('APPLICABLE','NOT_APPLICABLE')) |
| `ck_system_asset_4` | CHECK | - | CHECK (part11_applicability IS NULL OR part11_applicability IN ('APPLICABLE','NOT_APPLICABLE')) |
| `ck_system_asset_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_system_asset_6` | CHECK | `gamp_category` | CHECK (gamp_category IS NULL OR gamp_category IN ('CATEGORY_1','CATEGORY_3','CATEGORY_4','CATEGORY_5')) |
| `ck_system_asset_7` | CHECK | `approval_status` | CHECK (approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REAPPROVAL_REQUIRED','REJECTED','CANCELLED')) |
| `ck_system_asset_8` | CHECK | `lifecycle_status, disposed_at, disposal_reason` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0)) |
| `rule_system_asset_9` | Business validation | `lifecycle_status` | The transition to DISPOSED is saved in the same transaction as the DISPOSE event of system_asset_revision (disposal electronic signature required, rule_system_asset_revision_4), and disposed_at is equal to that signature time. |

#### Business Rules

Depending on whether the system is computerized, the screen-required values for software version, GAMP, GxP and Part 11 are checked. The 3-level classification allows only selectable combinations.

The list of linked projects is retrieved via validation_project.system_id. Revision, approval and disposal history is linked through system_asset_revision and the electronic signature / audit trail records.

When approved information is changed, the revision number is incremented and the re-approval status is displayed. Do not confuse a change to the software version itself with a revision of the inventory document.

Modifications to business fields that have been approved, rejected or cancelled are handled with a new revision_number and re-approval. Changes to the latest system information do not alter the original content of past system_asset_revision records or the approved revision adopted by a project.

[↑ Back to top](#top)

---

<a id="table-system_asset_revision"></a>
### 18. System Inventory Revision History (`system_asset_revision`)

| Item | Definition |
|---|---|
| Description | History of registration, modification, approval and disposal processing per inventory revision number |
| Primary Key | `system_revision_id` |
| Main References (FK) | `system_id, processed_by, signature_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 244 | Inventory History ID | `system_revision_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 245 | System ID | `system_id` | `uuid` | N | Y | `system_asset.system_id` | Y | - | N | Y | N | Y | References system_asset.system_id | `00000000-0000-0000-0000-000000000001` |
| 246 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Inventory revision number at the time of processing | `2` |
| 247 | Action Type | `action_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | REGISTER=register, SAVE=save, REVISE=revise, APPROVE=approve, DISPOSE=dispose | `REVISE` |
| 248 | Change / Action Reason | `change_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason, required for revision or disposal | - |
| 249 | Approval Status After Processing | `approval_status` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | Same codes as system_asset.approval_status | `REVIEW` |
| 250 | Processed By ID | `processed_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 251 | Processed At | `processed_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Date/time shown on the history screen | `2026-09-01T00:00:00Z` |
| 252 | Processing Electronic Signature ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature linked to the approval or disposal. Multiple signatures are retrieved from the signature history of the same target | `00000000-0000-0000-0000-000000000001` |
| 253 | Original Content Format Version | `snapshot_schema_version` | `varchar(50)` | N | N | - | Y | `'ASSET-1'` | N | N | N | Y | Version of the field composition and data types of asset_snapshot. Past format definitions are preserved. | `ASSET-1` |
| 254 | Inventory Original Content at Processing | `asset_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Stores all column values of system_asset after processing, including field names and NULLs. For an approval event, it must match the business original content that received final approval. | - |
| 255 | Original Content Hash | `snapshot_hash` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | Lowercase 64-character SHA-256 of {schema_version:snapshot_schema_version,body:asset_snapshot} serialized with the same canonicalization rules as the electronic signature. Distinct from the electronic signature's full signature-bundle hash. | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `ck_system_asset_revision_1` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `rule_system_asset_revision_2` | Business validation | - | Because multiple processing history records (register/save/approve, etc.) can exist for the same system and revision number, no UNIQUE is defined on those two columns alone. |
| `rule_system_asset_revision_3` | Business validation | - | If action_type is REVISE or DISPOSE, change_reason cannot be empty. |
| `rule_system_asset_revision_4` | Business validation | - | DISPOSE processing requires signature_id, with signature_action=DISPOSE, and the signature target is the system_id of system_asset and the corresponding revision_number. |
| `uq_asset_approved_revision` | UNIQUE | `system_id, revision_number, action_type` | UNIQUE (system_id, revision_number) WHERE action_type='APPROVE' |
| `ck_asset_event_kind` | CHECK | `action_type` | CHECK (action_type IN ('REGISTER','SAVE','REVISE','APPROVE','DISPOSE')) |
| `ck_asset_approval_signature` | CHECK | `action_type, approval_status, signature_id` | CHECK (action_type <> 'APPROVE' OR (approval_status='APPROVED' AND signature_id IS NOT NULL)) |
| `ck_asset_revision_status` | CHECK | `approval_status` | CHECK (approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REAPPROVAL_REQUIRED','REJECTED','CANCELLED')) |

#### Business Rules

Each processing event is appended together with the full original content at that time. Event rows and original content are not modified or deleted. Before/after comparisons are retrieved from audit_trail, and past approved original content is retrieved directly from the asset_snapshot of the APPROVE event. The approval target is identified by the system_id of system_asset and the corresponding revision_number.

The APPROVE event is a record of final approval completion, not a per-review-step event. The signature target's system_id and revision number must match the business fields of the approved original content. The comparison scope, excluding management metadata such as approval status and processing time, follows the electronic signature original content format. There is exactly one final approved original content per revision, and any change to business content after approval is assigned a new revision number. Save, revision, signature and final approval events are recorded in the same transaction as the change to the source record.

[↑ Back to top](#top)

---

## Library

<a id="table-library_item"></a>
### 19. Library Item Master (`library_item`)

| Item | Definition |
|---|---|
| Description | Master of reusable standard library items for URS, FRA, IQ, OQ and PQ |
| Primary Key | `library_id` |
| Main References (FK) | - |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 256 | Library ID | `library_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the library item | `00000000-0000-0000-0000-000000000001` |
| 257 | Module Type | `module_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Distinguishes the 5 library tabs: URS/FRA/IQ/OQ/PQ | `URS` |
| 258 | Code | `code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Item code within the library type. A composite UNIQUE on (module_type, code) applies | `URS-AT-L01` |
| 259 | Category / Major Category | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Stores the major category for URS and the category for FRA/IQ/OQ/PQ. | `Audit trail` |
| 260 | Item Name / Function Name | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Function name for URS, risk item name for FRA, and test item name for IQ/OQ/PQ | `Automatic generation of audit trail for data changes` |
| 261 | Requirement / Procedure | `requirement_text` | `text` | N | N | - | N | - | N | N | N | Y | URS requirement body. Required for the URS type; NULL allowed for other types | `On every data creation, modification or deletion...` |
| 262 | Expected Result | `expected_result` | `text` | N | N | - | N | - | N | N | N | Y | IQ/OQ/PQ expected result. Required in test libraries | - |
| 263 | Acceptance Criteria / Scope of Application | `acceptance_criteria` | `text` | N | N | - | N | - | N | N | N | Y | Scope of application for URS, acceptance criteria for IQ/OQ/PQ. NULL allowed for FRA | `Audit Trail automatically generated on data change...` |
| 264 | Regulatory Basis | `regulation` | `text` | N | N | - | N | - | N | N | N | Y | Regulatory basis displayed and manually entered on the screen. Structured clauses are linked in library_item_regulation, and the same basis is not edited in both places. | `21 CFR 11.10(e)` |
| 265 | In Use | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether active (TRUE/FALSE) | `TRUE` |
| 266 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-25T00:00:00Z` |
| 267 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-25T00:00:00Z` |
| 268 | Provision Type | `provision_type` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | URS only. BUILTIN=built-in, CUSTOM=user-defined | `CUSTOM` |
| 269 | Middle Category | `middle_category` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | URS-only middle category | - |
| 270 | Target | `target_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | URS-only applicable target | - |
| 271 | Test Content | `test_content` | `text` | N | N | - | N | - | N | N | N | Y | IQ/OQ/PQ-only test procedure / content | - |
| 272 | Risk Scenario | `risk_scenario` | `text` | N | N | - | N | - | N | N | N | Y | FRA-only risk scenario | - |
| 273 | Severity (SEV) | `severity` | `smallint` | N | N | - | N | - | N | N | N | Y | FRA only. 1–5 | - |
| 274 | Occurrence (OCC) | `occurrence` | `smallint` | N | N | - | N | - | N | N | N | Y | FRA only. 1–5 | - |
| 275 | Detectability (DET) | `detectability` | `char(1)` | N | N | - | N | - | N | N | N | Y | FRA only. H/M/L | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_library_item_1` | UNIQUE | `module_type, code` | UNIQUE (module_type, code) |
| `ck_library_item_2` | CHECK | `module_type` | CHECK (module_type IN ('URS','FRA','IQ','OQ','PQ')) |
| `rule_library_item_3` | Business validation | - | URS: provision type, major category, middle category, target, function name, requirement and scope of application are required. FRA: category, risk item name, scenario, SEV, OCC and DET are required. IQ/OQ/PQ: category, item name, test content, expected result and acceptance criteria are required. |
| `rule_library_item_4` | Business validation | `severity` | For FRA, severity/occurrence are 1–5 and detectability is one of H/M/L. |
| `ck_library_item_provision_type` | CHECK | `provision_type` | CHECK (provision_type IS NULL OR provision_type IN ('BUILTIN','CUSTOM')) |

#### Business Rules

For URS, the major category is managed in category, the function name in title, and the scope of application in acceptance_criteria.

When a template is applied to a requirement, risk or test, its body and input values are copied. Subsequent library modifications do not automatically change items that have already been authored or approved.

[↑ Back to top](#top)

---

## Validation

<a id="table-validation_project"></a>
### 20. Validation Project (`validation_project`)

| Item | Definition |
|---|---|
| Description | Per-system validation execution unit and scope (VP) |
| Primary Key | `project_id` |
| Main References (FK) | `system_id, created_by, updated_by, closure_requested_by, current_closure_request_id, current_system_baseline_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 276 | Project ID | `project_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique project identifier | `00000000-0000-0000-0000-000000000001` |
| 277 | Project Code | `project_code` | `varchar(100)` | N | N | - | Y | - | Y | Y | N | Y | Project identification code | `VP-SYS-008-20260422` |
| 278 | Project Name | `project_name` | `varchar(200)` | N | N | - | Y | - | N | Y | N | Y | Project Name | `Test Equipment 3 CSV Project` |
| 279 | System ID | `system_id` | `uuid` | N | Y | `system_asset.system_id` | Y | - | N | Y | N | Y | References system_asset.system_id | `00000000-0000-0000-0000-000000000001` |
| 280 | Status | `status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS=in progress, CLOSED_NORMAL=normal closure, CLOSED_FORCED=forced closure | `IN_PROGRESS` |
| 281 | Validation Type | `validation_type` | `varchar(50)` | N | N | - | Y | `'NEW'` | N | N | N | Y | NEW=new, CHANGE=change, REVALIDATION=revalidation | `NEW` |
| 282 | Validation Level | `validation_level` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | LEVEL_1/LEVEL_2/LEVEL_3/CUSTOM. CUSTOM=user-defined, and the activities are stored in project_activity | `LEVEL_2` |
| 283 | Remarks | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | Additional notes | - |
| 284 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-25T00:00:00Z` |
| 285 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the user who created the project | `00000000-0000-0000-0000-000000000001` |
| 286 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 287 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the user who updated the project | `00000000-0000-0000-0000-000000000001` |
| 288 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time | `2026-09-01T00:00:00Z` |
| 289 | Closure Type | `closure_type` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | Summary of the closure type of the current/final project_closure_request. Synchronized from the source request and not edited independently. | `FORCED` |
| 290 | Forced Closure Reason | `closure_reason` | `text` | N | N | - | N | - | N | N | N | Y | Summary of the forced closure reason of the current/final project_closure_request. Synchronized from the source request and not edited independently. | `Validation discontinued due to a change in business priorities` |
| 291 | Closure Requester ID | `closure_requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Summary of the closure requester ID of the current/final project_closure_request. Synchronized from the source request and not edited independently. | `00000000-0000-0000-0000-000000000001` |
| 292 | Closure Requested At | `closure_requested_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Summary of the closure request time of the current/final project_closure_request. Synchronized from the source request and not edited independently. | `2026-09-14T09:00:00Z` |
| 293 | Closure Completed At | `closed_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Summary of the closure completion time of the current/final project_closure_request. Synchronized from the source request and not edited independently. | `2026-09-14T09:05:00Z` |
| 294 | Current Closure Request ID | `current_closure_request_id` | `uuid` | N | Y | `project_closure_request.closure_request_id` | N | - | N | Y | N | Y | Closure request whose closure progress status and reason are displayed | `00000000-0000-0000-0000-000000000001` |
| 295 | Current Validation Baseline | `current_system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | The baseline of the same project that currently has is_current=TRUE. Required before the first business submission or test execution | - |
| 296 | Applied Dependency Condition Version | `dependency_set_version` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Version of the dependency rule set adopted at project creation (activity_dependency.dependency_set_version) | `DEPENDENCY-1` |
| 297 | Applied Dependency Condition Original Content | `dependency_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Preserves all conditions active at creation time and their judgment semantics. {schema_version,version,definitions,conditions:[{activity_dependency_id,successor_activity_id,predecessor_activity_id,dependency_type,required_status,condition_type,condition_value,evaluation_order}]} | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `ck_validation_project_1` | CHECK | `status` | CHECK (status IN ('IN_PROGRESS','CLOSED_NORMAL','CLOSED_FORCED')) |
| `ck_validation_project_2` | CHECK | `validation_type` | CHECK (validation_type IN ('NEW','CHANGE','REVALIDATION')) |
| `ck_validation_project_3` | CHECK | `validation_level` | CHECK (validation_level IN ('LEVEL_1','LEVEL_2','LEVEL_3','CUSTOM')) |
| `rule_project_current_refs` | Business validation | `current_closure_request_id, current_system_baseline_id` | The current closure request and the validation baseline must both belong to this project. If a baseline exists, the current baseline reference must match the single row with is_current=TRUE. |
| `ck_validation_project_closure_type` | CHECK | `closure_type` | CHECK (closure_type IS NULL OR closure_type IN ('NORMAL','FORCED')) |

#### Business Rules

The project name, target system, validation type, level and memo shown on the screen are managed in this table. Progress is calculated from the completion status of the selected activities, and RTM is excluded from the denominator.

The project's validation-target GAMP, GxP and software version are retrieved from the approved original content referenced by current_system_baseline_id. The latest system information and the validation baseline are displayed separately.

The review/approval progress of a closure request is stored in project_closure_request and workflow_instance. The project is changed to normal closure or forced closure only after final approval.

At project creation, dependency_set_version and dependency_snapshot are fixed together, and changes to the global activity_dependency are not automatically propagated to in-progress or closed projects. The condition definitions include per-status completion judgment, AND/OR combination, handling of unselected activities, and risk coverage judgment criteria. Project evaluation uses this copy, and it is not directly replaced after creation. A valid current_system_baseline_id is set before business approval or test execution, and the initial baseline registration and the setting of the current reference are processed in a single transaction.

[↑ Back to top](#top)

---

<a id="table-project_system_baseline"></a>
### 21. Project Validation Target Revision (`project_system_baseline`)

| Item | Definition |
|---|---|
| Description | Approved inventory revision adopted by the project as its validation target, and its change history |
| Primary Key | `baseline_id` |
| Main References (FK) | `project_id, system_revision_id, adopted_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period. Approved original content and past link history are not deleted. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 298 | Validation Baseline ID | `baseline_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Identifier of a single validation baseline adopted by the project | - |
| 299 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Project to which the validation baseline belongs | - |
| 300 | Inventory Approval History ID | `system_revision_id` | `uuid` | N | Y | `system_asset_revision.system_revision_id` | Y | - | N | Y | N | Y | Immutable original-content event with action_type=APPROVE | - |
| 301 | Baseline Number | `baseline_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Increments from 1 per project; a new row is created on each change. | `1` |
| 302 | Adoption / Change Reason | `change_reason` | `text` | N | N | - | Y | - | N | N | N | Y | Reason for the initial target selection or for a change of approved revision. Cannot be blank | - |
| 303 | Is Current | `is_current` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the current project baseline. Past rows are changed to FALSE, and the link to the original content is retained. | - |
| 304 | Adopted At | `adopted_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Application time recorded by the server | - |
| 305 | Adopted By | `adopted_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Processing user with permission to set the project target | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_project_baseline_number` | UNIQUE | `project_id, baseline_number` | UNIQUE (project_id, baseline_number) |
| `uq_project_baseline_current` | UNIQUE | `project_id, is_current` | UNIQUE (project_id) WHERE is_current=TRUE |
| `ck_project_baseline_number` | CHECK | `baseline_number` | CHECK (baseline_number >= 1) |
| `ck_project_baseline_reason` | CHECK | `change_reason` | CHECK (length(trim(change_reason)) > 0) |
| `rule_project_baseline_asset` | Business validation | `project_id, system_revision_id` | The system_id of the approval event must be the same as the project's system_id. At adoption, verify that the system is in use and that the revision is the currently valid approved version. A past approval event from a previous baseline must not be adopted as the latest baseline merely because it exists. |

#### Business Rules

The adopted system_revision_id, sequence number, reason, processor and time are not overwritten. On change, the project is locked, the previous is_current is switched to FALSE, and then the new row and the current reference are saved in the same transaction. The baseline of a closed project is not changed.

A change of the target revision updates the re-review and re-approval status of affected activities and documents, while the baseline links of already approved records are retained. Past test executions and documents retrieve the software version, category and GxP applicability at that time through their own baseline_id.

[↑ Back to top](#top)

---

<a id="table-project_member"></a>
### 22. Project Member (`project_member`)

| Item | Definition |
|---|---|
| Description | Management of participating users and their roles per project |
| Primary Key | `project_member_id` |
| Main References (FK) | `project_id, user_id, role_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 306 | Project Member ID | `project_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the project member role mapping. Duplicate (project_id, user_id, role_id) values are not allowed among rows where deleted_at IS NULL and member_status is ACTIVE. | `00000000-0000-0000-0000-000000000001` |
| 307 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | ID of the Validation project to which the member belongs | `00000000-0000-0000-0000-000000000001` |
| 308 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user participating in the project | `00000000-0000-0000-0000-000000000001` |
| 309 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | ID of the role the user performs within the project | `00000000-0000-0000-0000-000000000001` |
| 310 | Membership Status | `member_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | Y | N | Y | Project participation status. ACTIVE, INACTIVE, WITHDRAWN | `ACTIVE` |
| 311 | Joined At | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time project participation started | `2026-09-01T10:00:00Z` |
| 312 | Left At | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time project participation ended. NULL if currently participating | `2026-09-02T10:00:00Z` |
| 313 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project member information creation time (UTC) | `2026-09-01T10:00:00Z` |
| 314 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who registered the project member information | `00000000-0000-0000-0000-000000000001` |
| 315 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project member information last update time (UTC) | `2026-09-01T10:00:00Z` |
| 316 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who last updated the project member information | `00000000-0000-0000-0000-000000000001` |
| 317 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time of the project member information | `2026-09-02T10:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_project_member_1` | UNIQUE | `project_id, user_id, role_id, deleted_at, member_status` | UNIQUE (project_id, user_id, role_id) WHERE deleted_at IS NULL AND member_status='ACTIVE' |
| `ck_project_member_status` | CHECK | `member_status` | CHECK (member_status IN ('ACTIVE','INACTIVE','WITHDRAWN')) |

#### Business Rules

Manages participants and their roles per project. View/edit/disposal permissions on screens are managed separately in access_permission_grant.

[↑ Back to top](#top)

---

<a id="table-validation_activity"></a>
### 23. Validation Activity Master (`validation_activity`)

| Item | Definition |
|---|---|
| Description | Management of reference information for the VP, VA, QIA, URS, FDS_GROUP, FRA, DQ, IQ, OQ, PQ and VSR activities |
| Primary Key | `activity_id` |
| Main References (FK) | `created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 318 | Activity ID | `activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the validation activity | `00000000-0000-0000-0000-000000000001` |
| 319 | Activity Code | `activity_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | VP, VA, QIA, URS, FDS_GROUP, FRA, DQ, IQ, OQ, PQ, VSR. FDS_GROUP corresponds to the F&DS display name on the screen | `FDS_GROUP` |
| 320 | Activity Name | `activity_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Activity name for screen display | `Functional and Detailed Design Specification` |
| 321 | Activity Order | `display_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Default display order: VP→VA→QIA→URS→F&DS→FRA→DQ→IQ→OQ→PQ→VSR. Within F&DS, the DDS document follows the FDS document | `5` |
| 322 | In Use | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the activity master is in use | `TRUE` |
| 323 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Activity master creation time (UTC) | `2026-09-01T10:00:00Z` |
| 324 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the activity master | `00000000-0000-0000-0000-000000000001` |
| 325 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Activity master last update time (UTC) | `2026-09-01T10:00:00Z` |
| 326 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last updated the activity master | `00000000-0000-0000-0000-000000000001` |
| 327 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time of the activity master | `2026-09-02T10:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `ck_validation_activity_code` | CHECK | `activity_code` | CHECK (activity_code IN ('VP','VA','QIA','URS','FDS_GROUP','FRA','DQ','IQ','OQ','PQ','VSR')) |

#### Business Rules

RTM is a dashboard function for viewing the traceability relationships among requirements, design, risks and tests. The system inventory is managed as prerequisite baseline information for a project.

FDS and DDS are document types belonging to the single F&DS activity. The two types are not created redundantly as separate execution stages.

[↑ Back to top](#top)

---

<a id="table-project_activity"></a>
### 24. Project Activity (`project_activity`)

| Item | Definition |
|---|---|
| Description | Management of the activities to be performed per project, their activation and their progress status |
| Primary Key | `project_activity_id` |
| Main References (FK) | `project_id, activity_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 328 | Project Activity ID | `project_activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the project activity. Duplicate (project_id, activity_id) values are not allowed among rows where deleted_at IS NULL. This applies regardless of whether the activity is selected for execution. | `00000000-0000-0000-0000-000000000001` |
| 329 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Validation project to which the activity belongs | `00000000-0000-0000-0000-000000000001` |
| 330 | Activity ID | `activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | Activity to be performed in the project | `00000000-0000-0000-0000-000000000001` |
| 331 | Selected for Execution | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Whether this is an actual activity selected for execution in the project settings. No RTM selection row is created. | `TRUE` |
| 332 | Required Activity | `is_required` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Whether the activity cannot be omitted in this project | `TRUE` |
| 333 | Activity Status | `activity_status` | `varchar(20)` | N | N | - | Y | `'LOCKED'` | N | Y | N | Y | Activity status. LOCKED, READY, IN_PROGRESS, COMPLETED, APPROVED, SKIPPED. Aggregated based on the current Revision and re-evaluated when a deliverable is changed or revised. COMPLETED/APPROVED are not set by a simple screen input value | `READY` |
| 334 | Activated At | `activated_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the activity became READY because its predecessor conditions were satisfied | `2026-09-01T10:00:00Z` |
| 335 | Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the activity execution started | `2026-09-01T11:00:00Z` |
| 336 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the activity execution was completed | `2026-09-02T15:00:00Z` |
| 337 | Approved At | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the final approval of the activity was completed | `2026-09-02T17:00:00Z` |
| 338 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project activity creation time (UTC) | `2026-09-01T10:00:00Z` |
| 339 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the project activity | `00000000-0000-0000-0000-000000000001` |
| 340 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project activity last update time (UTC) | `2026-09-01T10:00:00Z` |
| 341 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last updated the project activity | `00000000-0000-0000-0000-000000000001` |
| 342 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time of the project activity | `2026-09-02T10:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_project_activity_1` | UNIQUE | `project_id, activity_id, deleted_at` | UNIQUE (project_id, activity_id) WHERE deleted_at IS NULL |
| `ck_project_activity_status` | CHECK | `activity_status` | CHECK (activity_status IN ('LOCKED','READY','IN_PROGRESS','COMPLETED','APPROVED','SKIPPED')) |

#### Business Rules

Progress, the number of completed activities and the normal closure conditions are based on the actual activities selected for execution. RTM's own approval and deliverables are not required. FDS/DDS progress is aggregated into the single F&DS activity.

[↑ Back to top](#top)

---

<a id="table-activity_dependency"></a>
### 25. Activity Dependency Condition (`activity_dependency`)

| Item | Definition |
|---|---|
| Description | Management of predecessor activities, relationship types and judgment conditions for activating successor activities |
| Primary Key | `activity_dependency_id` |
| Main References (FK) | `successor_activity_id, predecessor_activity_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 343 | Activity Dependency Condition ID | `activity_dependency_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the activity dependency condition | `00000000-0000-0000-0000-000000000001` |
| 344 | Condition Set Version | `dependency_set_version` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Version of the dependency condition set to which this condition belongs. All rows with the same version form one set, and rows of a version adopted by a project are not modified, deactivated or deleted. A change creates a full set of condition rows under a new version. validation_project.dependency_set_version points to this value | `DEPENDENCY-1` |
| 345 | Successor Activity ID | `successor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | Activity that is activated once the condition is satisfied | `00000000-0000-0000-0000-000000000001` |
| 346 | Predecessor Activity ID | `predecessor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | N | - | N | Y | N | Y | Activity to check before the successor activity is activated. NULL is allowed for an all-activities condition. Required for a STATUS condition | `00000000-0000-0000-0000-000000000001` |
| 347 | Dependency Type | `dependency_type` | `varchar(20)` | N | N | - | Y | `'REQUIRED'` | N | Y | N | Y | Type of the predecessor/successor relationship. REQUIRED, RECOMMENDED | `REQUIRED` |
| 348 | Required Status | `required_status` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | Judgment criterion for a STATUS condition. CREATED means a valid current deliverable exists, COMPLETED means execution is completed, and APPROVED means final approval of the current deliverable. CREATED is not stored in project_activity.activity_status. Required only for STATUS conditions; detailed judgment follows the business rules below. | `APPROVED` |
| 349 | Condition Type | `condition_type` | `varchar(50)` | N | N | - | Y | `'STATUS'` | N | Y | N | Y | STATUS, ACTIVITY_SELECTED, TRACEABILITY_EXISTS, HIGH_RISK_COVERED, OPEN_DEVIATION_ZERO, ALL_SELECTED_APPROVED. Type used to judge the predecessor steps and completion conditions on the screen | `STATUS` |
| 350 | Condition Value | `condition_value` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Argument for condition judgment. For STATUS + CREATED, stores the name of an allowed target deliverable table (e.g. requirement). Arguments for other conditions are validated in the implementation of that condition, and left NULL if not needed. | - |
| 351 | Condition Description | `condition_description` | `text` | N | N | - | Y | - | N | N | N | Y | Description of the predecessor activities and approval status required to activate the activity | `URS approved` |
| 352 | Evaluation Order | `evaluation_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Evaluation order of the conditions for the same successor activity. The evaluation order does not change the AND/OR relationship between conditions | `1` |
| 353 | In Use | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the activation condition is in use | `TRUE` |
| 354 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Condition creation time (UTC) | `2026-09-01T10:00:00Z` |
| 355 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the condition | `00000000-0000-0000-0000-000000000001` |
| 356 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Condition last update time (UTC) | `2026-09-01T10:00:00Z` |
| 357 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last updated the condition | `00000000-0000-0000-0000-000000000001` |
| 358 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time of the condition | `2026-09-02T10:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `ck_activity_dependency_type` | CHECK | `dependency_type` | CHECK (dependency_type IN ('REQUIRED','RECOMMENDED')) |
| `ck_activity_dependency_required_status` | CHECK | `required_status` | CHECK (required_status IS NULL OR required_status IN ('CREATED','COMPLETED','APPROVED')) |
| `ck_activity_dependency_condition_type` | CHECK | `condition_type` | CHECK (condition_type IN ('STATUS','ACTIVITY_SELECTED','TRACEABILITY_EXISTS','HIGH_RISK_COVERED','OPEN_DEVIATION_ZERO','ALL_SELECTED_APPROVED')) |
| `ck_activity_dependency_status_condition` | CHECK | `condition_type, predecessor_activity_id, required_status` | CHECK ((condition_type='STATUS' AND predecessor_activity_id IS NOT NULL AND required_status IS NOT NULL) OR (condition_type<>'STATUS' AND required_status IS NULL)) |
| `ck_activity_dependency_not_self` | CHECK | `successor_activity_id, predecessor_activity_id` | CHECK (predecessor_activity_id IS NULL OR predecessor_activity_id <> successor_activity_id) |
| `uq_activity_dependency_condition` | UNIQUE | `dependency_set_version, successor_activity_id, predecessor_activity_id, condition_type, condition_value, is_active, deleted_at` | UNIQUE (dependency_set_version, successor_activity_id, COALESCE(predecessor_activity_id, '00000000-0000-0000-0000-000000000000'::uuid), condition_type, COALESCE(condition_value, '')) WHERE is_active=TRUE AND deleted_at IS NULL |
| `rule_activity_dependency_cycle` | Business validation | `successor_activity_id, predecessor_activity_id` | If the active conditions of the same dependency_set_version would create a predecessor/successor cycle, the save is rejected. |
| `rule_activity_dependency_version_lock` | Business validation | `dependency_set_version` | Rows of a version adopted as validation_project.dependency_set_version are not modified, deactivated or deleted. |

#### Business Rules

The all-activities-approved condition evaluates only the actual activities selected for execution. RTM is not registered as a predecessor or successor activity, or as an independent approval condition.

For STATUS conditions, predecessor_activity_id and required_status are required, and required_status is CREATED/COMPLETED/APPROVED. For other conditions, predecessor activity and status values that are not needed are left NULL. Cycles are not allowed in active predecessor/successor relationships.

This table is the global condition template applied to new projects. On change, a full set of condition rows is created and published under a new dependency_set_version, and rows of a version adopted by a project are not modified, deactivated or deleted. The judgment semantics per condition type (definitions) are fixed definitions identified by version name, and a different definition is never registered under the same version name (the same approach as the QIA and FRA judgment rule versions). At project creation, the full content of the selected version is copied to validation_project.dependency_snapshot, and subsequent modification, deactivation or deletion of conditions does not change the conditions of existing projects.

[↑ Back to top](#top)

---

<a id="table-project_closure_request"></a>
### 26. Project Closure Request (`project_closure_request`)

| Item | Definition |
|---|---|
| Description | Normal/forced closure reasons and the review/approval progress of each request |
| Primary Key | `closure_request_id` |
| Main References (FK) | `project_id, requested_by, workflow_instance_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 359 | Closure Request ID | `closure_request_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 360 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 361 | Request Sequence | `request_version` | `integer` | N | N | - | Y | - | N | N | N | Y | Distinguishes closure re-requests for the same project | `1` |
| 362 | Closure Type | `closure_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | NORMAL=normal closure, FORCED=forced closure | `NORMAL` |
| 363 | Request Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT=in progress, REVIEW=under review, APPROVAL=under approval, APPROVED=completed, REJECTED=rejected, CANCELLED=cancelled | `REVIEW` |
| 364 | Closure Reason | `reason` | `text` | N | N | - | N | - | N | N | N | Y | Required for a forced closure | - |
| 365 | Progress at Request | `progress_snapshot` | `numeric(5,2)` | N | N | - | Y | - | N | N | N | Y | Progress at the time of the request, as shown on the closure confirmation screen. Excludes RTM | `100.00` |
| 366 | Requester ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 367 | Requested At | `requested_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Request time value | `2026-09-01T00:00:00Z` |
| 368 | Closure Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |
| 369 | Closure Processed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Closure processing time value | `2026-09-01T00:00:00Z` |
| 370 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation time value | `2026-09-01T00:00:00Z` |
| 371 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 372 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Update time value | `2026-09-01T00:00:00Z` |
| 373 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_project_closure_request_1` | UNIQUE | `project_id, request_version` | UNIQUE (project_id, request_version) |
| `ck_project_closure_request_2` | CHECK | `request_version` | CHECK (request_version >= 1) |
| `ck_project_closure_request_3` | CHECK | `progress_snapshot` | CHECK (progress_snapshot BETWEEN 0 AND 100) |
| `rule_project_closure_request_4` | Business validation | `closure_type` | If closure_type='FORCED', reason cannot be empty. |
| `rule_project_closure_request_5` | Business validation | - | There can be at most one closure request in DRAFT/REVIEW/APPROVAL status for the same project. |
| `uq_project_closure_active` | UNIQUE | `project_id` | UNIQUE (project_id) WHERE status IN ('DRAFT','REVIEW','APPROVAL') |
| `ck_project_closure_request_status` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_project_closure_request_type` | CHECK | `closure_type` | CHECK (closure_type IN ('NORMAL','FORCED')) |

#### Business Rules

Normal closure checks the completion and approval status of the selected activities and the required deliverables. Independent RTM approval and RTM deliverables are excluded from the closure conditions.

Signers are linked through the electronic signature and approval workflow processing history. The target of the workflow_instance is project_closure_request/closure_request_id/CLOSE-{request_version}, and the project must match. After final approval, validation_project.status and the closure summary are updated together, and under review/under approval is not treated as a project closed status.

[↑ Back to top](#top)

---

## VP

<a id="table-vp_plan"></a>
### 27. Validation Plan (`vp_plan`)

| Item | Definition |
|---|---|
| Description | Editing of VP sections and body, and management of business-approval revisions. |
| Primary Key | `vp_id` |
| Main References (FK) | `project_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 374 | VP Revision ID | `vp_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 375 | Cross-revision Logical ID | `vp_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision row PK | `00000000-0000-0000-0000-000000000001` |
| 376 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 377 | Plan Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Display title of the project validation plan | `System Validation Plan` |
| 378 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Version shown on screen for this revision | `ver1` |
| 379 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 per logical key | `1` |
| 380 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising. Required when revision_number ≥ 2 | `Requirement change` |
| 381 | Is Current Revision | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Current revision of the same logical key. Distinct from whether it is the latest approved version | `TRUE` |
| 382 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress) / REVIEW (under review) / APPROVAL (under approval) / APPROVED (approved) / REJECTED (rejected) / CANCELLED (cancelled). REJECTED and CANCELLED are terminal states of this revision; any later change is handled as a new revision | `DRAFT` |
| 383 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow of this revision. Reviewers, approvers and signatures are looked up from the linked workflow / processing history | `00000000-0000-0000-0000-000000000001` |
| 384 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 385 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 386 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 387 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 388 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from the approval status | `ACTIVE` |
| 389 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the disposal approval was completed | `2026-09-01T00:00:00Z` |
| 390 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 391 | Disposal Approval Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_vp_plan_1` | UNIQUE | `vp_key, revision_number` | UNIQUE (vp_key, revision_number) |
| `ck_vp_plan_1_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_vp_plan_2` | UNIQUE | `vp_key, is_current_version` | UNIQUE (vp_key) WHERE is_current_version = TRUE |
| `ck_vp_plan_3` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_vp_plan_4` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_vp_plan_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `rule_vp_plan_5` | Business validation | `workflow_instance_id` | There is one VP logical key per project, with multiple revisions. APPROVED requires workflow_instance_id. |
| `ck_vp_plan_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

Each row is a specific revision. The approved row as it stood before modification is preserved; a revision is made by issuing a new PK while keeping the logical key. is_current_version indicates whether the row is the current revision, which differs from whether it is the latest approved version. Referencing FKs and electronic signatures point to the exact revision row PK and version.

Section additions, deletions, ordering, inclusion flags and body are stored in vp_section. Edits to the generated deliverable and the document approval status are managed in deliverable_revision and are kept separate from the source VP.

All revisions of the same vp_key belong to the same project. Revision creation and switching of the current revision are processed in the same transaction with the project and the logical item locked.

[↑ Back to top](#top)

---

<a id="table-vp_section"></a>
### 28. Validation Plan Section (`vp_section`)

| Item | Definition |
|---|---|
| Description | Section title, content, display order and inclusion flag per VP revision. |
| Primary Key | `vp_section_id` |
| Main References (FK) | `vp_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 392 | Section ID | `vp_section_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 393 | VP Revision ID | `vp_id` | `uuid` | N | Y | `vp_plan.vp_id` | Y | - | N | Y | N | Y | References vp_plan.vp_id | `00000000-0000-0000-0000-000000000001` |
| 394 | Section Logical Key | `section_key` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Section identification key retained across revisions | `purpose` |
| 395 | Section Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Section Title | `Purpose` |
| 396 | Section Content | `content` | `text` | N | N | - | Y | - | N | N | N | Y | Section Content | `Validation purpose of this plan` |
| 397 | Display Order | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display Order | `1` |
| 398 | Included in Deliverable | `is_enabled` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Included in Deliverable | `TRUE` |
| 399 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 400 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 401 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 402 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_vp_section_1` | UNIQUE | `vp_id, section_key` | UNIQUE (vp_id, section_key) |
| `uq_vp_section_1_2` | UNIQUE | `vp_id, sort_order` | UNIQUE (vp_id, sort_order) |
| `ck_vp_section_1_3` | CHECK | `sort_order` | CHECK (sort_order >= 1) |

#### Business Rules

After submission (REVIEW), the child sections of a VP are not modified directly; if a change is needed, they are copied to a new VP revision and edited there. Content the user saves after AI draft generation is also recorded in content.

[↑ Back to top](#top)

---

## QIA

<a id="table-qia_assessment"></a>
### 29. Quality Impact Assessment Header (`qia_assessment`)

| Item | Definition |
|---|---|
| Description | Assessment header that manages the per-project QIA module list and the answers to the common Part 11 questions. |
| Primary Key | `qia_id` |
| Main References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 403 | QIA ID | `qia_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the QIA assessment | `00000000-0000-0000-0000-000000000001` |
| 404 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | Y | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 405 | Part 11 Q1 Electronic Records Replace Paper Records | `p11_q1` | `varchar(3)` | N | N | - | N | - | N | N | N | Y | Do electronic records replace paper records? Answer Yes / No. NULL while unanswered during authoring; required when the QIA document is submitted | `No` |
| 406 | Part 11 Q2 Electronic Signatures Used | `p11_q2` | `varchar(3)` | N | N | - | N | - | N | N | N | Y | Are electronic signatures used? Answer Yes / No. NULL while unanswered during authoring; required when the QIA document is submitted | `No` |
| 407 | Part 11 Q3 Record History Is Regulatory Evidence | `p11_q3` | `varchar(3)` | N | N | - | N | - | N | N | N | Y | Is the record creation/change history regulatory evidence? Answer Yes / No. NULL while unanswered during authoring; required when the QIA document is submitted | `No` |
| 408 | Part 11 Q4 Access Control / User Identification Required | `p11_q4` | `varchar(3)` | N | N | - | N | - | N | N | N | Y | Are access control and user identification required? Answer Yes / No. NULL while unanswered during authoring; required when the QIA document is submitted | `No` |
| 409 | Part 11 Q5 Long-term Retention / Retrieval Required | `p11_q5` | `varchar(3)` | N | N | - | N | - | N | N | N | Y | Is long-term retention and retrieval of records required? Answer Yes / No. NULL while unanswered during authoring; required when the QIA document is submitted | `No` |
| 410 | Part 11 Q6 Electronic Record Transfer Between Systems | `p11_q6` | `varchar(3)` | N | N | - | N | - | N | N | N | Y | Are electronic records transferred between systems? Answer Yes / No. NULL while unanswered during authoring; required when the QIA document is submitted | `No` |
| 411 | Part11 Assessment Conclusion | `part11_result` | `text` | N | N | - | N | - | N | N | N | N | Display value calculated automatically from the 6 answers. NULL (not evaluated) if any question is unanswered; when all are answered, applicable if at least one is Yes and not applicable if all are No. Not entered directly | `Part 11 not applicable` |
| 412 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 413 | Question Set Version | `question_set_version` | `varchar(50)` | N | N | - | Y | `'UI-P11-1'` | N | N | N | Y | Version that fixes the wording and order of the 6 questions | `UI-P11-1` |
| 414 | Judgment Rule Version | `rule_version` | `varchar(50)` | N | N | - | Y | `'UI-P11-1'` | N | N | N | Y | Version of the rule that judges "applicable" if one or more of the answers are Yes | `UI-P11-1` |
| 415 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 416 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 417 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 418 | Applied Questions Original Content | `question_set_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,questions:[{question_id,field_name,text,sort_order,answer_options}]}. Stores all questions, wording, order and answer options of the selected version. | - |
| 419 | Applied Judgment Rule Original Content | `rule_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,input_fields,normalization,expression,result_mapping}. Full definition including the logical expression, answer interpretation and judgment value mapping | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_qia_assessment_1` | UNIQUE | `project_id` | UNIQUE (project_id) |
| `rule_qia_assessment_2` | Business validation | `part11_result` | part11_result is a cache recalculated in the same transaction as the answers; modifying it independently is prohibited. It is NULL (not evaluated) if any question is unanswered. |
| `ck_qia_assessment_3` | CHECK | `p11_q1, p11_q2, p11_q3, p11_q4, p11_q5, p11_q6` | CHECK ((p11_q1 IS NULL OR p11_q1 IN ('Yes','No')) AND (p11_q2 IS NULL OR p11_q2 IN ('Yes','No')) AND (p11_q3 IS NULL OR p11_q3 IN ('Yes','No')) AND (p11_q4 IS NULL OR p11_q4 IN ('Yes','No')) AND (p11_q5 IS NULL OR p11_q5 IN ('Yes','No')) AND (p11_q6 IS NULL OR p11_q6 IN ('Yes','No'))) |
| `rule_qia_assessment_4` | Business validation | `p11_q1, p11_q2, p11_q3, p11_q4, p11_q5, p11_q6` | When the QIA document (deliverable_revision) is submitted, p11_q1~p11_q6 must be answered, every module included in the document must have at least 1 process, and q1_val~q10_val of every process must be answered. Submission is refused if any question is unanswered. |

#### Business Rules

The Part 11 questions are answered in common for the QIA, not per module. When the QIA document is created or approved, the question version, answers, rule version and result are fixed in the source/table snapshot of deliverable_revision. Changing the current answers does not alter previously approved documents.

The approval status and version of the document are managed in deliverable_document and deliverable_revision. The GxP judgment of each module is looked up from its child processes.

When an assessment is created, the question and rule definitions for the version are copied, and the server validates that their version values match question_set_version and rule_version. A different definition is not registered under the same version name. A new global version is not automatically applied to existing assessments. Calculations use the stored definitions; a bare function name or version string alone does not replace the original content.

Even if the common Part 11 answers or the applied definitions change, the evaluation_definition_snapshot of documents already submitted or approved is not changed. When the QIA document is submitted, the full questions, rules, answers and judgments of the common assessment and of all included processes are fixed together in evaluation_definition_snapshot.

[↑ Back to top](#top)

---

<a id="table-qia_module_item"></a>
### 30. QIA Module Detailed Assessment (`qia_module_item`)

| Item | Definition |
|---|---|
| Description | QIA module name and description, and per-module approval, disposal and revision management. |
| Primary Key | `qia_module_item_id` |
| Main References (FK) | `qia_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 420 | QIA Module Item ID | `qia_module_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the QIA module assessment item | `00000000-0000-0000-0000-000000000001` |
| 421 | Cross-revision Logical ID | `module_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision row PK | `00000000-0000-0000-0000-000000000001` |
| 422 | QIA ID | `qia_id` | `uuid` | N | Y | `qia_assessment.qia_id` | Y | - | N | Y | N | Y | Identifier of the parent QIA assessment | `00000000-0000-0000-0000-000000000001` |
| 423 | Module Code | `module_code` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | Optional internal module code. Distinct from the on-screen approval number item_number | `MOD-001` |
| 424 | Module Name | `module_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Top-level module name | `Quality Management` |
| 425 | Module Description | `module_description` | `text` | N | N | - | N | - | N | N | N | Y | Description of the scope and purpose of the module | `Overall assessment related to quality management` |
| 426 | Assessment Result | `result_type` | `varchar(20)` | N | N | - | N | - | N | N | N | N | Aggregation cache of the process judgments. NULL (not evaluated) if there are no processes or any process has an unanswered question; otherwise GXP if at least one process is GXP, else NON_GXP. Not edited directly | `GXP` |
| 427 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T10:00:00Z` |
| 428 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T10:00:00Z` |
| 429 | Module Display Order | `sort_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Module Display Order | `1` |
| 430 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Version shown on screen for this revision | `ver1` |
| 431 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 per logical key | `1` |
| 432 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising. Required when revision_number ≥ 2 | `Requirement change` |
| 433 | Is Current Revision | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Current revision of the same logical key. Distinct from whether it is the latest approved version | `TRUE` |
| 434 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress) / REVIEW (under review) / APPROVAL (under approval) / APPROVED (approved) / REJECTED (rejected) / CANCELLED (cancelled). REJECTED and CANCELLED are terminal states of this revision; any later change is handled as a new revision | `DRAFT` |
| 435 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow of this revision. Reviewers, approvers and signatures are looked up from the linked workflow / processing history | `00000000-0000-0000-0000-000000000001` |
| 436 | Item Approval Number | `item_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Assigned at first approval. NULL for drafts; the same number is kept across revisions | `QIA-001` |
| 437 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 438 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 439 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from the approval status | `ACTIVE` |
| 440 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the disposal approval was completed | `2026-09-01T00:00:00Z` |
| 441 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 442 | Disposal Approval Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_qia_module_item_1` | Business validation | `workflow_instance_id` | APPROVED requires item_number and workflow_instance_id. The same approval number is not assigned to different logical keys within the same project. |
| `uq_qia_module_item_2` | UNIQUE | `module_key, revision_number` | UNIQUE (module_key, revision_number) |
| `ck_qia_module_item_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_qia_module_item_3` | UNIQUE | `module_key, is_current_version` | UNIQUE (module_key) WHERE is_current_version = TRUE |
| `ck_qia_module_item_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_qia_module_item_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_qia_module_item_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_qia_module_item_6` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_qia_module_item_6` | Business validation | `sort_order` | Because a module can be registered first and processes added later, a draft with 0 child processes is allowed. Submission (REVIEW) requires at least 1 child process |
| `rule_qia_module_item_7` | Business validation | `status` | When a module revision is submitted (REVIEW), it must have at least 1 child process and q1_val~q10_val of every process must be answered. Submission is refused if any question is unanswered. |
| `ck_qia_module_item_result_type` | CHECK | `result_type` | CHECK (result_type IS NULL OR result_type IN ('GXP','NON_GXP')) |
| `ck_qia_module_item_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

Each row is a specific revision. The approved row as it stood before modification is preserved; a revision is made by issuing a new PK while keeping the logical key. is_current_version indicates whether the row is the current revision, which differs from whether it is the latest approved version. Referencing FKs and electronic signatures point to the exact revision row PK and version.

A module with no processes is not judged (result_type NULL, not evaluated) and cannot be submitted. This ensures that an unassessed module is not approved as NON_GXP.

Modules and processes have a 1:N relationship; process names and the answers to the 10 questions are stored in qia_process. On revision, the processes are also duplicated under the new module revision, and the original process content and answers of an approved module are immutable.

All revisions of the same module_key belong to the same project. The item number item_number, once first assigned, is kept in subsequent revisions. NULL is allowed for unapproved drafts of items whose number is assigned on approval. The number is not reused for a different logical key in the same project and same table, and is not reassigned to another item even after disposal. Number issuance and revision creation are serialized with a concurrency lock on the project and the logical item, and the duplication, project-membership and number-retention conditions are validated in the same transaction.

[↑ Back to top](#top)

---

<a id="table-qia_process"></a>
### 31. QIA Process Assessment (`qia_process`)

| Item | Definition |
|---|---|
| Description | Process name and description under a module revision, and the answers to the 10 GxP questions. |
| Primary Key | `qia_process_id` |
| Main References (FK) | `qia_module_item_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 443 | Process ID | `qia_process_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 444 | Module Revision ID | `qia_module_item_id` | `uuid` | N | Y | `qia_module_item.qia_module_item_id` | Y | - | N | Y | N | Y | References qia_module_item.qia_module_item_id | `00000000-0000-0000-0000-000000000001` |
| 445 | Process Logical ID | `process_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | N | N | Y | Process identifier retained across module revisions | `00000000-0000-0000-0000-000000000001` |
| 446 | Internal Process Code | `process_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Internal Process Code | - |
| 447 | Process Name | `process_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Process Name | `Test result entry` |
| 448 | Process Description | `process_description` | `text` | N | N | - | N | - | N | N | N | Y | Process Description | - |
| 449 | Display Order | `sort_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Display Order | `1` |
| 450 | Question Set Version | `question_set_version` | `varchar(50)` | N | N | - | Y | `'UI-GXP-1'` | N | N | N | Y | Question Set Version | `UI-GXP-1` |
| 451 | Judgment Rule Version | `rule_version` | `varchar(50)` | N | N | - | Y | `'UI-GXP-1'` | N | N | N | Y | Judgment Rule Version | `UI-GXP-1` |
| 452 | GxP Q1 | `q1_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | Does it directly affect product quality, patient safety or data integrity? Answer O / X / ▲. NULL while unanswered during authoring; required when the module is submitted | `X` |
| 453 | GxP Q2 | `q2_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | Does it create or process data used for GxP decision-making? Answer O / X / ▲. NULL while unanswered during authoring; required when the module is submitted | `X` |
| 454 | GxP Q3 | `q3_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | Does it support batch release or quality approval procedures? Answer O / X / ▲. NULL while unanswered during authoring; required when the module is submitted | `X` |
| 455 | GxP Q4 | `q4_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | Is it used for regulatory submissions or as inspection evidence? Answer O / X / ▲. NULL while unanswered during authoring; required when the module is submitted | `X` |
| 456 | GxP Q5 | `q5_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | Does it create, modify or retain electronic records? Answer O / X / ▲. NULL while unanswered during authoring; required when the module is submitted | `X` |
| 457 | GxP Q6 | `q6_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | Does it control critical parameters of equipment or processes? Answer O / X / ▲. NULL while unanswered during authoring; required when the module is submitted | `X` |
| 458 | GxP Q7 | `q7_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | Does it support deviation, CAPA or change control processes? Answer O / X / ▲. NULL while unanswered during authoring; required when the module is submitted | `X` |
| 459 | GxP Q8 | `q8_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | Do user permissions control access to quality-related functions? Answer O / X / ▲. NULL while unanswered during authoring; required when the module is submitted | `X` |
| 460 | GxP Q9 | `q9_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | Does it exchange critical data with external GxP systems? Answer O / X / ▲. NULL while unanswered during authoring; required when the module is submitted | `X` |
| 461 | GxP Q10 | `q10_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | Does a backup/restore failure affect GxP records? Answer O / X / ▲. NULL while unanswered during authoring; required when the module is submitted | `X` |
| 462 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 463 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 464 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 465 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 466 | Applied Questions Original Content | `question_set_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,questions:[{question_id,field_name,text,sort_order,answer_options}]}. Stores all questions, wording, order and answer options of the selected version. | - |
| 467 | Applied Judgment Rule Original Content | `rule_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,input_fields,normalization,expression,result_mapping}. Full definition including the logical expression, answer interpretation and judgment value mapping | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_qia_process_1` | UNIQUE | `qia_module_item_id, process_key` | UNIQUE (qia_module_item_id, process_key) |
| `uq_qia_process_1_2` | UNIQUE | `qia_module_item_id, sort_order` | UNIQUE (qia_module_item_id, sort_order) |
| `ck_qia_process_1_3` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_qia_process_2` | Business validation | `q1_val` | q1_val~q10_val allow NULL (unanswered) during authoring and must all be answered when the module is submitted (rule_qia_module_item_7). |
| `ck_qia_process_3` | CHECK | `q1_val, q2_val, q3_val, q4_val, q5_val, q6_val, q7_val, q8_val, q9_val, q10_val` | CHECK ((q1_val IS NULL OR q1_val IN ('O','X','▲')) AND (q2_val IS NULL OR q2_val IN ('O','X','▲')) AND (q3_val IS NULL OR q3_val IN ('O','X','▲')) AND (q4_val IS NULL OR q4_val IN ('O','X','▲')) AND (q5_val IS NULL OR q5_val IN ('O','X','▲')) AND (q6_val IS NULL OR q6_val IN ('O','X','▲')) AND (q7_val IS NULL OR q7_val IN ('O','X','▲')) AND (q8_val IS NULL OR q8_val IN ('O','X','▲')) AND (q9_val IS NULL OR q9_val IN ('O','X','▲')) AND (q10_val IS NULL OR q10_val IN ('O','X','▲'))) |

#### Business Rules

GxP judgment rule: GXP if Q1=O and at least one of Q2~Q10 is O; otherwise NON_GXP. ▲ does not count toward the O condition. The judgment is calculated from the answers at query time. If any question is unanswered (NULL), no judgment is made and the result is left as not evaluated (NULL).

After submission (REVIEW), the question version and answers belonging to the module are not overwritten directly.

When an assessment is created, the question and rule definitions for the version are copied, and the server validates that their version values match question_set_version and rule_version. A different definition is not registered under the same version name. A new global version is not automatically applied to existing assessments. Calculations use the stored definitions; a bare function name or version string alone does not replace the original content.

To change the questions, rules or answers of an approved module, create a new revision of the module and copy the processes and the applied definitions. The original content belonging to past modules is preserved as is.

[↑ Back to top](#top)

---

## VA

<a id="table-vendor_audit"></a>
### 32. Vendor Audit Assessment (`vendor_audit`)

| Item | Definition |
|---|---|
| Description | Vendor name, audit date, auditor and attachment, and per-assessment approval and revision management. |
| Primary Key | `audit_id` |
| Main References (FK) | `project_id, auditor_user_id, file_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 468 | Audit ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the vendor audit | `00000000-0000-0000-0000-000000000001` |
| 469 | Cross-revision Logical ID | `audit_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision row PK | `00000000-0000-0000-0000-000000000001` |
| 470 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 471 | Document Number | `document_number` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | Assigned at first approval. NULL for drafts; revisions of the same logical key keep the number | `VA-001` |
| 472 | Vendor Name | `vendor_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Name of the vendor being audited | `Sample Vendor` |
| 473 | Audit Date | `audit_date` | `date` | N | N | - | Y | - | N | N | N | Y | Date the audit was actually performed. Entered by the author; on revision, the value from the previous revision is copied | `2026-09-16` |
| 474 | Auditor | `auditor_name` | `varchar(100)` | N | N | - | Y | - | N | N | Y | Y | Display name of the auditor who performed the audit, as of the time of entry. The history is preserved even if the user name changes; on revision, the value from the previous revision is copied | `Gildong Hong · reviewer@example.test` |
| 475 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Version shown on screen for this revision | `ver1` |
| 476 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 per logical key | `1` |
| 477 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising. Required when revision_number ≥ 2 | `Requirement change` |
| 478 | Is Current Revision | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Current revision of the same logical key. Distinct from whether it is the latest approved version | `TRUE` |
| 479 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress) / REVIEW (under review) / APPROVAL (under approval) / APPROVED (approved) / REJECTED (rejected) / CANCELLED (cancelled). REJECTED and CANCELLED are terminal states of this revision; any later change is handled as a new revision | `DRAFT` |
| 480 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 481 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 482 | Auditor Account ID | `auditor_user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Account of the auditor who performed the audit. Selected by the author; on revision, the value from the previous revision is copied | `00000000-0000-0000-0000-000000000001` |
| 483 | Assessment Attachment ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | Optional uploaded attachment. File name, size and path are looked up from the file asset | `00000000-0000-0000-0000-000000000001` |
| 484 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow of this revision. Reviewers, approvers and signatures are looked up from the linked workflow / processing history | `00000000-0000-0000-0000-000000000001` |
| 485 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 486 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 487 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from the approval status | `ACTIVE` |
| 488 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the disposal approval was completed | `2026-09-01T00:00:00Z` |
| 489 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 490 | Disposal Approval Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_vendor_audit_1` | Business validation | `workflow_instance_id` | APPROVED requires document_number and workflow_instance_id. The same approval number is not assigned to different logical keys within the same project. |
| `uq_vendor_audit_2` | UNIQUE | `audit_key, revision_number` | UNIQUE (audit_key, revision_number) |
| `ck_vendor_audit_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_vendor_audit_3` | UNIQUE | `audit_key, is_current_version` | UNIQUE (audit_key) WHERE is_current_version = TRUE |
| `ck_vendor_audit_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_vendor_audit_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_vendor_audit_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_vendor_audit_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

Each row is a specific revision. The approved row as it stood before modification is preserved; a revision is made by issuing a new PK while keeping the logical key. is_current_version indicates whether the row is the current revision, which differs from whether it is the latest approved version. Referencing FKs and electronic signatures point to the exact revision row PK and version.

The system name is looked up from the system linked to the project. The attachment is optional and NULL is allowed.

All revisions of the same audit_key belong to the same project. The item number document_number, once first assigned, is kept in subsequent revisions. NULL is allowed for unapproved drafts of items whose number is assigned on approval. The number is not reused for a different logical key in the same project and same table, and is not reassigned to another item even after disposal. Number issuance and revision creation are serialized with a concurrency lock on the project and the logical item, and the duplication, project-membership and number-retention conditions are validated in the same transaction.

[↑ Back to top](#top)

---

## URS

<a id="table-requirement"></a>
### 33. User Requirements Specification (`requirement`)

| Item | Definition |
|---|---|
| Description | Category, item name, content, acceptance criteria, regulatory basis and registration source of on-screen requirements, and approval revision management. |
| Primary Key | `requirement_id` |
| Main References (FK) | `project_id, created_by, workflow_instance_id, updated_by, disposal_workflow_id, source_library_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 491 | Requirement ID | `requirement_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the requirement | `00000000-0000-0000-0000-000000000001` |
| 492 | Cross-revision Logical ID | `requirement_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision row PK | `00000000-0000-0000-0000-000000000001` |
| 493 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 494 | Item Number | `item_number` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Assigned at first approval. NULL for drafts; revisions of the same logical key keep the number | `URS-001` |
| 495 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | URS major category (same basis as library_item.category). E.g. system administration, audit trail, electronic signature | `Electronic signature` |
| 496 | Middle Category | `middle_category` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | URS middle category (same basis as library_item.middle_category). When registered from a library or system package, the source value is copied | - |
| 497 | Target | `target_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | URS target (same basis as library_item.target_type). When registered from a library or system package, the source value is copied | - |
| 498 | Item / Function | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Requirement item name and main function | `Record e-signature signer, date/time and meaning` |
| 499 | Requirement Details | `requirement_text` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed specification of the requirement | `On electronic signature, the signer ID, signing date/time...` |
| 500 | Acceptance Criteria | `acceptance_criteria` | `text` | N | N | - | N | - | N | N | N | Y | Acceptance criteria entered on screen | `Access is restricted according to the defined permissions` |
| 501 | Regulatory Basis | `regulation` | `text` | N | N | - | N | - | N | N | N | Y | Manually entered regulatory basis, or original text not linked to the master. Formal clause links are managed in requirement_regulation | `CSV regulatory basis review note` |
| 502 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress) / REVIEW (under review) / APPROVAL (under approval) / APPROVED (approved) / REJECTED (rejected) / CANCELLED (cancelled). REJECTED and CANCELLED are terminal states of this revision; any later change is handled as a new revision | `DRAFT` |
| 503 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Version shown on screen for this revision | `ver1` |
| 504 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 per logical key | `1` |
| 505 | Is Current Revision | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Current revision of the same logical key. Distinct from whether it is the latest approved version | `TRUE` |
| 506 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising. Required when revision_number ≥ 2 | `Requirement change` |
| 507 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the requirement author | `00000000-0000-0000-0000-000000000001` |
| 508 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 509 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 510 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow of this revision. Reviewers, approvers and signatures are looked up from the linked workflow / processing history | `00000000-0000-0000-0000-000000000001` |
| 511 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 512 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from the approval status | `ACTIVE` |
| 513 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the disposal approval was completed | `2026-09-01T00:00:00Z` |
| 514 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 515 | Disposal Approval Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |
| 516 | Registration Source | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL / LIBRARY / SYSTEM_PACKAGE / AI_DRAFT | `MANUAL` |
| 517 | Source Library ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | References library_item.library_id | `00000000-0000-0000-0000-000000000001` |
| 518 | Other Source Identifier | `source_reference` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Identifier of the system package or AI draft. Not a DB FK because the source is external/polymorphic | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_requirement_1` | Business validation | `workflow_instance_id` | APPROVED requires item_number and workflow_instance_id. The same approval number is not assigned to different logical keys within the same project. |
| `uq_requirement_2` | UNIQUE | `requirement_key, revision_number` | UNIQUE (requirement_key, revision_number) |
| `ck_requirement_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_requirement_3` | UNIQUE | `requirement_key, is_current_version` | UNIQUE (requirement_key) WHERE is_current_version = TRUE |
| `ck_requirement_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_requirement_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_requirement_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_requirement_6` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `rule_requirement_6` | Business validation | `source_type, source_library_id` | source_library_id is required when LIBRARY |
| `rule_requirement_7` | Business validation | `category, middle_category, target_type` | While in progress (DRAFT), NULL is allowed for middle_category and target_type; on submission (REVIEW), the major category (category), middle category (middle_category) and target (target_type) must all be entered. |
| `ck_requirement_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

Each row is a specific revision. The approved row as it stood before modification is preserved; a revision is made by issuing a new PK while keeping the logical key. is_current_version indicates whether the row is the current revision, which differs from whether it is the latest approved version. Referencing FKs and electronic signatures point to the exact revision row PK and version.

Content imported from a library, system package or AI is copied into a draft of this item and edited there. Changes to the source are not automatically reflected in approved business items.

The URS classification is major category (category), middle category (middle_category) and target (target_type), using the same basis as library_item. When registered from a library or system package, the three values are copied together, and system package grouping is also based on these three values.

requirement_id is the PK of a specific revision row. DQ/FRA/tests/regulations are linked to that revision row. For each requirement_key, the latest draft and the latest approved revision are distinguished.

Regulation clauses are linked via requirement_regulation, and the citations are also copied when a revision is duplicated. The link status in the RTM is looked up from the business relationships and judgment results.

All revisions of the same requirement_key belong to the same project. The item number item_number, once first assigned, is kept in subsequent revisions. NULL is allowed for unapproved drafts of items whose number is assigned on approval. The number is not reused for a different logical key in the same project and same table, and is not reassigned to another item even after disposal. Number issuance and revision creation are serialized with a concurrency lock on the project and the logical item, and the duplication, project-membership and number-retention conditions are validated in the same transaction.

[↑ Back to top](#top)

---

## FDS

<a id="table-fds_spec"></a>
### 34. Functional Design Specification (`fds_spec`)

| Item | Definition |
|---|---|
| Description | Uploaded FDS documents of the F&DS group and their file revision and approval history. Multiple documents are allowed per project. |
| Primary Key | `fds_id` |
| Main References (FK) | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 519 | FDS ID | `fds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FDS identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 520 | Cross-revision Logical ID | `fds_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision row PK | `00000000-0000-0000-0000-000000000001` |
| 521 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Associated Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 522 | FDS Number | `fds_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Assigned at first approval. NULL for drafts; revisions of the same logical key keep the number | `FDS-001` |
| 523 | FDS Document Name | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | The uploaded file name is used as the display title of the design document | `FDS_ver1.pdf` |
| 524 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Version shown on screen for this revision | `ver1` |
| 525 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 per logical key | `1` |
| 526 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising. Required when revision_number ≥ 2 | `Requirement change` |
| 527 | Is Current Revision | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Current revision of the same logical key. Distinct from whether it is the latest approved version | `TRUE` |
| 528 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress) / REVIEW (under review) / APPROVAL (under approval) / APPROVED (approved) / REJECTED (rejected) / CANCELLED (cancelled). REJECTED and CANCELLED are terminal states of this revision; any later change is handled as a new revision | `DRAFT` |
| 529 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 530 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the FDS author | `00000000-0000-0000-0000-000000000001` |
| 531 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 532 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the user who last updated the FDS | `00000000-0000-0000-0000-000000000001` |
| 533 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deletion time of an unapproved draft. Items with approval history go through disposal approval instead of deletion | `2026-09-01T00:00:00Z` |
| 534 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow of this revision. Reviewers, approvers and signatures are looked up from the linked workflow / processing history | `00000000-0000-0000-0000-000000000001` |
| 535 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from the approval status | `ACTIVE` |
| 536 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the disposal approval was completed | `2026-09-01T00:00:00Z` |
| 537 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 538 | Disposal Approval Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |
| 539 | Uploaded File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | File uploaded for this design revision. Replacing it creates a new revision row and a new file ID. A draft (DRAFT) created before the file scan may be NULL; it is filled per file_link_request (PRIMARY_FILE) when the scan passes. Required unless DRAFT (ck_fds_spec_file_required) | `00000000-0000-0000-0000-000000000001` |
| 540 | Uploaded At | `uploaded_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time this file revision was uploaded | `2026-09-01T00:00:00Z` |
| 541 | Uploader ID | `uploaded_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_fds_spec_1` | Business validation | `workflow_instance_id` | APPROVED requires fds_no and workflow_instance_id. The same approval number is not assigned to different logical keys within the same project. |
| `uq_fds_spec_2` | UNIQUE | `fds_key, revision_number` | UNIQUE (fds_key, revision_number) |
| `ck_fds_spec_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_fds_spec_3` | UNIQUE | `fds_key, is_current_version` | UNIQUE (fds_key) WHERE is_current_version = TRUE |
| `ck_fds_spec_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_fds_spec_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_fds_spec_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_fds_spec_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |
| `ck_fds_spec_file_required` | CHECK | `status, file_id` | CHECK (status = 'DRAFT' OR file_id IS NOT NULL) |

#### Business Rules

Each row is a specific revision. The approved row as it stood before modification is preserved; a revision is made by issuing a new PK while keeping the logical key. is_current_version indicates whether the row is the current revision, which differs from whether it is the latest approved version. Referencing FKs and electronic signatures point to the exact revision row PK and version.

F&DS is a single activity, and the display order of the document types is FDS followed by DDS. Prior FDS approval or a single parent FDS is not enforced as mandatory for DDS.

File replacement history is looked up from the previous revision rows of the same logical key. DQ references the PK of this table as the exact approved file revision.

A draft can be created right after upload, even before the file scan finishes; when the scan passes, the Worker fills file_id according to file_link_request. When a draft is deleted (deleted_at), any not-yet-linked file link requests targeting that draft are changed to CANCELLED in the same transaction (rule_file_link_request_3).

All revisions of the same fds_key belong to the same project. The item number fds_no, once first assigned, is kept in subsequent revisions. NULL is allowed for unapproved drafts of items whose number is assigned on approval. The number is not reused for a different logical key in the same project and same table, and is not reassigned to another item even after disposal. Number issuance and revision creation are serialized with a concurrency lock on the project and the logical item, and the duplication, project-membership and number-retention conditions are validated in the same transaction.

[↑ Back to top](#top)

---

## DDS

<a id="table-dds_spec"></a>
### 35. Detailed Design Specification (`dds_spec`)

| Item | Definition |
|---|---|
| Description | Uploaded DDS documents of the F&DS group and their file revision and approval history. Multiple documents are allowed per project. |
| Primary Key | `dds_id` |
| Main References (FK) | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 542 | DDS ID | `dds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the DDS document | `00000000-0000-0000-0000-000000000001` |
| 543 | Cross-revision Logical ID | `dds_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision row PK | `00000000-0000-0000-0000-000000000001` |
| 544 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Validation project to which the DDS belongs | `00000000-0000-0000-0000-000000000001` |
| 545 | DDS Number | `dds_no` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Assigned at first approval. NULL for drafts; revisions of the same logical key keep the number | `DDS-001` |
| 546 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | The uploaded file name is used as the display title of the design document | `DDS_ver1.pdf` |
| 547 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Version shown on screen for this revision | `ver1` |
| 548 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 per logical key | `1` |
| 549 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising. Required when revision_number ≥ 2 | `Requirement change` |
| 550 | Is Current Revision | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Current revision of the same logical key. Distinct from whether it is the latest approved version | `TRUE` |
| 551 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress) / REVIEW (under review) / APPROVAL (under approval) / APPROVED (approved) / REJECTED (rejected) / CANCELLED (cancelled). REJECTED and CANCELLED are terminal states of this revision; any later change is handled as a new revision | `DRAFT` |
| 552 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS creation time (UTC) | `2026-09-01T10:00:00Z` |
| 553 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who authored the DDS | `00000000-0000-0000-0000-000000000001` |
| 554 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS last update time (UTC) | `2026-09-01T10:00:00Z` |
| 555 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last updated the DDS | `00000000-0000-0000-0000-000000000001` |
| 556 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deletion time of an unapproved draft. Items with approval history go through disposal approval instead of deletion | `2026-09-02T10:00:00Z` |
| 557 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow of this revision. Reviewers, approvers and signatures are looked up from the linked workflow / processing history | `00000000-0000-0000-0000-000000000001` |
| 558 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from the approval status | `ACTIVE` |
| 559 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the disposal approval was completed | `2026-09-02T10:00:00Z` |
| 560 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 561 | Disposal Approval Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |
| 562 | Uploaded File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | File uploaded for this design revision. Replacing it creates a new revision row and a new file ID. May be NULL for a draft (DRAFT) created before the file scan; when the scan passes, it is filled according to file_link_request (PRIMARY_FILE). Required unless DRAFT (ck_dds_spec_file_required) | `00000000-0000-0000-0000-000000000001` |
| 563 | Uploaded At | `uploaded_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time this file revision was uploaded | `2026-09-01T00:00:00Z` |
| 564 | Uploader ID | `uploaded_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_dds_spec_1` | Business validation | `workflow_instance_id` | APPROVED requires dds_no and workflow_instance_id. The same approval number is not assigned to different logical keys within the same project. |
| `uq_dds_spec_2` | UNIQUE | `dds_key, revision_number` | UNIQUE (dds_key, revision_number) |
| `ck_dds_spec_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_dds_spec_3` | UNIQUE | `dds_key, is_current_version` | UNIQUE (dds_key) WHERE is_current_version = TRUE |
| `ck_dds_spec_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_dds_spec_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_dds_spec_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_dds_spec_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |
| `ck_dds_spec_file_required` | CHECK | `status, file_id` | CHECK (status = 'DRAFT' OR file_id IS NOT NULL) |

#### Business Rules

Each row is a specific revision. The approved row as it stood before modification is preserved; a revision is made by issuing a new PK while keeping the logical key. is_current_version indicates whether the row is the current revision, which differs from whether it is the latest approved version. Referencing FKs and electronic signatures point to the exact revision row PK and version.

F&DS is a single activity, and the display order of the document types is FDS followed by DDS. Prior FDS approval or a single parent FDS is not enforced as mandatory for DDS.

File replacement history is looked up from the previous revision rows of the same logical key. DQ references the PK of this table as the exact approved file revision.

A draft can be created right after upload, even before the file scan finishes; when the scan passes, the Worker fills file_id according to file_link_request. When a draft is deleted (deleted_at), any not-yet-linked file link requests targeting that draft are changed to CANCELLED in the same transaction (rule_file_link_request_3).

All revisions of the same dds_key belong to the same project. The item number dds_no is kept in subsequent revisions after it is first assigned. For items whose number is assigned at approval, NULL is allowed for unapproved drafts. The number is not reused for a different logical key in the same project and table, and is not reassigned to another item even after disposal. Number issuance and revision creation are serialized with a concurrency lock on the project and logical item, and the duplicate, ownership and number-retention conditions are validated in the same transaction.

[↑ Back to top](#top)

---

## DQ

<a id="table-dq_assessment"></a>
### 36. Design Qualification Assessment (`dq_assessment`)

| Item | Definition |
|---|---|
| Description | Per-project header that groups DQ assessment items. |
| Primary Key | `dq_id` |
| Main References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 565 | DQ ID | `dq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | DQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 566 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | Y | Y | N | Y | Associated Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 567 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Design qualification assessment document title | `Design Qualification Assessment (URS → FDS/DDS Mapping)` |
| 568 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 569 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DQ author identifier | `00000000-0000-0000-0000-000000000001` |
| 570 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 571 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DQ updater identifier | `00000000-0000-0000-0000-000000000001` |
| 572 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_dq_assessment_1` | UNIQUE | `project_id` | UNIQUE (project_id) |

#### Business Rules

Per-item business approval/revision is managed in the child table, and the number, version and approval status of the generated document are managed in deliverable_document/deliverable_revision. The header's approval status is not bulk-copied to all child items.

[↑ Back to top](#top)

---

<a id="table-dq_item"></a>
### 37. DQ Detailed Assessment Item (`dq_item`)

| Item | Definition |
|---|---|
| Description | Manages the comments, judgment, Fail reason, execution and item approval revisions from comparing the approved URS against the FDS/DDS file revisions. |
| Primary Key | `dq_item_id` |
| Main References (FK) | `dq_id, requirement_id, created_by, updated_by, fds_revision_id, dds_revision_id, executed_by, workflow_instance_id, disposal_workflow_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 573 | DQ Item ID | `dq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | DQ detail item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 574 | Cross-revision Logical ID | `dq_item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision row PK | `00000000-0000-0000-0000-000000000001` |
| 575 | DQ ID | `dq_id` | `uuid` | N | Y | `dq_assessment.dq_id` | Y | - | N | Y | N | Y | Parent DQ document identifier | `00000000-0000-0000-0000-000000000001` |
| 576 | URS Item ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | Y | N | Y | Identifier of the URS being mapped (FK) | `00000000-0000-0000-0000-000000000001` |
| 577 | URS Number | `urs_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Snapshot of the number of the approved revision referenced by requirement_id. Not entered directly | `URS-001` |
| 578 | URS Requirement | `urs_description` | `text` | N | N | - | Y | - | N | N | N | Y | Snapshot of the requirement content of the approved revision referenced by requirement_id. Not entered directly | `User login and electronic signature function` |
| 579 | Judgment | `result_status` | `varchar(10)` | N | N | - | N | - | N | N | N | Y | PASS / FAIL / NA. NA = not applicable; NULL if not yet judged | `PASS` |
| 580 | Remarks | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | Review-related notes and remarks | `Confirmed that the design is reflected in FDS and DDS` |
| 581 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 582 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item author identifier | `00000000-0000-0000-0000-000000000001` |
| 583 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 584 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item updater identifier | `00000000-0000-0000-0000-000000000001` |
| 585 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deletion time of an unapproved draft. Items with approval history go through disposal approval instead of deletion | `2026-09-01T00:00:00Z` |
| 586 | FDS Approved Revision ID | `fds_revision_id` | `uuid` | N | Y | `fds_spec.fds_id` | N | - | N | Y | N | Y | References fds_spec.fds_id | `00000000-0000-0000-0000-000000000001` |
| 587 | FDS Review Comment | `fds_comment` | `text` | N | N | - | N | - | N | N | N | Y | FDS Review Comment | - |
| 588 | DDS Approved Revision ID | `dds_revision_id` | `uuid` | N | Y | `dds_spec.dds_id` | N | - | N | Y | N | Y | References dds_spec.dds_id | `00000000-0000-0000-0000-000000000001` |
| 589 | DDS Review Comment | `dds_comment` | `text` | N | N | - | N | - | N | N | N | Y | DDS Review Comment | - |
| 590 | Fail Reason | `fail_reason` | `text` | N | N | - | N | - | N | N | N | Y | Fail Reason | - |
| 591 | Judgment Performer ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 592 | Judgment Time | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Judgment Time | `2026-09-01T00:00:00Z` |
| 593 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Version shown on screen for this revision | `ver1` |
| 594 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 per logical key | `1` |
| 595 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising. Required when revision_number ≥ 2 | `Requirement change` |
| 596 | Is Current Revision | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Current revision of the same logical key. Distinct from whether it is the latest approved version | `TRUE` |
| 597 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress) / REVIEW (under review) / APPROVAL (under approval) / APPROVED (approved) / REJECTED (rejected) / CANCELLED (cancelled). REJECTED and CANCELLED are terminal states of this revision; any later change is handled as a new revision | `DRAFT` |
| 598 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow of this revision. Reviewers, approvers and signatures are looked up from the linked workflow / processing history | `00000000-0000-0000-0000-000000000001` |
| 599 | Item Approval Number | `item_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Assigned at first approval. NULL for drafts; the same number is kept across revisions | `DQ-001` |
| 600 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from the approval status | `ACTIVE` |
| 601 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the disposal approval was completed | `2026-09-01T00:00:00Z` |
| 602 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 603 | Disposal Approval Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_dq_item_1` | Business validation | `workflow_instance_id` | APPROVED requires item_number and workflow_instance_id. The same approval number is not assigned to different logical keys within the same project. |
| `uq_dq_item_2` | UNIQUE | `dq_item_key, revision_number` | UNIQUE (dq_item_key, revision_number) |
| `ck_dq_item_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_dq_item_3` | UNIQUE | `dq_item_key, is_current_version` | UNIQUE (dq_item_key) WHERE is_current_version = TRUE |
| `ck_dq_item_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_dq_item_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_dq_item_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_dq_item_6` | CHECK | `result_status` | CHECK (result_status IS NULL OR result_status IN ('PASS','FAIL','NA')) |
| `rule_dq_item_6` | Business validation | `result_status, fail_reason` | If FAIL, a non-blank fail_reason is required |
| `rule_dq_item_7` | Business validation | `result_status, executed_by, executed_at` | On approval request, result_status, executed_by and executed_at are required, and at least one of FDS/DDS must be a valid approved revision of the same project. requirement_id must also be an approved revision of the same project. |
| `ck_dq_item_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

Each row is a specific revision. The approved row as it stood before modification is preserved; a revision is made by issuing a new PK while keeping the logical key. is_current_version indicates whether the row is the current revision, which differs from whether it is the latest approved version. Referencing FKs and electronic signatures point to the exact revision row PK and version.

fds_revision_id and dds_revision_id reference the exact revision row of the approved design document. The file name, document number and version are looked up from the linked fds_spec/dds_spec.

The performer is stored in executed_by, and reviewers/approvers are looked up from workflow_instance and approval_action. An NA (not applicable) judgment is also distinguished from an unjudged state, and the design link is checked. A DQ FAIL does not automatically create a test deviation.

All revisions of the same dq_item_key belong to the same project. The item number item_number is kept in subsequent revisions after it is first assigned. For items whose number is assigned at approval, NULL is allowed for unapproved drafts. The number is not reused for a different logical key in the same project and table, and is not reassigned to another item even after disposal. Number issuance and revision creation are serialized with a concurrency lock on the project and logical item, and the duplicate, ownership and number-retention conditions are validated in the same transaction.

[↑ Back to top](#top)

---

## FRA

<a id="table-fra_assessment"></a>
### 38. Functional Risk Assessment (`fra_assessment`)

| Item | Definition |
|---|---|
| Description | Per-project header that groups FRA assessment items. |
| Primary Key | `fra_id` |
| Main References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 604 | FRA ID | `fra_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FRA assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 605 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | Y | Y | N | Y | Associated Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 606 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the FMEA-based functional risk assessment document | `FMEA-based Functional Risk Assessment` |
| 607 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 608 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FRA author identifier | `00000000-0000-0000-0000-000000000001` |
| 609 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 610 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FRA updater identifier | `00000000-0000-0000-0000-000000000001` |
| 611 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_fra_assessment_1` | UNIQUE | `project_id` | UNIQUE (project_id) |

#### Business Rules

Per-item business approval/revision is managed in the child table, and the number, version and approval status of the generated document are managed in deliverable_document/deliverable_revision. The header's approval status is not bulk-copied to all child items.

[↑ Back to top](#top)

---

<a id="table-fra_item"></a>
### 39. FRA Risk Detail Item (`fra_item`)

| Item | Definition |
|---|---|
| Description | Manages the functions, risk scenarios, risk scores and registration sources linked to the approved URS, and the item approval revisions. |
| Primary Key | `fra_item_id` |
| Main References (FK) | `fra_id, requirement_id, created_by, updated_by, sop_clause_id, workflow_instance_id, disposal_workflow_id, source_library_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 612 | Risk Item ID | `fra_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FRA risk detail item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 613 | Cross-revision Logical ID | `fra_item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision row PK | `00000000-0000-0000-0000-000000000001` |
| 614 | FRA ID | `fra_id` | `uuid` | N | Y | `fra_assessment.fra_id` | Y | - | N | Y | N | Y | Parent FRA document identifier | `00000000-0000-0000-0000-000000000001` |
| 615 | URS Item ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | N | - | N | Y | N | Y | Approved URS revision of the same project. NULL allowed for drafts; required before approval request | `00000000-0000-0000-0000-000000000001` |
| 616 | URS Reference Number | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Display snapshot of the approval number of the linked requirement revision. Matches requirement_id and is not entered independently | `URS-001` |
| 617 | Function Name | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Name of the function subject to risk assessment | `Electronic signature` |
| 618 | Risk Scenario | `risk_scenario` | `text` | N | N | - | Y | - | N | N | N | Y | Description of the FMEA risk occurrence scenario | `Password not verified during electronic signature` |
| 619 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress) / REVIEW (under review) / APPROVAL (under approval) / APPROVED (approved) / REJECTED (rejected) / CANCELLED (cancelled). REJECTED and CANCELLED are terminal states of this revision; any later change is handled as a new revision | `DRAFT` |
| 620 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 621 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item author identifier | `00000000-0000-0000-0000-000000000001` |
| 622 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 623 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item updater identifier | `00000000-0000-0000-0000-000000000001` |
| 624 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deletion time of an unapproved draft. Items with approval history go through disposal approval instead of deletion | `2026-09-01T00:00:00Z` |
| 625 | Severity SEV | `severity` | `smallint` | N | N | - | N | - | N | N | N | Y | Entered on the screen, 1~5. NULL if not entered; required when the FRA item is submitted | `5` |
| 626 | Occurrence OCC | `occurrence` | `smallint` | N | N | - | N | - | N | N | N | Y | Entered on the screen, 1~5. NULL if not entered; required when the FRA item is submitted | `2` |
| 627 | Detectability DET | `detectability` | `varchar(1)` | N | N | - | N | - | N | N | N | Y | H / M / L. NULL if not entered; required when the FRA item is submitted | `M` |
| 628 | Judgment Rule Version | `rule_version` | `varchar(50)` | N | N | - | Y | `'UI-FRA-1'` | N | N | N | Y | Judgment Rule Version | `UI-FRA-1` |
| 629 | Calculation Result at Approval | `risk_result_snapshot` | `jsonb` | N | N | - | N | - | N | N | N | Y | Automatic RP, RC, RPG, NT and Action Plan results at the time of approval. Not entered directly | `{"RP":10,"RC":1,"RPG":"H","NT":"N","actionPlan":"Perform Test"}` |
| 630 | SOP Basis Clause ID | `sop_clause_id` | `uuid` | N | Y | `regulatory_clause.regulatory_clause_id` | N | - | N | Y | N | Y | Stores the SOP link from the RTM screen on the source risk item | `00000000-0000-0000-0000-000000000001` |
| 631 | Manual SOP Reference | `sop_reference_note` | `text` | N | N | - | N | - | N | N | N | Y | Original content of an SOP reference that is not yet in the clause master | - |
| 632 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Version shown on screen for this revision | `ver1` |
| 633 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 per logical key | `1` |
| 634 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising. Required when revision_number ≥ 2 | `Requirement change` |
| 635 | Is Current Revision | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Current revision of the same logical key. Distinct from whether it is the latest approved version | `TRUE` |
| 636 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow of this revision. Reviewers, approvers and signatures are looked up from the linked workflow / processing history | `00000000-0000-0000-0000-000000000001` |
| 637 | Item Approval Number | `item_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Assigned at first approval. NULL for drafts; the same number is kept across revisions | `FRA-001` |
| 638 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from the approval status | `ACTIVE` |
| 639 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the disposal approval was completed | `2026-09-01T00:00:00Z` |
| 640 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 641 | Disposal Approval Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |
| 642 | Registration Source | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL / LIBRARY / SYSTEM_PACKAGE / AI_DRAFT | `MANUAL` |
| 643 | Source Library ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | References library_item.library_id | `00000000-0000-0000-0000-000000000001` |
| 644 | Other Source Identifier | `source_reference` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Identifier of the system package or AI draft. Not a DB FK because the source is external/polymorphic | - |
| 645 | Applied Risk Judgment Rule (Original Content) | `rule_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,input_ranges,formulas,thresholds,matrix,action_mapping}. The complete set of SEV/OCC/DET allowed values, RP and RC formulas, RPG and NT judgment tables, and Action Plan mapping | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_fra_item_1` | Business validation | `workflow_instance_id` | APPROVED requires item_number and workflow_instance_id. The same approval number is not assigned to different logical keys within the same project. |
| `uq_fra_item_2` | UNIQUE | `fra_item_key, revision_number` | UNIQUE (fra_item_key, revision_number) |
| `ck_fra_item_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_fra_item_3` | UNIQUE | `fra_item_key, is_current_version` | UNIQUE (fra_item_key) WHERE is_current_version = TRUE |
| `ck_fra_item_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_fra_item_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_fra_item_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_fra_item_6` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `rule_fra_item_6` | Business validation | `source_type, source_library_id` | source_library_id is required when LIBRARY |
| `ck_fra_item_7` | CHECK | `severity` | CHECK (severity IS NULL OR severity BETWEEN 1 AND 5) |
| `ck_fra_item_7_2` | CHECK | `occurrence` | CHECK (occurrence IS NULL OR occurrence BETWEEN 1 AND 5) |
| `ck_fra_item_7_3` | CHECK | `detectability` | CHECK (detectability IS NULL OR detectability IN ('H','M','L')) |
| `rule_fra_item_8` | Business validation | `requirement_id, risk_result_snapshot` | requirement_id is required before approval request, and it is validated that the referenced revision is a valid approved version of the same project. APPROVED requires workflow_instance_id and risk_result_snapshot. |
| `rule_fra_item_9` | Business validation | - | sop_clause_id must be a clause of type INTERNAL_SOP. |
| `rule_fra_item_10` | Business validation | `severity, occurrence, detectability` | Missing values (NULL) are allowed while in progress (DRAFT); on submission (REVIEW), severity, occurrence and detectability must all be entered. If any value is missing, RP, RC, RPG, NT and Action Plan are not calculated, the item is shown as not evaluated, and the submission is refused. |
| `ck_fra_item_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

Each row is a specific revision. The approved row as it stood before modification is preserved; a revision is made by issuing a new PK while keeping the logical key. is_current_version indicates whether the row is the current revision, which differs from whether it is the latest approved version. Referencing FKs and electronic signatures point to the exact revision row PK and version.

Content imported from a library, system package or AI is copied into a draft of this item and edited there. Changes to the source are not automatically reflected in approved business items.

Risk assessment calculation rule: RP=SEV×OCC; RC is RP<5→3, RP<10→2, otherwise→1. RPG, by DET H/M/L, is RC1→M/H/H, RC2→L/M/H, RC3→L/L/M. NT is RP>24→Y, otherwise N. Action Plan, by RPG, is L→No Action, M→Revise/delete SOP, H→Perform Test.

Draft results are calculated at query time, and only the results at the time of approval are retained as a snapshot. Risk scores and the automatic action plan are derived from the judgment rules and are not stored as independent input values. Tests are traced through the actual test/FRA link relationships.

At authoring, the actual definition of rule_version is copied into rule_snapshot and the results are calculated with that definition. After approval, the rules, inputs and results are not changed; a change creates a new item revision. A definition with the same version name is not changed. When the FRA document is submitted, the definition, inputs and risk_result_snapshot of each basis item are fixed in evaluation_definition_snapshot.

All revisions of the same fra_item_key belong to the same project. The item number item_number is kept in subsequent revisions after it is first assigned. For items whose number is assigned at approval, NULL is allowed for unapproved drafts. The number is not reused for a different logical key in the same project and table, and is not reassigned to another item even after disposal. Number issuance and revision creation are serialized with a concurrency lock on the project and logical item, and the duplicate, ownership and number-retention conditions are validated in the same transaction.

[↑ Back to top](#top)

---

## IQ

<a id="table-iq_assessment"></a>
### 40. Installation Qualification Assessment (`iq_assessment`)

| Item | Definition |
|---|---|
| Description | Per-project IQ test set and document revision information. Distinguishes the approval status of the protocol from that of the execution results. |
| Primary Key | `iq_id` |
| Main References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 646 | IQ ID | `iq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | IQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 647 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Associated Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 648 | IQ Number | `iq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | IQ document number. Revision rows of the same document keep the same number. | `IQ-VP-SYS-010-20260529` |
| 649 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Installation qualification assessment document title | `Installation Qualification` |
| 650 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the IQ assessment document (e.g. v1.0, v1.1) | `v1.0` |
| 651 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision number of the IQ assessment document (e.g. 1, 2, 3...) | `1` |
| 652 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for the initial creation or revision of the IQ assessment document | `Initial creation` |
| 653 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active, latest version of the IQ assessment document (TRUE/FALSE) | `TRUE` |
| 654 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | IQ protocol approval summary. DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected). A display value derived from the statuses of the child rows in the composition using the aggregation rule in the business rules; it is not bulk-written to the child rows. Since the header is not subject to the approval workflow, it has no CANCELLED value. | `DRAFT` |
| 655 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | IQ execution result approval summary. DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected). A display value derived from the statuses of the child rows in the composition using the aggregation rule in the business rules; it is not bulk-written to the child rows. Since the header is not subject to the approval workflow, it has no CANCELLED value. | `DRAFT` |
| 656 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 657 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | IQ author identifier | `00000000-0000-0000-0000-000000000001` |
| 658 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 659 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | IQ updater identifier | `00000000-0000-0000-0000-000000000001` |
| 660 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time. An assessment revision cannot be deleted if its composition (item_revision_refs) contains a test revision with submission history, or if it is referenced by a submitted document or electronic signature (rule_iq_assessment_3) | `2026-09-01T00:00:00Z` |
| 661 | Test Composition per Assessment Revision | `item_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | [{table_name:"iq_item",record_id,version,sort_order}] array. Identifies the full test composition and order of this assessment revision; PKs of unchanged tests from the previous assessment revision may be reused. | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_iq_assessment_1` | UNIQUE | `project_id, iq_no, revision_number` | UNIQUE(project_id, iq_no, revision_number) |
| `rule_iq_assessment_1` | Business validation | `project_id, iq_no, revision_number, is_current_version` | At most 1 row with is_current_version = TRUE per document |
| `rule_iq_assessment_2` | Business validation | - | Document revisions are distinct from test item revisions. The approval summary on the document screen is derived from the child protocols and execution results. |
| `uq_iq_assessment_current` | UNIQUE | `project_id, iq_no` | UNIQUE (project_id, iq_no) WHERE is_current_version=TRUE |
| `rule_iq_assessment_3` | Business validation | `deleted_at` | deleted_at cannot be set if the composition (item_revision_refs) of the assessment revision contains a test revision with a submission history, or if the source_refs of a submitted deliverable_revision or an electronic signature references this assessment revision. |
| `ck_iq_assessment_protocol_status` | CHECK | `protocol_status` | CHECK (protocol_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_iq_assessment_record_status` | CHECK | `record_status` | CHECK (record_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_iq_assessment_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

Manages the test list, protocol approval, result approval and document generation information. The document body is managed in deliverable_document / deliverable_revision / deliverable_section.

When only the deliverable title, body or output format changes, only a new deliverable_revision is created, and the assessment row and test items are kept. When the assessment scope, parent business information or test composition changes, a new assessment revision row is created and the full composition at that point is fixed in item_revision_refs. Unchanged test items re-reference the same PK and version; they are not duplicated and their parent FK is not moved. A new revision row that keeps the same item_key is created only for items whose test protocol content has changed, and it is included in the new composition.

The entries of the composition are actual revisions of iq_item and must belong to the same project and IQ phase. The original parent assessment of the item must have the same iq_no as this assessment. A composition must not include different revisions of the same item_key, and duplicate record_id or sort_order values are also prohibited. The composition is the full list including the sort order and is locked when a given document is first submitted. An assessment composition referenced by a submitted document or a past signature is not modified or deleted. Rows that are not in the composition are not automatically included based solely on their original parent FK. The protocol and result approval summaries are derived from the items and executions belonging to that composition. The aggregation targets are the test revisions referenced by the composition (item_revision_refs) that have not been disposed (is_disposed=TRUE). protocol_status uses the protocol_status of each test revision; record_status uses, for each test revision, the record_status of the valid result (is_current_revision=TRUE) of the last attempt_no whose initial execution row (record_revision=1) was not aborted (ABORTED), and a test revision with no execution is treated as DRAFT. If any one is REJECTED, REJECTED is displayed; otherwise, treating CANCELLED as DRAFT, the least advanced status among DRAFT < REVIEW < APPROVAL < APPROVED is displayed. If there are no aggregation targets, the status is DRAFT.

[↑ Back to top](#top)

---

<a id="table-iq_item"></a>
### 41. IQ Detailed Test Item (`iq_item`)

| Item | Definition |
|---|---|
| Description | Manages the protocol revisions, content, expected results, acceptance criteria and registration source of IQ test items |
| Primary Key | `iq_item_id` |
| Main References (FK) | `iq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_workflow_id, disposal_signature_id, disposed_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 662 | IQ Item ID | `iq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | PK of a specific protocol revision row of an IQ test item. item_key stays the same across revisions. | `00000000-0000-0000-0000-000000000001` |
| 663 | IQ ID | `iq_id` | `uuid` | N | Y | `iq_assessment.iq_id` | Y | - | N | Y | N | Y | The parent assessment revision in which this logical test was first registered. Subsequent item revisions keep the same original parent. The actual composition of each assessment revision is looked up in item_revision_refs. | `00000000-0000-0000-0000-000000000001` |
| 664 | Test Item Logical ID | `item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier that groups multiple revisions of the same test item. Issued once for a new item and kept on revision. | `00000000-0000-0000-0000-000000000001` |
| 665 | Test ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Test display ID on the screen. Keeps the same value across revisions of the same item; duplicates between different item_keys within a project are prohibited. | `IQ-NEW-01` |
| 666 | Test Case | `test_case` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Test item title on the screen. Not a separate classification value. | `Hardware installation` |
| 667 | Test Description | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed procedure for performing the test verification | `Verify that the hardware specifications of the server to be installed meet the URS` |
| 668 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Test success criteria and expected result | `Hardware specifications must match the requirements stated in the URS` |
| 669 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected), CANCELLED (cancelled). Independent approval status of this test revision; not overwritten by the parent document status. | `DRAFT` |
| 670 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 671 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item author identifier | `00000000-0000-0000-0000-000000000001` |
| 672 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 673 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item updater identifier | `00000000-0000-0000-0000-000000000001` |
| 674 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deletion time of an unapproved draft. Only DRAFT revisions with no submission history can be deleted; items with approval history (an APPROVED revision exists for the same item_key) are handled by disposal (is_disposed, rule_iq_item_7) instead of deletion | `2026-09-01T00:00:00Z` |
| 675 | Protocol Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Protocol revision version displayed on the screen. | `ver1` |
| 676 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Revision number within the item_key. 1 or greater. | `1` |
| 677 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising an approved protocol. | - |
| 678 | Is Current Revision | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the current working revision of the same item_key. Independent of whether it is the approved version. | `TRUE` |
| 679 | Acceptance Criteria | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | Acceptance criteria on the screen. Distinct from expected_result (expected result). | `Matches the approved specification` |
| 680 | Registration Source | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL (manual entry), LIBRARY (library), SYSTEM_PACKAGE (system package), AI_DRAFT (AI draft). Same codes as requirement / fra_item.source_type | `MANUAL` |
| 681 | Source Library ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | The source when registered from the library. The test content is copied at registration and managed independently thereafter. | `00000000-0000-0000-0000-000000000001` |
| 682 | Source Package Code | `source_package_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Package code when registered from a system package. | `IQ-CORE` |
| 683 | Source Package Version | `source_package_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Package version selected at registration. | `ver2` |
| 684 | Source Template Code | `source_template_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Test template code selected within the package. | `IQ-PKG-001` |
| 685 | Item Approval Number | `approval_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Issued on first approval and displayed as the item number on the screen. The same logical item number is kept on revision. | - |
| 686 | Protocol Approval Workflow | `protocol_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Authoring/review/approval route and history for this protocol revision. | `00000000-0000-0000-0000-000000000001` |
| 687 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason entered in the disposal approval on the screen. Required when disposal is completed. | - |
| 688 | Disposal Approval Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Disposal approval workflow (approval_scope=DISPOSAL). Disposal is completed by the final approval of this approval workflow | `00000000-0000-0000-0000-000000000001` |
| 689 | Disposal Approval Signature | `disposal_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Final approval electronic signature of the disposal approval workflow (disposal_workflow_id). | `00000000-0000-0000-0000-000000000001` |
| 690 | Is Disposed | `is_disposed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Disposal status on the screen. Disposed items are excluded from new executions and aggregation; existing records are preserved. | `FALSE` |
| 691 | Disposed By | `disposed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Final approver of the disposal approval workflow | `00000000-0000-0000-0000-000000000001` |
| 692 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Final approval time of the disposal approval workflow. | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_iq_item_1` | UNIQUE | `item_key, revision_number` | UNIQUE(item_key, revision_number) |
| `rule_iq_item_1` | Business validation | `item_key, revision_number, is_current_version` | The same item_key belongs to the same project and IQ phase. At most 1 row with is_current_version = TRUE per item_key |
| `rule_iq_item_2` | Business validation | - | protocol_status is DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected), CANCELLED (cancelled). The protocol and steps are not modified directly after submission (REVIEW); modifications after rejection/cancellation and changes after approval are handled as a new revision row. |
| `rule_iq_item_3` | Business validation | - | approval_number is the first approval number of the logical item and is kept even when the revision changes. Approval and disposal signatures are preserved in the workflow_instance / approval_action / electronic_signature records that target the item revision. |
| `rule_iq_item_4` | Business validation | `source_library_id, source_package_code, source_package_version, source_template_code` | For LIBRARY registration, source_library_id is required. For SYSTEM_PACKAGE registration, source_package_code / source_package_version / source_template_code are required. |
| `rule_iq_item_5` | Business validation | - | Multiple URS links are managed as REQUIREMENT → IQ_ITEM / VERIFIED_BY in traceability_link. |
| `rule_iq_item_6` | Business validation | - | Registering an active test and approving its protocol require at least 1 linked URS revision and at least 1 detailed step with non-empty content. |
| `rule_iq_item_7` | Business validation | `disposal_reason, disposal_workflow_id, disposal_signature_id, is_disposed, disposed_by, disposed_at` | For disposal, a user with disposal permission enters a reason and submits a disposal approval workflow (approval_scope=DISPOSAL), and disposal is completed by its final approval. If is_disposed = TRUE, disposal_reason, disposal_workflow_id, disposal_signature_id, disposed_by and disposed_at are required, and disposal_signature_id / disposed_by / disposed_at must match the final approval signature, approver and time of that approval workflow. |
| `ck_iq_item_disposal` | CHECK | `is_disposed, disposal_reason, disposal_workflow_id, disposal_signature_id, disposed_by, disposed_at` | CHECK (is_disposed = FALSE OR (disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL AND disposal_signature_id IS NOT NULL AND disposed_by IS NOT NULL AND disposed_at IS NOT NULL)) |
| `rule_iq_item_8` | Business validation | - | Disposal applies to the active status of the same item_key, and past approval records are preserved. Adding the disposal status, reason and signature metadata is distinct from a revision that modifies the approved test body or steps, and the originally approved content is not changed. |
| `uq_iq_item_current` | UNIQUE | `item_key` | UNIQUE (item_key) WHERE is_current_version=TRUE |
| `rule_iq_item_9` | Business validation | `deleted_at` | deleted_at can be set only on DRAFT revisions with no submission history. If an APPROVED revision exists for the same item_key, deletion is not allowed, and discontinuing use is handled by disposal (is_disposed, rule_iq_item_7). |
| `ck_iq_item_source` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `ck_iq_item_protocol_status` | CHECK | `protocol_status` | CHECK (protocol_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_iq_item_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

The test content is stored in test_description, the expected result in expected_result and the acceptance criteria in acceptance_criteria. Repeating steps are managed in iq_step, and actual results, signatures and re-execution history in iq_execution.

All revisions of the same item_key belong to the same project. The item numbers approval_number and test_id are kept in subsequent revisions once first assigned. For items whose number is assigned on approval, an unapproved draft allows NULL. A number is not reused for a different logical key in the same project and the same table, and is not reassigned to another item even after disposal. Number issuance and revision creation are serialized with a concurrency lock on the project and the logical item, and the duplication, ownership and number-retention conditions are validated in the same transaction.

[↑ Back to top](#top)

---

<a id="table-iq_step"></a>
### 42. IQ Test Step (`iq_step`)

| Item | Definition |
|---|---|
| Description | Ordered detailed steps included in an IQ protocol revision |
| Primary Key | `step_id` |
| Main References (FK) | `iq_item_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 693 | Step ID | `step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 694 | Test Revision ID | `iq_item_id` | `uuid` | N | Y | `iq_item.iq_item_id` | Y | - | N | Y | N | Y | References iq_item.iq_item_id | `00000000-0000-0000-0000-000000000001` |
| 695 | Step Order | `step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display order of the detailed step on the screen. 1 or greater. | `1` |
| 696 | Step Content | `instruction` | `text` | N | N | - | Y | - | N | N | N | Y | Step body text that is added, edited and deleted on the screen. | - |
| 697 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 698 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 699 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 700 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_iq_step_1` | UNIQUE | `iq_item_id, step_order` | UNIQUE(iq_item_id, step_order) |
| `rule_iq_step_1` | Business validation | `iq_item_id, step_order` | step_order >= 1 |
| `rule_iq_step_2` | Business validation | - | After the parent protocol has been submitted (REVIEW), the step structure and body are not changed. Modifications after rejection/cancellation and changes after approval are recorded as a new test revision and its child steps. |

#### Business Rules

Step completion status and attached evidence are not design values of this table but per-execution-attempt records in iq_step_execution.

[↑ Back to top](#top)

---

<a id="table-iq_execution"></a>
### 43. IQ Test Execution (`iq_execution`)

| Item | Definition |
|---|---|
| Description | Manages the actual result, judgment, execution signature and result approval for each execution attempt of an IQ test revision |
| Primary Key | `execution_id` |
| Main References (FK) | `iq_item_id, executed_by, execution_signature_id, abort_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 701 | Execution ID | `execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 702 | Executed Protocol Revision ID | `iq_item_id` | `uuid` | N | Y | `iq_item.iq_item_id` | Y | - | N | Y | N | Y | References iq_item.iq_item_id | `00000000-0000-0000-0000-000000000001` |
| 703 | Execution Attempt | `attempt_no` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 1 for the first execution, 2 or greater for re-execution. The re-execution count on the screen is attempt_no - 1. | `1` |
| 704 | Result Correction Number | `record_revision` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Sequence number in the correction history of results of the same attempt. Distinct from re-execution. | `1` |
| 705 | Is Current Result Correction | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the currently valid result record for the same test revision and attempt. Result correction rows that are in progress or cancelled are FALSE; the value is switched when the correction is finalized as RECORDED. | `TRUE` |
| 706 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Actual result entered on the screen. Required when registering the judgment result. | - |
| 707 | Execution Judgment | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | PASS, FAIL, NA (not applicable). Not executed is represented as NULL; distinct from the approval status. | `PASS` |
| 708 | Execution Date | `performed_on` | `date` | N | N | - | N | - | N | N | N | Y | Execution date selected on the screen. Distinct from the electronic signature date and time. | `2026-09-16` |
| 709 | Executed By | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 710 | Result Recorded At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the judgment result was registered via electronic signature. | `2026-09-01T00:00:00Z` |
| 711 | Execution Signature ID | `execution_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature entered when registering the execution result. The password is not stored. | `00000000-0000-0000-0000-000000000001` |
| 712 | Execution Progress Status | `execution_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS (performing steps / entering results), RECORDED (judgment result registered), ABORTED (aborted without registering a result; for a result correction row, cancellation of the in-progress correction). Deviations and final approval are managed separately. | `IN_PROGRESS` |
| 713 | Result Approval Status | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected), CANCELLED (cancelled). Managed independently of the protocol approval status. A correction after rejection/cancellation is handled as a new record_revision of the same attempt_no. | `DRAFT` |
| 714 | Result Approval Workflow | `result_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Final result review and approval route for this attempt and result correction. | `00000000-0000-0000-0000-000000000001` |
| 715 | Result Approval Version | `approval_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Display version of the finally approved test result. Distinct from the protocol version. | `ver1` |
| 716 | Deviation Reason | `deviation_reason` | `text` | N | N | - | N | - | N | N | N | Y | Deviation reason as entered at the time the FAIL result was registered. | - |
| 717 | Immediate Action | `immediate_action` | `text` | N | N | - | N | - | N | N | N | Y | Immediate action as entered at the time the FAIL result was registered. | - |
| 718 | Result Correction Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for correcting the result when record_revision is incremented. | - |
| 719 | Abort Reason | `abort_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for aborting the execution without registering a result, or for cancelling an in-progress result correction (ABORTED). Required for ABORTED | - |
| 720 | Abort Signature ID | `abort_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature entered by the assignee after confirming the abort reason when aborting the execution or cancelling a result correction (signature_action=ABORT; the signature target is this execution_id). Distinct from the result registration signature (execution_signature_id); required for ABORTED | `00000000-0000-0000-0000-000000000001` |
| 721 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 722 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 723 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 724 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 725 | Validation Baseline at Execution | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | Fixes the project's current baseline at the start of execution. Required before result registration and signing; not overwritten by later changes to the project baseline. | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_iq_execution_1` | UNIQUE | `iq_item_id, attempt_no, record_revision` | UNIQUE(iq_item_id, attempt_no, record_revision) |
| `rule_iq_execution_1` | Business validation | `iq_item_id, attempt_no, record_revision, is_current_revision` | attempt_no / record_revision >= 1. At most 1 row with is_current_revision = TRUE per test revision and attempt (enforced by uq_iq_execution_current) |
| `rule_iq_execution_2` | Business validation | `actual_result, qualification_result, performed_on, executed_by, executed_at, execution_signature_id` | Execution is allowed only on an approved protocol. On transition to RECORDED, all steps must be completed, and actual_result / qualification_result / performed_on / executed_by / executed_at / execution_signature_id are required. |
| `rule_iq_execution_3` | Business validation | `qualification_result, deviation_reason, immediate_action` | If qualification_result = FAIL, deviation_reason and immediate_action are required. If there is no unclosed deviation (deviation_status<>'CLOSED') for the same logical test (item_key), a new deviation is created with source_entity_id = this execution_id. If there is an ongoing deviation (a FAIL in a re-execution under an approved action), no new deviation is created; instead, a deviation_action_round with the next round_number is created for that deviation, with failed_execution_id = this execution_id. Deviation status transitions follow rule_deviation_9. |
| `rule_iq_execution_4` | Business validation | - | A re-execution retains the existing results, step completions and evidence, and starts with a new attempt_no. If there is an unclosed deviation for the same logical test, a new attempt_no can be started only as a re-execution for that deviation, and deviation_status must be RERUN_ALLOWED and the rerun_execution_id of the current action round (current_action_round_id) must be empty. A result correction within the same attempt increments record_revision and does not overwrite previous approval records. |
| `rule_iq_execution_5` | Business validation | - | Final result approval is possible after protocol approval and execution result registration (RECORDED), and proceeds regardless of the status of the linked deviation. A FAIL result can also be approved before the deviation is closed. Result approval confirms that the judgment was recorded correctly and does not mean that deviation handling is complete. Prerequisites on the deviation side follow rule_deviation_4. |
| `ck_iq_execution_baseline` | CHECK | `execution_status, system_baseline_id` | CHECK (execution_status <> 'RECORDED' OR system_baseline_id IS NOT NULL) |
| `rule_iq_execution_6` | Business validation | `qualification_result, record_revision` | Result correction within the same attempt_no (record_revision incremented): if the pre-correction result is a FAIL already linked to a deviation or action round, no new deviation or new round is created even if the corrected result is FAIL; the existing link is kept and the judgment is looked up from the latest valid correction. If a result that was PASS or NA before correction becomes FAIL through correction, it is handled according to rule_iq_execution_3. Even if a FAIL is corrected to PASS, the linked deviation and the already-created action rounds are left as they are, and the deviation is closed through the deviation procedure, e.g. closure without re-execution (rule_deviation_5) after approval of that round. While a completion report approval workflow is in progress, re-execution results are not corrected. |
| `ck_iq_execution_result` | CHECK | `qualification_result` | CHECK (qualification_result IS NULL OR qualification_result IN ('PASS','FAIL','NA')) |
| `ck_iq_execution_record_status` | CHECK | `record_status` | CHECK (record_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `rule_iq_execution_7` | Business validation | `qualification_result` | Re-execution results for an unclosed deviation are registered only as PASS or FAIL (NA not allowed). If the test is judged not applicable during re-execution, the re-execution is aborted (rule_iq_execution_8), then the deviation is closed without re-execution (rule_deviation_5), and the rationale is recorded in the closure reason. |
| `rule_iq_execution_8` | Business validation | `execution_status, abort_reason` | If the initial execution row (record_revision=1) is IN_PROGRESS, it can be aborted (ABORTED) without registering a result. Cancellation of a result correction row (record_revision≥2) follows rule_iq_execution_9. abort_reason and an abort electronic signature (abort_signature_id, signature_action=ABORT) are required, and the user who aborted and the abort time are the signer_id and signed_at of that signature. After the abort, this row and its step executions and evidence are not modified. An ABORTED execution is not subject to result approval, and the next execution starts with a new attempt_no. If the aborted execution was a re-execution for a deviation, the rerun_execution_id links on the current action round and the deviation are cleared in the same transaction, and deviation_status remains RERUN_ALLOWED. |
| `rule_iq_execution_9` | Business validation | `record_revision, is_current_revision, execution_status` | A result correction row (record_revision≥2) can be created only when the currently valid result of the same attempt_no is RECORDED, and there can be at most 1 in-progress correction row per attempt_no. The correction row is created as IN_PROGRESS with is_current_revision=FALSE, and the previous row remains the currently valid result in the meantime. In the transaction that finalizes the correction row as RECORDED (correction signature), the correction row is set to TRUE and the previous row to FALSE, and the deviation handling of rule_iq_execution_6 and the update of deviation.rerun_execution_id are performed together. If an in-progress correction is cancelled, the correction row is closed as ABORTED (abort_reason and the abort electronic signature abort_signature_id required) and keeps is_current_revision=FALSE; the previous row, the re-execution link and the deviation status are not changed. To correct again, create the next record_revision. |
| `ck_iq_execution_status` | CHECK | `execution_status` | CHECK (execution_status IN ('IN_PROGRESS','RECORDED','ABORTED')) |
| `ck_iq_execution_abort` | CHECK | `execution_status, abort_reason, abort_signature_id, execution_signature_id` | CHECK (execution_status <> 'ABORTED' OR (abort_reason IS NOT NULL AND abort_signature_id IS NOT NULL AND execution_signature_id IS NULL)) |
| `uq_iq_execution_current` | UNIQUE | `iq_item_id, attempt_no, is_current_revision` | UNIQUE (iq_item_id, attempt_no) WHERE is_current_revision=TRUE |
| `ck_iq_execution_revision_reason` | CHECK | `record_revision, revision_reason` | CHECK (record_revision = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

Evidence for the test as a whole is linked through evidence_link with IQ_EXECUTION as the target. Electronic signatures target this execution_id, and a new signature is obtained after a result correction. Signatures for result registration and correction finalization are linked to execution_signature_id (EXECUTE), and signatures for execution abort and correction cancellation to abort_signature_id (ABORT).

system_baseline_id must belong to the same project as the test. The validity of the protocol approval and prerequisites is validated against the baseline adopted at the start of execution. A re-execution links the baseline current at that time to the new execution record, and the baselines of past executions are kept. A result correction is recorded as a new record_revision of the same logical test and attempt_no and keeps the baseline of the original execution.

[↑ Back to top](#top)

---

<a id="table-iq_step_execution"></a>
### 44. IQ Step Execution (`iq_step_execution`)

| Item | Definition |
|---|---|
| Description | Unit that records step completion status and links step evidence for an IQ execution attempt |
| Primary Key | `step_execution_id` |
| Main References (FK) | `execution_id, step_id, confirmed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 726 | Step Execution ID | `step_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 727 | Execution ID | `execution_id` | `uuid` | N | Y | `iq_execution.execution_id` | Y | - | N | Y | N | Y | References iq_execution.execution_id | `00000000-0000-0000-0000-000000000001` |
| 728 | Step ID | `step_id` | `uuid` | N | Y | `iq_step.step_id` | Y | - | N | Y | N | Y | References iq_step.step_id | `00000000-0000-0000-0000-000000000001` |
| 729 | Step Completed | `is_confirmed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Complete/cancel value of the detailed step on the screen. | `FALSE` |
| 730 | Completed By | `confirmed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 731 | Completed At | `confirmed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the step was marked as completed. | `2026-09-01T00:00:00Z` |
| 732 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 733 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 734 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 735 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_iq_step_execution_1` | UNIQUE | `execution_id, step_id` | UNIQUE(execution_id, step_id) |
| `rule_iq_step_execution_1` | Business validation | `execution_id, step_id` | The step and the execution must belong to the same protocol revision |
| `rule_iq_step_execution_2` | Business validation | `is_confirmed, confirmed_by, confirmed_at` | If is_confirmed = TRUE, confirmed_by and confirmed_at are required. After the result is recorded, completion/cancellation and evidence changes are locked and past records are preserved. |

#### Business Rules

Step attachment files are linked via evidence_link with IQ_STEP_EXECUTION / step_execution_id. On re-execution, new step execution rows are created.

[↑ Back to top](#top)

---

## OQ

<a id="table-oq_assessment"></a>
### 45. Operational Qualification Assessment (`oq_assessment`)

| Item | Definition |
|---|---|
| Description | Per-project OQ test set and document revision information. Distinguishes the approval status of the protocol from that of the execution results. |
| Primary Key | `oq_id` |
| Main References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 736 | OQ ID | `oq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | OQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 737 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Associated Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 738 | OQ Number | `oq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | OQ document number. Revision rows of the same document keep the same number. | `OQ-VP-SYS-010-20260529` |
| 739 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the operational qualification assessment document | `Operational Qualification` |
| 740 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the OQ assessment document (e.g. v1.0, v1.1) | `v1.0` |
| 741 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision number of the OQ assessment document (e.g. 1, 2, 3...) | `1` |
| 742 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for the initial creation or revision of the OQ assessment document | `Initial creation` |
| 743 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active, latest version of the OQ assessment document (TRUE/FALSE) | `TRUE` |
| 744 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | OQ protocol approval summary. DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected). A display value derived from the statuses of the child rows in the composition using the aggregation rule in the business rules; it is not bulk-written to the child rows. Since the header is not subject to the approval workflow, it has no CANCELLED value. | `DRAFT` |
| 745 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | OQ execution result approval summary. DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected). A display value derived from the statuses of the child rows in the composition using the aggregation rule in the business rules; it is not bulk-written to the child rows. Since the header is not subject to the approval workflow, it has no CANCELLED value. | `DRAFT` |
| 746 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 747 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | OQ author identifier | `00000000-0000-0000-0000-000000000001` |
| 748 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 749 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | OQ updater identifier | `00000000-0000-0000-0000-000000000001` |
| 750 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time. An assessment revision cannot be deleted if its composition (item_revision_refs) contains a test revision with a submission history, or if it is referenced by a submitted document or an electronic signature (rule_oq_assessment_3) | `2026-09-01T00:00:00Z` |
| 751 | Test Composition per Assessment Revision | `item_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | [{table_name:"oq_item",record_id,version,sort_order}] array. Identifies the full test composition and order of this assessment revision; PKs of unchanged tests from the previous assessment revision may be reused. | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_oq_assessment_1` | UNIQUE | `project_id, oq_no, revision_number` | UNIQUE(project_id, oq_no, revision_number) |
| `rule_oq_assessment_1` | Business validation | `project_id, oq_no, revision_number, is_current_version` | At most 1 row with is_current_version = TRUE per document |
| `rule_oq_assessment_2` | Business validation | - | Document revisions are distinct from test item revisions. The approval summary on the document screen is derived from the child protocols and execution results. |
| `uq_oq_assessment_current` | UNIQUE | `project_id, oq_no` | UNIQUE (project_id, oq_no) WHERE is_current_version=TRUE |
| `rule_oq_assessment_3` | Business validation | `deleted_at` | deleted_at cannot be set if the composition (item_revision_refs) of the assessment revision contains a test revision with a submission history, or if the source_refs of a submitted deliverable_revision or an electronic signature references this assessment revision. |
| `ck_oq_assessment_protocol_status` | CHECK | `protocol_status` | CHECK (protocol_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_oq_assessment_record_status` | CHECK | `record_status` | CHECK (record_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_oq_assessment_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

Manages the test list, protocol approval, result approval and document generation information. The document body is managed in deliverable_document / deliverable_revision / deliverable_section.

When only the deliverable title, body or output format changes, only a new deliverable_revision is created, and the assessment row and test items are kept. When the assessment scope, parent business information or test composition changes, a new assessment revision row is created and the full composition at that point is fixed in item_revision_refs. Unchanged test items re-reference the same PK and version; they are not duplicated and their parent FK is not moved. A new revision row that keeps the same item_key is created only for items whose test protocol content has changed, and it is included in the new composition.

The entries of the composition are actual revisions of oq_item and must belong to the same project and OQ phase. The original parent assessment of the item must have the same oq_no as this assessment. A composition must not include different revisions of the same item_key, and duplicate record_id or sort_order values are also prohibited. The composition is the full list including the sort order and is locked when a given document is first submitted. An assessment composition referenced by a submitted document or a past signature is not modified or deleted. Rows that are not in the composition are not automatically included based solely on their original parent FK. The protocol and result approval summaries are derived from the items and executions belonging to that composition. The aggregation targets are the test revisions referenced by the composition (item_revision_refs) that have not been disposed (is_disposed=TRUE). protocol_status uses the protocol_status of each test revision; record_status uses, for each test revision, the record_status of the valid result (is_current_revision=TRUE) of the last attempt_no whose initial execution row (record_revision=1) was not aborted (ABORTED), and a test revision with no execution is treated as DRAFT. If any one is REJECTED, REJECTED is displayed; otherwise, treating CANCELLED as DRAFT, the least advanced status among DRAFT < REVIEW < APPROVAL < APPROVED is displayed. If there are no aggregation targets, the status is DRAFT.

[↑ Back to top](#top)

---

<a id="table-oq_item"></a>
### 46. OQ Detailed Test Item (`oq_item`)

| Item | Definition |
|---|---|
| Description | Manages the protocol revisions, content, expected results, acceptance criteria and registration source of OQ test items |
| Primary Key | `oq_item_id` |
| Main References (FK) | `oq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_workflow_id, disposal_signature_id, disposed_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 752 | OQ Item ID | `oq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | PK of a specific protocol revision row of an OQ test item. item_key stays the same across revisions. | `00000000-0000-0000-0000-000000000001` |
| 753 | OQ ID | `oq_id` | `uuid` | N | Y | `oq_assessment.oq_id` | Y | - | N | Y | N | Y | The parent assessment revision in which this logical test was first registered. Subsequent item revisions keep the same original parent. The actual composition of each assessment revision is looked up in item_revision_refs. | `00000000-0000-0000-0000-000000000001` |
| 754 | Test Item Logical ID | `item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier that groups multiple revisions of the same test item. Issued once for a new item and kept on revision. | `00000000-0000-0000-0000-000000000001` |
| 755 | Test ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Test display ID on the screen. Keeps the same value across revisions of the same item; duplicates between different item_keys within a project are prohibited. | `OQ-AT-L01` |
| 756 | Test Case | `test_case` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Test item title on the screen. Not a separate classification value. | `Audit Trail` |
| 757 | Test Description | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed procedure for performing the test verification | `Audit trail automatically generated when user data is changed` |
| 758 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Test success criteria and expected result | `Before/after values, user, date/time and IP recorded` |
| 759 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected), CANCELLED (cancelled). Independent approval status of this test revision; not overwritten by the parent document status. | `DRAFT` |
| 760 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 761 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item author identifier | `00000000-0000-0000-0000-000000000001` |
| 762 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 763 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item updater identifier | `00000000-0000-0000-0000-000000000001` |
| 764 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deletion time of an unapproved draft. Only DRAFT revisions with no submission history can be deleted; an item with an approval history (an APPROVED revision exists for the same item_key) is handled by disposal (is_disposed, rule_oq_item_7) instead of deletion | `2026-09-01T00:00:00Z` |
| 765 | Protocol Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Protocol revision version displayed on the screen. | `ver1` |
| 766 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Revision number within the item_key. 1 or greater. | `1` |
| 767 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising an approved protocol. | - |
| 768 | Is Current Revision | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the current working revision of the same item_key. Independent of whether it is the approved version. | `TRUE` |
| 769 | Acceptance Criteria | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | Acceptance criteria on the screen. Distinct from expected_result (expected result). | `Matches the approved specification` |
| 770 | Registration Source | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL (manual entry), LIBRARY (library), SYSTEM_PACKAGE (system package), AI_DRAFT (AI draft). Same codes as requirement / fra_item.source_type | `MANUAL` |
| 771 | Source Library ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | The source when registered from the library. The test content is copied at registration and managed independently thereafter. | `00000000-0000-0000-0000-000000000001` |
| 772 | Source Package Code | `source_package_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Package code when registered from a system package. | `OQ-CORE` |
| 773 | Source Package Version | `source_package_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Package version selected at registration. | `ver2` |
| 774 | Source Template Code | `source_template_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Test template code selected within the package. | `OQ-PKG-001` |
| 775 | Item Approval Number | `approval_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Issued on first approval and displayed as the item number on the screen. The same logical item number is kept on revision. | - |
| 776 | Protocol Approval Workflow | `protocol_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Authoring/review/approval route and history for this protocol revision. | `00000000-0000-0000-0000-000000000001` |
| 777 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason entered in the disposal approval on the screen. Required when disposal is completed. | - |
| 778 | Disposal Approval Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Disposal approval workflow (approval_scope=DISPOSAL). Disposal is completed by the final approval of this approval workflow | `00000000-0000-0000-0000-000000000001` |
| 779 | Disposal Approval Signature | `disposal_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Final approval electronic signature of the disposal approval workflow (disposal_workflow_id). | `00000000-0000-0000-0000-000000000001` |
| 780 | Is Disposed | `is_disposed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Disposal status on the screen. Disposed items are excluded from new executions and aggregation; existing records are preserved. | `FALSE` |
| 781 | Disposed By | `disposed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Final approver of the disposal approval workflow | `00000000-0000-0000-0000-000000000001` |
| 782 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Final approval time of the disposal approval workflow. | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_oq_item_1` | UNIQUE | `item_key, revision_number` | UNIQUE(item_key, revision_number) |
| `rule_oq_item_1` | Business validation | `item_key, revision_number, is_current_version` | The same item_key belongs to the same project and OQ phase. At most 1 row with is_current_version = TRUE per item_key |
| `rule_oq_item_2` | Business validation | - | protocol_status is DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected), CANCELLED (cancelled). The protocol and steps are not modified directly after submission (REVIEW); modifications after rejection/cancellation and changes after approval are handled as a new revision row. |
| `rule_oq_item_3` | Business validation | - | approval_number is the first approval number of the logical item and is kept even when the revision changes. Approval and disposal signatures are preserved in the workflow_instance / approval_action / electronic_signature records that target the item revision. |
| `rule_oq_item_4` | Business validation | `source_library_id, source_package_code, source_package_version, source_template_code` | For LIBRARY registration, source_library_id is required. For SYSTEM_PACKAGE registration, source_package_code / source_package_version / source_template_code are required. |
| `rule_oq_item_5` | Business validation | - | Links to multiple URSs are managed in traceability_link as REQUIREMENT → OQ_ITEM / VERIFIED_BY. |
| `rule_oq_item_6` | Business validation | - | Registering an active test and approving its protocol require at least 1 linked URS revision and at least 1 detailed step with non-empty content. |
| `rule_oq_item_7` | Business validation | `disposal_reason, disposal_workflow_id, disposal_signature_id, is_disposed, disposed_by, disposed_at` | For disposal, a user with disposal permission enters a reason and submits a disposal approval workflow (approval_scope=DISPOSAL), and disposal is completed by its final approval. If is_disposed = TRUE, disposal_reason, disposal_workflow_id, disposal_signature_id, disposed_by and disposed_at are required, and disposal_signature_id / disposed_by / disposed_at must match the final approval signature, approver and time of that approval workflow. |
| `ck_oq_item_disposal` | CHECK | `is_disposed, disposal_reason, disposal_workflow_id, disposal_signature_id, disposed_by, disposed_at` | CHECK (is_disposed = FALSE OR (disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL AND disposal_signature_id IS NOT NULL AND disposed_by IS NOT NULL AND disposed_at IS NOT NULL)) |
| `rule_oq_item_8` | Business validation | - | Disposal applies to the active status of the same item_key, and past approval records are preserved. Adding the disposal status, reason and signature metadata is distinct from a revision that modifies the approved test body or steps, and the originally approved content is not changed. |
| `uq_oq_item_current` | UNIQUE | `item_key` | UNIQUE (item_key) WHERE is_current_version=TRUE |
| `rule_oq_item_9` | Business validation | `deleted_at` | deleted_at can only be set on DRAFT revisions with no submission history. If an APPROVED revision exists for the same item_key, the item cannot be deleted, and discontinuing its use is handled by disposal (is_disposed, rule_oq_item_7). |
| `ck_oq_item_source` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `ck_oq_item_protocol_status` | CHECK | `protocol_status` | CHECK (protocol_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_oq_item_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

The test content is stored in test_description, the expected result in expected_result and the acceptance criteria in acceptance_criteria. Repeating steps are managed in oq_step, and actual results, signatures and re-execution history in oq_execution.

All revisions of the same item_key belong to the same project. The item numbers approval_number and test_id are kept in subsequent revisions once first assigned. For items whose number is assigned on approval, an unapproved draft allows NULL. A number is not reused for a different logical key in the same project and the same table, and is not reassigned to another item even after disposal. Number issuance and revision creation are serialized with a concurrency lock on the project and the logical item, and the duplication, ownership and number-retention conditions are validated in the same transaction.

[↑ Back to top](#top)

---

<a id="table-oq_step"></a>
### 47. OQ Test Step (`oq_step`)

| Item | Definition |
|---|---|
| Description | Ordered detailed steps included in an OQ protocol revision |
| Primary Key | `step_id` |
| Main References (FK) | `oq_item_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 783 | Step ID | `step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 784 | Test Revision ID | `oq_item_id` | `uuid` | N | Y | `oq_item.oq_item_id` | Y | - | N | Y | N | Y | References oq_item.oq_item_id | `00000000-0000-0000-0000-000000000001` |
| 785 | Step Order | `step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display order of the detailed step on the screen. 1 or greater. | `1` |
| 786 | Step Content | `instruction` | `text` | N | N | - | Y | - | N | N | N | Y | Step body text that is added, edited and deleted on the screen. | - |
| 787 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 788 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 789 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 790 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_oq_step_1` | UNIQUE | `oq_item_id, step_order` | UNIQUE(oq_item_id, step_order) |
| `rule_oq_step_1` | Business validation | `oq_item_id, step_order` | step_order >= 1 |
| `rule_oq_step_2` | Business validation | - | After the parent protocol has been submitted (REVIEW), the step structure and body are not changed. Modifications after rejection/cancellation and changes after approval are recorded as a new test revision and its child steps. |

#### Business Rules

Step completion status and attached evidence are not design values of this table but per-execution-attempt records in oq_step_execution.

[↑ Back to top](#top)

---

<a id="table-oq_execution"></a>
### 48. OQ Test Execution (`oq_execution`)

| Item | Definition |
|---|---|
| Description | Manages the actual result, judgment, execution signature and result approval for each execution attempt of an OQ test revision |
| Primary Key | `execution_id` |
| Main References (FK) | `oq_item_id, executed_by, execution_signature_id, abort_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 791 | Execution ID | `execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 792 | Executed Protocol Revision ID | `oq_item_id` | `uuid` | N | Y | `oq_item.oq_item_id` | Y | - | N | Y | N | Y | References oq_item.oq_item_id | `00000000-0000-0000-0000-000000000001` |
| 793 | Execution Attempt | `attempt_no` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 1 for the first execution, 2 or greater for re-execution. The re-execution count on the screen is attempt_no - 1. | `1` |
| 794 | Result Correction Number | `record_revision` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Sequence number in the correction history of results of the same attempt. Distinct from re-execution. | `1` |
| 795 | Is Current Result Correction | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the currently valid result record for the same test revision and attempt. Result correction rows that are in progress or cancelled are FALSE; the value is switched when the correction is finalized as RECORDED. | `TRUE` |
| 796 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Actual result entered on the screen. Required when registering the judgment result. | - |
| 797 | Execution Judgment | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | PASS, FAIL, NA (not applicable). Not executed is represented as NULL; distinct from the approval status. | `PASS` |
| 798 | Execution Date | `performed_on` | `date` | N | N | - | N | - | N | N | N | Y | Execution date selected on the screen. Distinct from the electronic signature date and time. | `2026-09-16` |
| 799 | Executed By | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 800 | Result Recorded At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the judgment result was registered via electronic signature. | `2026-09-01T00:00:00Z` |
| 801 | Execution Signature ID | `execution_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature entered when registering the execution result. The password is not stored. | `00000000-0000-0000-0000-000000000001` |
| 802 | Execution Progress Status | `execution_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS (performing steps / entering results), RECORDED (judgment result registered), ABORTED (aborted without registering a result; for a result correction row, cancellation of the in-progress correction). Deviations and final approval are managed separately. | `IN_PROGRESS` |
| 803 | Result Approval Status | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected), CANCELLED (cancelled). Managed independently of the protocol approval status. A correction after rejection/cancellation is handled as a new record_revision of the same attempt_no. | `DRAFT` |
| 804 | Result Approval Workflow | `result_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Final result review and approval route for this attempt and result correction. | `00000000-0000-0000-0000-000000000001` |
| 805 | Result Approval Version | `approval_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Display version of the finally approved test result. Distinct from the protocol version. | `ver1` |
| 806 | Deviation Reason | `deviation_reason` | `text` | N | N | - | N | - | N | N | N | Y | Deviation reason as entered at the time the FAIL result was registered. | - |
| 807 | Immediate Action | `immediate_action` | `text` | N | N | - | N | - | N | N | N | Y | Immediate action as entered at the time the FAIL result was registered. | - |
| 808 | Result Correction Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for correcting the result when record_revision is incremented. | - |
| 809 | Abort Reason | `abort_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for aborting the execution without registering a result, or for cancelling an in-progress result correction (ABORTED). Required for ABORTED | - |
| 810 | Abort Signature ID | `abort_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature entered by the assignee after confirming the abort reason when aborting the execution or cancelling a result correction (signature_action=ABORT; the signature target is this execution_id). Distinct from the result registration signature (execution_signature_id); required for ABORTED | `00000000-0000-0000-0000-000000000001` |
| 811 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 812 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 813 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 814 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 815 | Validation Baseline at Execution | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | Fixes the project's current baseline at the start of execution. Required before result registration and signing; not overwritten by later changes to the project baseline. | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_oq_execution_1` | UNIQUE | `oq_item_id, attempt_no, record_revision` | UNIQUE(oq_item_id, attempt_no, record_revision) |
| `rule_oq_execution_1` | Business validation | `oq_item_id, attempt_no, record_revision, is_current_revision` | attempt_no / record_revision >= 1. At most 1 row with is_current_revision = TRUE per test revision and attempt (enforced by uq_oq_execution_current) |
| `rule_oq_execution_2` | Business validation | `actual_result, qualification_result, performed_on, executed_by, executed_at, execution_signature_id` | Execution is allowed only on an approved protocol. On transition to RECORDED, all steps must be completed, and actual_result / qualification_result / performed_on / executed_by / executed_at / execution_signature_id are required. |
| `rule_oq_execution_3` | Business validation | `qualification_result, deviation_reason, immediate_action` | If qualification_result = FAIL, deviation_reason and immediate_action are required. If there is no unclosed deviation (deviation_status<>'CLOSED') for the same logical test (item_key), a new deviation is created with source_entity_id = this execution_id. If there is an ongoing deviation (a FAIL in a re-execution under an approved action), no new deviation is created; instead, a deviation_action_round with the next round_number is created for that deviation, with failed_execution_id = this execution_id. Deviation status transitions follow rule_deviation_9. |
| `rule_oq_execution_4` | Business validation | - | A re-execution retains the existing results, step completions and evidence, and starts with a new attempt_no. If there is an unclosed deviation for the same logical test, a new attempt_no can be started only as a re-execution for that deviation, and deviation_status must be RERUN_ALLOWED and the rerun_execution_id of the current action round (current_action_round_id) must be empty. A result correction within the same attempt increments record_revision and does not overwrite previous approval records. |
| `rule_oq_execution_5` | Business validation | - | Final result approval is possible after protocol approval and execution result registration (RECORDED), and proceeds regardless of the status of the linked deviation. A FAIL result can also be approved before the deviation is closed. Result approval confirms that the judgment was recorded correctly and does not mean that deviation handling is complete. Prerequisites on the deviation side follow rule_deviation_4. |
| `ck_oq_execution_baseline` | CHECK | `execution_status, system_baseline_id` | CHECK (execution_status <> 'RECORDED' OR system_baseline_id IS NOT NULL) |
| `rule_oq_execution_6` | Business validation | `qualification_result, record_revision` | Result correction within the same attempt_no (record_revision incremented): if the pre-correction result was a FAIL already linked to a deviation or an action round, no new deviation or new round is created even if the corrected result is FAIL; the existing link is kept and the judgment is read from the latest valid correction. If a result that was PASS or NA before correction becomes FAIL through the correction, it is handled according to rule_oq_execution_3. Even if a FAIL is corrected to PASS, the linked deviation and the already-created action round are left as they are, and the deviation is closed through the deviation process, e.g. closure without re-execution (rule_deviation_5) after that round is approved. While the completion report approval workflow is in progress, re-execution results are not corrected. |
| `ck_oq_execution_result` | CHECK | `qualification_result` | CHECK (qualification_result IS NULL OR qualification_result IN ('PASS','FAIL','NA')) |
| `ck_oq_execution_record_status` | CHECK | `record_status` | CHECK (record_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `rule_oq_execution_7` | Business validation | `qualification_result` | A re-execution result for an open deviation can only be recorded as PASS or FAIL (NA not allowed). If the test is judged to be not applicable during the re-execution, the re-execution is aborted (rule_oq_execution_8), the deviation is then closed without re-execution (rule_deviation_5), and the justification is recorded in the closure reason. |
| `rule_oq_execution_8` | Business validation | `execution_status, abort_reason` | If the initial execution row (record_revision=1) is IN_PROGRESS, it can be aborted (ABORTED) without recording a result. Cancellation of a result correction row (record_revision≥2) follows rule_oq_execution_9. abort_reason and an abort electronic signature (abort_signature_id, signature_action=ABORT) are required, and the person who aborted and the abort time are the signer_id and signed_at of that signature. After the abort, this row and its step executions and evidence are not modified. An ABORTED execution is not subject to result approval, and the next execution starts with a new attempt_no. If the aborted execution was a re-execution for a deviation, the rerun_execution_id links of the current action round and of the deviation are cleared in the same transaction, and deviation_status remains RERUN_ALLOWED. |
| `rule_oq_execution_9` | Business validation | `record_revision, is_current_revision, execution_status` | A result correction row (record_revision≥2) can only be created when the current valid result of the same attempt_no is RECORDED, and there can be at most 1 in-progress correction row for the same attempt_no. A correction row is created as IN_PROGRESS with is_current_revision=FALSE, and in the meantime the previous row remains the current valid result. In the transaction that finalizes the correction row as RECORDED (correction signature), the correction row is set to TRUE and the previous row to FALSE, and the deviation handling of rule_oq_execution_6 and the update of deviation.rerun_execution_id are performed together. If an in-progress correction is cancelled, the correction row is closed as ABORTED (abort_reason and the abort electronic signature abort_signature_id are required) and keeps is_current_revision=FALSE; the previous row, the re-execution link and the deviation status are not changed. To correct again, the next record_revision is created. |
| `ck_oq_execution_status` | CHECK | `execution_status` | CHECK (execution_status IN ('IN_PROGRESS','RECORDED','ABORTED')) |
| `ck_oq_execution_abort` | CHECK | `execution_status, abort_reason, abort_signature_id, execution_signature_id` | CHECK (execution_status <> 'ABORTED' OR (abort_reason IS NOT NULL AND abort_signature_id IS NOT NULL AND execution_signature_id IS NULL)) |
| `uq_oq_execution_current` | UNIQUE | `oq_item_id, attempt_no, is_current_revision` | UNIQUE (oq_item_id, attempt_no) WHERE is_current_revision=TRUE |
| `ck_oq_execution_revision_reason` | CHECK | `record_revision, revision_reason` | CHECK (record_revision = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

Evidence for the test as a whole is linked via evidence_link with the OQ_EXECUTION target. Electronic signatures target this execution_id, and a new signature is obtained after a result correction. Signatures for recording a result or finalizing a correction are linked to execution_signature_id (EXECUTE), and signatures for aborting an execution or cancelling a correction to abort_signature_id (ABORT).

system_baseline_id must belong to the same project as the test. The validity of the protocol approval and prerequisites is validated against the baseline adopted at the start of execution. A re-execution links the baseline current at that time to the new execution record, and the baselines of past executions are kept. A result correction is recorded as a new record_revision of the same logical test and attempt_no and keeps the baseline of the original execution.

[↑ Back to top](#top)

---

<a id="table-oq_step_execution"></a>
### 49. OQ Step Execution (`oq_step_execution`)

| Item | Definition |
|---|---|
| Description | Unit that records step completion status and links step evidence for an OQ execution attempt |
| Primary Key | `step_execution_id` |
| Main References (FK) | `execution_id, step_id, confirmed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 816 | Step Execution ID | `step_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 817 | Execution ID | `execution_id` | `uuid` | N | Y | `oq_execution.execution_id` | Y | - | N | Y | N | Y | References oq_execution.execution_id | `00000000-0000-0000-0000-000000000001` |
| 818 | Step ID | `step_id` | `uuid` | N | Y | `oq_step.step_id` | Y | - | N | Y | N | Y | References oq_step.step_id | `00000000-0000-0000-0000-000000000001` |
| 819 | Step Completed | `is_confirmed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Complete/cancel value of the detailed step on the screen. | `FALSE` |
| 820 | Completed By | `confirmed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 821 | Completed At | `confirmed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the step was marked as completed. | `2026-09-01T00:00:00Z` |
| 822 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 823 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 824 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 825 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_oq_step_execution_1` | UNIQUE | `execution_id, step_id` | UNIQUE(execution_id, step_id) |
| `rule_oq_step_execution_1` | Business validation | `execution_id, step_id` | The step and the execution must belong to the same protocol revision |
| `rule_oq_step_execution_2` | Business validation | `is_confirmed, confirmed_by, confirmed_at` | If is_confirmed = TRUE, confirmed_by and confirmed_at are required. After the result is recorded, completion/cancellation and evidence changes are locked and past records are preserved. |

#### Business Rules

Step attachment files are linked via evidence_link with OQ_STEP_EXECUTION / step_execution_id. On re-execution, new step execution rows are created.

[↑ Back to top](#top)

---

## PQ

<a id="table-pq_assessment"></a>
### 50. Performance Qualification Assessment (`pq_assessment`)

| Item | Definition |
|---|---|
| Description | Per-project PQ test set and document revision information. Distinguishes the approval status of the protocol from that of the execution results. |
| Primary Key | `pq_id` |
| Main References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 826 | PQ ID | `pq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | PQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 827 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Associated Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 828 | PQ Number | `pq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | PQ document number. Revision rows of the same document keep the same number. | `PQ-VP-SYS-008-20260422` |
| 829 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the performance qualification assessment document | `Performance Qualification` |
| 830 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the PQ assessment document (e.g. v1.0, v1.1) | `v1.0` |
| 831 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision number of the PQ assessment document (e.g. 1, 2, 3...) | `1` |
| 832 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for the initial creation or revision of the PQ assessment document | `Initial creation` |
| 833 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active, latest version of the PQ assessment document (TRUE/FALSE) | `TRUE` |
| 834 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | PQ protocol approval summary. DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected). A display value derived from the statuses of the child rows in the composition using the aggregation rule in the business rules; it is not bulk-written to the child rows. Since the header is not subject to the approval workflow, it has no CANCELLED value. | `DRAFT` |
| 835 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | PQ execution result approval summary. DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected). A display value derived from the statuses of the child rows in the composition using the aggregation rule in the business rules; it is not bulk-written to the child rows. Since the header is not subject to the approval workflow, it has no CANCELLED value. | `DRAFT` |
| 836 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 837 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | PQ author identifier | `00000000-0000-0000-0000-000000000001` |
| 838 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 839 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | PQ updater identifier | `00000000-0000-0000-0000-000000000001` |
| 840 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time. An assessment revision cannot be deleted if its composition (item_revision_refs) contains a test revision with a submission history, or if it is referenced by a submitted document or an electronic signature (rule_pq_assessment_3) | `2026-09-01T00:00:00Z` |
| 841 | Test Composition per Assessment Revision | `item_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | [{table_name:"pq_item",record_id,version,sort_order}] array. Identifies the full test composition and order of this assessment revision; PKs of unchanged tests from the previous assessment revision may be reused. | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_pq_assessment_1` | UNIQUE | `project_id, pq_no, revision_number` | UNIQUE(project_id, pq_no, revision_number) |
| `rule_pq_assessment_1` | Business validation | `project_id, pq_no, revision_number, is_current_version` | At most 1 row with is_current_version = TRUE per document |
| `rule_pq_assessment_2` | Business validation | - | Document revisions are distinct from test item revisions. The approval summary on the document screen is derived from the child protocols and execution results. |
| `uq_pq_assessment_current` | UNIQUE | `project_id, pq_no` | UNIQUE (project_id, pq_no) WHERE is_current_version=TRUE |
| `rule_pq_assessment_3` | Business validation | `deleted_at` | deleted_at cannot be set if the composition (item_revision_refs) of the assessment revision contains a test revision with a submission history, or if the source_refs of a submitted deliverable_revision or an electronic signature references this assessment revision. |
| `ck_pq_assessment_protocol_status` | CHECK | `protocol_status` | CHECK (protocol_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_pq_assessment_record_status` | CHECK | `record_status` | CHECK (record_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_pq_assessment_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

Manages the test list, protocol approval, result approval and document generation information. The document body is managed in deliverable_document / deliverable_revision / deliverable_section.

When only the deliverable title, body or output format changes, only a new deliverable_revision is created, and the assessment row and test items are kept. When the assessment scope, parent business information or test composition changes, a new assessment revision row is created and the full composition at that point is fixed in item_revision_refs. Unchanged test items re-reference the same PK and version; they are not duplicated and their parent FK is not moved. A new revision row that keeps the same item_key is created only for items whose test protocol content has changed, and it is included in the new composition.

The entries of the composition are actual revisions of pq_item and must belong to the same project and PQ phase. The original parent assessment of the item must have the same pq_no as this assessment. A composition must not include different revisions of the same item_key, and duplicate record_id or sort_order values are also prohibited. The composition is the full list including the sort order and is locked when a given document is first submitted. An assessment composition referenced by a submitted document or a past signature is not modified or deleted. Rows that are not in the composition are not automatically included based solely on their original parent FK. The protocol and result approval summaries are derived from the items and executions belonging to that composition. The aggregation targets are the test revisions referenced by the composition (item_revision_refs) that have not been disposed (is_disposed=TRUE). protocol_status uses the protocol_status of each test revision; record_status uses, for each test revision, the record_status of the valid result (is_current_revision=TRUE) of the last attempt_no whose initial execution row (record_revision=1) was not aborted (ABORTED), and a test revision with no execution is treated as DRAFT. If any one is REJECTED, REJECTED is displayed; otherwise, treating CANCELLED as DRAFT, the least advanced status among DRAFT < REVIEW < APPROVAL < APPROVED is displayed. If there are no aggregation targets, the status is DRAFT.

[↑ Back to top](#top)

---

<a id="table-pq_item"></a>
### 51. PQ Detailed Test Item (`pq_item`)

| Item | Definition |
|---|---|
| Description | Manages the protocol revisions, content, expected results, acceptance criteria and registration source of PQ test items |
| Primary Key | `pq_item_id` |
| Main References (FK) | `pq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_workflow_id, disposal_signature_id, disposed_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 842 | PQ Item ID | `pq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | PK of a specific protocol revision row of a PQ test item. item_key stays the same across revisions. | `00000000-0000-0000-0000-000000000001` |
| 843 | PQ ID | `pq_id` | `uuid` | N | Y | `pq_assessment.pq_id` | Y | - | N | Y | N | Y | The parent assessment revision in which this logical test was first registered. Subsequent item revisions keep the same original parent. The actual composition of each assessment revision is looked up in item_revision_refs. | `00000000-0000-0000-0000-000000000001` |
| 844 | Test Display ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Test display ID on the screen. Keeps the same value across revisions of the same item; duplicates between different item_keys within a project are prohibited. | `PQ-NEW-01` |
| 845 | Test Item Name | `test_case` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Test item title on the screen. | `Business scenario verification` |
| 846 | Test Item Logical ID | `item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier that groups multiple revisions of the same test item. Issued once for a new item and kept on revision. | `00000000-0000-0000-0000-000000000001` |
| 847 | Test Description | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed procedure for performing the PQ test verification | `Verify normal completion of the production process for 3 or more consecutive batches` |
| 848 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Test success criteria and expected result | `All batches must be produced normally to completion in accordance with specifications` |
| 849 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 850 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item author identifier | `00000000-0000-0000-0000-000000000001` |
| 851 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 852 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item updater identifier | `00000000-0000-0000-0000-000000000001` |
| 853 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deletion time of an unapproved draft. Only DRAFT revisions with no submission history can be deleted; an item with an approval history (an APPROVED revision exists for the same item_key) is handled by disposal (is_disposed, rule_pq_item_7) instead of deletion | `2026-09-01T00:00:00Z` |
| 854 | Protocol Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Protocol revision version displayed on the screen. | `ver1` |
| 855 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Revision number within the item_key. 1 or greater. | `1` |
| 856 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising an approved protocol. | - |
| 857 | Is Current Revision | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the current working revision of the same item_key. Independent of whether it is the approved version. | `TRUE` |
| 858 | Protocol Approval Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected), CANCELLED (cancelled) | `DRAFT` |
| 859 | Acceptance Criteria | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | Acceptance criteria on the screen. Distinct from expected_result (expected result). | `Matches the approved specification` |
| 860 | Registration Source | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL (manual entry), LIBRARY (library), SYSTEM_PACKAGE (system package), AI_DRAFT (AI draft). Same codes as requirement / fra_item.source_type | `MANUAL` |
| 861 | Source Library ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | The source when registered from the library. The test content is copied at registration and managed independently thereafter. | `00000000-0000-0000-0000-000000000001` |
| 862 | Source Package Code | `source_package_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Package code when registered from a system package. | `PQ-CORE` |
| 863 | Source Package Version | `source_package_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Package version selected at registration. | `ver2` |
| 864 | Source Template Code | `source_template_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Test template code selected within the package. | `PQ-PKG-001` |
| 865 | Item Approval Number | `approval_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Issued on first approval and displayed as the item number on the screen. The same logical item number is kept on revision. | - |
| 866 | Protocol Approval Workflow | `protocol_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Authoring/review/approval route and history for this protocol revision. | `00000000-0000-0000-0000-000000000001` |
| 867 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason entered in the disposal approval on the screen. Required when disposal is completed. | - |
| 868 | Disposal Approval Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Disposal approval workflow (approval_scope=DISPOSAL). Disposal is completed by the final approval of this approval workflow | `00000000-0000-0000-0000-000000000001` |
| 869 | Disposal Approval Signature | `disposal_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Final approval electronic signature of the disposal approval workflow (disposal_workflow_id). | `00000000-0000-0000-0000-000000000001` |
| 870 | Is Disposed | `is_disposed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Disposal status on the screen. Disposed items are excluded from new executions and aggregation; existing records are preserved. | `FALSE` |
| 871 | Disposed By | `disposed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Final approver of the disposal approval workflow | `00000000-0000-0000-0000-000000000001` |
| 872 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Final approval time of the disposal approval workflow. | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_pq_item_1` | UNIQUE | `item_key, revision_number` | UNIQUE(item_key, revision_number) |
| `rule_pq_item_1` | Business validation | `item_key, revision_number, is_current_version` | The same item_key belongs to the same project and PQ phase. At most 1 row with is_current_version = TRUE per item_key |
| `rule_pq_item_2` | Business validation | - | protocol_status is DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected), CANCELLED (cancelled). The protocol and steps are not modified directly after submission (REVIEW); modifications after rejection/cancellation and changes after approval are handled as a new revision row. |
| `rule_pq_item_3` | Business validation | - | approval_number is the first approval number of the logical item and is kept even when the revision changes. Approval and disposal signatures are preserved in the workflow_instance / approval_action / electronic_signature records that target the item revision. |
| `rule_pq_item_4` | Business validation | `source_library_id, source_package_code, source_package_version, source_template_code` | For LIBRARY registration, source_library_id is required. For SYSTEM_PACKAGE registration, source_package_code / source_package_version / source_template_code are required. |
| `rule_pq_item_5` | Business validation | - | Links to multiple URSs are managed in traceability_link as REQUIREMENT → PQ_ITEM / VERIFIED_BY. |
| `rule_pq_item_6` | Business validation | - | Registering an active test and approving its protocol require at least 1 linked URS revision and at least 1 detailed step with non-empty content. |
| `rule_pq_item_7` | Business validation | `disposal_reason, disposal_workflow_id, disposal_signature_id, is_disposed, disposed_by, disposed_at` | For disposal, a user with disposal permission enters a reason and submits a disposal approval workflow (approval_scope=DISPOSAL), and disposal is completed by its final approval. If is_disposed = TRUE, disposal_reason, disposal_workflow_id, disposal_signature_id, disposed_by and disposed_at are required, and disposal_signature_id / disposed_by / disposed_at must match the final approval signature, approver and time of that approval workflow. |
| `ck_pq_item_disposal` | CHECK | `is_disposed, disposal_reason, disposal_workflow_id, disposal_signature_id, disposed_by, disposed_at` | CHECK (is_disposed = FALSE OR (disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL AND disposal_signature_id IS NOT NULL AND disposed_by IS NOT NULL AND disposed_at IS NOT NULL)) |
| `rule_pq_item_8` | Business validation | - | Disposal applies to the active status of the same item_key, and past approval records are preserved. Adding the disposal status, reason and signature metadata is distinct from a revision that modifies the approved test body or steps, and the originally approved content is not changed. |
| `uq_pq_item_current` | UNIQUE | `item_key` | UNIQUE (item_key) WHERE is_current_version=TRUE |
| `rule_pq_item_9` | Business validation | `deleted_at` | deleted_at can only be set on DRAFT revisions with no submission history. If an APPROVED revision exists for the same item_key, the item cannot be deleted, and discontinuing its use is handled by disposal (is_disposed, rule_pq_item_7). |
| `ck_pq_item_source` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `ck_pq_item_protocol_status` | CHECK | `protocol_status` | CHECK (protocol_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_pq_item_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

The test content is stored in test_description, the expected result in expected_result and the acceptance criteria in acceptance_criteria. Repeating steps are managed in pq_step, and actual results, signatures and re-execution history in pq_execution.

All revisions of the same item_key belong to the same project. The item numbers approval_number and test_id are kept in subsequent revisions once first assigned. For items whose number is assigned on approval, an unapproved draft allows NULL. A number is not reused for a different logical key in the same project and the same table, and is not reassigned to another item even after disposal. Number issuance and revision creation are serialized with a concurrency lock on the project and the logical item, and the duplication, ownership and number-retention conditions are validated in the same transaction.

[↑ Back to top](#top)

---

<a id="table-pq_step"></a>
### 52. PQ Test Step (`pq_step`)

| Item | Definition |
|---|---|
| Description | Ordered detailed steps included in a PQ protocol revision |
| Primary Key | `step_id` |
| Main References (FK) | `pq_item_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 873 | Step ID | `step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 874 | Test Revision ID | `pq_item_id` | `uuid` | N | Y | `pq_item.pq_item_id` | Y | - | N | Y | N | Y | References pq_item.pq_item_id | `00000000-0000-0000-0000-000000000001` |
| 875 | Step Order | `step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display order of the detailed step on the screen. 1 or greater. | `1` |
| 876 | Step Content | `instruction` | `text` | N | N | - | Y | - | N | N | N | Y | Step body text that is added, edited and deleted on the screen. | - |
| 877 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 878 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 879 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 880 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_pq_step_1` | UNIQUE | `pq_item_id, step_order` | UNIQUE(pq_item_id, step_order) |
| `rule_pq_step_1` | Business validation | `pq_item_id, step_order` | step_order >= 1 |
| `rule_pq_step_2` | Business validation | - | After the parent protocol has been submitted (REVIEW), the step structure and body are not changed. Modifications after rejection/cancellation and changes after approval are recorded as a new test revision and its child steps. |

#### Business Rules

Step completion status and attached evidence are not design values of this table but per-execution-attempt records in pq_step_execution.

[↑ Back to top](#top)

---

<a id="table-pq_execution"></a>
### 53. PQ Test Execution (`pq_execution`)

| Item | Definition |
|---|---|
| Description | Manages the actual result, judgment, execution signature and result approval for each execution attempt of a PQ test revision |
| Primary Key | `execution_id` |
| Main References (FK) | `pq_item_id, executed_by, execution_signature_id, abort_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 881 | Execution ID | `execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 882 | Executed Protocol Revision ID | `pq_item_id` | `uuid` | N | Y | `pq_item.pq_item_id` | Y | - | N | Y | N | Y | References pq_item.pq_item_id | `00000000-0000-0000-0000-000000000001` |
| 883 | Execution Attempt | `attempt_no` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 1 for the first execution, 2 or greater for re-execution. The re-execution count on the screen is attempt_no - 1. | `1` |
| 884 | Result Correction Number | `record_revision` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Sequence number in the correction history of results of the same attempt. Distinct from re-execution. | `1` |
| 885 | Is Current Result Correction | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the currently valid result record for the same test revision and attempt. Result correction rows that are in progress or cancelled are FALSE; the value is switched when the correction is finalized as RECORDED. | `TRUE` |
| 886 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Actual result entered on the screen. Required when registering the judgment result. | - |
| 887 | Execution Judgment | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | PASS, FAIL, NA (not applicable). Not executed is represented as NULL; distinct from the approval status. | `PASS` |
| 888 | Execution Date | `performed_on` | `date` | N | N | - | N | - | N | N | N | Y | Execution date selected on the screen. Distinct from the electronic signature date and time. | `2026-09-16` |
| 889 | Executed By | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 890 | Result Recorded At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the judgment result was registered via electronic signature. | `2026-09-01T00:00:00Z` |
| 891 | Execution Signature ID | `execution_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature entered when registering the execution result. The password is not stored. | `00000000-0000-0000-0000-000000000001` |
| 892 | Execution Progress Status | `execution_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS (performing steps / entering results), RECORDED (judgment result registered), ABORTED (aborted without registering a result; for a result correction row, cancellation of the in-progress correction). Deviations and final approval are managed separately. | `IN_PROGRESS` |
| 893 | Result Approval Status | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected), CANCELLED (cancelled). Managed independently of the protocol approval status. A correction after rejection/cancellation is handled as a new record_revision of the same attempt_no. | `DRAFT` |
| 894 | Result Approval Workflow | `result_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Final result review and approval route for this attempt and result correction. | `00000000-0000-0000-0000-000000000001` |
| 895 | Result Approval Version | `approval_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Display version of the finally approved test result. Distinct from the protocol version. | `ver1` |
| 896 | Deviation Reason | `deviation_reason` | `text` | N | N | - | N | - | N | N | N | Y | Deviation reason as entered at the time the FAIL result was registered. | - |
| 897 | Immediate Action | `immediate_action` | `text` | N | N | - | N | - | N | N | N | Y | Immediate action as entered at the time the FAIL result was registered. | - |
| 898 | Result Correction Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for correcting the result when record_revision is incremented. | - |
| 899 | Abort Reason | `abort_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for aborting the execution without registering a result, or for cancelling an in-progress result correction (ABORTED). Required for ABORTED | - |
| 900 | Abort Signature ID | `abort_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature entered by the assignee after confirming the abort reason when aborting the execution or cancelling a result correction (signature_action=ABORT; the signature target is this execution_id). Distinct from the result registration signature (execution_signature_id); required for ABORTED | `00000000-0000-0000-0000-000000000001` |
| 901 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 902 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 903 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 904 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 905 | Validation Baseline at Execution | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | Fixes the project's current baseline at the start of execution. Required before result registration and signing; not overwritten by later changes to the project baseline. | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_pq_execution_1` | UNIQUE | `pq_item_id, attempt_no, record_revision` | UNIQUE(pq_item_id, attempt_no, record_revision) |
| `rule_pq_execution_1` | Business validation | `pq_item_id, attempt_no, record_revision, is_current_revision` | attempt_no / record_revision >= 1. At most 1 row with is_current_revision = TRUE per test revision and attempt (enforced by uq_pq_execution_current) |
| `rule_pq_execution_2` | Business validation | `actual_result, qualification_result, performed_on, executed_by, executed_at, execution_signature_id` | Execution is allowed only on an approved protocol. On transition to RECORDED, all steps must be completed, and actual_result / qualification_result / performed_on / executed_by / executed_at / execution_signature_id are required. |
| `rule_pq_execution_3` | Business validation | `qualification_result, deviation_reason, immediate_action` | If qualification_result = FAIL, deviation_reason and immediate_action are required. If there is no unclosed deviation (deviation_status<>'CLOSED') for the same logical test (item_key), a new deviation is created with source_entity_id = this execution_id. If there is an ongoing deviation (a FAIL in a re-execution under an approved action), no new deviation is created; instead, a deviation_action_round with the next round_number is created for that deviation, with failed_execution_id = this execution_id. Deviation status transitions follow rule_deviation_9. |
| `rule_pq_execution_4` | Business validation | - | A re-execution retains the existing results, step completions and evidence, and starts with a new attempt_no. If there is an unclosed deviation for the same logical test, a new attempt_no can be started only as a re-execution for that deviation, and deviation_status must be RERUN_ALLOWED and the rerun_execution_id of the current action round (current_action_round_id) must be empty. A result correction within the same attempt increments record_revision and does not overwrite previous approval records. |
| `rule_pq_execution_5` | Business validation | - | Final result approval is possible after protocol approval and execution result registration (RECORDED), and proceeds regardless of the status of the linked deviation. A FAIL result can also be approved before the deviation is closed. Result approval confirms that the judgment was recorded correctly and does not mean that deviation handling is complete. Prerequisites on the deviation side follow rule_deviation_4. |
| `ck_pq_execution_baseline` | CHECK | `execution_status, system_baseline_id` | CHECK (execution_status <> 'RECORDED' OR system_baseline_id IS NOT NULL) |
| `rule_pq_execution_6` | Business validation | `qualification_result, record_revision` | Result correction within the same attempt_no (record_revision incremented): if the pre-correction result is a FAIL already linked to a deviation or action round, no new deviation or new round is created even if the corrected result is FAIL; the existing link is kept and the judgment is read from the latest valid correction. If a result that was PASS or NA before correction becomes FAIL through correction, it is handled according to rule_pq_execution_3. Even if a FAIL is corrected to PASS, the linked deviation and any action rounds already created are left as they are, and the deviation is closed through the deviation procedure, e.g. closure without re-execution (rule_deviation_5) after that round is approved. While a completion report approval workflow is in progress, the re-execution result is not corrected. |
| `ck_pq_execution_result` | CHECK | `qualification_result` | CHECK (qualification_result IS NULL OR qualification_result IN ('PASS','FAIL','NA')) |
| `ck_pq_execution_record_status` | CHECK | `record_status` | CHECK (record_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `rule_pq_execution_7` | Business validation | `qualification_result` | A re-execution result for an unclosed deviation may be registered only as PASS or FAIL (NA not allowed). If the test is judged not applicable during re-execution, abort the re-execution (rule_pq_execution_8), then perform closure without re-execution (rule_deviation_5) and record the justification in the closure reason. |
| `rule_pq_execution_8` | Business validation | `execution_status, abort_reason` | When the initial execution row (record_revision=1) is IN_PROGRESS, it can be aborted (ABORTED) without registering a result. Cancellation of a result correction row (record_revision≥2) follows rule_pq_execution_9. abort_reason and an abort electronic signature (abort_signature_id, signature_action=ABORT) are required, and the person and time of the abort are the signer_id and signed_at of that signature. After the abort, this row and its step executions and evidence are not modified. An ABORTED execution is not subject to result approval, and the next execution starts with a new attempt_no. If the aborted execution is a re-execution for a deviation, the rerun_execution_id link of the current action round and of the deviation is released in the same transaction, and deviation_status remains RERUN_ALLOWED. |
| `rule_pq_execution_9` | Business validation | `record_revision, is_current_revision, execution_status` | A result correction row (record_revision≥2) can be created only when the current valid result of the same attempt_no is RECORDED, and there can be at most 1 in-progress correction row per attempt_no. A correction row is created as IN_PROGRESS with is_current_revision=FALSE, and in the meantime the previous row remains the current valid result. In the transaction that finalizes the correction row as RECORDED (correction signature), the correction row is set to TRUE and the previous row to FALSE, and the deviation handling of rule_pq_execution_6 and the update of deviation.rerun_execution_id are performed together. If an in-progress correction is cancelled, the correction row is closed as ABORTED (abort_reason and the abort electronic signature abort_signature_id required) and keeps is_current_revision=FALSE; the previous row, the re-execution link and the deviation status are not changed. To correct again, create the next record_revision. |
| `ck_pq_execution_status` | CHECK | `execution_status` | CHECK (execution_status IN ('IN_PROGRESS','RECORDED','ABORTED')) |
| `ck_pq_execution_abort` | CHECK | `execution_status, abort_reason, abort_signature_id, execution_signature_id` | CHECK (execution_status <> 'ABORTED' OR (abort_reason IS NOT NULL AND abort_signature_id IS NOT NULL AND execution_signature_id IS NULL)) |
| `uq_pq_execution_current` | UNIQUE | `pq_item_id, attempt_no, is_current_revision` | UNIQUE (pq_item_id, attempt_no) WHERE is_current_revision=TRUE |
| `ck_pq_execution_revision_reason` | CHECK | `record_revision, revision_reason` | CHECK (record_revision = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

Evidence for the test as a whole is linked through evidence_link with the PQ_EXECUTION target. Electronic signatures target this execution_id, and a new signature is obtained after a result correction. Signatures for result registration and correction finalization are linked to execution_signature_id (EXECUTE), and signatures for execution abort and correction cancellation to abort_signature_id (ABORT).

system_baseline_id must belong to the same project as the test. The validity of the protocol approval and prerequisites is validated against the baseline adopted at the start of execution. A re-execution links the baseline current at that time to the new execution record, and the baselines of past executions are kept. A result correction is recorded as a new record_revision of the same logical test and attempt_no and keeps the baseline of the original execution.

[↑ Back to top](#top)

---

<a id="table-pq_step_execution"></a>
### 54. PQ Step Execution (`pq_step_execution`)

| Item | Definition |
|---|---|
| Description | Unit that records step completion status and links step evidence for a PQ execution attempt |
| Primary Key | `step_execution_id` |
| Main References (FK) | `execution_id, step_id, confirmed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 906 | Step Execution ID | `step_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 907 | Execution ID | `execution_id` | `uuid` | N | Y | `pq_execution.execution_id` | Y | - | N | Y | N | Y | References pq_execution.execution_id | `00000000-0000-0000-0000-000000000001` |
| 908 | Step ID | `step_id` | `uuid` | N | Y | `pq_step.step_id` | Y | - | N | Y | N | Y | References pq_step.step_id | `00000000-0000-0000-0000-000000000001` |
| 909 | Step Completed | `is_confirmed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Complete/cancel value of the detailed step on the screen. | `FALSE` |
| 910 | Completed By | `confirmed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 911 | Completed At | `confirmed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the step was marked as completed. | `2026-09-01T00:00:00Z` |
| 912 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 913 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 914 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 915 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_pq_step_execution_1` | UNIQUE | `execution_id, step_id` | UNIQUE(execution_id, step_id) |
| `rule_pq_step_execution_1` | Business validation | `execution_id, step_id` | The step and the execution must belong to the same protocol revision |
| `rule_pq_step_execution_2` | Business validation | `is_confirmed, confirmed_by, confirmed_at` | If is_confirmed = TRUE, confirmed_by and confirmed_at are required. After the result is recorded, completion/cancellation and evidence changes are locked and past records are preserved. |

#### Business Rules

Step attachment files are linked through evidence_link with PQ_STEP_EXECUTION / step_execution_id. On re-execution, new step execution rows are created.

[↑ Back to top](#top)

---

## VSR

<a id="table-vsr_assessment"></a>
### 55. Validation Summary Report (`vsr_assessment`)

| Item | Definition |
|---|---|
| Description | Manages the final validation conclusion and summary report for each project |
| Primary Key | `vsr_id` |
| Main References (FK) | `project_id, created_by, updated_by, workflow_instance_id, approval_signature_id` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 916 | VSR ID | `vsr_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | VSR assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 917 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Associated Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 918 | VSR Number | `vsr_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | VSR document number. The same number is kept across revisions. | `VSR-VP-SYS-008-20260422` |
| 919 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the validation summary report | `Validation Summary Report` |
| 920 | Validation Conclusion | `overall_conclusion` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Overall validation conclusion. Optional, and distinct from the business approval status. | - |
| 921 | Conclusion Details | `conclusion_remarks` | `text` | N | N | - | N | - | N | N | N | Y | Detailed explanation and conditions of the validation conclusion. NULL is allowed. | `Final approval possible after the OQ-GMP-02 deviation is resolved` |
| 922 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the VSR assessment document (e.g. v1.0, v1.1) | `v1.0` |
| 923 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision number of the VSR assessment document (e.g. 1, 2, 3...) | `1` |
| 924 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for the new creation or revision of the VSR assessment document | `Initial creation` |
| 925 | Is Current Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest version of the VSR assessment document (TRUE/FALSE) | `TRUE` |
| 926 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | VSR business confirmation status. DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected), CANCELLED (cancelled). Modifications after REJECTED or CANCELLED are handled as a new VSR revision. Distinct from the approval status of the separately generated deliverable. | `DRAFT` |
| 927 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 928 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the VSR author | `00000000-0000-0000-0000-000000000001` |
| 929 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 930 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the user who updated the VSR | `00000000-0000-0000-0000-000000000001` |
| 931 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time | `2026-09-01T00:00:00Z` |
| 932 | Source Fingerprint | `source_fingerprint` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | SHA-256 computed from the project's selected activities and the source revision references and summary values. Used to determine what needs re-confirmation when changes occur. | - |
| 933 | Aggregation Time | `captured_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time when the current VSR activity summary was generated. | `2026-09-01T00:00:00Z` |
| 934 | VSR Confirmation Workflow | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Request, review and approval route for VSR business confirmation. | `00000000-0000-0000-0000-000000000001` |
| 935 | VSR Confirmation Signature | `approval_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature on completion of the final VSR confirmation. | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_vsr_assessment_1` | UNIQUE | `project_id, vsr_no, revision_number` | UNIQUE(project_id, vsr_no, revision_number) |
| `rule_vsr_assessment_1` | Business validation | `project_id, vsr_no, revision_number` | At most 1 current revision per document (enforced by uq_vsr_assessment_current) |
| `rule_vsr_assessment_2` | Business validation | - | The approved VSR header and its child summary values are preserved. When the source changes, the values are re-aggregated into a new VSR revision as DRAFT and confirmation is obtained. |
| `rule_vsr_assessment_3` | Business validation | - | The VSR aggregates only the execution stages selected in the project and excludes RTM. The VSR's own confirmation is shown in the header and is not stored recursively in the detailed summary. |
| `ck_vsr_assessment_status` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `uq_vsr_assessment_current` | UNIQUE | `project_id, vsr_no, is_current_version` | UNIQUE (project_id, vsr_no) WHERE is_current_version=TRUE |
| `ck_vsr_assessment_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

This is a record for confirming the source status of the on-screen activity summary. Generation of the report body and PDF, and approval of that document, are managed separately in deliverable_document / deliverable_revision.

[↑ Back to top](#top)

---

<a id="table-vsr_item"></a>
### 56. VSR Activity Summary Item (`vsr_item`)

| Item | Definition |
|---|---|
| Description | Manages a summary of the documents, results, deviations and approval information for each validation activity |
| Primary Key | `vsr_item_id` |
| Main References (FK) | `vsr_id, created_by, updated_by, project_activity_id, document_revision_id` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 936 | VSR Item ID | `vsr_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | VSR activity summary item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 937 | VSR ID | `vsr_id` | `uuid` | N | Y | `vsr_assessment.vsr_id` | Y | - | N | Y | N | Y | Identifier of the parent VSR document | `00000000-0000-0000-0000-000000000001` |
| 938 | Activity Type | `activity_code` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Aggregated activity code: VP, VA, QIA, URS, FDS_GROUP (F&DS on screen), FRA, DQ, IQ, OQ, PQ. RTM excluded; the VSR's own status is shown in the header. | `IQ` |
| 939 | Document Number | `doc_no` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Document number of the activity as displayed at aggregation time. NULL if not approved or not generated. | `VP-SYS-008-20260422 · URS` |
| 940 | Revision No. | `revision_no` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Source approval version as displayed at aggregation time. NULL if not approved or not generated. Identification of the specific revision is preserved in source_revision_refs. | `ver1` |
| 941 | Deviation Count | `deviation_info` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Snapshot of the deviation count and open-item status displayed at aggregation time. Not edited manually on its own. | - |
| 942 | Conclusion / Status | `item_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | Activity approval status at aggregation time: DRAFT (in progress), REVIEW (under review), APPROVAL (under approval), APPROVED (approved), REJECTED (rejected), CANCELLED (cancelled). Snapshot automatically retrieved from the source business status. | `APPROVED` |
| 943 | Approver Name | `approver_name` | `varchar(50)` | N | N | - | N | - | N | N | Y | Y | Snapshot of the approver's display name at the time of source approval. | `Gildong Hong` |
| 944 | Approval Date | `approval_date` | `date` | N | N | - | N | - | N | N | N | Y | Snapshot of the approval date for screen display at the time of source approval. The exact signature time is retrieved from the source record. | `2024-02-20` |
| 945 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-26T00:00:00Z` |
| 946 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item author identifier | `00000000-0000-0000-0000-000000000001` |
| 947 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-26T00:00:00Z` |
| 948 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item updater identifier | `00000000-0000-0000-0000-000000000001` |
| 949 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time | `2026-09-01T00:00:00Z` |
| 950 | Project Activity ID | `project_activity_id` | `uuid` | N | Y | `project_activity.project_activity_id` | Y | - | N | Y | N | Y | Selected activity in the same project as the VSR. | `00000000-0000-0000-0000-000000000001` |
| 951 | Deliverable Revision ID | `document_revision_id` | `uuid` | N | Y | `deliverable_revision.document_revision_id` | N | - | N | Y | N | Y | Revision referenced when a deliverable document has been generated. For IQ/OQ/PQ it is a revision of a document_type=RECORD document, and for other activities a revision of a STAGE_DELIVERABLE document; the document's project_activity_id must be the same as this row's. May be NULL for stages that have only business item approval. | `00000000-0000-0000-0000-000000000001` |
| 952 | Source Revision References | `source_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | List of the exact business revisions on which the aggregation is based: [{table_name, record_id, version}]. table_name is the actual table name; IQ/OQ/PQ results use the execution_id of iq_execution / oq_execution / pq_execution, and FDS/DDS use the revision PK fds_spec.fds_id / dds_spec.dds_id. Empty array for unapproved stages. | `[]` |
| 953 | Source Summary Snapshot | `source_snapshot` | `jsonb` | N | N | - | Y | `'{}'::jsonb` | N | N | Y | Y | The columns / rows of the approval table and the approval information at aggregation time. Together with the source revision references it fixes the displayed values; empty object for unapproved stages. | `{}` |
| 954 | Activity Source Fingerprint | `source_fingerprint` | `varchar(64)` | N | N | - | N | - | N | N | N | Y | Hash of the source references and snapshot of the stage. NULL if there is no approved source. | - |
| 955 | Activity Display Order | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Order of the selected activities in the project. | `1` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_vsr_item_1` | UNIQUE | `vsr_id, project_activity_id` | UNIQUE(vsr_id, project_activity_id) |
| `rule_vsr_item_1` | Business validation | `vsr_id, project_activity_id` | activity_code must match the activity code of project_activity |
| `rule_vsr_item_2` | Business validation | `project_activity_id, document_revision_id` | project_activity_id / document_revision_id / source_revision_refs must point to the same project. The existence, type, revision and approval status of the referenced sources are validated. |
| `rule_vsr_item_3` | Business validation | `item_status` | If item_status = APPROVED, source_revision_refs must not be empty, and the source_snapshot and displayed values at the time of approval are preserved. |
| `rule_vsr_item_4` | Business validation | - | Displayed deviation values are aggregated from the deviation records of the activity. |
| `rule_vsr_item_5` | Business validation | - | Detail rows are not modified after the parent VSR is approved. A new aggregation is recorded as detail rows of a new VSR revision. |
| `ck_vsr_item_status` | CHECK | `item_status` | CHECK (item_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `rule_vsr_item_6` | Business validation | `document_revision_id, project_activity_id` | The deliverable_document.project_activity_id of document_revision_id must be the same as this row's project_activity_id; IQ/OQ/PQ activities reference only revisions of document_type=RECORD documents, and other activities only revisions of STAGE_DELIVERABLE documents. |

#### Business Rules

source_revision_refs is a polymorphic reference list of actual table names, revision PKs and versions, and is not a physical FK. It uses the same object structure as deliverable_revision.source_refs. Document file revisions and the business confirmation status are not treated as the same thing.

[↑ Back to top](#top)

---

## Workflow

<a id="table-workflow_instance"></a>
### 57. Workflow Instance (`workflow_instance`)

| Item | Definition |
|---|---|
| Description | Manages the progress of the review/approval workflow for each document |
| Primary Key | `workflow_instance_id` |
| Main References (FK) | `requested_by, created_by, updated_by, project_id, workflow_config_id` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 956 | Workflow Instance ID | `workflow_instance_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the per-document review/approval workflow | `00000000-0000-0000-0000-000000000001` |
| 957 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Name of the table subject to review/approval. Only the tables allowed for each approval_scope (see the table in the business rules) are permitted. | `fds_spec` |
| 958 | Target Record ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | N | N | Y | PK of the target table row. As a polymorphic reference it is not an actual FK; the target's existence, project and version are validated together. | `00000000-0000-0000-0000-000000000001` |
| 959 | Target Document Version | `target_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Display version or revision number of the approval target. For inventory it corresponds to system_asset.revision_number, and for a closure request to CLOSE-{request_version} | `v1.0` |
| 960 | Submitter ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who submitted the document to the review/approval process | `00000000-0000-0000-0000-000000000001` |
| 961 | Submission Time | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time when the document was submitted to the review/approval process | `2026-08-31T15:00:00Z` |
| 962 | Workflow Status | `workflow_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | Overall workflow status. IN_PROGRESS, APPROVED, REJECTED, CANCELLED. The instance is created as IN_PROGRESS at submission time and has no DRAFT (in progress) state | `IN_PROGRESS` |
| 963 | Current Step Order | `current_step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Order of the review/approval step currently being processed. On submission, the step_order of the first non-deleted step is stored; it is not fixed to 1 | `1` |
| 964 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the workflow ended through final approval, rejection or cancellation | `2026-08-31T17:00:00Z` |
| 965 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-31T15:00:00Z` |
| 966 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Creator of the workflow record | `00000000-0000-0000-0000-000000000001` |
| 967 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-31T16:00:00Z` |
| 968 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Last updater of the workflow record | `00000000-0000-0000-0000-000000000001` |
| 969 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time | `2026-09-01T00:00:00Z` |
| 970 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Required if the approval workflow is for project work. NULL for non-project approval workflows such as inventory | `00000000-0000-0000-0000-000000000001` |
| 971 | Approval Scope | `approval_scope` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | ITEM/PROTOCOL/RESULT/DOCUMENT/INVENTORY/CLOSURE/DISPOSAL/DEVIATION_ACTION/DEVIATION_COMPLETION/DEVIATION_CLOSE | `DOCUMENT` |
| 972 | Applied Approval Route Config ID | `workflow_config_id` | `uuid` | N | Y | `project_workflow_config.config_id` | N | - | N | Y | N | Y | Project approval route configuration used at submission. NULL for cases such as the inventory's own route | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_workflow_instance_1` | UNIQUE | `target_table_name, target_record_id, target_version, approval_scope, workflow_status` | UNIQUE (target_table_name, target_record_id, target_version, approval_scope) WHERE workflow_status = 'IN_PROGRESS' |
| `ck_workflow_instance_2` | CHECK | `workflow_status` | CHECK (workflow_status IN ('IN_PROGRESS','APPROVED','REJECTED','CANCELLED')) |
| `ck_workflow_instance_scope` | CHECK | `approval_scope` | CHECK (approval_scope IN ('ITEM','PROTOCOL','RESULT','DOCUMENT','INVENTORY','CLOSURE','DISPOSAL','DEVIATION_ACTION','DEVIATION_COMPLETION','DEVIATION_CLOSE')) |
| `rule_workflow_instance_3` | Business validation | `approval_scope, target_table_name, target_version` | approval_scope and the target_table_name and target_version format must match the per-scope target table in the business rules. |

#### Business Rules

On submission, the configured steps, modes and primary/substitute assignees are copied to workflow_step and workflow_step_assignee. Later changes to the default configuration do not change approval routes already in progress. The author, reviewers, approvers and substitutes must be assigned to distinct active accounts, and for inventory as well it is checked that the author, reviewer and approver are different.

Targets by approval_scope:

| approval_scope | target_table_name | target_version format |
|---|---|---|
| ITEM | vp_plan, qia_module_item, vendor_audit, requirement, fds_spec, dds_spec, dq_item, fra_item, vsr_assessment | version of the target revision |
| PROTOCOL | iq_item, oq_item, pq_item | version of the target test revision |
| RESULT | iq_execution, oq_execution, pq_execution | {attempt_no}-{record_revision} |
| DOCUMENT | deliverable_revision | version of the target document revision |
| INVENTORY | system_asset | system_asset.revision_number |
| CLOSURE | project_closure_request | CLOSE-{request_version} |
| DISPOSAL | vp_plan, qia_module_item, vendor_audit, requirement, fds_spec, dds_spec, dq_item, fra_item, iq_item, oq_item, pq_item | version of the revision to be disposed |
| DEVIATION_ACTION | deviation_action_round | ACTION-{round_number}-{revision_number} |
| DEVIATION_COMPLETION | deviation_completion_revision (closure_kind=COMPLETION) | COMPLETE-{action round}-{revision_number} |
| DEVIATION_CLOSE | deviation_completion_revision (closure_kind=CLOSE) | CLOSE-{action round}-{revision_number} |

Disposal of system_asset and approval route application (CONFIG_APPLY) are handled by electronic signatures in the respective tables, without a workflow.

When a workflow ends as REJECTED or CANCELLED, the approval status of the target revision is switched to the same value (REJECTED/CANCELLED) in the same transaction. If a disposal approval workflow (DISPOSAL) is rejected or cancelled, the target's approval status is not changed and lifecycle_status stays ACTIVE (for IQ/OQ/PQ test items, is_disposed=FALSE). For a closure request (CLOSURE), project_closure_request.status is switched to the same value. The general rule above is applied to deviation_action_round.approval_status for deviation actions (DEVIATION_ACTION), and to deviation_completion_revision.approval_status for deviation completion reports (DEVIATION_COMPLETION) and closures without re-execution (DEVIATION_CLOSE). Even if either of these two closing approval workflows is rejected or cancelled, deviation_status keeps its previous state (COMPLETION_PENDING or RERUN_ALLOWED), and the signature and approver columns of deviation are not filled.

Even for a single test item, PROTOCOL approval and execution RESULT approval are kept separate. The approval scope, target table, PK and version are identified together. The author is recorded through requested_by and the SUBMIT action history.

DEVIATION_ACTION targets the PK of deviation_action_round and ACTION-{round_number}-{revision_number}. DEVIATION_COMPLETION and DEVIATION_CLOSE target the PK of deviation_completion_revision and the COMPLETE- / CLOSE- version. When a rejected or cancelled action or closing report is modified and resubmitted, a new revision and a new workflow are created.

[↑ Back to top](#top)

---

<a id="table-workflow_step"></a>
### 58. Workflow Step (`workflow_step`)

| Item | Definition |
|---|---|
| Description | Manages the processing status and assignees of each step within a workflow |
| Primary Key | `workflow_step_id` |
| Main References (FK) | `workflow_instance_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 973 | Workflow Step ID | `workflow_step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the workflow review/approval step. Duplicate (workflow_instance_id, step_order) values are not allowed among rows where deleted_at IS NULL. | `00000000-0000-0000-0000-000000000001` |
| 974 | Workflow Instance ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | Y | - | N | Y | N | Y | Identifier of the workflow to which the step belongs | `00000000-0000-0000-0000-000000000001` |
| 975 | Step Order | `step_order` | `integer` | N | N | - | Y | - | N | Y | N | Y | Review/approval processing order within the same workflow | `1` |
| 976 | Step Type | `step_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Processing step type. REVIEW or APPROVE. SUBMIT/CANCEL are not step types; they are recorded as the action_type of the action history | `REVIEW` |
| 977 | Step Name | `step_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Review/approval step name displayed on screen | `Quality review` |
| 978 | Execution Mode | `execution_mode` | `varchar(20)` | N | N | - | Y | `'SERIAL'` | N | N | N | Y | SERIAL=serial, PARALLEL=parallel. Processing order of multiple assignees in the same step | `SERIAL` |
| 979 | Step Status | `step_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | Step status. PENDING, IN_PROGRESS, APPROVED, REJECTED, SKIPPED | `PENDING` |
| 980 | Due Date | `due_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Scheduled deadline for review/approval processing of this step | `2026-09-02T18:00:00Z` |
| 981 | Completion Time | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the step was completed by approval, rejection, etc. | `2026-09-01T10:00:00Z` |
| 982 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-08-31T15:00:00Z` |
| 983 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Creator of the workflow step | `00000000-0000-0000-0000-000000000001` |
| 984 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last update time (UTC) | `2026-08-31T16:00:00Z` |
| 985 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Last updater of the workflow step | `00000000-0000-0000-0000-000000000001` |
| 986 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion time | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_workflow_step_1` | UNIQUE | `workflow_instance_id, step_order, deleted_at` | UNIQUE (workflow_instance_id, step_order) WHERE deleted_at IS NULL |
| `ck_workflow_step_2` | CHECK | `execution_mode` | CHECK (execution_mode IN ('SERIAL','PARALLEL')) |
| `ck_workflow_step_3` | CHECK | `step_order` | CHECK (step_order >= 1) |
| `ck_workflow_step_type` | CHECK | `step_type` | CHECK (step_type IN ('REVIEW','APPROVE')) |
| `ck_workflow_step_status` | CHECK | `step_status` | CHECK (step_status IN ('PENDING','IN_PROGRESS','APPROVED','REJECTED','SKIPPED')) |

#### Business Rules

Primary assignees, substitutes and the assignee order are stored in workflow_step_assignee. step_order is the step order, and execution_mode distinguishes how the assignees within the same step process it.

Even for a parallel step, processing by all designated assignees must be completed before proceeding to the next step. A review step leaves a REVIEW action history record, and an approval step an APPROVE record.

[↑ Back to top](#top)

---

<a id="table-workflow_step_assignee"></a>
### 59. Approval Workflow Step Assignee (`workflow_step_assignee`)

| Item | Definition |
|---|---|
| Description | Multiple primary assignees and substitute assignees of a review/approval step |
| Primary Key | `assignment_id` |
| Main References (FK) | `workflow_step_id, assignee_id, substitute_user_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 987 | Assignment ID | `assignment_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 988 | Approval Step ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | Y | - | N | Y | N | Y | References workflow_step.workflow_step_id | `00000000-0000-0000-0000-000000000001` |
| 989 | Primary Assignee ID | `assignee_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 990 | Substitute Assignee ID | `substitute_user_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 991 | Assignee Order | `assignee_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Processing order in a serial step | `1` |
| 992 | Processing Status | `assignment_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | N | N | Y | PENDING/IN_PROGRESS/APPROVED/REJECTED/SKIPPED | `PENDING` |
| 993 | Completion Time | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Completion time value | `2026-09-01T00:00:00Z` |
| 994 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation time value | `2026-09-01T00:00:00Z` |
| 995 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 996 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Update time value | `2026-09-01T00:00:00Z` |
| 997 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_workflow_step_assignee_1` | UNIQUE | `workflow_step_id, assignee_order` | UNIQUE (workflow_step_id, assignee_order) |
| `uq_workflow_step_assignee_2` | UNIQUE | `workflow_step_id, assignee_id` | UNIQUE (workflow_step_id, assignee_id) |
| `ck_workflow_step_assignee_3` | CHECK | `assignee_order` | CHECK (assignee_order >= 1) |
| `rule_workflow_step_assignee_4` | Business validation | - | The primary assignee and the substitute assignee cannot be the same. |
| `ck_workflow_step_assignee_status` | CHECK | `assignment_status` | CHECK (assignment_status IN ('PENDING','IN_PROGRESS','APPROVED','REJECTED','SKIPPED')) |

#### Business Rules

The actual actor is recorded in approval_action.actor_id. Either the primary assignee or an authorized substitute assignee processes a given assignment.

In serial mode, an assignment can be processed after the preceding assignment is completed; in parallel mode, any incomplete assignment of the current step can be processed.

[↑ Back to top](#top)

---

<a id="table-approval_action"></a>
### 60. Approval Action History (`approval_action`)

| Item | Definition |
|---|---|
| Description | Manages the actual action history of each step, such as review, approval and rejection |
| Primary Key | `approval_action_id` |
| Main References (FK) | `workflow_step_id, workflow_step_assignee_id, actor_id, signature_id` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 998 | Approval Action ID | `approval_action_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the review/approval action history record | `00000000-0000-0000-0000-000000000001` |
| 999 | Workflow Step ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | Y | - | N | Y | N | Y | Workflow step to which the action history belongs. SUBMIT is linked to the first non-deleted step, CANCEL to the current step in progress (the first step if not yet started), and REVIEW/APPROVE/REJECT to the step actually processed. If there is no step to attribute it to, no history record is created. | `00000000-0000-0000-0000-000000000001` |
| 1000 | Processing Assignment ID | `workflow_step_assignee_id` | `uuid` | N | Y | `workflow_step_assignee.assignment_id` | N | - | N | Y | N | Y | Assignment for REVIEW/APPROVE/REJECT actions. NULL for SUBMIT/CANCEL | `00000000-0000-0000-0000-000000000001` |
| 1001 | Action Type | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Type of action performed. SUBMIT, REVIEW, APPROVE, REJECT, CANCEL | `APPROVE` |
| 1002 | Actor ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who actually performed the review, approval or rejection | `00000000-0000-0000-0000-000000000001` |
| 1003 | Action Comment | `action_comment` | `text` | N | N | - | N | - | N | N | N | Y | Comment entered during review/approval. For CANCEL, stores the cancellation reason and is required | `No issues found in review` |
| 1004 | Rejection Reason | `rejection_reason` | `text` | N | N | - | N | - | N | N | N | Y | Rejection reason entered for a REJECT action. Required for REJECT | `Evidence files need to be supplemented` |
| 1005 | Electronic Signature ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature record linked to the action. If a signature exists, signer_id must be the same as actor_id, and the target table, record and version must match the workflow target. For SUBMIT/REVIEW/APPROVE/REJECT, the signature action must be the same as action_type. CANCEL records the cancellation reason and an audit trail without a signature. | `00000000-0000-0000-0000-000000000001` |
| 1006 | Action Time | `acted_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when the submission, review, approval or rejection was actually processed | `2026-09-01T10:00:00Z` |
| 1007 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation time (UTC) | `2026-09-01T10:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_approval_action_1` | Business validation | - | For REVIEW/APPROVE/REJECT, workflow_step_assignee_id is required. For SUBMIT/CANCEL it is NULL. |
| `rule_approval_action_2` | Business validation | - | Only one valid REVIEW/APPROVE/REJECT action record exists per assignment. |
| `rule_approval_action_3` | Business validation | - | SUBMIT/REVIEW/APPROVE/REJECT require an electronic signature. CANCEL leaves a cancellation reason (action_comment) and an audit trail record without an electronic signature. |
| `uq_approval_assignee_action` | UNIQUE | `workflow_step_assignee_id, action_type` | UNIQUE (workflow_step_assignee_id) WHERE action_type IN ('REVIEW','APPROVE','REJECT') |
| `ck_approval_action_type` | CHECK | `action_type` | CHECK (action_type IN ('SUBMIT','REVIEW','APPROVE','REJECT','CANCEL')) |
| `ck_approval_action_reject_reason` | CHECK | `action_type, rejection_reason` | CHECK (action_type <> 'REJECT' OR (rejection_reason IS NOT NULL AND length(trim(rejection_reason)) > 0)) |
| `ck_approval_action_cancel_reason` | CHECK | `action_type, action_comment` | CHECK (action_type <> 'CANCEL' OR (action_comment IS NOT NULL AND length(trim(action_comment)) > 0)) |

#### Business Rules

The workflow_step_id of the assignment and the workflow_step_id of the history record must be the same. The actual actor is the primary assignee or the substitute assignee and must match the electronic signature's signer_id.

SUBMIT is recorded on the first step, and CANCEL on the current step (the first step if not yet started). The signature's target table, PK, version and action must match workflow_instance and action_type.

A rejection on the same assignment is not overwritten with an approval. On resubmission, a new workflow or new steps and assignments are created. After locking the target step and assignment, the current progress status, whether it is still unprocessed and the substitute's eligibility are re-checked, and the signature, action history, and step and overall status transitions are saved atomically.

[↑ Back to top](#top)

---

<a id="table-project_workflow_config"></a>
### 61. Project Approval Route Configuration (`project_workflow_config`)

| Item | Definition |
|---|---|
| Description | Project default and per-activity author/review/approval route configuration |
| Primary Key | `config_id` |
| Main References (FK) | `project_id, project_activity_id, applied_signature_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1008 | Approval Route Config ID | `config_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1009 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 1010 | Activity ID | `project_activity_id` | `uuid` | N | Y | `project_activity.project_activity_id` | N | - | N | Y | N | Y | Specified for a per-activity configuration. NULL for the project default configuration | `00000000-0000-0000-0000-000000000001` |
| 1011 | Config Version | `config_version` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Configuration version, incremented when it is changed after being applied | `1` |
| 1012 | Route Definition | `route_definition` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Author stage (author_stage), array of review stages (review_stages), array of approval stages (approval_stages). Each stage stores its order, SERIAL/PARALLEL and an array of primary/substitute assignees | - |
| 1013 | Applied | `is_applied` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Whether this configuration is currently applied. When a new configuration is applied, the previous row is switched to FALSE, and its earlier application signature, time and original content are preserved. | `FALSE` |
| 1014 | Application Signature ID | `applied_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | References electronic_signature.signature_id | `00000000-0000-0000-0000-000000000001` |
| 1015 | Applied At | `applied_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Application time value | `2026-09-01T00:00:00Z` |
| 1016 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation time value | `2026-09-01T00:00:00Z` |
| 1017 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1018 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Update time value | `2026-09-01T00:00:00Z` |
| 1019 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_project_workflow_config_1` | Business validation | - | The project_activity_id of a per-activity configuration must belong to the same project_id. |
| `rule_project_workflow_config_2` | Business validation | `project_id, project_activity_id, config_version` | (project_id, config_version) is unique for project default configurations, and (project_activity_id, config_version) for per-activity configurations. There is at most one currently applied configuration per project / per activity. |
| `rule_project_workflow_config_3` | Business validation | `is_applied` | When is_applied=TRUE, applied_signature_id and applied_at are required. |
| `uq_workflow_config_project_version` | UNIQUE | `project_id, config_version, project_activity_id` | UNIQUE (project_id, config_version) WHERE project_activity_id IS NULL |
| `uq_workflow_config_activity_version` | UNIQUE | `project_activity_id, config_version` | UNIQUE (project_activity_id, config_version) WHERE project_activity_id IS NOT NULL |
| `uq_workflow_config_project_applied` | UNIQUE | `project_id, project_activity_id, is_applied` | UNIQUE (project_id) WHERE project_activity_id IS NULL AND is_applied=TRUE |
| `uq_workflow_config_activity_applied` | UNIQUE | `project_activity_id, is_applied` | UNIQUE (project_activity_id) WHERE project_activity_id IS NOT NULL AND is_applied=TRUE |

#### Business Rules

If a per-activity approval route exists it is used; otherwise the project default approval route is used. No execution stage configuration is created for RTM.

Each stage in route_definition has the structure {step_order, execution_mode, assignees:[{user_id, substitute_user_id, assignee_order}]}. User identifiers inside the JSON are not physical FKs, so existence, permissions and duplicates are checked on save. The author, reviewers, approvers and substitute assignees must be distinct active personal accounts with project permissions.

The author stage designates author candidates. The actual submitter is recorded in workflow_instance.requested_by. For execution, the approval route is copied to workflow_step and workflow_step_assignee.

Switching the applied configuration is performed with the project locked. Clearing the previous is_applied and applying the new configuration are saved in the same transaction. is_applied is a management state and, by design, is not included in the signed original content of CONFIG_APPLY. The already applied route_definition, config_version, signature and application time are not changed, and a per-activity configuration must reference a project_activity of the same project.

[↑ Back to top](#top)

---

## Traceability

<a id="table-traceability_link"></a>
### 62. Common Traceability Link (`traceability_link`)

| Item | Definition |
|---|---|
| Description | Management of links, selected on the screen, between URS, design documents, DQ, FRA and test items, and of the source relationships for the dashboard RTM |
| Primary Key | `traceability_link_id` |
| Main References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1020 | Traceability Link ID | `traceability_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of a traceability link between URS, FDS, DDS, DQ, FRA, IQ, OQ and PQ items. For non-deleted data, duplicate (project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type) combinations are not allowed. | `00000000-0000-0000-0000-000000000001` |
| 1021 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | ID of the project to which the traceability link belongs | `00000000-0000-0000-0000-000000000001` |
| 1022 | Source Entity Type | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Link target type: REQUIREMENT, FDS_SPEC, DDS_SPEC, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM. Links the PK of a specific revision row of each type. | `REQUIREMENT` |
| 1023 | Source Entity ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | PK of a specific revision row in the table corresponding to the target type. A polymorphic reference, not a physical FK; it must not be replaced with a logical key or display number. | `00000000-0000-0000-0000-000000000001` |
| 1024 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Link target type: REQUIREMENT, FDS_SPEC, DDS_SPEC, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM. Links the PK of a specific revision row of each type. | `OQ_ITEM` |
| 1025 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | PK of a specific revision row in the table corresponding to the target type. A polymorphic reference, not a physical FK; it must not be replaced with a logical key or display number. | `00000000-0000-0000-0000-000000000001` |
| 1026 | Link Type | `link_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | IMPLEMENTED_BY, ASSESSED_BY, VERIFIED_BY, MITIGATED_BY | `VERIFIED_BY` |
| 1027 | Link Rationale | `link_reason` | `text` | N | N | - | N | - | N | N | N | Y | Business rationale for linking the two deliverable items | `URS requirement verified by IQ test` |
| 1028 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Traceability link creation time (UTC) | `2026-09-01T10:00:00Z` |
| 1029 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who created the traceability link | `00000000-0000-0000-0000-000000000001` |
| 1030 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Traceability link last update time (UTC) | `2026-09-01T10:00:00Z` |
| 1031 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who last updated the traceability link | `00000000-0000-0000-0000-000000000001` |
| 1032 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Traceability link soft-deletion time | `2026-09-02T10:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_traceability_link_1` | UNIQUE | `project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type` | UNIQUE(project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type) WHERE deleted_at IS NULL |
| `rule_traceability_link_2` | Business validation | - | Both ends of a link must belong to the same project and must exist as a registered target type and an actual revision row. |
| `rule_traceability_link_3` | Business validation | - | VERIFIED_BY for REQUIREMENT → IQ_ITEM / OQ_ITEM / PQ_ITEM is the screen relationship that links multiple URS items to a single test. |
| `rule_traceability_link_4` | Business validation | - | FDS_SPEC / DDS_SPEC refer to fds_spec.fds_id / dds_spec.dds_id. |
| `rule_traceability_link_5` | Business validation | - | The dashboard RTM queries the links and the currently valid source and execution statuses. No RTM-specific documents, tests or approval statuses are additionally created. |
| `rule_traceability_link_6` | Business validation | - | When a link is revised, the source relationships of past approval targets are not deleted or overwritten. A new relationship is created for the new revision and past links are preserved. |
| `ck_traceability_link_source_type` | CHECK | `source_entity_type` | CHECK (source_entity_type IN ('REQUIREMENT','FDS_SPEC','DDS_SPEC','DQ_ITEM','FRA_ITEM','IQ_ITEM','OQ_ITEM','PQ_ITEM')) |
| `ck_traceability_link_target_type` | CHECK | `target_entity_type` | CHECK (target_entity_type IN ('REQUIREMENT','FDS_SPEC','DDS_SPEC','DQ_ITEM','FRA_ITEM','IQ_ITEM','OQ_ITEM','PQ_ITEM')) |
| `ck_traceability_link_type` | CHECK | `link_type` | CHECK (link_type IN ('IMPLEMENTED_BY','ASSESSED_BY','VERIFIED_BY','MITIGATED_BY')) |

#### Business Rules

Coverage is aggregated based on whether URS items are linked to tests, and is distinguished from the test PASS ratio or the approved ratio. Direct URS links of FDS/DDS documents are stored only for relationships selected on the file approval screen.

[↑ Back to top](#top)

---

## Deviation

<a id="table-deviation"></a>
### 63. Deviation Management (`deviation`)

| Item | Definition |
|---|---|
| Description | Management of the reason and immediate action following a test FAIL, re-execution, corrective and preventive action, completion report, and closure processing |
| Primary Key | `deviation_id` |
| Main References (FK) | `project_id, approved_by, created_by, updated_by, action_workflow_id, action_signature_id, completion_signature_id, closure_signature_id, current_action_round_id, current_completion_revision_id` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1033 | Deviation ID | `deviation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the deviation | `00000000-0000-0000-0000-000000000001` |
| 1034 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Validation project in which the deviation occurred | `00000000-0000-0000-0000-000000000001` |
| 1035 | Originating Entity Type | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Type of the original execution that failed: IQ_EXECUTION, OQ_EXECUTION, PQ_EXECUTION. | `OQ_EXECUTION` |
| 1036 | Originating Entity ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | execution_id in the corresponding execution table. A polymorphic reference pointing to a specific attempt or corrected result; not a physical FK. | `00000000-0000-0000-0000-000000000001` |
| 1037 | Deviation Number | `deviation_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Deviation management number within the project | `DEV-001` |
| 1038 | Deviation Title | `title` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Report display title built from the test item name and the deviation number. Not a separately required input. | `Expected result mismatch` |
| 1039 | Deviation Reason | `description` | `text` | N | N | - | Y | - | N | N | N | Y | Display summary of the deviation reason from the current action revision. The original failure reason is retrieved from the execution record of source_entity_id. | `Mismatch between expected and actual results during OQ execution` |
| 1040 | Deviation Status | `deviation_status` | `varchar(30)` | N | N | - | Y | `'ACTION_PENDING'` | N | Y | N | Y | ACTION_PENDING (awaiting reason/action approval), RERUN_ALLOWED (re-execution allowed), COMPLETION_PENDING (awaiting completion report), CLOSED (closed). | `ACTION_PENDING` |
| 1041 | Approved At | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Final signature time of the completion report approval or of the closure processing for which a reason was entered. | `2026-09-03T17:00:00Z` |
| 1042 | Approver ID | `approved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who gave the final signature for the completion report approval or the closure processing. | `00000000-0000-0000-0000-000000000001` |
| 1043 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Deviation creation time (UTC) | `2026-09-01T10:00:00Z` |
| 1044 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the deviation | `00000000-0000-0000-0000-000000000001` |
| 1045 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Deviation last update time (UTC) | `2026-09-01T10:00:00Z` |
| 1046 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last updated the deviation | `00000000-0000-0000-0000-000000000001` |
| 1047 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deviation soft-deletion time | `2026-09-02T10:00:00Z` |
| 1048 | Immediate Action | `immediate_action` | `text` | N | N | - | Y | - | N | N | N | Y | Display summary of the immediate action from the current action revision. The approved original content of each round is preserved in deviation_action_round. | - |
| 1049 | Reason/Action Approval Status | `action_approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | Summary of the approval status of the current action revision. DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED / CANCELLED. Not edited independently. | `DRAFT` |
| 1050 | Reason/Action Approval Workflow | `action_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Current approval procedure synchronized from the workflow_instance_id of current_action_round_id. Previous rounds are retrieved from deviation_action_round; not edited independently. | `00000000-0000-0000-0000-000000000001` |
| 1051 | Reason/Action Final Signature | `action_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Final approval signature synchronized from the signature_id of the current action round. Past signatures are preserved per round. | `00000000-0000-0000-0000-000000000001` |
| 1052 | Latest Re-execution ID | `rerun_execution_id` | `uuid` | N | N | - | N | - | N | N | N | Y | PK of the valid result revision of the latest re-execution. Must belong to the same logical test and attempt_no as the round's initial re-execution. If the initial execution row of the re-execution is aborted (ABORTED), the link is released; if a result correction is finalized as RECORDED, it is updated to the correction row. Cancelling a correction that is still being written has no effect. The table is identified by source_entity_type. | `00000000-0000-0000-0000-000000000001` |
| 1053 | Corrective and Preventive Action | `corrective_action` | `text` | N | N | - | N | - | N | N | N | Y | Corrective and preventive action synchronized from the approved completion report revision (deviation_completion_revision). Authoring and editing are done in the revision table; not edited independently. | - |
| 1054 | Completion Report | `completion_report` | `text` | N | N | - | N | - | N | N | N | Y | Completion report synchronized from the approved completion report revision. Authoring and editing are done in the revision table; not edited independently. | - |
| 1055 | Completion Report Approval Signature | `completion_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Synchronized from the final approval signature (signature_id) of the approved completion report revision. | `00000000-0000-0000-0000-000000000001` |
| 1056 | Closure Processing Reason | `closure_reason` | `text` | N | N | - | N | - | N | N | N | Y | Closure reason synchronized from the approved closure-without-re-execution revision (closure_kind=CLOSE). Authoring and editing are done in the revision table; not edited independently. | - |
| 1057 | Closure Processing Signature | `closure_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Synchronized from the final approval signature of the approved closure-without-re-execution revision. Distinct from the completion report approval. | `00000000-0000-0000-0000-000000000001` |
| 1058 | Current Deviation Action Revision | `current_action_round_id` | `uuid` | N | Y | `deviation_action_round.action_round_id` | N | - | N | Y | N | Y | Current original-content revision in the last action round of this deviation. Required at the end of the transaction that creates the initial failure and the first action draft | - |
| 1059 | Current Closure Report Revision | `current_completion_revision_id` | `uuid` | N | Y | `deviation_completion_revision.completion_revision_id` | N | - | N | Y | N | Y | Current closure report revision of this deviation (is_current_revision=TRUE). NULL before a closure report is written | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_deviation_1` | UNIQUE | `project_id, deviation_no` | UNIQUE(project_id, deviation_no) |
| `uq_deviation_source` | UNIQUE | `source_entity_type, source_entity_id` | UNIQUE (source_entity_type, source_entity_id) WHERE deleted_at IS NULL |
| `rule_deviation_8` | Business validation | `source_entity_type, source_entity_id, deviation_status` | There is at most 1 deviation that is not closed (deviation_status<>'CLOSED') for the same logical test (item_key). On creation, a lock is taken at the logical-test level and then the check is performed. |
| `ck_deviation_action_approval_status` | CHECK | `action_approval_status` | CHECK (action_approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `rule_deviation_1` | Business validation | `project_id, source_entity_id, deviation_no` | source_entity_id and rerun_execution_id must belong to the same project and the same logical test |
| `rule_deviation_2` | Business validation | - | Transition to RERUN_ALLOWED requires an APPROVED reason/action and action_signature_id. |
| `rule_deviation_3` | Business validation | `source_entity_id, rerun_execution_id, current_action_round_id` | The deviation's original source_entity_id keeps the initial failure record. If the re-execution under an approved action is FAIL, no new deviation is created; instead, the valid result revision of that re-execution is recorded in deviation.rerun_execution_id, a deviation_action_round with the next round_number (failed_execution_id = that re-execution's execution_id) is created, and current_action_round_id is updated to the new round. |
| `rule_deviation_4` | Business validation | `corrective_action, completion_report, completion_signature_id` | After COMPLETION_PENDING, the deviation is closed by the final approval of a completion report revision (closure_kind=COMPLETION), and corrective_action / completion_report / completion_signature_id synchronized from the approved version are required. Immediately before the completion report approval, it is checked that the valid result of the latest re-execution is PASS and that the record_status of that result revision is APPROVED. |
| `rule_deviation_5` | Business validation | `closure_reason, closure_signature_id` | When closing without re-execution, the status must be RERUN_ALLOWED and there must be no re-execution (rerun_execution_id) linked to the current action round; the deviation is closed by the final approval of a closure-without-re-execution revision (closure_kind=CLOSE), and closure_reason / closure_signature_id synchronized from the approved version are required. The two closure paths are kept distinct, and past signatures and results are not deleted. |
| `rule_deviation_6` | Business validation | `approved_by, completion_signature_id, closure_signature_id` | If CLOSED, exactly one of completion_signature_id / closure_signature_id, plus approved_by / approved_at, are required, and they must match the final signer and signature time. |
| `rule_deviation_7` | Business validation | - | The re-execution judgment is retrieved from the qualification_result of rerun_execution_id. The original failure reason and immediate action can be reproduced as of that time from the execution record. |
| `rule_deviation_9` | Business validation | `deviation_status` | deviation_status allows only the following transitions. ACTION_PENDING→RERUN_ALLOWED: final approval of the current action round. RERUN_ALLOWED→ACTION_PENDING: registration of a FAIL re-execution result (in the same transaction as creating the next action round). RERUN_ALLOWED→COMPLETION_PENDING: registration of a PASS re-execution result. RERUN_ALLOWED→CLOSED: approval of closure without re-execution (rule_deviation_5). COMPLETION_PENDING→CLOSED: completion report approval (rule_deviation_4). COMPLETION_PENDING→ACTION_PENDING: correction of the re-execution result to FAIL while no completion report approval workflow is in progress (in the same transaction as creating the next action round). Aborting a re-execution (initial execution row ABORTED) does not change the status (RERUN_ALLOWED is kept), and cancelling a result correction that is still being written does not change the status either. Transitions after CLOSED and any other transitions are not allowed. |
| `rule_deviation_10` | Business validation | `corrective_action, completion_report, closure_reason, current_completion_revision_id` | The original content of the completion report and the closure without re-execution is authored and revised in deviation_completion_revision; the corresponding columns of this table are summaries synchronized from the approved version and are not edited directly. Edits after rejection or cancellation are handled as a new closure report revision. |
| `ck_deviation_source_type` | CHECK | `source_entity_type` | CHECK (source_entity_type IN ('IQ_EXECUTION','OQ_EXECUTION','PQ_EXECUTION')) |
| `ck_deviation_status` | CHECK | `deviation_status` | CHECK (deviation_status IN ('ACTION_PENDING','RERUN_ALLOWED','COMPLETION_PENDING','CLOSED')) |

#### Business Rules

The closing user and closing time are kept consistent with the final electronic signature values.

The initial FAIL registration and the creation of the first action round are processed in a single transaction. current_action_round_id must belong to this deviation and be the row with is_current_revision=TRUE in the last round_number. The current reason, action, approval status, Workflow and signature are synchronized from that row, and past original content is retrieved from the round rows. If the latest re-execution is FAIL, the existing deviation is kept, the next action round is created, and deviation_status is returned to ACTION_PENDING. If the re-execution is PASS, the status transitions to COMPLETION_PENDING.

The approval workflow and signature for the completion report and the closure without re-execution target the PK of deviation_completion_revision and follow the electronic signature version rules. signed_payload fixes the basis action round, the initial failure PK, the exact result revision PK of the latest re-execution, and the completion report or closure reason. Closing by completion report requires the latest re-execution to be PASS. After CLOSED, the reason, action, completion report, signatures and result references are not modified.

[↑ Back to top](#top)

---

<a id="table-deviation_action_round"></a>
### 64. Deviation Action Round (`deviation_action_round`)

| Item | Definition |
|---|---|
| Description | Revision and approval of the original action content per failed execution, and the link to the permitted re-execution |
| Primary Key | `action_round_id` |
| Main References (FK) | `deviation_id, workflow_instance_id, signature_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period. Approved original content and past link history are not deleted. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1060 | Deviation Action Revision ID | `action_round_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Identifier of a specific original-content revision written within one action round | - |
| 1061 | Deviation ID | `deviation_id` | `uuid` | N | Y | `deviation.deviation_id` | Y | - | N | Y | N | Y | Source deviation | - |
| 1062 | Action Round | `round_number` | `integer` | N | N | - | Y | - | N | N | N | Y | 1 for the initial failure. If the re-execution after approval fails, the next round is created. | `1` |
| 1063 | Original-Content Revision within Round | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Incremented when the content is edited and resubmitted after rejection for the same failure. | `1` |
| 1064 | Is Current Revision in Round | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Only one current authored revision exists per deviation and action round | - |
| 1065 | Failed Execution ID of the Round | `failed_execution_id` | `uuid` | N | N | - | Y | - | N | N | N | Y | Execution PK corresponding to deviation.source_entity_type. For the first round, the initial failure; for later rounds, the failed re-execution of the previous round. A polymorphic reference, not a physical FK | - |
| 1066 | Deviation Reason per Round | `description` | `text` | N | N | - | Y | - | N | N | N | Y | Original reason content subject to approval for that failure | - |
| 1067 | Immediate Action per Round | `immediate_action` | `text` | N | N | - | Y | - | N | N | N | Y | Original content of the action taken for that failure | - |
| 1068 | Action Approval Status | `approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED / CANCELLED. REJECTED and CANCELLED are terminal states of this original-content revision | - |
| 1069 | Action Approval Procedure | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | DEVIATION_ACTION approval workflow targeting this action_round_id and ACTION-{round_number}-{revision_number} | - |
| 1070 | Action Final Approval Signature | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Final approval signature of the corresponding original action content revision | - |
| 1071 | Re-execution ID Permitted by Approval | `rerun_execution_id` | `uuid` | N | N | - | N | - | N | N | N | Y | PK of the re-execution started under this approval. The table is identified by source_entity_type. NULL when closed without re-execution, or when the link was released because the re-execution was aborted (ABORTED) | - |
| 1072 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Original-content revision creation time | - |
| 1073 | Author | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Author of the original action content | - |
| 1074 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time of draft editing or management status change | - |
| 1075 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Processing user | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_deviation_action_revision` | UNIQUE | `deviation_id, round_number, revision_number` | UNIQUE (deviation_id, round_number, revision_number) |
| `uq_deviation_action_current` | UNIQUE | `deviation_id, round_number, is_current_revision` | UNIQUE (deviation_id, round_number) WHERE is_current_revision=TRUE |
| `uq_deviation_action_approved` | UNIQUE | `deviation_id, round_number, approval_status` | UNIQUE (deviation_id, round_number) WHERE approval_status='APPROVED' |
| `ck_deviation_action_sequence` | CHECK | `round_number, revision_number` | CHECK (round_number >= 1 AND revision_number >= 1) |
| `ck_deviation_action_status` | CHECK | `approval_status` | CHECK (approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_deviation_action_approval` | CHECK | `approval_status, workflow_instance_id, signature_id` | CHECK (approval_status <> 'APPROVED' OR (workflow_instance_id IS NOT NULL AND signature_id IS NOT NULL)) |
| `ck_deviation_action_rerun` | CHECK | `rerun_execution_id, approval_status` | CHECK (rerun_execution_id IS NULL OR approval_status='APPROVED') |
| `rule_deviation_action_chain` | Business validation | `failed_execution_id, rerun_execution_id` | Failures and re-executions belong to the same project, stage and logical test as the deviation. The first failure is the deviation's source_entity_id; a failure in a later round is the re-execution of the previously approved round, or a corrected result revision belonging to the same protocol and attempt_no as that re-execution, and must be FAIL. Revisions within the same round keep the same failed_execution_id. |

#### Business Rules

The original action content is locked after submission. To change the content after rejection or cancellation, a new revision_number and PK are created under the same round_number, and the previous row, Workflow and signature are preserved. An approved round does not get a new original-content revision. A re-execution FAIL is linked to the next round_number.

The approval Workflow and signature point to this table's exact action_round_id and ACTION-{round_number}-{revision_number}, not to deviation. The original reason/action content and failed_execution_id are included in the signed original content. After approval, the original action content, failure reference, approval status and final signature are not changed. Thereafter, only the initial re-execution link and management metadata such as the updater and update time may be updated. Once first linked, rerun_execution_id is not overwritten with a different attempt. However, if the initial execution row (record_revision=1) of the linked re-execution is ABORTED, the link may be released (NULL) and a new re-execution linked (cancelling a result correction row does not affect the link); the value before release and the aborted execution row are preserved in the Audit Trail and the corresponding execution table.

For REVIEW, APPROVAL and APPROVED, workflow_instance_id is required, and the target, scope and version of the approval Workflow, as well as the final signature and signed original content, must match this action revision.

Re-execution creation locks the deviation and the approved round, checks deviation_status=RERUN_ALLOWED, the approval status, the current round and the absence of an existing re-execution, and then processes test execution creation, round linking and the deviation summary update in a single transaction. An execution correction uses a new execution PK but keeps the original re-execution link, and the correction lineage is retrieved via the record_revision of the same attempt_no. The determination of the next failure and the completion report explicitly identify the valid result revision as of that point in time.

[↑ Back to top](#top)

---

<a id="table-deviation_completion_revision"></a>
### 65. Deviation Closure Report Revision (`deviation_completion_revision`)

| Item | Definition |
|---|---|
| Description | Revision and approval of the original content of the deviation completion report (corrective and preventive action, completion report) and of the reason for closure without re-execution |
| Primary Key | `completion_revision_id` |
| Main References (FK) | `deviation_id, action_round_id, workflow_instance_id, signature_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period. Submitted or approved original content and rejected or cancelled revisions are not deleted. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1076 | Closure Report Revision ID | `completion_revision_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Identifier of a specific original-content revision of the closure report | `00000000-0000-0000-0000-000000000001` |
| 1077 | Deviation ID | `deviation_id` | `uuid` | N | Y | `deviation.deviation_id` | Y | - | N | Y | N | Y | Deviation to be closed | `00000000-0000-0000-0000-000000000001` |
| 1078 | Closure Kind | `closure_kind` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | COMPLETION=completion report after successful re-execution, CLOSE=closure without re-execution | `COMPLETION` |
| 1079 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 per deviation. Editing after rejection or cancellation creates a new row with the next number | `1` |
| 1080 | Is Current Revision | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the current closure report revision of the same deviation | `TRUE` |
| 1081 | Basis Action Round | `action_round_id` | `uuid` | N | Y | `deviation_action_round.action_round_id` | Y | - | N | Y | N | Y | Current action round at the time of authoring (deviation.current_action_round_id) | `00000000-0000-0000-0000-000000000001` |
| 1082 | Basis Re-execution Result ID | `rerun_execution_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | PK of the valid re-execution result revision that is the basis for COMPLETION. A polymorphic reference whose table is identified by deviation.source_entity_type; not a physical FK. NULL for CLOSE | `00000000-0000-0000-0000-000000000001` |
| 1083 | Corrective and Preventive Action | `corrective_action` | `text` | N | N | - | N | - | N | N | N | Y | Original corrective and preventive action content for COMPLETION. Required on submission | `Inspected 3 units of the same equipment model; added an alarm threshold item to the installation checklist` |
| 1084 | Completion Report | `completion_report` | `text` | N | N | - | N | - | N | N | N | Y | Original completion report content for COMPLETION. Required on submission | `Alarm threshold setting error. Corrected to 8 degrees and retest passed` |
| 1085 | Closure Reason | `closure_reason` | `text` | N | N | - | N | - | N | N | N | Y | Original content of the reason for closing without re-execution, for CLOSE. Required on submission | - |
| 1086 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for rewriting after rejection or cancellation. Required if revision_number ≥ 2 | `Incorporated QA rejection comments` |
| 1087 | Approval Status | `approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED / CANCELLED. REJECTED and CANCELLED are terminal states of this revision | `DRAFT` |
| 1088 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | DEVIATION_COMPLETION or DEVIATION_CLOSE approval workflow targeting this revision | `00000000-0000-0000-0000-000000000001` |
| 1089 | Final Approval Signature ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Final approval electronic signature of this revision | `00000000-0000-0000-0000-000000000001` |
| 1090 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Original-content revision creation time | `2026-09-03T10:00:00Z` |
| 1091 | Author | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Closure report author | `00000000-0000-0000-0000-000000000001` |
| 1092 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time of draft editing or management status change | `2026-09-03T10:00:00Z` |
| 1093 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Processing user | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_deviation_completion_revision` | UNIQUE | `deviation_id, revision_number` | UNIQUE (deviation_id, revision_number) |
| `uq_deviation_completion_current` | UNIQUE | `deviation_id, is_current_revision` | UNIQUE (deviation_id) WHERE is_current_revision=TRUE |
| `uq_deviation_completion_approved` | UNIQUE | `deviation_id, approval_status` | UNIQUE (deviation_id) WHERE approval_status='APPROVED' |
| `ck_deviation_completion_sequence` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `ck_deviation_completion_kind` | CHECK | `closure_kind` | CHECK (closure_kind IN ('COMPLETION','CLOSE')) |
| `ck_deviation_completion_status` | CHECK | `approval_status` | CHECK (approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_deviation_completion_content` | CHECK | `closure_kind, corrective_action, completion_report, rerun_execution_id, closure_reason` | CHECK ((closure_kind='COMPLETION' AND closure_reason IS NULL) OR (closure_kind='CLOSE' AND corrective_action IS NULL AND completion_report IS NULL AND rerun_execution_id IS NULL)) |
| `ck_deviation_completion_approval` | CHECK | `approval_status, workflow_instance_id, signature_id` | CHECK (approval_status <> 'APPROVED' OR (workflow_instance_id IS NOT NULL AND signature_id IS NOT NULL)) |
| `ck_deviation_completion_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |
| `rule_deviation_completion_submit` | Business validation | `closure_kind, approval_status` | On submission, action_round_id must equal deviation.current_action_round_id. For COMPLETION, deviation_status must be COMPLETION_PENDING, corrective_action, completion_report and rerun_execution_id are required, and rerun_execution_id must be the valid result revision of the latest re-execution. For CLOSE, deviation_status must be RERUN_ALLOWED, there must be no re-execution linked to the current action round, and closure_reason is required. |
| `rule_deviation_completion_lock` | Business validation | `approval_status` | The original content is locked after submission (REVIEW). Editing after rejection or cancellation creates a new row with the next revision_number of the same deviation, and the previous row, Workflow and signature are preserved. At most 1 revision per deviation may be in progress (DRAFT, REVIEW, APPROVAL). |
| `rule_deviation_completion_sync` | Business validation | `approval_status` | In the final approval transaction, the deviation's corrective_action and completion_report or closure_reason, completion_signature_id or closure_signature_id, and approved_by and approved_at are synchronized from this revision, and deviation_status is transitioned to CLOSED (rule_deviation_9). |
| `rule_deviation_completion_reopen` | Business validation | `approval_status` | When deviation_status returns to ACTION_PENDING (rule_deviation_9), any not-yet-submitted DRAFT revision is terminated as CANCELLED. |

#### Business Rules

The completion report and the closure without re-execution manage original-content revisions in the same way as the deviation action (deviation_action_round). On the screen it appears as "editing" a rejected report, but a new revision row is created in the DB, and the 1st and 2nd original contents can be compared directly in this table.

The approval workflow and electronic signature target this table's completion_revision_id, not deviation, and target_version is COMPLETE-{action round}-{revision_number} or CLOSE-{action round}-{revision_number}. signed_payload fixes the basis action round, the initial failure PK, the basis re-execution result revision PK, and the corrective and preventive action and completion report, or the closure reason.

[↑ Back to top](#top)

---

## Document

<a id="table-deliverable_document"></a>
### 66. Deliverable Document (`deliverable_document`)

| Item | Definition |
|---|---|
| Description | Document number and persistent identification of deliverables per activity |
| Primary Key | `document_id` |
| Main References (FK) | `project_id, project_activity_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1094 | Document ID | `document_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1095 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 1096 | Activity ID | `project_activity_id` | `uuid` | N | Y | `project_activity.project_activity_id` | Y | - | N | Y | N | Y | References project_activity.project_activity_id | `00000000-0000-0000-0000-000000000001` |
| 1097 | Document Category | `document_type` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | STAGE_DELIVERABLE / PROTOCOL / RECORD / SUMMARY | `RECORD` |
| 1098 | Document Number | `document_number` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Document number on the deliverable screen. NULL allowed before approval | `OQ-R-001` |
| 1099 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1100 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1101 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1102 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_deliverable_document_1` | Business validation | - | The document's project_activity_id must belong to the same project_id. RTM is not included as an independent activity / required deliverable. |
| `rule_deliverable_document_2` | Business validation | - | If a document number exists, it is kept unique within the project. Test protocols and result documents are distinguished by document category. |
| `ck_deliverable_document_type` | CHECK | `document_type` | CHECK (document_type IN ('STAGE_DELIVERABLE','PROTOCOL','RECORD','SUMMARY')) |

[↑ Back to top](#top)

---

<a id="table-deliverable_revision"></a>
### 67. Deliverable Revision (`deliverable_revision`)

| Item | Definition |
|---|---|
| Description | Target of section/body editing, per-version viewing, author signature and document approval |
| Primary Key | `document_revision_id` |
| Main References (FK) | `document_id, workflow_instance_id, approved_by, file_id, created_by, updated_by, system_baseline_id` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1103 | Document Revision ID | `document_revision_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1104 | Document ID | `document_id` | `uuid` | N | Y | `deliverable_document.document_id` | Y | - | N | Y | N | Y | References deliverable_document.document_id | `00000000-0000-0000-0000-000000000001` |
| 1105 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Document Title | `Operational Qualification Result Report` |
| 1106 | Document Display Version | `version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Document Display Version | `v1.0` |
| 1107 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision Number | `1` |
| 1108 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason | `Test results incorporated` |
| 1109 | Is Latest Working Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Is Latest Working Version | `TRUE` |
| 1110 | Document Approval Status | `approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED / CANCELLED. Edits after REJECTED or CANCELLED are handled as a new document revision | `DRAFT` |
| 1111 | Document Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |
| 1112 | Basis Business Approval Version | `source_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Version of the approved data on the screen. Distinct from the document's own version | `v1.0` |
| 1113 | Basis Business Revision List | `source_refs` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Array of [{table_name,record_id,version}]. Fixes the exact revisions of multiple business items and the approved execution attempts | `[{"table_name":"oq_execution","record_id":"00000000-0000-0000-0000-000000000001","version":"1-1"}]` |
| 1114 | Approved Data Table | `approved_table_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Columns/rows of the approved data on the screen. {columns:string[],rows:string[][]} | `{"columns":["Item","Result"],"rows":[["OQ-001","Pass"]]}` |
| 1115 | Part 11 Table | `part11_table_snapshot` | `jsonb` | N | N | - | N | - | N | N | N | Y | Part 11 question/answer/judgment table of the QIA deliverable. NULL for other activities | - |
| 1116 | Revision History Table | `revision_table_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Revision history columns/rows as of authoring | `{"columns":["Version","Reason"],"rows":[["v1.0","Initial creation"]]}` |
| 1117 | Re-approval Trigger Activity | `reapproval_source_stage` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Activity code when re-approval is required due to a change in an upstream activity. RTM excluded | `URS` |
| 1118 | Re-approval Reason | `reapproval_reason` | `text` | N | N | - | N | - | N | N | N | Y | Re-approval Reason | `Content re-reviewed due to URS revision` |
| 1119 | Approval Invalidated At | `invalidated_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Approval Invalidated At | `2026-09-01T00:00:00Z` |
| 1120 | Final Approver ID | `approved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1121 | Final Approved At | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Final Approved At | `2026-09-01T00:00:00Z` |
| 1122 | PDF File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | Linked when the generated PDF is stored | `00000000-0000-0000-0000-000000000001` |
| 1123 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1124 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1125 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1126 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1127 | Document Validation Baseline | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | Baseline of the same project fixed at document submission. Required on submission and approval. For previous executions, each execution's baseline is displayed alongside. | - |
| 1128 | Original Evaluation Questions & Judgment Rules | `evaluation_definition_snapshot` | `jsonb` | N | N | - | N | - | N | N | N | Y | Array fixing, for each basis evaluation of a QIA/FRA document, the question/rule versions and their actual definitions, inputs and results. NULL for documents unrelated to evaluations | - |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_deliverable_revision_1` | UNIQUE | `document_id, revision_number` | UNIQUE(document_id,revision_number) |
| `uq_deliverable_revision_1_2` | UNIQUE | `document_id, version` | UNIQUE(document_id,version) |
| `rule_deliverable_revision_1` | Business validation | `document_id, version, revision_number, is_current_version` | At most 1 is_current_version=TRUE per document |
| `rule_deliverable_revision_2` | Business validation | - | Document approval and approval of the basis business items are separate. source_refs validates the exact revision, project and approval status of multiple source items and is preserved together with the tables as of that time. |
| `rule_deliverable_revision_3` | Business validation | - | If APPROVED, the document approval workflow history, final approver and time are required. Approved body / tables / basis links are not changed; a new revision is written instead. |
| `rule_deliverable_revision_4` | Business validation | - | Even if the source changes later, past approved versions are not deleted. Current validity is determined separately by invalidated_at and the re-approval reason. |
| `uq_document_current_revision` | UNIQUE | `document_id` | UNIQUE (document_id) WHERE is_current_version=TRUE |
| `ck_deliverable_revision_status` | CHECK | `approval_status` | CHECK (approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_deliverable_revision_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### Business Rules

PDF generation status does not replace document approval status. The VP source sections are vp_section, and since the VP deliverable reflects the approved source, the same content is not edited independently in two places.

For IQ/OQ/PQ, source_refs records the corresponding evaluation revision and the exact PKs and versions of all test items and executions included in the document. A protocol document fixes the complete composition of the evaluation's item_revision_refs, and a result document links the exact attempts and result revisions of the executions belonging to that composition. When only the document is revised, the same evaluation and test references may be reused as is. system_baseline_id and the referenced targets must belong to the same project. If results executed under a different baseline are included, the baseline of those executions and the rationale for the reuse decision are stated in the body.

On document submission and approval, system_baseline_id is required, and for QIA/FRA documents evaluation_definition_snapshot is also required. That copy must allow the results to be recalculated from the inputs and the actual per-version definitions; it must not store only a simple result table. After submission, the body, composition, and question and rule copies are locked, and edits after rejection or cancellation are handled as a new document revision. The generated PDF is linked after file generation but does not change the already-signed business original content, and the PDF file's identity with the original is verified separately using the file_asset hash.

[↑ Back to top](#top)

---

<a id="table-deliverable_section"></a>
### 68. Deliverable Section/Body (`deliverable_section`)

| Item | Definition |
|---|---|
| Description | Sections that are added, edited, deleted and reordered on the document editing screen |
| Primary Key | `section_id` |
| Main References (FK) | `document_revision_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1129 | Section ID | `section_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1130 | Document Revision ID | `document_revision_id` | `uuid` | N | Y | `deliverable_revision.document_revision_id` | Y | - | N | Y | N | Y | References deliverable_revision.document_revision_id | `00000000-0000-0000-0000-000000000001` |
| 1131 | Section Key | `section_key` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Section Key | `purpose` |
| 1132 | Section Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Section Title | `Purpose` |
| 1133 | Body | `content` | `text` | N | N | - | Y | - | N | N | N | Y | Body | `Describes the purpose of this test.` |
| 1134 | Display Order | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display Order | `1` |
| 1135 | Authoring Source | `source_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | MANUAL / AI / APPROVED / HISTORY / PART11 | `MANUAL` |
| 1136 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1137 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1138 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1139 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_deliverable_section_1` | UNIQUE | `document_revision_id, section_key` | UNIQUE(document_revision_id,section_key) |
| `uq_deliverable_section_1_2` | UNIQUE | `document_revision_id, sort_order` | UNIQUE(document_revision_id,sort_order) |
| `ck_deliverable_section_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_deliverable_section_2` | Business validation | - | MANUAL/AI body content can be edited only while the parent document revision is DRAFT, and is locked after submission (REVIEW). Edits after rejection or cancellation are made in a new document revision. APPROVED/HISTORY/PART11 sections are displayed from the corresponding snapshot. Sections of an approved document are not directly modified or deleted. |
| `ck_deliverable_section_source_type` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','AI','APPROVED','HISTORY','PART11')) |

[↑ Back to top](#top)

---

## Report

<a id="table-report_generation"></a>
### 69. Report Generation Job (`report_generation`)

| Item | Definition |
|---|---|
| Description | Deliverable/audit PDF generation requested on the screen, and download results |
| Primary Key | `report_generation_id` |
| Main References (FK) | `project_id, requested_by, result_file_id, created_by, updated_by, document_revision_id` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Report generation results and execution history are retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1140 | Report Generation ID | `report_generation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the report generation job | `00000000-0000-0000-0000-000000000001` |
| 1141 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Linked for a project-level report; NULL allowed for system-wide or organization-level reports | `00000000-0000-0000-0000-000000000001` |
| 1142 | Report Type | `report_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | DELIVERABLE / AUDIT_TRAIL. Deliverable or audit export on the screen | `AUDIT_TRAIL` |
| 1143 | Report Name | `report_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Name of the generated report displayed to users | `August 2026 Audit Trail Report` |
| 1144 | Query Start Date/Time | `period_from` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Start date/time for querying the report's source data. NULL allowed for reports without a query period | `2026-08-01T00:00:00Z` |
| 1145 | Query End Date/Time | `period_to` | `timestamptz` | N | N | - | N | - | N | N | N | Y | End date/time for querying the report's source data. Cannot be earlier than period_from | `2026-08-31T23:59:59Z` |
| 1146 | Query Conditions | `report_parameters` | `jsonb` | N | N | - | N | - | N | N | Y | Y | Export filters selected on the screen, such as period, user, role and menu | `{"action_types":["CREATE","UPDATE"]}` |
| 1147 | Output Format | `output_format` | `varchar(20)` | N | N | - | Y | `'PDF'` | N | Y | N | Y | Output format PDF | `PDF` |
| 1148 | Generation Status | `generation_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | PENDING / PROCESSING / COMPLETED / FAILED / CANCELLED | `PENDING` |
| 1149 | Requester ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who requested the PDF/export | `00000000-0000-0000-0000-000000000001` |
| 1150 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when the user requested the export | `2026-09-02T15:00:00Z` |
| 1151 | Execution Start Time | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when generation started | `2026-09-02T15:00:05Z` |
| 1152 | Execution Completion Time | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when generation success or failure was finalized | `2026-09-02T15:01:30Z` |
| 1153 | Result File ID | `result_file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | Linked when the result is stored as a file asset. May be NULL if only a download is performed | `00000000-0000-0000-0000-000000000001` |
| 1154 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | System error code classifying the cause of failure | `REPORT_FILE_CREATE_FAILED` |
| 1155 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Details of the report generation failure. Sensitive information such as passwords and tokens is not stored | `Error occurred while saving the result file` |
| 1156 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Report generation job record creation time (UTC) | `2026-09-02T15:00:00Z` |
| 1157 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who requested the export. For automatic/batch generation, the user_id of the SYSTEM account is stored | `00000000-0000-0000-0000-000000000001` |
| 1158 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Report generation job last update time (UTC) | `2026-09-02T15:01:30Z` |
| 1159 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who updated the export processing result. For automatic status updates, the user_id of the SYSTEM account is stored | `00000000-0000-0000-0000-000000000001` |
| 1160 | Deliverable Revision ID | `document_revision_id` | `uuid` | N | Y | `deliverable_revision.document_revision_id` | N | - | N | Y | N | Y | References deliverable_revision.document_revision_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_report_generation_1` | Business validation | `document_revision_id` | For DELIVERABLE, document_revision_id is required and must match the project. For AUDIT_TRAIL, the query scope is fixed by period/filters. |
| `rule_report_generation_2` | Business validation | `period_from, period_to` | If both period_from and period_to are present, period_from<=period_to. generation_status has no automatic-retry waiting state. |
| `ck_report_generation_type` | CHECK | `report_type` | CHECK (report_type IN ('DELIVERABLE','AUDIT_TRAIL')) |
| `ck_report_generation_status` | CHECK | `generation_status` | CHECK (generation_status IN ('PENDING','PROCESSING','COMPLETED','FAILED','CANCELLED')) |

#### Business Rules

Body, sections, versions and document approval status are stored in deliverable_revision and deliverable_section. This table manages only export results.

[↑ Back to top](#top)

---

## AI

<a id="table-ai_generation_job"></a>
### 70. AI Generation Job (`ai_generation_job`)

| Item | Definition |
|---|---|
| Description | AI draft generation requests on the screen, input conditions and progress results |
| Primary Key | `ai_job_id` |
| Main References (FK) | `project_id, requested_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1161 | AI Job ID | `ai_job_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI generation job identifier | `00000000-0000-0000-0000-000000000001` |
| 1162 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Project link | `00000000-0000-0000-0000-000000000001` |
| 1163 | Job Type | `job_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | ITEM_GENERATION, DOCUMENT_GENERATION | `ITEM_GENERATION` |
| 1164 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT / FRA_ITEM / IQ_ITEM / OQ_ITEM / PQ_ITEM / DELIVERABLE_REVISION | `IQ_ITEM` |
| 1165 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | For an existing target, the PK of the corresponding revision. NULL allowed for previews that generate new items | `00000000-0000-0000-0000-000000000001` |
| 1166 | AI Model Name | `model_name` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | Identifier of the model used for generation. Stored as a fixed version name that includes a date; an alias that points to the latest version is not stored. Recorded when confirmed; NULL is allowed when unconfirmed. | - |
| 1167 | Prompt Version | `prompt_version` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | Version of the prompt template used by the AI service (an item of the WAS↔AI contract). Required when COMPLETED. A different prompt is not deployed under the same version name | `IQ_ITEM_v2` |
| 1168 | Masked Raw Response | `masked_response` | `text` | N | N | - | N | - | N | N | Y | Y | The raw response returned by the AI service, kept per job in its masked state as is. The mapping table for unmasking is not stored, so it cannot be recreated later. Required when COMPLETED. Classified as sensitive because information not covered by the detection rules may remain. Which one is treated as the original in audits (this value or ai_result_item.content) is pending decision | - |
| 1169 | Input Parameters | `input_parameters` | `jsonb` | N | N | - | Y | - | N | N | Y | Y | Screen inputs: purpose, scope, basis URS, number of items to generate, or the sections of the document to generate | `{"purpose":"Access permission verification","scope":"Login","count":3}` |
| 1170 | Generation Status | `generation_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | PENDING / PROCESSING / COMPLETED / FAILED / CANCELLED | `PENDING` |
| 1171 | Requester ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who requested the generation. For automatic/batch requests, the user_id of the SYSTEM account is stored | `00000000-0000-0000-0000-000000000001` |
| 1172 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time the request was received | `2026-09-02T00:00:00Z` |
| 1173 | Execution Start Time | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI processing start time | `2026-09-02T00:00:00Z` |
| 1174 | Execution Completion Time | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI processing completion time | `2026-09-02T00:00:00Z` |
| 1175 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Failure cause code | `LLM_TIMEOUT` |
| 1176 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Detailed failure message | `Model response timeout` |
| 1177 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-02T00:00:00Z` |
| 1178 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Creating user. For automated/batch processing, the user_id of the SYSTEM account is stored | `00000000-0000-0000-0000-000000000001` |
| 1179 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-02T00:00:00Z` |
| 1180 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Updating user. For automated/batch processing, the user_id of the SYSTEM account is stored | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `rule_ai_generation_job_1` | Business validation | `target_entity_id` | New items of ITEM_GENERATION allow a NULL target_entity_id before being applied. DOCUMENT_GENERATION is linked to the corresponding document revision. |
| `rule_ai_generation_job_2` | Business validation | - | Generation completion does not mean approval of business items or document approval. After selection and application, the regular edit/approval procedure is followed. |
| `rule_ai_generation_job_3` | Business validation | `project_id, requested_by` | At job creation, organization.ai_enabled of the reference organization for the determination must be TRUE. The reference organization is the organization to which the project's system belongs if project_id is present, and otherwise the requester's organization. Because the SYSTEM account belongs to the SYSTEM organization (ai_enabled=FALSE), it cannot request AI jobs without a project_id. If FALSE, no PENDING row is created and the generation is rejected. Audit-response bot queries follow the same setting and are rejected on the server without separate storage. When ai_enabled is switched to FALSE, jobs that are already PENDING/PROCESSING are allowed to run to completion. |
| `rule_ai_generation_job_4` | Business validation | `generation_status, prompt_version, masked_response` | Before data is sent to the AI service, the sensitive-information detection result for the outbound data (ai_sensitive_detection) must be stored; if it cannot be stored, the data is not sent. WAS→AI sending happens once per job; per-section/per-item generation and retries are handled inside the AI service, and only the final result or failure is recorded. When COMPLETED, prompt_version and masked_response are stored. |
| `ck_ai_generation_job_type` | CHECK | `job_type` | CHECK (job_type IN ('ITEM_GENERATION','DOCUMENT_GENERATION')) |
| `ck_ai_generation_job_target_type` | CHECK | `target_entity_type` | CHECK (target_entity_type IN ('REQUIREMENT','FRA_ITEM','IQ_ITEM','OQ_ITEM','PQ_ITEM','DELIVERABLE_REVISION')) |
| `ck_ai_generation_job_status` | CHECK | `generation_status` | CHECK (generation_status IN ('PENDING','PROCESSING','COMPLETED','FAILED','CANCELLED')) |

[↑ Back to top](#top)

---

<a id="table-ai_generation_result"></a>
### 71. AI Generation Result (`ai_generation_result`)

| Item | Definition |
|---|---|
| Description | Manages AI generation result sets and their adoption status |
| Primary Key | `ai_result_id` |
| Main References (FK) | `ai_job_id, selected_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1181 | AI Result ID | `ai_result_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI result identifier | `00000000-0000-0000-0000-000000000001` |
| 1182 | AI Job ID | `ai_job_id` | `uuid` | N | Y | `ai_generation_job.ai_job_id` | Y | - | N | Y | N | Y | Parent AI job | `00000000-0000-0000-0000-000000000001` |
| 1183 | Result Title | `result_title` | `varchar(300)` | N | N | - | N | - | N | N | N | Y | Result Title | `OQ test draft` |
| 1184 | Selected | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | Whether adopted by the user | `TRUE` |
| 1185 | Applied | `is_applied` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | Whether actually applied to the deliverable | `TRUE` |
| 1186 | Selected By ID | `selected_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who adopted it | `00000000-0000-0000-0000-000000000001` |
| 1187 | Selected At | `selected_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Adopted At | `2026-09-02T00:00:00Z` |
| 1188 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-02T00:00:00Z` |
| 1189 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Creating user. For automated/batch processing, the user_id of the SYSTEM account is stored | `00000000-0000-0000-0000-000000000001` |
| 1190 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-02T00:00:00Z` |
| 1191 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Updating user. For automated/batch processing, the user_id of the SYSTEM account is stored | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `ck_ai_generation_result_selected` | CHECK | `is_selected, selected_by, selected_at` | CHECK (is_selected = FALSE OR (selected_by IS NOT NULL AND selected_at IS NOT NULL)) |

#### Business Rules

This is a bundle of AI results. Per-item selection and application are based on ai_result_item, and is_selected/is_applied are a summary of that bundle.

[↑ Back to top](#top)

---

<a id="table-ai_result_item"></a>
### 72. AI Generation Result Item (`ai_result_item`)

| Item | Definition |
|---|---|
| Description | Manages the details of AI-generated URS/FRA/IQ/OQ/PQ items and document sections |
| Primary Key | `ai_result_item_id` |
| Main References (FK) | `ai_result_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1192 | AI Result Item ID | `ai_result_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Result item identifier | `00000000-0000-0000-0000-000000000001` |
| 1193 | AI Result ID | `ai_result_id` | `uuid` | N | Y | `ai_generation_result.ai_result_id` | Y | - | N | Y | N | Y | References the parent result | `00000000-0000-0000-0000-000000000001` |
| 1194 | Item Order | `item_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Result display order | `1` |
| 1195 | Item Type | `item_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT, FRA_SCENARIO, IQ_TEST, OQ_TEST, PQ_TEST, DOCUMENT_SECTION | `REQUIREMENT` |
| 1196 | Title | `title` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | Generated item title | `Electronic signature record` |
| 1197 | Body Content | `content` | `text` | N | N | - | Y | - | N | N | N | Y | Draft content generated by AI per test/requirement/risk assessment/section. Stored as the unmasked value; it is the proposal the user first saw and the baseline for the edit history (the masked raw response is ai_generation_job.masked_response) | `Content` |
| 1198 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Type of the business table it was applied to: REQUIREMENT / FRA_ITEM / IQ_ITEM / OQ_ITEM / PQ_ITEM / DELIVERABLE_SECTION | `IQ_ITEM` |
| 1199 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | PK of the revision actually created/modified by applying it. NULL before application | `00000000-0000-0000-0000-000000000001` |
| 1200 | Adopted | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | Whether adopted by the user | `TRUE` |
| 1201 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-02T00:00:00Z` |
| 1202 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Creating user. For automated/batch processing, the user_id of the SYSTEM account is stored | `00000000-0000-0000-0000-000000000001` |
| 1203 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-02T00:00:00Z` |
| 1204 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Updating user. For automated/batch processing, the user_id of the SYSTEM account is stored | `00000000-0000-0000-0000-000000000001` |
| 1205 | Applied | `is_applied` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Applied | `FALSE` |
| 1206 | Applied At | `applied_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Applied At | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_ai_result_item_1` | UNIQUE | `ai_result_id, item_order` | UNIQUE(ai_result_id,item_order) |
| `rule_ai_result_item_1` | Business validation | `ai_result_id, item_order, is_applied` | If is_applied=TRUE, the actual target entity type and PK, and applied_at, are required |
| `rule_ai_result_item_2` | Business validation | - | Per-section document results are linked via DELIVERABLE_SECTION. They are not applied to items/documents from submission (REVIEW) onward; they are applied only to a DRAFT revision or a new revision. |
| `ck_ai_result_item_type` | CHECK | `item_type` | CHECK (item_type IN ('REQUIREMENT','FRA_SCENARIO','IQ_TEST','OQ_TEST','PQ_TEST','DOCUMENT_SECTION')) |
| `ck_ai_result_item_target_type` | CHECK | `target_entity_type` | CHECK (target_entity_type IS NULL OR target_entity_type IN ('REQUIREMENT','FRA_ITEM','IQ_ITEM','OQ_ITEM','PQ_ITEM','DELIVERABLE_SECTION')) |

[↑ Back to top](#top)

---

<a id="table-ai_sensitive_detection"></a>
### 73. AI Outbound Data Sensitive-Information Detection Result (`ai_sensitive_detection`)

| Item | Definition |
|---|---|
| Description | Number of detections per type of the sensitive information masked in an AI generation job before it is sent to the AI service |
| Primary Key | `detection_id` |
| Main References (FK) | `ai_job_id, created_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1207 | Detection ID | `detection_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Detection result row identifier | `00000000-0000-0000-0000-000000000001` |
| 1208 | AI Job ID | `ai_job_id` | `uuid` | N | Y | `ai_generation_job.ai_job_id` | Y | - | N | Y | N | Y | AI generation job that sent the checked data | `00000000-0000-0000-0000-000000000001` |
| 1209 | Detection Type | `detection_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Values in use (a reference list, not restricted by a CHECK): EMAIL (email), PHONE (phone number), RRN (resident registration number), ORGANIZATION (organization name such as customer, plant, or lab), PERSON (person name). Added to the list when detection items increase | `EMAIL` |
| 1210 | Detected Count | `detected_count` | `integer` | N | N | - | Y | - | N | N | N | Y | Number of detections of the type. A checked type is recorded even with 0 detections | `2` |
| 1211 | Detection Rule Version | `detector_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Version of the rules used for detection and masking. A different rule set is not deployed under the same version name | `MASK_RULE_v1` |
| 1212 | Detected At | `detected_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time the detection was performed | `2026-10-16T01:00:00Z` |
| 1213 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-10-16T01:00:00Z` |
| 1214 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Creating user. For automated/batch processing, the user_id of the SYSTEM account is stored | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_ai_sensitive_detection_type` | UNIQUE | `ai_job_id, detection_type` | UNIQUE (ai_job_id, detection_type) |
| `ck_ai_sensitive_detection_count` | CHECK | `detected_count` | CHECK (detected_count >= 0) |
| `rule_ai_sensitive_detection_1` | Business validation | - | The actual detected values (email addresses, names, etc.) and the mapping table for unmasking are not stored; only the number per type is stored. |
| `rule_ai_sensitive_detection_2` | Business validation | `ai_job_id, detection_type, detected_count` | A checked type is recorded as 1 row even with 0 detections. Storing is finished before the data is sent to the AI service; if it cannot be stored, the data is not sent (rule_ai_generation_job_4). |
| `rule_ai_sensitive_detection_3` | Business validation | - | Rows are only added and are not updated or deleted. |

#### Business Rules

Data that an AI generation job sends to the AI service (system and project information, deliverables from previous stages) has its sensitive information masked before it is sent, and the result is recorded as 1 row per job × type. Because WAS→AI sending happens once per job, (ai_job_id, detection_type) is unique. Per-section/per-item generation and retries are handled inside the AI service and do not create separate detection rows. If the sending structure changes so that a job sends more than once, a sending sequence column is added. Uploaded files are not used as AI input in the current scope.

[↑ Back to top](#top)

---

## Regulation

<a id="table-regulatory_source"></a>
### 74. Regulatory Basis Document (`regulatory_source`)

| Item | Definition |
|---|---|
| Description | Document editions of regulations, guidelines and SOPs referenced by URS, FRA and the library |
| Primary Key | `regulatory_source_id` |
| Main References (FK) | `file_id, verified_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1215 | Regulatory Document ID | `regulatory_source_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1216 | Document Code | `source_code` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Document Code | `REF-001` |
| 1217 | Document Type | `source_type` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | REGULATION / GUIDELINE / INTERNAL_SOP | `GUIDELINE` |
| 1218 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Document Title | `Reference document for review` |
| 1219 | Edition/Revision | `edition` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Edition/Revision | `Rev.1` |
| 1220 | Issuing Body/Department | `issuing_body` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Issuing Body/Department | `Quality Assurance Team` |
| 1221 | Source URL | `source_url` | `text` | N | N | - | N | - | N | N | N | Y | Source URL | `https://example.com/reference` |
| 1222 | Original Attachment ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | References file_asset.file_id | `00000000-0000-0000-0000-000000000001` |
| 1223 | Effective From | `effective_from` | `date` | N | N | - | N | - | N | N | N | Y | Effective From | `2026-09-01` |
| 1224 | Effective To | `effective_to` | `date` | N | N | - | N | - | N | N | N | Y | Effective To | `2026-09-01` |
| 1225 | Review Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT / VERIFIED / RETIRED | `DRAFT` |
| 1226 | Verified By ID | `verified_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1227 | Verified At | `verified_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Verified At | `2026-09-01T00:00:00Z` |
| 1228 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1229 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1230 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1231 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_regulatory_source_1` | UNIQUE | `source_code, edition` | UNIQUE(source_code,edition) |
| `rule_regulatory_source_1` | Business validation | `source_code, edition` | Editions in other languages are identified by a separate source_code |
| `rule_regulatory_source_2` | Business validation | - | If VERIFIED, the verifier and verification time, plus either a URL or an original attachment, are required. The effective end date cannot be earlier than the effective start date. |
| `rule_regulatory_source_3` | Business validation | - | A new edition is registered as a new row. A document edition referenced by approved work is not deleted or overwritten with new content. |
| `ck_regulatory_source_type` | CHECK | `source_type` | CHECK (source_type IN ('REGULATION','GUIDELINE','INTERNAL_SOP')) |
| `ck_regulatory_source_status` | CHECK | `status` | CHECK (status IN ('DRAFT','VERIFIED','RETIRED')) |

#### Business Rules

Regulatory bases are managed as reusable reference data. A manually entered basis is linked after its document edition and clause have been checked, and the verified status is granted only when the verifier and verification time have been recorded.

[↑ Back to top](#top)

---

<a id="table-regulatory_clause"></a>
### 75. Regulatory Clause (`regulatory_clause`)

| Item | Definition |
|---|---|
| Description | Clauses belonging to a regulatory source document edition, and their display text |
| Primary Key | `regulatory_clause_id` |
| Main References (FK) | `regulatory_source_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1232 | Clause ID | `regulatory_clause_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1233 | Regulatory Document ID | `regulatory_source_id` | `uuid` | N | Y | `regulatory_source.regulatory_source_id` | Y | - | N | Y | N | Y | References regulatory_source.regulatory_source_id | `00000000-0000-0000-0000-000000000001` |
| 1234 | Clause Code | `clause_code` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Clause Code | `4.1` |
| 1235 | Clause Title | `title` | `varchar(255)` | N | N | - | N | - | N | N | N | Y | Clause Title | `Access management` |
| 1236 | Clause Summary | `summary` | `text` | N | N | - | Y | - | N | N | N | Y | Clause Summary | `Summary of the requirements applicable to the relevant work` |
| 1237 | Original Content Location | `source_locator` | `text` | N | N | - | N | - | N | N | N | Y | Original Content Location | `Chapter 4, Section 1` |
| 1238 | Display Order | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display Order | `1` |
| 1239 | In Use | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | In Use | `TRUE` |
| 1240 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1241 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1242 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1243 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_regulatory_clause_1` | UNIQUE | `regulatory_source_id, clause_code` | UNIQUE(regulatory_source_id,clause_code) |
| `ck_regulatory_clause_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_regulatory_clause_2` | Business validation | - | New selections use active clauses of a VERIFIED document edition. Even if a clause already used in an approval is deactivated, past references to it are retained. |

[↑ Back to top](#top)

---

<a id="table-requirement_regulation"></a>
### 76. Requirement Regulatory Basis (`requirement_regulation`)

| Item | Definition |
|---|---|
| Description | Multiple links between a business item and its applicable regulatory clauses |
| Primary Key | `requirement_regulation_id` |
| Main References (FK) | `requirement_id, regulatory_clause_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1244 | Basis Link ID | `requirement_regulation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1245 | Business Item ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | Y | N | Y | References requirement.requirement_id | `00000000-0000-0000-0000-000000000001` |
| 1246 | Regulatory Clause ID | `regulatory_clause_id` | `uuid` | N | Y | `regulatory_clause.regulatory_clause_id` | Y | - | N | Y | N | Y | References regulatory_clause.regulatory_clause_id | `00000000-0000-0000-0000-000000000001` |
| 1247 | Application Rationale | `application_note` | `text` | N | N | - | N | - | N | N | N | Y | Application Rationale | `Applied to access permission requirements` |
| 1248 | Citation Text | `citation_snapshot` | `text` | N | N | - | Y | - | N | N | N | Y | Preserves the document title, edition and clause display as of the time of application | `Reference document for review Rev.1 / 4.1` |
| 1249 | Display Order | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display Order | `1` |
| 1250 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1251 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1252 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1253 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_requirement_regulation_1` | UNIQUE | `requirement_id, regulatory_clause_id` | UNIQUE(requirement_id,regulatory_clause_id) |
| `ck_requirement_regulation_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_requirement_regulation_2` | Business validation | - | When the business item is approved, the clauses and citation text as of that time are preserved. When an item is applied from the library to a URS, its basis links are also copied, and later template changes are not automatically propagated to existing URSs. |

[↑ Back to top](#top)

---

<a id="table-library_item_regulation"></a>
### 77. Library Regulatory Basis (`library_item_regulation`)

| Item | Definition |
|---|---|
| Description | Multiple links between a library item and its applicable regulatory clauses |
| Primary Key | `library_item_regulation_id` |
| Main References (FK) | `library_id, regulatory_clause_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Target | Y |
| Retention / Deletion | Retained for the auditable period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: personal / sensitive information

| No | Logical Name | Physical Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description / Business Rule | Example |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1254 | Basis Link ID | `library_item_regulation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1255 | Library Item ID | `library_id` | `uuid` | N | Y | `library_item.library_id` | Y | - | N | Y | N | Y | References library_item.library_id | `00000000-0000-0000-0000-000000000001` |
| 1256 | Regulatory Clause ID | `regulatory_clause_id` | `uuid` | N | Y | `regulatory_clause.regulatory_clause_id` | Y | - | N | Y | N | Y | References regulatory_clause.regulatory_clause_id | `00000000-0000-0000-0000-000000000001` |
| 1257 | Application Rationale | `application_note` | `text` | N | N | - | N | - | N | N | N | Y | Application Rationale | `Applied to access permission requirements` |
| 1258 | Citation Text | `citation_snapshot` | `text` | N | N | - | Y | - | N | N | N | Y | Preserves the document title, edition and clause display as of the time of application | `Reference document for review Rev.1 / 4.1` |
| 1259 | Display Order | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display Order | `1` |
| 1260 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1261 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1262 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1263 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint | Type | Columns | Condition / Rule |
|---|---|---|---|
| `uq_library_item_regulation_1` | UNIQUE | `library_id, regulatory_clause_id` | UNIQUE(library_id,regulatory_clause_id) |
| `ck_library_item_regulation_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_library_item_regulation_2` | Business validation | - | When a library item is applied to a business item such as a URS, its clause links and citation text are copied; later changes to the library item's basis are not automatically propagated to business items it has already been applied to. |

[↑ Back to top](#top)

---
