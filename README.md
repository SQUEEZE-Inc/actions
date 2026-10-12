# SQUEEZE-Inc/actions

Shared GitHub Actions for SQUEEZE repositories.

## safe-chain

Installs [Aikido Safe Chain](https://github.com/AikidoSec/safe-chain) on the runner. From the next step on, npm / npx / pnpm / yarn / bun / pip / uv / poetry:
- block known malware before download
- skip package versions published less than 48 hours ago

Tracked in SRE-306 (CI) and SRE-305 (developer machines).

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4        # setup-* first
    with:
      node-version-file: .nvmrc
  - uses: SQUEEZE-Inc/actions/safe-chain@<commit-sha>   # then Safe Chain
  - run: npm ci                         # then install
```

**Order matters.** `setup-node`, `setup-python`, `setup-uv` and `pnpm/action-setup` put their own binaries at the front of `PATH`. If one of them runs after this action, it hides the shims and the install is no longer checked. The action fails if a package manager is already ahead of the shims when it runs, but it cannot see setup steps that come later.

Not covered: `npm ci` / `pip install` inside `docker build` (the image does not use the runner's `PATH`).

### Inputs

| input | default | |
|---|---|---|
| `version` | `1.5.25` | Safe Chain release. Update together with `installer-sha256`. |
| `installer-sha256` | hash of `install-safe-chain.sh` 1.5.25 | Copy from the Safe Chain README for the release. |
| `minimum-package-age-hours` | Safe Chain default (48) | Raise or lower the minimum package age. |

### A version you need is younger than 48 hours

Set `SAFE_CHAIN_MINIMUM_PACKAGE_AGE_EXCLUSIONS` on the install step. Do not remove the action. Urgent security fixes are already exempted by Aikido.

### Updating Safe Chain

1. Take the new release tag and the `install-safe-chain.sh` checksum from the Safe Chain README.
2. Update both defaults in `safe-chain/action.yml` in a single PR.
3. Repositories with Dependabot `github-actions` updates get the new commit SHA as a PR. Update the others by hand.
