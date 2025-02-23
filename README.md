# Deploying an Application on Kubernetes with Argo CD on WSL Ubuntu

This guide will walk you through the process of installing Kubernetes on WSL Ubuntu, cloning a GitHub repository, creating the necessary files, and using Argo CD to manage the automatic deployment of your application.

---

## 🔹 1. Clone your GitHub repository
If you don't have a repository yet, create one on GitHub. Then, clone it to your machine:

git clone https://github.com/YOUR-USER/YOUR-REPO.git
cd YOUR-REPO
## 🔹 2. Create the file structure

### Inside your repository, create a dev/ folder with two YAML files and an application.yml file at the root.

👉 mkdir dev

👉 touch dev/deployment.yaml dev/service.yaml application.yml



## Content of dev/deployment.yaml (Example of a Deployment):


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: nginx
        ports:
        - containerPort: 80
``` 

## Explanation:

### Deployment: Defines how your application runs within Kubernetes.
### replicas: 1: Indicates that one instance will be running.
### containers: Defines the containers within the pod, in this case, using the nginx image.



## Content of dev/service.yaml (Example of a Service):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
``` 

## Explanation:

### Service: Exposes the deployment within the cluster.
### ClusterIP: Makes the service accessible internally in Kubernetes.



## Content of application.yml (for Argo CD):


```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/YOUR-USER/YOUR-REPO.git'
    path: dev
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
``` 


## Explanation:

### Argo CD controls this repository and automatically applies changes to Kubernetes.
### repoURL: Defines the GitHub repository URL containing the YAML files.
### path: Indicates the folder in the repository where the manifests (dev) are stored.
### syncPolicy.automated: Ensures Argo CD applies changes automatically.


## 🔹 3. Install Argo CD

### Run these commands in WSL to install Argo CD:

👉 kubectl create namespace argocd
👉 kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


### 🔹 4. Verify that the pods are running:

👉 kubectl get pods -n argocd


## 🔹 5. Expose Argo CD UI


### To access the Argo CD UI in your browser, run:

👉 kubectl port-forward svc/argocd-server -n argocd 8080:443

### Then open it in your browser:

 👉 https://localhost:8080


### 🚧 To get the admin user password:

👉 kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d


## 🔹 6. Create the Application in Argo CD

### Log in to Argo CD from the terminal:

👉 argocd login localhost:8080 --username admin --password YOUR-PASSWORD

### Create the application:

👉kubectl apply -f application.yaml

### Or manually:

👉 
argocd app create my-app \
  --repo https://github.com/YOUR-USER/YOUR-REPO.git \
  --path dev \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated



## 🔹 7. Delete the project

👉 kubectl delete namespace argocd
👉kubectl delete namespace default
