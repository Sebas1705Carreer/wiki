# Secrets — inventory and Doppler runbook

**Doppler is the single source of truth** for every secret in the career ecosystem: project `carreer` (the API and its deploy) and project `folio` (the Folio Android app build and release). Values never live in git, wikis or chats — only names and where they are consumed.

## Inventory

| Secret | Consumed by | Delivered via |
|---|---|---|
| `API_SECRET` | career-api-worker (Bearer auth for writes) · career-editor-kmp (login) | Doppler → Cloudflare Workers sync (or `wrangler secret put`) |
| `CF_API_TOKEN` | GitHub Actions deploy of career-api-worker | Doppler → GitHub Actions sync (repo secret) |
| `CF_ACCOUNT_ID` | GitHub Actions deploy of career-api-worker | Doppler → GitHub Actions sync (repo secret) |

### Folio (Doppler project `folio`)

Names only; values live in Doppler (config `prd` for releases, `dev` for local builds). `API_TOKEN` is the value of `API_SECRET` above, compiled into the APK.

| Secret | Consumed by | Delivered via |
|---|---|---|
| `API_TOKEN` | the Gradle build of career-editor-kmp (compiled into the APK as `BuildConfig.API_TOKEN`) | `doppler run` (`folio/dev` locally, `folio/prd` in CI) |
| `SIGNING_KEYSTORE_BASE64` | signing the release APK | `doppler run` in the release workflow |
| `SIGNING_STORE_PASSWORD` | signing the release APK | `doppler run` in the release workflow |
| `SIGNING_KEY_ALIAS` | signing the release APK | `doppler run` in the release workflow |
| `SIGNING_KEY_PASSWORD` | signing the release APK | `doppler run` in the release workflow |
| `FIREBASE_APP_ID` | Firebase App Distribution upload | `doppler run` in the release workflow |
| `FIREBASE_SERVICE_ACCOUNT_JSON` | Firebase App Distribution upload (service account with the *Firebase App Distribution Admin* role) | `doppler run` in the release workflow |
| `DOPPLER_TOKEN` | the only GitHub Actions secret of career-editor-kmp: a read-only Doppler service token for `folio/prd` | repo secret, set by hand |

Optional non-secret: the repository variable `FIREBASE_TESTER_GROUP` (defaults to `testers`). The release workflow fails naming the first missing Doppler secret. The APK carries the API token, so it is distributed only through Firebase App Distribution and never as a GitHub release asset; rotating `API_SECRET` means updating `API_TOKEN` in `folio/dev` and `folio/prd` and tagging a new version. See the career-editor-kmp README ("The API token", "Publishing a release").

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
   - Manual fallback (no integration): pass values as env vars, **never through PowerShell text pipes** (PS 5.1 re-encodes and can prepend a BOM that breaks the token — wrangler fails with `U+FEFF` in the Authorization header):

     ```bash
     doppler run --project carreer --config prd -- powershell -NoProfile -Command "gh secret set CF_API_TOKEN -R Sebas1705Carreer/career-api-worker --body $env:CF_API_TOKEN; gh secret set CF_ACCOUNT_ID -R Sebas1705Carreer/career-api-worker --body $env:CF_ACCOUNT_ID"
     ```
   - **Cloudflare Workers** → worker `career-api`, map `API_SECRET`. Alternatively push manually after each rotation: `doppler run --project carreer --config prd -- npx wrangler secret put API_SECRET` (paste the value when prompted).

5. After rotating `API_SECRET`: update `API_TOKEN` in the Doppler project `folio` and ship a new Folio build; an installed app keeps its own stored copy (Android: EncryptedSharedPreferences) until the token is entered again in its settings.

## Daily use

- `career-api-worker` has a `doppler.yaml`, so inside the repo a one-time `doppler setup` binds it to `carreer/dev`. Then:

  ```bash
  doppler run -- npx wrangler dev
  ```

  injects secrets as env vars — no `.dev.vars`/`.env` files (both are gitignored anyway).
- Rotate a secret: `doppler secrets set <NAME>` → syncs propagate it; re-login the editor if it was `API_SECRET`.
- Audit: the Doppler dashboard keeps per-secret version history and access logs.
