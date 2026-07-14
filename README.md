# restful-booker-platform

A platform of web services that forms a Bed and Breakfast booking system. The platform's primary purpose is for training others on how to explore and test web service platforms as well as strategise and implement automation in testing strategies.

## Attribution

This is a modified fork of [restful-booker-platform](https://github.com/mwinteringham/restful-booker-platform) by [Mark Winteringham](https://github.com/mwinteringham), licensed under the [GNU General Public License v3.0](LICENSE). This fork remains licensed under GPL-3.0.

Modifications in this fork (2026):

* Replaced the pre-built-JAR Dockerfiles with a single parameterized multi-stage `Dockerfile.backend` that builds any Java service from source inside the container
* Reworked `docker-compose.yml` to build and run the entire stack (six backends + Next.js frontend) with one command, with no host toolchain required
* Raised the Tomcat request-header limit across backends for local development
* Rewrote this README for the Docker-based workflow

All credit for the platform itself goes to the original author.

## Requirements

- Docker (with Docker Compose v2 — included in any recent Docker Desktop or docker-ce install)

That's it. All Java, Maven, and Node tooling runs inside the build containers, so nothing else needs to be installed on your machine.

## Quick start

From the repository root:

```bash
docker compose up -d --build
```

Then open **http://localhost:8080** to access the site.

The first run takes a while — it builds six Java services and the Next.js frontend from source and downloads their dependencies. Subsequent runs are much faster thanks to Docker layer caching and a shared Maven cache.

### Login

The admin login details are:

* Username: `admin`
* Password: `password`

### Stopping

```bash
docker compose down
```

## How it's put together

| Service | Description | Host port |
|---|---|---|
| rbp-assets | Next.js frontend | 8080 |
| rbp-booking | Booking API | 3000 |
| rbp-room | Room API | 3001 |
| rbp-branding | Branding API | 3002 |
| rbp-auth | Auth API | 3004 |
| rbp-report | Report API | 3005 |
| rbp-message | Message API | 3006 |

All Java backends are built from a single parameterized `Dockerfile.backend` (multi-stage: Maven+JDK build stage, slim JRE runtime) using a `MODULE` build argument. The frontend builds from `assets/Dockerfile`. Services communicate over the compose network using their service names (`rbp-auth`, `rbp-booking`, ...).

## API documentation (Swagger)

Each backend serves its own Swagger UI and OpenAPI spec under its context path:

| Service | Swagger UI | OpenAPI spec |
|---|---|---|
| Booking | http://localhost:3000/booking/swagger-ui/index.html | http://localhost:3000/booking/v3/api-docs |
| Room | http://localhost:3001/room/swagger-ui/index.html | http://localhost:3001/room/v3/api-docs |
| Branding | http://localhost:3002/branding/swagger-ui/index.html | http://localhost:3002/branding/v3/api-docs |
| Auth | http://localhost:3004/auth/swagger-ui/index.html | http://localhost:3004/auth/v3/api-docs |
| Report | http://localhost:3005/report/swagger-ui/index.html | http://localhost:3005/report/v3/api-docs |
| Message | http://localhost:3006/message/swagger-ui/index.html | http://localhost:3006/message/v3/api-docs |

> **Note:** Swagger UI only loads correctly when the services run with the `dev` profile (the compose default). The `prod` profile points Swagger at `/api/<service>/...` paths that only exist behind the hosted deployment's reverse proxy — running `prod` locally shows "Failed to load remote configuration".

Each service also exposes a health endpoint, e.g. http://localhost:3000/booking/actuator/health.

## Common tasks

Rebuild and restart a single service after changing its code:

```bash
docker compose up -d --build rbp-booking
```

Tail logs:

```bash
docker compose logs -f rbp-booking
```

Enable Honeycomb tracing (optional) by exporting the key before starting, or putting it in a `.env` file next to the compose file:

```bash
HONEYCOMB_API_KEY=your-key docker compose up -d --build
```

## Troubleshooting

* **`400 Bad Request` in the browser but APIs work via curl** — your browser is sending oversized headers (usually accumulated `localhost` cookies). Clear cookies for localhost or use a private window.
* **Errors in the first ~30 seconds after startup** — `depends_on` orders container start, not application readiness; give the Spring services a moment to finish booting.
* **A backend can't reach `rbp-auth` / `rbp-message`** — the service was started outside compose (e.g. plain `docker run`). The service hostnames only resolve on the compose network.

## Developing without Docker (legacy)

If you prefer running the stack directly on your machine, you'll need JDK 26+, Maven 3.9.14, Node 24.14.1, and NPM 11.11.0. Then:

1. Run `bash build_locally.sh` (Linux/Mac) or `build_locally.cmd` (Windows) to build and start RBP
2. Navigate to http://localhost:3003 to access the site
3. On subsequent runs, use `run_locally.sh` / `run_locally.cmd` (append `-e true` / `true` to include end-to-end checks)

## Development

### API details

The details on running checks, building APIs and additional details on documentation for development can be found in READMEs inside each of the API folders.