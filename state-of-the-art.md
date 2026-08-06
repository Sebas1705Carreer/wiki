# State of the art

**Snapshot: 2026-08-07** · maintained as the single up-to-date picture of the career ecosystem: what exists, where it runs and how the pieces fit.

## What this is

A self-hosted "portfolio as a product": one **API** is the single source of truth for all career data (jobs, projects, skills, education…), consumed by a **public portfolio site** and managed through a **native editor app**. Everything lives in the GitHub organization **[Sebas1705Carreer](https://github.com/Sebas1705Carreer)**.

```
career-editor-kmp (Android/Desktop)      carreerV2 (React, live site)
        │  writes [Bearer auth]                 │  reads (runtime)
        ▼                                       ▼
              career-api-worker  (Cloudflare Workers + KV)
              https://career-api.sebas1705.workers.dev
                                                ▲
                                                │  reads (build time)
                                         carreerV1 (Astro, legacy)
```

## Repos & live surfaces

| Repo | Role | Default branch | Live |
|---|---|---|---|
| [career-api-worker](https://github.com/Sebas1705Carreer/career-api-worker) | REST API — Cloudflare Workers + KV, TypeScript. Single source of truth for all career data | `development` | [career-api.sebas1705.workers.dev](https://career-api.sebas1705.workers.dev) · [Swagger UI](https://career-api.sebas1705.workers.dev/docs) |
| [carreerV2](https://github.com/Sebas1705Carreer/carreerV2) | **Active portfolio** — React 19, Vite, Tailwind v4, i18next, in-browser PDF CV generator. Reads the API at runtime, with local JSON fallback | `main` | [sebas1705carreer.github.io/carreerV2](https://sebas1705carreer.github.io/carreerV2/) |
| [carreerV1](https://github.com/Sebas1705Carreer/carreerV1) | **Legacy portfolio** — Astro 4 SSG, Clean Architecture, 10 languages, Playwright E2E. Reads the API at build time. Kept for reference, still deployed | `main` | [sebas1705carreer.github.io/carreerV1](https://sebas1705carreer.github.io/carreerV1/) |
| [career-editor-kmp](https://github.com/Sebas1705Carreer/career-editor-kmp) | Editor — Kotlin Multiplatform + Compose (Android + Desktop JVM), Ktor client, Clean Architecture. Writes to the API with the Bearer token | `development` | ships locally (no releases yet) |
| [wiki](https://github.com/Sebas1705Carreer/wiki) | This documentation hub | `main` | — |

The old personal-account repos redirect here; the previous portfolio repo `Sebas1705/my-portfolio` no longer exists (its successor is `carreerV1`). The personal profile page [Sebas1705](https://github.com/Sebas1705) links into this org.

## The API (v3)

- **Entities:** `languages` and `personal` (singular · GET/PUT/PATCH), plus `jobs`, `projects`, `skills`, `education`, `certifications`, `soft-skills` (arrays with `id` · full CRUD).
- **i18n model:** every localizable field is a `LocalizedString` — `{ [langCode]: value }`. `GET /languages` drives consumers' language switchers; adding a language = extending the maps + updating `/languages`. Currently `en` (default) and `es`.
- **Auth:** reads are public; writes require `Authorization: Bearer <API_SECRET>` (a Cloudflare secret).
- **Docs:** OpenAPI 3.0 at [`/openapi.json`](https://career-api.sebas1705.workers.dev/openapi.json), Swagger UI at [`/docs`](https://career-api.sebas1705.workers.dev/docs).
- **Storage:** Cloudflare KV (`CAREER_KV`). Seeding via `node seed-kv.mjs` (source of truth for re-seeds; `src/seed-data.ts` mirrors it for the worker bundle).
- **CORS:** `*` by default, configurable via `ALLOWED_ORIGIN`.

## CI/CD

| Repo | Workflow | Trigger |
|---|---|---|
| career-api-worker | Deploy to Cloudflare Workers (Wrangler 4, Node 22) | push to `development` |
| carreerV2 | Build + GitHub Pages deploy (`VITE_BASE_PATH=/carreerV2/`) | push to `main` |
| carreerV1 | Vitest unit → Astro build → Playwright E2E → Pages deploy | push to `main` |
| career-editor-kmp | *(no CI yet)* | — |

## Conventions

- **Branches:** app-style repos (`career-api-worker`, `career-editor-kmp`) work on `development`; site repos deploy from `main`.
- **Data ids:** kebab-case, unique per entity; POST returns `409` on duplicates.
- **Editing data:** prefer the editor app or `curl` PATCH calls (see the [API README](https://github.com/Sebas1705Carreer/career-api-worker#readme)); if you re-seed, keep `seed-kv.mjs` and `src/seed-data.ts` in sync.
- **Secrets:** Doppler project `carreer` is the single source of truth — inventory and runbook in [operations/secrets.md](operations/secrets.md). Never commit values; `.env`/`.dev.vars` are gitignored.

## Open backlog

- Add an `og-image.png` to carreerV2 and restore the removed Open Graph image metas.
- CI for career-editor-kmp (build + lint on push), and eventually packaged releases.
- Consider archiving carreerV1 on GitHub (it is legacy but still deployed) or retiring its Pages site.
- Editor: publish signed builds; API: per-language validation when adding locales.
