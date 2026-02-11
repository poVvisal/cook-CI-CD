# Cook CI/CD – Node/Express Demo

This repository contains a minimal Node/Express web server that powers a CI/CD tutorial. The app exposes a single `GET /` route that says hello, while the automation around it demonstrates how to ship containerized workloads to Google Cloud Run using GitHub Actions.

## Architecture At A Glance
- **Runtime:** Node.js + Express with CORS enabled
- **Testing:** Jest + Supertest request specs
- **Containerization:** Docker image built via GitHub Actions
- **Deployment target:** Google Cloud Run (managed) with staging + production revisions
- **Automation:** `.github/workflows/cd-pipeline.yml` runs tests, builds, pushes, and deploys

## Prerequisites
- Node.js 18+ and npm
- Docker (for local image builds)
- gcloud CLI (for manual Cloud Run deploys)
- A Google Cloud project with Cloud Run + Artifact Registry (or Docker Hub) access
- GitHub repository secrets for service-account JSON and registry credentials

## Getting Started
```bash
git clone https://github.com/onukwilip/ci-cd-tutorial.git
cd ci-cd-tutorial
npm install
npm start
```
The server listens on `PORT` (defaults to `5000`) and responds at `http://localhost:5000/` with a simple HTML page.

## Available Scripts
| Command      | Description                          |
|--------------|--------------------------------------|
| `npm start`  | Runs `index.js` and boots Express     |
| `npm test`   | Executes the Jest + Supertest suite   |

## Running Tests
```bash
npm test
```
The suite validates that `/` returns `200` and the response body mentions “web server,” ensuring the route stays healthy during refactors.

## Environment Variables
| Name | Default | Purpose |
|------|---------|---------|
| `PORT` | `5000` | Defines the listening port for Express |

You can create a `.env` file (ignored by Git) to override defaults locally. The CI pipeline injects the same variables via workflow `env` blocks.

## CI/CD Pipeline
The workflow in `.github/workflows/cd-pipeline.yml` orchestrates two jobs:
1. **test** – installs dependencies with `npm ci` and runs the Jest suite on Ubuntu runners.
2. **build** – authenticates to Google Cloud + Docker Hub, builds the image (`docker build -t $IMAGE .`), pushes it, and deploys to Cloud Run.

Deployment behavior:
- Every push to `staging` (or non-`main` refs) triggers the staging deploy step (`--tag staging`).
- Publishing a GitHub Release whose target commit is on `main` triggers the production deploy step (`--tag production`).
- Cloud Billing API enablement is automated before deploying to avoid first-time project issues.

Ensure the following GitHub secrets/variables exist:
- `GCP_SERVICE_ACCOUNT` – JSON key for a service account with Cloud Run + Artifact Registry + Billing permissions.
- `GCP_PROJECT_ID`, `GCR_REGION`, `GCR_STAGING_PROJECT_NAME`, `GCR_PROJECT_NAME` – used for gcloud deploy commands.
- `DOCKER_USER`, `DOCKER_PASSWORD`, and `IMAGE` (or adjust the workflow to point directly at Artifact Registry).

## Manual Deployment (optional)
```bash
docker build -t gcr.io/<PROJECT_ID>/ci-cd-tutorial-app:latest .
gcloud run deploy gcr-ci-cd-app \
	--image gcr.io/<PROJECT_ID>/ci-cd-tutorial-app:latest \
	--region us-central1 \
	--platform managed \
	--allow-unauthenticated
```

## Project Status & Roadmap
- ✅ Express baseline, health tests, GitHub Actions workflow, Cloud Run deployment
- 🛠️ Potential next steps: add API endpoints, integrate datastore, expand automated tests, wire up monitoring dashboards

## License
Licensed under the ISC License. See `package.json` for details.3