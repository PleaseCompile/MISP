# MISP Core ER Diagram (Mermaid)

เอกสารนี้รวมแผนภาพ ER (Mermaid) ของตารางหลักใน MISP พร้อมฟิลด์สำคัญเพื่อให้มองเห็นโครงสร้างและบริบทการแชร์ข้อมูลได้ละเอียดขึ้น นำโค้ดด้านล่างไปวางที่ [mermaid.live](https://mermaid.live/edit) เพื่อเรนเดอร์แบบโต้ตอบ

```mermaid
erDiagram
    ORGANISATIONS {
        bigint id PK
        string name
        string uuid
        string description
        string nationality
        string sector
        datetime date_created
    }
    ROLES {
        bigint id PK
        string name
        bool perm_admin
        bool perm_site_admin
        bool perm_sharing
        bool perm_sync
        bool perm_audit
    }
    USERS {
        bigint id PK
        string email
        string password_hash
        string authkey
        bool disabled
        string gpgkey
        bigint organisation_id FK
        bigint role_id FK
        datetime date_created
    }
    EVENTS {
        bigint id PK
        string info
        string uuid
        string distribution
        string analysis
        string threat_level_id
        bool published
        datetime date
        datetime timestamp
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
        datetime timestamp
    }
    ATTRIBUTES {
        bigint id PK
        string uuid
        string type
        string category
        text value
        text comment
        bool to_ids
        bool deleted
        datetime timestamp
        bigint event_id FK
        bigint object_id FK
    }
    SIGHTINGS {
        bigint id PK
        string uuid
        string type
        datetime sighting_timestamp
        text source
        bigint attribute_id FK
        bigint event_id FK
        bigint org_id FK
        bigint user_id FK
    }
    MISP_OBJECTS {
        bigint id PK
        string uuid
        string name
        text comment
        datetime timestamp
        bigint event_id FK
        bigint org_id FK
        string template_uuid
    }
    OBJECT_REFERENCES {
        bigint id PK
        string uuid
        string relationship_type
        text comment
        datetime timestamp
        bigint object_id FK
        bigint referenced_attribute_id FK
        bigint referenced_object_id FK
    }
    TAGS {
        bigint id PK
        string name
        string colour
        bool exportable
        bool hide_tag
        bigint user_id FK
    }
    EVENT_TAGS {
        bigint id PK
        bigint event_id FK
        bigint tag_id FK
        datetime timestamp
    }
    ATTRIBUTE_TAGS {
        bigint id PK
        bigint attribute_id FK
        bigint tag_id FK
        datetime timestamp
    }
    OBJECT_TAGS {
        bigint id PK
        bigint object_id FK
        bigint tag_id FK
        datetime timestamp
    }
    EVENT_REPORT_TAGS {
        bigint id PK
        bigint event_report_id FK
        bigint tag_id FK
        datetime timestamp
    }
    GALAXIES {
        bigint id PK
        string uuid
        string name
        string type
        datetime timestamp
    }
    GALAXY_CLUSTERS {
        bigint id PK
        string uuid
        string type
        string description
        datetime timestamp
        bigint galaxy_id FK
    }
    EVENT_GALAXY_CLUSTERS {
        bigint id PK
        bigint event_id FK
        bigint galaxy_cluster_id FK
        datetime timestamp
    }
    SHARING_GROUPS {
        bigint id PK
        string name
        string uuid
        string organisation_type
        string releasability
        datetime created
    }
    SHARING_GROUP_ORG {
        bigint id PK
        bigint sharing_group_id FK
        bigint org_id FK
        bool extend
        datetime created
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

## วิธีอ่านแผนภาพอย่างละเอียด
- **สี/กลุ่มใน mermaid.live:** สามารถใช้ Layout หรือจัดกลุ่ม node ด้วยตนเองเพื่อแยกโดเมน เช่น Metadata (ORG/USER/ROLE), Events, Objects, Context (Tag/Galaxy), และ Sharing
- **ฟิลด์ที่เชื่อมโยงบ่อย:** `uuid`, `org_id`, `event_id`, `object_id`, `sharing_group_id`, `timestamp` ช่วยไล่สายสัมพันธ์ของข้อมูลและการซิงก์
- **เส้นทางการสืบค้นตัวอย่าง:** เริ่มจาก `EVENTS` → `ATTRIBUTES` → `MISP_OBJECTS`/`OBJECT_REFERENCES` เพื่อดูรายละเอียดเชิงโครงสร้าง จากนั้นตามไปที่ `TAGS` หรือ `GALAXY_CLUSTERS` เพื่อวิเคราะห์บริบท และจบที่ `SHARING_GROUPS` เพื่อตรวจสอบสิทธิ์การเข้าถึง
- **ติดตามการเปลี่ยนแปลง:** ฟิลด์ `timestamp`/`created` ในหลายตารางช่วยระบุลำดับเวลา เหมาะสำหรับกระบวนการซิงก์หรือออดิตกิจกรรม

คัดลอกบล็อก Mermaid ไปเรนเดอร์บน mermaid.live เพื่อซูม, pan, และ export ภาพสำหรับใช้ในเอกสารหรือสไลด์การอบรมได้ทันที
