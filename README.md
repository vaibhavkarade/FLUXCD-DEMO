Prerequisites:
Kubernetes Cluster: You should have access to a Kubernetes cluster. You can use kubectl to interact with it.
kubectl: Make sure kubectl is installed and configured to access your Kubernetes cluster.
Helm: Helm is a package manager for Kubernetes, which will be used for deploying FluxCD.
GitHub Repository: We’ll use a Git repository to store your manifests. Make sure you have a GitHub account and create a repository for the demo.



Step 1: Install FluxCD
FluxCD can be installed via Helm, kubectl, or using the Flux CLI. In this case, we'll use the Flux CLI for simplicity.

Install Flux CLI: Follow the official documentation to install Flux. You can use the following command for most platforms:

bash
Copy code
curl -s https://fluxcd.io/install.sh | sudo bash
Alternatively, you can manually install it by downloading the latest release from Flux GitHub Releases.

Verify the installation:

bash
Copy code
flux --version
This will give you the installed version of Flux.

Step 2: Set up Flux on Kubernetes
Bootstrap Flux:

Flux needs a Git repository where it can watch for changes. This repository will hold your Kubernetes manifests.

bash
Copy code
flux bootstrap github \
  --owner=<your-github-username> \
  --repository=<your-repository-name> \
  --branch=main \
  --personal
Replace <your-github-username> with your GitHub username and <your-repository-name> with the repository you want Flux to sync with.

This will:

Install Flux into your Kubernetes cluster.
Set up a GitHub repository with Flux's configurations.
Configure Flux to sync changes from this repository.
Verify Flux installation:

To verify that Flux has been installed correctly, you can check the Flux pods:

bash
Copy code
kubectl get pods -n flux-system
You should see Flux components like flux, source-controller, kustomize-controller, etc.

Step 3: Create the Kubernetes Manifest for the Sample App
Prepare your GitHub Repository:

In the GitHub repository that Flux is syncing, create a folder for your manifests (e.g., ./deployments/).

Create the Sample Kubernetes Manifests:

In your repository, under ./deployments/, create a simple Nginx deployment YAML file (nginx-deployment.yaml):

yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
Add a Service for the Nginx Deployment:

Create a service YAML file (nginx-service.yaml) to expose the Nginx app.

yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: default
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
Push the Changes:

Commit and push these files to your GitHub repository:

bash
Copy code
git add .
git commit -m "Add nginx deployment and service"
git push origin main
Step 4: Configure Flux to Sync Your Manifests
Configure Flux to Watch Your Git Repository:

After the initial flux bootstrap, Flux will automatically create the necessary resources to sync with your repository. You can view these configurations in the flux-system namespace.

Check if Flux is syncing properly:

bash
Copy code
kubectl get sources git -n flux-system
Flux will sync every time there’s a change to the repository.

Apply a Kustomization resource:

In the GitHub repository, create a kustomization.yaml file in the same directory as your deployment and service manifests (e.g., ./deployments/kustomization.yaml):

yaml
Copy code
apiVersion: fluxcd.io/v1
kind: Kustomization
metadata:
  name: nginx-app
  namespace: flux-system
spec:
  interval: 10m
  path: "./deployments"
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
Commit and push the changes.

Step 5: Check Deployment Status
Verify the Nginx deployment:

After Flux syncs your repository, it will automatically deploy the application to the Kubernetes cluster. You can verify it by checking the deployment and the service:

bash
Copy code
kubectl get deployments
kubectl get services
Check if Nginx is running:

You can check if the Nginx pods are running:

bash
Copy code
kubectl get pods
And check if the service is accessible (assuming you have port forwarding or an ingress set up):

bash
Copy code
kubectl port-forward service/nginx 8080:80
Then access http://localhost:8080 in your browser to see the Nginx welcome page.
