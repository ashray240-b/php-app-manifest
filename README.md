# End-to-End DevOps CI/CD Pipeline with Jenkins & Argo CD

A GitOps-based CI/CD pipeline: Jenkins handles CI (build, quality checks, security scanning, image publishing), and Argo CD continuously syncs the desired state from a Git-stored manifest repo to the Kubernetes cluster.

> **Note:** This project intentionally reuses the same Blood Donation Management application from the [PHP Blood Donation CI/CD project](../blood-donation-cicd) as its base. The goal here wasn't to build a new app — it was to compare two different deployment models for the same application: a direct Docker deployment (Project 2) vs. a GitOps-based deployment to Kubernetes via Argo CD (this project). This let me evaluate trade-offs like declarative state management, drift detection/self-healing, and rollback behavior that direct Docker deployment doesn't offer.

## Architecture / Pipeline Flow

**CI Pipeline (Jenkins)**
- Stage 0: Clean Workspace
- Stage 1: Checkout source code from GitHub
- Stage 2: Code Quality Analysis (SonarQube)
- Stage 3: Quality Gate Validation
- Stage 4: Docker Image Build
- Stage 5: Security Scan (Trivy)
- Stage 6: Push Docker Image to Docker Hub

**CD Pipeline (GitOps via Argo CD)**
- Kubernetes manifest files stored in a separate GitHub repository
- Image tag updated in the manifests after each build
- Argo CD continuously monitors the manifest repository
- Argo CD automatically syncs changes and deploys to the Kubernetes cluster
- Deployment status and health monitored via the Argo CD dashboard

```
GitHub (app code) → Jenkins CI (Stages 0-6) → Docker Hub
                                                    │
                                    (image tag update)
                                                    ▼
                          GitHub (manifests repo) → Argo CD → Kubernetes Cluster
```
*(Replace with an actual diagram image for a stronger visual.)*

## Tech Stack

| Category | Tools |
|---|---|
| Version Control | Git, GitHub |
| CI Orchestration | Jenkins |
| Code Quality | SonarQube |
| Containerization | Docker |
| Security Scanning | Trivy |
| Image Registry | Docker Hub |
| Orchestration | Kubernetes |
| GitOps / CD | Argo CD |

## Repository Structure

```
├── app-repo/
│   ├── Jenkinsfile
│   ├── Dockerfile
│   └── src/
├── manifests-repo/            # Separate GitOps repo
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── argocd/
    └── application.yaml       # Argo CD Application definition
```
*(Update to match your actual repo layout — note CI code and GitOps manifests are typically kept in separate repos, as reflected above.)*

## How to Run

1. **Prerequisites**: Kubernetes cluster with Argo CD installed; Jenkins with Docker access; a dedicated GitHub repo for manifests.
2. Register the app with Argo CD:
   ```
   kubectl apply -f argocd/application.yaml
   ```
3. Jenkins CI pipeline (Stages 0–6) builds, scans, and pushes the image, then updates the image tag in the manifests repo.
4. Argo CD detects the change and syncs automatically (or manually: `argocd app sync <app-name>`).
5. Verify: `kubectl get pods` and check sync status in the Argo CD dashboard.

## Why This Project Is Valuable

Demonstrates the complete modern software delivery lifecycle:
- CI automation with Jenkins
- Static code quality analysis and quality gates
- Automated container security scanning
- Containerization
- GitOps-based continuous deployment (Git as the single source of truth)
- Kubernetes application deployment and monitoring via Argo CD

**Docker deployment (Project 2) vs. GitOps/Argo CD deployment (this project):**

| | Project 2 (Docker) | Project 5 (Argo CD) |
|---|---|---|
| Deployment trigger | Jenkins runs `docker` commands directly | Argo CD watches Git and auto-syncs |
| Source of truth | Pipeline script | Kubernetes manifests in Git |
| Drift handling | None — manual re-deploy needed | Auto-detected and self-healed |
| Rollback | Manual | Git revert / Argo CD history |
| Target | Single Docker host | Kubernetes cluster |

## Screenshots
