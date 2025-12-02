# คู่มือแก้ไข Conflict สำหรับ PR เอกสาร (Thai)

คู่มือนี้ช่วยให้คุณเคลียร์ conflict ที่ GitHub แจ้งใน PR เกี่ยวกับไฟล์เอกสาร (`ER_diagram.md`, `README.md`, `data_flow.html`) โดยเน้นขั้นตอนสั้น ๆ สำหรับกรณีที่ฐาน (main) มีการอัปเดตแล้วคุณต้องปรับสาขางานให้ตามทัน

## 1) เตรียมอัปเดตฐาน
```bash
# ดึงฐานล่าสุด (ปรับชื่อรีโมต/สาขาให้ตรงโปรเจ็กต์จริง)
git fetch origin
# สลับไปที่ main และอัปเดต
git checkout main
git pull --ff-only origin main
# กลับมาที่สาขางาน (ตัวอย่าง: work)
git checkout work
```

## 2) รวมการเปลี่ยนจาก main เข้าสาขางาน
เลือกวิธีที่ถนัด: `rebase` (ประวัติเรียบ) หรือ `merge` (ปลอดภัยสำหรับมือใหม่)

### ตัวอย่าง rebase (แนะนำ)
```bash
git rebase origin/main
```
- ถ้ามี conflict Git จะหยุดที่คอมมิตที่ชนกัน ให้ไปแก้ไฟล์ แล้ว `git add` และ `git rebase --continue`

### ตัวอย่าง merge (ง่าย)
```bash
git merge origin/main
```
- ถ้ามี conflict ให้แก้ไฟล์แล้ว `git add` และ `git commit`

## 3) จุดที่มัก conflict และวิธีแก้เร็ว
- `ER_diagram.md` / `README.md`: เกิดเมื่อทั้งสองฝั่งแก้ย่อหน้าเดียวกัน ให้ตัด `<<<<<<<`, `=======`, `>>>>>>>` ออก แล้วเรียงข้อความที่ต้องการเก็บ
- `data_flow.html`: ระวังบล็อก `<mermaid>` หรือ `<pre><code>` ที่เหมือนกัน ให้คงเวอร์ชันที่มีข้อมูลใหม่ที่สุด แล้วตรวจปิดแท็ก HTML ให้ครบ

หลังแก้ไข ลองเปิดดูคร่าว ๆ:
```bash
git diff --check   # ตรวจว่าไม่มีบรรทัดมี conflict markers หรือช่องว่างท้ายบรรทัด
```

## 4) ทดสอบเรนเดอร์คร่าว ๆ (ทางเลือก)
- `ER_diagram.md`: คัดลอกโค้ด Mermaid ไปที่ https://mermaid.live
- `data_flow.html`: เปิดไฟล์ในเบราว์เซอร์เพื่อดูว่ากล่อง/ลูกศรยังแสดงถูกต้อง

## 5) สรุปและส่งขึ้น PR
```bash
git status           # ตรวจว่าไฟล์ที่แก้ถูก add แล้ว
git commit -m "Fix merge conflicts for docs"
git push origin work # หรือชื่อสาขาของคุณ
```

> หาก rebase ผิดพลาด สามารถยกเลิกด้วย `git rebase --abort` แล้วลองใหม่หรือเปลี่ยนมาใช้ `git merge` แทน
