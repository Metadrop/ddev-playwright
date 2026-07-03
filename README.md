# DDEV Playwright add-on

Bakes Playwright browsers and their system dependencies into the DDEV web image
and adds commands to run the tests and view the HTML report.

## Install

From the target project's root:

```bash
ddev add-on get ~/proyectos/ddev/ddev-playwright
ddev restart   # rebuilds the web image so the browsers are baked in
```

> This add-on currently lives only as a local git repo. Once it's pushed to a
> shared remote, swap the path above for that URL so other machines/teammates
> can install it too.

## What it adds

- `.ddev/web-build/Dockerfile.playwright` — installs Chromium + Firefox and their
  OS dependencies into `/ms-playwright` at image build time
  (`PLAYWRIGHT_BROWSERS_PATH`). No install happens at runtime.
- `.ddev/commands/web/playwright` — run tests inside the web container.
- `.ddev/commands/web/playwright-report` — serve the HTML report.
- `.ddev/commands/web/playwright-update-snapshots` — regenerate visual baselines.
- `.ddev/commands/web/playwright-a11y-summary` — print a Markdown accessibility
  summary from the JSON report (via `@lullabot/playwright-drupal`).
- `.ddev/config.playwright.yaml` — exposes the report port (9323) via the router.

## Usage

```bash
# Run the whole suite (passes any Playwright args through)
ddev playwright
ddev playwright --project=chrome-desktop
ddev playwright homepage.spec.js --grep @responsive

# View the HTML report at https://<project>.ddev.site:9323
ddev playwright-report

# Regenerate visual regression baselines (after intentional UI changes)
ddev playwright-update-snapshots
ddev playwright-update-snapshots cisac-visual.spec.js

# Print a Markdown accessibility summary (needs a prior run; reads the JSON report)
ddev playwright --grep @a11y
ddev playwright-a11y-summary
ddev playwright-a11y-summary --mode=annotations   # GitHub ::error annotations
```

> The summary reads `test-results/results.json`, so your `playwright.config.ts`
> must include the `['json', { outputFile: 'test-results/results.json' }]` reporter.

## Configuration

### Test directory

The commands default to `tests/playwright`. To point them elsewhere, set the
variable in `.ddev/config.yaml` (it overrides the default because the add-on does
not fix it in any `config.*.yaml` file):

```yaml
web_environment:
  - PLAYWRIGHT_TEST_DIR=path/to/your/playwright
```

Run `ddev restart` afterwards.

### Playwright version

The browser version is the `PLAYWRIGHT_VERSION` build ARG, whose default is read
from `.ddev/web-build/playwright-version.txt`. Keep it in sync with
`@playwright/test` in your project's `package.json`. After changing it, run
`ddev restart` to rebuild the image. (DDEV has no native build-arg field in
`config.yaml`, so the version lives in this file; you can still override it with
`docker build --build-arg PLAYWRIGHT_VERSION=...` for manual builds.)

## Remove

```bash
ddev add-on remove playwright
ddev restart
```
