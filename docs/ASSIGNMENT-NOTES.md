# Submission notes

Project: Practice Assignment on Container Orchestration

GitHub repository: https://github.com/ranjit-git-jenkins/shopNow

Prepared artifacts: k8s manifests, Helm chart, Jenkins Groovy pipeline, configuration and validation guide.

Known backend facts from provided server.js: port 5000; MONGODB_URI; health path /api/health; CORS allows localhost 3000/3001 only.

Pending verification: actual Docker builds, frontend/admin port and API URL behavior, Kubernetes cluster deployment, Jenkins execution, Helm lint on an installed Helm binary, image pushes, screenshots.

The existing app code is preserved; review API URL and CORS before running the UI in Kubernetes.
