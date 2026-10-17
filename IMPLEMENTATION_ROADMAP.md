# Implementation Roadmap

This document defines the implementation phases for Organization MCP Intelligence Server, from the initial architecture to the first production-ready self-hosted release.

The project will be developed incrementally, with tests, clear acceptance criteria, and runnable implementations at every phase.

## Development Principles

- Start with a modular monolith.
- Write tests before implementation where practical.
- Keep PostgreSQL as the source of truth for structured data.
- Keep Git as the source of truth for code.
- Treat Qdrant as a rebuildable search index.
- Keep embedding providers replaceable.
- Avoid unnecessary infrastructure and abstractions.
- Make indexing jobs idempotent and recoverable.
- Never report successful indexing when required operations have failed.
- Document important architecture decisions before they become difficult to change.

## Phase 1 — Architecture and Contracts

### Objectives
Define the system's structure and behavior before implementation.

### Tasks
- Select the implementation language and MCP SDK.
- Define application modules and their responsibilities.
- Design the initial database schema.
- Define project lifecycle and indexing states.
- Design authentication and project permissions.
- Define knowledge provenance and review states.
- Specify MCP tool inputs and outputs.
- Define Git integration and embedding configuration.
- Design PostgreSQL-to-Qdrant synchronization.

### Deliverables
- `ARCHITECTURE.md`
- `docs/DATA_MODEL.md`
- `docs/MCP_TOOLS.md`
- `docs/SECURITY.md`
- Initial architecture decision records.

### Completion Criteria
The main entities, interfaces, state transitions, security boundaries, and synchronization behavior are documented.

## Phase 2 — Project Foundation

### Tasks
- Create the project structure.
- Configure application settings.
- Implement configuration validation.
- Add structured logging and error handling.
- Set up unit and integration tests.
- Add development tooling.

### Tests
- Invalid configuration is rejected.
- Missing required settings produce useful errors.
- Sensitive values are not written to logs.
- The application starts with valid configuration.

### Completion Criteria
The application runs locally and has a reproducible development and test workflow.

## Phase 3 — PostgreSQL and Data Layer

### Tasks
- Configure PostgreSQL.
- Create database migrations.
- Define domain models and repository interfaces.
- Implement initial database repositories.
- Establish transaction boundaries.
- Add database constraints and indexes.

### Tests
- Migrations work on a clean database.
- Repository operations behave correctly.
- Transactions roll back after failures.
- Duplicate records and invalid relationships are handled safely.

### Completion Criteria
The core data model can be created and tested reproducibly.

## Phase 4 — Authentication and Authorization

### Tasks
- Implement user and service identities.
- Implement agent credentials.
- Add credential revocation.
- Define project memberships and permissions.
- Add authentication and authorization middleware.
- Record sensitive operations in audit logs.

### Tests
- Invalid or revoked credentials are rejected.
- Agents cannot access unauthorized projects.
- Project IDs supplied by an agent cannot bypass permissions.
- Credentials never appear in logs.

### Completion Criteria
Every project-scoped operation enforces authorization on the server.

## Phase 5 — Project Registry and Onboarding

### Tasks
- Register and update projects.
- Configure repository locations and branches.
- Store project indexing settings.
- Implement project status tracking.
- Accept project analysis from agents.
- Track missing requirements and open questions.
- Provide project context retrieval.

### Tests
- Invalid project configuration is rejected.
- Incomplete setup remains explicitly incomplete.
- Repeated operations behave consistently.
- Unauthorized agents cannot modify project configuration.

### Completion Criteria
An organization can register a project and track its onboarding status.

## Phase 6 — Shared Knowledge and Decisions

### Tasks
- Store architecture analyses and technical knowledge.
- Store project requirements and open questions.
- Record technical decisions and their rationale.
- Track source references and relevant revisions.
- Add review and superseding workflows.
- Define which knowledge types are eligible for semantic indexing.

### Tests
- Knowledge records preserve provenance.
- Unreviewed information remains distinguishable from verified information.
- Superseded decisions remain traceable.
- Knowledge cannot leak between projects.

### Completion Criteria
One authorized agent can submit project knowledge and another authorized agent can retrieve it.

## Phase 7 — Git Integration

### Tasks
- Define a Git provider interface.
- Implement repository registration and access.
- Fetch repository metadata and source files.
- Track branches and commit revisions.
- Generate file manifests and hashes.
- Detect added, modified, renamed, and deleted files.
- Validate repository URLs and credentials.

### Tests
- Invalid repository URLs are rejected.
- Repository scripts are never executed.
- File changes are detected correctly.
- Repeated synchronization is safe.
- Git credentials are protected.
- An older synchronization run cannot overwrite the status of a newer revision.

### Completion Criteria
The server can safely inspect a repository and determine which files have changed.

## Phase 8 — Embedding Provider

### Tasks
- Define a provider-neutral embedding interface.
- Define embedding results and model metadata.
- Implement a fake provider for unit tests.
- Implement the initial Ollama adapter.
- Validate vector dimensions and response structure.
- Store model and embedding profile configuration.
- Define re-indexing behavior when the model changes.

### Tests
- Vector count matches the number of input texts.
- Vector dimensions are consistent.
- Invalid or non-finite values are rejected.
- Provider errors are handled safely.
- Model changes mark affected indexes as stale.

