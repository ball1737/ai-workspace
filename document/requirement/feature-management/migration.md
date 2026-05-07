---
feature: feature-management
type: feature-migration-plan
created: 2026-05-04
updated: 2026-05-04
owner: SA (initial), Backend Dev / DevOps (execution)
status: draft
---

# Feature Management — Migration Plan

> แผน migration ลำดับ + rollback + data backfill rules

---

## ลำดับ Migration

ทุกไฟล์อยู่ที่ `happywork-backend/src/database/postgresql/migrations/data/` และใช้ `tableTemplate` (id, uuid, created_at, updated_at, archived_at, created_by, updated_by, archived_by, status_type)

| Step   | Filename Pattern                                      | สิ่งที่ทำ                                                                                 | Risk     |
| ------ | ----------------------------------------------------- | ----------------------------------------------------------------------------------------- | -------- |
| **M1** | `YYYYMMDD-HHMM-create-comp_features.ts`               | สร้าง `comp_features` + GIN index บน `name` + seed                                        | Low      |
| **M2** | `YYYYMMDD-HHMM-create-comp_packages.ts`               | สร้าง `comp_packages` + GIN index + seed จาก `packageMasterData`                          | Low      |
| **M3** | `YYYYMMDD-HHMM-create-comp_package_features.ts`       | สร้าง link table                                                                          | Low      |
| **M4** | `YYYYMMDD-HHMM-create-comp_addons.ts`                 | สร้าง `comp_addons` + GIN index + seed จาก `addonsMasterData`                             | Low      |
| **M5** | `YYYYMMDD-HHMM-create-comp_addon_features.ts`         | สร้าง link table                                                                          | Low      |
| **M6** | `YYYYMMDD-HHMM-create-comp_company_features.ts`       | สร้าง override table (empty)                                                              | Low      |
| **M7** | `YYYYMMDD-HHMM-create-comp_company_addons.ts`         | สร้าง table + migrate data จาก `comp_companies.add_ons` (jsonb)                           | Mid      |
| **M8** | `YYYYMMDD-HHMM-alter-comp_companies-package-id-fk.ts` | (a) backfill `package_id` ใหม่ (b) drop FK เก่า (c) recreate FK ใหม่ → `comp_packages.id` | **High** |

---

## รายละเอียดแต่ละ Migration

### M1 — Create `comp_features`

```typescript
import { Knex } from "knex";
import { tableTemplate } from "@database/postgresql/migrations/tableTemplate";
import { alterTableOnUpdate } from "@database/postgresql/migrations/alterTableOnUpdate";

const tableName = "comp_features";

export async function up(knex: Knex): Promise<void> {
  await knex.schema
    .createTable(tableName, (table) => {
      tableTemplate(knex, table, "active");
      table.string("code", 50).notNullable().unique();
      table.jsonb("name").notNullable();
      table.jsonb("description").notNullable().defaultTo("{}");
      table.jsonb("menu_keys").notNullable().defaultTo("[]");
      table.integer("sort_order").notNullable().defaultTo(0);
    })
    .then(() => knex.raw(alterTableOnUpdate(tableName)));

  // GIN index สำหรับ JSONB query
  await knex.raw(
    `CREATE INDEX idx_comp_features_name_gin ON ${tableName} USING GIN (name)`,
  );

  // Seed feature เริ่มต้นจาก featureDefinitions.ts (FEATURES + ADDONS)
  // menu_keys เริ่มเป็น [] ให้ admin มาแก้ใน UI ภายหลัง
  const seedFeatures = [
    // ตัวอย่าง — list จริงจะ derive จาก FEATURES_NAME_MAP
    {
      code: "dashboard",
      name: { th: "แดชบอร์ด", en: "Dashboard" },
      menu_keys: ["dashboard"],
      sort_order: 1,
    },
    {
      code: "time_attendance",
      name: { th: "การมาทำงาน", en: "Time Attendance" },
      menu_keys: [],
      sort_order: 2,
    },
    {
      code: "employee_management",
      name: { th: "จัดการพนักงาน", en: "Employee Management" },
      menu_keys: [],
      sort_order: 3,
    },
    {
      code: "payroll",
      name: { th: "เงินเดือน", en: "Payroll" },
      menu_keys: [],
      sort_order: 4,
    },
    // ... ฯลฯ
  ];

  await knex(tableName).insert(
    seedFeatures.map((f) => ({
      ...f,
      name: JSON.stringify(f.name),
      menu_keys: JSON.stringify(f.menu_keys),
      description: "{}",
    })),
  );
}

export async function down(knex: Knex): Promise<void> {
  await knex.schema.dropTableIfExists(tableName);
}
```

