---
name: dialog-action-edges
description: >-
  Pin dialog/wizard action buttons to far left and far right. Use when
  editing st.dialog footers, wizard Back/Next, confirm/cancel rows, or
  dialog_action_row.
---

# Dialog action edges

ปุ่มท้าย dialog / แถบวิซาร์ด ต้อง**ชิดซ้ายสุดและขวาสุดเสมอ**

## Must

1. ใช้ `dialog_action_row()` จาก `frontend/adapters/web/components/dialog_actions.py`
2. **ซ้าย** = ยืนยัน / บันทึก / ลบ / ย้อนกลับ (`type="primary"` เมื่อเป็นงานหลัก)
3. **ขวา** = ยกเลิก / ปิด / ถัดไป
4. ปุ่มเดียวก็ต้องอยู่ในช่องซ้ายหรือขวา — ห้ามลอยกลางแถว
5. ปุ่มใช้ `width="content"` — ห้าม `width="stretch"` บนปุ่มท้าย dialog

```python
from frontend.adapters.web.components.dialog_actions import dialog_action_row

# ✅ คู่ — ยืนยันซ้ายสุด · ยกเลิกขวาสุด
with dialog_action_row() as (left, right):
    with left:
        st.button("ย้อนกลับ", type="primary", width="content")
    with right:
        st.button("ยกเลิก", width="content")

# ✅ ปุ่มเดียวขวา
with dialog_action_row() as (_left, right):
    with right:
        st.button("ปิด", width="content")

# ✅ ปุ่มเดียวซ้าย
with dialog_action_row() as (left, _right):
    with left:
        st.button("บันทึก", type="primary", width="content")
```

## Forbid

```python
# ❌ กองชิดซ้ายทั้งคู่
st.columns([1, 1, 4])

# ❌ ไม่มี stretch / ไม่เต็มความกว้าง
with st.container(horizontal=True):
    st.button("ย้อนกลับ")
    st.button("ยกเลิก")

# ❌ ยืดปุ่มเต็มแถว
st.button("ยกเลิก", width="stretch")
```

## Files

| ไฟล์ | หน้าที่ |
|------|--------|
| `frontend/adapters/web/components/dialog_actions.py` | `dialog_action_row` — ซ้าย content · stretch · ขวา content |
| `ThaiSnapAI/AGENTS.md` §4 | กฎ UI เดียวกัน |
