---
layout: default
title: "CI/CD Cheat Sheet: GitHub Actions-Centered Flow"
permalink: /ci-cd-github-actions-centered-cheat-sheet/
---

# CI/CD Cheat Sheet: GitHub Actions-Centered Flow
_Last Updated: 2026-07-02_

## 1. Introduction

This cheat sheet summarizes a GitHub Actions-centered CI/CD strategy for projects that use `uv` and, where applicable, Python Semantic Release.

The core rule is simple: **GitHub is the source of truth, GitHub Actions is the orchestrator, and deployment/publishing targets consume artifacts or repository events.** When there is a choice between a target-driven flow and a GitHub-driven flow, prefer the GitHub-driven flow.

---

## 2. Data Schema

### A. Column Definitions

* **Target**: The destination or consumer of the project output, such as PyPI, Cloud Run, or Read the Docs.
* **GitHub Actions Role**: What GitHub Actions should do for that target.
* **Release / Trigger**: The event that should start the workflow, such as push, pull request, tag, or Semantic Release decision.
* **Notes**: Important assumptions, tradeoffs, or preferred implementation details.

---

### B. Design Policy

* **GitHub Actions-centered:** Prefer GitHub Actions whenever there is a choice between GitHub-driven and target-driven automation.
* **`uv` by default:** Use `uv` for dependency sync, test setup, package builds, and container build workflows where practical.
* **Python Semantic Release when applicable:** Use Python Semantic Release for packages or apps that benefit from conventional commits, automated versioning, changelog generation, tags, and releases.
* **Targets do not own the source:** PyPI, Cloud Run, and Read the Docs should not be treated as the source of truth.
* **Trusted Publishing preferred:** For PyPI, prefer GitHub Actions plus PyPI Trusted Publishing instead of long-lived PyPI API tokens.
* **Containers are deployment artifacts:** For Cloud Run, build and deploy containers from GitHub Actions unless there is a strong reason to delegate builds to Google Cloud Build.

---

## 3. Target Flow Cheat Sheet

| Target | GitHub Actions Role | Release / Trigger | Notes |
|---|---|---|---|
| General CI | Install `uv`, sync dependencies, run tests, lint, type checks, and build checks | Push and pull request | This is the baseline workflow for both packages and apps. |
| PyPI package | Build `dist/*` and publish to PyPI | Prefer Semantic Release-generated tag/release | PyPI does not pull code from GitHub. It receives built artifacts uploaded by the publisher. Use Trusted Publishing where possible. |
| Python Semantic Release | Decide next version, update changelog, create commit/tag/GitHub release | Usually push to main after conventional commits | Best for reusable packages and versioned apps. Use only when automated semantic versioning adds value. |
| Read the Docs | Optional: run docs checks before merge | RTD watches GitHub push/tag/PR depending on RTD settings | RTD consumes the repo event. GitHub Actions can still validate docs before RTD builds them. |
| Cloud Run | Build container image, push image, deploy to Cloud Run | Push to deployment branch, manual dispatch, or release tag | Prefer GitHub Actions-driven deployment. Cloud Build-driven deployment is acceptable but not the default for this strategy. |
| Docker / container apps | Use `uv` in the Dockerfile or build pipeline for reproducible Python setup | App deployment workflow | Keep runtime image lean. Use `gunicorn` or another production server for Django production containers, not Django `runserver`. |
| GitHub Pages docs / notes | Build or copy Markdown/static pages as needed | Push to docs branch or main, depending on Pages settings | Good for lightweight project memory, cheat sheets, and public notes. |
{: .table-sticky-scope .table-scroll }

---

## 4. Recommended Default Workflows

### A. Python Package / PyPI Project

```text
GitHub repo
  -> GitHub Actions CI
  -> uv sync / tests / lint / build check
  -> Python Semantic Release
  -> version bump / changelog / tag / GitHub release
  -> uv build
  -> publish dist/* to PyPI via Trusted Publishing
  -> RTD builds docs from repo event or tag
```

Use this for projects like reusable Python packages, CLI tools, Django helper packages, and anything intended for PyPI.

### B. Django / Cloud Run Web App

```text
GitHub repo
  -> GitHub Actions CI
  -> uv sync / tests / deploy checks
  -> container build
  -> push image
  -> deploy to Cloud Run
```

Use Python Semantic Release only if the web app needs formal versions, release notes, or versioned container tags.

### C. Documentation / GitHub Pages Memory

```text
GitHub repo
  -> Markdown files under docs/ or GitHub Pages source directory
  -> optional GitHub Actions validation
  -> GitHub Pages or RTD publishes docs
```

Use this for durable external memory: architecture notes, CI/CD policy, project conventions, WBS notes, and operational cheat sheets.

---

## 5. Mental Model

```text
GitHub owns the code.
GitHub Actions checks and orchestrates the flow.
uv standardizes Python installation, dependency sync, tests, and builds.
Python Semantic Release decides versions and release artifacts when useful.
PyPI receives packages.
Cloud Run receives deployed containers.
Read the Docs watches repository events and builds documentation.
GitHub Pages publishes lightweight project memory.
```

---

## 6. Maintenance Notes

- **Extensibility:** Add new targets such as Docker Hub, Artifact Registry, TestPyPI, or npm as additional rows in the target table. Keep GitHub Actions as the default orchestrator unless a target-native workflow is clearly better.
- **Generation:** This page can be generated from a small JSON/YAML registry of targets, triggers, and policies if the table grows.
- **Interpretation:** This is a strategy cheat sheet, not a full workflow implementation. Actual `.github/workflows/*.yml`, `pyproject.toml`, `.readthedocs.yaml`, and Dockerfile details should live in each project repository.
- **Security:** Prefer OIDC/Trusted Publishing and short-lived credentials. Avoid long-lived tokens where a trusted publisher or workload identity option exists.
- **Naming:** Use clear workflow names such as `ci.yml`, `release.yml`, `publish.yml`, `deploy-cloud-run.yml`, and `docs.yml`.
