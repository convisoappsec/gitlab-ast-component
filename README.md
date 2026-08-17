<img src="assets/logo.png" alt="Conviso Application Security" width="96" align="right">

# Conviso AST — GitLab CI/CD component

Runs [Conviso AST](https://docs.convisoappsec.com/) in a GitLab CI job and sends
findings to Conviso Platform. One include covers SAST, SCA, IaC, SBOM, secret and
container scanning.

This is the GitLab counterpart of:

- [Conviso AST GitHub Action](https://github.com/marketplace/actions/conviso-ast)
- [Conviso AST Azure Pipelines task](https://marketplace.visualstudio.com/items?itemName=Conviso.convisoAstTask)

Source of truth is this GitHub repository. `include: component` needs a GitLab.com
copy of the same project (mirror). See [GitHub and GitLab](#github-and-gitlab).

## Quick start

In the GitLab project that should be scanned, add a **masked** CI/CD variable
`CONVISO_API_KEY`, then:

```yaml
include:
  - component: gitlab.com/convisoappsec/gitlab-ast-component/ast@1
    inputs:
      company_id: "YOUR_COMPANY_ID"
```

That is the whole integration. The component reads the branch from GitLab CI
and runs `conviso-ast` inside `convisoappsec/convisoast_v2`.

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `company_id` | **yes** | — | Company ID in Conviso Platform. |
| `stage` | no | `test` | Pipeline stage for the job. |
| `job_name` | no | `conviso-ast` | Change if you include the component twice. |
| `base_url` | no | `https://app.convisoappsec.com` | Dedicated / on-premise URL. |
| `asset_id` | no | *(auto)* | Pins the scan to one asset. |
| `scan_types` | no | *(all)* | Comma-separated: `sast`, `sca`, `iac`, `sbom`, `secret`, `container`. |
| `image_name` | no | — | Image for the `container` scanner. |
| `path` | no | `.` | Directory to scan. |
| `branch` | no | *(auto)* | Override the branch the CLI detects from GitLab CI. |
| `baseline_ref` | no | — | Diff against this branch (e.g. `main`). |
| `dry_run` | no | `false` | Scan without writing to Conviso Platform. |
| `image_repository` | no | `convisoappsec/convisoast_v2` | Image mirror. |
| `image_tag` | no | `latest` | Pin for reproducible builds. |

The API key is **not** an input. Store it as the CI/CD variable `CONVISO_API_KEY`
(masked). The job maps it to `CONVISO_APIKEY`, which the CLI requires.

## Examples

SAST + SCA only, on `main`:

```yaml
include:
  - component: gitlab.com/convisoappsec/gitlab-ast-component/ast@1
    inputs:
      company_id: $CONVISO_COMPANY_ID
      scan_types: sast,sca
      baseline_ref: main
```

Dry run (no write to the Platform):

```yaml
include:
  - component: gitlab.com/convisoappsec/gitlab-ast-component/ast@1
    inputs:
      company_id: $CONVISO_COMPANY_ID
      dry_run: true
```

## GitHub and GitLab

Developers push and tag **here on GitHub**. GitLab CI/CD components are addressed
as `gitlab.com/<group>/<project>/ast@version`, so a public GitLab.com copy is
required. GitLab pull-mirroring is Premium; this repo uses a GitHub Action
(`.github/workflows/mirror-to-gitlab.yml`) that pushes `main` and tags to GitLab.

A semver tag on GitHub (`1.0.0`) is mirrored, then GitLab CI creates a **Release**
from that tag — that is what makes the component show up in the
[CI/CD Catalog](https://gitlab.com/explore/catalog).

Setup:

1. This repository on GitHub (`convisoappsec/gitlab-ast-component`).
2. Public GitLab.com project `convisoappsec/gitlab-ast-component` with
   **CI/CD Catalog resource** enabled.
3. GitLab **Project Access Token** (`write_repository`) stored as the GitHub
   Actions secret `GITLAB_TOKEN`.
4. Tag on GitHub: `git tag 1.0.0 && git push origin 1.0.0`.

## What this is not

This component scans **the repository that runs the pipeline**. Centralized
scans after merge (one orchestrator project, many apps) are
[GitLab AST Orchestrator](https://docs.convisoappsec.com/integrations/gitlab-ast-orchestrator).

## License

Apache License 2.0. See [LICENSE](LICENSE).