### M2 — Create `comp_packages` + Seed

```typescript
const tableName = "comp_packages";

export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable(tableName, (table) => {
    tableTemplate(knex, table, "active");
    table.string("code", 50).notNullable().unique();
    table.jsonb("name").notNullable();
    table.string("short_name", 50);
    table.jsonb("description").defaultTo("{}");
    table.string("billing_interval", 20).nullable();
    table.decimal("price_amount", 12, 2).defaultTo(0);
    table.string("currency", 3).defaultTo("thb");
    table.integer("user_limit_min").nullable();
    table.integer("user_limit_max").nullable();
    table.boolean("is_recommend").defaultTo(false);
    table.boolean("is_contact_us").defaultTo(false);
    table.boolean("is_active").defaultTo(true);
    table.string("stripe_product_id", 255).nullable();
    table.string("stripe_price_id", 255).nullable();
    table.integer("sort_order").defaultTo(0);
  });

  await knex.raw(
    `CREATE INDEX idx_comp_packages_name_gin ON ${tableName} USING GIN (name)`,
  );

  // Seed จาก packageMasterData — preserve UUID เดิม (สำคัญ! เพราะ comp_companies.selected_package_uuid ใช้)
  const seedPackages = [
    {
      uuid: "0c076567-7812-4602-bc6c-3474c61abf6f", // Seed
      code: "seed",
      name: { th: "Seed", en: "Seed" },
      short_name: "Seed",
      description: {
        th: "For teams of 1-10 users",
        en: "For teams of 1-10 users",
      },
      billing_interval: null,
      price_amount: 0,
      user_limit_min: 1,
      user_limit_max: 10,
      is_recommend: false,
      sort_order: 1,
    },
    {
      uuid: "a1b2c3d4-5e6f-7a8b-9c0d-e1f2a3b4c5d6", // Lite Monthly
      code: "lite",
      code: "lite_monthly" /* ... */,
    },
    // ... 7 records ตาม packageMasterData (Seed + LITE×2 + CORE×2 + PRO×2)
  ];

  await knex(tableName).insert(
    seedPackages.map((p) => ({
      ...p,
      name: JSON.stringify(p.name),
      description: JSON.stringify(p.description),
    })),
  );
}
```

> **สำคัญ:** Seed ต้อง preserve UUID เดิมจาก `packageMasterData` เพราะ `comp_companies.selected_package_uuid` อ้างอิง UUID เหล่านี้อยู่แล้ว — ใช้สำหรับ M8 backfill

### M3 — Create `comp_package_features` + Seed Cartesian (Q1=C)

> **Decision Q1 = C (resolved 2026-05-04):** seed Cartesian product (ทุก package × ทุก feature) เพื่อไม่ให้ลูกค้าเห็นเมนูหายตอน deploy → admin ค่อยเข้ามาปิดเฉพาะ feature ที่ขายไม่ครบใน CMS หลัง deploy

