# Validation Management Platform Overall Data Structure

This diagram groups all **54 tables** into **16 business areas**. Each box lists the actual table names.

Solid lines represent organizational relationships and business flows between areas, while dotted lines represent the reuse of standard items and the application of common functions. **These lines do not represent individual table FKs or a fixed execution sequence.** For detailed columns and FKs, refer to the data dictionary.

## Overall Structure Diagram

```mermaid
flowchart TB
    subgraph BASE["Reference Data and Project Setup"]
        direction TB
        SECURITY["① Organizations, Users, and Permissions
organization
app_user
role
user_role
user_group
user_group_member
group_role"]
        PROJECT["② Systems, Projects, and Participants
system_asset
validation_project
project_member"]
        ACTIVITY["③ Activities and Dependencies
validation_activity
project_activity
activity_dependency"]
        SECURITY --> PROJECT --> ACTIVITY
    end

    subgraph DOCUMENTS["Project Assessments, Design, Testing, and Closure"]
        direction TB
        subgraph DESIGN["Scope Assessment and Design"]
            direction LR
            QIA["④ QIA and Vendor Audits
qia_assessment
qia_module_item
vendor_audit"]
            URSFDS["⑤ URS and FDS
requirement
fds_spec
fds_item
fds_interface"]
            DDSDQ["⑥ DDS and DQ
dds_spec
dds_item
dq_assessment
dq_item"]
            QIA --> URSFDS --> DDSDQ
        end
        subgraph VERIFICATION["Risk Assessment, Testing, and Traceability"]
            direction LR
            FRA["⑦ FRA Risk Assessment
fra_assessment
fra_item"]
            TEST["⑧ IQ, OQ, and PQ Testing
iq_assessment
iq_item
oq_assessment
oq_item
pq_assessment
pq_item"]
            TRACE["⑨ Traceability and RTM
traceability_link
rtm_assessment
rtm_item"]
            FRA --> TEST --> TRACE
        end
        CLOSE["⑩ VSR and Deviations
vsr_assessment
vsr_item
deviation"]
        DESIGN --> VERIFICATION --> CLOSE
    end

    LIBRARY["⑪ Standard Library · Independent Master
library_item"]

    subgraph COMMON["Common Functions Applied Across Business Processes"]
        direction TB
        subgraph CONTROL["Approval, Evidence, and Audit"]
            direction LR
            WORKFLOW["⑫ Workflow and Electronic Signatures
workflow_instance
workflow_step
approval_action
electronic_signature"]
            FILES["⑬ Files, Evidence, and Cleanup
file_asset
evidence_link
file_cleanup_execution"]
            AUDIT["⑯ Audit Trail
audit_trail"]
        end
        subgraph SUPPORT["Operations and Authoring Support"]
            direction LR
            OPS["⑭ Reports, Notifications, and Backups
report_generation
report_schedule
notification_delivery
backup_execution"]
            AI["⑮ AI Generation
ai_generation_job
ai_generation_result
ai_result_item"]
        end
        CONTROL ~~~ SUPPORT
    end

    BASE --> DOCUMENTS
    LIBRARY -. "Use of standard items" .-> DOCUMENTS
    DOCUMENTS -. "Application of common functions" .-> COMMON

    classDef base fill:#eaf2ff,stroke:#5278b5,color:#172b4d
    classDef document fill:#eaf6ef,stroke:#548c6b,color:#193c29
    classDef common fill:#f1edfa,stroke:#8569ad,color:#32224d
    classDef library fill:#fff5df,stroke:#b38a39,color:#574018
    class SECURITY,PROJECT,ACTIVITY base
    class QIA,URSFDS,DDSDQ,FRA,TEST,TRACE,CLOSE document
    class WORKFLOW,FILES,OPS,AI,AUDIT common
    class LIBRARY library
```

## Key Connections

| Structure | Connection Meaning |
|---|---|
| `organization` → `system_asset` → `validation_project` | Projects are set up for systems and equipment belonging to an organization. |
| `validation_project` → `project_member` | Connects the users participating in a project with their roles. |
| `validation_project` → `project_activity` → `validation_activity` | Links the activities to be performed in a project to the activity master. |
| `validation_activity` ↔ `activity_dependency` | Defines predecessor and successor activities and activation conditions. |
| Document header → Detail items | Manages document-level information separately from item-level content, as in `fds_spec` and `fds_item`. |
| `traceability_link` → Requirements, design, risk, and test items | Manages traceability relationships between deliverables using target types and IDs. |
| `workflow_instance` → `workflow_step` → `approval_action` | Manages document-specific approval processes, individual steps, and actual action history. Action records with a signature reference `electronic_signature`. |
| `file_asset` → `evidence_link` → Business items | Links files as evidence for documents or test items. Connections to business items are managed using target types and IDs. |
| `ai_generation_job` → `ai_generation_result` → `ai_result_item` | Separates generation requests, generation results, and detailed result items. |

`library_item` is an independent master with no direct FK connections to other tables. It provides standard items for preparing URS, IQ, and OQ.

Not all tables in the common functions area are directly linked to a project. Target business data for Workflow, electronic signatures, Audit Trail, and similar functions is linked by storing the target type and ID together. `backup_execution` is an independent operational execution history.
