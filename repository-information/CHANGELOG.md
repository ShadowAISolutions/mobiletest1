# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

Version suffixes: `w` = website (HTML pages), `g` = Google Apps Script.

## [Unreleased] — 2026-02-16 23:53:55 EST

### Fixed
- `2026-02-16 23:53:55 EST` — Fix Website Ready sound not playing on auto-refresh reload: replaced fragile 500ms setTimeout fallback with _resumePromise.then() chain so playback fires exactly when AudioContext.resume() completes (v01.01w)

Developed by: ShadowAISolutions

