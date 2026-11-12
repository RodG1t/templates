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
| `ci-node.yml` | Gitleaks 8.30.1 (checksum-verified binary) over the full history; `npm ci`, `npm run typecheck`, `npm run lint`, `npm test` | input `node_version`; scripts `typecheck`, `lint`, `test`; committed `package-lock.json` |

Every action is pinned to a full commit SHA.

## License

No license is granted. The repository is public so the owner's own repositories can use it; all rights are reserved.
