# kiarashakbari.github.io

Personal portfolio of **Kiarash Akbari**, AI software engineer (backend & AI-integrated systems).

Live: https://kiarashakbari.github.io · Persian: https://kiarashakbari.github.io/?lang=fa

Single static page: vanilla JS + Canvas 2D, no build step or dependencies at runtime.
Light/dark theme, English/Persian (RTL).

## Navigation and scrolling

The hero name block remains the page's masthead, so the fixed header stays tucked away until
that block has scrolled past and the original name → pixel-field → intro spacing is untouched.
After that, it hides on a deliberate scroll down and returns on scroll up, when keyboard focus
enters it, or when the cursor reaches the top edge. Its links also mark the section currently
in view. The header deliberately uses the translucent paper token without `backdrop-filter`;
repeatedly blurring the full-screen animated canvas underneath is an easy way to lose frames
while scrolling.

Desktop wheel/trackpad scrolling uses a small native-position glide. It does **not** create a
virtual scroll container or transform the body: wheel deltas update a target and the real
`window` position eases toward it with a roughly 92 ms time constant, so every existing
`scrollY` reader (pixel field, reveals, observers and themes) sees the same motion. In-page
anchors ride a separate eased glide and respect the fixed header through `scroll-margin-top`.

Native input always wins. Touch, keyboard, scrollbar dragging, find-in-page, horizontal wheels,
pinch-zoom (`Ctrl` + wheel), the work carousel's horizontal gesture, and nested scrollers
(`textarea`, `select`, `[data-noglide]`) are not intercepted. Touch scrolling is 100% native
without custom touch inertia, keeping the real scroll position throughout. On coarse/touch
pointers, tapped in-page anchors ride a quick managed scroll that cancels immediately on native
touch or scroll and respects `prefers-reduced-motion` with an instant jump. The desktop wheel glide
is off for coarse pointers and `prefers-reduced-motion`. Use `?scroll=native` to disable it for a page
load, or `KA.glideTo(y)` / `KA.quickScrollTo(y)` from the console to test programmatic scrolling.

## Mobile layout and lightweight rendering

Mobile viewports use safe-area padding (`env(safe-area-inset-*)`), consistent gutters across all
sections, an overflow-proof header, comfortable section/hero typography, and touch targets sized to
at least 44 px. Persian RTL correctly formats Arabic script, form controls, process flows, and numerals.

On coarse, small-screen, or Save-Data devices (`KA.light`), a shared lightweight rendering mode
protects mobile scrolling performance:
- Caps canvas DPR at 1 and rendering at 30 FPS.
- Pauses decorative canvas drawing (`#field`, project vizzes, `#flow`, footer skyline) during active page scroll.
- Calms canvas particle counts and background splat calculations.
- Suppresses during-scroll blur and dynamic masks, and switches carousel GPU layer promotion to active gestures only (`will-change: auto` at rest).
- Routes resize events through debounced mostly width-based handling (`KA.onResize`) to ignore vertical address-bar jitter on iOS and Android.

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

## Deploying

One host: **GitHub Pages**, deploying the `main` branch from the repository root. The repo has to stay
public for the free plan, and `github.io` is the hostname that keeps resolving for the audience this
page is written for — Cloudflare's free hostnames (`*.pages.dev`, `*.workers.dev`) are filtered in Iran
as a class, and putting the site behind Cloudflare's edge inherits the IP-range blocking that happens
whenever filtering tightens. During a total blackout no hosting choice helps, so the résumé PDF is sent
directly rather than linked.

- Push to `main` and Pages publishes in under a minute; a push that fails to trigger a build silently
  keeps serving the previous version, so check *Settings → Pages* after anything structural.
- `.nojekyll` is what makes Pages serve this folder verbatim — no Jekyll pass, so nothing rewrites the
  front matter-free HTML or the underscore-free asset paths. Keep the file.
- The `<link rel="canonical">` and `og:url` in the head name the live host. If a custom domain is ever
  attached, those two lines are the edit, plus the domain field in Settings → Pages.
- GitHub Pages does not read `_headers`, so there is no cache-header lever here — and none is needed:
  the site is one main HTML document, a small standalone `404.html`, self-hosted font files,
  a résumé PDF and a self-hosted social share image (`assets/og.png`).
- A domain can be bought anywhere and simply pointed at `KiarashAkbari.github.io` with an apex
  `ALIAS`/flattened record and a `www CNAME`; GitHub issues the certificate once the domain is saved in
  Settings → Pages. Keeping Cloudflare out of the request path (DNS only, or another registrar
  entirely) is deliberate.

No build step, no config files for the host, nothing to install: the repository *is* the deployment.

## Notes

- Chrome's Auto Dark Mode skips the page because `<meta name="color-scheme" content="light dark">`
  and `color-scheme` on the root declare that dark is already handled natively.
- No third-party resource is *loaded* (all font faces are self-hosted), so ad/privacy blockers have no
  request to cancel and no class or id name collides with common cosmetic-filter selectors. The single
  outbound request is the form submission above, fired on submit, never on page load.
- Persian text renders in Lahzeh first, with Vazirmatn retained as the Arabic-script fallback;
  Latin text remains Host Grotesk. The owner-supplied Lahzeh files are not open-source assets —
  see `fonts/Lahzeh-NOTICE.md` before reusing, converting, or subsetting them.
- Social previews use the self-hosted 1200 × 630 `assets/og.png`; unknown paths receive the branded
  standalone `404.html` from GitHub Pages.
- No build step. Open `index.html` or serve the folder: `python3 -m http.server 8000`.
