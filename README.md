# kiarashakbari.github.io

Personal portfolio of **Kiarash Akbari**, AI software engineer (backend & AI-integrated systems).

Live: https://kiarashakbari.github.io · Persian: https://kiarashakbari.github.io/?lang=fa

Single static page: vanilla JS + Canvas 2D, no build step or dependencies at runtime.
Light/dark theme, English/Persian (RTL).

## Render integrity: browser extensions do not restyle this page

Dark-mode, reader and style-injector extensions (Dark Reader, Night Eye, Super Dark Mode, most
Stylus lookalikes) change how a site looks by appending their own `<style>` to `<head>`, sometimes
by hanging a filter on `<html>`, and — in dynamic-theme mode — by patching the canvas 2D context so
every `fillStyle` gets recoloured. Two layers in `<head>` neutralise that:

- **`#ka-hard`** (last stylesheet in `<head>`) declares the palette and the root colours at
  `html:not(#ka-sentinel):not(#ka-sentinel2)` weight. Injected sheets live in the *author* origin, so
  they win on source order alone; two phantom ids outrank anything they generate while leaving the
  `data-theme` switch working. The colour tokens are declared **only** here — edit them in this block,
  not in `:root`, or the hardened copy wins silently and the change looks like it did nothing.
- **the guard script** after it keeps their sheets out of the cascade (`media="not all"` + `disabled`,
  emptied where `disabled` is ignored), drops filter/mix-blend properties hung inline on the root,
  restores native `fillStyle`/`strokeStyle`/`drawImage`/`createPattern`/`fillText` if an extension
  patched them, and re-arms itself when an extension rewrites or re-enables its sheet. If a sheet
  fights back more than twelve times it is removed; a storm parks the guard instead of spinning.

Deliberately **not** fought: user-origin `!important` styles (Stylus, browser high contrast, forced
colors, font and size substitutions for reading/dyslexia support) and any extension UI in a shadow
root. Those are accessibility, not decoration — the guard turns itself off under
`prefers-contrast`/`forced-colors`. A stylesheet injected at the user origin is not in the DOM, so it
cannot be neutralised from the page; that asymmetry is on purpose.

For the visitor: `?ext=keep` respects whatever the extension wants (remembered in `localStorage`
under `ka-ext`), `?ext=deep` also strips Dark Reader's per-element inline overrides, `?ext=block`
forces the guard back on. From the console: `KA.guard.stats()`, `KA.guard.off()`, `KA.guard.reset()`.

When adding a stylesheet to the page, mark it `<style data-ka>` — the guard treats anything in the
document without that marker as foreign and disables it.

## Contact form

