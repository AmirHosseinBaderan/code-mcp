# Organization MCP Intelligence Server

A self-hosted MCP server that provides shared project knowledge and code intelligence for an organization's AI coding agents.

## Overview

Organization MCP Intelligence Server is a centralized knowledge system that connects different AI coding agents to the same project context.

Developers can use any MCP-compatible agent, whether it runs locally on their computer or remotely on a server or cloud environment.

Instead of each agent independently analyzing the project and losing important context between sessions, agents can share architecture knowledge, technical decisions, requirements, code search results, and project updates through a central server.

The system is designed to be self-hosted by organizations. Each organization deploys and manages its own instance and connects its development tools to it.

## Goals

- Share project knowledge between different AI coding agents.
- Analyze and index Git repositories.
- Store project architecture, requirements, and technical decisions.
- Search source code and knowledge using semantic search.
- Detect changes in repositories and keep indexes up to date.
- Track knowledge sources, revisions, and review status.
- Provide secure, project-scoped access to information.
- Support different embedding models through a configurable provider.
- Make deployment and maintenance straightforward.
- Keep the architecture extensible for a future lightweight local edition.

## The Problem

When multiple developers use different AI agents, project knowledge becomes fragmented.

Each agent may analyze the same code repeatedly, miss decisions made in another session, or rely on outdated information.

Traditional vector databases can help retrieve relevant text, but they do not independently manage project lifecycle, permissions, source revisions, knowledge validity, or synchronization.

This project combines structured project knowledge, Git-based code indexing, and semantic search behind a shared MCP interface.

## How It Works

The system accepts project information through two complementary paths.

### 1. Git Integration

The server connects to a project's Git repository and:

- Reads source files and commit history.
- Detects added, modified, and deleted files.
- Indexes code incrementally.
- Generates embeddings for eligible content.
- Tracks which repository revision has been indexed.
- Detects when the current index is outdated.

Git remains the source of truth for source code and its history.

### 2. Agent-Submitted Knowledge

Agents can submit structured information such as:

- Project architecture analysis.
- Functional and technical requirements.
- Architecture decisions and their rationale.
- Important implementation details.
- Open questions and unresolved issues.
- References to relevant files and code revisions.

The server validates and stores this information so that other authorized agents can retrieve it.

Agent-submitted information is not automatically considered verified. Its source, author, revision, and review status should be recorded whenever applicable.

## Architecture

The initial architecture consists of the following components:

- **MCP Server:** Exposes project intelligence through MCP Streamable HTTP.
- **PostgreSQL:** Stores projects, permissions, knowledge, decisions, indexing metadata, jobs, and audit records.
- **Qdrant:** Provides vector storage and semantic search.
- **Git Integration:** Retrieves repository content and detects source-code changes.
- **Embedding Provider:** Generates vectors through a configurable provider, initially supporting Ollama.
- **Indexing Worker:** Processes indexing jobs and retries failed synchronization operations.

These components can initially run using Docker Compose. They do not need to be deployed as separate microservices.

## Technology Stack

| Component | Technology |
|---|---|
| Agent integration | MCP Streamable HTTP |
| Main database | PostgreSQL |
| Vector database | Qdrant |
| Embedding provider | Ollama initially |
| Source control | Git-compatible repositories |
| Deployment | Docker Compose |
| Architecture | Modular monolith with background jobs |

The implementation language and framework will be finalized in the architecture design before development begins.

## Core Design Principles

### Agent Independence

The system must not depend on a specific coding agent or AI model. Any compatible MCP client should be able to use the same server-side capabilities.

### Shared Knowledge

Project knowledge belongs to the project, not to a particular agent or conversation.

### Source of Truth

Git owns source-code history. PostgreSQL owns structured application data and knowledge records. Qdrant is a rebuildable search index.

### Incremental Indexing

Only changed files should normally require reprocessing.

### Explicit Freshness

The system must distinguish between current and outdated code indexes, incomplete setup, and knowledge that requires review.

### Security

Every operation must be authenticated and authorized. Agents may only access projects and perform operations for which they have permission.

### Reliability

Indexing and synchronization operations must be idempotent, observable, and recoverable after failures.

## Main Capabilities

### Project Management
- Register and configure projects.
- Connect Git repositories.
- Define project access permissions.
- Track onboarding and synchronization status.

### Project Intelligence
- Store and retrieve project context.
- Submit architecture analyses.
- Record requirements and open questions.
- Store and search technical decisions.
- Track knowledge provenance and review status.

### Code Intelligence
- Index repository files.
- Detect source changes.
- Search code semantically.
- Retrieve file paths and relevant source locations.
- Track indexed revisions and embedding configuration.

### Synchronization
- Trigger repository synchronization.
- Support webhook, scheduled, and manual synchronization.
- Process changed files incrementally.
- Retry failed indexing operations.
- Detect stale or incomplete indexes.

## Security Considerations

- Use authenticated MCP connections.
- Enforce project-level authorization.
- Store credentials securely.
- Prefer read-only Git credentials for indexing.
- Validate repository URLs and restrict network access.
- Never execute repository scripts during indexing.
- Exclude secrets, generated files, dependencies, and binary files by default.
- Avoid exposing sensitive information in logs or search results.
- Maintain audit records for security-sensitive operations.

## Deployment Model

Each organization operates its own instance.

The initial deployment includes the MCP application, PostgreSQL, Qdrant, and an embedding provider. These components may run on one server or be distributed across infrastructure controlled by the organization.

Local and remote agents connect to the same MCP endpoint, subject to network accessibility and authentication requirements.

## Future Direction

A future local edition may use SQLite and a local deployment profile.

The initial priority, however, is a reliable self-hosted server for teams. The local edition should reuse the domain logic and interfaces developed for the main product.

## Success Criteria

The first usable version should allow an organization to:

1. Deploy the system on its own infrastructure.
2. Connect different MCP-compatible agents.
3. Register and index a Git repository.
4. Detect and process repository changes.
5. Share knowledge between authorized agents.
6. Search code and knowledge with useful source references.
7. Detect stale indexes and recover from synchronization failures.
8. Manage access securely and back up its data.

See [IMPLEMENTATION_ROADMAP.md](IMPLEMENTATION_ROADMAP.md) for the development plan and acceptance criteria.
