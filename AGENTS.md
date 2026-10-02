# Set up Fortress v3

This file is self-contained: it is the only file a fresh sandbox needs.
Do not fetch another guide or assume a helper script, cache, engine, or license
is already present. Download public packages and release assets as instructed.

These instructions are for an agent helping a user install and use Fortress
**v153.0.8010.36**. Complete installation, activation, engine startup, and the
requested integration. Return to the user when email verification or account
approval requires their action, then resume from that step. A running browser
alone does not mean v3 setup is complete.

Fortress is a Chromium engine accessed through the Chrome DevTools Protocol
(CDP). Connect the user's existing automation to it. The target is
[Fortress v3](https://github.com/tiliondev/fortress/releases/tag/v153.0.8010.36),
which uses the Fortress Source Available License 1.1. Consult the release's
license information for development and production use.

## 1. Detect the host and install once

Determine OS, architecture, current user, shell, Python availability, and the
requested integration. Inspect an existing installation before downloading
another; preserve working installations and unrelated files.

| Host | Native artifact | Dedicated instructions |
| --- | --- | --- |
| Linux x64 | `fortress-v153-linux-x64.tar.gz` | [Linux](#linux-commands) |
| Linux arm64 | `fortress-v153-linux-arm64.tar.gz` | [Linux](#linux-commands) |
| Linux x86 / armhf, 32 bit | `fortress-v153-linux-x86.tar.gz` / `fortress-v153-linux-armhf.tar.gz` | [Linux](#linux-commands) |
| Windows x64 | `fortress-v153-win-x64.zip` | [Windows / PowerShell](#windows-commands) |
| macOS Apple Silicon | `fortress-v153-mac-arm64.tar.gz` | [macOS](#macos-commands) |
| macOS Intel, Windows ARM64, other hosts | No matching native artifact in this release | Use a supported remote host; see section 5 |

Follow the matching command section embedded below; all required commands
are in this file. WSL is a separate Linux installation: activate and run as the
same Linux user. Links in the references are optional background information.

Use the pinned native archive. Linux's default native/CDP path below uses its
bundled activator with system Python; no separate SDK is required. When a
separate CLI is needed (including Windows), use
`tilion-fortress==153.0.8010.36.post1` in a dedicated virtual environment.
Persist the selected commands on **user PATH**, without an administrator-wide
PATH change. Install Playwright/Puppeteer only when that integration is needed.

Release details verified on 2026-10-02:

- The published Python wheel exposes **`tilion-fortress`** and `tilion-activate`,
  not `tilion`. Its module is **`tillion_fortress`**, with two consecutive `l`s.
  Use the commands here or the venv's absolute Python path with
  `-m tillion_fortress`. Do not overwrite an existing `tilion` command.
- The Windows bundle contains **`tillion.cmd`** and no bundled activator.
  Install the activation package even when using the portable engine.
- npm's published `tilion-fortress` is still `151.0.7910`. Node projects should
  connect over CDP to the separately installed v3 engine.
- The hashes in the release description differ from the current **`SHA256SUMS`
  asset**. Download that file alongside the pinned archive and verify its exact
  entry before extraction. Stop on missing hashes or mismatches; never skip
  verification or replace the requested release with `latest`.

Keep each extracted bundle together and record its actual launcher path.

## 2. Save paths and complete activation with the user

Use the same OS account for activation and engine startup. Resolve these paths
to absolute paths for subsequent sessions:

| Purpose | Linux / macOS | Windows |
| --- | --- | --- |
| License secret | `~/.tilion/license.jwt` | `%USERPROFILE%\.tilion\license.jwt` |
| Activation CLI directory on PATH | `~/.tilion/cli/bin` | `%USERPROFILE%\.tilion\cli\Scripts` |
| Engine installation | `~/.tilion/installs/v153.0.8010.36` | `%USERPROFILE%\.tilion\installs\v153.0.8010.36` |
| Profiles and logs | `~/.tilion/profiles`, `~/.tilion/logs` | `%USERPROFILE%\.tilion\profiles`, `%USERPROFILE%\.tilion\logs` |
| Agent's resume record | `~/.tilion/setup.json` | `%USERPROFILE%\.tilion\setup.json` |

The Linux native path instead exposes `fortress-activate` and `fortress-v3`
from `~/.tilion/bin`. For that path, substitute `fortress-activate` for
`tilion-fortress` in the activation/license commands below; engine startup
still uses the verified native launcher. These commands use the same key path.

`setup.json` is an agent-maintained record, not an engine configuration file.
Merge the release, platform, absolute CLI/launcher/license/profile/log paths,
CDP endpoint, owned process ID, and stage into it. Stages are `installed`,
`awaiting_user`, `activated`, `running`, and `verified`. Do not store the JWT,
email verification link, or approval codes in this record. Do not create an
empty license file as a placeholder.

First run:

```sh
tilion-fortress license status --json
```

Inspect `licensed`, `mode`, `exp`, and `source`; exit code zero alone does not
mean licensed. This command decodes claims and checks expiry. The **engine**
validates the signature when it starts. `TILION_LICENSE_KEY` overrides the file;
check its presence without printing its value. An expired environment key can
shadow a fresh file. Resolve that runtime override without deleting the user's
persistent credentials.

### Email/device approval and resumption

If activation is needed, run:

```sh
tilion-fortress activate --headless
```

When an agent captures stdout through a pipe, enable unbuffered Python output
so the approval link appears immediately: prefix the command with
`PYTHONUNBUFFERED=1` on Linux/macOS; on Windows set `$env:PYTHONUNBUFFERED = '1'`
in that process session. Alternatively use the activation environment's Python
with `-u -m tillion_fortress activate --headless` on any OS.

1. Show the user the exact verification URL and user code printed by the CLI:
   **"Open this link, sign in with your email or preferred account, complete
   any email verification, and approve this device. Tell me when you've
   finished; I'll verify the saved key and start Fortress."**
2. Mark `awaiting_user`. Keep the activation process alive while the user
   completes the flow; it polls and writes `license.jwt` itself. Do not access
   their inbox or send email without explicit permission. Do not treat elapsed
   time or the user's message alone as proof of successful activation.
3. Check process success, the nonempty saved file, and license status. Apply
   the selected OS guide's permissions. Never fabricate a JWT or ask the user
   to paste the secret into chat.
4. Mark `activated` and continue to engine startup automatically.

If approval times out or the agent session ends, check status/file first when
resuming. If activation did not finish, run it again and show a **new** URL/code.
The CLI does not persist device-flow sessions. Do not reinstall the engine or
reuse an expired code. Interactive `tilion-fortress activate` opens a browser
instead; the same completion checks apply.

For CI or a service account, use an existing key injected by its secret manager
as `TILION_LICENSE_KEY`; a local license file is then optional. Keep keys out of
repositories, shell startup files, command transcripts, MCP configuration, and
PATH. For an expired file, try `tilion-fortress license refresh`, check status,
and return to activation if necessary. Do not assume refresh happened or use
`license logout` as a routine repair step.

## 3. Start the actual engine and verify the result

After activation, use the **native launcher** in the OS guide and capture
stdout/stderr in the protected log directory. The pinned Python CLI's engine
launch suppresses engine output; the native path keeps fallback notices visible.

Inspect port 9222 before starting. Reuse a running engine only after checking
its executable, version, profile, and ownership. Otherwise select a free port
and use it consistently. Never kill an unrelated browser or trust any process
that happens to answer on 9222. Use a separate profile per concurrent engine.

Poll `http://127.0.0.1:9222/json/version` with a finite deadline, for example
60 seconds. Require `webSocketDebuggerUrl` and Chromium version
`153.0.8010.36` in `Browser`. Inspect startup output for license rejection or
v1 fallback. A CDP response or matching Chromium version alone does not prove
v3 activation.

Run the selected integration's smoke check below. Mark `verified` only when
download integrity, activation, engine startup, and integration have succeeded.
If license acceptance cannot be established from activation and engine output,
report that limitation instead of asserting full v3 operation. Keep the
requested engine running. Report the executable, CDP URL, license **path**,
and log path, never the license value.

## 4. Connect the user's integration

These examples work on all supported OSes. Use the project's existing dependency
environment. No second Playwright browser download is needed for CDP.

**Python / Playwright** (`python -m pip install playwright` if needed):

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.connect_over_cdp("http://127.0.0.1:9222")
    page = browser.contexts[0].new_page()
    page.goto("data:text/html,<title>Fortress ready</title>")
    assert page.title() == "Fortress ready"
    page.close()
    browser.close()  # Disconnect this client after the check.
```

**Node / Playwright** (`npm install playwright` if needed):

```js
import { chromium } from "playwright";
const browser = await chromium.connectOverCDP("http://127.0.0.1:9222");
const page = await browser.contexts()[0].newPage();
await page.goto("data:text/html,<title>Fortress ready</title>");
if (await page.title() !== "Fortress ready") throw new Error("Smoke check failed");
await page.close();
await browser.close();
```

**Node / Puppeteer** (`npm install puppeteer-core` if needed):

```js
import puppeteer from "puppeteer-core";
const browser = await puppeteer.connect({ browserURL: "http://127.0.0.1:9222" });
const page = await browser.newPage();
await page.goto("data:text/html,<title>Fortress ready</title>");
if (await page.title() !== "Fortress ready") throw new Error("Smoke check failed");
await page.close();
browser.disconnect();
```

For an agent framework, configure its external CDP connection using the
installed version's supported options. Verify it uses this endpoint instead
of launching stock Chromium or an older SDK engine.

**MCP:** `tilion-fortress mcp` delegates to the separate `tilion` framework;
the activation package alone does not provide a server. Consult
[MCP instructions](mcp/README.md) and the installed server's configuration.
Some MCP documentation still uses an older Docker engine. Verify that the
server selects this pinned, activated engine before reporting v3 integration.
If it cannot, use direct CDP and report the MCP limitation. GUI client configs
should use absolute executable paths because they may not inherit shell PATH.
Keep secrets out of those configs.

## 5. Remote hosts, containers, and recovery

On an unsupported host, install on a supported Linux host using this same flow.
Keep remote CDP on loopback and forward it:

```sh
ssh -N -L 9222:127.0.0.1:9222 user@fortress-host
```

If local 9222 is occupied, forward `9223:127.0.0.1:9222` and connect locally to
9223. Activate on the runtime host; the user can approve its URL on their laptop.
The saved key belongs to the remote runtime user.

Docker is optional. Verify the image's engine and activation support, pin its
digest, provision the container user's key through a secret manager or read-only
mount, and publish CDP on loopback only. Neither `latest` nor the host's saved
key establishes container v3 activation. This guide asserts no v3 image digest;
use the pinned native bundle if the container cannot be verified.

For installation testing, use a clean OS container with only this file copied
in. The sandbox test contract below uses free rootless Podman in WSL/Linux
and works independently of Docker Desktop.
Record prerequisite installation, downloads, approval wait, startup, and the
integration check separately. A prepared Fortress image is not a first-install test.

| Symptom | Next step |
| --- | --- |
| Activation command missing | Resolve PATH with `command -v tilion-fortress` or `Get-Command tilion-fortress`; use the pinned CLI's absolute path. |
| Approval pending or expired | Resume section 2; issue a fresh flow only when needed. |
| Saved key but v1 fallback | Check effective user/home, expiry, environment override, and engine output. Status JSON is not signature verification. |
| Wrong engine version | Check the launcher, port owner, and integration defaults. |
| Checksum mismatch | Stop before extraction/execution and report the asset and mismatch. |
| CDP does not start | Inspect stderr, missing libraries, profile locks, and port ownership. |
| GUI client cannot find CLI | Restart the client after PATH changes or use an absolute executable path. |

## Operating rules

- Use raw CDP, not ChromeDriver. Do not add JavaScript fingerprint patches,
  `puppeteer-stealth`, or `undetected-chromedriver`.
- Keep coherent engine defaults. Do not pass `--user-agent`; use documented
  `--uxr-*` overrides when needed.
- Run as an ordinary user, keep CDP on loopback, and use dedicated profiles.
  Do not disable the browser sandbox as a routine installation fix.
- Preserve keys and profiles between runs. Stop only processes owned by this
  setup when cleanup is requested.
- Report checks and remaining blockers; installation alone is not completion.

## References

- [Pinned release](https://github.com/tiliondev/fortress/releases/tag/v153.0.8010.36)
- [SHA256SUMS](https://github.com/tiliondev/fortress/releases/download/v153.0.8010.36/SHA256SUMS)
- [Pinned activation package](https://pypi.org/project/tilion-fortress/153.0.8010.36.post1/)

## Linux commands

Follow the main flow above in order. Run as the ordinary user
who will own the engine and key, including inside WSL or on an SSH host.

### Default native/CDP setup

For a native engine and direct CDP connection, use this smaller dependency
path. The verified Linux bundle includes `tilion-activate` and
`tilion_activate.py`; the latter uses only Python's standard library. Do not
install a second CLI or a new automation framework merely to launch the engine.
If Python/Playwright is requested, use the separate CLI/integration path below
and include those extra dependencies in the reported timing.

In a fresh Ubuntu 24.04 or Debian 12 sandbox, first create the ordinary account
using `sandbox-user` below and run this small bootstrap as container root:

<!-- recipe: native-bootstrap -->
```bash
set -e
apt-get update -qq
DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends curl ca-certificates
```

Then run two jobs concurrently: the `linux-engine` download recipe as the
ordinary user, and `native-runtime` below as root. Wait for both to succeed.

<!-- recipe: native-runtime -->
```bash
set -e
. /etc/os-release
case "$ID:$VERSION_ID" in
  ubuntu:24.04) fortress_libraries='libatk-bridge2.0-0t64 libcups2t64 libasound2t64' ;;
  debian:12) fortress_libraries='libatk-bridge2.0-0 libcups2 libasound2' ;;
  *) printf 'Unsupported automatic dependency recipe\n' >&2; exit 1 ;;
esac
DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  python3 fontconfig libnss3 libxkbcommon0 $fortress_libraries \
  libgbm1 libxcomposite1 libxdamage1 libxfixes3 libxrandr2 libpango-1.0-0 libcairo2
```

Run `linux-preflight`, then expose dedicated commands without replacing an
existing `tilion` command. On a subsequent run, inspect and reuse these links
only if they resolve to this verified release; never overwrite unrelated files.

<!-- recipe: native-path -->
```bash
set -e
umask 077
mkdir -p "$HOME/.tilion/bin"
chmod 700 "$HOME/.tilion"
fortress_bundle="$HOME/.tilion/installs/v153.0.8010.36/fortress-v153"
test -x "$fortress_bundle/tilion-activate"
test -f "$fortress_bundle/tilion_activate.py"
ln -s "$fortress_bundle/tilion-activate" "$HOME/.tilion/bin/fortress-activate"
ln -s "$fortress_bundle/tilion" "$HOME/.tilion/bin/fortress-v3"
touch "$HOME/.bashrc"
if ! grep -Fqx 'export PATH="$HOME/.tilion/bin:$PATH"' "$HOME/.bashrc"; then
  printf '\nexport PATH="$HOME/.tilion/bin:$PATH"\n' >> "$HOME/.bashrc"
fi
export PATH="$HOME/.tilion/bin:$PATH"
fortress-activate activate --help
```

For another shell, persist this directory using that shell's startup mechanism.
Start `PYTHONUNBUFFERED=1 fortress-activate activate --headless`, hand the URL
to the user, and keep it polling. After success, run the following as the same
ordinary user. It validates the saved claims, launches the native engine,
opens a page through CDP's HTTP interface, checks its title, closes that page,
and checks PATH/license access in a new shell. It does not claim to test an
uninstalled Playwright, Puppeteer, or MCP client.

<!-- recipe: native-verify -->
```bash
set -e
umask 077
export PATH="$HOME/.tilion/bin:$PATH"
test -z "${TILION_LICENSE_KEY+x}"
test -s "$HOME/.tilion/license.jwt"
chmod 600 "$HOME/.tilion/license.jwt"
python3 - <<'PY'
import json, subprocess
status = json.loads(subprocess.check_output(['fortress-activate', 'license', 'status', '--json']))
assert status['licensed'] and status['mode'] == 'v3'
print('Saved license is live; engine will verify its signature')
PY
fortress_launcher="$HOME/.tilion/installs/v153.0.8010.36/fortress-v153/tilion"
mkdir -p "$HOME/.tilion/profiles/v153-default" "$HOME/.tilion/logs"
fortress_log="$HOME/.tilion/logs/native-$(date +%Y%m%d-%H%M%S).log"
nohup "$fortress_launcher" --headless=new \
  --remote-debugging-address=127.0.0.1 --remote-debugging-port=9222 \
  --user-data-dir="$HOME/.tilion/profiles/v153-default" \
  > "$fortress_log" 2>&1 < /dev/null &
export FORTRESS_SETUP_PID=$! FORTRESS_SETUP_LOG="$fortress_log"
python3 - <<'PY'
import json, os, stat, time, urllib.parse, urllib.request
from pathlib import Path
base = 'http://127.0.0.1:9222'
def request(path, method='GET'):
    with urllib.request.urlopen(urllib.request.Request(base + path, method=method), timeout=2) as r:
        return r.read()
deadline = time.monotonic() + 60
while True:
    try:
        version = json.loads(request('/json/version'))
        assert version['Browser'] == 'Chrome/153.0.8010.36'
        assert version['webSocketDebuggerUrl']
        break
    except (OSError, AssertionError):
        if time.monotonic() >= deadline:
            raise
        time.sleep(.1)
target = json.loads(request('/json/new?' + urllib.parse.quote('data:text/html,<title>Fortress ready</title>', safe=''), 'PUT'))
try:
    deadline = time.monotonic() + 10
    while True:
        pages = json.loads(request('/json/list'))
        if any(p['id'] == target['id'] and p.get('title') == 'Fortress ready' for p in pages):
            break
        if time.monotonic() >= deadline:
            raise RuntimeError('CDP page title did not match')
        time.sleep(.1)
finally:
    request('/json/close/' + target['id'])
assert json.loads(request('/json/version'))['Browser'] == version['Browser']
root = Path.home() / '.tilion'
assert stat.S_IMODE((root / 'license.jwt').stat().st_mode) == 0o600
record = root / 'setup.json'
state = json.loads(record.read_text()) if record.exists() else {}
state.update(release='v153.0.8010.36', platform='linux', stage='running',
    launcher=str(root / 'installs/v153.0.8010.36/fortress-v153/tilion'),
    cli=str(root / 'bin/fortress-activate'), license_path=str(root / 'license.jwt'),
    profile=str(root / 'profiles/v153-default'), log=os.environ['FORTRESS_SETUP_LOG'],
    pid=int(os.environ['FORTRESS_SETUP_PID']), cdp=base)
record.write_text(json.dumps(state, indent=2))
print(version['Browser'] + ': CDP page and license permissions passed')
PY
bash -ic 'command -v fortress-activate; fortress-activate license status --json' </dev/null
if grep -Ei 'invalid.*licen[cs]e|licen[cs]e.*(invalid|reject)|falling back|fallback.*v1' "$fortress_log"; then
  printf 'Engine reported a licensing failure\n' >&2; exit 1
fi
python3 - <<'PY'
import json
from pathlib import Path
record = Path.home() / '.tilion/setup.json'
state = json.loads(record.read_text())
state['stage'] = 'verified'
record.write_text(json.dumps(state, indent=2))
print('Native/CDP setup verified; engine remains running')
PY
```

The generic ownership/port checks in section 3 still apply. The fresh sandbox
contract guarantees no previous engine; an existing machine does not.

### Prerequisites on Ubuntu 24.04 and Debian 12

A fresh Linux container may not include Python, curl, or Chromium's runtime
libraries. Run these package-manager blocks as root (or use `sudo` on an
existing machine). Run downloads, activation, and the engine as the ordinary
runtime user. First install the small bootstrap dependencies:

<!-- recipe: linux-prerequisites -->
```bash
set -e
apt-get update -qq
DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  python3 python3-venv curl ca-certificates
```

Then start the following three independent jobs concurrently: (1) the runtime
libraries below as root, (2) engine download and verification as the runtime
user, and (3) CLI/PATH setup as that user. After CLI setup, start unbuffered
activation immediately and install the requested integration while approval
is pending. Wait for all installation jobs to succeed before dependency
preflight or engine launch. Never run two apt/dpkg jobs at once.

<!-- recipe: linux-runtime-libraries -->
```bash
set -e
. /etc/os-release
case "$ID:$VERSION_ID" in
  ubuntu:24.04) fortress_libraries='libatk-bridge2.0-0t64 libcups2t64 libasound2t64' ;;
  debian:12) fortress_libraries='libatk-bridge2.0-0 libcups2 libasound2' ;;
  *) printf 'No tested automatic prerequisite recipe for %s %s\n' "$ID" "$VERSION_ID" >&2; exit 1 ;;
