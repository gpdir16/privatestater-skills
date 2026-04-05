# PrivateStater — Analytics API Reference

Base path: `https://privatestater.com/oapi/analytics/websites/:websiteId`

All requests require `Authorization: Bearer <apiKey>`.

---

## Common Parameters

### `range`

Applies to all endpoints that accept a `range` query param.

| Value | Period |
|-------|--------|
| `1h`  | Last 1 hour |
| `12h` | Last 12 hours |
| `1d`  | Last 24 hours |
| `3d`  | Last 3 days |
| `7d`  | Last 7 days |
| `14d` | Last 14 days |
| `30d` | Last 30 days |
| `6m`  | Last 6 months |
| `1y`  | Last 1 year |

Default when omitted: `7d`

### `filters` (optional)

JSON-encoded array passed as a query param. Narrows results to a specific dimension value.

```
?filters=[{"dim":"browser","value":"Chrome"}]
```

Valid `dim` values: `browser` `deviceType` `language` `referrer` `pages` `clicks`

---

## GET `/visitors`

Returns daily visitor counts for the given range.

**Query params**: `range` (required), `filters` (optional)

**Response**:
```json
{
  "data": {
    "series": {
      "2026-03-30": 38,
      "2026-03-31": 54,
      "2026-04-01": 42
    },
    "range": "7d"
  }
}
```

- Keys are `YYYY-MM-DD` date strings in UTC
- Values are unique visitor counts (IP-hashed, deduplicated per hour)
- To get the **total**: sum all values in `series`
- To get the **peak day**: find the entry with the highest value

**Answer format**: "You had **X visitors** over the last 7 days. Peak day was **YYYY-MM-DD** with **Y visitors**."

---

## GET `/breakdown`

Returns a count breakdown for a single dimension (e.g. which browsers your visitors use).

**Query params**:
- `dimension` (required): one of `browser` `deviceType` `language` `referrer` `pages` `clicks`
- `range` (required)
- `filters` (optional)

**Response**:
```json
{
  "data": {
    "counts": {
      "Chrome": 210,
      "Safari": 98,
      "Firefox": 34
    },
    "dimension": "browser",
    "range": "7d"
  }
}
```

- Keys are the dimension values (browser name, page path, device type, etc.)
- Values are counts for the given range
- Sort by value descending to rank entries

**Answer format examples**:
- Browser: "Most used browsers: **Chrome** (210), **Safari** (98), **Firefox** (34)"
- Pages: "Top pages: **/home** (180 views), **/pricing** (74 views)..."
- Device: "**Desktop** 65%, **Mobile** 30%, **Tablet** 5%"

---

## GET `/metrics/average-duration`

Returns the average session duration across all visitors for the given range.

**Query params**: `range` (required), `filters` (optional)

**Response**:
```json
{
  "data": {
    "avgSeconds": 143,
    "range": "7d"
  }
}
```

- Convert to human-readable: `Math.floor(avgSeconds / 60)` min `avgSeconds % 60` sec

**Answer format**: "Average session duration: **2 min 23 sec**"

---

## GET `/metrics/duration-by-dimension`

Returns average session duration broken down by a dimension value.

**Query params**:
- `dimension` (required): same valid values as `/breakdown`
- `range` (required)
- `filters` (optional)

**Response**:
```json
{
  "data": {
    "avgSecondsByKey": {
      "Desktop": 187,
      "Mobile": 94,
      "Tablet": 121
    },
    "dimension": "deviceType",
    "range": "7d"
  }
}
```

- Response key is `avgSecondsByKey` (not `durations`)
- Values are average seconds per dimension value
- Useful for comparing engagement across devices, browsers, or pages

**Answer format**: "Desktop visitors average **3 min 7 sec**, Mobile **1 min 34 sec**."

---

## GET `/heatmap/hourly`

Returns a traffic heatmap showing activity by day-of-week and hour-of-day.

**Query params**: `range` (required — recommend `30d` or longer for meaningful data)

**Response**:
```json
{
  "data": {
    "cells": {
      "Mon-09": 12,
      "Mon-10": 34,
      "Tue-14": 28
    },
    "range": "30d"
  }
}
```

- Keys follow the format `DDD-HH` — three-letter day abbreviation + zero-padded UTC hour
- Values are visitor counts aggregated across all matching periods in the range
- Days: `Mon` `Tue` `Wed` `Thu` `Fri` `Sat` `Sun`
- Hours: `00` through `23` (UTC)

**Answer format**: "Your busiest traffic window is **Mon–Wed around 10:00–14:00 UTC**."

