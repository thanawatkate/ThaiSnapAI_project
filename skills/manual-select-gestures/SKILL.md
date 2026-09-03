---
name: manual-select-gestures
description: >-
  Manual image editor region tools in ThaiSnapAI — separate Object Select
  (auto edge) from Manual outline. Use when changing clip_image_picker,
  keep/erase outlines, st_canvas drawing modes, or object-select gestures.
---

# Manual select gestures

แก้แมนนวลใน `ThaiSnapAI/frontend/adapters/web/clip/clip_image_picker.py`

## Job strip (visible)

| ปุ่ม | Key | ทำอะไร |
|------|-----|--------|
| ตัด | `cut` | เก็บชิ้นสินค้า — เลือกได้หลายจุด แล้วกด **ใช้ที่เลือก** (ค่าเริ่ม Magnetic) |
| ลบ | `remove` | เจาะรูโปร่งใส — เลือกได้หลายจุด แล้วกด **ใช้ที่เลือก** (ค่าเริ่ม Magnetic) |
| ครอบ | `crop` | ลากกรอบสี่เหลี่ยม ตัดตามกรอบทันที (`keep` + `manual` + `rect`) |
| แปรง | `brush` | ลากทับเพื่อลบทันที (`erase` + `brush`) · วงกลมนำทางตามหัวแปรง |

แถบ Lasso โชว์เฉพาะ **ตัด / ลบ**. เพิ่ม/หัก และหมุนอยู่ใน expander «เพิ่มเติม»

## Lasso (Photoshop-style)

| ปุ่ม | Key | ติดขอบ | Canvas | ปิดวง |
|------|-----|--------|--------|-------|
| อิสระ | `lasso` | ไม่ | `freedraw` | ปล่อยเมาส์ → เส้นกลับไปจุดเริ่ม — ค้างบนแคนวาสจนกดใช้ที่เลือก |
| เหลี่ยม | `polygonal` | ไม่ | `point` | คลิกมุม · คลิกใกล้จุดแรก (≥4 จุด) แล้วเริ่มวงใหม่ได้ |
| ดูดขอบ | `magnetic` | ใช่ — `select_object` | `freedraw` | snap แล้วค้าง outline · แตะสั้น = คลิกวัตถุ · กดใช้ที่เลือกทีเดียว |

`_MANUAL_LASSO_MODES = ("lasso", "polygonal", "magnetic")`

อิสระ/เหลี่ยม = `manual` — ห้ามเรียก `select_object`  
ดูดขอบ = `object` + `lasso` — วงบนจอคือ silhouette **วัตถุจริง** (+ เงาที่ติดกัน) **ห้าม**เป็นเส้นที่ลาก · snap ไม่ได้ = snack แล้วไม่แสดงเส้นวง · แตะสั้น (<3 จุด) = คลิกวัตถุ

## Two region modes (engine)

| ปุ่ม | Key | ติดขอบ | พื้นที่เก็บ/ลบ |
|------|-----|--------|----------------|
| ติดขอบอัตโนมัติ | `object` | ใช่ — เรียก `select_object` API | ตาม polygon ที่ snap แล้ว |
| วงเอง | `manual` | ไม่ — ครอปตามเครื่องหมาย | rect / เหลี่ยมปิดวง / lasso ตามที่วาด |

`_MANUAL_TOOLS = ("object", "manual", "brush", "transform")`
`_MANUAL_JOBS = ("cut", "remove", "crop", "brush")`

Internal gestures (`_MANUAL_AUTO_GESTURES`): `rect` / `click` / `lasso` / `polygonal`

## Object (`object`)

1. gesture → `_handle_auto_select` → `select_object` → remount วงที่ติดขอบจริง (`keep_overlay`) — **ยังไม่บันทึก**
2. คลิก / แตะ Magnetic = ชี้วัตถุ (ไม่ใช่ polygon หลายจุด)
3. กรอบ/เส้น = hint ให้ engine ติดขอบวัตถุ — วงบนจอคือขอบสินค้าจริง ไม่ใช่เส้นที่ลาก และห้าม fallback เป็น `_gesture_region_shapes`
4. เรียก API ผ่าน HTTP เท่านั้น (`frontend/api_client`) · `_lasso_mask` ห้ามคืน filled loop
5. กด **ใช้ที่เลือก** → `_commit_pending_manual_selection` → `_queue_manual_autosave`

## Manual (`manual`)

1. กรอบ (ครอบ) = สี่เหลี่ยมจริง — autosave ทันที อย่าเรียก `select_object`
2. เหลี่ยม = คลิกมุม ห้ามถือ 3 จุดเป็น polygon — ปิดเมื่อ `_polygonal_ring` แล้ว bake เป็น outline เริ่มวงใหม่
3. อิสระ = drag freehand ค้างบนแคนวาส — อย่า snap
4. ตัด/ลบอ่านจาก `_pending_manual_shapes` ตอนกดใช้ที่เลือก — แปรง/ครอบยัง finalize ทันที