### Completion Criteria
Embeddings can be generated through a configurable provider without coupling the application to Ollama.

## Phase 9 — Code Indexing Pipeline

### Tasks
- Define file inclusion and exclusion rules.
- Exclude secrets, binaries, generated output, and dependency directories by default.
- Implement deterministic chunk identifiers.
- Extract file paths and line ranges.
- Calculate file hashes.
- Implement full and incremental indexing jobs.
- Track indexing progress and errors.

### Tests
- Excluded files are not indexed.
- Unchanged files are not unnecessarily reprocessed.
- Changed files replace their previous active chunks.
- Deleted files disappear from active search results.
- Repeated jobs do not create duplicate active records.
- Partial failures do not produce a false success status.

### Completion Criteria
A repository can be indexed and updated incrementally after a new commit.

## Phase 10 — Qdrant and Semantic Search

### Tasks
- Configure Qdrant collections.
- Define vector payload schemas.
- Store project and source metadata.
- Implement code search.
- Implement knowledge search.
- Apply server-controlled project filters.
- Validate search results against canonical records.
- Return source references and freshness information.

### Tests
- Embedding dimensions match collection configuration.
- Search results respect project permissions.
- Deleted or superseded content is not returned as active content.
- Stale results are identified according to the defined policy.
- Qdrant failures do not delete canonical PostgreSQL records.

### Completion Criteria
Authorized agents can search code and knowledge semantically with useful provenance.

## Phase 11 — Reliable Synchronization

### Tasks
- Implement a PostgreSQL outbox.
- Persist pending Qdrant operations.
- Implement idempotent background processing.
- Add bounded retries and failure tracking.
- Recover interrupted jobs.
- Reconcile PostgreSQL and Qdrant inconsistencies.
- Prevent incomplete indexing jobs from marking an index ready.

### Tests
- Failed vector operations remain recoverable.
- Retrying operations is safe.
- Duplicate events do not create duplicate active points.
- Worker restarts do not lose pending work.
- Index readiness reflects successful processing of the target revision.

### Completion Criteria
Temporary infrastructure failures do not silently corrupt index state or lose canonical data.

## Phase 12 — MCP Interface

### Tasks
- Implement MCP Streamable HTTP.
- Expose project context and status tools.
- Expose onboarding and analysis submission tools.
- Expose Git synchronization and index status tools.
- Expose code and knowledge search tools.
- Expose knowledge and decision management tools.
- Validate all tool inputs and outputs.
- Return bounded, structured responses.
- Represent long-running operations as asynchronous jobs.

### Tests
- MCP initialization and tool discovery work.
- Invalid inputs produce structured errors.
- Unauthorized operations are rejected.
- Long-running jobs can be started and inspected.
- Multiple MCP clients can use the same server-side contracts.

### Completion Criteria
At least two different MCP-compatible clients can connect to the same server and access the same authorized project knowledge.

## Phase 13 — Self-hosted Deployment

### Tasks
- Create Docker images.
- Provide Docker Compose configuration.
- Configure environment variables and secrets.
- Add health and readiness checks.
- Document HTTPS and reverse proxy configuration.
- Document local and remote agent connectivity.
- Add PostgreSQL backup and restore instructions.
- Document Qdrant persistence and recovery.
- Document upgrades and database migrations.

### Tests
- A clean installation works using documented steps.
- Dependency failures are reported accurately.
- Application restarts preserve pending jobs.
- Backup and restore are tested.
- Secrets are not committed or logged.

### Completion Criteria
An organization can install, configure, operate, and recover the system using the deployment documentation.

## Phase 14 — Integration Testing and First Release

### Tasks
- Test the complete Git-to-index pipeline.
- Test agent-submitted knowledge retrieval.
- Test authentication and project isolation.
- Test incremental indexing and deleted files.
- Test embedding provider failures.
- Test Qdrant outages and recovery.
- Add operational metrics and useful logs.
- Document supported limits and known issues.
- Prepare release notes and versioning rules.

### Release Checklist

- [ ] Fresh installation works.
- [ ] Database migrations are reproducible.
- [ ] Authentication and project permissions are tested.
- [ ] Multiple MCP clients can connect.
- [ ] Git repositories can be indexed.
- [ ] Incremental synchronization works.
- [ ] Knowledge is shared with provenance.
- [ ] Code and knowledge search respect access permissions.
- [ ] Stale indexes are reported accurately.
- [ ] Failed vector synchronization can recover.
- [ ] Backup and restore have been tested.
- [ ] Deployment and security documentation is complete.

## Future Phase — Local Edition

After the self-hosted team edition is stable:

- Add SQLite persistence where compatible with the domain repository interfaces.
- Provide a local deployment profile.
- Support local embedding and vector-store configurations.
- Reuse domain models, validation, and MCP contracts.
- Document the differences between local and team modes.

The local edition must not delay the initial team-server release.

## Definition of Done

A phase is complete when:

1. Its acceptance criteria are satisfied.
2. Tests cover normal behavior and important failure cases.
3. The code runs locally.
4. Configuration and execution instructions are documented.
5. No failure is hidden behind a success status.
6. The next phase can rely on stable interfaces.
