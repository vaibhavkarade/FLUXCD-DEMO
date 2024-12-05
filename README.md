Prerequisites:
Kubernetes Cluster: A running Kubernetes cluster (either local using minikube, or on a cloud provider like AWS, GKE, or Azure).
kubectl: CLI tool to interact with your Kubernetes cluster.
FluxCD: A GitOps tool to synchronize Kubernetes with your Git repository.
GitHub/Other Git Repo: A Git repository containing Kubernetes manifests (YAML files) for your applications.

Fluxcd works on pull mechanism where as argocd works on push mechanism and both are gitops tools used for deployment.


step 1) install fluxctl
curl -s https://fluxcd.io/install.sh | sudo bash


step2)install flux 
flux install
This will install FluxCD components like the controller and the source controller in your cluster.


step3)  Create a Git Repository with Kubernetes Manifests
To set up GitOps with FluxCD, you need a Git repository where your Kubernetes manifests are stored. Create a new Git repository or use an existing one, and add some simple Kubernetes manifests. Here's an example structure:

my-app-repo/
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml


Step4) Connect Flux to Your Git Repository
In Flux, you need to create a GitRepository custom resource that points to your Git repository where the manifests are stored.

Create a file named gitrepository.yaml:

apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: demo-repo
  namespace: flux-system
spec:
  interval: 10m0s
  url: https://github.com/your-user/your-repo-name
  ref:
    branch: main

kubectl apply -f gitrepository.yaml



Step5)  Create a Kustomization Resource
Next, create a Kustomization resource that tells Flux how to apply the manifests from the Git repository.

Create a file named kustomization.yaml:


apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: demo-app
  namespace: flux-system
spec:
  interval: 10m0s
  path: "./k8s"
  prune: true
  sourceRef:
    kind: GitRepository
    name: demo-repo
  targetNamespace: default


kubectl apply -f kustomization.yaml


Step6) You can also check the status of the resources Flux is managing:
kubectl get gitrepositories -n flux-system
kubectl get kustomizations -n flux-system
kubectl get deployments


Step7) testing the fluxcd
5. Test FluxCD Detecting Changes
Now, let's test if FluxCD detects changes made to the Git repository.

a. Make Changes to Your Git Repository
Go to your Git repository and make a change to the Kubernetes manifests. For example, modify the deployment.yaml to use a different image:
        image: nginx:1.21
