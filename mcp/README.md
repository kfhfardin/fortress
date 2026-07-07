# Fortress MCP — a stealth browser for AI agents

> **Beta** · 28 tools · runs **local & free** · **hosted cloud coming soon**

An [MCP](https://modelcontextprotocol.io) server that gives any AI agent the **Fortress
stealth engine** the moment it gets blocked. When a fetch hits Cloudflare, DataDome,
PerimeterX, a 403, or a CAPTCHA, the agent calls these tools and gets the page — driving a
real, recompiled Chromium on your own machine and IP.

<p align="center"><img src="demo.gif" alt="Same site, same prompt: a vanilla browser is blocked by PerimeterX while an agent with the Fortress MCP returns clean JSON" width="760"/></p>

<sub><i>Real, dated run against <b>stockx.com</b> (PerimeterX). A stock browser gets <b>HTTP 403 — “Access denied”</b>; an agent with the Fortress MCP returns clean JSON. Reproduce with the demo scripts in the framework repo.</i></sub>

## Install

```bash
pip install "tilion[mcp]"        # pulls the Fortress engine (tilion-fortress) automatically
tilion-mcp                       # or:  python -m tilion.mcp   (stdio transport)
```

The MCP server is a thin, open wrapper (BSD-3) over the `tilion` framework, which drives the
Fortress engine. On Linux and Windows the stealth Chromium downloads on first run and is cached
locally. On macOS the engine runs as a Docker image instead. Read the macOS section below before
you start.

## macOS setup (Apple Silicon and Intel)

There is no native macOS engine binary yet, so on a Mac the Fortress engine runs as the official
Docker image (`tilion/fortress:149`). `tilion-mcp` itself runs natively in Python. Only the
browser engine is containerised, and the server starts and stops that container for you. All you
supply is a running Docker daemon.

### Setup

1. Install the MCP and engine wrapper:
   ```bash
   pip install "tilion[mcp]"
   ```
2. Give it a Docker daemon. Colima is lighter than Docker Desktop and needs no license or GUI:
   ```bash
   brew install colima docker
   # Apple Silicon: vz + Rosetta runs the amd64 engine image fast.
   colima start --cpu 4 --memory 6 --disk 30 --vm-type=vz --vz-rosetta
   # Intel Macs: plain `colima start` works. Docker Desktop is also fine if you already run it.
   ```
   Give the VM at least 4 CPUs and 4 GB of RAM, since it runs a real Chromium.
3. Register the server with your client. For Claude Code:
   ```bash
   claude mcp add fortress -- tilion-mcp
   ```
4. The first tool call pulls the image once (about 300 MB), then the container stays warm and
   calls are fast. Call `get_egress_info` to confirm the engine is alive; it returns the public
   IP the target sees.

### What not to do on macOS

| Pitfall | What happens, and the fix |
|---|---|
| Setting `FORTRESS_CHANNEL=latest` | That channel points at `tilion/fortress:151`, which is not published to Docker Hub, so the pull 404s and the engine never starts. Stay on the default `stable` channel, `tilion/fortress:149`. Native Linux and Windows are unaffected, since they fetch the GitHub release rather than the image. |
| A leftover Docker Desktop credential helper | `docker pull` fails with `docker-credential-desktop … executable file not found`. Open `~/.docker/config.json` and delete the `"credsStore": "desktop"` line. |
| Worrying about the platform warning | `The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8)` is expected on Apple Silicon and harmless. The amd64 engine runs under Rosetta, a little slower than native Linux. |
| Running two `tilion-mcp` servers at once | Each launches a Fortress container on host port `9222`, so the second fails with `docker … exit status 125` (port already allocated). Run one server per machine. |
| Assuming Colima survives a reboot | If tool calls fail with a Docker error after a reboot, run `colima start`. To start it at login, run `brew services start colima`. |
| Setting `TILION_MCP_HEADLESS=0` for a visible window | This has no effect on macOS. The containerised engine is headless only. A visible window needs the native Linux or Windows binary. |

### Still getting blocked on a hard target

Your Mac's home or office IP is residential, which suits most sites. For the hardest targets, or
when running from a datacenter, route the engine through a residential proxy. Set
`TILION_PROXY=http://user:pass@host:port` (and optionally `TILION_REGION=us`) before starting the
server, then confirm with `get_egress_info`.

## Add to your client

**Claude Desktop / Cursor** — add to the MCP config:

```json
{ "mcpServers": { "fortress": { "command": "tilion-mcp" } } }
```

**Cline / Windsurf** (VS Code settings → MCP servers):
```json
{ "fortress": { "command": "tilion-mcp" } }
```

If `tilion-mcp` isn't on PATH, use `"command": "python", "args": ["-m", "tilion.mcp"]`.

## The 28 tools

| Tool | What the agent uses it for |
|---|---|
| `fetch_protected_page` | get a page behind Cloudflare / DataDome / 403 / CAPTCHA |
| `read_page` | clean reader-mode **markdown of any page** (+ tables) |
| `extract_page` | markdown + tables + metadata (or a schema-shaped record) |
| `extract_document` | extract a PDF/DOCX/XLSX/CSV/HTML file (path or URL) → markdown |
| `page_elements` | the page's buttons / links / fields / headings |
| `click_button` · `fill_field` · `press_key` | drive a form by visible text / selector / key |
| `current_page` · `get_page_html` · `evaluate_js` · `wait_for` | inspect / script / wait on the working page |
| `crawl_site` | crawl a whole site (auto-handles SPA/JS) → pages + sitemap |
| `recon_site_apis` | reverse-engineer a site's private XHR/JSON API (secret-scrubbed) |
| `run_browser_task` · `list_browser_tasks` | 20 multi-step flows: login, paginate, infinite-scroll, checkout… |
| `search_web` | web search through the stealth browser (no SERP API) |
| `screenshot_page` · `save_page` · `download_file` | capture PNG / export pdf·html·text / download a file |
| `get_cookies` · `save_profile` · `load_profile` | read cookies · persist/restore an authenticated session |
| `list_tabs` · `close_tab` | manage open tabs |
| `get_stealth_cdp_endpoint` | a CDP url to point your OWN browser-use / Playwright / Puppeteer at |
| `solve_captcha` | detect + solve a reCAPTCHA/hCaptcha/Turnstile (needs `CAPTCHA_API_KEY`) |
| `get_egress_info` | report proxy/region + the real public IP the target sees (verify residential egress) |

Tools are **annotated** (`readOnlyHint` / `destructiveHint`) so clients auto-approve reads
and gate writes. Every tool is **timeout- and SSRF-guarded**, caps its output, and returns a
structured error instead of hanging. The browser is **pre-warmed at startup**, so the first
call is ~100 ms.

## Why reach for it — benchmarks

Real head-to-head — an agent with only its built-in web fetch vs. the same task through the
Fortress MCP:

| Task | Built-in web fetch | **Fortress MCP** |
|------|--------------------|------------------|
| Reddit r/programming titles | ✗ 0 items | **✓ 26 titles** |
| JS-rendered page (quotes) | ✗ 0 quotes | **✓ 10 quotes** |
| Wikipedia article | ✗ 403 to bots | **✓ 52 k markdown** |
| Hacker News top stories | ✓ 30 · 24 s | **✓ 30 · 2 s** (~12× faster) |

Fingerprint suites: **Sannysoft all-green · CreepJS 0% headless · BrowserScan “Normal.”**

## Configuration (env)

| Env var | Default | Effect |
|---|---|---|
| `TILION_MCP_PREWARM` | `1` | boot the browser at startup; `0` = lazy |
| `TILION_MCP_HEADLESS` | `1` | `0` to show a visible window |
| `TILION_ALLOW_PRIVATE_EGRESS` | `0` | `1` to allow localhost / private IPs (SSRF guard off) |
| `TILION_MCP_TOOL_TIMEOUT` | `120` | per-tool wall-clock cap (seconds) |
| `TILION_BASE_URL` / `TILION_API_KEY` | — | hosted mode (**coming soon**) |
| `TILION_PROXY` | — | egress proxy `http://user:pass@host:port` (residential/mobile) |
| `TILION_REGION` | — | egress region hint (e.g. `us`) — aligns timezone/locale to the IP |
| `CAPTCHA_API_KEY` | — | solver key; `fetch` then auto-solves + `solve_captcha` works |
| `CAPTCHA_PROVIDER` | `2captcha` | `2captcha` \| `anticaptcha` \| `capsolver` |

## How it works

`tilion-mcp` → the `tilion` framework (local mode) → attaches over CDP to the Fortress
engine. Stealth is applied **natively in the C++ engine**, so there's no detectable JS
injection. One warm browser backs every tool for the server's lifetime.

Registry manifests: [`server.json`](server.json) (MCP registry) · [`smithery.yaml`](smithery.yaml) (Smithery).
Agent skill: [`skill/SKILL.md`](skill/SKILL.md).

## License

BSD-3-Clause (the MCP server and framework funnel). The engine binary ships via
`tilion-fortress`. Hosted cloud with residential egress is coming soon.
