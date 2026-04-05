---
name: privatestater-install
description: Adds PrivateStater Analytics, Captcha, and Feedback Widget to any website. Use when the user wants to install or integrate PrivateStater into their site, add the tracking script, implement captcha on a form, enable the feedback widget, or track button clicks. Also use when the user references PrivateStater scripts, the privatestater.js file, or any PrivateStater dashboard installation step.
---

# PrivateStater — Installation

PrivateStater provides three features via a single script: Analytics, Captcha, and Feedback Widget.

Base URL: `https://privatestater.com`

For a complete working example, see [example.html](example.html).

---

## Agent Workflow

1. **Check if the base script is already in `<head>`** — look for `PrivateStaterConfig` and `privatestater.js`
2. **If not installed**, add the Setup snippet first
3. **Determine what to integrate**:
   - Analytics — script install is enough; optionally add click tracking
   - Captcha — must be enabled in dashboard first, then implement widget + server validation
   - Feedback Widget — must be enabled in dashboard first; floating button appears automatically
4. **Remind the user**: localhost testing requires enabling **Allow Localhost** in dashboard settings

---

## Setup (Required for All Features)

1. Sign up at `https://privatestater.com/dashboard`
2. Create a **Project** → Add a **Website** (Website ID: globally unique, e.g. `my-blog`; Host: `example.com`)

Add to `<head>` of every page:

```html
<script>window.PrivateStaterConfig = { prstSite: "YOUR_SITE_ID" };</script>
<script src="https://privatestater.com/privatestater.js"></script>
```

> `window.PrivateStater` is available after this script loads. Place any calling scripts **after** this tag.

---

## Analytics

Auto-active after script install. No additional code needed for page view tracking.

**Auto-collected**: Visitors (IP-hashed, hourly dedup), Page Path, Browser, Device Type, Language, Referrer.

### Traffic Source Parameters

Append to URLs to override HTTP Referer: `src`, `source`, `ref`, `utm_source`

```
https://example.com/landing?src=newsletter
https://example.com?utm_source=google
```

### Click Tracking

```javascript
// Inline handler
<button onclick="window.PrivateStater.statsClick('signup_button')">Sign Up</button>

// Event listener
document.getElementById('cta').addEventListener('click', () => {
  window.PrivateStater.statsClick('cta_main');
});
```

Button ID rules: alphanumeric + `-` + `_`, max 100 chars, no spaces.

---

## Captcha

**Enable first**: Dashboard → Website → Captcha → Settings → **Enable Captcha**

Settings: `honeypotEnabled` (default: on), `powLevel`: `low` (~1s) / `medium` (~2s) / `high` (~3s)

### Method 1: HTML Widget (Recommended)

```html
<form id="contact-form">
  <input type="text" name="name">
  <input type="email" name="email">
  <input type="hidden" id="captcha-token" name="captchaToken">
  <div data-captcha="prst"></div>
  <button type="submit">Submit</button>
</form>

<script>
  window.onPrivateStaterCaptchaSuccess = function(token) {
    document.getElementById('captcha-token').value = token;
  };
  window.onPrivateStaterCaptchaError = function(error) {
    alert('Captcha verification failed. Please try again.');
  };
</script>
```

### Method 2: Programmatic

```javascript
const captcha = await window.PrivateStater.showCaptcha({
  container: document.getElementById('captcha-container'), // optional
  onSuccess: (token) => { /* send token to server */ },
  onError: (error) => { console.error(error); }
});
captcha.destroy(); // remove widget when done
```

### Server-Side Token Validation (Required)

Tokens are **single-use** and **expire in 5 minutes**. Always validate server-side.

Two endpoints available:
- `/api/captcha/validate` — no API key required (recommended for most use cases)
- `/oapi/captcha/validate` — requires API key (use when already calling the Open API)

```javascript
// Node.js/Express
app.post('/submit', async (req, res) => {
  const result = await fetch('https://privatestater.com/api/captcha/validate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ websiteId: 'YOUR_SITE_ID', verifyToken: req.body.captchaToken })
  }).then(r => r.json());

  if (!result.success) return res.status(400).json({ error: 'Captcha failed' });
  // proceed...
});
```

| Error Code | Cause |
|-----------|-------|
| `token_invalid_or_expired` | Expired (>5 min) or already used |
| `website_id_mismatch` | Token and websiteId don't match |
| `captcha_not_enabled` | Captcha is disabled in dashboard |
| `host_mismatch` | Request origin doesn't match registered Host |

> Widget uses Shadow DOM — external CSS does not affect it. Auto-detects dark/light mode.

---

## Feedback Widget

**Enable first**: Dashboard → Website → Feedback → Settings → **Enable Feedback**

Once enabled, a floating button appears automatically. No HTML changes needed.

```javascript
// Open programmatically (e.g. from a custom button)
window.PrivateStater.showFeedback();

// With close callback
window.PrivateStater.showFeedback({ onClose: () => console.log('closed') });
```

---

## Common Patterns

### Captcha in AJAX Form

```javascript
let captchaToken = null;

window.onPrivateStaterCaptchaSuccess = (token) => {
  captchaToken = token;
};

form.addEventListener('submit', async (e) => {
  e.preventDefault();
  if (!captchaToken) return alert('Please complete the captcha');
  await fetch('/api/submit', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ ...formData, captchaToken })
  });
  captchaToken = null; // single-use — reset after submit
});
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Stats not appearing | Wait ~1 min; verify `prstSite` matches Website ID |
| Captcha widget invisible | Enable in dashboard; check console for errors |
| Captcha/widget not on localhost | Enable **Allow Localhost** in dashboard settings |
| `token_invalid_or_expired` | Token expired (>5 min) or already used |
| `host_mismatch` | Request Origin must match the registered Host |

---

## Open API

To query analytics data, manage feedback, or automate settings via API, use the **`privatestater-openapi`** skill.
