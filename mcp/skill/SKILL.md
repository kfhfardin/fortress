---
name: fortress-stealth-browser
description: >-
  Use when a web fetch is blocked — Cloudflare, DataDome, PerimeterX, Akamai, a 403/429,
  a CAPTCHA/"Press & Hold", "Access denied", an empty JavaScript shell, or a page whose
  data only appears after client-side rendering. Drives a real recompiled-Chromium stealth
  engine (Fortress) via the Fortress MCP to fetch, extract, crawl, recon, search, and
  automate. Also use to get clean markdown/tables/JSON from any page, or a CDP endpoint to
  run existing Playwright/Puppeteer/browser-use code through the stealth browser.
---

# Fortress stealth-browser skill

## When to use this
Reach for the `fortress` MCP tools the moment a normal fetch fails or returns junk:
- HTTP **403 / 429 / 503**, a Cloudflare "Just a moment", DataDome/PerimeterX walls,
  "Access to this page has been denied", "Press & Hold to confirm you are human".
- The page is a **JavaScript SPA** and the plain HTML is an empty shell.
- You need **structured data** (markdown, tables, or a schema-shaped record), a whole-site
  **crawl**, or the site's **private JSON API**.
- You have Playwright/Puppeteer/browser-use code that keeps getting **detected** — get a
  stealth **CDP endpoint** and point your code at it.

## Setup
```bash
pip install "tilion[mcp]"
```
Add to the client MCP config: `{ "mcpServers": { "fortress": { "command": "tilion-mcp" } } }`

## Tool cheat-sheet (pick by intent)
- **Blocked / need the page** → `fetch_protected_page(url)` → `{status, title, text, blocked}`.
- **Clean content** → `read_page(url)` (markdown) or `extract_page(url, schema?)` (record).
- **A file (PDF/DOCX/XLSX/CSV)** → `extract_document(source)`.
- **Whole site** → `crawl_site(url, depth, max_pages)`.
- **Find the data source** → `recon_site_apis(url)` → discovered XHR/JSON endpoints.
- **Search the web** → `search_web(query, count)` (real browser, no SERP key).
- **Interact** → `page_elements` → `click_button` / `fill_field` / `press_key` / `wait_for`
  / `evaluate_js`; check with `current_page`.
- **Multi-step flow** (login, paginate, infinite-scroll, checkout) →
  `list_browser_tasks()` then `run_browser_task(task, url, params?)`.
- **Auth reuse** → `save_profile(name)` / `load_profile(name)`.
- **Capture** → `screenshot_page` / `save_page(format=pdf|html|text|png)` / `download_file`.
- **Bring your own automation** → `get_stealth_cdp_endpoint()` → a CDP url for
  Playwright / Puppeteer / browser-use / Crawl4AI.

## Good habits
- Prefer `extract_page`/`read_page` over dumping raw HTML — outputs are capped and clean.
- For an authenticated site, `load_profile` before fetching so cookies + solved challenges
  are reused.
- Reads are safe/auto-approved; `run_browser_task`, `save_page`, `download_file` write or
  act — confirm intent first.
- A great fingerprint on a datacenter IP can still be blocked by the hardest sites; on a
  residential IP the local engine clears most walls. Hosted residential egress is coming soon.

## Reference
Full docs and the 26-tool table live in the repo's [`mcp/README.md`](../README.md).
