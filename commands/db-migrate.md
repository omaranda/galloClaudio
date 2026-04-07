---
name: db-migrate
description: Design and generate a safe, reversible database migration for PostgreSQL using Alembic or Django migrations, following best-practice data modeling standards.
---

You are acting as the data-manager agent. Design and generate a database migration.

**Step 1 — Gather requirements (ask if not provided):**
- What change is needed? (new table, new column, index, constraint, rename, drop)
- Which database and migration system? (PostgreSQL/Alembic or PostgreSQL/Django)
- Is this for a relational entity or time-series data? (if time-series → recommend QuestDB)
- Are there any foreign key relationships?
- Expected data volume? (affects index and partition decisions)

**Step 2 — Read existing schema:**
- Read relevant `models.py` or SQLAlchemy model files
- Read the latest migration file to understand current schema state
- Check for any existing indexes or constraints on related tables

**Step 3 — Design the migration:**
- Design schema changes with proper types, constraints, and indexes
- Plan for PostGIS geometry columns if spatial data is involved
- Identify indexes needed based on query patterns
- Consider backward compatibility (can existing code run against new schema?)

**Step 4 — Generate migration file:**

For **Alembic**:
```python
def upgrade() -> None:
    # All changes here

def downgrade() -> None:
    # Exact reversal here — this is required
```

For **Django**:
- Auto-generated migration with `makemigrations` instructions
- Manual RunSQL for complex cases

**Step 5 — Safety checklist:**
- [ ] `downgrade()` / backward migration is complete and correct
- [ ] No data-destructive operations without explicit confirmation
- [ ] `NOT NULL` columns have a default or are added nullable then constrained
- [ ] Indexes created `CONCURRENTLY` on large existing tables (note: requires separate transaction)
- [ ] Foreign key constraints added after data is valid
- [ ] Table and column comments added

Provide the complete migration file and a rollback verification query.
