---
name: scraper-safe
description: >-
  Safe product image/data scraping rules for ThaiSnapAI. Use when changing
  scrapers, import products, Playwright/HTML fetch, or image download pipelines.
---

# Scraper Safe

Details + file map: `ThaiSnapAI/AGENTS.md` section on product images.

## Allow

- Scrape product links the user is allowed to use (affiliate / own clip)
- Store under `ThaiSnapAI/shared/assets/raw_images/{product_id}/` — one folder per product
- Prefer official platform API when credentials exist — HTML/Playwright is fallback

## Forbid

- Sweep whole page (recommend, banner, avatar, logo, voucher, shop cover)
- Mix images across products or dump into one folder
- Take more images than the real PDP gallery
- Bypass login / captcha / paywall
- `query_selector_all("img")` or intercept every CDN response on the page

## Order

1. Parse product id from URL
2. Read item `images` from JSON/HTML first
3. Else Playwright on PDP gallery DOM only
4. Filter product CDNs only → dedupe sizes → download on import confirm

## Import flow

นำเข้า → บันทึก (`is_kept=false`) → แท็บรอตัดสินใจแก้รูป → เก็บคลัง (`is_kept=true`)  
ห้ามครอป/แก้รูปเต็มรูปแบบในหน้านำเข้า — ทำที่รอตัดสินใจเท่านั้น
