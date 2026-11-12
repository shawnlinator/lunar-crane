# Threat model

## What this project does and where untrusted input enters
Self-hosted web scraping service ("TRAWL"): HTTP requests, cached browser sessions (Camoufox
Firefox), CAPTCHA/proof-of-work challenge solving, an MCP server for AI agents, a
FlareSolverr-compatible /v1 endpoint, and a challenge-aware HTTP/HTTPS forward proxy.
Untrusted input: arbitrary target URLs submitted via the scrape/MCP/proxy APIs, the fetched
web content (HTML, JS, cookies, downloads), third-party solver responses (2Captcha), Redis
cache contents, and configuration via environment variables (proxy credentials, API keys,
Redis URL).

## Components that matter most / least
- most: the API/MCP entrypoints and request-assembly code (arbitrary URL -> fetch/proxy
  connection), the JavaScript-eval and browser-session features (code run against scraped
  pages), the challenge/CAPTCHA solver pipeline, and any shell/process launch around the
  browser binary.
- less: the docs app, dashboard UI, packaging.

## How to exercise it
Point the API at hostile target URLs (SSRF to internal addresses, huge/malformed responses,
crafted JS pages); drive the MCP tools and the forward proxy with malformed requests; feed
hostile environment values (proxy URLs, API keys, Redis DSN) at startup.

## How you rate severity
- RCE (via JS eval, process launch, or dependency confusion): critical. Auth bypass on the
  API: critical. SSRF through the scrape/proxy paths: high. Credential or session-cookie
  leakage: high. DoS via unbounded fetch/queue resources: medium.

## Anything to leave alone
- The Camoufox/Firefox binaries and browser internals are out of scope for this repo (they are
  pinned third-party downloads, sha256-verified at build time).
