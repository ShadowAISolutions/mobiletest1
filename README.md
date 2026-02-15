# ​‌‌‌‌‌‌‌​​‌‌‌‌‌​autoupdatehtmltemplate

A GitHub Pages deployment framework with automatic version polling, auto-refresh, and Google Apps Script (GAS) embedding support.

Last updated: `2026-02-15 02:21:03 PM EST`

You are currently using the **autoupdatehtmltemplate**, update your code and claude will update the live site link here

## How It Works

### Auto-Refresh via Version Polling
Every hosted page polls a lightweight `.version.txt` file every 10 seconds. When a new version is deployed, the page detects the mismatch and auto-reloads — showing a green "Website Ready" splash with audio feedback.

### CI/CD Auto-Merge Flow
1. Push to a `claude/*` branch
2. GitHub Actions automatically merges into `main`, deploys to GitHub Pages, and cleans up the branch
3. No pull requests needed — the workflow handles everything

### GAS Embedding Architecture
Google Apps Script projects are embedded as iframes in GitHub Pages. The framework handles:
- Automatic GAS deployment via `doPost` when `.gs` files change
- "Code Ready" blue splash on GAS updates (client-side polling)
- Google Sign-In from the parent page (stable OAuth origin)

## Project Structure

```
autoupdatehtmltemplate/
├── live-site-pages/             # Deployed to GitHub Pages
│   ├── index.html              # Live landing page
│   ├── index.version.txt       # Version file for auto-refresh
│   └── sounds/                 # Audio feedback files
├── live-site-templates/        # Template for new pages
├── .github/
│   ├── workflows/              # CI/CD pipeline
│   ├── ISSUE_TEMPLATE/         # Bug report & feature request forms
│   ├── PULL_REQUEST_TEMPLATE.md # PR checklist
│   ├── CODE_OF_CONDUCT.md      # Community standards
│   ├── CONTRIBUTING.md         # How to contribute
│   ├── SECURITY.md             # Vulnerability reporting
│   └── FUNDING.yml             # Sponsor button config
├── repository-information/
│   ├── ARCHITECTURE.md         # System diagram (Mermaid)
│   ├── CHANGELOG.md            # Version history
│   ├── GOVERNANCE.md           # Project governance
│   ├── IMPROVEMENTS.md         # Potential improvements
│   ├── STATUS.md               # Project status dashboard
│   ├── TODO.md                 # Actionable to-do items
│   └── SUPPORT.md              # Getting help
├── CITATION.cff                # Citation metadata
├── CLAUDE.md                   # Developer instructions
└── LICENSE                     # Proprietary license
```

## Documentation

| Document | Description |
|----------|-------------|
| [ARCHITECTURE.md](repository-information/ARCHITECTURE.md) | Visual system diagram (Mermaid) |
| [CHANGELOG.md](repository-information/CHANGELOG.md) | Version history |
| [CLAUDE.md](CLAUDE.md) | Developer instructions and conventions |
| [IMPROVEMENTS.md](repository-information/IMPROVEMENTS.md) | Potential improvements to explore |
| [STATUS.md](repository-information/STATUS.md) | Current project status and versions |
| [TODO.md](repository-information/TODO.md) | Actionable planned items |

## Community

| Document | Description |
|----------|-------------|
| [Code of Conduct](.github/CODE_OF_CONDUCT.md) | Community standards and expectations |
| [Contributing](.github/CONTRIBUTING.md) | How to contribute to this project |
| [Security Policy](.github/SECURITY.md) | How to report vulnerabilities |
| [Support](repository-information/SUPPORT.md) | Getting help |
| [Governance](repository-information/GOVERNANCE.md) | Project ownership and decision making |

Developed by: ShadowAISolutions









