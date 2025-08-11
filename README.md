# BSI GitOps - Centralized CI/CD Workflows

This repository contains reusable GitHub Actions workflows for standardized CI/CD processes across BSI projects.

##  Architecture

The workflows are designed as reusable components that can be called from any repository, providing:
- **Consistency**: Standardized quality gates across all projects
- **Maintainability**: Centralized workflow management
- **Modularity**: Independent, focused workflows for specific concerns
- **Scalability**: Easy to extend and customize per project needs

##  Available Workflows

### 1. Static Code Analysis (`.github/workflows/static-analysis.yml`)
Performs comprehensive static code analysis including:
- .NET build validation
- StyleCop/analyzer enforcement  
- Unit test execution
- Code coverage reporting with thresholds

**Inputs:**
- `solution-path` (required): Path to .sln file
- `dotnet-version` (optional): .NET version (default: 9.0.x)
- `configuration` (optional): Build config (default: Release)
- `coverage-threshold` (optional): Min coverage % (default: 80)

### 2. Security Analysis (`.github/workflows/security-analysis.yml`)
Comprehensive security scanning including:
- CodeQL static analysis
- Trivy vulnerability scanning
- SARIF reporting integration

**Inputs:**
- `solution-path` (required): Path to .sln file
- `scan-path` (optional): Custom scan directory
- `dotnet-version` (optional): .NET version (default: 9.0.x)
- `languages` (optional): CodeQL languages (default: csharp)

### 3. AI Code Review (`.github/workflows/ai-code-review.yml`)
Automated code review using AI tools:
- GitHub Super-linter validation
- ChatGPT-powered code analysis
- Custom business rule validation

**Inputs:**
- `validate-languages` (optional): Languages to validate
- `openai-model` (optional): AI model (default: gpt-4)
- `custom-prompt` (optional): Custom review prompt

**Secrets Required:**
- `OPENAI_API_KEY`: OpenAI API key for ChatGPT integration

### 4. Docker Validation (`.github/workflows/docker-validation.yml`)
Container build and security validation:
- Multi-platform Docker builds
- Container security scanning
- Registry integration support

**Inputs:**
- `dockerfile-path` (required): Path to Dockerfile
- `context-path` (optional): Build context directory
- `image-name` (required): Docker image name
- `registry` (optional): Container registry URL
- `push-image` (optional): Whether to push image

**Secrets (optional):**
- `REGISTRY_USERNAME`: Registry username
- `REGISTRY_PASSWORD`: Registry password

### 5. Review Enforcement (`.github/workflows/review-enforcement.yml`)
Human review requirement validation:
- Approval count verification
- Code owner review enforcement
- Change request validation

**Inputs:**
- `required-approvals` (optional): Min approvals (default: 1)
- `require-code-owner-review` (optional): Require code owner (default: true)

### 6. Gated Build Orchestrator (`.github/workflows/gated-build-orchestrator.yml`)
Coordinates all workflows and provides final status:
- Multi-workflow status monitoring
- Configurable check requirements
- Final status aggregation and reporting

**Inputs:**
- `required-checks` (optional): Comma-separated check names
- `max-wait-minutes` (optional): Max wait time (default: 15)
- `check-interval-seconds` (optional): Check interval (default: 30)

##  Usage

### Basic Implementation

Create a workflow in your service repository (`.github/workflows/gated-build.yml`):

