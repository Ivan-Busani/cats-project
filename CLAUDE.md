# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This is a Git monorepo using submodules. Each submodule is an independent repo under `Ivan-Busani/` on GitHub, implementing the same "cats" API in a different language/framework:

| Submodule | Stack | Port |
|---|---|---|
| `cats-go-api` | Go + GORM | 8001 |
| `cats-java-api` | Java 25 + Spring Boot | 8002 |
| `cats-node-api` | TypeScript + Express + Prisma | 8003 |
| `cats-python-api` | Python + Django REST Framework | 8004 |
| `cats-kotlin-api` | Kotlin + Ktor | 8005 |
| `cats-android-app` | Kotlin + Jetpack Compose | — |
| `cats-web-app` | Next.js 16 + React 19 + Mantine | — |

All backend services share a single PostgreSQL 15 database managed via Docker Compose.

## Environment Setup

Copy `.env.example` to `.env` and fill in values before running anything. The `DATABASE_URL` for the Node API must be constructed manually from the Postgres vars.

## Running Everything

```bash
# Start all services + database
docker compose up

# Start a single service
docker compose up cats_go_api
```

## Per-Service Commands

### cats-go-api
```bash
cd cats-go-api
go run ./cmd/server          # run locally
go build ./...               # build
```

### cats-java-api
```bash
cd cats-java-api
./gradlew bootRun            # run locally
./gradlew build              # build
./gradlew test               # test
```

### cats-kotlin-api
```bash
cd cats-kotlin-api
./gradlew run                # run locally
./gradlew build              # build
./gradlew test               # test
./gradlew buildFatJar        # build fat JAR for Docker
./gradlew buildImage         # build Docker image
```

### cats-node-api
```bash
cd cats-node-api
npm install                  # also runs prisma generate
npm run dev                  # run locally with nodemon
npm run build                # TypeScript compilation
npm start                    # run compiled output
```

### cats-python-api
```bash
cd cats-python-api
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver 0.0.0.0:8004
```

### cats-web-app
```bash
cd cats-web-app
npm install
npm run dev -- --webpack     # dev server
npm run build                # production build
npm run lint                 # ESLint
```

### cats-android-app
```bash
cd cats-android-app
./gradlew build              # build
./gradlew test               # unit tests
./gradlew connectedAndroidTest  # instrumented tests (requires device/emulator)
```

## Architecture Notes

**Shared database pattern:** All backend APIs connect to the same PostgreSQL instance (`cats_database`). Each service manages its own schema/migrations independently. In Docker, services wait for `pg_isready` before starting.

**Go API layout:** Follows `cmd/server` entrypoint + `internal/` with handler → service → repository layers.

**Java/Kotlin APIs:** Standard layered architecture (controller/handler → service → repository), built with Gradle. Java uses Spring Boot; Kotlin uses Ktor with Kotlin Serialization.

**Node API:** Uses Prisma ORM — run `prisma generate` (done automatically on `npm install`) and `prisma migrate` before first run.

**Android app:** Jetpack Compose + Material3. Navigation is defined in `navigation/` package. `MainActivity.kt` sets up the scaffold (top bar, bottom nav, FAB). Uses Coil3 for async image loading, Retrofit for HTTP, Room for local database.

**Web app:** Next.js App Router with Mantine component library and Tailwind CSS. Run dev with `--webpack` flag (not Turbopack).

**Submodule workflow:** When working in a submodule, commits are made inside that submodule's repo. After committing inside a submodule, update the pointer in the root repo with a separate commit.
