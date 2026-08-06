# Secrets — inventory and Doppler runbook

**Doppler is the single source of truth** for every secret in the career ecosystem (project `carreer`). Values never live in git, wikis or chats — only names and where they are consumed.

## Inventory

| Secret | Consumed by | Delivered via |
|---|---|---|
| `API_SECRET` | career-api-worker (Bearer auth for writes) · career-editor-kmp (login) | Doppler → Cloudflare Workers sync (or `wrangler secret put`) |
| `CF_API_TOKEN` | GitHub Actions deploy of career-api-worker | Doppler → GitHub Actions sync (repo secret) |
| `CF_ACCOUNT_ID` | GitHub Actions deploy of career-api-worker | Doppler → GitHub Actions sync (repo secret) |

Non-secrets (public, hardcoded in workflows/config on purpose): `VITE_CAREER_API_URL`, `VITE_BASE_PATH` (carreerV2), `ALLOWED_ORIGIN` (worker `wrangler.toml` var).

## One-time setup

1. Create the account at [doppler.com](https://dashboard.doppler.com) and install the CLI ([instructions](https://docs.doppler.com/docs/install-cli)). On Windows: `winget install doppler.doppler` or scoop.
2. Authenticate and create the project (configs `dev`/`stg`/`prd` are created automatically):

   ```bash
   doppler login
   doppler projects create carreer
   ```

3. Rotate `API_SECRET` first (the old value should be considered exposed), then store the three values — each command prompts for the value so it never touches shell history:

   ```bash
   doppler secrets set API_SECRET --project carreer --config prd
   doppler secrets set CF_API_TOKEN --project carreer --config prd
   doppler secrets set CF_ACCOUNT_ID --project carreer --config prd
   ```

   Mirror into `dev` if the values are shared: `doppler secrets download --project carreer --config prd --no-file --format env` piped as needed, or just set them again in `dev`.

4. **Integrations** (Doppler dashboard → project `carreer` → Integrations):
   - **GitHub Actions** → repo `Sebas1705Carreer/career-api-worker`, map `CF_API_TOKEN` and `CF_ACCOUNT_ID`. The workflow keeps reading `secrets.CF_API_TOKEN` — no CI changes needed; Doppler keeps the repo secrets in sync on every rotation.
   - **Cloudflare Workers** → worker `career-api`, map `API_SECRET`. Alternatively push manually after each rotation: `doppler run --project carreer --config prd -- npx wrangler secret put API_SECRET` (paste the value when prompted).

5. After rotating `API_SECRET`: log in again in career-editor-kmp (it stores its own copy — Android: EncryptedSharedPreferences; Desktop: Java Preferences under `dev/sebas1705/careereditor`).

## Daily use

- `career-api-worker` has a `doppler.yaml`, so inside the repo a one-time `doppler setup` binds it to `carreer/dev`. Then:

  ```bash
  doppler run -- npx wrangler dev
  ```

  injects secrets as env vars — no `.dev.vars`/`.env` files (both are gitignored anyway).
- Rotate a secret: `doppler secrets set <NAME>` → syncs propagate it; re-login the editor if it was `API_SECRET`.
- Audit: the Doppler dashboard keeps per-secret version history and access logs.
