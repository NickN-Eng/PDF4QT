# Security Posture Report

## Summary

| Field | Value |
|-------|-------|
| Repository | `PDF4QT` (local clone; upstream: github.com/JakubMelka/PDF4QT) |
| Review date | 2026-05-11 |
| Scorecard available | No (manual assessment only) |
| Overall posture | Weak |

## Repository Hygiene

| Indicator | Status | Evidence | Notes |
|-----------|--------|----------|-------|
| Security policy (SECURITY.md) | Absent | Not found in repo root or `.github/` | No vulnerability reporting process documented |
| Licence file | Present | `LICENSE` (MIT) | Clear, permissive licence |
| Code of conduct | Absent | Not found | |
| Contributing guidelines | Absent | Not found | |
| README documentation | Adequate | `README.md` present | |

## Security Process Maturity

| Indicator | Status | Evidence | Notes |
|-----------|--------|----------|-------|
| SAST configured | No | No CodeQL, Semgrep, or similar in workflows | |
| Dependency scanning | No | No Dependabot/Renovate config, no dependency-review action | |
| Secret scanning | Unclear | No explicit config; GitHub may enable by default for public repos | |
| Code review enforced | Unclear | PRs exist (e.g., #298) but most commits pushed directly by maintainer | |
| Branch protection | Unclear | Direct pushes to `master` visible in commit history | Likely not enforced |
| Signed releases | Yes (partial) | GPG signing in `LinuxInstall.yml`; code signing via DigiCert Keylocker in `WindowsInstall.yml` | Releases are signed |

## CI/CD Security

| Indicator | Status | Risk | Files |
|-----------|--------|------|-------|
| Workflow permissions | Unset (defaults) | Medium | All workflow files lack explicit `permissions:` key |
| Actions pinned to SHA | No | Medium | Uses `actions/checkout@v4`, `actions/cache@v4`, `jurplel/install-qt-action@v4` — tag-based, not SHA-pinned |
| Dangerous patterns | None found | Low | No `pull_request_target` with checkout patterns detected |
| Token scoping | Adequate | Low | `secrets.MY_GITHUB_TOKEN` used for release draft; no broad write tokens exposed |

### Workflow Details

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `ci.yml` | push to master | Build on Ubuntu |
| `WindowsInstall.yml` | manual dispatch | Windows MSI build + code signing |
| `LinuxInstall.yml` | manual dispatch | Linux AppImage build + GPG signing |
| `LinuxFlatpak.yml` | manual dispatch | Flatpak build |
| `CreateReleaseDraft.yml` | manual dispatch | Aggregate artifacts into GitHub release draft |

## Supply-Chain Risk

| Factor | Assessment | Evidence |
|--------|-----------|----------|
| Single maintainer risk | Yes | 112 of 119 commits in last 6 months by Jakub Melka |
| Bus factor | 1 | Effectively sole maintainer; few external contributors |
| Dependency update automation | Not configured | No Dependabot or Renovate |
| Lockfiles committed | No | vcpkg uses `vcpkg.json` manifest but no lockfile (`vcpkg.json` has no baseline pinning beyond `vcpkg-configuration.json`) |
| Build reproducibility | Unlikely | vcpkg fetches latest matching versions; Qt installed via action; no hash pinning |
| Package publishing workflow | Adequate | Signed MSI/AppImage via CI; manual dispatch required |

## Project Maturity

| Indicator | Value | Notes |
|-----------|-------|-------|
| Age | ~8 years | Copyright 2018-2025 |
| Last commit | 2026-05-09 | Very active |
| Contributors (6 months) | 5 | Dominated by single maintainer |
| Release cadence | Regular | Multiple releases visible |
| Archived/Deprecated | No | Actively developed |

## Maintenance Health

| Signal | Status | Notes |
|--------|--------|-------|
| Active development | Yes | Multiple commits per week |
| Issues triaged | Yes | Commits reference issue numbers regularly |
| Security issues addressed | Unknown | No SECURITY.md; no visible security advisories |
| Dependencies updated | Occasionally | vcpkg deps updated with new releases; no automation |
| Breaking changes managed | Unclear | No CHANGELOG or migration guides found |

## Risk Summary

| Risk Category | Level | Key Concerns |
|--------------|-------|--------------|
| Repository hygiene | Medium | Missing SECURITY.md, CONTRIBUTING.md, CODE_OF_CONDUCT |
| Security processes | High | No SAST, no dependency scanning, no enforced code review |
| Supply chain | High | Single maintainer, no lockfiles, no dependency automation, actions not SHA-pinned |
| Maintenance | Low | Actively maintained, issues triaged promptly |
| **Overall** | **Weak** | Strong maintenance signals offset by lack of security tooling and supply-chain controls |

## Unknowns

| Area | What Cannot Be Assessed | Why |
|------|------------------------|-----|
| Branch protection | Actual branch protection rules | Requires admin API access |
| Secret scanning | Whether GitHub secret scanning is active | Cannot verify from local clone |
| Past incidents | Whether supply-chain compromises occurred | No public advisories found; cannot confirm absence |
| Internal security reviews | Whether code undergoes security review | No public evidence |

## Recommended Actions

1. **Create `SECURITY.md`** — Document vulnerability reporting process and contact information
2. **Enable Dependabot or Renovate** — Automate dependency update PRs for vcpkg and GitHub Actions
3. **Pin GitHub Actions to SHA** — Replace `@v4` tags with full commit hashes to prevent tag-hijacking
4. **Add explicit `permissions:` to workflows** — Use least-privilege principle (e.g., `contents: read`)
5. **Configure CodeQL or similar SAST** — Add static analysis to CI pipeline
6. **Enable branch protection on `master`** — Require PR reviews and status checks before merge
7. **Commit vcpkg lockfile** — Pin exact dependency versions for reproducible builds
8. **Add `CONTRIBUTING.md`** — Document contribution process and code review expectations
