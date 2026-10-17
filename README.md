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

## Notes

- Chrome's Auto Dark Mode skips the page because `<meta name="color-scheme" content="light dark">`
  and `color-scheme` on the root declare that dark is already handled natively.
- Nothing is fetched from a third party (both font faces are self-hosted), so ad/privacy blockers
  have no request to cancel, and no class or id name collides with common cosmetic-filter selectors.
- No build step. Open `index.html` or serve the folder: `python3 -m http.server 8000`.
