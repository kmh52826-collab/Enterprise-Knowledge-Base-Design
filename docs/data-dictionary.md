<a id="top"></a>

# Data Table Specifications

> Validation Management Platform Data Model Documentation  
> **Example Value Convention**: Personal and sensitive information is represented using de-identified sample values.

## Table of Contents

- [1. Document Overview](#1-document-overview)
- [2. Notation Conventions](#2-notation-conventions)
- [3. Table List](#3-table-list)
- [4. Domain Navigation](#4-domain-navigation)
- [5. Detailed Table and Column Definitions](#5-detailed-table-and-column-definitions)

## 1. Document Overview

- Total tables: **73**
- Total columns: **1,177**
- Domains: **26**
- Naming conventions: Tables and columns use `snake_case`; PKs use entity-specific identifier columns.
- Date/time conventions: Store UTC in the database; apply the user's or site's time zone for display.
- GxP principles: Do not modify approved records directly. Manage them through revisions and status history, and record significant changes in the Audit Trail.

## 2. Notation Conventions

| Notation | Meaning |
| --- | --- |
| `Y` | Applicable or required |
| `N` | Not applicable or optional |
| `-` | Not applicable or unspecified |
| PK | Primary Key |
| FK | Foreign Key |
| Not Null | The inverse of whether NULL is allowed |
| Audit | Whether the item is recorded in the Audit Trail |
| UQ | Whether an individual column is unique. Composite and conditional uniqueness are defined separately in the constraints for the relevant table. |
| IDX | Whether an index is specified. Indexes created by a PK or an individual UNIQUE constraint are also marked Y. For composite and conditional indexes, also consult the constraints. |
| Sensitive | Personal/sensitive information classification |
| Default | SQL expression. Strings use single quotes, Booleans use TRUE/FALSE, and functions retain their function expressions. `-` indicates that no default is specified. |

`date` examples use `YYYY-MM-DD`; `timestamptz` examples use `YYYY-MM-DDTHH:mm:ssZ` in UTC.

Do not mark each constituent column of a composite or conditional unique constraint as `UQ=Y`. Active records are determined by the `WHERE` clause of the relevant constraint.

Values marked Sensitive=Y may be viewed or exported only by authorized users and for authorized processing purposes. This classification includes real names, email addresses, IP addresses, auditor and approver names, and Audit JSON that may contain personal information. Exclude password_hash from responses, general exports, and Audit JSON, and retain Audit=N.

NN=N does not mean that a value is always optional, even at approval or completion. Conditional requirements follow the constraints and business rules for the relevant table. Distinguish type-plus-ID references to multiple tables and references inside JSON from actual FKs.

Database PK, FK, CHECK, and UNIQUE constraints are enforced when data is saved. Each row in a constraints table defines one database constraint or business validation. A conditional UNIQUE applies only to rows satisfying its WHERE clause. Perform business validation on the server; UI selection restrictions alone do not replace it.

When saving references, submitting for approval, or approving, verify that targets exist; that parent, project, and protocol revisions match; and that current status permits use and approval. For polymorphic and JSON references, validate the permitted actual tables, PKs, versions, and object formats. Links between project business records require the same project. For non-project materials such as shared regulations and libraries, verify permission to view and use the material. Do not allow deleted targets or business records from another project in new business links. When viewing historical approval references, preserve the original content from that time and distinguish it from subsequent disposal or invalidation status.

Control concurrent requests using server transactions and target-row locks or stored-version comparisons. Do not separate validation from saving when preventing duplicate approvals, issuing numbers, switching the latest revision, or creating an in-progress closure request. Treat requests whose stored version has changed as conflicts; do not overwrite the latest data with values from an outdated screen. Commit or roll back status transitions, signatures, and audit records together, and do not arbitrarily omit changed fields from audit records.

## 3. Table List

| No | Domain | Logical Table Name | Physical Table Name | PK | Key References (FK) | GxP | Audit |
| ---: | --- | --- | --- | --- | --- | :---: | :---: |
| 1 | Organization | Organization/Client Company | [`organization`](#table-organization) | `organization_id` | - | High | Y |
| 2 | Security | User | [`app_user`](#table-app_user) | `user_id` | `organization_id` | High | Y |
| 3 | Security | Role | [`role`](#table-role) | `role_id` | - | High | Y |
| 4 | Security | User Role | [`user_role`](#table-user_role) | `user_role_id` | `user_id, role_id` | High | Y |
| 5 | Security | User Group | [`user_group`](#table-user_group) | `user_group_id` | `organization_id, created_by, updated_by` | High | Y |
| 6 | Security | User Group Member | [`user_group_member`](#table-user_group_member) | `user_group_member_id` | `user_group_id, user_id, created_by, updated_by` | High | Y |
| 7 | Security | User Group Role | [`group_role`](#table-group_role) | `group_role_id` | `user_group_id, role_id, project_id, created_by, updated_by` | High | Y |
| 8 | Security | Menu/Project Access Permission | [`access_permission_grant`](#table-access_permission_grant) | `grant_id` | `user_id, user_group_id, project_id, created_by, updated_by` | High | Y |
| 9 | Security | Inventory Business Role | [`inventory_role_grant`](#table-inventory_role_grant) | `grant_id` | `user_id, user_group_id, created_by, updated_by` | High | Y |
| 10 | Compliance | Electronic Signature | [`electronic_signature`](#table-electronic_signature) | `signature_id` | `signer_id` | Critical | Y |
| 11 | Compliance | Audit Trail | [`audit_trail`](#table-audit_trail) | `audit_id` | `actor_id` | Critical | Y |
| 12 | File | File Asset | [`file_asset`](#table-file_asset) | `file_id` | `uploader_id` | High | Y |
| 13 | File | File Malware Scan Job | [`file_scan_job`](#table-file_scan_job) | `scan_job_id` | `previous_scan_job_id, project_id, uploaded_by, requested_by, result_file_id` | High | Y |
| 14 | File | Evidence File Link | [`evidence_link`](#table-evidence_link) | `evidence_link_id` | `project_id, file_id, created_by, updated_by` | High | Y |
| 15 | System | System/Equipment Identification Information | [`system_asset`](#table-system_asset) | `system_id` | `organization_id` | High | Y |
| 16 | System | System Inventory Revision History | [`system_asset_revision`](#table-system_asset_revision) | `system_revision_id` | `system_id, processed_by, signature_id` | High | Y |
| 17 | Library | Library Item Master | [`library_item`](#table-library_item) | `library_id` | - | High | Y |
| 18 | Validation | Validation Project | [`validation_project`](#table-validation_project) | `project_id` | `system_id, created_by, updated_by, closure_requested_by, current_closure_request_id, current_system_baseline_id` | High | Y |
| 19 | Validation | Project Validation Target Revision | [`project_system_baseline`](#table-project_system_baseline) | `baseline_id` | `project_id, system_revision_id, adopted_by` | Critical | Y |
| 20 | Validation | Project Member | [`project_member`](#table-project_member) | `project_member_id` | `project_id, user_id, role_id, created_by, updated_by` | High | Y |
| 21 | Validation | Validation Activity Master | [`validation_activity`](#table-validation_activity) | `activity_id` | `created_by, updated_by` | High | Y |
| 22 | Validation | Project Activity | [`project_activity`](#table-project_activity) | `project_activity_id` | `project_id, activity_id, created_by, updated_by` | Critical | Y |
| 23 | Validation | Activity Dependency | [`activity_dependency`](#table-activity_dependency) | `activity_dependency_id` | `successor_activity_id, predecessor_activity_id, created_by, updated_by` | Critical | Y |
| 24 | Validation | Project Closure Request | [`project_closure_request`](#table-project_closure_request) | `closure_request_id` | `project_id, requested_by, workflow_instance_id, created_by, updated_by` | High | Y |
| 25 | VP | Validation Plan | [`vp_plan`](#table-vp_plan) | `vp_id` | `project_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` | High | Y |
| 26 | VP | Validation Plan Section | [`vp_section`](#table-vp_section) | `vp_section_id` | `vp_id, created_by, updated_by` | High | Y |
| 27 | QIA | Quality Impact Assessment Header | [`qia_assessment`](#table-qia_assessment) | `qia_id` | `project_id, created_by, updated_by` | High | Y |
| 28 | QIA | QIA Module Detailed Assessment | [`qia_module_item`](#table-qia_module_item) | `qia_module_item_id` | `qia_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` | High | Y |
| 29 | QIA | QIA Process Assessment | [`qia_process`](#table-qia_process) | `qia_process_id` | `qia_module_item_id, created_by, updated_by` | High | Y |
| 30 | VA | Vendor Audit Assessment | [`vendor_audit`](#table-vendor_audit) | `audit_id` | `project_id, auditor_user_id, file_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` | High | Y |
| 31 | URS | User Requirements Specification | [`requirement`](#table-requirement) | `requirement_id` | `project_id, created_by, workflow_instance_id, updated_by, disposal_workflow_id, source_library_id` | High | Y |
| 32 | FDS | Functional Design Specification | [`fds_spec`](#table-fds_spec) | `fds_id` | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` | High | Y |
| 33 | DDS | Detailed Design Specification | [`dds_spec`](#table-dds_spec) | `dds_id` | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` | High | Y |
| 34 | DQ | Design Qualification Assessment | [`dq_assessment`](#table-dq_assessment) | `dq_id` | `project_id, created_by, updated_by` | High | Y |
| 35 | DQ | DQ Detailed Assessment Item | [`dq_item`](#table-dq_item) | `dq_item_id` | `dq_id, requirement_id, created_by, updated_by, fds_revision_id, dds_revision_id, executed_by, workflow_instance_id, disposal_workflow_id` | High | Y |
| 36 | FRA | Functional Risk Assessment | [`fra_assessment`](#table-fra_assessment) | `fra_id` | `project_id, created_by, updated_by` | High | Y |
| 37 | FRA | FRA Detailed Risk Item | [`fra_item`](#table-fra_item) | `fra_item_id` | `fra_id, requirement_id, created_by, updated_by, sop_clause_id, workflow_instance_id, disposal_workflow_id, source_library_id` | High | Y |
| 38 | IQ | Installation Qualification Assessment | [`iq_assessment`](#table-iq_assessment) | `iq_id` | `project_id, created_by, updated_by` | High | Y |
| 39 | IQ | IQ Detailed Test Item | [`iq_item`](#table-iq_item) | `iq_item_id` | `iq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_signature_id, disposed_by` | High | Y |
| 40 | IQ | IQ Test Step | [`iq_step`](#table-iq_step) | `step_id` | `iq_item_id, created_by, updated_by` | High | Y |
| 41 | IQ | IQ Test Execution | [`iq_execution`](#table-iq_execution) | `execution_id` | `iq_item_id, executed_by, execution_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` | High | Y |
| 42 | IQ | IQ Step Execution | [`iq_step_execution`](#table-iq_step_execution) | `step_execution_id` | `execution_id, step_id, confirmed_by, created_by, updated_by` | High | Y |
| 43 | OQ | Operational Qualification Assessment | [`oq_assessment`](#table-oq_assessment) | `oq_id` | `project_id, created_by, updated_by` | High | Y |
| 44 | OQ | OQ Detailed Test Item | [`oq_item`](#table-oq_item) | `oq_item_id` | `oq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_signature_id, disposed_by` | High | Y |
| 45 | OQ | OQ Test Step | [`oq_step`](#table-oq_step) | `step_id` | `oq_item_id, created_by, updated_by` | High | Y |
| 46 | OQ | OQ Test Execution | [`oq_execution`](#table-oq_execution) | `execution_id` | `oq_item_id, executed_by, execution_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` | High | Y |
| 47 | OQ | OQ Step Execution | [`oq_step_execution`](#table-oq_step_execution) | `step_execution_id` | `execution_id, step_id, confirmed_by, created_by, updated_by` | High | Y |
| 48 | PQ | Performance Qualification Assessment | [`pq_assessment`](#table-pq_assessment) | `pq_id` | `project_id, created_by, updated_by` | High | Y |
| 49 | PQ | PQ Detailed Test Item | [`pq_item`](#table-pq_item) | `pq_item_id` | `pq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_signature_id, disposed_by` | High | Y |
| 50 | PQ | PQ Test Step | [`pq_step`](#table-pq_step) | `step_id` | `pq_item_id, created_by, updated_by` | High | Y |
| 51 | PQ | PQ Test Execution | [`pq_execution`](#table-pq_execution) | `execution_id` | `pq_item_id, executed_by, execution_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` | High | Y |
| 52 | PQ | PQ Step Execution | [`pq_step_execution`](#table-pq_step_execution) | `step_execution_id` | `execution_id, step_id, confirmed_by, created_by, updated_by` | High | Y |
| 53 | VSR | Validation Summary Report | [`vsr_assessment`](#table-vsr_assessment) | `vsr_id` | `project_id, created_by, updated_by, workflow_instance_id, approval_signature_id` | Critical | Y |
| 54 | VSR | VSR Activity Summary Item | [`vsr_item`](#table-vsr_item) | `vsr_item_id` | `vsr_id, created_by, updated_by, project_activity_id, document_revision_id` | Critical | Y |
| 55 | Workflow | Workflow Instance | [`workflow_instance`](#table-workflow_instance) | `workflow_instance_id` | `requested_by, created_by, updated_by, project_id, workflow_config_id` | Critical | Y |
| 56 | Workflow | Workflow Step | [`workflow_step`](#table-workflow_step) | `workflow_step_id` | `workflow_instance_id, created_by, updated_by` | Critical | Y |
| 57 | Workflow | Approval Step Assignee | [`workflow_step_assignee`](#table-workflow_step_assignee) | `assignment_id` | `workflow_step_id, assignee_id, substitute_user_id, created_by, updated_by` | Critical | Y |
| 58 | Workflow | Approval Action History | [`approval_action`](#table-approval_action) | `approval_action_id` | `workflow_step_id, workflow_step_assignee_id, actor_id, signature_id` | Critical | Y |
| 59 | Workflow | Project Approval Route Configuration | [`project_workflow_config`](#table-project_workflow_config) | `config_id` | `project_id, project_activity_id, applied_signature_id, created_by, updated_by` | Critical | Y |
| 60 | Traceability | Common Traceability Link | [`traceability_link`](#table-traceability_link) | `traceability_link_id` | `project_id, created_by, updated_by` | Critical | Y |
| 61 | Deviation | Deviation Management | [`deviation`](#table-deviation) | `deviation_id` | `project_id, approved_by, created_by, updated_by, action_workflow_id, action_signature_id, completion_signature_id, closure_signature_id, current_action_round_id` | Critical | Y |
| 62 | Deviation | Deviation Action Round | [`deviation_action_round`](#table-deviation_action_round) | `action_round_id` | `deviation_id, workflow_instance_id, signature_id, created_by, updated_by` | Critical | Y |
| 63 | Document | Deliverable Document | [`deliverable_document`](#table-deliverable_document) | `document_id` | `project_id, project_activity_id, created_by, updated_by` | Critical | Y |
| 64 | Document | Deliverable Revision | [`deliverable_revision`](#table-deliverable_revision) | `document_revision_id` | `document_id, workflow_instance_id, approved_by, file_id, created_by, updated_by, system_baseline_id` | Critical | Y |
| 65 | Document | Deliverable Section/Content | [`deliverable_section`](#table-deliverable_section) | `section_id` | `document_revision_id, created_by, updated_by` | Critical | Y |
| 66 | Report | Report Generation Job | [`report_generation`](#table-report_generation) | `report_generation_id` | `project_id, requested_by, result_file_id, created_by, updated_by, document_revision_id` | High | Y |
| 67 | AI | AI Generation Job | [`ai_generation_job`](#table-ai_generation_job) | `ai_job_id` | `project_id, requested_by, created_by, updated_by` | High | Y |
| 68 | AI | AI Generation Result | [`ai_generation_result`](#table-ai_generation_result) | `ai_result_id` | `ai_job_id, selected_by, created_by, updated_by` | High | Y |
| 69 | AI | AI Generation Result Item | [`ai_result_item`](#table-ai_result_item) | `ai_result_item_id` | `ai_result_id, created_by, updated_by` | High | Y |
| 70 | Regulation | Regulatory Source Document | [`regulatory_source`](#table-regulatory_source) | `regulatory_source_id` | `file_id, verified_by, created_by, updated_by` | High | Y |
| 71 | Regulation | Regulatory Clause | [`regulatory_clause`](#table-regulatory_clause) | `regulatory_clause_id` | `regulatory_source_id, created_by, updated_by` | High | Y |
| 72 | Regulation | Requirement Regulatory Reference | [`requirement_regulation`](#table-requirement_regulation) | `requirement_regulation_id` | `requirement_id, regulatory_clause_id, created_by, updated_by` | High | Y |
| 73 | Regulation | Library Regulatory Reference | [`library_item_regulation`](#table-library_item_regulation) | `library_item_regulation_id` | `library_id, regulatory_clause_id, created_by, updated_by` | High | Y |

## 4. Domain Navigation

- **Organization**: [`organization`](#table-organization)
- **Security**: [`app_user`](#table-app_user), [`role`](#table-role), [`user_role`](#table-user_role), [`user_group`](#table-user_group), [`user_group_member`](#table-user_group_member), [`group_role`](#table-group_role), [`access_permission_grant`](#table-access_permission_grant), [`inventory_role_grant`](#table-inventory_role_grant)
- **Compliance**: [`electronic_signature`](#table-electronic_signature), [`audit_trail`](#table-audit_trail)
- **File**: [`file_asset`](#table-file_asset), [`file_scan_job`](#table-file_scan_job), [`evidence_link`](#table-evidence_link)
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
- **Deviation**: [`deviation`](#table-deviation), [`deviation_action_round`](#table-deviation_action_round)
- **Document**: [`deliverable_document`](#table-deliverable_document), [`deliverable_revision`](#table-deliverable_revision), [`deliverable_section`](#table-deliverable_section)
- **Report**: [`report_generation`](#table-report_generation)
- **AI**: [`ai_generation_job`](#table-ai_generation_job), [`ai_generation_result`](#table-ai_generation_result), [`ai_result_item`](#table-ai_result_item)
- **Regulation**: [`regulatory_source`](#table-regulatory_source), [`regulatory_clause`](#table-regulatory_clause), [`requirement_regulation`](#table-requirement_regulation), [`library_item_regulation`](#table-library_item_regulation)

---

# 5. Detailed Table and Column Definitions

## Organization

<a id="table-organization"></a>
### 1. Organization/Client Company (`organization`)

| Item | Definition |
| --- | --- |
| Description | Basic information about a client company or operating organization |
| Primary Key | `organization_id` |
| Key References (FK) | - |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1 | Organization ID | `organization_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique organization identifier | `00000000-0000-0000-0000-000000000001` |
| 2 | Organization Code | `organization_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Organization/company identification code | `ORG-SAMPLE-01` |
| 3 | Organization Name | `organization_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Company/site name | `샘플 주식회사` |
| 4 | Organization Type | `organization_type` | `varchar(50)` | N | N | - | Y | `'본사'` | N | N | N | Y | 본사 (Headquarters) \| 공장 (Plant) \| 연구소 (Research Institute) \| 해외법인 (Overseas Subsidiary) | `본사` |
| 5 | Status | `status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE | `ACTIVE` |
| 6 | Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Organization description | `기업 IT 및 데이터 운영 조직` |
| 7 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-01T00:00:00Z` |
| 8 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T16:00:00Z` |

#### Business Rules

Manages the organizational affiliation of users, groups, and systems.

[↑ Back to Top](#top)

---

## Security

<a id="table-app_user"></a>
### 2. User (`app_user`)

| Item | Definition |
| --- | --- |
| Description | User account and basic profile information |
| Primary Key | `user_id` |
| Key References (FK) | `organization_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 9 | User ID | `user_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique user account identifier | `00000000-0000-0000-0000-000000000001` |
| 10 | Organization ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | References organization.organization_id | `00000000-0000-0000-0000-000000000001` |
| 11 | Password Hash | `password_hash` | `varchar(255)` | N | N | - | Y | - | N | N | Y | N | Hash used for password verification. Do not include the raw hash in responses, general exports, or audit logs. Retain Audit=N. | `[REDACTED_PASSWORD_HASH]` |
| 12 | User Full Name | `full_name` | `varchar(100)` | N | N | - | Y | - | N | Y | Y | Y | User name | `홍길동` |
| 13 | Email | `email` | `varchar(254)` | N | N | - | Y | - | Y | Y | Y | Y | Email used for account invitations and login. This is the account value shown on the screen; no separate login ID is maintained. | `user@example.com` |
| 14 | Administrative Permission Level | `permission_level` | `varchar(20)` | N | N | - | Y | `'USER'` | N | N | N | Y | GLOBAL_ADMIN=Global Administrator, PROJECT_ADMIN=Project Administrator, USER=User. Distinct from roles such as author and reviewer. | `USER` |
| 15 | Department Name | `department_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Name of the affiliated department | `정보전략팀` |
| 16 | Position/Job Title | `position_title` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Position information | `선임` |
| 17 | Account Status | `status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE \| LOCKED | `ACTIVE` |
| 18 | Last Login At | `last_login_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Timestamp of the most recent system access | `2026-08-26T16:00:00Z` |
| 19 | Password Changed At | `password_changed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time of the most recent password change | `2026-08-01T09:00:00Z` |
| 20 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-01T00:00:00Z` |
| 21 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T16:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_app_user_1` | Business validation | - | Email addresses must be unique after trimming leading/trailing whitespace and normalizing case. |
| `ck_app_user_2` | CHECK | `permission_level` | CHECK (permission_level IN ('GLOBAL_ADMIN','PROJECT_ADMIN','USER')) |

#### Business Rules

Manage multiple business roles in user_role and membership in multiple groups in user_group_member. Active/inactive on the screen corresponds to ACTIVE/INACTIVE. Manage account lock status, password information, and access timestamps.

Use email as the login identifier. Manage administrative permission levels separately from menu-specific and project-specific access permissions.

[↑ Back to Top](#top)

---

<a id="table-role"></a>
### 3. Role (`role`)

| Item | Definition |
| --- | --- |
| Description | Master of permission roles such as author, reviewer, and approver |
| Primary Key | `role_id` |
| Key References (FK) | - |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 22 | Role ID | `role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique role/permission identifier | `00000000-0000-0000-0000-000000000001` |
| 23 | Role Code | `role_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | SYSTEM_ADMIN=System Administrator, ADMIN=Admin, AUTHOR=Author, REVIEWER=Reviewer, APPROVER=Approver, VIEWER=Viewer | `AUTHOR` |
| 24 | Role Name | `role_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Display name of the business role on the account screen. Multiple roles may be assigned to one user. | `전체관리자` |
| 25 | Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Detailed description of the role's permission scope | `시스템 전체 관리 권한` |
| 26 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-01T00:00:00Z` |

#### Business Rules

Store menu/project view, edit, and disposal permissions in access_permission_grant and inventory business roles in inventory_role_grant.

[↑ Back to Top](#top)

---

<a id="table-user_role"></a>
### 4. User Role (`user_role`)

| Item | Definition |
| --- | --- |
| Description | User-to-role mapping information |
| Primary Key | `user_role_id` |
| Key References (FK) | `user_id, role_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 27 | Mapping ID | `user_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique user-to-role mapping identifier. Apply composite UNIQUE (user_id, role_id) to all rows in this table. No separate active or deletion columns exist. | `00000000-0000-0000-0000-000000000001` |
| 28 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 29 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | References role.role_id | `00000000-0000-0000-0000-000000000001` |
| 30 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_user_role_1` | UNIQUE | `user_id, role_id` | UNIQUE (user_id, role_id) |

[↑ Back to Top](#top)

---

<a id="table-user_group"></a>
### 5. User Group (`user_group`)

| Item | Definition |
| --- | --- |
| Description | Manages basic information and active status of user groups by organization |
| Primary Key | `user_group_id` |
| Key References (FK) | `organization_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 31 | User Group ID | `user_group_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique user group identifier | `UUID` |
| 32 | Organization ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | Organization to which the user group belongs | `UUID` |
| 33 | Group Code | `group_code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | User group identification code within the organization. Do not allow duplicate (organization_id, group_code) values for rows where deleted_at IS NULL and is_active is TRUE. | `QA_REVIEWER_GROUP` |
| 34 | Group Name | `group_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Group name displayed to users | `QA 검토자 그룹` |
| 35 | Group Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Description of the group's purpose and permission scope | `Validation 문서 QA 검토 담당자 그룹` |
| 36 | In Use | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the user group is active | `TRUE` |
| 37 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | User group creation timestamp (UTC) | `2026-09-14T09:00:00Z` |
| 38 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who created the user group | `UUID` |
| 39 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | User group last modification timestamp (UTC) | `2026-09-14T09:00:00Z` |
| 40 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the user group | `UUID` |
| 41 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | User group soft-deletion timestamp | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_user_group_1` | UNIQUE | `organization_id, group_code, deleted_at, is_active` | UNIQUE (organization_id, group_code) WHERE deleted_at IS NULL AND is_active=TRUE |

[↑ Back to Top](#top)

---

<a id="table-user_group_member"></a>
### 6. User Group Member (`user_group_member`)

| Item | Definition |
| --- | --- |
| Description | Manages the N:M relationship between users and user groups, and group membership status and periods |
| Primary Key | `user_group_member_id` |
| Key References (FK) | `user_group_id, user_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 42 | Group Member ID | `user_group_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique user group membership mapping identifier | `UUID` |
| 43 | User Group ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | Y | - | N | Y | N | Y | User group to which the member belongs | `UUID` |
| 44 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User included in the group. Do not allow duplicate (user_group_id, user_id) values for rows where deleted_at IS NULL and member_status is ACTIVE. | `UUID` |
| 45 | Membership Status | `member_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | Y | N | Y | Group membership status: ACTIVE, INACTIVE, WITHDRAWN | `ACTIVE` |
| 46 | Joined At | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time when the user joined the group | `2026-09-14T09:00:00Z` |
| 47 | Left At | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the user's group membership ended. NULL for a current member. | `2026-09-01T00:00:00Z` |
| 48 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Group membership mapping creation timestamp (UTC) | `2026-09-14T09:00:00Z` |
| 49 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the group member | `UUID` |
| 50 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Group membership mapping last modification timestamp (UTC) | `2026-09-14T09:00:00Z` |
| 51 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the group membership mapping | `UUID` |
| 52 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Group membership mapping soft-deletion timestamp | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_user_group_member_1` | UNIQUE | `user_group_id, user_id, deleted_at, member_status` | UNIQUE (user_group_id, user_id) WHERE deleted_at IS NULL AND member_status='ACTIVE' |

#### Business Rules

A user may join multiple groups. Aggregate group permissions from the user's active group memberships.

[↑ Back to Top](#top)

---

<a id="table-group_role"></a>
### 7. User Group Role (`group_role`)

| Item | Definition |
| --- | --- |
| Description | Assigns roles to user groups and manages their global or project-specific scope |
| Primary Key | `group_role_id` |
| Key References (FK) | `user_group_id, role_id, project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 53 | Group Role ID | `group_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique permission mapping identifier between a user group and a role | `UUID` |
| 54 | User Group ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | Y | - | N | Y | N | Y | User group receiving the role | `UUID` |
| 55 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | Role assigned to the group | `UUID` |
| 56 | Permission Scope Type | `scope_type` | `varchar(20)` | N | N | - | Y | `'GLOBAL'` | N | Y | N | Y | Scope of the group role: GLOBAL, PROJECT | `PROJECT` |
| 57 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Required when scope_type is PROJECT; NULL when GLOBAL. Validate the correspondence between scope and NULL using a CHECK constraint. | `UUID` |
| 58 | In Use | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the group role mapping is in use. Enforce scope-specific uniqueness for rows where deleted_at IS NULL and is_active is TRUE. Use (user_group_id, role_id) for GLOBAL and (user_group_id, role_id, project_id) for PROJECT. | `TRUE` |
| 59 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Group role mapping creation timestamp (UTC) | `2026-09-14T09:00:00Z` |
| 60 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who assigned the group role | `UUID` |
| 61 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Group role mapping last modification timestamp (UTC) | `2026-09-14T09:00:00Z` |
| 62 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the group role mapping | `UUID` |
| 63 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Group role mapping soft-deletion timestamp | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `ck_group_role_1` | CHECK | `scope_type, project_id` | CHECK ((scope_type='GLOBAL' AND project_id IS NULL) OR (scope_type='PROJECT' AND project_id IS NOT NULL)) |

#### Business Rules

Maps the group's business roles. Manage the screen's menu-specific and project-specific permission checkboxes in access_permission_grant.

[↑ Back to Top](#top)

---

<a id="table-access_permission_grant"></a>
### 8. Menu/Project Access Permission (`access_permission_grant`)

| Item | Definition |
| --- | --- |
| Description | Screen view, edit, and disposal permissions granted to an individual or group |
| Primary Key | `grant_id` |
| Key References (FK) | `user_id, user_group_id, project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 64 | Permission ID | `grant_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 65 | Individual User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 66 | User Group ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | N | - | N | Y | N | Y | References user_group.user_group_id | `00000000-0000-0000-0000-000000000001` |
| 67 | Permission Scope | `scope_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | MENU=Menu, PROJECT=Project | `MENU` |
| 68 | Menu Code | `menu_code` | `varchar(40)` | N | N | - | N | - | N | N | N | Y | SYSTEM_INVENTORY/PROJECT_MANAGEMENT/LIBRARY/AUDIT_TRAIL/ACCOUNT_PERMISSION. Required for MENU scope. | `LIBRARY` |
| 69 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Required for PROJECT scope; NULL for MENU scope. | `00000000-0000-0000-0000-000000000001` |
| 70 | View Allowed | `can_view` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | View checkbox on the screen | `FALSE` |
| 71 | Edit Allowed | `can_edit` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Edit checkbox on the screen | `FALSE` |
| 72 | Disposal Allowed | `can_dispose` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Disposal checkbox on the project permissions screen. FALSE for menu scope. | `FALSE` |
| 73 | Is Active | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Set to FALSE when the permission is revoked. | `TRUE` |
| 74 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation timestamp value | `2026-09-01T00:00:00Z` |
| 75 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 76 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Modification timestamp value | `2026-09-01T00:00:00Z` |
| 77 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_access_permission_grant_1` | CHECK | `user_id, user_group_id` | CHECK ((user_id IS NOT NULL AND user_group_id IS NULL) OR (user_id IS NULL AND user_group_id IS NOT NULL)) |
| `ck_access_permission_grant_2` | CHECK | `scope_type, menu_code, project_id, can_dispose` | CHECK ((scope_type='MENU' AND menu_code IS NOT NULL AND project_id IS NULL AND can_dispose=FALSE) OR (scope_type='PROJECT' AND project_id IS NOT NULL AND menu_code IS NULL)) |
| `rule_access_permission_grant_3` | Business validation | `can_edit` | If can_edit or can_dispose is TRUE, can_view must also be TRUE. can_edit is FALSE for the Audit Trail menu. |
| `rule_access_permission_grant_4` | Business validation | - | Do not allow duplicate active permissions for any individual+menu, group+menu, individual+project, or group+project combination. |
| `uq_access_user_id_menu` | UNIQUE | `user_id, menu_code, is_active, scope_type` | UNIQUE (user_id, menu_code) WHERE is_active=TRUE AND user_id IS NOT NULL AND scope_type='MENU' |
| `uq_access_user_id_project` | UNIQUE | `user_id, project_id, is_active, scope_type` | UNIQUE (user_id, project_id) WHERE is_active=TRUE AND user_id IS NOT NULL AND scope_type='PROJECT' |
| `uq_access_user_group_id_menu` | UNIQUE | `user_group_id, menu_code, is_active, scope_type` | UNIQUE (user_group_id, menu_code) WHERE is_active=TRUE AND user_group_id IS NOT NULL AND scope_type='MENU' |
| `uq_access_user_group_id_project` | UNIQUE | `user_group_id, project_id, is_active, scope_type` | UNIQUE (user_group_id, project_id) WHERE is_active=TRUE AND user_group_id IS NOT NULL AND scope_type='PROJECT' |
| `ck_access_view_required` | CHECK | `can_edit, can_dispose, can_view` | CHECK ((NOT can_edit AND NOT can_dispose) OR can_view) |

#### Business Rules

Calculate effective screen permissions by combining direct individual grants with grants inherited from active group memberships. Do not duplicate inherited group permissions as direct grants for each user.

To determine final permissions, first verify that the account is active, then combine direct individual permissions and permissions granted through active groups and memberships using OR for each function within the relevant menu/project scope. Deny any permission that has not been granted. Editing and disposal require both the relevant access permission and business role. Review and approval also require a current Workflow assignment or valid substitute assignment. Access permissions do not automatically grant approval roles, and business roles do not automatically grant access permissions.

Administrative permission levels determine the scope of administrative functions. Calculate global roles from user_role and GLOBAL group_role, and project roles from project_member and PROJECT group_role for the relevant project. Apply inventory_role_grant to inventory operations. Administrators must also comply with electronic-signature identity verification, assignee assignment, separation of authors/reviewers/approvers, and approved-content locking rules. Permission changes, group withdrawal, and deactivation affect effective permissions from the next request; the server makes the final authorization decision.

[↑ Back to Top](#top)

---

<a id="table-inventory_role_grant"></a>
### 9. Inventory Business Role (`inventory_role_grant`)

| Item | Definition |
| --- | --- |
| Description | Selections of inventory author, reviewer, approver, and disposal-authorized roles for individuals and groups |
| Primary Key | `grant_id` |
| Key References (FK) | `user_id, user_group_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 78 | Inventory Role ID | `grant_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 79 | Individual User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 80 | User Group ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | N | - | N | Y | N | Y | References user_group.user_group_id | `00000000-0000-0000-0000-000000000001` |
| 81 | Inventory Role | `role_code` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | AUTHOR=Author, REVIEWER=Reviewer, APPROVER=Approver, DISPOSER=Disposal-authorized User | `APPROVER` |
| 82 | Is Active | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Active status value | `TRUE` |
| 83 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation timestamp value | `2026-09-01T00:00:00Z` |
| 84 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 85 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Modification timestamp value | `2026-09-01T00:00:00Z` |
| 86 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_inventory_role_grant_1` | CHECK | `user_id, user_group_id` | CHECK ((user_id IS NOT NULL AND user_group_id IS NULL) OR (user_id IS NULL AND user_group_id IS NOT NULL)) |
| `ck_inventory_role_grant_2` | CHECK | `role_code` | CHECK (role_code IN ('AUTHOR','REVIEWER','APPROVER','DISPOSER')) |
| `rule_inventory_role_grant_3` | Business validation | - | Do not allow duplicate active individual+role or active group+role combinations. |
| `uq_inventory_user_id` | UNIQUE | `user_id, role_code, is_active` | UNIQUE (user_id, role_code) WHERE is_active=TRUE AND user_id IS NOT NULL |
| `uq_inventory_user_group_id` | UNIQUE | `user_group_id, role_code, is_active` | UNIQUE (user_group_id, role_code) WHERE is_active=TRUE AND user_group_id IS NOT NULL |

#### Business Rules

Inventory permissions specified on the screen separately from the account's general business roles. Combine individual grants and inheritance from active groups.

[↑ Back to Top](#top)

---

## Compliance

<a id="table-electronic_signature"></a>
### 10. Electronic Signature (`electronic_signature`)

| Item | Definition |
| --- | --- |
| Description | Electronic-signature records for submission, review, approval, disposal, execution confirmation, approval-route application, and closure requests |
| Primary Key | `signature_id` |
| Key References (FK) | `signer_id` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 87 | Electronic Signature ID | `signature_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique electronic-signature record identifier | `00000000-0000-0000-0000-000000000001` |
| 88 | Signer ID | `signer_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 89 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Actual target table name. Distinguishes business item revisions, deliverable revisions, and closure requests. Identify inventory by the system_asset PK and revision_number. | `requirement` |
| 90 | Target Record ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | PK of the record to which the signature applies | `00000000-0000-0000-0000-000000000001` |
| 91 | Signature Action | `signature_action` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | SUBMIT / REVIEW / APPROVE / REJECT / DISPOSE / CONFIG_APPLY / EXECUTE. Record the signature meaning together with the target business operation. | `APPROVE` |
| 92 | Signature Meaning | `signature_meaning` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Business meaning of submission/review/approval/disposal/test execution confirmation. For closure requests, PROJECT_CLOSE_NORMAL or PROJECT_CLOSE_FORCED. | `문서 최종 승인` |
| 93 | Signature Timestamp | `signed_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp of electronic-signature execution | `2026-08-26T16:30:00Z` |
| 94 | Target Version | `target_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Revision display version or revision number. Closure requests: CLOSE-{request_version}; deviation actions: ACTION-{round_number}-{revision_number}; completion reports: COMPLETE-{last action round}-{latest rerun PK}; closure without execution: CLOSE-{last action round}. Test execution identifies both the attempt and the result revision. | `v1.0` |
| 95 | Signed Content Hash | `content_hash` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | SHA-256 in 64 lowercase hexadecimal characters of the entire canonical UTF-8 JSON representation of signed_payload, including payload_schema_version. The same value must be reproducible from the stored original content. | `[SAMPLE_SHA256_HASH]` |
| 96 | Reauthentication Method | `authentication_method` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Reauthentication method used to verify the signer's identity when executing the electronic signature | `PASSWORD` |
| 97 | Reauthentication Result | `authentication_result` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Result of reauthentication performed for the electronic signature | `SUCCESS` |
| 98 | Signer Display Name | `signer_name_snapshot` | `varchar(100)` | N | N | - | Y | - | N | N | Y | Y | User name displayed on the screen at the time of signing | `홍길동` |
| 99 | Signer Role | `signer_role_snapshot` | `varchar(100)` | N | N | - | Y | - | N | N | Y | Y | Role at the time of signing, such as author/reviewer/approver | `승인자` |
| 100 | Signed Payload Format Version | `payload_schema_version` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Version identifying the included fields and canonicalization rules for each target type and signature purpose | `SIGN-1` |
| 101 | Signed Payload | `signed_payload` | `jsonb` | N | N | - | Y | - | N | N | Y | Y | Server-finalized {schema_version,target,baseline_id,body,children,source_refs,files,context}. Include sections that do not apply to a target as empty arrays or NULL. Exclude passwords and authentication secrets. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_electronic_signature_1` | Business validation | - | target_table_name and target_record_id must together identify the target's actual PK and revision. This is a polymorphic reference to multiple tables, not a physical FK. |
| `rule_electronic_signature_2` | Business validation | - | The signer must match the acting user. Store only successful reauthentication results as valid signatures. |
| `rule_electronic_signature_3` | Business validation | - | Do not modify content after signing. Changes to the target require a new revision and a new signature. |
| `ck_signature_hash` | CHECK | `content_hash` | CHECK (content_hash ~ '^[0-9a-f]{64}$') |

#### Business Rules

Link disposal, test execution confirmation, and approval-route application through the signature FK of the relevant business record. Use workflow and approval_action for general approval processing.

Identify closure signatures by the project_closure_request PK and CLOSE-{request_version}. Preserve historical signed payloads and hashes.

signed_payload includes the body reviewed by the signer, ordered child steps, the table/PK/version/content hash of each supporting target, the file_id/content_hash/size of original attachments already present at signing, and the baseline_id of the project's validation target. Do not retroactively add PDF output files generated later to this original attachment list in the approved payload. context includes the signature purpose and action reason. Completion reports include corrective/preventive actions and the original completion report; closure without execution includes the closure reason; disposal includes the disposal reason and the original approved target identifier. source_refs references only targets whose exact revision content has been preserved. Content changed for resubmission after rejection is also preserved separately in each signature's signed_payload.

For canonicalization, sort object keys by character code and serialize whitespace-free JSON as UTF-8. Preserve the order of arrays with business ordering, and sort set-like reference and file lists by identifier. Use standard UTC strings for dates, lowercase UUIDs, and explicit NULLs. Preserve the numeric and string conversion rules defined for each payload_schema_version. After the server rereads and validates the target, child data, and file hashes, commit signed-payload storage, the status transition, and audit records in the same transaction.

Exclude administrative fields such as approval status, current-revision indicators, updated_at, approval-progress summaries, subsequent disposal status, and invalidated_at from the original business approval body. Audit changes to administrative fields as well, without updating the original signed_payload or content_hash. For separate signatures such as disposal or closure, include the reason for that action in context. Do not overwrite or reapprove the final approved payload for the same target, purpose, and version. Changes to the business content require a new version or new action revision.

[↑ Back to Top](#top)

---

<a id="table-audit_trail"></a>
### 11. Audit Trail (`audit_trail`)

| Item | Definition |
| --- | --- |
| Description | Audit trail records of significant data changes, before/after values, and acting users |
| Primary Key | `audit_id` |
| Key References (FK) | `actor_id` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 102 | Audit Trail ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique audit trail record identifier | `00000000-0000-0000-0000-000000000001` |
| 103 | Actor ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Required for user actions. NULL allowed for system events. | `00000000-0000-0000-0000-000000000001` |
| 104 | Action Type | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | CREATE, UPDATE, DELETE, EXPORT, LOGIN, LOGOUT, REPORT_GENERATE, REPORT_DOWNLOAD, REPORT_CANCEL, PROJECT_CLOSE, PROJECT_FORCE_CLOSE | `REPORT_GENERATE` |
| 105 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | Actual target table name. NULL for events without a target row. | `requirement` |
| 106 | Target Record ID | `target_record_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Target PK. NULL for events without a target row, such as login/logout. | `00000000-0000-0000-0000-000000000001` |
| 107 | Data Before Change | `old_values` | `jsonb` | N | N | - | N | - | N | N | Y | Y | Business values before/after the change. Exclude passwords, tokens, and password_hash. | `{"status": "작성중"}` |
| 108 | Data After Change | `new_values` | `jsonb` | N | N | - | N | - | N | N | Y | Y | Business values before/after the change. Exclude passwords, tokens, and password_hash. | `{"status": "승인완료"}` |
| 109 | Reason for Change | `reason_for_change` | `text` | N | N | - | N | - | N | N | N | Y | Reason for a data change under 21 CFR Part 11 | `요구사항 오탈자 수정 및 규정 항목 보완` |
| 110 | Client IP Address | `client_ip` | `varchar(45)` | N | N | - | N | - | N | N | Y | Y | IP address of the user's client | `192.0.2.10` |
| 111 | Occurred At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Audit trail log creation timestamp (UTC) | `2026-08-26T16:30:00Z` |
| 112 | Actor Type | `actor_type` | `varchar(20)` | N | N | - | Y | `'USER'` | N | Y | N | Y | Type of entity performing the change: USER, SYSTEM, or BATCH. actor_id is required for USER. | `USER` |
| 113 | Request ID | `request_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Identifier grouping multiple audit trail records generated by a single screen/API request | `00000000-0000-0000-0000-000000000001` |
| 114 | Session ID | `session_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | User login session identifier in which the change occurred. NULL allowed for system/batch processing or requests without a session. | `00000000-0000-0000-0000-000000000001` |
| 115 | Target Document Version | `target_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | For revision-controlled documents, stores the display version at the time of the change. NULL allowed for ordinary tables. For project closure requests and closure records, stores the same CLOSE-n version as the electronic signature. | `v1.0` |
| 116 | Target Revision Number | `target_revision_number` | `integer` | N | N | - | N | - | N | N | N | Y | For revision-controlled documents, stores the numeric revision sequence at the time of the change. NULL allowed for ordinary tables. | `1` |
| 117 | Request Path | `request_uri` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | Screen or API request path that caused the change. NULL allowed when no path exists, such as system/batch processing. | `/api/fds/approve` |
| 118 | Client Information | `user_agent` | `text` | N | N | - | N | - | N | N | Y | Y | Browser, operating system, or client application information used for the change request | `Sample-Client/1.0` |
| 119 | Menu Category | `menu_code` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | INVENTORY / PROJECT / LIBRARY / ACCOUNTS / VP / VA / QIA / URS / FDS_GROUP / FRA / DQ / IQ / OQ / PQ / VSR / DASHBOARD / AUTH | `URS` |
| 120 | Actor Role | `actor_role_snapshot` | `varchar(100)` | N | N | - | N | - | N | N | Y | Y | Role filter on the audit screen and display of the role at the time of the action | `작성자` |
| 121 | Actor Display Name | `actor_display_snapshot` | `varchar(100)` | N | N | - | N | - | N | N | Y | Y | User name at the time of the action, displayed on the audit screen | `홍길동` |
| 122 | Changed Field | `field_path` | `text` | N | N | - | N | - | N | N | N | Y | Column/item path displayed in the field-level change view. NULL for a whole event. | `requirement_text` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_audit_trail_1` | Business validation | - | The target table and ID must either both be present or both be NULL. |
| `rule_audit_trail_2` | Business validation | `actor_id` | actor_id is required for USER actors and may be NULL for SYSTEM actors. |
| `rule_audit_trail_3` | Business validation | - | Audit records are append-only and cannot be modified or deleted through the UI. |

[↑ Back to Top](#top)

---

## File

<a id="table-file_asset"></a>
### 12. File Asset (`file_asset`)

| Item | Definition |
| --- | --- |
| Description | Manages storage metadata for user attachments, evidence files, generated reports, and data export files |
| Primary Key | `file_id` |
| Key References (FK) | `uploader_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the auditable period according to the retention policies of the file type and linked target. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 123 | File ID | `file_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique attachment metadata identifier | `00000000-0000-0000-0000-000000000001` |
| 124 | Uploader ID | `uploader_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Uploader ID is required for user-uploaded files. NULL allowed for files generated by the system or scheduled batch processing. | `UUID` |
| 125 | Original File Name | `original_file_name` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | File name at the time of upload | `sample_document.pdf` |
| 126 | Stored File Path | `stored_file_path` | `text` | N | N | - | Y | - | N | N | N | Y | Storage path/S3 key | `documents/2026/08/sample_001.pdf` |
| 127 | File Category | `file_category` | `varchar(30)` | N | N | - | Y | `'ATTACHMENT'` | N | Y | N | Y | Business file category: ATTACHMENT, EVIDENCE, REPORT, EXPORT | `REPORT` |
| 128 | File Size | `file_size_bytes` | `bigint` | N | N | - | Y | `0` | N | N | N | Y | File size in bytes | `1048576` |
| 129 | MIME Type | `mime_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Attachment format. NULL allowed if unknown. | `application/pdf` |
| 130 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T16:30:00Z` |
| 131 | File Content Hash | `content_hash` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | SHA-256 in 64 lowercase hexadecimal characters of the original bytes of the fully stored file. Calculated and verified by the server. | - |
| 132 | Storage Object Version | `storage_version_id` | `varchar(255)` | N | N | - | N | - | N | N | N | Y | Exact object version ID in versioned storage. NULL if versioning is unavailable; overwriting stored_file_path is prohibited. | - |
| 133 | File Origin | `file_origin` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | UPLOAD / SYSTEM_GENERATED. The server assigns this based on the actual creation path. User uploads must pass scanning and be linked to a scan job. Distinct from file_category. | `UPLOAD` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `ck_file_hash` | CHECK | `content_hash` | CHECK (content_hash ~ '^[0-9a-f]{64}$') |
| `ck_file_size` | CHECK | `file_size_bytes` | CHECK (file_size_bytes >= 0) |
| `ck_file_origin` | CHECK | `file_origin` | CHECK (file_origin IN ('UPLOAD','SYSTEM_GENERATED')) |
| `rule_file_asset_scan` | Business validation | `file_origin, file_id` | Register UPLOAD files by linking to result_file_id of a file_scan_job completed with no threats found. Verify matching hash, size, uploader, file name, and file category. |
| `rule_file_asset_origin` | Business validation | `file_origin` | Allow SYSTEM_GENERATED only when a server-internal creation path has been verified. Do not decide to skip scanning based on user-supplied origin or file category. |

#### Business Rules

Link VA/FDS/DDS attachments and test evidence by file ID. Preserve file replacement history with the linked target's revision/execution attempt.

Do not overwrite an attachment used at approval with another file.

Create a usable file_asset only after file storage and server-side hash verification are complete. Record user uploads with file_origin=UPLOAD; additionally, the latest file_scan_job attempt must be COMPLETED and NO_THREATS_FOUND, and the scanned original and stored file must have identical bytes. Commit file asset creation and the scan job's result_file_id link in the same transaction. Do not register files with scans pending, in progress, failed, unscannable, or threats found as usable files in this table.

The server determines file_origin by verifying the actual file creation path. Record PDF/export files generated by trusted server-internal processing as SYSTEM_GENERATED; they may be registered without a user-upload scan job. External uploaded files used for generation must first pass scanning. User-uploaded files remain UPLOAD and require scanning even when file_category is REPORT or EXPORT.

Do not use uploads with only metadata, or incomplete scanning/registration, for business attachments, download/preview, document parsing/AI input, evidence, approval, or report completion. File references in VA/FDS/DDS, regulatory originals, evidence, and documents may link only to file_id values satisfying these registration conditions.

Replace a file using a new file_id and a new storage object. During retention, preserve the original bytes, path, version, and hash, and block object overwriting and deletion. If storage versioning exists, specify that version when retrieving the file. Distinct attachments with identical content are allowed, so content_hash is not UNIQUE. Downloads require view permission for the linked business target; being the uploader alone does not grant access to other business materials.

[↑ Back to Top](#top)

---

<a id="table-file_scan_job"></a>
### 13. File Malware Scan Job (`file_scan_job`)

| Item | Definition |
| --- | --- |
| Description | Quarantine storage information, malware scan attempts/results, and links to file asset registration after user uploads pass scanning |
| Primary Key | `scan_job_id` |
| Key References (FK) | `previous_scan_job_id, project_id, uploaded_by, requested_by, result_file_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain scan attempts, verdicts, and file registration history for the auditable period. Manage quarantined originals under a separate quarantine-file retention/deletion policy, distinct from scan results. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 134 | Scan Job ID | `scan_job_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of one scan attempt. Used to prevent duplicate processing of queue deliveries and received results. | `UUID` |
| 135 | Upload ID | `upload_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Server-issued logical identifier for one upload. Retry attempts for the same original retain this value; it is not an FK to a separate table. | `UUID` |
| 136 | Scan Attempt Number | `attempt_no` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | The initial scan is 1. Add retries/rescans as the next attempt for the same upload_id; duplicate message receipt alone must not increment it. | `1` |
| 137 | Previous Scan Job ID | `previous_scan_job_id` | `uuid` | N | Y | `file_scan_job.scan_job_id` | N | - | Y | Y | N | Y | Immediately preceding attempt for a retry/rescan. NULL for the initial attempt. A preceding job must not branch into multiple subsequent attempts. | `UUID` |
| 138 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Relevant project for a project attachment. NULL allowed for non-project uploads such as regulatory originals; validate the separate menu/material access permissions. | `UUID` |
| 139 | Uploader ID | `uploaded_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who uploaded the original. Preserve the original uploader even during automatic retries. | `UUID` |
| 140 | Original File Name | `original_file_name` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | File name at upload. Use the same value when registering the file asset. | `design_document.pdf` |
| 141 | Quarantine Storage Name | `storage_bucket` | `varchar(255)` | N | N | - | Y | - | N | Y | N | Y | Identifier of the bucket or storage location containing the scan original. Do not store connection secrets or signed URLs. | `upload-quarantine-example` |
| 142 | Quarantine File Path | `stored_file_path` | `text` | N | N | - | Y | - | N | Y | N | Y | Quarantined object path unique to the upload. Do not expose it as a business download path. | `uploads/sample-upload/design_document.pdf` |
| 143 | Quarantine Object Version | `storage_version_id` | `varchar(255)` | N | N | - | N | - | N | Y | N | Y | Exact object version scanned. NULL for storage without versioning; overwriting the original path is prohibited. | - |
| 144 | Original Content Hash | `content_hash` | `varchar(64)` | N | N | - | N | - | N | N | N | Y | SHA-256 in 64 lowercase hexadecimal characters of the original bytes, verified by the server or trusted scanning process. NULL before verification; required for completion with no threats found. | - |
| 145 | File Size | `file_size_bytes` | `bigint` | N | N | - | Y | - | N | N | N | Y | Actual byte size of the fully stored original | `1048576` |
| 146 | MIME Type | `mime_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Verified original file format. Do not trust the file type based solely on user input. | `application/pdf` |
| 147 | File Category | `file_category` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | ATTACHMENT / EVIDENCE / REPORT / EXPORT. A business-purpose classification, not a basis for skipping scanning. | `ATTACHMENT` |
| 148 | Job Status | `job_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | PENDING / PROCESSING / COMPLETED / FAILED / CANCELLED. COMPLETED means the scan has ended; permission to use the file depends on scan_result. | `PENDING` |
| 149 | Scan Result | `scan_result` | `varchar(30)` | N | N | - | N | - | N | Y | N | Y | NO_THREATS_FOUND / THREATS_FOUND / UNSCANNABLE. Distinguishes no threats found, threats found, and inability to scan. NULL before completion or for technical failure/cancellation. | `NO_THREATS_FOUND` |
| 150 | Scan Service/Engine Name | `scanner_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Identifier of the scanning service or engine assigned to this attempt. Not tied to a specific product. | `MALWARE_SCANNER` |
| 151 | Scanner Version | `scanner_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Engine version actually used. NULL allowed if the scanning service does not provide it. | - |
| 152 | Detection Signature Version | `signature_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Version of the malware detection information used in scanning. NULL allowed if unavailable. | - |
| 153 | Scan Service Request ID | `scanner_request_id` | `varchar(255)` | N | N | - | N | - | N | Y | N | Y | Execution/request identifier issued by the scanning service. Used to match results against the job and original object. | - |
| 154 | Scan Details | `scan_details` | `jsonb` | N | N | - | N | - | N | N | Y | Y | Verifiable scanning evidence such as provider_result, reason_code, threat_names, and report_reference. Exclude passwords, tokens, signed URLs, and file contents. | `{"provider_result":"NO_THREATS_FOUND","threat_names":[]}` |
| 155 | Scan Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User requesting a manual scan/rescan. NULL allowed for automated processing, which is recorded with a SYSTEM/BATCH actor in the Audit Trail. | `UUID` |
| 156 | Scan Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time this scan attempt was registered | `2026-09-18T01:00:00Z` |
| 157 | Scan Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time scanning started for this attempt. May be NULL for failure/cancellation before execution. | `2026-09-18T01:00:05Z` |
| 158 | Scan Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time completion, failure, or cancellation was finalized. Distinct from the file asset registration time. | `2026-09-18T01:00:20Z` |
| 159 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Last modification time of the job status, scan result, or result-file link | `2026-09-18T01:00:21Z` |
| 160 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Technical error code for FAILED. Threat detection is not a technical error and is recorded in scan_result. | `SCAN_TIMEOUT` |
| 161 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | Y | Y | Detailed reason for technical failure. Must not include secrets, signed URLs, or file contents. | `검사 응답시간 초과` |
| 162 | Registered File ID | `result_file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | Y | Y | N | Y | File asset registered after no threats were found and identity with the original was verified. NULL before registration; only one attempt for the same upload may hold this link. | `UUID` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
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
| `rule_file_scan_chain` | Business validation | `upload_id, attempt_no, previous_scan_job_id` | The previous attempt must be a terminal row with attempt_no-1 for the same upload_id. Preserve storage, path, version, size, uploader, project, and file category for the same upload, and do not change a verified hash. |
| `rule_file_scan_registration` | Business validation | `result_file_id, content_hash` | Register only if the latest attempt for the upload found no threats and the upload has not yet been registered. The file_asset must have file_origin=UPLOAD, and its uploader, file name, category, size, and hash must match the scanned original. |
| `rule_file_scan_access` | Business validation | `project_id, uploaded_by, requested_by` | Validate the association of the database, storage, user, and project with the authenticated client company, and permission to register files for the relevant business operation. Do not select the client company's database or determine access permissions solely from an organization ID or storage path supplied in a request. |

#### Business Rules

**Scan Scope and Quarantine.** After a user-uploaded file has been fully stored, register its initial scan attempt in this table. Verify file type, size, and upload permission; allow only the scanning actor to read the quarantined original. Before scanning passes and the file asset is registered, the file must not be used for ordinary-user download/preview, business attachments, document parsing/AI input, approval, evidence, or report completion. Do not skip scanning of user uploads even when file_category is REPORT or EXPORT.

**Jobs and Verdicts.** One row represents one scan attempt, transitioning from PENDING to PROCESSING and then terminating as COMPLETED or FAILED. Failure before starting and cancellation may also be recorded. Only COMPLETED with NO_THREATS_FOUND is eligible for file registration; THREATS_FOUND and UNSCANNABLE remain blocked. If the entire original cannot be scanned because of encryption, unsupported format, scan-scope limitations, or similar reasons, record UNSCANNABLE and the reason in scan_details. Record access errors, timeouts, and scanning service failures as FAILED. Do not convert unknown service responses or missing results into a no-threats-found verdict.

**Retries and Concurrency.** Requests/results exchanged with queues and scanning services use scan_job_id and verified client-company connection information, and are checked against the original storage location, path, and object version. Update status conditionally on the current status; duplicate deliveries/results must not repeat scanning or duplicate file asset registration. Do not overwrite the status, original, verdict, or scanning evidence of a terminal attempt. To run again after a technical failure, create the next attempt for the same upload_id. Rescans after UNSCANNABLE or threats found require an authorized request and recorded reason; if file contents change, register a new upload_id. A delayed response for an earlier attempt must not turn failure/cancellation into success or replace the latest attempt's verdict.

**Job Delivery and Recovery.** Pending jobs registered in the database must remain redeliverable after queue transmission failure. For in-progress jobs with no result within the time limit, verify the actual scan execution before finalizing failure and registering the next attempt for recovery. Manage maximum attempts and operational review procedures for failed jobs through processing policies; do not permit unlimited retries. If service-internal retries are not exposed as individual attempts, record only verifiable request identifiers and final results; do not fabricate execution history that was not received.

**Scanned Originals and Registered Files.** A scan result is valid only for the specified object version or an original protected against overwriting. When copying to business storage after a no-threats-found result, verify that the final stored object's hash and size match the scanned original, and record its final path and version in file_asset. Do not register a file transformed or edited after scanning as the same scanned original. Preserve the location and version of the initial scan target in this table.

**Consistency of File Registration.** Commit file_asset creation, this attempt's result_file_id link, and audit records in the same database transaction. External storage operations are separate from the database transaction, so do not expose a file as usable before final storage verification. Lock the upload's initial attempt or apply equivalent concurrency control so that creating the next attempt and registering the file cannot both commit concurrently. Even if no threats were found, result_file_id remains NULL and use is prohibited until registration completes. After storage/registration failure, registration may resume for the same attempt after identity with the original is reverified; do not change the terminal scan result. Do not create additional scan attempts in this table for an upload that has already been registered.

**Authorization and Result Trust.** Workers and result receivers select the database and storage using client-company connection information managed by the server. Validate the sender, execution identity, and original object for scan results; do not finalize a verdict from user input. Passing a scan does not imply file access permission or business approval. Recheck existing business permissions when actually linking or downloading the file.

**History and Retention.** Record scan requests, status transitions, final verdicts, and file registration in the Audit Trail. Retain scan results linked to retained files for those files' auditable period. Manage deletion and retention periods for unregistered, malicious, and unscannable originals under the quarantine-file policy; preserve scan results and processing reasons even after originals are cleaned up. Do not apply the long-term locking policy for approved evidence indiscriminately to quarantined originals.

[↑ Back to Top](#top)

---

<a id="table-evidence_link"></a>
### 14. Evidence File Link (`evidence_link`)

| Item | Definition |
| --- | --- |
| Description | Manages N:M links between documents/test items and evidence files |
| Primary Key | `evidence_link_id` |
| Key References (FK) | `project_id, file_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 163 | Evidence Link ID | `evidence_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique link identifier between a document/test item and an evidence file. Do not allow duplicate (project_id, file_id, target_entity_type, target_entity_id, evidence_type) values where deleted_at IS NULL. | `UUID` |
| 164 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | ID of the project to which the evidence link belongs | `UUID` |
| 165 | File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | Y | - | N | Y | N | Y | ID of the linked evidence file | `UUID` |
| 166 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT / FDS_SPEC / DDS_SPEC / VP_PLAN / QIA_MODULE_ITEM / VENDOR_AUDIT / DQ_ITEM / FRA_ITEM / IQ_ITEM / OQ_ITEM / PQ_ITEM / IQ_EXECUTION / OQ_EXECUTION / PQ_EXECUTION / IQ_STEP_EXECUTION / OQ_STEP_EXECUTION / PQ_STEP_EXECUTION / VSR_ASSESSMENT / DEVIATION / DELIVERABLE_REVISION | `IQ_STEP_EXECUTION` |
| 167 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | PK of the table corresponding to target_entity_type. No physical FK is defined because this is a polymorphic reference. | `UUID` |
| 168 | Evidence Type | `evidence_type` | `varchar(50)` | N | N | - | Y | `'TEST_RESULT'` | N | Y | N | Y | TEST_RESULT, SCREENSHOT, LOG, REPORT, APPROVAL_DOCUMENT | `TEST_RESULT` |
| 169 | Evidence Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Contents of the evidence file and purpose of the link | `IQ 수행 결과 화면 캡처` |
| 170 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Evidence link creation timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 171 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who created the evidence link | `UUID` |
| 172 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Evidence link last modification timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 173 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who last modified the evidence link | `UUID` |
| 174 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Evidence link soft-deletion timestamp | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_evidence_link_1` | UNIQUE | `project_id, file_id, target_entity_type, target_entity_id, evidence_type, deleted_at` | UNIQUE (project_id,file_id,target_entity_type,target_entity_id,evidence_type) WHERE deleted_at IS NULL |
| `rule_evidence_link_2` | Business validation | - | Validate the actual PK for the target type, the same project, and the exact revision. Do not represent polymorphic references as physical FKs. |
| `rule_evidence_link_3` | Business validation | - | Link step-specific attachments to the respective IQ/OQ/PQ_STEP_EXECUTION, and attachments for the entire attempt to the respective EXECUTION. Do not overwrite attachments of approved targets. |
| `rule_evidence_link_4` | Business validation | `file_id` | Link only files satisfying file_asset registration conditions. For UPLOAD, verify a no-threats-found scan and identity between the registered file and its original. Do not use a quarantine path or scan job ID in place of file_id. |

#### Business Rules

For the primary VA/FDS/DDS attachment, file_id in the respective table is the authoritative reference. Do not independently edit a duplicate of the same primary attachment here as another original.

[↑ Back to Top](#top)

---

## System

<a id="table-system_asset"></a>
### 15. System/Equipment Identification Information (`system_asset`)

| Item | Definition |
| --- | --- |
| Description | Reference information for systems/equipment subject to validation (management number, GAMP category) |
| Primary Key | `system_id` |
| Key References (FK) | `organization_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 175 | System ID | `system_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique system identifier | `00000000-0000-0000-0000-000000000001` |
| 176 | Organization ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | Client company or operating organization to which the system/equipment belongs | `00000000-0000-0000-0000-000000000001` |
| 177 | Management Number | `management_number` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Asset/equipment management number | `EQ-MES-2024-001` |
| 178 | System Name | `system_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | System/equipment name | `Sample System` |
| 179 | Major Category | `major_category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Computerized system / equipment / facilities and utilities | `컴퓨터화 시스템` |
| 180 | Middle Category | `middle_category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Middle category belonging to the selected major category | `품질 시스템` |
| 181 | Target | `target_type` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Target belonging to the selected major and middle categories | `LIMS` |
| 182 | Responsible Department | `department_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Department responsible for management/operation | `생산기술팀` |
| 183 | Installation Location | `location` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Physical/logical installation location | `서버실 A동 3F` |
| 184 | Vendor | `vendor` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Equipment/system vendor name | `Sample Vendor` |
| 185 | Model Name | `model_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Equipment/system model name | `FillMaster 500` |
| 186 | System Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Description of the system's purpose and operational scope | `생산 공정 데이터 수집 및 제어` |
| 187 | CS Included | `is_cs_included` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether a Computerized System is included | `TRUE` |
| 188 | Software Version | `software_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Software version of the computerized system. Distinct from the inventory revision number. | `4.8.2` |
| 189 | Inventory Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Integer version incremented by the revision function. Separate from the software version. | `2` |
| 190 | GAMP Category | `gamp_category` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Screen selection: CATEGORY_1/CATEGORY_2/CATEGORY_3/CATEGORY_4. NULL if not selected. | `CATEGORY_4` |
| 191 | GxP Applicability | `gxp_applicability` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | APPLICABLE=Applicable, NOT_APPLICABLE=Not Applicable. NULL if not selected. | `APPLICABLE` |
| 192 | Part 11 Applicability | `part11_applicability` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | APPLICABLE=Applicable, NOT_APPLICABLE=Not Applicable. NULL if not selected. | `APPLICABLE` |
| 193 | Approval Status | `approval_status` | `varchar(30)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT=Draft, REVIEW=Under Review, APPROVAL=Pending Approval, APPROVED=Approved, REAPPROVAL_REQUIRED=Reapproval Required, REJECTED=Rejected | `DRAFT` |
| 194 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-25T00:00:00Z` |
| 195 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-25T00:00:00Z` |
| 196 | Use/Disposal Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE=In Use, DISPOSED=Disposed. Separate from approval status. | `ACTIVE` |
| 197 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the disposal electronic signature was completed | `2026-09-01T00:00:00Z` |
| 198 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason entered on the disposal screen | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_system_asset_1` | UNIQUE | `organization_id, management_number` | UNIQUE (organization_id, management_number) |
| `ck_system_asset_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `ck_system_asset_3` | CHECK | `gxp_applicability` | CHECK (gxp_applicability IS NULL OR gxp_applicability IN ('APPLICABLE','NOT_APPLICABLE')) |
| `ck_system_asset_4` | CHECK | - | CHECK (part11_applicability IS NULL OR part11_applicability IN ('APPLICABLE','NOT_APPLICABLE')) |
| `ck_system_asset_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |

#### Business Rules

Validate the screen's required software version, GAMP, GxP, and Part 11 values according to whether the system is computerized. Allow only selectable combinations across the three classification levels.

Retrieve linked projects through validation_project.system_id. Link revision, approval, and disposal history through system_asset_revision and electronic-signature/audit records.

When changing approved information, increment the revision number and indicate reapproval status. Distinguish changes to the software version itself from inventory document revisions.

Handle changes to approved business fields with a new revision_number and reapproval. Changes to current system information must not alter historical system_asset_revision content or the approved revision adopted by a project.

[↑ Back to Top](#top)

---

<a id="table-system_asset_revision"></a>
### 16. System Inventory Revision History (`system_asset_revision`)

| Item | Definition |
| --- | --- |
| Description | Registration, modification, approval, and disposal history by inventory revision number |
| Primary Key | `system_revision_id` |
| Key References (FK) | `system_id, processed_by, signature_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 199 | Inventory History ID | `system_revision_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 200 | System ID | `system_id` | `uuid` | N | Y | `system_asset.system_id` | Y | - | N | Y | N | Y | References system_asset.system_id | `00000000-0000-0000-0000-000000000001` |
| 201 | Revision Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Inventory revision number at the time of the action | `2` |
| 202 | Action Category | `action_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | REGISTER=Register, SAVE=Save, REVISE=Revise, APPROVE=Approve, DISPOSE=Dispose | `REVISE` |
| 203 | Reason for Change/Action | `change_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason required for revision or disposal | - |
| 204 | Approval Status After Action | `approval_status` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | Uses the same codes as system_asset.approval_status | `REVIEW` |
| 205 | Processed By ID | `processed_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 206 | Processed At | `processed_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Date/time on the history screen | `2026-09-01T00:00:00Z` |
| 207 | Action Electronic Signature ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature linked to approval/disposal. Retrieve multiple signatures from the signature history for the same target. | `00000000-0000-0000-0000-000000000001` |
| 208 | Snapshot Format Version | `snapshot_schema_version` | `varchar(50)` | N | N | - | Y | `'ASSET-1'` | N | N | N | Y | Version of the field structure and data types of asset_snapshot. Preserve historical format definitions. | `ASSET-1` |
| 209 | Inventory Snapshot at Action Time | `asset_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Store all system_asset column values after the action, including field names and NULLs. Approval events must match the business content that received final approval. | - |
| 210 | Snapshot Hash | `snapshot_hash` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | SHA-256 in 64 lowercase hexadecimal characters of {schema_version:snapshot_schema_version,body:asset_snapshot}, serialized using the same canonicalization rules as electronic signatures. Distinct from the hash of the entire signed bundle. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `ck_system_asset_revision_1` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `rule_system_asset_revision_2` | Business validation | - | Multiple action records, such as registration/save/approval, may exist for the same system and revision number, so do not impose UNIQUE on those two columns alone. |
| `rule_system_asset_revision_3` | Business validation | - | change_reason must not be empty when action_type is REVISE or DISPOSE. |
| `rule_system_asset_revision_4` | Business validation | - | DISPOSE requires signature_id with signature_action=DISPOSE. The signature target is the system_id of system_asset and the relevant revision_number. |
| `uq_asset_approved_revision` | UNIQUE | `system_id, revision_number, action_type` | UNIQUE (system_id, revision_number) WHERE action_type='APPROVE' |
| `ck_asset_event_kind` | CHECK | `action_type` | CHECK (action_type IN ('REGISTER','SAVE','REVISE','APPROVE','DISPOSE')) |
| `ck_asset_approval_signature` | CHECK | `action_type, approval_status, signature_id` | CHECK (action_type <> 'APPROVE' OR (approval_status='APPROVED' AND signature_id IS NOT NULL)) |

#### Business Rules

Append each action event together with its complete content snapshot at that time. Do not modify or delete event rows or snapshots. Retrieve before/after comparisons from audit_trail and historical approved content directly from asset_snapshot of the APPROVE event. Identify the approval target by system_asset.system_id and its revision_number.

An APPROVE event records final approval completion, not individual review-stage events. The signed system_id/revision number and business fields of the approved content must match. The electronic-signature payload format defines the comparison scope excluding administrative metadata such as approval status and action time. There is one final approved snapshot for a revision; business content changes after approval require a new revision number. Record save, revision, signature, and final approval events in the same transaction as the original data change.

[↑ Back to Top](#top)

---

## Library

<a id="table-library_item"></a>
### 17. Library Item Master (`library_item`)

| Item | Definition |
| --- | --- |
| Description | Master of reusable standard library items for URS, FRA, IQ, OQ, and PQ |
| Primary Key | `library_id` |
| Key References (FK) | - |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 211 | Library ID | `library_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique library item identifier | `00000000-0000-0000-0000-000000000001` |
| 212 | Module Type | `module_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Distinguishes the five library tabs: URS/FRA/IQ/OQ/PQ | `URS` |
| 213 | Code | `code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Item code within a library type. Apply composite UNIQUE (module_type, code). | `URS-AT-L01` |
| 214 | Category/Major Category | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Stores the major category for URS and the category for FRA/IQ/OQ/PQ. | `감사추적` |
| 215 | Item/Function Name | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Function name for URS, risk item name for FRA, and test item name for IQ/OQ/PQ | `데이터 변경 감사추적 자동 생성` |
| 216 | Requirement / Procedure | `requirement_text` | `text` | N | N | - | N | - | N | N | N | Y | URS requirement body. Required for the URS type; NULL allowed for other types. | `모든 데이터 생성·수정·삭제 시...` |
| 217 | Expected Result | `expected_result` | `text` | N | N | - | N | - | N | N | N | Y | Expected result for IQ/OQ/PQ. Required for test libraries. | - |
| 218 | Acceptance Criteria/Scope | `acceptance_criteria` | `text` | N | N | - | N | - | N | N | N | Y | Scope for URS and acceptance criteria for IQ/OQ/PQ. NULL allowed for FRA. | `데이터 변경 시 Audit Trail 자동 생성...` |
| 219 | Regulatory Reference | `regulation` | `text` | N | N | - | N | - | N | N | N | Y | Regulatory reference displayed or entered manually on the screen. Link structured clauses through library_item_regulation; do not edit the same reference independently in both places. | `21 CFR 11.10(e)` |
| 220 | In Use | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether active (TRUE/FALSE) | `TRUE` |
| 221 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-25T00:00:00Z` |
| 222 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-25T00:00:00Z` |
| 223 | Provision Type | `provision_type` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | URS only. BUILTIN=Built-in, CUSTOM=User-defined | `CUSTOM` |
| 224 | Middle Category | `middle_category` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Middle category for URS only | - |
| 225 | Target | `target_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Applicable target for URS only | - |
| 226 | Test Content | `test_content` | `text` | N | N | - | N | - | N | N | N | Y | Test procedure/content for IQ/OQ/PQ only | - |
| 227 | Risk Scenario | `risk_scenario` | `text` | N | N | - | N | - | N | N | N | Y | Risk scenario for FRA only | - |
| 228 | Severity (SEV) | `severity` | `smallint` | N | N | - | N | - | N | N | N | Y | FRA only. 1–5 | - |
| 229 | Occurrence (OCC) | `occurrence` | `smallint` | N | N | - | N | - | N | N | N | Y | FRA only. 1–5 | - |
| 230 | Detectability (DET) | `detectability` | `char(1)` | N | N | - | N | - | N | N | N | Y | FRA only. H/M/L | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_library_item_1` | UNIQUE | `module_type, code` | UNIQUE (module_type, code) |
| `ck_library_item_2` | CHECK | `module_type` | CHECK (module_type IN ('URS','FRA','IQ','OQ','PQ')) |
| `rule_library_item_3` | Business validation | - | URS requires provision type, major category, middle category, target, function name, requirement, and scope. FRA requires category, risk item name, scenario, SEV, OCC, and DET. IQ/OQ/PQ require category, item name, test content, expected result, and acceptance criteria. |
| `rule_library_item_4` | Business validation | `severity` | For FRA, severity/occurrence range from 1 to 5; detectability is H, M, or L. |

#### Business Rules

For URS, manage the major category in category, function name in title, and scope in acceptance_criteria.

Copy the body and input values when applying a template to requirements, risks, or tests. Subsequent library changes must not automatically change items already authored or approved.

[↑ Back to Top](#top)

---

## Validation

<a id="table-validation_project"></a>
### 18. Validation Project (`validation_project`)

| Item | Definition |
| --- | --- |
| Description | Unit and scope of validation activities by system (VP) |
| Primary Key | `project_id` |
| Key References (FK) | `system_id, created_by, updated_by, closure_requested_by, current_closure_request_id, current_system_baseline_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 231 | Project ID | `project_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique project identifier | `00000000-0000-0000-0000-000000000001` |
| 232 | Project Code | `project_code` | `varchar(100)` | N | N | - | Y | - | Y | Y | N | Y | Project identification code | `VP-SYS-008-20260422` |
| 233 | Project Name | `project_name` | `varchar(200)` | N | N | - | Y | - | N | Y | N | Y | Project Name | `테스트 장비3 CSV 프로젝트` |
| 234 | System ID | `system_id` | `uuid` | N | Y | `system_asset.system_id` | Y | - | N | Y | N | Y | References system_asset.system_id | `00000000-0000-0000-0000-000000000001` |
| 235 | Status | `status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS=In Progress, CLOSED_NORMAL=Normally Closed, CLOSED_FORCED=Forcibly Closed | `IN_PROGRESS` |
| 236 | Validation Type | `validation_type` | `varchar(50)` | N | N | - | Y | `'NEW'` | N | N | N | Y | NEW=New, CHANGE=Change, REVALIDATION=Revalidation | `NEW` |
| 237 | Validation Level | `validation_level` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | LEVEL_1/LEVEL_2/LEVEL_3/CUSTOM. CUSTOM=User-defined; performed activities are stored in project_activity. | `LEVEL_2` |
| 238 | Remarks | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | Additional notes | - |
| 239 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-25T00:00:00Z` |
| 240 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the project creator | `00000000-0000-0000-0000-000000000001` |
| 241 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 242 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the user who modified the project | `00000000-0000-0000-0000-000000000001` |
| 243 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |
| 244 | Closure Type | `closure_type` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | Summary of the closure type from the current/final project_closure_request. Synchronize from the original request; do not edit independently. | `FORCED` |
| 245 | Forced Closure Reason | `closure_reason` | `text` | N | N | - | N | - | N | N | N | Y | Summary of the forced closure reason from the current/final project_closure_request. Synchronize from the original request; do not edit independently. | `사업 우선순위 변경으로 Validation 중단` |
| 246 | Closure Requested By ID | `closure_requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Summary of the closure requester ID from the current/final project_closure_request. Synchronize from the original request; do not edit independently. | `00000000-0000-0000-0000-000000000001` |
| 247 | Closure Requested At | `closure_requested_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Summary of the closure request timestamp from the current/final project_closure_request. Synchronize from the original request; do not edit independently. | `2026-09-14T09:00:00Z` |
| 248 | Closed At | `closed_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Summary of the closure completion timestamp from the current/final project_closure_request. Synchronize from the original request; do not edit independently. | `2026-09-14T09:05:00Z` |
| 249 | Current Closure Request ID | `current_closure_request_id` | `uuid` | N | Y | `project_closure_request.closure_request_id` | N | - | N | Y | N | Y | Closure request used to display closure progress and reason | `00000000-0000-0000-0000-000000000001` |
| 250 | Current Validation Target Baseline | `current_system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | Baseline for the same project with is_current=TRUE. Required before the first business submission or test execution. | - |
| 251 | Applied Dependency Set Version | `dependency_set_version` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Version of the dependency rule set adopted when the project was created | `DEPENDENCY-1` |
| 252 | Applied Dependency Snapshot | `dependency_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Preserves all active conditions and their evaluation meanings at creation. {schema_version,version,definitions,conditions:[{activity_dependency_id,successor_activity_id,predecessor_activity_id,dependency_type,required_status,condition_type,condition_value,evaluation_order}]} | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `ck_validation_project_1` | CHECK | `status` | CHECK (status IN ('IN_PROGRESS','CLOSED_NORMAL','CLOSED_FORCED')) |
| `ck_validation_project_2` | CHECK | `validation_type` | CHECK (validation_type IN ('NEW','CHANGE','REVALIDATION')) |
| `ck_validation_project_3` | CHECK | `validation_level` | CHECK (validation_level IN ('LEVEL_1','LEVEL_2','LEVEL_3','CUSTOM')) |
| `rule_project_current_refs` | Business validation | `current_closure_request_id, current_system_baseline_id` | Both the current closure request and validation target baseline must belong to this project. If a baseline exists, the current baseline reference must match the unique row with is_current=TRUE. |

#### Business Rules

Manage the screen's project name, target system, validation type, level, and notes in this table. Calculate progress from completion of the selected activities, excluding RTM from the denominator.

Retrieve the project's validation-target GAMP, GxP, and software version from the approved content referenced by current_system_baseline_id. Display the latest system information separately from the validation target baseline.

Store closure-request review/approval progress in project_closure_request and workflow_instance. Change the project to normal or forced closure only after final approval.

Fix dependency_set_version and dependency_snapshot together when creating the project. Do not automatically propagate global activity_dependency changes to in-progress or closed projects. Condition definitions include completion evaluation by status, AND/OR combination, handling of unselected activities, and risk coverage criteria. Use this snapshot for project evaluation and do not directly replace it after creation. Specify a valid current_system_baseline_id before business approval or test execution. Register the initial baseline and set the current reference in one transaction.

[↑ Back to Top](#top)

---

<a id="table-project_system_baseline"></a>
### 19. Project Validation Target Revision (`project_system_baseline`)

| Item | Definition |
| --- | --- |
| Description | Approved inventory revision adopted as the project's validation target and its change history |
| Primary Key | `baseline_id` |
| Key References (FK) | `project_id, system_revision_id, adopted_by` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the auditable period. Do not delete approved content or historical link records. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 253 | Validation Target Baseline ID | `baseline_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Identifier of one adoption of a validation target baseline by a project | - |
| 254 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Project to which the validation target baseline belongs | - |
| 255 | Inventory Approval History ID | `system_revision_id` | `uuid` | N | Y | `system_asset_revision.system_revision_id` | Y | - | N | Y | N | Y | Immutable original-content event with action_type=APPROVE | - |
| 256 | Baseline Number | `baseline_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Increments from 1 within each project. Create a new row when changing the baseline. | `1` |
| 257 | Adoption/Change Reason | `change_reason` | `text` | N | N | - | Y | - | N | N | N | Y | Reason for initial target selection or a change to the approved revision. Must not be blank. | - |
| 258 | Currently Applied | `is_current` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the current project baseline. Set historical rows to FALSE while retaining their original-content links. | - |
| 259 | Adopted At | `adopted_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Application timestamp recorded by the server | - |
| 260 | Adopted By | `adopted_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Acting user authorized to configure the project target | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_project_baseline_number` | UNIQUE | `project_id, baseline_number` | UNIQUE (project_id, baseline_number) |
| `uq_project_baseline_current` | UNIQUE | `project_id, is_current` | UNIQUE (project_id) WHERE is_current=TRUE |
| `ck_project_baseline_number` | CHECK | `baseline_number` | CHECK (baseline_number >= 1) |
| `ck_project_baseline_reason` | CHECK | `change_reason` | CHECK (length(trim(change_reason)) > 0) |
| `rule_project_baseline_asset` | Business validation | `project_id, system_revision_id` | The approval event's system_id must match the project's system_id. At adoption, verify that the system is in use and that the revision is a currently valid approved version. The mere existence of a historical approval event for a previous baseline does not justify adopting it as the latest baseline. |

#### Business Rules

Do not overwrite the adopted system_revision_id, sequence number, reason, actor, or timestamp. When changing the baseline, lock the project, set the previous is_current to FALSE, and save the new row and current reference in the same transaction. Do not change the baseline of a closed project.

A change to the target revision updates the rereview and reapproval status of affected activities/documents while preserving baseline links of already approved records. Historical test executions and documents retrieve the software version, classification, and GxP applicability at that time through their own baseline_id.

[↑ Back to Top](#top)

---

<a id="table-project_member"></a>
### 20. Project Member (`project_member`)

| Item | Definition |
| --- | --- |
| Description | Manages participating users and their roles for each project |
| Primary Key | `project_member_id` |
| Key References (FK) | `project_id, user_id, role_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 261 | Project Member ID | `project_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique project-member role mapping identifier. Do not allow duplicate (project_id, user_id, role_id) values where deleted_at IS NULL and member_status is ACTIVE. | `UUID` |
| 262 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | ID of the Validation project to which the member belongs | `UUID` |
| 263 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user participating in the project | `UUID` |
| 264 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | ID of the role performed by the user within the project | `UUID` |
| 265 | Participation Status | `member_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | Y | N | Y | Project participation status: ACTIVE, INACTIVE, WITHDRAWN | `ACTIVE` |
| 266 | Joined At | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time project participation started | `2026-09-01T10:00:00Z` |
| 267 | Left At | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time project participation ended. NULL while participation is current. | `2026-09-01T00:00:00Z` |
| 268 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project participation record creation timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 269 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who registered the project participation record | `UUID` |
| 270 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project participation record last modification timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 271 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who last modified the project participation record | `UUID` |
| 272 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Project participation record soft-deletion timestamp | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_project_member_1` | UNIQUE | `project_id, user_id, role_id, deleted_at, member_status` | UNIQUE (project_id, user_id, role_id) WHERE deleted_at IS NULL AND member_status='ACTIVE' |

#### Business Rules

Manage members and roles per project. Manage the screen's view/edit/disposal permissions separately in access_permission_grant.

[↑ Back to Top](#top)

---

<a id="table-validation_activity"></a>
### 21. Validation Activity Master (`validation_activity`)

| Item | Definition |
| --- | --- |
| Description | Manages reference data for VP, VA, QIA, URS, FDS_GROUP, FRA, DQ, IQ, OQ, PQ, and VSR activities |
| Primary Key | `activity_id` |
| Key References (FK) | `created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 273 | Activity ID | `activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique validation activity identifier | `UUID` |
| 274 | Activity Code | `activity_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | VP, VA, QIA, URS, FDS_GROUP, FRA, DQ, IQ, OQ, PQ, VSR. FDS_GROUP corresponds to the screen display name F&DS. | `FDS_GROUP` |
| 275 | Activity Name | `activity_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Activity name for screen display | `사용자 요구사항 명세` |
| 276 | Activity Order | `display_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Default display order: VP→VA→QIA→URS→F&DS→FRA→DQ→IQ→OQ→PQ→VSR. Within F&DS, FDS precedes DDS. | `5` |
| 277 | In Use | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the activity master record is in use | `TRUE` |
| 278 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Activity master creation timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 279 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the activity master record | `UUID` |
| 280 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Activity master last modification timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 281 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the activity master record | `UUID` |
| 282 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Activity master soft-deletion timestamp | `2026-09-01T00:00:00Z` |

#### Business Rules

RTM is a dashboard function for viewing traceability among requirements, designs, risks, and tests. Manage system inventory as prerequisite reference information for projects.

FDS and DDS are document types within one F&DS activity. Do not duplicate them as separate performed stages.

[↑ Back to Top](#top)

---

<a id="table-project_activity"></a>
### 22. Project Activity (`project_activity`)

| Item | Definition |
| --- | --- |
| Description | Manages selected activities, activation, and progress status for each project |
| Primary Key | `project_activity_id` |
| Key References (FK) | `project_id, activity_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 283 | Project Activity ID | `project_activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique project activity identifier. Do not allow duplicate (project_id, activity_id) values where deleted_at IS NULL, regardless of whether the activity is selected for execution. | `UUID` |
| 284 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Validation project to which the activity belongs | `UUID` |
| 285 | Activity ID | `activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | Activity to perform in the project | `UUID` |
| 286 | Selected for Execution | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Whether this is an actual activity selected in project settings. Do not create an RTM selection row. | `TRUE` |
| 287 | Mandatory Activity | `is_required` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Whether this activity cannot be omitted from the project | `TRUE` |
| 288 | Activity Status | `activity_status` | `varchar(20)` | N | N | - | Y | `'LOCKED'` | N | Y | N | Y | Activity status: LOCKED, READY, IN_PROGRESS, COMPLETED, APPROVED, SKIPPED. Aggregate based on the current revision and reevaluate when deliverables change or are revised. Do not set COMPLETED/APPROVED through simple screen input. | `READY` |
| 289 | Activated At | `activated_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time the activity became READY after prerequisites were satisfied | `2026-09-01T10:00:00Z` |
| 290 | Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time activity execution started | `2026-09-01T11:00:00Z` |
| 291 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time activity execution completed | `2026-09-02T15:00:00Z` |
| 292 | Approved At | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time final approval of the activity completed | `2026-09-02T17:00:00Z` |
| 293 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project activity creation timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 294 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the project activity | `UUID` |
| 295 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project activity last modification timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 296 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the project activity | `UUID` |
| 297 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Project activity soft-deletion timestamp | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_project_activity_1` | UNIQUE | `project_id, activity_id, deleted_at` | UNIQUE (project_id, activity_id) WHERE deleted_at IS NULL |

#### Business Rules

Progress, completed activity counts, and normal closure conditions apply to the selected actual activities. Do not require separate RTM approval or deliverables. Aggregate FDS/DDS progress into one F&DS activity.

[↑ Back to Top](#top)

---

<a id="table-activity_dependency"></a>
### 23. Activity Dependency (`activity_dependency`)

| Item | Definition |
| --- | --- |
| Description | Manages predecessor activities, relationship types, and evaluation conditions for activating successor activities |
| Primary Key | `activity_dependency_id` |
| Key References (FK) | `successor_activity_id, predecessor_activity_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 298 | Activity Dependency ID | `activity_dependency_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique activity dependency identifier | `UUID` |
| 299 | Successor Activity ID | `successor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | Activity activated after conditions are satisfied | `UUID` |
| 300 | Predecessor Activity ID | `predecessor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | N | - | N | Y | N | Y | Activity to check before activating the successor. NULL allowed for conditions covering all activities. Required for STATUS conditions. | `UUID` |
| 301 | Relationship Type | `dependency_type` | `varchar(20)` | N | N | - | Y | `'REQUIRED'` | N | Y | N | Y | Dependency relationship type: REQUIRED, RECOMMENDED | `REQUIRED` |
| 302 | Required Status | `required_status` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | Evaluation criterion for STATUS conditions. CREATED means a valid current deliverable exists; COMPLETED means execution is complete; APPROVED means final approval of the current deliverable. Do not store CREATED in project_activity.activity_status. Required only for STATUS conditions; detailed evaluation follows the business rules below. | `APPROVED` |
| 303 | Condition Type | `condition_type` | `varchar(50)` | N | N | - | Y | `'STATUS'` | N | Y | N | Y | STATUS, ACTIVITY_SELECTED, TRACEABILITY_EXISTS, HIGH_RISK_COVERED, OPEN_DEVIATION_ZERO, ALL_SELECTED_APPROVED. Categories for evaluating the screen's prerequisite stages and completion conditions. | `STATUS` |
| 304 | Condition Value | `condition_value` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Condition evaluation parameter. For STATUS + CREATED, store an allowed target deliverable table name, such as requirement. Validate parameters for other conditions in their respective implementations; use NULL when unnecessary. | - |
| 305 | Condition Description | `condition_description` | `text` | N | N | - | Y | - | N | N | N | Y | Description of the predecessor activity and approval status required to activate the activity | `URS 승인완료` |
| 306 | Evaluation Order | `evaluation_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Order of condition evaluation for the same successor activity. Evaluation order does not change the AND/OR relationships between conditions. | `1` |
| 307 | In Use | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the activation condition is in use | `TRUE` |
| 308 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Condition creation timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 309 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the condition | `UUID` |
| 310 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Condition last modification timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 311 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the condition | `UUID` |
| 312 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Condition soft-deletion timestamp | `2026-09-01T00:00:00Z` |

#### Business Rules

Conditions requiring approval of all activities evaluate only the selected actual activities. Do not register RTM as a predecessor/successor activity or an independent approval condition.

STATUS conditions require predecessor_activity_id and required_status, with required_status being CREATED/COMPLETED/APPROVED. For other conditions, leave unnecessary predecessor activity/status values NULL. Do not allow cycles in active dependency relationships.

This table is a global condition template for new projects. On change, issue a new condition-set version and preserve the complete conditions and evaluation meanings as immutable version-specific configuration. At project creation, copy the selected version's complete content into validation_project.dependency_snapshot. Subsequent changes, deactivation, or deletion of conditions do not change existing projects' conditions.

[↑ Back to Top](#top)

---

<a id="table-project_closure_request"></a>
### 24. Project Closure Request (`project_closure_request`)

| Item | Definition |
| --- | --- |
| Description | Normal/forced closure reasons and review/approval progress per request |
| Primary Key | `closure_request_id` |
| Key References (FK) | `project_id, requested_by, workflow_instance_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 313 | Closure Request ID | `closure_request_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 314 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 315 | Request Version | `request_version` | `integer` | N | N | - | Y | - | N | N | N | Y | Distinguishes repeated closure requests for the same project | `1` |
| 316 | Closure Type | `closure_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | NORMAL=Normal Closure, FORCED=Forced Closure | `NORMAL` |
| 317 | Request Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT=Draft, REVIEW=Under Review, APPROVAL=Pending Approval, APPROVED=Completed, REJECTED=Rejected, CANCELLED=Cancelled | `REVIEW` |
| 318 | Closure Reason | `reason` | `text` | N | N | - | N | - | N | N | N | Y | Required for forced closure | - |
| 319 | Progress at Request Time | `progress_snapshot` | `numeric(5,2)` | N | N | - | Y | - | N | N | N | Y | Progress at the time of the request as displayed on the closure confirmation screen. Excludes RTM. | `100.00` |
| 320 | Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 321 | Requested At | `requested_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Request timestamp value | `2026-09-01T00:00:00Z` |
| 322 | Closure Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |
| 323 | Closure Processed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Closure processing timestamp value | `2026-09-01T00:00:00Z` |
| 324 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation timestamp value | `2026-09-01T00:00:00Z` |
| 325 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 326 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Modification timestamp value | `2026-09-01T00:00:00Z` |
| 327 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_project_closure_request_1` | UNIQUE | `project_id, request_version` | UNIQUE (project_id, request_version) |
| `ck_project_closure_request_2` | CHECK | `request_version` | CHECK (request_version >= 1) |
| `ck_project_closure_request_3` | CHECK | `progress_snapshot` | CHECK (progress_snapshot BETWEEN 0 AND 100) |
| `rule_project_closure_request_4` | Business validation | `closure_type` | reason must not be empty when closure_type='FORCED'. |
| `rule_project_closure_request_5` | Business validation | - | There may be at most one closure request in DRAFT/REVIEW/APPROVAL status for the same project. |
| `uq_project_closure_active` | UNIQUE | `project_id` | UNIQUE (project_id) WHERE status IN ('DRAFT','REVIEW','APPROVAL') |

#### Business Rules

Normal closure checks completion and approval of the selected activities and required deliverables. Exclude independent RTM approval and RTM deliverables from closure conditions.

Link signers through electronic signatures and approval action history. The workflow_instance target is project_closure_request/closure_request_id/CLOSE-{request_version}, and the project must match. After final approval, update validation_project.status and the closure summary together. Do not treat review-in-progress or approval-in-progress as project closure.

[↑ Back to Top](#top)

---

## VP

<a id="table-vp_plan"></a>
### 25. Validation Plan (`vp_plan`)

| Item | Definition |
| --- | --- |
| Description | Manages VP section/body editing and revisions for business approval. |
| Primary Key | `vp_id` |
| Key References (FK) | `project_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 328 | VP Revision ID | `vp_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 329 | Logical ID Across Revisions | `vp_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision-row PK. | `00000000-0000-0000-0000-000000000001` |
| 330 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 331 | Plan Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Display title of the project's validation plan | `시스템 검증 계획` |
| 332 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Screen display version of this revision | `ver1` |
| 333 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 for each logical key | `1` |
| 334 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising | `요구사항 변경` |
| 335 | Latest Revision Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Latest revision for the same logical key. Distinct from the latest approved version. | `TRUE` |
| 336 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft) / REVIEW (Under Review) / APPROVAL (Pending Approval) / APPROVED (Approved) / REJECTED (Rejected) | `DRAFT` |
| 337 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow for this revision. Retrieve reviewers, approvers, and signatures from the linked workflow/action history. | `00000000-0000-0000-0000-000000000001` |
| 338 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 339 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 340 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 341 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 342 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from approval status. | `ACTIVE` |
| 343 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time disposal approval completed | `2026-09-01T00:00:00Z` |
| 344 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 345 | Disposal Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_vp_plan_1` | UNIQUE | `vp_key, revision_number` | UNIQUE (vp_key, revision_number) |
| `uq_vp_plan_1_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_vp_plan_2` | UNIQUE | `vp_key, is_current_version` | UNIQUE (vp_key) WHERE is_current_version = TRUE |
| `ck_vp_plan_3` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_vp_plan_4` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_vp_plan_4_rule` | Business validation | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED requires disposed_at, disposal_reason, and disposal_workflow_id. |
| `rule_vp_plan_5` | Business validation | `workflow_instance_id` | There is one VP logical key per project, with multiple revisions. APPROVED requires workflow_instance_id. |

#### Business Rules

Each row represents a specific revision. Preserve the previously approved row, issue a new PK for the revision, and retain the logical key. is_current_version indicates the latest revision, not the latest approved version. Referencing FKs and electronic signatures point to the exact revision-row PK and version.

Store section additions, deletions, ordering, inclusion, and content in vp_section. Manage edited content and document approval status of generated deliverables in deliverable_revision, separately from the original VP.

All revisions of the same vp_key belong to the same project. Lock the project and logical item when creating a revision or switching the latest revision, and process both within the same transaction.

[↑ Back to Top](#top)

---

<a id="table-vp_section"></a>
### 26. Validation Plan Section (`vp_section`)

| Item | Definition |
| --- | --- |
| Description | Section titles, content, display order, and inclusion for each VP revision. |
| Primary Key | `vp_section_id` |
| Key References (FK) | `vp_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 346 | Section ID | `vp_section_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 347 | VP Revision ID | `vp_id` | `uuid` | N | Y | `vp_plan.vp_id` | Y | - | N | Y | N | Y | References vp_plan.vp_id | `00000000-0000-0000-0000-000000000001` |
| 348 | Section Logical Key | `section_key` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Section identifier retained across revisions | `purpose` |
| 349 | Section Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Section Title | `목적` |
| 350 | Section Content | `content` | `text` | N | N | - | Y | - | N | N | N | Y | Section Content | `본 계획의 검증 목적` |
| 351 | Display Order | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display Order | `1` |
| 352 | Included in Deliverable | `is_enabled` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Included in Deliverable | `TRUE` |
| 353 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 354 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 355 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 356 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_vp_section_1` | UNIQUE | `vp_id, section_key` | UNIQUE (vp_id, section_key) |
| `uq_vp_section_1_2` | UNIQUE | `vp_id, sort_order` | UNIQUE (vp_id, sort_order) |
| `uq_vp_section_1_3` | CHECK | `sort_order` | CHECK (sort_order >= 1) |

#### Business Rules

Do not directly modify sections of an approved VP; copy them into a new VP revision. Content saved by the user after AI draft generation is also recorded in content.

[↑ Back to Top](#top)

---

## QIA

<a id="table-qia_assessment"></a>
### 27. Quality Impact Assessment Header (`qia_assessment`)

| Item | Definition |
| --- | --- |
| Description | Assessment header managing project-specific QIA modules and shared Part 11 question responses. |
| Primary Key | `qia_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 357 | QIA ID | `qia_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique QIA assessment identifier | `00000000-0000-0000-0000-000000000001` |
| 358 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | Y | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 359 | Part 11 Q1: Electronic Records Replacing Paper Records | `p11_q1` | `varchar(3)` | N | N | - | Y | `'No'` | N | N | N | Y | Do electronic records replace paper records? Response: Yes / No | `No` |
| 360 | Part 11 Q2: Use of Electronic Signatures | `p11_q2` | `varchar(3)` | N | N | - | Y | `'No'` | N | N | N | Y | Are electronic signatures used? Response: Yes / No | `No` |
| 361 | Part 11 Q3: Record History as Regulatory Evidence | `p11_q3` | `varchar(3)` | N | N | - | Y | `'No'` | N | N | N | Y | Does the history of record creation and changes serve as regulatory evidence? Response: Yes / No | `No` |
| 362 | Part 11 Q4: Need for Access Control and User Identification | `p11_q4` | `varchar(3)` | N | N | - | Y | `'No'` | N | N | N | Y | Are access control and user identification required? Response: Yes / No | `No` |
| 363 | Part 11 Q5: Need for Long-Term Retention and Retrieval | `p11_q5` | `varchar(3)` | N | N | - | Y | `'No'` | N | N | N | Y | Are long-term retention and retrieval of records required? Response: Yes / No | `No` |
| 364 | Part 11 Q6: Transfer of Electronic Records Between Systems | `p11_q6` | `varchar(3)` | N | N | - | Y | `'No'` | N | N | N | Y | Are electronic records transferred between systems? Response: Yes / No | `No` |
| 365 | Part 11 Assessment Conclusion | `part11_result` | `text` | N | N | - | N | - | N | N | N | N | Display value calculated automatically from the six responses. Applicable if any response is Yes; not applicable if all are No. Not entered directly. | `Part 11 비적용` |
| 366 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 367 | Question Set Version | `question_set_version` | `varchar(50)` | N | N | - | Y | `'UI-P11-1'` | N | N | N | Y | Version fixing the wording and order of the six questions | `UI-P11-1` |
| 368 | Evaluation Rule Version | `rule_version` | `varchar(50)` | N | N | - | Y | `'UI-P11-1'` | N | N | N | Y | Version of the rule determining applicability when at least one response is Yes | `UI-P11-1` |
| 369 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 370 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 371 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 372 | Applied Question Set Snapshot | `question_set_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,questions:[{question_id,field_name,text,sort_order,answer_options}]}. Stores all questions, wording, ordering, and answer options for the selected version. | - |
| 373 | Applied Evaluation Rule Snapshot | `rule_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,input_fields,normalization,expression,result_mapping}. Complete definition including the logical expression, response interpretation, and result-value mapping. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_qia_assessment_1` | UNIQUE | `project_id` | UNIQUE (project_id) |
| `rule_qia_assessment_2` | Business validation | `p11_q1` | p11_q1 through p11_q6 must be 'Yes' or 'No'. part11_result is a cache recalculated in the same transaction as the responses; independent modification is prohibited. |

#### Business Rules

Part 11 responses are shared across QIA, rather than per module. When generating/approving a QIA document, fix the question version, responses, rule version, and result in deliverable_revision's source/table snapshots. Changes to current responses do not change previously approved documents.

Manage document approval status and versions in deliverable_document and deliverable_revision. Retrieve module-level GxP determinations from child processes.

When authoring an assessment, copy the version-specific question/rule definitions and validate on the server that their version values match question_set_version and rule_version. Do not register different definitions under the same version name. Do not automatically apply a new global version to an existing assessment. Calculate using the stored definitions; a function name or version string alone must not replace the original definition.

Changes to shared Part 11 responses or applied definitions must not change evaluation_definition_snapshot in documents already submitted or approved. When submitting a QIA document, fix the complete questions, rules, responses, and determinations for the shared assessment and all included processes, together with the determination policy for modules without processes, in evaluation_definition_snapshot.

[↑ Back to Top](#top)

---

<a id="table-qia_module_item"></a>
### 28. QIA Module Detailed Assessment (`qia_module_item`)

| Item | Definition |
| --- | --- |
| Description | Manages QIA module names/descriptions and approval, disposal, and revisions per module. |
| Primary Key | `qia_module_item_id` |
| Key References (FK) | `qia_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 374 | QIA Module Item ID | `qia_module_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique QIA module assessment item identifier | `UUID` |
| 375 | Logical ID Across Revisions | `module_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision-row PK. | `00000000-0000-0000-0000-000000000001` |
| 376 | QIA ID | `qia_id` | `uuid` | N | Y | `qia_assessment.qia_id` | Y | - | N | Y | N | Y | Parent QIA assessment identifier | `UUID` |
| 377 | Module Code | `module_code` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | Optional internal module code. Distinct from the screen approval number item_number. | `MOD-001` |
| 378 | Module Name | `module_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Parent module name | `품질관리` |
| 379 | Module Description | `module_description` | `text` | N | N | - | N | - | N | N | N | Y | Description of the module's scope and purpose | `품질관리 관련 종합 평가` |
| 380 | Assessment Result | `result_type` | `varchar(20)` | N | N | - | N | - | N | N | N | N | Cache aggregating process determinations. GXP if any process is GXP; otherwise NON_GXP. Not edited directly. | `GXP` |
| 381 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T10:00:00Z` |
| 382 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T10:00:00Z` |
| 383 | Module Display Order | `sort_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Module Display Order | `1` |
| 384 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Screen display version of this revision | `ver1` |
| 385 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 for each logical key | `1` |
| 386 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising | `요구사항 변경` |
| 387 | Latest Revision Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Latest revision for the same logical key. Distinct from the latest approved version. | `TRUE` |
| 388 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft) / REVIEW (Under Review) / APPROVAL (Pending Approval) / APPROVED (Approved) / REJECTED (Rejected) | `DRAFT` |
| 389 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow for this revision. Retrieve reviewers, approvers, and signatures from the linked workflow/action history. | `00000000-0000-0000-0000-000000000001` |
| 390 | Item Approval Number | `item_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Assigned at initial approval. NULL for drafts; retained across revisions. | `QIA-001` |
| 391 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 392 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 393 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from approval status. | `ACTIVE` |
| 394 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time disposal approval completed | `2026-09-01T00:00:00Z` |
| 395 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 396 | Disposal Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_qia_module_item_1` | Business validation | `workflow_instance_id` | APPROVED requires item_number and workflow_instance_id. Do not assign the same approval number to different logical keys within the same project. |
| `uq_qia_module_item_2` | UNIQUE | `module_key, revision_number` | UNIQUE (module_key, revision_number) |
| `uq_qia_module_item_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_qia_module_item_3` | UNIQUE | `module_key, is_current_version` | UNIQUE (module_key) WHERE is_current_version = TRUE |
| `ck_qia_module_item_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_qia_module_item_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_qia_module_item_5_rule` | Business validation | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED requires disposed_at, disposal_reason, and disposal_workflow_id. |
| `ck_qia_module_item_6` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `ck_qia_module_item_6_rule` | Business validation | `sort_order` | A module may be registered before processes are added, so drafts with zero child processes are allowed. |

#### Business Rules

Each row represents a specific revision. Preserve the previously approved row, issue a new PK for the revision, and retain the logical key. is_current_version indicates the latest revision, not the latest approved version. Referencing FKs and electronic signatures point to the exact revision-row PK and version.

A module without processes is classified as NON_GXP; the existence of processes is not an approval restriction. A NON_GXP display alone does not mean that actual process assessment has been completed.

Modules and processes have a 1:N relationship. Store process names and responses to the ten questions in qia_process. On revision, clone processes under the new module revision as well. Process content and responses belonging to an approved module are immutable.

All revisions of the same module_key belong to the same project and business type. Retain item_number in subsequent revisions after initial assignment. For items numbered upon approval, unapproved drafts may have NULL. Do not reuse that number for another logical key within the same project/business type or reassign it to another item after disposal. Serialize number issuance and revision creation using concurrency locks on the project and logical item, and validate duplication, affiliation, and number retention within the same transaction.

[↑ Back to Top](#top)

---

<a id="table-qia_process"></a>
### 29. QIA Process Assessment (`qia_process`)

| Item | Definition |
| --- | --- |
| Description | Process names/descriptions and responses to ten GxP questions under a module revision. |
| Primary Key | `qia_process_id` |
| Key References (FK) | `qia_module_item_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 397 | Process ID | `qia_process_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 398 | Module Revision ID | `qia_module_item_id` | `uuid` | N | Y | `qia_module_item.qia_module_item_id` | Y | - | N | Y | N | Y | References qia_module_item.qia_module_item_id | `00000000-0000-0000-0000-000000000001` |
| 399 | Process Logical ID | `process_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | N | N | Y | Process identifier retained across module revisions | `00000000-0000-0000-0000-000000000001` |
| 400 | Internal Process Code | `process_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Internal Process Code | - |
| 401 | Process Name | `process_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Process Name | `시험 결과 입력` |
| 402 | Process Description | `process_description` | `text` | N | N | - | N | - | N | N | N | Y | Process Description | - |
| 403 | Display Order | `sort_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Display Order | `1` |
| 404 | Question Set Version | `question_set_version` | `varchar(50)` | N | N | - | Y | `'UI-GXP-1'` | N | N | N | Y | Question Set Version | `UI-GXP-1` |
| 405 | Evaluation Rule Version | `rule_version` | `varchar(50)` | N | N | - | Y | `'UI-GXP-1'` | N | N | N | Y | Evaluation Rule Version | `UI-GXP-1` |
| 406 | GxP Q1 | `q1_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | Does it directly affect product quality, patient safety, or data integrity? Response: O / X / ▲ | `X` |
| 407 | GxP Q2 | `q2_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | Does it generate or process data used for GxP decisions? Response: O / X / ▲ | `X` |
| 408 | GxP Q3 | `q3_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | Does it support batch release or quality approval procedures? Response: O / X / ▲ | `X` |
| 409 | GxP Q4 | `q4_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | Is it used for regulatory submissions or inspection evidence? Response: O / X / ▲ | `X` |
| 410 | GxP Q5 | `q5_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | Does it create, modify, or retain electronic records? Response: O / X / ▲ | `X` |
| 411 | GxP Q6 | `q6_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | Does it control critical equipment or process parameters? Response: O / X / ▲ | `X` |
| 412 | GxP Q7 | `q7_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | Does it support deviation, CAPA, or change management processes? Response: O / X / ▲ | `X` |
| 413 | GxP Q8 | `q8_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | Do user permissions control access to quality-related functions? Response: O / X / ▲ | `X` |
| 414 | GxP Q9 | `q9_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | Does it exchange critical data with external GxP systems? Response: O / X / ▲ | `X` |
| 415 | GxP Q10 | `q10_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | Would backup/recovery failure affect GxP records? Response: O / X / ▲ | `X` |
| 416 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 417 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 418 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 419 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 420 | Applied Question Set Snapshot | `question_set_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,questions:[{question_id,field_name,text,sort_order,answer_options}]}. Stores all questions, wording, ordering, and answer options for the selected version. | - |
| 421 | Applied Evaluation Rule Snapshot | `rule_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,input_fields,normalization,expression,result_mapping}. Complete definition including the logical expression, response interpretation, and result-value mapping. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_qia_process_1` | UNIQUE | `qia_module_item_id, process_key` | UNIQUE (qia_module_item_id, process_key) |
| `uq_qia_process_1_2` | UNIQUE | `qia_module_item_id, sort_order` | UNIQUE (qia_module_item_id, sort_order) |
| `uq_qia_process_1_3` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_qia_process_2` | Business validation | `q1_val` | q1_val through q10_val allow only O/X/▲. |

#### Business Rules

GxP determination rule: GXP when Q1=O and at least one of Q2 through Q10 is O; otherwise NON_GXP. ▲ does not satisfy the O condition. Calculate the determination from responses when queried.

Do not directly overwrite question versions or responses belonging to an approved module.

When authoring an assessment, copy the version-specific question/rule definitions and validate on the server that their version values match question_set_version and rule_version. Do not register different definitions under the same version name. Do not automatically apply a new global version to an existing assessment. Calculate using the stored definitions; a function name or version string alone must not replace the original definition.

To change questions, rules, or responses in an approved module, create a new module revision and copy its processes and applied definitions. Preserve the original content belonging to the historical module.

[↑ Back to Top](#top)

---

## VA

<a id="table-vendor_audit"></a>
### 30. Vendor Audit Assessment (`vendor_audit`)

| Item | Definition |
| --- | --- |
| Description | Manages vendor name, audit date, auditor, attachments, and approval/revisions per assessment. |
| Primary Key | `audit_id` |
| Key References (FK) | `project_id, auditor_user_id, file_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 422 | Audit ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique vendor audit identifier | `00000000-0000-0000-0000-000000000001` |
| 423 | Logical ID Across Revisions | `audit_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision-row PK. | `00000000-0000-0000-0000-000000000001` |
| 424 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 425 | Document Number | `document_number` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | Assigned at initial approval. NULL for drafts; retained across revisions of the same logical key. | `VA-001` |
| 426 | Vendor Name | `vendor_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Name of the vendor being audited | `Sample Vendor` |
| 427 | Audit Date | `audit_date` | `date` | N | N | - | Y | `CURRENT_DATE` | N | N | N | Y | Audit date automatically recorded by the screen as the current date when saving | `2026-09-16` |
| 428 | Auditor | `auditor_name` | `varchar(100)` | N | N | - | Y | - | N | N | Y | Y | Display name of the logged-in auditor at saving. Preserve history even if the user name changes. | `홍길동 · reviewer@example.test` |
| 429 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Screen display version of this revision | `ver1` |
| 430 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 for each logical key | `1` |
| 431 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising | `요구사항 변경` |
| 432 | Latest Revision Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Latest revision for the same logical key. Distinct from the latest approved version. | `TRUE` |
| 433 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft) / REVIEW (Under Review) / APPROVAL (Pending Approval) / APPROVED (Approved) / REJECTED (Rejected) | `DRAFT` |
| 434 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 435 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 436 | Auditor Account ID | `auditor_user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Account logged in at the time of saving | `00000000-0000-0000-0000-000000000001` |
| 437 | Assessment Attachment File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | Optional uploaded attachment. Retrieve its name, size, and path from the file asset. | `00000000-0000-0000-0000-000000000001` |
| 438 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow for this revision. Retrieve reviewers, approvers, and signatures from the linked workflow/action history. | `00000000-0000-0000-0000-000000000001` |
| 439 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 440 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 441 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from approval status. | `ACTIVE` |
| 442 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time disposal approval completed | `2026-09-01T00:00:00Z` |
| 443 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 444 | Disposal Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_vendor_audit_1` | Business validation | `workflow_instance_id` | APPROVED requires document_number and workflow_instance_id. Do not assign the same approval number to different logical keys within the same project. |
| `uq_vendor_audit_2` | UNIQUE | `audit_key, revision_number` | UNIQUE (audit_key, revision_number) |
| `uq_vendor_audit_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_vendor_audit_3` | UNIQUE | `audit_key, is_current_version` | UNIQUE (audit_key) WHERE is_current_version = TRUE |
| `ck_vendor_audit_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_vendor_audit_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_vendor_audit_5_rule` | Business validation | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED requires disposed_at, disposal_reason, and disposal_workflow_id. |

#### Business Rules

Each row represents a specific revision. Preserve the previously approved row, issue a new PK for the revision, and retain the logical key. is_current_version indicates the latest revision, not the latest approved version. Referencing FKs and electronic signatures point to the exact revision-row PK and version.

Retrieve the system name from the system linked to the project. Attachments are optional and may be NULL.

All revisions of the same audit_key belong to the same project and business type. Retain document_number in subsequent revisions after initial assignment. For items numbered upon approval, unapproved drafts may have NULL. Do not reuse that number for another logical key within the same project/business type or reassign it to another item after disposal. Serialize number issuance and revision creation using concurrency locks on the project and logical item, and validate duplication, affiliation, and number retention within the same transaction.

[↑ Back to Top](#top)

---

## URS

<a id="table-requirement"></a>
### 31. User Requirements Specification (`requirement`)

| Item | Definition |
| --- | --- |
| Description | Manages the screen's requirement categories, item names, content, acceptance criteria, regulatory references, registration sources, and approved revisions. |
| Primary Key | `requirement_id` |
| Key References (FK) | `project_id, created_by, workflow_instance_id, updated_by, disposal_workflow_id, source_library_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 445 | Requirement ID | `requirement_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique requirement identifier | `00000000-0000-0000-0000-000000000001` |
| 446 | Logical ID Across Revisions | `requirement_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision-row PK. | `00000000-0000-0000-0000-000000000001` |
| 447 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 448 | Item Number | `item_number` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Assigned at initial approval. NULL for drafts; retained across revisions of the same logical key. | `URS-001` |
| 449 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | System management, audit trail, electronic signatures, etc. | `전자서명` |
| 450 | Item / Function | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Requirement item name and main function | `전자서명 서명자·일시·의미 기록` |
| 451 | Requirement Details | `requirement_text` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed requirement specification content | `전자서명 시 서명자 ID, 서명 일시...` |
| 452 | Acceptance Criteria | `acceptance_criteria` | `text` | N | N | - | N | - | N | N | N | Y | Acceptance criteria entered on the screen | `정의한 권한별 접근이 제한된다` |
| 453 | Regulatory Reference | `regulation` | `text` | N | N | - | N | - | N | N | N | Y | Manually entered regulatory reference or original text not linked to the master. Manage structured clause links in requirement_regulation. | `CSV 규정 근거 검토 메모` |
| 454 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft) / REVIEW (Under Review) / APPROVAL (Pending Approval) / APPROVED (Approved) / REJECTED (Rejected) | `DRAFT` |
| 455 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Screen display version of this revision | `ver1` |
| 456 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 for each logical key | `1` |
| 457 | Latest Revision Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Latest revision for the same logical key. Distinct from the latest approved version. | `TRUE` |
| 458 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising | `요구사항 변경` |
| 459 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Requirement author identifier | `00000000-0000-0000-0000-000000000001` |
| 460 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 461 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 462 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow for this revision. Retrieve reviewers, approvers, and signatures from the linked workflow/action history. | `00000000-0000-0000-0000-000000000001` |
| 463 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 464 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from approval status. | `ACTIVE` |
| 465 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time disposal approval completed | `2026-09-01T00:00:00Z` |
| 466 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 467 | Disposal Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |
| 468 | Registration Source | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL / LIBRARY / SYSTEM_PACKAGE / AI_DRAFT | `MANUAL` |
| 469 | Source Library ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | References library_item.library_id | `00000000-0000-0000-0000-000000000001` |
| 470 | Other Source Reference | `source_reference` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Identifier of a system package or AI draft. Not a database FK because the source is external/polymorphic. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_requirement_1` | Business validation | `workflow_instance_id` | APPROVED requires item_number and workflow_instance_id. Do not assign the same approval number to different logical keys within the same project. |
| `uq_requirement_2` | UNIQUE | `requirement_key, revision_number` | UNIQUE (requirement_key, revision_number) |
| `uq_requirement_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_requirement_3` | UNIQUE | `requirement_key, is_current_version` | UNIQUE (requirement_key) WHERE is_current_version = TRUE |
| `ck_requirement_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_requirement_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_requirement_5_rule` | Business validation | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED requires disposed_at, disposal_reason, and disposal_workflow_id. |
| `ck_requirement_6` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `ck_requirement_6_rule` | Business validation | `source_type, source_library_id` | LIBRARY requires source_library_id. |

#### Business Rules

Each row represents a specific revision. Preserve the previously approved row, issue a new PK for the revision, and retain the logical key. is_current_version indicates the latest revision, not the latest approved version. Referencing FKs and electronic signatures point to the exact revision-row PK and version.

Copy content from a library/system package/AI into this item's draft for editing. Changes to the source do not automatically propagate to approved business items.

requirement_id is the PK of a specific revision row. DQ/FRA/tests/regulations link to that revision row. Distinguish the latest draft from the latest approved revision for each requirement_key.

Link regulatory clauses through requirement_regulation and copy citations when cloning a revision. Retrieve RTM link status from business relationships and determination results.

All revisions of the same requirement_key belong to the same project and business type. Retain item_number in subsequent revisions after initial assignment. For items numbered upon approval, unapproved drafts may have NULL. Do not reuse that number for another logical key within the same project/business type or reassign it to another item after disposal. Serialize number issuance and revision creation using concurrency locks on the project and logical item, and validate duplication, affiliation, and number retention within the same transaction.

[↑ Back to Top](#top)

---

## FDS

<a id="table-fds_spec"></a>
### 32. Functional Design Specification (`fds_spec`)

| Item | Definition |
| --- | --- |
| Description | Uploaded FDS documents and file revision/approval history within the F&DS group. Multiple documents per project are allowed. |
| Primary Key | `fds_id` |
| Key References (FK) | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 471 | FDS ID | `fds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FDS identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 472 | Logical ID Across Revisions | `fds_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision-row PK. | `00000000-0000-0000-0000-000000000001` |
| 473 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Related Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 474 | FDS Number | `fds_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Assigned at initial approval. NULL for drafts; retained across revisions of the same logical key. | `FDS-001` |
| 475 | FDS Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Uses the uploaded file name as the design document's display title | `FDS_ver1.pdf` |
| 476 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Screen display version of this revision | `ver1` |
| 477 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 for each logical key | `1` |
| 478 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising | `요구사항 변경` |
| 479 | Latest Revision Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Latest revision for the same logical key. Distinct from the latest approved version. | `TRUE` |
| 480 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft) / REVIEW (Under Review) / APPROVAL (Pending Approval) / APPROVED (Approved) / REJECTED (Rejected) | `DRAFT` |
| 481 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 482 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FDS author identifier | `00000000-0000-0000-0000-000000000001` |
| 483 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 484 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FDS modifier identifier | `00000000-0000-0000-0000-000000000001` |
| 485 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deletion timestamp for unapproved drafts. Items with approval history undergo disposal approval instead of deletion. | `2026-09-01T00:00:00Z` |
| 486 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow for this revision. Retrieve reviewers, approvers, and signatures from the linked workflow/action history. | `00000000-0000-0000-0000-000000000001` |
| 487 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from approval status. | `ACTIVE` |
| 488 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time disposal approval completed | `2026-09-01T00:00:00Z` |
| 489 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 490 | Disposal Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |
| 491 | Uploaded File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | Y | - | N | Y | N | Y | File uploaded for this design revision. Replacement creates a new revision row and new file ID. | `00000000-0000-0000-0000-000000000001` |
| 492 | Uploaded At | `uploaded_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time this file revision was uploaded | `2026-09-01T00:00:00Z` |
| 493 | Uploaded By ID | `uploaded_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_fds_spec_1` | Business validation | `workflow_instance_id` | APPROVED requires fds_no and workflow_instance_id. Do not assign the same approval number to different logical keys within the same project. |
| `uq_fds_spec_2` | UNIQUE | `fds_key, revision_number` | UNIQUE (fds_key, revision_number) |
| `uq_fds_spec_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_fds_spec_3` | UNIQUE | `fds_key, is_current_version` | UNIQUE (fds_key) WHERE is_current_version = TRUE |
| `ck_fds_spec_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_fds_spec_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_fds_spec_5_rule` | Business validation | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED requires disposed_at, disposal_reason, and disposal_workflow_id. |

#### Business Rules

Each row represents a specific revision. Preserve the previously approved row, issue a new PK for the revision, and retain the logical key. is_current_version indicates the latest revision, not the latest approved version. Referencing FKs and electronic signatures point to the exact revision-row PK and version.

F&DS is one performed activity; display document types in the order FDS then DDS. Do not require prior FDS approval or a single parent FDS for DDS.

Retrieve file replacement history from prior revision rows of the same logical key. DQ references this table's PK as the exact approved file revision.

All revisions of the same fds_key belong to the same project and business type. Retain fds_no in subsequent revisions after initial assignment. For items numbered upon approval, unapproved drafts may have NULL. Do not reuse that number for another logical key within the same project/business type or reassign it to another item after disposal. Serialize number issuance and revision creation using concurrency locks on the project and logical item, and validate duplication, affiliation, and number retention within the same transaction.

[↑ Back to Top](#top)

---

## DDS

<a id="table-dds_spec"></a>
### 33. Detailed Design Specification (`dds_spec`)

| Item | Definition |
| --- | --- |
| Description | Uploaded DDS documents and file revision/approval history within the F&DS group. Multiple documents per project are allowed. |
| Primary Key | `dds_id` |
| Key References (FK) | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 494 | DDS ID | `dds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique DDS document identifier | `UUID` |
| 495 | Logical ID Across Revisions | `dds_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision-row PK. | `00000000-0000-0000-0000-000000000001` |
| 496 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Validation project to which the DDS belongs | `UUID` |
| 497 | DDS Number | `dds_no` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Assigned at initial approval. NULL for drafts; retained across revisions of the same logical key. | `DDS-001` |
| 498 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Uses the uploaded file name as the design document's display title | `DDS_ver1.pdf` |
| 499 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Screen display version of this revision | `ver1` |
| 500 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 for each logical key | `1` |
| 501 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising | `요구사항 변경` |
| 502 | Latest Revision Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Latest revision for the same logical key. Distinct from the latest approved version. | `TRUE` |
| 503 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft) / REVIEW (Under Review) / APPROVAL (Pending Approval) / APPROVED (Approved) / REJECTED (Rejected) | `DRAFT` |
| 504 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS creation timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 505 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who authored the DDS | `UUID` |
| 506 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS last modification timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 507 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the DDS | `UUID` |
| 508 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deletion timestamp for unapproved drafts. Items with approval history undergo disposal approval instead of deletion. | `2026-09-01T00:00:00Z` |
| 509 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow for this revision. Retrieve reviewers, approvers, and signatures from the linked workflow/action history. | `00000000-0000-0000-0000-000000000001` |
| 510 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from approval status. | `ACTIVE` |
| 511 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time disposal approval completed | `2026-09-01T00:00:00Z` |
| 512 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 513 | Disposal Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |
| 514 | Uploaded File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | Y | - | N | Y | N | Y | File uploaded for this design revision. Replacement creates a new revision row and new file ID. | `00000000-0000-0000-0000-000000000001` |
| 515 | Uploaded At | `uploaded_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time this file revision was uploaded | `2026-09-01T00:00:00Z` |
| 516 | Uploaded By ID | `uploaded_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_dds_spec_1` | Business validation | `workflow_instance_id` | APPROVED requires dds_no and workflow_instance_id. Do not assign the same approval number to different logical keys within the same project. |
| `uq_dds_spec_2` | UNIQUE | `dds_key, revision_number` | UNIQUE (dds_key, revision_number) |
| `uq_dds_spec_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_dds_spec_3` | UNIQUE | `dds_key, is_current_version` | UNIQUE (dds_key) WHERE is_current_version = TRUE |
| `ck_dds_spec_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_dds_spec_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_dds_spec_5_rule` | Business validation | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED requires disposed_at, disposal_reason, and disposal_workflow_id. |

#### Business Rules

Each row represents a specific revision. Preserve the previously approved row, issue a new PK for the revision, and retain the logical key. is_current_version indicates the latest revision, not the latest approved version. Referencing FKs and electronic signatures point to the exact revision-row PK and version.

F&DS is one performed activity; display document types in the order FDS then DDS. Do not require prior FDS approval or a single parent FDS for DDS.

Retrieve file replacement history from prior revision rows of the same logical key. DQ references this table's PK as the exact approved file revision.

All revisions of the same dds_key belong to the same project and business type. Retain dds_no in subsequent revisions after initial assignment. For items numbered upon approval, unapproved drafts may have NULL. Do not reuse that number for another logical key within the same project/business type or reassign it to another item after disposal. Serialize number issuance and revision creation using concurrency locks on the project and logical item, and validate duplication, affiliation, and number retention within the same transaction.

[↑ Back to Top](#top)

---

## DQ

<a id="table-dq_assessment"></a>
### 34. Design Qualification Assessment (`dq_assessment`)

| Item | Definition |
| --- | --- |
| Description | Project-specific header grouping DQ assessment items. |
| Primary Key | `dq_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 517 | DQ ID | `dq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | DQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 518 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | Y | Y | N | Y | Related Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 519 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Design qualification assessment document title | `설계 적격성 평가 (URS → FDS/DDS 매핑)` |
| 520 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 521 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DQ author identifier | `00000000-0000-0000-0000-000000000001` |
| 522 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 523 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DQ modifier identifier | `00000000-0000-0000-0000-000000000001` |
| 524 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_dq_assessment_1` | UNIQUE | `project_id` | UNIQUE (project_id) |

#### Business Rules

Manage item-level business approvals/revisions in child tables and generated document numbers, versions, and approval status in deliverable_document/deliverable_revision. Do not copy the header's approval status indiscriminately to all child items.

[↑ Back to Top](#top)

---

<a id="table-dq_item"></a>
### 35. DQ Detailed Assessment Item (`dq_item`)

| Item | Definition |
| --- | --- |
| Description | Manages comments, determinations, failure reasons, execution, and item approval revisions from comparing approved URS and FDS/DDS file revisions. |
| Primary Key | `dq_item_id` |
| Key References (FK) | `dq_id, requirement_id, created_by, updated_by, fds_revision_id, dds_revision_id, executed_by, workflow_instance_id, disposal_workflow_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 525 | DQ Item ID | `dq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | DQ detailed item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 526 | Logical ID Across Revisions | `dq_item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision-row PK. | `00000000-0000-0000-0000-000000000001` |
| 527 | DQ ID | `dq_id` | `uuid` | N | Y | `dq_assessment.dq_id` | Y | - | N | Y | N | Y | Parent DQ document identifier | `00000000-0000-0000-0000-000000000001` |
| 528 | URS Item ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | Y | N | Y | Identifier of the URS being mapped (FK) | `00000000-0000-0000-0000-000000000001` |
| 529 | URS Number | `urs_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Snapshot of the number of the approved revision referenced by requirement_id. Not entered directly. | `URS-001` |
| 530 | URS Requirement | `urs_description` | `text` | N | N | - | Y | - | N | N | N | Y | Snapshot of the requirement content of the approved revision referenced by requirement_id. Not entered directly. | `사용자 로그인 및 전자서명 기능` |
| 531 | Determination | `result_status` | `varchar(10)` | N | N | - | N | - | N | N | N | Y | PASS / FAIL / N/A. NULL when not yet determined. | `PASS` |
| 532 | Remarks | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | Review-related observations and remarks | `FDS 및 DDS 설계 반영 완료 확인` |
| 533 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 534 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item author identifier | `00000000-0000-0000-0000-000000000001` |
| 535 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 536 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item modifier identifier | `00000000-0000-0000-0000-000000000001` |
| 537 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deletion timestamp for unapproved drafts. Items with approval history undergo disposal approval instead of deletion. | `2026-09-01T00:00:00Z` |
| 538 | Approved FDS Revision ID | `fds_revision_id` | `uuid` | N | Y | `fds_spec.fds_id` | N | - | N | Y | N | Y | References fds_spec.fds_id | `00000000-0000-0000-0000-000000000001` |
| 539 | FDS Review Comment | `fds_comment` | `text` | N | N | - | N | - | N | N | N | Y | FDS Review Comment | - |
| 540 | Approved DDS Revision ID | `dds_revision_id` | `uuid` | N | Y | `dds_spec.dds_id` | N | - | N | Y | N | Y | References dds_spec.dds_id | `00000000-0000-0000-0000-000000000001` |
| 541 | DDS Review Comment | `dds_comment` | `text` | N | N | - | N | - | N | N | N | Y | DDS Review Comment | - |
| 542 | Failure Reason | `fail_reason` | `text` | N | N | - | N | - | N | N | N | Y | Failure Reason | - |
| 543 | Determination Performed By ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 544 | Determination Performed At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Determination Performed At | `2026-09-01T00:00:00Z` |
| 545 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Screen display version of this revision | `ver1` |
| 546 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 for each logical key | `1` |
| 547 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising | `요구사항 변경` |
| 548 | Latest Revision Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Latest revision for the same logical key. Distinct from the latest approved version. | `TRUE` |
| 549 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft) / REVIEW (Under Review) / APPROVAL (Pending Approval) / APPROVED (Approved) / REJECTED (Rejected) | `DRAFT` |
| 550 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow for this revision. Retrieve reviewers, approvers, and signatures from the linked workflow/action history. | `00000000-0000-0000-0000-000000000001` |
| 551 | Item Approval Number | `item_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Assigned at initial approval. NULL for drafts; retained across revisions. | `DQ-001` |
| 552 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from approval status. | `ACTIVE` |
| 553 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time disposal approval completed | `2026-09-01T00:00:00Z` |
| 554 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 555 | Disposal Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_dq_item_1` | Business validation | `workflow_instance_id` | APPROVED requires item_number and workflow_instance_id. Do not assign the same approval number to different logical keys within the same project. |
| `uq_dq_item_2` | UNIQUE | `dq_item_key, revision_number` | UNIQUE (dq_item_key, revision_number) |
| `uq_dq_item_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_dq_item_3` | UNIQUE | `dq_item_key, is_current_version` | UNIQUE (dq_item_key) WHERE is_current_version = TRUE |
| `ck_dq_item_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_dq_item_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_dq_item_5_rule` | Business validation | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED requires disposed_at, disposal_reason, and disposal_workflow_id. |
| `ck_dq_item_6` | CHECK | `result_status` | CHECK (result_status IS NULL OR result_status IN ('PASS','FAIL','N/A')) |
| `ck_dq_item_6_rule` | Business validation | `result_status, fail_reason` | FAIL requires a nonblank fail_reason. |
| `rule_dq_item_7` | Business validation | `result_status, executed_by, executed_at` | An approval request requires result_status, executed_by, and executed_at, and at least one of FDS/DDS must be a valid approved revision in the same project. requirement_id must also reference an approved revision in the same project. |

#### Business Rules

Each row represents a specific revision. Preserve the previously approved row, issue a new PK for the revision, and retain the logical key. is_current_version indicates the latest revision, not the latest approved version. Referencing FKs and electronic signatures point to the exact revision-row PK and version.

fds_revision_id and dds_revision_id reference exact revision rows of approved design documents. Retrieve file names, document numbers, and versions from the linked fds_spec/dds_spec.

Store the performing user in executed_by and retrieve reviewers/approvers through workflow_instance and approval_action. Distinguish N/A from an undetermined result and verify design links. DQ FAIL does not automatically create a test deviation.

All revisions of the same dq_item_key belong to the same project and business type. Retain item_number in subsequent revisions after initial assignment. For items numbered upon approval, unapproved drafts may have NULL. Do not reuse that number for another logical key within the same project/business type or reassign it to another item after disposal. Serialize number issuance and revision creation using concurrency locks on the project and logical item, and validate duplication, affiliation, and number retention within the same transaction.

[↑ Back to Top](#top)

---

## FRA

<a id="table-fra_assessment"></a>
### 36. Functional Risk Assessment (`fra_assessment`)

| Item | Definition |
| --- | --- |
| Description | Project-specific header grouping FRA assessment items. |
| Primary Key | `fra_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 556 | FRA ID | `fra_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FRA assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 557 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | Y | Y | N | Y | Related Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 558 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the FMEA-based functional risk assessment document | `FMEA 기반 기능 위험평가` |
| 559 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 560 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FRA author identifier | `00000000-0000-0000-0000-000000000001` |
| 561 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 562 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FRA modifier identifier | `00000000-0000-0000-0000-000000000001` |
| 563 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_fra_assessment_1` | UNIQUE | `project_id` | UNIQUE (project_id) |

#### Business Rules

Manage item-level business approvals/revisions in child tables and generated document numbers, versions, and approval status in deliverable_document/deliverable_revision. Do not copy the header's approval status indiscriminately to all child items.

[↑ Back to Top](#top)

---

<a id="table-fra_item"></a>
### 37. FRA Detailed Risk Item (`fra_item`)

| Item | Definition |
| --- | --- |
| Description | Manages functions linked to approved URS, risk scenarios, risk scores, registration sources, and item approval revisions. |
| Primary Key | `fra_item_id` |
| Key References (FK) | `fra_id, requirement_id, created_by, updated_by, sop_clause_id, workflow_instance_id, disposal_workflow_id, source_library_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 564 | Risk Item ID | `fra_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FRA detailed risk item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 565 | Logical ID Across Revisions | `fra_item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier retained across revisions. Distinct from the revision-row PK. | `00000000-0000-0000-0000-000000000001` |
| 566 | FRA ID | `fra_id` | `uuid` | N | Y | `fra_assessment.fra_id` | Y | - | N | Y | N | Y | Parent FRA document identifier | `00000000-0000-0000-0000-000000000001` |
| 567 | URS Item ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | N | - | N | Y | N | Y | Approved URS revision in the same project. NULL allowed in a draft; required before requesting approval. | `00000000-0000-0000-0000-000000000001` |
| 568 | URS Reference Number | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Display snapshot of the linked requirement revision's approval number. Must match requirement_id; not entered independently. | `URS-001` |
| 569 | Function Name | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Name of the function subject to risk assessment | `전자서명` |
| 570 | Risk Scenario | `risk_scenario` | `text` | N | N | - | Y | - | N | N | N | Y | Description of the FMEA risk occurrence scenario | `전자서명 시 비밀번호 검증 미수행` |
| 571 | Approval Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft) / REVIEW (Under Review) / APPROVAL (Pending Approval) / APPROVED (Approved) / REJECTED (Rejected) | `DRAFT` |
| 572 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 573 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item author identifier | `00000000-0000-0000-0000-000000000001` |
| 574 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 575 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Item modifier identifier | `00000000-0000-0000-0000-000000000001` |
| 576 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deletion timestamp for unapproved drafts. Items with approval history undergo disposal approval instead of deletion. | `2026-09-01T00:00:00Z` |
| 577 | Severity SEV | `severity` | `smallint` | N | N | - | Y | `1` | N | N | N | Y | Screen input from 1 to 5 | `5` |
| 578 | Occurrence OCC | `occurrence` | `smallint` | N | N | - | Y | `1` | N | N | N | Y | Screen input from 1 to 5 | `2` |
| 579 | Detectability DET | `detectability` | `varchar(1)` | N | N | - | Y | `'M'` | N | N | N | Y | H / M / L | `M` |
| 580 | Evaluation Rule Version | `rule_version` | `varchar(50)` | N | N | - | Y | `'UI-FRA-1'` | N | N | N | Y | Evaluation Rule Version | `UI-FRA-1` |
| 581 | Calculated Results at Approval | `risk_result_snapshot` | `jsonb` | N | N | - | N | - | N | N | N | Y | Automatically calculated RP, RC, RPG, NT, and Action Plan results at approval. Not entered directly. | `{"RP":10,"RC":1,"RPG":"H","NT":"N","actionPlan":"Test 수행"}` |
| 582 | SOP Reference Clause ID | `sop_clause_id` | `uuid` | N | Y | `regulatory_clause.regulatory_clause_id` | N | - | N | Y | N | Y | Stores the RTM screen's SOP link in the original risk item | `00000000-0000-0000-0000-000000000001` |
| 583 | Manual SOP Reference | `sop_reference_note` | `text` | N | N | - | N | - | N | N | N | Y | Original SOP reference text not yet in the clause master | - |
| 584 | Display Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Screen display version of this revision | `ver1` |
| 585 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Increments from 1 for each logical key | `1` |
| 586 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising | `요구사항 변경` |
| 587 | Latest Revision Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Latest revision for the same logical key. Distinct from the latest approved version. | `TRUE` |
| 588 | Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Business approval workflow for this revision. Retrieve reviewers, approvers, and signatures from the linked workflow/action history. | `00000000-0000-0000-0000-000000000001` |
| 589 | Item Approval Number | `item_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Assigned at initial approval. NULL for drafts; retained across revisions. | `FRA-001` |
| 590 | Lifecycle Status | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. Separate from approval status. | `ACTIVE` |
| 591 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time disposal approval completed | `2026-09-01T00:00:00Z` |
| 592 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Disposal Reason | - |
| 593 | Disposal Workflow ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |
| 594 | Registration Source | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL / LIBRARY / SYSTEM_PACKAGE / AI_DRAFT | `MANUAL` |
| 595 | Source Library ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | References library_item.library_id | `00000000-0000-0000-0000-000000000001` |
| 596 | Other Source Reference | `source_reference` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Identifier of a system package or AI draft. Not a database FK because the source is external/polymorphic. | - |
| 597 | Applied Risk Evaluation Rule Snapshot | `rule_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,input_ranges,formulas,thresholds,matrix,action_mapping}. Complete definition of allowed SEV/OCC/DET values, RP/RC formulas, RPG/NT decision tables, and Action Plan mappings. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_fra_item_1` | Business validation | `workflow_instance_id` | APPROVED requires item_number and workflow_instance_id. Do not assign the same approval number to different logical keys within the same project. |
| `uq_fra_item_2` | UNIQUE | `fra_item_key, revision_number` | UNIQUE (fra_item_key, revision_number) |
| `uq_fra_item_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_fra_item_3` | UNIQUE | `fra_item_key, is_current_version` | UNIQUE (fra_item_key) WHERE is_current_version = TRUE |
| `ck_fra_item_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_fra_item_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_fra_item_5_rule` | Business validation | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED requires disposed_at, disposal_reason, and disposal_workflow_id. |
| `ck_fra_item_6` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `ck_fra_item_6_rule` | Business validation | `source_type, source_library_id` | LIBRARY requires source_library_id. |
| `ck_fra_item_7` | CHECK | `severity` | CHECK (severity BETWEEN 1 AND 5) |
| `ck_fra_item_7_2` | CHECK | `occurrence` | CHECK (occurrence BETWEEN 1 AND 5) |
| `ck_fra_item_7_3` | CHECK | `detectability` | CHECK (detectability IN ('H','M','L')) |
| `rule_fra_item_8` | Business validation | `requirement_id, risk_result_snapshot` | requirement_id is required before requesting approval. Validate that the referenced revision is a valid approved version in the same project. APPROVED requires workflow_instance_id and risk_result_snapshot. |
| `rule_fra_item_9` | Business validation | - | sop_clause_id must reference a clause of type INTERNAL_SOP. |

#### Business Rules

Each row represents a specific revision. Preserve the previously approved row, issue a new PK for the revision, and retain the logical key. is_current_version indicates the latest revision, not the latest approved version. Referencing FKs and electronic signatures point to the exact revision-row PK and version.

Copy content from a library/system package/AI into this item's draft for editing. Changes to the source do not automatically propagate to approved business items.

Risk calculation rules: RP=SEV×OCC; RC: RP<5→3, RP<10→2, otherwise→1. RPG: for RC1, DET H/M/L→M/H/H; RC2→L/M/H; RC3→L/L/M. NT: RP>24→Y, otherwise N. Action Plan: RPG L→No Action, M→Modify/Delete SOP, H→Perform Test.

Calculate draft results when queried and preserve only the results at approval as a snapshot. Derive risk scores and automatic action plans from evaluation rules; do not store them as independently entered values. Trace tests through actual test/FRA links.

When authoring, copy the actual definition for rule_version into rule_snapshot and calculate results using it. Do not change rules, inputs, or results after approval; create a new item revision for changes. Do not change definitions under the same version name. When submitting an FRA document, fix each supporting item's definition, inputs, and risk_result_snapshot in evaluation_definition_snapshot.

All revisions of the same fra_item_key belong to the same project and business type. Retain item_number in subsequent revisions after initial assignment. For items numbered upon approval, unapproved drafts may have NULL. Do not reuse that number for another logical key within the same project/business type or reassign it to another item after disposal. Serialize number issuance and revision creation using concurrency locks on the project and logical item, and validate duplication, affiliation, and number retention within the same transaction.

[↑ Back to Top](#top)

---

## IQ

<a id="table-iq_assessment"></a>
### 38. Installation Qualification Assessment (`iq_assessment`)

| Item | Definition |
| --- | --- |
| Description | Project-specific IQ test grouping and document revision information. Distinguishes protocol approval status from execution-result approval status. |
| Primary Key | `iq_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 598 | IQ ID | `iq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | IQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 599 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 600 | IQ Number | `iq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | IQ document number. Retain the same number across revision rows of the same document. | `IQ-VP-SYS-010-20260529` |
| 601 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Installation qualification assessment document title | `Installation Qualification` |
| 602 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | IQ assessment document display version (e.g., v1.0, v1.1) | `v1.0` |
| 603 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | IQ assessment document revision sequence (e.g., 1, 2, 3...) | `1` |
| 604 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the IQ assessment document | `최초 작성` |
| 605 | Current Latest Version Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest IQ assessment document version (TRUE/FALSE) | `TRUE` |
| 606 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | IQ protocol approval summary. DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). Aggregates individual item/execution statuses without overwriting child rows en masse. | `DRAFT` |
| 607 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | IQ execution-result approval summary. DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). Aggregates individual item/execution statuses without overwriting child rows en masse. | `DRAFT` |
| 608 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 609 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | IQ author identifier | `00000000-0000-0000-0000-000000000001` |
| 610 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 611 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | IQ modifier identifier | `00000000-0000-0000-0000-000000000001` |
| 612 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |
| 613 | Test Composition per Assessment Revision | `item_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | Array of [{table_name:"iq_item",record_id,version,sort_order}]. Identifies the complete test composition and order for this assessment revision. PKs of unchanged tests from prior assessment revisions may be reused. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_iq_assessment_1` | UNIQUE | `project_id, iq_no, revision_number` | UNIQUE(project_id, iq_no, revision_number) |
| `uq_iq_assessment_1_rule` | Business validation | `project_id, iq_no, revision_number, is_current_version` | At most one row per document may have is_current_version = TRUE. |
| `rule_iq_assessment_2` | Business validation | - | Distinguish document revisions from test item revisions. Derive the document screen's approval summary from child protocols and execution results. |
| `uq_iq_assessment_current` | UNIQUE | `project_id, iq_no` | UNIQUE (project_id, iq_no) WHERE is_current_version=TRUE |

#### Business Rules

Manages test lists, protocol approval, result approval, and document generation information. Manage document content in deliverable_document / deliverable_revision / deliverable_section.

When only the deliverable title, body, or output format changes, create only a new deliverable_revision and retain the assessment row and test items. If the assessment scope, parent business information, or test composition changes, create a new assessment revision row and fix the complete composition at that time in item_revision_refs. Reference unchanged test items using the same PK/version; do not clone them or move their parent FK. Only items whose test protocol content changes receive a new revision row retaining the same item_key and are included in the new composition.

Composition targets must be actual revisions of iq_item belonging to the same project and IQ stage. The item's original parent assessment and this assessment must have the same iq_no. Do not include different revisions of the same item_key in one composition; record_id and sort_order must also be unique. The composition is the complete ordered list, locked when a document is first submitted. Do not modify or delete compositions referenced by submitted documents or historical signatures. Do not automatically include rows outside the composition solely on the basis of their original parent FK. Derive protocol/result approval summaries from items and executions belonging to that composition.

[↑ Back to Top](#top)

---

<a id="table-iq_item"></a>
### 39. IQ Detailed Test Item (`iq_item`)

| Item | Definition |
| --- | --- |
| Description | Manages IQ test item protocol revisions, content, expected results, acceptance criteria, and registration sources |
| Primary Key | `iq_item_id` |
| Key References (FK) | `iq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_signature_id, disposed_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 614 | IQ Item ID | `iq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | PK of a specific protocol revision row for an IQ test item. Retain the same item_key across revisions. | `00000000-0000-0000-0000-000000000001` |
| 615 | IQ ID | `iq_id` | `uuid` | N | Y | `iq_assessment.iq_id` | Y | - | N | N | N | Y | Parent assessment revision in which this logical test was first registered. Retain the same original parent in subsequent item revisions. Retrieve the actual composition per assessment revision from item_revision_refs. | `00000000-0000-0000-0000-000000000001` |
| 616 | Test Item Logical ID | `item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier grouping revisions of the same test item. Issue once for a new item and retain on revision. | `00000000-0000-0000-0000-000000000001` |
| 617 | Test ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Test display ID on the screen. Retain the same value across revisions of the same item; prohibit duplication across different item_key values within the project. | `IQ-NEW-01` |
| 618 | Test Case | `test_case` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Test item title on the screen. Not a separate classification value. | `하드웨어 설치` |
| 619 | Test Description | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed procedure for performing the verification test | `설치될 서버의 하드웨어 사양이 URS를 충족하는지 확인` |
| 620 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Test success criteria and expected result | `하드웨어 사양이 URS에 명시된 요구사항과 일치해야 함` |
| 621 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). Independent approval status of this test revision; do not overwrite it with the parent document's status. | `DRAFT` |
| 622 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 623 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Item author identifier | `00000000-0000-0000-0000-000000000001` |
| 624 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 625 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Item modifier identifier | `00000000-0000-0000-0000-000000000001` |
| 626 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |
| 627 | Protocol Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Protocol revision version displayed on the screen. | `ver1` |
| 628 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Revision sequence within item_key. At least 1. | `1` |
| 629 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising an approved protocol. | - |
| 630 | Latest Revision Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the latest working revision for the same item_key. Separate from approved-version status. | `TRUE` |
| 631 | Acceptance Criteria | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | Acceptance criteria on the screen. Distinct from expected_result (expected result). | `승인된 사양과 일치` |
| 632 | Registration Source | `source_type` | `varchar(20)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL (Direct Registration), LIBRARY (Library), PACKAGE (System Package), AI (AI Draft). | `MANUAL` |
| 633 | Source Library ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | Original source when registered from a library. Test content is copied at registration and managed independently thereafter. | `00000000-0000-0000-0000-000000000001` |
| 634 | Source Package Code | `source_package_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Package code when registered from a system package. | `IQ-CORE` |
| 635 | Source Package Version | `source_package_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Package version selected at registration. | `ver2` |
| 636 | Source Template Code | `source_template_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Code of the test template selected within the package. | `IQ-PKG-001` |
| 637 | Item Approval Number | `approval_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Issued at initial approval and displayed as the item number. Retain the same logical item number on revision. | - |
| 638 | Protocol Approval Workflow | `protocol_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Authoring, review, and approval route and history for this protocol revision. | `00000000-0000-0000-0000-000000000001` |
| 639 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason entered on the disposal approval screen. Required when disposal is completed. | - |
| 640 | Disposal Approval Signature | `disposal_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature provided by an authorized disposer after confirming the reason and target. | `00000000-0000-0000-0000-000000000001` |
| 641 | Disposed Flag | `is_disposed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Disposal status shown on the screen. Disposed items are excluded from new executions and aggregation; existing records are preserved. | `FALSE` |
| 642 | Disposed By | `disposed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 643 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when disposal was completed on the screen. | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_iq_item_1` | UNIQUE | `item_key, revision_number` | UNIQUE(item_key, revision_number) |
| `uq_iq_item_1_rule` | Business validation | `item_key, revision_number, is_current_version` | Rows with the same item_key belong to the same project and IQ stage. At most one row per item_key may have is_current_version = TRUE. |
| `rule_iq_item_2` | Business validation | - | protocol_status is DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), or REJECTED (Rejected). Approved protocols and steps are not edited directly; a new revision row is created. |
| `rule_iq_item_3` | Business validation | - | approval_number is the initial approval number of the logical item and is retained across revisions. Approval and disposal signatures are preserved in workflow_instance / approval_action / electronic_signature for the corresponding item revision. |
| `rule_iq_item_4` | Business validation | `source_library_id, source_package_code, source_package_version, source_template_code` | LIBRARY registration requires source_library_id. PACKAGE registration requires source_package_code / source_package_version / source_template_code. |
| `rule_iq_item_5` | Business validation | - | Links to multiple URS items are managed through REQUIREMENT → IQ_ITEM / VERIFIED_BY in traceability_link. |
| `rule_iq_item_6` | Business validation | - | Registration of an active test and protocol approval require at least one linked URS revision and at least one detailed step with nonempty content. |
| `rule_iq_item_7` | Business validation | `disposal_reason, disposal_signature_id, is_disposed, disposed_by, disposed_at` | When is_disposed = TRUE, disposal_reason, disposal_signature_id, disposed_by, and disposed_at are required. The actual target table and PK of the disposal signature must identify this test revision, and the signer and timestamp must match disposed_by / disposed_at. |
| `rule_iq_item_8` | Business validation | - | Disposal applies to the active status of the same item_key, while historical approval records are preserved. Adding disposal status, reason, and signature metadata is distinct from a revision that changes approved test content or steps; the originally approved content remains unchanged. |
| `uq_iq_item_current` | UNIQUE | `item_key` | UNIQUE (item_key) WHERE is_current_version=TRUE |

#### Business Rules

Test content is stored in test_description, expected results in expected_result, and acceptance criteria in acceptance_criteria. Repeated steps are managed in iq_step, while actual results, signatures, and rerun history are managed in iq_execution.

All revisions with the same item_key belong to the same project and business type. Item numbers approval_number and test_id are retained in subsequent revisions after initial assignment. Unapproved drafts may have NULL for numbers assigned upon approval. A number must not be reused for another logical key within the same project and business type, or reassigned to another item after disposal. Number allocation and revision creation are serialized using concurrency locks on the project and logical item, and duplication, ownership, and number retention conditions are validated in the same transaction.

[↑ Back to Top](#top)

---

<a id="table-iq_step"></a>
### 40. IQ Test Step (`iq_step`)

| Item | Definition |
| --- | --- |
| Description | Ordered detailed steps included in a IQ protocol revision |
| Primary Key | `step_id` |
| Key References (FK) | `iq_item_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 644 | Step ID | `step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 645 | Test Revision ID | `iq_item_id` | `uuid` | N | Y | `iq_item.iq_item_id` | Y | - | N | Y | N | Y | References iq_item.iq_item_id | `00000000-0000-0000-0000-000000000001` |
| 646 | Step Order | `step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display order of detailed steps on the screen. Must be at least 1. | `1` |
| 647 | Step Content | `instruction` | `text` | N | N | - | Y | - | N | N | N | Y | Step content added, edited, or deleted on the screen. | - |
| 648 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 649 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 650 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 651 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_iq_step_1` | UNIQUE | `iq_item_id, step_order` | UNIQUE(iq_item_id, step_order) |
| `uq_iq_step_1_rule` | Business validation | `iq_item_id, step_order` | step_order >= 1 |
| `rule_iq_step_2` | Business validation | - | When the parent protocol is APPROVED, its step structure and content must not be changed. Changes are recorded as a new test revision and its child steps. |

#### Business Rules

Step completion and attached evidence are execution-attempt records in iq_step_execution, rather than design values in this table.

[↑ Back to Top](#top)

---

<a id="table-iq_execution"></a>
### 41. IQ Test Execution (`iq_execution`)

| Item | Definition |
| --- | --- |
| Description | Management of actual results, outcomes, execution signatures, and result approval for each execution attempt of a IQ test revision |
| Primary Key | `execution_id` |
| Key References (FK) | `iq_item_id, executed_by, execution_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 652 | Execution ID | `execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 653 | Executed Protocol Revision ID | `iq_item_id` | `uuid` | N | Y | `iq_item.iq_item_id` | Y | - | N | Y | N | Y | References iq_item.iq_item_id | `00000000-0000-0000-0000-000000000001` |
| 654 | Execution Attempt Number | `attempt_no` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 1 for the initial execution and 2 or higher for reruns. The rerun count shown on the screen is attempt_no - 1. | `1` |
| 655 | Result Correction Sequence | `record_revision` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Sequence number in the correction history of results for the same attempt. Distinct from a rerun. | `1` |
| 656 | Current Result Correction Flag | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the currently valid result record for the same test revision and attempt. | `TRUE` |
| 657 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Actual result entered on the screen. Required when registering an outcome. | - |
| 658 | Execution Outcome | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | PASS, FAIL, or NA. Not executed is represented by NULL and is distinct from approval status. | `PASS` |
| 659 | Execution Date | `performed_on` | `date` | N | N | - | N | - | N | N | N | Y | Execution date selected on the screen. Distinct from the electronic signature timestamp. | `2026-09-16` |
| 660 | Executed By | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 661 | Result Registration Timestamp | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the outcome was registered using an electronic signature. | `2026-09-01T00:00:00Z` |
| 662 | Execution Signature ID | `execution_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature provided when registering execution results. Passwords are not stored. | `00000000-0000-0000-0000-000000000001` |
| 663 | Execution Progress Status | `execution_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS (Performing steps / entering results), RECORDED (Outcome registered). Deviations and final approval are managed separately. | `IN_PROGRESS` |
| 664 | Result Approval Status | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). Managed independently of protocol approval status. | `DRAFT` |
| 665 | Result Approval Workflow | `result_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Final result review and approval route for the corresponding attempt and result correction. | `00000000-0000-0000-0000-000000000001` |
| 666 | Result Approval Version | `approval_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Display version of the finally approved test result. Distinct from the protocol version. | `ver1` |
| 667 | Deviation Reason | `deviation_reason` | `text` | N | N | - | N | - | N | N | N | Y | Snapshot of the deviation reason entered when registering a FAIL result. | - |
| 668 | Immediate Action | `immediate_action` | `text` | N | N | - | N | - | N | N | N | Y | Snapshot of the immediate action entered when registering a FAIL result. | - |
| 669 | Result Correction Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for correcting the result when record_revision is incremented. | - |
| 670 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 671 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 672 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 673 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 674 | Validation Target Baseline at Execution | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | The project's current baseline is fixed at the start of execution. Required before result registration and signing, and must not be overwritten by subsequent project baseline changes. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_iq_execution_1` | UNIQUE | `iq_item_id, attempt_no, record_revision` | UNIQUE(iq_item_id, attempt_no, record_revision) |
| `uq_iq_execution_1_rule` | Business validation | `iq_item_id, attempt_no, record_revision, is_current_revision` | attempt_no / record_revision >= 1. At most one row for the same test revision and attempt may have is_current_revision = TRUE. |
| `rule_iq_execution_2` | Business validation | `actual_result, qualification_result, performed_on, executed_by, executed_at, execution_signature_id` | Execution is allowed only for an approved protocol. Transition to RECORDED requires completion of all steps and actual_result / qualification_result / performed_on / executed_by / executed_at / execution_signature_id. |
| `rule_iq_execution_3` | Business validation | `qualification_result, deviation_reason, immediate_action` | When qualification_result = FAIL, deviation_reason and immediate_action are required, and a deviation is linked using the corresponding execution_id as its source. |
| `rule_iq_execution_4` | Business validation | - | A rerun starts with a new attempt_no while preserving existing results, step completion records, and evidence. Correcting results for the same attempt increments record_revision and does not overwrite prior approval records. |
| `rule_iq_execution_5` | Business validation | - | Final result approval is possible after protocol approval and execution result registration. Required approval and closure statuses of linked deviations are also checked. |
| `ck_iq_execution_baseline` | CHECK | `execution_status, system_baseline_id` | CHECK (execution_status <> 'RECORDED' OR system_baseline_id IS NOT NULL) |

#### Business Rules

Evidence for the entire test is linked to the IQ_EXECUTION target in evidence_link. The electronic signature targets this execution_id, and a new signature is obtained after a result correction.

system_baseline_id must belong to the same project as the test. Protocol approval and prerequisites are validated against the baseline adopted at the start of execution. A rerun links the baseline applicable at that time to a new execution record, while preserving historical execution baselines. A result correction is recorded as a new record_revision for the same logical test and attempt_no, retaining the baseline of the initial execution.

[↑ Back to Top](#top)

---

<a id="table-iq_step_execution"></a>
### 42. IQ Step Execution (`iq_step_execution`)

| Item | Definition |
| --- | --- |
| Description | Step completion and step evidence linkage unit for a IQ execution attempt |
| Primary Key | `step_execution_id` |
| Key References (FK) | `execution_id, step_id, confirmed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 675 | Step Execution ID | `step_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 676 | Execution ID | `execution_id` | `uuid` | N | Y | `iq_execution.execution_id` | Y | - | N | Y | N | Y | References iq_execution.execution_id | `00000000-0000-0000-0000-000000000001` |
| 677 | Step ID | `step_id` | `uuid` | N | Y | `iq_step.step_id` | Y | - | N | Y | N | Y | References iq_step.step_id | `00000000-0000-0000-0000-000000000001` |
| 678 | Step Completion Flag | `is_confirmed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Completion/cancellation value for a detailed step on the screen. | `FALSE` |
| 679 | Confirmed By | `confirmed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 680 | Confirmation Timestamp | `confirmed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the step was marked complete. | `2026-09-01T00:00:00Z` |
| 681 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 682 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 683 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 684 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_iq_step_execution_1` | UNIQUE | `execution_id, step_id` | UNIQUE(execution_id, step_id) |
| `uq_iq_step_execution_1_rule` | Business validation | `execution_id, step_id` | The step and execution must belong to the same protocol revision. |
| `rule_iq_step_execution_2` | Business validation | `is_confirmed, confirmed_by, confirmed_at` | When is_confirmed = TRUE, confirmed_by and confirmed_at are required. After result registration, completion/cancellation and evidence changes are locked, and historical records are preserved. |

#### Business Rules

Step attachments are linked through IQ_STEP_EXECUTION / step_execution_id in evidence_link. A new step execution row is created for a rerun.

[↑ Back to Top](#top)

---

## OQ

<a id="table-oq_assessment"></a>
### 43. Operational Qualification Assessment (`oq_assessment`)

| Item | Definition |
| --- | --- |
| Description | Project-specific OQ test grouping and document revision information. Distinguishes protocol approval status from execution-result approval status. |
| Primary Key | `oq_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 685 | OQ ID | `oq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | OQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 686 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 687 | OQ Number | `oq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | OQ document number. Retain the same number across revision rows of the same document. | `OQ-VP-SYS-010-20260529` |
| 688 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Operational Qualification document title | `Operational Qualification` |
| 689 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | OQ assessment document display version (e.g., v1.0, v1.1) | `v1.0` |
| 690 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | OQ assessment document revision sequence (e.g., 1, 2, 3...) | `1` |
| 691 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the OQ assessment document | `최초 작성` |
| 692 | Current Latest Version Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest OQ assessment document version (TRUE/FALSE) | `TRUE` |
| 693 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | OQ protocol approval summary. DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). Aggregates individual item/execution statuses without overwriting child rows en masse. | `DRAFT` |
| 694 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | OQ execution-result approval summary. DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). Aggregates individual item/execution statuses without overwriting child rows en masse. | `DRAFT` |
| 695 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 696 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | OQ author identifier | `00000000-0000-0000-0000-000000000001` |
| 697 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 698 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | OQ modifier identifier | `00000000-0000-0000-0000-000000000001` |
| 699 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |
| 700 | Test Composition per Assessment Revision | `item_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | Array of [{table_name:"oq_item",record_id,version,sort_order}]. Identifies the complete test composition and order for this assessment revision. PKs of unchanged tests from prior assessment revisions may be reused. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_oq_assessment_1` | UNIQUE | `project_id, oq_no, revision_number` | UNIQUE(project_id, oq_no, revision_number) |
| `uq_oq_assessment_1_rule` | Business validation | `project_id, oq_no, revision_number, is_current_version` | At most one row per document may have is_current_version = TRUE. |
| `rule_oq_assessment_2` | Business validation | - | Distinguish document revisions from test item revisions. Derive the document screen's approval summary from child protocols and execution results. |
| `uq_oq_assessment_current` | UNIQUE | `project_id, oq_no` | UNIQUE (project_id, oq_no) WHERE is_current_version=TRUE |

#### Business Rules

Manages test lists, protocol approval, result approval, and document generation information. Manage document content in deliverable_document / deliverable_revision / deliverable_section.

When only the deliverable title, body, or output format changes, create only a new deliverable_revision and retain the assessment row and test items. If the assessment scope, parent business information, or test composition changes, create a new assessment revision row and fix the complete composition at that time in item_revision_refs. Reference unchanged test items using the same PK/version; do not clone them or move their parent FK. Only items whose test protocol content changes receive a new revision row retaining the same item_key and are included in the new composition.

Composition targets must be actual revisions of oq_item belonging to the same project and OQ stage. The item's original parent assessment and this assessment must have the same oq_no. Do not include different revisions of the same item_key in one composition; record_id and sort_order must also be unique. The composition is the complete ordered list, locked when a document is first submitted. Do not modify or delete compositions referenced by submitted documents or historical signatures. Do not automatically include rows outside the composition solely on the basis of their original parent FK. Derive protocol/result approval summaries from items and executions belonging to that composition.

[↑ Back to Top](#top)

---

<a id="table-oq_item"></a>
### 44. OQ Detailed Test Item (`oq_item`)

| Item | Definition |
| --- | --- |
| Description | Manages OQ test item protocol revisions, content, expected results, acceptance criteria, and registration sources |
| Primary Key | `oq_item_id` |
| Key References (FK) | `oq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_signature_id, disposed_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 701 | OQ Item ID | `oq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | PK of a specific protocol revision row for an OQ test item. Retain the same item_key across revisions. | `00000000-0000-0000-0000-000000000001` |
| 702 | OQ ID | `oq_id` | `uuid` | N | Y | `oq_assessment.oq_id` | Y | - | N | N | N | Y | Parent assessment revision in which this logical test was first registered. Retain the same original parent in subsequent item revisions. Retrieve the actual composition per assessment revision from item_revision_refs. | `00000000-0000-0000-0000-000000000001` |
| 703 | Test Item Logical ID | `item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier grouping revisions of the same test item. Issue once for a new item and retain on revision. | `00000000-0000-0000-0000-000000000001` |
| 704 | Test ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Test display ID on the screen. Retain the same value across revisions of the same item; prohibit duplication across different item_key values within the project. | `OQ-AT-L01` |
| 705 | Test Case | `test_case` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Test item title on the screen. Not a separate classification value. | `Audit Trail` |
| 706 | Test Description | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed procedure for performing the verification test | `사용자 데이터 변경 시 감사추적 자동 생성` |
| 707 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Test success criteria and expected result | `변경 전/후 값, 사용자, 날짜/시간, IP 기록` |
| 708 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). Independent approval status of this test revision; do not overwrite it with the parent document's status. | `DRAFT` |
| 709 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 710 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Item author identifier | `00000000-0000-0000-0000-000000000001` |
| 711 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 712 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Item modifier identifier | `00000000-0000-0000-0000-000000000001` |
| 713 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |
| 714 | Protocol Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Protocol revision version displayed on the screen. | `ver1` |
| 715 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Revision sequence within item_key. At least 1. | `1` |
| 716 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising an approved protocol. | - |
| 717 | Latest Revision Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the latest working revision for the same item_key. Separate from approved-version status. | `TRUE` |
| 718 | Acceptance Criteria | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | Acceptance criteria on the screen. Distinct from expected_result (expected result). | `승인된 사양과 일치` |
| 719 | Registration Source | `source_type` | `varchar(20)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL (Direct Registration), LIBRARY (Library), PACKAGE (System Package), AI (AI Draft). | `MANUAL` |
| 720 | Source Library ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | Original source when registered from a library. Test content is copied at registration and managed independently thereafter. | `00000000-0000-0000-0000-000000000001` |
| 721 | Source Package Code | `source_package_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Package code when registered from a system package. | `IQ-CORE` |
| 722 | Source Package Version | `source_package_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Package version selected at registration. | `ver2` |
| 723 | Source Template Code | `source_template_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Code of the test template selected within the package. | `IQ-PKG-001` |
| 724 | Item Approval Number | `approval_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Issued at initial approval and displayed as the item number. Retain the same logical item number on revision. | - |
| 725 | Protocol Approval Workflow | `protocol_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Authoring, review, and approval route and history for this protocol revision. | `00000000-0000-0000-0000-000000000001` |
| 726 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason entered on the disposal approval screen. Required when disposal is completed. | - |
| 727 | Disposal Approval Signature | `disposal_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature provided by an authorized disposer after confirming the reason and target. | `00000000-0000-0000-0000-000000000001` |
| 728 | Disposed Flag | `is_disposed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Disposal status shown on the screen. Disposed items are excluded from new executions and aggregation; existing records are preserved. | `FALSE` |
| 729 | Disposed By | `disposed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 730 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when disposal was completed on the screen. | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_oq_item_1` | UNIQUE | `item_key, revision_number` | UNIQUE(item_key, revision_number) |
| `uq_oq_item_1_rule` | Business validation | `item_key, revision_number, is_current_version` | Rows with the same item_key belong to the same project and OQ stage. At most one row per item_key may have is_current_version = TRUE. |
| `rule_oq_item_2` | Business validation | - | protocol_status is DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), or REJECTED (Rejected). Approved protocols and steps are not edited directly; a new revision row is created. |
| `rule_oq_item_3` | Business validation | - | approval_number is the initial approval number of the logical item and is retained across revisions. Approval and disposal signatures are preserved in workflow_instance / approval_action / electronic_signature for the corresponding item revision. |
| `rule_oq_item_4` | Business validation | `source_library_id, source_package_code, source_package_version, source_template_code` | LIBRARY registration requires source_library_id. PACKAGE registration requires source_package_code / source_package_version / source_template_code. |
| `rule_oq_item_5` | Business validation | - | Links to multiple URS items are managed through REQUIREMENT → OQ_ITEM / VERIFIED_BY in traceability_link. |
| `rule_oq_item_6` | Business validation | - | Registration of an active test and protocol approval require at least one linked URS revision and at least one detailed step with nonempty content. |
| `rule_oq_item_7` | Business validation | `disposal_reason, disposal_signature_id, is_disposed, disposed_by, disposed_at` | When is_disposed = TRUE, disposal_reason, disposal_signature_id, disposed_by, and disposed_at are required. The actual target table and PK of the disposal signature must identify this test revision, and the signer and timestamp must match disposed_by / disposed_at. |
| `rule_oq_item_8` | Business validation | - | Disposal applies to the active status of the same item_key, while historical approval records are preserved. Adding disposal status, reason, and signature metadata is distinct from a revision that changes approved test content or steps; the originally approved content remains unchanged. |
| `uq_oq_item_current` | UNIQUE | `item_key` | UNIQUE (item_key) WHERE is_current_version=TRUE |

#### Business Rules

Test content is stored in test_description, expected results in expected_result, and acceptance criteria in acceptance_criteria. Repeated steps are managed in oq_step, while actual results, signatures, and rerun history are managed in oq_execution.

All revisions with the same item_key belong to the same project and business type. Item numbers approval_number and test_id are retained in subsequent revisions after initial assignment. Unapproved drafts may have NULL for numbers assigned upon approval. A number must not be reused for another logical key within the same project and business type, or reassigned to another item after disposal. Number allocation and revision creation are serialized using concurrency locks on the project and logical item, and duplication, ownership, and number retention conditions are validated in the same transaction.

[↑ Back to Top](#top)

---

<a id="table-oq_step"></a>
### 45. OQ Test Step (`oq_step`)

| Item | Definition |
| --- | --- |
| Description | Ordered detailed steps included in a OQ protocol revision |
| Primary Key | `step_id` |
| Key References (FK) | `oq_item_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 731 | Step ID | `step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 732 | Test Revision ID | `oq_item_id` | `uuid` | N | Y | `oq_item.oq_item_id` | Y | - | N | Y | N | Y | References oq_item.oq_item_id | `00000000-0000-0000-0000-000000000001` |
| 733 | Step Order | `step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display order of detailed steps on the screen. Must be at least 1. | `1` |
| 734 | Step Content | `instruction` | `text` | N | N | - | Y | - | N | N | N | Y | Step content added, edited, or deleted on the screen. | - |
| 735 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 736 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 737 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 738 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_oq_step_1` | UNIQUE | `oq_item_id, step_order` | UNIQUE(oq_item_id, step_order) |
| `uq_oq_step_1_rule` | Business validation | `oq_item_id, step_order` | step_order >= 1 |
| `rule_oq_step_2` | Business validation | - | When the parent protocol is APPROVED, its step structure and content must not be changed. Changes are recorded as a new test revision and its child steps. |

#### Business Rules

Step completion and attached evidence are execution-attempt records in oq_step_execution, rather than design values in this table.

[↑ Back to Top](#top)

---

<a id="table-oq_execution"></a>
### 46. OQ Test Execution (`oq_execution`)

| Item | Definition |
| --- | --- |
| Description | Management of actual results, outcomes, execution signatures, and result approval for each execution attempt of a OQ test revision |
| Primary Key | `execution_id` |
| Key References (FK) | `oq_item_id, executed_by, execution_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 739 | Execution ID | `execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 740 | Executed Protocol Revision ID | `oq_item_id` | `uuid` | N | Y | `oq_item.oq_item_id` | Y | - | N | Y | N | Y | References oq_item.oq_item_id | `00000000-0000-0000-0000-000000000001` |
| 741 | Execution Attempt Number | `attempt_no` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 1 for the initial execution and 2 or higher for reruns. The rerun count shown on the screen is attempt_no - 1. | `1` |
| 742 | Result Correction Sequence | `record_revision` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Sequence number in the correction history of results for the same attempt. Distinct from a rerun. | `1` |
| 743 | Current Result Correction Flag | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the currently valid result record for the same test revision and attempt. | `TRUE` |
| 744 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Actual result entered on the screen. Required when registering an outcome. | - |
| 745 | Execution Outcome | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | PASS, FAIL, or NA. Not executed is represented by NULL and is distinct from approval status. | `PASS` |
| 746 | Execution Date | `performed_on` | `date` | N | N | - | N | - | N | N | N | Y | Execution date selected on the screen. Distinct from the electronic signature timestamp. | `2026-09-16` |
| 747 | Executed By | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 748 | Result Registration Timestamp | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the outcome was registered using an electronic signature. | `2026-09-01T00:00:00Z` |
| 749 | Execution Signature ID | `execution_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature provided when registering execution results. Passwords are not stored. | `00000000-0000-0000-0000-000000000001` |
| 750 | Execution Progress Status | `execution_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS (Performing steps / entering results), RECORDED (Outcome registered). Deviations and final approval are managed separately. | `IN_PROGRESS` |
| 751 | Result Approval Status | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). Managed independently of protocol approval status. | `DRAFT` |
| 752 | Result Approval Workflow | `result_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Final result review and approval route for the corresponding attempt and result correction. | `00000000-0000-0000-0000-000000000001` |
| 753 | Result Approval Version | `approval_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Display version of the finally approved test result. Distinct from the protocol version. | `ver1` |
| 754 | Deviation Reason | `deviation_reason` | `text` | N | N | - | N | - | N | N | N | Y | Snapshot of the deviation reason entered when registering a FAIL result. | - |
| 755 | Immediate Action | `immediate_action` | `text` | N | N | - | N | - | N | N | N | Y | Snapshot of the immediate action entered when registering a FAIL result. | - |
| 756 | Result Correction Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for correcting the result when record_revision is incremented. | - |
| 757 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 758 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 759 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 760 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 761 | Validation Target Baseline at Execution | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | The project's current baseline is fixed at the start of execution. Required before result registration and signing, and must not be overwritten by subsequent project baseline changes. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_oq_execution_1` | UNIQUE | `oq_item_id, attempt_no, record_revision` | UNIQUE(oq_item_id, attempt_no, record_revision) |
| `uq_oq_execution_1_rule` | Business validation | `oq_item_id, attempt_no, record_revision, is_current_revision` | attempt_no / record_revision >= 1. At most one row for the same test revision and attempt may have is_current_revision = TRUE. |
| `rule_oq_execution_2` | Business validation | `actual_result, qualification_result, performed_on, executed_by, executed_at, execution_signature_id` | Execution is allowed only for an approved protocol. Transition to RECORDED requires completion of all steps and actual_result / qualification_result / performed_on / executed_by / executed_at / execution_signature_id. |
| `rule_oq_execution_3` | Business validation | `qualification_result, deviation_reason, immediate_action` | When qualification_result = FAIL, deviation_reason and immediate_action are required, and a deviation is linked using the corresponding execution_id as its source. |
| `rule_oq_execution_4` | Business validation | - | A rerun starts with a new attempt_no while preserving existing results, step completion records, and evidence. Correcting results for the same attempt increments record_revision and does not overwrite prior approval records. |
| `rule_oq_execution_5` | Business validation | - | Final result approval is possible after protocol approval and execution result registration. Required approval and closure statuses of linked deviations are also checked. |
| `ck_oq_execution_baseline` | CHECK | `execution_status, system_baseline_id` | CHECK (execution_status <> 'RECORDED' OR system_baseline_id IS NOT NULL) |

#### Business Rules

Evidence for the entire test is linked to the OQ_EXECUTION target in evidence_link. The electronic signature targets this execution_id, and a new signature is obtained after a result correction.

system_baseline_id must belong to the same project as the test. Protocol approval and prerequisites are validated against the baseline adopted at the start of execution. A rerun links the baseline applicable at that time to a new execution record, while preserving historical execution baselines. A result correction is recorded as a new record_revision for the same logical test and attempt_no, retaining the baseline of the initial execution.

[↑ Back to Top](#top)

---

<a id="table-oq_step_execution"></a>
### 47. OQ Step Execution (`oq_step_execution`)

| Item | Definition |
| --- | --- |
| Description | Step completion and step evidence linkage unit for a OQ execution attempt |
| Primary Key | `step_execution_id` |
| Key References (FK) | `execution_id, step_id, confirmed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 762 | Step Execution ID | `step_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 763 | Execution ID | `execution_id` | `uuid` | N | Y | `oq_execution.execution_id` | Y | - | N | Y | N | Y | References oq_execution.execution_id | `00000000-0000-0000-0000-000000000001` |
| 764 | Step ID | `step_id` | `uuid` | N | Y | `oq_step.step_id` | Y | - | N | Y | N | Y | References oq_step.step_id | `00000000-0000-0000-0000-000000000001` |
| 765 | Step Completion Flag | `is_confirmed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Completion/cancellation value for a detailed step on the screen. | `FALSE` |
| 766 | Confirmed By | `confirmed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 767 | Confirmation Timestamp | `confirmed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the step was marked complete. | `2026-09-01T00:00:00Z` |
| 768 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 769 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 770 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 771 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_oq_step_execution_1` | UNIQUE | `execution_id, step_id` | UNIQUE(execution_id, step_id) |
| `uq_oq_step_execution_1_rule` | Business validation | `execution_id, step_id` | The step and execution must belong to the same protocol revision. |
| `rule_oq_step_execution_2` | Business validation | `is_confirmed, confirmed_by, confirmed_at` | When is_confirmed = TRUE, confirmed_by and confirmed_at are required. After result registration, completion/cancellation and evidence changes are locked, and historical records are preserved. |

#### Business Rules

Step attachments are linked through OQ_STEP_EXECUTION / step_execution_id in evidence_link. A new step execution row is created for a rerun.

[↑ Back to Top](#top)

---

## PQ

<a id="table-pq_assessment"></a>
### 48. Performance Qualification Assessment (`pq_assessment`)

| Item | Definition |
| --- | --- |
| Description | Project-specific PQ test grouping and document revision information. Distinguishes protocol approval status from execution-result approval status. |
| Primary Key | `pq_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 772 | PQ ID | `pq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | PQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 773 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 774 | PQ Number | `pq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | PQ document number. Retain the same number across revision rows of the same document. | `PQ-VP-SYS-008-20260422` |
| 775 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Performance Qualification document title | `Performance Qualification` |
| 776 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | PQ assessment document display version (e.g., v1.0, v1.1) | `v1.0` |
| 777 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | PQ assessment document revision sequence (e.g., 1, 2, 3...) | `1` |
| 778 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the PQ assessment document | `최초 작성` |
| 779 | Current Latest Version Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest PQ assessment document version (TRUE/FALSE) | `TRUE` |
| 780 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | PQ protocol approval summary. DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). Aggregates individual item/execution statuses without overwriting child rows en masse. | `DRAFT` |
| 781 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | PQ execution-result approval summary. DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). Aggregates individual item/execution statuses without overwriting child rows en masse. | `DRAFT` |
| 782 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 783 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | PQ author identifier | `00000000-0000-0000-0000-000000000001` |
| 784 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 785 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | PQ modifier identifier | `00000000-0000-0000-0000-000000000001` |
| 786 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |
| 787 | Test Composition per Assessment Revision | `item_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | Array of [{table_name:"pq_item",record_id,version,sort_order}]. Identifies the complete test composition and order for this assessment revision. PKs of unchanged tests from prior assessment revisions may be reused. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_pq_assessment_1` | UNIQUE | `project_id, pq_no, revision_number` | UNIQUE(project_id, pq_no, revision_number) |
| `uq_pq_assessment_1_rule` | Business validation | `project_id, pq_no, revision_number, is_current_version` | At most one row per document may have is_current_version = TRUE. |
| `rule_pq_assessment_2` | Business validation | - | Distinguish document revisions from test item revisions. Derive the document screen's approval summary from child protocols and execution results. |
| `uq_pq_assessment_current` | UNIQUE | `project_id, pq_no` | UNIQUE (project_id, pq_no) WHERE is_current_version=TRUE |

#### Business Rules

Manages test lists, protocol approval, result approval, and document generation information. Manage document content in deliverable_document / deliverable_revision / deliverable_section.

When only the deliverable title, body, or output format changes, create only a new deliverable_revision and retain the assessment row and test items. If the assessment scope, parent business information, or test composition changes, create a new assessment revision row and fix the complete composition at that time in item_revision_refs. Reference unchanged test items using the same PK/version; do not clone them or move their parent FK. Only items whose test protocol content changes receive a new revision row retaining the same item_key and are included in the new composition.

Composition targets must be actual revisions of pq_item belonging to the same project and PQ stage. The item's original parent assessment and this assessment must have the same pq_no. Do not include different revisions of the same item_key in one composition; record_id and sort_order must also be unique. The composition is the complete ordered list, locked when a document is first submitted. Do not modify or delete compositions referenced by submitted documents or historical signatures. Do not automatically include rows outside the composition solely on the basis of their original parent FK. Derive protocol/result approval summaries from items and executions belonging to that composition.

[↑ Back to Top](#top)

---

<a id="table-pq_item"></a>
### 49. PQ Detailed Test Item (`pq_item`)

| Item | Definition |
| --- | --- |
| Description | Manages PQ test item protocol revisions, content, expected results, acceptance criteria, and registration sources |
| Primary Key | `pq_item_id` |
| Key References (FK) | `pq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_signature_id, disposed_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 788 | PQ Item ID | `pq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | PK of a specific protocol revision row for an PQ test item. Retain the same item_key across revisions. | `00000000-0000-0000-0000-000000000001` |
| 789 | PQ ID | `pq_id` | `uuid` | N | Y | `pq_assessment.pq_id` | Y | - | N | N | N | Y | Parent assessment revision in which this logical test was first registered. Retain the same original parent in subsequent item revisions. Retrieve the actual composition per assessment revision from item_revision_refs. | `00000000-0000-0000-0000-000000000001` |
| 790 | Test Display ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Test display ID on the screen. Retain the same value across revisions of the same item; prohibit duplication across different item_key values within the project. | `PQ-NEW-01` |
| 791 | Test Item Name | `test_case` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Test item title shown on the screen. | `업무 시나리오 검증` |
| 792 | Test Item Logical ID | `item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | Identifier grouping revisions of the same test item. Issue once for a new item and retain on revision. | `00000000-0000-0000-0000-000000000001` |
| 793 | Test Description | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed procedure for performing the PQ test verification | `연속 3배치 이상 생산 공정 정상 완료 검증` |
| 794 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Test success criteria and expected result | `모든 배치가 사양에 맞게 정상 생산 완료되어야 함` |
| 795 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 796 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Item author identifier | `00000000-0000-0000-0000-000000000001` |
| 797 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 798 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Item modifier identifier | `00000000-0000-0000-0000-000000000001` |
| 799 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |
| 800 | Protocol Version | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | Protocol revision version displayed on the screen. | `ver1` |
| 801 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Revision sequence within item_key. At least 1. | `1` |
| 802 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for change entered when revising an approved protocol. | - |
| 803 | Latest Revision Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the latest working revision for the same item_key. Separate from approved-version status. | `TRUE` |
| 804 | Protocol Approval Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected) | `DRAFT` |
| 805 | Acceptance Criteria | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | Acceptance criteria on the screen. Distinct from expected_result (expected result). | `승인된 사양과 일치` |
| 806 | Registration Source | `source_type` | `varchar(20)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL (Direct Registration), LIBRARY (Library), PACKAGE (System Package), AI (AI Draft). | `MANUAL` |
| 807 | Source Library ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | Original source when registered from a library. Test content is copied at registration and managed independently thereafter. | `00000000-0000-0000-0000-000000000001` |
| 808 | Source Package Code | `source_package_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Package code when registered from a system package. | `IQ-CORE` |
| 809 | Source Package Version | `source_package_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Package version selected at registration. | `ver2` |
| 810 | Source Template Code | `source_template_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Code of the test template selected within the package. | `IQ-PKG-001` |
| 811 | Item Approval Number | `approval_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Issued at initial approval and displayed as the item number. Retain the same logical item number on revision. | - |
| 812 | Protocol Approval Workflow | `protocol_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Authoring, review, and approval route and history for this protocol revision. | `00000000-0000-0000-0000-000000000001` |
| 813 | Disposal Reason | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason entered on the disposal approval screen. Required when disposal is completed. | - |
| 814 | Disposal Approval Signature | `disposal_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature provided by an authorized disposer after confirming the reason and target. | `00000000-0000-0000-0000-000000000001` |
| 815 | Disposed Flag | `is_disposed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Disposal status shown on the screen. Disposed items are excluded from new executions and aggregation; existing records are preserved. | `FALSE` |
| 816 | Disposed By | `disposed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 817 | Disposed At | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when disposal was completed on the screen. | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_pq_item_1` | UNIQUE | `item_key, revision_number` | UNIQUE(item_key, revision_number) |
| `uq_pq_item_1_rule` | Business validation | `item_key, revision_number, is_current_version` | Rows with the same item_key belong to the same project and PQ stage. At most one row per item_key may have is_current_version = TRUE. |
| `rule_pq_item_2` | Business validation | - | protocol_status is DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), or REJECTED (Rejected). Approved protocols and steps are not edited directly; a new revision row is created. |
| `rule_pq_item_3` | Business validation | - | approval_number is the initial approval number of the logical item and is retained across revisions. Approval and disposal signatures are preserved in workflow_instance / approval_action / electronic_signature for the corresponding item revision. |
| `rule_pq_item_4` | Business validation | `source_library_id, source_package_code, source_package_version, source_template_code` | LIBRARY registration requires source_library_id. PACKAGE registration requires source_package_code / source_package_version / source_template_code. |
| `rule_pq_item_5` | Business validation | - | Links to multiple URS items are managed through REQUIREMENT → PQ_ITEM / VERIFIED_BY in traceability_link. |
| `rule_pq_item_6` | Business validation | - | Registration of an active test and protocol approval require at least one linked URS revision and at least one detailed step with nonempty content. |
| `rule_pq_item_7` | Business validation | `disposal_reason, disposal_signature_id, is_disposed, disposed_by, disposed_at` | When is_disposed = TRUE, disposal_reason, disposal_signature_id, disposed_by, and disposed_at are required. The actual target table and PK of the disposal signature must identify this test revision, and the signer and timestamp must match disposed_by / disposed_at. |
| `rule_pq_item_8` | Business validation | - | Disposal applies to the active status of the same item_key, while historical approval records are preserved. Adding disposal status, reason, and signature metadata is distinct from a revision that changes approved test content or steps; the originally approved content remains unchanged. |
| `uq_pq_item_current` | UNIQUE | `item_key` | UNIQUE (item_key) WHERE is_current_version=TRUE |

#### Business Rules

Test content is stored in test_description, expected results in expected_result, and acceptance criteria in acceptance_criteria. Repeated steps are managed in pq_step, while actual results, signatures, and rerun history are managed in pq_execution.

All revisions with the same item_key belong to the same project and business type. Item numbers approval_number and test_id are retained in subsequent revisions after initial assignment. Unapproved drafts may have NULL for numbers assigned upon approval. A number must not be reused for another logical key within the same project and business type, or reassigned to another item after disposal. Number allocation and revision creation are serialized using concurrency locks on the project and logical item, and duplication, ownership, and number retention conditions are validated in the same transaction.

[↑ Back to Top](#top)

---

<a id="table-pq_step"></a>
### 50. PQ Test Step (`pq_step`)

| Item | Definition |
| --- | --- |
| Description | Ordered detailed steps included in a PQ protocol revision |
| Primary Key | `step_id` |
| Key References (FK) | `pq_item_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 818 | Step ID | `step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 819 | Test Revision ID | `pq_item_id` | `uuid` | N | Y | `pq_item.pq_item_id` | Y | - | N | Y | N | Y | References pq_item.pq_item_id | `00000000-0000-0000-0000-000000000001` |
| 820 | Step Order | `step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display order of detailed steps on the screen. Must be at least 1. | `1` |
| 821 | Step Content | `instruction` | `text` | N | N | - | Y | - | N | N | N | Y | Step content added, edited, or deleted on the screen. | - |
| 822 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 823 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 824 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 825 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_pq_step_1` | UNIQUE | `pq_item_id, step_order` | UNIQUE(pq_item_id, step_order) |
| `uq_pq_step_1_rule` | Business validation | `pq_item_id, step_order` | step_order >= 1 |
| `rule_pq_step_2` | Business validation | - | When the parent protocol is APPROVED, its step structure and content must not be changed. Changes are recorded as a new test revision and its child steps. |

#### Business Rules

Step completion and attached evidence are execution-attempt records in pq_step_execution, rather than design values in this table.

[↑ Back to Top](#top)

---

<a id="table-pq_execution"></a>
### 51. PQ Test Execution (`pq_execution`)

| Item | Definition |
| --- | --- |
| Description | Management of actual results, outcomes, execution signatures, and result approval for each execution attempt of a PQ test revision |
| Primary Key | `execution_id` |
| Key References (FK) | `pq_item_id, executed_by, execution_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 826 | Execution ID | `execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 827 | Executed Protocol Revision ID | `pq_item_id` | `uuid` | N | Y | `pq_item.pq_item_id` | Y | - | N | Y | N | Y | References pq_item.pq_item_id | `00000000-0000-0000-0000-000000000001` |
| 828 | Execution Attempt Number | `attempt_no` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 1 for the initial execution and 2 or higher for reruns. The rerun count shown on the screen is attempt_no - 1. | `1` |
| 829 | Result Correction Sequence | `record_revision` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Sequence number in the correction history of results for the same attempt. Distinct from a rerun. | `1` |
| 830 | Current Result Correction Flag | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether this is the currently valid result record for the same test revision and attempt. | `TRUE` |
| 831 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Actual result entered on the screen. Required when registering an outcome. | - |
| 832 | Execution Outcome | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | PASS, FAIL, or NA. Not executed is represented by NULL and is distinct from approval status. | `PASS` |
| 833 | Execution Date | `performed_on` | `date` | N | N | - | N | - | N | N | N | Y | Execution date selected on the screen. Distinct from the electronic signature timestamp. | `2026-09-16` |
| 834 | Executed By | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 835 | Result Registration Timestamp | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the outcome was registered using an electronic signature. | `2026-09-01T00:00:00Z` |
| 836 | Execution Signature ID | `execution_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature provided when registering execution results. Passwords are not stored. | `00000000-0000-0000-0000-000000000001` |
| 837 | Execution Progress Status | `execution_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS (Performing steps / entering results), RECORDED (Outcome registered). Deviations and final approval are managed separately. | `IN_PROGRESS` |
| 838 | Result Approval Status | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). Managed independently of protocol approval status. | `DRAFT` |
| 839 | Result Approval Workflow | `result_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Final result review and approval route for the corresponding attempt and result correction. | `00000000-0000-0000-0000-000000000001` |
| 840 | Result Approval Version | `approval_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Display version of the finally approved test result. Distinct from the protocol version. | `ver1` |
| 841 | Deviation Reason | `deviation_reason` | `text` | N | N | - | N | - | N | N | N | Y | Snapshot of the deviation reason entered when registering a FAIL result. | - |
| 842 | Immediate Action | `immediate_action` | `text` | N | N | - | N | - | N | N | N | Y | Snapshot of the immediate action entered when registering a FAIL result. | - |
| 843 | Result Correction Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for correcting the result when record_revision is incremented. | - |
| 844 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 845 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 846 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 847 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 848 | Validation Target Baseline at Execution | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | The project's current baseline is fixed at the start of execution. Required before result registration and signing, and must not be overwritten by subsequent project baseline changes. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_pq_execution_1` | UNIQUE | `pq_item_id, attempt_no, record_revision` | UNIQUE(pq_item_id, attempt_no, record_revision) |
| `uq_pq_execution_1_rule` | Business validation | `pq_item_id, attempt_no, record_revision, is_current_revision` | attempt_no / record_revision >= 1. At most one row for the same test revision and attempt may have is_current_revision = TRUE. |
| `rule_pq_execution_2` | Business validation | `actual_result, qualification_result, performed_on, executed_by, executed_at, execution_signature_id` | Execution is allowed only for an approved protocol. Transition to RECORDED requires completion of all steps and actual_result / qualification_result / performed_on / executed_by / executed_at / execution_signature_id. |
| `rule_pq_execution_3` | Business validation | `qualification_result, deviation_reason, immediate_action` | When qualification_result = FAIL, deviation_reason and immediate_action are required, and a deviation is linked using the corresponding execution_id as its source. |
| `rule_pq_execution_4` | Business validation | - | A rerun starts with a new attempt_no while preserving existing results, step completion records, and evidence. Correcting results for the same attempt increments record_revision and does not overwrite prior approval records. |
| `rule_pq_execution_5` | Business validation | - | Final result approval is possible after protocol approval and execution result registration. Required approval and closure statuses of linked deviations are also checked. |
| `ck_pq_execution_baseline` | CHECK | `execution_status, system_baseline_id` | CHECK (execution_status <> 'RECORDED' OR system_baseline_id IS NOT NULL) |

#### Business Rules

Evidence for the entire test is linked to the PQ_EXECUTION target in evidence_link. The electronic signature targets this execution_id, and a new signature is obtained after a result correction.

system_baseline_id must belong to the same project as the test. Protocol approval and prerequisites are validated against the baseline adopted at the start of execution. A rerun links the baseline applicable at that time to a new execution record, while preserving historical execution baselines. A result correction is recorded as a new record_revision for the same logical test and attempt_no, retaining the baseline of the initial execution.

[↑ Back to Top](#top)

---

<a id="table-pq_step_execution"></a>
### 52. PQ Step Execution (`pq_step_execution`)

| Item | Definition |
| --- | --- |
| Description | Step completion and step evidence linkage unit for a PQ execution attempt |
| Primary Key | `step_execution_id` |
| Key References (FK) | `execution_id, step_id, confirmed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 849 | Step Execution ID | `step_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 850 | Execution ID | `execution_id` | `uuid` | N | Y | `pq_execution.execution_id` | Y | - | N | Y | N | Y | References pq_execution.execution_id | `00000000-0000-0000-0000-000000000001` |
| 851 | Step ID | `step_id` | `uuid` | N | Y | `pq_step.step_id` | Y | - | N | Y | N | Y | References pq_step.step_id | `00000000-0000-0000-0000-000000000001` |
| 852 | Step Completion Flag | `is_confirmed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Completion/cancellation value for a detailed step on the screen. | `FALSE` |
| 853 | Confirmed By | `confirmed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 854 | Confirmation Timestamp | `confirmed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the step was marked complete. | `2026-09-01T00:00:00Z` |
| 855 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 856 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 857 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 858 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_pq_step_execution_1` | UNIQUE | `execution_id, step_id` | UNIQUE(execution_id, step_id) |
| `uq_pq_step_execution_1_rule` | Business validation | `execution_id, step_id` | The step and execution must belong to the same protocol revision. |
| `rule_pq_step_execution_2` | Business validation | `is_confirmed, confirmed_by, confirmed_at` | When is_confirmed = TRUE, confirmed_by and confirmed_at are required. After result registration, completion/cancellation and evidence changes are locked, and historical records are preserved. |

#### Business Rules

Step attachments are linked through PQ_STEP_EXECUTION / step_execution_id in evidence_link. A new step execution row is created for a rerun.

[↑ Back to Top](#top)

---

## VSR

<a id="table-vsr_assessment"></a>
### 53. Validation Summary Report (`vsr_assessment`)

| Item | Definition |
| --- | --- |
| Description | Management of the final validation conclusion and summary report for each project |
| Primary Key | `vsr_id` |
| Key References (FK) | `project_id, created_by, updated_by, workflow_instance_id, approval_signature_id` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 859 | VSR ID | `vsr_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | VSR assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 860 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related Validation project ID | `00000000-0000-0000-0000-000000000001` |
| 861 | VSR Number | `vsr_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | VSR document number. Retained across revisions. | `VSR-VP-SYS-008-20260422` |
| 862 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Validation Summary Report title | `Validation Summary Report` |
| 863 | Validation Conclusion | `overall_conclusion` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Overall validation conclusion. Optional and distinct from business approval status. | - |
| 864 | Detailed Conclusion | `conclusion_remarks` | `text` | N | N | - | N | - | N | N | N | Y | Detailed explanation and conditions of the validation conclusion. NULL is allowed. | `OQ-GMP-02 일탈 해결 완료 후 최종 승인 가능` |
| 865 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | VSR assessment document display version (e.g., v1.0, v1.1) | `v1.0` |
| 866 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | VSR assessment document revision sequence (e.g., 1, 2, 3...) | `1` |
| 867 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for initial creation or revision of the VSR assessment document | `최초 작성` |
| 868 | Current Latest Version Indicator | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest VSR assessment document version (TRUE/FALSE) | `TRUE` |
| 869 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | VSR business confirmation status. DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). Distinct from the approval status of a separately generated deliverable. | `DRAFT` |
| 870 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 871 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | VSR creator identifier | `00000000-0000-0000-0000-000000000001` |
| 872 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 873 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | VSR updater identifier | `00000000-0000-0000-0000-000000000001` |
| 874 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |
| 875 | Source Fingerprint | `source_fingerprint` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | SHA-256 calculated from the project's selected activities, source revision references, and summary values. Used to identify records requiring reconfirmation after changes. | - |
| 876 | Aggregation Timestamp | `captured_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time when the current VSR activity summary was generated. | `2026-09-01T00:00:00Z` |
| 877 | VSR Confirmation Workflow | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Request, review, and approval route for VSR business confirmation. | `00000000-0000-0000-0000-000000000001` |
| 878 | VSR Confirmation Signature | `approval_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature upon completion of final VSR confirmation. | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_vsr_assessment_1` | UNIQUE | `project_id, vsr_no, revision_number` | UNIQUE(project_id, vsr_no, revision_number) |
| `uq_vsr_assessment_1_rule` | Business validation | `project_id, vsr_no, revision_number` | At most one latest revision is allowed for the same document. |
| `rule_vsr_assessment_2` | Business validation | - | The approved VSR header and child summary values are preserved. When source data changes, the summary is regenerated as DRAFT in a new VSR revision for confirmation. |
| `rule_vsr_assessment_3` | Business validation | - | The VSR aggregates only execution stages selected in the project and excludes RTM. Confirmation of the VSR itself is displayed in the header and is not stored recursively in detailed summaries. |

#### Business Rules

This record confirms the source status of the activity summary shown on the screen. Report content, PDF generation, and approval of the corresponding document are managed separately in deliverable_document / deliverable_revision.

[↑ Back to Top](#top)

---

<a id="table-vsr_item"></a>
### 54. VSR Activity Summary Item (`vsr_item`)

| Item | Definition |
| --- | --- |
| Description | Management of summaries of documents, results, deviations, and approval information for each validation activity |
| Primary Key | `vsr_item_id` |
| Key References (FK) | `vsr_id, created_by, updated_by, project_activity_id, document_revision_id` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 879 | VSR Item ID | `vsr_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | VSR activity summary item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 880 | VSR ID | `vsr_id` | `uuid` | N | Y | `vsr_assessment.vsr_id` | Y | - | N | N | N | Y | Parent VSR document identifier | `00000000-0000-0000-0000-000000000001` |
| 881 | Activity Type | `activity_code` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Aggregated activity codes: VP, VA, QIA, URS, FDS_GROUP (F&DS on the screen), FRA, DQ, IQ, OQ, PQ. RTM is excluded; the VSR's own status is displayed in the header. | `IQ` |
| 882 | Document Number | `doc_no` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Activity document number displayed at aggregation time. NULL if not approved or not generated. | `VP-SYS-008-20260422 · URS` |
| 883 | Revision Level | `revision_no` | `varchar(20)` | N | N | - | N | `'1'` | N | N | N | Y | Source approval version displayed at aggregation time. The specific revision identity is preserved in source_revision_refs. | `ver1` |
| 884 | Deviation Count | `deviation_info` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Display snapshot of the deviation count and open deviation status at aggregation time. Not edited manually and independently. | - |
| 885 | Conclusion/Status | `item_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | Activity approval status at aggregation time: DRAFT (Draft), REVIEW (Under Review), APPROVAL (Pending Approval), APPROVED (Approved), REJECTED (Rejected). A snapshot retrieved automatically from the source business status. | `APPROVED` |
| 886 | Approver Name | `approver_name` | `varchar(50)` | N | N | - | N | - | N | N | Y | Y | Snapshot of the approver's display name at the time of source approval. | `홍길동` |
| 887 | Approval Date | `approval_date` | `date` | N | N | - | N | - | N | N | N | Y | Snapshot of the approval date displayed on the screen at the time of source approval. The exact signature timestamp is retrieved from the source record. | `2024-02-20` |
| 888 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 889 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Item author identifier | `00000000-0000-0000-0000-000000000001` |
| 890 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00Z` |
| 891 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Item modifier identifier | `00000000-0000-0000-0000-000000000001` |
| 892 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |
| 893 | Project Activity ID | `project_activity_id` | `uuid` | N | Y | `project_activity.project_activity_id` | Y | - | N | Y | N | Y | Selected execution activity belonging to the same project as the VSR. | `00000000-0000-0000-0000-000000000001` |
| 894 | Deliverable Revision ID | `document_revision_id` | `uuid` | N | Y | `deliverable_revision.document_revision_id` | N | - | N | Y | N | Y | Referenced revision when a deliverable document has been generated. NULL is allowed for stages with only business item approval. | `00000000-0000-0000-0000-000000000001` |
| 895 | Source Revision Reference List | `source_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | List of exact business revisions supporting the aggregation: [{table_name, record_id, version}]. table_name is the actual table name. IQ/OQ/PQ results use execution_id in iq_execution / oq_execution / pq_execution; FDS/DDS use the revision PK fds_spec.fds_id / dds_spec.dds_id. An empty array is used for unapproved stages. | `[]` |
| 896 | Source Summary Snapshot | `source_snapshot` | `jsonb` | N | N | - | Y | `'{}'::jsonb` | N | N | Y | Y | Approval table columns / rows and approval information at aggregation time. Display values are fixed together with source revision references; an empty object is used for unapproved stages. | `{}` |
| 897 | Activity Source Fingerprint | `source_fingerprint` | `varchar(64)` | N | N | - | N | - | N | N | N | Y | Hash of the source references and snapshots for the stage. NULL if there is no unapproved source record. | - |
| 898 | Activity Display Order | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Order of the project's selected activities. | `1` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_vsr_item_1` | UNIQUE | `vsr_id, project_activity_id` | UNIQUE(vsr_id, project_activity_id) |
| `uq_vsr_item_1_rule` | Business validation | `vsr_id, project_activity_id` | activity_code must match the activity code of project_activity. |
| `rule_vsr_item_2` | Business validation | `project_activity_id, document_revision_id` | project_activity_id / document_revision_id / source_revision_refs must refer to the same project. The existence, type, revision, and approval status of the referenced sources are validated. |
| `rule_vsr_item_3` | Business validation | `item_status` | When item_status = APPROVED, source_revision_refs must not be empty, and source_snapshot and display values at the time of approval are preserved. |
| `rule_vsr_item_4` | Business validation | - | Deviation display values are aggregated from deviation records for the activity. |
| `rule_vsr_item_5` | Business validation | - | Detail rows are not modified after approval of the parent VSR. A new aggregation is recorded as detail rows of a new VSR revision. |

#### Business Rules

source_revision_refs is a list of polymorphic references containing actual table names, revision PKs, and versions, rather than physical FKs. It uses the same object structure as deliverable_revision.source_refs. Document file revisions and business confirmation status are not treated as the same thing.

[↑ Back to Top](#top)

---

## Workflow

<a id="table-workflow_instance"></a>
### 55. Workflow Instance (`workflow_instance`)

| Item | Definition |
| --- | --- |
| Description | Management of review and approval workflow progress for each document |
| Primary Key | `workflow_instance_id` |
| Key References (FK) | `requested_by, created_by, updated_by, project_id, workflow_config_id` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 899 | Workflow Instance ID | `workflow_instance_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the review and approval workflow for a document | `UUID` |
| 900 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Name of the table subject to review and approval. In addition to business items and documents, system_asset, project_closure_request, and other supported targets are allowed. | `fds_spec` |
| 901 | Target Record ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | N | N | Y | PK of the target table row. This is a polymorphic reference rather than an actual FK; target existence, project, and version are validated together. | `UUID` |
| 902 | Target Document Version | `target_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Display version or revision number of the approval target. Corresponds to system_asset.revision_number for inventory and CLOSE-{request_version} for closure requests. | `v1.0` |
| 903 | Submitter ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who submitted the document for review and approval | `UUID` |
| 904 | Submission Timestamp | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time when the document was submitted for review and approval | `2026-08-31T15:00:00Z` |
| 905 | Workflow Status | `workflow_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | Overall workflow status. DRAFT, IN_PROGRESS, APPROVED, REJECTED, CANCELLED | `IN_PROGRESS` |
| 906 | Current Step Order | `current_step_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Order of the review or approval step currently being processed. At submission, the step_order of the first nondeleted step is stored; it is not fixed at 1. | `1` |
| 907 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the workflow ended through final approval, rejection, or cancellation | `2026-08-31T17:00:00Z` |
| 908 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-31T15:00:00Z` |
| 909 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Workflow record creator | `UUID` |
| 910 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-31T16:00:00Z` |
| 911 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Workflow record last updater | `UUID` |
| 912 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |
| 913 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Required for approval of project business activities. NULL for approvals outside a project, such as inventory. | `00000000-0000-0000-0000-000000000001` |
| 914 | Approval Type | `approval_scope` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | ITEM/PROTOCOL/RESULT/DOCUMENT/INVENTORY/CLOSURE/DISPOSAL/DEVIATION_ACTION/DEVIATION_COMPLETION/DEVIATION_CLOSE | `DOCUMENT` |
| 915 | Applied Approval Route Configuration ID | `workflow_config_id` | `uuid` | N | Y | `project_workflow_config.config_id` | N | - | N | Y | N | Y | Project approval route configuration used at submission. NULL for routes such as the inventory's own approval route. | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_workflow_instance_1` | UNIQUE | `target_table_name, target_record_id, target_version, approval_scope, workflow_status` | UNIQUE (target_table_name, target_record_id, target_version, approval_scope) WHERE workflow_status IN ('DRAFT','IN_PROGRESS') |

#### Business Rules

At submission, the configuration's steps, modes, primary assignees, and substitutes are copied to workflow_step and workflow_step_assignee. Subsequent changes to the default configuration do not change approval routes already in progress. Authors, reviewers, approvers, and substitutes must be assigned to distinct active accounts; inventory processing also verifies separation of authors, reviewers, and approvers.

PROTOCOL approval and execution RESULT approval are separated even for a single test item. Approval scope, target table, PK, and version are identified together. The author is recorded through requested_by and the SUBMIT action history.

DEVIATION_ACTION targets the PK of deviation_action_round and ACTION-{round_number}-{revision_number}. DEVIATION_COMPLETION and DEVIATION_CLOSE use the PK of deviation and the completion/closure versions defined for electronic signatures. When rejected or cancelled action content is revised and resubmitted, a new action revision and workflow are created.

[↑ Back to Top](#top)

---

<a id="table-workflow_step"></a>
### 56. Workflow Step (`workflow_step`)

| Item | Definition |
| --- | --- |
| Description | Management of processing status and assignees for each workflow step |
| Primary Key | `workflow_step_id` |
| Key References (FK) | `workflow_instance_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 916 | Workflow Step ID | `workflow_step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of a workflow review or approval step. Duplicate (workflow_instance_id, step_order) combinations are not allowed for rows where deleted_at IS NULL. | `UUID` |
| 917 | Workflow Instance ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | Y | - | N | Y | N | Y | Identifier of the workflow to which the step belongs | `UUID` |
| 918 | Step Order | `step_order` | `integer` | N | N | - | Y | - | N | Y | N | Y | Review and approval processing order within the same workflow | `1` |
| 919 | Step Type | `step_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Processing step type: REVIEW or APPROVE. SUBMIT/CANCEL are recorded as action_type in the action history, rather than as step types. | `REVIEW` |
| 920 | Step Name | `step_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Review or approval step name displayed on the screen | `품질 검토` |
| 921 | Execution Mode | `execution_mode` | `varchar(20)` | N | N | - | Y | `'SERIAL'` | N | N | N | Y | SERIAL = Serial, PARALLEL = Parallel. Processing order for multiple assignees within the same step. | `SERIAL` |
| 922 | Step Status | `step_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | Step status. PENDING, IN_PROGRESS, APPROVED, REJECTED, SKIPPED | `PENDING` |
| 923 | Due Date | `due_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Scheduled deadline for review or approval of the step | `2026-09-02T18:00:00Z` |
| 924 | Completion Timestamp | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the step was completed through approval, rejection, or another outcome | `2026-09-01T10:00:00Z` |
| 925 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-31T15:00:00Z` |
| 926 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Workflow step creator | `UUID` |
| 927 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-31T16:00:00Z` |
| 928 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Workflow step last updater | `UUID` |
| 929 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft-deletion timestamp | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_workflow_step_1` | UNIQUE | `workflow_instance_id, step_order, deleted_at` | UNIQUE (workflow_instance_id, step_order) WHERE deleted_at IS NULL |
| `ck_workflow_step_2` | CHECK | `execution_mode` | CHECK (execution_mode IN ('SERIAL','PARALLEL')) |
| `ck_workflow_step_3` | CHECK | `step_order` | CHECK (step_order >= 1) |

#### Business Rules

Primary assignees, substitutes, and assignee order are stored in workflow_step_assignee. step_order specifies the step sequence, while execution_mode distinguishes how assignees within the same step process their assignments.

A parallel step also requires all designated assignees to complete their actions before proceeding to the next step. Review steps record REVIEW actions, and approval steps record APPROVE actions.

[↑ Back to Top](#top)

---

<a id="table-workflow_step_assignee"></a>
### 57. Approval Step Assignee (`workflow_step_assignee`)

| Item | Definition |
| --- | --- |
| Description | Multiple primary and substitute assignees for review and approval steps |
| Primary Key | `assignment_id` |
| Key References (FK) | `workflow_step_id, assignee_id, substitute_user_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 930 | Assignment ID | `assignment_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 931 | Approval Step ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | Y | - | N | Y | N | Y | References workflow_step.workflow_step_id | `00000000-0000-0000-0000-000000000001` |
| 932 | Primary Assignee ID | `assignee_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 933 | Substitute Assignee ID | `substitute_user_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 934 | Assignee Order | `assignee_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Processing order within a serial step | `1` |
| 935 | Processing Status | `assignment_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | N | N | Y | PENDING/IN_PROGRESS/APPROVED/REJECTED/SKIPPED | `PENDING` |
| 936 | Completion Timestamp | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Processing completion timestamp value | `2026-09-01T00:00:00Z` |
| 937 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation timestamp value | `2026-09-01T00:00:00Z` |
| 938 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 939 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Modification timestamp value | `2026-09-01T00:00:00Z` |
| 940 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_workflow_step_assignee_1` | UNIQUE | `workflow_step_id, assignee_order` | UNIQUE (workflow_step_id, assignee_order) |
| `uq_workflow_step_assignee_2` | UNIQUE | `workflow_step_id, assignee_id` | UNIQUE (workflow_step_id, assignee_id) |
| `ck_workflow_step_assignee_3` | CHECK | `assignee_order` | CHECK (assignee_order >= 1) |
| `rule_workflow_step_assignee_4` | Business validation | - | The primary assignee and substitute assignee cannot be the same person. |

#### Business Rules

The actual actor is recorded in approval_action.actor_id. Either the primary assignee or an authorized substitute processes a given assignment.

In serial mode, an assignment can be processed after the preceding assignment is completed. In parallel mode, incomplete assignments in the current step can be processed.

[↑ Back to Top](#top)

---

<a id="table-approval_action"></a>
### 58. Approval Action History (`approval_action`)

| Item | Definition |
| --- | --- |
| Description | Management of actual action history for each step, including review, approval, and rejection |
| Primary Key | `approval_action_id` |
| Key References (FK) | `workflow_step_id, workflow_step_assignee_id, actor_id, signature_id` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 941 | Approval Action History ID | `approval_action_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of a review or approval action history record | `UUID` |
| 942 | Workflow Step ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | Y | - | N | Y | N | Y | Workflow step to which the action history belongs. SUBMIT is linked to the first nondeleted step; CANCEL to the current in-progress step (or the first step if processing has not started); and REVIEW/APPROVE/REJECT to the step actually processed. No history record is created if there is no step to which it can belong. | `UUID` |
| 943 | Action Assignment ID | `workflow_step_assignee_id` | `uuid` | N | Y | `workflow_step_assignee.assignment_id` | N | - | N | Y | N | Y | Assignment processed by REVIEW/APPROVE/REJECT. NULL for SUBMIT/CANCEL. | `00000000-0000-0000-0000-000000000001` |
| 944 | Action Type | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Type of action performed. SUBMIT, REVIEW, APPROVE, REJECT, CANCEL | `APPROVE` |
| 945 | Processed By ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who actually performed the review, approval, or rejection | `UUID` |
| 946 | Action Comment | `action_comment` | `text` | N | N | - | N | - | N | N | N | Y | Comment entered during review or approval | `검토 결과 이상 없음` |
| 947 | Rejection Reason | `rejection_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason entered when processing REJECT | `증적 파일 보완 필요` |
| 948 | Electronic Signature ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature record linked to the action. If a signature exists, signer_id must equal actor_id, and the target table, record, and version must match the workflow target. For SUBMIT/REVIEW/APPROVE/REJECT, the signature action must match action_type. CANCEL records a cancellation reason and Audit Trail without a signature. | `UUID` |
| 949 | Processed At | `acted_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when submission, review, approval, or rejection was actually processed | `2026-09-01T10:00:00Z` |
| 950 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-09-01T10:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_approval_action_1` | Business validation | - | workflow_step_assignee_id is required for REVIEW/APPROVE/REJECT. It is NULL for SUBMIT/CANCEL. |
| `rule_approval_action_2` | Business validation | - | Only one valid REVIEW/APPROVE/REJECT action history record may exist for an assignment. |
| `rule_approval_action_3` | Business validation | - | SUBMIT/REVIEW/APPROVE/REJECT require an electronic signature. CANCEL records a cancellation reason and audit record without an electronic signature. |
| `uq_approval_assignee_action` | UNIQUE | `workflow_step_assignee_id, action_type` | UNIQUE (workflow_step_assignee_id) WHERE action_type IN ('REVIEW','APPROVE','REJECT') |

#### Business Rules

The assignment's workflow_step_id must match the history record's workflow_step_id. The actual actor must be the primary assignee or substitute and must match the electronic signature's signer_id.

SUBMIT is recorded against the first step; CANCEL against the current step (or the first step if processing has not started). The signature target table, PK, version, and action must match workflow_instance and action_type.

A rejection for an assignment is not overwritten with an approval. Resubmission creates a new workflow or new steps and assignments. After locking the target step and assignment, the current progress status, unprocessed status, and substitute eligibility are checked again, and the signature, action history, step transitions, and overall status transition are saved atomically.

[↑ Back to Top](#top)

---

<a id="table-project_workflow_config"></a>
### 59. Project Approval Route Configuration (`project_workflow_config`)

| Item | Definition |
| --- | --- |
| Description | Project default and activity-specific author, reviewer, and approver route configuration |
| Primary Key | `config_id` |
| Key References (FK) | `project_id, project_activity_id, applied_signature_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 951 | Approval Route Configuration ID | `config_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 952 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 953 | Execution Activity ID | `project_activity_id` | `uuid` | N | Y | `project_activity.project_activity_id` | N | - | N | Y | N | Y | Specified for an activity-specific configuration. NULL for the project default configuration. | `00000000-0000-0000-0000-000000000001` |
| 954 | Configuration Version | `config_version` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Configuration version incremented when changes are made after application | `1` |
| 955 | Approval Route Definition | `route_definition` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Author stage (author_stage), review stage array (review_stages), and approval stage array (approval_stages). Each stage stores its order, SERIAL/PARALLEL mode, and array of primary/substitute assignees. | - |
| 956 | Applied Flag | `is_applied` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Whether the configuration is currently applied. When a new configuration is applied, the previous row is set to FALSE, preserving its application signature, timestamp, and original content. | `FALSE` |
| 957 | Application Signature ID | `applied_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | References electronic_signature.signature_id | `00000000-0000-0000-0000-000000000001` |
| 958 | Application Timestamp | `applied_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Application timestamp value | `2026-09-01T00:00:00Z` |
| 959 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation timestamp value | `2026-09-01T00:00:00Z` |
| 960 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 961 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Modification timestamp value | `2026-09-01T00:00:00Z` |
| 962 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_project_workflow_config_1` | Business validation | - | project_activity_id in an activity-specific configuration must belong to the same project_id. |
| `rule_project_workflow_config_2` | Business validation | `project_id, project_activity_id, config_version` | (project_id, config_version) must be unique for project default configurations, and (project_activity_id, config_version) must be unique for activity-specific configurations. At most one currently applied configuration is allowed for each project/activity scope. |
| `rule_project_workflow_config_3` | Business validation | `is_applied` | When is_applied=TRUE, applied_signature_id and applied_at are required. |
| `uq_workflow_config_project_version` | UNIQUE | `project_id, config_version, project_activity_id` | UNIQUE (project_id, config_version) WHERE project_activity_id IS NULL |
| `uq_workflow_config_activity_version` | UNIQUE | `project_activity_id, config_version` | UNIQUE (project_activity_id, config_version) WHERE project_activity_id IS NOT NULL |
| `uq_workflow_config_project_applied` | UNIQUE | `project_id, project_activity_id, is_applied` | UNIQUE (project_id) WHERE project_activity_id IS NULL AND is_applied=TRUE |
| `uq_workflow_config_activity_applied` | UNIQUE | `project_activity_id, is_applied` | UNIQUE (project_activity_id) WHERE project_activity_id IS NOT NULL AND is_applied=TRUE |

#### Business Rules

Use the activity-specific approval route if one exists; otherwise, use the project default route. No execution-stage configuration is created for RTM.

Stages in route_definition use the structure {step_order, execution_mode, assignees:[{user_id, substitute_user_id, assignee_order}]}. User identifiers within JSON are not physical FKs, so existence, permissions, and duplication are checked on save. Authors, reviewers, approvers, and substitutes must be distinct active individual accounts with project permissions.

The author stage specifies candidate authors. The actual submitter is recorded in workflow_instance.requested_by. The execution approval route is copied to workflow_step and workflow_step_assignee.

Switching the applied configuration requires locking the project. Clearing the previous is_applied flag and applying the new configuration are saved in the same transaction. is_applied is a management status and is excluded from the original CONFIG_APPLY signed payload. An already applied route_definition, config_version, signature, and application timestamp must not be changed. Activity-specific configurations must reference project_activity in the same project.

[↑ Back to Top](#top)

---

## Traceability

<a id="table-traceability_link"></a>
### 60. Common Traceability Link (`traceability_link`)

| Item | Definition |
| --- | --- |
| Description | Management of links selected on the screen among URS, design documents, DQ, FRA, and test items, and the source relationships used by the dashboard RTM |
| Primary Key | `traceability_link_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 963 | Traceability Link ID | `traceability_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of a traceability relationship among URS, FDS, DDS, DQ, FRA, IQ, OQ, and PQ items. Nondeleted data must not contain duplicate (project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type) combinations. | `UUID` |
| 964 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Project ID to which the traceability relationship belongs | `UUID` |
| 965 | Source Entity Type | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Linked target types: REQUIREMENT, FDS_SPEC, DDS_SPEC, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM. Links identify the PK of a specific revision row of each type. | `REQUIREMENT` |
| 966 | Source Entity ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | PK of a specific revision row in the table corresponding to the target type. A polymorphic reference rather than a physical FK; must not be replaced with a logical key or display number. | `UUID` |
| 967 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Linked target types: REQUIREMENT, FDS_SPEC, DDS_SPEC, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM. Links identify the PK of a specific revision row of each type. | `OQ_ITEM` |
| 968 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | PK of a specific revision row in the table corresponding to the target type. A polymorphic reference rather than a physical FK; must not be replaced with a logical key or display number. | `UUID` |
| 969 | Link Type | `link_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | IMPLEMENTED_BY, ASSESSED_BY, VERIFIED_BY, MITIGATED_BY | `VERIFIED_BY` |
| 970 | Link Rationale | `link_reason` | `text` | N | N | - | N | - | N | N | N | Y | Business rationale for linking the two deliverable items | `URS 요구사항을 IQ 시험으로 검증` |
| 971 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Traceability link creation timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 972 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who created the traceability link | `UUID` |
| 973 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Traceability link last update timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 974 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who last updated the traceability link | `UUID` |
| 975 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Traceability link soft deletion timestamp | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_traceability_link_1` | UNIQUE | `project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type` | UNIQUE(project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type) WHERE deleted_at IS NULL |
| `rule_traceability_link_2` | Business validation | - | Both ends of a link must belong to the same project and exist as registered target types and actual revision rows. |
| `rule_traceability_link_3` | Business validation | - | VERIFIED_BY from REQUIREMENT → IQ_ITEM / OQ_ITEM / PQ_ITEM is the relationship selected on the screen to link multiple URS items to one test. |
| `rule_traceability_link_4` | Business validation | - | FDS_SPEC / DDS_SPEC mean fds_spec.fds_id / dds_spec.dds_id. |
| `rule_traceability_link_5` | Business validation | - | The dashboard RTM retrieves link relationships and currently valid source and execution statuses. No additional RTM-specific documents, tests, or approval statuses are created. |
| `rule_traceability_link_6` | Business validation | - | When revising a link, the original relationships of historically approved targets must not be deleted or overwritten. New relationships are created for the new revision, and historical links are preserved. |

#### Business Rules

Coverage is aggregated based on whether URS items are linked to tests and is distinct from the test PASS rate or approval completion rate. Direct URS links for FDS/DDS documents are stored only for relationships selected on the file approval screen.

[↑ Back to Top](#top)

---

## Deviation

<a id="table-deviation"></a>
### 61. Deviation Management (`deviation`)

| Item | Definition |
| --- | --- |
| Description | Management of reasons and immediate actions following test FAIL results, reruns, corrective and preventive actions, completion reports, and closure processing |
| Primary Key | `deviation_id` |
| Key References (FK) | `project_id, approved_by, created_by, updated_by, action_workflow_id, action_signature_id, completion_signature_id, closure_signature_id, current_action_round_id` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 976 | Deviation ID | `deviation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique deviation identifier | `UUID` |
| 977 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Validation project in which the deviation occurred | `UUID` |
| 978 | Source Entity Type | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Type of the original failed execution: IQ_EXECUTION, OQ_EXECUTION, PQ_EXECUTION. | `OQ_EXECUTION` |
| 979 | Source Entity ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | execution_id of the corresponding execution table. A polymorphic reference to the result of a specific attempt and correction, rather than a physical FK. | `UUID` |
| 980 | Deviation Number | `deviation_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Deviation management number within the project | `DEV-001` |
| 981 | Deviation Title | `title` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Report display title based on the test item name and deviation number. Not a separate required input. | `예상 결과 불일치` |
| 982 | Deviation Reason | `description` | `text` | N | N | - | Y | - | N | N | N | Y | Display summary of the deviation reason in the current action revision. The original failure reason is retrieved from the execution record identified by source_entity_id. | `OQ 수행 중 예상 결과와 실제 결과 불일치` |
| 983 | Deviation Status | `deviation_status` | `varchar(30)` | N | N | - | Y | `'ACTION_PENDING'` | N | Y | N | Y | ACTION_PENDING (Reason/action approval pending), RERUN_ALLOWED (Rerun allowed), COMPLETION_PENDING (Completion report pending), CLOSED (Closed). | `ACTION_PENDING` |
| 984 | Approved At | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Final signature timestamp for completion report approval or closure with an entered reason. | `2026-09-03T17:00:00Z` |
| 985 | Approver ID | `approved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Final signing user for completion report approval or closure processing. | `UUID` |
| 986 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Deviation creation timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 987 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the deviation | `UUID` |
| 988 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Deviation last update timestamp (UTC) | `2026-09-01T10:00:00Z` |
| 989 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last updated the deviation | `UUID` |
| 990 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deviation soft deletion timestamp | `2026-09-01T00:00:00Z` |
| 991 | Immediate Action | `immediate_action` | `text` | N | N | - | Y | - | N | N | N | Y | Display summary of the immediate action in the current action revision. The approved original content for each round is preserved in deviation_action_round. | - |
| 992 | Reason/Action Approval Status | `action_approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | Summary of the current action revision's approval status. DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED. Not edited independently. | `DRAFT` |
| 993 | Reason/Action Approval Workflow | `action_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Current approval workflow synchronized from workflow_instance_id of current_action_round_id. Previous rounds are retrieved from deviation_action_round; this value is not edited independently. | `00000000-0000-0000-0000-000000000001` |
| 994 | Reason/Action Final Signature | `action_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Final approval signature synchronized from signature_id of the current action round. Historical signatures are preserved for each round. | `00000000-0000-0000-0000-000000000001` |
| 995 | Latest Rerun ID | `rerun_execution_id` | `uuid` | N | N | - | N | - | N | N | N | Y | PK of the valid result revision of the latest rerun. Must belong to the same logical test and attempt_no as the round's initial rerun. The table is identified by source_entity_type. | `00000000-0000-0000-0000-000000000001` |
| 996 | Corrective and Preventive Actions | `corrective_action` | `text` | N | N | - | N | - | N | N | N | Y | Corrective and preventive actions entered when preparing the completion report on the screen. | - |
| 997 | Completion Report | `completion_report` | `text` | N | N | - | N | - | N | N | N | Y | Completion report entered on the screen after checking the rerun results. | - |
| 998 | Completion Report Approval Signature | `completion_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature approving the corrective and preventive actions and completion report after review. | `00000000-0000-0000-0000-000000000001` |
| 999 | Closure Reason | `closure_reason` | `text` | N | N | - | N | - | N | N | N | Y | Required reason entered on the screen when closing without a rerun. | - |
| 1000 | Closure Signature | `closure_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature for closure with a reason and without a rerun. Distinct from completion report approval. | `00000000-0000-0000-0000-000000000001` |
| 1001 | Current Deviation Action Revision | `current_action_round_id` | `uuid` | N | Y | `deviation_action_round.action_round_id` | N | - | N | Y | N | Y | Latest content revision in this deviation's last action round. Required at the end of the transaction that creates the initial failure and first action draft. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_deviation_1` | UNIQUE | `project_id, deviation_no` | UNIQUE(project_id, deviation_no) |
| `uq_deviation_1_rule` | Business validation | `project_id, source_entity_id, deviation_no` | source_entity_id and rerun_execution_id must belong to the same project and logical test. |
| `rule_deviation_2` | Business validation | - | Transition to RERUN_ALLOWED requires reason/action status APPROVED and action_signature_id. |
| `rule_deviation_3` | Business validation | - | The deviation's source_entity_id retains the initial failure record. If a rerun results in FAIL, the latest execution is linked and a new reason/action approval history is added. |
| `rule_deviation_4` | Business validation | `corrective_action, completion_report, completion_signature_id` | When closing through completion report approval after COMPLETION_PENDING, corrective_action / completion_report / completion_signature_id are required. Immediately before completion report approval, verify that the valid result of the latest rerun is PASS. |
| `rule_deviation_5` | Business validation | `closure_reason, closure_signature_id` | Closure without a rerun requires RERUN_ALLOWED status and closure_reason / closure_signature_id. The two closure paths are distinguished, and historical signatures and results are not deleted. |
| `rule_deviation_6` | Business validation | `approved_by, completion_signature_id, closure_signature_id` | When CLOSED, exactly one of completion_signature_id / closure_signature_id, plus approved_by / approved_at, is required. The latter must match the final signer and signature timestamp. |
| `rule_deviation_7` | Business validation | - | The rerun outcome is retrieved from qualification_result of rerun_execution_id. The original failure reason and immediate action can be reproduced from the values recorded in the execution record at that time. |

#### Business Rules

The closure actor and timestamp must match the final electronic signature values.

Initial FAIL registration and creation of the first action round are handled in one transaction. current_action_round_id must belong to this deviation and identify the row with is_current_revision=TRUE in the last round_number. The current reason, action, approval status, workflow, and signature are synchronized from that row; historical original content is retrieved from round rows. If the latest rerun results in FAIL, the existing deviation is retained and the next action round is created.

Completion report and closure-without-execution signatures target the deviation PK and follow the electronic signature version rules. signed_payload fixes current_action_round_id, the initial failure PK, the exact result revision PK of the latest rerun, and the completion report or closure reason. Closure through a completion report requires the latest rerun to be PASS. After CLOSED, the reason, action, completion report, signatures, and result references must not be changed.

[↑ Back to Top](#top)

---

<a id="table-deviation_action_round"></a>
### 62. Deviation Action Round (`deviation_action_round`)

| Item | Definition |
| --- | --- |
| Description | Linkage of action content revisions and approvals for each failed execution to the authorized rerun |
| Primary Key | `action_round_id` |
| Key References (FK) | `deviation_id, workflow_instance_id, signature_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the auditable period. Do not delete approved content or historical link records. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1002 | Deviation Action Revision ID | `action_round_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Identifier of a specific content revision created within an action round | - |
| 1003 | Deviation ID | `deviation_id` | `uuid` | N | Y | `deviation.deviation_id` | Y | - | N | Y | N | Y | Source Deviation | - |
| 1004 | Action Round | `round_number` | `integer` | N | N | - | Y | - | N | N | N | Y | 1 for the initial failure. If a rerun after approval fails, the next round is created. | `1` |
| 1005 | Content Revision Within the Round | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Incremented when content for the same failure is modified and resubmitted after rejection. | `1` |
| 1006 | Current Round Revision Flag | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Only one latest working revision exists for the same deviation and action round. | - |
| 1007 | Failed Execution ID for the Round | `failed_execution_id` | `uuid` | N | N | - | Y | - | N | N | N | Y | Execution PK corresponding to deviation.source_entity_type. The first round refers to the initial failure; subsequent rounds refer to the failed rerun of the preceding round. A polymorphic reference rather than a physical FK. | - |
| 1008 | Deviation Reason for the Round | `description` | `text` | N | N | - | Y | - | N | N | N | Y | Original reason content subject to approval for the failure | - |
| 1009 | Immediate Action for the Round | `immediate_action` | `text` | N | N | - | Y | - | N | N | N | Y | Original content of the action performed for the failure | - |
| 1010 | Action Approval Status | `approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED | - |
| 1011 | Action Approval Workflow | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | DEVIATION_ACTION approval targeting this action_round_id and ACTION-{round_number}-{revision_number} | - |
| 1012 | Final Action Approval Signature | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Final approval signature for the corresponding action content revision | - |
| 1013 | Authorized Rerun ID | `rerun_execution_id` | `uuid` | N | N | - | N | - | N | N | N | Y | PK of the single rerun started under this approval. The table is identified by source_entity_type. NULL if closed without a rerun. | - |
| 1014 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Content revision creation timestamp | - |
| 1015 | Author | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Author of the action content | - |
| 1016 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time of draft editing or management status change | - |
| 1017 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Processed By | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_deviation_action_revision` | UNIQUE | `deviation_id, round_number, revision_number` | UNIQUE (deviation_id, round_number, revision_number) |
| `uq_deviation_action_current` | UNIQUE | `deviation_id, round_number, is_current_revision` | UNIQUE (deviation_id, round_number) WHERE is_current_revision=TRUE |
| `uq_deviation_action_approved` | UNIQUE | `deviation_id, round_number, approval_status` | UNIQUE (deviation_id, round_number) WHERE approval_status='APPROVED' |
| `ck_deviation_action_sequence` | CHECK | `round_number, revision_number` | CHECK (round_number >= 1 AND revision_number >= 1) |
| `ck_deviation_action_status` | CHECK | `approval_status` | CHECK (approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_deviation_action_approval` | CHECK | `approval_status, workflow_instance_id, signature_id` | CHECK (approval_status <> 'APPROVED' OR (workflow_instance_id IS NOT NULL AND signature_id IS NOT NULL)) |
| `ck_deviation_action_rerun` | CHECK | `rerun_execution_id, approval_status` | CHECK (rerun_execution_id IS NULL OR approval_status='APPROVED') |
| `rule_deviation_action_chain` | Business validation | `failed_execution_id, rerun_execution_id` | Failed executions and reruns belong to the same project, stage, and logical test as the deviation. The first failure is deviation.source_entity_id. A subsequent round's failure is the rerun of the preceding approved round, or a corrected result revision belonging to the same protocol and attempt_no as that rerun, and must be FAIL. Revisions within the same round retain failed_execution_id. |

#### Business Rules

Action content is locked after submission. When changing content after rejection or cancellation, a new revision_number and PK are created under the same round_number, preserving the previous row, workflow, and signatures. No new content revision is created for an approved round. A rerun FAIL is linked to the next round_number.

The approval workflow and signature refer to the exact action_round_id and ACTION-{round_number}-{revision_number} in this table, rather than to deviation. The original reason/action content and failed_execution_id are included in the signed payload. After approval, the action content, failure reference, approval status, and final signature must not be changed. Only the initial rerun linkage and management metadata, such as the updater and update timestamp, may subsequently be updated. After its initial linkage, rerun_execution_id must not be overwritten with another attempt.

REVIEW, APPROVAL, and APPROVED require workflow_instance_id. The approval workflow's target, scope, and version, and the final signature and signed payload, must match this action revision.

Rerun creation locks the deviation and approved round, then checks approval status, the current round, and the absence of an existing rerun. Test execution creation, round linkage, and deviation summary updates are handled in one transaction. An execution correction uses a new execution PK while retaining the original rerun link; the correction lineage is retrieved through record_revision for the same attempt_no. Determination of the next failure and the completion report explicitly identify the result revision valid at that time.

[↑ Back to Top](#top)

---

## Document

<a id="table-deliverable_document"></a>
### 63. Deliverable Document (`deliverable_document`)

| Item | Definition |
| --- | --- |
| Description | Document numbers and persistent identification of deliverables for each activity |
| Primary Key | `document_id` |
| Key References (FK) | `project_id, project_activity_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1018 | Document ID | `document_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1019 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 1020 | Execution Activity ID | `project_activity_id` | `uuid` | N | Y | `project_activity.project_activity_id` | Y | - | N | Y | N | Y | References project_activity.project_activity_id | `00000000-0000-0000-0000-000000000001` |
| 1021 | Document Type | `document_type` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | STAGE_DELIVERABLE / PROTOCOL / RECORD / SUMMARY | `RECORD` |
| 1022 | Document Number | `document_number` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Document number on the deliverable screen. NULL is allowed before approval. | `OQ-R-001` |
| 1023 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1024 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1025 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1026 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_deliverable_document_1` | Business validation | - | The document's project_activity_id must belong to the same project_id. RTM is excluded from independent execution activities and required deliverables. |
| `rule_deliverable_document_2` | Business validation | - | If a document number exists, it is managed uniquely within the project. Test protocol and result documents are distinguished by document type. |

[↑ Back to Top](#top)

---

<a id="table-deliverable_revision"></a>
### 64. Deliverable Revision (`deliverable_revision`)

| Item | Definition |
| --- | --- |
| Description | Table of contents and body editing, version-specific viewing, author signatures, and document approval targets |
| Primary Key | `document_revision_id` |
| Key References (FK) | `document_id, workflow_instance_id, approved_by, file_id, created_by, updated_by, system_baseline_id` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1027 | Document Revision ID | `document_revision_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1028 | Document ID | `document_id` | `uuid` | N | Y | `deliverable_document.document_id` | Y | - | N | Y | N | Y | References deliverable_document.document_id | `00000000-0000-0000-0000-000000000001` |
| 1029 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Document Title | `운전 적격성 평가 결과서` |
| 1030 | Document Display Version | `version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Document Display Version | `v1.0` |
| 1031 | Revision Sequence | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision Sequence | `1` |
| 1032 | Revision Reason | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Revision Reason | `시험 결과 반영` |
| 1033 | Current Working Version Flag | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Current Working Version Flag | `TRUE` |
| 1034 | Document Approval Status | `approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED | `DRAFT` |
| 1035 | Document Approval Workflow ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | References workflow_instance.workflow_instance_id | `00000000-0000-0000-0000-000000000001` |
| 1036 | Supporting Business Approval Version | `source_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Approved data version shown on the screen. Distinct from the document's own version. | `v1.0` |
| 1037 | Supporting Business Revision List | `source_refs` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Array of [{table_name,record_id,version}]. Fixes the exact revisions of multiple business items and approved execution attempts. | `[{"table_name":"oq_execution","record_id":"00000000-0000-0000-0000-000000000001","version":"1-1"}]` |
| 1038 | Approved Data Table | `approved_table_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Approved data columns/rows shown on the screen. {columns:string[],rows:string[][]} | `{"columns":["항목","결과"],"rows":[["OQ-001","Pass"]]}` |
| 1039 | Part 11 Table | `part11_table_snapshot` | `jsonb` | N | N | - | N | - | N | N | N | Y | Table of Part 11 questions, responses, and outcomes for a QIA deliverable. NULL for other activities. | - |
| 1040 | Revision History Table | `revision_table_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Revision history columns/rows at the time of writing | `{"columns":["버전","사유"],"rows":[["v1.0","최초 작성"]]}` |
| 1041 | Activity Triggering Reapproval | `reapproval_source_stage` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Activity code when reapproval is required due to a change in an upstream activity. Excludes RTM. | `URS` |
| 1042 | Reapproval Reason | `reapproval_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reapproval Reason | `URS 개정으로 내용 재검토` |
| 1043 | Approval Invalidation Timestamp | `invalidated_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Approval Invalidation Timestamp | `2026-09-01T00:00:00Z` |
| 1044 | Final Approver ID | `approved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1045 | Final Approval Timestamp | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Final Approval Timestamp | `2026-09-01T00:00:00Z` |
| 1046 | PDF File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | Linked when the generated PDF is retained | `00000000-0000-0000-0000-000000000001` |
| 1047 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1048 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1049 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1050 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1051 | Document Validation Target Baseline | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | Baseline in the same project, fixed at document submission. Required at submission and approval. Previous executions also display their own baselines. | - |
| 1052 | Original Evaluation Questions and Decision Rules | `evaluation_definition_snapshot` | `jsonb` | N | N | - | N | - | N | N | N | Y | Array fixing the question/rule versions, actual definitions, inputs, and results for each supporting assessment of a QIA/FRA document. NULL for documents unrelated to assessments. | - |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_deliverable_revision_1` | UNIQUE | `document_id, revision_number` | UNIQUE(document_id,revision_number) |
| `uq_deliverable_revision_1_2` | UNIQUE | `document_id, version` | UNIQUE(document_id,version) |
| `uq_deliverable_revision_1_rule` | Business validation | `document_id, version, revision_number, is_current_version` | At most one row per document may have is_current_version=TRUE. |
| `rule_deliverable_revision_2` | Business validation | - | Document approval and approval of supporting business items are separate. source_refs validates the exact revisions, projects, and approval statuses of multiple source items and is preserved together with the tables as they existed at that time. |
| `rule_deliverable_revision_3` | Business validation | - | APPROVED requires document approval history, the final approver, and the approval timestamp. Approved content, tables, and supporting references are not changed; a new revision is prepared. |
| `rule_deliverable_revision_4` | Business validation | - | Historical approved versions are not deleted even if source data subsequently changes. Current validity is assessed separately using invalidated_at and the reapproval reason. |
| `uq_document_current_revision` | UNIQUE | `document_id` | UNIQUE (document_id) WHERE is_current_version=TRUE |

#### Business Rules

PDF generation status does not replace document approval status. The source VP table of contents is vp_section. Because the VP deliverable reflects the approved source, the same content must not be edited independently in both locations.

For IQ/OQ/PQ, source_refs records the exact PKs and versions of the assessment revision and all test items and executions included in the document. Protocol documents fix the full composition of the assessment's item_revision_refs, while result documents link the exact attempts and result revisions of executions belonging to that composition. When only the document is revised, the same assessment and test references may be reused unchanged. system_baseline_id and referenced targets must belong to the same project. If results executed under a different baseline are included, the execution baseline and rationale for reuse must be stated in the document body.

system_baseline_id is required at document submission and approval; QIA/FRA documents also require evaluation_definition_snapshot. This snapshot must enable recalculation of results from the inputs and actual version-specific definitions; a result table alone is insufficient. After submission, the body, composition, and question/rule snapshots are locked, and modifications after rejection are handled as a new document revision. The generated PDF is linked after file generation without changing the already signed business content; the integrity of the original PDF file is verified separately using the hash in file_asset.

[↑ Back to Top](#top)

---

<a id="table-deliverable_section"></a>
### 65. Deliverable Section/Content (`deliverable_section`)

| Item | Definition |
| --- | --- |
| Description | Sections added, edited, deleted, or reordered on the document editing screen |
| Primary Key | `section_id` |
| Key References (FK) | `document_revision_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1053 | Section ID | `section_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1054 | Document Revision ID | `document_revision_id` | `uuid` | N | Y | `deliverable_revision.document_revision_id` | Y | - | N | Y | N | Y | References deliverable_revision.document_revision_id | `00000000-0000-0000-0000-000000000001` |
| 1055 | Section Key | `section_key` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Section Key | `purpose` |
| 1056 | Section Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Section Title | `목적` |
| 1057 | Body | `content` | `text` | N | N | - | Y | - | N | N | N | Y | Body | `본 시험의 목적을 기술한다.` |
| 1058 | Display Order | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display Order | `1` |
| 1059 | Content Source | `source_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | MANUAL / AI / APPROVED / HISTORY / PART11 | `MANUAL` |
| 1060 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1061 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1062 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1063 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_deliverable_section_1` | UNIQUE | `document_revision_id, section_key` | UNIQUE(document_revision_id,section_key) |
| `uq_deliverable_section_1_2` | UNIQUE | `document_revision_id, sort_order` | UNIQUE(document_revision_id,sort_order) |
| `ck_deliverable_section_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_deliverable_section_2` | Business validation | - | MANUAL/AI content can be edited before approval. APPROVED/HISTORY/PART11 sections are displayed from their respective snapshots. Approved document sections must not be directly edited or deleted. |

[↑ Back to Top](#top)

---

## Report

<a id="table-report_generation"></a>
### 66. Report Generation Job (`report_generation`)

| Item | Definition |
| --- | --- |
| Description | Results of deliverable and audit PDF generation and downloads requested on the screen |
| Primary Key | `report_generation_id` |
| Key References (FK) | `project_id, requested_by, result_file_id, created_by, updated_by, document_revision_id` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain report generation results and execution history throughout the auditable retention period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1064 | Report Generation ID | `report_generation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique report generation job identifier | `UUID` |
| 1065 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Linked for a project-level report; NULL is allowed for system-wide or organization-level reports. | `UUID` |
| 1066 | Report Type | `report_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | DELIVERABLE / AUDIT_TRAIL. Deliverable or audit export from the screen. | `AUDIT_TRAIL` |
| 1067 | Report Name | `report_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Generated report name displayed to the user | `2026년 8월 Audit Trail 리포트` |
| 1068 | Query Period Start | `period_from` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Start timestamp for querying report source data. NULL is allowed for reports without a query period. | `2026-08-01T00:00:00Z` |
| 1069 | Query Period End | `period_to` | `timestamptz` | N | N | - | N | - | N | N | N | Y | End timestamp for querying report source data. Must not precede period_from. | `2026-08-31T23:59:59Z` |
| 1070 | Query Conditions | `report_parameters` | `jsonb` | N | N | - | N | - | N | N | Y | Y | Export filters selected on the screen, such as period, user, role, and menu | `{"action_types":["CREATE","UPDATE"]}` |
| 1071 | Output Format | `output_format` | `varchar(20)` | N | N | - | Y | `'PDF'` | N | Y | N | Y | PDF output format | `PDF` |
| 1072 | Generation Status | `generation_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | PENDING / PROCESSING / COMPLETED / FAILED / CANCELLED | `PENDING` |
| 1073 | Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who requested the PDF/export | `UUID` |
| 1074 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when the user requested the export | `2026-09-02T15:00:00Z` |
| 1075 | Execution Start Timestamp | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when generation started | `2026-09-02T15:00:05Z` |
| 1076 | Execution Completion Timestamp | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when generation success or failure was confirmed | `2026-09-02T15:01:30Z` |
| 1077 | Result File ID | `result_file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | Linked when the result is retained as a file asset. NULL is allowed if only a download is performed. | `UUID` |
| 1078 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | System error code classifying the cause of failure | `REPORT_FILE_CREATE_FAILED` |
| 1079 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Details of a report generation failure. Sensitive information such as passwords and tokens is not stored. | `결과 파일 저장 중 오류 발생` |
| 1080 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Report generation job record creation timestamp (UTC) | `2026-09-02T15:00:00Z` |
| 1081 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who requested the export on the screen | `UUID` |
| 1082 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Report generation job last update timestamp (UTC) | `2026-09-02T15:01:30Z` |
| 1083 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who updated the export processing result. NULL is allowed for automatic status updates. | `UUID` |
| 1084 | Deliverable Revision ID | `document_revision_id` | `uuid` | N | Y | `deliverable_revision.document_revision_id` | N | - | N | Y | N | Y | References deliverable_revision.document_revision_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_report_generation_1` | Business validation | `document_revision_id` | DELIVERABLE requires document_revision_id, which must match the project. For AUDIT_TRAIL, the query scope is fixed by the period and filters. |
| `rule_report_generation_2` | Business validation | `period_from, period_to` | If both period_from and period_to exist, period_from<=period_to. generation_status does not include an automatic retry waiting state. |

#### Business Rules

The body, table of contents, version, and document approval status are stored in deliverable_revision and deliverable_section. This table manages only export results.

[↑ Back to Top](#top)

---

## AI

<a id="table-ai_generation_job"></a>
### 67. AI Generation Job (`ai_generation_job`)

| Item | Definition |
| --- | --- |
| Description | AI draft generation requests, input conditions, and progress results from the screen |
| Primary Key | `ai_job_id` |
| Key References (FK) | `project_id, requested_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1085 | AI Job ID | `ai_job_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI generation job identifier | `UUID` |
| 1086 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Project Link | `UUID` |
| 1087 | Action Type | `job_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | ITEM_GENERATION, DOCUMENT_GENERATION | `ITEM_GENERATION` |
| 1088 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT / FRA_ITEM / IQ_ITEM / OQ_ITEM / PQ_ITEM / DELIVERABLE_REVISION | `IQ_ITEM` |
| 1089 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Revision PK for an existing target. NULL is allowed for previews that create a new item. | `UUID` |
| 1090 | AI Model Name | `model_name` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | Identifier of the model used for generation. Recorded when known; NULL is allowed when unknown. | - |
| 1091 | Input Parameters | `input_parameters` | `jsonb` | N | N | - | Y | - | N | N | Y | Y | Purpose, scope, baseline URS, number of items to generate, or document table of contents to generate, as entered on the screen | `{"purpose":"접근권한 검증","scope":"로그인","count":3}` |
| 1092 | Generation Status | `generation_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | PENDING / PROCESSING / COMPLETED / FAILED / CANCELLED | `PENDING` |
| 1093 | Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who requested generation | `UUID` |
| 1094 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Request receipt timestamp | `2026-09-02T00:00:00Z` |
| 1095 | Execution Start Timestamp | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI processing start timestamp | `2026-09-02T00:00:00Z` |
| 1096 | Execution Completion Timestamp | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI processing completion timestamp | `2026-09-02T00:00:00Z` |
| 1097 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Failure cause code | `LLM_TIMEOUT` |
| 1098 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Detailed failure message | `모델 응답시간 초과` |
| 1099 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-02T00:00:00Z` |
| 1100 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Created By | `UUID` |
| 1101 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-02T00:00:00Z` |
| 1102 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Updated By | `UUID` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `rule_ai_generation_job_1` | Business validation | `target_entity_id` | New items in ITEM_GENERATION may have NULL target_entity_id before application. DOCUMENT_GENERATION is linked to the corresponding document revision. |
| `rule_ai_generation_job_2` | Business validation | - | Completed generation does not mean business item approval or document approval. After selection and application, the normal editing and approval procedures are followed. |

[↑ Back to Top](#top)

---

<a id="table-ai_generation_result"></a>
### 68. AI Generation Result (`ai_generation_result`)

| Item | Definition |
| --- | --- |
| Description | Management of AI-generated result sets and adoption status |
| Primary Key | `ai_result_id` |
| Key References (FK) | `ai_job_id, selected_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1103 | AI Result ID | `ai_result_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI result identifier | `UUID` |
| 1104 | AI Job ID | `ai_job_id` | `uuid` | N | Y | `ai_generation_job.ai_job_id` | Y | - | N | Y | N | Y | Parent AI Job | `UUID` |
| 1105 | Result Title | `result_title` | `varchar(300)` | N | N | - | N | - | N | N | N | Y | Result Title | `OQ 테스트 초안` |
| 1106 | Selected Flag | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | Whether the user adopted the result | `TRUE` |
| 1107 | Applied Flag | `is_applied` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | Whether the result has been applied to the actual deliverable | `TRUE` |
| 1108 | Selecting User ID | `selected_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who adopted the result | `UUID` |
| 1109 | Selection Timestamp | `selected_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Adopted At | `2026-09-02T00:00:00Z` |
| 1110 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-02T00:00:00Z` |
| 1111 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Created By | `UUID` |
| 1112 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-02T00:00:00Z` |
| 1113 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Updated By | `UUID` |

#### Business Rules

A collection of AI results. Item-level selection and application are based on ai_result_item; is_selected/is_applied summarize the collection.

[↑ Back to Top](#top)

---

<a id="table-ai_result_item"></a>
### 69. AI Generation Result Item (`ai_result_item`)

| Item | Definition |
| --- | --- |
| Description | Management of details of AI-generated URS/FRA/IQ/OQ/PQ items and document sections |
| Primary Key | `ai_result_item_id` |
| Key References (FK) | `ai_result_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1114 | AI Result Item ID | `ai_result_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Result item identifier | `UUID` |
| 1115 | AI Result ID | `ai_result_id` | `uuid` | N | Y | `ai_generation_result.ai_result_id` | Y | - | N | Y | N | Y | Parent Result Reference | `UUID` |
| 1116 | Item Sequence Number | `item_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Result display order | `1` |
| 1117 | Item Type | `item_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT, FRA_SCENARIO, IQ_TEST, OQ_TEST, PQ_TEST, DOCUMENT_SECTION | `REQUIREMENT` |
| 1118 | Title | `title` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | Generated item title | `전자서명 기록` |
| 1119 | Body Content | `content` | `text` | N | N | - | Y | - | N | N | N | Y | AI-generated draft content for tests, requirements, risk assessments, or individual table-of-contents sections | `내용` |
| 1120 | Applied Target Type | `target_entity_type` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Business table type to which the result was applied: REQUIREMENT / FRA_ITEM / IQ_ITEM / OQ_ITEM / PQ_ITEM / DELIVERABLE_SECTION | `IQ_ITEM` |
| 1121 | Applied Target ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | PK of the revision actually created or updated by applying the result. NULL before application. | `UUID` |
| 1122 | Adopted Flag | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | Whether the user adopted the result | `TRUE` |
| 1123 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-02T00:00:00Z` |
| 1124 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Created By | `UUID` |
| 1125 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-02T00:00:00Z` |
| 1126 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Updated By | `UUID` |
| 1127 | Applied Flag | `is_applied` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Applied Flag | `FALSE` |
| 1128 | Application Timestamp | `applied_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Application Timestamp | `2026-09-01T00:00:00Z` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_ai_result_item_1` | UNIQUE | `ai_result_id, item_order` | UNIQUE(ai_result_id,item_order) |
| `uq_ai_result_item_1_rule` | Business validation | `ai_result_id, item_order, is_applied` | When is_applied=TRUE, the actual applied target type, PK, and applied_at are required. |
| `rule_ai_result_item_2` | Business validation | - | Results for individual document table-of-contents sections are linked to DELIVERABLE_SECTION. Approved items/documents are not overwritten; changes are applied to drafts or new revisions. |

[↑ Back to Top](#top)

---

## Regulation

<a id="table-regulatory_source"></a>
### 70. Regulatory Source Document (`regulatory_source`)

| Item | Definition |
| --- | --- |
| Description | Editions of regulations, guidelines, and SOPs referenced by URS, FRA, and the library |
| Primary Key | `regulatory_source_id` |
| Key References (FK) | `file_id, verified_by, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1129 | Regulatory Document ID | `regulatory_source_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1130 | Document Code | `source_code` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Document Code | `REF-001` |
| 1131 | Document Type | `source_type` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | REGULATION / GUIDELINE / INTERNAL_SOP | `GUIDELINE` |
| 1132 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Document Title | `검토용 기준 문서` |
| 1133 | Edition/Revision | `edition` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Edition/Revision | `Rev.1` |
| 1134 | Issuing Authority/Department | `issuing_body` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Issuing Authority/Department | `품질보증팀` |
| 1135 | Source URL | `source_url` | `text` | N | N | - | N | - | N | N | N | Y | Source URL | `https://example.com/reference` |
| 1136 | Original Attachment ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | References file_asset.file_id | `00000000-0000-0000-0000-000000000001` |
| 1137 | Effective Start Date | `effective_from` | `date` | N | N | - | N | - | N | N | N | Y | Effective Start Date | `2026-09-01` |
| 1138 | Effective End Date | `effective_to` | `date` | N | N | - | N | - | N | N | N | Y | Effective End Date | `2026-09-01` |
| 1139 | Review Status | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT / VERIFIED / RETIRED | `DRAFT` |
| 1140 | Verifier ID | `verified_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1141 | Verification Timestamp | `verified_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Verification Timestamp | `2026-09-01T00:00:00Z` |
| 1142 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1143 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1144 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1145 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_regulatory_source_1` | UNIQUE | `source_code, edition` | UNIQUE(source_code,edition) |
| `uq_regulatory_source_1_rule` | Business validation | `source_code, edition` | Different language editions are identified by separate source_code values. |
| `rule_regulatory_source_2` | Business validation | - | VERIFIED requires the verifier, verification timestamp, and either a URL or an original attachment. The effective end date must not precede the start date. |
| `rule_regulatory_source_3` | Business validation | - | A new edition is registered as a new row. Document editions referenced by approved business records must not be deleted or overwritten with new content. |

#### Business Rules

Regulatory references are managed as reusable reference data. Manually entered references are linked after confirming the document edition and clause; verified status is assigned only when the verifier and verification timestamp have been recorded.

[↑ Back to Top](#top)

---

<a id="table-regulatory_clause"></a>
### 71. Regulatory Clause (`regulatory_clause`)

| Item | Definition |
| --- | --- |
| Description | Clauses and display text belonging to an edition of a regulatory reference document |
| Primary Key | `regulatory_clause_id` |
| Key References (FK) | `regulatory_source_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1146 | Clause ID | `regulatory_clause_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1147 | Regulatory Document ID | `regulatory_source_id` | `uuid` | N | Y | `regulatory_source.regulatory_source_id` | Y | - | N | Y | N | Y | References regulatory_source.regulatory_source_id | `00000000-0000-0000-0000-000000000001` |
| 1148 | Clause Code | `clause_code` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Clause Code | `4.1` |
| 1149 | Clause Title | `title` | `varchar(255)` | N | N | - | N | - | N | N | N | Y | Clause Title | `접근 관리` |
| 1150 | Clause Summary | `summary` | `text` | N | N | - | Y | - | N | N | N | Y | Clause Summary | `해당 업무에 적용할 요구사항 요약` |
| 1151 | Source Location | `source_locator` | `text` | N | N | - | N | - | N | N | N | Y | Source Location | `제4장 1절` |
| 1152 | Display Order | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display Order | `1` |
| 1153 | In Use | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | In Use | `TRUE` |
| 1154 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1155 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1156 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1157 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_regulatory_clause_1` | UNIQUE | `regulatory_source_id, clause_code` | UNIQUE(regulatory_source_id,clause_code) |
| `ck_regulatory_clause_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_regulatory_clause_2` | Business validation | - | New selections use active clauses in VERIFIED document editions. Historical references are retained even if a clause previously used for approval becomes inactive. |

[↑ Back to Top](#top)

---

<a id="table-requirement_regulation"></a>
### 72. Requirement Regulatory Reference (`requirement_regulation`)

| Item | Definition |
| --- | --- |
| Description | Multiple links between business items and applicable regulatory clauses |
| Primary Key | `requirement_regulation_id` |
| Key References (FK) | `requirement_id, regulatory_clause_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1158 | Reference Link ID | `requirement_regulation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1159 | Business Item ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | Y | N | Y | References requirement.requirement_id | `00000000-0000-0000-0000-000000000001` |
| 1160 | Regulatory Clause ID | `regulatory_clause_id` | `uuid` | N | Y | `regulatory_clause.regulatory_clause_id` | Y | - | N | Y | N | Y | References regulatory_clause.regulatory_clause_id | `00000000-0000-0000-0000-000000000001` |
| 1161 | Applicability Rationale | `application_note` | `text` | N | N | - | N | - | N | N | N | Y | Applicability Rationale | `접근권한 요구사항에 적용` |
| 1162 | Cited Text | `citation_snapshot` | `text` | N | N | - | Y | - | N | N | N | Y | Preserves the document title, edition, and clause display as they existed at the time of application | `검토용 기준 문서 Rev.1 / 4.1` |
| 1163 | Display Order | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display Order | `1` |
| 1164 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1165 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1166 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1167 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_requirement_regulation_1` | UNIQUE | `requirement_id, regulatory_clause_id` | UNIQUE(requirement_id,regulatory_clause_id) |
| `ck_requirement_regulation_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_requirement_regulation_2` | Business validation | - | The clause and cited text as they existed at business approval are preserved. When applying a library item to URS, the reference relationships are also copied, and subsequent template changes are not automatically propagated to existing URS items. |

[↑ Back to Top](#top)

---

<a id="table-library_item_regulation"></a>
### 73. Library Regulatory Reference (`library_item_regulation`)

| Item | Definition |
| --- | --- |
| Description | Multiple links between business items and applicable regulatory clauses |
| Primary Key | `library_item_regulation_id` |
| Key References (FK) | `library_id, regulatory_clause_id, created_by, updated_by` |
| GxP Criticality | High |
| Audit Applicable | Y |
| Retention/Deletion Policy | Retain for the period during which auditing must remain possible. |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/sensitive information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 1168 | Reference Link ID | `library_item_regulation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of this row | `00000000-0000-0000-0000-000000000001` |
| 1169 | Business Item ID | `library_id` | `uuid` | N | Y | `library_item.library_id` | Y | - | N | Y | N | Y | References library_item.library_id | `00000000-0000-0000-0000-000000000001` |
| 1170 | Regulatory Clause ID | `regulatory_clause_id` | `uuid` | N | Y | `regulatory_clause.regulatory_clause_id` | Y | - | N | Y | N | Y | References regulatory_clause.regulatory_clause_id | `00000000-0000-0000-0000-000000000001` |
| 1171 | Applicability Rationale | `application_note` | `text` | N | N | - | N | - | N | N | N | Y | Applicability Rationale | `접근권한 요구사항에 적용` |
| 1172 | Cited Text | `citation_snapshot` | `text` | N | N | - | Y | - | N | N | N | Y | Preserves the document title, edition, and clause display as they existed at the time of application | `검토용 기준 문서 Rev.1 / 4.1` |
| 1173 | Display Order | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | Display Order | `1` |
| 1174 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Created At | `2026-09-01T00:00:00Z` |
| 1175 | Created By | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 1176 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Updated At | `2026-09-01T00:00:00Z` |
| 1177 | Updated By | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |

#### Constraints

| Constraint Name | Type | Applicable Columns | Condition / Rule |
| --- | --- | --- | --- |
| `uq_library_item_regulation_1` | UNIQUE | `library_id, regulatory_clause_id` | UNIQUE(library_id,regulatory_clause_id) |
| `ck_library_item_regulation_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_library_item_regulation_2` | Business validation | - | The clause and cited text as they existed at business approval are preserved. When applying a library item to URS, the reference relationships are also copied, and subsequent template changes are not automatically propagated to existing URS items. |

[↑ Back to Top](#top)

---
