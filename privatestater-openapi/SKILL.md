---
name: privatestater-openapi
description: Queries and manages PrivateStater data using the Open API. Use when the user asks about visitor counts, page traffic, browser or device breakdowns, session duration, traffic heatmaps, feedback messages, feedback ratings, captcha success rates, or any question about data in their PrivateStater dashboard. Also use when the user wants to update settings, manage feedback status, validate captcha tokens, or automate any PrivateStater operation via API.
---

# PrivateStater — Open API

Use this skill to answer user questions about their PrivateStater data or automate operations via the Open API.

Base URL: `https://privatestater.com`

## Feature References

Read the relevant file before making API calls:

- **Analytics** (visitors, pages, devices, duration, heatmap, sessions): [analytics.md](analytics.md)
- **Captcha** (stats, settings, token validation): [captcha.md](captcha.md)
- **Feedback** (list, stats, manage status, screenshots): [feedback.md](feedback.md)

---

## Agent Workflow

1. **Identify the feature** — analytics, captcha, or feedback
2. **Confirm credentials** if not already provided:
   - **API Key** (`prst_xxxx_yyyy`) — Dashboard → Project → API Keys → Create Key
   - **Website ID** — the unique identifier set when creating the website (e.g. `my-blog`)
3. **Read the feature reference file** for the exact endpoint, params, and response shape
4. **Call the API** and present results naturally — summarize numbers, highlight trends, do not dump raw JSON

---

## Authentication

```http
Authorization: Bearer prst_xxxx_yyyy
```

Alternative: `X-API-Key: prst_xxxx_yyyy`

Token format: `prst_<16hex>_<64hex>`

---

## Rate Limits

| Feature | Limit |
|---------|-------|
| Analytics | 120 req/min |
| Feedback | 120 req/min |
| Captcha | 90 req/min |

On `429`: wait briefly and retry.

---

## Error Reference

| HTTP | Error Code | Cause | Action |
|------|-----------|-------|--------|
| `401` | `authentication_required` | Missing or malformed API key | Ask user to provide a valid key |
| `403` | `access_denied` | Website not in this project | Confirm website ID and project |
| `404` | `website_not_found` | Website ID does not exist | Confirm website ID with user |
| `429` | `rate_limit_exceeded` | Too many requests | Wait and retry |
| `500` | `server_error` | Server-side error | Retry once; report if it persists |
