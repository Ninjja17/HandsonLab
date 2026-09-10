# Signal Room Scalable Migration Plan

## Goal

Migrate the frontend-first incident response demo to a production-oriented architecture without changing the locked command-center experience or losing the working challenge workflows.

## Target Architecture

- **Web:** Next.js App Router, TypeScript, TanStack Query for server state, Zustand for workspace state.
- **API:** Java 21+ Spring Boot modular monolith with versioned REST endpoints.
- **Database:** PostgreSQL 16 as the authoritative store, managed with Flyway.
- **Coordination:** Redis for cache, rate limits, locks, and live-update fanout.
- **Events:** Kafka behind an outbox publisher and idempotent consumers.
- **Search/reporting:** Elasticsearch as a read projection only, added after operational query needs are measured.
- **Observability:** Spring Boot Actuator and OpenTelemetry traces, metrics, and structured logs.
- **Security:** OIDC/OAuth2, tenant boundaries, RBAC, signed webhooks, least-privilege scopes, and immutable audit history.

## Migration Rules

- Preserve the approved frontend visual identity and interaction model.
- Keep PostgreSQL as the single source of truth.
- Keep domain rules in the backend, not in controllers or UI components.
- Use additive API changes and version endpoints under `/api/v1`.
- Keep a local fallback only while the API is unavailable; make the fallback explicit and observable.
- Do not introduce Kafka or Elasticsearch into request paths until the transactional PostgreSQL flow is stable.
- Validate every phase before moving to the next one.

## Phases

### Phase 0 - Baseline and Contracts

**Status:** Complete

- [x] Lock the current command-center frontend visual design.
- [x] Capture incident, RCA, SLA, timeline, ownership, and related-incident requirements.
- [x] Define the backend package and infrastructure boundaries.
- [x] Create this migration plan.

**Exit gate:** Existing frontend build passes and the locked UI remains unchanged.

### Phase 1 - Spring Boot Incident API

**Status:** In progress

- [x] Create Spring Boot project metadata and dependency set.
- [x] Add PostgreSQL/Flyway schema for incidents, timeline, and outbox.
- [x] Add incident domain entity and lifecycle state machine.
- [x] Enforce 24-hour reopen behavior in the domain.
- [x] Add REST endpoints for list, detail, create, update, and transitions.
- [x] Add structured not-found and invalid-operation responses.
- [x] Add CORS and Actuator configuration.
- [ ] Add timeline persistence and outbox writes in the application service.
- [ ] Add API integration tests with Testcontainers.
- [ ] Compile and run the service after Maven and Docker are available.

**Exit gate:** API integration tests prove CRUD, valid transitions, invalid transitions, and reopen expiry.

### Phase 2 - Frontend API Adapter

**Status:** Next

- [ ] Add a typed API client for `/api/v1/incidents`.
- [ ] Add runtime API configuration through `NEXT_PUBLIC_API_URL`.
- [ ] Add TanStack Query provider and incident query/mutation hooks.
- [ ] Switch reads to the API when configured.
- [ ] Switch create, owner/severity updates, transitions, and escalation writes to the API.
- [ ] Keep local persistence as a development fallback only.
- [ ] Preserve the current optimistic visual behavior and error notices.

**Exit gate:** The existing UI can run against the API without changing its layout or user flow.

### Phase 3 - Identity and Authorization

- [ ] Add OIDC login with an enterprise identity provider.
- [ ] Add workspace/tenant claims to API requests.
- [ ] Add RBAC roles: Admin, Incident Commander, Responder, Service Owner, Viewer, Auditor.
- [ ] Enforce authorization at the service boundary.
- [ ] Add audit attribution from authenticated identity.

**Exit gate:** Every incident read/write is tenant-scoped and role-checked.

### Phase 4 - SLA, Timeline, and Notifications

- [ ] Move SLA calculation to a backend application service.
- [ ] Persist immutable timeline and escalation events.
- [ ] Add scheduled SLA evaluation with idempotent escalation handling.
- [ ] Add notification ports and adapters for email, Teams, and webhook delivery.
- [ ] Add Redis-backed live update fanout.

**Exit gate:** SLA breach and reopen behavior remains consistent across refreshes and users.

### Phase 5 - Event Platform and Reporting

- [ ] Publish transactional outbox events to Kafka.
- [ ] Add idempotent consumers and dead-letter handling.
- [ ] Project searchable incident data into Elasticsearch only when needed.
- [ ] Add executive reporting queries and export-safe read models.
- [ ] Add correlation IDs across API, database, events, and notifications.

**Exit gate:** Event replay does not duplicate timeline entries, notifications, or audit records.

### Phase 6 - Production Readiness

- [ ] Add OpenTelemetry traces, metrics, and structured logs.
- [ ] Add rate limiting, secret management, dependency scanning, and secure headers.
- [ ] Add load tests for active incident and timeline workloads.
- [ ] Add deployment, rollback, backup, restore, and disaster-recovery runbooks.
- [ ] Validate accessibility, responsive behavior, and reduced motion in the locked UI.
- [ ] Complete the challenge demonstration checklist.

**Exit gate:** Security, performance, operability, and rollback evidence is documented.

## Immediate Work Queue

1. Add the typed frontend API client and environment configuration.
2. Add backend timeline and outbox persistence.
3. Add service tests for transition and reopen policies.
4. Install Maven and Docker, then compile the backend and start local infrastructure.
5. Wire the frontend to the API behind an explicit environment flag.

## Risks and Controls

| Risk | Control |
| --- | --- |
| Local fallback hides API failures | Show a visible fallback mode and log the reason. |
| Kafka introduces duplicate work | Transactional outbox, event IDs, and idempotent consumers. |
| Search diverges from truth | PostgreSQL remains authoritative; search is rebuildable. |
| Reopen policy differs between clients | Backend owns transition and 24-hour rules. |
| SLA clock differs by timezone | Store and calculate with UTC timestamps only. |
| AI suggests unsafe actions | Evidence-linked suggestions, read-only defaults, human approval. |
| Migration changes the approved UI | Treat visual files as locked and validate screenshots before release. |
