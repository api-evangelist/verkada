---
name: Provision a Verkada access user and grant door access
description: Create an access user, issue a card credential, add them to an access group, and unlock a door.
api: https://apidocs.verkada.com/reference/access-api-101
region_base_url: https://api.verkada.com
operations: [getaccessmembersviewv1, postaccesscardviewv1, putaccessgroupuserviewv1, getaccessdoorinformationviewv1, postaccessuserapiunlockviewv1]
---

# Provision a Verkada access user and grant door access

Requires an API Key with Read/Write scope for Access Control. Authenticate as in `verkada-authenticate-and-read-cameras.md` (mint token, send `x-verkada-auth`).

## Steps
1. **Find or create the user** — `getaccessmembersviewv1` (`GET /access/v1/access_users`) to look up by email/external_id. Users may be identified by Verkada `user_id` or your `external_id` (send one, not both).
2. **Issue a card** — `postaccesscardviewv1`: add an access card for the user (facility code + card number, or `card_number_hex`).
3. **Add to an access group** — `putaccessgroupuserviewv1`: add the user (by `external_id` or `user_id`) to a `group_id`. The group's Access Levels determine which doors and schedules apply.
4. **List doors** — `getaccessdoorinformationviewv1` to resolve `door_id`s (optionally filter by `site_ids`).
5. **Unlock (as user)** — `postaccessuserapiunlockviewv1`: unlock a `door_id` as the user; granted only if the user's Access Level covers that door. Use `postaccessadminapiunlockviewv1` to override privileges (treat as safety-critical).

## Rules
- Door unlock is a physical, safety-critical action — gate it behind explicit human intent; prefer user-scoped unlock over admin override.
- PUT/DELETE credential operations are idempotent by HTTP semantics; Verkada exposes no `Idempotency-Key` header.
- Respect the 300 req/min org rate limit; back off on `429`.
