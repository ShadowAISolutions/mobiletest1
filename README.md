# demorepo4

A GitHub Pages deployment framework with automatic version polling, auto-refresh, and Google Apps Script (GAS) embedding support.

**Live site:** [ShadowAISolutions.github.io/demorepo4](https://ShadowAISolutions.github.io/demorepo4)

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
demorepo4/
├── httpsdocs/                  # Deployed to GitHub Pages
│   ├── index.html              # Live landing page
│   ├── index.version.txt       # Version file for auto-refresh
│   └── sounds/                 # Audio feedback files
├── autoUpdateTemplateFiles/    # Template for new pages
├── .github/
│   ├── workflows/              # CI/CD pipeline
│   ├── ISSUE_TEMPLATE/         # Bug report & feature request forms
│   ├── PULL_REQUEST_TEMPLATE.md # PR checklist
│   └── FUNDING.yml             # Sponsor button config
├── repo-info/
│   ├── ARCHITECTURE.md         # System diagram (Mermaid)
│   ├── CHANGELOG.md            # Version history
│   ├── CODE_OF_CONDUCT.md      # Community standards
│   ├── CONTRIBUTING.md         # How to contribute
│   ├── GOVERNANCE.md           # Project governance
│   ├── SECURITY.md             # Vulnerability reporting
│   ├── STATUS.md               # Project status dashboard
│   └── SUPPORT.md              # Getting help
├── CITATION.cff                # Citation metadata
├── CLAUDE.md                   # Developer instructions
└── LICENSE                     # Proprietary license
```

## Documentation

| Document | Description |
|----------|-------------|
| [ARCHITECTURE.md](repo-info/ARCHITECTURE.md) | Visual system diagram (Mermaid) |
| [CHANGELOG.md](repo-info/CHANGELOG.md) | Version history |
| [CLAUDE.md](CLAUDE.md) | Developer instructions and conventions |
| [STATUS.md](repo-info/STATUS.md) | Current project status and versions |

## Community

| Document | Description |
|----------|-------------|
| [Code of Conduct](repo-info/CODE_OF_CONDUCT.md) | Community standards and expectations |
| [Contributing](repo-info/CONTRIBUTING.md) | How to contribute to this project |
| [Security Policy](repo-info/SECURITY.md) | How to report vulnerabilities |
| [Support](repo-info/SUPPORT.md) | Getting help |
| [Governance](repo-info/GOVERNANCE.md) | Project ownership and decision making |

Developed by: ShadowAISolutions

