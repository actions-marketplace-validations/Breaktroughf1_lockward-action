# Lockward Verify (GitHub Action)

Fail CI when the skills, MCP servers, hooks, rules files or agent settings your AI coding agents load drift from `lockward.lock`, or break your policy. Works for Claude Code, Cursor, Codex, Gemini CLI, Windsurf and GitHub Copilot. Offline, no account, no LLM in the decision path.

[Lockward](https://github.com/Breaktroughf1/lockward) is the tool; this repository is its GitHub Action, listed on the [GitHub Marketplace](https://github.com/marketplace/actions/lockward-verify).

## Usage

Create the lockfile once with the CLI (`lockward init && lockward lock`, then commit `lockward.lock`), then add:

```yaml
name: lockward
on: [pull_request, push]
permissions:
  contents: read
  security-events: write   # only needed for SARIF upload
jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<sha>
        with:
          persist-credentials: false
      - uses: Breaktroughf1/lockward-action@<commit sha of v0.2.3>   # v0.2.3
        with:
          version: 0.2.3
```

Pin the action to a full commit SHA, as you would any third-party action. The job fails with Lockward's exit code when assets drift (1) or a policy rule fails (2), uploads SARIF so each finding appears in code scanning, and writes the result to the job summary.

## Inputs

| Input | Default | Description |
|---|---|---|
| `version` | `0.2.3` | Lockward release to install, without the leading `v` |
| `args` | `verify --frozen` | Arguments passed to `lockward` |
| `sarif` | `true` | For `verify`, also upload a SARIF report to code scanning (needs `security-events: write`) |
| `working-directory` | `.` | Directory to run in |

## Outputs

| Output | Description |
|---|---|
| `exit-code` | `0` clean, `1` drift, `2` policy violation, `3` usage or configuration error, `4` internal error |

## What it does

1. Downloads `lockward_<version>_<os>_<arch>` from the [Lockward release](https://github.com/Breaktroughf1/lockward/releases), with `checksums.txt` and its Sigstore bundle.
2. Verifies the bundle with `cosign verify-blob`, requiring the certificate identity `https://github.com/Breaktroughf1/lockward/.github/workflows/release.yml@refs/tags/v<version>` and the GitHub Actions OIDC issuer.
3. Verifies the archive's SHA-256 against the signed `checksums.txt`. The binary runs only if both checks pass.
4. Runs `lockward` with `args`. Output from the repository being checked is printed with GitHub workflow commands suspended, so it cannot inject `::error::`-style commands, and the job summary's code fence is sized so content cannot break out of it.
5. Uploads SARIF (for `verify`), then fails the step with Lockward's exit code.

Supported runners: Linux, macOS and Windows on x64 and arm64. Every change is tested against the published release on all of them from the [Lockward repository](https://github.com/Breaktroughf1/lockward/actions/workflows/action-test.yml).

## Versions

Tags match Lockward releases: `v0.2.3` of this action installs Lockward 0.2.3 by default. There are no moving major-version tags; pin a commit SHA.

## Security

Report vulnerabilities privately as described in [Lockward's security policy](https://github.com/Breaktroughf1/lockward/security/policy).

## Licence

Apache-2.0. See [LICENSE](LICENSE).
