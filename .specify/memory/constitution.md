<!-- Sync Impact Report
  Version change: template → 1.0.0 (initial ratification)
  Added sections: Core Principles (I–V), Technical Constraints, Development Standards, Governance
  Templates reviewed: spec-template.md ✅, plan-template.md ✅, tasks-template.md ✅
  Deferred TODOs: none
-->

# ContosoDashboard Constitution

## Core Principles

### I. Offline-First with Cloud Migration Path

All features MUST work without internet connectivity or external cloud services.
Every infrastructure dependency (file storage, authentication, database) MUST be implemented
behind an interface abstraction that allows swapping local implementations for Azure equivalents
via dependency injection — with zero changes to business logic, UI, or database schema.

- Local file storage → `IFileStorageService` (swap to `AzureBlobStorageService` in production)
- Local database → SQL Server LocalDB (swap to Azure SQL via connection string only)
- Mock auth → Cookie-based (swap to Microsoft Entra ID in production)

### II. Interface Abstraction (NON-NEGOTIABLE)

Every infrastructure dependency MUST be accessed through an interface, registered in DI, and
never instantiated directly from pages or services. This enables:
- Testability: mock implementations for unit tests
- Cloud migration: swap implementations without touching business logic
- Separation of concerns: pages depend on abstractions, not concretions

New features MUST define an interface (e.g., `IDocumentService`, `IFileStorageService`) before
implementing the concrete class.

### III. Layered Architecture (NON-NEGOTIABLE)

All code MUST follow strict layer separation — no cross-layer shortcuts:

```
Models  →  Data (EF Core DbContext)  →  Services  →  Pages (Blazor)
```

- **Pages** MUST only call Service interfaces — never DbContext directly
- **Services** MUST enforce all authorization and business rules before data access
- **Models** MUST be plain C# entities with no business logic
- **DbContext** is exclusively accessed by Services

### IV. Security and Authorization

Defense in depth is MANDATORY. Every feature must apply authorization at multiple levels:

- Razor pages/components MUST use `[Authorize]` attribute (or `<AuthorizeView>`)
- Service methods MUST validate user permissions before returning or modifying data
- IDOR (Insecure Direct Object Reference) protection: users MUST only access resources
  they are authorized for, checked at the service layer
- Role-based access MUST use the established hierarchy:
  Employee < TeamLead < ProjectManager < Administrator
- Files MUST be stored outside `wwwroot` and served only through authorized controller endpoints
- File paths MUST use GUID-based names (never user-supplied filenames) to prevent path traversal

### V. Async Operations

All I/O operations (database queries, file reads/writes, service calls) MUST use `async/await`.
No blocking calls (`.Result`, `.Wait()`) are permitted. EF Core queries MUST use
`ToListAsync()`, `FirstOrDefaultAsync()`, `SaveChangesAsync()`, etc.

## Technical Constraints

**Framework**: ASP.NET Core 8.0 — do not upgrade without team approval
**UI**: Blazor Server — no client-side Blazor or separate frontend framework
**Database**: SQL Server LocalDB with Entity Framework Core (code-first, `EnsureCreated()`)
**ORM**: Entity Framework Core with `.Include()` for eager loading (prevents N+1 queries)
**Styling**: Bootstrap 5.3 + Bootstrap Icons — no additional CSS frameworks
**Authentication**: Cookie-based mock auth (training only) — all pages require `[Authorize]`
**Primary keys**: Integer keys for all entities (consistent with existing User/Project tables)
**No external services**: No Azure SDK, no email, no cloud dependencies in training implementation
**File upload**: Generate unique GUID path BEFORE database insert to prevent orphaned records

## Development Standards

- **Upload sequence**: Validate → Authorize → Generate unique path → Save file → Save to DB
  (this order prevents both orphaned DB records and duplicate key violations)
- **Blazor file uploads**: Copy `IBrowserFile` stream to `MemoryStream` immediately; clear
  `IBrowserFile` reference after copy to prevent stream disposal errors
- **Seeding**: Database is seeded via `EnsureCreated()` on first run — no EF migrations needed
- **Error messages**: MUST be user-friendly (no stack traces, no internal paths exposed)
- **Claims**: Login MUST include all required claims — NameIdentifier, Name, Email, Role, Department
- **Performance**: Use database indexes on frequently queried fields; use `.Include()` to avoid N+1

## Governance

This constitution supersedes all other guidelines for ContosoDashboard development.
All new features MUST be reviewed against these principles before implementation begins.

**Amendment procedure**:
- MAJOR bump: removal or redefinition of a principle
- MINOR bump: new principle or section added
- PATCH bump: clarifications, wording, typo fixes

All pull requests MUST verify compliance with principles I–V before merge.
Complexity violations MUST be documented and justified in the feature's `plan.md`.

**Version**: 1.0.0 | **Ratified**: 2026-06-23 | **Last Amended**: 2026-06-23
