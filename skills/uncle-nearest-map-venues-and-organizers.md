---
name: Map Uncle Nearest venues and organizers
description: >-
  Build a location and host directory from the Uncle Nearest event calendar — every venue with its address and
  coordinates, every organizer with its contact details — for mapping, itinerary building or answering "where does
  Uncle Nearest show up".
api: openapi/uncle-nearest-tec-v1-openapi.json
operations:
  - getVenues
  - getVenue
  - getOrganizers
  - getOrganizer
  - getEvents
  - getSeries
generated: '2026-08-02'
method: generated
source: openapi/uncle-nearest-tec-v1-openapi.json (operationIds verified verbatim in the spec)
---

# Map Uncle Nearest venues and organizers

Venues and organizers are first-class records in the Events Calendar API, not just fields on an event. That makes a
directory build cheap: list them once, then join events onto them.

## Base

```
https://unclenearest.com/wp-json/tec/v1
```

Send `Accept-Language: en-US,en;q=0.9` or the host answers `406`. No authentication for reads.

## Steps

1. **List venues** with `getVenues` (`GET /venues`), paging with `page` / `per_page` and reading `X-WP-Total` /
   `X-WP-TotalPages`. Each record carries the venue name, address, city, state/province, zip, country, phone, website
   and — when set — `geo_lat` / `geo_lng`.
2. **List organizers** with `getOrganizers` (`GET /organizers`) for name, phone, website and email.
3. **Join events onto them.** Call `getEvents` (`GET /events`) with the `venue` or `organizer` filter to get
   everything happening at one place or run by one host. This is the join direction the API supports; there is no
   "events for venue" sub-resource.
4. **Fetch detail on demand** with `getVenue` (`GET /venues/{id}`) and `getOrganizer` (`GET /organizers/{id}`).
5. **Group recurring programming** with `getSeries` (`GET /series`), then filter events with the `series` /
   `in_series` parameters on `getEvents`. Series is read-only in the published spec.

## Data shape

Every one of these entities extends the same `TEC_Post_Entity` base schema (id, slug, status, title, content, excerpt,
link, dates), so a single normalizer handles all four. The relationship graph is written out in
`data-model/uncle-nearest-data-model.yml`.

## Notes

- Geolocation is optional per venue — do not assume `geo_lat`/`geo_lng` are populated; fall back to geocoding the
  address yourself.
- The older `tribe/events/v1` API exposes the same three entities plus event categories and tags as `Term` records
  (`GET /categories`, `GET /tags`) if you need the taxonomy — that spec has no operationIds, so address it by path
  (`openapi/uncle-nearest-events-calendar-v1-openapi.json`).
- Treat this as read-only. All create/update/delete operations require WordPress capabilities and would edit the
  brand's live public calendar.
