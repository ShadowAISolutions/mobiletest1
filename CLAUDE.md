# Claude Code Instructions

## Session Start Checklist (MANDATORY — RUN FIRST)
> **MANDATORY FIRST ACTION — DO NOT SKIP**
> Complete ALL checks below and commit any fixes BEFORE reading or acting on the user's request.
> If checklist items produce changes, commit them as a separate commit with message:
> `Session start: fix template drift`
> **The user's task is NOT urgent enough to skip this. Do it first. Every time.**

### Always Run (every repo, every session — NEVER skip)
These rules apply universally — they are **NOT** skipped by the template repo short-circuit.

**Branch hygiene** — run `git remote set-head origin main` to ensure `origin/HEAD` points to `main`. If a local `master` branch exists and points to the same commit as `origin/main`, delete it with `git branch -D master`. This prevents the auto-merge workflow from failing with exit code 128 due to branch misconfiguration.

**Deployment Flow**
- Never push directly to `main`
- Push to `claude/*` branches only
- `.github/workflows/auto-merge-claude.yml` handles everything automatically:
  1. Merges the claude branch into main
  2. Deletes the claude branch
  3. Deploys to GitHub Pages
- The "Create a pull request" message in push output is just GitHub boilerplate — ignore it, the workflow handles merging automatically
- **Push only once per branch** — do NOT push multiple times to the same `claude/*` branch in a single session. The workflow uses a shared concurrency group (`"pages"`) with `cancel-in-progress: false`, so each push queues a separate workflow run. If an earlier run merges and deletes the branch, subsequent queued runs fail with exit code 128 because the branch no longer exists. **This includes sequential user requests** — if the user asks for task A and then task B in the same session, commit both locally and push once after all work is done. Do NOT push after task A and then push again after task B. The only exception is if a re-push is needed to recover from a failed workflow (e.g. the branch still exists on the remote but the merge didn't happen)

**Template repo short-circuit** — run `git remote -v` and extract the repo name. If it is `autoupdatehtmltemplate`, skip the Template Drift Checks below and proceed directly to the user's request.

**Initialized repo short-circuit** — check if `README.md` contains the placeholder text `You are currently using the **`. If it does NOT, the repo has already been initialized — skip the Template Drift Checks below and proceed directly to the user's request. If it DOES, the repo is a fresh fork that needs initialization — continue to the Template Drift Checks.

### Template Drift Checks (forks/clones only)
These checks catch template drift that accumulates when the repo is cloned/forked into a new name. They do **not** apply to the template repo itself.

1. **Org name auto-detect** — run `git remote -v` and extract the org/owner from the remote URL (e.g. `github.com/NewOrg/myrepo` → `NewOrg`). Compare it to the `YOUR_ORG_NAME` value in the Template Variables table. If they differ, update the table value and propagate to every file in the "Where it appears" column by **finding and replacing** the old org name (`ShadowAISolutions`) with the new org name in all occurrences (URLs, text, branding). Also update `DEVELOPER_NAME` to match the new org name (unless the user has explicitly set `DEVELOPER_NAME` to a different value)
2. **Repo name auto-detect** — compare the actual repo name (from the same remote URL) to the `YOUR_REPO_NAME` value in the Template Variables table. If they differ, update the table value and propagate it to every file in the "Where it appears" column by **finding and replacing** the old repo name (`autoupdatehtmltemplate`) with the new repo name in all occurrences (URLs, text, structure references)
3. **Relative links (already dynamic — do NOT modify)** — certain markdown files use relative paths that automatically resolve to the correct repo via GitHub's blob-view URL structure (see *Relative Path Resolution on GitHub* reference section for how this works). These links work on any fork/clone without initialization and must **never** be converted to absolute URLs or modified during drift checks. Files with relative links:
   - `SECURITY.md` — private security advisory link (`../../security/advisories/new`)
   - `repository-information/SUPPORT.md` — issue creation links (`../../../issues/new`)
   - `README.md` — repository settings links in "Initialize This Template" section (`../../settings/pages`, `../../settings/environments`). These are removed by step #6 but must not be converted to absolute URLs if encountered before removal
4. **Absolute URL propagation** — some files contain absolute URLs with the org and repo name that cannot use relative paths (YAML metadata fields, GitHub Pages URLs on a different domain, Mermaid diagram text). After steps 1–2, find and replace the template repo's values (`ShadowAISolutions`/`autoupdatehtmltemplate`) with the fork's actual values in these files. **For each file, verify every URL listed below — not just the first one you find:**
   - `.github/ISSUE_TEMPLATE/config.yml` — YAML `url` fields require absolute URLs:
     - `url: https://github.com/ShadowAISolutions/autoupdatehtmltemplate/blob/main/repository-information/SUPPORT.md` (support link)
     - `url: https://github.com/ShadowAISolutions/autoupdatehtmltemplate/security/advisories/new` (security advisory link)
   - `CITATION.cff` — citation metadata (not rendered markdown):
     - `repository-code:` URL (`https://github.com/ShadowAISolutions/autoupdatehtmltemplate`)
     - `url:` field (`https://ShadowAISolutions.github.io/autoupdatehtmltemplate`)
   - `repository-information/STATUS.md` — placeholder in Hosted Pages table (`github.io` is a different domain, can't use relative paths):
     - If the Live URL column still contains `*(deploy to activate)*`, replace it with `[View](https://YOUR_ORG_NAME.github.io/YOUR_REPO_NAME/)` (resolved values)
   - `repository-information/ARCHITECTURE.md` — Mermaid diagram text (not a clickable link):
     - `LIVE["Live Site\nShadowAISolutions.github.io/autoupdatehtmltemplate"]`
   - `README.md` — live site link and source repo URL:
     - Live site `[View](https://...)` or `[...github.io/...](https://...)` link
     - Source repo URL in "Copy This Repository" section (gets removed in step #6, but update if still present)
   When replacing, change **only the org and repo name portions** of each URL — preserve the rest of the URL path and structure intact (e.g. `/security/advisories/new` stays the same, only `ShadowAISolutions/autoupdatehtmltemplate` changes to `NewOrg/newrepo`)
   **Verification:** after replacements, run `grep -r 'ShadowAISolutions\|autoupdatehtmltemplate' --include='*.md' --include='*.yml' --include='*.cff' --include='*.html'` (excluding CLAUDE.md) and confirm zero hits remain outside of `Developed by:` branding lines and provenance markers
5. **README live site link** — check if `README.md` still contains the placeholder text (`You are currently using the **YOUR_REPO_NAME**...`). If so, replace it with: `**Live site:** [YOUR_ORG_NAME.github.io/YOUR_REPO_NAME](https://YOUR_ORG_NAME.github.io/YOUR_REPO_NAME)` (resolved values)
6. **Remove "Copy This Repository" and "Initialize This Template" sections** — if `README.md` contains the `## Copy This Repository` or `## Initialize This Template` sections, delete each entirely (from the `##` heading through to the line immediately before the next `##` heading). These sections are only useful on the template repo itself; forks/clones should not keep them
7. **Unresolved placeholders** — scan for any literal `YOUR_ORG_NAME`, `YOUR_REPO_NAME`, `YOUR_PROJECT_TITLE`, or `DEVELOPER_NAME` strings in code files (not CLAUDE.md) and replace them with resolved values
8. **Variable propagation** — if any value in the Template Variables table was changed (in this or a prior session), verify the new value has been propagated to every file listed in the "Where it appears" column
9. **Confirm completion** — after all checks pass, briefly state to the user: "Session start checklist complete — no issues found" (or list what was fixed). Then proceed to their request

---
> **--- END OF SESSION START CHECKLIST ---**
---

## Initialize Command
If the user's prompt is just **"initialize"** (after the Session Start Checklist has completed):
1. **Verify placeholders are resolved** — confirm that `repository-information/STATUS.md` no longer contains `*(deploy to activate)*` (drift check step #4 should have replaced it). If it's still there, replace it now with `[View](https://YOUR_ORG_NAME.github.io/YOUR_REPO_NAME/)` (resolved values)
2. Update the `Last updated:` timestamp in `README.md` to the real current time
3. Commit with message `Initialize deployment`
4. Push to the `claude/*` branch

This triggers the auto-merge workflow, which merges into `main` and deploys to GitHub Pages — populating the live site for the first time. No other changes are needed.

---
> **--- END OF INITIALIZE COMMAND ---**
---

## Template Repo Guard
> When the actual repo name (from `git remote -v`) is `autoupdatehtmltemplate` (i.e. this is the template repo itself, not a fork/clone):
> - **Session Start Checklist template drift checks are skipped** — the "Template repo short-circuit" in the Always Run section skips the entire numbered checklist. The "Always Run" section (branch hygiene and deployment flow) still applies every session
> - **All version bumps are skipped** — Pre-Commit Checklist items #1 (`.gs` version bump), #2 (HTML build-version), #3 (version.txt sync), #5 (STATUS.md), **#7 (CHANGELOG.md)**, and #9 (version prefix in commit message) are all skipped unless the user explicitly requests them. **DO NOT add CHANGELOG entries on the template repo** — the CHANGELOG must stay clean with `*(No changes yet)*` so that forks start with a blank history
> - **GitHub Pages deployment is skipped** — the workflow's `deploy` job checks `github.event.repository.name != 'autoupdatehtmltemplate'` and won't run on the template repo
> - **`YOUR_ORG_NAME` and `YOUR_REPO_NAME` are frozen as placeholders** — in the Template Variables table, these values must stay as `YourOrgName` and `YourRepoName` (generic placeholders). Do NOT update them to match the actual org/repo (`ShadowAISolutions`/`autoupdatehtmltemplate`). The code files throughout the repo use the real `ShadowAISolutions/autoupdatehtmltemplate` values so that links are functional. On forks, the Session Start drift checks detect the mismatch between the placeholder table values and the actual `git remote -v` values, then find and replace the template repo's real values (`ShadowAISolutions`/`autoupdatehtmltemplate`) in the listed files with the fork's actual org/repo
> - Pre-Commit items #4, #6, #8, #10, #11, #12 still apply normally

---
> **--- END OF TEMPLATE REPO GUARD ---**
---

## Pre-Commit Checklist
**Before every commit, verify ALL of the following:**

1. **Version bump (.gs)** — if any `.gs` file was modified, increment its `VERSION` variable by 0.01 (e.g. `"01.13g"` → `"01.14g"`)
2. **Version bump (HTML)** — if any embedding HTML page in `live-site-pages/` was modified, increment its `<meta name="build-version">` by 0.01 (e.g. `"01.01w"` → `"01.02w"`). **Skip if Template Repo Guard applies (see above)**
3. **Version.txt sync** — if a `build-version` was bumped, update the corresponding `<page-name>.version.txt` to the same value. **Skip if Template Repo Guard applies**
4. **Template version freeze** — never bump `live-site-templates/AutoUpdateOnlyHtmlTemplate.html` — its version must always stay at `01.00w`
5. **STATUS.md** — if any version was bumped, update the matching version in `repository-information/STATUS.md`. **Skip if Template Repo Guard applies**
6. **ARCHITECTURE.md** — if any version was bumped or the project structure changed, update the diagram in `repository-information/ARCHITECTURE.md`
7. **CHANGELOG.md** — every user-facing change must have an entry under `## [Unreleased]` in `repository-information/CHANGELOG.md`. Each entry must include an EST timestamp down to the second (format: `` `YYYY-MM-DD HH:MM:SS EST` — Description``). The `[Unreleased]` section header must also show the date/time of the most recent entry. **Timestamps must be real** — run `TZ=America/New_York date '+%Y-%m-%d %H:%M:%S EST'` to get the actual current time; never fabricate or increment timestamps. **Skip if Template Repo Guard applies (see above)**
8. **README.md structure tree** — if files or directories were added, moved, or deleted, update the ASCII tree in `README.md`
9. **Commit message format** — if versions were bumped, the commit message must start with the version prefix(es): `v{VERSION}` for `.gs`, `v{BUILD_VERSION}` for HTML (e.g. `v01.14g v01.02w Fix bug`)
10. **Developer branding** — any newly created file must have `Developed by: DEVELOPER_NAME` as the last line (using the appropriate comment syntax for the file type), where `DEVELOPER_NAME` is resolved from the Template Variables table
11. **README.md `Last updated:` timestamp** — on every commit, update the `Last updated:` timestamp near the top of `README.md` to the real current time (run `TZ=America/New_York date '+%Y-%m-%d %I:%M:%S %p EST'`). **This rule always applies — it is NOT skipped by the Template Repo Guard**
12. **Internal link integrity** — if any markdown file is added, moved, or renamed, verify that all internal links (`[text](path)`) in the repo still resolve to existing files. Pay special attention to cross-directory links — see the Internal Link Reference section for the correct relative paths
13. **README section link tips** — every `##` section in `README.md` that contains (or will contain) any clickable links must have this blockquote as the first line after the heading (before any other content): `> **Tip:** The links below navigate away from this page. **Ctrl + click** (or right-click → *Open in new tab*) to keep these instructions visible while you work.` — Sections with no links (e.g. a section with only a code block or plain text) do not need the tip

### Maintaining these checklists
- The Session Start and Pre-Commit checklists are the **single source of truth** for all actionable rules. Detailed sections below provide reference context only
- When adding new rules to CLAUDE.md, add the actionable check to the appropriate checklist and put supporting details in a reference section — do not duplicate the rule in both places
- When editing CLAUDE.md, check whether any existing reference section restates a checklist item — if so, remove the duplicate and add a `*Rule: see ... Checklist item #N*` pointer instead
- **Section separators** — every `##` section in CLAUDE.md must end with a double-ruled banner. When adding a new `##` section, add the following block between the end of its content and the next `##` heading:
  ```
  ---
  > **--- END OF SECTION NAME ---**
  ---
  ```
  Replace `SECTION NAME` with the section's heading in ALL CAPS. The only exception is Developer Branding (the final section), which has no separator after it

---
> **--- END OF PRE-COMMIT CHECKLIST ---**
---

## Template Variables

These variables are the **single source of truth** for repo-specific values. When a variable value is changed here, Claude Code must propagate the new value to every file in the repo that uses it.

| Variable | Value | Where it appears |
|----------|-------|------------------|
| `YOUR_ORG_NAME` | `YourOrgName` | README (live site link), CITATION.cff (repository URL, site URL), STATUS (live URL), ARCHITECTURE (diagram URL), issue template config (URLs) |
| `YOUR_ORG_LOGO_URL` | `https://logoipsum.com/logoipsum-avatar.png` | `index.html` and template HTML (`YOUR_ORG_LOGO_URL` JS variable), available for use in pages that need the org logo |
| `YOUR_REPO_NAME` | `YourRepoName` | README (structure tree, live site link), CITATION.cff (repository URL, site URL), STATUS (live URL), ARCHITECTURE (diagram URL), issue template config (URLs) |
| `YOUR_PROJECT_TITLE` | `Auto Update HTML Template` | README (title), `<title>` tag in `live-site-pages/index.html` and `live-site-templates/AutoUpdateOnlyHtmlTemplate.html` |
| `DEVELOPER_NAME` | `ShadowAISolutions` | LICENSE (copyright), README ("Developed by:" footer), CITATION.cff (author name), "Developed by:" footers (all files including `index.html`, template HTML, workflow, issue templates, YAML, Markdown), FUNDING.yml (sponsor handle), GOVERNANCE (ownership), CONTRIBUTING (convention text), PR template (checklist + footer) |
| `DEVELOPER_LOGO_URL` | `https://www.shadowaisolutions.com/SAIS%20Logo.png` | HTML splash screen `LOGO_URL` variable (in `index.html` and template) |

### How variables work
- **In code files** (HTML, YAML, Markdown, etc.): use the **resolved value** (e.g. write `MyOrgName`, not `YOUR_ORG_NAME`)
- **In CLAUDE.md instructions**: the placeholder names (`YOUR_ORG_NAME`, `DEVELOPER_NAME`, etc.) may appear in examples and rules — Claude Code resolves them using the table above

---
> **--- END OF TEMPLATE VARIABLES ---**
---

## Version Bumping
*Rule: see Pre-Commit Checklist item #1. Reference details below.*
- The `VERSION` variable is near the top of each `.gs` file (look for `var VERSION = "..."`)
- Format includes a `g` suffix: e.g. `"01.13g"` → `"01.14g"`
- Do NOT bump VERSION if the commit doesn't touch the `.gs` file

### GAS Projects
Each GAS project has a code file and a corresponding embedding page. Register them in the table below as you add them.

| Project | Code File | Embedding Page |
|---------|-----------|----------------|
| *(Project name)* | `googleAppsScripts/<Project Name>/<CodeFile>.gs` | `live-site-pages/<page-name>.html` |

---
> **--- END OF VERSION BUMPING ---**
---

## Build Version (Auto-Refresh for embedding pages)
*Rules: see Pre-Commit Checklist items #2, #3, #4. Reference details below.*
- Look for `<meta name="build-version" content="...">` in the `<head>`
- Format includes a `w` suffix: e.g. `"01.11w"` → `"01.12w"`
- Each embedding page polls `version.txt` every 10 seconds — when the deployed version differs from the loaded version, it auto-reloads

### Auto-Refresh via version.txt Polling
- **All embedding pages must use the `version.txt` polling method** — do NOT poll the page's own HTML
- **Version file naming**: the version file must be named `<page-name>.version.txt`, matching the HTML file it tracks (e.g. `index.html` → `index.version.txt`, `dashboard.html` → `dashboard.version.txt`). The `.version.txt` double extension ensures the version file sorts **after** the `.html` file alphabetically
- Each version file holds only the current build-version string (e.g. `01.08w`)
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
- The template in `live-site-templates/AutoUpdateOnlyHtmlTemplate.html` already implements this pattern — use it as a starting point for new projects

### New Embedding Page Setup Checklist
When creating a **new** HTML embedding page, follow every step below:

1. **Copy the template** — start from `live-site-templates/AutoUpdateOnlyHtmlTemplate.html`, which already includes:
   - `<meta name="build-version" content="...">` in the `<head>`
   - Version file polling logic (10-second interval)
   - Version indicator pill (bottom-right corner)
   - Green "Website Ready" splash overlay + sound playback
   - AudioContext handling and screen wake lock
2. **Choose the directory** — create a new subdirectory under `live-site-pages/` named after the project (e.g. `live-site-pages/my-project/`)
3. **Create the version file** — place a `<page-name>.version.txt` file in the **same directory** as the HTML page (e.g. `index.version.txt` for `index.html`), containing only the initial build-version string (e.g. `01.00w`)
4. **Update the polling URL in the template** — ensure the JS version-file URL derivation matches the HTML filename (the template defaults to deriving it from the page's own filename)
5. **Create `sounds/` directory** — copy the `sounds/` folder (containing `Website_Ready_Voice_1.mp3`) into the new page's directory so the splash sound works
6. **Set the initial build-version** — in the HTML `<head>`, set `<meta name="build-version" content="01.00w">` and match it in `<page-name>.version.txt`
7. **Update the page title** — replace `YOUR_PROJECT_TITLE` in `<title>` with the actual project name
8. **Register in GAS Projects table** — if this page embeds a GAS iframe, add a row to the GAS Projects table in the Version Bumping section above
9. **Add developer branding** — ensure `<!-- Developed by: DEVELOPER_NAME -->` is the last line of the HTML file

### Directory Structure (per embedding page)
```
live-site-pages/
├── <page-name>/
│   ├── index.html               # The embedding page (from template)
│   ├── index.version.txt        # Tracks index.html build-version (e.g. "01.00w")
│   └── sounds/
│       └── Website_Ready_Voice_1.mp3
```
For pages that live directly in `live-site-pages/` (not in a subdirectory), the version file and `sounds/` folder sit alongside the HTML file (e.g. `live-site-pages/index.html` + `live-site-pages/index.version.txt`).

---
> **--- END OF BUILD VERSION ---**
---

## Commit Message Naming
*Rule: see Pre-Commit Checklist item #9. Reference details below.*
- Both version types use the `v` prefix — suffix indicates type: `g` = Google Apps Script, `w` = website
- If neither a `.gs` file nor an embedding HTML page was updated: no version prefix needed
- Example: `v01.19g Fix sign-in popup to auto-close after authentication`
- Example: `v01.19g v01.12w Add auth wall with build version bump`

---
> **--- END OF COMMIT MESSAGE NAMING ---**
---

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

---
> **--- END OF GAS CODE CONSTRAINTS ---**
---

## Race Conditions — Config vs. Data Fetch
- **Never fire `saveConfig` and a dependent data-fetch (`getFormData`) in parallel** — the data-fetch may read stale config values from the sheet
- When the client switches a config value (e.g. year) and needs fresh data for that value, **pass the value directly as a parameter** to the server function (e.g. `getFormData(_token, year)`) rather than relying on `saveConfig` completing first
- Server functions that read config should accept optional override parameters (e.g. `opt_yearOverride`) so the client can bypass the saved config when needed
- This pattern avoids race conditions without needing to chain callbacks (which adds latency)

---
> **--- END OF RACE CONDITIONS ---**
---

## API Call Optimization (Scaling Goal)
- **Minimize Google API calls** in every GAS function — the app is designed to scale to many users
- **Cache `getUserInfo` results** in `CacheService` (keyed by token suffix) for 5 minutes to avoid hitting the OAuth userinfo endpoint on every `google.script.run` call
- **Cache `checkSpreadsheetAccess` results** in `CacheService` (keyed by email) for 10 minutes to avoid listing editors/viewers on every call
- **Open `SpreadsheetApp.openById()` once per function** — pass the `ss` object to `checkSpreadsheetAccess(email, opt_ss)` instead of opening the spreadsheet twice
- When adding new server-side functions, always consider: can this result be cached? Can I reuse an already-opened spreadsheet object? Avoid redundant `UrlFetchApp` or `SpreadsheetApp` calls
- Cache TTLs are intentionally short (5–10 min) so permission changes and token revocations take effect quickly

---
> **--- END OF API CALL OPTIMIZATION ---**
---

## UI Dialogs — No Browser Defaults
- **Never use `alert()`, `confirm()`, or `prompt()`** — all confirmation dialogs, alerts, and input prompts must use custom styled HTML/CSS modals
- This applies to both GAS `.gs` code and parent embedding pages (`.html`)
- Use overlay + modal patterns consistent with the existing sheet/modal styles in the codebase

---
> **--- END OF UI DIALOGS ---**
---

## Phantom Edit (Timestamp Alignment)
- When the user asks for a **phantom edit** or **phantom update**, touch every file in the repo with a no-op change so all files share the same commit timestamp on GitHub
- **Skip all version bumps** — do NOT increment `build-version` in HTML pages or `VERSION` in `.gs` files
- For text files: add a trailing newline
- For binary files (e.g. `.mp3`): append a null byte
- **Reset `repository-information/CHANGELOG.md`** — replace all entries with a fresh template (keep the header, version suffix note, and an empty `[Unreleased]` section with `*(No changes yet)*`). This gives the repo a clean history starting point
- **Update `Last updated:` in `README.md`** — set the timestamp to the real current time (run `TZ=America/New_York date '+%Y-%m-%d %I:%M:%S %p EST'`). This is the only substantive edit besides the no-op touches
- Commit message: `Phantom edit to align all file timestamps on GitHub` (no version prefix)

---
> **--- END OF PHANTOM EDIT ---**
---

## Execution Style
- For clear, straightforward requests: **just do it** — make the changes, commit, and push without asking for plan approval
- Only ask clarifying questions when the request is genuinely ambiguous or has multiple valid interpretations
- Do not use formal plan-mode approval workflows for routine tasks (version bumps, file moves, feature additions, bug fixes, etc.)

---
> **--- END OF EXECUTION STYLE ---**
---

## AudioContext & Browser Autoplay Policy
- **AudioContext starts as `'suspended'`** on every page load — browsers require a user gesture (click/touch) before allowing audio playback
- **`resume()` without a gesture** generally stays pending or silently fails. It does NOT reject — the promise just never resolves, which causes dangling `.then()` callbacks that fire unexpectedly when the user eventually clicks
- **Never schedule `decodeAudioData` + `source.start(0)` while context is suspended** — the audio gets queued and plays the moment the context resumes (on user click), causing surprise delayed sound. Instead, gate playback behind `if (ctx.state !== 'running') return`
- **JS-triggered `window.location.reload()` vs manual F5 refresh** behave differently: JS reload carries forward the gesture allowance (AudioContext can auto-resume), F5 does NOT. So auto-refresh reloads can play sound, but manual refreshes cannot
- **`onstatechange` fires before DOM is ready**: when `resume()` is called early in the script, `onstatechange` may fire before the `#audio-status` element exists — the `updateAudioStatus()` call bails silently and never retries. Fix: save the resume promise and chain `updateAudioStatus` onto it after the element is created
- **Use `sessionStorage` (not `localStorage`) for audio state flags** — `sessionStorage` is per-tab, so a flag set in one tab doesn't leak into new tabs that have no gesture context. `localStorage` would cause false-positive "sound ready" icons in fresh tabs
- **The `audio-unlocked` sessionStorage flag** remembers that audio was successfully unlocked in this tab. On F5 refresh, the AudioContext is suspended but the flag tells the icon to show "ready" instead of "muted" — because a click will instantly restore it. Without this flag, the icon flashes "muted" on every refresh even though audio works fine
- **Chrome "Duplicate Tab" copies `sessionStorage`** — including the `audio-unlocked` flag — into the new tab, but the new tab has no gesture context, so the AudioContext is suspended. The stale flag causes the icon to falsely show "sound ready." Fix: use `performance.getEntriesByType('navigation')` to detect the navigation type; if it's anything other than `'reload'` (e.g. `'navigate'` for duplicate tab, back/forward, or new navigation), clear the `audio-unlocked` flag before creating the AudioContext. This must run **before** `new AudioContext()` so that `updateAudioStatus()` sees the correct flag state from the start

---
> **--- END OF AUDIOCONTEXT & BROWSER AUTOPLAY POLICY ---**
---

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

---
> **--- END OF GOOGLE SIGN-IN (GIS) ---**
---

## Keeping Documentation Files in Sync
*Mandatory rules: see Pre-Commit Checklist items #5, #6, #7, #8. Reference table below for additional files to consider.*

| File | Update when... |
|------|---------------|
| `.gitignore` | New file types or tooling is introduced that generates artifacts (e.g. adding Node tooling, Python venvs, build outputs) |
| `.editorconfig` | New file types are introduced that need specific formatting rules |
| `CONTRIBUTING.md` | Development workflow changes, new conventions are added to CLAUDE.md that contributors need to know |
| `SECURITY.md` | New attack surfaces are added (e.g. new API endpoints, new OAuth flows, new deployment targets) |
| `CITATION.cff` | Project name, description, authors, or URLs change |
| `.github/ISSUE_TEMPLATE/*.yml` | New project areas are added (update the "Affected Area" / "Area" dropdown options) |
| `.github/PULL_REQUEST_TEMPLATE.md` | New checklist items become relevant (e.g. new conventions, new mandatory checks) |

Update these only when the change is genuinely relevant — don't force unnecessary edits.

---
> **--- END OF KEEPING DOCUMENTATION FILES IN SYNC ---**
---

## Internal Link Reference
*Rule: see Pre-Commit Checklist item #12. Correct relative paths below.*

Files live in three locations: repo root, `.github/`, and `repository-information/`. Cross-directory links must use `../` to traverse up before descending into the target directory.

### Why community health files live at root (not `.github/`)
GitHub renders community health files (`CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`) in two contexts: the **blob view** (direct file URL) and the **community sidebar tabs** on the repo's main page. These two contexts use different base URLs for resolving relative links — blob view uses `/org/repo/blob/main/.github/`, while the sidebar tab uses `/org/repo/tree/main`. Any relative link with `../` (needed to escape `.github/`) breaks in the tab context because the `../` traverses into GitHub's URL structure instead of the file system. Moving these files to the repo root eliminates the mismatch — root-level relative links resolve identically in both contexts, just like `README.md` links. `CODE_OF_CONDUCT.md` is included even though it currently has no links, to ensure future links work correctly without needing a file move.

### File locations
| File | Actual path |
|------|-------------|
| README.md | `./README.md` (root) |
| CLAUDE.md | `./CLAUDE.md` (root) |
| LICENSE | `./LICENSE` (root) |
| CODE_OF_CONDUCT.md | `./CODE_OF_CONDUCT.md` (root) |
| CONTRIBUTING.md | `./CONTRIBUTING.md` (root) |
| SECURITY.md | `./SECURITY.md` (root) |
| PULL_REQUEST_TEMPLATE.md | `.github/PULL_REQUEST_TEMPLATE.md` |
| ARCHITECTURE.md | `repository-information/ARCHITECTURE.md` |
| CHANGELOG.md | `repository-information/CHANGELOG.md` |
| GOVERNANCE.md | `repository-information/GOVERNANCE.md` |
| IMPROVEMENTS.md | `repository-information/IMPROVEMENTS.md` |
| STATUS.md | `repository-information/STATUS.md` |
| SUPPORT.md | `repository-information/SUPPORT.md` |
| TODO.md | `repository-information/TODO.md` |

### Common cross-directory link patterns
| From directory | To file in `repository-information/` | Correct relative path |
|----------------|--------------------------------------|----------------------|
| `.github/` | `repository-information/SUPPORT.md` | `../repository-information/SUPPORT.md` |
| `.github/` | `repository-information/CHANGELOG.md` | `../repository-information/CHANGELOG.md` |

| From directory | To root files | Correct relative path |
|----------------|--------------|----------------------|
| `repository-information/` | `README.md` | `../README.md` |
| `repository-information/` | `CLAUDE.md` | `../CLAUDE.md` |
| `repository-information/` | `CONTRIBUTING.md` | `../CONTRIBUTING.md` |
| `repository-information/` | `SECURITY.md` | `../SECURITY.md` |
| `repository-information/` | `CODE_OF_CONDUCT.md` | `../CODE_OF_CONDUCT.md` |
| `.github/` | `README.md` | `../README.md` |
| `.github/` | `CLAUDE.md` | `../CLAUDE.md` |

---
> **--- END OF INTERNAL LINK REFERENCE ---**
---

## Relative Path Resolution on GitHub
*Rule: see Template Drift Checks item #3. Reference details below.*

GitHub renders markdown files at blob-view URLs with the structure:
```
https://github.com/{org}/{repo}/blob/{branch}/{path-to-file}
```

The browser resolves relative links from the **directory** portion of that URL. This means `../` traversals climb through GitHub's URL segments (`blob/`, `main/`, subdirectories) to reach the repo root — which is dynamically determined by the org and repo name.

### Traversal math by directory depth

| File location | Blob-view directory | `../` count to repo root |
|---------------|-------------------|--------------------------|
| Root (`README.md`) | `/org/repo/blob/main/` | 2 (`../../`) |
| `.github/` | `/org/repo/blob/main/.github/` | 3 (`../../../`) |
| `repository-information/` | `/org/repo/blob/main/repository-information/` | 3 (`../../../`) |
| `.github/ISSUE_TEMPLATE/` | `/org/repo/blob/main/.github/ISSUE_TEMPLATE/` | 4 (`../../../../`) |

### Example: SECURITY.md advisory link

```
File:      SECURITY.md
Blob URL:  https://github.com/AnyOrg/AnyRepo/blob/main/SECURITY.md
Directory: /AnyOrg/AnyRepo/blob/main/

Link:      ../../security/advisories/new
Step 1:    ../   removes main/          → /AnyOrg/AnyRepo/blob/
Step 2:    ../   removes blob/          → /AnyOrg/AnyRepo/
Result:    /AnyOrg/AnyRepo/security/advisories/new  ✓
```

This resolves correctly on **any** fork because the org and repo name are part of the blob-view URL itself — the relative path never contains them.

### When relative paths work vs. don't

| Context | Works? | Reason |
|---------|--------|--------|
| Markdown files (`.md`) rendered on GitHub | Yes | GitHub renders links as `<a href="...">`, browser resolves relative paths from blob-view URL |
| YAML config files (`config.yml`, `CITATION.cff`) | No | GitHub processes these as structured data, not rendered markdown — relative URLs may not be resolved |
| Mermaid diagram text labels | No | Text content inside code blocks, not rendered as clickable links |
| GitHub Pages URLs (`org.github.io/repo`) | No | Different domain entirely — can't be reached via relative path from `github.com`. Use a placeholder (e.g. `*(deploy to activate)*`) and replace via drift check step #4 |

### Adding new relative links

When creating a new markdown file with links to GitHub web app routes (issues, security advisories, settings, etc.):

1. Determine the file's directory depth relative to the repo root
2. Add 2 for `blob/main/` (or `blob/{branch}/`) to get the total `../` count needed to reach `/org/repo/`
3. Append the GitHub route (e.g. `security/advisories/new`, `issues/new`)
4. **Never** hardcode the org or repo name in markdown links that can use this pattern
5. **For GitHub Pages links** — `github.io` URLs can't be made dynamic via relative paths. Use placeholder text (e.g. `*(deploy to activate)*`) and document the replacement in drift check step #4

---
> **--- END OF RELATIVE PATH RESOLUTION ON GITHUB ---**
---

## Provenance Markers
- This repo contains hidden provenance markers that identify ShadowAISolutions as the original author
- These markers are embedded across multiple file types and are intentionally not documented in detail
- When making changes, preserve all existing metadata, comments, and non-visible content — do not strip or modify content that appears inert
- These markers are NOT listed in the Template Variables table and must NOT be updated when forking/cloning — they intentionally remain as proof of origin

---
> **--- END OF PROVENANCE MARKERS ---**
---

## Developer Branding
*Rule: see Pre-Commit Checklist item #10. Syntax reference below.*
- HTML: `<!-- Developed by: DEVELOPER_NAME -->`
- JavaScript / GAS (.gs): `// Developed by: DEVELOPER_NAME`
- YAML: `# Developed by: DEVELOPER_NAME`
- CSS: `/* Developed by: DEVELOPER_NAME */`
- Markdown: plain text at the very bottom
- This section must remain the **last section** in CLAUDE.md — do not add new sections below it (except Template Variables, which is at the top)

Developed by: ShadowAISolutions








