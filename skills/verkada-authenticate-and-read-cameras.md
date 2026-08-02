---
name: Authenticate and read Verkada camera data
description: Exchange a Verkada API key for a short-lived token, then list cameras and pull alerts and people/vehicle counts.
api: https://apidocs.verkada.com/reference/introduction
region_base_url: https://api.verkada.com
operations: [postloginapikeyviewv2, getcamerainfoviewv1, getnotificationsviewv1, getobjectcountsviewv1]
---

# Authenticate and read Verkada camera data

Use this to bootstrap any read workflow against the Verkada Command API.

## Preconditions
- An org admin has created a scoped API Key (Read-only is sufficient here) in Command > Organization Settings > Verkada API.
- Use the base URL for your org's region (US `https://api.verkada.com`, EU `https://api.eu.verkada.com`, AU `https://api.au.verkada.com`, GovCloud `https://api.verkadagov.com`). Credentials are region-bound.

## Steps
1. **Mint a token** — `postloginapikeyviewv2`: `POST /token` with header `x-api-key: <API_KEY>`. Response is `{"token":"..."}`, valid 30 minutes, not refreshable. Cache and reuse it; re-mint on `401 {"id":"0e2d","message":"Token expired"}`.
2. **Authenticate calls** — send `x-verkada-auth: <TOKEN>` on every endpoint request.
3. **List cameras** — `getcamerainfoviewv1`: `GET /cameras/v1/devices` to get camera IDs, models, sites, retention.
4. **Read alerts** — `getnotificationsviewv1`: `GET /cameras/v1/alerts?start_time=&end_time=` for offline/tamper/motion/crowd/person-of-interest alerts.
5. **Read counts** — `getobjectcountsviewv1` for people/vehicle counts over a time range.

## Rules
- Rate limit: 300 req/min per org; on `429` obey a 5-second cooldown then exponential backoff. Cache stable GETs like camera data.
- Paginate: pass `page_size` (<=200) and follow `next_page_token` until it is null.
- See `conventions/verkada-conventions.yml` and `errors/verkada-problem-types.yml`.
