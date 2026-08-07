# 2026-08-07 — Quality pass across the ecosystem, org profile

Second wave of work after the migration audit, in one day:

## Data

- `/projects` grew from 10 to **20**: Solusoft work (ApiMovil, Wemob API, Transportes Chinchón Android + iOS) and GitHub projects (Templetry, Career Ecosystem/Folio, KMP Native Base, TutorsAtHome, Computer Vision Labs, Enigma HPC). The `senior-solusoft` job lists the new work projects. Seeds mirrored for projects/jobs; ⚠️ they still lag live for skills/education/certs (warning added to the API README).

## Folio (career-editor-kmp)

- **Root cause of "loads but never saves"**: a certification with `url: null` broke `GET /certifications` deserialization (non-nullable model field) and aborted the whole `loadAll()` — empty state, disabled save buttons. Fixed (nullable + `coerceInputValues`).
- Silent auth failures fixed: `expectSuccess = true`, login validates the token against a real no-op write, tokens sanitized of BOM/zero-width chars.
- New visual identity ("mono as the voice of data"): file headers `~/portfolio/<slug>`, refined ink/teal/amber palette, FolioMark, compact headers, overlap-proofing pass (FAB padding, imePadding, ellipsis everywhere, min desktop window).

## carreerV2

- Audit found and fixed: hero "Download CV" 404 (now opens the CV generator), stale CV project id (`vpsorchestrator`), in-progress certs rendered as dead links + duplicate React keys, ghost desc line.
- Offline fallback actually wired (public/data snapshot; previously dead files), CV generator lazy-loaded (main chunk 1.9MB → 446KB), woff2 Font.register removed, session cache v7, lint green and gating CI.

## Operations & org

- Secrets fully in Doppler (`carreer/prd`), API_SECRET and CF_API_TOKEN rotated and verified end-to-end (deploy green with new credentials).
- Org profile: `.github` repo with overview/diagram/live links + family mark; org name/description/website set; all repos got proper descriptions, homepages and topics.
- GitHub Actions outage (08-06) aftermath: zombie queued runs identified; Templetry kmp verify re-run green.
