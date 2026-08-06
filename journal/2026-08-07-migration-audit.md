# 2026-08-07 — Post-migration audit (Sebas1705 → Sebas1705Carreer)

All four career repos were migrated (transferred) from the personal account to the org in June–August 2026. This audit verified everything published and fixed what the move had broken.

## Verified healthy

- **API live** at `career-api.sebas1705.workers.dev` (v3.0.0) — root metadata, `/projects`, `/personal` and Swagger `/docs` respond. (Note: the worker does not answer `HEAD`, only `GET` — monitoring should use `GET`.)
- **Both Pages sites live** on the new org domain: [carreerV2](https://sebas1705carreer.github.io/carreerV2/) (active) and [carreerV1](https://sebas1705carreer.github.io/carreerV1/) (legacy). Repo `homepage` fields already pointed at the new URLs.
- **CI green** on the last completed runs of the three repos that have workflows.
- Old-account career repos are gone (transfers redirect); unrelated personal repos remain on `Sebas1705` by design.

## Broken by the migration — fixed

- `sebas1705.github.io/carreerV1|carreerV2` (old Pages URLs) return **404**: they were still the "Live site" links in both READMEs, the **canonical/og:url** in carreerV2's `index.html` (which even pointed at the pre-rename path `/carreer/`), the `site` in carreerV1's `astro.config.mjs`, and the **Portfolio badge** on the `Sebas1705/Sebas1705` profile. All now point at `sebas1705carreer.github.io`.
- carreerV2's `og:image`/`twitter:image` referenced a nonexistent `og-image.png` — metas removed (`twitter:card` → `summary`) until an image exists.
- Clone commands and cross-repo "Related repositories" tables referenced `github.com/Sebas1705/...`, including a dead `Sebas1705/career-api` link and a stale "Hono + D1" description — now `Sebas1705Carreer/...` with the real stack (Workers + KV).
- **Dead project links in data:** `Sebas1705/my-portfolio` no longer exists → replaced with `Sebas1705Carreer/carreerV1`; the Omni-Impostor repo is private → github link nulled and name updated. Fixed in carreerV2 fallback JSON, carreerV1 data, and both worker seed files (`seed-kv.mjs` had missed the earlier rename that `seed-data.ts` got).
- Worker README documented the **v2 API model** (bilingual `_en`/`_es` fields, no `/languages`, no Swagger) — rewritten to match the deployed v3.

## Live KV data

The two dead links were also live in KV (`projects/impostor` → private repo, `projects/portfolio` → deleted `my-portfolio`). Patched via the API on 2026-08-07 and verified: `impostor` renamed to Omni-Impostor with no github link, `portfolio` pointing at `Sebas1705Carreer/carreerV1`.
