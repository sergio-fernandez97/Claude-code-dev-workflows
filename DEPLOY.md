# Deployed Services

After running `docker compose up --build` in the `full-stack-fastapi-template` directory, the following services are available:

| Service | Description | URL | Port |
|---------|-------------|-----|------|
| **Frontend** | React/Vue web application | http://localhost:5173 | 5173 |
| **Backend API** | FastAPI REST API | http://localhost:8000 | 8000 |
| **Backend Docs** | Swagger UI | http://localhost:8000/docs | 8000 |
| **Adminer** | Database administration UI | http://localhost:8080 | 8080 |
| **Mailcatcher** | SMTP testing server (web UI) | http://localhost:1080 | 1080 |
| **Mailcatcher SMTP** | SMTP endpoint for testing emails | localhost:1025 | 1025 |
| **Traefik Dashboard** | Reverse proxy dashboard | http://localhost:8090 | 8090 |

## Traefik Routing (if DOMAIN=localhost.tiangolo.com)

With `DOMAIN=localhost.tiangolo.com` configured in `.env`:

| Service | URL |
|---------|-----|
| Frontend | http://dashboard.localhost.tiangolo.com |
| Backend API | http://api.localhost.tiangolo.com |
| Adminer | http://adminer.localhost.tiangolo.com |

## Default Credentials

| Service | Username | Password |
|---------|----------|----------|
| PostgreSQL | postgres | changethis |
| First Superuser | admin@example.com | changethis |

## Health Checks

- **Backend**: http://localhost:8000/api/v1/utils/health-check/
- **Database**: Use `docker compose exec db pg_isready -U postgres`

## Container Names

| Service | Container Name |
|---------|----------------|
| Backend | full-stack-fastapi-project-backend-1 |
| Frontend | full-stack-fastapi-project-frontend-1 |
| PostgreSQL | full-stack-fastapi-project-db-1 |
| Adminer | full-stack-fastapi-project-adminer-1 |
| Mailcatcher | full-stack-fastapi-project-mailcatcher-1 |
| Traefik | full-stack-fastapi-project-proxy-1 |

## Useful Commands

```bash
# View logs
docker compose logs -f

# View logs for specific service
docker compose logs -f backend

# Stop all services
docker compose down

# Stop and remove volumes
docker compose down -v
```