# Messaging App (Django + DRF)

Simple REST API for user-to-user conversations and messages.

## Features

- Custom user model [`User`](chats/models.py) with UUID primary key, email uniqueness, roles.
- Conversation model [`Conversation`](chats/models.py) with ManyToMany participants.
- Message model [`Message`](chats/models.py) linked to conversations and senders.
- Nested and flat endpoints via routers ([`chats/urls.py`](chats/urls.py)).
- Validation for participant count and non-empty message bodies in [`ConversationSerializer`](chats/serializers.py).
- Last message and message count summary fields.
- Auth: Session + Basic (see [`REST_FRAMEWORK` settings](messaging_app/settings.py)).
- Query filtering, search, ordering in viewsets ([`ConversationViewSet`](chats/views.py), [`MessageViewSet`](chats/views.py)).

## Tech Stack

- Django 3.1
- Django REST Framework
- django-filter
- SQLite (default; configurable in [`settings.py`](messaging_app/settings.py))

## Data Models

- [`User`](chats/models.py): UUID id, email, optional phone, role, mirrored `password_hash`.
- [`Conversation`](chats/models.py): UUID id, participants M2M, created_at.
- [`Message`](chats/models.py): UUID id, FK sender, FK conversation, body, timestamp, ordering by `-sent_at`.

## Serializers

- [`UserSerializer`](chats/serializers.py): write-only password; sets Django password + mirrors hash.
- [`MessageSerializer`](chats/serializers.py): includes sender detail via [`UserSerializer`](chats/serializers.py).
- [`ConversationSerializer`](chats/serializers.py): nested messages (optional), participants, computed `message_count`, `last_message`.

Validation rules:

- At least 2 participants.
- Each nested message must have non-blank `message_body`.

## Notable recent improvements

- API documentation (OpenAPI/Swagger) powered by drf-spectacular:
  - Schema: `/api/schema/`
  - Swagger UI: `/api/docs/`
  - Redoc: `/api/redoc/`
  - Config in [messaging_app/settings.py](messaging_app/settings.py) and routes in [messaging_app/urls.py](messaging_app/urls.py).
- JWT authentication added via SimpleJWT:
  - Obtain: `POST /api/token/`
  - Refresh: `POST /api/token/refresh/`
  - See [messaging_app/urls.py](messaging_app/urls.py) and `SIMPLE_JWT` in [messaging_app/settings.py](messaging_app/settings.py).
- Linting and hooks:
  - flake8 config in [.flake8](.flake8) and pre-commit hooks in [.pre-commit-config.yaml](.pre-commit-config.yaml).
- CI/CD:
  - GitHub Actions pipeline runs flake8, tests with MySQL, and uploads coverage ([.github/workflows/ci.yml](.github/workflows/ci.yml)).
- Containerization and Kubernetes:
  - Docker dev setup ([Dockerfile](Dockerfile), [docker-compose.yml](docker-compose.yml)).
  - Blue/green manifests and service/ingress ([blue_deployment.yaml](blue_deployment.yaml), [green_deployment.yaml](green_deployment.yaml), [kubeservice.yaml](kubeservice.yaml), [ingress.yaml](ingress.yaml)) with helper scripts ([kubctl-0x02](kubctl-0x02), [kubctl-0x03](kubctl-0x03), [messaging_app/kubctl-0x01](messaging_app/kubctl-0x01)).
- CORS enabled for development (see middleware and app in [messaging_app/settings.py](messaging_app/settings.py)).
- Database flexibility:
  - Uses MySQL in CI via env vars and falls back to SQLite locally (see `DB_ENGINE` block in [messaging_app/settings.py](messaging_app/settings.py)).
- Pagination defaults standardized: PageNumberPagination with page size 20 (see `REST_FRAMEWORK` in [messaging_app/settings.py](messaging_app/settings.py)).
- Nested message routes using DRF nested routers (see [chats/urls.py](chats/urls.py)).
- Postman collection and guide included ([post_man-Collections](post_man-Collections), [post_man-Collections.md](post_man-Collections.md)).

## Views / Endpoints

Base API prefix: `/api/` (see [messaging_app/urls.py](messaging_app/urls.py)).

- Swagger UI: `/api/docs/`, Redoc: `/api/redoc/`, Schema: `/api/schema/`
- JWT: `POST /api/token/`, `POST /api/token/refresh/`
- Conversations and messages routes (including nested): see [chats/urls.py](chats/urls.py)

## Quick Start

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt  # if present
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

Log in at `/api-auth/login/` for browsable API.

## Example Usage

Create a conversation with participants and initial messages:

```bash
curl -u user:pass -X POST http://localhost:8000/api/conversations/ \
  -H "Content-Type: application/json" \
  -d '{
        "participants": ["<user_uuid_1>", "<user_uuid_2>"],
        "messages": [
          {"message_body": "Hello there"},
          {"message_body": "Follow up"}
        ]
      }'
```

Send a message to an existing conversation:

```bash
curl -u user:pass -X POST http://localhost:8000/api/conversations/<conversation_uuid>/messages/ \
  -H "Content-Type: application/json" \
  -d '{"message_body": "New message"}'
```

Flat message creation:

```bash
curl -u user:pass -X POST http://localhost:8000/api/messages/ \
  -H "Content-Type: application/json" \
  -d '{"conversation": "<conversation_uuid>", "message_body": "Ping"}'
```

## Security Notes

- Development `SECRET_KEY` in [messaging_app/settings.py](messaging_app/settings.py); replace for production.
- Auth: Session, Basic, and JWT (SimpleJWT) are enabled. Rotate keys and tune lifetimes in `SIMPLE_JWT` in [messaging_app/settings.py](messaging_app/settings.py).
- Switch `USERNAME_FIELD` in [`chats.models.User`](chats/models.py) to email if email-based login is preferred.

## Testing

Placeholder test file [`tests.py`](chats/tests.py). Add API tests using `APITestCase` and `APIClient`.

## Extending

- Add pagination tuning in [`REST_FRAMEWORK`](messaging_app/settings.py).
- Register models in admin (`[`admin.py`](chats/admin.py)` currently empty).
- Add websockets (Channels) for real-time messaging if needed.

## License

MIT

## Maintenance

Run migrations on model changes:

```bash
python manage.py makemigrations
python manage.py migrate
```

Rebuild environment caches on user model alterations.

## References

- DRF docs: [https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)
- Django docs: [https://docs.djangoproject.com/](https://docs.djangoproject.com/)
