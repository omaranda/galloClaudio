---
name: scaffold-api
description: Scaffold a complete new Django REST or FastAPI endpoint, including model, serializer, view, URL routing, permissions, and tests.
---

You are acting as the web-developer agent. Scaffold a complete new API endpoint.

**Step 1 — Gather requirements (ask if not provided):**
- What resource is this endpoint for? (e.g., species, deployment, detection)
- What HTTP methods are needed? (GET list, GET detail, POST, PUT, PATCH, DELETE)
- What authentication/permissions are required? (Auth0 JWT, public, admin only)
- Which Django app or FastAPI router should it belong to?
- Any filtering, search, or ordering fields?

**Step 2 — Read existing code before writing:**
- Read the relevant app's `models.py`, `serializers.py`, `views.py`, `urls.py`
- Follow the exact patterns already established (ViewSets vs APIViews, naming conventions)

**Step 3 — Generate all required files:**
1. **Model** (if new): migration-ready Django model or SQLAlchemy model
2. **Serializer**: with validation, nested relations if needed
3. **View/ViewSet**: with proper queryset, permissions, and filter backends
4. **URL**: registered in the correct router or urlpatterns
5. **Migration**: if a new model was added
6. **Tests**: happy path + auth failure + edge cases

**Step 4 — Security checklist before finishing:**
- [ ] Authentication check on all methods
- [ ] Object-level permissions (user can only access their own data?)
- [ ] Input validation at serializer level
- [ ] No raw SQL with user input
- [ ] Sensitive fields excluded from serializer output

Provide complete file contents, not snippets. Note any breaking changes.
