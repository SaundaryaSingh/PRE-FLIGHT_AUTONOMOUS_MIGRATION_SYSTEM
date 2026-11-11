# PreFlight — USB handoff

This folder is a clean, self-contained copy of the **PreFlight** project
(`v1.1.0-demo` snapshot, 2026-11-11). It contains source code, synthetic
fixtures, tests, and documentation. It does **not** contain a virtual
environment, Git history, generated run data, or any `.env` secrets.

> PreFlight is an offline, evidence-first cloud-readiness review workstation
> for synthetic Guidewire-style policy administration estates. Everything is
> local; the demo does not contact a tenant or the internet.

The full project documentation is in [`README.md`](README.md). This file is
the practical setup and run guide.

---

## 1. Requirements

| Requirement | Notes |
|---|---|
| Linux or macOS | On Windows, use **WSL2** (Ubuntu recommended). The launcher uses Bash and `make`. |
| Python 3.12 | `uv` can download and manage Python 3.12 for you. |
| `uv` | Python package/environment manager. Install with the command below. |
| `make` | Used by the convenience commands. |
| `tmux` | Optional. Only needed for the background `make demo` mode; `make demo-fg` works without it. |
| Internet (first run only) | To download Python packages (and Chromium if you install the test extras). |

### Install the tools

**Ubuntu / Debian / WSL2:**

```bash
sudo apt update
sudo apt install -y make tmux curl
curl -LsSf https://astral.sh/uv/install.sh | sh
```

If `uv` is not found after installation, open a new terminal or run:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

**macOS (Homebrew):**

```bash
brew install make tmux uv
```

Verify:

```bash
uv --version
make --version
python3 --version   # 3.12 is preferred; uv can install it if needed
```

If Python 3.12 is missing:

```bash
uv python install 3.12
```

---

## 2. Quick start

This package includes the seeded run store from the handoff machine
(`data/runs/`), so you can open the exact state that was running, or reseed a
clean demo. Choose one option.

### Option A — open the bundled live state (recommended)

This serves the included run data without regenerating or reseeding it:

```bash
cd Preflight-CloudMigration

# Runtime only; skips the Chromium/test download
uv venv .venv --python 3.12
uv pip install --python .venv/bin/python -e .

.venv/bin/python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Open **<http://127.0.0.1:8000/?run=R-DEMO-001>**.

Press `Ctrl+C` to stop the server.

### Option B — fresh demo run

This regenerates the synthetic fixtures, reseeds `R-DEMO-001`, and starts the
server. It overwrites the demo run state included in the package.

```bash
cd Preflight-CloudMigration
make install       # create .venv, install runtime + test deps, download Chromium
make demo-fg       # regenerate fixtures, reseed R-DEMO-001, run in foreground
```

Open **<http://127.0.0.1:8000/?run=R-DEMO-001>**.

`make demo-fg` runs in the foreground; press `Ctrl+C` to stop it. For a fresh
run with only runtime dependencies, use the manual commands in section 5
instead of `make install`.

### Background app mode with tmux

```bash
make demo
```

The app keeps running in a tmux session named `preflight` and restarts if
Uvicorn stops. Detach with `Ctrl+b`, then `d`. Stop it deliberately with:

```bash
make stop
```

---

## 3. What to open

| Page | URL |
|---|---|
| Workbench / demo run | <http://127.0.0.1:8000/?run=R-DEMO-001> |
| Case studies | <http://127.0.0.1:8000/case-studies> |
| Codebase lab — Cascade Mutual | <http://127.0.0.1:8000/codebase?case=cascade-mutual> |
| Codebase lab — Harbor PC | <http://127.0.0.1:8000/codebase?case=harbor-pc> |
| Autonomous migrator | <http://127.0.0.1:8000/migrator?run=R-DEMO-001> |

Suggested first pass: open the Workbench, click a feature in the spine, then
open **Manifest**, **Reconstruction**, **Findings**, **Evaluation**, and
**Packet** from the navigation.

---

## 4. Presentation deck, slides, and tmux

The latest slides and presentation frontend are included under
`docs/presentation/`:

| File | Served at | Purpose |
|---|---|---|
| `index.html` | <http://127.0.0.1:8090/> | Main 23-slide deck, title “Evidence before the estimate” |
| `PreFlight-deck.pdf` | — | PDF render of the 23-slide deck |
| `compact-10min.html` | <http://127.0.0.1:8092/compact-10min.html> | 12-slide, self-contained 10-minute cut |
| `PreFlight-10min.pdf` | — | PDF render of the compact deck |
| `demo-visual.html` | <http://127.0.0.1:8092/demo-visual.html> | Full-run visualization linked from slide 9 |
| `assets/` | — | Deck screenshots, slides captures, and evidence images |
| `DEMO-10MIN.md` | — | Minute-by-minute presenter script |

Serve the presentation folder (either port; both point at
`docs/presentation/`):

```bash
cd Preflight-CloudMigration

# Main 23-slide deck (the one served on 8090 at handoff)
python3 -m http.server 8090 --bind 127.0.0.1 -d docs/presentation
# open http://127.0.0.1:8090/

# Compact 10-minute deck (the one served on 8092 at handoff)
python3 -m http.server 8092 --bind 127.0.0.1 -d docs/presentation
# open http://127.0.0.1:8092/compact-10min.html
```

To run both in tmux exactly as they were during handoff:

```bash
tmux new-session -d -s pf-presentation \
  "cd '$PWD/docs/presentation' && python3 -m http.server 8090 --bind 127.0.0.1"
