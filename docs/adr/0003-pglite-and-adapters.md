# ADR 0003: PGlite as the development and test database, behind an adapter

Status: accepted. Seller: The Diamond Empire.

## Context

No PostgreSQL server, no Redis and no working Docker were available. Tests of row-level security need a real PostgreSQL engine, not a mock.

## Decision

Run PGlite (PostgreSQL compiled to WebAssembly, in-process) for development and tests, behind a two-method `Db` port (`system`, `tenant`). Write a node-postgres adapter for a networked server and test it against PGlite served over the PostgreSQL wire protocol. Replace Redis with a PostgreSQL job table and an in-process rate limiter, each behind an interface.

## Consequences

- Tenant isolation was tested against a real PostgreSQL engine, but never against a real PostgreSQL server: server-specific behavior (connection pooling under load, lock contention across connections, role membership rules, versions other than the engine inside PGlite) is untested.
- PGlite has one connection, so lock behavior across concurrent connections (for example the audit chain's advisory lock) is exercised only in the trivial case.
- The in-process rate limiter is per instance; a multi-instance deployment needs a shared store implementing `RateLimitStore`.
