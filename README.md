# native-shared-workflows

Reusable GitHub Actions workflows and shared security configurations for Native Insurance services.

## Workflows

| Workflow | Purpose |
|---|---|
| `security-audit.yml` | TruffleHog + Bandit + Semgrep + image scanning + SBOMs |
| `docker-build-push.yml` | WIF auth + Docker build + push to Artifact Registry |
| `deploy-cloud-run.yml` | WIF auth + Cloud Run deploy |
| `scheduled-cve-check.yml` | Pull latest images, Trivy scan, Slack notification |

## Shared Configurations

| File | Purpose |
|---|---|
| `bandit.yaml` | Python SAST exclusions and skips |
| `.semgrep/shared-rules.yaml` | Common Python security rules (JWT, SQL injection, deserialization, cookies, CORS, open redirect) |
| `.pre-commit/base.yaml` | Base pre-commit hooks to extend in service repos |

## Usage

### security-audit

```yaml
jobs:
  security:
    uses: NativeInsurance/native-shared-workflows/.github/workflows/security-audit.yml@main
    with:
      python-version: '3.12'
      bandit-target: src/
      images: '[{"dockerfile":"Dockerfile.prod","name":"my-service"}]'
    secrets: inherit
```

### docker-build-push

```yaml
jobs:
  build:
    needs: [tests, security]
    uses: NativeInsurance/native-shared-workflows/.github/workflows/docker-build-push.yml@main
    with:
      project: my-gcp-project
      images: '[{"dockerfile":"Dockerfile.prod","name":"my-service","build_args":"KEY=VAL"}]'
    secrets:
      WIF_PROVIDER: ${{ secrets.WIF_PROVIDER }}
      WIF_SA: ${{ secrets.WIF_SA }}
```

### deploy-cloud-run

```yaml
jobs:
  deploy:
    needs: build
    uses: NativeInsurance/native-shared-workflows/.github/workflows/deploy-cloud-run.yml@main
    with:
      project: my-gcp-project
      service: my-service
      image: europe-west2-docker.pkg.dev/my-project/my-repo/my-service:${{ github.sha }}
    secrets:
      WIF_PROVIDER: ${{ secrets.WIF_PROVIDER }}
      WIF_SA: ${{ secrets.WIF_SA }}
```

### scheduled-cve-check

In your service repo, create a workflow triggered on a schedule:

```yaml
name: Scheduled CVE Scan

on:
  schedule:
    - cron: '0 6 * * *'
  workflow_dispatch:

jobs:
  scan:
    uses: NativeInsurance/native-shared-workflows/.github/workflows/scheduled-cve-check.yml@main
    with:
      registry: europe-west2-docker.pkg.dev/my-project/my-repo
      images: '[{"name":"my-service"},{"name":"my-frontend"}]'
    secrets:
      WIF_PROVIDER: ${{ secrets.WIF_PROVIDER }}
      WIF_SA: ${{ secrets.WIF_SA }}
      SLACK_WEBHOOK_URL: ${{ secrets.CVE_SLACK_WEBHOOK_URL }}
```

## Versioning

Reference workflows by git ref:

- `@main` — latest stable
- `@v1` — major version tag (to be created)

## Contributing

When adding new shared rules or updating tool versions:

1. Test on at least one consuming repo via branch reference
2. Update this README if the change adds new inputs or changes behavior
3. Tag a new version after merging if breaking changes are made
