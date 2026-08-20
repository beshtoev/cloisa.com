# cloisa.com

Static marketing site for Cloisa AI Inc. One page, no build step, no dependencies.
Hosted on **GitHub Pages**.

## Contents

| File | Purpose |
| --- | --- |
| `index.html` | The entire site (v2). Inline CSS, no JavaScript, no external requests. Sticky nav, hero, boundary-architecture SVG diagram, control cards |
| `404.html` | Not-found page |
| `CNAME` | Custom domain binding for GitHub Pages |
| `robots.txt`, `sitemap.xml` | Crawling |
| `.well-known/security.txt` | RFC 9116 vulnerability contact |
| `_headers` | Security headers. **GitHub Pages ignores this.** Kept for the Cloudflare path below |
| `.nojekyll` | Disables Jekyll on GitHub Pages. **Required**: without it Jekyll silently drops `.well-known/` (dot-directory) and `_headers` (underscore prefix), so `security.txt` would 404 |

## Deploy

1. Push to `main`
2. Settings, Pages, Source: **Deploy from a branch**, `main`, `/ (root)`
3. Custom domain: `cloisa.com`. Wait for the DNS check to pass, then tick **Enforce HTTPS**
4. DNS at your registrar:

```
A     @     185.199.108.153
A     @     185.199.109.153
A     @     185.199.110.153
A     @     185.199.111.153
CNAME www   beshtoev.github.io.
```

## Security posture

Deliberate properties, all of which survive on GitHub Pages:

- **Zero third-party requests.** No CDN, no web fonts, no analytics, no embeds. System font stack,
  inline SVG favicon as a data URI. Nothing about a visitor reaches anyone else
- **No JavaScript.** Nothing to inject into, no supply chain
- **No cookies, no localStorage, no sessionStorage, no forms.** Nothing to leak, nothing to consent to
- **CSP `default-src 'none'`** via meta tag
- **`referrer: no-referrer`** so outbound clicks disclose nothing
- Contact is `mailto:`, so no form handler and no third-party form processor

### What GitHub Pages cannot do, stated plainly

GitHub Pages does not let you set HTTP response headers. It ignores `_headers`. So the site ships
without `Strict-Transport-Security`, `X-Frame-Options`, `X-Content-Type-Options`,
`Permissions-Policy`, and without CSP `frame-ancestors`, which browsers ignore when CSP arrives in a
meta tag rather than a header.

Practical exposure on this specific site is low: there is no JavaScript, no cookie, no session and
no form, so clickjacking has nothing to steal and MIME sniffing has nothing to sniff. **The real
cost is reputational.** Anyone evaluating a company whose product is a control model may run
securityheaders.com against it, and today that returns a poor grade.

### Fix it without leaving GitHub Pages

Put Cloudflare in front as a **proxied DNS** layer. GitHub Pages stays the origin and the deploy
flow does not change; Cloudflare adds the headers on the way out.

1. Move the domain's nameservers to Cloudflare (free plan)
2. Recreate the four `A` records and the `www` `CNAME` above, **proxied** (orange cloud)
3. SSL/TLS, encryption mode: **Full (strict)**
4. SSL/TLS, Edge Certificates: enable **Always Use HTTPS** and **HSTS**
5. Rules, **Transform Rules**, Modify Response Header, apply to all requests, add:

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
Content-Security-Policy: default-src 'none'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; base-uri 'none'; form-action 'none'; frame-ancestors 'none'; upgrade-insecure-requests
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: no-referrer
Permissions-Policy: accelerometer=(), camera=(), geolocation=(), microphone=(), payment=(), usb=()
```

Roughly thirty minutes, no code change, and the header scan then comes back clean.

## DNS hardening, do this regardless

- Enable **DNSSEC** at the registrar
- Add a **CAA** record so only your CA can issue. GitHub Pages uses Let's Encrypt:
  `cloisa.com. CAA 0 issue "letsencrypt.org"` (add `"digicert.com"` too if you proxy through Cloudflare)
- Lock the domain against spoofing before you send any mail from it:
  - `TXT @        "v=spf1 -all"`
  - `TXT _dmarc   "v=DMARC1; p=reject; rua=mailto:security@cloisa.com"`
  - `TXT *._domainkey  "v=DKIM1; p="`
  - Replace the SPF record when real mail goes live. Leaving `-all` after that will silently kill
    your own outbound mail

## Editing

Two things to change first:

1. **Email addresses must actually receive mail.** `hello@cloisa.com` in `index.html` and
   `security@cloisa.com` in `.well-known/security.txt`. A dead contact address on a page asking for
   design partners is worse than no page
2. **The beachhead section** is fenced in `index.html` between
   `<!-- ==== BEACHHEAD BLOCK ... -->` comments. It currently reads telecom operators, ISPs and
   equipment distributors. Swapping verticals touches nothing else

## Local preview

```bash
python3 -m http.server 8000
```

## Check once live

- https://securityheaders.com
- https://observatory.mozilla.org
- https://www.ssllabs.com/ssltest/
- https://validator.w3.org
