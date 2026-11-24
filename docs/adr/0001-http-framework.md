# ADR 0001: Hono and a plain-JavaScript console instead of Next.js

Status: accepted. Seller: The Diamond Empire.

## Context

The blueprint named Next.js (App Router), Tailwind CSS and Auth.js. The build machine was shared and loaded, had no browser test setup and no container runtime. The product's value is in tenancy, billing correctness, security and audit, which are server-side logic.

## Decision

Use Hono for HTTP (it has no build step and can be driven in tests with `app.request`, so the whole stack is testable without opening sockets) and a small console in plain JavaScript with a system font stack, served by the same process. Write authentication directly on `node:crypto`.

## Consequences

- Every route is covered by fast in-process tests; there is no browser test.
- The console is deliberately small and has not been tested with an automated browser or accessibility tool.
- Auth.js features (passkeys, many OAuth providers) are not available; the `OAuthProvider` port is the seam for real providers.
- A team that wants Next.js can keep the services and replace `src/http` and `src/ui`.
