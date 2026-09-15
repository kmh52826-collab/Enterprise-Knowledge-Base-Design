<a id="top"></a>

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

- Total tables: **54**
- Total columns: **821**
- Domains: **25**
- Naming conventions: Tables and columns use `snake_case`; primary keys use entity-specific identifier columns.
- Date/time conventions: Store UTC in the database; display dates and times in the user's or site's time zone.
- GxP principles: Manage approved records through revisions and status history without directly modifying them, and record significant changes in the Audit Trail.

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
| 8 | Compliance | Electronic Signature | [`electronic_signature`](#table-electronic_signature) | `signature_id` | `signer_id` | Critical | Y |
| 9 | Compliance | Audit Trail | [`audit_trail`](#table-audit_trail) | `audit_id` | `actor_id` | Critical | Y |
| 10 | File | File Asset | [`file_asset`](#table-file_asset) | `file_id` | `uploader_id, cleanup_execution_id` | High | Y |
| 11 | File | Evidence File Link | [`evidence_link`](#table-evidence_link) | `evidence_link_id` | `project_id, file_id, created_by, updated_by` | High | Y |
| 12 | File | File Cleanup Execution History | [`file_cleanup_execution`](#table-file_cleanup_execution) | `file_cleanup_execution_id` | `requested_by, created_by, updated_by` | High | Y |
| 13 | System | System/Equipment Identification Information | [`system_asset`](#table-system_asset) | `system_id` | `organization_id` | High | Y |
| 14 | System | Backup Execution History | [`backup_execution`](#table-backup_execution) | `backup_execution_id` | `requested_by, created_by, updated_by` | Critical | Y |
| 15 | Library | Library Item Master | [`library_item`](#table-library_item) | `library_id` | - | High | Y |
| 16 | Validation | Validation Project | [`validation_project`](#table-validation_project) | `project_id` | `system_id, created_by, updated_by, closure_requested_by` | High | Y |
| 17 | Validation | Project Member | [`project_member`](#table-project_member) | `project_member_id` | `project_id, user_id, role_id, created_by, updated_by` | High | Y |
| 18 | Validation | Validation Activity Master | [`validation_activity`](#table-validation_activity) | `activity_id` | `created_by, updated_by` | High | Y |
| 19 | Validation | Project Activity | [`project_activity`](#table-project_activity) | `project_activity_id` | `project_id, activity_id, created_by, updated_by` | Critical | Y |
| 20 | Validation | Activity Dependency | [`activity_dependency`](#table-activity_dependency) | `activity_dependency_id` | `successor_activity_id, predecessor_activity_id, created_by, updated_by` | Critical | Y |
| 21 | QIA | Quality Impact Assessment Header | [`qia_assessment`](#table-qia_assessment) | `qia_id` | `project_id` | High | Y |
| 22 | QIA | QIA Module Detailed Assessment | [`qia_module_item`](#table-qia_module_item) | `qia_module_item_id` | `qia_id` | High | Y |
| 23 | VA | Vendor Audit Assessment | [`vendor_audit`](#table-vendor_audit) | `audit_id` | `project_id` | High | Y |
| 24 | URS | User Requirements Specification | [`requirement`](#table-requirement) | `requirement_id` | `project_id, created_by` | High | Y |
| 25 | FDS | Functional Design Specification | [`fds_spec`](#table-fds_spec) | `fds_id` | `project_id, created_by, updated_by` | High | Y |
| 26 | FDS | FDS Detailed Item | [`fds_item`](#table-fds_item) | `fds_item_id` | `fds_id, created_by, updated_by` | High | Y |
| 27 | FDS | FDS Interface Definition | [`fds_interface`](#table-fds_interface) | `fds_interface_id` | `fds_id, created_by, updated_by` | High | Y |
| 28 | DQ | Design Qualification Assessment | [`dq_assessment`](#table-dq_assessment) | `dq_id` | `project_id, created_by, updated_by` | High | Y |
| 29 | DQ | DQ Detailed Assessment Item | [`dq_item`](#table-dq_item) | `dq_item_id` | `dq_id, requirement_id, reviewed_by, created_by, updated_by` | High | Y |
| 30 | FRA | Functional Risk Assessment | [`fra_assessment`](#table-fra_assessment) | `fra_id` | `project_id, created_by, updated_by` | High | Y |
| 31 | FRA | FRA Detailed Risk Item | [`fra_item`](#table-fra_item) | `fra_item_id` | `fra_id, requirement_id, created_by, updated_by` | High | Y |
| 32 | IQ | Installation Qualification Assessment | [`iq_assessment`](#table-iq_assessment) | `iq_id` | `project_id, created_by, updated_by` | High | Y |
| 33 | IQ | IQ Detailed Test Item | [`iq_item`](#table-iq_item) | `iq_item_id` | `iq_id, executed_by, created_by, updated_by` | High | Y |
| 34 | OQ | Operational Qualification Assessment | [`oq_assessment`](#table-oq_assessment) | `oq_id` | `project_id, created_by, updated_by` | High | Y |
| 35 | OQ | OQ Detailed Test Item | [`oq_item`](#table-oq_item) | `oq_item_id` | `oq_id, executed_by, created_by, updated_by` | High | Y |
| 36 | PQ | Performance Qualification Assessment | [`pq_assessment`](#table-pq_assessment) | `pq_id` | `project_id, created_by, updated_by` | High | Y |
| 37 | PQ | PQ Detailed Test Item | [`pq_item`](#table-pq_item) | `pq_item_id` | `pq_id, executed_by, created_by, updated_by` | High | Y |
| 38 | RTM | Requirements Traceability Matrix | [`rtm_assessment`](#table-rtm_assessment) | `rtm_id` | `project_id, created_by, updated_by` | Critical | Y |
| 39 | RTM | RTM Detailed Traceability Item | [`rtm_item`](#table-rtm_item) | `rtm_item_id` | `rtm_id, requirement_id, created_by, updated_by` | Critical | Y |
| 40 | VSR | Validation Summary Report | [`vsr_assessment`](#table-vsr_assessment) | `vsr_id` | `project_id, created_by, updated_by` | Critical | Y |
| 41 | VSR | VSR Activity Summary Item | [`vsr_item`](#table-vsr_item) | `vsr_item_id` | `vsr_id, created_by, updated_by` | Critical | Y |
| 42 | Workflow | Workflow Instance | [`workflow_instance`](#table-workflow_instance) | `workflow_instance_id` | `requested_by, created_by, updated_by` | Critical | Y |
| 43 | Workflow | Workflow Step | [`workflow_step`](#table-workflow_step) | `workflow_step_id` | `workflow_instance_id, assignee_id, created_by, updated_by` | Critical | Y |
| 44 | Workflow | Approval Action History | [`approval_action`](#table-approval_action) | `approval_action_id` | `workflow_step_id, actor_id, signature_id` | Critical | Y |
| 45 | Traceability | Common Traceability Link | [`traceability_link`](#table-traceability_link) | `traceability_link_id` | `project_id, created_by, updated_by` | Critical | Y |
| 46 | DDS | Detailed Design Specification | [`dds_spec`](#table-dds_spec) | `dds_id` | `project_id, created_by, updated_by` | High | Y |
| 47 | DDS | DDS Detailed Item | [`dds_item`](#table-dds_item) | `dds_item_id` | `dds_id, created_by, updated_by` | High | Y |
| 48 | Deviation | Deviation Management | [`deviation`](#table-deviation) | `deviation_id` | `project_id, resolved_by, approved_by, created_by, updated_by` | Critical | Y |
| 49 | Report | Report Generation Job | [`report_generation`](#table-report_generation) | `report_generation_id` | `project_id, requested_by, result_file_id, created_by, updated_by, report_schedule_id` | High | Y |
| 50 | Report | Report Schedule | [`report_schedule`](#table-report_schedule) | `report_schedule_id` | `project_id, last_report_generation_id, created_by, updated_by` | High | Y |
| 51 | AI | AI Generation Job | [`ai_generation_job`](#table-ai_generation_job) | `ai_job_id` | `project_id, requested_by, created_by, updated_by` | High | Y |
| 52 | AI | AI Generation Result | [`ai_generation_result`](#table-ai_generation_result) | `ai_result_id` | `ai_job_id, selected_by, created_by, updated_by` | High | Y |
| 53 | AI | AI Generation Result Item | [`ai_result_item`](#table-ai_result_item) | `ai_result_item_id` | `ai_result_id, created_by, updated_by` | High | Y |
| 54 | Notification | Notification Delivery | [`notification_delivery`](#table-notification_delivery) | `notification_delivery_id` | `project_id, workflow_instance_id, workflow_step_id, recipient_id, created_by, updated_by` | High | Y |

## 4. Domain Navigation

- **Organization**: [`organization`](#table-organization)
- **Security**: [`app_user`](#table-app_user), [`role`](#table-role), [`user_role`](#table-user_role), [`user_group`](#table-user_group), [`user_group_member`](#table-user_group_member), [`group_role`](#table-group_role)
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
| --- | --- |
| Description | Basic information about client companies or operating organizations |
| Primary Key | `organization_id` |
| Key References (FK) | - |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 14 | Organization ID | `organization_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique organization identifier | `00000000-0000-0000-0000-000000000001` |
| 15 | Organization Code | `organization_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Organization/company identification code | `ORG-SAMPLE-01` |
| 16 | Organization Name | `organization_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Company/site name | `Sample Corporation` |
| 17 | Organization Type | `organization_type` | `varchar(50)` | N | N | - | Y | `Headquarters'` | N | N | N | Y | Headquarters \| Factory \| Research Institute \| Overseas Subsidiary | `Headquarters` |
| 18 | Status | `status` | `varchar(20)` | N | N | - | Y | `ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE | `ACTIVE` |
| 19 | Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Organization description | `Corporate IT and data operations organization` |
| 20 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-01T00:00:00` |
| 21 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T16:00:00` |

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
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
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

[↑ Back to Top](#top)

---

<a id="table-role"></a>
### 3. Role (`role`)

| Item | Definition |
| --- | --- |
| Description | Master definitions of permission roles, such as author, reviewer, and approver |
| Primary Key | `role_id` |
| Key References (FK) | - |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 22 | Role ID | `role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique role/permission identifier | `00000000-0000-0000-0000-000000000001` |
| 23 | Role Code | `role_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Role identification code | `SYSTEM_ADMIN` |
| 24 | Role Name | `role_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Role name (system administrator, author, approver, etc.) | `Super Administrator` |
| 25 | Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Detailed description of the role's permission scope | `System-wide administrative permissions` |
| 26 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-01T00:00:00` |

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
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 27 | Mapping ID | `user_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique user-to-role mapping identifier. Duplicate (user_id, role_id) combinations are not allowed in active data. | `00000000-0000-0000-0000-000000000001` |
| 28 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 29 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | References role.role_id | `00000000-0000-0000-0000-000000000001` |
| 30 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-01T00:00:00` |

[↑ Back to Top](#top)

---

<a id="table-user_group"></a>
### 5. User Group (`user_group`)

| Item | Definition |
| --- | --- |
| Description | Manages basic information and active status for user groups within each organization |
| Primary Key | `user_group_id` |
| Key References (FK) | `organization_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 789 | User Group ID | `user_group_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique user group identifier | `UUID` |
| 790 | Organization ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | Organization to which the user group belongs | `UUID` |
| 791 | Group Code | `group_code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | User group identification code within the organization. Active records must have a unique (organization_id, group_code) combination | `QA_REVIEWER_GROUP` |
| 792 | Group Name | `group_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Group name displayed to users | `QA Reviewer Group` |
| 793 | Group Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Description of the group’s purpose and permission scope | `QA reviewers responsible for Validation documents` |
| 794 | Is Active | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the user group is active | `True` |
| 795 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | User group creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 796 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who created the user group | `UUID` |
| 797 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | User group last-modified timestamp (UTC) | `2026-09-01T10:00:00` |
| 798 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the user group | `UUID` |
| 799 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | User group soft-deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-user_group_member"></a>
### 6. User Group Member (`user_group_member`)

| Item | Definition |
| --- | --- |
| Description | Manages the N:M relationship between users and user groups, including membership status and period |
| Primary Key | `user_group_member_id` |
| Key References (FK) | `user_group_id, user_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 800 | Group Member ID | `user_group_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the user group membership mapping | `UUID` |
| 801 | User Group ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | Y | - | N | Y | N | Y | User group to which the member belongs | `UUID` |
| 802 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User included in the group. Active records must have a unique (user_group_id, user_id) combination | `UUID` |
| 803 | Member Status | `member_status` | `varchar(20)` | N | N | - | Y | `ACTIVE` | N | Y | N | Y | Group member status: ACTIVE, INACTIVE, WITHDRAWN | `ACTIVE` |
| 804 | Participation Started At | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp when the user joined the group | `2026-09-01T10:00:00` |
| 805 | Participation Ended At | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Timestamp when the user’s group membership ended. NULL for a current member | - |
| 806 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Group membership mapping creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 807 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the group member | `UUID` |
| 808 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Group membership mapping last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 809 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the group membership mapping | `UUID` |
| 810 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Group membership mapping soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-group_role"></a>
### 7. User Group Role (`group_role`)

| Item | Definition |
| --- | --- |
| Description | Assigns roles to user groups and manages whether they apply globally or to a specific project |
| Primary Key | `group_role_id` |
| Key References (FK) | `user_group_id, role_id, project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 811 | Group Role ID | `group_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for the permission mapping between a user group and a role | `UUID` |
| 812 | User Group ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | Y | - | N | Y | N | Y | User group assigned the role | `UUID` |
| 813 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | Role assigned to the group | `UUID` |
| 814 | Permission Scope Type | `scope_type` | `varchar(20)` | N | N | - | Y | `GLOBAL` | N | Y | N | Y | Scope of the group role: GLOBAL, PROJECT | `PROJECT` |
| 815 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Required when scope_type is PROJECT. NULL when scope_type is GLOBAL | `UUID` |
| 816 | Is Active | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the group role mapping is active. Active records must have a unique group, role, scope, and project combination | `TRUE` |
| 817 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Group role mapping creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 818 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who assigned the group role | `UUID` |
| 819 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Group role mapping last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 820 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the group role mapping | `UUID` |
| 821 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Group role mapping soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## Compliance

<a id="table-electronic_signature"></a>
### 8. Electronic Signature (`electronic_signature`)

| Item | Definition |
| --- | --- |
| Description | Records electronic signature evidence for document review/approval |
| Primary Key | `signature_id` |
| Key References (FK) | `signer_id` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 31 | Electronic Signature ID | `signature_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique electronic signature record identifier | `00000000-0000-0000-0000-000000000001` |
| 32 | Signer ID | `signer_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | References app_user.user_id | `00000000-0000-0000-0000-000000000001` |
| 33 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Name of the table to which the signature applies (e.g., requirement) | `requirement` |
| 34 | Target Record ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Primary key value of the signed record | `00000000-0000-0000-0000-000000000001` |
| 35 | Signature Step | `signature_action` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | SUBMIT \| REVIEW \| APPROVE \| REJECT. The author’s submission signature is recorded as SUBMIT | `APPROVE` |
| 36 | Signature Purpose | `signature_meaning` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Reason for signing under 21 CFR Part 11 | `Final approval of URS requirements` |
| 37 | Signature Timestamp | `signed_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Timestamp when the electronic signature was executed | `2026-08-26T16:30:00` |
| 38 | Target Document Version | `target_version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the target document revision to which the electronic signature applies | `v1.0` |
| 39 | Signed Content Hash | `content_hash` | `varchar(128)` | N | N | - | Y | - | N | N | N | Y | SHA-256 hash for verifying the integrity of the target document and its details at the time of signing | `[SAMPLE_SHA256_HASH]` |
| 40 | Reauthentication Method | `authentication_method` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Reauthentication method used to verify the signer's identity when executing the electronic signature | `PASSWORD` |
| 41 | Reauthentication Result | `authentication_result` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Result of reauthentication when executing the electronic signature | `SUCCESS` |

[↑ Back to Top](#top)

---

<a id="table-audit_trail"></a>
### 9. Audit Trail (`audit_trail`)

| Item | Definition |
| --- | --- |
| Description | Audit trail records of before and after values for significant data changes and the actors responsible |
| Primary Key | `audit_id` |
| Key References (FK) | `actor_id` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 42 | Audit Trail ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique audit trail record identifier | `00000000-0000-0000-0000-000000000001` |
| 43 | Actor ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | References app_user.user_id (NULL allowed for automated system operations) | `00000000-0000-0000-0000-000000000001` |
| 44 | Action Type | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | CREATE, UPDATE, DELETE, EXPORT, LOGIN, LOGOUT, REPORT_GENERATE, REPORT_DOWNLOAD, REPORT_CANCEL, PROJECT_CLOSE, PROJECT_FORCE_CLOSE | `REPORT_GENERATE` |
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

[↑ Back to Top](#top)

---

## File

<a id="table-file_asset"></a>
### 10. File Asset (`file_asset`)

| Item | Definition |
| --- | --- |
| Description | Manages storage metadata for user attachments, evidence files, generated reports, and data export files |
| Primary Key | `file_id` |
| Key References (FK) | `uploader_id, cleanup_execution_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the required audit period according to the retention policies for the file type and linked entity |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 59 | File ID | `file_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique attachment metadata identifier | `00000000-0000-0000-0000-000000000001` |
| 60 | Uploader ID | `uploader_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Uploader ID is required for files uploaded by users. NULL allowed for files generated by the system or scheduled batch jobs. | `UUID` |
| 61 | Original File Name | `original_file_name` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | File name at the time of upload | `sample_document.pdf` |
| 62 | Storage File Path | `stored_file_path` | `text` | N | N | - | Y | - | N | N | N | Y | Storage path/S3 key | `documents/2026/08/sample_001.pdf` |
| 63 | File Category | `file_category` | `varchar(30)` | N | N | - | Y | `ATTACHMENT` | N | Y | N | Y | Business category of the file: ATTACHMENT, EVIDENCE, REPORT, EXPORT | `REPORT` |
| 64 | File Size | `file_size_bytes` | `bigint` | N | N | - | Y | `0` | N | N | N | Y | File size (bytes) | `1048576` |
| 65 | MIME Type | `mime_type` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | File format | `application/pdf` |
| 66 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T16:30:00` |
| 67 | Is Temporary File | `is_temporary` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | Whether the file is a temporary file created during upload or generation | `True` |
| 68 | Expires At | `expires_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Time when the file becomes eligible for cleanup. NULL for permanent files or files subject to business retention requirements. | `2026-09-09 18:00:00+00` |
| 69 | Cleanup Status | `cleanup_status` | `varchar(20)` | N | N | - | Y | `ACTIVE` | N | Y | N | Y | File cleanup status: ACTIVE, CLEANUP_PENDING, CLEANED, CLEANUP_FAILED | `ACTIVE` |
| 70 | Last Cleanup Execution ID | `cleanup_execution_id` | `uuid` | N | Y | `file_cleanup_execution.file_cleanup_execution_id` | N | - | N | Y | N | Y | ID of the file cleanup job that last processed this file | `UUID` |
| 71 | Cleaned Up At | `cleaned_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when cleanup of the actual stored file and its metadata was completed | `2026-09-09 19:00:00+00` |
| 72 | Cleanup Failure Reason | `cleanup_error_message` | `text` | N | N | - | N | - | N | N | N | Y | Detailed reason for failure to clean up an individual file. Sensitive information such as access keys must not be stored. | `No permission to access the file` |

[↑ Back to Top](#top)

---

<a id="table-evidence_link"></a>
### 11. Evidence File Link (`evidence_link`)

| Item | Definition |
| --- | --- |
| Description | Manages N:M links between documents/test items and evidence files |
| Primary Key | `evidence_link_id` |
| Key References (FK) | `project_id, file_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 528 | Evidence Link ID | `evidence_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for a link between a document/test item and an evidence file. Duplicate (project_id, file_id, target_entity_type, target_entity_id, evidence_type) combinations are not allowed in active data. | `UUID` |
| 529 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | ID of the project to which the evidence link belongs | `UUID` |
| 530 | File ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | Y | - | N | Y | N | Y | ID of the linked evidence file | `UUID` |
| 531 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Type of entity linked to the evidence file: REQUIREMENT, FDS_SPEC, FDS_ITEM, FDS_INTERFACE, DDS_SPEC, DDS_ITEM, QIA_ASSESSMENT, QIA_MODULE_ITEM, VENDOR_AUDIT, DQ_ASSESSMENT, DQ_ITEM, FRA_ASSESSMENT, FRA_ITEM, IQ_ASSESSMENT, IQ_ITEM, OQ_ASSESSMENT, OQ_ITEM, PQ_ASSESSMENT, PQ_ITEM, RTM_ASSESSMENT, RTM_ITEM, VSR_ASSESSMENT, VSR_ITEM, DEVIATION | `IQ_ITEM` |
| 532 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Primary key value of the table corresponding to target_entity_type. No physical FK is defined because this is a polymorphic reference. | `UUID` |
| 533 | Evidence Type | `evidence_type` | `varchar(50)` | N | N | - | Y | `TEST_RESULT` | N | Y | N | Y | TEST_RESULT, SCREENSHOT, LOG, REPORT, APPROVAL_DOCUMENT | `TEST_RESULT` |
| 534 | Evidence Description | `description` | `text` | N | N | - | N | - | N | N | N | Y | Content of the evidence file and purpose of the link | `Screenshot of IQ execution results` |
| 535 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Evidence link creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 536 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who created the evidence link | `UUID` |
| 537 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Evidence link last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 538 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who last modified the evidence link | `UUID` |
| 539 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Evidence link soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-file_cleanup_execution"></a>
### 12. File Cleanup Execution History (`file_cleanup_execution`)

| Item | Definition |
| --- | --- |
| Description | Manages execution criteria, processed item counts, execution statuses, failures, and retry history for temporary and expired file cleanup jobs |
| Primary Key | `file_cleanup_execution_id` |
| Key References (FK) | `requested_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain file cleanup results and execution history for the required audit period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 767 | File Cleanup Execution ID | `file_cleanup_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier for a temporary/expired file cleanup job | `UUID` |
| 768 | Cleanup Type | `cleanup_type` | `varchar(30)` | N | N | - | Y | - | N | Y | N | Y | Cleanup type: TEMPORARY_FILE, EXPIRED_FILE, ORPHAN_FILE | `EXPIRED_FILE` |
| 769 | Execution Mode | `execution_type` | `varchar(20)` | N | N | - | Y | `SCHEDULED` | N | Y | N | Y | Execution mode: SCHEDULED, ON_DEMAND | `SCHEDULED` |
| 770 | Target Cutoff Time | `target_base_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Cutoff time used to find files that expired or became eligible for cleanup before this time | `2026-09-02 00:00:00+00` |
| 771 | Execution Status | `execution_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Execution status: PENDING, PROCESSING, COMPLETED, RETRY_WAIT, FAILED, CANCELLED | `COMPLETED` |
| 772 | Scanned File Count | `scanned_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of files scanned to determine cleanup eligibility. Must be 0 or greater. | `100` |
| 773 | Cleanup Target Count | `target_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of files identified as eligible for cleanup. Must be 0 or greater. | `20` |
| 774 | Cleaned File Count | `cleaned_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of files for which the actual file and metadata cleanup was completed. Must be 0 or greater. | `19` |
| 775 | Cleanup Failure Count | `failed_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of files that failed cleanup. Must be 0 or greater. | `1` |
| 776 | Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who requested manual cleanup. NULL allowed for scheduled batch executions. | `UUID` |
| 777 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when a manual request was received or a scheduled job was registered | `2026-09-02 01:00:00+00` |
| 778 | Execution Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the file cleanup job actually started | `2026-09-02 01:00:05+00` |
| 779 | Execution Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the cleanup job ended in success or final failure | `2026-09-02 01:05:00+00` |
| 780 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of retries after the initial execution failed. Must be 0 or greater. | `0` |
| 781 | Maximum Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Maximum number of automatic retries allowed. Must be 0 or greater. | `3` |
| 782 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next scheduled execution time for a job in RETRY_WAIT status | `2026-09-02 01:15:00+00` |
| 783 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | System error code categorizing the cause of file cleanup failure | `FILE_DELETE_FAILED` |
| 784 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Details of file cleanup failure. Sensitive information such as access keys must not be stored. | `Failed to delete the stored file` |
| 785 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | File cleanup execution history creation timestamp (UTC) | `2026-09-02 01:00:00+00` |
| 786 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when created by a user. NULL allowed when created by the system or a batch job. | `UUID` |
| 787 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | File cleanup execution history last modification timestamp (UTC) | `2026-09-02 01:05:00+00` |
| 788 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when modified by a user. NULL allowed for system/batch processing. | `UUID` |

[↑ Back to Top](#top)

---

## System

<a id="table-system_asset"></a>
### 13. System/Equipment Identification Information (`system_asset`)

| Item | Definition |
| --- | --- |
| Description | Reference information for systems/equipment subject to validation (asset management number, GAMP category) |
| Primary Key | `system_id` |
| Key References (FK) | `organization_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
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
| 84 | Includes CS | `is_cs_included` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether a Computerized System is included | `True` |
| 85 | Version | `version` | `varchar(50)` | N | N | - | N | - | N | N | N | N | Software or equipment version | `v3.2.1` |
| 86 | GAMP Category | `gamp_category` | `varchar(50)` | N | N | - | N | - | N | N | N | N | Category 3 \| Category 4 \| Category 5, etc. | `Category 4` |
| 87 | GxP Classification | `gxp_type` | `varchar(50)` | N | N | - | N | - | N | N | N | N | GMP \| GLP \| GDP \| Non-GxP, etc. | `GMP` |
| 88 | Status | `status` | `varchar(20)` | N | N | - | Y | `Active'` | N | N | N | Y | Active \| Under Review \| Inactive | `Active` |
| 89 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-25T00:00:00` |
| 90 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-25T00:00:00` |

[↑ Back to Top](#top)

---

<a id="table-backup_execution"></a>
### 14. Backup Execution History (`backup_execution`)

| Item | Definition |
| --- | --- |
| Description | Manages execution statuses, backup scopes, storage locations, failures, and retry history for scheduled and manual system data and file backups |
| Primary Key | `backup_execution_id` |
| Key References (FK) | `requested_by, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain backup execution history for the required audit period according to backup policies and regulations |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 728 | Backup Execution ID | `backup_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique backup execution history identifier | `UUID` |
| 729 | Backup Type | `backup_type` | `varchar(30)` | N | N | - | Y | - | N | Y | N | Y | Backup type: FULL, INCREMENTAL, DATABASE, FILE | `FULL` |
| 730 | Execution Mode | `execution_type` | `varchar(20)` | N | N | - | Y | `SCHEDULED` | N | Y | N | Y | Execution mode: SCHEDULED, ON_DEMAND | `SCHEDULED` |
| 731 | Backup Target | `backup_target` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Backup target category: DATABASE, FILE_STORAGE, ALL | `ALL` |
| 732 | Backup Reference Time | `backup_base_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Reference time for the data to be backed up | `2026-09-02 18:00:00+00` |
| 733 | Execution Status | `execution_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Execution status: PENDING, PROCESSING, COMPLETED, RETRY_WAIT, FAILED, CANCELLED | `COMPLETED` |
| 734 | Backup Storage Location | `backup_location` | `text` | N | N | - | N | - | N | N | N | Y | Backup file location or storage path. Authentication information such as access tokens must not be stored. | `backups/YYYY/MM/DD/full` |
| 735 | Backup File Size | `backup_size_bytes` | `bigint` | N | N | - | N | - | N | N | N | Y | Total size of generated backup files (bytes) | `1073741824` |
| 736 | Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User requesting manual execution. NULL allowed for scheduled batch executions. | `UUID` |
| 737 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when a manual request was received or a scheduled backup was registered | `2026-09-02 18:00:00+00` |
| 738 | Execution Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the backup job actually started | `2026-09-02 18:00:05+00` |
| 739 | Execution Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the backup job ended in success or final failure | `2026-09-02 18:20:00+00` |
| 740 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of retries after the initial execution failed. Must be 0 or greater. | `0` |
| 741 | Maximum Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Maximum number of automatic retries allowed. Must be 0 or greater. | `3` |
| 742 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next scheduled execution time for a job in RETRY_WAIT status | `2026-09-02 18:30:00+00` |
| 743 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | System error code categorizing the cause of backup failure | `BACKUP_STORAGE_UNAVAILABLE` |
| 744 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Details of backup failure. Sensitive information such as passwords and access keys must not be stored. | `Failed to connect to backup storage` |
| 745 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Backup execution history creation timestamp (UTC) | `2026-09-02 18:00:00+00` |
| 746 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when created by a user. NULL allowed when created by the system or a batch job. | `UUID` |
| 747 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Backup execution history last modification timestamp (UTC) | `2026-09-02 18:20:00+00` |
| 748 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when modified by a user. NULL allowed for system/batch processing. | `UUID` |

[↑ Back to Top](#top)

---

## Library

<a id="table-library_item"></a>
### 15. Library Item Master (`library_item`)

Under the current dictionary, this is an independent master with no direct FK connections to other tables. Its use in preparing URS, IQ, and OQ is a business relationship, distinct from a database reference relationship.

| Item | Definition |
| --- | --- |
| Description | Master list of reusable standard library items for URS, IQ, and OQ |
| Primary Key | `library_id` |
| Key References (FK) | - |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 91 | Library ID | `library_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique library item identifier | `00000000-0000-0000-0000-000000000001` |
| 92 | Module Type | `module_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | URS \| IQ \| OQ classification | `URS` |
| 93 | Code | `code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | Item code (e.g., URS-AT-L01) | `URS-AT-L01` |
| 94 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Audit trail, calibration/verification, security, etc. | `Audit Trail` |
| 95 | Item Name | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Library item name/overview | `Automatic generation of audit trails for data changes` |
| 96 | Requirement/Procedure | `requirement_text` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed requirement specification or execution procedure | `Whenever any data is created, modified, or deleted...` |
| 97 | Expected Result | `expected_result` | `text` | N | N | - | N | - | N | N | N | Y | Expected result for IQ/OQ tests (not used for URS) | - |
| 98 | Acceptance Criteria | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | Criteria for a pass/fail determination | `Automatically generate an Audit Trail when data changes...` |
| 99 | Regulatory Reference | `regulation` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Related regulations (e.g., 21 CFR 11.10(e), KGMP) | `21 CFR 11.10(e)` |
| 100 | Is Active | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Whether active (TRUE/FALSE) | `True` |
| 101 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-25T00:00:00` |
| 102 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-25T00:00:00` |

[↑ Back to Top](#top)

---

## Validation

<a id="table-validation_project"></a>
### 16. Validation Project (`validation_project`)

| Item | Definition |
| --- | --- |
| Description | Unit and scope of validation execution for each system (VP) |
| Primary Key | `project_id` |
| Key References (FK) | `system_id, created_by, updated_by, closure_requested_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 103 | Project ID | `project_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique project identifier | `00000000-0000-0000-0000-000000000001` |
| 104 | Project Code | `project_code` | `varchar(100)` | N | N | - | Y | - | Y | Y | N | Y | Project identification code | `VP-SYS-008-20260422` |
| 105 | Project Name | `project_name` | `varchar(200)` | N | N | - | Y | - | N | Y | N | Y | Project Name | `Test Equipment 3 CSV Project` |
| 106 | System ID | `system_id` | `uuid` | N | Y | `system_asset.system_id` | Y | - | N | Y | N | Y | References system_asset.system_id | `00000000-0000-0000-0000-000000000001` |
| 107 | Progress Percentage | `progress_rate` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Project progress (%) | `0` |
| 108 | Status | `status` | `varchar(20)` | N | N | - | Y | `In Progress'` | N | N | N | Y | In Progress \| Completed \| Forced Closure \| On Hold | `In Progress` |
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
| 120 | Closure Type | `closure_type` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | Project closure type: NORMAL, FORCED. NULL is allowed before closure | `FORCED` |
| 121 | Forced Closure Reason | `closure_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for forced closure, which must be recorded when closure_type is FORCED | `Validation discontinued due to a change in business priorities` |
| 122 | Closure Requester ID | `closure_requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | ID of the user who requested normal or forced closure and provided an electronic signature | `00000000-0000-0000-0000-000000000001` |
| 123 | Closure Requested At | `closure_requested_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Timestamp when project closure was requested and the closure electronic signature was provided | `2026-09-14 09:00:00+00` |
| 124 | Closed At | `closed_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Timestamp when the status was finally changed to normal or forced closure | `2026-09-14 09:05:00+00` |

[↑ Back to Top](#top)

---

<a id="table-project_member"></a>
### 17. Project Member (`project_member`)

| Item | Definition |
| --- | --- |
| Description | Manages participating users and assigned roles for each project |
| Primary Key | `project_member_id` |
| Key References (FK) | `project_id, user_id, role_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 540 | Project Member ID | `project_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique project member role mapping identifier. Duplicate (project_id, user_id, role_id) combinations are not allowed in active data. | `UUID` |
| 541 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | ID of the validation project to which the member belongs | `UUID` |
| 542 | User ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user participating in the project | `UUID` |
| 543 | Role ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | ID of the role performed by the user within the project | `UUID` |
| 544 | Participation Status | `member_status` | `varchar(20)` | N | N | - | Y | `ACTIVE` | N | Y | N | Y | Project participation status: ACTIVE, INACTIVE, WITHDRAWN | `ACTIVE` |
| 545 | Participation Started At | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time when project participation began | `2026-09-01T10:00:00` |
| 546 | Participation Ended At | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when project participation ended. NULL if currently participating. | - |
| 547 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project participation information creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 548 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who registered the project participation information | `UUID` |
| 549 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project participation information last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 550 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who last modified the project participation information | `UUID` |
| 551 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Project participation information soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-validation_activity"></a>
### 18. Validation Activity Master (`validation_activity`)

| Item | Definition |
| --- | --- |
| Description | Manages reference information for VP, QIA, VA, URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ, RTM, and VSR activities |
| Primary Key | `activity_id` |
| Key References (FK) | `created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 552 | Activity ID | `activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique validation activity identifier | `UUID` |
| 553 | Activity Code | `activity_code` | `varchar(20)` | N | N | - | Y | - | Y | Y | N | Y | Activity identification code: SYSTEM_IDENTIFICATION, VP, QIA, VA, URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ, RTM, VSR | `URS` |
| 554 | Activity Name | `activity_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Activity name displayed in the UI | `User Requirements Specification` |
| 555 | Activity Order | `display_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Default display order in the UI and business process | `5` |
| 556 | Is Active | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the activity master record is enabled | `True` |
| 557 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Activity master creation timestamp (UTC) | `09/01/2026 10:00:00` |
| 558 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the activity master record | `UUID` |
| 559 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Activity master last modification timestamp (UTC) | `09/01/2026 10:00:00` |
| 560 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the activity master record | `UUID` |
| 561 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Activity master soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-project_activity"></a>
### 19. Project Activity (`project_activity`)

| Item | Definition |
| --- | --- |
| Description | Manages activities included in each project, their enablement, and progress statuses |
| Primary Key | `project_activity_id` |
| Key References (FK) | `project_id, activity_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 562 | Project Activity ID | `project_activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique project activity identifier. Duplicate (project_id, activity_id) combinations are not allowed in active data. | `UUID` |
| 563 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Validation project to which the activity belongs | `UUID` |
| 564 | Activity ID | `activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | Activity to be performed in the project | `UUID` |
| 565 | Is Included | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Whether the activity is included in the activities to be performed under the VP | `True` |
| 566 | Is Mandatory Activity | `is_required` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Whether the activity cannot be omitted from this project | `True` |
| 567 | Activity Status | `activity_status` | `varchar(20)` | N | N | - | Y | `LOCKED` | N | Y | N | Y | Activity status: LOCKED, READY, IN_PROGRESS, COMPLETED, APPROVED, SKIPPED | `READY` |
| 568 | Enabled At | `activated_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the activity became READY after its prerequisites were satisfied | `2026-09-01T10:00:00` |
| 569 | Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Activity execution start time | `2026-09-01T11:00:00` |
| 570 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Activity execution completion time | `2026-09-02T15:00:00` |
| 571 | Approved At | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when final approval of the activity was completed | `2026-09-02T17:00:00` |
| 572 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project activity creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 573 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the project activity | `UUID` |
| 574 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Project activity last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 575 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the project activity | `UUID` |
| 576 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Project activity soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-activity_dependency"></a>
### 20. Activity Dependency (`activity_dependency`)

| Item | Definition |
| --- | --- |
| Description | Manages predecessor activities, relationship classifications, and evaluation conditions for enabling successor activities |
| Primary Key | `activity_dependency_id` |
| Key References (FK) | `successor_activity_id, predecessor_activity_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 577 | Activity Dependency ID | `activity_dependency_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique activity dependency identifier | `UUID` |
| 578 | Successor Activity ID | `successor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | Activity enabled after the conditions are satisfied | `UUID` |
| 579 | Predecessor Activity ID | `predecessor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | N | - | N | Y | N | Y | Activity to check before enabling the successor activity. NULL allowed for conditions covering all activities. | `UUID` |
| 580 | Relationship Classification | `dependency_type` | `varchar(20)` | N | N | - | Y | `REQUIRED` | N | Y | N | Y | Dependency relationship classification: REQUIRED, RECOMMENDED | `REQUIRED` |
| 581 | Required Status | `required_status` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | Status required of the predecessor activity: CREATED, COMPLETED, APPROVED | `APPROVED` |
| 582 | Condition Type | `condition_type` | `varchar(50)` | N | N | - | Y | `STATUS` | N | Y | N | Y | STATUS, ACTIVITY_SELECTED, CONTEXT_CONFIRMED, GXP_SCOPE_CONFIRMED, TRACEABILITY_EXISTS, HIGH_RISK_COVERED, OPEN_DEVIATION_ZERO, ALL_SELECTED_APPROVED | `STATUS` |
| 583 | Condition Value | `condition_value` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Additional value or target entity type required to evaluate the condition | `APPROVED` |
| 584 | Condition Description | `condition_description` | `text` | N | N | - | Y | - | N | N | N | Y | Human-readable description of the enablement condition | `URS approval completed` |
| 585 | Evaluation Order | `evaluation_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Order in which conditions for the same successor activity are evaluated | `1` |
| 586 | Is Active | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the enablement condition is enabled | `True` |
| 587 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Condition creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 588 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the condition | `UUID` |
| 589 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Condition last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 590 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the condition | `UUID` |
| 591 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Condition soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## QIA

<a id="table-qia_assessment"></a>
### 21. Quality Impact Assessment Header (`qia_assessment`)

| Item | Definition |
| --- | --- |
| Description | Overall QIA assessment and 21 CFR Part 11 assessment results for each project |
| Primary Key | `qia_id` |
| Key References (FK) | `project_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 125 | QIA ID | `qia_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique QIA assessment identifier | `00000000-0000-0000-0000-000000000001` |
| 126 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 127 | Part 11 Q1: Electronic Record Creation | `p11_q1` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Q1: Whether electronic records are created/modified/maintained (Yes/No) | `Yes` |
| 128 | Part 11 Q2: Electronic Storage | `p11_q2` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Q2: Whether records are stored electronically (Yes/No) | `Yes` |
| 129 | Part 11 Q3: Submission to Regulatory Authorities | `p11_q3` | `varchar(10)` | N | N | - | Y | `No'` | N | N | N | Y | Q3: Whether electronic records are submitted to regulatory authorities (Yes/No) | `No` |
| 130 | Part 11 Q4: Electronic Signature Use | `p11_q4` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Q4: Whether electronic signatures are used (Yes/No) | `Yes` |
| 131 | Part 11 Q5: Replacement of Handwritten Signatures | `p11_q5` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Q5: Whether electronic signatures replace handwritten signatures (Yes/No) | `Yes` |
| 132 | Part 11 Q6: System Type | `p11_q6` | `varchar(20)` | N | N | - | Y | `Closed'` | N | N | N | Y | Closed System \| Open System | `Closed` |
| 133 | Part 11 Assessment Conclusion | `part11_result` | `text` | N | N | - | Y | - | N | N | N | Y | Conclusion on the applicability of 21 CFR Part 11 requirements | `System subject to 21 CFR Part 11 (Closed System)` |
| 134 | GxP Scope Status | `gxp_scope_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | Confirmation status of the GxP scope determined through QIA: DRAFT, CONFIRMED | `CONFIRMED` |
| 135 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the QIA assessment document (e.g., v1.0, v1.1) | `v1.0` |
| 136 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the QIA assessment document (e.g., 1, 2, 3...) | `1` |
| 137 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the QIA assessment document | `Initial creation` |
| 138 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest QIA assessment document version (TRUE/FALSE) | `True` |
| 139 | Authoring/Review/Approval Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Document progress status (DRAFT, REVIEW, APPROVED, etc.) | `DRAFT` |
| 140 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |

[↑ Back to Top](#top)

---

<a id="table-qia_module_item"></a>
### 22. QIA Module Detailed Assessment (`qia_module_item`)

| Item | Definition |
| --- | --- |
| Description | GxP Q1–Q10 items and assessment results for each module/process |
| Primary Key | `qia_module_item_id` |
| Key References (FK) | `qia_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 141 | QIA Module Item ID | `qia_module_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique QIA module assessment item identifier | `UUID` |
| 142 | QIA ID | `qia_id` | `uuid` | N | Y | `qia_assessment.qia_id` | Y | - | N | Y | N | Y | Parent QIA assessment identifier | `UUID` |
| 143 | Module Code | `module_code` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Parent module code | `QM` |
| 144 | Module Name | `module_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Parent module name | `Quality Control` |
| 145 | Module Description | `module_description` | `text` | N | N | - | N | - | N | N | N | Y | Description of the module's scope and purpose | `Overall assessment related to quality control` |
| 146 | Process Code | `process_code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Process identification code | `PRC-001` |
| 147 | Process Name | `process_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Detailed process name | `Work Order` |
| 148 | Q1 GxP Purpose | `q1_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `O` |
| 149 | Q2 Production Process | `q2_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `O` |
| 150 | Q3 Production Data | `q3_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 151 | Q4 Quality Impact | `q4_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `O` |
| 152 | Q5 Storage Management | `q5_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 153 | Q6 Release Approval | `q6_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 154 | Q7 Recall | `q7_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 155 | Q8 Regulatory Documents | `q8_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 156 | Q9 Document Management | `q9_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 157 | Q10 Security | `q10_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 158 | Assessment Result | `result_type` | `varchar(20)` | N | N | - | Y | `Non-GxP'` | N | N | N | Y | GxP \| Non-GxP | `GxP` |
| 159 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T10:00:00` |
| 160 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T10:00:00` |

[↑ Back to Top](#top)

---

## VA

<a id="table-vendor_audit"></a>
### 23. Vendor Audit Assessment (`vendor_audit`)

| Item | Definition |
| --- | --- |
| Description | Records vendor inspection/audit plans, execution, and deficiency counts |
| Primary Key | `audit_id` |
| Key References (FK) | `project_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 161 | Audit ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique vendor audit identifier | `00000000-0000-0000-0000-000000000001` |
| 162 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 163 | Document Number | `document_number` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Audit report document number | `VA-2026-001` |
| 164 | Vendor Name | `vendor_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Name of the vendor being audited | `Sample Vendor` |
| 165 | Target System | `system_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | Name/version of the system being audited | `Sample System v1.0` |
| 166 | Audit Method | `audit_type` | `varchar(50)` | N | N | - | Y | `On-site Audit'` | N | N | N | Y | On-site Audit \| Document Audit \| Remote Audit | `On-site Audit` |
| 167 | Audit Date | `audit_date` | `date` | N | N | - | Y | `CURRENT_DATE` | N | N | N | Y | Date of audit execution (or scheduled date) | `2026-08-26T00:00:00` |
| 168 | Auditor | `auditor_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Name of the person responsible for/conducting the audit | `Hong Gil-dong` |
| 169 | Audit Result | `audit_result` | `varchar(20)` | N | N | - | Y | `Compliant'` | N | N | N | Y | Compliant \| Conditionally Compliant \| Noncompliant | `Compliant` |
| 170 | Critical Deficiency Count | `critical_defects` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of critical deficiencies identified | `0` |
| 171 | Major Deficiency Count | `major_defects` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of major deficiencies identified | `0` |
| 172 | Minor Deficiency Count | `minor_defects` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of minor deficiencies identified | `1` |
| 173 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the audit document (e.g., v1.0, v1.1) | `v1.0` |
| 174 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the audit document (e.g., 1, 2, 3...) | `1` |
| 175 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the audit document | `Initial creation` |
| 176 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest audit document version (TRUE/FALSE) | `True` |
| 177 | Status | `status` | `varchar(20)` | N | N | - | Y | `Draft'` | N | N | N | Y | Draft \| Completed \| Approved | `Draft` |
| 178 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 179 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |

[↑ Back to Top](#top)

---

## URS

<a id="table-requirement"></a>
### 24. User Requirements Specification (`requirement`)

| Item | Definition |
| --- | --- |
| Description | Manages URS requirement items and document revision versions for each project |
| Primary Key | `requirement_id` |
| Key References (FK) | `project_id, created_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 180 | Requirement ID | `requirement_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique requirement identifier | `00000000-0000-0000-0000-000000000001` |
| 181 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | References validation_project.project_id | `00000000-0000-0000-0000-000000000001` |
| 182 | Item Number | `item_number` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Requirement management number (e.g., URS-001) | `URS-001` |
| 183 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | System management, audit trail, electronic signatures, etc. | `Electronic Signature` |
| 184 | Item/Function | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Requirement item name and main functionality | `Record electronic signature signer, date/time, and meaning` |
| 185 | Requirement Details | `requirement_text` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed requirement specification | `When electronically signing, the signer ID, signing date/time...` |
| 186 | Regulatory Reference | `regulation` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Related laws and regulations (e.g., 21 CFR 11.50) | `21 CFR 11.50` |
| 187 | Status | `status` | `varchar(20)` | N | N | - | Y | `Draft'` | N | N | N | Y | Draft \| Under Review \| Approved \| Rejected | `Draft` |
| 188 | Document Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0'` | N | N | N | Y | Requirement document revision version (e.g., v1.0, v2.0) | `v1.0` |
| 189 | Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Version revision sequence number (1, 2, 3...) | `1` |
| 190 | Is Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest version (TRUE/FALSE) | `True` |
| 191 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating/revising the version | `Initial creation` |
| 192 | Is Linked to RTM | `is_rtm_linked` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | Whether linked to the Traceability Matrix | `False` |
| 193 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the requirement author | `00000000-0000-0000-0000-000000000001` |
| 194 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 195 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |

[↑ Back to Top](#top)

---

## FDS

<a id="table-fds_spec"></a>
### 25. Functional Design Specification (`fds_spec`)

| Item | Definition |
| --- | --- |
| Description | Manages FDS document headers, versions, and statuses for each project |
| Primary Key | `fds_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 196 | FDS ID | `fds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | FDS identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 197 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 198 | FDS Number | `fds_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | FDS document number | `VP-SYS-008-20260422` |
| 199 | FDS Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the functional design specification | `Test Equipment 3 CSV Project` |
| 200 | Document Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | Standard FDS document version | `v1.0` |
| 201 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Version revision sequence number (e.g., 1, 2, 3...) | `1` |
| 202 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating/revising the version | `Initial creation` |
| 203 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest version (TRUE/FALSE) | `True` |
| 204 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Document status (Draft, Under Review, Approved) | `Draft` |
| 205 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 206 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the FDS author | `00000000-0000-0000-0000-000000000001` |
| 207 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 208 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the FDS modifier | `00000000-0000-0000-0000-000000000001` |
| 209 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-fds_item"></a>
### 26. FDS Detailed Item (`fds_item`)

| Item | Definition |
| --- | --- |
| Description | Manages functional, screen, and detailed design items in FDS documents |
| Primary Key | `fds_item_id` |
| Key References (FK) | `fds_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 210 | FDS Item ID | `fds_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | FDS item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 211 | FDS ID | `fds_id` | `uuid` | N | Y | `fds_spec.fds_id` | Y | - | N | N | N | Y | Parent FDS document identifier | `00000000-0000-0000-0000-000000000001` |
| 212 | Item Type | `item_type` | `varchar(20)` | N | N | - | Y | `FUNCTION` | N | N | N | Y | Subtab classification (FUNCTION, SCREEN, INTERFACE) | `FUNCTION` |
| 213 | FDS Item Number | `fds_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | FDS item management number | `FDS-001` |
| 214 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Function Category | `System Management` |
| 215 | Item/Function | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Function name and main functionality | `Electronic Signature` |
| 216 | Description | `description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed function description | `Electronic signature verification during user login` |
| 217 | Related Screen | `related_screen` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Name of the related screen | `User Management Screen` |
| 218 | Individual Revision Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Revision sequence number (1, 2, 3...) | `1` |
| 219 | Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Progress status (Draft, Under Review, Approved) | `Draft` |
| 220 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 221 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 222 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 223 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 224 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-fds_interface"></a>
### 27. FDS Interface Definition (`fds_interface`)

| Item | Definition |
| --- | --- |
| Description | Manages inter-system interfaces, exchanged data, and transmission methods in FDS documents |
| Primary Key | `fds_interface_id` |
| Key References (FK) | `fds_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 225 | FDS Interface ID | `fds_interface_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique FDS interface record identifier | `UUID` |
| 226 | FDS ID | `fds_id` | `uuid` | N | Y | `fds_spec.fds_id` | Y | - | N | Y | N | Y | Parent FDS document identifier | `UUID` |
| 227 | Interface Management Number | `interface_id` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Interface management number within the FDS document | `IF-001` |
| 228 | Source System | `source_system` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Name of the sending system | `Sample System` |
| 229 | Target System | `target_system` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Name of the receiving system | `Target System` |
| 230 | Exchanged Data | `interface_data` | `text` | N | N | - | Y | - | N | N | N | Y | Data items to be transmitted | `Material consumption, batch results` |
| 231 | Integration Frequency | `transfer_cycle` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Real-time, periodic, event-driven, etc. | `Real-time` |
| 232 | Integration Method | `transfer_method` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | REST API, Message Queue, etc. | `REST API` |
| 233 | Linked FDS Number | `fds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Display value of the related FDS item number | `FDS-008` |
| 234 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T10:00:00` |
| 235 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the interface author | `UUID` |
| 236 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T10:00:00` |
| 237 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Identifier of the user who last modified the interface | `UUID` |
| 238 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## DQ

<a id="table-dq_assessment"></a>
### 28. Design Qualification Assessment (`dq_assessment`)

| Item | Definition |
| --- | --- |
| Description | Manages design qualification assessment document headers for each project |
| Primary Key | `dq_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 239 | DQ ID | `dq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | DQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 240 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 241 | DQ Number | `dq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | DQ document number (e.g., DQ-VP-SYS-008-20260422) | `DQ-VP-SYS-008-20260422` |
| 242 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the design qualification assessment document | `Design Qualification Assessment (URS → FDS/DDS Mapping)` |
| 243 | Document Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | Standard DQ document version | `v1.0` |
| 244 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the DQ assessment document (e.g., 1, 2, 3...) | `1` |
| 245 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the DQ assessment document | `Initial creation` |
| 246 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest DQ assessment document version (TRUE/FALSE) | `True` |
| 247 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Document status (Draft, Under Review, Approved) | `Draft` |
| 248 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 249 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the DQ author | `00000000-0000-0000-0000-000000000001` |
| 250 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 251 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the DQ modifier | `00000000-0000-0000-0000-000000000001` |
| 252 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-dq_item"></a>
### 29. DQ Detailed Assessment Item (`dq_item`)

| Item | Definition |
| --- | --- |
| Description | Manages qualification assessment results for URS requirements and FDS/DDS designs |
| Primary Key | `dq_item_id` |
| Key References (FK) | `dq_id, requirement_id, reviewed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 253 | DQ Item ID | `dq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Detailed DQ item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 254 | DQ ID | `dq_id` | `uuid` | N | Y | `dq_assessment.dq_id` | Y | - | N | N | N | Y | Parent DQ document identifier | `00000000-0000-0000-0000-000000000001` |
| 255 | URS Item ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | N | N | Y | Identifier of the URS requirement to be mapped (FK) | `00000000-0000-0000-0000-000000000001` |
| 256 | URS Number | `urs_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | URS number for display in the UI (e.g., URS-001) | `URS-001` |
| 257 | URS Requirement | `urs_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed URS requirement content | `User login and electronic signature functionality` |
| 258 | FDS Link | `fds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked FDS number (e.g., FDS-001) | `FDS-001` |
| 259 | FDS Function | `fds_feature_name` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | Name of the linked FDS function | `Electronic signature verification` |
| 260 | DDS Link | `dds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked DDS design number | `DDS-001` |
| 261 | DDS Design | `dds_description` | `text` | N | N | - | N | - | N | N | N | Y | Linked DDS database/component design | `User Auth Table Schema` |
| 262 | Outcome | `result_status` | `varchar(20)` | N | N | - | Y | `Pending Review` | N | N | N | Y | Assessment outcome (PASS, FAIL, PENDING) | `PASS` |
| 263 | Reviewer ID | `reviewed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | Identifier of the user who reviewed the item | `00000000-0000-0000-0000-000000000001` |
| 264 | Remarks | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | Review-related observations and remarks | `Confirmed incorporation into FDS and DDS designs` |
| 265 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 266 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 267 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 268 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 269 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## FRA

<a id="table-fra_assessment"></a>
### 30. Functional Risk Assessment (`fra_assessment`)

| Item | Definition |
| --- | --- |
| Description | Manages functional risk assessment document headers for each project |
| Primary Key | `fra_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 270 | FRA ID | `fra_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | FRA assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 271 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 272 | FRA Number | `fra_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | FRA document number (e.g., FRA-VP-SYS-008-20260422) | `FRA-VP-SYS-008-20260422` |
| 273 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the FMEA-based functional risk assessment document | `FMEA-Based Functional Risk Assessment` |
| 274 | Document Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | Standard FRA document version | `v1.0` |
| 275 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the FRA assessment document (e.g., 1, 2, 3...) | `1` |
| 276 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the FRA assessment document | `Initial creation` |
| 277 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest FRA assessment document version (TRUE/FALSE) | `True` |
| 278 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Document status (Draft, Under Review, Approved) | `Draft` |
| 279 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 280 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the FRA author | `00000000-0000-0000-0000-000000000001` |
| 281 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 282 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the FRA modifier | `00000000-0000-0000-0000-000000000001` |
| 283 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-fra_item"></a>
### 31. FRA Detailed Risk Item (`fra_item`)

| Item | Definition |
| --- | --- |
| Description | Manages risk scenarios, risk scores, and mitigation strategies for each function |
| Primary Key | `fra_item_id` |
| Key References (FK) | `fra_id, requirement_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 284 | Risk Item ID | `fra_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Detailed FRA risk item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 285 | FRA ID | `fra_id` | `uuid` | N | Y | `fra_assessment.fra_id` | Y | - | N | N | N | Y | Parent FRA document identifier | `00000000-0000-0000-0000-000000000001` |
| 286 | URS Item ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | N | - | N | N | N | Y | Identifier of the mapped URS requirement (FK) | `00000000-0000-0000-0000-000000000001` |
| 287 | URS Reference Number | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | URS reference number for display in the UI (e.g., URS-001) | `URS-001` |
| 288 | Function Name | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Name of the function being assessed for risk | `Electronic Signature` |
| 289 | Risk Scenario | `risk_scenario` | `text` | N | N | - | Y | - | N | N | N | Y | Description of the FMEA risk occurrence scenario | `Password verification not performed during electronic signing` |
| 290 | Product Impact (PI) | `pi_score` | `varchar(10)` | N | N | - | Y | `H` | N | N | N | Y | Product Impact (H, M, L) | `H` |
| 291 | Likelihood of Occurrence (LL) | `ll_score` | `varchar(10)` | N | N | - | Y | `H` | N | N | N | Y | Likelihood of Occurrence (H, M, L) | `M` |
| 292 | Detectability (DL) | `dl_score` | `varchar(10)` | N | N | - | Y | `H` | N | N | N | Y | Detectability (H, M, L) | `L` |
| 293 | Risk Value (RV) | `risk_value` | `integer` | N | N | - | N | `1` | N | N | N | Y | Risk value (RV) calculated from the combination of PI, LL, and DL | `1` |
| 294 | Risk Rating | `risk_level` | `varchar(20)` | N | N | - | Y | `LOW` | N | Y | N | Y | Calculated risk rating: LOW, MEDIUM, HIGH | `HIGH` |
| 295 | Mitigation Strategy | `mitigation_strategy` | `varchar(20)` | N | N | - | Y | `Test` | N | N | N | Y | Risk mitigation strategy (Test, SOP, No Action) | `Test` |
| 296 | Linked Test | `test_reference` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked verification test code (e.g., OQ-AT-01) | `OQ-AT-01` |
| 297 | Progress Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Risk item status (Draft, Review Completed) | `Draft` |
| 298 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 299 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 300 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 301 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 302 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## IQ

<a id="table-iq_assessment"></a>
### 32. Installation Qualification Assessment (`iq_assessment`)

| Item | Definition |
| --- | --- |
| Description | Manages installation qualification assessment document headers for each project |
| Primary Key | `iq_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 303 | IQ ID | `iq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | IQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 304 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 305 | IQ Number | `iq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | IQ document number (e.g., IQ-VP-SYS-010-20260529) | `IQ-VP-SYS-010-20260529` |
| 306 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the installation qualification assessment document | `Installation Qualification` |
| 307 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the IQ assessment document (e.g., v1.0, v1.1) | `v1.0` |
| 308 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the IQ assessment document (e.g., 1, 2, 3...) | `1` |
| 309 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the IQ assessment document | `Initial creation` |
| 310 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest IQ assessment document version (TRUE/FALSE) | `True` |
| 311 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | IQ protocol status: DRAFT, REVIEW, APPROVED | `APPROVED` |
| 312 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | IQ execution record status. After the protocol is APPROVED, progresses through DRAFT, REVIEW, and APPROVED. | `DRAFT` |
| 313 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 314 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the IQ author | `00000000-0000-0000-0000-000000000001` |
| 315 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 316 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the IQ modifier | `00000000-0000-0000-0000-000000000001` |
| 317 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-iq_item"></a>
### 33. IQ Detailed Test Item (`iq_item`)

| Item | Definition |
| --- | --- |
| Description | Manages IQ test procedures, results, evidence, and executors |
| Primary Key | `iq_item_id` |
| Key References (FK) | `iq_id, executed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 318 | IQ Item ID | `iq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Detailed IQ test item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 319 | IQ ID | `iq_id` | `uuid` | N | Y | `iq_assessment.iq_id` | Y | - | N | N | N | Y | Parent IQ document identifier | `00000000-0000-0000-0000-000000000001` |
| 320 | Procedure Number | `step_no` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Test execution procedure number (e.g., 01, 02) | `1` |
| 321 | Test ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Test item ID (e.g., IQ-NEW-01) | `IQ-NEW-01` |
| 322 | Test Case | `test_case` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Hardware installation, software installation, etc. | `Hardware Installation` |
| 323 | URS Link | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Mapped URS number (e.g., URS-001) | `URS-001` |
| 324 | Test Details | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed procedure for executing the verification test | `Verify that the hardware specifications of the server to be installed meet the URS requirements` |
| 325 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Test success criteria and expected results | `Hardware specifications must match the requirements specified in the URS` |
| 326 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `Approved` | N | N | N | Y | Protocol approval status for each item | `Approved` |
| 327 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Record of actual results after test execution | `IQ Test 1 results` |
| 328 | Outcome | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Final test outcome (Pass, Fail, Not Executed) | `Pass` |
| 329 | Actor ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | Test executor identifier (FK) | `00000000-0000-0000-0000-000000000001` |
| 330 | Executed At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Date and time when test execution was completed | `2026-06-22T00:00:00` |
| 331 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Record approval status for each item | `Approved` |
| 332 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 333 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 334 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 335 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 336 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## OQ

<a id="table-oq_assessment"></a>
### 34. Operational Qualification Assessment (`oq_assessment`)

| Item | Definition |
| --- | --- |
| Description | Manages operational qualification assessment document headers for each project |
| Primary Key | `oq_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 337 | OQ ID | `oq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | OQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 338 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 339 | OQ Number | `oq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | OQ document number (e.g., OQ-VP-SYS-010-20260529) | `OQ-VP-SYS-010-20260529` |
| 340 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the operational qualification assessment document | `Operational Qualification` |
| 341 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the OQ assessment document (e.g., v1.0, v1.1) | `v1.0` |
| 342 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the OQ assessment document (e.g., 1, 2, 3...) | `1` |
| 343 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the OQ assessment document | `Initial creation` |
| 344 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest OQ assessment document version (TRUE/FALSE) | `True` |
| 345 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | OQ protocol status: DRAFT, REVIEW, APPROVED | `APPROVED` |
| 346 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | OQ execution record status. After the protocol is APPROVED, progresses through DRAFT, REVIEW, and APPROVED. | `DRAFT` |
| 347 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 348 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the OQ author | `00000000-0000-0000-0000-000000000001` |
| 349 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 350 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the OQ modifier | `00000000-0000-0000-0000-000000000001` |
| 351 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-oq_item"></a>
### 35. OQ Detailed Test Item (`oq_item`)

| Item | Definition |
| --- | --- |
| Description | Manages OQ test procedures, results, evidence, and executors |
| Primary Key | `oq_item_id` |
| Key References (FK) | `oq_id, executed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 352 | OQ Item ID | `oq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Detailed OQ test item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 353 | OQ ID | `oq_id` | `uuid` | N | Y | `oq_assessment.oq_id` | Y | - | N | N | N | Y | Parent OQ document identifier | `00000000-0000-0000-0000-000000000001` |
| 354 | Procedure Number | `step_no` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Test execution procedure number (e.g., 01, 02, 03) | `1` |
| 355 | Test ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | Test item ID (e.g., OQ-AT-L01) | `OQ-AT-L01` |
| 356 | Test Case | `test_case` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Audit Trail, user management, backup and recovery, etc. | `Audit Trail` |
| 357 | URS Link | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Mapped URS number (e.g., URS-001) | `URS-001` |
| 358 | Test Details | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed procedure for executing the verification test | `Automatically generate an audit trail when user data changes` |
| 359 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Test success criteria and expected results | `Record before/after values, user, date/time, and IP address` |
| 360 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `Approved` | N | N | N | Y | Protocol approval status for each item | `Approved` |
| 361 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Record of actual results after test execution | `Actual result Test` |
| 362 | Outcome | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Final test outcome (Pass, Fail, Not Executed) | `Pass` |
| 363 | Actor ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | Test executor identifier (FK) | `00000000-0000-0000-0000-000000000001` |
| 364 | Executed At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Date and time when test execution was completed | `2026-06-22T00:00:00` |
| 365 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Record approval status for each item | `Approved` |
| 366 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 367 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 368 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 369 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 370 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## PQ

<a id="table-pq_assessment"></a>
### 36. Performance Qualification Assessment (`pq_assessment`)

| Item | Definition |
| --- | --- |
| Description | Manages performance qualification assessment document headers and execution plans for each project |
| Primary Key | `pq_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 371 | PQ ID | `pq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | PQ assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 372 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 373 | PQ Number | `pq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | PQ document number (e.g., PQ-VP-SYS-008-20260422) | `PQ-VP-SYS-008-20260422` |
| 374 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the performance qualification assessment document | `Performance Qualification` |
| 375 | Planned PQ Start Date | `start_scheduled_date` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Basis for the planned PQ start date (e.g., OQ completion + 5 business days) | `OQ completion + 5 business days` |
| 376 | Target PQ Completion Date | `target_completion_date` | `date` | N | N | - | N | - | N | N | N | Y | Target date for PQ completion | `2024-04-15T00:00:00` |
| 377 | Execution Method | `execution_method` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | PQ execution method (e.g., actual production data from at least 3 batches) | `Actual production data from at least 3 batches` |
| 378 | Primary Executor | `primary_executor` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Team/department primarily responsible for execution (e.g., Process Owner (Production Team)) | `Process Owner (Production Team)` |
| 379 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the PQ assessment document (e.g., v1.0, v1.1) | `v1.0` |
| 380 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the PQ assessment document (e.g., 1, 2, 3...) | `1` |
| 381 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the PQ assessment document | `Initial creation` |
| 382 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest PQ assessment document version (TRUE/FALSE) | `True` |
| 383 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Overall PQ assessment progress status (Awaiting Execution, In Progress, Completed) | `Awaiting PQ Execution` |
| 384 | Protocol Status | `protocol_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | PQ protocol status: DRAFT, REVIEW, APPROVED | `APPROVED` |
| 385 | Record Status | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | PQ execution record status. After the protocol is APPROVED, progresses through DRAFT, REVIEW, APPROVED. | `DRAFT` |
| 386 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 387 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the PQ author | `00000000-0000-0000-0000-000000000001` |
| 388 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 389 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the PQ modifier | `00000000-0000-0000-0000-000000000001` |
| 390 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-pq_item"></a>
### 37. PQ Detailed Test Item (`pq_item`)

| Item | Definition |
| --- | --- |
| Description | Manages PQ test procedures, results, evidence, and executors |
| Primary Key | `pq_item_id` |
| Key References (FK) | `pq_id, executed_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 391 | PQ Item ID | `pq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | Detailed PQ test item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 392 | PQ ID | `pq_id` | `uuid` | N | Y | `pq_assessment.pq_id` | Y | - | N | N | N | Y | Parent PQ document identifier | `00000000-0000-0000-0000-000000000001` |
| 393 | Procedure Number | `step_no` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Test execution procedure number (e.g., 01, 02) | `1` |
| 394 | Category | `category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | PQ assessment category (e.g., consecutive batch verification, data integrity, etc.) | `Consecutive Batch Verification` |
| 395 | Test Details | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed procedure for executing the PQ verification test | `Verify successful completion of the production process for at least 3 consecutive batches` |
| 396 | Expected Result | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | Test success criteria and expected results | `All batches must complete production successfully in accordance with specifications` |
| 397 | Actual Result | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | Record of actual results after test execution | `Confirmed successful completion of 3 consecutive batches` |
| 398 | Outcome | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Final test outcome (Pass, Fail, Not Executed) | `Pass` |
| 399 | Actor ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | Test executor identifier (FK) | `00000000-0000-0000-0000-000000000001` |
| 400 | Executed At | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Date and time when test execution was completed | `2026-08-26T00:00:00` |
| 401 | Progress Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | Progress status for each item (Draft, Approved) | `Draft` |
| 402 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 403 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 404 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 405 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 406 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## RTM

<a id="table-rtm_assessment"></a>
### 38. Requirements Traceability Matrix (`rtm_assessment`)

| Item | Definition |
| --- | --- |
| Description | Manages project-level requirements traceability, aggregated deliverable coverage, and RTM snapshots at the time of approval |
| Primary Key | `rtm_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 407 | RTM ID | `rtm_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | RTM assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 408 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 409 | RTM Number | `rtm_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | RTM document number (e.g., RTM-VP-SYS-008-20260422) | `RTM-VP-SYS-008-20260422` |
| 410 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the requirements traceability matrix document | `Requirements Traceability Matrix (Automatically Generated)` |
| 411 | Total URS Count | `total_urs_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Total number of URS requirements in the project | `8` |
| 412 | FRA Linkage Rate | `fra_link_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | FRA risk assessment linkage rate (%) | `88` |
| 413 | IQ Coverage | `iq_coverage_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | IQ test coverage (%) | `0` |
| 414 | OQ Coverage | `oq_coverage_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | OQ test coverage (%) | `44` |
| 415 | Overall Average Coverage | `avg_coverage_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | Integrated average coverage across all deliverables (%) | `44` |
| 416 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the RTM assessment document (e.g., v1.0, v1.1) | `v1.0` |
| 417 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the RTM assessment document (e.g., 1, 2, 3...) | `1` |
| 418 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the RTM assessment document | `Initial creation` |
| 419 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest RTM assessment document version (TRUE/FALSE) | `True` |
| 420 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `Draft` | N | N | N | Y | RTM progress status (Draft, Approved) | `Draft` |
| 421 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 422 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the RTM author | `00000000-0000-0000-0000-000000000001` |
| 423 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 424 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the RTM modifier | `00000000-0000-0000-0000-000000000001` |
| 425 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-rtm_item"></a>
### 39. RTM Detailed Traceability Item (`rtm_item`)

| Item | Definition |
| --- | --- |
| Description | Manages FRA, FDS, DDS, IQ, OQ, and PQ traceability results for each URS requirement generated from traceability_link, and detailed snapshots at the time of approval |
| Primary Key | `rtm_item_id` |
| Key References (FK) | `rtm_id, requirement_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 426 | RTM Item ID | `rtm_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | RTM matrix item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 427 | RTM ID | `rtm_id` | `uuid` | N | Y | `rtm_assessment.rtm_id` | Y | - | N | N | N | Y | Parent RTM document identifier | `00000000-0000-0000-0000-000000000001` |
| 428 | URS Item ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | N | N | Y | Identifier of the mapped URS requirement (FK) | `00000000-0000-0000-0000-000000000001` |
| 429 | URS Number | `urs_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | URS number for display in the UI (e.g., URS-001) | `URS-001` |
| 430 | Item/Function | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | URS requirement function name | `Record electronic signature signer, date/time, and meaning` |
| 431 | Requirement Content | `requirement_desc` | `text` | N | N | - | Y | - | N | N | N | Y | Detailed URS requirement content | `The signer ID and signing date/time must be recorded when an electronic signature is executed.` |
| 432 | Is Adopted | `adoption_status` | `varchar(10)` | N | N | - | Y | `O` | N | N | N | Y | Whether the URS requirement is adopted (O, X) | `O` |
| 433 | FRA Mapping | `fra_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked FRA number (e.g., FRA-001, N/A) | `N/A` |
| 434 | FDS Mapping | `fds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked FDS number (e.g., FDS-001, Mapping Failed) | `Mapping Failed` |
| 435 | DDS Mapping | `dds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked DDS number (e.g., DDS-001, Mapping Failed) | `Mapping Failed` |
| 436 | IQ Result | `iq_result` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked IQ test item number (e.g., IQ-NEW-07) | `IQ-NEW-07` |
| 437 | OQ Result | `oq_result` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked OQ test item number (e.g., OQ-AT-L01, Mapping Failed) | `Mapping Failed` |
| 438 | PQ Result | `pq_result` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Linked PQ test item number (e.g., PQ-01, N/A) | `N/A` |
| 439 | Item Coverage | `item_coverage_rate` | `numeric(5,2)` | N | N | - | Y | `0` | N | N | N | Y | Deliverable coverage percentage for the URS item (%) | `50` |
| 440 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 441 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 442 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 443 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 444 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## VSR

<a id="table-vsr_assessment"></a>
### 40. Validation Summary Report (`vsr_assessment`)

| Item | Definition |
| --- | --- |
| Description | Manages the final validation conclusion and summary report for each project |
| Primary Key | `vsr_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 445 | VSR ID | `vsr_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | VSR assessment document identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 446 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | Related validation project ID | `00000000-0000-0000-0000-000000000001` |
| 447 | VSR Number | `vsr_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | VSR document number (e.g., VSR-VP-SYS-008-20260422) | `VSR-VP-SYS-008-20260422` |
| 448 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the validation summary report | `Validation Summary Report` |
| 449 | Validation Conclusion | `overall_conclusion` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Final acceptability conclusion (Acceptable, Conditionally Acceptable, Unacceptable) | `Conditionally Acceptable` |
| 450 | Detailed Conclusion | `conclusion_remarks` | `text` | N | N | - | N | - | N | N | N | Y | Reasons and conditions for the conclusion (e.g., final approval possible after resolution of deviation OQ-GMP-02) | `Final approval possible after resolution of deviation OQ-GMP-02` |
| 451 | Document Display Version | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the VSR assessment document (e.g., v1.0, v1.1) | `v1.0` |
| 452 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | Revision sequence number of the VSR assessment document (e.g., 1, 2, 3...) | `1` |
| 453 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for creating or revising the VSR assessment document | `Initial creation` |
| 454 | Is Current Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently active latest VSR assessment document version (TRUE/FALSE) | `True` |
| 455 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `Under Review` | N | N | N | Y | VSR progress status (Draft, Under Review, Approved) | `Under Review` |
| 456 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 457 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the VSR author | `00000000-0000-0000-0000-000000000001` |
| 458 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 459 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the VSR modifier | `00000000-0000-0000-0000-000000000001` |
| 460 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-vsr_item"></a>
### 41. VSR Activity Summary Item (`vsr_item`)

| Item | Definition |
| --- | --- |
| Description | Manages summaries of documents, results, deviations, and approval information for each validation activity |
| Primary Key | `vsr_item_id` |
| Key References (FK) | `vsr_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 461 | VSR Item ID | `vsr_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | VSR activity summary item identifier (PK) | `00000000-0000-0000-0000-000000000001` |
| 462 | VSR ID | `vsr_id` | `uuid` | N | Y | `vsr_assessment.vsr_id` | Y | - | N | N | N | Y | Parent VSR document identifier | `00000000-0000-0000-0000-000000000001` |
| 463 | Activity Type | `activity_code` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Validation activity type (VP, QIA, VA, URS, FDS, DQ, FRA, IQ, OQ, PQ, RTM) | `URS` |
| 464 | Document Number | `doc_no` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Document number for the corresponding stage | `VP-SYS-008-20260422 · URS` |
| 465 | Revision Number | `revision_no` | `varchar(20)` | N | N | - | Y | `1` | N | N | N | Y | Document revision number (REV) | `2.1` |
| 466 | Executed At | `execution_date` | `date` | N | N | - | N | - | N | N | N | Y | Activity execution completion date | `2024-02-15T00:00:00` |
| 467 | Pass Count | `pass_count` | `integer` | N | N | - | N | - | N | N | N | Y | Number of PASS tests/requirements | `125` |
| 468 | Fail Count | `fail_count` | `integer` | N | N | - | N | - | N | N | N | Y | Number of FAIL results | `0` |
| 469 | Deviation Count | `deviation_info` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Number of deviations raised and resolved (e.g., 3 deviations (2 resolved), 1 (open)) | - |
| 470 | Conclusion/Status | `item_status` | `varchar(20)` | N | N | - | Y | `Pending` | N | N | N | Y | Conclusion for the document/activity (Approved, Under Review, Pending) | `Approved` |
| 471 | Approver Name | `approver_name` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | Full name of the final approver | `Hong Gil-dong` |
| 472 | Approval Date | `approval_date` | `date` | N | N | - | N | - | N | N | N | Y | Final approval date | `2024-02-20T00:00:00` |
| 473 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-26T00:00:00` |
| 474 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item author | `00000000-0000-0000-0000-000000000001` |
| 475 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-26T00:00:00` |
| 476 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Identifier of the item modifier | `00000000-0000-0000-0000-000000000001` |
| 477 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## Workflow

<a id="table-workflow_instance"></a>
### 42. Workflow Instance (`workflow_instance`)

| Item | Definition |
| --- | --- |
| Description | Manages review and approval workflow progress information for each document |
| Primary Key | `workflow_instance_id` |
| Key References (FK) | `requested_by, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 478 | Workflow Instance ID | `workflow_instance_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of the document's review and approval workflow | `UUID` |
| 479 | Target Table Name | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Physical table name of the document subject to the workflow | `fds_spec` |
| 480 | Target Record ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | N | N | Y | Primary key value of the document revision row subject to the workflow | `UUID` |
| 481 | Target Document Version | `target_version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Display version of the document revision subject to review and approval | `v1.0` |
| 482 | Submitted By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | User who submitted the document for review and approval | `UUID` |
| 483 | Submitted At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Time when the document was submitted for review and approval | `2026-08-31T15:00:00` |
| 484 | Workflow Status | `workflow_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | Overall workflow status: DRAFT, IN_PROGRESS, APPROVED, REJECTED, CANCELLED | `IN_PROGRESS` |
| 485 | Current Step Order | `current_step_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | Order of the review/approval step currently being processed | `1` |
| 486 | Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the workflow ended through final approval, rejection, or cancellation | `2026-08-31T17:00:00` |
| 487 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-31T15:00:00` |
| 488 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Creator of the workflow record | `UUID` |
| 489 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-31T16:00:00` |
| 490 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | User who last modified the workflow record | `UUID` |
| 491 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-workflow_step"></a>
### 43. Workflow Step (`workflow_step`)

| Item | Definition |
| --- | --- |
| Description | Manages processing statuses and assignees for individual workflow steps |
| Primary Key | `workflow_step_id` |
| Key References (FK) | `workflow_instance_id, assignee_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 492 | Workflow Step ID | `workflow_step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique workflow review/approval step identifier. Duplicate (workflow_instance_id, step_order) combinations are not allowed in active data. | `UUID` |
| 493 | Workflow Instance ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | Y | - | N | Y | N | Y | Identifier of the workflow to which the step belongs | `UUID` |
| 494 | Step Order | `step_order` | `integer` | N | N | - | Y | - | N | Y | N | Y | Review/approval processing order within the same workflow | `1` |
| 495 | Step Type | `step_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | Processing step type: REVIEW or APPROVE | `REVIEW` |
| 496 | Step Name | `step_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Review/approval step name displayed in the UI | `Quality Review` |
| 497 | Assignee ID | `assignee_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User assigned to process this step | `UUID` |
| 498 | Step Status | `step_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Step status: PENDING, IN_PROGRESS, APPROVED, REJECTED, SKIPPED | `PENDING` |
| 499 | Due Date | `due_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Scheduled deadline for reviewing/approving this step | `2026-09-02T18:00:00` |
| 500 | Action Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the step was completed through approval, rejection, or another outcome | `2026-09-01T10:00:00` |
| 501 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-08-31T15:00:00` |
| 502 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Creator of the workflow step | `UUID` |
| 503 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record last modification timestamp (UTC) | `2026-08-31T16:00:00` |
| 504 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | User who last modified the workflow step | `UUID` |
| 505 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-approval_action"></a>
### 44. Approval Action History (`approval_action`)

| Item | Definition |
| --- | --- |
| Description | Manages the history of actual actions at each step, such as review, approval, and rejection |
| Primary Key | `approval_action_id` |
| Key References (FK) | `workflow_step_id, actor_id, signature_id` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 506 | Approval Action History ID | `approval_action_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique review/approval action history identifier | `UUID` |
| 507 | Workflow Step ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | Y | - | N | Y | N | Y | Workflow step where the action occurred | `UUID` |
| 508 | Action Type | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Type of action performed: SUBMIT, REVIEW, APPROVE, REJECT, CANCEL | `APPROVE` |
| 509 | Actor ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who actually performed the review, approval, or rejection | `UUID` |
| 510 | Action Comments | `action_comment` | `text` | N | N | - | N | - | N | N | N | Y | Comments entered during review/approval | `No issues found during review` |
| 511 | Reason for Rejection | `rejection_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason entered when performing a REJECT action | `Additional evidence files required` |
| 512 | Electronic Signature ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | Electronic signature record linked to the approval/rejection action | `UUID` |
| 513 | Action Timestamp | `acted_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when submission, review, approval, or rejection was actually performed | `2026-09-01T10:00:00` |
| 514 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Record creation timestamp (UTC) | `2026-09-01T10:00:00` |

[↑ Back to Top](#top)

---

## Traceability

<a id="table-traceability_link"></a>
### 45. Common Traceability Link (`traceability_link`)

| Item | Definition |
| --- | --- |
| Description | Manages traceability relationships among URS, FDS, DQ, FRA, IQ, OQ, and PQ items |
| Primary Key | `traceability_link_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 515 | Traceability Link ID | `traceability_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique identifier of a traceability relationship between deliverable items. Duplicate (project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type) combinations are not allowed in active data. | `UUID` |
| 516 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | ID of the project to which the traceability relationship belongs | `UUID` |
| 517 | Source Entity Type | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Source entity type: REQUIREMENT, FDS_ITEM, DDS_ITEM, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM | `REQUIREMENT` |
| 518 | Source Entity ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Primary key value of the table corresponding to source_entity_type. No physical FK is defined because this is a polymorphic reference. | `UUID` |
| 519 | Linked Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Linked entity type: REQUIREMENT, FDS_ITEM, DDS_ITEM, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM | `IQ_ITEM` |
| 520 | Linked Entity ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Primary key value of the table corresponding to target_entity_type. No physical FK is defined because this is a polymorphic reference. | `UUID` |
| 521 | Link Type | `link_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | IMPLEMENTED_BY, ASSESSED_BY, VERIFIED_BY, MITIGATED_BY | `VERIFIED_BY` |
| 522 | Link Rationale | `link_reason` | `text` | N | N | - | N | - | N | N | N | Y | Business rationale for linking the two deliverable items | `Verify the URS requirement through an IQ test` |
| 523 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Traceability link creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 524 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who created the traceability link | `UUID` |
| 525 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Traceability link last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 526 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | ID of the user who last modified the traceability link | `UUID` |
| 527 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Traceability link soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## DDS

<a id="table-dds_spec"></a>
### 46. Detailed Design Specification (`dds_spec`)

| Item | Definition |
| --- | --- |
| Description | Manages DDS document headers, versions, and approval statuses for each project |
| Primary Key | `dds_id` |
| Key References (FK) | `project_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 592 | DDS ID | `dds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique DDS document identifier | `UUID` |
| 593 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Validation project to which the DDS belongs | `UUID` |
| 594 | DDS Number | `dds_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | DDS document number | `DDS-VP-SYS-001` |
| 595 | Document Title | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | Title of the detailed design specification | `Detailed Design Specification` |
| 596 | Document Version | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | DDS document display version | `v1.0` |
| 597 | Revision Sequence Number | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | DDS revision sequence number | `1` |
| 598 | Reason for Revision | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | Reason for initial creation or revision of the DDS | `Initial creation` |
| 599 | Is Latest Version | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether this is the currently valid latest DDS version | `True` |
| 600 | Document Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | DDS document status: DRAFT, REVIEW, APPROVED, REJECTED | `APPROVED` |
| 601 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS creation timestamp (UTC) | `2026-09-01T10:00:00` |
| 602 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who authored the DDS | `UUID` |
| 603 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS last modification timestamp (UTC) | `2026-09-01T10:00:00` |
| 604 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the DDS | `UUID` |
| 605 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | DDS soft deletion timestamp | - |

[↑ Back to Top](#top)

---

<a id="table-dds_item"></a>
### 47. DDS Detailed Item (`dds_item`)

| Item | Definition |
| --- | --- |
| Description | Manages detailed database, interface, and component design items in DDS documents |
| Primary Key | `dds_item_id` |
| Key References (FK) | `dds_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 606 | DDS Item ID | `dds_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique DDS detailed item identifier | `UUID` |
| 607 | DDS ID | `dds_id` | `uuid` | N | Y | `dds_spec.dds_id` | Y | - | N | Y | N | Y | Parent DDS document identifier | `UUID` |
| 608 | DDS Item Number | `item_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | DDS detailed item management number | `DDS-001` |
| 609 | Design Type | `design_type` | `varchar(30)` | N | N | - | Y | `COMPONENT` | N | Y | N | Y | DATABASE, COMPONENT, INTERFACE, SECURITY, BATCH | `DATABASE` |
| 610 | Design Name | `design_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Detailed design item name | `User Authentication Table Design` |
| 611 | Design Description | `description` | `text` | N | N | - | Y | - | N | N | N | Y | DDS detailed design content | `User authentication and permission table structure` |
| 612 | Status | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | DDS item status: DRAFT, REVIEW, APPROVED | `APPROVED` |
| 613 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS item creation timestamp (UTC) | `09/01/2026 10:00:00` |
| 614 | Author ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who authored the DDS item | `UUID` |
| 615 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS item last modification timestamp (UTC) | `09/01/2026 10:00:00` |
| 616 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the DDS item | `UUID` |
| 617 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | DDS item soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## Deviation

<a id="table-deviation"></a>
### 48. Deviation Management (`deviation`)

| Item | Definition |
| --- | --- |
| Description | Manages deviations arising in documents or during test execution and their investigation, resolution, and approval statuses |
| Primary Key | `deviation_id` |
| Key References (FK) | `project_id, resolved_by, approved_by, created_by, updated_by` |
| GxP Criticality | Critical |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 618 | Deviation ID | `deviation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique deviation identifier | `UUID` |
| 619 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | Validation project in which the deviation occurred | `UUID` |
| 620 | Source Entity Type | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Type of entity where the deviation occurred: IQ_ITEM, OQ_ITEM, PQ_ITEM, etc. | `OQ_ITEM` |
| 621 | Source Entity ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | Primary key value of the target entity. No physical FK is defined because this is a polymorphic reference. | `UUID` |
| 622 | Deviation Number | `deviation_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Deviation management number within the project | `DEV-001` |
| 623 | Deviation Title | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Deviation Title | `Mismatch with Expected Result` |
| 624 | Deviation Description | `description` | `text` | N | N | - | Y | - | N | N | N | Y | Deviation details and circumstances of occurrence | `Mismatch between expected and actual results during OQ execution` |
| 625 | Severity | `severity` | `varchar(20)` | N | N | - | Y | `MINOR` | N | Y | N | Y | Deviation severity: MINOR, MAJOR, CRITICAL | `MAJOR` |
| 626 | Deviation Status | `deviation_status` | `varchar(20)` | N | N | - | Y | `OPEN` | N | Y | N | Y | OPEN, INVESTIGATING, RESOLVED, CLOSED, CANCELLED | `OPEN` |
| 627 | Resolution Details | `resolution` | `text` | N | N | - | N | - | N | N | N | Y | Investigation of the deviation's cause and resolution details | `Retesting completed after correcting configuration values` |
| 628 | Resolved At | `resolved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when deviation resolution was completed | `09/03/2026 15:00:00` |
| 629 | Resolved By ID | `resolved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who resolved the deviation | `UUID` |
| 630 | Approved At | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deviation closure approval timestamp | `09/03/2026 17:00:00` |
| 631 | Approver ID | `approved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who approved deviation closure | `UUID` |
| 632 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Deviation creation timestamp (UTC) | `09/01/2026 10:00:00` |
| 633 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the deviation | `UUID` |
| 634 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Deviation last modification timestamp (UTC) | `09/01/2026 10:00:00` |
| 635 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the deviation | `UUID` |
| 636 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Deviation soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## Report

<a id="table-report_generation"></a>
### 49. Report Generation Job (`report_generation`)

| Item | Definition |
| --- | --- |
| Description | Manages generation requests, execution statuses, failures, retries, and output files for audit and operational reports |
| Primary Key | `report_generation_id` |
| Key References (FK) | `project_id, requested_by, result_file_id, created_by, updated_by, report_schedule_id` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain report generation results and execution history for the required audit period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 637 | Report Generation ID | `report_generation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique report generation job identifier | `UUID` |
| 638 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Linked for project-level reports. NULL allowed for system-wide or organization-level reports. | `UUID` |
| 639 | Report Type | `report_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Report type: AUDIT_TRAIL, PROJECT_STATUS, WORKFLOW_STATUS, DEVIATION_STATUS, TRACEABILITY, SYSTEM_OPERATION | `AUDIT_TRAIL` |
| 640 | Report Name | `report_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Name of the generated report displayed to the user | `August 2026 Audit Trail Report` |
| 641 | Execution Mode | `execution_type` | `varchar(20)` | N | N | - | Y | `ON_DEMAND` | N | Y | N | Y | Execution mode: ON_DEMAND for user requests; SCHEDULED for scheduled batch execution | `ON_DEMAND` |
| 642 | Reporting Period Start | `period_from` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Start date/time for querying the report's source data. NULL allowed for reports without a reporting period. | `2026-08-01 00:00:00+00` |
| 643 | Reporting Period End | `period_to` | `timestamptz` | N | N | - | N | - | N | N | N | Y | End date/time for querying the report's source data. Cannot be earlier than period_from. | `2026-08-31 23:59:59+00` |
| 644 | Query Criteria | `report_parameters` | `jsonb` | N | N | - | N | - | N | N | N | Y | Stores report generation criteria, such as organization, project, user, action type, and status, as JSON | `{"action_types":["CREATE","UPDATE"]}` |
| 645 | Output Format | `output_format` | `varchar(20)` | N | N | - | Y | `PDF` | N | Y | N | Y | Output file format: PDF, XLSX, CSV | `PDF` |
| 646 | Generation Status | `generation_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Processing status: PENDING, PROCESSING, COMPLETED, RETRY_WAIT, FAILED, CANCELLED | `PENDING` |
| 647 | Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Requester ID for user requests. NULL allowed for generation by the system or scheduled batch jobs. | `UUID` |
| 648 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when the report generation request was received or the batch job was registered | `2026-09-02 15:00:00+00` |
| 649 | Execution Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the report generation worker actually started processing | `2026-09-02 15:00:05+00` |
| 650 | Execution Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the report generation job ended in success or final failure | `2026-09-02 15:01:30+00` |
| 651 | Result File ID | `result_file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | Y | Y | N | Y | ID of the generated report file. Required in COMPLETED status; NULL allowed before completion. | `UUID` |
| 652 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of retries after the initial execution failed. Must be 0 or greater. | `0` |
| 653 | Maximum Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Maximum number of automatic retries allowed. Must be 0 or greater. | `3` |
| 654 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next scheduled execution time for a job in RETRY_WAIT status | `2026-09-02 15:10:00+00` |
| 655 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | System error code categorizing the cause of failure | `REPORT_FILE_CREATE_FAILED` |
| 656 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Details of report generation failure. Sensitive information such as passwords and tokens is not stored. | `An error occurred while saving the result file` |
| 657 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Report generation job record creation timestamp (UTC) | `2026-09-02 15:00:00+00` |
| 658 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when created by a user. NULL allowed when created by the system or a batch job. | `UUID` |
| 659 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Report generation job last modification timestamp (UTC) | `2026-09-02 15:01:30+00` |
| 660 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when modified by a user. NULL allowed for system/batch processing. | `UUID` |
| 661 | Report Schedule ID | `report_schedule_id` | `uuid` | N | Y | `report_schedule.report_schedule_id` | N | - | N | Y | N | Y | Stores the original report schedule ID when generated by a scheduled execution. NULL allowed for user-requested executions. | `UUID` |

[↑ Back to Top](#top)

---

<a id="table-report_schedule"></a>
### 50. Report Schedule (`report_schedule`)

| Item | Definition |
| --- | --- |
| Description | Manages execution frequencies, reporting periods, output formats, next execution times, and active statuses for audit and operational reports |
| Primary Key | `report_schedule_id` |
| Key References (FK) | `project_id, last_report_generation_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain execution schedules and change history for the required audit period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 749 | Report Schedule ID | `report_schedule_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique scheduled report execution schedule identifier | `UUID` |
| 750 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Linked for project-level report schedules. NULL allowed for organization-level or system-level schedules. | `UUID` |
| 751 | Schedule Name | `schedule_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Name of the scheduled report schedule displayed to the user | `Monthly Audit Trail Report` |
| 752 | Report Type | `report_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Report type: AUDIT_TRAIL, PROJECT_STATUS, WORKFLOW_STATUS, DEVIATION_STATUS, TRACEABILITY, SYSTEM_OPERATION | `AUDIT_TRAIL` |
| 753 | Schedule Type | `schedule_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | Schedule type: DAILY, WEEKLY, MONTHLY, CRON | `MONTHLY` |
| 754 | Schedule Expression | `schedule_expression` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Scheduling configuration, such as an execution date, day of the week, time, or Cron expression | `0 0 1 * *` |
| 755 | Reporting Period Type | `period_type` | `varchar(30)` | N | N | - | Y | `PREVIOUS_MONTH` | N | N | N | Y | Basis for calculating the source data reporting period: PREVIOUS_DAY, PREVIOUS_WEEK, PREVIOUS_MONTH, CUSTOM | `PREVIOUS_MONTH` |
| 756 | Query Criteria | `report_parameters` | `jsonb` | N | N | - | N | - | N | N | N | Y | Query criteria for scheduled reports, such as organization, project, user, action type, and status | `{"action_types":["CREATE","UPDATE"]}` |
| 757 | Output Format | `output_format` | `varchar(20)` | N | N | - | Y | `PDF` | N | Y | N | Y | Output file format: PDF, XLSX, CSV | `PDF` |
| 758 | Next Run At | `next_run_at` | `timestamptz` | N | N | - | Y | - | N | Y | N | Y | Next scheduled execution time for this schedule | `2026-10-01 00:00:00+00` |
| 759 | Last Run At | `last_run_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when this schedule was last executed | `2026-09-01 00:00:00+00` |
| 760 | Last Report Generation Job ID | `last_report_generation_id` | `uuid` | N | Y | `report_generation.report_generation_id` | N | - | N | Y | N | Y | Most recent report generation job created by this schedule | `UUID` |
| 761 | Is Active | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | Whether the scheduled report schedule is active | `True` |
| 762 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Report schedule creation timestamp (UTC) | `2026-09-02 18:00:00+00` |
| 763 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who registered the report schedule | `UUID` |
| 764 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Report schedule last modification timestamp (UTC) | `2026-09-02 18:00:00+00` |
| 765 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | User who last modified the report schedule | `UUID` |
| 766 | Deleted At | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Report schedule soft deletion timestamp | - |

[↑ Back to Top](#top)

---

## AI

<a id="table-ai_generation_job"></a>
### 51. AI Generation Job (`ai_generation_job`)

| Item | Definition |
| --- | --- |
| Description | Manages AI generation requests, models, input parameters, execution statuses, failures, and retries |
| Primary Key | `ai_job_id` |
| Key References (FK) | `project_id, requested_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 662 | AI Job ID | `ai_job_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI generation job identifier | `UUID` |
| 663 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Project link | `UUID` |
| 664 | Action Type | `job_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | ITEM_GENERATION, DOCUMENT_GENERATION | `ITEM_GENERATION` |
| 665 | Target Entity Type | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM, OQ_REPORT, PQ_REPORT, VSR_REPORT | `REQUIREMENT` |
| 666 | Target Entity ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Identifier of the generation target | `UUID` |
| 667 | AI Model Name | `model_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | GPT, Claude, Gemini | `GPT-5` |
| 668 | Input Parameters | `input_parameters` | `jsonb` | N | N | - | Y | - | N | N | N | Y | Stores generation criteria as JSON | `JSON` |
| 669 | Generation Status | `generation_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | PENDING, PROCESSING, COMPLETED, FAILED, RETRY_WAIT, CANCELLED | `COMPLETED` |
| 670 | Requested By ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User requesting generation | `UUID` |
| 671 | Requested At | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | Time when the request was received | `2026-09-02T00:00:00` |
| 672 | Execution Started At | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI processing start time | `2026-09-02T00:00:00` |
| 673 | Execution Completed At | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI processing completion time | `2026-09-02T00:00:00` |
| 674 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of re-executions | `0` |
| 675 | Maximum Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Retry limit | `3` |
| 676 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Scheduled retry time | `2026-09-02T15:10:00` |
| 677 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | Failure cause code | `LLM_TIMEOUT` |
| 678 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Detailed failure message | `Model response timed out` |
| 679 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation timestamp | `2026-09-02T00:00:00` |
| 680 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who created the record | `UUID` |
| 681 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Modification timestamp | `2026-09-02T00:00:00` |
| 682 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who modified the record | `UUID` |

[↑ Back to Top](#top)

---

<a id="table-ai_generation_result"></a>
### 52. AI Generation Result (`ai_generation_result`)

| Item | Definition |
| --- | --- |
| Description | Manages AI-generated result sets and adoption statuses |
| Primary Key | `ai_result_id` |
| Key References (FK) | `ai_job_id, selected_by, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 683 | AI Result ID | `ai_result_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI result identifier | `UUID` |
| 684 | AI Job ID | `ai_job_id` | `uuid` | N | Y | `ai_generation_job.ai_job_id` | Y | - | N | Y | N | Y | Parent AI job | `UUID` |
| 685 | Result Title | `result_title` | `varchar(300)` | N | N | - | N | - | N | N | N | Y | Result Title | `OQ Test Draft` |
| 686 | Is Selected | `is_selected` | `boolean` | N | N | - | Y | `N` | N | Y | N | Y | Whether adopted by the user | `True` |
| 687 | Is Applied | `is_applied` | `boolean` | N | N | - | Y | `N` | N | Y | N | Y | Whether applied to the actual deliverable | `True` |
| 688 | Selected By ID | `selected_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who adopted the result | `UUID` |
| 689 | Selected At | `selected_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the result was adopted | `2026-09-02T00:00:00` |
| 690 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation timestamp | `2026-09-02T00:00:00` |
| 691 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who created the record | `UUID` |
| 692 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Modification timestamp | `2026-09-02T00:00:00` |
| 693 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who modified the record | `UUID` |

[↑ Back to Top](#top)

---

<a id="table-ai_result_item"></a>
### 53. AI Generation Result Item (`ai_result_item`)

| Item | Definition |
| --- | --- |
| Description | Manages details of AI-generated URS/FRA/IQ/OQ/PQ items and document sections |
| Primary Key | `ai_result_item_id` |
| Key References (FK) | `ai_result_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain for the period during which records must remain available for audit |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 694 | AI Result Item ID | `ai_result_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Result item identifier | `UUID` |
| 695 | AI Result ID | `ai_result_id` | `uuid` | N | Y | `ai_generation_result.ai_result_id` | Y | - | N | Y | N | Y | Reference to the parent result | `UUID` |
| 696 | Item Order | `item_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | Result display order | `1` |
| 697 | Item Type | `item_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT, FRA_SCENARIO, IQ_TEST, OQ_TEST, PQ_TEST, DOCUMENT_SECTION | `REQUIREMENT` |
| 698 | Title | `title` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | Title of the generated item | `Electronic Signature Record` |
| 699 | Body Content | `content` | `text` | N | N | - | Y | - | N | N | N | Y | Body of the generated result | `Content` |
| 700 | Application Target Type | `target_entity_type` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | REQUIREMENT, FRA_ITEM, IQ_ITEM, etc. | `REQUIREMENT` |
| 701 | Application Target ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | Primary key of the actual storage target | `UUID` |
| 702 | Is Adopted | `is_selected` | `boolean` | N | N | - | Y | `N` | N | Y | N | Y | Whether adopted by the user | `True` |
| 703 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Creation timestamp | `2026-09-02T00:00:00` |
| 704 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who created the record | `UUID` |
| 705 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Modification timestamp | `2026-09-02T00:00:00` |
| 706 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | User who modified the record | `UUID` |

[↑ Back to Top](#top)

---

## Notification

<a id="table-notification_delivery"></a>
### 54. Notification Delivery (`notification_delivery`)

| Item | Definition |
| --- | --- |
| Description | Manages recipients, delivery statuses, failures, and retry history for review/approval requests, processing delay notices, and system operation notifications |
| Primary Key | `notification_delivery_id` |
| Key References (FK) | `project_id, workflow_instance_id, workflow_step_id, recipient_id, created_by, updated_by` |
| GxP Criticality | High |
| Audited | Y |
| Retention/Deletion Policy | Retain notification content and execution history for the required audit period |

#### Column Definitions

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `Sensitive`: Personal/Sensitive Information

| No | Logical Column Name | Physical Column Name | Type | PK | FK | Reference | NN | Default | UQ | IDX | Sensitive | Audit | Description/Business Rules | Example Value |
| ---: | --- | --- | --- | :---: | :---: | --- | :---: | --- | :---: | :---: | :---: | :---: | --- | --- |
| 707 | Notification Delivery ID | `notification_delivery_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Unique notification delivery job identifier | `UUID` |
| 708 | Project ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | Linked for project-related notifications. NULL allowed for system-wide notifications. | `UUID` |
| 709 | Workflow Instance ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | Linked for notifications related to a review/approval workflow | `UUID` |
| 710 | Workflow Step ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | N | - | N | Y | N | Y | Linked for notifications related to a review/approval step | `UUID` |
| 711 | Notification Type | `notification_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | Business type of the notification: APPROVAL_REQUEST, REVIEW_REQUEST, DUE_REMINDER, OVERDUE, REJECTION, COMPLETION, SYSTEM | `OVERDUE` |
| 712 | Delivery Channel | `delivery_channel` | `varchar(20)` | N | N | - | Y | `EMAIL` | N | Y | N | Y | Delivery channel: EMAIL, MESSENGER, PUSH, IN_APP | `EMAIL` |
| 713 | Recipient ID | `recipient_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | Y | Y | ID of the user receiving the notification | `UUID` |
| 714 | Notification Title | `notification_title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | Title of the notification sent to the user | `Review Deadline Overdue Notice` |
| 715 | Notification Content | `notification_content` | `text` | N | N | - | Y | - | N | N | N | Y | Body of the notification sent to the user. Sensitive information such as passwords and tokens must not be stored. | `The FDS review deadline has passed.` |
| 716 | Delivery Status | `delivery_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | Delivery status: PENDING, PROCESSING, SENT, RETRY_WAIT, FAILED, CANCELLED | `PENDING` |
| 717 | Scheduled Delivery At | `scheduled_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Scheduled notification delivery time. NULL allowed for immediate delivery. | `2026-09-02 18:00:00+00` |
| 718 | Sent At | `sent_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | Time when the notification was successfully handed off to the external delivery channel | `2026-09-02 18:00:05+00` |
| 719 | Retry Count | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | Number of retries after the initial delivery failed. Must be 0 or greater. | `0` |
| 720 | Maximum Retry Count | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | Maximum number of automatic delivery retries allowed. Must be 0 or greater. | `3` |
| 721 | Next Retry At | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | Next scheduled delivery time for a notification in RETRY_WAIT status | `2026-09-02 18:10:00+00` |
| 722 | Error Code | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | System error code categorizing the cause of notification delivery failure | `EMAIL_SEND_TIMEOUT` |
| 723 | Error Message | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | Details of notification delivery failure. Sensitive information such as authentication credentials must not be stored. | `Mail server response timed out` |
| 724 | Created At | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Notification delivery job creation timestamp (UTC) | `2026-09-02 18:00:00+00` |
| 725 | Created By ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when created by a user. NULL allowed when created by the system or a batch job. | `UUID` |
| 726 | Updated At | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | Notification delivery job last modification timestamp (UTC) | `2026-09-02 18:00:05+00` |
| 727 | Updated By ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | Stores the user ID when modified by a user. NULL allowed for system/batch processing. | `UUID` |

[↑ Back to Top](#top)

---
