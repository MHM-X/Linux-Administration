# Lab 46: "Luxor": PostgreSQL analytics queries crawl

## Description
 Our analytics API serves customer sales counts from a PostgreSQL database. Requests to the customer lookup endpoint take several seconds and often time out, even though the database server looks healthy — no obvious CPU, memory, or disk exhaustion.

The API runs as systemd service sad-analytics-api on port 9090. Application code lives under /opt/sad/. Database credentials for debugging: connect as saduser to database analytics_db (password: sadpassword). The main table is sales_data.

Find why customer lookups are slow and restore acceptable API response times.

Test: This API request must complete quickly (total response time under 500 ms):
curl -s -w "\nTotal time: %{time_total}s\n" http://127.0.0.1:9090/api/customers/1234/sales/count

🔗 **Lab Link:** [SadServers - "Luxor": PostgreSQL analytics queries crawl](https://sadservers.com/scenario/luxor)

<br>

## 🪜 Steps

## Lab Description

The analytics API provides customer sales counts from a PostgreSQL database.

The API was responding very slowly to customer lookup requests, often taking several seconds and sometimes timing out, even though the database server showed no obvious CPU, memory, or disk exhaustion.

The goal was to identify why the database query was slow and restore the API response time to under 500 ms.

---

## Architecture

```text
Client
   │
   │ HTTP request
   ▼
sad-analytics-api :9090
   │
   │ SQL query
   ▼
PostgreSQL
   │
   ▼
sales_data
```

The API performs a query similar to:

```sql
SELECT COUNT(*)
FROM sales_data
WHERE customer_id = 1234;
```

---

## Investigation

First, verify that the API service is running:

```bash
systemctl status sad-analytics-api
```

Then measure the API response time:

```bash
curl -s -w "\nTotal time: %{time_total}s\n" \
http://127.0.0.1:9090/api/customers/1234/sales/count
```

The request was taking several seconds.

To determine whether the problem was inside the API or the database, connect directly to PostgreSQL:

```bash
PGPASSWORD=sadpassword psql -U saduser -d analytics_db
```

Run the customer lookup query directly:

```sql
SELECT COUNT(*)
FROM sales_data
WHERE customer_id = 1234;
```

The query itself was slow, indicating that the bottleneck was in PostgreSQL rather than the HTTP or Python layer.

---

## Query Plan

Use `EXPLAIN ANALYZE` to see how PostgreSQL executes the query:

```sql
EXPLAIN ANALYZE
SELECT COUNT(*)
FROM sales_data
WHERE customer_id = 1234;
```

The query was performing a sequential scan (`Seq Scan`) over the `sales_data` table.

A sequential scan means PostgreSQL has to examine a large number of rows to find rows matching:

```text
customer_id = 1234
```

This becomes expensive when the table contains a large amount of data.

---

## Root Cause

The `customer_id` column did not have an appropriate index.

Without an index:

```text
Query
  │
  ▼
Sequential Scan
  │
  ▼
Scan many rows
  │
  ▼
Find matching customer_id values
  │
  ▼
COUNT(*)
```

This caused the customer lookup to take several seconds.

---

## Solution

Create an index on `customer_id`:

```bash
sudo -u postgres psql -d analytics_db -c \
"CREATE INDEX idx_customer_id ON sales_data (customer_id);"
```

The index allows PostgreSQL to locate rows for a specific customer much more efficiently instead of scanning the entire table.

The query can now use the index:

```text
Query
  │
  ▼
idx_customer_id
  │
  ▼
Find matching rows
  │
  ▼
COUNT(*)
```

---

## Verification

Test the API again:

```bash
curl -s -w "\nTotal time: %{time_total}s\n" \
http://127.0.0.1:9090/api/customers/1234/sales/count
```

The response should complete in under 500 ms.

---

## Key Lessons

* Slow API responses can be caused by slow database queries rather than the HTTP or application layer.
* `EXPLAIN ANALYZE` is a key tool for diagnosing PostgreSQL query performance.
* `Seq Scan` can become expensive when filtering a large table.
* Indexes provide efficient lookup structures for frequently filtered columns.
* An index should be created on columns commonly used in `WHERE`, `JOIN`, or similar lookup conditions.
* Database performance problems should be diagnosed from the query execution plan before changing application code or database configuration.
