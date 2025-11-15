# Design Decisions

This document outlines the "why" behind the technical choices made for this project, the trade-offs considered, and future improvements, as required by the assignment.

## 1. Why These Approaches Were Chosen

* *Gunicorn:* The Dockerfile uses gunicorn as the production web server (WSGI) for the Python app. This is a battle-hardened, production-ready server, unlike Flask's built-in server which is only for development.
* *Nginx for Reverse Proxy:* Nginx handles SSL termination and request proxying. This is a standard production pattern that separates concerns: Nginx is excellent at handling web traffic and SSL, letting Gunicorn/Flask focus only on application logic.
* *Multi-Stage Dockerfile:* The Dockerfile uses a multi-stage build. This results in a smaller, more secure final image that doesn't contain build-time tools or caches.
* *Docker Compose Overrides:* We use a base docker-compose.yml to define the services, and environment-specific files (docker-compose.dev.yml, etc.) to override it. This is a clean, scalable pattern that avoids one giant, confusing file and satisfies the "same docker-compose file" requirement.
* **docker-compose.dev.yml:** This file is crucial. It overrides the CMD from the Dockerfile to run python app.py (which uses Flask's debug server) and mounts the local code. This enables the hot-reloading needed for development.
* *.env Files:* All configuration (ports, log levels, DB credentials) is kept out of the code and managed in .env files, following the 12-Factor App methodology.

## 2. Trade-offs Considered

* *Hot Reload vs. Production:* The dev environment runs a different server (Flask's debug server) than production (Gunicorn). This is a standard trade-off for developer speed, but it's not a 100% production-parity environment.
* *Docker Compose vs. Kubernetes:* Docker Compose is perfect for local development and simple single-server deployments. For a true, scalable production environment, this application would be packaged using Helm and deployed to a Kubernetes cluster for orchestration, auto-scaling, and rolling updates.
* *CI Package Installs:* The CI pipeline installs Python packages manually (pip install Flask...). This was a debugging step to bypass persistent ModuleNotFoundError errors caused by a Git history mismatch on the requirements.txt file. The trade-off is that the CI pipeline is not testing the requirements.txt file itself.

## 3. What to Improve With More Time

1.  *Observability (Bonus):* I would implement the bonus challenges by adding structured JSON logging, adding a /metrics endpoint with Prometheus metrics, and running Prometheus/Grafana in Docker Compose.
2.  *Database Migrations:* I would add a migration tool (like alembic) to manage database schema changes in an automated, version-controlled way. This is critical for production.
3.  *Fix CI/Git History:* I would fix the underlying Git history issue (likely by resetting the main branch) so the CI pipeline can reliably use pip install -r requirements.txt instead of manual installs.
