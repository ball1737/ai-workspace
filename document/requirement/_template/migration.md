---
feature: { feature-name }
type: feature-migration-plan
created: { YYYY-MM-DD }
updated: { YYYY-MM-DD }
owner: SA (initial), Backend Dev / DevOps (execution)
status: draft
---

# {Feature Name} — Migration Plan

> แผน migration ลำดับ + rollback + data backfill rules

---

## ลำดับ Migration

> ไฟล์ migration อยู่ที่ `{project}/src/database/postgresql/migrations/data/`
> ใช้ `tableTemplate` (id, uuid, created*at, updated_at, archived_at, created_by, updated_by, archived_by, status_type) หากมี
> Filename pattern: `YYYYMMDD-HHMM-{action}*{table-or-purpose}.ts`

| Step   | Filename Pattern                         | สิ่งที่ทำ                                          | Risk     |
| ------ | ---------------------------------------- | -------------------------------------------------- | -------- |
| **M1** | `YYYYMMDD-HHMM-create-{table1}.ts`       | สร้าง table A + index + seed                       | Low      |
| **M2** | `YYYYMMDD-HHMM-create-{table2}.ts`       | สร้าง table B + index                              | Low      |
| **M3** | `YYYYMMDD-HHMM-create-{linkTable}.ts`    | สร้าง link table                                   | Low      |
| **M4** | `YYYYMMDD-HHMM-alter-{existingTable}.ts` | (a) backfill (b) drop FK เก่า (c) recreate FK ใหม่ | **High** |

---

## รายละเอียดแต่ละ Migration

### M1 — Create `{table1}`

```typescript
import { Knex } from "knex";
import { tableTemplate } from "@database/postgresql/migrations/tableTemplate";
import { alterTableOnUpdate } from "@database/postgresql/migrations/alterTableOnUpdate";

const tableName = "{table1}";

export async function up(knex: Knex): Promise<void> {
  await knex.schema
    .createTable(tableName, (table) => {
      tableTemplate(knex, table, "active");
      // columns
      table.string("code", 50).notNullable().unique();
      table.jsonb("name").notNullable();
      // ...
    })
    .then(() => knex.raw(alterTableOnUpdate(tableName)));

  // Index (ถ้าต้องการ GIN สำหรับ JSONB query)
  await knex.raw(
    `CREATE INDEX idx_${tableName}_name_gin ON ${tableName} USING GIN (name)`,
  );

  // Seed (ถ้ามี)
  const seedRows = [
    /* ... */
  ];
  await knex(tableName).insert(seedRows);
}

export async function down(knex: Knex): Promise<void> {
  await knex.schema.dropTableIfExists(tableName);
}
```

### M2 — Create `{table2}` (+ Seed)

(โครงสร้างเดียวกับ M1)

### M3 — Create `{linkTable}`

```typescript
const tableName = "{linkTable}";

export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable(tableName, (table) => {
    table
      .bigInteger("{a}_id")
      .notNullable()
      .references("id")
      .inTable("{tableA}")
      .onDelete("CASCADE");
    table
      .bigInteger("{b}_id")
      .notNullable()
      .references("id")
      .inTable("{tableB}")
      .onDelete("CASCADE");
    table.timestamp("created_at").defaultTo(knex.fn.now());
    table.primary(["{a}_id", "{b}_id"]);
    table.index("{b}_id");
  });
}

export async function down(knex: Knex): Promise<void> {
  await knex.schema.dropTableIfExists(tableName);
}
```

### M4 — Alter `{existingTable}` (CRITICAL ถ้ามี)

> Migration ที่มีความเสี่ยงสูง — ต้อง pre-validate + backfill ก่อน drop/recreate FK

```typescript
export async function up(knex: Knex): Promise<void> {
  // Step (a): Backfill — set column ใหม่ให้ point ไปที่ table ใหม่
  await knex.raw(`
    UPDATE {existingTable} AS old
    SET {fkColumn} = newTbl.id
    FROM {newTable} newTbl
    WHERE newTbl.uuid = old.{lookupColumn}
      AND old.{lookupColumn} IS NOT NULL
  `);

  // Step (b): สำหรับ row ที่ไม่มี match → set default
  await knex.raw(`
    UPDATE {existingTable}
    SET {fkColumn} = (SELECT id FROM {newTable} WHERE code = '{defaultCode}' LIMIT 1)
    WHERE {lookupColumn} IS NULL
  `);

  // Step (c): Drop FK เก่า + Recreate FK ใหม่
  await knex.raw(`
    ALTER TABLE {existingTable}
    DROP CONSTRAINT IF EXISTS {oldConstraintName}
  `);

  await knex.raw(`
    ALTER TABLE {existingTable}
    ADD CONSTRAINT {newConstraintName}
    FOREIGN KEY ({fkColumn}) REFERENCES {newTable}(id) ON DELETE SET NULL
  `);
}

export async function down(knex: Knex): Promise<void> {
  // ⚠️ Rollback: revert FK กลับไป table เดิม
  // หากค่า fkColumn ปัจจุบัน point ไปที่ id ของ table ใหม่ที่ไม่ตรงกับ id ของ table เดิม
  // จะ rollback ไม่สมบูรณ์ → ต้อง restore จาก backup
}
```

---

## Pre-Migration Checklist

> ทำก่อนรัน migration บน production

- [ ] Backup full database (snapshot)
- [ ] รัน migration บน dev environment ก่อน — verify all rows
- [ ] รันบน staging — full QA cycle
- [ ] ตรวจสอบว่า lookup ของทุก row match ต้นทาง (pre-validation query ด้านล่าง)
- [ ] เก็บ snapshot ของ column ที่จะแก้ก่อน migration (กรณี rollback)

```sql
-- Pre-validation: หาค่าที่ไม่ match (ตัวอย่าง)
SELECT id, uuid, {lookupColumn}
FROM {existingTable}
WHERE {lookupColumn} IS NOT NULL
  AND {lookupColumn} NOT IN (
    SELECT uuid FROM {newTable}
  );
-- ต้องเป็น empty
```

---

## Rollback Strategy

แต่ละ migration มี `down()` function — แต่กรณีที่ data ถูก mutate (M4 step a/b)
rollback จะ revert schema ได้ แต่ data state จะกลับไม่ได้สมบูรณ์ ต้อง:

1. `npm run db:migrate:rollback` — undo last batch
2. ถ้า rollback ไม่สมบูรณ์ (data ไม่ตรงกัน) → restore จาก backup snapshot

---

## Post-Migration Verification

หลังรัน migration:

```sql
-- 1. Tables ครบ
SELECT table_name FROM information_schema.tables
WHERE table_name IN ('{table1}', '{table2}', '{linkTable}');
-- ต้องได้ N rows

-- 2. Seed data ครบ
SELECT COUNT(*) FROM {table1};   -- คาดว่า X
SELECT COUNT(*) FROM {table2};   -- คาดว่า Y

-- 3. FK ทุก row ถูกต้อง (ถ้ามี M4)
SELECT COUNT(*) FROM {existingTable} ex
LEFT JOIN {newTable} newTbl ON ex.{fkColumn} = newTbl.id
WHERE newTbl.id IS NULL;
-- ต้องเป็น 0
```

---

## Future Cleanup (ถ้ามี — แยก scope)

> รายการ migration ที่จะทำหลัง deploy stable แล้ว (เช่น drop legacy table)

- M{N+1}: drop table `{legacyTable1}`
- M{N+2}: drop column `{legacyColumn}` ที่ไม่ใช้แล้ว

> ทั้งหมดนี้ **แยกออกจาก scope หลัก** ของ feature นี้
