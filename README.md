# Product Browser Backend

This project is a take-home backend task for browsing approximately **200,000 products**. It supports newest-first product listing, category filtering, and efficient cursor-based pagination.

---

## Live Links

### GitHub Repository

https://github.com/KomalPatil4717/Product-Browser

### Live Backend

https://product-browser.onrender.com

---

## What I Built

* A FastAPI backend to browse products.
* A product table with `id`, `name`, `category`, `price`, `created_at`, and `updated_at`.
* A seed script to generate approximately **200,000 products**.
* Fast pagination using cursor/keyset pagination.
* Category filtering.
* A simple browser UI as a bonus.
* Tests for API and pagination behavior.

---

## Tech Stack

* **Python** - Main programming language
* **FastAPI** - Backend API framework
* **SQLAlchemy** - ORM and query builder
* **SQLite** - Local development database
* **PostgreSQL** - Production database (Neon/Render/Supabase)
* **Pydantic** - Request and response validation
* **Pytest** - Automated testing
* **Uvicorn** - ASGI server

---

## Why This Approach

The primary requirement was to ensure pagination remains fast and correct even when products are inserted or updated while users are browsing.

I used **keyset (cursor-based) pagination** instead of offset pagination.

Offset pagination example:

```text
page=1000&limit=50
```

This becomes slower for large datasets because the database needs to skip many rows.

Keyset pagination uses a cursor:

```text
/products?limit=50&cursor=...
```

This is faster because the database continues from the last seen product using indexed columns.

Products are sorted by:

```text
created_at DESC, id DESC
```

This ensures newest products appear first.

I intentionally use `created_at` as the pagination order because it is immutable after creation.

`updated_at` may change when a product is modified, but it is not used for pagination.

If pagination relied on `updated_at`, records could move between pages and cause duplicates or missing items.

The cursor stores:

```text
last_created_at
last_id
snapshot_created_at
snapshot_id
```

The snapshot boundary is captured from the first page.

If new products are inserted while users browse, they are newer than the snapshot and do not suddenly appear midway through the session.

Updates do not affect `created_at`, ensuring products never jump between pages.

---

## Important Files

```text
app/main.py          FastAPI routes
app/models.py        Product database model and indexes
app/schemas.py       Request and response schemas
app/crud.py          Product create, update, filter, pagination logic
app/database.py      Database connection
app/config.py        App settings
app/utils.py         Cursor encode/decode helpers

scripts/seed.py      Product generation script

static/index.html    Simple browser UI

tests/               Automated tests

render.yaml          Render deployment config
Dockerfile           Docker deployment config
requirements.txt
README.md
```

---

## API Endpoints

### Health Check

```text
GET /health
```

Checks whether the server is running.

---

### Browse Products

```text
GET /products?limit=50&category=books&cursor=...
```

Returns products sorted newest first.

Parameters:

* `limit`
* `category`
* `cursor`

All parameters are optional.

---

### Categories

```text
GET /categories
```

Returns available categories.

---

### Create Product

```text
POST /products
```

Creates a new product.

---

### Update Product

```text
PATCH /products/{product_id}
```

Updates an existing product.

---

## Local Setup

Open PowerShell:

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

Generate assignment dataset:

```powershell
python scripts/seed.py --count 200000 --reset
```

Run server:

```powershell
python -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8001
```

Open UI:

```text
http://127.0.0.1:8001
```

Swagger Docs:

```text
http://127.0.0.1:8001/docs
```

---

## Run Tests

```powershell
python -m pytest
```

---

## Database

By default, the project uses SQLite locally.

```text
products.db
```

For production:

* Neon PostgreSQL
* Supabase
* Render PostgreSQL

Create `.env`

```env
DATABASE_URL=postgresql+psycopg://USER:PASSWORD@HOST:5432/DBNAME
```

---

## Deployment

This project includes `render.yaml` and can be deployed easily on Render.

Environment variable:

```text
DATABASE_URL
```

After deployment, run the seed script once to populate approximately **200,000 products**.

---

## Tests Covered

* Product creation
* Product listing
* Category filtering
* Cursor pagination
* Pagination stability during inserts

---

## What I Would Improve With More Time

* Alembic migrations
* Authentication
* Rate limiting
* Redis caching
* Performance benchmarking
* Deployment seed jobs

---

## AI Usage Note

I used AI tools to understand cursor pagination concepts, improve seed generation, debug deployment issues, and structure documentation.

All final code was reviewed, tested, and fully understood before submission.

---

## Author

**Komal Patil**

Backend Take-Home Assignment Submission
