# Signal Room

Signal Room is an incident response and root cause analysis command center for engineering teams. It helps responders create incidents, coordinate ownership, track lifecycle progress, monitor SLA risk, capture decisions, and prepare an editable RCA.

The interface is intentionally an asymmetric command center rather than a generic admin dashboard. It uses dense operational panels, a live-style response timeline, contextual RCA tooling, related-incident signals, and subtle hover motion.

## Features

- Incident creation with unique IDs and validation
- P1, P2, and P3 severity handling
- Ownership assignment and responder activity notes
- Controlled lifecycle: `OPEN -> INVESTIGATING -> MITIGATED -> CLOSED`
- Invalid transition protection with clear feedback
- 24-hour reopen window after resolution
- SLA targets and status tracking:

	| Severity | Target |
	| --- | ---: |
	| P1 | 2 hours |
	| P2 | 4 hours |
	| P3 | 8 hours |

- Within-SLA, approaching-SLA, and breached-SLA filters
- SLA escalation events recorded in the incident timeline
- Editable, rule-based RCA draft generation
- Related-incident detection using shared services and keywords
- Executive operational posture signals
- Browser persistence through `localStorage` in demo mode
- Responsive layouts, reduced-motion support, and accessible form labels
- Hover shine and subtle tilt interactions on operational surfaces

## Technology

### Current Demo

- Next.js 15, React, and TypeScript
- Tailwind CSS and custom CSS design tokens
- Browser-local incident persistence
- Framer Motion, Recharts, TanStack Query, and Zustand dependencies prepared for further frontend integration

### Scalable Backend Foundation

- Java 21+ and Spring Boot 3.4
- PostgreSQL 16 with Flyway migrations
- Redis and Kafka integration configuration
- Spring Boot Actuator health and metrics endpoints
- Versioned REST API under `/api/v1/incidents`
- Domain-owned lifecycle and 24-hour reopen rules
- Timeline and transactional outbox schema foundations

## Project Structure

```text
.
├── app/                         # Next.js command-center frontend
│   ├── lib/incident-api.ts      # Typed Spring Boot API adapter
│   ├── page.tsx                 # Incident workspace and workflows
│   └── globals.css              # Locked visual system and interactions
├── backend/                     # Spring Boot incident API foundation
│   ├── src/main/java/           # Domain, application, and REST layers
│   ├── src/main/resources/      # Runtime configuration and Flyway SQL
│   └── pom.xml                  # Maven project definition
├── docs/                        # Architecture and migration plan
├── docker-compose.yml           # PostgreSQL, Redis, and Kafka services
├── next.config.mjs
├── package.json
└── README.md
```

## Run the Frontend

Requirements: Node.js 20 or later.

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

Validate a production build:

```bash
npm run build
```

## Demo Walkthrough

1. Review active incidents, SLA posture, and response velocity.
2. Select `INC-1042` to see related `INC-1043` and `INC-1044` checkout incidents.
3. Create a new P1, P2, or P3 incident and assign an owner.
4. Advance it through the controlled lifecycle.
5. Add a responder note and confirm it appears in activity and the response timeline.
6. Generate and edit the RCA draft.
7. Select a breached incident, raise an escalation, and show the timeline event.
8. Select `INC-1035` and demonstrate reopening within the 24-hour window.

## Backend Foundation

The backend is scaffolded for the next deployment phase:

```text
GET   /api/v1/incidents
GET   /api/v1/incidents/{incidentId}
POST  /api/v1/incidents
PATCH /api/v1/incidents/{incidentId}
POST  /api/v1/incidents/{incidentId}/transitions
```

Start local infrastructure when Docker and Maven are available:

```bash
docker compose up -d
cd backend
mvn spring-boot:run
```

Configure the frontend API origin with:

```text
NEXT_PUBLIC_API_URL=http://localhost:8080
```

When this variable is absent, the frontend stays in browser-persistence demo mode. PostgreSQL is intended to be the source of truth once the API is enabled; JSON is used only for HTTP/event transport.

## Deployment Direction

```text
GitHub repository
├── Next.js frontend -> Cloudflare Pages
└── Spring Boot API  -> Railway
												 └── Railway PostgreSQL
```

Redis and Kafka remain optional follow-up infrastructure until production traffic and integration requirements justify their operational cost.

## Project Plan

See [docs/scalable-migration-plan.md](docs/scalable-migration-plan.md) for phase gates covering API integration, identity, SLA persistence, event processing, observability, security, and production readiness.

## License

This project is a hands-on engineering challenge implementation. Add the repository license that matches your intended distribution before publishing it as an open-source project.