esac
DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  fontconfig \
  libnss3 libxkbcommon0 $fortress_libraries \
  libgbm1 libxcomposite1 libxdamage1 libxfixes3 libxrandr2 \
  libpango-1.0-0 libcairo2
```

For a free disposable test environment, see the sandbox-test section below.
Do not mount a host home directory or inject a preinstalled engine into a
first-installation test.

### Install activation CLI and PATH

Use Python 3.10+ with `venv`/`pip`, `curl`, `tar`, and `sha256sum`. Install missing
prerequisites through the detected distribution's package manager. Inspect and
reuse an existing dedicated environment if it already has the pinned package.

<!-- recipe: linux-cli -->
```bash
set -e
umask 077
mkdir -p "$HOME/.tilion"
chmod 700 "$HOME/.tilion"
python3 -m venv "$HOME/.tilion/cli"
"$HOME/.tilion/cli/bin/python" -m pip install --disable-pip-version-check --no-compile 'tilion-fortress==153.0.8010.36.post1'
export PATH="$HOME/.tilion/cli/bin:$PATH"
tilion-fortress --version
tilion-fortress activate --help
```

Persist PATH in the actual shell's startup file. For Bash:

<!-- recipe: linux-path -->
```bash
touch "$HOME/.bashrc"
if ! grep -Fqx 'export PATH="$HOME/.tilion/cli/bin:$PATH"' "$HOME/.bashrc"; then
  printf '\nexport PATH="$HOME/.tilion/cli/bin:$PATH"\n' >> "$HOME/.bashrc"
