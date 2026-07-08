---
layout: default
title: "Django Infra Utils for Cloud Run"
permalink: /dj-infra-utils-cloud-run
---

# Django Infra Utils for Cloud Run
_Last Updated: 2026-07-08_

## 1. Introduction

[`dj-infra-utils-cloud-run`](https://github.com/yeiichi/dj-infra-utils-cloud-run)
is a small copy-in Django cassette for deployment helpers I tend to need when
running Django on Google Cloud Run.

The first helper is `setup_admin`, an idempotent management command that creates
the initial Django superuser from environment variables. It is meant for a
Cloud Run Job or another one-off management task, not the normal web service
startup path.

---

## 2. Cassette Shape

This is intentionally not a Python package yet. The intended use is to copy the
`infra_utils/` directory into a Django project and add it to `INSTALLED_APPS`.

```python
INSTALLED_APPS = [
    # ...
    "infra_utils",
]
```

---

## 3. Deployment Note

For Cloud Run deployments, run migrations and admin bootstrap as a one-off job
using the same image and environment as the web service.

```shell
python manage.py migrate
python manage.py setup_admin
```

`DJANGO_SUPERUSER_PASSWORD` should come from Google Secret Manager or an
equivalent secret source, not from source code or image layers.

---

## 4. Maintenance Notes

- **Scope:** keep the cassette small and copyable.
- **Packaging:** consider PyPI only if reuse across projects becomes frequent.
- **Runtime:** keep operational commands outside the web service startup path.
