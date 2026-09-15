# Motor Portal — Setup Guide (From Zero)

Full instructions to get all three application repos running locally.
After following this guide, use `docs/smoke-test.md` for the complete
verification checklist — it is not duplicated here.

## 1. Prerequisites

| Tool | Version used/verified | Notes |
|---|---|---|
| .NET SDK | 8.0 | `dotnet --version` should report an 8.x SDK |
| Node.js | LTS compatible with Angular 22 (Node 20.19+ / 22.12+) | `node --version` |
| Angular CLI | 22.x | `npm install -g @angular/cli` (or use the local `ng` via `npx ng`) |
| PostgreSQL | 16 (15+ also supported per MotorPortalDB) | `psql --version` |
| Git | any recent version | to clone all 4 repos |

## 2. Clone all 4 repos into sibling folders

```bash
mkdir MotorPortal && cd MotorPortal
git clone https://github.com/imsunilg/MotorPortalDB.git
git clone https://github.com/imsunilg/MotorPortalAPI.git
git clone https://github.com/imsunilg/MotorPortalWEB.git
git clone https://github.com/imsunilg/MotorPortalDOC.git
```

Resulting layout:

```
MotorPortal/
  MotorPortalDB/
  MotorPortalAPI/
  MotorPortalWEB/
  MotorPortalDOC/
```

## 3. Set up the database (MotorPortalDB)

```bash
cd MotorPortalDB

# Configure connection env vars (adjust to your local Postgres install)
export PGHOST=localhost PGPORT=5432 PGUSER=postgres PGPASSWORD=yourpassword
# On Windows, if psql isn't on PATH, point at the real binary:
export PSQL="/c/Program Files/PostgreSQL/16/bin/psql"

bash migrate.sh
```

This is idempotent (safe to re-run) and runs, in order: `00_create_database.sql`
(creates `SGInsuranceDB`), `01_create_schema.sql` (creates schema
`SGInsurance`), all 15 files in `scripts/02_tables/`, `03_constraints_indexes.sql`,
all files in `scripts/04_functions/`, `scripts/05_views/`, and finally
`06_seed_data.sql`.

To start over: `psql -d SGInsuranceDB -f rollback.sql`, then re-run
`migrate.sh`.

Seeded login credentials (created by `06_seed_data.sql`, bcrypt-hashed via
`pgcrypto`): **username `admin`, password `admin123`**.

## 4. Configure and run the API (MotorPortalAPI)

```bash
cd ../MotorPortalAPI
cp MotorPortal.API/appsettings.Development.json.example MotorPortal.API/appsettings.Development.json
```

The example file already matches the database set up in step 3:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=SGInsuranceDB;Username=postgres;Password=284228"
  },
  "Jwt": {
    "Issuer": "MotorPortalAPI",
    "Audience": "MotorPortalWEB",
    "SigningKey": "dev-only-super-secret-signing-key-change-me-32chars-min!",
    "ExpiryMinutes": 60
  }
}
```

Adjust the password (and any other value) to match your local Postgres
instance if it differs from what you used in step 3.

```bash
dotnet restore
dotnet build
dotnet run --project MotorPortal.API
```

The API listens on the URL printed at startup — typically
`http://localhost:5287` (see `MotorPortal.API/Properties/launchSettings.json`).
Swagger UI is available at `http://localhost:<port>/swagger` in the
Development environment; use the **Authorize** button with a JWT obtained
from `POST /api/auth/login` to call protected endpoints from the UI.

Quick check the API is actually talking to the database:

```bash
curl http://localhost:5287/api/health
# {"status":"healthy","database":"connected",...}
```

## 5. Configure and run the web app (MotorPortalWEB)

```bash
cd ../MotorPortalWEB
npm install
```

Check `src/environments/environment.development.ts` — it should already
point at the API from step 4:

```ts
export const environment = {
  production: false,
  apiBaseUrl: 'http://localhost:5287/api',
};
```

If MotorPortalAPI's port differs in your environment (check
`MotorPortal.API/Properties/launchSettings.json` in the API repo), update
`apiBaseUrl` here to match before serving.

```bash
ng serve
```

Open `http://localhost:4200` and log in with the seeded credentials:

**Login ID:** `admin` &nbsp;&nbsp; **Password:** `admin123`

## 6. Verify the whole system

Once all three are running (DB reachable, API on `:5287`, WEB on `:4200`),
run through **`docs/smoke-test.md`** in this repo — it is the full,
step-by-step, ~15-20 minute checklist covering login, upload, batch
processing, invalid records, payment tagging (including the seeded
insufficient-CD-balance case), certificate view/download, bulk print,
search & print, report export, and policy cancel. Do not duplicate that
checklist here — this guide only gets the stack up; that document verifies
it works.

## Troubleshooting quick reference

| Symptom | Likely cause |
|---|---|
| `dotnet run` fails to connect to Postgres | Check `appsettings.Development.json`'s connection string against the Postgres instance actually running; confirm `SGInsuranceDB` exists (`psql -l`) |
| `ng serve` compiles but every API call 401s/fails | Confirm the API is actually running on the port `environment.development.ts` points at; confirm you're logged in (token in `sessionStorage`, not `localStorage`) |
| Swagger doesn't show the upload endpoints correctly | Expected quirk documented in MotorPortalAPI's README — multipart file-upload actions need the `BatchUploadRequest` wrapper / `FileUploadOperationFilter` workaround already built into the shipped code; this doesn't affect the actual wire format |
| `migrate.sh` fails partway through | It is safe to re-run after fixing the underlying issue — every script uses `CREATE TABLE IF NOT EXISTS` / `CREATE OR REPLACE` |
