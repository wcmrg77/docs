# TalkPilot Docs (docs.talkpilot.io)

Quelle dieser Doku ist `docs/` im Repo `wcmrg77/talkpilot-dashboard`. Das Repo `wcmrg77/docs`
ist nur der Spiegel, aus dem Mintlify baut: die GitHub Action `docs-sync.yml` überschreibt ihn
bei jedem Merge auf `main` komplett (rsync mit `--delete`).

**Nicht direkt in `wcmrg77/docs` editieren** — Änderungen dort gehen beim nächsten Sync verloren.
Änderungen gehören in `talkpilot-dashboard/docs/` (PR gegen `main`).

Mintlify ignoriert diese Datei beim Build.
