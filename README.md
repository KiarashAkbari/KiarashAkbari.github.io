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

## Notes

- Chrome's Auto Dark Mode skips the page because `<meta name="color-scheme" content="light dark">`
  and `color-scheme` on the root declare that dark is already handled natively.
- No third-party resource is *loaded* (both font faces are self-hosted), so ad/privacy blockers have no
  request to cancel and no class or id name collides with common cosmetic-filter selectors. The single
  outbound request is the form submission above, fired on submit, never on page load.
- No build step. Open `index.html` or serve the folder: `python3 -m http.server 8000`.
