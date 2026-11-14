# templates

Copier templates and reusable GitHub Actions workflows for the owner's projects.

## Templates

| Template | What it generates |
|---|---|
| `static-site` | Privacy notice and support pages, Spanish and English, plain HTML, with a Cloudflare Pages deploy caller |
| `expo-eas` | Into an existing Expo app: `eas.json` (development, preview, production; remote app versions; IDs-free Android submit) and a CI caller for `ci-node` |

Generate (Copier 9.19.1 or later; no `--trust` needed):

```sh
copier copy --vcs-ref <tag> gh:RodG1t/templates <destination>
```

Copier asks which template to use. Update a generated project with `copier update`. Generate from GitHub at a tag, not from a local copy: generated callers pin the templates commit hash, which must exist on GitHub.

## Reusable workflows

| Workflow | Does | Caller supplies |
|---|---|---|
| `pages-deploy.yml` | Refuses unwritten template slots, then deploys a folder to Cloudflare Pages (wrangler-action v4.1.3, Wrangler 4.155.0) | inputs `project_name`, `directory`; secrets `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID` |
| `gitleaks.yml` | Gitleaks 8.30.1 (checksum-verified release binary, no third-party action) over the full history. Every repo calls it as its own job | nothing |
| `ci-node.yml` | `npm ci`, `npm run typecheck`, `npm run lint`, `npm test` | input `node_version`; scripts `typecheck`, `lint`, `test`; committed `package-lock.json` |
| `tofu-plan-apply.yml` | OpenTofu from `.opentofu-version` (zip hash checked), `fmt -check`, `init -lockfile=readonly`, `validate`, plan to the job summary; applies that saved plan only with `apply: true` on `main`. One run at a time per repo; waiting runs queue in order (`queue: max`) | inputs `tofu_zip_sha256`, `apply`, `working_directory`; secrets `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_ENDPOINT_URL`, `STATE_PASSPHRASE`, optional `CLOUDFLARE_API_TOKEN`, optional `CLOUDFLARE_ACCOUNT_ID` (exported as `TF_VAR_cloudflare_account_id` only when set) |

A caller for `tofu-plan-apply` plans on pull requests and pushes to `main`, and applies only from a manual run:

```yaml
on:
  pull_request:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      apply: { type: boolean, default: false }
jobs:
  gitleaks:
    uses: RodG1t/templates/.github/workflows/gitleaks.yml@<templates commit SHA>
  tofu:
    needs: gitleaks
    uses: RodG1t/templates/.github/workflows/tofu-plan-apply.yml@<templates commit SHA>
    with:
      tofu_zip_sha256: <sha256 of tofu_<version>_linux_amd64.zip>
      apply: ${{ github.event_name == 'workflow_dispatch' && inputs.apply }}
    secrets:
      R2_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
      R2_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
      R2_ENDPOINT_URL: ${{ secrets.R2_ENDPOINT_URL }}
      STATE_PASSPHRASE: ${{ secrets.STATE_PASSPHRASE }}
      CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
      CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

`CLOUDFLARE_ACCOUNT_ID` is for root modules that declare `variable "cloudflare_account_id"`; leave it out otherwise.

Every action is pinned to a full commit SHA.

## License

No license is granted. The repository is public so the owner's own repositories can use it; all rights are reserved.
