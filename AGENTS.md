# AGENTS.md

Personal portfolio of Kiarash Akbari (AI software engineer). Live at
https://KiarashAkbari.github.io — deployed by GitHub Pages from `main`, repo root.
Pushing to `main` is deploying; there is no staging.

## Shape of the repo

The main page is one self-contained file, with only a standalone 404 companion.
No build step, no package manager, no dependencies, no framework — do not
introduce any of those.

| Path | What it is |
|---|---|
| `index.html` (~188 KB, ~1760 lines) | The whole main page: markup, CSS, JS, i18n dictionaries, and the Host Grotesk font inlined as base64 |
| `404.html` | Small standalone branded not-found page for GitHub Pages; keep asset paths root-absolute |
| `fonts/Lahzeh-*.ttf` | Primary Persian/Arabic-script faces (100–900), owner-supplied; see `fonts/Lahzeh-NOTICE.md` before reuse/conversion |
| `fonts/Vazirmatn-arabic.woff2` | Arabic-script fallback face, loaded from disk (referenced in `index.html`) |
| `fonts/HostGrotesk-latin.woff2` | Source copy of the inlined Latin face — `index.html` uses the data-URI, not this file |
| `assets/Kiarash_Akbari_Resume.pdf` | Résumé download target |
| `assets/og.png` | 1200 × 630 social share image referenced by Open Graph/Twitter meta |
| `.nojekyll` | Makes Pages serve the folder verbatim. Keep it. |
| `README.md` | Long-form rationale for the guard, contact form, and hosting choices — read it before touching those areas |

Run locally: `python3 -m http.server 8000` then open http://localhost:8000
(add `?lang=fa` for Persian / RTL).

## Layout inside `index.html`

In document order:

1. **Pre-paint script** (top of `<head>`): resolves theme + language before
   first paint. Priority: `localStorage` (`ka-theme`, `ka-lang`) → URL
   (`?lang=fa|en`, `#fa`) → `prefers-color-scheme` / browser language.
2. **Main stylesheet** `<style data-ka>`: all layout/component CSS, `@font-face`.
3. **`<style id="ka-hard" data-ka>`**: the hardened palette — see gotchas.
4. **Render guard** `<script data-ka>`: neutralises styles injected by
   browser extensions (Dark Reader etc.). Exposes `KA.guard.stats()/.off()/.reset()`.
   Escape hatches: `?ext=keep|deep|block`.
5. **Body chrome + sections**: fixed `#sitehead` (brand + section nav + GitHub),
   `#hero`, `#intro`, `#work` (project carousel, canvas vizzes
   `bim|nids|clone|vino`), `#about`, `#focus`, `#process` (`#flow` canvas),
   `#stack`, `#experience`, `#contact` (`#sendbox` form), footer, plus a Tetris
   easter egg (`#tetris`).
6. **Main script** (end of body): builds `window.KA` (theme reader, event bus
   `KA.on/emit`, helpers `clamp/smooth/hash/faNum`), then IIFE modules for
   i18n, the theme/lang panel (`#pxctl`), 0c glide/header, the interactive
   `#field` canvas background, text splitting/reveals, carousel, vizzes, and
   the contact form.

## Gotchas — read before editing

- **Colour tokens live only in `#ka-hard`.** They are declared at
  `html:not(#ka-sentinel):not(#ka-sentinel2)` specificity to outrank
  extension sheets. Editing the same tokens in `:root` silently does
  nothing — the hardened copy wins. Edit colours in `#ka-hard`.
- **Any new stylesheet must be `<style data-ka>`.** The guard treats any
  unmarked style in the document as foreign and disables it. Same marker
  on scripts you add to `<head>`.
- **Don't "fix" the guard's blind spots.** It deliberately ignores
  user-origin `!important` styles, forced-colors / high-contrast, and
  turns itself off under `prefers-contrast`/`forced-colors`. That is
  accessibility, on purpose (see README).
- **Every user-visible string needs both languages.** Copy is keyed via
  `data-i18n` (and `data-i18n-aria` for `aria-label`s) against the EN/FA
  dictionaries in the main script. Adding text in English only leaves the
  Persian version wrong. Persian digits go through `faNum`. Test RTL.
- **Keep the Persian font order and ranges.** `--sans` is Host Grotesk →
  Lahzeh → Vazirmatn; Lahzeh/Vazirmatn faces carry the Arabic-script
  `unicode-range`, so Latin text and ASCII digits stay on Host Grotesk.
  Lahzeh is owner-supplied and commercially licensed — read
  `fonts/Lahzeh-NOTICE.md` before converting, subsetting, or replacing it.
- **Canvas code reads colours from CSS.** `KA.readTheme()` pulls
  `--grid/--tile-bg/--tile-grid/--paper/--ink` via `getComputedStyle`;
  renaming tokens breaks the canvases, not the CSS.
- **Keep scrolling on the real window position.** Module 0c eases the actual
  `window.scrollTo()` toward accumulated fine-pointer wheel deltas; do not
  replace it with a transformed/virtual body, because canvases and observers
  rely on real `scrollY`. Preserve its native-input exclusions (touch,
  keyboard, scrollbar/find, Ctrl-wheel zoom, horizontal carousel gestures,
  nested `[data-noglide]` scrollers), `?scroll=native`, reduced-motion bypass,
  and the section `scroll-margin-top` that clears `#sitehead`. Do not add
  backdrop blur to the fixed header/panel over the animated field. Likewise,
  do not add header padding or extra CTA rows inside `.hhead`: its height sets
  where the pixel cloud starts, and `.intro` overlaps `.hero` by a fixed
  negative margin, so either change silently compresses the name → pixels →
  lead rhythm. `#sitehead` therefore stays hidden until `.hhead` has passed.
- **No third-party requests on load.** Fonts are self-hosted/inlined by
  design (privacy blockers, and filtered networks). The only outbound
  request is the contact form submit. Don't add CDN fonts/scripts/analytics.
- **Contact form is FormSubmit** (`#sendbox` → `formsubmit.co`). JS posts
  JSON to `/ajax/` with a native `form.submit()` fallback; JS-off uses the
  real `action`/`method`. Keep both paths and the `_honey` honeypot.
- **Custom domain = two lines**: `<link rel="canonical">` and `og:url`
  in `<head>`, plus Settings → Pages. Keep Cloudflare out of the request
  path (README explains why).

## Verify a change

No test suite — verify in a browser after serving locally:

- [ ] Light and dark theme (`#pxctl` toggle, and fresh load with each OS setting)
- [ ] English LTR and Persian RTL (`?lang=fa`), including numerals
- [ ] Narrow/mobile viewport: work carousel, panel, Tetris teaser
- [ ] Fine-pointer trackpad/wheel glide; hero masthead unobstructed at the top,
      then header hide-down/show-up with no jitter; scrollbar, keyboard, touch
      and Ctrl-wheel zoom still native
- [ ] `prefers-reduced-motion`: canvas/animations should calm down and glide is off
- [ ] JavaScript disabled: content visible (the `<noscript data-ka>` style),
      contact form still submittable
- [ ] Console clean; `KA.guard.stats()` sane; no new network requests on load
