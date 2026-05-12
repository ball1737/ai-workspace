# Stripe Local Test — Summary

## สถานะปัจจุบัน (Current State)

`happywork-backend` มี Stripe v2 integration พร้อมใช้งานครบแล้ว — ต่อยอดได้ทันทีโดยไม่ต้อง implement เพิ่ม.

### โครงสร้างที่มีอยู่

| Component                | Path                                                                   | บทบาท                                                  |
| ------------------------ | ---------------------------------------------------------------------- | ------------------------------------------------------ |
| Stripe SDK config        | `src/configs/stripe.config.ts`                                         | secretKey / publishableKey / webhookSecret / `apiVersion '2025-08-27.basil'` / currency `thb` / `taxRateId` |
| Stripe service utilities | `src/utils/stripeService.ts` (912 บรรทัด)                              | createProduct / createPrice / Checkout Session / Subscription / Invoice helpers |
| Webhook middleware       | `src/middlewares/stripeWebhook.middleware.ts`                          | raw-body parser + signature verification               |
| Webhook controller       | `src/api/v2/webhooks/stripe-webhook.controller.ts` (796 บรรทัด)        | จับ 4 events: `checkout.session.completed`, `invoice.paid`, `invoice.payment_failed`, `customer.subscription.updated` |
| Webhook route            | `src/api/v2/webhooks/stripe-webhook.routes.ts`                         | `POST /api/v2/webhooks/stripe`                         |
| Subscription routes      | `src/api/v2/admin/subscription/subscription.routes.ts`                 | current / create / update / cancel / resume / calculate / compare / invoices |
| Package routes           | `src/api/v2/admin/package/package.routes.ts`                           | สร้าง package master + addons master ใน Stripe         |
| Identifier sync (ใหม่)   | `src/api/v2/sale-dashboard/{packageCrud,addon}/*.routes.ts`            | `PATCH /sale-dashboard/{packages,addons}/:uuid/identifier` (metadata-only ไม่ยิง Stripe API) |

### DB schema ที่เกี่ยวข้อง

- `comp_packages` — `stripe_product_id` (varchar nullable), `stripe_price_id` (varchar nullable)
- `comp_companies` — `stripe_customer_id`, `stripe_subscription_id`, `selected_package_uuid`, `subscription_status`, `subscription_expiration`, `is_customer`, `lead_status`, `trial_status`, `max_users`, `add_ons` (JSONB legacy)
- `comp_company_addons` — Phase 1-7 schema ใหม่ (ไม่มี stripe column ตรง — ใช้ FK `addon_id`)

### เอกสารอ้างอิง (Existing)

- `_docs/requirement/stripe-package-payment/stripe-setup-guide.md` (900 บรรทัด)
- `_docs/requirement/stripe-package-payment/environment-configuration.md`
- `_docs/requirement/stripe-package-payment/test-flow/user/01..06-*.md`
- `_docs/requirement/stripe-package-payment/api-usage-guide.md`

## สิ่งที่จะทำ (Planned Changes)

**ไม่มีการแก้โค้ด** — เป้าหมายคือเตรียม environment + ทดสอบ flow ที่มีอยู่บน local เพื่อ:

1. ยืนยันว่า Stripe webhook chain ทำงานถูกต้อง (signature → handler → DB update)
2. รัน scenario สำคัญ 3 อย่าง: trial signup → checkout conversion → subscription update
3. เป็น baseline ก่อนทำ `stripe-sync-rewrite` (handoff doc รออยู่)

### Output ของงาน

- ติดตั้ง Stripe CLI + login
- ตั้งค่า `.env` ครบ (Stripe keys + webhook secret + DB)
- รัน backend ที่ `localhost:3003`
- forward webhook event ผ่าน `stripe listen`
- ผ่าน 3 test scenarios (ดูใน checklist)
- verify DB state เปลี่ยนตาม `customer_flag_logic`

### ผลกระทบ

- ไม่กระทบ production (test mode keys + local DB)
- ไม่เปลี่ยน schema / migration
- ไม่กระทบ frontend (sale-cms) — flow ทดสอบจะ trigger จาก curl/Stripe Checkout URL ตรง

## Drift / ข้อควรระวัง

| ประเด็น                         | รายละเอียด                                                               |
| ------------------------------- | ------------------------------------------------------------------------ |
| `STRIPE_TAX_RATE_IDS` (plural)  | `.env.example` ใช้ plural แต่ `stripe.config.ts` อ่าน `STRIPE_TAX_RATE_ID` (singular). ใช้ singular |
| Port mismatch                   | `.env.example` = 4305, `.env` ปัจจุบัน = 3003. ใช้ 3003 (ตรงกับ frontend `NEXT_PUBLIC_SALE_DASHBOARD_API`) |
| Webhook signing secret rotation | ทุกครั้งที่ stop/start `stripe listen` จะได้ `whsec_...` ใหม่ → ต้อง update `.env` + restart backend |
| `apiVersion` 2025-08-27.basil   | เป็น beta — Stripe SDK v18.5.0 รองรับแต่ควรย้ายไป stable เมื่อ Stripe release |
| 4 events เท่านั้น               | event อื่นที่ Stripe ส่งจะ no-op (200 + ไม่ทำอะไร) — ถ้าต้องการ logic เพิ่ม ต้องแก้ controller |
