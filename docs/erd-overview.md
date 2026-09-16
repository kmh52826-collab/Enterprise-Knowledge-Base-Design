# Validation Management Platform Overall Data Structure

This diagram groups all **54 tables** into **16 business areas**. The boxes display the actual table names.

The business area numbers identify areas within this diagram and differ in some cases from the section numbers in the ERD specifications.

Solid lines in the diagram represent the organization of areas and business flows between them; dotted lines represent the reuse of standard items and the application of shared functions. **These lines do not represent individual table FKs or a fixed execution sequence.** Refer to the data dictionary for detailed columns and FKs.

## Overall Structure Diagram

```mermaid
flowchart TB
    subgraph BASE["Master Data and Project Setup"]
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

    subgraph COMMON["Shared Functions Across Business Processes"]
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
    LIBRARY -. "Reuse of standard items" .-> DOCUMENTS
    DOCUMENTS -. "Application of shared functions" .-> COMMON

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

| Structure | Meaning of the Connection |
|---|---|
| `organization` → `system_asset` → `validation_project` | Projects are set up for systems and equipment belonging to an organization. |
| `validation_project` → `project_member` | Links the users participating in a project to their roles. |
| `validation_project` → `project_activity` → `validation_activity` | Links activities assigned to a project to the activity master, using `is_selected` to indicate whether they are selected for execution. |
| `validation_activity` ↔ `activity_dependency` | Defines predecessor and successor activities and their activation conditions. |
| Document header → Detail items | Separates document-level information from individual item content, as with `fds_spec` and `fds_item`. |
| `traceability_link` → Requirement, design, risk, and test items | Manages traceability relationships between deliverables using entity types and IDs. |
| `workflow_instance` → `workflow_step` → `approval_action` | Manages document approval processes, individual steps, and actual action history. Action records with signatures reference `electronic_signature`. |
| `file_asset` → `evidence_link` → Business items | Links files as evidence for documents or test items. Links to business items are managed using entity types and IDs. |
| `ai_generation_job` → `ai_generation_result` → `ai_result_item` | Distinguishes generation requests, generated results, and the detailed items within those results. |

`library_item` is an independent master with no direct FK connections to other tables. It provides standard items for authoring URS, IQ, and OQ.

Not all tables in the shared functions area are directly linked to a project. Target business data for Workflow, electronic signatures, Audit Trail, and similar functions is linked by storing the target type and ID together. `backup_execution` is an independent operational execution history.
