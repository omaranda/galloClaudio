---
name: query-design
description: Design and optimize SQL queries for PostgreSQL (with PostGIS) or QuestDB time-series queries.
---

You are acting as the data-manager agent. Design an optimized SQL query.

**Step 1 — Understand the query goal:**
- What data needs to be retrieved or aggregated?
- Which database? (PostgreSQL/PostGIS or QuestDB)
- What are the filter conditions? (species, date range, geographic area, confidence threshold)
- What is the expected result shape? (single row, list, time-series, aggregation)
- Performance requirements? (interactive UI query vs. background batch)

**Step 2 — Read the schema:**
Read relevant model files, migration files, or ask for table definitions before writing the query.

**Step 3 — Write the query:**

**For PostgreSQL/PostGIS queries:**
- Use CTEs for readability over deeply nested subqueries
- Use `ST_Within`, `ST_DWithin`, `ST_Intersects` for geospatial filters — leverage GiST indexes
- Use window functions for ranking and time-series aggregations
- Always filter on indexed columns first to limit the scan

**For QuestDB queries:**
- Use `SAMPLE BY` for time-series aggregation (e.g., `SAMPLE BY 1h FILL(NULL)`)
- Filter by the designated timestamp column first — this uses the partition index
- Use `LATEST ON` for last-known-value queries per entity
- Avoid JOINs in hot paths; denormalize for performance

**Step 4 — Explain and optimize:**
- Provide the query with inline comments explaining non-obvious parts
- Include an EXPLAIN ANALYZE plan for complex queries (run against real data if possible)
- Suggest indexes if the query would benefit from ones not yet present
- Note expected performance characteristics (should be < 100ms for UI, acceptable for batch)

**Step 5 — Provide a test:**
Include a verification query or sample data setup to confirm the query returns expected results.
