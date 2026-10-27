# Portfolio assets standard (v1)

Every project shown in the portfolio carries its own visuals in its repository. The portfolio
([carreerV2](https://github.com/Sebas1705Carreer/carreerV2) at runtime and in its build-time snapshot,
[carreerV1](https://github.com/Sebas1705Carreer/carreerV1) at build time) reads them from
the repo instead of hosting copies.

## Location

```
.github/portfolio/
  icon.png            required  512×512 PNG, the project's real icon/logo (transparent or solid bg)
  banner.png          required  1280×640 PNG (GitHub social-preview size, 2:1), < 1 MB
  portfolio.json      optional  metadata, see below
  case-study.en.md    optional  long-form write-up (English)
  case-study.es.md    optional  long-form write-up (Spanish)
```

PNG only: `raw.githubusercontent.com` serves SVG as text/plain, so it cannot be used in <img>.

## portfolio.json

```json
{
  "accent": "#0F766E",
  "icon_alt":   { "en": "Owlsplit app icon", "es": "Icono de la app Owlsplit" },
  "banner_alt": { "en": "Owlsplit: shared expenses for Android and iOS", "es": "Owlsplit: gastos compartidos para Android e iOS" }
}
```

All fields optional. `accent` is a hex colour the portfolio may use for the card/fallback.

## Resolution (from the project's `github` field in the [career API](https://api.sebas1705.dev/projects))

| `github` value                         | asset repo          |
|----------------------------------------|---------------------|
| `https://github.com/<owner>/<repo>`    | `<owner>/<repo>`    |
| `https://github.com/<org>` (org link)  | `<org>/.github`     |
| null (work/NDA project)                | none → fallback     |

File URL: `https://raw.githubusercontent.com/<owner>/<repo>/HEAD/.github/portfolio/<file>`

Precedence for the banner: API `image_url` (explicit override) → repo `banner.png` → generated
fallback (project initials on the accent colour, or a neutral tone). The icon follows the same
order without an API override. Alt text: API `image_alt` → `portfolio.json` → project name.

Archived repositories are read-only and cannot receive assets; they use the fallback unless the
API sets `image_url`.

## Banner template

Generated from one template so all banners look like a family: accent-coloured background with a
subtle gradient, the icon on the left, project name and one-line tagline on the right, optional
small stack chips. Real screenshots may be composed on the right for apps that have them.

## Adoption

Assets were added by one `portfolio-assets` PR per public project repo (and in the
`.github` repos of the Templetry, Sebas1705Carreer and AxiomNode orgs), all generated from the
same template. `banner.png` also fits GitHub's social preview (Settings → General → Social
preview), which each repo can upload by hand.
