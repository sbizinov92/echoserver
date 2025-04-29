# ⚙️ GitHub Actions Workflows: CI/CD + GitOps for EKS

This repository uses GitHub Actions to automate the entire CI/CD process, from testing and building a Go application, to publishing a Docker image to Amazon ECR, and finally updating a GitOps repository for Kubernetes deployment.

## ⚠️ Disclaimer

> ⚠️ This setup is simplified and intentionally includes some non-production best practices:
>
> - The GitOps repository is self-updated directly from within the workflow using a commit push. While functional, this approach is **not ideal for long-term production environments** due to potential risks of automation loops, commit race conditions, or credential misuse. Tools like [Flux](https://fluxcd.io) with automated image update controllers, or external CD pipelines, are more robust for this use case.
>
> - Vulnerability scanning with Trivy is included, but the scanner configuration is **minimal** and may not cover all desired policies or CVE severities. Additional tuning and integration with policy engines like OPA or AWS Inspector may be necessary.

---

## 🔧 Workflows Overview

### 1. **Continuous Integration**
- **Triggers:** On `push` or `pull_request` to the `main` branch.
- **Steps:**
  - Linting Go code with `golangci-lint`
  - Running unit tests via `make test`
  - Building the application binary with `make build`

---

### 2. **Continuous Delivery – Docker Image**
- **Triggers:**
  - On successful completion of CI
  - On push of a tag like `v*`
  - Manual trigger (`workflow_dispatch`)
- **Steps:**
  - Builds and pushes a Docker image to AWS ECR using Buildx
  - Generates dynamic image tags (`semver`, `sha`, `branch`)
  - Scans the image for vulnerabilities using **Trivy**
  - Uploads scan results to GitHub’s Security tab

---

### 3. **GitOps Repository Update**
- **Triggers:** On successful completion of Docker delivery workflow or manual trigger
- **Steps:**
  - Updates the `charts/echoserver/values.yaml` file
    - Sets the image tag (`sha-<short-sha>` or custom)
    - Sets the image repository to your ECR
  - Commits and pushes the updated `values.yaml` to the GitOps repo


---

## 🚀 How It Works

1. Push code to `main` → triggers **CI**
2. CI passes → triggers **Docker Build & Push**
3. Docker workflow completes → triggers **GitOps Update**
4. Your ArgoCD (or Flux) deployment syncs updated values

---

## 💡 Notes

- Image scanning is handled by [Trivy](https://github.com/aquasecurity/trivy)
- Image metadata is managed using [docker/metadata-action](https://github.com/docker/metadata-action)
- Kubernetes deployments assume use of a Helm chart (`charts/echoserver/values.yaml`)

---

## ✅ Example Use Cases

- CI with static analysis and unit tests
- Automatic Docker publishing on PR merge
- GitOps-friendly updates to Kubernetes YAMLs

---

## 🛠 Dependencies

Ensure the following are in your project:
- `go.mod` and `go.sum` for Go project
- `Dockerfile` in `./cmd/echoserver`
- Helm chart with editable `values.yaml`

---