`#sendbox` in the closing section posts to [FormSubmit](https://formsubmit.co), a relay that forwards the
submission to the address already printed in the page. No server to run, no key to keep secret; the
trade-off is that the inbox address sits in the markup (it is already a `mailto:` link twice over) and the
**first visitor to submit hits an activation step** — click the link FormSubmit emails to that inbox once
and every message after that arrives normally.

Two paths, deliberately:

- **JavaScript on** — the page posts JSON to the relay's `/ajax/` endpoint, keeps the visitor on the page
  and reports through a status line (`role="status"`, politely announced). If the relay is unreachable,
  blocked, or refuses the cross-origin read, the catch hands the form to `form.submit()`, which still
  delivers.
- **JavaScript off** — a real `action` and `method="POST"` mean the browser posts the form as plain HTML
  and lands on `_next`, which the script rewrites to `location.href` so a custom domain or a preview
  deploy returns to the right place.

Spam handling is a honeypot (`_honey`, hidden with `.sr` rather than `display:none`, which bots read as a
trap) plus `_captcha:false` to keep the relay's own Google reCAPTCHA out of the page. Name and email are
remembered in `localStorage` so a returning visitor retypes nothing; nothing is stored on any server here.

Swapping providers is one attribute: Web3Forms wants `action="https://api.web3forms.com/submit"` and a
hidden `access_key` instead of an address in the URL (250/month free), Formspree wants
`action="https://formspree.io/f/ID"` (50/month, attachments paid). Drop the `/ajax/` rewrite if the new
relay already answers JSON.

## Deploying (GitHub Pages primary, Cloudflare as mirror)

`main` is the only branch that publishes, and both hosts serve the folder as-is with no build step:

- **GitHub Pages is the primary.** It requires a public repo (this one is), and `github.io` is the
  hostname that keeps resolving for the audience this site is written for.
- **The Cloudflare Workers service (`kiarashakbari-website`) is a mirror and a preview rig.** Every
  branch pushed to GitHub gets a `https://<hash>.kiarashakbari-website.<subdomain>.workers.dev` build,
  which is the comfortable way to review a change before merging it.

That split exists because Cloudflare's free hostnames are collateral damage from proxy abuse:
`*.workers.dev` and `*.pages.dev` are filtered in Iran, and blocked wholesale by a lot of corporate
proxies elsewhere, so neither is a place to point a résumé link. The `<link rel="canonical">` in the head
plus `X-Robots-Tag: noindex` in `_headers` (which only Workers reads) keep the mirror out of the search
index. If the Worker ever becomes primary, flip both.

New projects on Cloudflare go to Workers rather than Pages — Cloudflare's own guidance since 2025 is to
use Workers static assets — and Workers Builds reads `wrangler.jsonc` from the repo instead of dashboard
fields, so that file *is* the deploy config for the mirror:

- `assets.directory = "./"` — the repo root is the site.
- `not_found_handling = "none"` — a junk URL gets a real 404, not `index.html` with a 200 (SPA mode
  would poison the search index with soft 404s).
- No `main` script and no `assets.binding` — nothing invokes Worker code, so the 100,000 requests/day
  free quota never applies: requests for static assets are free and unlimited on every plan, with
  20,000 files and 25 MiB per file as the ceilings. This site is 5 files and 162 KB.

`.assetsignore` keeps repo-only files (`README.md`, `wrangler.jsonc`, `.nojekyll`, `.git`) out of the
upload — Workers does not exclude them the way Pages did. `_headers` adds `nosniff`, a referrer policy, a
permissions policy and the `noindex` above to every response, plus a year of immutable caching on the two
font files and the PDF. There is no CSP on purpose: the page is one file of inline `<script>` and
`<style>` blocks, which any sane `script-src`/`style-src` policy would block.

A custom domain reads better on a CV than `kiarashakbari.github.io`, and it can go on either host. **On
GitHub Pages, keep Cloudflare out of the request path:** point the apex (`ALIAS`/flattened `CNAME`) and
`www` (`CNAME`) at `KiarashAkbari.github.io` with the cloud **grey — DNS only** — then save the domain in
Settings → Pages and GitHub issues the certificate. Cloudflare then only answers DNS, so a bad day for
Cloudflare's IP ranges does not take the site down. Proxying it (orange cloud) — or using a Workers
*custom domain*, which requires a Cloudflare zone on the account with Cloudflare nameservers, while
Pages tolerates external DNS — puts Cloudflare's edge in front of every visitor, which is the path that
gets throttled during filtering episodes. Do not enable **Access**: it hides the whole page behind a
Cloudflare login, which is for private dashboards, not portfolios.

Local loop, if you want it: `npx wrangler dev` serves the assets directory at `localhost:8787`, and
`npx wrangler deploy` publishes the mirror from your machine instead of from a push (needs
`npx wrangler login`). Neither is required, and neither adds a dependency to the repo.

## Notes

- Chrome's Auto Dark Mode skips the page because `<meta name="color-scheme" content="light dark">`
  and `color-scheme` on the root declare that dark is already handled natively.
- No third-party resource is *loaded* (both font faces are self-hosted), so ad/privacy blockers have no
  request to cancel and no class or id name collides with common cosmetic-filter selectors. The single
  outbound request is the form submission above, fired on submit, never on page load.
- No build step. Open `index.html` or serve the folder: `python3 -m http.server 8000`.
