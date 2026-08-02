---
name: Find upcoming Uncle Nearest events
description: >-
  Look up the events Uncle Nearest has published on its own calendar — tastings, distillery experiences, festival
  appearances and brand activations — filtered by date range, location or category, and resolve each one to its venue
  and organizer.
api: openapi/uncle-nearest-tec-v1-openapi.json
operations:
  - getEvents
  - getEvent
  - getVenue
  - getOrganizer
generated: '2026-08-02'
method: generated
source: openapi/uncle-nearest-tec-v1-openapi.json (operationIds verified verbatim in the spec)
---

# Find upcoming Uncle Nearest events

Uncle Nearest publishes its public event calendar through The Events Calendar REST API on its own web host. There is
no developer program, no API key and no rate-limit contract — this is a marketing site's CMS surface. Read it
politely, cache aggressively, and never write to it.

## Base

```
https://unclenearest.com/wp-json/tec/v1
```

## Before you call

- **Send an `Accept-Language` header.** The host returns `406 Not Acceptable` on `wp-json` requests that omit it.
  `Accept-Language: en-US,en;q=0.9` returns `200`.
- **No authentication is needed** for published content. Do not send credentials.
- **Do not call any write operation.** `createEvent`, `updateEvent`, `deleteEvent` and their venue/organizer siblings
  mutate the brand's live public calendar and have no idempotency key. See
  `agentic-access/uncle-nearest-agentic-access.yml`.

## Steps

1. **List the events in your window** with `getEvents` (`GET /events`). Useful parameters, all present in the spec:
   `start_date`, `end_date`, `starts_after`, `ends_before`, `search`, `categories`, `tags`, `venue`, `organizer`,
   `featured`, `status`, `orderby`, `order`, `page`, `per_page`. For "what's coming up", set `starts_after` to today
   and `orderby=start_date&order=asc`.
2. **Page correctly.** Read `X-WP-Total` and `X-WP-TotalPages` from the response headers, or follow the RFC 5988
   `Link` header's `rel="next"`. `per_page` caps at 100. Requesting a page past the last one returns `404`, not an
   empty list — check `X-WP-TotalPages` first.
3. **Resolve a specific event** with `getEvent` (`GET /events/{id}`) when you need the full record.
4. **Resolve place and host.** Call `getVenue` (`GET /venues/{id}`) for the address and geolocation, and
   `getOrganizer` (`GET /organizers/{id}`) for the contact details. In the older `tribe/events/v1` API these objects
   arrive already embedded inside each event, which is cheaper if you need them for every row — see
   `openapi/uncle-nearest-events-calendar-v1-openapi.json`.
5. **Filter by proximity** if the user asks "near me": `geoloc_lat`, `geoloc_lng`, `distance` and `has_geoloc` are all
   supported on `getEvents`.

## Handling errors

The response body is the WordPress envelope — `{ "code": ..., "message": ..., "data": { "status": ... } }`, not RFC
9457 problem+json. Full catalogue in `errors/uncle-nearest-problem-types.yml`.

- `400` — a query variable has a bad format (dates must be parseable, `geoloc_lat`/`geoloc_lng` must be paired).
- `404` — no such event, or the requested page is past the end.
- `410` — the event was trashed; treat as removed from the calendar.

## Caveats to surface to the user

- These endpoints are unversioned in practice and can change with any plugin update — `lifecycle/uncle-nearest-lifecycle.yml`.
- Event data reflects only what the brand publishes on unclenearest.com. Ticketing, distillery tour booking and
  retail purchase all happen off-API (`https://unclenearest.com/tours/`, `https://unclenearest.com/shop-whiskey/`).