fi
```

For Zsh use `.zshrc`; for Fish run `fish_add_path "$HOME/.tilion/cli/bin"` in
Fish. Preserve existing configuration and verify resolution in a new terminal.
Services/GUI clients should use an absolute executable path and the same user
home as activation.

### Download and verify the engine

Select for the OS userspace architecture, not just the CPU. If userspace and
kernel architectures differ, resolve that before running this selection.
`armhf` requires ARM hard-float userspace.

<!-- recipe: linux-engine -->
```bash
set -e
case "$(uname -m)" in
  x86_64|amd64) fortress_arch=x64 ;;
  aarch64|arm64) fortress_arch=arm64 ;;
  i386|i486|i586|i686) fortress_arch=x86 ;;
  armv7l|armv7|armhf) fortress_arch=armhf ;;
  *) printf 'No native artifact for this architecture\n' >&2; exit 1 ;;
esac
fortress_asset="fortress-v153-linux-${fortress_arch}.tar.gz"
fortress_base='https://github.com/tiliondev/fortress/releases/download/v153.0.8010.36'
fortress_download="$(mktemp -d)"
curl --fail --location "$fortress_base/$fortress_asset" -o "$fortress_download/$fortress_asset"
curl --fail --location "$fortress_base/SHA256SUMS" -o "$fortress_download/SHA256SUMS"
(
  cd "$fortress_download"
  tr -d '\r' < SHA256SUMS | awk -v name="$fortress_asset" '$2 == name {print}' > selected.sha256
  test "$(wc -l < selected.sha256)" -eq 1
  sha256sum --check selected.sha256
)
fortress_install="$HOME/.tilion/installs/v153.0.8010.36"
mkdir -p "$fortress_install"
tar -xzf "$fortress_download/$fortress_asset" -C "$fortress_install"
fortress_launcher="$(find "$fortress_install" -type f -name tilion -print -quit)"
test -n "$fortress_launcher"
test -x "$fortress_launcher"
printf 'Engine launcher: %s\n' "$fortress_launcher"
```

Before reusing an installation directory, verify its recorded provenance; do
not extract over a running or unverified copy. Retain the archive/checksum and
record the resolved launcher. Stop on a missing launcher or checksum failure.

After both download/extraction and runtime-library installation have succeeded,
check the verified engine's dependencies. Restore the launcher path in a new
shell before running this check:

<!-- recipe: linux-preflight -->
```bash
set -e
fortress_launcher="$HOME/.tilion/installs/v153.0.8010.36/fortress-v153/tilion"
test -x "$fortress_launcher"
ldd "$(dirname "$fortress_launcher")/chrome" | awk '/not found/ {missing=1; print} END {exit missing}'
```

### Activate and protect the key

Return to [activation](#2-save-paths-and-complete-activation-with-the-user).
Run `PYTHONUNBUFFERED=1 tilion-fortress activate --headless` and hand its URL/code
to the user. Unbuffered output makes the link visible immediately when an agent
captures the CLI through a pipe.
After the CLI saves the file:

<!-- recipe: linux-license -->
```bash
test -s "$HOME/.tilion/license.jwt"
chmod 700 "$HOME/.tilion"
chmod 600 "$HOME/.tilion/license.jwt"
tilion-fortress license status --json
```

For a secret-manager environment key, check status/source instead of requiring
a local file. Activate on the runtime host and never display the key.

### Start and integrate

Complete the main guide's port/ownership checks first. Restore the launcher
variable from the setup record when resuming in a new shell.

<!-- recipe: linux-start -->
```bash
umask 077
mkdir -p "$HOME/.tilion/profiles/v153-default" "$HOME/.tilion/logs"
fortress_log="$HOME/.tilion/logs/v153-$(date +%Y%m%d-%H%M%S).log"
nohup "$fortress_launcher" --headless=new \
  --remote-debugging-address=127.0.0.1 --remote-debugging-port=9222 \
  --user-data-dir="$HOME/.tilion/profiles/v153-default" \
  > "$fortress_log" 2>&1 < /dev/null &