```typescript
const tableName = "comp_package_features";

export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable(tableName, (table) => {
    table
      .bigInteger("package_id")
      .notNullable()
      .references("id")
      .inTable("comp_packages")
      .onDelete("CASCADE");
    table
      .bigInteger("feature_id")
      .notNullable()
      .references("id")
      .inTable("comp_features")
      .onDelete("CASCADE");
    table.timestamp("created_at").defaultTo(knex.fn.now());
    table.string("created_by", 50);
    table.primary(["package_id", "feature_id"]);
    table.index("feature_id");
  });

  // Seed Cartesian product: ทุก package × ทุก feature ที่ status_type='active'
  // เหตุผล: ตอนแรก deploy ทุก package เปิดทุก feature → ลูกค้าเห็นเมนูเหมือนเดิม
  // หลัง deploy แล้ว admin เข้ามาปิดเฉพาะ feature ที่ package นั้นไม่ควรขายผ่าน CMS
  await knex.raw(`
    INSERT INTO comp_package_features (package_id, feature_id, created_at, created_by)
    SELECT cp.id, cf.id, NOW(), 'migration:M3'
    FROM comp_packages cp
    CROSS JOIN comp_features cf
    WHERE cp.status_type = 'active'
      AND cf.status_type = 'active'
  `);
}
```

### M4 — Create `comp_addons` + Seed

```typescript
const tableName = "comp_addons";

export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable(tableName, (table) => {
    tableTemplate(knex, table, "active");
    table.string("code", 50).notNullable().unique();
    table.jsonb("name").notNullable();
    table.string("short_name", 50);
    table.jsonb("description").defaultTo("{}");
    table.string("billing_interval", 20).nullable();
    table.decimal("price_amount", 12, 2).defaultTo(0);
    table.string("currency", 3).defaultTo("thb");
    table.boolean("is_quantifiable").defaultTo(false);
    table.integer("max_quantity").nullable();
    table.boolean("is_recommend").defaultTo(false);
    table.boolean("is_active").defaultTo(true);
    table.string("stripe_product_id", 255).nullable();
    table.string("stripe_price_id", 255).nullable();
    table.integer("sort_order").defaultTo(0);
  });

  await knex.raw(
    `CREATE INDEX idx_comp_addons_name_gin ON ${tableName} USING GIN (name)`,
  );

  // Seed จาก addonsMasterData — preserve UUID
  const seedAddons = [
    // HappyBeacon — quantifiable (เป็น device)
    {
      uuid: "<from addonsMasterData>",
      code: "happybeacon",
      name: { th: "HappyBeacon", en: "HappyBeacon" },
      is_quantifiable: true,
      max_quantity: 100,
      // ...
    },
    {
      code: "e_slip",
      is_quantifiable: false,
      // ...
    },
    // ฯลฯ
  ];
}
```

### M5 — Create `comp_addon_features`

```typescript
const tableName = "comp_addon_features";

export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable(tableName, (table) => {
    table
      .bigInteger("addon_id")
      .notNullable()
      .references("id")
      .inTable("comp_addons")
      .onDelete("CASCADE");
    table
      .bigInteger("feature_id")
      .notNullable()
      .references("id")
      .inTable("comp_features")
      .onDelete("CASCADE");
    table.timestamp("created_at").defaultTo(knex.fn.now());
    table.primary(["addon_id", "feature_id"]);
    table.index("feature_id");
  });
}
```

### M6 — Create `comp_company_features` (Q4=B: FK feature_id = RESTRICT)

> **Decision Q4 = B (resolved 2026-05-04):** ถ้า feature ถูก soft-delete (status_type='archived'), row override ยังอยู่เพื่อ audit; FK `feature_id` เป็น `ON DELETE RESTRICT` เพื่อกัน hard-delete; Resolver จะ filter `comp_features.status_type = 'active'` เอง

```typescript
const tableName = "comp_company_features";

export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable(tableName, (table) => {
    tableTemplate(knex, table, "active");
    table
      .bigInteger("company_id")
      .notNullable()
      .references("id")
      .inTable("comp_companies")
      .onDelete("CASCADE"); // company hard-delete = orphan rows ไม่ make sense → CASCADE
    table
      .bigInteger("feature_id")
      .notNullable()
      .references("id")
      .inTable("comp_features")
      .onDelete("RESTRICT"); // Q4=B: keep audit trail; soft-delete via status_type only
    table.string("override_status", 20).notNullable();
    table.text("reason").nullable();
    table.unique(["company_id", "feature_id"]);
    table.index("company_id");
    table.index("feature_id");
    table.index("override_status");
  });
}
```

