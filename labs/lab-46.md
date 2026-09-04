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
