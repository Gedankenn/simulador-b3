# simulador-b3

Early-stage backend for a B3 (Brazilian stock exchange) trading simulator: FastAPI + SQLAlchemy 2.0 + PostgreSQL, with Alembic migrations and Redis, run with Docker Compose.

## Current state

- **API** (`app/main.py`): FastAPI app "Simulador B3" with one endpoint, `GET /health` → `{"status": "ok"}`.
- **Data model** (`app/db/models/`, SQLAlchemy 2.0 typed `Mapped` columns, UUID primary keys):
  - `User`: email, password hash
  - `Asset`: ticker, name, tick size
  - `Order`: user, asset, side (`BUY`/`SELL`), type (`MARKET`/`LIMIT`), price, quantity, status (`OPEN`, `PARTIAL`, `FILLED`, `CANCELLED`)
  - `Trade`: order, asset, execution price, quantity, time
  - `LedgerEntry`: double-entry style ledger (`DEPOSIT`, `BUY`, `SELL`, `FEE`; `DEBIT`/`CREDIT`) with amount and reference id
- **Migrations**: Alembic, with an initial schema revision (`alembic/versions/b24391f70e90_initial_schema.py`). `alembic/env.py` reads `DATABASE_URL` from the environment.
- **Infrastructure** (`docker-compose.yml`): `api` (python:3.11-slim, installs `requirements.txt` and runs `uvicorn --reload` on port 8000), `db` (postgres:16, port 5432) and `redis` (redis:7, port 6379). Configuration comes from `.env` (`POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`, `DATABASE_URL`, `REDIS_URL`).

Order matching, trading endpoints and Redis usage are not implemented yet.

## Run

```bash
docker compose up -d                                  # api on http://localhost:8000
docker compose exec api alembic upgrade head          # create the tables
curl http://localhost:8000/health
```

FastAPI's interactive docs are served at `http://localhost:8000/docs`.

## Layout

```
app/
├── main.py            # FastAPI app
├── api/health.py      # /health router
├── core/config.py     # DATABASE_URL / REDIS_URL from the environment
└── db/
    ├── base.py        # declarative Base
    ├── session.py     # engine + SessionLocal
    └── models/        # User, Asset, Order, Trade, LedgerEntry, enums
alembic/               # migrations
docker-compose.yml
requirements.txt
```

## License

MIT, see [LICENSE](LICENSE).
