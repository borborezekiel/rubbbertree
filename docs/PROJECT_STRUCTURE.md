# Rubber Tree Field Observation and Maintenance Management MVP

## Phase 1 Output (Structure + Backend Architecture)

This document defines the **full project folder structure** and the **backend clean architecture blueprint** before implementation files are added.

## Monorepo Folder Structure

```text
rubbbertree/
├── backend/
│   ├── prisma/
│   │   ├── migrations/
│   │   └── seeds/
│   ├── scripts/
│   └── src/
│       ├── config/
│       ├── common/
│       │   ├── errors/
│       │   ├── middleware/
│       │   ├── types/
│       │   ├── utils/
│       │   └── validators/
│       ├── docs/
│       ├── infrastructure/
│       │   ├── db/
│       │   ├── queue/
│       │   └── storage/
│       ├── modules/
│       │   ├── audit/
│       │   │   ├── controllers/
│       │   │   ├── dtos/
│       │   │   ├── repositories/
│       │   │   ├── routes/
│       │   │   └── services/
│       │   ├── auth/
│       │   │   ├── controllers/
│       │   │   ├── dtos/
│       │   │   ├── repositories/
│       │   │   ├── routes/
│       │   │   └── services/
│       │   ├── blocks/
│       │   │   ├── controllers/
│       │   │   ├── dtos/
│       │   │   ├── repositories/
│       │   │   ├── routes/
│       │   │   └── services/
│       │   ├── dashboard/
│       │   │   ├── controllers/
│       │   │   ├── dtos/
│       │   │   ├── repositories/
│       │   │   ├── routes/
│       │   │   └── services/
│       │   ├── files/
│       │   │   ├── controllers/
│       │   │   ├── dtos/
│       │   │   ├── repositories/
│       │   │   ├── routes/
│       │   │   └── services/
│       │   ├── notifications/
│       │   │   ├── controllers/
│       │   │   ├── dtos/
│       │   │   ├── repositories/
│       │   │   ├── routes/
│       │   │   └── services/
│       │   ├── observations/
│       │   │   ├── controllers/
│       │   │   ├── dtos/
│       │   │   ├── repositories/
│       │   │   ├── routes/
│       │   │   └── services/
│       │   ├── plantations/
│       │   │   ├── controllers/
│       │   │   ├── dtos/
│       │   │   ├── repositories/
│       │   │   ├── routes/
│       │   │   └── services/
│       │   ├── tasks/
│       │   │   ├── controllers/
│       │   │   ├── dtos/
│       │   │   ├── repositories/
│       │   │   ├── routes/
│       │   │   └── services/
│       │   ├── trees/
│       │   │   ├── controllers/
│       │   │   ├── dtos/
│       │   │   ├── repositories/
│       │   │   ├── routes/
│       │   │   └── services/
│       │   └── users/
│       │       ├── controllers/
│       │       ├── dtos/
│       │       ├── repositories/
│       │       ├── routes/
│       │       └── services/
│       └── tests/
│           ├── integration/
│           └── unit/
├── docs/
│   └── PROJECT_STRUCTURE.md
├── frontend/
│   └── app/
│       ├── assets/
│       │   ├── fonts/
│       │   ├── icons/
│       │   └── images/
│       ├── docs/
│       ├── mocks/
│       ├── sqlite/
│       ├── src/
│       │   ├── api/
│       │   ├── components/
│       │   │   ├── cards/
│       │   │   ├── common/
│       │   │   ├── forms/
│       │   │   ├── lists/
│       │   │   └── map/
│       │   ├── constants/
│       │   ├── hooks/
│       │   ├── navigation/
│       │   ├── screens/
│       │   │   ├── auth/
│       │   │   ├── dashboard/
│       │   │   ├── map/
│       │   │   ├── observations/
│       │   │   ├── profile/
│       │   │   ├── settings/
│       │   │   └── tasks/
│       │   ├── services/
│       │   ├── store/
│       │   │   ├── middleware/
│       │   │   └── slices/
│       │   ├── types/
│       │   └── utils/
│       └── tests/
└── infra/
    └── docker/
        ├── backend/
        ├── frontend/
        └── postgres/
```

## Backend Architecture (Clean + Modular)

### Layering

1. **Route layer (`modules/*/routes`)**
   - Defines REST endpoints.
   - Applies auth guards, RBAC middleware, and request validation middleware.

2. **Controller layer (`modules/*/controllers`)**
   - Handles HTTP input/output only.
   - Delegates business execution to services.

3. **Service layer (`modules/*/services`)**
   - Core business logic and orchestration.
   - Enforces domain rules (e.g., severity and follow-up constraints).

4. **Repository layer (`modules/*/repositories`)**
   - Data access abstraction for Prisma models.
   - Keeps persistence details out of services.

5. **DTO + Validator layer (`modules/*/dtos`, `common/validators`)**
   - Strongly typed contracts for create/update/query payloads.
   - Input validation for all APIs.

6. **Infrastructure layer (`infrastructure/*`)**
   - Database client (Prisma + Postgres/PostGIS).
   - Storage adapter (S3-compatible uploads).
   - Queue/notification adapters.

7. **Cross-cutting (`common/*`)**
   - Error handling, auth middleware, RBAC, pagination, logging helpers.

### Module Ownership

- `auth`: register, login, refresh token, forgot password, OTP verification.
- `users`: user profile and role management.
- `plantations`, `blocks`, `trees`: hierarchy and field asset master data.
- `observations`: full observation lifecycle (draft, submitted, synced).
- `tasks`: assignment, due dates, status transitions.
- `dashboard`: aggregated metrics and recent activity.
- `files`: pre-signed URL and metadata management for photos.
- `notifications`: in-app notifications and read/unread state.
- `audit`: immutable activity trail and operational compliance logs.

### Security + Auth Strategy

- JWT access tokens + refresh tokens.
- Optional login by phone number or username/password.
- OTP flow for password reset and secondary verification.
- Role-based authorization for Observer, Supervisor, Agronomist, Plantation Manager.

### API Versioning + Docs

- REST routes under `/api/v1`.
- Swagger/OpenAPI generated from route schemas and DTO contracts.

### Data + Geospatial Strategy

- PostgreSQL as primary database.
- PostGIS geometry/geography types for GPS point storage and map filters.
- Prisma used as ORM with raw SQL fallback for advanced geospatial queries.

### Offline-first Integration Contract

- Mobile app stores pending observations/tasks in SQLite.
- `sync_status` states: `pending`, `synced`, `failed`, `conflict`.
- Backend supports idempotent upserts using client-generated UUIDs.
- Sync endpoints support batch push/pull with server timestamps.

### Next Implementation Step

Proceed to create Phase 1 code files in this order:
1. Backend Express TypeScript bootstrap.
2. Prisma schema models + initial migration.
3. Auth module endpoints + JWT issuance.
4. Docker Compose (API + Postgres/PostGIS + optional pgAdmin).
5. Base README and `.env.example` files.
