# test-manager-library-3145556-infra

> **Technical test · ADSO 3145556** — *Manager Library* infrastructure: containers, network, volumes and an **empty** database engine.

## One command (RNF-5, T-20)

Clone the six repos side by side, then:

```bash
cd test-manager-library-3145556-infra
cp .env.example .env        # set DB_PASSWORD, JWT_SECRET (openssl rand -hex 32) and ADMIN_PASSWORD
docker compose up --build
```

| Service | URL | What it is |
|---|---|---|
| `db` | internal | PostgreSQL 16, starts with **no tables** |
| `api` | http://localhost:8000/api/v1 | Laravel API; on start it runs the migrations and creates the admin from `.env` |
| `app` | http://localhost:5173 | React frontend |
| `page` | http://localhost:8080 | Static public site (does not call the API) |

Demo data (T-21), through the API:

```bash
docker compose --profile seed up --build seed
```

The `seed` profile adds `api-past`, a second API instance whose `Clock` port is shifted 20 days back.
The seeder creates the loans that must end up overdue through that instance, so overdue data exists without SQL
and without any backdoor endpoint (see `docs/laravel-decisions`, trap 6).

## Rules this repo follows

- **No DDL here** (ADR-001, constitution art. V): the schema belongs to the API migrations.
  Check: `grep -ri "create table" .` returns nothing.
- Secrets only in `.env`, which is git-ignored; only `.env.example` is versioned (art. IX).
- Two networks: `backend` (db, api) and `frontend` (api, app, page). The database is never published to the host.
- Volumes: `db-data` for PostgreSQL, `covers` for cover binaries (the database only stores the opaque key).
