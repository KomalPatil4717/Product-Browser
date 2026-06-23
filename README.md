# Product Browser Backend

This project is a take-home backend task for browsing around 200,000 products. It supports newest-first product listing, category filtering, and fast cursor-based pagination.

## What I Built

- A FastAPI backend to browse products.
- A product table with `id`, `name`, `category`, `price`, `created_at`, and `updated_at`.
- A seed script to generate 200,000 products.
- Fast pagination using cursor/keyset pagination.
- Category filtering.
- A simple browser UI as a bonus.
- Tests for API and pagination behavior.

## Tech Stack

- **Python** - main programming language.
- **FastAPI** - backend API framework.
- **SQLAlchemy** - database ORM and query builder.
- **SQLite** - default local database for development.
- **PostgreSQL** - recommended production database for Render/Neon/Supabase.
- **Pydantic** - request and response validation.
- **Pytest** - automated tests.
- **Uvicorn** - ASGI server to run FastAPI.

## Why This Approach

The main requirement is that pagination should be fast and correct even when new products are added or products are updated while someone is browsing.

I used **keyset pagination** instead of offset pagination.

Offset pagination example:

```text
page=1000&limit=50
```

This becomes slower for large data because the database has to skip many rows.

Keyset pagination uses a cursor:

```text
/products?limit=50&cursor=...
```

This is faster because the database continues from the last seen product using indexed columns.

Products are sorted by:

```text
created_at DESC, id DESC
```

This means newest products come first. I intentionally use `created_at` as the pagination order because it is immutable after creation. `updated_at` still changes when a product is updated, but it is not used to move rows between pages. If pagination were ordered by `updated_at`, an update could move a not-yet-seen product above the current cursor and cause it to be skipped, unless the system used database snapshot isolation or kept historical row versions.

The cursor stores:

```text
last_created_at
last_id
snapshot_created_at
snapshot_id
```

The snapshot boundary is captured from the first page. If new products are inserted while the user is browsing, those new rows are newer than the snapshot and do not appear midway through the same browsing session. Updates to product fields do not change `created_at`, so products do not jump between pages and the user does not see duplicates or miss rows.

## Important Files

```text
app/main.py          FastAPI routes
app/models.py        Product database model and indexes
app/schemas.py       Request and response schemas
app/crud.py          Product create, update, filter, pagination logic
app/database.py      Database connection
app/config.py        App settings
app/utils.py         Cursor encode/decode helpers
scripts/seed.py      Script to generate product data
static/index.html    Simple UI to browse products
tests/               Automated tests
render.yaml          Render deployment config
Dockerfile           Docker deployment config
```

## API Endpoints

```text
GET /health
```

Checks if the server is running.

```text
GET /products?limit=50&category=books&cursor=...
```

Returns products newest first. `category` and `cursor` are optional.

```text
GET /categories
```

Returns all available categories.

```text
POST /products
```

Creates a new product.

```text
PATCH /products/{product_id}
```

Updates an existing product.

## Local Setup

Open PowerShell in the project folder:

```powershell
cd C:\Users\komal\Desktop\product-browser
```

Install dependencies:

```powershell
python -m pip install -r requirements.txt
```

Generate sample products:

```powershell
python scripts/seed.py --count 200 --reset
```

Generate full assignment data:

```powershell
python scripts/seed.py --count 200 --reset
```

Run the server:

```powershell
python -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8001
```

Open the UI:

```text
http://127.0.0.1:8001
```

Open API docs:

```text
http://127.0.0.1:8001/docs
```

## Run Tests

```powershell
python -m pytest
```

## Database

By default, this project uses SQLite locally:

```text
products.db
```

For production, use PostgreSQL with Neon, Supabase, Render Postgres, or any hosted PostgreSQL provider.

Create a `.env` file:

```env
DATABASE_URL=postgresql+psycopg://USER:PASSWORD@HOST:5432/DBNAME
```

## Deployment

This project includes `render.yaml`, so it can be deployed on Render.

Set this environment variable on Render:

```text
DATABASE_URL
```

Use a PostgreSQL database from Neon or Supabase.

After deployment, run the seed script once to create 200,000 products.

## Tests Covered

- Product creation.
- Product listing.
- Category filtering.
- Cursor pagination.
- Pagination stability when new products are added while browsing.

## What I Would Improve With More Time

- Add Alembic migrations.
- Add authentication for write APIs.
- Add rate limiting.
- Add more performance benchmarks with a hosted PostgreSQL database.
- Add deployment seed job or admin-only seed endpoint.

## AI Usage Note

I used AI to help structure the FastAPI project, write the first version of the code, and create tests. I reviewed the implementation and verified the important behavior with tests, especially cursor pagination and stable browsing while new products are inserted.