tmux new-session -d -s pf-compact \
  "cd '$PWD/docs/presentation' && python3 -m http.server 8092 --bind 127.0.0.1"
tmux attach -t pf-compact
```

At handoff, four tmux sessions were running on the source machine:

| tmux session | Port | What it served |
|---|---:|---|
| `preflight` | 8000 | PreFlight app (`uvicorn app.main:app`) |
| `pf-presentation` | 8090 | `docs/presentation/` — main 23-slide `index.html` |
| `pf-compact` | 8092 | `docs/presentation/` — compact 10-minute deck |
| `pf-deck` | 8091 | `docs/presentation/` — duplicate static server |

Only `preflight`, `pf-presentation`, and `pf-compact` are needed. `pf-deck`
was a duplicate of the same folder. A tmux server is a live process and cannot
be archived, but every file those sessions served is included, and the
commands above recreate the layout. The full captured session list, ports, and
live URLs are in [`TMUX-SESSIONS.md`](TMUX-SESSIONS.md).

---

## 5. Manual run (no `make`)

If `make` is unavailable, run the same steps directly:

```bash
# 1. Environment
uv venv .venv --python 3.12
uv pip install --python .venv/bin/python -e .

# 2. Synthetic fixtures + seeded demo run
.venv/bin/python scripts/generate_fixture.py
.venv/bin/python -m scripts.seed_demo --force

# 3. Server
.venv/bin/python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Then open <http://127.0.0.1:8000/?run=R-DEMO-001>.

On native Windows PowerShell (not WSL), the equivalent startup is:

```powershell
py -3.12 -m venv .venv
.venv\Scripts\python -m pip install -e .
.venv\Scripts\python scripts\generate_fixture.py
.venv\Scripts\python -m scripts.seed_demo --force
.venv\Scripts\python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

WSL2 is still recommended because `make` and the demo scripts assume a
POSIX environment.

---

## 6. Useful commands

Run these from the project folder after `make install`:

```bash
make test       # full pytest suite (481 tests in this snapshot)
make e2e        # Playwright browser golden path (needs Chromium)
make claims     # claims-ledger and rule-coverage gate
make verify     # verify the hash-chained audit log for R-DEMO-001
make lint       # compile check + claims lint
make freeze     # regenerate and freeze fixture hashes
make clean      # remove generated runs, caches, and __pycache__
```

Useful CLI checks after seeding:

```bash
.venv/bin/python -m app.cli verify-audit R-DEMO-001
.venv/bin/python -m app.cli verify-migration R-DEMO-001
.venv/bin/python -m app.cli migrate R-DEMO-001
```

The demo data is written under `data/runs/` and can be regenerated at any
time. `make demo` and `make demo-fg` regenerate the fixtures and reseed
`R-DEMO-001` automatically.

---

## 7. Optional configuration

The demo runs fully offline and needs no configuration. An optional live
reasoning endpoint can be configured by copying `.env.example` to `.env` and
filling it in. Leave `PREFLIGHT_LLM_MODE=off` for the normal demo.

Do **not** ship or commit a populated `.env`; this package intentionally
contains only `.env.example`.

---

## 8. Troubleshooting

| Symptom | Fix |
|---|---|
| `uv: command not found` | Install `uv` (see above), then run `export PATH="$HOME/.local/bin:$PATH"` or open a new shell. |
| `Python 3.12 not found` | Run `uv python install 3.12`, then retry `make install`. |
| `make: command not found` | Install `make` (`sudo apt install make` on Debian/Ubuntu, `brew install make` on macOS), or use the manual commands in section 5. |
| `tmux: command not found` | Install `tmux`, or use `make demo-fg` instead of `make demo`. |
| Port 8000 already in use | Stop the other process, or run Uvicorn on another port and open that port: `.venv/bin/python -m uvicorn app.main:app --host 127.0.0.1 --port 8001` |
| Browser tests fail with a missing browser | Run `.venv/bin/python -m playwright install chromium`, or skip `make e2e`. |
| Demo page is empty or run not found | Re-run `make demo-fg` (or `make seed`) so `R-DEMO-001` is seeded, then use the full URL with `?run=R-DEMO-001`. |
| Need a clean slate | `make clean`, then `make demo-fg`. |

---

## 9. Package contents and boundaries

Included: `app/`, `rules/`, `fixtures/`, `ground_truth/`, `templates/`,
`tests/`, `scripts/`, `plans/`, `docs/` (including the latest
`docs/presentation/` slides and assets), `Docs/`, the seeded `data/runs/`
state, project metadata, and the project README.

Intentionally excluded: `.venv/`, `.git/`, `.env`, stale run locks, caches,
nested ZIPs, and audit screenshot dumps. The environment is rebuilt with `uv`
on the target machine; the bundled run data is portable JSON/HTML.

PreFlight is a research prototype, not an official Guidewire product and not
an official Cloud Assurance result. All fixtures are synthetic; the demo does
not contact a tenant, deploy changes, or make external network calls. See
[`docs/REALITY-BOUNDARY.md`](docs/REALITY-BOUNDARY.md) and
[`README.md`](README.md) for the full claims and limitations.
