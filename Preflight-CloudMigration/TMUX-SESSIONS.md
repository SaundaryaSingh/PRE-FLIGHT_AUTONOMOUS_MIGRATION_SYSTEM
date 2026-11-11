# tmux and live server state at handoff

Captured from the source machine on **2026-11-11** while the v2 demo stack was
running. A tmux server is a live process, so it cannot be put inside a ZIP;
this file records exactly what was running and how to recreate it. All files
those processes were serving are included in this package.

## tmux sessions

| Session | Created | Current directory | Command |
|---|---|---|---|
| `preflight` | 15:48 | `v2/` | `.venv/bin/python -m uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload --log-level info` |
| `pf-compact` | 20:53 | `v2/docs/presentation/` | `python3 -m http.server 8092 --bind 127.0.0.1` |
| `pf-deck` | 17:24 | `v2/` | `.venv/bin/python -m http.server 8091 -d docs/presentation` |
| `pf-presentation` | 20:02 | `v2/` | `python3 -m http.server 8090 -d docs/presentation` |

The three presentation sessions served the same `docs/presentation/` folder on
different ports. `pf-presentation` (8090) served the main 23-slide `index.html`
shown in the handoff screenshots; `pf-compact` (8092) served the 12-slide
`compact-10min.html`. `pf-deck` (8091) was a duplicate. Only `preflight`,
`pf-presentation`, and `pf-compact` are needed for the handoff demo.

## Listening ports

| Port | Process | Purpose |
|---:|---|---|
| 8000 | `uvicorn app.main:app` | PreFlight app (tmux session `preflight`) |
| 8010 | `uvicorn app.main:app` | Standalone app process, not in tmux |
| 8090 | `python3 -m http.server` | Main 23-slide deck (`index.html`) |
| 8091 | `python3 -m http.server` | Duplicate presentation folder |
| 8092 | `python3 -m http.server` | Compact 10-minute deck (tmux session `pf-compact`) |

## Recreate the two-session demo stack

Run from the project root after section 2 of `START-HERE.md` has created
`.venv`:

```bash
# App on 8000 in tmux session "preflight"
make demo

# Compact deck on 8092 in tmux session "pf-compact"
tmux new-session -d -s pf-compact \
  "cd '$PWD/docs/presentation' && python3 -m http.server 8092 --bind 127.0.0.1"

# Attach to either session
tmux attach -t preflight
# Ctrl+b, then d to detach
tmux attach -t pf-compact
# Ctrl+b, then d to detach

# Stop the app session
make stop

# Stop the deck session
tmux kill-session -t pf-compact
```

## URLs that were live

- App: <http://127.0.0.1:8000/?run=R-DEMO-001&feature=PA-VEH-01>
- Main 23-slide deck: <http://127.0.0.1:8090/#slide-1>
- Compact 10-minute deck: <http://127.0.0.1:8092/compact-10min.html#slide-1>
- Companion visualization: <http://127.0.0.1:8092/demo-visual.html>
- Case studies: <http://127.0.0.1:8000/case-studies>
- Codebase lab: <http://127.0.0.1:8000/codebase?case=cascade-mutual>
- Migrator: <http://127.0.0.1:8000/migrator?run=R-DEMO-001>
- Packet: <http://127.0.0.1:8000/packet?run=R-DEMO-001>

The bundled `data/runs/` directory is the run state those app processes were
serving. Starting Uvicorn directly (section 2, Option A) preserves it. Running
`make demo` or `make demo-fg` regenerates the fixtures and reseeds
`R-DEMO-001`, which intentionally replaces that state.
