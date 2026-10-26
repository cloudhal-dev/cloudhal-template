# Cloudhal blog template

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/cloudhal-dev/cloudhal-template)

A [Cloudhal](https://github.com/cloudhal-dev/cloudhal) blog you can deploy with one click. Cloudhal is the agent-native site platform: agents write and edit through MCP, every change is a snapshot with a diff and a preview, and a person approves each publish.

## What the button creates

On your Cloudflare account, through [Workers Builds](https://developers.cloudflare.com/workers/ci-cd/builds/):

- a copy of this repository in your GitHub account; every push to its default branch deploys again
- **`cloudhal-blog-site`**: the site, rendered on demand and cached at the edge
- **`cloudhal-blog-admin`**: Studio, the API and the MCP server
- the D1 database `cloudhal-blog-db` and the R2 bucket `cloudhal-blog-blobs`, shared by both Workers
- a starter blog: a home page, an about page and a first post

You can rename the Worker and the resources in the form. The admin Worker is always named after the site Worker, without `-site`, plus `-admin`. Workers Free is enough to start.

## After the first build: create the owner account

The build does not print a setup link, because build logs are kept. Its last lines tell you what to do instead:

1. In the Cloudflare dashboard, open **Workers & Pages › cloudhal-blog-admin › Settings › Variables and Secrets**. Edit `CLOUDHAL_SETUP_TOKEN` and enter a random value of your own, 32 characters or more. Saving deploys it.
2. Open `https://cloudhal-blog-admin.<your-subdomain>.workers.dev/login#setup=<that value>` and create your account. You become the owner, and sign-up closes.

In Studio, create a key for each agent and connect them to the MCP server at `https://cloudhal-blog-admin.<your-subdomain>.workers.dev/mcp`.

## Working with this repository

- **Content** lives in the deployment, not in Git. Edit it in Studio or through agents.
- **Theme and code** live here (`src/`). Push to the default branch to deploy. The build reuses the Workers' secrets and never pushes over the deployment's content.
- **Settings** go in `cloudhal.json` (analytics, comments, mail, AI and others). The site id in it must not change.
- A clone has no local site, so `npm run dev` and a local `npm run deploy` do not work in it. To work on a theme locally, create a project with `npm create cloudhal@latest`.

The [deploy button guide](https://github.com/cloudhal-dev/cloudhal/blob/main/docs/guides/deploy-button.md) covers how the build works and its limits: custom domains need more token permissions, and there is no downgrade check or cache purge between builds.

## Staying current

- **This repository**: every Cloudhal release opens a PR here that moves `@cloudhal/*` to the new version and replaces `src/`, `public/`, `astro.config.mjs` and `tsconfig.json` with that version's blog starter. Its `cloudhal/template-check` status (`npm ci`, then `cloudhal deploy --dry-run` in Workers Builds mode, run by Cloudhal's release workflow) must be green before it is merged, so the button always deploys the latest release.
- **Your copy**: the theme is yours from the moment the copy is made. If the copy has `.github/dependabot.yml`, Dependabot opens a PR for each `@cloudhal/*` release (dependencies only). Nothing checks it in your copy, and merging it deploys: run `npm ci` and `WORKERS_CI=1 npx cloudhal deploy --dry-run` on its branch first. Releases keep older themes working; in 0.x a minor version may break, so read the [release notes](https://github.com/cloudhal-dev/cloudhal/releases) before merging a minor bump. Delete `.github/dependabot.yml` to stop the PRs.

## Theme reference

The theme in `src/` is Cloudhal's blog starter, as released with the version in `package.json` when the copy was made. Its reference (routes, styles and fonts, theme settings, blocks, design settings, analytics, redirects in static builds) is the starter's README: [`starters/blog/README.md`](https://github.com/cloudhal-dev/cloudhal/blob/main/starters/blog/README.md) on `main`, or the same file under that version's tag (`https://github.com/cloudhal-dev/cloudhal/tree/v<version>/starters/blog`).
