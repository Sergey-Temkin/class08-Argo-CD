# ArgoCD-GitOps


## Project Structure:

```
class08/
├── .github/
│   └── workflows/
│       └── docker-build.yaml      # GitHub Actions workflow for Docker builds
│
├── mychart/                       # Helm chart folder
│   ├── templates/                 # Holds template YAML files (not expanded here)
│   ├── Chart.yaml                 # Metadata about the Helm chart
│   ├── values-image.yaml          # Custom values file for image configs
│   └── values.yaml                # Default values file for the chart
│
├── app.py                         # Python application file
├── Dockerfile                     # Docker build instructions
├── example.yaml                   # Example Kubernetes YAML (maybe Deployment/Config)
├── requirements.txt               # Python dependencies
└── svc.yaml                       # Kubernetes Service manifest
```


## In: "docker-build.yaml" - Add name of image  
```
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/arm64
          push: true
          tags: "${{ secrets.DOCKER_USERNAME }}/<name of image>:v0.${{ github.run_number }}"
```



## In: "values.yaml - Add the username in DockerHub and name of image 
```
image:
  repository: <DockerHub-username>/<name of image>
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "0.1"

```

## After completion of adding all files , create new repository in GitHub
```
Go to settings --> Secrets and variables --> Actions --> New repository secret:
Name: 
DOCKER_USERNAME
Secret: 
<DockerHub username>
Add secret
```

```
In DockerHub --> Account settings(Push the logo on the right upper side) --> Personal access tokens --> Generate new token
Access token description:
argocd-class-8
Access permissions:
Read,Write,Delete
Generate
Copy from: "At the password prompt, enter the personal access token"
<copy the token>
```

```
Back in GitHub go to settings --> Secrets and variables --> Actions --> New repository secret:
Name:
DOCKER_PASSWORD
Secret:
< the copy'd token from DockerHub>
Add secret
```
## In VsCode
```
git init
git add .
git commit -m " "
git remote add origin https://github.com/Sergey-Temkin/class08-Argo-CD.git
git branch -M main
```
```
minikube start -p class08 --driver=docker --cpus=2 --memory=2200 --disk-size=10g
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get all -n argocd
kubectl get pods -n argocd
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" -n argocd| base64 -d; echo
Copy the secret key: <password>
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## To Kill port-forward and start it again:
```
ss -ltnp '( sport = :8080 )'
kill -9 <number of pid>
```

## Open browser and go to:    
```
https://localhost:8080/applications
```
```
User:
admin
Password:
<password>  (The password you generated above in VsCode)
```

## Create new app:
```
Push:"New app"
Application Name:class08
Project name:default
Sync Policy:Manual

SOURCE:
https://github.com/Sergey-Temkin/class08-Argo-CD.git --> GIT
Revision:HEAD
Path: ./

DESTINATION:
Cluster URL:
https://kubernetes.default.svc
Namespace:default

Create
SYNC --> Synchronize
```
## In WSL check Application status:
```
kubectl get applications -A
```

## To upload new APP using .YAML file in Argo-CD:

NEW APP --> EDIT AS YAML(On the right top side button) --> And paste in it: (Notice that repoURL is your working repository)
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myrelease
spec:
  project: default
  source:
    repoURL: https://github.com/Sergey-Temkin/class08-Argo-CD.git
    path: ./mychart
    targetRevision: HEAD
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```     
Save --> create 

