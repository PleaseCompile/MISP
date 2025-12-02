# MISP Core ER Diagram (Mermaid)

This document provides a detailed entity-relationship (ER) diagram for key parts of the MISP data model using Mermaid syntax. You can paste the code block into [mermaid.live](https://mermaid.live/edit) to view the interactive diagram.

```mermaid
erDiagram
    ORGANISATIONS {
        bigint id PK
        string name
        string uuid
        string description
        string nationality
    }
    ROLES {
        bigint id PK
        string name
        bool perm_admin
        bool perm_site_admin
        bool perm_sharing
    }
    USERS {
        bigint id PK
        string email
        string password_hash
        string authkey
        bool disabled
        bigint organisation_id FK
        bigint role_id FK
    }
    EVENTS {
        bigint id PK
        string info
        string uuid
        string distribution
        datetime date
        bigint org_id FK
        bigint creator_user_id FK
        bigint sharing_group_id FK
    }
    EVENT_REPORTS {
        bigint id PK
        string uuid
        text content
        bigint event_id FK
        bigint user_id FK
    }
    ATTRIBUTES {
        bigint id PK
        string uuid
        string type
        string category
        text value
        bool to_ids
        bigint event_id FK
        bigint object_id FK
    }
    SIGHTINGS {
        bigint id PK
        string uuid
        datetime sighting_timestamp
        bigint attribute_id FK
        bigint event_id FK
        bigint org_id FK
        bigint user_id FK
    }
    MISP_OBJECTS {
        bigint id PK
        string uuid
        string name
        bigint event_id FK
        bigint org_id FK
    }
    OBJECT_REFERENCES {
        bigint id PK
        string uuid
        string relationship_type
        bigint object_id FK
        bigint referenced_attribute_id FK
        bigint referenced_object_id FK
    }
    TAGS {
        bigint id PK
        string name
        string colour
        bool exportable
    }
    EVENT_TAGS {
        bigint id PK
        bigint event_id FK
        bigint tag_id FK
    }
    ATTRIBUTE_TAGS {
        bigint id PK
        bigint attribute_id FK
        bigint tag_id FK
    }
    OBJECT_TAGS {
        bigint id PK
        bigint object_id FK
        bigint tag_id FK
    }
    EVENT_REPORT_TAGS {
        bigint id PK
        bigint event_report_id FK
        bigint tag_id FK
    }
    GALAXIES {
        bigint id PK
        string uuid
        string name
    }
    GALAXY_CLUSTERS {
        bigint id PK
        string uuid
        string type
        string description
        bigint galaxy_id FK
    }
    EVENT_GALAXY_CLUSTERS {
        bigint id PK
        bigint event_id FK
        bigint galaxy_cluster_id FK
    }
    SHARING_GROUPS {
        bigint id PK
        string name
        string uuid
        string organisation_type
    }
    SHARING_GROUP_ORG {
        bigint id PK
        bigint sharing_group_id FK
        bigint org_id FK
        bool extend
    }

    ORGANISATIONS ||--o{ USERS : "employs"
    ROLES ||--o{ USERS : "assigns"
    ORGANISATIONS ||--o{ EVENTS : "owns"
    USERS ||--o{ EVENTS : "creates"
    EVENTS ||--o{ ATTRIBUTES : "contains"
    EVENTS ||--o{ EVENT_REPORTS : "documents"
    USERS ||--o{ EVENT_REPORTS : "writes"
    ATTRIBUTES ||--o{ SIGHTINGS : "observed_in"
    EVENTS ||--o{ MISP_OBJECTS : "groups"
    MISP_OBJECTS ||--o{ ATTRIBUTES : "composed_of"
    MISP_OBJECTS ||--o{ OBJECT_REFERENCES : "links"
    ATTRIBUTES ||--o{ OBJECT_REFERENCES : "referenced_by"

    TAGS ||--o{ EVENT_TAGS : "labels"
    TAGS ||--o{ ATTRIBUTE_TAGS : "labels"
    TAGS ||--o{ OBJECT_TAGS : "labels"
    TAGS ||--o{ EVENT_REPORT_TAGS : "labels"
    EVENTS ||--o{ EVENT_TAGS : "tagged_with"
    ATTRIBUTES ||--o{ ATTRIBUTE_TAGS : "tagged_with"
    MISP_OBJECTS ||--o{ OBJECT_TAGS : "tagged_with"
    EVENT_REPORTS ||--o{ EVENT_REPORT_TAGS : "tagged_with"

    GALAXIES ||--o{ GALAXY_CLUSTERS : "contains"
    GALAXY_CLUSTERS ||--o{ EVENT_GALAXY_CLUSTERS : "linked_to"
    EVENTS ||--o{ EVENT_GALAXY_CLUSTERS : "classified_by"

    SHARING_GROUPS ||--o{ EVENTS : "scope"
    SHARING_GROUPS ||--o{ SHARING_GROUP_ORG : "members"
    ORGANISATIONS ||--o{ SHARING_GROUP_ORG : "participates"
```

**How to use**
1. Copy the Mermaid block above.
2. Visit [mermaid.live/edit](https://mermaid.live/edit) and paste the block into the editor.
3. The diagram is interactive: zoom, pan, and export as PNG/SVG for documentation.

The diagram focuses on common, high-traffic tables and their relationships to help understand how events, attributes, tags, and sharing controls connect across MISP.
