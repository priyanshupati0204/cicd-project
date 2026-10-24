# CI/CD Pipeline: Flask App on Jenkins, Docker Hub and Minikube

## Project
A small Flask web app (`app.py`) with one pytest test (`test_app.py`).
The focus of this project is the automated delivery pipeline.

## Pipeline stages (Jenkinsfile)
1. **Checkout**: Jenkins pulls the latest code from GitHub.
2. **Build and Test**: creates a virtual environment, installs `requirements.txt` and runs `pytest`.
3. **Docker Build**: builds the image from the Dockerfile, tagged with the build number.
4. **Push Image**: logs in to Docker Hub with credentials stored in Jenkins and pushes the image.
5. **Minikube Deployment**: puts the build number into `k8s/deployment.yaml` and applies the Deployment and Service.
6. **Verification**: runs `kubectl get pods`, `deployment` and `svc`.

## Infrastructure
- AWS EC2 instance (Amazon Linux 2023, c7i-flex.large, 20 GB disk)
- Installed on it: Docker, Java 21, Jenkins, kubectl, Minikube, Git
- Security group ports: 22 (SSH), 8080 (Jenkins), 30080 (app)

## Jenkins setup
- Plugins: Git, Pipeline, Docker Pipeline
- Credential `dockerhub-creds`: Docker Hub username and access token
- Pipeline job `cicd-pipeline`: Pipeline script from SCM, branch `main`, script path `Jenkinsfile`
- The `jenkins` user was added to the `docker` group and given access to the Minikube config

## Image registry
Image: `priyanshu0204/cicd-project:<build-number>` on Docker Hub.

## Deployment and access
- `k8s/deployment.yaml`: 2 replicas of the app
- `k8s/service.yaml`: NodePort service on port 30080
- The app is exposed with `kubectl port-forward` and opened at `http://<EC2-public-IP>:30080`

## Second run (code change)
Changed the heading in `app.py` from "Version 1" to "Version 2", pushed to GitHub and ran the pipeline again. A new image tag was pushed, the pods were recreated, and the browser showed Version 2.