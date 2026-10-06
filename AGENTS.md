# Repository Guidelines

## Project Overview
The static landing page for **Subclave**, a local-first Tauri 2 desktop password manager with S3/WebDAV sync and a browser extension. It is served at `https://subclave.rendy.dev/` from Cloudflare Workers static assets: Workers Builds runs `npx wrangler deploy` on every push to `main`, and the whole repo root is the assets directory. The app itself lives in the sibling repo `../subclave` (GitHub `rendyuwu/subclave`). The page's structure and behavior follow its sibling `../tervia-web`; the visual identity is Subclave's own.

There is no framework, no build step, no `package.json` and no dependencies. The site is three hand-written files plus `assets/`.

## Architecture & Data Flow
- `index.html`: a single page. It has a header/nav, a hero with an inline SVG "route", then `.parts`, which holds one `<section class="part" id="…">` per area: `vault`, `browser`, `sync`, `import`, `threats`, `download`. After those come the footer and `<dialog class="viewer">`.
- Vault shows `main.webp` full width under its claim (`.shot-wide`). Browser puts `browser.webp` beside its facts (`.cols.with-shot`, 5fr to 7fr) and the fill chain (`.hops`) below, as four steps in a row that stack at 1100px and below. `.shot img` is never wider than the capture, so the small browser crop is not upscaled.
- Each `.part` is two columns: `.part-head` (the `h2`, sticky on wide screens, and for the four data areas a `.where` line saying where that area's secrets are kept) and `.part-body` (the claim, facts and any figure). `threats` and `download` have no `.where` because they are not a place secrets live.
- There is an inline `<script>` in `<head>` that sets `data-theme="light"` on `<html>` before first paint if `localStorage["subclave-theme"] === "light"`. `main.js` loads with `defer`.
- `main.js` has three independent top-level blocks and no modules:
  1. **Downloads.** `detectOs()` returns `mac`, `linux`, `windows` or `null` (mobile and iPad give `null`). It relabels the hero `[data-primary]` button and adds `.is-yours` to `[data-os="…"]`. It then fetches `api.github.com/repos/rendyuwu/subclave/releases/latest`, matches `release.assets` against the `FILES` regexes and rewrites each `[data-file="<key>"]` link's `href`/`title`/`data-size`. It also fills `[data-version]` with the tag and `[data-primary-file]` with the file name and size. If the fetch fails, it does nothing (`.catch(() => {})`).
  2. **Theme toggle.** `[data-theme-toggle]` starts `hidden` and JS unhides it. The page is dark by default. Choosing light stores `subclave-theme=light`, and switching back to dark removes the key. `aria-pressed` means "dark is on".
  3. **Screenshot viewer.** A click on `.zoom` (an `<a>` pointing at the image file) opens `dialog.viewer` via `showModal()`. Clicking anywhere or pressing Esc closes it.
- **Progressive enhancement is the core pattern.** Every link works without JS: download links default to `https://github.com/rendyuwu/subclave/releases/latest`, and `.zoom` links open the raw image. Keep that property when adding behavior.

## Key Directories
- `assets/`: `logo.svg` (favicon plus header logo, from `src-tauri/icons/subclave-mark.svg` in the app repo), `icon.png` (750×750, apple-touch and `og:image`, the app repo's `subclave.png`), `fonts/atkinson-next-latin.woff2` and `fonts/atkinson-mono-latin.woff2` (Atkinson Hyperlegible Next and Mono, variable weight 200 to 800, Latin subset from Fontsource, both preloaded).
- `assets/shots/`: `main.webp` (in Vault, the top 1020×400 of the main window, cropped so the empty lower half of a one-entry vault does not show) and `browser.webp` (in Browser, GitHub's sign-in form cropped to 432×345 around the inline icons). Both are WebP at quality 90.
- `.omp/dev/*.png`: the full PNG captures (`main.png`, `browser.png`) and the crops shipped as WebP (`main-crop.png` is box `0,0,1020,400` of `main.png`, `browser-crop.png` is box `443,40,875,385` of `browser.png`). Gitignored and not served.
- `.playwright-cli/`: browser scratch. Gitignored, not tests.

## Development Commands
There is nothing to build, lint or install. Serve from the repo root, because the brand link uses the root-relative `href="/"`:
```bash
python3 -m http.server 4173   # then open http://127.0.0.1:4173/
```
The GitHub API is unauthenticated (60 requests/hour per IP). If it gets rate-limited, download links quietly stay on `releases/latest`, and that is expected.

## Code Conventions & Common Patterns
- **Formatting:** Prettier-like style with no config in the repo, so match it by hand: 2-space indent, double quotes, semicolons, trailing commas, lines of roughly 120 characters, self-closing void tags in HTML (`<meta … />`).
- **JS:** vanilla ES2020+ (`?.`, `for…of`, arrow functions, `const`). Elements are found with eager top-level `querySelector`s on `data-*` hooks. Use `data-*` attributes as the JS hooks, not styling classes (`.zoom` and `.viewer` are the only class hooks). Set untrusted strings with `textContent` or `replaceChildren`, never through `innerHTML`. Wrap `localStorage` in `try {} catch {}`.
- **Comments** explain intent and browser quirks in full sentences. Keep that style.
- **CSS theming:** every themed colour token in `:root` is `light-dark(light, dark)`. The default `color-scheme: dark` is switched by `:root[data-theme="light"]`. **A new themed token must also be added to the `@supports not (color: light-dark(…))` fallback block** (dark values). `--gold`, `--navy`, `--term` and `--on-term` are the same in both themes.
- **Brand colours:** navy `#1d2b53` to `#0d1530` and gold `#f5b83d` come from the app icon. The hero `.route` tile stays navy in both themes, like the icon, because brand gold on the light paper is too faint. Gold used as text goes through `--gold-ink` (`#8a5a00` in light), never `--gold`.
- **Fonts:** `--sans` (Atkinson Next) for everything, `--mono` (Atkinson Mono) for code, hashes, file names and the `.where` lines. The typeface was drawn so `l`, `I`, `1`, `0` and `O` cannot be confused, which is why a password manager uses it; do not swap it for a generic sans.
- **CSS layout:** plain descriptive class names (no BEM, no utilities), sections marked with `/* Header */`-style comments, `clamp()` for fluid sizing, two breakpoints (`max-width: 960px`, `640px`), plus one at `1100px` that stacks the `.hops` chain, whose four columns get too narrow for `subclave-proxy` below it.
- **Hero route:** the S path is the app icon's path (`M327.4 154 A76 76 0 1 0 256 256 A76 76 0 1 1 184.6 358`, two r=76 arcs, caps 20° off horizontal) rescaled to r=17 with the centre at (36, 50). Each label block sits beside the point of the S it names: the top cap, the centre and the bottom cap. The hash in the middle block is the second object name in the Sync listing.
- **Motion:** all animation and smooth scrolling sit inside `@media (prefers-reduced-motion: no-preference)`. The only animation is the hero S drawing itself and its labels fading in.
- **CSS reads JS state:** `.platform.is-yours` shows the "Your system" label, and `.files a::after { content: attr(data-size) }` shows the file size.
- **Accessibility:** every section has `aria-labelledby` pointing at its `h2` id (`<id>-h`). Decorative SVGs get `aria-hidden`, informative ones get `role="img"` plus `<title>`. Screen-reader text uses `.sr`. Scrollable `pre` blocks get `tabindex="0"`. Images need `alt` plus `width`/`height`, and below-the-fold images get `loading="lazy"`.
- **Copy:** English, plain and factual, with concrete claims and no hype, and no em-dashes (the app repo's rule). Every number on the page (Argon2id 64 MiB and 3 passes, 10 history versions, 30 s reveal and clipboard clear, 10 min auto-lock, 8 to 128 generator length, 90-day tombstones, Firefox 115, 6-hour update check) is a constant or default in `../subclave`; check it there before changing it. Limitations are stated openly (unsigned builds, extension loaded by hand, the threat model's "does not protect against" list).

## Important Files
- `index.html`: all markup, meta/OG tags and the inline theme bootstrap script.
- `main.js`: `FILES` (asset key → filename regex), `DOWNLOAD_BASE`, `PLATFORMS` (OS → label and default file), `formatSize`.
- `styles.css`: design tokens (`:root`), the dark fallback and all layout.
- `.assetsignore`: gitignore-style list of repo files that must not be deployed (`.git`, `.omp`, `AGENTS.md`, `README.md`, `wrangler.jsonc`, …). **Any new non-site file at the repo root must be added here, or it becomes publicly reachable.**

## Couplings That Break Silently
- `FILES` keys must match the `data-file` values (`mac-arm`, `mac-intel`, `appimage`, `deb`, `rpm`, `exe`, `chrome`, `firefox`), and each regex must match the release asset names produced by `../subclave` CI (`_aarch64.dmg`, `_x64.dmg`, `_amd64.AppImage`, `_amd64.deb`, `.x86_64.rpm`, `_x64-setup.exe`, `subclave-chrome.zip`, `subclave-firefox.zip`). Adding a platform or file means updating `FILES`, the `data-file` link and possibly `PLATFORMS`.
- `data-os` values must match the `detectOs()` return values.
- `main.js` assumes `[data-primary]`, `[data-primary-file]`, `[data-theme-toggle]` and `.viewer` exist. If one is missing, it throws and stops every block that runs after it.
- Do not hardcode versions or download URLs. `[data-version]` and the links are filled at runtime.
- The extension install steps summarise the app repo's README section "Install the browser extension", which the page links to. Change both together.
- To add or replace a screenshot, put the PNG in `.omp/dev/`, ship it as WebP in `assets/shots/`, and set the `<img>` `width`/`height` to its size. The demo vault holds one GitHub entry with the username `rendy`, which the hero route labels repeat; change both together.

## Runtime/Tooling Preferences
- The only runtime is a browser. Don't add npm, a bundler, a framework, a CSS preprocessor or a CDN dependency. The only network request is the GitHub API call.
- Keep the file count as it is: behavior goes in `main.js`, styles in `styles.css`.

## Testing & QA
There are no automated tests, CI or coverage, and none are expected for this small page. Check changes by hand in a browser served as above:
- Dark and light theme both render, and the choice survives a reload.
- Download links resolve to real assets, and the `.is-yours` card and hero label match your OS.
- The hero S draws once on load, and not at all with reduced motion.
- A screenshot opens and closes in the viewer.
- Layout holds at roughly 390px, 640px and 960px widths.
- With JS disabled, the links still work.
