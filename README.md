# Deploy to Evolving Edge CDN

A GitHub Action that publishes a built static site to the [Evolving Edge](https://www.evolvingedge.ai) CDN. Build your site with any tool, then add this step. It packages the output folder and deploys it to your domain.

The workflow needs no Node.js setup of its own, so it works the same after a Hugo, Jekyll, Zola, MkDocs or Sphinx build as after an npm one.

Evolving Edge is in open beta.

## Before you begin

You need:

1. **A project** in the [Evolving Edge portal](https://portal.3dge.app), and its project ID (`proj_...`).
2. **A deploy token** from the portal, under **Workload → Deploy Tokens**, with `workload:write`, scoped to that project. Save it as a repository secret, for example `EE_CDN_TOKEN`.
3. **Your domain, claimed by that project.** Only the project that claimed a domain can deploy to it. See [Custom domains](https://docs.evolvingedge.ai/custom-domains/).

## Usage

```yaml
name: Deploy

on:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5

      - uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true

      - run: hugo build --gc --minify

      - uses: evolving-edge/deploy-action@v1
        with:
          directory: public
          domain: www.example.com
          project: proj_abc123
          token: ${{ secrets.EE_CDN_TOKEN }}
          trailing-slash: true
```

Each deploy replaces the whole site on that domain. Allow up to about 90 seconds for it to reach every node.

There are step-by-step guides for [Astro, Hugo, Build Awesome (formerly Eleventy), Jekyll, VitePress and Docusaurus](https://docs.evolvingedge.ai/static-sites/).

## Inputs

| Input | Required | Description |
|---|---|---|
| `domain` | Yes | The domain to deploy to, as `www.example.com`. It must be claimed by the project. |
| `token` | Yes, unless `dry-run` | The deploy token. Pass it from a secret. |
| `project` | If the token is scoped to a project | The project ID, `proj_...`. |
| `directory` | No | The folder to deploy, such as `public`, `dist` or `_site`. Left empty, the output folder of most generators is found by itself. |
| `working-directory` | No | Where the project is, if it isn't at the root of the repository. Default `.`. |
| `level` | No | `0`, public (the default), or `2`, released only to visitors with an access token. Level 1 isn't available here: see [Encryption levels](#encryption-levels). |
| `name` | No | The workload's name. Defaults to the domain. |
| `trailing-slash` | No | `true` redirects `/about` to `/about/`. For generators that write `about/index.html`. |
| `clean-urls` | No | `true` serves `/about` from `about.html`. |
| `images` | No | `webp` or `avif` converts JPEG and PNG images while packaging and rewrites every reference to them. `off` overrides `ee.jsonc`. See [Images](https://docs.evolvingedge.ai/image-conversion/). |
| `image-hosts` | No | Other hosts the site is served from, separated by spaces, commas or new lines, so full addresses on them count as the site's own. |
| `image-report` | No | Write the image conversion report as JSON to this path. |
| `image-max-width` | No | Scale JPEG and PNG images wider than this many pixels down to it, keeping their names and formats. |
| `builder-version` | No | Pin the packaging tool to a version instead of the latest. |
| `dry-run` | No | `true` packages the site and uploads nothing. Needs no token. |

## Outputs

| Output | Description |
|---|---|
| `url` | `https://<domain>` once the deploy is accepted. Empty on a dry run. |

## Encryption levels

- **Level 0** is a public site.
- **Level 2** content is released only to visitors with a short-lived access token that your own server requests. See [Workloads](https://docs.evolvingedge.ai/workloads/).
- **Level 1** isn't available from a workflow. Its key is made for each deploy and must reach only the browser, and a workflow run has no safe place to give it to you. Deploy Level 1 sites from the portal or from your own computer with the [command line](https://docs.evolvingedge.ai/cli/).

## Node.js

The Action needs Node.js 22.12 or later. If the runner already has it, the Action uses it and changes nothing. Otherwise it installs Node.js 22.12, which stays first on `PATH` for the rest of the job: if a later step needs another version, set that up after this step.

## Versions

- **`@v1`** gets every 1.x release, including new versions of the deploy tool.
- **An exact version, such as `@v1.0.0`,** keeps your deploys on one release until you change it.

Each release of this Action runs one fixed version of [`@evolving-edge/ee-cdn-cli`](https://www.npmjs.com/package/@evolving-edge/ee-cdn-cli), checked against its integrity hash on every run. The release notes say which.

## Test a workflow first

Run it once with `dry-run: true`. The site is packaged exactly as it would be deployed, nothing is uploaded, and no token is needed.

## More

- [Deploying from GitHub Actions](https://docs.evolvingedge.ai/cli/#deploying-from-github-actions)
- [Redirects](https://docs.evolvingedge.ai/redirects/), [headers](https://docs.evolvingedge.ai/headers/) and [custom 404 pages](https://docs.evolvingedge.ai/custom-404/)
- [Support](https://docs.evolvingedge.ai/customer-support/)

## License

[MIT](LICENSE)
