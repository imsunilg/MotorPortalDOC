# Motor Portal — ER Diagram

**Naming note:** the SQL scripts in `MotorPortalDB/scripts/02_tables/*.sql`
are *written* in UPPER_SNAKE_CASE (e.g. `CREATE TABLE USER_MASTER (...)`),
but PostgreSQL folds all unquoted identifiers to lowercase — so the real,
physical table and column names in the running database are lowercase
snake_case (`user_master`, `batch_master`, `master_policy_id`, ...). This is
confirmed independently by `MotorPortalAPI`'s EF Core mapping, which maps
every entity to the lowercase form explicitly (e.g.
`e.ToTable("user_master", Schema); e.Property(x => x.UserId).HasColumnName("user_id")`
in `AppDbContext`), and by the DB repo's own smoke-tested `psql` queries
(`SELECT count(*) FROM user_master`). The original LLD's use of
UPPER_SNAKE_CASE names is fine as **logical/conceptual** shorthand in prose
— this document and any literal SQL/JSON below use the real lowercase
names.

## The 15 entities

`user_master`, `product_master`, `function_master`, `master_policy`,
`batch_master`, `batch_detail`, `invalid_records`, `premium_details`,
`gst_details`, `proposal_master`, `payment_details`, `policy_master`,
`policy_certificate`, `report_log`, `audit_log` — all in schema
`"SGInsurance"` of database `SGInsuranceDB`.

## Diagram

```mermaid
erDiagram
    user_master ||--o{ batch_master : "uploads (user_id)"
    user_master ||--o{ report_log : "exports (user_id)"
    user_master ||--o{ audit_log : "performs (user_id)"

    product_master ||--o{ batch_master : "used in (product_id)"
    product_master ||--o{ master_policy : "belongs to (product_id)"

    function_master ||--o{ batch_master : "used in (function_id)"

    batch_master ||--o{ batch_detail : "contains (batch_id)"
    batch_master ||--o{ invalid_records : "flags (batch_id)"

    batch_detail ||--o{ invalid_records : "rejected as (detail_id)"
    batch_detail ||--o{ premium_details : "priced as (detail_id)"
    batch_detail ||--o{ proposal_master : "proposed as (detail_id)"

    premium_details ||--|| gst_details : "taxed as (premium_id, unique FK)"

    proposal_master ||--o{ payment_details : "paid via (proposal_id)"
    proposal_master ||--o| policy_master : "issues (proposal_id)"

    master_policy ||--o{ payment_details : "funds (master_policy_id)"

    payment_details ||--o| policy_master : "confirms (payment_id)"

    policy_master ||--o{ policy_certificate : "certified by (policy_id)"

    user_master {
        bigint user_id PK
        varchar username UK
        varchar password_hash
        varchar role
        char status
        timestamp created_on
    }
    product_master {
        int product_id PK
        varchar product_code UK
        varchar product_name
        char status
    }
    function_master {
        int function_id PK
        varchar function_code UK
        varchar function_name
    }
    master_policy {
        bigint master_policy_id PK
        int product_id FK
        varchar master_policy_no UK
        varchar customer_no
        varchar cdbg_no
        numeric cd_balance
    }
    batch_master {
        bigint batch_id PK
        bigint user_id FK
        int product_id FK
        int function_id FK
        varchar file_name
        int total_records "CHECK <= 200"
        int valid_records
        int invalid_records
        varchar status "9-state lifecycle"
        timestamp created_on
    }
    batch_detail {
        bigint detail_id PK
        bigint batch_id FK
        varchar master_policy_no
        varchar engine_no
        varchar chassis_no
        varchar tc_no
        varchar invoice_no
        date transit_date
        varchar make
        varchar model
        varchar record_status
    }
    invalid_records {
        bigint invalid_id PK
        bigint batch_id FK
        bigint detail_id FK
        date transit_date
        varchar invoice_no
        varchar engine_no
        varchar chassis_no
        varchar error_remarks
    }
    premium_details {
        bigint premium_id PK
        bigint detail_id FK
        numeric base_premium
        numeric addon_premium
        numeric discount
        numeric net_premium
    }
    gst_details {
        bigint gst_id PK
        bigint premium_id FK "UNIQUE"
        numeric gst_rate
        numeric gst_amount
        numeric final_premium
    }
    proposal_master {
        bigint proposal_id PK
        bigint detail_id FK
        varchar proposal_no UK
        numeric premium_amount
        varchar status
        varchar remarks
    }
    payment_details {
        bigint payment_id PK
        bigint proposal_id FK
        bigint master_policy_id FK
        varchar pf_ref_no
        numeric amount
        varchar payment_status
        timestamp tagged_on
    }
    policy_master {
        bigint policy_id PK
        bigint proposal_id FK
        bigint payment_id FK
        varchar policy_no UK
        varchar make
        varchar model
        varchar engine_no
        varchar chassis_no
        numeric premium
        varchar status
        timestamp issued_on
    }
    policy_certificate {
        bigint cert_id PK
        bigint policy_id FK
        varchar cert_path
        timestamp generated_on
    }
    report_log {
        bigint report_id PK
        bigint user_id FK
        varchar report_type
        date from_date
        date to_date
        timestamp generated_on
    }
    audit_log {
        bigint audit_id PK
        bigint user_id FK
        varchar entity_name
        varchar action
        varchar ref_id
        timestamp timestamp
    }
```

## Cardinality notes (verified against `scripts/02_tables` and `03_constraints_indexes.sql`)

- `batch_detail.batch_id`, `invalid_records.batch_id`/`detail_id` are
  `ON DELETE CASCADE` — deleting a batch cascades to its details and any
  invalid-record rows.
- `premium_details.detail_id` and `gst_details.premium_id` are also
  `ON DELETE CASCADE` from `batch_detail`/`premium_details` respectively;
  `gst_details.premium_id` carries a `UNIQUE` constraint, making
  `premium_details ||--|| gst_details` a strict one-to-one (a premium row
  has at most one GST row, and never more).
- `proposal_master.detail_id` cascades from `batch_detail` but is **not**
  unique in the DDL itself — in practice the API's `ProcessBatchAsync`
  creates at most one proposal per valid `batch_detail` row, one-to-zero-or-one
  by application convention rather than a DB constraint.
- `payment_details.proposal_id` cascades from `proposal_master`;
  `payment_details.master_policy_id` is a plain (non-cascading) FK to
  `master_policy` — a master policy is never deleted by a payment being
  deleted, and `sp_tag_payment` deducts `master_policy.cd_balance`
  transactionally with the `payment_details` insert.
- `policy_master.proposal_id` and `policy_master.payment_id` are plain FKs
  (no cascade) — a policy, once issued, is not automatically removed if the
  originating proposal/payment row is (there is no delete path for either
  in the shipped API).
- `policy_certificate.policy_id` is `ON DELETE CASCADE` from
  `policy_master`.
- `batch_master.total_records` carries `CHECK (total_records <= 200)` —
  the 200-case batch cap is enforced at the database layer, not only in
  the UI/API.
- `invalid_records` is intentionally **not** a blocking join: it exists
  alongside `batch_detail`, not instead of it, precisely so a rejected case
  is isolated without ever removing or blocking the batch's valid rows (see
  `docs/batch-lifecycle.md`).
