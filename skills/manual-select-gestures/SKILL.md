---
name: manual-select-gestures
description: >-
  Manual image editor region tools in ThaiSnapAI — separate Object Select
  (auto edge) from Manual outline. Use when changing clip_image_picker,
  keep/erase outlines, st_canvas drawing modes, or object-select gestures.
---

# Manual select gestures

แก้แมนนวลใน `ThaiSnapAI/frontend/adapters/web/clip/clip_image_picker.py`

## Two region modes

| ปุ่ม | Key | ติดขอบ | พื้นที่เก็บ/ลบ |
|------|-----|--------|----------------|
| ติดขอบอัตโนมัติ | `object` | ใช่ — เรียก `select_object` API | ตาม polygon ที่ snap แล้ว |
| วงเอง | `manual` | ไม่ — ครอปตามเครื่องหมาย | rect / จุด≥3 / lasso ตามที่วาด |

`_MANUAL_TOOLS = ("object", "manual", "brush", "transform")`

Gestures ใช้ร่วมกันทั้ง `object` และ `manual`:

| ปุ่ม | Key | Canvas |
|------|-----|--------|
| กรอบ | `rect` | `drawing_mode="rect"` |
| คลิก | `click` | `drawing_mode="point"` |
| เส้น | `lasso` | `drawing_mode="freedraw"` |

`_MANUAL_AUTO_GESTURES = ("rect", "click", "lasso")`

## Object (`object`)

1. gesture → `_handle_auto_select` → `select_object` → `_queue_manual_autosave` (บันทึกทันที)
2. คลิก = คลิกเดียวชี้วัตถุ (ไม่ใช่ polygon หลายจุด)
3. กรอบ/เส้น = hint ให้ engine ติดขอบ — ไม่ใช่ crop ตามสี่เหลี่ยม/lasso ตรงๆ
4. เรียก API ผ่าน HTTP เท่านั้น (`frontend/api_client`)

## Manual (`manual`)

1. กรอบ = สี่เหลี่ยมจริง — อย่าเรียก `select_object`
2. คลิก = ≥3 จุด เส้นเชื่อมเป็น polygon
3. เส้น = drag freehand — อย่า snap
4. เก็บ/ลบอ่านจาก `shapes` ใน `_shapes_from_objects`

## Canvas / anti-flicker

- โหมด `manual` / `brush`: เมื่อวงครบ → บันทึกทันที + ประวัติ แล้ว remount บนผลลัพธ์ (`_handle_manual_finalize` → `_queue_manual_autosave`) — ห้าม bake preview วน และห้ามรอปุ่มบันทึก
- โหมด `object`: snap แล้วบันทึกทันที + ประวัติ — อย่าค้าง polygon รอ confirm
- remount (`CLIP_MANUAL_CANVAS_REV`) เฉพาะ: clear, tool switch, gesture switch, object snap, manual finalize, หลังบันทึก
- ห้าม `st.image` พรีวิวแยกใต้แคนวาส
- native Streamlit เท่านั้น — ห้าม inject CSS overlay

## Must (ทั้งสองโหมด)

1. วงครบแล้ว**บันทึกทันที** — อยู่รูปผลลัพธ์ วงต่อได้จนกว่าจะพอใจ ไม่มีปุ่ม «บันทึกแล้วแก้ต่อ»
2. ประวัติด้านข้างขึ้นทันทีหลังแก้ — คืนค่าแล้วแก้ต่อ
3. ตัวแก้เป็น**หน้าเต็ม** (`render_manual_editor_page`) — `st.dialog` เหลือไว้ยืนยันคืนค่า / แจ้งเตือน ห้ามหุ้มแคนวาสด้วย dialog
4. native Streamlit เท่านั้น — ห้าม inject CSS overlay
5. ห้าม `st.image` พรีวิวแยกใต้แคนวาส

## Forbid

- เรียก `select_object` จากโหมด `manual`
- ครอปตามกรอบ/lasso ตรงๆ ในโหมด `object` (ต้อง snap ก่อน)
- bake ผล + rerun backdrop ทุก stroke
- ลบวงที่วาดเสร็จเมื่อสลับ gesture (remount เพื่อเปลี่ยน `drawing_mode` ได้ แต่คง objects)
- รอปุ่ม «บันทึกแล้วแก้ต่อ» ก่อนเขียนไฟล์ / ประวัติ

## Files

| ไฟล์ | หน้าที่ |
|------|--------|
| `clip_image_picker.py` | หน้าแก้รูป + แคนวาส + ประวัติ · popup ยืนยันคืนค่า |
| `shared/image_ops/polygon_crop.py` | ครอป/ลบตาม `shapes` |
| `shared/image_ops/object_select.py` | engine ติดขอบ — โหมด `object` เท่านั้น |
| `backend/api/routes/files.py` | `POST /files/select-object` |
