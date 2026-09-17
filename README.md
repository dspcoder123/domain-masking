# Domain Masking — Setup Prints

This project masks the real backend domain using a **Netlify server-side proxy rewrite**.  
Visitors and search engine crawlers only ever see the Netlify domain. The real Render URL is never exposed in the browser, page source, or HTTP responses.

---

## Live URLs

| Role | URL |
|---|---|
| Masked domain (Netlify) | https://joyful-druid-4c6b3b.netlify.app |
| Real backend (Render) | https://setupprints.onrender.com |
| GitHub repo | https://github.com/dspcoder123/domain-masking |

---

## How It Works

```
Visitor hits joyful-druid-4c6b3b.netlify.app
        ↓
Netlify edge server proxies the request server-side
        ↓
Fetches content from setupprints.onrender.com
        ↓
Returns it to visitor — real URL never visible
```

All proxying is handled by `netlify.toml` using a `status = 200` rewrite rule.  
This is a true server-side proxy — not an iframe.

---

## File Structure

```
domain-masking/
├── _headers        → Security & crawler-blocking HTTP headers
├── index.html      → Minimal placeholder (no iframe, no real URL)
├── netlify.toml    → Server-side proxy rewrite rules
├── robots.txt      → Blocks Googlebot and all major crawlers
└── README.md       → This file
```

---

## Key Configuration

### `netlify.toml` — Proxy Rule
```toml
[[redirects]]
  from   = "/*"
  to     = "https://setupprints.onrender.com/:splat"
  status = 200
  force  = true
```
`status = 200` is what makes this a **proxy** (not a redirect).  
The browser URL never changes. The real domain stays hidden.

### `robots.txt` — Crawler Blocking
Explicitly blocks Googlebot, AdsBot-Google, Bingbot, and all other major crawlers from every path.

### `_headers` — Security Layer
- `X-Robots-Tag: noindex, nofollow` — prevents indexing at HTTP level
- `Referrer-Policy: no-referrer` — prevents real origin leaking via referrer
- `Strict-Transport-Security` — forces HTTPS
- `Content-Security-Policy` — blocks browser from directly calling the Render URL
- `Cache-Control: no-store` — prevents proxied responses from being cached

---

## Deployment

1. Connect this GitHub repo to your Netlify site (`joyful-druid-4c6b3b.netlify.app`)
2. Netlify auto-deploys on every push to `main`
3. No build command needed — `publish = "."` serves files from root
