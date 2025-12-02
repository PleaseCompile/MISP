# ภาพรวม ER Diagram ของ MISP (Mermaid)
เอกสารนี้สรุปแผนภาพ Entity-Relationship (ER) สำหรับโครงสร้างข้อมูลหลักของ MISP ที่อยู่ในไฟล์ `ER_diagram.md` โดยจัดเป็นหมวดหมู่ อธิบายว่าตารางแต่ละตัวมีไว้ทำอะไร เชื่อมโยงกันอย่างไร และมีตัวอย่างลำดับการไหลของข้อมูลเพื่อช่วยอ่านแผนภาพได้ชัดเจนขึ้น

## วิธีใช้แผนภาพ
1. เปิดไฟล์ [`ER_diagram.md`](./ER_diagram.md) แล้วคัดลอกโค้ด Mermaid ทั้งบล็อก
2. วางลงใน [mermaid.live](https://mermaid.live/edit) แล้วกด **Render** เพื่อดูแผนภาพแบบโต้ตอบ
3. ใช้ฟังก์ชัน **Layout** เพื่อจัดผัง หรือส่งออกเป็น **PNG/SVG** จากเมนู **Export**

## หมวดหมู่ข้อมูลหลัก (Key Domains)
- **แกนเหตุการณ์ (Events & Attributes):**
  - `EVENTS` เก็บเหตุการณ์หลัก มี `info`, `uuid`, ระดับการเปิดเผย `distribution`, วันที่ `date`, เจ้าของ `org_id`, ผู้สร้าง `creator_user_id`, และขอบเขตการแชร์ `sharing_group_id`.
  - `ATTRIBUTES` เก็บ Indicator รายการย่อยที่อยู่ในเหตุการณ์ เช่น IP, Domain, Hash โดยอ้างอิงกลับไปที่ `event_id` และวัตถุ `object_id` ถ้ามาจาก Object
  - `EVENT_REPORTS` เอกสารสรุปเหตุการณ์ที่เชื่อมกับ `event_id` และผู้เขียน `user_id`
  - `SIGHTINGS` บันทึกการพบเห็น Indicator เชื่อมกับ `attribute_id`, `event_id`, องค์กร (`org_id`) และผู้รายงาน (`user_id`)

- **โครงร่างวัตถุ (Objects & References):**
  - `MISP_OBJECTS` ใช้รวมคอลเลกชัน Attribute ที่มีโครงสร้างเดียวกัน เช่น ไฟล์, อีเมล โดยผูกกับ `event_id` และเจ้าของ `org_id`
  - `OBJECT_REFERENCES` เชื่อมวัตถุหรือแอตทริบิวต์เข้าด้วยกันด้วย `relationship_type` และตัวชี้ `referenced_attribute_id` หรือ `referenced_object_id`

- **การติดป้ายและบริบท (Tags & Galaxy):**
  - `TAGS` คลังแท็กหลัก (ชื่อ สี และสถานะ export ได้หรือไม่)
  - ตารางเชื่อมแท็ก เช่น `EVENT_TAGS`, `ATTRIBUTE_TAGS`, `OBJECT_TAGS`, `EVENT_REPORT_TAGS` ใช้บันทึกว่าข้อมูลใดถูกติดแท็กอะไร
  - `GALAXIES` และ `GALAXY_CLUSTERS` เป็นชุดข้อมูลบริบท (เช่น กลุ่มผู้โจมตี, TTP) ที่ลิงก์กับเหตุการณ์ผ่าน `EVENT_GALAXY_CLUSTERS`

- **การแชร์และการเป็นสมาชิก (Sharing):**
  - `SHARING_GROUPS` กำหนดขอบเขตการแชร์ข้ามองค์กร
  - `SHARING_GROUP_ORG` ระบุสมาชิกองค์กร (`org_id`) ที่อยู่ในกลุ่มแชร์แต่ละอัน
  - ฟิลด์ `sharing_group_id` ใน `EVENTS` บ่งบอกว่าเหตุการณ์นั้นใช้กลุ่มแชร์ใด

- **สิทธิ์และบทบาท (ACL & Ownership):**
  - `ORGANISATIONS` เก็บข้อมูลหน่วยงานที่เป็นเจ้าของหรือผู้ใช้ระบบ
  - `ROLES` และฟิลด์ permission (`perm_admin`, `perm_site_admin`, `perm_sharing`) อธิบายสิทธิ์ของแต่ละบทบาท
  - `USERS` อ้างอิง `organisation_id` และ `role_id` เพื่อผูกผู้ใช้กับองค์กรและบทบาทที่ได้รับ

## โครงสร้างความสัมพันธ์สำคัญ
- **จากเหตุการณ์สู่ Indicator:** `EVENTS` เชื่อมกับ `ATTRIBUTES` ผ่าน `event_id` และกับ `EVENT_REPORTS` เพื่อเก็บรายละเอียดข้อความ
- **จากวัตถุสู่ Attribute ย่อย:** `MISP_OBJECTS` มีชุด `ATTRIBUTES` ที่อธิบายรายละเอียด และเชื่อมโยงต่อกันผ่าน `OBJECT_REFERENCES`
- **การเพิ่มบริบทและการค้นหา:** `TAGS` และ `GALAXY_CLUSTERS` ผูกกับเหตุการณ์/แอตทริบิวต์เพื่อเพิ่มบริบท ช่วยให้ค้นหาและกรองข้อมูลได้เร็วขึ้น
- **สิทธิ์และขอบเขตการแชร์:** ฟิลด์ `distribution` และ `sharing_group_id` ใน `EVENTS` บ่งบอกระดับการเปิดเผย ขณะที่ `SHARING_GROUP_ORG` ระบุว่าองค์กรใดเข้าถึงได้

## ตัวอย่างลำดับการไหลของข้อมูล (Reading Flow)
1. เริ่มจาก `EVENTS` ดู metadata และขอบเขตการแชร์ (`distribution`, `sharing_group_id`)
2. สำรวจ `ATTRIBUTES` ที่เชื่อมผ่าน `event_id` และตรวจสอบว่า Attribute มาจาก `MISP_OBJECTS` หรือไม่
3. ตรวจสอบ `OBJECT_REFERENCES` เพื่อดูความสัมพันธ์ระหว่างวัตถุ/แอตทริบิวต์ (เช่น ไฟล์ ↔ อีเมลแนบ)
4. ดู `EVENT_REPORTS` เพื่ออ่านสรุปหรือบริบทเชิงข้อความของเหตุการณ์
5. ตรวจสอบแท็กและ Galaxy ที่ผูกกับเหตุการณ์หรือแอตทริบิวต์เพื่อเข้าใจมิติด้านผู้โจมตี/เทคนิค
6. หากต้องการดูขอบเขตการเข้าถึง ให้ตรวจสอบ `SHARING_GROUPS`, `SHARING_GROUP_ORG` และบทบาทผู้ใช้ (`USERS`, `ROLES`)

## เคล็ดลับการอ่านแผนภาพ
- ไล่เส้นเชื่อมจาก `EVENTS` ออกไปหาตารางรอบข้าง เพื่อเห็นภาพรวมก่อนลงลึกรายละเอียด
- สังเกตฟิลด์เชื่อมหลัก: `uuid`, `org_id`, `event_id`, `sharing_group_id` และ `distribution` มักเป็นเส้นทางหลักของข้อมูล
- ใน mermaid.live สามารถเลือก Node เพื่อ highlight ความสัมพันธ์ หรือปรับตำแหน่งเพื่อลดเส้นไขว้

หากต้องการแก้ไขหรือขยาย ER Diagram สามารถปรับโค้ดใน `ER_diagram.md` แล้วเรนเดอร์ใหม่บน mermaid.live เพื่อทดสอบความเชื่อมโยงทันที
