# dotfiles

My home-directory dotfiles (`~/.*`).

## Development

Install the tooling once after cloning:

```sh
brew install gitleaks
nvm use
npm install
```

`npm install` runs `husky`, which points `core.hooksPath` at `.husky/_` for this repo only. That overrides a global `core.hooksPath`, if you have one.

Node comes from `.nvmrc` (`24`). `npm-run-all2` v9 needs Node 24.15 or newer, so an older Node 24 install prints `EBADENGINE` warnings. Run `nvm install` to get the current 24.x.

### Scripts

- `npm run lint`: checks formatting with Prettier.
- `npm run format`: rewrites files with Prettier. Files Prettier has no parser for, such as `.zshrc`, are skipped.

### Git hooks

- `pre-commit`: formats staged files with `lint-staged`, then scans the staged changes with gitleaks. The commit is refused if gitleaks finds a secret, or if gitleaks is not installed. Failing closed matters here, because a dotfiles repo is an easy place to paste a token by accident.
- `post-merge`: runs `npm ci` so dependencies match the lockfile after a pull.

The hook uses `gitleaks git --pre-commit --staged`. The older `gitleaks protect --staged`, used in some of my other repos, is deprecated since gitleaks v8.19.

Never bypass the hooks with `--no-verify`.

### Secret scanning on GitHub

Two layers run after the local hook:

- `.github/workflows/secrets-scan.yml` runs gitleaks over the full git history on every push and pull request, daily at 4 AM UTC, and on demand. The daily run re-checks old commits against rules added in newer gitleaks releases.
- GitHub's own secret scanning and push protection are enabled in the repo settings. Push protection rejects a push that contains a known provider token.

### Dependencies

Renovate keeps dependencies current, using the shared preset in `JaneJeon/dev` (`packages/renovate-config/default.json`).
