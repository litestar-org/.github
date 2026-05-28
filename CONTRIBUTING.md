# Contribution guide

> This guide applies to **litestar.dev** — the marketing and community website
> for the Litestar framework. If you are looking to contribute to the
> framework itself, see
> [`litestar-org/litestar/CONTRIBUTING.rst`](https://github.com/litestar-org/litestar/blob/main/CONTRIBUTING.rst).

## Setting up the environment

1. Install [Node.js 24](https://nodejs.org/) (the version pinned in
   `.github/workflows/*.yml` — `nvm install 24` works well).
2. Enable Corepack so the project's pinned `pnpm` version is used:
   ```bash
   corepack enable
   ```
3. Install dependencies:
   ```bash
   pnpm install
   ```
   This also runs `nuxt prepare` (regenerates `.nuxt/`) and installs the
   git hooks via [`prek`](https://github.com/j178/prek) using the local
   `.pre-commit-config.yaml`. The hooks run `oxfmt` (format) and `oxlint --fix`
   on changed files before every commit.

## Code contributions

### Workflow

1. [`Fork`](https://docs.github.com/en/get-started/quickstart/fork-a-repo) the
   repository.
2. Clone your fork locally with git.
3. [Set up the environment](#setting-up-the-environment).
4. Create a branch for your change.
5. Make your changes. Run `pnpm dev` (http://localhost:3000) to see them
   live.
6. **(Optional)** Run `pnpm format && pnpm lint && pnpm test-all` to mirror
   what CI checks. The git hooks already run format + lint on commit, but
   running it manually surfaces issues sooner.
7. Commit your changes to git.
8. Push the changes to your fork.
9. Open a [`pull request`](https://docs.github.com/en/pull-requests). Give
   the pull request a descriptive title indicating what it changes. If it
   has a corresponding open issue, the issue number should be included.
   Example: an issue titled `Bug: Header logo jumps on first paint #42` →
   PR titled `Fix #42 - Reserve space for the header logo to prevent CLS`.

### Guidelines for writing code

- **Language**: TypeScript + Vue 3 SFCs using `<script setup lang="ts">` and
  the Composition API. Keep `.vue` files small and prefer composables
  (`app/composables/`) over duplicated logic.
- **Types**: All code should be fully typed. Avoid `any`; prefer narrower
  types and helpers like `unknown` + type guards.
  - When a type is non-trivial, define it in `app/types/` and import it.
  - If a third-party type is wrong and you can't reasonably work around it,
    use a targeted `@ts-expect-error` or `as` cast with a comment explaining
    why. Don't blanket-suppress.
- **Lint & format**: enforced by [`oxlint`](https://oxc.rs/) and
  [`oxfmt`](https://oxc.rs/). CI runs `pnpm format:check` and
  `pnpm lint:check`; both must be clean. The pre-commit hook auto-fixes
  most issues.
- **Nuxt UI**: prefer composing `@nuxt/ui` v4 components
  (`UButton`, `UPageHero`, `UPageCard`, etc.) over hand-rolled markup —
  this keeps the design system coherent. The full component reference
  lives at https://ui.nuxt.com/.
- **Styling**: Tailwind CSS v4 via the `@theme` block in
  `app/assets/css/main.css`. There is no `tailwind.config` — design tokens
  (the `litestar-*` palette, fonts) live in CSS. Use the `text-primary`,
  `text-highlighted`, `text-muted` semantic aliases instead of raw color
  classes where possible.
- **Accessibility**: build-time HTML validation runs via
  `@nuxtjs/html-validator`. Keep landmarks unique, label form controls,
  and avoid nesting interactive elements.
- **Tests**: changes to composables, utilities, or any non-trivial logic
  must include tests. See [Writing and running tests](#writing-and-running-tests).
- **Comments**: write them only when the *why* is non-obvious. Don't
  describe what the code does — name things well instead.

### Writing and running tests

Tests live under `test/` and are organized into three Vitest projects:

| Project | Command | Purpose |
|---------|---------|---------|
| `unit`  | `pnpm test-unit`  | Pure-function unit tests (helpers, composables) |
| `nuxt`  | `pnpm test-nuxt`  | Tests that need the Nuxt runtime (`@nuxt/test-utils`) |
| `build` | `pnpm test-build` | Post-build smoke tests (e.g. SEO/OG meta scan over `dist/`) — run **after** `pnpm generate-github` |

Run unit + nuxt projects together with `pnpm test-all`. CI runs the same
matrix on every pull request (`.github/workflows/pr.yml`).

If you change anything that affects the static output (routing, SEO,
sitemap, OG images, prerendering), also run:

```bash
pnpm generate-github   # produces .output/public/ (also symlinked as dist/)
pnpm test-build        # SEO meta scan over the generated HTML
```

## Content contributions

This site is content-driven via [`@nuxt/content`](https://content.nuxt.com/).
You usually don't need to touch Vue files to add a blog post, a plugin
entry, or a starter template — drop a Markdown / YAML file in the right
place and `nuxt generate` picks it up.

| What you want to add | Where it lives | Format |
|----------------------|----------------|--------|
| Blog post            | `content/blog/`        | Markdown with YAML frontmatter (`title`, `description`, `date`, `authors`, `category`) |
| Maintainer / team member | `content/maintainers/` | Markdown / YAML — see existing entries for the schema |
| Sponsor              | `content/sponsors/`    | Markdown / YAML |
| Starter / template   | `content/starters/` or `content/templates/` | Markdown / YAML |
| Page-level copy      | `content/about.yml`, `content/index.yml`, `content/deploy.yml`, `content/template.yml` | YAML — schemas are defined in `content.config.ts` |

After editing content, run `pnpm dev` and confirm the page renders correctly.
Run `pnpm test-all` afterwards — Nuxt Content validates frontmatter against
the typed collection schemas at build time, so a malformed entry surfaces
during tests or generation.

## Visual / UI contributions

For any change that affects how a page looks or behaves:

1. Start the dev server: `pnpm dev`.
2. Verify the golden path **and** at least one edge case (dark mode,
   mobile width via DevTools, keyboard navigation if relevant).
3. Run `pnpm generate-github` to catch SSR / prerender / OG-image render
   errors that only surface at build time.
4. If you changed any global layout (header, footer, error page), confirm
   the affected routes still render — spot-check `/`, `/about`,
   `/blog`, `/plugins`, plus any 404 path (e.g. `/does-not-exist`).

## Releasing and deploying

> This section is informational — there is no manual release step.

- The `main` branch is the source of truth.
- `.github/workflows/deploy.yml` deploys to GitHub Pages on every push to
  `main`, and again every day at 03:00 UTC via cron (the cron run also
  refreshes `data/stats.json` via `pnpm run update-stats`).
- The deployed site lives at https://litestar-org.github.io/litestar.dev-v2/
  (baseURL `/litestar.dev-v2/`). Local builds default to `/` so
  `npx serve .output/public` works without env tweaks.

Merging to `main` is therefore the release. Keep PRs focused, ensure CI is
green, and the rest is automatic.