## Canvas / anti-flicker

กฏโปรเจค: `skills/no-image-flicker/SKILL.md` — หลังแก้ภาพห้ามกะพริบ ตรวจทุกครั้งก่อนจบงาน

- ดูดขอบ: snap แล้ว**แสดงวงตามขอบจริง** บนแคนวาส (`keep_overlay` + วาดวงลงจานรูปก่อน remount) — ห้าม `st.rerun` ทั้งหน้า
- อิสระ: เส้นค้างบน Fabric — ห้าม remount ทุก stroke
- เหลี่ยม: remount เฉพาะตอนปิดวง (`_polygonal_ring`) เพื่อเริ่มวงใหม่
- แปรง / ครอบ: วงครบ → บันทึกทันที (`_handle_manual_finalize` → `_queue_manual_autosave`)
- แปรง: วงกลมตาม `stroke_width` ใน iframe ของแคนวาส (`brush_cursor.js`) — ห้าม CSS overlay หน้า Streamlit · ปรับขนาดไม่ remount
- เหลี่ยมที่ยังไม่ปิด: จุดบนแคนวาส (โหมด point) ห้าม inject Fabric guide และห้าม remount ทุกคลิก
- remount (`CLIP_MANUAL_CANVAS_REV`) เฉพาะ: clear, ซูม, สลับ tool/gesture, **snap bake ดูดขอบ**, ปิดวงเหลี่ยม, apply/บันทึก
- ตัด / ลบ: วงครบแล้วสะสม — กด **ใช้ที่เลือก** ครั้งเดียวแล้วบันทึก + ประวัติ
- สลับ lasso ระหว่างตัด/ลบ: bake outline จาก snap ลงแคนวาส แล้วคง objects
- แถบเครื่องมือ (job / lasso / dock) ต้องคงค่าเดิมหลังผลลัพธ์เปลี่ยน — เก็บใน `CLIP_MANUAL_TOOL_PREFS` แล้วคืนหลัง `st.rerun(scope="app")`
- ห้าม `st.image` พรีวิวแยกใต้แคนวาส
- native Streamlit เท่านั้น — ห้าม inject CSS overlay

## Must (ทั้งสองโหมด)

1. ตัด / ลบ: เลือกได้หลายจุดแล้วกด **ใช้ที่เลือก** ครั้งเดียว — แปรงและครอบยังทำทันทีหลังวงครบ ไม่มีปุ่ม «บันทึกแล้วแก้ต่อ»
2. ประวัติด้านข้างขึ้นทันทีหลังแก้ — คืนค่าแล้วแก้ต่อ
3. ตัวแก้เป็น**หน้าเต็ม** (`render_manual_editor_page`) — `st.dialog` เหลือไว้ยืนยันคืนค่า / แจ้งเตือน ห้ามหุ้มแคนวาสด้วย dialog
4. native Streamlit เท่านั้น — ห้าม inject CSS overlay
5. ห้าม `st.image` พรีวิวแยกใต้แคนวาส

## Forbid

- เรียก `select_object` จากโหมด `manual` (อิสระ / เหลี่ยม)
- ครอปตามกรอบ/lasso ตรงๆ ในโหมด `object` (ต้อง snap ก่อน — ห้ามใช้วงที่ลากเป็นผลลัพธ์)
- autosave เหลี่ยมที่ยังไม่ปิดวง (อย่าใช้ `_shapes_from_objects` 3 จุดในโหมด polygonal — ใช้ `_pending_manual_shapes`)
- autosave ตัด/ลบทุกครั้งที่วงครบ (ต้องรอใช้ที่เลือก)
- bake ผล + rerun backdrop ทุก stroke
- remount แคนวาสทุกครั้งที่อิสระวงครบ (ทำให้รูปกระพริบ)
- remount ดูดขอบโดยไม่มี `keep_overlay` / ไม่วาดวงลงจานรูปก่อน (รูปว่างกะพริบ)
- ลบวงที่วาดเสร็จเมื่อสลับ lasso ในตัด/ลบ (remount เพื่อเปลี่ยน `drawing_mode` ได้ แต่คง objects)
- รอปุ่ม «บันทึกแล้วแก้ต่อ» ก่อนเขียนไฟล์ / ประวัติ

## Files

| ไฟล์ | หน้าที่ |
|------|--------|
| `clip_image_picker.py` | หน้าแก้รูป + แคนวาส + ประวัติ · popup ยืนยันคืนค่า |
| `clip/brush_cursor.js` | วงกลมนำทางตามหัวแปรงใน iframe ของแคนวาส |
| `shared/image_ops/polygon_crop.py` | ครอป/ลบตาม `shapes` |
| `shared/image_ops/object_select.py` | engine ติดขอบ — โหมด `object` เท่านั้น |
| `backend/api/routes/files.py` | `POST /files/select-object` |
