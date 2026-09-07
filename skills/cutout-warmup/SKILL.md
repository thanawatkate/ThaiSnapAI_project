---
name: cutout-warmup
description: >-
  On API start, probe this machine and load die-cut / object-select models
  in a background thread. Use when changing FastAPI lifespan, rembg session,
  device_runtime, runtime_warmup, or /health.
---

# Cutout warmup on boot

ตอนเปิด API ต้อง**เช็คสเปกเครื่องแล้วอุ่นโมเดลตัดรูปเบื้องหลัง** — ห้ามให้ผู้ใช้รอโหลดตอนกดไดคัทครั้งแรก

ใช้เมื่อแตะ `backend/api/main.py` `runtime_warmup` `device_runtime` `product_crop` `_get_rembg_session` หรือ `/health`

กติกาทีม: `ThaiSnapAI/AGENTS.md` §1 → **ตอนเปิดแอปโหลดเครื่องมือตัดรูปเบื้องหลัง**

## Must

1. `probe_cutout_device()` ตอน lifespan — เลือก tier จาก RAM / CPU / ONNX
2. โหลด rembg ตาม `die_cut_models()` ใน **daemon thread** — `start_cutout_warmup(fetch=True)`
3. `/health` ตอบทันที (`status=ok`) พร้อม `cutout.tier` / `ready` — ห้ามรอโมเดล
4. ดึงน้ำหนักโมเดลจาก **API เท่านั้น** (`allow_die_cut_model_fetch`) — UI / เทสห้ามบล็อกเพื่อดาวน์โหลด
5. งานไดคัทใช้เซสชันที่อุ่นแล้ว — ห้ามเปิด `new_session` ซ้ำบนเธรดหลักของ request

```python
# ✅ API start
start_cutout_warmup(fetch=True)

# ❌ โหลด birefnet/isnet ใน create_app() ก่อน yield
session = rembg.new_session("birefnet-general")
```

## ห้าม

- บล็อก uvicorn startup / `/health` จนกว่า ONNX จะโหลดจบ
- ดึงโมเดลหนักจาก Streamlit หรือ `python tests/test_quick.py`
- ข้าม probe แล้วบังคับโมเดล GPU บนเครื่อง light

## Verify

เปิด API แล้ว `GET /health` ต้องได้ `status=ok` ทันที · log มี `cutout probe` แล้วตามด้วย `cutout warmup · ready=` โดยไม่ทำให้พอร์ต 8000 ค้าง