---

## GET `/sessions`

Returns a paginated list of individual sessions.

**Query params**:
- `range` (required)
- `page` (optional, default: `1`)
- `limit` (optional, default: `30`, max: `100`)
- `filters` (optional)

**Response**:
```json
{
  "data": {
    "sessions": [
      {
        "startedAt": "2026-04-01T10:23:00Z",
        "durationSeconds": 94,
        "pageViews": 3,
        "browser": "Chrome",
        "deviceType": "Desktop",
        "language": "en-US",
        "referrer": "https://google.com",
        "entryPage": "/pricing",
        "exitPage": "/signup"
      }
    ]
  },
  "meta": {
    "page": 1,
    "limit": 30,
    "total": 842
  }
}
```

- Pagination info (`total`, `page`, `limit`) is under `meta`, not `data`

**Answer format**: "Found **842 sessions** in the last 7 days. Showing page 1."

---

## POST `/query-batch`

Runs up to **15 analytics queries** in a single request. Use this whenever the user asks multiple questions or requests a dashboard overview.

**Request body**:
```json
{
  "requests": [
    { "id": "visitors", "type": "visitors", "range": "7d" },
    { "id": "pages", "type": "pages", "range": "7d" },
    { "id": "browsers", "type": "browser", "range": "7d" },
    { "id": "devices", "type": "deviceType", "range": "7d" },
    { "id": "languages", "type": "language", "range": "7d" },
    { "id": "referrers", "type": "referrer", "range": "7d" },
    { "id": "clicks", "type": "clicks", "range": "7d" },
    { "id": "avgDur", "type": "avgDuration", "range": "7d" },
    { "id": "durByDev",  "type": "durationBreakdown", "dimension": "deviceType", "range": "7d" },
    { "id": "heatmap",   "type": "hourlyHeatmap", "range": "30d" }
  ]
}
```

Each request object:
- `id` (string, required) — arbitrary key used to look up the result
- `type` (string, required) — see table below
- `range` (string, required) — same values as individual endpoints
- `dimension` (string) — required only when `type` is `durationBreakdown`
- `filters` (array, optional) — same filter syntax as individual endpoints

### Valid `type` values and batch result shapes

> **Important**: Batch result shapes differ from individual endpoint responses — they return the inner data directly, without the outer `{ data: {...} }` wrapper.

| `type` | Returns (inside `results.<id>`) |
|--------|--------------------------------|
| `visitors` | `{ "YYYY-MM-DD": N, ... }` — raw date→count map (no `series` wrapper) |
| `avgDuration` | `{ "avg": N }` — note: key is `avg`, not `avgSeconds` |
| `durationBreakdown` | `{ "key": N, ... }` — raw avgSecondsByKey map (requires `dimension`) |
| `hourlyHeatmap` | `{ "DDD-HH": N, ... }` — raw heatmap cell map |
| `browser` | `{ "Chrome": N, ... }` — raw breakdown counts map |
| `deviceType` | `{ "Desktop": N, ... }` — raw breakdown counts map |
| `language` | `{ "en-US": N, ... }` — raw breakdown counts map |
| `referrer` | `{ "https://...": N, ... }` — raw breakdown counts map |
| `pages` | `{ "/path": N, ... }` — raw breakdown counts map |
| `clicks` | `{ "button_id": N, ... }` — raw breakdown counts map |

**Response**:
```json
{
  "data": {
    "results": {
      "visitors": { "2026-04-01": 42, "2026-04-02": 38 },
      "pages":    { "/home": 120, "/pricing": 45 },
      "browsers": { "Chrome": 210, "Safari": 98 },
      "avgDur":   { "avg": 143 }
    }
  },
  "errors": {
    "durByDev": "invalid_dimension"
  }
}
```

- Each key in `results` corresponds to the `id` provided in the request
- Failed sub-queries appear in `errors` (key → error code), not in `results`
- The `errors` field is omitted from the response when there are no errors

**Full example**:
```javascript
const res = await fetch(
  `https://privatestater.com/oapi/analytics/websites/${websiteId}/query-batch`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      requests: [
        { id: 'visitors', type: 'visitors',   range: '7d' },
        { id: 'pages',    type: 'pages',      range: '7d' },
        { id: 'devices',  type: 'deviceType', range: '7d' },
        { id: 'avgDur',   type: 'avgDuration', range: '7d' }
      ]
    })
  }
);
const { data } = await res.json();
// data.results.visitors["2026-04-01"], data.results.avgDur.avg, etc.
```
