# ​‌‌‌‌‌‌‌​​‌‌‌‌‌​autoupdatehtmltemplate

A GitHub Pages deployment framework with automatic version polling, auto-refresh, and Google Apps Script (GAS) embedding support.

Last updated: `2026-02-15 03:06:36 PM EST`

You are currently using the **autoupdatehtmltemplate**, update your code and claude will update the live site link here

## Initialize This Template

After copying this template to your own repository, follow these steps to get your live site running:

### 1. Enable GitHub Pages

Go to your repository's [**Pages settings**](https://github.com/ShadowAISolutions/autoupdatehtmltemplate/settings/pages) and configure:

- **Source**: Select **GitHub Actions** (not "Deploy from a branch")
- This allows the included workflow to deploy your `live-site-pages/` directory automatically

### 2. Create the `github-pages` Environment

Go to your repository's [**Environments settings**](https://github.com/ShadowAISolutions/autoupdatehtmltemplate/settings/environments) and:

- Click **New environment**
- Name it exactly `github-pages`
- No additional protection rules are needed — the workflow handles deployment automatically

### 3. Run Claude Code

Open the repo with Claude Code. On first run, the **Session Start Checklist** in `CLAUDE.md` will automatically:

- Detect your new repo name and org
- Update all references throughout the codebase
- Replace the placeholder text above with your live site link

### 4. Push Your First Change

Push to any `claude/*` branch — the workflow will:

1. Merge into `main`
2. Deploy to GitHub Pages
3. Clean up the branch

Your site will be live at `https://<your-org>.github.io/<your-repo>/`

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









