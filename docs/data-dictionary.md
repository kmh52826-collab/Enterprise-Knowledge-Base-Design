# Data Table Specifications

> Validation Management Platform Data Model Documentation  
> **Security Measures**: Example values have been replaced with de-identified sample values for public release.

## Table of Contents

- [1. Document Overview](#1-document-overview)
- [2. Notation Conventions](#2-notation-conventions)
- [3. Table List](#3-table-list)
- [4. Domain Navigation](#4-domain-navigation)
- [5. Detailed Table and Column Definitions](#5-detailed-table-and-column-definitions)

## 1. Document Overview

- Total tables: **51**
- Total columns: **783**
- Domains: **25**
- Naming conventions: Tables and columns use `snake_case`; primary keys use entity-specific identifier columns.
- Date/time conventions: Store UTC in the database; display dates and times in the user's or site's time zone.
- GxP principles: Manage approved records through revisions and status history without directly modifying them, and record significant changes in the Audit Trail.

## 2. Notation Conventions

| Notation | Meaning |
|---|---|
| `Y` | Applicable or required |
| `N` | Not applicable or optional |
| `-` | Not applicable or unspecified |
| PK | Primary Key |
| FK | Foreign Key |
| Not Null | The inverse of whether NULL is allowed |
| Audit | Whether the item is recorded in the Audit Trail |

## 3. Table List

| No | Domain | Logical Table Name | Physical Table Name | PK | Key References (FK) | GxP | Audit |
|---:|---|---|---|---|---|:---:|:---:|
| 1 | Organization | Organization/Client Company | [`organization`](#table-organization) | `organization_id` | - | High | Y |
| 2 | Security | User | [`app_user`](#table-app_user) | `user_id` | `organization_id` | High | Y |
| 3 | Security | Role | [`role`](#table-role) | `role_id` | - | High | Y |
| 4 | Security | User Role | [`user_role`](#table-user_role) | `user_role_id` | `user_id, role_id` | High | Y |
| 5 | Compliance | Electronic Signature | [`electronic_signature`](#table-electronic_signature) | `signature_id` | `signer_id` | Critical | Y |
| 6 | Compliance | Audit Trail | [`audit_trail`](#table-audit_trail) | `audit_id` | `actor_id` | Critical | Y |
| 7 | File | File Asset | [`file_asset`](#table-file_asset) | `file_id` | `uploader_id` | High | Y |
| 8 | System | System/Equipment Identification Information | [`system_asset`](#table-system_asset) | `system_id` | `organization_id` | High | Y |
| 9 | Library | Library Item Master | [`library_item`](#table-library_item) | `library_id` | - | High | Y |
| 10 | Validation | Validation Project | [`validation_project`](#table-validation_project) | `project_id` | `system_id, created_by, updated_by` | High | Y |
| 11 | QIA | Quality Impact Assessment Header | [`qia_assessment`](#table-qia_assessment) | `qia_id` | `project_id` | High | Y |
| 12 | QIA | QIA Module Detailed Assessment | [`qia_module_item`](#table-qia_module_item) | `qia_module_item_id` | `qia_id` | High | Y |
| 13 | VA | Vendor Audit Assessment | [`vendor_audit`](#table-vendor_audit) | `audit_id` | `project_id` | High | Y |
| 14 | URS | User Requirements Specification | [`requirement`](#table-requirement) | `requirement_id` | `project_id, created_by` | High | Y |
| 15 | FDS | Functional Design Specification | [`fds_spec`](#table-fds_spec) | `fds_id` | `project_id, created_by, updated_by` | High | Y |
| 16 | FDS | FDS Detailed Item | [`fds_item`](#table-fds_item) | `fds_item_id` | `fds_id, created_by, updated_by` | High | Y |
| 17 | FDS | FDS Interface Definition | [`fds_interface`](#table-fds_interface) | `fds_interface_id` | `fds_id, created_by, updated_by` | High | Y |
| 18 | DQ | Design Qualification Assessment | [`dq_assessment`](#table-dq_assessment) | `dq_id` | `project_id, created_by, updated_by` | High | Y |
| 19 | DQ | DQ Detailed Assessment Item | [`dq_item`](#table-dq_item) | `dq_item_id` | `dq_id, requirement_id, reviewed_by, created_by, updated_by` | High | Y |
| 20 | FRA | Functional Risk Assessment | [`fra_assessment`](#table-fra_assessment) | `fra_id` | `project_id, created_by, updated_by` | High | Y |
| 21 | FRA | FRA Detailed Risk Item | [`fra_item`](#table-fra_item) | `fra_item_id` | `fra_id, requirement_id, created_by, updated_by` | High | Y |
| 22 | IQ | Installation Qualification Assessment | [`iq_assessment`](#table-iq_assessment) | `iq_id` | `project_id, created_by, updated_by` | High | Y |
| 23 | IQ | IQ Detailed Test Item | [`iq_item`](#table-iq_item) | `iq_item_id` | `iq_id, executed_by, created_by, updated_by` | High | Y |
| 24 | OQ | Operational Qualification Assessment | [`oq_assessment`](#table-oq_assessment) | `oq_id` | `project_id, created_by, updated_by` | High | Y |
| 25 | OQ | OQ Detailed Test Item | [`oq_item`](#table-oq_item) | `oq_item_id` | `oq_id, executed_by, created_by, updated_by` | High | Y |
| 26 | PQ | Performance Qualification Assessment | [`pq_assessment`](#table-pq_assessment) | `pq_id` | `project_id, created_by, updated_by` | High | Y |
| 27 | PQ | PQ Detailed Test Item | [`pq_item`](#table-pq_item) | `pq_item_id` | `pq_id, executed_by, created_by, updated_by` | High | Y |
| 28 | RTM | Requirements Traceability Matrix | [`rtm_assessment`](#table-rtm_assessment) | `rtm_id` | `project_id, created_by, updated_by` | Critical | Y |
| 29 | RTM | RTM Detailed Traceability Item | [`rtm_item`](#table-rtm_item) | `rtm_item_id` | `rtm_id, requirement_id, created_by, updated_by` | Critical | Y |
| 30 | VSR | Validation Summary Report | [`vsr_assessment`](#table-vsr_assessment) | `vsr_id` | `project_id, created_by, updated_by` | Critical | Y |
| 31 | VSR | VSR Activity Summary Item | [`vsr_item`](#table-vsr_item) | `vsr_item_id` | `vsr_id, created_by, updated_by` | Critical | Y |
| 32 | Workflow | Workflow Instance | [`workflow_instance`](#table-workflow_instance) | `workflow_instance_id` | `requested_by, created_by, updated_by` | Critical | Y |
| 33 | Workflow | Workflow Step | [`workflow_step`](#table-workflow_step) | `workflow_step_id` | `workflow_instance_id, assignee_id, created_by, updated_by` | Critical | Y |
| 34 | Workflow | Approval Action History | [`approval_action`](#table-approval_action) | `approval_action_id` | `workflow_step_id, actor_id, signature_id` | Critical | Y |
| 35 | Traceability | Common Traceability Link | [`traceability_link`](#table-traceability_link) | `traceability_link_id` | `project_id, created_by, updated_by` | Critical | Y |
| 36 | File | Evidence File Link | [`evidence_link`](#table-evidence_link) | `evidence_link_id` | `project_id, file_id, created_by, updated_by` | High | Y |
| 37 | Validation | Project Member | [`project_member`](#table-project_member) | `project_member_id` | `project_id, user_id, role_id, created_by, updated_by` | High | Y |
| 38 | Validation | Validation Activity Master | [`validation_activity`](#table-validation_activity) | `activity_id` | `created_by, updated_by` | High | Y |
| 39 | Validation | Project Activity | [`project_activity`](#table-project_activity) | `project_activity_id` | `project_id, activity_id, created_by, updated_by` | Critical | Y |
| 40 | Validation | Activity Dependency | [`activity_dependency`](#table-activity_dependency) | `activity_dependency_id` | `successor_activity_id, predecessor_activity_id, created_by, updated_by` | Critical | Y |
| 41 | DDS | Detailed Design Specification | [`dds_spec`](#table-dds_spec) | `dds_id` | `project_id, created_by, updated_by` | High | Y |
| 42 | DDS | DDS Detailed Item | [`dds_item`](#table-dds_item) | `dds_item_id` | `dds_id, created_by, updated_by` | High | Y |
| 43 | Deviation | Deviation Management | [`deviation`](#table-deviation) | `deviation_id` | `project_id, resolved_by, approved_by, created_by, updated_by` | Critical | Y |
| 44 | Report | Report Generation Job | [`report_generation`](#table-report_generation) | `report_generation_id` | `project_id, requested_by, result_file_id, report_schedule_id, created_by, updated_by` | High | Y |
| 45 | AI | AI Generation Job | [`ai_generation_job`](#table-ai_generation_job) | `ai_job_id` | `project_id, requested_by, created_by, updated_by` | High | Y |
| 46 | AI | AI Generation Result | [`ai_generation_result`](#table-ai_generation_result) | `ai_result_id` | `ai_job_id, created_by, updated_by` | High | Y |
| 47 | AI | AI Generation Result Item | [`ai_result_item`](#table-ai_result_item) | `ai_result_item_id` | `ai_result_id, created_by, updated_by` | High | Y |
| 48 | Notification | Notification Delivery | [`notification_delivery`](#table-notification_delivery) | `notification_delivery_id` | `project_id, workflow_instance_id, workflow_step_id, recipient_id, created_by, updated_by` | High | Y |
| 49 | System | Backup Execution History | [`backup_execution`](#table-backup_execution) | `backup_execution_id` | `requested_by, created_by, updated_by` | Critical | Y |
| 50 | Report | Report Schedule | [`report_schedule`](#table-report_schedule) | `report_schedule_id` | `project_id, created_by, updated_by` | High | Y |
| 51 | File | File Cleanup Execution History | [`file_cleanup_execution`](#table-file_cleanup_execution) | `file_cleanup_execution_id` | `requested_by, created_by, updated_by` | High | Y |

## 4. Domain Navigation

- **Organization**: [`organization`](#table-organization)
- **Security**: [`app_user`](#table-app_user), [`role`](#table-role), [`user_role`](#table-user_role)
- **Compliance**: [`electronic_signature`](#table-electronic_signature), [`audit_trail`](#table-audit_trail)
- **File**: [`file_asset`](#table-file_asset), [`evidence_link`](#table-evidence_link), [`file_cleanup_execution`](#table-file_cleanup_execution)
- **System**: [`system_asset`](#table-system_asset), [`backup_execution`](#table-backup_execution)
- **Library**: [`library_item`](#table-library_item)
- **Validation**: [`validation_project`](#table-validation_project), [`project_member`](#table-project_member), [`validation_activity`](#table-validation_activity), [`project_activity`](#table-project_activity), [`activity_dependency`](#table-activity_dependency)
- **QIA**: [`qia_assessment`](#table-qia_assessment), [`qia_module_item`](#table-qia_module_item)
- **VA**: [`vendor_audit`](#table-vendor_audit)
- **URS**: [`requirement`](#table-requirement)
- **FDS**: [`fds_spec`](#table-fds_spec), [`fds_item`](#table-fds_item), [`fds_interface`](#table-fds_interface)
- **DQ**: [`dq_assessment`](#table-dq_assessment), [`dq_item`](#table-dq_item)
- **FRA**: [`fra_assessment`](#table-fra_assessment), [`fra_item`](#table-fra_item)
- **IQ**: [`iq_assessment`](#table-iq_assessment), [`iq_item`](#table-iq_item)
- **OQ**: [`oq_assessment`](#table-oq_assessment), [`oq_item`](#table-oq_item)
- **PQ**: [`pq_assessment`](#table-pq_assessment), [`pq_item`](#table-pq_item)
- **RTM**: [`rtm_assessment`](#table-rtm_assessment), [`rtm_item`](#table-rtm_item)
- **VSR**: [`vsr_assessment`](#table-vsr_assessment), [`vsr_item`](#table-vsr_item)
- **Workflow**: [`workflow_instance`](#table-workflow_instance), [`workflow_step`](#table-workflow_step), [`approval_action`](#table-approval_action)
- **Traceability**: [`traceability_link`](#table-traceability_link)
- **DDS**: [`dds_spec`](#table-dds_spec), [`dds_item`](#table-dds_item)
- **Deviation**: [`deviation`](#table-deviation)
- **Report**: [`report_generation`](#table-report_generation), [`report_schedule`](#table-report_schedule)
- **AI**: [`ai_generation_job`](#table-ai_generation_job), [`ai_generation_result`](#table-ai_generation_result), [`ai_result_item`](#table-ai_result_item)
- **Notification**: [`notification_delivery`](#table-notification_delivery)

---

# 5. Detailed Table and Column Definitions

## Organization

<a id="table-organization"></a>
### 1. Organization/Client Company (`organization`)

| Item | Definition |
|---|---|
| Description | Basic information about client companies or operating organizations |
| Primary Key | `organization_id` |
| Key References (FK) | - |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 14 | Organization ID | `organization_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique organization identifier | `00000000-0000-0000-0000-000000000001` |
| 15 | Organization Code | `organization_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Organization/company identification code | `ORG-SAMPLE-01` |
| 16 | Organization Name | `organization_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Company/site name | `Sample Corporation` |
| 17 | Organization Type | `organization_type` | `varchar(50)` | N | N | - | Y | `Headquarters'` | N | N | N | Y | Headquarters \| Factory \| Research Institute \| Overseas Subsidiary | `Headquarters` |
| 18 | Status | `status` | `varchar(20)` | N | N | - | Y | `ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE | `ACTIVE` |
| 19 | Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Organization description | `Corporate IT and data operations organization` |
| 20 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-01T00:00:00` |
| 21 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T16:00:00` |

[↑ Back to Top](#data-table-specifications)

---

## Security

<a id="table-app_user"></a>
### 2. User (`app_user`)

| Item | Definition |
|---|---|
| Description | User account and basic profile information |
| Primary Key | `user_id` |
| Key References (FK) | `organization_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1 | User ID | `user_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique user account identifier | `00000000-0000-0000-0000-000000000001` |
| 2 | Organization ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | References organization.organization_id | `00000000-0000-0000-0000-000000000001` |
| 3 | Username | `username` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Login ID (duplicates not allowed) | `user.sample` |
| 4 | Password Hash | `password_hash` | `varchar(255)` | N | N | - | Y | - | N | N | N | N | Hashed password value | `[REDACTED_PASSWORD_HASH]` |
| 5 | Full Name | `full_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | User name | `Hong Gil-dong` |
| 6 | Email | `email` | `varchar(100)` | N | N | - | Y | - | Y | Y | N | Y | Email address | `user.sample@example.com` |
| 7 | Department Name | `department_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Name of the user's department | `Information Strategy Team` |
| 8 | Position/Job Title | `position_title` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Position information | `Senior` |
| 9 | Account Status | `status` | `varchar(20)` | N | N | - | Y | `ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE \| LOCKED | `ACTIVE` |
| 10 | Last Login At | `last_login_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Timestamp of the most recent system login | `2026-08-26T16:00:00` |
| 11 | Password Changed At | `password_changed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Timestamp of the last password change | `2026-08-01T09:00:00` |
| 12 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-01T00:00:00` |
| 13 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T16:00:00` |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-role"></a>
### 3. Role (`role`)

| Item | Definition |
|---|---|
| Description | Master definitions of permission roles, such as author, reviewer, and approver |
| Primary Key | `role_id` |
| Key References (FK) | - |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 22 | Role ID | `role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique role/permission identifier | `00000000-0000-0000-0000-000000000001` |
| 23 | Role Code | `role_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Role identification code | `SYSTEM_ADMIN` |
| 24 | Role Name | `role_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Role name (system administrator, author, approver, etc.) | `Super Administrator` |
| 25 | Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Detailed description of the role's permission scope | `System-wide administrative permissions` |
| 26 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-01T00:00:00` |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-user_role"></a>
### 4. User Role (`user_role`)

| Item | Definition |
|---|---|
| Description | User-to-role mapping information |
| Primary Key | `user_role_id` |
| Key References (FK) | `user_id, role_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 27 | Mapping ID | `user_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique user-to-role mapping identifier. Duplicate (user_id, role_id) combinations are not allowed in active data. | `00000000-0000-0000-0000-000000000001` |
| 28 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 29 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | References role.role_id | `00000000-0000-0000-0000-000000000001` |
| 30 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-01T00:00:00` |

[↑ Back to Top](#data-table-specifications)

---

## Compliance

<a id="table-electronic_signature"></a>
### 5. Electronic Signature (`electronic_signature`)

| Item | Definition |
|---|---|
| Description | Records electronic signature evidence for document review/approval |
| Primary Key | `signature_id` |
| Key References (FK) | `signer_id` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 31 | Electronic Signature ID | `signature_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique electronic signature record identifier | `00000000-0000-0000-0000-000000000001` |
| 32 | Signer ID | `signer_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 33 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Name of the table to which the signature applies (e.g., requirement) | `requirement` |
| 34 | Target Record ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Primary key value of the signed record | `00000000-0000-0000-0000-000000000001` |
| 35 | Signature Step | `signature_action` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | REVIEW \| APPROVE \| REJECT | `APPROVE` |
| 36 | Signature Purpose | `signature_meaning` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Reason for signing under 21 CFR Part 11 | `Final approval of URS requirements` |
| 37 | Signature Timestamp | `signed_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp when the electronic signature was executed | `2026-08-26T16:30:00` |
| 38 | Target Document Version | `target_version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the target document revision to which the electronic signature applies | `v1.0` |
| 39 | Signed Content Hash | `content_hash` | `varchar(128)` | N | N | - | Y | - | N | N | N | Y | SHA-256 hash for verifying the integrity of the target document and its details at the time of signing | `[SAMPLE_SHA256_HASH]` |
| 40 | Reauthentication Method | `authentication_method` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Reauthentication method used to verify the signer's identity when executing the electronic signature | `PASSWORD` |
| 41 | Reauthentication Result | `authentication_result` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Result of reauthentication when executing the electronic signature | `SUCCESS` |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-audit_trail"></a>
### 6. Audit Trail (`audit_trail`)

| Item | Definition |
|---|---|
| Description | Audit trail records of before and after values for significant data changes and the actors responsible |
| Primary Key | `audit_id` |
| Key References (FK) | `actor_id` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 42 | Audit Trail ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique audit trail record identifier | `00000000-0000-0000-0000-000000000001` |
| 43 | Actor ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id (NULL allowed for automated system operations) | `00000000-0000-0000-0000-000000000001` |
| 44 | Action Type | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | CREATE, UPDATE, DELETE, EXPORT, LOGIN, REPORT_GENERATE, REPORT_DOWNLOAD, REPORT_CANCEL | `REPORT_GENERATE` |
| 45 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Physical table name where the change occurred | `requirement` |
| 46 | Target Record ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Primary key value of the changed record | `00000000-0000-0000-0000-000000000001` |
| 47 | Before Data | `old_values` | `jsonb` | N | N | - | N | - | N | N | N | Y | JSON data before modification | `{"status": "Draft"}` |
| 48 | After Data | `new_values` | `jsonb` | N | N | - | N | - | N | N | N | Y | JSON data after modification | `{"status": "Approved"}` |
| 49 | Reason for Change | `reason_for_change` | `text` | N | N | - | N | - | N | N | N | Y | Reason for the data change under 21 CFR Part 11 | `Corrected requirement typos and supplemented regulatory items` |
| 50 | Client IP Address | `client_ip` | `varchar(45)` | N | N | - | N | - | N | N | N | Y | User's client IP address | `192.0.2.10` |
| 51 | Occurred At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Audit trail log creation timestamp (UTC) | `2026-08-26T16:30:00` |
| 52 | Actor Type | `actor_type` | `varchar(20)` | N | N | - | Y | `USER` | N | Y | N | Y | Type of actor making the change: USER, SYSTEM, or BATCH. actor_id is required when the type is USER. | `USER` |
| 53 | Request ID | `request_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Identifier for grouping multiple audit trail records generated by a single UI or API request | `00000000-0000-0000-0000-000000000001` |
| 54 | Session ID | `session_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Identifier of the user login session in which the change occurred. NULL allowed for system/batch processing or requests without a session. | `00000000-0000-0000-0000-000000000001` |
| 55 | Target Document Version | `target_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | For revision-controlled documents, stores the display version at the time of the change. NULL allowed for ordinary tables. | `v1.0` |
| 56 | Target Revision Number | `target_revision_number` | `integer` | N | N | - | N | - | N | N | N | Y | For revision-controlled documents, stores the numeric revision sequence number at the time of the change. NULL allowed for ordinary tables. | `1` |
| 57 | Request Path | `request_uri` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | UI or API request path that triggered the change. NULL allowed when no path exists, such as for system/batch processing. | `/api/fds/approve` |
| 58 | Client Information | `user_agent` | `text` | N | N | - | N | - | N | N | Y | Y | Browser, operating system, or client application information for the change request | `Sample-Client/1.0` |

[↑ Back to Top](#data-table-specifications)

---

## File

<a id="table-file_asset"></a>
### 7. File Asset (`file_asset`)

| Item | Definition |
|---|---|
| Description | Manages storage metadata for user attachments, evidence files, generated reports, and data export files |
| Primary Key | `file_id` |
| Key References (FK) | `uploader_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the required audit period according to the retention policies for the file type and linked entity |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 59 | File ID | `file_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique attachment metadata identifier | `00000000-0000-0000-0000-000000000001` |
| 60 | Uploader ID | `uploader_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Uploader ID is required for files uploaded by users. NULL allowed for files generated by the system or scheduled batch jobs. | `UUID` |
| 61 | Original File Name | `original_file_name` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | File name at the time of upload | `sample_document.pdf` |
| 62 | Storage File Path | `stored_file_path` | `text` | N | N | - | Y | - | N | N | N | Y | Storage path/S3 key | `documents/2026/08/sample_001.pdf` |
| 63 | File Category | `file_category` | `varchar(30)` | N | N | - | Y | `ATTACHMENT` | N | Y | N | Y | Business category of the file: ATTACHMENT, EVIDENCE, REPORT, EXPORT | `REPORT` |
| 64 | File Size | `file_size_bytes` | `bigint` | N | N | - | Y | `0` | N | N | N | Y | File size (bytes) | `1048576` |
| 65 | MIME Type | `mime_type` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | File format | `application/pdf` |
| 66 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T16:30:00` |
| 67 | Is Temporary File | `is_temporary` | `boolean` | N | N | - | Y | `False` | N | Y | N | Y | Whether the file is a temporary file created during upload or generation | `True` |
| 68 | Expires At | `expires_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Time when the file becomes eligible for cleanup. NULL for permanent files or files subject to business retention requirements. | `2026-09-09 18:00:00+00` |
| 69 | Cleanup Status | `cleanup_status` | `varchar(20)` | N | N | - | Y | `ACTIVE` | N | Y | N | Y | File cleanup status: ACTIVE, CLEANUP_PENDING, CLEANED, CLEANUP_FAILED | `ACTIVE` |
| 70 | Last Cleanup Execution ID | `cleanup_execution_id` | `uuid` | N | Y | `file_cleanup_execution.file_cleanup_execution_id` | N | - | N | Y | N | Y | ID of the file cleanup job that last processed this file | `UUID` |
| 71 | Cleaned Up At | `cleaned_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when cleanup of the actual stored file and its metadata was completed | `2026-09-09 19:00:00+00` |
| 72 | Cleanup Failure Reason | `cleanup_error_message` | `text` | N | N | - | N | - | N | N | N | Y | Detailed reason for failure to clean up an individual file. Sensitive information such as access keys must not be stored. | `No permission to access the file` |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-evidence_link"></a>
### 36. Evidence File Link (`evidence_link`)

| Item | Definition |
|---|---|
| Description | Manages N:M links between documents/test items and evidence files |
| Primary Key | `evidence_link_id` |
| Key References (FK) | `project_id, file_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 523 | Evidence Link ID | `evidence_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for a link between a document/test item and an evidence file. Duplicate (project_id, file_id, target_entity_type, target_entity_id, evidence_type) combinations are not allowed in active data. | `UUID` |
| 524 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | ID of the project to which the evidence link belongs | `UUID` |
| 525 | File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | Y | - | N | Y | N | Y | ID of the linked evidence file | `UUID` |
| 526 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Type of entity linked to the evidence file: REQUIREMENT, FDS_SPEC, FDS_ITEM, FDS_INTERFACE, DDS_SPEC, DDS_ITEM, QIA_ASSESSMENT, QIA_MODULE_ITEM, VENDOR_AUDIT, DQ_ASSESSMENT, DQ_ITEM, FRA_ASSESSMENT, FRA_ITEM, IQ_ASSESSMENT, IQ_ITEM, OQ_ASSESSMENT, OQ_ITEM, PQ_ASSESSMENT, PQ_ITEM, RTM_ASSESSMENT, RTM_ITEM, VSR_ASSESSMENT, VSR_ITEM, DEVIATION | `IQ_ITEM` |
| 527 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Primary key value of the table corresponding to target_entity_type. No physical FK is defined because this is a polymorphic reference. | `UUID` |
| 528 | Evidence Type | `evidence_type` | `varchar(50)` | N | N | - | Y | `TEST_RESULT` | N | Y | N | Y | TEST_RESULT, SCREENSHOT, LOG, REPORT, APPROVAL_DOCUMENT | `TEST_RESULT` |
| 529 | Evidence Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Content of the evidence file and purpose of the link | `Screenshot of IQ execution results` |
| 530 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Evidence link creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 531 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who created the evidence link | `UUID` |
| 532 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Evidence link last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 533 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who last modified the evidence link | `UUID` |
| 534 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Evidence link soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-file_cleanup_execution"></a>
### 51. File Cleanup Execution History (`file_cleanup_execution`)

| Item | Definition |
|---|---|
| Description | Manages execution criteria, processed item counts, execution statuses, failures, and retry history for temporary and expired file cleanup jobs |
| Primary Key | `file_cleanup_execution_id` |
| Key References (FK) | `requested_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain file cleanup results and execution history for the required audit period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 762 | File Cleanup Execution ID | `file_cleanup_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for a temporary/expired file cleanup job | `UUID` |
| 763 | Cleanup Type | `cleanup_type` | `varchar(30)` | N | N | - | Y | - | N | Y | N | Y | Cleanup type: TEMPORARY_FILE, EXPIRED_FILE, ORPHAN_FILE | `EXPIRED_FILE` |
| 764 | Execution Mode | `execution_type` | `varchar(20)` | N | N | - | Y | `SCHEDULED` | N | Y | N | Y | Execution mode: SCHEDULED, ON_DEMAND | `SCHEDULED` |
| 765 | Target Cutoff Time | `target_base_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Cutoff time used to find files that expired or became eligible for cleanup before this time | `2026-09-02 00:00:00+00` |
| 766 | Execution Status | `execution_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Execution status: PENDING, PROCESSING, COMPLETED, RETRY_WAIT, FAILED, CANCELLED | `COMPLETED` |
| 767 | Scanned File Count | `scanned_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of files scanned to determine cleanup eligibility. Must be 0 or greater. | `100` |
| 768 | Cleanup Target Count | `target_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of files identified as eligible for cleanup. Must be 0 or greater. | `20` |
| 769 | Cleaned File Count | `cleaned_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of files for which the actual file and metadata cleanup was completed. Must be 0 or greater. | `19` |
| 770 | Cleanup Failure Count | `failed_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of files that failed cleanup. Must be 0 or greater. | `1` |
| 771 | Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who requested manual cleanup. NULL allowed for scheduled batch executions. | `UUID` |
| 772 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when a manual request was received or a scheduled job was registered | `2026-09-02 01:00:00+00` |
| 773 | Execution Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the file cleanup job actually started | `2026-09-02 01:00:05+00` |
| 774 | Execution Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the cleanup job ended in success or final failure | `2026-09-02 01:05:00+00` |
| 775 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of retries after the initial execution failed. Must be 0 or greater. | `0` |
| 776 | Maximum Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Maximum number of automatic retries allowed. Must be 0 or greater. | `3` |
| 777 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next scheduled execution time for a job in RETRY_WAIT status | `2026-09-02 01:15:00+00` |
| 778 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | System error code categorizing the cause of file cleanup failure | `FILE_DELETE_FAILED` |
| 779 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Details of file cleanup failure. Sensitive information such as access keys must not be stored. | `Failed to delete the stored file` |
| 780 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | File cleanup execution history creation timestamp (UTC) | `2026-09-02 01:00:00+00` |
| 781 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when created by a user. NULL allowed when created by the system or a batch job. | `UUID` |
| 782 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | File cleanup execution history last modification timestamp (UTC) | `2026-09-02 01:05:00+00` |
| 783 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when modified by a user. NULL allowed for system/batch processing. | `UUID` |

[↑ Back to Top](#data-table-specifications)

---

## System

<a id="table-system_asset"></a>
### 8. System/Equipment Identification Information (`system_asset`)

| Item | Definition |
|---|---|
| Description | Reference information for systems/equipment subject to validation (asset management number, GAMP category) |
| Primary Key | `system_id` |
| Key References (FK) | `organization_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 73 | System ID | `system_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique system identifier | `00000000-0000-0000-0000-000000000001` |
| 74 | Organization ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | Client company or operating organization to which the system/equipment belongs | `00000000-0000-0000-0000-000000000001` |
| 75 | Asset Management Number | `management_number` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Asset/equipment management number | `EQ-MES-2024-001` |
| 76 | System Name | `system_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | System/equipment name | `Sample System` |
| 77 | System Type | `system_type` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | IT System \| Production Equipment \| Quality Equipment \| Utility | `IT System` |
| 78 | Responsible Department | `department_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | N | Department responsible for management/operation | `Production Engineering Team` |
| 79 | Installation Location | `location` | `varchar(200)` | N | N | - | N | - | N | N | N | N | Physical/logical installation location | `Server Room, Building A, 3F` |
| 80 | Vendor | `vendor` | `varchar(100)` | N | N | - | N | - | N | N | N | N | Equipment/system vendor name | `Sample Vendor` |
| 81 | Model Name | `model_name` | `varchar(100)` | N | N | - | N | - | N | N | N | N | Equipment/system model name | `FillMaster 500` |
| 82 | System Description | `description` | `text` | N | N | - | N | - | N | N | N | N | Description of the system's purpose and operational scope | `Production process data collection and control` |
| 83 | System Identification Status | `identification_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Registration/identification status of the target system: PENDING, COMPLETED | `COMPLETED` |
| 84 | Includes CS | `is_cs_included` | `boolean` | N | N | - | Y | `True` | N | N | N | Y | Whether a Computerized System is included | `True` |
| 85 | Version | `version` | `varchar(50)` | N | N | - | N | - | N | N | N | N | Software or equipment version | `v3.2.1` |
| 86 | GAMP Category | `gamp_category` | `varchar(50)` | N | N | - | N | - | N | N | N | N | Category 3 \| Category 4 \| Category 5, etc. | `Category 4` |
| 87 | GxP Classification | `gxp_type` | `varchar(50)` | N | N | - | N | - | N | N | N | N | GMP \| GLP \| GDP \| Non-GxP, etc. | `GMP` |
| 88 | Status | `status` | `varchar(20)` | N | N | - | Y | `Active'` | N | N | N | Y | Active \| Under Review \| Inactive | `Active` |
| 89 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-25T00:00:00` |
| 90 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-25T00:00:00` |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-backup_execution"></a>
### 49. Backup Execution History (`backup_execution`)

| Item | Definition |
|---|---|
| Description | Manages execution statuses, backup scopes, storage locations, failures, and retry history for scheduled and manual system data and file backups |
| Primary Key | `backup_execution_id` |
| Key References (FK) | `requested_by, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain backup execution history for the required audit period according to backup policies and regulations |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 723 | Backup Execution ID | `backup_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique backup execution history identifier | `UUID` |
| 724 | Backup Type | `backup_type` | `varchar(30)` | N | N | - | Y | - | N | Y | N | Y | Backup type: FULL, INCREMENTAL, DATABASE, FILE | `FULL` |
| 725 | Execution Mode | `execution_type` | `varchar(20)` | N | N | - | Y | `SCHEDULED` | N | Y | N | Y | Execution mode: SCHEDULED, ON_DEMAND | `SCHEDULED` |
| 726 | Backup Target | `backup_target` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Backup target category: DATABASE, FILE_STORAGE, ALL | `ALL` |
| 727 | Backup Reference Time | `backup_base_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Reference time for the data to be backed up | `2026-09-02 18:00:00+00` |
| 728 | Execution Status | `execution_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Execution status: PENDING, PROCESSING, COMPLETED, RETRY_WAIT, FAILED, CANCELLED | `COMPLETED` |
| 729 | Backup Storage Location | `backup_location` | `text` | N | N | - | N | - | N | N | N | Y | Backup file location or storage path. Authentication information such as access tokens must not be stored. | `backups/YYYY/MM/DD/full` |
| 730 | Backup File Size | `backup_size_bytes` | `bigint` | N | N | - | N | - | N | N | N | Y | Total size of generated backup files (bytes) | `1073741824` |
| 731 | Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User requesting manual execution. NULL allowed for scheduled batch executions. | `UUID` |
| 732 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when a manual request was received or a scheduled backup was registered | `2026-09-02 18:00:00+00` |
| 733 | Execution Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the backup job actually started | `2026-09-02 18:00:05+00` |
| 734 | Execution Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the backup job ended in success or final failure | `2026-09-02 18:20:00+00` |
| 735 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of retries after the initial execution failed. Must be 0 or greater. | `0` |
| 736 | Maximum Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Maximum number of automatic retries allowed. Must be 0 or greater. | `3` |
| 737 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next scheduled execution time for a job in RETRY_WAIT status | `2026-09-02 18:30:00+00` |
| 738 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | System error code categorizing the cause of backup failure | `BACKUP_STORAGE_UNAVAILABLE` |
| 739 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Details of backup failure. Sensitive information such as passwords and access keys must not be stored. | `Failed to connect to backup storage` |
| 740 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Backup execution history creation timestamp (UTC) | `2026-09-02 18:00:00+00` |
| 741 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when created by a user. NULL allowed when created by the system or a batch job. | `UUID` |
| 742 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Backup execution history last modification timestamp (UTC) | `2026-09-02 18:20:00+00` |
| 743 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when modified by a user. NULL allowed for system/batch processing. | `UUID` |

[↑ Back to Top](#data-table-specifications)

---

## Library

<a id="table-library_item"></a>
### 9. Library Item Master (`library_item`)

| Item | Definition |
|---|---|
| Description | Master list of reusable standard library items for URS, IQ, and OQ |
| Primary Key | `library_id` |
| Key References (FK) | - |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 91 | Library ID | `library_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique library item identifier | `00000000-0000-0000-0000-000000000001` |
| 92 | Module Type | `module_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | URS \| IQ \| OQ classification | `URS` |
| 93 | Code | `code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Item code (e.g., URS-AT-L01) | `URS-AT-L01` |
| 94 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Audit trail, calibration/verification, security, etc. | `Audit Trail` |
| 95 | Item Name | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Library item name/overview | `Automatic generation of audit trails for data changes` |
| 96 | Requirement/Procedure | `requirement_text` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed requirement specification or execution procedure | `Whenever any data is created, modified, or deleted...` |
| 97 | Expected Result | `expected_result` | `text` | N | N | - | N | - | N | N | N | Y | Expected result for IQ/OQ tests (not used for URS) | - |
| 98 | Acceptance Criteria | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | Criteria for a pass/fail determination | `Automatically generate an Audit Trail when data changes...` |
| 99 | Regulatory Reference | `regulation` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Related regulations (e.g., 21 CFR 11.10(e), KGMP) | `21 CFR 11.10(e)` |
| 100 | Is Active | `is_active` | `boolean` | N | N | - | Y | `True` | N | N | N | Y | Whether active (TRUE/FALSE) | `True` |
| 101 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-25T00:00:00` |
| 102 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-25T00:00:00` |

[↑ Back to Top](#data-table-specifications)

---

## Validation

<a id="table-validation_project"></a>
### 10. Validation Project (`validation_project`)

| Item | Definition |
|---|---|
| Description | Unit and scope of validation execution for each system (VP) |
| Primary Key | `project_id` |
| Key References (FK) | `system_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 103 | Project ID | `project_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique project identifier | `00000000-0000-0000-0000-000000000001` |
| 104 | Project Code | `project_code` | `varchar(100)` | N | N | - | Y | - | Y | Y | N | Y | Project identification code | `VP-SYS-008-20260422` |
| 105 | Project Name | `project_name` | `varchar(200)` | N | N | - | Y | - | N | Y | N | Y | Project Name | `Test Equipment 3 CSV Project` |
| 106 | System ID | `system_id` | `uuid` | N | Y | `system_asset.system_id` | Y | - | N | Y | N | Y | References system_asset.system_id | `00000000-0000-0000-0000-000000000001` |
| 107 | Progress Percentage | `progress_rate` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Project progress (%) | `0` |
| 108 | Status | `status` | `varchar(20)` | N | N | - | Y | `In Progress'` | N | N | N | Y | In Progress \| Completed \| On Hold | `In Progress` |
| 109 | Start Date | `start_date` | `date` | N | N | - | Y | `CURRENT_DATE` | N | N | N | Y | Project start date | `2026-04-15T00:00:00` |
| 110 | Validation Approach | `validation_type` | `varchar(50)` | N | N | - | Y | `Initial Validation'` | N | N | N | Y | Initial Validation \| Change Validation \| Revalidation | `Initial Validation` |
| 111 | Validation Level | `validation_level` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Level 1 \| Level 2 \| Level 3 \| Level 4 | `Level 4` |
| 112 | Project Context Status | `context_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | Confirmation status of the project scope and context: DRAFT, CONFIRMED | `CONFIRMED` |
| 113 | Remarks | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | Additional notes | - |
| 114 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-25T00:00:00` |
| 115 | GAMP Category | `gamp_category` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | GAMP 5 classification (Category 3, 4, 5) | `Category 3` |
| 116 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the project creator | `00000000-0000-0000-0000-000000000001` |
| 117 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 118 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the project modifier | `00000000-0000-0000-0000-000000000001` |
| 119 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-project_member"></a>
### 37. Project Member (`project_member`)

| Item | Definition |
|---|---|
| Description | Manages participating users and assigned roles for each project |
| Primary Key | `project_member_id` |
| Key References (FK) | `project_id, user_id, role_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 535 | Project Member ID | `project_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique project member role mapping identifier. Duplicate (project_id, user_id, role_id) combinations are not allowed in active data. | `UUID` |
| 536 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | ID of the validation project to which the member belongs | `UUID` |
| 537 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user participating in the project | `UUID` |
| 538 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | ID of the role performed by the user within the project | `UUID` |
| 539 | Participation Status | `member_status` | `varchar(20)` | N | N | - | Y | `ACTIVE` | N | Y | N | Y | Project participation status: ACTIVE, INACTIVE, WITHDRAWN | `ACTIVE` |
| 540 | Participation Started At | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time when project participation began | `2026-09-01T10:00:00` |
| 541 | Participation Ended At | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when project participation ended. NULL if currently participating. | - |
| 542 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project participation information creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 543 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who registered the project participation information | `UUID` |
| 544 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project participation information last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 545 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who last modified the project participation information | `UUID` |
| 546 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Project participation information soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-validation_activity"></a>
### 38. Validation Activity Master (`validation_activity`)

| Item | Definition |
|---|---|
| Description | Manages reference information for VP, QIA, VA, URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ, RTM, and VSR activities |
| Primary Key | `activity_id` |
| Key References (FK) | `created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 547 | Activity ID | `activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique validation activity identifier | `UUID` |
| 548 | Activity Code | `activity_code` | `varchar(20)` | N | N | - | Y | - | Y | Y | N | Y | Activity identification code: SYSTEM_IDENTIFICATION, VP, QIA, VA, URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ, RTM, VSR | `URS` |
| 549 | Activity Name | `activity_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Activity name displayed in the UI | `User Requirements Specification` |
| 550 | Activity Order | `display_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Default display order in the UI and business process | `5` |
| 551 | Is Active | `is_active` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether the activity master record is enabled | `True` |
| 552 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Activity master creation timestamp (UTC) | `09/01/2026 10:00:00` |
| 553 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the activity master record | `UUID` |
| 554 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Activity master last modification timestamp (UTC) | `09/01/2026 10:00:00` |
| 555 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the activity master record | `UUID` |
| 556 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Activity master soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-project_activity"></a>
### 39. Project Activity (`project_activity`)

| Item | Definition |
|---|---|
| Description | Manages activities included in each project, their enablement, and progress statuses |
| Primary Key | `project_activity_id` |
| Key References (FK) | `project_id, activity_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 557 | Project Activity ID | `project_activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique project activity identifier. Duplicate (project_id, activity_id) combinations are not allowed in active data. | `UUID` |
| 558 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Validation project to which the activity belongs | `UUID` |
| 559 | Activity ID | `activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | Activity to be performed in the project | `UUID` |
| 560 | Is Included | `is_selected` | `boolean` | N | N | - | Y | `False` | N | N | N | Y | Whether the activity is included in the activities to be performed under the VP | `True` |
| 561 | Is Mandatory Activity | `is_required` | `boolean` | N | N | - | Y | `False` | N | N | N | Y | Whether the activity cannot be omitted from this project | `True` |
| 562 | Activity Status | `activity_status` | `varchar(20)` | N | N | - | Y | `LOCKED` | N | Y | N | Y | Activity status: LOCKED, READY, IN_PROGRESS, COMPLETED, APPROVED, SKIPPED | `READY` |
| 563 | Enabled At | `activated_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the activity became READY after its prerequisites were satisfied | `2026-09-01T10:00:00` |
| 564 | Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Activity execution start time | `2026-09-01T11:00:00` |
| 565 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Activity execution completion time | `2026-09-02T15:00:00` |
| 566 | Approved At | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when final approval of the activity was completed | `2026-09-02T17:00:00` |
| 567 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project activity creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 568 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the project activity | `UUID` |
| 569 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project activity last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 570 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the project activity | `UUID` |
| 571 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Project activity soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-activity_dependency"></a>
### 40. Activity Dependency (`activity_dependency`)

| Item | Definition |
|---|---|
| Description | Manages predecessor activities, relationship classifications, and evaluation conditions for enabling successor activities |
| Primary Key | `activity_dependency_id` |
| Key References (FK) | `successor_activity_id, predecessor_activity_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 572 | Activity Dependency ID | `activity_dependency_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique activity dependency identifier | `UUID` |
| 573 | Successor Activity ID | `successor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | Activity enabled after the conditions are satisfied | `UUID` |
| 574 | Predecessor Activity ID | `predecessor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | N | - | N | Y | N | Y | Activity to check before enabling the successor activity. NULL allowed for conditions covering all activities. | `UUID` |
| 575 | Relationship Classification | `dependency_type` | `varchar(20)` | N | N | - | Y | `REQUIRED` | N | Y | N | Y | Dependency relationship classification: REQUIRED, RECOMMENDED | `REQUIRED` |
| 576 | Required Status | `required_status` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | Status required of the predecessor activity: CREATED, COMPLETED, APPROVED | `APPROVED` |
| 577 | Condition Type | `condition_type` | `varchar(50)` | N | N | - | Y | `STATUS` | N | Y | N | Y | STATUS, ACTIVITY_SELECTED, CONTEXT_CONFIRMED, GXP_SCOPE_CONFIRMED, TRACEABILITY_EXISTS, HIGH_RISK_COVERED, OPEN_DEVIATION_ZERO, ALL_SELECTED_APPROVED | `STATUS` |
| 578 | Condition Value | `condition_value` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Additional value or target entity type required to evaluate the condition | `APPROVED` |
| 579 | Condition Description | `condition_description` | `text` | N | N | - | Y | - | N | N | N | Y | Human-readable description of the enablement condition | `URS approval completed` |
| 580 | Evaluation Order | `evaluation_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Order in which conditions for the same successor activity are evaluated | `1` |
| 581 | Is Active | `is_active` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether the enablement condition is enabled | `True` |
| 582 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Condition creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 583 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the condition | `UUID` |
| 584 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Condition last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 585 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the condition | `UUID` |
| 586 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Condition soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## QIA

<a id="table-qia_assessment"></a>
### 11. Quality Impact Assessment Header (`qia_assessment`)

| Item | Definition |
|---|---|
| Description | Overall QIA assessment and 21 CFR Part 11 assessment results for each project |
| Primary Key | `qia_id` |
| Key References (FK) | `project_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 120 | QIA ID | `qia_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique QIA assessment identifier | `00000000-0000-0000-0000-000000000001` |
| 121 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 122 | Part 11 Q1: Electronic Record Creation | `p11_q1` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Q1: Whether electronic records are created/modified/maintained (Yes/No) | `Yes` |
| 123 | Part 11 Q2: Electronic Storage | `p11_q2` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Q2: Whether records are stored electronically (Yes/No) | `Yes` |
| 124 | Part 11 Q3: Submission to Regulatory Authorities | `p11_q3` | `varchar(10)` | N | N | - | Y | `No'` | N | N | N | Y | Q3: Whether electronic records are submitted to regulatory authorities (Yes/No) | `No` |
| 125 | Part 11 Q4: Electronic Signature Use | `p11_q4` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Q4: Whether electronic signatures are used (Yes/No) | `Yes` |
| 126 | Part 11 Q5: Replacement of Handwritten Signatures | `p11_q5` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Q5: Whether electronic signatures replace handwritten signatures (Yes/No) | `Yes` |
| 127 | Part 11 Q6: System Type | `p11_q6` | `varchar(20)` | N | N | - | Y | `Closed'` | N | N | N | Y | Closed System \| Open System | `Closed` |
| 128 | Part 11 Assessment Conclusion | `part11_result` | `text` | N | N | - | Y | - | N | N | N | Y | Conclusion on the applicability of 21 CFR Part 11 requirements | `System subject to 21 CFR Part 11 (Closed System)` |
| 129 | GxP Scope Status | `gxp_scope_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | Confirmation status of the GxP scope determined through QIA: DRAFT, CONFIRMED | `CONFIRMED` |
| 130 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the QIA assessment document (e.g., v1.0, v1.1) | `v1.0` |
| 131 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the QIA assessment document (e.g., 1, 2, 3...) | `1` |
| 132 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the QIA assessment document | `Initial creation` |
| 133 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether this is the currently active latest QIA assessment document version (TRUE/FALSE) | `True` |
| 134 | Authoring/Review/Approval Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Document progress status (DRAFT, REVIEW, APPROVED, etc.) | `DRAFT` |
| 135 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-qia_module_item"></a>
### 12. QIA Module Detailed Assessment (`qia_module_item`)

| Item | Definition |
|---|---|
| Description | GxP Q1–Q10 items and assessment results for each module/process |
| Primary Key | `qia_module_item_id` |
| Key References (FK) | `qia_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 136 | QIA Module Item ID | `qia_module_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique QIA module assessment item identifier | `UUID` |
| 137 | QIA ID | `qia_id` | `uuid` | N | Y | `qia_assessment.qia_id` | Y | - | N | Y | N | Y | Parent QIA assessment identifier | `UUID` |
| 138 | Module Code | `module_code` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Parent module code | `QM` |
| 139 | Module Name | `module_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Parent module name | `Quality Control` |
| 140 | Module Description | `module_description` | `text` | N | N | - | N | - | N | N | N | Y | Description of the module's scope and purpose | `Overall assessment related to quality control` |
| 141 | Process Code | `process_code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Process identification code | `PRC-001` |
| 142 | Process Name | `process_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Detailed process name | `Work Order` |
| 143 | Q1 GxP Purpose | `q1_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `O` |
| 144 | Q2 Production Process | `q2_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `O` |
| 145 | Q3 Production Data | `q3_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 146 | Q4 Quality Impact | `q4_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `O` |
| 147 | Q5 Storage Management | `q5_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 148 | Q6 Release Approval | `q6_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 149 | Q7 Recall | `q7_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 150 | Q8 Regulatory Documents | `q8_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 151 | Q9 Document Management | `q9_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 152 | Q10 Security | `q10_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 153 | Assessment Result | `result_type` | `varchar(20)` | N | N | - | Y | `Non-GxP'` | N | N | N | Y | GxP \| Non-GxP | `GxP` |
| 154 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T10:00:00` |
| 155 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T10:00:00` |

[↑ Back to Top](#data-table-specifications)

---

## VA

<a id="table-vendor_audit"></a>
### 13. Vendor Audit Assessment (`vendor_audit`)

| Item | Definition |
|---|---|
| Description | Records vendor inspection/audit plans, execution, and deficiency counts |
| Primary Key | `audit_id` |
| Key References (FK) | `project_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 156 | Audit ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique vendor audit identifier | `00000000-0000-0000-0000-000000000001` |
| 157 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 158 | Document Number | `document_number` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Audit report document number | `VA-2026-001` |
| 159 | Vendor Name | `vendor_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Name of the vendor being audited | `Sample Vendor` |
| 160 | Target System | `system_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Name/version of the system being audited | `Sample System v1.0` |
| 161 | Audit Method | `audit_type` | `varchar(50)` | N | N | - | Y | `On-site Audit'` | N | N | N | Y | On-site Audit \| Document Audit \| Remote Audit | `On-site Audit` |
| 162 | Audit Date | `audit_date` | `date` | N | N | - | Y | `CURRENT_DATE` | N | N | N | Y | Date of audit execution (or scheduled date) | `2026-08-26T00:00:00` |
| 163 | Auditor | `auditor_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Name of the person responsible for/conducting the audit | `Hong Gil-dong` |
| 164 | Audit Result | `audit_result` | `varchar(20)` | N | N | - | Y | `Compliant'` | N | N | N | Y | Compliant \| Conditionally Compliant \| Noncompliant | `Compliant` |
| 165 | Critical Deficiency Count | `critical_defects` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of critical deficiencies identified | `0` |
| 166 | Major Deficiency Count | `major_defects` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of major deficiencies identified | `0` |
| 167 | Minor Deficiency Count | `minor_defects` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of minor deficiencies identified | `1` |
| 168 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the audit document (e.g., v1.0, v1.1) | `v1.0` |
| 169 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the audit document (e.g., 1, 2, 3...) | `1` |
| 170 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the audit document | `Initial creation` |
| 171 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether this is the currently active latest audit document version (TRUE/FALSE) | `True` |
| 172 | Status | `status` | `varchar(20)` | N | N | - | Y | `Draft'` | N | N | N | Y | Draft \| Completed \| Approved | `Draft` |
| 173 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 174 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |

[↑ Back to Top](#data-table-specifications)

---

## URS

<a id="table-requirement"></a>
### 14. User Requirements Specification (`requirement`)

| Item | Definition |
|---|---|
| Description | Manages URS requirement items and document revision versions for each project |
| Primary Key | `requirement_id` |
| Key References (FK) | `project_id, created_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 175 | Requirement ID | `requirement_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique requirement identifier | `00000000-0000-0000-0000-000000000001` |
| 176 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 177 | Item Number | `item_number` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Requirement management number (e.g., URS-001) | `URS-001` |
| 178 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | System management, audit trail, electronic signatures, etc. | `Electronic Signature` |
| 179 | Item/Function | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Requirement item name and main functionality | `Record electronic signature signer, date/time, and meaning` |
| 180 | Requirement Details | `requirement_text` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed requirement specification | `When electronically signing, the signer ID, signing date/time...` |
| 181 | Regulatory Reference | `regulation` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Related laws and regulations (e.g., 21 CFR 11.50) | `21 CFR 11.50` |
| 182 | Status | `status` | `varchar(20)` | N | N | - | Y | `Draft'` | N | N | N | Y | Draft \| Under Review \| Approved \| Rejected | `Draft` |
| 183 | Document Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0'` | N | N | N | Y | Requirement document revision version (e.g., v1.0, v2.0) | `v1.0` |
| 184 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Version revision sequence number (1, 2, 3...) | `1` |
| 185 | Is Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether this is the currently active latest version (TRUE/FALSE) | `True` |
| 186 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating/revising the version | `Initial creation` |
| 187 | Is Linked to RTM | `is_rtm_linked` | `boolean` | N | N | - | Y | `False` | N | N | N | Y | Whether linked to the Traceability Matrix | `False` |
| 188 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the requirement author | `00000000-0000-0000-0000-000000000001` |
| 189 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 190 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |

[↑ Back to Top](#data-table-specifications)

---

## FDS

<a id="table-fds_spec"></a>
### 15. Functional Design Specification (`fds_spec`)

| Item | Definition |
|---|---|
| Description | Manages FDS document headers, versions, and statuses for each project |
| Primary Key | `fds_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 191 | FDS ID | `fds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | FDS identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 192 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 193 | FDS Number | `fds_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | FDS document number | `VP-SYS-008-20260422` |
| 194 | FDS Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the functional design specification | `Test Equipment 3 CSV Project` |
| 195 | Document Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | Standard FDS document version | `v1.0` |
| 196 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Version revision sequence number (e.g., 1, 2, 3...) | `1` |
| 197 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating/revising the version | `Initial creation` |
| 198 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether this is the currently active latest version (TRUE/FALSE) | `True` |
| 199 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Document status (Draft, Under Review, Approved) | `Draft` |
| 200 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 201 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the FDS author | `00000000-0000-0000-0000-000000000001` |
| 202 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 203 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the FDS modifier | `00000000-0000-0000-0000-000000000001` |
| 204 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-fds_item"></a>
### 16. FDS Detailed Item (`fds_item`)

| Item | Definition |
|---|---|
| Description | Manages functional, screen, and detailed design items in FDS documents |
| Primary Key | `fds_item_id` |
| Key References (FK) | `fds_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 205 | FDS Item ID | `fds_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | FDS item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 206 | FDS ID | `fds_id` | `uuid` | N | Y | `fds_spec.fds_id` | Y | - | N | N | N | Y | Parent FDS document identifier | `00000000-0000-0000-0000-000000000001` |
| 207 | Item Type | `item_type` | `varchar(20)` | N | N | - | Y | `FUNCTION` | N | N | N | Y | Subtab classification (FUNCTION, SCREEN, INTERFACE) | `FUNCTION` |
| 208 | FDS Item Number | `fds_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | FDS item management number | `FDS-001` |
| 209 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Function Category | `System Management` |
| 210 | Item/Function | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Function name and main functionality | `Electronic Signature` |
| 211 | Description | `description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed function description | `Electronic signature verification during user login` |
| 212 | Related Screen | `related_screen` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Name of the related screen | `User Management Screen` |
| 213 | Individual Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Revision sequence number (1, 2, 3...) | `1` |
| 214 | Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Progress status (Draft, Under Review, Approved) | `Draft` |
| 215 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 216 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 217 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 218 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 219 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-fds_interface"></a>
### 17. FDS Interface Definition (`fds_interface`)

| Item | Definition |
|---|---|
| Description | Manages inter-system interfaces, exchanged data, and transmission methods in FDS documents |
| Primary Key | `fds_interface_id` |
| Key References (FK) | `fds_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 220 | FDS Interface ID | `fds_interface_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique FDS interface record identifier | `UUID` |
| 221 | FDS ID | `fds_id` | `uuid` | N | Y | `fds_spec.fds_id` | Y | - | N | Y | N | Y | Parent FDS document identifier | `UUID` |
| 222 | Interface Management Number | `interface_id` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Interface management number within the FDS document | `IF-001` |
| 223 | Source System | `source_system` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Name of the sending system | `Sample System` |
| 224 | Target System | `target_system` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Name of the receiving system | `Target System` |
| 225 | Exchanged Data | `interface_data` | `text` | N | N | - | Y | - | N | N | N | Y | Data items to be transmitted | `Material consumption, batch results` |
| 226 | Integration Frequency | `transfer_cycle` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Real-time, periodic, event-driven, etc. | `Real-time` |
| 227 | Integration Method | `transfer_method` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | REST API, Message Queue, etc. | `REST API` |
| 228 | Linked FDS Number | `fds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Display value of the related FDS item number | `FDS-008` |
| 229 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T10:00:00` |
| 230 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the interface author | `UUID` |
| 231 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T10:00:00` |
| 232 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the user who last modified the interface | `UUID` |
| 233 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## DQ

<a id="table-dq_assessment"></a>
### 18. Design Qualification Assessment (`dq_assessment`)

| Item | Definition |
|---|---|
| Description | Manages design qualification assessment document headers for each project |
| Primary Key | `dq_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 234 | DQ ID | `dq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | DQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 235 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 236 | DQ Number | `dq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | DQ document number (e.g., DQ-VP-SYS-008-20260422) | `DQ-VP-SYS-008-20260422` |
| 237 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the design qualification assessment document | `Design Qualification Assessment (URS → FDS/DDS Mapping)` |
| 238 | Document Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | Standard DQ document version | `v1.0` |
| 239 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the DQ assessment document (e.g., 1, 2, 3...) | `1` |
| 240 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the DQ assessment document | `Initial creation` |
| 241 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether this is the currently active latest DQ assessment document version (TRUE/FALSE) | `True` |
| 242 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Document status (Draft, Under Review, Approved) | `Draft` |
| 243 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 244 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the DQ author | `00000000-0000-0000-0000-000000000001` |
| 245 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 246 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the DQ modifier | `00000000-0000-0000-0000-000000000001` |
| 247 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-dq_item"></a>
### 19. DQ Detailed Assessment Item (`dq_item`)

| Item | Definition |
|---|---|
| Description | Manages qualification assessment results for URS requirements and FDS/DDS designs |
| Primary Key | `dq_item_id` |
| Key References (FK) | `dq_id, requirement_id, reviewed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 248 | DQ Item ID | `dq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Detailed DQ item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 249 | DQ ID | `dq_id` | `uuid` | N | Y | `dq_assessment.dq_id` | Y | - | N | N | N | Y | Parent DQ document identifier | `00000000-0000-0000-0000-000000000001` |
| 250 | URS Item ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | N | N | Y | Identifier of the URS requirement to be mapped (FK) | `00000000-0000-0000-0000-000000000001` |
| 251 | URS Number | `urs_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | URS number for display in the UI (e.g., URS-001) | `URS-001` |
| 252 | URS Requirement | `urs_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed URS requirement content | `User login and electronic signature functionality` |
| 253 | FDS Link | `fds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked FDS number (e.g., FDS-001) | `FDS-001` |
| 254 | FDS Function | `fds_feature_name` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Name of the linked FDS function | `Electronic signature verification` |
| 255 | DDS Link | `dds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked DDS design number | `DDS-001` |
| 256 | DDS Design | `dds_description` | `text` | N | N | - | N | - | N | N | N | Y | Linked DDS database/component design | `User Auth Table Schema` |
| 257 | Outcome | `result_status` | `varchar(20)` | N | N | - | Y | `Pending Review` | N | N | N | Y | Assessment outcome (PASS, FAIL, PENDING) | `PASS` |
| 258 | Reviewer ID | `reviewed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | Identifier of the user who reviewed the item | `00000000-0000-0000-0000-000000000001` |
| 259 | Remarks | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | Review-related observations and remarks | `Confirmed incorporation into FDS and DDS designs` |
| 260 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 261 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 262 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 263 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 264 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## FRA

<a id="table-fra_assessment"></a>
### 20. Functional Risk Assessment (`fra_assessment`)

| Item | Definition |
|---|---|
| Description | Manages functional risk assessment document headers for each project |
| Primary Key | `fra_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 265 | FRA ID | `fra_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | FRA assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 266 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 267 | FRA Number | `fra_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | FRA document number (e.g., FRA-VP-SYS-008-20260422) | `FRA-VP-SYS-008-20260422` |
| 268 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the FMEA-based functional risk assessment document | `FMEA-Based Functional Risk Assessment` |
| 269 | Document Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | Standard FRA document version | `v1.0` |
| 270 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the FRA assessment document (e.g., 1, 2, 3...) | `1` |
| 271 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the FRA assessment document | `Initial creation` |
| 272 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether this is the currently active latest FRA assessment document version (TRUE/FALSE) | `True` |
| 273 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Document status (Draft, Under Review, Approved) | `Draft` |
| 274 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 275 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the FRA author | `00000000-0000-0000-0000-000000000001` |
| 276 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 277 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the FRA modifier | `00000000-0000-0000-0000-000000000001` |
| 278 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-fra_item"></a>
### 21. FRA Detailed Risk Item (`fra_item`)

| Item | Definition |
|---|---|
| Description | Manages risk scenarios, risk scores, and mitigation strategies for each function |
| Primary Key | `fra_item_id` |
| Key References (FK) | `fra_id, requirement_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 279 | Risk Item ID | `fra_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Detailed FRA risk item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 280 | FRA ID | `fra_id` | `uuid` | N | Y | `fra_assessment.fra_id` | Y | - | N | N | N | Y | Parent FRA document identifier | `00000000-0000-0000-0000-000000000001` |
| 281 | URS Item ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | N | - | N | N | N | Y | Identifier of the mapped URS requirement (FK) | `00000000-0000-0000-0000-000000000001` |
| 282 | URS Reference Number | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | URS reference number for display in the UI (e.g., URS-001) | `URS-001` |
| 283 | Function Name | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Name of the function being assessed for risk | `Electronic Signature` |
| 284 | Risk Scenario | `risk_scenario` | `text` | N | N | - | Y | - | N | N | N | Y | Description of the FMEA risk occurrence scenario | `Password verification not performed during electronic signing` |
| 285 | Product Impact (PI) | `pi_score` | `varchar(10)` | N | N | - | Y | `H` | N | N | N | Y | Product Impact (H, M, L) | `H` |
| 286 | Likelihood of Occurrence (LL) | `ll_score` | `varchar(10)` | N | N | - | Y | `H` | N | N | N | Y | Likelihood of Occurrence (H, M, L) | `M` |
| 287 | Detectability (DL) | `dl_score` | `varchar(10)` | N | N | - | Y | `H` | N | N | N | Y | Detectability (H, M, L) | `L` |
| 288 | Risk Value (RV) | `risk_value` | `integer` | N | N | - | N | `1` | N | N | N | Y | Risk value (RV) calculated from the combination of PI, LL, and DL | `1` |
| 289 | Risk Rating | `risk_level` | `varchar(20)` | N | N | - | Y | `LOW` | N | Y | N | Y | Calculated risk rating: LOW, MEDIUM, HIGH | `HIGH` |
| 290 | Mitigation Strategy | `mitigation_strategy` | `varchar(20)` | N | N | - | Y | `Test` | N | N | N | Y | Risk mitigation strategy (Test, SOP, No Action) | `Test` |
| 291 | Linked Test | `test_reference` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked verification test code (e.g., OQ-AT-01) | `OQ-AT-01` |
| 292 | Progress Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Risk item status (Draft, Review Completed) | `Draft` |
| 293 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 294 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 295 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 296 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 297 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## IQ

<a id="table-iq_assessment"></a>
### 22. Installation Qualification Assessment (`iq_assessment`)

| Item | Definition |
|---|---|
| Description | Manages installation qualification assessment document headers for each project |
| Primary Key | `iq_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 298 | IQ ID | `iq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | IQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 299 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 300 | IQ Number | `iq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | IQ document number (e.g., IQ-VP-SYS-010-20260529) | `IQ-VP-SYS-010-20260529` |
| 301 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the installation qualification assessment document | `Installation Qualification` |
| 302 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the IQ assessment document (e.g., v1.0, v1.1) | `v1.0` |
| 303 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the IQ assessment document (e.g., 1, 2, 3...) | `1` |
| 304 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the IQ assessment document | `Initial creation` |
| 305 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether this is the currently active latest IQ assessment document version (TRUE/FALSE) | `True` |
| 306 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | IQ protocol status: DRAFT, REVIEW, APPROVED | `APPROVED` |
| 307 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | IQ execution record status. After the protocol is APPROVED, progresses through DRAFT, REVIEW, and APPROVED. | `DRAFT` |
| 308 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 309 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the IQ author | `00000000-0000-0000-0000-000000000001` |
| 310 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 311 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the IQ modifier | `00000000-0000-0000-0000-000000000001` |
| 312 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-iq_item"></a>
### 23. IQ Detailed Test Item (`iq_item`)

| Item | Definition |
|---|---|
| Description | Manages IQ test procedures, results, evidence, and executors |
| Primary Key | `iq_item_id` |
| Key References (FK) | `iq_id, executed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 313 | IQ Item ID | `iq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Detailed IQ test item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 314 | IQ ID | `iq_id` | `uuid` | N | Y | `iq_assessment.iq_id` | Y | - | N | N | N | Y | Parent IQ document identifier | `00000000-0000-0000-0000-000000000001` |
| 315 | Procedure Number | `step_no` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Test execution procedure number (e.g., 01, 02) | `1` |
| 316 | Test ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Test item ID (e.g., IQ-NEW-01) | `IQ-NEW-01` |
| 317 | Test Case | `test_case` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Hardware installation, software installation, etc. | `Hardware Installation` |
| 318 | URS Link | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Mapped URS number (e.g., URS-001) | `URS-001` |
| 319 | Test Details | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed procedure for executing the verification test | `Verify that the hardware specifications of the server to be installed meet the URS requirements` |
| 320 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Test success criteria and expected results | `Hardware specifications must match the requirements specified in the URS` |
| 321 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `Approved` | N | N | N | Y | Protocol approval status for each item | `Approved` |
| 322 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Record of actual results after test execution | `IQ Test 1 results` |
| 323 | Outcome | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Final test outcome (Pass, Fail, Not Executed) | `Pass` |
| 324 | Actor ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | Test executor identifier (FK) | `00000000-0000-0000-0000-000000000001` |
| 325 | Executed At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Date and time when test execution was completed | `2026-06-22T00:00:00` |
| 326 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Record approval status for each item | `Approved` |
| 327 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 328 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 329 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 330 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 331 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## OQ

<a id="table-oq_assessment"></a>
### 24. Operational Qualification Assessment (`oq_assessment`)

| Item | Definition |
|---|---|
| Description | Manages operational qualification assessment document headers for each project |
| Primary Key | `oq_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 332 | OQ ID | `oq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | OQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 333 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 334 | OQ Number | `oq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | OQ document number (e.g., OQ-VP-SYS-010-20260529) | `OQ-VP-SYS-010-20260529` |
| 335 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the operational qualification assessment document | `Operational Qualification` |
| 336 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the OQ assessment document (e.g., v1.0, v1.1) | `v1.0` |
| 337 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the OQ assessment document (e.g., 1, 2, 3...) | `1` |
| 338 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the OQ assessment document | `Initial creation` |
| 339 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether this is the currently active latest OQ assessment document version (TRUE/FALSE) | `True` |
| 340 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | OQ protocol status: DRAFT, REVIEW, APPROVED | `APPROVED` |
| 341 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | OQ execution record status. After the protocol is APPROVED, progresses through DRAFT, REVIEW, and APPROVED. | `DRAFT` |
| 342 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 343 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the OQ author | `00000000-0000-0000-0000-000000000001` |
| 344 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 345 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the OQ modifier | `00000000-0000-0000-0000-000000000001` |
| 346 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-oq_item"></a>
### 25. OQ Detailed Test Item (`oq_item`)

| Item | Definition |
|---|---|
| Description | Manages OQ test procedures, results, evidence, and executors |
| Primary Key | `oq_item_id` |
| Key References (FK) | `oq_id, executed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 347 | OQ Item ID | `oq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Detailed OQ test item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 348 | OQ ID | `oq_id` | `uuid` | N | Y | `oq_assessment.oq_id` | Y | - | N | N | N | Y | Parent OQ document identifier | `00000000-0000-0000-0000-000000000001` |
| 349 | Procedure Number | `step_no` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Test execution procedure number (e.g., 01, 02, 03) | `1` |
| 350 | Test ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Test item ID (e.g., OQ-AT-L01) | `OQ-AT-L01` |
| 351 | Test Case | `test_case` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Audit Trail, user management, backup and recovery, etc. | `Audit Trail` |
| 352 | URS Link | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Mapped URS number (e.g., URS-001) | `URS-001` |
| 353 | Test Details | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed procedure for executing the verification test | `Automatically generate an audit trail when user data changes` |
| 354 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Test success criteria and expected results | `Record before/after values, user, date/time, and IP address` |
| 355 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `Approved` | N | N | N | Y | Protocol approval status for each item | `Approved` |
| 356 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Record of actual results after test execution | `Actual result Test` |
| 357 | Outcome | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Final test outcome (Pass, Fail, Not Executed) | `Pass` |
| 358 | Actor ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | Test executor identifier (FK) | `00000000-0000-0000-0000-000000000001` |
| 359 | Executed At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Date and time when test execution was completed | `2026-06-22T00:00:00` |
| 360 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Record approval status for each item | `Approved` |
| 361 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 362 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 363 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 364 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 365 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## PQ

<a id="table-pq_assessment"></a>
### 26. Performance Qualification Assessment (`pq_assessment`)

| Item | Definition |
|---|---|
| Description | Manages performance qualification assessment document headers and execution plans for each project |
| Primary Key | `pq_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 366 | PQ ID | `pq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | PQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 367 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 368 | PQ Number | `pq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | PQ document number (e.g., PQ-VP-SYS-008-20260422) | `PQ-VP-SYS-008-20260422` |
| 369 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the performance qualification assessment document | `Performance Qualification` |
| 370 | Planned PQ Start Date | `start_scheduled_date` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Basis for the planned PQ start date (e.g., OQ completion + 5 business days) | `OQ completion + 5 business days` |
| 371 | Target PQ Completion Date | `target_completion_date` | `date` | N | N | - | N | - | N | N | N | Y | Target date for PQ completion | `2024-04-15T00:00:00` |
| 372 | Execution Method | `execution_method` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | PQ execution method (e.g., actual production data from at least 3 batches) | `Actual production data from at least 3 batches` |
| 373 | Primary Executor | `primary_executor` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Team/department primarily responsible for execution (e.g., Process Owner (Production Team)) | `Process Owner (Production Team)` |
| 374 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the PQ assessment document (e.g., v1.0, v1.1) | `v1.0` |
| 375 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the PQ assessment document (e.g., 1, 2, 3...) | `1` |
| 376 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the PQ assessment document | `Initial creation` |
| 377 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether this is the currently active latest PQ assessment document version (TRUE/FALSE) | `True` |
| 378 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Overall PQ assessment progress status (Awaiting Execution, In Progress, Completed) | `Awaiting PQ Execution` |
| 379 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | PQ protocol status: DRAFT, REVIEW, APPROVED | `APPROVED` |
| 380 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | PQ execution record status. After the protocol is APPROVED, progresses through DRAFT, REVIEW, APPROVED. | `DRAFT` |
| 381 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 382 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the PQ author | `00000000-0000-0000-0000-000000000001` |
| 383 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 384 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the PQ modifier | `00000000-0000-0000-0000-000000000001` |
| 385 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-pq_item"></a>
### 27. PQ Detailed Test Item (`pq_item`)

| Item | Definition |
|---|---|
| Description | Manages PQ test procedures, results, evidence, and executors |
| Primary Key | `pq_item_id` |
| Key References (FK) | `pq_id, executed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 386 | PQ Item ID | `pq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Detailed PQ test item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 387 | PQ ID | `pq_id` | `uuid` | N | Y | `pq_assessment.pq_id` | Y | - | N | N | N | Y | Parent PQ document identifier | `00000000-0000-0000-0000-000000000001` |
| 388 | Procedure Number | `step_no` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Test execution procedure number (e.g., 01, 02) | `1` |
| 389 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | PQ assessment category (e.g., consecutive batch verification, data integrity, etc.) | `Consecutive Batch Verification` |
| 390 | Test Details | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed procedure for executing the PQ verification test | `Verify successful completion of the production process for at least 3 consecutive batches` |
| 391 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Test success criteria and expected results | `All batches must complete production successfully in accordance with specifications` |
| 392 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Record of actual results after test execution | `Confirmed successful completion of 3 consecutive batches` |
| 393 | Outcome | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Final test outcome (Pass, Fail, Not Executed) | `Pass` |
| 394 | Actor ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | Test executor identifier (FK) | `00000000-0000-0000-0000-000000000001` |
| 395 | Executed At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Date and time when test execution was completed | `2026-08-26T00:00:00` |
| 396 | Progress Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Progress status for each item (Draft, Approved) | `Draft` |
| 397 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 398 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 399 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 400 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 401 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## RTM

<a id="table-rtm_assessment"></a>
### 28. Requirements Traceability Matrix (`rtm_assessment`)

| Item | Definition |
|---|---|
| Description | Manages project-level requirements traceability, aggregated deliverable coverage, and RTM snapshots at the time of approval |
| Primary Key | `rtm_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 402 | RTM ID | `rtm_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | RTM assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 403 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 404 | RTM Number | `rtm_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | RTM document number (e.g., RTM-VP-SYS-008-20260422) | `RTM-VP-SYS-008-20260422` |
| 405 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the requirements traceability matrix document | `Requirements Traceability Matrix (Automatically Generated)` |
| 406 | Total URS Count | `total_urs_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Total number of URS requirements in the project | `8` |
| 407 | FRA Linkage Rate | `fra_link_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | FRA risk assessment linkage rate (%) | `88` |
| 408 | IQ Coverage | `iq_coverage_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | IQ test coverage (%) | `0` |
| 409 | OQ Coverage | `oq_coverage_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | OQ test coverage (%) | `44` |
| 410 | Overall Average Coverage | `avg_coverage_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | Integrated average coverage across all deliverables (%) | `44` |
| 411 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the RTM assessment document (e.g., v1.0, v1.1) | `v1.0` |
| 412 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the RTM assessment document (e.g., 1, 2, 3...) | `1` |
| 413 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the RTM assessment document | `Initial creation` |
| 414 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether this is the currently active latest RTM assessment document version (TRUE/FALSE) | `True` |
| 415 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | RTM progress status (Draft, Approved) | `Draft` |
| 416 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 417 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the RTM author | `00000000-0000-0000-0000-000000000001` |
| 418 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 419 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the RTM modifier | `00000000-0000-0000-0000-000000000001` |
| 420 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-rtm_item"></a>
### 29. RTM Detailed Traceability Item (`rtm_item`)

| Item | Definition |
|---|---|
| Description | Manages FRA, FDS, DDS, IQ, OQ, and PQ traceability results for each URS requirement generated from traceability_link, and detailed snapshots at the time of approval |
| Primary Key | `rtm_item_id` |
| Key References (FK) | `rtm_id, requirement_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 421 | RTM Item ID | `rtm_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | RTM matrix item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 422 | RTM ID | `rtm_id` | `uuid` | N | Y | `rtm_assessment.rtm_id` | Y | - | N | N | N | Y | Parent RTM document identifier | `00000000-0000-0000-0000-000000000001` |
| 423 | URS Item ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | N | N | Y | Identifier of the mapped URS requirement (FK) | `00000000-0000-0000-0000-000000000001` |
| 424 | URS Number | `urs_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | URS number for display in the UI (e.g., URS-001) | `URS-001` |
| 425 | Item/Function | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | URS requirement function name | `Record electronic signature signer, date/time, and meaning` |
| 426 | Requirement Content | `requirement_desc` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed URS requirement content | `The signer ID and signing date/time must be recorded when an electronic signature is executed.` |
| 427 | Is Adopted | `adoption_status` | `varchar(10)` | N | N | - | Y | `O` | N | N | N | Y | Whether the URS requirement is adopted (O, X) | `O` |
| 428 | FRA Mapping | `fra_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked FRA number (e.g., FRA-001, N/A) | `N/A` |
| 429 | FDS Mapping | `fds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked FDS number (e.g., FDS-001, Mapping Failed) | `Mapping Failed` |
| 430 | DDS Mapping | `dds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked DDS number (e.g., DDS-001, Mapping Failed) | `Mapping Failed` |
| 431 | IQ Result | `iq_result` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked IQ test item number (e.g., IQ-NEW-07) | `IQ-NEW-07` |
| 432 | OQ Result | `oq_result` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked OQ test item number (e.g., OQ-AT-L01, Mapping Failed) | `Mapping Failed` |
| 433 | PQ Result | `pq_result` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked PQ test item number (e.g., PQ-01, N/A) | `N/A` |
| 434 | Item Coverage | `item_coverage_rate` | `numeric(5,2)` | N | N | - | Y | `0` | N | N | N | Y | Deliverable coverage percentage for the URS item (%) | `50` |
| 435 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 436 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 437 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 438 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 439 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## VSR

<a id="table-vsr_assessment"></a>
### 30. Validation Summary Report (`vsr_assessment`)

| Item | Definition |
|---|---|
| Description | Manages the final validation conclusion and summary report for each project |
| Primary Key | `vsr_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 440 | VSR ID | `vsr_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | VSR assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 441 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 442 | VSR Number | `vsr_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | VSR document number (e.g., VSR-VP-SYS-008-20260422) | `VSR-VP-SYS-008-20260422` |
| 443 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the validation summary report | `Validation Summary Report` |
| 444 | Validation Conclusion | `overall_conclusion` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Final acceptability conclusion (Acceptable, Conditionally Acceptable, Unacceptable) | `Conditionally Acceptable (Conditionally Acceptable)` |
| 445 | Detailed Conclusion | `conclusion_remarks` | `text` | N | N | - | N | - | N | N | N | Y | Reasons and conditions for the conclusion (e.g., final approval possible after resolution of deviation OQ-GMP-02) | `Final approval possible after resolution of deviation OQ-GMP-02` |
| 446 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the VSR assessment document (e.g., v1.0, v1.1) | `v1.0` |
| 447 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the VSR assessment document (e.g., 1, 2, 3...) | `1` |
| 448 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the VSR assessment document | `Initial creation` |
| 449 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether this is the currently active latest VSR assessment document version (TRUE/FALSE) | `True` |
| 450 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `Under Review` | N | N | N | Y | VSR progress status (Draft, Under Review, Approved) | `Under Review` |
| 451 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 452 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the VSR author | `00000000-0000-0000-0000-000000000001` |
| 453 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 454 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the VSR modifier | `00000000-0000-0000-0000-000000000001` |
| 455 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-vsr_item"></a>
### 31. VSR Activity Summary Item (`vsr_item`)

| Item | Definition |
|---|---|
| Description | Manages summaries of documents, results, deviations, and approval information for each validation activity |
| Primary Key | `vsr_item_id` |
| Key References (FK) | `vsr_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 456 | VSR Item ID | `vsr_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | VSR activity summary item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 457 | VSR ID | `vsr_id` | `uuid` | N | Y | `vsr_assessment.vsr_id` | Y | - | N | N | N | Y | Parent VSR document identifier | `00000000-0000-0000-0000-000000000001` |
| 458 | Activity Type | `activity_code` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Validation activity type (VP, QIA, VA, URS, FDS, DQ, FRA, IQ, OQ, PQ, RTM) | `URS` |
| 459 | Document Number | `doc_no` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Document number for the corresponding stage | `VP-SYS-008-20260422 · URS` |
| 460 | Revision Number | `revision_no` | `varchar(20)` | N | N | - | Y | `1` | N | N | N | Y | Document revision number (REV) | `2.1` |
| 461 | Executed At | `execution_date` | `date` | N | N | - | N | - | N | N | N | Y | Activity execution completion date | `2024-02-15T00:00:00` |
| 462 | Pass Count | `pass_count` | `integer` | N | N | - | N | - | N | N | N | Y | Number of PASS tests/requirements | `125` |
| 463 | Fail Count | `fail_count` | `integer` | N | N | - | N | - | N | N | N | Y | Number of FAIL results | `0` |
| 464 | Deviation Count | `deviation_info` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Number of deviations raised and resolved (e.g., 3 deviations (2 resolved), 1 (open)) | - |
| 465 | Conclusion/Status | `item_status` | `varchar(20)` | N | N | - | Y | `Pending` | N | N | N | Y | Conclusion for the document/activity (Approved, Under Review, Pending) | `Approved` |
| 466 | Approver Name | `approver_name` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Full name of the final approver | `Hong Gil-dong` |
| 467 | Approval Date | `approval_date` | `date` | N | N | - | N | - | N | N | N | Y | Final approval date | `2024-02-20T00:00:00` |
| 468 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 469 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 470 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 471 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 472 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## Workflow

<a id="table-workflow_instance"></a>
### 32. Workflow Instance (`workflow_instance`)

| Item | Definition |
|---|---|
| Description | Manages review and approval workflow progress information for each document |
| Primary Key | `workflow_instance_id` |
| Key References (FK) | `requested_by, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 473 | Workflow Instance ID | `workflow_instance_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the document's review and approval workflow | `UUID` |
| 474 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Physical table name of the document subject to the workflow | `fds_spec` |
| 475 | Target Record ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | N | N | Y | Primary key value of the document revision row subject to the workflow | `UUID` |
| 476 | Target Document Version | `target_version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the document revision subject to review and approval | `v1.0` |
| 477 | Submitted By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | User who submitted the document for review and approval | `UUID` |
| 478 | Submitted At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time when the document was submitted for review and approval | `2026-08-31T15:00:00` |
| 479 | Workflow Status | `workflow_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Overall workflow status: DRAFT, IN_PROGRESS, APPROVED, REJECTED, CANCELLED | `IN_PROGRESS` |
| 480 | Current Step Order | `current_step_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Order of the review/approval step currently being processed | `1` |
| 481 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the workflow ended through final approval, rejection, or cancellation | `2026-08-31T17:00:00` |
| 482 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-31T15:00:00` |
| 483 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Creator of the workflow record | `UUID` |
| 484 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-31T16:00:00` |
| 485 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | User who last modified the workflow record | `UUID` |
| 486 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-workflow_step"></a>
### 33. Workflow Step (`workflow_step`)

| Item | Definition |
|---|---|
| Description | Manages processing statuses and assignees for individual workflow steps |
| Primary Key | `workflow_step_id` |
| Key References (FK) | `workflow_instance_id, assignee_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 487 | Workflow Step ID | `workflow_step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique workflow review/approval step identifier. Duplicate (workflow_instance_id, step_order) combinations are not allowed in active data. | `UUID` |
| 488 | Workflow Instance ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | Y | - | N | Y | N | Y | Identifier of the workflow to which the step belongs | `UUID` |
| 489 | Step Order | `step_order` | `integer` | N | N | - | Y | - | N | Y | N | Y | Review/approval processing order within the same workflow | `1` |
| 490 | Step Type | `step_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Processing step type: REVIEW or APPROVE | `REVIEW` |
| 491 | Step Name | `step_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Review/approval step name displayed in the UI | `Quality Review` |
| 492 | Assignee ID | `assignee_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User assigned to process this step | `UUID` |
| 493 | Step Status | `step_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Step status: PENDING, IN_PROGRESS, APPROVED, REJECTED, SKIPPED | `PENDING` |
| 494 | Due Date | `due_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Scheduled deadline for reviewing/approving this step | `2026-09-02T18:00:00` |
| 495 | Action Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the step was completed through approval, rejection, or another outcome | `2026-09-01T10:00:00` |
| 496 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-31T15:00:00` |
| 497 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Creator of the workflow step | `UUID` |
| 498 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-31T16:00:00` |
| 499 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | User who last modified the workflow step | `UUID` |
| 500 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-approval_action"></a>
### 34. Approval Action History (`approval_action`)

| Item | Definition |
|---|---|
| Description | Manages the history of actual actions at each step, such as review, approval, and rejection |
| Primary Key | `approval_action_id` |
| Key References (FK) | `workflow_step_id, actor_id, signature_id` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 501 | Approval Action History ID | `approval_action_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique review/approval action history identifier | `UUID` |
| 502 | Workflow Step ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | Y | - | N | Y | N | Y | Workflow step where the action occurred | `UUID` |
| 503 | Action Type | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Type of action performed: SUBMIT, REVIEW, APPROVE, REJECT, CANCEL | `APPROVE` |
| 504 | Actor ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who actually performed the review, approval, or rejection | `UUID` |
| 505 | Action Comments | `action_comment` | `text` | N | N | - | N | - | N | N | N | Y | Comments entered during review/approval | `No issues found during review` |
| 506 | Reason for Rejection | `rejection_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason entered when performing a REJECT action | `Additional evidence files required` |
| 507 | Electronic Signature ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature record linked to the approval/rejection action | `UUID` |
| 508 | Action Timestamp | `acted_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when submission, review, approval, or rejection was actually performed | `2026-09-01T10:00:00` |
| 509 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-09-01T10:00:00` |

[↑ Back to Top](#data-table-specifications)

---

## Traceability

<a id="table-traceability_link"></a>
### 35. Common Traceability Link (`traceability_link`)

| Item | Definition |
|---|---|
| Description | Manages traceability relationships among URS, FDS, DQ, FRA, IQ, OQ, and PQ items |
| Primary Key | `traceability_link_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 510 | Traceability Link ID | `traceability_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of a traceability relationship between deliverable items. Duplicate (project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type) combinations are not allowed in active data. | `UUID` |
| 511 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | ID of the project to which the traceability relationship belongs | `UUID` |
| 512 | Source Entity Type | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Source entity type: REQUIREMENT, FDS_ITEM, DDS_ITEM, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM | `REQUIREMENT` |
| 513 | Source Entity ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Primary key value of the table corresponding to source_entity_type. No physical FK is defined because this is a polymorphic reference. | `UUID` |
| 514 | Linked Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Linked entity type: REQUIREMENT, FDS_ITEM, DDS_ITEM, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM | `IQ_ITEM` |
| 515 | Linked Entity ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Primary key value of the table corresponding to target_entity_type. No physical FK is defined because this is a polymorphic reference. | `UUID` |
| 516 | Link Type | `link_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | IMPLEMENTED_BY, ASSESSED_BY, VERIFIED_BY, MITIGATED_BY | `VERIFIED_BY` |
| 517 | Link Rationale | `link_reason` | `text` | N | N | - | N | - | N | N | N | Y | Business rationale for linking the two deliverable items | `Verify the URS requirement through an IQ test` |
| 518 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Traceability link creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 519 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who created the traceability link | `UUID` |
| 520 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Traceability link last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 521 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who last modified the traceability link | `UUID` |
| 522 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Traceability link soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## DDS

<a id="table-dds_spec"></a>
### 41. Detailed Design Specification (`dds_spec`)

| Item | Definition |
|---|---|
| Description | Manages DDS document headers, versions, and approval statuses for each project |
| Primary Key | `dds_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 587 | DDS ID | `dds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique DDS document identifier | `UUID` |
| 588 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Validation project to which the DDS belongs | `UUID` |
| 589 | DDS Number | `dds_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | DDS document number | `DDS-VP-SYS-001` |
| 590 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the detailed design specification | `Detailed Design Specification` |
| 591 | Document Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | DDS document display version | `v1.0` |
| 592 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | DDS revision sequence number | `1` |
| 593 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for initial creation or revision of the DDS | `Initial creation` |
| 594 | Is Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether this is the currently valid latest DDS version | `True` |
| 595 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | DDS document status: DRAFT, REVIEW, APPROVED, REJECTED | `APPROVED` |
| 596 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 597 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who authored the DDS | `UUID` |
| 598 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 599 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the DDS | `UUID` |
| 600 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | DDS soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-dds_item"></a>
### 42. DDS Detailed Item (`dds_item`)

| Item | Definition |
|---|---|
| Description | Manages detailed database, interface, and component design items in DDS documents |
| Primary Key | `dds_item_id` |
| Key References (FK) | `dds_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 601 | DDS Item ID | `dds_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique DDS detailed item identifier | `UUID` |
| 602 | DDS ID | `dds_id` | `uuid` | N | Y | `dds_spec.dds_id` | Y | - | N | Y | N | Y | Parent DDS document identifier | `UUID` |
| 603 | DDS Item Number | `item_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | DDS detailed item management number | `DDS-001` |
| 604 | Design Type | `design_type` | `varchar(30)` | N | N | - | Y | `COMPONENT` | N | Y | N | Y | DATABASE, COMPONENT, INTERFACE, SECURITY, BATCH | `DATABASE` |
| 605 | Design Name | `design_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Detailed design item name | `User Authentication Table Design` |
| 606 | Design Description | `description` | `text` | N | N | - | Y | - | N | N | N | Y | DDS detailed design content | `User authentication and permission table structure` |
| 607 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | DDS item status: DRAFT, REVIEW, APPROVED | `APPROVED` |
| 608 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS item creation timestamp (UTC) | `09/01/2026 10:00:00` |
| 609 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who authored the DDS item | `UUID` |
| 610 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS item last modification timestamp (UTC) | `09/01/2026 10:00:00` |
| 611 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the DDS item | `UUID` |
| 612 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | DDS item soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## Deviation

<a id="table-deviation"></a>
### 43. Deviation Management (`deviation`)

| Item | Definition |
|---|---|
| Description | Manages deviations arising in documents or during test execution and their investigation, resolution, and approval statuses |
| Primary Key | `deviation_id` |
| Key References (FK) | `project_id, resolved_by, approved_by, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 613 | Deviation ID | `deviation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique deviation identifier | `UUID` |
| 614 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Validation project in which the deviation occurred | `UUID` |
| 615 | Source Entity Type | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Type of entity where the deviation occurred: IQ_ITEM, OQ_ITEM, PQ_ITEM, etc. | `OQ_ITEM` |
| 616 | Source Entity ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Primary key value of the target entity. No physical FK is defined because this is a polymorphic reference. | `UUID` |
| 617 | Deviation Number | `deviation_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Deviation management number within the project | `DEV-001` |
| 618 | Deviation Title | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Deviation Title | `Mismatch with Expected Result` |
| 619 | Deviation Description | `description` | `text` | N | N | - | Y | - | N | N | N | Y | Deviation details and circumstances of occurrence | `Mismatch between expected and actual results during OQ execution` |
| 620 | Severity | `severity` | `varchar(20)` | N | N | - | Y | `MINOR` | N | Y | N | Y | Deviation severity: MINOR, MAJOR, CRITICAL | `MAJOR` |
| 621 | Deviation Status | `deviation_status` | `varchar(20)` | N | N | - | Y | `OPEN` | N | Y | N | Y | OPEN, INVESTIGATING, RESOLVED, CLOSED, CANCELLED | `OPEN` |
| 622 | Resolution Details | `resolution` | `text` | N | N | - | N | - | N | N | N | Y | Investigation of the deviation's cause and resolution details | `Retesting completed after correcting configuration values` |
| 623 | Resolved At | `resolved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when deviation resolution was completed | `09/03/2026 15:00:00` |
| 624 | Resolved By ID | `resolved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who resolved the deviation | `UUID` |
| 625 | Approved At | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deviation closure approval timestamp | `09/03/2026 17:00:00` |
| 626 | Approver ID | `approved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who approved deviation closure | `UUID` |
| 627 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Deviation creation timestamp (UTC) | `09/01/2026 10:00:00` |
| 628 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the deviation | `UUID` |
| 629 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Deviation last modification timestamp (UTC) | `09/01/2026 10:00:00` |
| 630 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the deviation | `UUID` |
| 631 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deviation soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## Report

<a id="table-report_generation"></a>
### 44. Report Generation Job (`report_generation`)

| Item | Definition |
|---|---|
| Description | Manages generation requests, execution statuses, failures, retries, and output files for audit and operational reports |
| Primary Key | `report_generation_id` |
| Key References (FK) | `project_id, requested_by, result_file_id, report_schedule_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain report generation results and execution history for the required audit period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 632 | Report Generation ID | `report_generation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique report generation job identifier | `UUID` |
| 633 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Linked for project-level reports. NULL allowed for system-wide or organization-level reports. | `UUID` |
| 634 | Report Type | `report_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Report type: AUDIT_TRAIL, PROJECT_STATUS, WORKFLOW_STATUS, DEVIATION_STATUS, TRACEABILITY, SYSTEM_OPERATION | `AUDIT_TRAIL` |
| 635 | Report Name | `report_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Name of the generated report displayed to the user | `August 2026 Audit Trail Report` |
| 636 | Execution Mode | `execution_type` | `varchar(20)` | N | N | - | Y | `ON_DEMAND` | N | Y | N | Y | Execution mode: ON_DEMAND for user requests; SCHEDULED for scheduled batch execution | `ON_DEMAND` |
| 637 | Reporting Period Start | `period_from` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Start date/time for querying the report's source data. NULL allowed for reports without a reporting period. | `2026-08-01 00:00:00+00` |
| 638 | Reporting Period End | `period_to` | `timestamptz` | N | N | - | N | - | N | N | N | Y | End date/time for querying the report's source data. Cannot be earlier than period_from. | `2026-08-31 23:59:59+00` |
| 639 | Query Criteria | `report_parameters` | `jsonb` | N | N | - | N | - | N | N | N | Y | Stores report generation criteria, such as organization, project, user, action type, and status, as JSON | `{"action_types":["CREATE","UPDATE"]}` |
| 640 | Output Format | `output_format` | `varchar(20)` | N | N | - | Y | `PDF` | N | Y | N | Y | Output file format: PDF, XLSX, CSV | `PDF` |
| 641 | Generation Status | `generation_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Processing status: PENDING, PROCESSING, COMPLETED, RETRY_WAIT, FAILED, CANCELLED | `PENDING` |
| 642 | Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Requester ID for user requests. NULL allowed for generation by the system or scheduled batch jobs. | `UUID` |
| 643 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when the report generation request was received or the batch job was registered | `2026-09-02 15:00:00+00` |
| 644 | Execution Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the report generation worker actually started processing | `2026-09-02 15:00:05+00` |
| 645 | Execution Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the report generation job ended in success or final failure | `2026-09-02 15:01:30+00` |
| 646 | Result File ID | `result_file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | Y | Y | N | Y | ID of the generated report file. Required in COMPLETED status; NULL allowed before completion. | `UUID` |
| 647 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of retries after the initial execution failed. Must be 0 or greater. | `0` |
| 648 | Maximum Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Maximum number of automatic retries allowed. Must be 0 or greater. | `3` |
| 649 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next scheduled execution time for a job in RETRY_WAIT status | `2026-09-02 15:10:00+00` |
| 650 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | System error code categorizing the cause of failure | `REPORT_FILE_CREATE_FAILED` |
| 651 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Details of report generation failure. Sensitive information such as passwords and tokens is not stored. | `An error occurred while saving the result file` |
| 652 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Report generation job record creation timestamp (UTC) | `2026-09-02 15:00:00+00` |
| 653 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when created by a user. NULL allowed when created by the system or a batch job. | `UUID` |
| 654 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Report generation job last modification timestamp (UTC) | `2026-09-02 15:01:30+00` |
| 655 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when modified by a user. NULL allowed for system/batch processing. | `UUID` |
| 656 | Report Schedule ID | `report_schedule_id` | `uuid` | N | Y | `report_schedule.report_schedule_id` | N | - | N | Y | N | Y | Stores the original report schedule ID when generated by a scheduled execution. NULL allowed for user-requested executions. | `UUID` |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-report_schedule"></a>
### 50. Report Schedule (`report_schedule`)

| Item | Definition |
|---|---|
| Description | Manages execution frequencies, reporting periods, output formats, next execution times, and active statuses for audit and operational reports |
| Primary Key | `report_schedule_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain execution schedules and change history for the required audit period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 744 | Report Schedule ID | `report_schedule_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique scheduled report execution schedule identifier | `UUID` |
| 745 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Linked for project-level report schedules. NULL allowed for organization-level or system-level schedules. | `UUID` |
| 746 | Schedule Name | `schedule_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Name of the scheduled report schedule displayed to the user | `Monthly Audit Trail Report` |
| 747 | Report Type | `report_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Report type: AUDIT_TRAIL, PROJECT_STATUS, WORKFLOW_STATUS, DEVIATION_STATUS, TRACEABILITY, SYSTEM_OPERATION | `AUDIT_TRAIL` |
| 748 | Schedule Type | `schedule_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Schedule type: DAILY, WEEKLY, MONTHLY, CRON | `MONTHLY` |
| 749 | Schedule Expression | `schedule_expression` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Scheduling configuration, such as an execution date, day of the week, time, or Cron expression | `0 0 1 * *` |
| 750 | Reporting Period Type | `period_type` | `varchar(30)` | N | N | - | Y | `PREVIOUS_MONTH` | N | N | N | Y | Basis for calculating the source data reporting period: PREVIOUS_DAY, PREVIOUS_WEEK, PREVIOUS_MONTH, CUSTOM | `PREVIOUS_MONTH` |
| 751 | Query Criteria | `report_parameters` | `jsonb` | N | N | - | N | - | N | N | N | Y | Query criteria for scheduled reports, such as organization, project, user, action type, and status | `{"action_types":["CREATE","UPDATE"]}` |
| 752 | Output Format | `output_format` | `varchar(20)` | N | N | - | Y | `PDF` | N | Y | N | Y | Output file format: PDF, XLSX, CSV | `PDF` |
| 753 | Next Run At | `next_run_at` | `timestamptz` | N | N | - | Y | - | N | Y | N | Y | Next scheduled execution time for this schedule | `2026-10-01 00:00:00+00` |
| 754 | Last Run At | `last_run_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when this schedule was last executed | `2026-09-01 00:00:00+00` |
| 755 | Last Report Generation Job ID | `last_report_generation_id` | `uuid` | N | Y | `report_generation.report_generation_id` | N | - | N | Y | N | Y | Most recent report generation job created by this schedule | `UUID` |
| 756 | Is Active | `is_active` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | Whether the scheduled report schedule is active | `True` |
| 757 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Report schedule creation timestamp (UTC) | `2026-09-02 18:00:00+00` |
| 758 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the report schedule | `UUID` |
| 759 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Report schedule last modification timestamp (UTC) | `2026-09-02 18:00:00+00` |
| 760 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the report schedule | `UUID` |
| 761 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Report schedule soft deletion timestamp | - |

[↑ Back to Top](#data-table-specifications)

---

## AI

<a id="table-ai_generation_job"></a>
### 45. AI Generation Job (`ai_generation_job`)

| Item | Definition |
|---|---|
| Description | Manages AI generation requests, models, input parameters, execution statuses, failures, and retries |
| Primary Key | `ai_job_id` |
| Key References (FK) | `project_id, requested_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 657 | AI Job ID | `ai_job_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI generation job identifier | `UUID` |
| 658 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Project link | `UUID` |
| 659 | Action Type | `job_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | ITEM_GENERATION, DOCUMENT_GENERATION | `ITEM_GENERATION` |
| 660 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM, OQ_REPORT, PQ_REPORT, VSR_REPORT | `REQUIREMENT` |
| 661 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Identifier of the generation target | `UUID` |
| 662 | AI Model Name | `model_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | GPT, Claude, Gemini | `GPT-5` |
| 663 | Input Parameters | `input_parameters` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Stores generation criteria as JSON | `JSON` |
| 664 | Generation Status | `generation_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | PENDING, PROCESSING, COMPLETED, FAILED, RETRY_WAIT, CANCELLED | `COMPLETED` |
| 665 | Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User requesting generation | `UUID` |
| 666 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when the request was received | `2026-09-02T00:00:00` |
| 667 | Execution Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI processing start time | `2026-09-02T00:00:00` |
| 668 | Execution Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI processing completion time | `2026-09-02T00:00:00` |
| 669 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of re-executions | `0` |
| 670 | Maximum Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Retry limit | `3` |
| 671 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Scheduled retry time | `2026-09-02T15:10:00` |
| 672 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Failure cause code | `LLM_TIMEOUT` |
| 673 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Detailed failure message | `Model response timed out` |
| 674 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation timestamp | `2026-09-02T00:00:00` |
| 675 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who created the record | `UUID` |
| 676 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Modification timestamp | `2026-09-02T00:00:00` |
| 677 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who modified the record | `UUID` |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-ai_generation_result"></a>
### 46. AI Generation Result (`ai_generation_result`)

| Item | Definition |
|---|---|
| Description | Manages AI-generated result sets and adoption statuses |
| Primary Key | `ai_result_id` |
| Key References (FK) | `ai_job_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 678 | AI Result ID | `ai_result_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI result identifier | `UUID` |
| 679 | AI Job ID | `ai_job_id` | `uuid` | N | Y | `ai_generation_job.ai_job_id` | Y | - | N | Y | N | Y | Parent AI job | `UUID` |
| 680 | Result Title | `result_title` | `varchar(300)` | N | N | - | N | - | N | N | N | Y | Result Title | `OQ Test Draft` |
| 681 | Is Selected | `is_selected` | `boolean` | N | N | - | Y | `N` | N | Y | N | Y | Whether adopted by the user | `True` |
| 682 | Is Applied | `is_applied` | `boolean` | N | N | - | Y | `N` | N | Y | N | Y | Whether applied to the actual deliverable | `True` |
| 683 | Selected By ID | `selected_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who adopted the result | `UUID` |
| 684 | Selected At | `selected_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the result was adopted | `2026-09-02T00:00:00` |
| 685 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation timestamp | `2026-09-02T00:00:00` |
| 686 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who created the record | `UUID` |
| 687 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Modification timestamp | `2026-09-02T00:00:00` |
| 688 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who modified the record | `UUID` |

[↑ Back to Top](#data-table-specifications)

---

<a id="table-ai_result_item"></a>
### 47. AI Generation Result Item (`ai_result_item`)

| Item | Definition |
|---|---|
| Description | Manages details of AI-generated URS/FRA/IQ/OQ/PQ items and document sections |
| Primary Key | `ai_result_item_id` |
| Key References (FK) | `ai_result_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 689 | AI Result Item ID | `ai_result_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Result item identifier | `UUID` |
| 690 | AI Result ID | `ai_result_id` | `uuid` | N | Y | `ai_generation_result.ai_result_id` | Y | - | N | Y | N | Y | Reference to the parent result | `UUID` |
| 691 | Item Order | `item_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Result display order | `1` |
| 692 | Item Type | `item_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT, FRA_SCENARIO, IQ_TEST, OQ_TEST, PQ_TEST, DOCUMENT_SECTION | `REQUIREMENT` |
| 693 | Title | `title` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | Title of the generated item | `Electronic Signature Record` |
| 694 | Body Content | `content` | `text` | N | N | - | Y | - | N | N | N | Y | Body of the generated result | `Content` |
| 695 | Application Target Type | `target_entity_type` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | REQUIREMENT, FRA_ITEM, IQ_ITEM, etc. | `REQUIREMENT` |
| 696 | Application Target ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Primary key of the actual storage target | `UUID` |
| 697 | Is Adopted | `is_selected` | `boolean` | N | N | - | Y | `N` | N | Y | N | Y | Whether adopted by the user | `True` |
| 698 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation timestamp | `2026-09-02T00:00:00` |
| 699 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who created the record | `UUID` |
| 700 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Modification timestamp | `2026-09-02T00:00:00` |
| 701 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who modified the record | `UUID` |

[↑ Back to Top](#data-table-specifications)

---

## Notification

<a id="table-notification_delivery"></a>
### 48. Notification Delivery (`notification_delivery`)

| Item | Definition |
|---|---|
| Description | Manages recipients, delivery statuses, failures, and retry history for review/approval requests, processing delay notices, and system operation notifications |
| Primary Key | `notification_delivery_id` |
| Key References (FK) | `project_id, workflow_instance_id, workflow_step_id, recipient_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain notification content and execution history for the required audit period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 702 | Notification Delivery ID | `notification_delivery_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique notification delivery job identifier | `UUID` |
| 703 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Linked for project-related notifications. NULL allowed for system-wide notifications. | `UUID` |
| 704 | Workflow Instance ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Linked for notifications related to a review/approval workflow | `UUID` |
| 705 | Workflow Step ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | N | - | N | Y | N | Y | Linked for notifications related to a review/approval step | `UUID` |
| 706 | Notification Type | `notification_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Business type of the notification: APPROVAL_REQUEST, REVIEW_REQUEST, DUE_REMINDER, OVERDUE, REJECTION, COMPLETION, SYSTEM | `OVERDUE` |
| 707 | Delivery Channel | `delivery_channel` | `varchar(20)` | N | N | - | Y | `EMAIL` | N | Y | N | Y | Delivery channel: EMAIL, MESSENGER, PUSH, IN_APP | `EMAIL` |
| 708 | Recipient ID | `recipient_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | Y | Y | ID of the user receiving the notification | `UUID` |
| 709 | Notification Title | `notification_title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Title of the notification sent to the user | `Review Deadline Overdue Notice` |
| 710 | Notification Content | `notification_content` | `text` | N | N | - | Y | - | N | N | N | Y | Body of the notification sent to the user. Sensitive information such as passwords and tokens must not be stored. | `The FDS review deadline has passed.` |
| 711 | Delivery Status | `delivery_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Delivery status: PENDING, PROCESSING, SENT, RETRY_WAIT, FAILED, CANCELLED | `PENDING` |
| 712 | Scheduled Delivery At | `scheduled_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Scheduled notification delivery time. NULL allowed for immediate delivery. | `2026-09-02 18:00:00+00` |
| 713 | Sent At | `sent_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the notification was successfully handed off to the external delivery channel | `2026-09-02 18:00:05+00` |
| 714 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of retries after the initial delivery failed. Must be 0 or greater. | `0` |
| 715 | Maximum Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Maximum number of automatic delivery retries allowed. Must be 0 or greater. | `3` |
| 716 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next scheduled delivery time for a notification in RETRY_WAIT status | `2026-09-02 18:10:00+00` |
| 717 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | System error code categorizing the cause of notification delivery failure | `EMAIL_SEND_TIMEOUT` |
| 718 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Details of notification delivery failure. Sensitive information such as authentication credentials must not be stored. | `Mail server response timed out` |
| 719 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Notification delivery job creation timestamp (UTC) | `2026-09-02 18:00:00+00` |
| 720 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when created by a user. NULL allowed when created by the system or a batch job. | `UUID` |
| 721 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Notification delivery job last modification timestamp (UTC) | `2026-09-02 18:00:05+00` |
| 722 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when modified by a user. NULL allowed for system/batch processing. | `UUID` |

[↑ Back to Top](#data-table-specifications)

---
