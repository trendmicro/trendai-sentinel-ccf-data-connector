# Go-Live Checklist

One-time setup required by the Trend Micro
[OpenSource Community Standards Policy](https://trendmicro.atlassian.net/wiki/spaces/rdsecpub/pages/501337248)
before flipping this repository from internal to public.

Most of these items live in GitHub **repository settings**, not in files, so they cannot be checked into the repo. The maintainer who publishes the repo is responsible for ticking them off.

## Before going public

- [ ] **Snowden / Secret Finder on-demand scan** completed against the full history
      — <https://secretfinder.infosec.trendmicro.com/> and
      <https://snowden.devsecops.trendmicro.com/>
- [ ] All secret-finder findings remediated (rewrite history if needed —
      `git filter-repo` is safer than `filter-branch`)
- [ ] `.gitignore` covers any secret patterns specific to your workflow
- [ ] All branches except `main` are intentional (delete stale feature branches)
- [ ] README clearly describes the project, install/usage, contributing, and license
- [ ] LICENSE, SECURITY.md, CODE_OF_CONDUCT.md, CONTRIBUTING.md present and current
- [ ] Issue templates and PR template present under `.github/`

## At time of going public

- [ ] Change repository visibility to **Public**
      (Settings → General → Danger Zone → Change visibility)
- [ ] Update repository **Description** and **Topics** (top-right of repo page)
- [ ] Set repository **Website** to the docs URL if applicable

## Repository settings to enable (Required by policy)

### Settings → Code security and analysis

- [ ] **Secret scanning** → Enable
- [ ] **Push protection** → Enable (blocks pushes containing known secret patterns)
- [ ] **Dependabot alerts** → Enable
- [ ] **Dependabot security updates** → Enable
- [ ] **Code scanning** → "Set up" — picks up `.github/workflows/codeql.yml`

### Settings → Branches → Branch protection rule for `main`

- [ ] Require a pull request before merging
- [ ] Require approvals: **1** minimum (more for sensitive components)
- [ ] Dismiss stale approvals when new commits are pushed
- [ ] Require review from **Code Owners** (uses `CODEOWNERS`)
- [ ] Require status checks to pass:
  - [ ] `CodeQL`
  - [ ] `Validate Templates`
  - [ ] `Gitleaks`
- [ ] Require branches to be up to date before merging
- [ ] Block force pushes
- [ ] Block deletions

### Settings → General → Pull Requests

- [ ] Allow **squash merging** only (matches contributing guidelines)
- [ ] Automatically delete head branches after merge

## Repository settings to enable (Suggested by policy)

- [ ] **Settings → General → Require contributors to sign off on web-based commits**
- [ ] Org-level policy: Personal Access Tokens require expiration
      (org admin, not repo setting)
- [ ] Encourage contributors to enable **GPG / SSH commit signature verification**
      locally (linked from `CONTRIBUTING.md`)

## After going public

- [ ] Confirm Dependabot opened its first scan (Insights → Dependency graph)
- [ ] Confirm CodeQL ran successfully on the default branch
      (Security → Code scanning alerts)
- [ ] Confirm Gitleaks workflow ran clean on `main`
- [ ] Add real GitHub team handle to `CODEOWNERS`
      (currently `@trendmicro/mxdr-team` placeholder)
- [ ] Set up notifications / on-call rotation for security alerts

## References

- Trend Micro OpenSource policy:
  <https://trendmicro.atlassian.net/wiki/spaces/rdsecpub/pages/501337248>
- GitHub community standards:
  <https://opensource.guide/>
- GitHub branch protection docs:
  <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches>
