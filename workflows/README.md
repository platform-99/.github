# GitHub Actions Workflows

This directory contains GitHub Actions workflows for the platform-99 repository.

## Structure

### Main Workflows

- **`workload-a-ci.yml`** - Main CI/CD pipeline orchestrator that calls reusable workflows

### Reusable Workflows

These workflows can be called from other workflows or used independently:

1. **`reusable-test-lint.yml`** - Test and lint Go applications
2. **`reusable-build-docker.yml`** - Build and push Docker images
3. **`reusable-security-scan.yml`** - Security vulnerability scanning with Trivy
4. **`reusable-helm-verify-publish.yml`** - Verify and publish Helm charts

## Pipeline Flow

```
┌─────────────────┐
│  Test & Lint    │  ← reusable-test-lint.yml
└────────┬────────┘
         │
┌────────▼────────┐
│  Build Docker   │  ← reusable-build-docker.yml
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
┌───▼───┐ ┌──▼──────────────┐
│Security│ │ Helm Chart      │
│ Scan   │ │ Verify/Publish  │
└───────┘ └──────────────────┘
```

## Usage

### Running the Full Pipeline

The main pipeline runs automatically on:
- Push to `main` or `develop` branches
- Pull requests to `main` or `develop`
- Manual trigger via `workflow_dispatch`

### Using Individual Reusable Workflows

You can call reusable workflows from other workflows:

```yaml
jobs:
  test:
    uses: ./.github/workflows/reusable-test-lint.yml
    with:
      working_directory: './workload-a'
      go_version: '1.21'
```

## Workflow Details

### 1. Test and Lint (`reusable-test-lint.yml`)

**What it does:**
- Verifies Go dependencies
- Runs `go vet` for static analysis
- Checks code formatting with `go fmt`
- Runs tests with race detection
- Uploads coverage to Codecov (optional)

**Inputs:**
- `working_directory` - Path to Go application (default: `./workload-a`)
- `go_version` - Go version (default: `1.21`)
- `upload_coverage` - Upload coverage (default: `true`)

### 2. Build Docker (`reusable-build-docker.yml`)

**What it does:**
- Sets up Docker Buildx
- Builds multi-platform Docker images
- Pushes to container registry (optional)
- Generates semantic version tags

**Inputs:**
- `context_path` - Docker build context (required)
- `registry` - Container registry URL (required)
- `image_name` - Image name (required)
- `push_image` - Push to registry (default: `true`)
- `platforms` - Build platforms (default: `linux/amd64,linux/arm64`)
- `include_semver_tags` - Include semantic version tags (default: `true`)

**Outputs:**
- `image_digest` - Image digest
- `image_tags` - Generated tags

### 3. Security Scan (`reusable-security-scan.yml`)

**What it does:**
- Scans container images with Trivy
- Focuses on CRITICAL and HIGH severity vulnerabilities
- Uploads results to GitHub Security tab

**Inputs:**
- `image_ref` - Image to scan (required)
- `scan_format` - Output format (default: `sarif`)
- `upload_to_github` - Upload to GitHub Security (default: `true`)

### 4. Helm Chart Verify and Publish (`reusable-helm-verify-publish.yml`)

**What it does:**
- Lints Helm chart
- Validates chart templates
- Builds chart dependencies (if any)
- Packages chart as `.tgz`
- Publishes to OCI registry (optional)

**Inputs:**
- `chart_path` - Path to chart directory (required)
- `chart_name` - Chart name (optional, defaults to directory name)
- `registry` - OCI registry URL (optional)
- `publish_chart` - Publish to registry (default: `false`)
- `helm_version` - Helm version (default: `3.12.0`)

**Outputs:**
- `chart_version` - Chart version
- `chart_name` - Chart name

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