```yaml
name: Gated Build Pipeline

on:
  pull_request:
    branches: [main, develop]
    types: [opened, synchronize, reopened, ready_for_review]
  pull_request_review:
    types: [submitted]

jobs:
  static-analysis:
    if: ${{ !github.event.pull_request.draft }}
    uses: BrainStorm-bsi/bsi-gitops/.github/workflows/static-analysis.yml@main
    with:
      solution-path: 'YourProject/YourProject.sln'
      coverage-threshold: 85

  security-analysis:
    if: ${{ !github.event.pull_request.draft }}
    uses: BrainStorm-bsi/bsi-gitops/.github/workflows/security-analysis.yml@main
    with:
      solution-path: 'YourProject/YourProject.sln'

  ai-code-review:
    if: ${{ !github.event.pull_request.draft }}
    uses: BrainStorm-bsi/bsi-gitops/.github/workflows/ai-code-review.yml@main
    secrets:
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}

  docker-validation:
    if: ${{ !github.event.pull_request.draft }}
    uses: BrainStorm-bsi/bsi-gitops/.github/workflows/docker-validation.yml@main
    with:
      dockerfile-path: 'YourProject/Dockerfile'
      image-name: 'your-org/your-service'

  review-enforcement:
    if: ${{ !github.event.pull_request.draft }}
    uses: BrainStorm-bsi/bsi-gitops/.github/workflows/review-enforcement.yml@main
    with:
      required-approvals: 2

  orchestrator:
    if: ${{ !github.event.pull_request.draft }}
    needs: [static-analysis, security-analysis, ai-code-review, docker-validation, review-enforcement]
    uses: BrainStorm-bsi/bsi-gitops/.github/workflows/gated-build-orchestrator.yml@main
```

### Advanced Configuration

You can customize each workflow with specific parameters:

```yaml
  static-analysis:
    uses: BrainStorm-bsi/bsi-gitops/.github/workflows/static-analysis.yml@main
    with:
      solution-path: 'src/MyService.sln'
      dotnet-version: '8.0.x'
      configuration: 'Debug'
      coverage-threshold: 90

  ai-code-review:
    uses: BrainStorm-bsi/bsi-gitops/.github/workflows/ai-code-review.yml@main
    with:
      validate-languages: 'CSHARP,DOCKERFILE'
      openai-model: 'gpt-4-turbo'
      custom-prompt: |
        Focus on microservice patterns and cloud-native best practices.
        Pay special attention to:
        1. Service boundaries and API design
        2. Error handling and resilience patterns
        3. Security and authentication
    secrets:
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

##  Repository Setup

### Required Secrets

For full functionality, configure these secrets in your repository:

1. **OPENAI_API_KEY**: Required for AI code review
   - Get from: https://platform.openai.com/api-keys
   - Scope: Repository or Organization level

2. **REGISTRY_USERNAME/REGISTRY_PASSWORD**: Optional for Docker workflows
   - For pushing images to container registries

### Branch Protection Rules

Configure branch protection to require all workflow checks:

```json
{
  "required_status_checks": {
    "strict": true,
    "contexts": [
      "Static Code Analysis",
      "Security Analysis",
      "AI Code Review", 
      "Docker Build Validation",
      "Review Requirements Check",
      "Gated Build Result"
    ]
  },
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": true
  }
}
```

##  Benefits

### For Developers
- **Consistent Experience**: Same quality gates across all projects
- **Faster Feedback**: Parallel execution of all checks
- **Clear Status**: Individual check results with detailed reporting
- **AI Assistance**: Automated code review suggestions

### For Teams
- **Standardization**: Consistent quality standards organization-wide
- **Governance**: Centralized policy enforcement
- **Visibility**: Clear quality metrics and trends
- **Compliance**: Automated security and compliance checking

### For Operations
- **Maintainability**: Single point of truth for CI/CD logic
- **Scalability**: Easy to onboard new projects
- **Monitoring**: Centralized workflow performance metrics
- **Updates**: Version-controlled workflow improvements

##  Versioning Strategy

- **main**: Latest stable version (recommended for production)
- **develop**: Development version with latest features
- **v1.x.x**: Tagged releases for specific versions

Example usage with version pinning:
```yaml
uses: BrainStorm-bsi/bsi-gitops/.github/workflows/static-analysis.yml@v1.2.0
```

##  Contributing

1. Fork this repository
2. Create a feature branch
3. Make your changes
4. Test with a sample project
5. Submit a pull request

##  Support

For questions or issues:
- Create an issue in this repository
- Contact the BSI DevOps team
- Check existing documentation and examples

---

**BSI GitOps** - Empowering consistent, scalable, and secure software delivery