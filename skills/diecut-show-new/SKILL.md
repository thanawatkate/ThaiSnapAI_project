---
name: diecut-show-new
description: >-
  After die-cut / background removal finishes, the UI must show the new
  *_product photo, not the pre-cut original from a stale cache or fragment.
  Use when changing product_crop, apply_prepare_images_result, latest_path,
  zoomable_image thumbs, or the image manager gallery.
---

# Die-cut shows the new photo

หลังไดคัท/ลบพื้นหลัง ภาพบนจอต้องเป็น**ไฟล์ผลลัพธ์ใหม่** (`*_product`) — ห้ามค้างรูปเดิม

ใช้เมื่อแตะ `product_crop` `apply_prepare_images_result` `edit_history.latest_path` แกลเลอรีจัดการรูป หรือรูปย่อ `zoomable_image`

กติกาทีม: `ThaiSnapAI/AGENTS.md` §4 → **ไดคัทเสร็จต้องโชว์รูปใหม่**

## Must

1. เชื่อพาธที่ worker เขียนถ้าไฟล์มีจริง — ห้าม `latest_path` เดินกลับไปต้นฉบับ
2. หลังเขียนไฟล์เรียก `invalidate_latest` — ดัชนีโฟลเดอร์ห้ามค้างก่อนมี `*_product`
3. แกลเลอรี/การ์ดใช้ `display_path` — ไฟล์แก้ที่มีอยู่ชนะต้นฉบับเมื่อแคชโฟลเดอร์เก่า
4. รูปย่อระบุตัวตนด้วย mtime (`st.container(key=…)` + `_local_thumb_bytes(path, mtime_ns)`) — ไฟล์ทับที่เดิมต้องขึ้นใบใหม่
5. อุ่นแคชรูปย่อตอน apply แล้วข้ามช่องเทา — แทนที่ในที่ ไม่กะพริบเป็นต้นฉบับ (ดู `skills/no-image-flicker`)

```python
# ✅ worker output มีไฟล์ → โชว์ไฟล์นั้น
invalidate_latest(out)
item["path"] = out

# ❌ แปลง *_product กลับเป็นต้นฉบับด้วย latest_path ที่แคชไว้ก่อนไดคัท
item["path"] = latest_path(out)
```

## ห้าม

- คงรูปย่อเดิมเพราะ fragment id เป็น delta path เดิม ทั้งที่ไฟล์บนดิสก์เปลี่ยนแล้ว
- `latest_path` คืนต้นฉบับทั้งที่ `*_product` มีอยู่
- โชว์ช่องเทาทับรูปไดคัทที่อุ่นแคชแล้ว

## Verify

ไดคัทอย่างน้อยหนึ่งใบในหน้าจัดการรูป — แกลเลอรีต้องเป็นผลลัพธ์ทันที ไม่ใช่รูปก่อนตัด
