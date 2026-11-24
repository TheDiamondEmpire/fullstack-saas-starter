# ADR 0002: Plain SQL migrations instead of an ORM

Status: accepted. Seller: The Diamond Empire.

## Context

Row-level security policies, role grants, column privileges, triggers and advisory locks are the core of the tenancy and audit design. They are written in SQL whatever ORM is used.

## Decision

Hand-written SQL migrations, applied in order by a small runner that records a checksum and refuses to run if an applied migration was edited. Each table is preceded by a `-- tenancy: <class>` declaration. A text linter flags blocking operations and missing declarations. Queries are written as SQL through a thin `Tx.query` port.

## Consequences

- Policies and grants are visible in one place and reviewed as SQL.
- No generated types: row types are declared next to the code that uses them and are checked only by the tests.
- The linter is a text heuristic, and the runner is transactional (no concurrent index builds).
