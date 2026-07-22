# Endpoints Design Practices

The best practice is neither "backend dictates everything" nor "frontend dictates everything." The most maintainable approach is to define an API contract first, then let both teams implement against that contract.

In practice, there are three common approaches:

1. Backend-first

* Backend developers design the API.
* Frontend consumes whatever the backend exposes.

```text
Frontend --> adapts to --> Backend API
```

**Pros**

* Faster for backend-heavy projects.
* Easier to maintain database and business logic.

**Cons**

* Frontend may require multiple API calls.
* Payloads can become difficult for UI developers to work with.

---

2. Frontend-first

* Frontend designs the expected payload.
* Backend implements endpoints to satisfy UI requirements.

```text
Backend <-- adapts to <-- Frontend requirements
```

**Pros**

* Better user experience.
* Optimized API responses.

**Cons**

* Frontend may request data that doesn't align with business/domain models.
* Can lead to duplicated or overly specialized endpoints.

---

3. API Contract First (Recommended)
   Before development:

```yaml
POST /api/jobs/

Request:
{
    "title": "Software Engineer",
    "department": 1,
    "location": "Nairobi",
    "extra_questions": [
        {
            "key": "github",
            "label": "Github Profile",
            "type": "url"
        }
    ]
}

Response:
{
    "id": 100,
    "title": "Software Engineer",
    "department_name": "Engineering",
    "location": "Nairobi"
}
```

Both frontend and backend agree on:

* Endpoint URLs.
* Request payloads.
* Response payloads.
* Validation rules.
* Error messages.
* Authentication requirements.

Then:

* Frontend builds forms.
* Backend builds serializers/views.
* QA tests against the agreed specification.

---

For enterprise applications (like your Django debt collection/CRM system), I would recommend:

```text
1. UI/UX designs.
2. API contract definition (Swagger/OpenAPI).
3. Backend implementation.
4. Frontend implementation.
5. Integration testing.
```

A good API should be:

* Predictable.
* Consistent.
* Versioned.
* Independent of database structure.
* Optimized for client consumption.

For example, avoid making the frontend do this:

```javascript
response.company.department.name
```

when you can return:

```json
{
    "department": {
        "id": 1,
        "name": "Engineering"
    }
}
```

or even:

```json
{
    "department_id": 1,
    "department_name": "Engineering"
}
```

depending on your standards.

For your current Django + Vue projects, I would suggest:

* Define all endpoints in OpenAPI/Swagger first.
* Review payloads with frontend developers.
* Build serializers to match the agreed contract.
* Never expose models directly.
* Keep request/response formats stable.

The endpoint should ultimately serve the business use case, not the database model and not the UI implementation alone. The API contract becomes the source of truth that both frontend and backend teams follow.
