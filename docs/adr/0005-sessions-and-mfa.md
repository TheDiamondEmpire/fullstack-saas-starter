# ADR 0005: Opaque server-side sessions, CSRF tokens and TOTP

Status: accepted. Seller: The Diamond Empire.

## Decision

- Sessions are random tokens stored as SHA-256 hashes, with idle and absolute expiry, individually revocable. They are not JWTs, so revocation, "sign out everywhere" and account disabling take effect immediately.
- The cookie is HttpOnly and SameSite=Lax, `__Host-` prefixed and Secure in production. Mutating requests carry a per-session CSRF token in a header and, if sent, a same-origin `Origin`.
- After a correct password a user with MFA receives a pending session that expires in ten minutes and can only complete MFA; completing it issues a new session (no session fixation).
- Passwords: scrypt with parameters stored in the hash; a length-first policy with a small deny list. No external breach check (no network calls).
- TOTP (RFC 6238), one step of drift either side, each counter accepted once, hashed single-use recovery codes, secrets sealed at rest with AES-256-GCM. Failed MFA attempts count towards the same lockout as failed passwords.
- Responses do not reveal whether an account exists (login, sign-up, password reset), and login performs one password-hash verification whether or not the account exists. Response timing was not measured.

## Consequences

- One database read per authenticated request. A shared cache can be added later behind `authenticate`.
- Passkeys are not implemented.
