# Mellow - Android CI/CD Implementation

This repository demonstrates a **simple and focused CI/CD pipeline** for Android development using GitHub Actions, designed for learning and demonstration purposes. It showcases DevOps practices and automated workflows while keeping complexity manageable.

## 🏗️ CI/CD Architecture

### Workflow Structure

The CI/CD pipeline is built using a **reusable workflow pattern** for modularity and maintainability:

```
android-ci-cd.yml (Main Orchestrator)
├── android-ci.yml (Continuous Integration)
│   ├── android-unit-tests.yml (Unit Testing)
│   └── android-instrumented-tests.yml (UI/Integration Tests - Available but commented out)
└── android-cd.yml (Continuous Deployment)
```

### Workflow Files

| File | Purpose | Triggers |
|------|---------|----------|
| `android-ci-cd.yml` | Main orchestrator workflow | `push`, `pull_request` |
| `android-ci.yml` | Calls unit tests | `workflow_call` |
| `android-cd.yml` | Builds signed APKs | `workflow_call` |
| `android-unit-tests.yml` | Executes unit tests | `workflow_call` |
| `android-instrumented-tests.yml` | UI/Integration tests | `workflow_call`, `workflow_dispatch` |

## 🔄 Pipeline Flow

### Pull Request Flow
```
PR Created/Updated → CI Pipeline → Unit Tests → ✅ Success/❌ Failure
```

### Master Branch Flow
```
Push to Master → CI Pipeline → Unit Tests → ✅ Success → CD Pipeline → Build Signed APK
```

> **Note**: Instrumented tests are available in `android-instrumented-tests.yml` but are commented out in the CI pipeline to optimize Standard GitHub-hosted runners usage. They can be run manually or enabled for production environments on Larger GitHub-hosted runners
 or self-hosted runners.

### Key Features

- **Sequential Execution**: CD only runs after successful CI
- **Branch Protection**: CD restricted to `master` branch only
- **Manual Triggers**: Individual workflows can be run manually for debugging
- **Demonstration Focus**: Architecture designed for learning and showcasing CI/CD concepts

## ⚙️ Technical Implementation

### Reusable Workflows

All sub-workflows use `workflow_call` triggers, enabling:
- **Code Reusability**: No duplication across workflows
- **Modular Architecture**: Each workflow has a single responsibility

### Caching Strategy

Unified Gradle cache across all workflows:
```yaml
key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
```

**Benefits:**
- Faster build times
- Reduced resource usage

### Security & Signing

The CD pipeline implements secure APK signing:
- **Base64 Encoded Keystore**: Stored as GitHub secret
- **Environment Variables**: Signing credentials passed securely
- **Secret Inheritance**: `secrets: inherit` for reusable workflows

## 🔧 Setup Instructions

### Required GitHub Secrets

Add these secrets to your repository -> settings -> secrets -> actions -> Repository secrets :

| Secret | Description |
|--------|-------------|
| `KEYSTORE_BASE64` | Base64 encoded release keystore |
| `SIGNING_KEY_ALIAS` | Keystore key alias |
| `SIGNING_KEY_PASSWORD` | Key password |
| `SIGNING_STORE_PASSWORD` | Keystore password |

### Keystore Setup

1. Generate or locate your release keystore
2. Encode to base64:
   ```bash
   # macOS
   base64 -i release.jks | pbcopy
   
   # Linux
   base64 release.jks | xclip -selection clipboard
   
   # Windows PowerShell
   [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("release.jks")) | Set-Clipboard
   ```
3. Add to GitHub Secrets as `KEYSTORE_BASE64`

## 🎯 Workflow Behavior

### Conditional Execution

```yaml
if: success() && github.ref == 'refs/heads/master'
```

- **`success()`**: Only run if CI passed
- **`github.ref == 'refs/heads/master'`**: Only on master branch pushes

### Artifact Management

- **Unit Tests**: Test reports uploaded for 7 days
- **CD Pipeline**: APKs and reports uploaded for 30 days
- **Automatic Zipping**: GitHub Actions automatically creates zip archives

## 🔍 Debugging & Monitoring

### Manual Execution
Individual workflows can be triggered manually:
- Navigate to Actions tab
- Select workflow (including `android-instrumented-tests.yml` for UI testing)
- Click "Run workflow"

> **Instrumented Tests**: Can be run manually via `workflow_dispatch` trigger when comprehensive testing is needed.

### Build Artifacts
APK and Test reports are accessible in the Artifacts section of each workflow run.

## 🚀 Possible Enhancements

Potential improvements for production use:
- **Enable Instrumented Tests**: Uncomment instrumented tests in CI for comprehensive testing
- **Code Coverage**: Add coverage reporting and thresholds
- **Static Analysis**: Integrate linting and security scanning
- **Deployment Targets**: Add Firebase App Distribution or Play Console upload
- **Release Management**: Automated versioning and release notes
- **Performance Testing**: Add UI and performance benchmarks
- **Paid Runner Options**: Consider GitHub Actions paid plans for heavier builds and instrumented tests

---

This **demonstration CI/CD implementation** showcases Android DevOps practices while maintaining simplicity and cost-effectiveness. It provides a foundation that can be extended for production use and serves as a practical learning resource for CI/CD concepts.
