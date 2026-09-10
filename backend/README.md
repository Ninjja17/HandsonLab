# Incident Platform API

Spring Boot API boundary for the Signal Room frontend.

## Stack

- Java 21+
- Spring Boot 3.4
- PostgreSQL 16 with Flyway migrations
- Redis 7 for cache, rate limits, and live coordination
- Kafka for domain-event delivery
- Actuator for health and metrics

## API

- `GET /api/v1/incidents`
- `GET /api/v1/incidents/{incidentId}`
- `POST /api/v1/incidents`
- `PATCH /api/v1/incidents/{incidentId}`
- `POST /api/v1/incidents/{incidentId}/transitions`

Lifecycle transitions are enforced in the domain:

`OPEN -> INVESTIGATING -> MITIGATED -> CLOSED`

A closed incident can be reopened as `INVESTIGATING` within 24 hours of resolution. Invalid transitions return a structured conflict response.

## Local infrastructure

From the repository root:

```powershell
docker compose up -d
```

Then from `backend/`:

```powershell
mvn spring-boot:run
```

Maven and Docker are not currently installed in this environment, so compilation and container startup remain pending.
