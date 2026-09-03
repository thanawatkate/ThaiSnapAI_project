---
name: no-image-flicker
description: >-
  After image/UI edits the photo must not flicker, blank, or reload.
  Always verify before finishing any change that reruns, remounts, or
  replaces a photo widget (canvas, st.image, work dialog, gallery).
---

# No image flicker

หลังแก้รูป/UI ภาพบนจอต้อง**อยู่ต่อเนื่อง** — ห้ามกะพริบ ว่าง ดำ หรือโหลดทั้งใบใหม่

ใช้ทุกครั้งที่แตะแคนวาส แกลเลอรี `st.image` `st.rerun` remount widget หรือไฟล์รูป

## Must

1. ผลลัพธ์แทนที่ภาพเดิมในที่ — ห้ามเคลียร์เป็นว่าง/ดำ/placeholder แล้วค่อยใส่รูปใหม่
2. แคชรูปที่ decode แล้ว (`_manual_src_image`) จนกว่า path/mtime เปลี่ยน
3. remount แคนวาส (`CLIP_MANUAL_CANVAS_REV` / เปลี่ยน `key` / `initial_drawing`) เฉพาะเคลียร์ สลับเครื่องมือ snap bake บันทึก — ห้ามทุก stroke
4. poll งานช้าด้วย `@st.fragment(run_every=…)` — ห้าม `st.rerun()` ทั้งหน้าเพื่อรีเฟรชรูป
5. ห้าม `st.image` พรีวิวแยกใต้แคนวาส

```python
# ✅ แคชข้าม rerun ของแคนวาส
img = _manual_src_image(path)

# ✅ poll โดยไม่กระพริบหน้าหลัก
@st.fragment(run_every=0.5)
def _poll():
    render_terminal_backdrop(...)

# ❌ เปิดไฟล์ใหม่ / remount ทุกครั้งที่วาด
Image.open(path)
st.session_state[CLIP_MANUAL_CANVAS_REV] += 1  # ใน stroke handler
st.rerun()  # ทั้งหน้าหลังทุก stroke
```

## Verify (บังคับก่อนจบงาน)

เปิดหน้าแก้จริง แล้วทำอย่างน้อยหนึ่งอย่าง: ตัด · ลบ · ครอบ · แปรง · ใช้ที่เลือก · ย้อน

ภาพต้องไม่กะพริบ/ว่างแม้ชั่วขณะ แถบเครื่องมือคงค่าเดิม งานยังไม่เสร็จถ้ากะพริบ

รายละเอียดแคนวาส: `skills/manual-select-gestures/SKILL.md` · กติกาทีม: `ThaiSnapAI/AGENTS.md` §4
