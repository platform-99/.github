# GitHub Actions Workflows

This directory contains GitHub Actions workflows for the platform-99 repository.

## Structure

### Example Workflows

- **`examples/workload-a-ci.yml`** - Example CI/CD pipeline orchestrator that demonstrates how to call reusable workflows

### Reusable Workflows

These workflows can be called from other workflows in the same repository or from other repositories:

**Application Development:**
1. **`golang-test-lint.yml`** - Test and lint Go applications
2. **`docker-build-and-push.yml`** - Build and push Docker images (multi-platform)
3. **`docker-build-and-push-jfrog.yml`** - Build and push Docker images with JFrog integration
4. **`security-scan-source.yml`** - Security vulnerability scanning of source code with Trivy

**Helm Charts:**
5. **`helm-validate.yml`** - Validate Helm charts (lint, template validation)
6. **`helm-publish-oci.yml`** - Publish Helm charts to OCI registry
7. **`helm-publish-github-pages.yml`** - Publish Helm charts to GitHub Pages

**Terraform:**
8. **`terraform-module.yml`** - Terraform module validation and documentation
9. **`terraform-plan.yml`** - Terraform plan generation
10. **`terraform-apply.yml`** - Terraform apply operations
11. **`terraform-apply-with-approval.yml`** - Terraform apply with manual approval
12. **`terraform-drift.yml`** - Detect Terraform configuration drift

**GitOps:**
13. **`gitops-validate.yml`** - GitOps configuration validation

## Example Pipeline Flow

```
┌─────────────────┐
│  Test & Lint    │  ← golang-test-lint.yml
└────────┬────────┘
         │
┌────────▼────────┐
│  Build Docker   │  ← docker-build-and-push.yml
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
┌───▼───┐ ┌──▼──────────────┐
│Security│ │ Helm Chart      │
│ Scan   │ │ Verify/Publish  │
│ Source │ │ helm-publish-   │
│        │ │   oci.yml       │
└───────┘ └──────────────────┘
```

## Usage

### Running the Full Pipeline

The main pipeline runs automatically on:
- Push to `main` or `develop` branches
- Pull requests to `main` or `develop`
- Manual trigger via `workflow_dispatch`

### Using Individual Reusable Workflows

You can call reusable workflows from other workflows in the same repository:

```yaml
jobs:
  test:
    uses: ./.github/workflows/golang-test-lint.yml
    with:
      working_directory: './workload-a'
      go_version: '1.21'
```

#### Cross-Repository Usage

These workflows can also be called from other repositories using the `owner/repo/.github/workflows/file.yml@ref` format:

```yaml
jobs:
  test:
    uses: michaelpatsula/platform-99/.github/workflows/golang-test-lint.yml@main
    with:
      working_directory: './my-app'
      go_version: '1.21'
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

**Note:** When calling from another repository:
- Use a specific branch/tag/commit SHA (e.g., `@main`, `@v1.0.0`, `@abc123`) for stability
- Ensure the calling repository has access to the workflow repository (public repos work automatically; private repos require proper permissions)
- Pass all required secrets from the calling repository

## Workflow Details

### 1. Test and Lint (`golang-test-lint.yml`)

**What it does:**
- Verifies Go dependencies
- Runs `go vet` for static analysis
- Checks code formatting with `go fmt`
- Runs tests with race detection
- Uploads coverage to Codecov (optional)

**Inputs:**
- `working_directory` - Path to Go application (default: `./workload-a`)
- `go_version` - Go version (default: `1.21`)
- `upload_coverage_to_codecov` - Upload coverage to Codecov (default: `true`)
- `generate_coverage_report` - Generate coverage report (text and HTML) (default: `false`)

### 2. Build Docker (`docker-build-and-push.yml`)

**What it does:**
- Sets up Docker Buildx
- Builds multi-platform Docker images
- Pushes to container registry (optional)
- Generates semantic version tags

**Inputs:**
- `context_path` - Docker build context (required)
- `registry` - Container registry URL (required)
- `image_name` - Image name without registry (required)
- `push_image` - Push to registry (default: `true`)
- `platforms` - Build platforms, comma-separated (default: `linux/amd64,linux/arm64`)
- `include_semver_tags` - Include semantic version tags (default: `true`)
- `include_sha_tags` - Include SHA tags (default: `true`)
- `include_branch_tags` - Include branch tags (default: `true`)
- `include_pr_tags` - Include PR tags (default: `true`)
- `scan_image` - Scan image with Trivy (default: `true`)
- `trivy_exit_code` - Trivy exit code: `0` (don't fail) or `1` (fail on vulnerabilities) (default: `1`)
- `skip_if_semver_exists` - Skip pushing if semver tags already exist (default: `false`)

**Secrets:**
- `REGISTRY_USERNAME` - Registry username (optional)
- `REGISTRY_PASSWORD` - Registry password/token (optional)

**Outputs:**
- `image_digest` - Image digest
- `image_tags` - Generated tags

### 3. Security Scan Source Code (`security-scan-source.yml`)

**What it does:**
- Scans source code/filesystem with Trivy
- Detects vulnerabilities in dependencies and configuration files
- Focuses on CRITICAL and HIGH severity vulnerabilities
- Uploads results to GitHub Security tab

**Inputs:**
- `scan_path` - Path to scan (directory or file) (default: `.`)
- `scan_type` - Type of scan: `fs` (filesystem) or `repo` (repository) (default: `fs`)
- `severity` - Minimum severity level (default: `CRITICAL,HIGH`)
- `upload_to_github` - Upload results to GitHub Security (default: `true`)

### 4. Helm Chart Validate (`helm-validate.yml`)

**What it does:**
- Lints Helm chart
- Validates chart templates
- Builds chart dependencies (if enabled)
- Validates against values files

**Inputs:**
- `chart_path` - Path to chart directory (required)
- `chart_name` - Chart name (optional, defaults to directory name)
- `helm_version` - Helm version (default: `3.12.0`)
- `strict_lint` - Run helm lint with --strict flag (default: `false`)
- `validate_dependencies` - Validate and build chart dependencies (default: `true`)
- `values_files` - Comma-separated list of values files to validate against

### 5. Helm Chart Publish OCI (`helm-publish-oci.yml`)

**What it does:**
- Lints and validates Helm chart
- Packages chart as `.tgz`
- Publishes to OCI registry (optional)
- Supports skipping if chart version already exists

**Inputs:**
- `chart_path` - Path to chart directory (required)
- `chart_name` - Chart name (optional, defaults to directory name)
- `registry_namespace` - Registry namespace for charts (optional)
- `registry` - OCI registry URL (optional)
- `registry_owner` - Registry owner/organization (required for GHCR)
- `publish_chart` - Publish to registry (default: `false`)
- `skip_if_exists` - Skip publishing if chart version already exists (default: `true`)
- `helm_version` - Helm version (default: `3.12.0`)

**Secrets:**
- `REGISTRY_USERNAME` - Registry username (optional)
- `REGISTRY_PASSWORD` - Registry password/token (optional)

**Outputs:**
- `chart_version` - Chart version
- `chart_name` - Chart name

### 6. Terraform Module (`terraform-module.yml`)

**What it does:**
- Validates Terraform module
- Runs terraform fmt check
- Runs terraform validate
- Generates terraform-docs
- Validates example configurations

**Inputs:**
- `working_directory` - Working directory (default: `.`)
- `terraform_version` - Terraform version (default: `1.9.5`)
- `examples` - JSON array of example directories to validate
- `examples_base_path` - Base path for examples (default: `examples`)
- `run_format_check` - Run terraform fmt check (default: `true`)
- `run_validate` - Run terraform validate (default: `true`)
- `run_docs` - Generate terraform-docs and push to PR (default: `true`)

### 7. Terraform Plan (`terraform-plan.yml`)

**What it does:**
- Generates Terraform plan
- Supports multiple environments
- Detects changes per environment
- Outputs plan files as artifacts

**Inputs:**
- `environment_folders` - Space-separated list of environment folder names (required)
- `terraform_version` - Terraform version (default: `1.6.0`)
- `working_directory` - Working directory (default: `.`)
- `base_ref` - Base ref for change detection
- `head_ref` - Head ref for change detection
- `selected_environment` - Selected environment for workflow_dispatch (optional)

### 8. Terraform Apply (`terraform-apply.yml`)

**What it does:**
- Applies Terraform configurations
- Supports multiple environments
- Azure Key Vault secret injection
- OIDC authentication

**Inputs:**
- `environment_folders` - Space-separated list of environment folder names (required)
- `terraform_version` - Terraform version (default: `1.6.0`)
- `working_directory` - Working directory (default: `.`)
- `base_ref` - Base ref for change detection
- `head_ref` - Head ref for change detection
- `selected_environment` - Selected environment for workflow_dispatch (optional)
- `kv_names` - Space-separated list of Azure Key Vault names

**Secrets:**
- `AZURE_CLIENT_ID` - Azure client ID (required)
- `AZURE_TENANT_ID` - Azure tenant ID (required)
- `AZURE_SUBSCRIPTION_ID` - Azure subscription ID (required)

### 9. GitOps Validate (`gitops-validate.yml`)

**What it does:**
- Validates YAML syntax
- Validates Helm chart references
- Checks chart availability in registry
- Supports private chart repositories

**Inputs:**
- `chart_repo_url` - Helm chart repository URL (OCI or HTTP) (required)
- `chart_name` - Helm chart name (required)
- `chart_version` - Helm chart version (required)
- `private_chart_repo` - Whether chart repository is private (default: `false`)
- `environments_path` - Path to environments directory (default: `environments`)
- `helm_version` - Helm version (default: `3.12.0`)

**Secrets:**
- `HELM_REGISTRY_USERNAME` - Registry username for private repos (optional)
- `HELM_REGISTRY_PASSWORD` - Registry password/token for private repos (optional)

## Benefits of Modular Design

✅ **Reusability** - Use workflows across multiple projects  
✅ **Maintainability** - Update workflows in one place  
✅ **Testability** - Test individual components independently  
✅ **Flexibility** - Compose pipelines as needed  
✅ **Clarity** - Each workflow has a single, clear purpose  

## Customization

### Adding New Reusable Workflows

1. Create a new workflow file with `workflow_call` trigger
2. Define inputs and secrets
3. Implement the job logic
4. Document in this README

### Modifying Existing Workflows

1. Update the reusable workflow file
2. All calling workflows automatically use the new version
3. Test thoroughly before merging

## Troubleshooting

### Workflow Not Triggering

- Check path filters in the workflow `on:` section
- Verify branch names match
- Ensure files changed match the path patterns

### Reusable Workflow Not Found

- Verify the workflow file exists in `.github/workflows/`
- Check the path in the `uses:` statement
- Ensure the workflow has `workflow_call` trigger

### Secrets Not Available

- Verify secrets are passed in the `secrets:` section
- Check repository secrets are configured
- Ensure secrets are marked as required if needed



