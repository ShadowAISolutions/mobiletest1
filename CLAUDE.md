# Claude Code Instructions

## Deployment Flow
- Never push directly to `main`
- Push to `claude/*` branches only
- `.github/workflows/auto-merge-claude.yml` handles everything automatically:
  1. Merges the claude branch into main
  2. Deletes the claude branch
  3. Deploys to GitHub Pages
- The "Create a pull request" message in push output is just GitHub boilerplate — ignore it, the workflow handles merging automatically

## Version Bumping
- **Every commit that modifies a GAS project's `.gs` file MUST also increment its `VERSION` variable by 0.01**
- The `VERSION` variable is near the top of each `.gs` file (look for `var VERSION = "..."`)
- Format includes a `g` suffix: e.g. `"01.13g"`
- Example: if VERSION is `"01.13g"`, change it to `"01.14g"`
- Do NOT bump VERSION if the commit doesn't touch the `.gs` file

### GAS Projects
Each GAS project has a code file and a corresponding embedding page. Register them in the table below as you add them.

| Project | Code File | Embedding Page |
|---------|-----------|----------------|
| *(Project name)* | `googleAppsScripts/<Project Name>/<CodeFile>.gs` | `httpsdocs/<page-name>.html` |

## Build Version (Auto-Refresh for embedding pages)
- **Every commit that modifies an embedding HTML page MUST increment its `build-version` meta tag by 0.01**
- Look for `<meta name="build-version" content="...">` in the `<head>`
- Format includes a `w` suffix: e.g. `"01.11w"`
- Example: if build-version is `"01.11w"`, change it to `"01.12w"`
- Each embedding page polls `version.txt` every 10 seconds — when the deployed version differs from the loaded version, it auto-reloads

### Auto-Refresh via version.txt Polling
- **All embedding pages must use the `version.txt` polling method** — do NOT poll the page's own HTML
- **Version file naming**: the version file must be named `<page-name>.version.txt`, matching the HTML file it tracks (e.g. `index.html` → `index.version.txt`, `dashboard.html` → `dashboard.version.txt`). The `.version.txt` double extension ensures the version file sorts **after** the `.html` file alphabetically
- Each version file holds only the current build-version string (e.g. `01.08w`)
- **When bumping `build-version` in an HTML page, also update the corresponding `<page-name>.version.txt`** to the same value
- The polling logic fetches the version file (~7 bytes) instead of the full HTML page, reducing bandwidth per poll from kilobytes to bytes
- URL resolution: derive the version file URL relative to the current page's directory, using the page's own filename:
  ```javascript
  var basePath = window.location.href.split('?')[0];
  var pageName = basePath.substring(basePath.lastIndexOf('/') + 1).replace('.html', '');
  if (!pageName) pageName = 'index';
  var versionUrl = basePath.substring(0, basePath.lastIndexOf('/') + 1) + pageName + '.version.txt';
  ```
- **The `if (!pageName)` fallback is critical** — when a page is accessed via a directory URL (e.g. `https://example.github.io/myapp/` instead of `.../myapp/index.html`), `pageName` resolves to an empty string. Without the fallback, the poll fetches `.version.txt` (wrong file), gets a 404 whose body doesn't match the build-version, and triggers an infinite reload loop
- Cache-bust with a query param: `fetch(versionUrl + '?_cb=' + Date.now(), { cache: 'no-store' })`
- Compare the trimmed response text against the page's `<meta name="build-version">` content
- The template in `autoUpdateTemplateFiles/AutoUpdateOnlyHtmlTemplate.html` already implements this pattern — use it as a starting point for new projects
- **The template's build-version must always remain at `01.01w`** — never bump the template's version, even when editing the template itself. The template is a starting point, not a deployed page. Only bump versions on actual embedding pages copied from it

### New Embedding Page Setup Checklist
When creating a **new** HTML embedding page, follow every step below:

1. **Copy the template** — start from `autoUpdateTemplateFiles/AutoUpdateOnlyHtmlTemplate.html`, which already includes:
   - `<meta name="build-version" content="...">` in the `<head>`
   - Version file polling logic (10-second interval)
   - Version indicator pill (bottom-right corner)
   - Green "Website Ready" splash overlay + sound playback
   - AudioContext handling and screen wake lock
