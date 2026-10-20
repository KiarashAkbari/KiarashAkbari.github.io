# Kiarash Akbari — Portfolio

Personal portfolio of **Kiarash Akbari**, AI software engineer (backend & AI-integrated systems).

**Live:** https://kiarashakbari.github.io · **Persian (RTL):** https://kiarashakbari.github.io/?lang=fa

A single static page built with vanilla JS and Canvas 2D. No build step, no framework, no runtime dependencies.

## Features

- Interactive pixel-field background that reacts to scroll and cursor
- Light/dark theme and English/Persian (RTL) with Persian numerals
- Project carousel with live canvas visualisations (BIM, NIDS, clone, vino)
- Smooth desktop scroll glide on the real window position; touch and keyboard stay native
- Contact form that works with and without JavaScript
- Mobile-first: safe-area padding, 44 px touch targets, lightweight mode on weak/Save-Data devices
- Respects `prefers-reduced-motion`, `prefers-contrast` and forced colors
- Privacy-friendly: no third-party requests on load, self-hosted fonts

## Tech

HTML, CSS and JavaScript in one file, Canvas 2D, self-hosted Host Grotesk, Lahzeh and Vazirmatn fonts, FormSubmit for the contact relay, GitHub Pages for hosting.

## Project structure

| Path | Purpose |
|---|---|
| `index.html` | The whole page: markup, CSS, JS, i18n dictionaries, inlined Latin font |
| `404.html` | Standalone branded not-found page |
| `assets/` | Résumé PDF and `og.png` (1200 × 630 social image) |
| `fonts/` | Persian/Arabic-script fonts and the Latin source font |
| `.nojekyll` | Makes Pages serve the folder as-is. Keep it |
| `AGENTS.md` | Contributor/agent guide with gotchas |

## Run locally

```bash
git clone https://github.com/KiarashAkbari/KiarashAkbari.github.io.git
cd KiarashAkbari.github.io
python3 -m http.server 8000
```

Open http://localhost:8000 (add `?lang=fa` for Persian).

Useful URL flags: `?scroll=native` disables the scroll glide, `?ext=keep|deep|block` controls extension handling.

## Design decisions

**Motion.** Interface chrome moves in `steps(n)` like sprite animation; text reveals use an eased curve. Pixel canvases use `image-rendering: pixelated`.

**Scrolling.** The glide eases the real `window.scrollTo()` position, never a transformed body, so canvases and observers see true `scrollY`. The fixed header uses no `backdrop-filter`, to protect frame rate.

**Mobile.** On coarse/small/Save-Data devices decorative canvases idle at 30 FPS and pause during scroll. The pixel field keeps 60 FPS and repaints every frame while scrolling.

**Render integrity.** Colour tokens live only in `#ka-hard`. A guard script neutralises style-injecting extensions such as Dark Reader, but deliberately leaves user-origin accessibility styles alone. Every added stylesheet or script in `<head>` needs the `data-ka` attribute. Debug from the console with `KA.guard.stats()`, `.off()`, `.reset()`.

**Contact form.** `#sendbox` posts to [FormSubmit](https://formsubmit.co) via `/ajax/`, falling back to a native POST. A `_honey` honeypot handles spam. The first submission triggers a one-time activation email. Web3Forms or Formspree can replace it by changing the form `action`.

## Deployment

GitHub Pages deploys `main` from the repo root. **Pushing to `main` is deploying; there is no staging.**

- Keep the repo public on the free plan.
- `github.io` is used because Cloudflare free hostnames (`*.pages.dev`, `*.workers.dev`) are filtered in Iran. Cloudflare stays out of the request path.
- Custom domain: update `<link rel="canonical">` and `og:url` in `<head>`, then set the domain in Settings → Pages. Point an apex ALIAS and a `www` CNAME at `KiarashAkbari.github.io`.

## Contributing / editing checklist

Read `AGENTS.md` first. Before pushing, check:

- [ ] Light and dark theme
- [ ] English LTR and Persian RTL
- [ ] Narrow mobile viewport
- [ ] Reduced-motion behaviour
- [ ] JavaScript disabled: content visible, form submittable
- [ ] Console clean and no new network requests on load

## Credits and licensing

The Lahzeh font files are owner-supplied and commercially licensed. See `fonts/Lahzeh-NOTICE.md` before reusing, converting or subsetting them. Vazirmatn and Host Grotesk are open-source fonts.

© Kiarash Akbari. All rights reserved unless stated otherwise.
