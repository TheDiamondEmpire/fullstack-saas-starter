# Security checklist (OWASP ASVS Level 1 topics)

Seller: The Diamond Empire, <https://thedempire.shop/>.

This maps what the starter does to the topic chapters of the OWASP Application Security Verification Standard, Level 1. It is a self-assessment written by the author. It is **not** a certification, not a complete walk through every ASVS requirement, and not a penetration test. "Tested" means an automated test in this repository covers it; the test file is named so you can read it. Statuses: **done and tested**, **done, not tested**, **not done**, **not applicable here**.

| ASVS topic | What the starter does | Status | Evidence |
|---|---|---|---|
| Authentication: password storage | scrypt with per-hash salt and stored parameters; cost raised in production; rehash on login | done and tested | `tests/auth.test.ts` |
| Authentication: password policy | Minimum 12 characters, maximum 256, small deny list, not the email name, no repeated character; no composition rules; no external breach check | done and tested (no breach list: no network calls) | `tests/auth.test.ts` |
| Authentication: credential recovery | Single-use hashed reset tokens, one hour, revoke all sessions, no account enumeration | done and tested | `tests/auth.test.ts`, `tests/e2e.test.ts` |
| Authentication: brute force | Lockout after 5 failures for 15 minutes (also for MFA codes), per-address rate limit on auth endpoints, uniform error messages | done and tested | `tests/auth.test.ts`, `tests/http.test.ts` |
| Authentication: multi-factor | TOTP (RFC 6238 vectors), replay protection, recovery codes, MFA required for the back office | done and tested | `tests/totp.test.ts`, `tests/auth.test.ts`, `tests/platform.test.ts` |
| Authentication: passkeys / WebAuthn | Not implemented | not done | README |
| Session management | Opaque tokens stored hashed, idle and absolute expiry, revocation, new session after MFA, logout-all, HttpOnly SameSite cookie, `__Host-` and Secure in production | done and tested | `tests/auth.test.ts`, `tests/http.test.ts` |
| Session management: CSRF | Per-session token in a header plus Origin check on mutating requests | done and tested | `tests/http.test.ts` |
| Access control | Role permission matrix tested exhaustively; checks in the service layer as well as the route; no privilege escalation by role assignment; last-owner protection; non-members get 404 | done and tested | `tests/permissions.test.ts`, `tests/orgs.test.ts`, `tests/http.test.ts` |
| Access control: multi-tenancy | Row-level security on every tenant table, fail-closed, catalog checked against declarations, fuzzed | done and tested on PGlite only; **never tested on a real PostgreSQL server** | `tests/tenancy.test.ts`, experiment E1 |
| Access control: API keys | Scopes, per-key rate limit, revocation, expiry, hashed storage, organization and plan checks | done and tested | `tests/apikeys-audit.test.ts`, `tests/http.test.ts` |
| Validation and encoding | Inputs validated per route and in services; parameterized SQL only; HTML email templates escape every field; the console inserts text, never HTML | done and tested (no fuzzing of every route) | `tests/platform.test.ts`, `tests/http.test.ts` |
| Business logic | Seat, project and quota limits, idempotent usage events, billing state changes only through verified webhooks | done and tested | `tests/webhooks.test.ts`, `tests/http.test.ts` |
| Error handling and logging | Generic 500 body, request ids, structured JSON logs with secret redaction, append-only audit log of security-relevant events | done and tested | `tests/http.test.ts`, `tests/platform.test.ts`, `tests/apikeys-audit.test.ts` |
| Data protection | TOTP secrets sealed with AES-256-GCM; tokens and keys stored as hashes; column privileges keep password hashes away from tenant code; responses omit hashes; job payloads with links scrubbed after delivery | done and tested | `tests/auth.test.ts`, `tests/tenancy.test.ts`, `tests/platform.test.ts` |
| Data protection: encryption at rest of the database | Left to the database host (the Terraform example turns on RDS storage encryption) | done, not tested (Terraform never applied) | `deploy/terraform/main.tf` |
| Communications | Security headers (CSP without inline script, frame-ancestors none, nosniff, no referrer, HSTS in production); TLS is terminated in front of the service | headers tested; TLS not applicable here | `tests/http.test.ts` |
| Webhooks | Signature over the raw body with timestamp tolerance, constant-time comparison, event-id idempotency | done and tested against this repository's own signer, **never against a Stripe-produced signature** | `tests/billing-pure.test.ts`, `tests/webhooks.test.ts` |
| Configuration | Production refuses default or weak secrets, a missing database URL, and the billing simulator unless acknowledged; no secrets in the repository (scanned); `.env` ignored | done and tested | `tests/config.test.ts`, `tests/repo-hygiene.test.ts` |
| Dependencies | Three direct runtime dependencies plus node-postgres; license check; `npm audit` of production packages recorded | done (scan results depend on the advisory database on the day) | `results/licenses.json`, `results/dependency-audit.json` |
| Migrations | Linter for blocking operations and missing tenancy declarations; checksum check on applied migrations | done and tested (text heuristic) | `tests/migrations.test.ts` |
| Denial of service | Body size limits, rate limits, bounded in-memory rate-limit store; no load test | partly done; not load tested | `tests/http.test.ts` |
| Client-side: console | Plain JavaScript, no framework, no third-party scripts, no external requests | checked by tests of the served files; not tested in an automated browser | `tests/http.test.ts`, `tests/repo-hygiene.test.ts` |

## Known gaps

- No penetration test, no fuzzing of HTTP routes beyond the listed tests, no static analysis other than the type checker and ESLint's recommended rules.
- Real Stripe, email and OAuth providers were never contacted, so their failure modes are untested.
- The in-process rate limiter does not share state between instances.
- The audit chain does not detect truncation of its newest records (see [ADR 0006](adr/0006-audit-hash-chain.md)).
- Account recovery relies on email only; there is no support-assisted recovery flow.
- Sessions are not bound to a device or address.
