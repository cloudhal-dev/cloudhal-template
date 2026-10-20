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

## Theme reference

### Routes
- `[...path].astro` — pages at their path, posts at `/<collection>/<slug>`, category and tag archives at `/category/<slug>` and `/tag/<slug>` (`/page/<n>` for older posts). Anything else follows the site's redirects (301), answers 410 for a retired term, or 404.
- `[...path]/rss.xml.ts` — a feed per category and tag (`/category/<slug>/rss.xml`).
- `rss.xml.ts`, `sitemap.xml.ts`, `robots.txt.ts` — the site feed, the sitemap (pages, posts and indexable archives) and robots rules.
- `search.json.ts` — the search modal's answers (RFC-0029), where the deployment answers searches (`cloudhal dev`, the Cloudflare Site Worker). `components/SearchDialog.astro` is the modal's markup and words; Cloudhal's `/_cloudhal/search.1.js` wires it, and without it the header form opens `/search`.

### Styles (`src/styles`)
`global.css` (imported by `layouts/Base.astro`) loads Tailwind, `fonts.css` and one file per feature from `styles/features/` (`tokens.css` with `@theme`, `prose.css`, `surfaces.css`, `motion.css`, `chrome.css`, `carousel.css` …) in cascade order. The rules are unlayered and later files override earlier ones, so keep the order; add a feature as a new file and import it from `global.css`. `source("..")` on the Tailwind import is relative to `global.css`, so that import stays there.

### Theme settings (`src/theme.ts`)
- `ARCHIVE_PAGE_SIZE` — posts per archive page (20).
- `NAV_CATEGORIES` — append top-level categories (by position) to the header after the pages; `false` for a pages-only menu.
- `RELATED_POSTS` — how many related posts to show under an article (0 hides the list).

### Blocks (`src/blocks.ts`)
Typed blocks (RFC-0015) go in page sections and inside article and rich-text bodies. Cloudhal ships `callout`, `button`, `embed` and `faq`; this theme adds `testimonial`.
- **Declare** a type in `src/blocks.ts` (`blockTypes`): a plain English name shared by every theme (`pricing-table`, never `mytheme-pricing`), where it may go, its props in the JSON Schema subset, whether it wraps text, and an example. The file is data only, with type imports only (`import type { BlockType }`); the CLI checks every type.
- **Render** it with a component in `theme.ts` → `blocks` (`{ testimonial: Testimonial }`). The component gets `{ props, content, contentHtml, media }`. `contentHtml` is the wrapped text, already rendered safely; props are checked against the type first. A built-in may be listed there too, to restyle it.
- `embed` shows a link card; Cloudhal's small player module turns it into the YouTube (no-cookie) or Vimeo player on click, and the page's CSP allows those two frames only where a video is shown. The page route sets `x-cloudhal-embeds` for that (`usesEmbeds`); keep it if you rewrite the route.
- `cloudhal studio` and `cloudhal mcp` read the file, so Studio's forms and `/` menu and agents (`list_block_types`) offer the type. `cloudhal deploy` checks it and bundles it into the admin Worker.

### Design settings (`src/settings.ts`)
What Studio's Settings › Appearance, agents (`set_theme_settings`) and the generator may change about the look (RFC-0017): brand, link, background and text colors, the typeface and the corners. The file is data only, in the same JSON Schema subset as block props, with `x-contrast` rules that hold color pairs to WCAG AA and `x-labels` for the choices' words. `layouts/Base.astro` turns the site's values into `--ch-*` CSS variables and `data-ch-*` attributes (`themeVariables`), and the stylesheets in `styles/features/` use them. Add a setting by declaring it here and using its variable or attribute in your CSS.

### Analytics
Themes need nothing for analytics (RFC-0006). Deployed with `cloudhal deploy`, the Site Worker adds the first-party script tag to published pages while analytics are on (`cloudhal.json` → `analytics`, then per site in Studio). `cloudhal dev` records your own visits only with `"dev": true` in that block, which `astro.config.mjs` passes to the `cloudhal()` integration. Static builds (`cloudhal build`) carry no script. What is collected, and a paragraph for your privacy policy: `docs/guides/analytics-privacy.md` in the Cloudhal repository.

### Redirects in static builds
A static host cannot be told to answer with a status code by HTML alone. `cloudhal build` therefore writes each redirected path as a small meta-refresh page and also writes `dist/_redirects`, which Cloudflare (Workers/Pages static assets) and Netlify turn into real 301s. Deployed with `cloudhal deploy`, the Site Worker answers redirects itself.
