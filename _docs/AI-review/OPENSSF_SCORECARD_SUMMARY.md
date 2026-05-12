# OpenSSF Scorecard Summary

## Repository
`github.com/JakubMelka/PDF4QT`

## Scan Details

| Field | Value |
|-------|-------|
| Scan date | 2026-05-11 |
| Scorecard version | Not run (tool unavailable) |
| Commit scanned | e8d121c (HEAD) |
| Aggregate score | N/A — manual estimate below |

## Estimated Check Results (Manual Assessment)

| Check | Estimated Score | Risk | Rationale |
|-------|----------------|------|-----------|
| Binary-Artifacts | ~8/10 | Low | No checked-in binaries found in source (assets are icons/SVGs) |
| Branch-Protection | ~2/10 | High | Direct pushes to master observed; likely no branch protection |
| CI-Tests | ~5/10 | Medium | CI builds on push to master but no PR-triggered test gate visible |
| CII-Best-Practices | 0/10 | High | No OpenSSF Best Practices badge |
| Code-Review | ~3/10 | High | Most commits pushed directly; occasional PRs from external contributors |
| Contributors | ~3/10 | Medium | Essentially single-maintainer; few external contributors |
| Dangerous-Workflow | 10/10 | Low | No dangerous patterns (pull_request_target, script injection) found |
| Dependency-Update-Tool | 0/10 | High | No Dependabot or Renovate configured |
| Fuzzing | 0/10 | High | No fuzz testing infrastructure found |
| License | 10/10 | Low | MIT licence clearly present |
| Maintained | 10/10 | Low | Very active development (multiple commits per week) |
| Packaging | ~7/10 | Low | Published as AppImage, MSI, Flatpak |
| Pinned-Dependencies | ~2/10 | High | GitHub Actions use tags, not SHA; vcpkg has no lockfile |
| SAST | 0/10 | High | No static analysis tools in CI |
| SBOM | 0/10 | High | No SBOM generation or publication |
| Security-Policy | 0/10 | High | No SECURITY.md present |
| Signed-Releases | ~7/10 | Low | Windows MSI code-signed; Linux AppImage GPG-signed |
| Token-Permissions | ~3/10 | Medium | No explicit permissions in workflows; defaults used |
| Vulnerabilities | N/A | Unknown | Cannot assess without Scorecard/OSV integration |
| Webhooks | N/A | Unknown | Cannot assess from local clone |

## Key Findings

### Critical Issues (Estimated Score 0-3)
- **No SECURITY.md** — No documented vulnerability disclosure process
- **No dependency update automation** — Dependabot/Renovate not configured
- **No SAST** — No CodeQL, Semgrep, or similar in CI
- **No fuzzing** — PDF parsing is a high-risk surface; fuzz testing absent
- **No SBOM** — No software bill of materials generated
- **No CII Best Practices badge** — Project hasn't pursued OpenSSF badge
- **Weak branch protection** — Direct pushes to master observed
- **Dependencies not pinned** — Actions use tags; vcpkg lacks lockfile

### Warnings (Estimated Score 4-6)
- **Limited CI testing** — Build verification but no PR gate
- **Low contributor diversity** — Single maintainer dominates

### Passing (Estimated Score 7-10)
- **Actively maintained** — Frequent commits, issues addressed
- **No dangerous workflows** — CI patterns are safe
- **Licence present** — MIT, clearly documented
- **Signed releases** — Both Windows (code signing) and Linux (GPG) releases signed
- **No binary artifacts** — Clean source tree

## Recommendations Based on Assessment

| Priority | Check | Current Est. | Recommendation |
|----------|-------|--------------|----------------|
| Critical | Security-Policy | 0 | Create SECURITY.md with disclosure process |
| Critical | SAST | 0 | Add CodeQL workflow for C++ analysis |
| Critical | Fuzzing | 0 | Implement fuzzing for PDF parser (OSS-Fuzz or local AFL++) |
| High | Dependency-Update-Tool | 0 | Configure Dependabot for GitHub Actions and vcpkg |
| High | Pinned-Dependencies | 2 | Pin all actions to SHA; commit vcpkg lockfile |
| High | Branch-Protection | 2 | Enable branch protection with required reviews |
| Medium | Token-Permissions | 3 | Add `permissions: read-all` default with targeted write scopes |
| Medium | SBOM | 0 | Add CycloneDX or SPDX SBOM generation to release workflow |
| Low | CII-Best-Practices | 0 | Apply for OpenSSF Best Practices badge |

## Limitations

- Scorecard CLI was not available; all scores are manual estimates
- Branch protection settings cannot be verified without repository admin API access
- Vulnerability status cannot be assessed without OSV/Scorecard integration
- Internal security practices not visible from public repository alone