### M7 — Create `comp_company_addons` + Migrate `comp_companies.add_ons`

```typescript
const tableName = "comp_company_addons";

export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable(tableName, (table) => {
    tableTemplate(knex, table, "active");
    table
      .bigInteger("company_id")
      .notNullable()
      .references("id")
      .inTable("comp_companies")
      .onDelete("CASCADE");
    table
      .bigInteger("addon_id")
      .notNullable()
      .references("id")
      .inTable("comp_addons")
      .onDelete("CASCADE");
    table.integer("quantity").notNullable().defaultTo(1);
    table.timestamp("purchased_at", { useTz: true }).notNullable();
    table.timestamp("expires_at", { useTz: true }).nullable();
    table.index("company_id");
    table.index("addon_id");
  });

  // Migrate data จาก comp_companies.add_ons (jsonb array of {uuid, quantity})
  // ใช้ knex.raw เพื่อทำ insert แบบ bulk
  await knex.raw(`
    INSERT INTO comp_company_addons (uuid, company_id, addon_id, quantity, purchased_at, status_type, created_at, updated_at)
    SELECT
      gen_random_uuid(),
      cc.id,
      ca.id,
      COALESCE((addon_item->>'quantity')::int, 1),
      cc.created_at,
      'active',
      NOW(),
      NOW()
    FROM comp_companies cc
    CROSS JOIN LATERAL jsonb_array_elements(COALESCE(cc.add_ons, '[]'::jsonb)) AS addon_item
    JOIN comp_addons ca ON ca.uuid = (addon_item->>'uuid')::uuid
    WHERE cc.add_ons IS NOT NULL AND jsonb_array_length(cc.add_ons) > 0
  `);
}
```

> **หมายเหตุ:** ตรวจสอบ schema ของ `comp_companies.add_ons` ก่อน migrate — ถ้ารูปแบบต่างจาก `[{uuid, quantity}]` ต้องปรับ SQL ให้เหมาะสม

### M8 — Alter `comp_companies.package_id` FK (CRITICAL)

```typescript
export async function up(knex: Knex): Promise<void> {
  // Step (a): Backfill — set package_id ให้ point ไปที่ comp_packages.id (ใหม่) แทน const_packages.id (เก่า)
  // ใช้ comp_companies.selected_package_uuid (ที่ v2 stripe set ไว้แล้ว) เป็นตัว lookup
  await knex.raw(`
    UPDATE comp_companies cc
    SET package_id = cp.id
    FROM comp_packages cp
    WHERE cp.uuid = cc.selected_package_uuid
      AND cc.selected_package_uuid IS NOT NULL
  `);

  // Step (b): สำหรับ row ที่ไม่มี selected_package_uuid (legacy) → set เป็น Seed (free tier) เป็น default
  await knex.raw(`
    UPDATE comp_companies
    SET package_id = (SELECT id FROM comp_packages WHERE code = 'seed' LIMIT 1)
    WHERE selected_package_uuid IS NULL OR selected_package_uuid = ''
  `);

  // Step (c): Drop FK เก่า + Recreate FK ใหม่ → comp_packages.id
  // หา constraint name เดิม:
  // SELECT constraint_name FROM information_schema.table_constraints
  // WHERE table_name = 'comp_companies' AND constraint_type = 'FOREIGN KEY'
  // -- (อาจเป็น 'comp_companies_package_id_foreign')

  await knex.raw(`
    ALTER TABLE comp_companies
    DROP CONSTRAINT IF EXISTS comp_companies_package_id_foreign
  `);

  await knex.raw(`
    ALTER TABLE comp_companies
    ADD CONSTRAINT comp_companies_package_id_foreign
    FOREIGN KEY (package_id) REFERENCES comp_packages(id) ON DELETE SET NULL
  `);
}

export async function down(knex: Knex): Promise<void> {
  // Rollback: revert FK กลับไป const_packages
  // (ก่อนทำ ต้อง map package_id กลับ — แต่ถ้า data ใน comp_packages ไม่ตรงกับ const_packages ก็ rollback ไม่ได้สมบูรณ์)
  await knex.raw(`
    ALTER TABLE comp_companies DROP CONSTRAINT IF EXISTS comp_companies_package_id_foreign
  `);
  await knex.raw(`
    ALTER TABLE comp_companies ADD CONSTRAINT comp_companies_package_id_foreign
    FOREIGN KEY (package_id) REFERENCES const_packages(id)
  `);
  // ⚠️ ค่า package_id ใน comp_companies ตอนนี้ point ไปที่ comp_packages.id (ใหม่)
  // ซึ่งอาจไม่ตรงกับ const_packages.id เดิม — ต้องระวัง
}
```

