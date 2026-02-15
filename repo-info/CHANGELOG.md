# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

Version suffixes: `w` = website (HTML pages), `g` = Google Apps Script.

## [Unreleased] - 2026-02-15
### Changed
- Reverted license back to proprietary (All Rights Reserved) for work use
- Reset all build-versions to 01.00w (index.html, template, version.txt files) for clean template baseline
- Renamed `docs/` folder to `repo-info/` for clarity; updated all references

## [v01.04w] - 2026-02-14
### Changed
- Bumped index.html build-version to 01.04w

## [v01.03w] - 2026-02-14
### Changed
- Bumped index.html build-version to 01.03w

## [v01.02w] - 2026-02-14
### Changed
- Bumped index.html build-version to 01.02w

## [v01.01w] - 2026-02-14
### Changed
- Reset index.html build-version to 01.01w
- Added `pageName` fallback to template and index to prevent infinite reload loops when accessed via directory URL

### Fixed
- Constant self-reload when page accessed without `index.html` in URL

## Earlier Development

### Audio System
- Fixed audio icon staying muted when sound is playing (v01.10w)
- Fixed muted icon on manual refresh, cleaned up audio resume flow (v01.11w)
- Fixed delayed sound on first click after no-gesture auto-refresh (v01.14w)
- Fixed muted icon flash on manual refresh (v01.17w–v01.18w)
- Fixed false positive audio icon in new/duplicated tabs (v01.21w)
- Added `sessionStorage` flag for audio state, Chrome duplicate tab detection

### Version Polling System
- Switched auto-refresh from full HTML polling to lightweight `version.txt` (v01.07w)
- Switched template to `version.txt` polling method (v01.03w template)
- Renamed version files to `.version.txt` for alphabetical sorting (v01.10w)
- Added `pageName` fallback for directory URL access

### Infrastructure
- Added CI/CD workflow (`auto-merge-claude.yml`) for auto-merge and deployment
- Created auto-update HTML template with splash screen, audio, and version polling
- Added developer branding ("Developed by: ShadowAISolutions") to all code files
- Added `CLAUDE.md` with comprehensive project conventions
- Added `ARCHITECTURE.md` with Mermaid system diagram

Developed by: ShadowAISolutions

