# Day 12 — Docker Compose

A containerized Flask application orchestrated using Docker Compose with PostgreSQL and Redis.

## Project Overview

This project demonstrates how to run a multi-container application using Docker Compose. It includes a Flask web application, a PostgreSQL database for persistent visit records, and Redis for tracking request counts.

## Architecture

See [`architecture.mmd`](architecture.mmd) for the architecture diagram.

### Services

| Service | Image / Build | Purpose |
|---|---|---|
| Web | Built from `./app` | Flask application on port 5000 |
| PostgreSQL | `postgres:16-alpine` | Stores visit records |
| Redis | `redis:7-alpine` | Tracks request hits |

## Features

- Multi-container orchestration with Docker Compose
- PostgreSQL named volume for persistent database storage
- Redis request counter
- Health checks for PostgreSQL and Redis
- Web service waits for healthy dependencies
- `unless-stopped` restart policy
- Environment variables configured through a local `.env` file
- Non-root user in the web container
- Flask health endpoint at `/health`

## Project Structure

```text
day12-docker-compose/
├── app/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
├── evidence/
│   ├── app-response.png
│   ├── compose-status.png
│   └── persistence-test.png
├── .env.example
├── .gitignore
├── architecture.mmd
├── compose.yaml
└── README.md
```

## Prerequisites

- Docker Engine
- Docker Compose plugin

## Setup

1. Clone this repository.
2. Create a local `.env` file using `.env.example` as a template.
3. Set your local PostgreSQL password in `.env`.
4. Start the application:

```bash
docker compose up -d --build
```

Check service status:

```bash
docker compose ps
```

View web logs:

```bash
docker compose logs --tail=50 web
```

## Testing

Open the application at `http://localhost:5000` or run:

```bash
curl http://localhost:5000/
curl http://localhost:5000/health
```

The root endpoint displays Redis hits and PostgreSQL visit counts. Repeated requests increase both counters.

The health endpoint returns `ok` with HTTP 200 when the web application is responding.

## Persistence Test

The PostgreSQL data is stored in the named `pgdata` volume. To test persistence:

```bash
docker compose down
docker compose up -d
curl http://localhost:5000/
```

PostgreSQL visit records persist after container recreation because the named volume is retained. Redis data may reset because no Redis persistence volume is configured.

**Warning:** Do not use `docker compose down -v` unless you intentionally want to remove the Compose-managed volume and its stored database data.

## Environment and Security

- `.env` contains local credentials and must not be committed.
- `.env.example` documents the required environment variables without the real local password.
- Change the example password before running the project.
- The Flask development server is intended for this learning exercise, not production deployment.

## Key Learning Outcomes

- Orchestrating multiple services with Docker Compose
- Service-to-service communication through Compose DNS
- Configuring health checks and startup dependencies
- Persisting PostgreSQL data with named volumes
- Managing local environment variables
- Understanding container restart policies

## Evidence

The `evidence/` directory contains screenshots of the application response, Compose service status, and persistence testing.

## Interview Questions

**1. What is Docker Compose?**  
A tool for defining and running multi-container applications from a YAML configuration file.

**2. Why use `depends_on` with `service_healthy`?**  
It allows the dependent service to wait until the configured dependency health check reports healthy.

**3. Why use a named volume for PostgreSQL?**  
It preserves database data independently of the database container's lifecycle.

**4. How do containers communicate in Compose?**  
Services on the same Compose network can use service names such as `db` and `redis` as DNS hostnames.

**5. What does `restart: unless-stopped` mean?**  
Docker restarts the container after qualifying failures unless it has been explicitly stopped.

**6. What is the difference between `docker compose down` and `docker compose down -v`?**  
The first removes the Compose containers and network while retaining named volumes. The second also removes Compose-managed named volumes.

## Conclusion

This project demonstrates a practical Docker Compose workflow using Flask, PostgreSQL, and Redis, with health checks, environment configuration, restart policies, and persistent database storage.