---

## Pre-Migration Checklist

ก่อนรัน migration บน production:

- [ ] Backup full database
- [ ] รัน migration บน dev environment ก่อน — verify all rows
- [ ] รันบน staging — full QA cycle
- [ ] ตรวจ count: `SELECT COUNT(*) FROM comp_companies WHERE package_id IS NULL` — ต้องเป็น 0 หลัง M8 step (b)
- [ ] ตรวจ FK validity: `SELECT cc.id FROM comp_companies cc LEFT JOIN comp_packages cp ON cc.package_id = cp.id WHERE cp.id IS NULL` — ต้องเป็น empty
- [ ] เก็บ snapshot ของ `comp_companies.package_id` ก่อน migration เพื่อ rollback
- [ ] ตรวจสอบว่า `selected_package_uuid` ของ company ทั้งหมด match กับ `packageMasterData` UUID จริงๆ (ทำ pre-validation query)

```sql
-- Pre-validation: check ว่ามี selected_package_uuid ที่ไม่ match กับ packageMasterData หรือไม่
SELECT cc.id, cc.uuid, cc.selected_package_uuid
FROM comp_companies cc
WHERE cc.selected_package_uuid IS NOT NULL
  AND cc.selected_package_uuid NOT IN (
    -- list 8 UUIDs จาก packageMasterData
    '0c076567-7812-4602-bc6c-3474c61abf6f',
    'a1b2c3d4-5e6f-7a8b-9c0d-e1f2a3b4c5d6',
    'b2c3d4e5-6f7a-8b9c-0d1e-f2a3b4c5d6e7',
    '66a6f3ef-dd54-48b3-bf9a-01e2daeb2672',
    'ba87cc81-92a2-4aca-b7b0-19fe108eb03d',
    'f34f86a7-4e9c-4ece-a98a-2a6f4180da00',
    'cc3c8f7f-64c4-4245-8f57-49462f6a01c7'
    -- + Pro UUIDs
  );
```

---

## Rollback Strategy

แต่ละ migration มี `down()` function ทำ DROP TABLE หรือ revert FK

ถ้าเจอปัญหาหลัง migrate:

1. `npm run db:migrate:rollback` — undo last batch
2. ถ้า M8 rollback ไม่สมบูรณ์ (เพราะ data ไม่ตรงกัน) → restore จาก backup snapshot

---

## Post-Migration Verification

หลังรัน migration:

