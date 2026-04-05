# PrivateStater — Feedback API Reference

Base path: `https://privatestater.com/oapi/feedback`

All requests require `Authorization: Bearer <apiKey>`.

---

## Feedback Item Shape

Every feedback object in list responses has this structure:

```json
{
  "id": "664fabc123def456",
  "createdAt": "2026-04-01T10:00:00Z",
  "status": "unread",
  "category": "issue",
  "rating": 3,
  "text": "The login page is broken on mobile.",
  "email": "user@example.com",
  "pageUrl": "https://example.com/login",
  "browser": "Chrome",
  "deviceType": "Mobile",
  "language": "en-US",
  "hasScreenshot": true,
  "consoleLogs": [
    { "level": "error", "message": "TypeError: Cannot read properties of null" }
  ]
}
```

### Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique feedback ID — use this for PATCH, DELETE, screenshot, and console-log requests |
| `createdAt` | ISO 8601 string | When the feedback was submitted |
| `status` | string | Workflow status — see Status Workflow below |
| `category` | string | `"issue"` / `"idea"` / `"other"` — selected by the user |
| `rating` | number (1–5) | Emoji rating (1 = worst, 5 = best). `null` if rating was disabled or skipped |
| `text` | string | Written feedback message. Empty string if text was disabled or skipped |
| `email` | string | User-provided email. Empty string if not provided or email collection is disabled |
| `pageUrl` | string | The URL of the page where feedback was submitted |
| `browser` | string | Browser name — collected only with user consent |
| `deviceType` | string | `"Desktop"` / `"Mobile"` / `"Tablet"` — collected only with user consent |
| `language` | string | Browser language (e.g. `"en-US"`) — collected only with user consent |
| `hasScreenshot` | boolean | `true` if a screenshot was attached. Fetch it via the screenshot endpoint |
| `consoleLogs` | array \| null | Browser console log entries captured at submission time. `null` if none captured |

### Status Workflow

```
unread → in_progress → completed
unread →                blocked
```

| Status | Meaning |
|--------|---------|
| `unread` | Newly submitted, not yet reviewed |
| `in_progress` | Being investigated or worked on |
| `completed` | Resolved or acknowledged |
| `blocked` | Spam or irrelevant — excluded from default list views |

> Note: By default, the list endpoint excludes `blocked` items unless you explicitly filter by `status=blocked`.

---

## GET `/websites/:websiteId/settings`

Returns the current feedback widget configuration.

**Response**:
```json
{
  "data": {
    "settings": {
      "enabled": true,
      "allowLocalhost": false,
      "textEnabled": true,
      "ratingEnabled": true,
      "emailEnabled": false,
      "widgetPosition": "bottom-right",
      "primaryColor": "#6366f1",
      "textColor": "#ffffff",
      "blockedWords": "spam, test"
    }
  }
}
```

### Settings Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enabled` | boolean | `false` | Whether the feedback widget is active on this website |
| `allowLocalhost` | boolean | `false` | Allows the widget to load on `localhost` for local testing |
| `textEnabled` | boolean | `true` | Shows the free-text input field in the widget |
| `ratingEnabled` | boolean | `true` | Shows the emoji rating step in the widget |
| `emailEnabled` | boolean | `false` | Shows the optional email input field in the widget |
| `widgetPosition` | string | `"bottom-right"` | `"bottom-right"` or `"bottom-left"` — floating button position |
| `primaryColor` | string | `""` | Hex color for the widget button (e.g. `"#6366f1"`). Empty string = default |
| `textColor` | string | `""` | Hex color for button text. Empty string = default |
| `blockedWords` | string | `""` | Comma-separated words — feedbacks containing these are automatically blocked |

---

## PUT `/websites/:websiteId/settings`

Updates feedback widget settings. All fields are optional — only include what you want to change.

**Request body**:
```json
{
  "enabled": true,
  "allowLocalhost": false,
  "textEnabled": true,
  "ratingEnabled": true,
  "emailEnabled": false,
  "widgetPosition": "bottom-left",
  "primaryColor": "#6366f1",
  "textColor": "#ffffff",
  "blockedWords": "spam, advertisement"
}
```

**Response**: same shape as GET settings

