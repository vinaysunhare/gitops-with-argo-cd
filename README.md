![Screenshot from 2025-05-19 08-32-01](https://github.com/user-attachments/assets/a5cf4203-0a40-4860-87a8-cd782f5ec427)
![Screenshot from 2025-05-19 08-48-53](https://github.com/user-attachments/assets/9c6ca530-0e6a-483d-867b-45c5c5210b36)
![Screenshot from 2025-05-19 09-42-51](https://github.com/user-attachments/assets/d02bff37-d157-441f-aa0b-52083d91fa2d)
![Screenshot from 2025-05-19 09-43-41](https://github.com/user-attachments/assets/2bc5fe46-45bb-4dcb-bcbf-14d78999407d)
![Screenshot from 2025-05-19 11-21-23](https://github.com/user-attachments/assets/338926aa-fa00-4662-9101-42ca28fb898c)
![Screenshot from 2025-05-19 11-21-39](https://github.com/user-attachments/assets/7a18280e-c35a-4c53-9211-4ed4ea4d5acb)



# Manufacturing Co. Website with GitOps and Argo CD

## Description

A step-by-step guide to containerize and deploy a static manufacturing website using Docker and Kubernetes, and automate continuous delivery with GitOps principles via Argo CD. Perfect for learning modern infrastructure-as-code and continuous deployment workflows.

## Overview

This repository demonstrates how to containerize a static HTML/CSS manufacturing website, deploy it to a local Kubernetes cluster (Minikube/Kind), and automate continuous delivery using GitOps principles with Argo CD.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Project Structure](#project-structure)
3. [Local Development with Docker](#local-development-with-docker)
4. [Kubernetes Deployment](#kubernetes-deployment)
5. [Setting Up Argo CD](#setting-up-argo-cd)
6. [GitOps Workflow](#gitops-workflow)
7. [Updating the Application](#updating-the-application)
8. [Troubleshooting](#troubleshooting)

---

## Prerequisites

* [Docker](https://www.docker.com/) installed and running
* [kubectl](https://kubernetes.io/docs/tasks/tools/) configured for Minikube or Kind
* [Minikube](https://minikube.sigs.k8s.io/docs/) or [Kind](https://kind.sigs.k8s.io/) cluster up
* [Argo CD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/) (optional, for CLI interactions)
* GitHub account for hosting manifests

---

## Project Structure

```
manufacturing-site/
├── index.html         # Static website
├── styles.css         # Site styles
├── Dockerfile         # Container build definition
├── k8s-web-deploy.yaml# Kubernetes Deployment & Service
└── manufacturing-site-app.yaml # Argo CD Application manifest
```

---

## Local Development with Docker

1. Build the Docker image:

   ```bash
   docker build -t <dockerhub-username>/manufacturing-site:1.0 .
   ```
2. Push to Docker Hub:

   ```bash
   docker login
   docker push <dockerhub-username>/manufacturing-site:1.0
   ```
3. Run locally on port 7080 (if 8080 is occupied):

   ```bash
   docker run -d -p 7080:80 <dockerhub-username>/manufacturing-site:1.0
   ```
4. Visit: [http://localhost:7080/](http://localhost:7080/)

---

## Kubernetes Deployment

1. Ensure cluster is running:

   ```bash
   minikube start --memory=4096 --cpus=2
   ```
2. Apply the Deployment & Service manifest:

   ```bash
   kubectl apply -f k8s-web-deploy.yaml
   ```
3. Verify pods & service:

   ```bash
   kubectl get pods -l app=mfg-site
   kubectl get svc mfg-site-svc
   ```
4. Access the service:

   ```bash
   minikube service mfg-site-svc --url
   ```

---

## Setting Up Argo CD

1. Create namespace and install:

   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```
2. Expose the Argo CD API server:

   ```bash
   kubectl port-forward svc/argocd-server -n argocd 6080:443
   ```
3. Retrieve the initial admin password:

   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```
4. (Optional) Install Argo CD CLI:

   ```bash
   curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
   chmod +x argocd && sudo mv argocd /usr/local/bin/
   ```

---

## GitOps Workflow

### Create an Argo CD Application

Using CLI:

```bash
argocd login localhost:6080 --username admin --password <password> --insecure
argocd app create manufacturing-site \
  --repo https://github.com/<username>/gitops-with-argo-cd.git \
  --path . \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

Or apply the `manufacturing-site-app.yaml` manifest:

```bash
kubectl apply -f manufacturing-site-app.yaml
```

1. The application will sync the `deploy` folder (or root) automatically.
2. Changes pushed to `main` will be detected and deployed by Argo CD.

---

## Updating the Application

1. Update `index.html` or `styles.css` locally.
2. Bump the Docker image tag in `k8s-web-deploy.yaml` (e.g., `:v2`).
3. Build and push new image:

   ```bash
   docker build -t <dockerhub-username>/manufacturing-site:v2 .
   docker push <dockerhub-username>/manufacturing-site:v2
   ```
4. Commit changes and push to GitHub:

   ```bash
   git add .
   git commit -m "Bump image to v2 with updated HTML/CSS"
   git push origin main
   ```
5. In Argo CD UI or CLI, sync the application to pull the new manifest and rollout.

---

## Troubleshooting

* **Minikube API server errors**: `minikube stop && minikube delete && minikube start`
* **Port conflicts**: Use a different host port mapping.
* **Argo CD sync issues**: Check logs:

  ```bash
  kubectl logs deployment/argocd-server -n argocd
  ```

---

Happy deploying! 🚀
