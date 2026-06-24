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
* A product table containing:

  * `id`
  * `name`
  * `category`
  * `price`
  * `created_at`
  * `updated_at`
* A seed script to generate approximately **200,000 products**.
* Fast cursor-based (keyset) pagination.
* Category filtering.
* A simple browser UI as a bonus.
* Automated tests for API behavior and pagination stability.

---

## Tech Stack

* **Python** – Main programming language
* **FastAPI** – Backend API framework
* **SQLAlchemy** – ORM and query builder
* **SQLite** – Local development database
* **PostgreSQL** – Production database (Neon / Render / Supabase)
* **Pydantic** – Request and response validation
* **Pytest** – Automated testing
* **Uvicorn** – ASGI server

---

## Why This Approach

The primary requirement was to ensure pagination remains fast and correct even when products are inserted or updated while users are browsing.

I implemented **keyset (cursor-based) pagination** instead of traditional offset pagination.

Offset pagination example:

```text
page=1000&limit=50
```

For large datasets, offset pagination becomes slower because the database needs to skip many rows.

Cursor pagination uses:

```text
/products?limit=50&cursor=...
```

This approach is faster because queries continue from the last retrieved record using indexed columns.

Products are sorted by:

```text
created_at DESC, id DESC
```

This guarantees newest products appear first.

The project intentionally uses `created_at` as the pagination key because it never changes after insertion.

The cursor stores:

```text
last_created_at
last_id
snapshot_created_at
snapshot_id
```

A snapshot boundary is captured from the first page.

Newly inserted products do not suddenly appear in the middle of an active browsing session, preventing duplicates and missing records.

---

## Important Files

```text
app/main.py          FastAPI routes
app/models.py        Product model and indexes
app/schemas.py       Request and response schemas
app/crud.py          CRUD and pagination logic
app/database.py      Database connection
app/config.py        Application settings
app/utils.py         Cursor helpers

scripts/seed.py      Product generation script

static/index.html    Simple browser UI

tests/               Automated tests

render.yaml          Render deployment configuration
Dockerfile           Docker deployment configuration

requirements.txt
README.md
```

---

## API Endpoints

### Health Check

```text
GET /health
```

Checks whether the application is running.

---

### Browse Products

```text
GET /products?limit=50&category=books&cursor=...
```

Parameters:

* `limit`
* `category`
* `cursor`

All parameters are optional.

Returns products sorted newest first.

---

### Categories

```text
GET /categories
```

Returns all available categories.

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

Clone the repository:

```bash
git clone https://github.com/KomalPatil4717/Product-Browser.git

cd Product-Browser
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Generate sample data:

```bash
python scripts/seed.py --count 200 --reset
```

Generate assignment dataset:

```bash
python scripts/seed.py --count 200000 --reset
```

Run the application:

```bash
uvicorn app.main:app --reload
```

Open the UI:

```text
http://127.0.0.1:8001
```

Swagger Documentation:

```text
http://127.0.0.1:8001/docs
```

---

## Run Tests

```bash
pytest
```

---

## Database

By default, the project uses SQLite locally:

```text
products.db
```

For production environments, PostgreSQL can be used with:

* Neon
* Supabase
* Render PostgreSQL

Example `.env` file:

```env
DATABASE_URL=postgresql+psycopg://USER:PASSWORD@HOST:5432/DBNAME
```

---

## Deployment

The project includes a `render.yaml` file and can be deployed directly on Render.

Required environment variable:

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

## Future Improvements

* Alembic migrations
* Authentication
* Rate limiting
* Redis caching
* Performance benchmarking
* Deployment seed jobs

---

## AI Usage Note

AI tools were used for research, deployment debugging, and documentation improvements.

All final code was reviewed, tested, and fully understood before submission.

---

## Author

**Komal Patil**

Backend Take-Home Assignment Submission
