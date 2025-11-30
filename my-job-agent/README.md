# my-job-agent

This template should help get you started developing with Vue 3 in Vite.

## Recommended IDE Setup

[VS Code](https://code.visualstudio.com/) + [Vue (Official)](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Recommended Browser Setup

- Chromium-based browsers (Chrome, Edge, Brave, etc.):
  - [Vue.js devtools](https://chromewebstore.google.com/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd) 
  - [Turn on Custom Object Formatter in Chrome DevTools](http://bit.ly/object-formatters)
- Firefox:
  - [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)
  - [Turn on Custom Object Formatter in Firefox DevTools](https://fxdx.dev/firefox-devtools-custom-object-formatters/)

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```

## 🚀 GCP Cloud Run Deployment

### Prerequisites

1. Google Cloud SDK installed and authenticated
2. A GCP project with billing enabled
3. Cloud Run, Cloud Build, and Artifact Registry APIs enabled

### One-Time Setup

```bash
# Set your project ID
export PROJECT_ID=your-project-id
export REGION=asia-southeast2

# Enable required APIs
gcloud services enable cloudbuild.googleapis.com run.googleapis.com artifactregistry.googleapis.com

# Create Artifact Registry repository
gcloud artifacts repositories create myjobagent \
  --repository-format=docker \
  --location=$REGION \
  --description="MyJobAgent Docker images"

# Grant Cloud Build permission to deploy to Cloud Run
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')@cloudbuild.gserviceaccount.com" \
  --role="roles/run.admin"

gcloud iam service-accounts add-iam-policy-binding \
  $(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')-compute@developer.gserviceaccount.com \
  --member="serviceAccount:$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')@cloudbuild.gserviceaccount.com" \
  --role="roles/iam.serviceAccountUser"
```

### Option 1: Connect Repository for Auto-Deploy (Recommended)

1. Go to [Cloud Build Triggers](https://console.cloud.google.com/cloud-build/triggers)
2. Click **Connect Repository** and select your GitHub repo
3. Create a trigger:
   - **Name**: `myjobagent-frontend-deploy`
   - **Event**: Push to branch `main`
   - **Source**: Select your repo
   - **Configuration**: Cloud Build configuration file: `my-job-agent/cloudbuild.yaml`
   - **Substitution variables** (optional override):
     - `_VITE_API_BASE_URL`: Your backend URL

### Option 2: Manual Deploy with gcloud

```bash
# Navigate to frontend directory
cd my-job-agent

# Build and deploy using Cloud Build
gcloud builds submit --config=cloudbuild.yaml \
  --substitutions=_VITE_API_BASE_URL="https://myjobmatch-backend-355671284742.asia-southeast2.run.app"
```

### Option 3: Direct Docker Build & Deploy

```bash
# Build Docker image locally
docker build \
  --build-arg VITE_API_BASE_URL="https://myjobmatch-backend-355671284742.asia-southeast2.run.app" \
  -t myjobagent-frontend .

# Tag and push to Artifact Registry
docker tag myjobagent-frontend $REGION-docker.pkg.dev/$PROJECT_ID/myjobagent/myjobagent-frontend
docker push $REGION-docker.pkg.dev/$PROJECT_ID/myjobagent/myjobagent-frontend

# Deploy to Cloud Run
gcloud run deploy myjobagent-frontend \
  --image=$REGION-docker.pkg.dev/$PROJECT_ID/myjobagent/myjobagent-frontend \
  --region=$REGION \
  --platform=managed \
  --allow-unauthenticated
```

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `VITE_API_BASE_URL` | Backend API URL | `https://myjobmatch-backend-355671284742.asia-southeast2.run.app` |

## 📁 Project Structure

```
my-job-agent/
├── src/                    # Vue source code
├── public/                 # Static assets
├── Dockerfile              # Container build config
├── nginx.conf              # Nginx config for serving SPA
├── cloudbuild.yaml         # Cloud Build CI/CD config
├── .env.example            # Environment variables template
└── package.json            # Node.js dependencies
```