fortress_pid=$!
printf 'Process: %s; log: %s\n' "$fortress_pid" "$fortress_log"
```

Record the PID, log, and profile. Poll
`curl --fail --max-time 2 http://127.0.0.1:9222/json/version` for up to 60 seconds,
then complete [engine verification](#3-start-the-actual-engine-and-verify-the-result)
and the main guide's Python, Node, or MCP integration check. SSH clients use
the forwarding instructions there.

For missing shared libraries, inspect stderr and `ldd` on the verified browser
binary; install the missing libraries through the distribution's package
manager. Do not download arbitrary `.so` files or add `--no-sandbox` as a blanket
fix. A headed browser needs a working display; headless does not.

## Windows commands

Follow the main flow above in order. The native build supports
Windows x64. Use PowerShell and Python 3.10+ for activation. Windows ARM64 has no
native artifact in this release; use a supported remote host instead of assuming
x64 emulation works.

### Install activation CLI and user PATH

Inspect/reuse an existing dedicated environment first. `py -3` must select a
supported Python; if only `python` is available, verify it and use that instead.

<!-- recipe: windows-cli -->
```powershell
$ErrorActionPreference = 'Stop'
$fortressRoot = Join-Path $env:USERPROFILE '.tilion'
$fortressVenv = Join-Path $fortressRoot 'cli'
New-Item -ItemType Directory -Force -Path $fortressRoot | Out-Null
py -3 -m venv $fortressVenv
if ($LASTEXITCODE -ne 0) { throw 'Could not create activation environment' }
$fortressPython = Join-Path $fortressVenv 'Scripts\python.exe'
$fortressBin = Join-Path $fortressVenv 'Scripts'
$fortressCli = Join-Path $fortressBin 'tilion-fortress.exe'
& $fortressPython -m pip install --disable-pip-version-check --no-compile 'tilion-fortress==153.0.8010.36.post1'
if ($LASTEXITCODE -ne 0) { throw 'Activation package installation failed' }
$fortressUserPath = [Environment]::GetEnvironmentVariable('Path', 'User')
if (($fortressUserPath -split ';') -notcontains $fortressBin) {
    [Environment]::SetEnvironmentVariable('Path', "$fortressBin;$fortressUserPath", 'User')
}
if (($env:Path -split ';') -notcontains $fortressBin) {
    $env:Path = "$fortressBin;$env:Path"
}
& $fortressCli --version
& $fortressCli activate --help
Get-Command tilion-fortress
```

This updates current-session and persistent user PATH without admin rights or
replacing machine PATH. Restart GUI clients or use absolute executable paths.
No PowerShell activation script or execution-policy change is needed.

### Download and verify the engine

<!-- recipe: windows-engine -->
```powershell
$ErrorActionPreference = 'Stop'
$fortressAsset = 'fortress-v153-win-x64.zip'
$fortressBase = 'https://github.com/tiliondev/fortress/releases/download/v153.0.8010.36'
$fortressDownload = Join-Path $env:TEMP ('fortress-' + [guid]::NewGuid().ToString('N'))
New-Item -ItemType Directory -Path $fortressDownload | Out-Null
$fortressArchive = Join-Path $fortressDownload $fortressAsset
$fortressSums = Join-Path $fortressDownload 'SHA256SUMS'
Invoke-WebRequest -Uri "$fortressBase/$fortressAsset" -OutFile $fortressArchive
Invoke-WebRequest -Uri "$fortressBase/SHA256SUMS" -OutFile $fortressSums
$fortressPattern = '^([0-9a-fA-F]{64})\s+\*?' + [regex]::Escape($fortressAsset) + '$'
$fortressMatches = @(Get-Content -LiteralPath $fortressSums | ForEach-Object {
    if ($_ -match $fortressPattern) { $Matches[1] }
})
if ($fortressMatches.Count -ne 1) { throw 'Expected exactly one checksum entry' }
$fortressActual = (Get-FileHash -LiteralPath $fortressArchive -Algorithm SHA256).Hash
if ($fortressActual -ne $fortressMatches[0]) { throw 'Fortress checksum mismatch' }
$fortressInstall = Join-Path $fortressRoot 'installs\v153.0.8010.36'
New-Item -ItemType Directory -Force -Path $fortressInstall | Out-Null
tar.exe -xf $fortressArchive -C $fortressInstall
if ($LASTEXITCODE -ne 0) { throw 'Engine extraction failed' }
$fortressLaunchers = @(Get-ChildItem -LiteralPath $fortressInstall -Recurse -File -Filter 'tillion.cmd')
if ($fortressLaunchers.Count -ne 1) { throw 'Expected one native tillion.cmd launcher' }
$fortressLauncher = $fortressLaunchers[0].FullName
Write-Output "Engine launcher: $fortressLauncher"
```

The double `l` in `tillion.cmd` is the shipped filename. Keep it beside
`chrome.exe` and the other bundle files. Use `tilion-fortress.exe` for activation.
Verify existing installations before reuse; do not overwrite a running engine.
Retain the archive/checksum and record the resolved launcher.

### Activate and restrict access to the key

Return to [activation](#2-save-paths-and-complete-activation-with-the-user).
Run `& $fortressCli activate --headless`, show its URL/code to the user, and keep
the process alive during email/account approval. After success:

<!-- recipe: windows-license -->
```powershell
$fortressLicense = Join-Path $fortressRoot 'license.jwt'
if (!(Test-Path -LiteralPath $fortressLicense -PathType Leaf)) { throw 'License was not saved' }
if ((Get-Item -LiteralPath $fortressLicense).Length -eq 0) { throw 'License file is empty' }
$fortressOwner = [Security.Principal.WindowsIdentity]::GetCurrent().User.Value
icacls.exe $fortressLicense /inheritance:r /grant:r "*${fortressOwner}:(F)" '*S-1-5-18:(F)'
if ($LASTEXITCODE -ne 0) { throw 'Could not protect the license file' }
icacls.exe $fortressLicense
& $fortressCli license status --json
```

Inspect the ACL: only the intended user and SYSTEM should have access. Existing
explicit grants survive disabling inheritance; review and remove those specific
grants if present. Do not recursively change unrelated ACLs. Keep profile/log
directories private too. A service account needs its own effective home/license
path or secret-manager environment; the interactive user's file is not shared
automatically. For an environment key, check status/source instead of requiring
a file. Do not print the JWT or persist it using `setx`.

### Start and integrate

Complete the main guide's port/ownership checks first. Restore variables from
the setup record when resuming in a new shell.

<!-- recipe: windows-start -->
```powershell
$fortressProfile = Join-Path $fortressRoot 'profiles\v153-default'
$fortressLogs = Join-Path $fortressRoot 'logs'
New-Item -ItemType Directory -Force -Path $fortressProfile,$fortressLogs | Out-Null
$fortressStamp = Get-Date -Format 'yyyyMMdd-HHmmss'
$fortressOut = Join-Path $fortressLogs "$fortressStamp.stdout.log"
$fortressErr = Join-Path $fortressLogs "$fortressStamp.stderr.log"
$fortressArguments = '/d /s /c ""{0}" --headless=new --remote-debugging-address=127.0.0.1 --remote-debugging-port=9222 --user-data-dir="{1}""' -f $fortressLauncher,$fortressProfile
$fortressProcess = Start-Process -FilePath $env:ComSpec -ArgumentList $fortressArguments `
    -WindowStyle Hidden -PassThru -RedirectStandardOutput $fortressOut -RedirectStandardError $fortressErr
Write-Output "Launcher process: $($fortressProcess.Id); stderr: $fortressErr"
```

The PID belongs to the command wrapper; identify/record its browser child
before later cleanup. Never stop all `chrome.exe` processes. Poll
`Invoke-RestMethod -TimeoutSec 2 http://127.0.0.1:9222/json/version` for up to
60 seconds, then complete [engine verification](#3-start-the-actual-engine-and-verify-the-result)
and the Python, Node, or MCP integration check. All use the same loopback URL;
desktop clients should use absolute executable paths. Do not add a public
firewall exception for local CDP.

## macOS commands

Follow the main flow above in order. This release ships a
native Apple Silicon / arm64 engine. There is no macOS Intel artifact in
v153.0.8010.36; use the main guide's supported remote-host path or a separately
verified container.

### Install activation CLI and PATH

Confirm Apple Silicon hardware and use a native Terminal/Python environment.
If `uname -m` reports `x86_64` on Apple Silicon, check for Rosetta before choosing
an artifact. Use Python 3.10+ with `venv`/`pip`; install missing Python through
the user's existing package manager or an official installer. Inspect/reuse an
existing dedicated environment if it already has the pinned package.

<!-- recipe: mac-cli -->
```bash
set -e
umask 077
mkdir -p "$HOME/.tilion"
chmod 700 "$HOME/.tilion"
python3 -m venv "$HOME/.tilion/cli"
"$HOME/.tilion/cli/bin/python" -m pip install --disable-pip-version-check --no-compile 'tilion-fortress==153.0.8010.36.post1'
export PATH="$HOME/.tilion/cli/bin:$PATH"
tilion-fortress --version
tilion-fortress activate --help
```

Persist PATH for the actual shell. For default interactive Zsh:

<!-- recipe: mac-path -->
```bash
touch "$HOME/.zshrc"
if ! grep -Fqx 'export PATH="$HOME/.tilion/cli/bin:$PATH"' "$HOME/.zshrc"; then
  printf '\nexport PATH="$HOME/.tilion/cli/bin:$PATH"\n' >> "$HOME/.zshrc"
fi
```

For Bash use the startup file Terminal loads (`.bash_profile` for login shells,
`.bashrc` for interactive non-login shells). Preserve existing contents and
verify resolution in a new terminal. GUI clients may not inherit shell PATH;
configure their executable with an absolute path.

### Download and verify the Apple Silicon engine

<!-- recipe: mac-engine -->
```bash
set -e
fortress_asset='fortress-v153-mac-arm64.tar.gz'
fortress_base='https://github.com/tiliondev/fortress/releases/download/v153.0.8010.36'
fortress_download="$(mktemp -d)"
curl --fail --location "$fortress_base/$fortress_asset" -o "$fortress_download/$fortress_asset"
curl --fail --location "$fortress_base/SHA256SUMS" -o "$fortress_download/SHA256SUMS"
(
  cd "$fortress_download"
  tr -d '\r' < SHA256SUMS | awk -v name="$fortress_asset" '$2 == name {print}' > selected.sha256
  test "$(wc -l < selected.sha256)" -eq 1
  shasum -a 256 --check selected.sha256
)
fortress_install="$HOME/.tilion/installs/v153.0.8010.36"
mkdir -p "$fortress_install"
tar -xzf "$fortress_download/$fortress_asset" -C "$fortress_install"
fortress_launcher="$fortress_install/fortress-v153/tilion"
test -x "$fortress_launcher"
printf 'Engine launcher: %s\n' "$fortress_launcher"
```

Verify existing installations before reuse; do not overwrite a running engine.
Retain the archive/checksum and record the launcher. Keep the app bundle together.
The macOS `tilion` launcher is a symlink; `find -type f -name tilion` skips it.
Test the exact verified bundle path with `test -x`, which follows that link.
The release is ad-hoc signed. If macOS blocks a verified download, inspect
quarantine attributes with `xattr -lr "$fortress_install"` and use the normal
macOS approval flow. If quarantine is the identified blocker, the release
documents `xattr -dr com.apple.quarantine "$fortress_install"` for this verified
bundle only. Do not disable Gatekeeper globally or modify unrelated apps.

### Activate and protect the key

Return to [activation](#2-save-paths-and-complete-activation-with-the-user).
Run `tilion-fortress activate --headless` and hand its URL/code to the user.
After the CLI saves the key:

<!-- recipe: mac-license -->
```bash
test -s "$HOME/.tilion/license.jwt"
chmod 700 "$HOME/.tilion"
chmod 600 "$HOME/.tilion/license.jwt"
tilion-fortress license status --json
```

For a secret-manager environment key, check status/source instead of requiring
a file. This release uses the file or `TILION_LICENSE_KEY`; it does not
automatically save to Keychain. PATH contains executable directories, never keys.

### Start and integrate

Complete the main guide's port/ownership checks first. Restore the launcher
variable from the setup record when resuming in a new shell.

<!-- recipe: mac-start -->
```bash
umask 077
mkdir -p "$HOME/.tilion/profiles/v153-default" "$HOME/.tilion/logs"
fortress_log="$HOME/.tilion/logs/v153-$(date +%Y%m%d-%H%M%S).log"
nohup "$fortress_launcher" --headless=new \
  --remote-debugging-address=127.0.0.1 --remote-debugging-port=9222 \
  --user-data-dir="$HOME/.tilion/profiles/v153-default" \
  > "$fortress_log" 2>&1 < /dev/null &
fortress_pid=$!
printf 'Process: %s; log: %s\n' "$fortress_pid" "$fortress_log"
```

Record the PID, log, and profile. Poll
`curl --fail --max-time 2 http://127.0.0.1:9222/json/version` for up to 60 seconds,
then complete [engine verification](#3-start-the-actual-engine-and-verify-the-result)
and the Python, Node, or MCP integration check. Native Apple Silicon installation
does not require Docker. Omit `--headless=new` only when a visible browser is
requested. GUI clients should use absolute executable paths.

## Sandbox test contract

Start from a clean OS image with **only `/instructions/AGENTS.md` copied in**.
No other guides, repository mount, installer script, host home directory, key,
or cache may be present at creation. The agent may subsequently install the
public dependencies and release artifacts described in this file.

For a bare container, install the Linux prerequisites as container root, then
create an ordinary runtime account using this block. Run all remaining recipes
as that account with its actual home directory:

<!-- recipe: sandbox-user -->
```bash
set -e
getent passwd fortress >/dev/null || useradd --create-home --shell /bin/bash fortress
```

Every fresh sandbox must complete its own activation flow and obtain its key
directly from the activation service. Never provide a key, copy another
sandbox's key, inject `TILION_LICENSE_KEY`, or use `activate --token`. Each
sandbox must work independently. The user approves its link; the agent resumes
after the CLI confirms receipt and saves the key. Do not fabricate approval
or treat the user's permission as a substitute for server-issued credentials.

The current comparison target is **39.3 seconds of automated installation and verification**, with
the human approval step assumed complete when the agent resumes. Measure the
critical path, not the sum of parallel task durations. Report separately:

- Runtime/image preparation, outside the application-install timer.
- Time from the first install command until all artifacts/integrations are
  ready and the approval link has been produced.
- Approval-gated idle time, from that point until the CLI confirms the saved
  key (zero if already approved). This includes polling delay; it is not an
  exact measurement of the human's interaction time.
- Time after that gate through license checks, engine startup, integration
  smoke check, and a successful new-shell check.
- Total wall time and automated time (wall time minus approval-gated idle).

Never claim a 39.3-second end-to-end install when approval or downloads took
longer. Record fresh-flow success for every qualifying result. Earlier trials
that reused credentials are diagnostic baselines only.

Use this learning loop: record the exact OS/image and instruction hash, run the
recipes, record the failing command, correct this file, and repeat in another
fresh container. Change one demonstrated source of friction at a time. Separate
image/runtime preparation and human waiting from installation timings. Report
all failures and distinguish scripted recipe replay from a blind agent test.

Require the pinned binary, accepted activation flow, a live CDP endpoint, a
passing integration check, and a successful subsequent-session check. Record
the actual browser version and secret-file permissions without reading the key
into output. Linux containers validate Linux distributions only; native Windows
and macOS require their own hosts/VMs and must not be reported as tested here.

## Hosted sandbox replay

For reproducible cross-platform measurements, the following optional runner
replays the recipes in this file on a fresh stock hosted OS. It is a controlled
recipe test, not a blind autonomous-agent evaluation. Python and system tools
supplied by that OS image are recorded as part of the environment. No Fortress
installation, cached Fortress download, or license may be supplied.

Extract this Python fence only after supplying AGENTS.md. Run `prepare`, expose
only `activation-public.json` to the approving human, then run `finish`. The
background worker performs the real activation. Never upload activation logs,
the user's home, or `.tilion`; only the public approval link and `report.json`
are intended for collection. Report CI orchestration time separately from both
the human gate and automated work. Each job creates a separate activation flow.

<!-- recipe: sandbox-replay -->
```python
"""Controlled recipe replay. The only supplied setup artifact is AGENTS.md."""
import concurrent.futures
import hashlib
import json
import os
from pathlib import Path
import platform
import re
import shutil
import subprocess
import sys
import time
import urllib.request

os.umask(0o077)
work = Path.cwd()
doc = (work / 'AGENTS.md').read_text(encoding='utf-8')
recipes = dict((name, (lang, body)) for name, lang, body in re.findall(
    r'<!-- recipe: ([\w-]+) -->\n```(bash|powershell)\n(.*?)^```', doc, re.M | re.S))
system = platform.system()
prefix = {'Windows': 'windows', 'Darwin': 'mac', 'Linux': 'linux'}[system]
root = Path.home() / '.tilion'
venv = root / 'cli'
bin_dir = venv / ('Scripts' if system == 'Windows' else 'bin')
python = bin_dir / ('python.exe' if system == 'Windows' else 'python')
cli = bin_dir / ('tilion-fortress.exe' if system == 'Windows' else 'tilion-fortress')
state_path = work / 'sandbox-state.json'
stage_records = []

def save(path, value):
    temporary = path.with_suffix('.tmp')
    temporary.write_text(json.dumps(value, indent=2), encoding='utf-8')
    temporary.replace(path)

def command(stage, body, lang='bash', env=None):
    start = time.monotonic()
    script = work / ('generated-' + stage + ('.ps1' if lang == 'powershell' else '.sh'))
    script.write_text(body, encoding='utf-8')
    argv = (['pwsh', '-NoProfile', '-File', str(script)] if lang == 'powershell'
            else ['bash', str(script)])
    # Logs stay on this VM. Never upload the home directory or activation output.
    with (work / (stage + '.log')).open('wb') as log:
        result = subprocess.run(argv, stdout=log, stderr=subprocess.STDOUT, env=env)
    row = dict(stage=stage, seconds=round(time.monotonic()-start, 3), exit_code=result.returncode)
    stage_records.append(row)
    print(json.dumps(row), flush=True)
    if result.returncode:
        # Installation recipes contain no credential values; keep other logs private.
        if stage in ('cli', 'engine', 'playwright', 'prerequisites'):
            print((work / (stage + '.log')).read_text(encoding='utf-8', errors='replace')[-5000:], flush=True)
        raise RuntimeError(stage + ' failed')

def recipe(stage, name, prologue='', env=None):
    lang, body = recipes[name]
    command(stage, prologue + '\n' + body, lang, env)

def cli_setup():
    recipe('cli', prefix + '-cli')
    if system != 'Windows':
        recipe('path', prefix + '-path')
    env = os.environ.copy()
    env['PYTHONUNBUFFERED'] = '1'
    with (work / 'activation-worker.log').open('wb') as log:
        subprocess.Popen([sys.executable, str(Path(__file__).resolve()), '_activate'],
                         stdout=log, stderr=log, env=env,
                         creationflags=subprocess.CREATE_NO_WINDOW if system == 'Windows' else 0)
    command('playwright', ('& ' if system == 'Windows' else '') + '"' + str(python) + '" -m pip install --disable-pip-version-check --no-compile playwright==1.63.0' +
            ("\nif ($LASTEXITCODE -ne 0) { throw 'Playwright install failed' }" if system == 'Windows' else ''),
            'powershell' if system == 'Windows' else 'bash')

def engine_setup():
    prologue = "$fortressRoot = Join-Path $env:USERPROFILE '.tilion'" if system == 'Windows' else 'set -e\numask 077'
    recipe('engine', prefix + '-engine', prologue)

def prepare():
    assert not (root / 'license.jwt').exists(), 'Sandbox already has a key'
    assert not root.exists(), 'Sandbox already has Fortress state'
    assert 'TILION_LICENSE_KEY' not in os.environ, 'Injected key is forbidden'
    assert platform.machine().lower() in ({'arm64', 'aarch64'} if system == 'Darwin' else {'amd64', 'x86_64', 'arm64', 'aarch64'})
    started = time.monotonic()
    with concurrent.futures.ThreadPoolExecutor(max_workers=2) as pool:
        tasks = [pool.submit(cli_setup), pool.submit(engine_setup)]
        for task in tasks:
            task.result()
    if system == 'Linux':
        binary = root / 'installs/v153.0.8010.36/fortress-v153/chrome'
        dependencies = subprocess.check_output(['ldd', str(binary)], text=True)
        if 'not found' in dependencies:
            lang, body = recipes['linux-runtime-libraries']
            command('prerequisites', 'set -e\nsudo apt-get update -qq\n' + body.replace(
                'DEBIAN_FRONTEND=noninteractive apt-get', 'sudo DEBIAN_FRONTEND=noninteractive apt-get'))
        recipe('preflight', 'linux-preflight')
    deadline = time.monotonic() + 45
    while not (work / 'activation-public.json').exists():
        if (work / 'activation-result.json').exists() or time.monotonic() > deadline:
            raise RuntimeError('Activation failed before issuing an approval URL')
        time.sleep(.1)
    ready = time.monotonic()
    state = dict(started=started, ready=ready, stages=stage_records,
                 system=system, architecture=platform.machine(),
                 os_version=platform.platform(), image_os=os.getenv('ImageOS'), image_version=os.getenv('ImageVersion'),
                 instruction_sha256=hashlib.sha256((work / 'AGENTS.md').read_bytes()).hexdigest(),
                 input_files=['AGENTS.md'], integration='Python Playwright 1.63.0',
                 trial='stock hosted OS; independent fresh activation; controlled recipe replay')
    save(state_path, state)
    print(json.dumps({'installation_ready_seconds':round(ready-started,3), 'system':system}), flush=True)

def activate():
    env = os.environ.copy()
    env['PYTHONUNBUFFERED'] = '1'
    process = subprocess.Popen([str(python), '-u', '-m', 'tillion_fortress', 'activate', '--headless'],
                               stdout=subprocess.PIPE, stderr=subprocess.STDOUT, text=True, encoding='utf-8', errors='replace', env=env)
    for line in process.stdout:
        match = re.search(r'https://[^\s]+/activate\?code=[A-Z0-9-]+', line)
        if match:
            save(work / 'activation-public.json', dict(system=system, architecture=platform.machine(),
                approval_url=match.group(0), user_code=match.group(0).split('code=')[1]))
    save(work / 'activation-result.json', dict(exit_code=process.wait(), finished=time.monotonic()))

def finish():
    state = json.loads(state_path.read_text())
    deadline = time.monotonic() + 930
    while not (work / 'activation-result.json').exists():
        if time.monotonic() > deadline:
            raise RuntimeError('Activation worker timed out')
        time.sleep(.1)
    activated = json.loads((work / 'activation-result.json').read_text())
    assert activated['exit_code'] == 0, 'Real activation did not finish successfully'
    verify_start = time.monotonic()
    env = os.environ.copy()
    env['PATH'] = str(bin_dir) + os.pathsep + env['PATH']
    license_path = root / 'license.jwt'
    assert license_path.stat().st_size > 0
    if system == 'Windows':
        prologue = "\n".join(["$fortressRoot = Join-Path $env:USERPROFILE '.tilion'", "$fortressCli = Join-Path $fortressRoot 'cli\\Scripts\\tilion-fortress.exe'"])
    else:
        prologue = 'export PATH="$HOME/.tilion/cli/bin:$PATH"'
    recipe('license-permissions', prefix + '-license', prologue, env)
    status = json.loads(subprocess.check_output([str(cli), 'license', 'status', '--json'], env=env))
    assert status['licensed'] and status['mode'] == 'v3'
    launchers = list((root / 'installs/v153.0.8010.36').rglob('tillion.cmd' if system == 'Windows' else 'tilion'))
    launchers = [p for p in launchers if p.is_file()]
    assert len(launchers) == 1
    launcher = launchers[0]
    if system == 'Windows':
        quoted = str(launcher).replace("'", "''")
        recipe('engine-start', 'windows-start', prologue + "\n$fortressLauncher = '" + quoted + "'", env)
    else:
        import shlex
        recipe('engine-start', prefix + '-start', 'set -e\nfortress_launcher=' + shlex.quote(str(launcher)), env)
    deadline = time.monotonic() + 60
    while True:
        try:
            with urllib.request.urlopen('http://127.0.0.1:9222/json/version', timeout=2) as response:
                version = json.load(response)
            assert version['Browser'] == 'Chrome/153.0.8010.36' and version['webSocketDebuggerUrl']
            break
        except (OSError, AssertionError):
            if time.monotonic() >= deadline:
                raise
            time.sleep(.1)
    smoke = re.findall(r'^```python\n(.*?)^```', doc, re.M | re.S)[0]
    subprocess.run([str(python), '-c', smoke], check=True, env=env)
    if system == 'Windows':
        check = "$env:Path = [Environment]::GetEnvironmentVariable('Path','User') + ';' + [Environment]::GetEnvironmentVariable('Path','Machine'); (Get-Command tilion-fortress).Source; tilion-fortress license status --json"
        subprocess.run(['pwsh', '-NoProfile', '-Command', check], check=True)
    else:
        subprocess.run(['zsh' if system == 'Darwin' else 'bash', '-ic', 'command -v tilion-fortress; tilion-fortress license status --json'], check=True, env=env)
    with urllib.request.urlopen('http://127.0.0.1:9222/json/version', timeout=2) as response:
        assert json.load(response)['Browser'] == version['Browser']
    for logfile in (root / 'logs').glob('*.log'):
        log = logfile.read_text(encoding='utf-8', errors='replace')
        assert not re.search(r'invalid.*licen[cs]e|licen[cs]e.*(?:invalid|reject)|falling back|fallback.*v1', log, re.I), 'Engine reported rejected activation'
    if system != 'Windows':
        assert license_path.stat().st_mode & 0o777 == 0o600
    finished = time.monotonic()
    idle = max(0, activated['finished']-state['ready'])
    # CI artifact handoff can delay the next step after approval; report it
    # separately rather than attributing scheduler delay to installation.
    orchestration = max(0, verify_start-max(state['ready'], activated['finished']))
    report = dict(state, browser=version['Browser'], fresh_activation=True,
        total_seconds=round(finished-state['started'],3), approval_gated_idle_seconds=round(idle,3),
        orchestration_seconds=round(orchestration,3),
        automated_seconds=round((state['ready']-state['started'])+(finished-verify_start),3),
        installation_ready_seconds=round(state['ready']-state['started'],3),
        after_approval_seconds=round(finished-verify_start,3), result='passed', verification_stages=stage_records)
    save(work / 'report.json', report)
    print(json.dumps(report), flush=True)

{'prepare': prepare, '_activate': activate, 'finish': finish}[sys.argv[1]]()

```
