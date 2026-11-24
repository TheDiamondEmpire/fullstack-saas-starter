# ADR 0004: The webhook handler is an event log first, and the only writer of billing state

Status: accepted. Seller: The Diamond Empire.

## Context

Payment providers deliver events at least once, possibly out of order, and sometimes replay them. A handler that applies effects directly can double-apply a plan change or let an old event overwrite newer state.

## Decision

1. Verify the signature over the raw body.
2. Insert the event under its provider id before processing; a repeat delivery only increments a counter.
3. Apply the effects and mark the event processed in one transaction that locks the event row.
4. Keep the `created` time of the last event applied to each subscription and invoice; apply an event only if it is not older; otherwise record it as `ignored_stale`.
5. Never write subscription or invoice rows from anywhere else. The API asks the provider to change things and waits for the resulting event. Tenants have read-only grants on billing tables.

## Consequences

- Replays, duplicates, concurrent duplicates and mid-processing failures are safe (tested, and measured in experiment E2).
- Event times have one-second resolution, so same-second events about one object cannot be ordered; they apply in arrival order, a canceled subscription is never revived by one, and `reconcile()` repairs the rest from the provider API.
- With real Stripe the response to a billing action may arrive before the webhook; a client must expect that state changes a moment later. The simulator delivers synchronously, which hides this in the tests.
