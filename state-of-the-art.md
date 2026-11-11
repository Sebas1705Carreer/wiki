# State of the art

**Snapshot: 2026-11-11** · maintained as the single up-to-date picture of the career ecosystem: what exists, where it runs and how the pieces fit.

## What this is

A self-hosted "portfolio as a product": one **API** is the single source of truth for all career data (jobs, projects, skills, education…), consumed by a **public portfolio site** and managed through a **native editor app**. Everything lives in the GitHub organization **[Sebas1705Carreer](https://github.com/Sebas1705Carreer)**.

```
Folio · career-editor-kmp (Android)           carreerV2 (React, live site)
        │  writes [Bearer auth]                 │  reads (runtime, offline fallback)
        ▼                                       ▼
              career-api-worker  (Cloudflare Workers + KV)
              https://api.sebas1705.dev
                                                ▲
                                                │  reads (build time)
                                         carreerV1 (Astro, legacy)
```

## Repos & live surfaces

| Repo | Role | Default branch | Live |
|---|---|---|---|
| [career-api-worker](https://github.com/Sebas1705Carreer/career-api-worker) | REST API — Cloudflare Workers + KV, TypeScript. Single source of truth for all career data | `development` | [api.sebas1705.dev](https://api.sebas1705.dev) · [Swagger UI](https://api.sebas1705.dev/docs) |
| [carreerV2](https://github.com/Sebas1705Carreer/carreerV2) | **Active portfolio** — React 19, Vite 8, Tailwind v4, i18next, in-browser PDF CV generator (lazy chunk; the hero "Download CV" opens it). Reads the API at runtime with a wired offline fallback (`public/data/`). Family identity: ink/teal `@theme` tokens, Space Grotesk + JetBrains Mono, Folio-style file headers, hash deep links (`#projects`), live hero stats, reduced-motion honoured. CV generator: 4 curated role profiles × 2 variants — visual (HR) and ATS/AI (single column, standard headings, YYYY-MM dates, plain URLs) | `main` | [sebas1705carreer.github.io/carreerV2](https://sebas1705carreer.github.io/carreerV2/) |
| [carreerV1](https://github.com/Sebas1705Carreer/carreerV1) | **Legacy portfolio** — Astro 5 SSG, Clean Architecture, 10 languages, Playwright E2E. Reads the API at build time. Kept for reference, still deployed | `main` | [sebas1705carreer.github.io/carreerV1](https://sebas1705carreer.github.io/carreerV1/) |
| [career-editor-kmp](https://github.com/Sebas1705Carreer/career-editor-kmp) | **Folio** — Kotlin Multiplatform + Compose editor, **Android only** (the Desktop target was removed), Ktor client. Writes to the API with the Bearer token; login validates the token with `GET /auth/check` before saving it, API errors surface (expectSuccess), tokens sanitized of BOM. The release build embeds the token (Doppler project `folio`), so the APK is distributed through Firebase App Distribution, never as a GitHub release asset. Own visual identity (mono-as-data, `~/portfolio/<slug>` file headers) | `development` | Firebase App Distribution (tag `v*` → `Release APK`); latest tag `v1.3.2` |
| [wiki](https://github.com/Sebas1705Carreer/wiki) | This documentation hub | `main` | — |
| [.github](https://github.com/Sebas1705Carreer/.github) | Organization profile (overview, live links, family mark) | `main` | [org page](https://github.com/Sebas1705Carreer) |

The old personal-account repos redirect here; the previous portfolio repo `Sebas1705/my-portfolio` no longer exists (its successor is `carreerV1`). The personal profile page [Sebas1705](https://github.com/Sebas1705) links into this org.

## The API (v3)

- **Entities:** `languages` and `personal` (singular · GET/PUT/PATCH), plus `jobs`, `projects`, `skills`, `education`, `certifications`, `soft-skills` (arrays with `id` · full CRUD).
- **i18n model:** every localizable field is a `LocalizedString` — `{ [langCode]: value }`. `GET /languages` drives consumers' language switchers; adding a language = extending the maps + updating `/languages`. Currently `en` (default) and `es`.
- **Auth:** reads are public; writes require `Authorization: Bearer <API_SECRET>` (a Cloudflare secret); `GET /auth/check` answers 204 or 401 so a client can validate a token without writing.
- **Reads:** every public `GET` carries an `ETag` and a short `Cache-Control` (304 on `If-None-Match`); `GET /all` returns all eight entities in one request. Writes are validated (`400` with `details`).
- **Docs:** OpenAPI 3.0 at [`/openapi.json`](https://api.sebas1705.dev/openapi.json), Swagger UI at [`/docs`](https://api.sebas1705.dev/docs).
- **Storage:** Cloudflare KV (`CAREER_KV`). Seeding via `node seed-kv.mjs` (source of truth for re-seeds; `src/seed-data.ts` mirrors it for the worker bundle).
- **CORS:** `*` by default, configurable via `ALLOWED_ORIGIN`.
- **Content (live API, read 2026-11-11):** 27 projects (8 work, 8 academic, 11 personal), 3 jobs, 71 skills (in the `languages`/`frameworks`/`tools`/`databases`/`cloud`/`ai`/`architecture`/`methodologies` taxonomy), 3 education entries, 10 certifications and 12 soft skills. The 2026-08-07 snapshot of this page said 20 projects and 137 skills; the skills list has been curated since (see `data-updates/` in career-api-worker). These counts will drift again: `GET /all` is the source, not this page.
- ⚠️ **Seeds lag live data**: `seed-kv.mjs`/`seed-data.ts` are far behind the live data (about 30 skills against 71 live, and no `long_desc`) — re-seeding would overwrite it (warning in the API README). Live data changes go through Folio or the scripts in `data-updates/`.

## CI/CD

| Repo | Workflow | Trigger |
|---|---|---|
| career-api-worker | `CI`: type-check + tests | pull requests |
| career-api-worker | Deploy to Cloudflare Workers (Wrangler 4, Node 22) | push to `development` |
| carreerV2 | Lint → Vitest → build → Playwright E2E + axe → GitHub Pages deploy (`VITE_BASE_PATH=/carreerV2/`) | push to `main` |
| carreerV1 | Type-check + unit tests → Astro build → Playwright E2E → Pages deploy | push to `main` |
| career-editor-kmp | `Tests`: unit tests (`testDebugUnitTest`) | push to `development`/`main`, pull requests |
| career-editor-kmp | `Release APK`: signed APK → Firebase App Distribution (secrets read from Doppler `folio`) | push of a `v*` tag, or manual |

## Conventions

- **Branches:** app-style repos (`career-api-worker`, `career-editor-kmp`) work on `development`; site repos deploy from `main`.
- **Data ids:** kebab-case, unique per entity; POST returns `409` on duplicates.
- **Editing data:** prefer the editor app or `curl` PATCH calls (see the [API README](https://github.com/Sebas1705Carreer/career-api-worker#readme)); if you re-seed, keep `seed-kv.mjs` and `src/seed-data.ts` in sync.
- **Secrets:** Doppler project `carreer` is the single source of truth — inventory and runbook in [operations/secrets.md](operations/secrets.md). Never commit values; `.env`/`.dev.vars` are gitignored.

## Open backlog

- Write `long_desc` for the 10 projects added on 2026-08-07 (their modal falls back to the short desc) and decide whether any of them join the CV role lists in carreerV2's generator.
- Regenerate the worker seeds from production (skills/education/certs currently lag live).
- Folio: lint in CI (unit tests and signed releases through App Distribution exist already).
- Doppler dashboard syncs (GitHub Actions, Cloudflare Workers) — rotation currently propagates manually.
- Consider archiving carreerV1 on GitHub (it is legacy but still deployed) or retiring its Pages site.
- API: per-language validation when adding locales.
