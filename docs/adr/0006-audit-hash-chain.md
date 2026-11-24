# ADR 0006: An append-only, hash-chained audit log written in the same transaction

Status: accepted. Seller: The Diamond Empire.

## Decision

Each organization has its own chain, and a platform chain holds events without an organization. A record's hash covers its fields and the previous hash. The audit row is inserted in the same transaction as the change it records, so a change without its record cannot commit. Writers on a chain are serialized by a transaction-scoped advisory lock. The application role may only insert and select; a trigger refuses update, delete and truncate for everyone.

## Consequences

- An attacker with owner access who disables the trigger can still be caught editing, deleting from the middle, or reordering records (tested). Truncating the newest records is not detectable from the chain alone: anchor the latest hash somewhere the database owner cannot write (for example periodically to external storage) if that matters.
- Chains grow without bound; archiving is not built.