```sql
-- 1. Tables ทั้งหมดสร้างครบ
SELECT table_name FROM information_schema.tables
WHERE table_name IN ('comp_features', 'comp_packages', 'comp_package_features', 'comp_addons', 'comp_addon_features', 'comp_company_features', 'comp_company_addons');
-- ต้องได้ 7 rows

-- 2. Seed data ครบ
SELECT COUNT(*) FROM comp_features;     -- คาดว่า ~14-30 (ตาม FEATURES + ADDONS)
SELECT COUNT(*) FROM comp_packages;     -- 7 rows (Seed + LITE mo+yr + CORE mo+yr + PRO mo+yr)
SELECT COUNT(*) FROM comp_addons;       -- 5 rows ขั้นต่ำ

-- 3. comp_companies.package_id ทุก row point ไปที่ comp_packages
SELECT COUNT(*) FROM comp_companies cc
LEFT JOIN comp_packages cp ON cc.package_id = cp.id
WHERE cp.id IS NULL;
-- ต้องเป็น 0

-- 4. comp_company_addons migrate มาครบ
SELECT
  (SELECT COUNT(*) FROM comp_companies WHERE jsonb_array_length(COALESCE(add_ons, '[]'::jsonb)) > 0) AS old_count,
  (SELECT COUNT(DISTINCT company_id) FROM comp_company_addons) AS new_count;
-- old_count <= new_count
```

---

## Phase 5 — Mobile Menu Parity (Added 2026-05-06)

> เพิ่ม `mobile_menu_keys` ให้ master data + backfill SEED tier features ก่อน deploy

### M9: ALTER comp_features ADD mobile_menu_keys

**Filename:** `20260506-1800-alter-comp_features-add-mobile_menu_keys.ts` _(implemented 2026-05-06, Phase 5 W1)_

**Purpose:** เพิ่ม column `mobile_menu_keys` JSONB NOT NULL DEFAULT `'[]'` + GIN index คู่ขนานกับ `menu_keys`

**Rollback:** down() drops index แล้ว drop column — safe เพราะ column ใหม่ + GIN index ใหม่ ไม่กระทบ data ที่มีอยู่

```typescript
import { Knex } from 'knex';

const tableName = 'comp_features';
const columnName = 'mobile_menu_keys';
const ginIndexName = 'idx_comp_features_mobile_menu_keys_gin';

export async function up(knex: Knex): Promise<void> {
  await knex.raw(`
    ALTER TABLE ${tableName}
    ADD COLUMN ${columnName} JSONB NOT NULL DEFAULT '[]'::jsonb
  `);

  await knex.raw(`
    CREATE INDEX IF NOT EXISTS ${ginIndexName}
    ON ${tableName} USING GIN (${columnName})
  `);
}

export async function down(knex: Knex): Promise<void> {
  await knex.raw(`DROP INDEX IF EXISTS ${ginIndexName}`);
  await knex.raw(`ALTER TABLE ${tableName} DROP COLUMN IF EXISTS ${columnName}`);
}
```

### M10: Backfill SEED tier mobile_menu_keys

**Filename:** `20260506-1801-backfill-comp_features-mobile_menu_keys-seed.ts` _(implemented 2026-05-06, Phase 5 W1)_

**Purpose:** กรอก `mobile_menu_keys` ให้ 4 features ของ SEED tier (`basic_attendance`, `basic_leave`, `basic_reports`, `max_10_users`) ตาม mapping ที่ user-confirmed (Q7=A+B) — feature ของ CORE/PRO ปล่อย `[]` ไว้ ให้ admin populate ผ่าน CMS

**Idempotency:** `WHERE code = ? AND status_type = 'active'` — กัน update row archived ในอนาคต (Q4=B compatibility) + run ซ้ำได้

**Validation note:** ทุก key ในรายการคือ leaf path ของ `PermissionMobileDefault` (verified ตรงกับ `getAvailableMobileMenuKeys()` ที่ Wave 1 helper exports) — ถ้า PermissionMobileDefault ถูกแก้ในอนาคตให้ key ไม่ตรง ต้องเพิ่ม migration ใหม่ ห้ามแก้ไฟล์เก่าย้อนหลัง

