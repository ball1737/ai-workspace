# Stripe Local Test — Checklist

## ขั้นตอนการดำเนินงาน

### 1. ติดตั้ง Stripe CLI

- [ ] ติดตั้ง Stripe CLI: `brew install stripe/stripe-cli/stripe`
- [ ] Login: `stripe login` (เปิด browser ให้ authorize)
- [ ] ตรวจสอบ: `stripe --version` แสดง version

### 2. เตรียม Stripe test mode

- [ ] เปิด Dashboard <https://dashboard.stripe.com> → toggle ไป **Test mode** (มุมขวาบน)
- [ ] Developers → API keys → copy `pk_test_...` และ `sk_test_...`
- [ ] (Optional) Tax rates → สร้าง VAT 7% → copy `txr_...`

### 3. ตั้งค่า `.env` ของ `happywork-backend`

ไฟล์: `happywork-backend/.env`

- [ ] ใส่ `PUBLIC_STRIPE_KEY=pk_test_...`
- [ ] ใส่ `STRIPE_SECRET_KEY=sk_test_...`
- [ ] ใส่ `STRIPE_WEBHOOK_SECRET=whsec_...` (จะได้จาก Step 5 — ใส่ทีหลัง)
- [ ] (Optional) `STRIPE_TAX_RATE_ID=txr_...` ⚠ singular ไม่ใช่ plural
- [ ] ยืนยัน `PORT=3003`
- [ ] ยืนยัน `DB_DATA_HOST/PORT/USER/PASSWORD/NAME` ตรงกับ Postgres local
- [ ] ตรวจ verify: `grep -E "^(PORT|STRIPE)" happywork-backend/.env`

### 4. Migrate DB schema

- [ ] `cd happywork-backend && pnpm db:migrate:status` — เช็คว่า migration ตรง
- [ ] `pnpm db:migrate:latest` — apply migration ที่เหลือ
- [ ] ตรวจ verify: SQL `\d comp_packages` มี `stripe_product_id`, `stripe_price_id`
- [ ] ตรวจ verify: SQL `\d comp_companies` มี `stripe_customer_id`, `stripe_subscription_id`, `selected_package_uuid`

### 5. Forward webhook ด้วย Stripe CLI

เปิด terminal แยก (ปล่อยทำงานค้าง):

- [ ] รัน `stripe listen --forward-to localhost:3003/api/v2/webhooks/stripe`
- [ ] copy `whsec_...` ที่ออกใน console → paste ลง `STRIPE_WEBHOOK_SECRET` ใน `.env`
- [ ] ⚠ ทุกครั้งที่ stop/start `stripe listen` จะได้ secret ใหม่ → ต้อง update `.env` + restart backend

### 6. Run backend

- [ ] `cd happywork-backend && pnpm dev`
- [ ] ตรวจ verify: log boot ไม่มี error (`Server listening on 3003`)
- [ ] ตรวจ verify: ไม่มี `Invalid API key` หรือ `Stripe authentication failed`

### 7. (Optional) Seed mock data

- [ ] `npx ts-node scripts/seed-stripe-webhook-payments.ts` (mock payment + addon + invoice)
- [ ] `npx ts-node scripts/seed-sale-dashboard-mock-data.ts` (บริษัท trial / customer mock)

### 8. (Optional) สร้าง Package master ใน Stripe

ถ้ายังไม่มี product/price ใน Stripe Dashboard → ยิง backend สร้างให้ (จะ writeback `stripe_product_id` ลง `comp_packages`):

- [ ] `curl -X POST http://localhost:3003/api/v2/package/master -H 'Content-Type: application/json' -d '{...}'` (payload ตาม `_docs/requirement/stripe-package-payment/api-usage-guide.md`)
- [ ] ตรวจ verify: Stripe Dashboard → Products → มี product ใหม่
- [ ] ตรวจ verify: SQL `SELECT name, stripe_product_id, stripe_price_id FROM comp_packages` มี ID populated

### 9. Test Scenario A — Trial signup (no Stripe)

อ้างอิง: `_docs/requirement/stripe-package-payment/test-flow/user/01-registration-and-trial.md`

- [ ] สมัครบริษัทใหม่ผ่าน registration endpoint
- [ ] ตรวจ verify: SQL row ใน `comp_companies` มี `is_customer=false`, `trial_status='active'`, `lead_status='TRIAL_ACTIVE'`, `trial_end_date = trial_start_date + 30 days`

### 10. Test Scenario B — Checkout conversion (Stripe webhook)

อ้างอิง: `_docs/requirement/stripe-package-payment/test-flow/user/03-self-subscription-and-stripe-payment.md`

