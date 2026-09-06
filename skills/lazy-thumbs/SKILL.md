---
name: lazy-thumbs
description: >-
  Product list and gallery thumbnails must paint the page first, show a
  skeleton tile, then decode in parallel. Use when editing zoomable_image,
  product cards, the image manager grid, or any Streamlit gallery of local
  photos.
---

# Lazy thumbs

หน้ารายการ/แกลเลอรีต้อง**ขึ้นการ์ดทันที** — ห้ามถอดรหัสทุกรูปย่อบนเธรดหลักก่อน paint

ใช้เมื่อแตะ `zoomable_image.py` การ์ดคลัง/รอตัดสินใจ แกลเลอรีจัดการรูป หรือลูป `st.image` จากไฟล์ท้องถิ่น

กติกาทีม: `ThaiSnapAI/AGENTS.md` §4 → **รูปย่อในรายการ**

## Must

1. ใช้ `parallel_zoomable_image` ในรายการสินค้าและกริดแกลเลอรี — ไม่ใช่ `zoomable_image` ในลูปยาว
2. ช่องเทา (`_skeleton_thumb_bytes`) เฉพาะรอบ `@st.fragment(parallel=True)` แรก แล้วแทนที่รูปใน `st.empty()` เดิม
3. รอบถัดไป (ติ๊กการ์ด / fragment ลำดับ) ใช้แคช `_local_thumb_bytes` — ห้ามโชว์ช่องเทาซ้ำ
4. `latest_path` และ decode อยู่ใน worker — เธรดหลักส่ง path ที่เก็บไว้พอ
5. คลิกดูขนาดจริงเปิดแกลเลอรี**ใน fragment** — ค่า return กลับไปพ่อไม่ได้

```python
# ✅ หน้าขึ้นก่อน · ช่องเทา · รูปแทนที่ในที่
parallel_zoomable_image(src, key=f"lib_zoom_{pid}", width=72, gallery=paths, title=title)

# ❌ รอ decode ทั้งหน้า / ไล่โหลดด้วย fragment rerun ตอนรันหน้าเต็ม
zoomable_image(path, key=..., width=72)
st.rerun(scope="fragment")  # ตอนโหลดหน้าเต็ม
```

## ห้าม

- `st.rerun(scope="fragment")` เพื่อไล่รูปย่อตอนรันหน้าเต็ม (Streamlit ยก exception)
- `st.rerun()` ทั้งหน้าเพื่อรีเฟรชรูปย่อ
- inject CSS shimmer / HTML theme
- `st.container(height=…)` คลิปภาพแนวตั้งรอบรูปย่อ
- skeleton หลังรูปขึ้นแล้ว (นั่นคือกะพริบ — ดู `skills/no-image-flicker`)
- ค้างรูปย่อเดิมหลังไดคัท ทั้งที่ `*_product` มีแล้ว (ดู `skills/diecut-show-new`)

## Verify

เปิดคลังหรือรอตัดสินใจที่มีหลายรูป — การ์ดโผล่ก่อน ช่องเทาแล้วรูปจริง · ติ๊กการ์ดแล้วรูปที่ขึ้นแล้วไม่กลับเป็นเทา
