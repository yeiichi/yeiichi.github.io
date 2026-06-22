---
layout: default
title: "Django Docker Deployment: DigitalOcean Droplet and Google Cloud Run"
permalink: /django-docker-deployment-droplet-cloud-run
---

# Django Docker Deployment: DigitalOcean Droplet and Google Cloud Run
_Last Updated: 2026-06-22_

## 1. Introduction

This guide provides a reusable Docker pattern for deploying a Django
application to either a DigitalOcean Droplet or Google Cloud Run from the same
multi-stage Dockerfile.

It is a reusable baseline for Django projects that follow the documented
conventions, not a universally project-agnostic Dockerfile. Adapt the project
layout, settings and WSGI module paths, system packages, and build-time
requirements to the application.

The pattern assumes:

- a `src/` application layout;
- `pyproject.toml` and `uv.lock` for dependency management;
- Gunicorn as the production application server;
- separate image targets for the Droplet and Cloud Run runtimes.

The Django project remains responsible for its application code, settings,
dependencies, environment variables, database migrations, and external
services.

---

## 2. Deployment Design

### A. Expected Project Layout

```text
my-django-project/
├── .dockerignore
├── Dockerfile
├── manage.py
├── pyproject.toml
├── uv.lock
└── src/
    ├── config/
    │   ├── settings/
    │   │   └── production.py
    │   └── wsgi.py
    ├── app_a/
    └── app_b/
```

The Docker image sets `PYTHONPATH=/app/src`, allowing Python to import
`config.wsgi` and application modules without a `src.` prefix.

### B. Application Requirements

Declare Django, Gunicorn, the database driver, and other runtime packages in
`pyproject.toml`, then commit the generated `uv.lock`.

For example:

```toml
[project]
dependencies = [
    "django",
    "gunicorn",
    "psycopg[binary]",
]
```

The production settings module must permit `collectstatic` to run during the
image build. If those settings require additional environment variables, pass
safe build-time values or use a dedicated build settings module. Never bake
production secrets into the image.

### C. Design Policy

- **Shared base image:** Install dependencies and copy application code once.
- **Runtime-specific targets:** Give each deployment platform its own command
  and port configuration.
- **Immutable dependencies:** Build from the committed `uv.lock` with
  `uv sync --frozen`.
- **Externalized configuration:** Supply secrets and environment-specific
  settings at runtime.
- **Deliberate migrations:** Run database migrations as a deployment step
  rather than automatically in every web-container replica.

---

## 3. Multi-Target Dockerfile

```dockerfile
# syntax=docker/dockerfile:1
ARG PYTHON_VERSION=3.12-slim

FROM python:${PYTHON_VERSION} AS base-core

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    UV_COMPILE_BYTECODE=1 \
    UV_PYTHON_DOWNLOADS=0 \
    PATH="/app/.venv/bin:$PATH" \
    PYTHONPATH=/app/src

WORKDIR /app

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Pin uv to a tested release or image digest for reproducible builds.
COPY --from=ghcr.io/astral-sh/uv:0.11.23 /uv /uvx /bin/

COPY pyproject.toml uv.lock ./

RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev --no-install-project

COPY . .

ARG DJANGO_SETTINGS=config.settings.production
ENV DJANGO_SETTINGS_MODULE=${DJANGO_SETTINGS}

ARG WSGI_MODULE=config.wsgi
ENV DJANGO_WSGI_MODULE=${WSGI_MODULE}

RUN SECRET_KEY=build-time-dummy-key \
    python manage.py collectstatic --noinput

FROM base-core AS droplet

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "3", "config.wsgi:application"]

FROM base-core AS cloud-run

ENV PORT=8080
EXPOSE 8080

# sh expands runtime variables; exec makes Gunicorn PID 1.
CMD ["sh", "-c", "exec gunicorn --bind 0.0.0.0:${PORT} --workers 1 --threads 8 --timeout 0 ${DJANGO_WSGI_MODULE}:application"]
```

The Droplet command uses `config.wsgi` directly. If the project uses a
different module, change the command or provide a small entrypoint script that
expands `DJANGO_WSGI_MODULE`.

---

## 4. Docker Build Context

Because the Dockerfile uses `COPY . .`, add a `.dockerignore` file. A useful
starting point is:

```dockerignore
.git
.gitignore
.venv
__pycache__/
*.py[cod]
*.sqlite3
*.env
.env*
build/
dist/
staticfiles/
media/
```

Adjust this list if the application intentionally includes any of these files.
Do not copy local virtual environments, credentials, development databases, or
user-uploaded media into the image.

---

## 5. Build Targets

### A. DigitalOcean Droplet

```bash
docker build --target droplet -t myapp-droplet:latest .
```

### B. Google Cloud Run

```bash
docker build \
    --target cloud-run \
    -t REGION-docker.pkg.dev/PROJECT/REPOSITORY/myapp:latest \
    .
```

These commands build local images. They do not deploy them.

---

## 6. Deployment Outline

### A. DigitalOcean Droplet

Push the image to a registry accessible by the Droplet, pull it on the server,
and run it with the required environment variables:

```bash
docker run --detach \
    --name myapp \
    --restart unless-stopped \
    --env-file /path/to/production.env \
    --publish 127.0.0.1:8000:8000 \
    REGISTRY/myapp-droplet:TAG
```

Place a reverse proxy such as Nginx or Caddy in front of Gunicorn for public
HTTP and HTTPS traffic.

### B. Google Cloud Run

Push the image to Artifact Registry, then deploy that registry image:

```bash
docker push REGION-docker.pkg.dev/PROJECT/REPOSITORY/myapp:TAG

gcloud run deploy SERVICE \
    --image REGION-docker.pkg.dev/PROJECT/REPOSITORY/myapp:TAG \
    --region REGION
```

Cloud Run injects the `PORT` environment variable. The container must listen
on `0.0.0.0` at that port. Configure secrets, database connectivity, service
accounts, migrations, and other deployment settings separately.

---

## 7. Runtime Tuning

The supplied Gunicorn values are starting points, not universal production
defaults:

- **Droplet workers:** Choose the worker count based on available CPU, memory,
  and workload.
- **Cloud Run concurrency:** Test worker and thread counts against instance
  concurrency, memory, database connection limits, and request behavior.
- **Worker timeout:** `--timeout 0` disables Gunicorn's worker timeout. Use it
  only when that behavior matches the application's request and shutdown
  requirements.
- **Build cache:** BuildKit's uv cache mount can speed up repeat builds when
  the builder preserves its cache. CI performance still depends on cache
  configuration, network conditions, and compiled dependencies.

---

## 8. Validation

Before publishing an image, check the Dockerfile and build both targets:

```bash
docker build --check --target droplet .
docker build --check --target cloud-run .
docker build --target droplet -t myapp-droplet:test .
docker build --target cloud-run -t myapp-cloud-run:test .
```

Then run each relevant image with representative production settings and
verify:

- Django and Gunicorn startup;
- static asset delivery;
- database connectivity;
- platform-specific port binding;
- graceful shutdown.

---

## 9. Maintenance Notes

- **Extensibility:** Add another final-stage target when a platform needs a
  different command, port, or runtime package.
- **Reproducibility:** Keep the Python base image and uv release pinned to
  tested versions, or pin image digests for stricter control.
- **Security:** Keep credentials out of image layers and supply them through
  the deployment platform's secret-management mechanism.
- **Interpretation:** Treat the Dockerfile and Gunicorn settings as a starting
  pattern that must be tested against the application's actual workload.