- [ ] เรียก `POST /api/v2/subscription/create` (auth required) → ได้ `checkoutUrl`
- [ ] เปิด checkoutUrl ใน browser → ใส่ test card `4242 4242 4242 4242` / exp อนาคต / CVC อะไรก็ได้
- [ ] กด Pay → Stripe redirect กลับ `success_url`
- [ ] ตรวจ verify: terminal `stripe listen` แสดง `--> checkout.session.completed [evt_xxx]` + `<-- [200] POST .../webhooks/stripe`
- [ ] ตรวจ verify: SQL row บริษัทเปลี่ยนเป็น `is_customer=true`, `stripe_customer_id`, `stripe_subscription_id` populated, `subscription_status='active'`, `lead_status='CONVERTED'`, `trial_status='converted'`

### 11. Test Scenario C — Subscription update

อ้างอิง: `_docs/requirement/stripe-package-payment/test-flow/user/04-subscription-after-payment.md`

- [ ] เรียก `POST /api/v2/subscription/update` (เปลี่ยน package หรือ quantity)
- [ ] ตรวจ verify: terminal `stripe listen` แสดง `--> customer.subscription.updated [evt_xxx]` + 200
- [ ] ตรวจ verify: SQL `max_users` หรือ `selected_package_uuid` ใน `comp_companies` update ตามที่ส่ง

### 12. Test Scenario D — Webhook retry (เป็น optional)

- [ ] `stripe events resend evt_xxxxxxxxxxxx` (ID จาก Step 10) → backend รับซ้ำ → handler ต้อง idempotent
- [ ] `stripe trigger invoice.payment_failed` → backend log handler ตอบ 200 + DB เปลี่ยน flag (ถ้ามี logic)

## Unit Test (Manual)

| Test Case                                         | ผลที่คาดหวัง                                                              | ผลการ Test         | สถานะ      |
| ------------------------------------------------- | ------------------------------------------------------------------------- | ------------------ | ---------- |
| `stripe listen` แสดง `whsec_...`                  | Console พิมพ์ `Your webhook signing secret is whsec_...`                  | -                  | - [ ] ผ่าน |
| `pnpm dev` boot สำเร็จ                            | Log `Server listening on 3003` ไม่มี Stripe error                        | -                  | - [ ] ผ่าน |
| `stripe trigger checkout.session.completed`       | terminal `stripe listen` แสดง `<-- [200]`                                 | -                  | - [ ] ผ่าน |
| `stripe trigger invoice.paid`                     | terminal `stripe listen` แสดง `<-- [200]`                                 | -                  | - [ ] ผ่าน |
| `stripe trigger invoice.payment_failed`           | terminal `stripe listen` แสดง `<-- [200]`                                 | -                  | - [ ] ผ่าน |
| `stripe trigger customer.subscription.updated`    | terminal `stripe listen` แสดง `<-- [200]`                                 | -                  | - [ ] ผ่าน |
| Test Scenario A (trial signup)                    | DB row `is_customer=false`, `trial_status='active'`                       | -                  | - [ ] ผ่าน |
| Test Scenario B (checkout conversion)             | DB row `is_customer=true`, `stripe_customer_id` populated, `lead_status='CONVERTED'` | -                  | - [ ] ผ่าน |
| Test Scenario C (subscription update)             | DB row `max_users` / `selected_package_uuid` update ตาม payload           | -                  | - [ ] ผ่าน |
| Webhook signature mismatch (ใช้ `whsec_...` เก่า) | terminal `stripe listen` แสดง `<-- [400]`                                 | -                  | - [ ] ผ่าน |

## Reference Test Cards

| Card                  | Behaviour                          |
| --------------------- | ---------------------------------- |
| `4242 4242 4242 4242` | Success                            |
| `4000 0027 6000 3184` | 3D Secure required                 |
| `4000 0000 0000 9995` | Declined — insufficient funds      |
| `5555 5555 5555 4444` | Mastercard success                 |
| `3782 822463 10005`   | American Express success           |

## Reference Commands

```bash
# Stripe CLI
stripe login
stripe listen --forward-to localhost:3003/api/v2/webhooks/stripe
stripe trigger checkout.session.completed
stripe trigger invoice.paid
stripe trigger invoice.payment_failed
stripe trigger customer.subscription.updated
stripe events resend evt_xxxxxxxxxxxx
stripe events list --limit 10

# Backend
pnpm db:migrate:status
pnpm db:migrate:latest
pnpm dev
npx ts-node scripts/seed-stripe-webhook-payments.ts
npx ts-node scripts/seed-sale-dashboard-mock-data.ts

# DB verify (psql)
SELECT uuid, is_customer, stripe_customer_id, stripe_subscription_id,
       selected_package_uuid, subscription_status, subscription_expiration,
       trial_status, lead_status, max_users
FROM comp_companies
WHERE uuid = '<company_uuid>';
```