2. **Choose the directory** — create a new subdirectory under `httpsdocs/` named after the project (e.g. `httpsdocs/my-project/`)
3. **Create the version file** — place a `<page-name>.version.txt` file in the **same directory** as the HTML page (e.g. `index.version.txt` for `index.html`), containing only the initial build-version string (e.g. `01.00w`)
4. **Update the polling URL in the template** — ensure the JS version-file URL derivation matches the HTML filename (the template defaults to deriving it from the page's own filename)
5. **Create `sounds/` directory** — copy the `sounds/` folder (containing `Website_Ready_Voice_1.mp3`) into the new page's directory so the splash sound works
6. **Set the initial build-version** — in the HTML `<head>`, set `<meta name="build-version" content="01.00w">` and match it in `<page-name>.version.txt`
7. **Update the page title** — replace `YOUR_PROJECT_TITLE` in `<title>` with the actual project name
8. **Register in GAS Projects table** — if this page embeds a GAS iframe, add a row to the GAS Projects table in the Version Bumping section above
9. **Add developer branding** — ensure `<!-- Developed by: ShadowAISolutions -->` is the last line of the HTML file

### Directory Structure (per embedding page)
```
httpsdocs/
├── <page-name>/
│   ├── index.html               # The embedding page (from template)
│   ├── index.version.txt        # Tracks index.html build-version (e.g. "01.00w")
│   └── sounds/
│       └── Website_Ready_Voice_1.mp3
```
For pages that live directly in `httpsdocs/` (not in a subdirectory), the version file and `sounds/` folder sit alongside the HTML file (e.g. `httpsdocs/index.html` + `httpsdocs/index.version.txt`).

## Commit Message Naming
- **Every commit message MUST start with the version number(s) being updated**
- Both version types use the `v` prefix (meaning "version") — the suffix indicates the type: `g` = Google Apps Script, `w` = website
- If a `.gs` file was updated: prefix with `v{VERSION}` (e.g. `v01.19g`)
- If an embedding HTML page was updated: prefix with `v{BUILD_VERSION}` (e.g. `v01.12w`)
- If both were updated in the same commit: include both (e.g. `v01.19g v01.12w`)
- If neither was updated: no version prefix needed
- Example: `v01.19g Fix sign-in popup to auto-close after authentication`
- Example: `v01.19g v01.12w Add auth wall with build version bump`

## GAS Code Constraints
- **All GAS `.gs` code must be valid Google Apps Script syntax** — test mentally that strings, escapes, and quotes parse correctly before committing
- Avoid deeply nested quote escaping in HTML strings built inside `.gs` files. Instead, store values in global JS variables and reference them in `onclick` handlers (e.g. `_signInUrl` pattern)
- **`readPushedVersionFromCache()` must NOT delete the cache entry** — it must return the value without calling `cache.remove()`. Deleting it causes only the first polling client to see the update; all others miss the "Code Ready" blue splash reload. The cache has a 1-hour TTL and expires naturally.
- The GAS auto-update "Code Ready" splash flow works as follows:
  1. GitHub Actions workflow calls `doPost(?action=deploy)` on the **old** deployed GAS
  2. `pullAndDeployFromGitHub()` fetches new code from GitHub, updates the script, creates a new version, updates the deployment
  3. It writes the new version string to `CacheService.getScriptCache()` with key `"pushed_version"`
  4. Client-side JS polls `readPushedVersionFromCache()` every 15 seconds
  5. If the returned version differs from the version displayed in `#gv`, it sends a `gas-reload` postMessage to the parent embedding page
  6. The embedding page receives the message, sets session storage flags, reloads, and shows the blue "Code Ready" splash

## Race Conditions — Config vs. Data Fetch
- **Never fire `saveConfig` and a dependent data-fetch (`getFormData`) in parallel** — the data-fetch may read stale config values from the sheet
- When the client switches a config value (e.g. year) and needs fresh data for that value, **pass the value directly as a parameter** to the server function (e.g. `getFormData(_token, year)`) rather than relying on `saveConfig` completing first
- Server functions that read config should accept optional override parameters (e.g. `opt_yearOverride`) so the client can bypass the saved config when needed
- This pattern avoids race conditions without needing to chain callbacks (which adds latency)

## API Call Optimization (Scaling Goal)
- **Minimize Google API calls** in every GAS function — the app is designed to scale to many users
- **Cache `getUserInfo` results** in `CacheService` (keyed by token suffix) for 5 minutes to avoid hitting the OAuth userinfo endpoint on every `google.script.run` call
- **Cache `checkSpreadsheetAccess` results** in `CacheService` (keyed by email) for 10 minutes to avoid listing editors/viewers on every call
- **Open `SpreadsheetApp.openById()` once per function** — pass the `ss` object to `checkSpreadsheetAccess(email, opt_ss)` instead of opening the spreadsheet twice
- When adding new server-side functions, always consider: can this result be cached? Can I reuse an already-opened spreadsheet object? Avoid redundant `UrlFetchApp` or `SpreadsheetApp` calls
- Cache TTLs are intentionally short (5–10 min) so permission changes and token revocations take effect quickly

## UI Dialogs — No Browser Defaults
- **Never use `alert()`, `confirm()`, or `prompt()`** — all confirmation dialogs, alerts, and input prompts must use custom styled HTML/CSS modals
- This applies to both GAS `.gs` code and parent embedding pages (`.html`)
- Use overlay + modal patterns consistent with the existing sheet/modal styles in the codebase

## Phantom Edit (Timestamp Alignment)
- When the user asks for a **phantom edit** or **phantom update**, touch every file in the repo with a no-op change so all files share the same commit timestamp on GitHub
- **Skip all version bumps** — do NOT increment `build-version` in HTML pages or `VERSION` in `.gs` files
- For text files: add a trailing newline
- For binary files (e.g. `.mp3`): append a null byte
- Commit message: `Phantom edit to align all file timestamps on GitHub` (no version prefix)

## Execution Style
- For clear, straightforward requests: **just do it** — make the changes, commit, and push without asking for plan approval
- Only ask clarifying questions when the request is genuinely ambiguous or has multiple valid interpretations
- Do not use formal plan-mode approval workflows for routine tasks (version bumps, file moves, feature additions, bug fixes, etc.)

## AudioContext & Browser Autoplay Policy
- **AudioContext starts as `'suspended'`** on every page load — browsers require a user gesture (click/touch) before allowing audio playback
- **`resume()` without a gesture** generally stays pending or silently fails. It does NOT reject — the promise just never resolves, which causes dangling `.then()` callbacks that fire unexpectedly when the user eventually clicks
- **Never schedule `decodeAudioData` + `source.start(0)` while context is suspended** — the audio gets queued and plays the moment the context resumes (on user click), causing surprise delayed sound. Instead, gate playback behind `if (ctx.state !== 'running') return`
- **JS-triggered `window.location.reload()` vs manual F5 refresh** behave differently: JS reload carries forward the gesture allowance (AudioContext can auto-resume), F5 does NOT. So auto-refresh reloads can play sound, but manual refreshes cannot
- **`onstatechange` fires before DOM is ready**: when `resume()` is called early in the script, `onstatechange` may fire before the `#audio-status` element exists — the `updateAudioStatus()` call bails silently and never retries. Fix: save the resume promise and chain `updateAudioStatus` onto it after the element is created
- **Use `sessionStorage` (not `localStorage`) for audio state flags** — `sessionStorage` is per-tab, so a flag set in one tab doesn't leak into new tabs that have no gesture context. `localStorage` would cause false-positive "sound ready" icons in fresh tabs
- **The `audio-unlocked` sessionStorage flag** remembers that audio was successfully unlocked in this tab. On F5 refresh, the AudioContext is suspended but the flag tells the icon to show "ready" instead of "muted" — because a click will instantly restore it. Without this flag, the icon flashes "muted" on every refresh even though audio works fine
- **Chrome "Duplicate Tab" copies `sessionStorage`** — including the `audio-unlocked` flag — into the new tab, but the new tab has no gesture context, so the AudioContext is suspended. The stale flag causes the icon to falsely show "sound ready." Fix: use `performance.getEntriesByType('navigation')` to detect the navigation type; if it's anything other than `'reload'` (e.g. `'navigate'` for duplicate tab, back/forward, or new navigation), clear the `audio-unlocked` flag before creating the AudioContext. This must run **before** `new AudioContext()` so that `updateAudioStatus()` sees the correct flag state from the start

## Google Sign-In (GIS) for GAS Embedded Apps
When a GAS app embedded in a GitHub Pages iframe needs Google sign-in (e.g. to restrict access to authorized users), the sign-in **must run from the parent embedding page**, not from inside the GAS iframe.

### Why
- GAS iframes are served from dynamic `*.googleusercontent.com` subdomains with long hash-based hostnames that change when the deployment changes — they can't be reliably registered as OAuth origins
- Google OAuth requires the JavaScript origin to be registered in Cloud Console
- The parent GitHub Pages domain (e.g. `<your-org>.github.io`) is a stable origin that can be registered once

### Architecture
1. **GAS iframe** detects auth is needed → sends a `gas-needs-auth` postMessage to the parent (with `authStatus` and `email` fields)
2. **Parent embedding page** receives the message → shows an auth wall overlay → loads GIS and triggers sign-in popup
3. After successful sign-in → parent hides the auth wall → reloads just the iframe (`iframe.src = iframe.src`)
4. GIS code (Google Identity Services library) lives **only** in the parent HTML, never in the `.gs` file

### OAuth Setup (Google Cloud Console)
- **OAuth Client ID**: Create or use an existing OAuth 2.0 Client ID from your Google Cloud project (format: `<client-id>.apps.googleusercontent.com`)
- **Authorized JavaScript origins** must include your GitHub Pages domain (e.g. `https://<your-org>.github.io`) and any custom domains pointing to it
- To configure: Google Cloud Console → APIs & Services → Credentials → OAuth 2.0 Client IDs → edit the client → add the origin
- If you add new embedding domains (e.g. a custom domain), add those origins too

### Key postMessage Types for Auth
| Message Type | Direction | Purpose |
|---|---|---|
| `gas-needs-auth` | GAS iframe → parent | Tells parent to show sign-in wall (includes `authStatus`, `email`) |
| `gas-auth-complete` | GAS iframe → parent | Tells parent auth succeeded (hides wall, reloads iframe) |

## Developer Branding
- **Every code file** in this repo must have a comment at the very bottom: `Developed by: ShadowAISolutions`
- Use the appropriate comment syntax for each file type:
  - HTML: `<!-- Developed by: ShadowAISolutions -->`
  - JavaScript / GAS (.gs): `// Developed by: ShadowAISolutions`
  - YAML: `# Developed by: ShadowAISolutions`
  - CSS: `/* Developed by: ShadowAISolutions */`
  - Markdown: plain text at the very bottom
- When creating new code files, always add this comment as the last line
- This section must remain the **last section** in CLAUDE.md — do not add new sections below it

Developed by: ShadowAISolutions