> **User-confirmed mapping (2026-05-06):**
>
> | feature_code | mobile_menu_keys (leaf paths ของ `PermissionMobileDefault`) |
> | --- | --- |
> | `basic_attendance` | `homepage.attendance`, `timeAttendance.myReport` |
> | `basic_leave` | `homepage.request`, `homepage.myRequest`, `homepage.calendar` |
> | `basic_reports` | `timeAttendance.teamReport` |
> | `max_10_users` | `[]` _(capacity feature ไม่มี UI menu)_ |

**Rollback:** down() set กลับเป็น `[]` เฉพาะ 4 SEED rows — column ยังคงอยู่ (column drop เป็น scope ของ M9 down)

```typescript
import { Knex } from 'knex';

const tableName = 'comp_features';

interface SeedMobileMapping {
  readonly code: string;
  readonly mobileMenuKeys: readonly string[];
}

const SEED_MOBILE_MAPPING: readonly SeedMobileMapping[] = [
  { code: 'basic_attendance', mobileMenuKeys: ['homepage.attendance', 'timeAttendance.myReport'] },
  { code: 'basic_leave', mobileMenuKeys: ['homepage.request', 'homepage.myRequest', 'homepage.calendar'] },
  { code: 'basic_reports', mobileMenuKeys: ['timeAttendance.teamReport'] },
  { code: 'max_10_users', mobileMenuKeys: [] },
];

export async function up(knex: Knex): Promise<void> {
  for (const entry of SEED_MOBILE_MAPPING) {
    await knex(tableName)
      .where({ code: entry.code, status_type: 'active' })
      .update({ mobile_menu_keys: JSON.stringify(entry.mobileMenuKeys) });
  }
}

export async function down(knex: Knex): Promise<void> {
  for (const entry of SEED_MOBILE_MAPPING) {
    await knex(tableName)
      .where({ code: entry.code, status_type: 'active' })
      .update({ mobile_menu_keys: JSON.stringify([]) });
  }
}
```

### Phase 5 Pre-Migration Checklist

- [ ] User เคาะ mapping `feature_code → mobile_menu_keys` ก่อน implement M10
- [ ] รัน M9 บน dev → verify column + GIN index มี
- [ ] รัน M10 บน dev → verify SEED tier มี mobile_menu_keys ไม่ว่าง: `SELECT code, jsonb_array_length(mobile_menu_keys) FROM comp_features WHERE sort_order <= 4 ORDER BY sort_order`
- [ ] Backup full database ก่อนรันบน production
- [ ] รันบน staging → full QA cycle (combined parameterized tests Phase 5 W9)

### Phase 5 Post-Migration Verification

```sql
-- 1. Column มีจริง + ข้อมูลเริ่มต้นถูก
SELECT code, sort_order, jsonb_array_length(menu_keys) AS web_count, jsonb_array_length(mobile_menu_keys) AS mobile_count
FROM comp_features
ORDER BY sort_order;
-- คาดหวัง: SEED tier (sort_order <= 4) → mobile_count > 0; CORE/PRO → mobile_count = 0

-- 2. GIN index ใช้งานได้
EXPLAIN ANALYZE
SELECT id FROM comp_features
WHERE mobile_menu_keys @> '["homepage.attendance.read"]'::jsonb;
-- คาดหวัง: Bitmap Index Scan idx_comp_features_mobile_menu_keys_gin

-- 3. Resolver mobile path คืน effective mobile menu keys
-- (ใช้ companyFeature endpoint หรือ permissionResolver service ผ่าน unit test)
```

---

## Phase 6 (Future) — Cleanup (เลื่อนจาก Phase 5 เดิม — 2026-05-06)

หลังจาก deploy Phase 5 stable แล้ว Phase 6:

- drop table `sale_dashboard_disabled_features`
- drop table `const_packages` (เมื่อ confirm ว่า v1 caller ทุกตัวถูก migrate แล้ว)
- drop column `comp_companies.add_ons` (jsonb) ที่ไม่ใช้แล้ว

> ทั้ง 3 ตัวนี้แยกออกจาก scope หลักของงานนี้
