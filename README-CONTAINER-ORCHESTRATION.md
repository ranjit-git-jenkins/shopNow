# Practice Assignment on Container Orchestration — ShopNow

Repository: https://github.com/ranjit-git-jenkins/shopNow

## Scope
This add-on provides Kubernetes deployments and Services for the existing backend, customer frontend, admin dashboard, a demo MongoDB StatefulSet, Helm templates, backend HPA and a Jenkins declarative Groovy pipeline. Keep the original application directories unchanged.

## Grounded application facts
`backend/server.js` uses `PORT` (default 5000), `MONGODB_URI` (default mongodb://localhost:27017/shopnow), and `/api/health`. Its current CORS list only permits localhost:3000 and localhost:3001.

## Prerequisites
- Build and push images named `YOUR_DOCKERHUB_USERNAME/shopnow-{backend,frontend,admin}:latest` (or a unique tag). Replace placeholders in `k8s/*.yaml` or use Helm overrides.
- Docker, kubectl, Helm, a Kubernetes cluster with a default StorageClass, and metrics-server for HPA.
- Jenkins agent with Docker, kubectl, Helm and access to a Kubernetes cluster.
- Jenkins credentials: `dockerhub-credentials` (username/password) and `kubeconfig-shopnow` (secret file).
- Verify actual frontend/admin Dockerfiles expose port 80 and their browser API base URLs before deployment. React build-time variables may require Docker build args.

## Kubernetes deployment (manual)
```sh
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/mongodb.yaml
kubectl apply -f k8s/backend.yaml -f k8s/frontend.yaml -f k8s/admin.yaml
kubectl apply -f k8s/hpa.yaml
kubectl -n shopnow get pods,svc,hpa
kubectl -n shopnow port-forward svc/shopnow-backend 5000:5000
# In another terminal: curl http://localhost:5000/api/health
```

## Helm deployment (recommended alternative to manual manifests)
```sh
helm lint helm/shopnow
helm template shopnow helm/shopnow --namespace shopnow
helm upgrade --install shopnow helm/shopnow -n shopnow --create-namespace \
  --set image.registry=YOUR_DOCKERHUB_USERNAME --set image.tag=latest
kubectl -n shopnow get deployments,statefulsets,svc,hpa
```
Do not deploy the manual manifests and Helm release with the same resource names into the same namespace simultaneously.

## Networking and limitations
The example ingress is optional and requires an ingress controller. Same-origin `/api` routing is recommended because current backend CORS origins are limited to localhost. Browser code must use relative `/api` or compatible production URL; review the React source before enabling ingress. MongoDB in this example is a single-instance **development** database without authentication or backups; not production-ready. Image registry credentials, security hardening, TLS, backup, and production DB architecture are outside this demo.

## Jenkins pipeline
Pipeline builds 3 images, pushes to Docker Hub, and deploys with Helm. Configure Jenkins credentials, Docker permissions, kubeconfig and registry username. Use GitHub webhook or SCM polling for automatic triggers (not configured by this file). Do not store tokens in source control.

## Validation and challenges
- `helm lint` and `helm template` verify template structure; `kubectl get pods`, `describe`, `logs` and `rollout status` verify deployment.
- Common issues: `ImagePullBackOff` (registry/tag), `Pending` MongoDB PVC (StorageClass), failed frontend API calls (browser URL/CORS), HPA unknown metrics (metrics-server), Jenkins Docker permission denied.
- These files are deployment templates. No cluster deployment, Docker image push, Jenkins run, or end-to-end tests are claimed as completed.