**Example — disable email collection and move widget to bottom-left**:
```javascript
await fetch(
  `https://privatestater.com/oapi/feedback/websites/${websiteId}/settings`,
  {
    method: 'PUT',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ emailEnabled: false, widgetPosition: 'bottom-left' })
  }
);
```

---

## GET `/websites/:websiteId/feedbacks`

Returns a paginated, filterable list of feedback items.

**Query params**:

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `range` | string | `"7d"` | `"1d"` / `"7d"` / `"30d"` / `"all"` |
| `status` | string | (all except blocked) | Filter by status: `"unread"` / `"in_progress"` / `"completed"` / `"blocked"` |
| `category` | string | (all) | Filter by category: `"issue"` / `"idea"` / `"other"` |
| `page` | number | `1` | Page number (1-indexed) |
| `limit` | number | `20` | Items per page (max `100`) |

> When `status` is not specified or is an unrecognized value, `blocked` items are excluded automatically.

**Response**:
```json
{
  "data": {
    "items": [ /* feedback item array */ ],
    "unreadCount": 34
  },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 142,
    "totalPages": 8
  }
}
```

- Pagination info (`total`, `page`, `limit`, `totalPages`) is under `meta`, not `data`
- `unreadCount` reflects unread items for the same range, regardless of active status filter

**Answer format**: "You have **142 feedbacks** total (**34 unread**). Showing page 1 of 8."

**Example — fetch all unread bug reports**:
```javascript
const res = await fetch(
  `https://privatestater.com/oapi/feedback/websites/${websiteId}/feedbacks?status=unread&category=issue&range=7d&limit=50`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
const { data, meta } = await res.json();
// data.items, meta.total
```

---

## GET `/websites/:websiteId/stats`

Returns aggregated feedback statistics for a given range.

**Query params**:
- `range` (optional, default: `"7d"`): `"1d"` / `"7d"` / `"30d"` / `"all"`

**Response**:
```json
{
  "data": {
    "total": 142,
    "avgRating": 3.4,
    "ratingCount": 118,
    "unreadCount": 34,
    "range": "7d"
  }
}
```

> Note: Fields are at the top level of `data` — there is no nested `stats` object.

### Stats Fields

| Field | Description |
|-------|-------------|
| `total` | Total feedbacks in the given range (all statuses) |
| `avgRating` | Average rating (1–5 scale) across feedbacks that included a rating. `0` if no ratings |
| `ratingCount` | Number of feedbacks that included a rating (denominator for `avgRating`) |
| `unreadCount` | Number of feedbacks in `unread` status within the range |
| `range` | The range value that was used for the query |

**Answer format**: "In the last 7 days you have **142 feedbacks** with an average rating of **3.4/5**. **34** are unread."

---

## PATCH `/websites/:websiteId/feedbacks/:feedbackId`

Updates the status of a single feedback item.

**Path params**:
- `websiteId` (string)
- `feedbackId` (string) — the `id` field from the feedback item

**Request body**:
```json
{ "status": "in_progress" }
```

Valid values: `"unread"` `"in_progress"` `"completed"` `"blocked"`

**Response**:
```json
{ "data": { "ok": true } }
```

---

## DELETE `/websites/:websiteId/feedbacks/:feedbackId`

Permanently deletes a single feedback item. This action is irreversible.

**Response**:
```json
{ "data": { "ok": true } }
```

---

## POST `/websites/:websiteId/feedbacks/batch-update`

Updates the status of up to **100 feedback items** in a single request.

**Request body**:
```json
{
  "feedbackIds": ["664fabc1", "664fabc2", "664fabc3"],
  "status": "completed"
}
```

- `feedbackIds`: array of feedback ID strings (max 100)
- `status`: target status for all items — `"unread"` `"in_progress"` `"completed"` `"blocked"`

**Response**:
```json
{ "data": { "modifiedCount": 3 } }
```

> Response key is `modifiedCount` (not `updatedCount`).

**Example — auto-triage all unread bug reports to in_progress**:
```javascript
const listRes = await fetch(
  `https://privatestater.com/oapi/feedback/websites/${websiteId}/feedbacks?status=unread&category=issue&limit=100`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
const { data } = await listRes.json();
const ids = data.items.map(f => f.id);

if (ids.length > 0) {
  await fetch(
    `https://privatestater.com/oapi/feedback/websites/${websiteId}/feedbacks/batch-update`,
    {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${apiKey}`, 'Content-Type': 'application/json' },
      body: JSON.stringify({ feedbackIds: ids, status: 'in_progress' })
    }
  );
}
```

---

## POST `/websites/:websiteId/feedbacks/batch-delete`

Permanently deletes up to **100 feedback items** in a single request. Irreversible.

**Request body**:
```json
{ "feedbackIds": ["664fabc1", "664fabc2"] }
```

**Response**:
```json
{ "data": { "deletedCount": 2 } }
```

---

## GET `/websites/:websiteId/feedbacks/:feedbackId/screenshot`

Returns the screenshot attached to a feedback item (only if `hasScreenshot: true`).

**Response**:
```json
{ "data": { "screenshot": "<base64-encoded image string>" } }
```

- The base64 string is a PNG image
- To display: `<img src="data:image/png;base64,${screenshot}">`
- Returns `404` with `{ "error": "screenshot_not_found" }` if no screenshot exists

---

## GET `/websites/:websiteId/feedbacks/:feedbackId/console-logs`

Returns browser console logs captured at the time of feedback submission.

**Response**:
```json
{
  "data": {
    "consoleLogs": [
      { "level": "error", "message": "Uncaught TypeError: Cannot read properties of null" },
      { "level": "warn",  "message": "Deprecated API usage detected" }
    ]
  }
}
```

> Response key is `consoleLogs` (not `logs`).

- Returns `404` with `{ "error": "console_logs_not_found" }` if no logs exist
- The `consoleLogs` array is the same format as the `consoleLogs` field in the feedback list item
